---
layout: post
title: "How to use phantom types in Swift"
author: "Linh Vo"
tags: "UIKit"
---

Phantom types are a powerful way to give the Swift compiler extra information about our code so that it can stop us from making mistakes. In this article I’m going to explain how they work and why you’d want them, as well as providing lots of hands-on examples you can try.

# What are phantom types?

The basic definition of a phantom type is this: a type that doesn’t use at least one of its generic type parameters. That’s it – it’s not very complicated, at least when looking at the code.

So, this is a phantom type:

```swift
struct Employee<Role>: Equatable {
    var name: String
}
```

That is generic over some sort of `Role`, but `Role` doesn’t appear in the type’s definition – there’s only that one string.

In comparison, here is one of Apple’s Foundation APIs, slightly simplified:

```swift
struct Measurement<UnitType> {
    let unit: UnitType
    var value: Double
}
```

That is not a phantom type, no matter what you might hear elsewhere – it is generic over `UnitType` but that parameter is used right there for `unit`, so this is just a regular generic type.

So, the definition of a phantom type is simple, but the real question here is this: why the heck would you want to specify a generic parameter and never use it?

# What’s the point?

Our trivial phantom type was this:

```swift
struct Employee<Role>: Equatable {
    var name: String
}
```

That doesn’t use `Role` at all, so we could just have used this instead:

```swift
struct Employee: Equatable {
    var name: String
}
```

However, there’s an important difference: although we don’t use the generic parameter, Swift does. This allows Swift to enforce rules for us much more strictly.

For example, if you work for a software company you might have employees who work in a coding team and other employees who work in a sales team. We can represent them with no-case enums like these:

```swift
enum Sales { }
enum Programmer { }
```

**Tip:** It’s a good idea to use a no-case enum rather than a struct, because the enum cannot be instantiated, whereas someone could create an instance of the struct and wonder what it’s good for.

Now, if you had a friend called Zoe who worked in the coding team, she might look like this:

```swift
let zoe1 = Employee<Programmer>(name: "Zoe")
```

And if you had another friend called Zoe who worked in the sales team, she might look like this:

```swift
let zoe2 = Employee<Sales>(name: "Zoe")
```

Now, we made `Employee` conform to `Equatable`, and because its single property already conforms to `Equatable` Swift will be able to synthesize an `==` function to let us compare two employees. So, we can check whether our two employees are actually the same person like this:

```swift
print(zoe1 == zoe2)
```

Except that won’t work – we aren’t comparing two instances of `Employee` any more, we’re comparing an `Employee<Programmer>` and an `Employee<Sales>`, and Swift considers them to be different. In comparison, if we had made someone’s role a string property, Swift would say that the two Zoe’s are different at runtime, but it wouldn’t stop us from compiling.

That’s the power of phantom types: we give Swift extra information that clarify how things should work, and allow it to work harder on our behalf. When done well, this will allow logic errors to be surfaced as compiler errors: Swift literally won’t build our code because what we’re trying to do doesn’t make sense.

# Hands-on examples

If you were working in a hospital that analyzes blood samples, you might start by defining some of the different blood types:

```swift
enum OPositive { }
enum APositive { }
enum BPositive { }
```

Now you can create a `BloodSample` struct that is generic over some sort of blood type, like this:

```swift
struct BloodSample<Type> {
    let amount: Double

    static func +(lhs: BloodSample, rhs: BloodSample) -> BloodSample {
        BloodSample(amount: lhs.amount + rhs.amount)
    }
}
```

That has a `+` operator that lets us combine two blood samples, but Swift will automatically understand that two samples from different blood types are different and stop us from mixing them by accident.

So, this code will work:

```swift
let sample1 = BloodSample<OPositive>(amount: 5)
let sample2 = BloodSample<APositive>(amount: 5)
let sample3 = BloodSample<OPositive>(amount: 7)

let combined1 = sample1 + sample3
```

But this will not:

```swift
let combined2 = sample1 + sample2
```

Sometimes the differences between types are less subtle. For example, consider a user struct like this one:

```swift
struct User {
    let id: Int
    let age: Int
}
```

Both `id` and `age` are integers, so there’s nothing stopping us from comparing them like this

```swift
let user = User(id: 53, age: 53)
print(user.id == user.age)
```

I mean, yes, the two integers are identical, but if you’re comparing user IDs to ages then it’s almost certainly a mistake – and it’s a mistake phantom types can help us catch.

First, we make no-case enums to represent out variations:

```swift
enum UserID { }
enum Age { }
```

Next, we create a struct that is generic over some kind of type, but doesn’t actually use that type in its definition:

```swift
struct Tag<Type>: Equatable {
    var value: Int
}
```

