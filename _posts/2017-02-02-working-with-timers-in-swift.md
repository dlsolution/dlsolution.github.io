---
layout: post
title: "Working with Timers in Swift"
author: "Linh Vo"
tags: "UIKit"
---

Timers are super handy in Swift, from creating repeating tasks to scheduling work with a delay. This tutorial explains how to create a timer in Swift.

# How To Create a Repeating Timer

```swift
let timer = Timer.scheduledTimer(timeInterval: 1.0, target: self, selector: #selector(fire(timer:)), userInfo: ["score": 10], repeats: true)

@objc func fire(timer: Timer) 
{
    if  let userInfo = timer.userInfo as? [String: Int],
        let score = userInfo["score"] {

        print("You scored \(score) points!")
    }
}
```

We’ve used 5 parameters to create a timer. They are:

- `timeInterval`: the interval between timer fires in seconds, its type is Double

- `target`: a class instance that the function for selector should be called on, often `self`

- `selector`: the function to call when the timer fires, with `#selector(...)`

- `userInfo`: a dictionary with data that’s provided to the `selector`, or `nil`

- `repeats`: whether this timer is repeating or non-repeating

Then how do you stop a repeating timer? It’s simple. Here’s how:

```swift
timer.invalidate()
```

# Creating A Countdown Timer

Imagine you’re creating a game. The user has 60 seconds to solve a puzzle and score points. As the count down timer runs out, you’re keeping track of how many seconds are left.

First, we’re creating two properties at the top of our game class. Something like:

```swift
var timer:Timer?
var timeLeft = 60
```

The timer doesn’t start immediately. Until it starts, the `timer` property is `nil`. At some point the game has started, so we also start the timer:

```swift
timer = Timer.scheduledTimer(timeInterval: 1.0, target: self, selector: #selector(onTimerFires), userInfo: nil, repeats: true)
```

The timer calls `onTimerFires()` every second, indefinitely. And here’s that function:

```swift
@objc func onTimerFires()
{
    timeLeft -= 1
    timeLabel.text = "\(timeLeft) seconds left"

    if timeLeft <= 0 {
        timer?.invalidate()
        timer = nil
    }
}
```

Every time the timer fires it subtracts 1 from `timeLeft`, and it updates the “time left” label. When the timer reaches zero, `timer` is invalidated and set to `nil`.

This will effectively create a countdown timer that counts down from 60 to zero. And at a point in the future, you can of course reset the countdown and start again.

# Timers, Runloops and Tolerance

Timers work in conjunction with `run loops`. Run loops are a fundamental part of threads and concurrency.

It’s easiest to imagine run loops like a ferris wheel. People can enter a passenger car and get moved around by the wheel. In a similar way, you can schedule a task on a run loop. The run loop keeps “looping” and executing tasks. When there are no tasks to execute, the run loop waits or quits.

The way run loops and timers work together, is that the run loop checks if a timer should fire. A timer isn’t a real-time mechanism, because the firing of the timer can coincide with the runloop already executing another task. You can compare this to wanting to enter the ferris wheel when there’s no car yet at the bottom entrance. The result is that a timer can fire later than it’s scheduled.

This isn’t necessarily a bad thing. Don’t forget you’re running code on a mobile device that has energy and power constraints! The device can schedule run loops more efficiently to save power.

You can also help the system save power by using a timer property called tolerance. This will tell the scheduling system: “Look, I want this to run every second, but I don’t care if it’s 0.2 seconds too late.” This of course depends on your app. Apple recommends to set the tolerance to at least 10% of the interval time for a repeating timer.

Here’s an example:

```swift
let timer = Timer.scheduledTimer(timeInterval: 1.0, target: self, selector: #selector(fire), userInfo: nil, repeats: true)
timer.tolerance = 0.2
```

The tolerance property uses the same units as the `timeInterval` property, so the above code will use a `tolerance` of 200 milliseconds.

The tolerance will never cause a timer to fire early, only later. And the tolerance will neither cause a timer to “drift”, i.e. when one timer fire is too late, it won’t affect the scheduled time of the next timer fire.

When using the `Timer.scheduledTimer(...)` class method, the timer is automatically scheduled on the current run loop in the default mode. This is typically the run loop of the main thread. As a result, timers may not fire when the run loop on the main thread is busy, for example when the user of your app is interacting with the UI.

You can solve this by manually scheduling the timer on a run loop yourself. Here’s how:

```swift
let timer = Timer(timeInterval: 1.0, target: self, selector: #selector(fire), userInfo: nil, repeats: true)

RunLoop.current.add(timer, forMode: .commonModes)
```

The first line of code will create an instance of `Timer` using the `Timer(...)` initializer. The second line of code adds the timer to the current run loop using the `.commonModes` input mode. In short, this tells the run loop that the timer should be checked for all “common” input modes, which effectively lets the timer fire during the interaction with UI.

By the way, a common source for frustration is accidentally using the `Timer(...)` initializer when you wanted to use the `Timer.scheduledTimer(...)` to create a timer. If you use the former, the timer won’t fire until you’ve added it to a run loop! And you’ll pull your hair out, because you’re certain you’ve scheduled the timer correctly…

# Executing Code with a Delay

A common scenario in practical iOS development is executing some code with a small delay. It’s easiest to use `Grand Central Dispatch` for that purpose, and not use a `Timer`.

The following code executes a task on the main thread with a 300 millisecond delay:

```swift
DispatchQueue.main.asyncAfter(deadline: .now() + .milliseconds(300)) {  
    print("BOOYAH!")
}
```

It’s more concise than using `Timer`. And because you’re only running it once, there’s no need to keep track of a timer anyway.

