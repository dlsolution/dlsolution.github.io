---
layout: post
title: "Understanding generics"
author: "Linh Vo"
tags: "UIKit"
---

Generics are one of the most powerful features of Swift, allowing us to write code once and reuse it in many ways. In this article we’ll explore how they work, why adding constraints actually helps us write more code, and how generics help solve one of the biggest problems in Swift.

# Alphabetti spaghetti

In Swift, both types and functions can be generic, and you can recognize them immediately because we write things in angle brackets after their name.

So, here’s a regular function:

```swift
func count(numbers: [Int]) {
}
```

And here’s a generic function:

```swift
func count<Number>(numbers: [Number]) {
}
```

The parts inside angle brackets are called generic parameters: they specify the different ways the function or type can be used. These generic parameters are placeholders for real data types such as `Int` or `String` – we don’t know what `Number` will be, but we don’t actually care because or function will work regardless.

It’s common to use single-letter generic parameter names, such as `T`, `U`, and `V`. The convention is to use them in that order, to mean “here are three placeholders”. They might end up being the same type or different types, it doesn’t really matter – they are just placeholders.

**Swift does not care if you use** `count<Number>(numbers: [Number])` **or** `count<T>(numbers: [T])` **– the** `Number` **and** `T` **are both placeholders, so use whichever you prefer.**

# Resolving generics

When we use a generic function in code, Swift looks at how we use it to figure out what the placeholders mean. For example, we might call our generic `count()` function like this:

```swift
count(numbers: [1, 2, 3])
```

Swift can now see that the `Number` parameter is actually an integer, so it’s able to look at the `count()` function as if it were this:

```swift
func count(numbers: [Int]) {
}
```

Yes, that’s exactly what we had started with, so why not just write that?

Well, if we had specifically asked for an integer, we could fill in the body like this:

```swift
func count(numbers: [Int]) {
    let total = numbers.reduce(0, +)
    print("Total is \(total)")
}
```

That works great, because we know for sure that the array coming must contain integers, so we can add the together. But what if we had doubles instead?

```swift
count(numbers: [1.5, 2.5, 3.5])
```

That won’t work, because our function can only accept integer arrays, even though internally it should be able to add up doubles just fine.

This isn’t great: our function should be able to run on any kind of numeric data, whether it’s an integer array, a double array, or even a `CGFloat` array.

Let’s compare that to our generic version, where we would write this:

```swift
func count<Number>(numbers: [Number]) {
    let total = numbers.reduce(0, +)
    print("Total is \(total)")
}
```

…and that code is worse – it won’t actually work! You see, when we had integers coming in Swift knew it could rely on the `+` function to add them together. Now that we’re using generics, `Number` is a placeholder for literally anything at all. Yes, it might be an integer, but it might also be a string, an array, a dictionary, or a custom struct.

# Adding constraints

On the surface you might think generics are worse, because rather than knowing what type of data we have we now instead are forced to accept anything at all.

It’s true that using `<T>` for your generic parameters is effectively `Any` – you’re allowing your function or type to be used with any kind of data. But Swift allows us to constrain the generic parameters so they only work with some kinds of data.

For example, if we wanted to make our `count()` function actually work, we constrain the `Number` type so that it can only be used with types that conform to `Numeric`:

```swift
func count<Number: Numeric>(numbers: [Number]) {
    let total = numbers.reduce(0, +)
    print("Total is \(total)")
}
```

Tip: In case you haven’t seen it before, `Numeric` sits behind both `Int`, `Double`, and more.

So now our generic function is better than the original, because it adds more functionality – it works on both integers and doubles, and other numeric types too.

This sounds counter-intuitive, but it’s absolutely true: adding a constraint here might sound like it limits what we can do, but in fact constraints often let us do more because the compiler now understands what kind of data we have.

# Why not just use protocols?

When you see some generic functions you might wonder why the generic type is needed at all – why not use a protocol?

Well, let’s try it:

