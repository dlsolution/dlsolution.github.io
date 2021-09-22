---
layout: post
title: "iOS Location Tracking"
author: "Linh Vo"
tags: "UIKit"
---

If you are building a location-based application from scratch, this short summary presents a few caveats for Location Tracking in iOS which might save you some time.

Recently, here at Badoo, I had to add some new features to the existing Location Update Service, which is used in one of our products. The task turned out to be slightly more than adding a few methods and associated tests for them.

# Goal

Report the user’s location at least every X minutes.

Of course, there are many other subtle requirements, which are very domain specific and therefore omitted for simplicity.

# Problem

> iOS application cannot be woken up by a timer.

This fact is super confusing for product managers and newcomers to the iOS environment. I believe Apple did this to prevent unconstrained work in the background which would drain the battery.

The existing implementation was relying on significant location changes and visit events. There was no GPS usage in the background. It was not sufficient to solve the problem.

# Solution

- Use GPS to determine a precise location.

- Fallback to imprecise location services for battery conservation.

- Send silent push notifications, if a user does not move.

- Extract Location Tracking and Sending settings.

The solution is a combination of the approaches mentioned above. I like simple and stupid designs, and this is the easiest solution that meets the desired goal.

{% include image.html
img="assets/2017-04-25/image.jpg" %}

## GPS usage

We assume GPS is used when we call ‘startUpdatingLocation’ with the appropriate ‘desiredAccuracy’. GPS is utilised for the duration specified in the GPSTracking Settings. After that GPS is turned off, locations are sent to the server and the Imprecise Tracking is turned on.

> Don’t use GPS for any longer than you need.

## Imprecise location services

There are 3 APIs for battery-efficient location tracking:

- Significant location changes — the location is delivered approximately every 500 metres (usually up to 1 km)

- Region monitoring — track enter/exit events from circular regions with a radius equal to 100m or more.

- Visit events — monitor place Visit events which are enters/exits from a place (home/office).

The most precise among them is region monitoring: when GPS is turned off, I set a 100m region around the last known location. Exit from this region is the most common reason why the app wakes up, at which point GPS is re-enabled if needed. Otherwise, a new region is set up.

> Region monitoring is the most precise API after GPS.

## Silent push notifications

If a user goes to a place and sits there like a duck without triggering a region exit or receiving a new visit event, we send a silent push notification from the server. A silent push notification is like a regular notification, but the only payload it has is ‘content-available: 1’. Such notification wakes up the app and gives it a time to kick off the GPS-tracking/location sending.

> Use Silent push notifications as a last resort.

## Location Tracking and Sending settings

It is quite natural to extract configurations to a separate object rather than having them hardcoded. There are two configurations: one for foreground and a second one for the background. They have same fields, only their values are different. The next step for us is to send those configurations from the server.

> Avoid using different logic for the background and foreground: just use different settings.

# Making sure the solution is sufficient

Our product and QA people were concerned with potential battery drainage due to the need to use GPS in the background and wake up the app frequently. They did not want to waste time on an unacceptable solution. So I had to persuade them that other applications do the same.

My colleague advised that I should dispel the concern by checking what other top iOS location-based applications are doing.
I ended up using a jailbroken device to add some runtime hooks to those apps, so every time any callback from `CLLocationManager` was called, I saw an appropriate local push notification. Additionally, Charles can be used to check when locations are sent to their servers.

This is what a hook for visits might look like:

```swift
- (void)locationManager:(CLLocationManager *)manager
     didVisit:(CLVisit *)visit {
     %log; // log hook call
     if ([visit.departureDate isEqual: [NSDate distantFuture]]) {
         sendNotification(@”didVisit: We arrived somewhere!”);
     } else {
         sendNotification(@”didVisit: We left somewhere!”);
     }
     %orig; // this calls original implementation
}
```

> Don’t hesitate to use all available tools to validate ideas.

This investigation confirmed my assumption that top apps are using GPS in the background and receive silent pushes from a server. I was good to proceed with the implementation.

# The implementation of the solution

We ended up with LocationUpdateService which is a mediator for multiple micro services, one of which is LocationTracker.

The class hierarchy looks like this:

LocationUpdateService:

- LocationTracker

- LocationStorage

- LocationSender

LocationTracker is the facade for `CLLocationManager`. The tracker is started with certain LocationTrackingSettings. Among other things, the settings control for how long GPS is used before falling back to imprecise location tracking.

