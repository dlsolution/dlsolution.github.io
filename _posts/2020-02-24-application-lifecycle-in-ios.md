---
layout: post
title: "Application life cycle in iOS"
author: "Linh Vo"
tags: "UIKit"
---

The application life cycle constitutes the sequence of events that occurs between the launch and termination of application.

It is very important to understand for all the iOS Developers, who wants smooth user experience.

## Execution States for Apps

- **Not Running state** : The app has not been launched or terminated by the system.

- **Inactive state** : The app is entering the foreground state but not receiving events.

- **Active state** : The app enters the foreground state and can process event.

- **Background state** : In this state, if there is executable code, it will execute and if there is no executable code or the execution is complete, the application will be suspended immediately.

- **Suspended state** : The app is in the background(in memory) but is not executing code and if system does not have enough memory, it will terminate the app.

{% include image.html
            img="assets/2020-02-24/application-lifecycle.png"
            title=""
            caption=""
            %}

## Flow of app life cycle from launch to suspended states

{% include image.html
            img="assets/2020-02-24/flow-of-app-lifecycle.png"
            title=""
            caption=""
            %}

## The Main Run Loop:

- An app’s main run loop processes all user-related events.

- App delegate sets up the main run loop at launch time and uses it to process events and handle updates to view-based interfaces.

- main run loop executes on the app’s main thread

- main thread is serial thread and this ensures that user-related events are processed serially in the order in which they were received.

## Interview Questions on App life cycle

**1/ How background iOS app gets resumed in foreground?**

When user launches an app that is currently in background, the system moves app to the inactive state and then to the active state.
{% include image.html
            img="assets/2020-02-24/question-1.png"
            title=""
            caption=""
            %}

**2/ What are the steps involved when app enter to foreground after device rebooted?**

When user launches an app for the first time or after device reboot or after system terminate the app, the system moves app to the active state.
{% include image.html
            img="assets/2020-02-24/question-2.png"
            title=""
            caption=""
            %}

**3/ What are the steps involved when app moves from foreground to background?**

{% include image.html
            img="assets/2020-02-24/question-3.png"
            title=""
            caption=""
            %}

**4/ How can you opt out background execution?**

- You can explicitly opt out background execution by adding UIApplicationExitsOnSuspend key to application’s Info.plist file and setting its value to YES.

- When you opt out background state, app life cycles will be between the not running, inactive, and active states and never enters the background or suspended states.

**5/ Which is the app state when device is rebooted?**

- Ans: Not Running state.

**6/ When app is running but not receiving event. In which state app is in?**

- Ans: Inactive state.

**7/ How an iOS app responds to interrupts like SMS, Incoming Call, Calendar, etc.?**

Application moves to the inactive state temporarily and it remains in this state until the user decides whether to accept or ignore the interruption.

- If the user ignores the interruption, the application is reactivated.

- If the user accepts the interruption, the application moves into the suspended state.

{% include image.html
            img="assets/2020-02-24/question-7.png"
            title=""
            caption=""
            %}

**8/ What are the uses of background state?**

- It gives opportunity to save any application data that will help user to relaunch the app where they left.

- App release any resources that don’t need.

**9/ How can you add additional background execution time for your app?**

- App can stay in background for few seconds and you can execute any code within that time.

- You can call `beginBackgroundTask(expirationHandler handler: (() -> Void)? = nil)` method which requests additional background execution time for your app

- To extend the execution time of an app extension, use the `performExpiringActivity(withReason:using:)` method

**10/ How can you check maximum amount of time available for app in background?**

- `backgroundTimeRemaining` helps to get maximum amount of time remaining for the app to run in the background.

- The value is valid only after the app enters the background and has started at least one task using beginBackgroundTask(expirationHandler:) in the foreground.

**11/ How can you debug your background task?**

- `beginBackgroundTask(withName:expirationHandler:)` for debugging background task.

## Conclusion

I hope, above tutorial will help to clear the concepts of application lifecycle.