Now we can modify the `User` struct so that each of its properties use `Tag`, like this:

```swift
struct User {
    let id: Tag<UserID>
    let age: Tag<Age>

    init(id: Int, age: Int) {
        self.id = Tag(value: id)
        self.age = Tag(value: age)
    }
}
```

And now trying to compare `id` and `age` simply won’t work.

In practice, I’d prefer to change that a little so that `Tag` conformed to the `ExpressibleByIntegerLiteral` protocol like this:

```swift
struct Tag<Type>: Equatable, ExpressibleByIntegerLiteral {
    init(integerLiteral value: Int) {
        self.value = value
    }

    var value: Int
}
```

That allows Swift to make instances of `Tag` directly from an integer, so we can simplify all the types that use `Tag` so they rely on Swift’s synthesized memberwise initializer:

```swift
struct User {
    let id: Tag<UserID>
    let age: Tag<Age>
}
```

The most important thing to remember is that although our phantom types are empty, the Swift compiler is fully aware of them. So, you can use them as constraints in extensions and more – they aren’t special.

# Building a state machine

There are lots of practical ways you can use phantom types to build interesting stuff, and I came across one fascinating experiment from Soroush Khanlou. In his example, Soroush uses phantom types to build a state machine where invalid transitions can’t compile – Swift won’t allow it.

**Tip:** State machines are pieces of code that are designed to move between a series of predefined states. For example, a vending machine might move between the states “waiting for customer”, “coin inserted”, “fetching selection” and “serving selection”. It wouldn’t make sense for the vending machine to go from “waiting for customer” to “serving selection”, because no coin was inserted.

Soroush very kindly gave me permission to use his code here, although I’m going to simplify a little so we can focus on the part that matters.

First, we create the various states our machine can be in. For this example we’ll have have the four that represent a vending machine:

```swift
enum Waiting {}
enum CoinInserted {}
enum Fetching {}
enum Serving {}
```

Second, we create a phantom type to store a transition between two states:

```swift
struct Transition<From, To> {}
```

Neither of those types are used – the struct is literally empty. But that’s okay, because Swift is still able to track them.

Third, we’ll create a `Machine` struct that represents our state machine in one given state. This has one method that will transition from the current state to a new state, like this:

```swift
struct Machine<State> {
    func transition<To>(with transition: Transition<State, To>) -> Machine<To> {
        .init()
    }
}
```

That’s doing a lot of work in hardly any code, so let’s break it down:

- The `Machine` struct is generic over some kind of `State`.

- The `transition()` method is generic over some kind of `To`, but also relies on the `State` generic parameter from `Machine`.

- We’re telling Swift the transition must be from the current state of our machine, and to some other kind of state.

- The function returns a new machine that uses the new `To` state.

- Using `.init()` will automatically make a new `Machine` struct of the correct type: `Machine<To>`.

Now we can go ahead and create all valid transitions by specifying where they come from and where they go to, like this:

```swift
let start = Transition<Waiting, CoinInserted>()
let selectionMade = Transition<CoinInserted, Fetching>()
let delivery = Transition<Fetching, Serving>()
let reset = Transition<Serving, Waiting>()
```

And that’s it! We can now create a `Machine` with a particular state, and step it through any other possible valid state, like this:

```swift
let m1 = Machine<Waiting>()
let m2 = m1.transition(with: start)
let m3 = m2.transition(with: selectionMade)
let m4 = m3.transition(with: delivery)
let m5 = m4.transition(with: reset)
```

The magic here is that Swift will check all those transitions at compile time – it’s not possible to make an illegal transition.

For example, this won’t compile:

```swift
let m6 = m5.transition(with: delivery)
```

The problem is that `m5` is a `Machine<Waiting>`, so we can’t go straight from there to a `Machine<Serving>`.

I think this is a fantastic experiment, because it shows how we can leverage the Swift compiler to prove that our code is correct. Plus, it makes for a fascinating exploration of phantom types in action.

However, in practice this solution doesn’t work so well. To explain why, I asked Soroush himself – here’s what he said:

> “I originally wrote this as an experiment to see if I could have the compiler protect me from performing invalid transitions on a state machine, but I quickly realized that changes to the state would cause the type of the entire machine to change, meaning that it couldn’t be stored back into the same variable. This prevents you from having it as a property on, say, a view controller, with transitions modifying the state machine. So: fun to try out, a little impractical in actual usage.”

# Undefined as a type

Before I finish, there’s one more technique I want to demonstrate, and it’s quite different from all the earlier examples. It’s this piece of code:

```swift
func undefined<T>(_ message: String = "") -> T {
    fatalError("Undefined: \(message)")
}
```