```swift
func count(numbers: [Numeric]) {
    let total = numbers.reduce(0, +)
    print("Total is \(total)")
}
```

What you’ll find is that Swift will refuse to build our code – we’ll get the dreaded error, “Protocol ‘Numeric’ can only be used as a generic constraint because it has Self or associated type requirements.”

What this means is that the `Numeric` protocol covers a wide variety of number types, such as integers and doubles, that aren’t interchangeable – we can’t use an `Int` like a `Double` in Swift, because it wouldn’t make any sense.

So, it also doesn’t make sense to accept them into a function like this: Swift can’t know at compile time whether it will be an integer, a double, or something else, which means we can’t add them together using `reduce()`, we can’t compare them, and so on.

# Understanding dispatch

This distinction between generic type constraints and simple protocols is important for two reasons.

As you’ve seen when we use generic constraints Swift resolves them at compile time. So, if we had a method like this:

```swift
func count<Number: Numeric>(numbers: [Number]) {
    let total = numbers.reduce(0, +)
    print("Total is \(total)")
}
```

Then we could write this:

```swift
count([1, 2, 3])
```

And we could also write this:

```swift
count([1.5, 2.5, 3.5])
```

Both work, because Swift is able to create a `count()` function that works for integers, and separately create a `count()` function that works for doubles. We call this process specialization, and it might result in many different forms of `count()` being generated behind the scenes – one for each different way we use it, and each optimized for its particular type of data.

The second reason is called static dispatch, and it dramatically affects performance.

To demonstrate this, I want some meaningful code to work with. So, here’s a protocol called `Prioritized` that lets us mark different kinds of data as being important or not:

```swift
protocol Prioritized {
    var priority: Int { get }
    func alertIfImportant()
}
```

And here are two conforming types that have different implementations of the `alertIfImportant()` method:

```swift
struct Work: Prioritized {
    let priority: Int

    func alertIfImportant() {
        if priority > 3 {
            print("I'm important work!")
        }
    }
}

struct Document: Prioritized {
    let priority: Int

    func alertIfImportant() {
        if priority > 5 {
            print("I'm an important document!")
        }
    }
}
```

There’s nothing complicated in there – just one protocol and two conforming types.

Where things get more interesting is how we use that protocol. For example, we might write a method to check the priority of a document and alert if it’s important, like this:

```swift
func checkPriority(of item: Prioritized) {
    print("Checking priority…")
    item.alertIfImportant()
}
```

That uses the `Prioritized` protocol, which means only types conforming to that protocol can be passed in. But at compile time, Swift can’t actually know what kind of `Prioritized` data it’s going to be used with: it could be a `Document`, it could be `Work`, or perhaps it could be both at different times.

So, when the compiler is trying to build that code, it has to keep its options open: it will look at the item object’s type at runtime – when the program is running on a user’s device – and call the right function each time.

We call this dynamic dispatch: Swift has to figure out dynamically where to send the function call, which takes some extra work.

Now look at this version instead:

```swift
func checkPriority<P: Prioritized>(of item: P) {
    print("Checking priority…")
    item.alertIfImportant()
}
```

That does the same thing, but now Swift knows at compile time how it’s being used, which means it can generate optimized `checkPriority()` functions for each type that uses it. This means no more type lookup at runtime, and instead it opens up a huge range of inlining possibilities – Swift can literally copy the `alertIfImportant()` function directly into `checkPriority()` if it wants to, because it always knows precisely which type is called.

We call this static dispatch: Swift can determine exactly which function will be called at compile time, and can therefore eliminate the runtime lookup and perform additional optimizations.

So, generics let us write types and functions that work across different kinds of data, add constraints to limit which types they can be used with, write code that would otherwise not be possible with simple protocols, and even help Swift generate faster code thanks to static dispatch.

Reference: [https://www.hackingwithswift.com/plus/intermediate-swift/understanding-generics-part-1](https://www.hackingwithswift.com/plus/intermediate-swift/understanding-generics-part-1)
