---
layout: post
title: "Dependency injection and Generics to create a modular app in Swift"
author: "Linh Vo"
tags: "Clean-Architecture"
---

When we talk about modular app, we rarely mention how complex it can be over time and get out of hand. In most cases, importing frameworks into one another is a reasonable solution but we can do more. Let’s explore how with dependency inversion in Swift and how to create order into our components.

A while back, I shared the first steps [how to build a modular architecture for an iOS app](https://dlsolution.github.io/2019-11-20/how-to-build-a-modular-architecture). By creating a framework and importing into the main app, I could decouple the business logic from the UI layer. Simple and easy, it works great.

But then, when it comes to bigger projects, it gets a bit different. We can easily reach dozens of modules, which themselves depends of others. It exposes the code to break due to necessary cascading changes.

The idea of decoupling the layers and logic is the great, but in the execution, if you change a low level framework method signature, how many dependent frameworks you need to change? Ideally, if it’s decoupled, only one.

Let’s see it through an example.

{% include image.html
img="assets/2019-12-07/image.png" %}

I have 2 frameworks, an `Analytics` used for company metrics, and a `LoginUI` to isolate Login journey. At the moment `LoginUI` depends from `Analytics`, it imports it as dependency.

```swift
// Analytics
public protocol EventHandlerProtocol {
    func send(key: String, params: [String: Any]?)
}
public class EventHandler: EventHandlerProtocol {
    func send(key: String, params: [String: Any]?) {
        print("SEND - \(event.key)")
    }
}

// LoginUI
import Analytics
public class LoginViewController: UIViewController {

    let eventHandler: EventHandlerProtocol
    public init(eventHandler: EventHandlerProtocol) {
        self.eventHandler = eventHandler
        super.init(nibName: nil, bundle: nil)
    }

    public required init?(coder: NSCoder) {
        fatalError("not implemented")
    }

    public override func viewDidLoad() {
        super.viewDidLoad()
        eventHandler.send("login.screen", params: nil)
    }
}
```

The `EventHandler` is the implementation of the interface `EventHandlerProtocol`. The analytics logic is decoupled. So far so good.

On the other end, I use dependency injection in the `LoginViewController` to pass along the event handler from the interface. When the `viewDidLoad` is executed, we get a log.

The final piece to make it work is in the main app to glue it together.

```swift
import Analytics
import LoginUI

func makeLoginViewController() -> UIViewController {
    return LoginViewController(eventHandler: EventHandler())
}
```

So far so good. All the implementation is hidden from the frameworks, and the app puts all the pieces together.

But here comes trouble.

We have to make a change in the Analytics to pass a data structure rather than `key` and `params`. Turns out, it will require to make changed in `LoginUI` framework. Actually, it requires to make changes across any feature components that uses it. Tough one.

What could we have done differently to support this changes in future without breaking anything? That’s where come dependency inversion.

Part of SOLID principle, Dependency Inversion Principle aims to loosen-up the coupling between two components. Instead of one depending on the other, both relies on the same interface.

Following this direction, on paper, `Analytics` would depend of `AnalyticsProtocol`, so does `LoginUI`.

{% include image.html
img="assets/2019-12-07/image1.png" %}

```swift
// AnalyticsProtocol
public protocol EventHandlerProtocol {
    func send(key: String, params: [String: Any]?)
}

// Analytics
import AnalyticsProtocol
public class EventHandler: EventHandlerProtocol {
    func send(key: String, params: [String: Any]?) {
        print("SEND - \(event.key)")
    }
}

// LoginUI
import AnalyticsProtocol
public class LoginViewController: UIViewController {

    let eventHandler: EventHandlerProtocol
    // same code ...
}
```

It doesn’t look much but we’ve decoupled a little more `Analytics` and `LoginUI` modules. We created a middle layer for the abstraction and we could replace `Analytics` implementation tomorrow with another similar component, as long as it follows `AnalyticsProtocol`.

However, the problem stays the same: if the interface has to change, we have still as many dependencies to change. So what else can we do?

One way I found interesting is through generic type. With a small refactoring, we can completely remove the dependencies between components.

At the moment, `LoginUI` still depends of a `AnalyticsProtocol`, but what if it didn’t. We could create a bridge in the main app to connect the two components without an extra interface.

{% include image.html
img="assets/2019-12-07/image2.png" %}

Let’s revisit the code to check this out.

```swift
// LoginUI
public struct LoginEvent {
    public let key: String
    public let params: [String: Any]?
}

public protocol LoginEventHandler {
    func send(_ event: LoginEvent)
}

public class LoginViewController: UIViewController {

    let eventHandler: LoginEventHandler
    public init(eventHandler: LoginEventHandler) {
        self.eventHandler = eventHandler
        super.init(nibName: nil, bundle: nil)
    }

    public required init?(coder: NSCoder) {
        fatalError("not implemented")
    }

    public override func viewDidLoad() {
        super.viewDidLoad()
        eventHandler.send(LoginEvent(key: "login.screen", params: nil))
    }
}
```

In this new version, we create a new interface, and still handle the tracking the same way. But how is it implemented?

We need first to update the `Analytics` implementation for a more generic approach.

```swift
// Analytics
public protocol EventProtocol {
    var key: String { get }
    var params: [String: Any]? { get }
}

public protocol EventHandlerProtocol {
    func send<T: EventProtocol>(_ event: T)
}

public class EventHandler: EventHandlerProtocol {
    public init() { }

    public func send<T: EventProtocol>(_ event: T) {
        // TODO private implementation
        print("SEND - \(event.key)")
    }
}
```

With this new interface based on generic type, the implementation can be much more modular as long as the event follows the same interface.

> Wait, but how does `Analytics` and `LoginUI` communicates?

That’s the beauty part, they are not exposed to each other. The app will create the connection

```swift
// App
import Analytics
import LoginUI

// extends LoginUI events to Analytics event definition
extension LoginEvent: EventProtocol { }

// extends Analytics handler to LoginUI handler definition
extension EventHandler: LoginEventHandler { }

func makeLoginViewController() -> UIViewController {
    return LoginViewController(eventHandler: EventHandler())
}
```

Creating this bridge allows us to avoid importing `Analytics` into `LoginUI` at the first place. If one of those components had to change tomorrow, it doesn’t automatically enforce to cascade to many more changes.

In this case, similar to Dependency Inversion Principle, we loose-up the coupling of those two modules enough to be flexible and replaceable. It also allow new components to do the same as well.

At the end of the day, only the application knows how those layers are interfaced and we can interface a lot more components without risking creating unnecessary dependencies between one another.

{% include image.html
img="assets/2019-12-07/image3.png" %}

As usual, there is never one solution for all problems, maybe your app architecture design works already well today, but it’s still interesting to know what’s out there that might help you find a new solution to face any upcoming problems.

In this case, we created a modular app and leveraged generic type to follow the dependency inversion principle. Each layer is separated, testable and maintainable. Pretty cool.

What about you? What are you tips and tweaks to face complex architectures? Feel free to share.

Original post: [https://benoitpasquier.com/modular-app-dependency-injection-generics-swift/](https://benoitpasquier.com/modular-app-dependency-injection-generics-swift/)