That isn’t a phantom type, but it is a great example of using Swift’s type system in interesting ways. I first saw this being used by Johannes Weiss from the SwiftNIO team, where he presented this as a useful global function to have around while you’re still building your app.

This function, `undefined<T>()`, can act as pretty much any other kind of value you want, because it is generic over `T` and promises to return a `T`. It won’t actually return a T, but it doesn’t need to: calling `fatalError()` is enough to see that nothing needs to be returned.

This set up it can act as a placeholder to make your code compile when you’re part-way through your work and don’t want to fill in all the various parts just yet.

You can make simple instances of types using `undefined()`, like this:

```swift
let name: String = undefined("Example string")
let score: Int = undefined("Example int")
```

You can make it act as a placeholder inside a function you haven’t written yet:

```swift
func userID(for username: String) -> Int? {
    undefined(username)
}
```

In that instance you could use `fatalError()` to mark the missing code, but then again `fatalError()` does have legitimate uses outside of marking unfinished work so it becomes harder to find and replace later.

And you can even use it repeatedly when there are lots of parameters you haven’t filled in yet:

```swift
let timer = Timer(timeInterval: undefined(), target: undefined(), selector: undefined(), userInfo: undefined(), repeats: undefined())
```

Obviously many of these places could be replaced with hard-coded placeholders, such as 0 or an empty string, but the magic of `undefined()` is that it will crash as soon as it’s touched – it will work only at compile time, to help you move your project forward, but can’t possibly work at runtime. Also, it’s easier to search your code for “undefined” than some magic value!

Of course, you might have the question “if `undefined<T>()` isn’t a phantom type, what is it?” And to be honest I had exactly the same question, so I went back to the source and asked Johannes. Here’s what he said:

> I wouldn't call `undefined<T>()` a phantom type because first of all, really it's a value. Well, a value that we'll never generate, but still. Even calling `T` a phantom type isn't quite right: `T` can be anything, so it could be a `String` which is far from a phantom type. I think the closest somewhat correct term I have for this is that it's called the "bottom value", usually written like ⏊, and even that is probably not right because the "bottom type" is defined as not having values! So the term "bottom value" isn't great either, but I guess we could say "undefined's return type is bottom"?

I love that answer, because it shows just how nuanced the discussion is.

Anyway, that finishes up our discussion about phantom types. You’ve seen how they work and I’ve given you some hands-on examples, plus I’ve walked you through how we can build state machines using them as an interesting experiment, and even how we can create simple placeholder functions that can act as helpers to make our code compile while we’re still working. They might not be phantom types, but they are still fascinating!

# Further reading

Phantom types are a fascinating area of discussion, and there’s a lot of high-quality material I recommend you check out.

First, Johannes Weiss delivered the talk that originally introduced to me to phantom types in Swift: [The Type System is Your Friend](https://academy.realm.io/posts/swift-summit-johannes-weiss-the-type-system-is-your-friend/). He also introduced the `undefined()` function there, although I recommend you check out his [repository on GitHub](https://github.com/weissi/swift-undefined) because it’s a little more advanced.

Second, Brandon Case delivered a talk called [Strings Are Evil](https://www.youtube.com/watch?v=UTm5p96KlEc), where he builds a file-handling system where phantom types make it impossible to created badly-formed file paths.

Third, if the idea of tagged integers interested you, you might want to check out the [Tagged repository on GitHub](https://github.com/pointfreeco/swift-tagged), which takes the concept significantly further. As an alternative, you should check out my article [Improving your Swift code using value objects](https://www.hackingwithswift.com/articles/188/improving-your-swift-code-using-value-objects).

Fourth, I walked you through a simplified version of Soroush Khanlou’s state machine, but if you’d like to read the full version [it’s in this GitHub Gist](https://gist.github.com/khanlou/60c4fd0baa457b7a621a74ee599be867)

Finally, Ole Begemann walks through what it would look like to rewrite Apple’s `Measurement` API using phantom types in this article: [Measurements and Units with Phantom Types](https://oleb.net/blog/2016/08/measurements-and-units-with-phantom-types/)

# Challenges

If you’d like to take your knowledge of phantom types further, try creating phantom types to solve these problems:

1. A `Temperature` type that is generic over whether it uses Celsius or Fahrenheit.

2. A `Car` type that is generic over whether it uses diesel or gas for the engine.

I also have a question that I’d like you to consider: what benefits of phantom types could we gain by using a class hierarchy instead? Just as importantly, what benefits would we lose?

Reference: [https://www.hackingwithswift.com/plus/advanced-swift/how-to-use-phantom-types-in-swift](https://www.hackingwithswift.com/plus/advanced-swift/how-to-use-phantom-types-in-swift)