```swift
self.gpsStopTimer = self.timerType.oneShotTimer(withTimeInterval:
    self.settings.gpsTracking.duration) { [weak self] in
    self?.timerFired()
}
```

Instead of diving into implementation details I want to point out the main caveats you have to be aware of when working with ‘CLLocationManager’.

# Caveats when using GPS in the background

An application which has appropriate permissions (to use location service in the background) is woken up whenever any location-related event occurs. The problem is, that if you start any asynchronous process like sending locations to your server, the app is likely to be in the suspended state by the time the app has to handle the response. In other words, the app is suspended almost immediately after callback.

We ended up using ‘UIBackgroundTask’ while we are doing any work, and stopping the task as soon as the work is finished. It is important to remember that the time given for any background task is limited to 3 minutes, and you must use the expiration handler appropriately.

> Use ‘UIBackgroundTask’ for updating tracker state/sending locations.

# Other ‘CLLocationManager’ fun facts

### Significant location changes are delivered using the same delegate callback method as locations from GPS

In `didUpdateLocations` callback there is no way to check if the location comes from GPS or from a significant location change.

As a workaround, it is possible to filter imprecise locations delivered due to significant location change. In our case, we needed additional filtering, and so this issue did not bother us at first.

> Filter locations if needed.

```swift
func locationSignificantlyChanged(_ location: CLLocation) -> Bool {
    guard let filteringSettings = self.settings.locationsFiltering,
        let lastTrackedLocation = self.lastTrackedLocation else {
        self.lastTrackedLocation = location
        return true
    }
    ...
    if timeChangedSignificantly || accuracyImprovedSignificantly
        || distanceChangedDramatically
        || (distanceChangedSignificantly && !accuracyDecreased) {
        self.lastTrackedLocation = location
        return true
    }
    return false
 }
```

### Starting significant location changes might return you a location that is already a few hours old.

Due to this fact we end up using one instance of `CLLocationManager` for GPS tracking and another one for imprecise tracking.

```swift
public func locationManager(manager: CLLocationManager,
    didUpdateLocations locations: [CLLocation]) {
    guard manager == self.locationManager || self.state ==
       .impreciseTracking else {
       return
    }
    if self.state == .impreciseTracking {
       self.updateTracking()
    }
    let filteredLocations =   locations.filter
        (self.locationSignificantlyChanged)
    guard filteredLocations.count > 0 else { return }
    self.delegate?.locationTracker(self, didUpdateLocations:
        filteredLocations)
}
```

### Region-related events are delivered by all instances of ‘CLLocationManager’

Regions are considered to be shared resources, so if you use multiple instances of `CLLocationManager` and one delegate for all of them, you must make sure that you handle callbacks appropriately.

```swift
public func locationManager(manager: CLLocationManager,
    didExitRegion region: CLRegion) {
    guard manager == self.impreciseLocationManager else {
       return
    }
    self.updateTracking()
    self.delegate?.locationTracker(self, didExitRegion: region)
 }
```

> Use two instances of ‘CLLocationManager’.

### Disabling significant location changes tracking results in a decreased amount of visit events being delivered

We figured this out when we were conducting our regular manual QA benchmark. Now we are monitoring significant location changes tracking at all times even when GPS is turned on.

### Misleading CLVisit.arrivalDate and CLVisit.departureDate date

We had an Objective-C code that was a few years old:

```swift
NSDate *dateToUseAsStorageKey = clVisit.arrivalDate ||
   clVisit.departureDate;
```

They cannot be ‘nil’, but instead, they can be `Date.distantPast()` and `Date.distantFuture()` appropriately, and the only way to know this is to read the documentation.

For me this is an example of a poorly designed API, and I prefer to be able to make assumptions simply by a reading public API.

> Write clear API, instead of explaining it in comments or redundant documentation.

# Results

We got 3 times more location coordinates sent to the server from most of the users.

{% include image.html
img="assets/2017-04-25/image1.png" %}

We can request the user’s location at any time using a silent push.

Battery consumption increased from 1–2% to 2–4% during our manual QA benchmark. We are still below the top location-based application in the Settings->Battery screen during normal usage.

Also, we are capable of decreasing battery load by tweaking GPSTracking settings.

Original Post: [https://medium.com/bumble-tech/ios-location-tracking-aac4e2323629](https://medium.com/bumble-tech/ios-location-tracking-aac4e2323629)
