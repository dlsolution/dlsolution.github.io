---
layout: post
title: "Working with Dates in Swift"
author: "Linh Vo"
tags: "UIKit"
---

# Date
> A specific point in time, independent of any calendar or time zone.

The simplest way to get the current date is by:

```swift
let date = Date()
print(date) // prints 2017-02-01 10:48:09 +0000
```

This date object supports comparison, time interval calculation, and the creation of a new date relative to another date. For example:

```swift
let now = Date()
let hourAgo = now - 3600 // an hour is 3600 seconds
let timeInterval = now.timeIntervalSince(hourAgo)
print(timeInterval) // prints 3600, which is an hour in secs
print(now) // prints 2017-02-01 12:59:44 +0000
print(hourAgo) // prints 2017-02-01 11:59:44 +0000
```

This is already cool but if you need to have more control over the date you should use `DateFormatter` coming up next.

# DateFormatter

If you want to show the date information in another format, e.g. to show the date in some different country’s style, then you need to use `DateFormatter`.

### Date Formatting with Style Presets

`DateFormatter` provides you with a variety of different date formats you can utilize.

As an example, let’s create a `DateFormatter` object, and configure the `dateStyle` and `timeStyle` properties to change the style of the string representation of the date.

```swift
let date = Date() // The current date
let dateFormatter = DateFormatter()
dateFormatter.dateStyle = .full
dateFormatter.timeStyle = .full
let dateText = dateFormatter.string(from: date)
print(dateText)
// Prints:
// Saturday, February 01, 2017 at 10:44:34 AM Indochina Time
```

The complete list of all the possible style presets for `dateStyle` and `timeStyle` is `.none`, `.short`, `.medium`, `.long`, `.full`. Feel free to experiment with these.

### DateFormat — Another Way to Customize Date Format

In addition to `dateStyle` and `timeStyle`, dateFormat is also a way to customize the date formatting. However, it should be used only when working with fixed format representations, such as `ISO8601DateFormatter`.

Also, it does not make sense to use the previously described `dateStyle` or `timeStyle` properties while using the `dateFormat` in the `DateFormatter`.

### Date Style Customization + Localization

If the date format you prefer is not possible with the earlier style presets, you can use the `setLocalizedDateFormatFromTemplate` function in `DateFormatter` to achieve the style you are lacking with.

As an example, let’s say we want to show the current date in the Finnish calendar in a way it is shown in Finland, that is, for example `01.02.2017`. This format is not plausible with the earlier style presets, so we need to customize the `DateFormatter`:

```swift
let dateFormatter = DateFormatter()
dateFormatter.locale = Locale(identifier:"fi_FI") // Date in Finland
dateFormatter.setLocalizedDateFormatFromTemplate("yyyy-MM-dd")
let dateStr = dateFormatter.string(from: Date())
print(dateStr) // prints 01.02.2017
```

Notice that we used a `locale` property of the `DateFormatter`. With `locale` the date format gets automatically converted to the format that is used in a specific country defined by the `locale`.

You should always call `setLocalizedDateFormatFromTemplate` only after setting the `locale` of the `DateFormatter` in order to show the correctly formatted date in the region.

# DateComponents

`DateComponents` represents the date in the units of time. E.g. hours, minutes, seconds, etc. It lets you easily access and modify all the time components of the date separately.

In the following example, the current date is extracted to its time components using `DateComponents`.

```swift
let currentDate = Date()
let calendar = Calendar.current // The user's current calendar
let date = calendar.dateComponents(in: .current, from: currentDate)
let year = date.year
let month = date.month
let day = date.day
let hour = date.hour
let minute = date.minute
let second = date.second
```

Reference: [Medium](https://medium.com/codex/working-with-dates-in-swift-9f50390bbc81)