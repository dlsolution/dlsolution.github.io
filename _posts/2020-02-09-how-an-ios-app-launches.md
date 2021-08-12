---
layout: post
title: "How an iOS App Launches"
author: "Linh Vo"
tags: "UIKit"
---

Have you ever wondered about what happens under the hood between tapping an app icon and [application:didFinishLaunchingWithOptions:](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1622921-application)?

I really like to take a deep dive into topics like this that seem trivial at first but can greatly advance your understanding of the iPhone’s operating system.

Every single iPhone app that exists today could technically be described as a single call to the [UIApplicationMain(_:_:_:_:)](https://developer.apple.com/documentation/uikit/1622933-uiapplicationmain) function.

Wait, what? Yep, you read that right. Read on to find out why.

# UIApplication

If you check the `UIApplicationMain(_:_:_:_:)` function, you can see that it has a return type of `Int32`. However, this function never actually returns. It’s alive as long as your app is.

The first thing this method does is to create a class called [UIApplication](https://developer.apple.com/documentation/uikit/uiapplication). This class is very important. `UIApplication` is the centralized point of control and coordination for apps running in iOS.

Every iOS app has exactly one instance of this object. From this, you can correctly deduce that `UIApplication` is, indeed, implemented as a `singleton`. The rationale behind the singleton design pattern is to ensure that only one instance of a class is alive at any given time, so it’s the perfect option here.

When you write `UIApplication.shared`, you are accessing this singleton object.

# UIApplicationDelegate

The next thing the function does is to create the `AppDelegate`, which is basically a class that conforms to the [UIApplicationDelegate](https://developer.apple.com/documentation/uikit/uiapplicationdelegate) protocol. You’re probably already familiar with the app delegate. It’s effectively the root object of your app that you can interact with.

`UIApplicationDelegate` is a set of methods that you use to manage shared behaviors for your app.

Have you noticed the `@UIApplicationMain` mark above the class declaration? It’s there so the `UIApplicationMain(_:_:_:_:)` function knows which class is the app delegate. This class is also retained and persists during the lifetime of the app. It is then assigned as the delegate of `UIApplication`.

The application object informs the delegate of significant runtime events — for example, app launch, low-memory warnings, and app termination — giving it an opportunity to respond appropriately.

In iOS 13, some responsibilities of the `AppDelegate` are handed over to the `SceneDelegate`. The `AppDelegate` is still responsible for the app’s lifecycle, but the `SceenDelegate` is now responsible for what’s shown on the screen. It handles things like screens and windows.

# Storyboard?

At this point, the foundation of our application is already in place. `UIApplicationMain(_:_:_:_:)` can now start to construct the user interface.

The first step in this process is to find out whether our app uses the main storyboard or not. This is done by checking the `Info.plist` file to see if it contains a key called “Main storyboard file base name.”

If the app uses the storyboard, `UIApplicationMain(_:_:_:_:)` will handle instantiating the [app’s window](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1623056-window) for us. The window is an instance of `UIWindow`, which is a subclass of `UIView`. The window is basically what contains your user interface.

Once it’s instantiated, the window is assigned to the window property of the `SceneDelegate` (the `AppDelegate` used to contain this property before iOS 13).

If you take a peek inside the `scene(_:willConnectTo:options:)` method of your `SceneDelegate`, you see the following comment:

“If using a storyboard, the `window` property will automatically be initialized and attached to the scene.”

The `window` property is retained and persists for the lifetime of the app.

# The Initial View Controller

The user will not see the app’s window, and it’s not often that you need to manipulate it directly. What the user will see is the window’s [rootViewController](https://developer.apple.com/documentation/uikit/uiwindow/1621581-rootviewcontroller) property.

The `rootViewController` contains your app’s initial view controller. You can set the initial view controller by selecting a view controller in the storyboard, opening the Attributes inspector, and ticking the “Is initial view controller” checkbox.

Once this is found, it’s assigned to and retained in the `rootViewController` property. It is called root because it’s the only immediate subview of the window. All other views will be subviews of the root view.

This is when `UIApplicationMain(_:_:_:_:)` decides it’s time to call our good old `application:didFinishLaunchingWithOptions:` function.

The window is not visible yet. To make it so, the final touch is to call the `makeKeyAndVisible()` instance method.

At this point, your iOS app is ready and the fun can begin.

# No Storyboard?

However, as you probably know, it’s not mandatory to use a storyboard. In this case, you have to do in code what `UIApplicationMain` did for you out of the box.

Prior to iOS 13, the place to do this would have been `didFinishLaunchingWithOptions`. Now, however, you have the `SceneDelegate`’s `(_willConnectTo:options:)` method at your disposal for this.

What you have to do is:

- Instantiate `UIWindow` and assign it to the `SceneDelegate`’s window property.

- Instantiate your initial view controller and assign it to the `window`’s `rootViewController` property.

- Call `makeKeyAndVisible()` to present the interface to the user.

A basic implementation of the above might look something like this:

```swift
func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
    window = self.window ?? UIWindow()
    window?.backgroundColor = .white
    window?.rootViewController = ViewController()
    window?.makeKeyAndVisible()

    guard let _ = (scene as? UIWindowScene) else { return }
}
```

# Conclusion

The procedure I described above happens in the blink of an eye, and most of us don’t really think about this when developing our apps. Nevertheless, it is still a quintessential part of the application lifecycle.

Thank you for taking the time to read this article. If you have any ideas, suggestions, or questions, please leave them in the comments below.

Reference: [https://betterprogramming.pub/how-an-ios-app-launches-ae62bbd4ae8e](https://betterprogramming.pub/how-an-ios-app-launches-ae62bbd4ae8e)
