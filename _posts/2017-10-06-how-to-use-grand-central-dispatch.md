---
layout: post
title: "Grand Central Dispatch in Swift"
author: "Linh Vo"
tags: "UIKit"
---

Computers can’t live without multithreading and concurrency. You know why? CPUs are kinda dumb – they can only do one thing at a time! You can use Grand Central Dispatch (GCD) with Swift to make your app execute multiple tasks concurrently. Grand Central Dispatch is paramount to keep your app responsive, with buttery smooth animations and transitions.

In this tutorial we’re taking a look at multithreading, background processing and Grand Central Dispatch. You’ll write a bit of code and I’ll show you how Grand Central Dispatch works.

# Multithreading and Concurrency

Here’s how you can execute a task asynchronously in Swift:

```swift
DispatchQueue.global(qos: .userInitiated).async {
    // Download file or perform expensive task asynchronously
}
```

But what exactly is “async”? To answer that question, we’ll need to discuss multithreading first.

An iPhone has a CPU, its Central Processing Unit. Technically, a CPU can only perform one operation at a time – once per clock cycle. Multithreading allows the processor to create concurrent threads it can switch between, so multiple tasks can be executed at the same time.

It appears as if the two threads are executed at the same time, because the processor switches rapidly between executing them. As a smartphone or desktop user, you don’t notice the switches because they occur so rapidly. Performing multiple tasks at the same time is called concurrency.

A common example is the UI thread. Ever noticed how apps can still appear responsive and have smooth animations, even though the CPU is doing a lot of work? That’s because of multithreading.

On an iPhone, the UI always has its own thread. When you’re downloading a file from the web, or executing a complex task, the UI remains responsive and smooth.

This is because the iPhone switches between drawing the screen and downloading the file quickly, until both tasks are complete. There’s enough parallel CPU power available to draw the screen and download the file, without having to execute the tasks sequentially.

Then what about multicore CPUs? Such a CPU, often called a System-on-a-Chip (SoC), typically has multiple CPUs in one. This allows for truly parallel processing, instead of the concurrency by rapidly switching of multithreading.

In reality, multicore processing isn’t exactly parallel. It depends greatly on the type of work the processors are executing – but that’s a topic for another article.

To summarize, multithreading allows a CPU to rapidly switch between multiple tasks in such a way that it appears as if the tasks are executed simultaneously.

# Running Code in the Background

When coding apps multithreading is often used synonymously with `background processing`.

As an app developer, you want to keep your app responsive. Say you’re sending an update back to your cloud-based database, saving a note the user has stored in your app.

You don’t want the UI to freeze up when you’re saving this note, so you use background processing to do two things at the same time. Even though it really is multithreading, you’re calling it background processing because one task remains in the foreground (the UI) and one task is sent to the background (saving data).

```swift
var note = Note()

note.saveInBackground {
    var alert = "Your note has been saved!"
    alert.show()
}
```

In the pseudo-code example above we’re creating a `note` object, which is saved in the background. When it has been saved the completion handler (a closure) is executed, showing an alert dialog to the user.

The `saveInBackground()` function will use some form of concurrency. The Cocoa Touch SDK has several options for concurrency, of which Grand Central Dispatch (GCD) is the most common.

You regularly see APIs mention that they use background or “async” processing to complete a task. You now know that this API uses any of the approaches for multithreading, and that this task is performed parallel to the main thread. Good to know!

# Don’t Block The UI Thread!

You can’t update an app’s UI outside the main thread. User Interface operations, like showing a dialog or updating the text on a button can only be performed on the main thread. But… why?

There are a number of reasons for that, but the foremost reason is to avoid race conditions. A race condition occurs when two tasks are executed concurrently, when they should be executed sequentially in order to be done correctly. Conversely, some tasks should only be dispatched asynchronously to avoid a deadlock.

Drawing the UI on the screen of the iPhone is such a sequential task. The same goes for resolving Auto Layout constraints. You can’t change the constraints while they’re being calculated.

What if you execute a synchronous, blocking task on the main UI thread? The thread will wait until the synchronous task is executed. As a result, your apps UI may stutter, lag or become unresponsive for some time.

We can derive a few first principles from this:

1. Keep the UI thread free, to keep your app responsive

2. Avoid blocking the UI thread with synchronous code

The good news is that we can do both with Grand Central Dispatch! We’ll move some task to the background, and then bring its result back onto the main thread to update the UI.

# Executing Async Code with Grand Central Dispatch

When you need tasks executed concurrently, should you just go about creating a bunch of threads? Fortunately not! iOS has a terrific mechanism for working with concurrent tasks called Grand Central Dispatch.

The name “Grand Central Dispatch” is a reference to Grand Central Terminal in downtown New York. Imagine the CPU as a bunch of railroads, with the traincars as tasks being executed on those lines. A dispatcher is moving the cars along the railroads to ensure that they reach their destination quickly on a limited number of lines.

Grand Central Dispatch is a wrapper around low-level code, to create threads and manage code. Its emphasis is on dispatching, i.e. making sure that a number of tasks of variable importance and length are executed in a timeframe as reasonable as possible.

Let’s take a look at an example:

```swift
DispatchQueue.global(qos: .userInitiated).async {

    // Download file or perform expensive task asynchronously

    DispatchQueue.main.async {
        // Update the UI
    }
}
```

In the example above, an asynchonous task is dispatched with `async()`. The code that’s executed asynchronously is written in the first set of squiggly brackets { }. It’s a closure that’s executed in the background.

