---
layout: post
title: "UIAlertController with Function Builders"
author: "Linh Vo"
tags: "UIKit"
---

I always found the `UIAlertController` API too verbose. You first have to create an instance of `UIAlertController`, then create multiple instances of `UIAlertAction` and finally add the actions to the controller.

In this post, I will show you how we can leverage the new Swift 5.1 feature of _Function Builders_ to create a simplified and highly readable API.

## Our goal

Let's take this sample code:

```swift
let alertController = UIAlertController(
    title: "Deletion",
    message: "Are you sure?",
    preferredStyle: .alert
)
let deleteAction = UIAlertAction(title: "Delete", style: .destructive) { _ in
    // Perform deletion
}
let cancelAction = UIAlertAction(title: "Cancel", style: .cancel)
alertController.addAction(deleteAction)
alertController.addAction(cancelAction)
present(alertController, animated: true)
```

{% include image.html
            img="assets/2020-03-10/alert-controller-deletion.png"
            title=""
            caption="" %}

As I said earlier, there is a lot to write and we can do better. We could improve the API with regular Swift [patterns](https://gist.github.com/dlsolution), but the goal here is to use function builders to create this SwiftUI like sample code:

```swift
let alertController = Alert(title: "Deletion", message: "Are you sure?") {
Action.destructive("Delete") {
    // Perform deletion
}
Action.cancel("Cancel")
}
present(alertController, animated: true)
```

## Function Builders

This new feature introduced in Swift 5.1 is not fully implemented yet. Instead of the public `@functionBuilder` annotation, you have to use the private `@_functionBuilder` one. Though, you can find the details of the proposal [here](https://github.com/apple/swift-evolution/blob/9992cf3c11c2d5e0ea20bee98657d93902d5b174/proposals/XXXX-function-builders.md#function-building-methods).

To better understand the purpose of this feature, this extract highlights the use cases of function builders:

> This proposal does not aim to enable all kinds of embedded DSL in Swift. It is focused on one specific class of problem that can be significantly improved with the use of a DSL: creating lists and trees of (typically) heterogenous data from regular patterns. This is a common need across a variety of different domains, including generating structured data (e.g. XML or JSON), UI view hierarchies (notably including Apple's new SwiftUI framework, which is obviously a strong motivator for the authors), and similar use cases.

The proposal focuses on a DSL to represent HTML [tree](https://github.com/apple/swift-evolution/blob/9992cf3c11c2d5e0ea20bee98657d93902d5b174/proposals/XXXX-function-builders.md#declaring-list-and-tree-structures) but it is also heavily used in SwiftUI to create view hierarchies with the `@ViewBuilder` [attribute](https://developer.apple.com/documentation/swiftui/viewbuilder).

The [documentation](https://github.com/apple/swift-evolution/blob/9992cf3c11c2d5e0ea20bee98657d93902d5b174/proposals/XXXX-function-builders.md#function-building-methods) is not official, but after some digging, here are the methods we can implement:

- `buildBlock(_ components: Component...) -> Component` _required_
- `buildIf(_ component: Component?) -> Component` _optional_, [previously named](https://forums.swift.org/t/function-builders/25167/53) `buildOptional`.
- `buildEither(first: Component) -> Component` and `buildEither(second: Component) -> Component`, _optionals_

The `buildExpression`, `buildDo` and `buildFunction` have no effect for the moment.

## In practice

What we want to build in our case is a list of _alert actions_. An `Action` is composed of a title, a style (`default`, `destructive` or `cancel`) and a function, triggered when the user taps the alert button on screen.

```swift
struct Action {
    let title: String
    let style: UIAlertAction.Style
    let action: () -> Void
}
```

Once we have an array of `Action` we can build an alert controller with this factory method:

```swift
func makeAlertController(title: String, message: String, style:  UIAlertController.Style, actions: [Action]) -> UIAlertController {
    let controller = UIAlertController(
        title: title,
        message: message,
        preferredStyle: style
    )
    for action in actions {
        let uiAction = UIAlertAction(title: action.title, style: action.style) { _ in
            action.action()
        }
        controller.addAction(uiAction)
    }
    return controller
}
```

So far so good, but we didn't bring anything new yet.

Let's see how to retrieve this array of actions and create our function builder. We saw earlier that the only required method is `buildBlock`, so we will start here. `buildBlock` combines a list of components to a single one.

_Note_: If we think about views and SwiftUI, that makes sense. From multiple subviews (the components for the `@ViewBuilder`), `buildBlock` will create a superview that contains all the subviews.

In our case, the `Component` type is an array of actions, so the `buildBlock` method can be written as follows:

```swift
@_ActionBuilder
struct ActionBuilder {

    typealias Component = [Action]

    static func buildBlock(_ children: Component...) -> Component {
        return children.flatMap { $0 }
    }

}
```

Now that we have our `ActionBuilder`, let's use it to retrieve an array of actions and pass it to the `makeAlertController` function.

```swift
func Alert(title: String, message: String, @ActionBuilder _ makeActions: () -> [Action]) -> UIAlertController {
    makeAlertController(
        title: title,
        message: message,
        style: .alert,
        actions: makeActions()
    )
}
```

Note the use of the `@ActionBuilder` attribute, that will allow us to use our new DSL in the `makeActions` closure.

That way we can build the following:

```swift
let alertController = Alert(title: "Deletion", message: "Are you sure?") {
    [Action(title: "Delete", style: .destructive, action: { /* ... */ })]
    [Action(title: "Cancel", style: .cancel, action: {})]
}
```

But... how is this supposed to be better ?!

I guess that's not what you expected...

It's really weird to see a list of arrays of actions. Why not a list of actions? That would make more sense to write.

The explanation here, as we saw earlier, is that our `Component` is the type `[Action]`. And `buildBlock` takes a list of components `Component...`, meaning `[[Action]]`. That's why we have to write it like this.

An evolution, in _future_ implementations of function builders, would be to use the `buildExpression` method (not yet available):

> `buildExpression(_ expression: Expression) -> Component` is used to lift the results of expression-statements into the `Component` internal currency type. It is only necessary if the DSL wants to either (1) distinguish `Expression` types from `Component` types or (2) provide contextual type information for statement-expressions.

If `buildExpression` was working, we could write:

```swift
@_functionBuilder
struct ActionBuilder {

    typealias Expression = Action
    typealias Component = [Action]

    static func buildExpression(_ expression: Expression) -> Component {
        return [expression]
    }

    static func buildBlock(_ children: Component...) -> Component {
        return children.flatMap { $0 }
    }

}
```

```swift
let alertController = Alert(title: "Deletion", message: "Are you sure?") {
    Action(title: "Delete", style: .destructive) { /_ ... _/ }
    Action(title: "Cancel", style: .cancel) {}
}
```

But for now, we are stuck with our list of `[Action]`. That's not really an issue though, because we can extract this weird behavior into factories:

```swift
extension Action {
    static func `default`(\_ title: String, action: @escaping () -> Void) -> [Action] {
        return [Action(title: title, style: .default, action: action)]
    }

    static func destructive(_ title: String, action: @escaping () -> Void) -> [Action] {
        return [Action(title: title, style: .destructive, action: action)]
    }

    static func cancel(_ title: String, action: @escaping () -> Void = {}) -> [Action] {
        return [Action(title: title, style: .cancel, action: action)]
    }
}
```

And our code now matches our initial goal !

```swift
let alertController = Alert(title: "Deletion", message: "Are you sure?") {
Action.destructive("Delete") {
    // Perform deletion
}
Action.cancel("Cancel")
}
```

## Conditions

We can now add multiple actions to the same alert. But that's not very dynamic yet...

What if we want to add actions conditionally ?
What if we want to add multiple actions at the same time ?

Let's try to add a condition.

```swift
let alertController = Alert(title: "Deletion", message: "Are you sure?") {
    Action.destructive("Delete") { /_ ... _/ }
    if canEdit {
        Action.default("Edit") { /_ ... _/ }
    }
    Action.cancel("Cancel")
}
```

We hit a compiler error:

```swift
closure containing control flow statement cannot be used with function builder
```

That's because we only have implemented the `buildBlock` method and we need to add `buildIf`. The implementation is simple, either we have a list of actions or else we return an empty list.

```swift
@_functionBuilder
struct ActionBuilder {

    typealias Component = [Action]

    static func buildBlock(_ children: Component...) -> Component {
        return children.flatMap { $0 }
    }

    static func buildIf(_ component: Component?) -> Component {
        return component ?? []
    }

}
```

With the `buildIf` implemented, everything compiles and runs, the `if` statement is taken into account.

But that's not enough yet, because if we try to have a `else` condition in our code, we hit the same compiler error again...

```swift
let alertController = Alert(title: "Deletion", message: "Are you sure?") {
    Action.destructive("Delete") { /_ ... _/ }
    if canEdit {
        Action.default("Edit") { /_ ... _/ }
    } else {
        Action.default("Share") { /_ ... _/ }
    }
    Action.cancel("Cancel")
}

// error: closure containing control flow statement cannot be used with function builder
```

To overcome this, we need to add two more functions: `buildEither(first:)` and `buildEither(second:)` used when there is a decision tree with optional sub blocks.

```swift
@_functionBuilder
struct ActionBuilder {

    typealias Component = [Action]

    static func buildBlock(_ children: Component...) -> Component {
        return children.flatMap { $0 }
    }

    static func buildIf(_ component: Component?) -> Component {
        return component ?? []
    }

    static func buildEither(first component: Component) -> Component {
        return component
    }

    static func buildEither(second component: Component) -> Component {
        return component
    }

}
```

And that makes our code compiling again with `if` / `else` conditions.

## Loops

At the moment we have no way to loop an array of strings and create actions out of them.

If we try, we always hit the same compiler error.

```swift
let alertController = Alert(title: "Title", message: "Message") {
    for string in ["Action1", "Action2"] {
        Action.default(string) { print(string) }
    }
}

// error: closure containing control flow statement cannot be used with function builder
```

One way to solve this problem is to create an helper function that generates a list of actions for each element of a sequence, and then aggregates all those lists of actions into a single one.

```swift
func ForIn<S: Sequence>(_ sequence: S, @ActionBuilder makeActions: (S.Element) -> [Action]) -> [Action] {
    return sequence
        .map(makeActions) // of type [[Action]]
        .flatMap { $0 }   // of type [Action]
}
```

And finally, we can use it like so:

```swift
let alertController = Alert(title: "Title", message: "Message") {
    ForIn(["Action1", "Action2"]) { string in
        Action.default(string) { print(string) }
    }
}
```

## Conclusion

We saw how we could improve the `UIAlertController` API with very little code. The difficulty with function builders is to find [documentation](https://forums.swift.org/t/function-builders/25167/357) and to understand the cryptic error messages. The feature is really limited at the moment and maybe it's for the best, to avoid overly complicated DSLs that would not be understandable. But be patient, swift folks are [working on it](https://forums.swift.org/t/function-builders-implementation-progress/32981).
