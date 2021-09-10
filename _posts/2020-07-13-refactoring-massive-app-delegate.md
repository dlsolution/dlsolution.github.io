---
layout: post
title: "Refactoring Massive App Delegate"
author: "Linh Vo"
tags: "Clean-Architecture"
---

App delegate connects your app and the system and is usually considered to be the core of every iOS project. The common tendency is that it keeps growing as the development goes, gradually sprouting with new features and responsibilities, being called here and there and eventually turning into spaghetti code.

The cost of breaking anything inside the app delegate is extremely high due to the influence it has over your app. Undoubtedly, keeping this class clean and concise is crucial for the healthy iOS project.

In this article let’s have a look at different methods of how app delegates can be made concise, reusable and testable.

# Problem Statement

The app delegate is the root object of your app. It ensures the app interacts properly with the system and with other apps. It’s very common for app delegate to have dozens of responsibilities which is makes it difficult to change, expand and test.

Even [Apple encourages](https://developer.apple.com/documentation/uikit/uiapplicationdelegate) you to put at least 3 responsibilities in your `AppDelegate`.

By investigating a couple of dozens of most popular open-source iOS apps I composed a list of responsibilities that are often put into app delegates. I am sure each of us either wrote such code or had a luck to support a project with similar mess.

By investigating a couple of dozens of [most popular open-source iOS apps](https://github.com/dkhamsing/open-source-ios-apps) I composed a list of responsibilities that are often put into app delegates. I am sure each of us either wrote such code or had a luck to support a project with similar mess.

- Initialize numerous third-party libraries
- Initialize Core Data stack and manage migrations
- Configure app state for unit or UI tests
- Manage UserDefaults: setups first launch flags, save and load data
- Handle Home screen quick actions
- Manage notifications: request permissions, store token, handle custom actions, propagate notifications to the rest of the app
- Configure UIAppearance
- Manage app badge counter
- Manage background tasks
- Manage UI stack configuration: pick initial view controller, perform root view controller transitions
  Play audio
- Manage analytics
- Print debug logs
- Manage device orientation
- Conform to various delegate protocols, especially from third parties
- Prompt alerts

I am sure the list is not limited to the aforementioned.

Such bloated app delegates fall under the definitions of the Blob anti-pattern and spaghetti code. Obviously, supporting, expanding and testing such class is very complex and error-prone. For example, looking at [Telegram’s AppDelegate’s source code](https://github.com/peter-iakovlev/Telegram/blob/public/Telegraph/TGAppDelegate.mm) inspires so much terror in me.

Let’s call such classes **Massive App Delegates** to follow a renown term Massive View Controller that describes very similar view controller symptoms.

# Solution

After we agreed that the problem of Massive App Delegate exists and is highly important, let’s take look at possible solutions or how I call them ‘recipes’.

Each recipe must satisfy next criteria:

- Follow [single responsibility principle](https://en.wikipedia.org/wiki/Single_responsibility_principle).

- Easy to expand.

- Easy to test.

# Recipe #1: Command Design Pattern

The **command** pattern describes objects, called commands, that represent a single action or event. Such objects encapsulate all parameters required to trigger themselves, thus the caller of a command does not posses any knowledge about what the command does and who is the responder.

For each app delegate responsibility we define a command. The name of the command suggests its designation.

```swift
protocol Command {
    func execute()
}

struct InitializeThirdPartiesCommand: Command {
    func execute() {
        // Third parties are initialized here
    }
}

struct InitialViewControllerCommand: Command {
    let keyWindow: UIWindow

    func execute() {
        // Pick root view controller here
        keyWindow.rootViewController = UIViewController()
    }
}

struct InitializeAppearanceCommand: Command {
    func execute() {
        // Setup UIAppearance
    }
}

struct RegisterToRemoteNotificationsCommand: Command {
    func execute() {
        // Register for remote notifications here
    }
}
```

Next we define `StartupCommandsBuilder` that encapsulates the details about how the commands are created. `AppDelegate` calls the builder to initialize commands and then triggers them.

```swift
// MARK: - Builder
final class StartupCommandsBuilder {
    private var window: UIWindow!

    func setKeyWindow(_ window: UIWindow) -> StartupCommandsBuilder {
        self.window = window
        return self
    }

    func build() -> [Command] {
        return [
            InitializeThirdPartiesCommand(),
            InitialViewControllerCommand(keyWindow: window),
            InitializeAppearanceCommand(),
            RegisterToRemoteNotificationsCommand()
        ]
    }
}

// MARK: - App Delegate
@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {
    var window: UIWindow?

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
        StartupCommandsBuilder()
            .setKeyWindow(window!)
            .build()
            .forEach { $0.execute() }

        return true
    }
}
```

New commands can be added directly to the builder without any changed to `AppDelegate`.

Our solution satisfies the defined criteria:

- Each command has single responsibility.

- It is easy to add new commands without changing `AppDelegate`’s code.

- Commands can be easily tested in isolation.

# Recipe #2: Composite Design Pattern

**Composite** design pattern allows to treat hierarchies of objects as if it were a single instance. A prominent example in iOS is `UIView` with its subviews.

The main idea is to have a composite and leaf app delegates each having one responsibility, where the composite propagates all methods to the leafs.

```swift
typealias AppDelegateType = UIResponder & UIApplicationDelegate

class CompositeAppDelegate: AppDelegateType {
    private let appDelegates: [AppDelegateType]

    init(appDelegates: [AppDelegateType]) {
        self.appDelegates = appDelegates
    }

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
        appDelegates.forEach { _ = $0.application?(application, didFinishLaunchingWithOptions: launchOptions) }
        return true
    }

    func application(_ application: UIApplication, didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
        appDelegates.forEach { _ = $0.application?(application, didRegisterForRemoteNotificationsWithDeviceToken: deviceToken) }
    }
}
```

Next, implement leaf `AppDelegate` that do the actual work.

```swift
class PushNotificationsAppDelegate: AppDelegateType {
    func application(_ application: UIApplication, didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
        // Registered successfully
    }
}

class StartupConfiguratorAppDelegate: AppDelegateType {
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
        // Perform startup configurations, e.g. build UI stack, setup UIApperance
        return true
    }
}

class ThirdPartiesConfiguratorAppDelegate: AppDelegateType {
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
        // Setup third parties
        return true
    }
}
```

We define `AppDelegateFactory` that encapsulates the creation logic. Our main `AppDelegate` creates the composite delegates via the factory and passes to them all the method calls.

```swift
enum AppDelegateFactory {
    static func makeDefault() -> AppDelegateType {
        return CompositeAppDelegate(appDelegates: [PushNotificationsAppDelegate(), StartupConfiguratorAppDelegate(), ThirdPartiesConfiguratorAppDelegate()])
    }
}

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {

    var window: UIWindow?
    let appDelegate = AppDelegateFactory.makeDefault()

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
        _ = appDelegate.application?(application, didFinishLaunchingWithOptions: launchOptions)
        return true
    }

    func application(_ application: UIApplication, didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
        appDelegate.application?(application, didRegisterForRemoteNotificationsWithDeviceToken: deviceToken)
    }
}
```

It satisfies the criteria we defined at the beginning:

- Each sub-app-delegate has single responsibility.

- It is easy to add new `AppDelegate` without changing the main ones code.

- App delegates can be easily tested in isolation.

# Recipe #3: Mediator Design Pattern

**Mediator** object encapsulates the interaction policies in a hidden and unconstraining way. Objects being manipulated by mediator have no idea it exists. It sits quietly behind the scenes and imposes its policies without their permission or knowledge.

If you want to learn more about this pattern, I recommend checking [Mediator Pattern Case Study](https://www.vadimbulavin.com/mediator-pattern-case-study/).

Let’s define `AppLifecycleMediator` that propagates `UIApplication` life cycle events to underlying listeners. The listeners must conform to `AppLifecycleListener` protocol that can be expanded with new methods if needed.

```swift
// MARK: - AppLifecycleListener
protocol AppLifecycleListener {
    func onAppWillEnterForeground()
    func onAppDidEnterBackground()
    func onAppDidFinishLaunching()
}

// MARK: - Mediator
class AppLifecycleMediator: NSObject {
    private let listeners: [AppLifecycleListener]

    init(listeners: [AppLifecycleListener]) {
        self.listeners = listeners
        super.init()
        subscribe()
    }

    deinit {
        NotificationCenter.default.removeObserver(self)
    }

    private func subscribe() {
        NotificationCenter.default.addObserver(self, selector: #selector(onAppWillEnterForeground), name: .UIApplicationWillEnterForeground, object: nil)
        NotificationCenter.default.addObserver(self, selector: #selector(onAppDidEnterBackground), name: .UIApplicationDidEnterBackground, object: nil)
        NotificationCenter.default.addObserver(self, selector: #selector(onAppDidFinishLaunching), name: .UIApplicationDidFinishLaunching, object: nil)
    }

    @objc private func onAppWillEnterForeground() {
        listeners.forEach { $0.onAppWillEnterForeground() }
    }

    @objc private func onAppDidEnterBackground() {
        listeners.forEach { $0.onAppDidEnterBackground() }
    }

    @objc private func onAppDidFinishLaunching() {
        listeners.forEach { $0.onAppDidFinishLaunching() }
    }
}

extension AppLifecycleMediator {
    static func makeDefaultMediator() -> AppLifecycleMediator {
        let listener1 = ...
        let listener2 = ...
        return AppLifecycleMediator(listeners: [listener1, listener2])
    }
}
```

Now it can be added to `AppDelegate` with 1 line of code.

```swift
@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {
    var window: UIWindow?

    let mediator = AppLifecycleMediator.makeDefaultMediator()

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
        return true
    }
}
```

The mediator automatically subscribes to all events. `AppDelegate` only needs to initialize it once and let it do its work.

It satisfies the criteria we defined at the beginning:

- Each listener has single responsibility.

- It is easy to add new listeners without changing `AppDelegate`’s code.

- Each listener as well as the mediator itself can be easily tested in isolation.

# Summary

We agreed that most of `AppDelegate` are unreasonably big, overcomplicated and have too much responsibilities. We called such classes Massive App Delegates.

By applying software design patterns, Massive App Delegate can be split into several classes each of which has single responsibility and can be tested in isolation.

Such code is easy to change, as it will not result in a cascade of changes all over your app. It is very flexible and can extracted and reused in future.

Original post: [https://www.vadimbulavin.com/refactoring-massive-app-delegate](https://www.vadimbulavin.com/refactoring-massive-app-delegate)