The async task is dispatched to an operation queue, and given a Quality-of-Service priority. Queues help us schedule tasks after each other, and the QoS priority indicates which tasks are more important to complete soon – but more on that later.

When the async task finishes, another async() call is made to execute another task on the main thread. You can’t update your app’s UI outside the main thread, as a rule, so that’s what that inner closure is for.

Next, we’ll discuss Quality-of-Service and queues.

> Why are we calling `async()` on the main thread? Wasn’t that forbidden and/or isn’t the main thread synchronous-only? Without getting into too much detail, it’s important to understand that `async()` dispatches code asynchronously. When it’s synchronous code, the thread will block until the task completes. If we’d call `sync` from our background thread, that thread would block until code completes on the main thread (mind-boggling, I know). That’s not only inefficient, it could also potentially create a so-called “deadlock”. A deadlock happens when two resources (i.e., tasks or threads) are waiting on each other to complete. As a result, they never do, and your app crashes.

# Quality-of-Service (QoS) and Grand Central Dispatch

Let’s check out that code again:

```swift
DispatchQueue.global(qos: .userInitiated).async {
    // ...
}
```

The function `global(qos:)` is called on class `DispatchQueue`. This function returns a reference to the global dispatch queue, on which we call `async(_:)`. The `global(qos:)` function takes a parameter: the Quality-of-Service (QoS) or “priority”.

In essence, the QoS tells the dispatcher how important the task that’s dispatched is. More important tasks are executed earlier on the dispatch queue, have more system resources available, and thus complete quicker than the same task with a lower priority.

Higher QoS tasks use more battery power, so defining the right QoS ensures that your app uses power and memory as efficiently as possible. Avoid the urge to mark every task as “important,” because it’ll only mark them as equally important and thus negate the entire priority system.

It’s important that `Quality-of-Service` and priority aren’t exactly the same. iOS will give tasks with a higher QoS more priority, but it’ll also throttle tasks up and down based on other aspects of the OS environment.

iOS has four `Quality-of-Service` levels, from highest to lowest priority:

1. `.userInteractive` is intended for user-interactive tasks, such as animations, events, and updating UI. As a rule of thumb, use this QoS for tasks that your app’s user is actively using.

2. `.userInitiated` is intended for tasks that prevent the user from using your app, like saving a file. Use it for tasks that your app’s user is actively waiting on.

3. `.utility` is intended for tasks that don’t require an immediate result, such as background downloads with a progress bar. Use it for background tasks that need a balance between responsiveness, performance and energy efficiency.

4. `.background` is intended for tasks that aren’t visible to your app’s user, like indexing, sync and backups. This setting prioritizes energy efficiency.

You can also use `.default`, after which QoS information is inferred from the context of your code.

# Queues and Grand Central Dispatch

Alright, back to the code:

```swift
DispatchQueue.global(qos: .userInitiated).async {
    // ...
    DispatchQueue.main.async {
        // ...
    }
}
```

We’re seeing two calls to `async()`. The first one on the global queue, and the second one on the main queue. So… what’s a queue?

A dispatch queue is simply a pool of tasks. Grand Central Dispatch will determine which tasks are executed. It’ll also determine how much CPU time those tasks get. More CPU time means a quicker completion.

Think back to that train terminal dispatcher. Which traincar is more important; which should get to its destination the quickest? The dispatcher can only move one car at a time, and moving one car forward more means delaying all the other cars.

The dispatcher takes an informed decision and lets all tasks run until they complete, giving preference to more important tasks (QoS). It’ll also assign tasks to different CPUs – the iPhone 11 Pro, for example, has a multicore SoC with 6 CPUs. Execute two tasks on two CPU cores, and they’re executed in parallel!

Finally, in the example above, another task is dispatched to the main thread. You need to inform the user, for instance, that their download has completed. This is something you need to do on the main thread.

It’s compelling to think that tasks on a dispatch queue are executed one after the other, and you wouldn’t be wrong. After all, when we queue up at a supermarket or icecream parlor each of us gets served after the other. However, a queue has a prioritization mechanism based on Quality-of-Service. A background task is less important than a user-interactive task, so the user-interactive task gets more CPU power and completes sooner.

Fortunately you don’t have to manage all those priorities, queues, threads and tasks yourself. That’s the kind of managing Grand Central Dispatch does!

# Delaying Tasks with “asyncAfter()”

Grand Central Dispatch has a number of options for concurrent execution, with `async()` being the most notable one. Another is `asyncAfter()` for delaying a task to be executed in the future:

```swift
let delay = DispatchTime.now() + .seconds(60)
DispatchQueue.main.asyncAfter(deadline: delay) {
    // Dodge this!
}
```

In the example above, a constant `delay` is defined. Its value is a point in time relative to the default clock, kind of like a time interval measured from “now.” The `asyncAfter(deadline:)` function then simply takes that interval, and a closure, and executes the code when the delay interval time is reached, i.e. 60 seconds into the future. Neat!

# Conclusion

You use multithreading and concurrency to ensure a smooth user experience, while doing lots of stuff at the same time. The UI remains responsive, while you complete an intensive or lengthy task in the background.

Grand Central Dispatch is one approach for concurrency on iOS, which we’ve discussed in this tutorial. Awesome!

Origin Post: [https://learnappmaking.com/grand-central-dispatch-swift/](https://learnappmaking.com/grand-central-dispatch-swift/)