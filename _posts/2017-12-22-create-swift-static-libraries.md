---
layout: post
title: "Create Swift Static Library"
author: "Linh Vo"
tags: "UIKit"
---

In this article, we will consider creating a static library and its integration to another project.

# What is a static library?

A static library is a collection of compiled source code files. Let’s say we have `FileA.swift`, `FileB.swift` and `FileC.swift`. With a static library we can compile these files and wrap them inside a single file containing all of these. The file has has an extension of `.a`; short for archive. Sort of like getting some pages and making a book out of it.

{% include image.html
img="assets/2017-12-22/image2.png"
title=""
caption="" %}

# Static library vs Dynamic frameworks

iOS allows us two methods of grouping code together for distribution:

- dynamic frameworks

- static libraries

The difference between the two that we will cover in this post is:

- dynamic frameworks can hold resources (i.e images), static libraries can’t

- dynamic frameworks are loosely linked to the app, static libraries are hard linked to the app

To explain linking further let’s look at examples for each type of code grouping mechanisms.

## Dynamic framework linking

When our app consumes code dynamically the code is not loaded with the app when the app is launched. The system loads our app which in turns tells the system that it requires to consume some code to run. The system then loads the necessary chunks of code that our app relies on. All of this happens at runtime.

An example of this is `UIKit`. iOS apps make use of `UIKit`. However `UIKit` is not included with each app. Rather the app has reference to it and the functionality it requires to function but not the actual code. `UIKit` is included in the operating system. The system loads `UIKit` if it was not already loaded.

{% include image.html
img="assets/2017-12-22/image11.png"
title=""
caption="" %}

What about dynamic frameworks that are not included within the operating system? What about frameworks that we build and distribute with our apps?

ConsumerApp can’t work without `MyDynamicFramework` or it will crash. Thus the system must load it. It must be thus included with the app package.

Xcode creates a directory within the app package called `Frameworks`. This directory is where to host dynamic frameworks required by the app.

{% include image.html
img="assets/2017-12-22/image12.png"
title=""
caption="" %}

{% include image.html
img="assets/2017-12-22/image13.png"
title=""
caption="" %}

What if the framework is not included within our app’s package `Framework` directory or within the system? App crash at runtime 💥.

## Static library linking

When our app consumes code statically the code that it consumes gets copied into the app executable binary.

{% include image.html
img="assets/2017-12-22/image14.png"
title=""
caption="" %}

When the system loads the app, the static library functionality is loaded with it as single executable.

{% include image.html
img="assets/2017-12-22/image15.png"
title=""
caption="" %}

# Why use static libraries?

Some reasons to use static libraries similar as for dynamic frameworks:

- code reusability

- hiding code that is not related to the app (using private and internal access)

For both static libraries and dynamic frameworks if we distribute only the compiled form then we can also:

- avoid sharing our secret code and only distribute the functionality

- reduce app compilation time

But why use static libraries over dynamic frameworks? Apps with static libraries load faster than those with dynamic frameworks. Let’s dive a little deeper to understand why.

Dynamic frameworks have the advantage of being loaded into memory lazily at runtime and the possibility of being shared with multiple processes. That works great if the dynamic framework is used by multiple apps in iOS.

iOS system dynamic frameworks are used by multiple apps and these are likely to be loaded before your app launches thus saving app launch time. However in case of embedded dynamic frameworks in apps (these are the ones included inside your app and not by operating system) the launch times can be much slower. How much slower?

To test it out I wrapped three Swift files in two different targets:

- Dynamic framework

- Static library

I created an empty app. I configured the app to print loading times to console. Then I linked the app to a dynamic framework. I ran the app on a simulator from cold(was not cached in memory). The results:

```
Total pre-main time: 1.0 seconds (100.0%)
        dylib loading time: 193.54 milliseconds (18.3%)
        rebase/binding time: 760.93 milliseconds (71.9%)
        ObjC setup time:  60.19 milliseconds (5.6%)
        initializer time:  42.81 milliseconds (4.0%)
        slowest intializers :
            libSystem.B.dylib :  18.22 milliseconds (1.7%)
```

I repeated the process with one difference. I linked the app to a static framework. The static framework contains the same code as the dynamic framework as in the previous run. The results:

```
Total pre-main time: 290.34 milliseconds (100.0%)
        dylib loading time:  44.15 milliseconds (15.2%)
        rebase/binding time:  54.60 milliseconds (18.8%).
        ObjC setup time:  60.76 milliseconds (20.9%)
        initializer time: 130.68 milliseconds (45.0%)
        slowest intializers :
            libSystem.B.dylib :   3.08 milliseconds (1.0%)
            libMainThreadChecker.dylib : 118.62 milliseconds (40.8%)
```

The app launch time was nearly 4 times faster when linked statically!

# Create Static Library Project

Let’s create a library project first. Open Xcode and select `Cocoa Touch Static Library`.

{% include image.html
img="assets/2017-12-22/image1.png"
title=""
caption="" %}

Give it a name and select `Swift` as the development language. In our case, we will call it `Networking` and assume that it will contain the code to communicate with a back end.

Press `Cmd+N` and select `Swift file`.

{% include image.html
img="assets/2017-12-22/image3.png"
title=""
caption="" %}

Give it a name, in our case, we will create a dummy class to communicate with the authentication part of the back-end API and call it `AuthenticationService`.

{% include image.html
img="assets/2017-12-22/image4.png"
title=""
caption="" %}

Paste the code below to the created file:

```swift
import Foundation

public class AuthenticationService {

    public init() {
    }

    public func login(_ username: String, _ password: String) -> String {
        return UUID().uuidString
    }
}
```

Important: Make sure you made your class and methods `public`. Otherwise, those methods and classes won’t be available from a project that you are going to use your library.

Moreover, you have to write an implementation for `init` and explicitly make it `public`.

Then, press `Cmd+B` to build the project, and in the `Project Navigator` in the `Products` section, you will find a binary file called `libNetworking.a`.

{% include image.html
img="assets/2017-12-22/image5.png"
title=""
caption="" %}

That’s actually it for static library creation. You’ve just built your library for the simulator. You can also build it for mobile devices, but in this tutorial we will skip it to keep the tutorial simple and concentrate on the general idea.

# Integration to Another Project

OK, now that we have a binary code file, let’s use it in another project.

In Xcode, press `Cmd+Shift+N` and from the window that appears, select `Single View Project`. Press `Next`.

Call the project whatever you want, for instance, `SimpleApplication`, and press `Next` again.

After the project is created, we are ready to integrate the library we created in the previous section.

The first thing is that we need to add the compiled library’s files to the new project. To reach that, right-click on the project name in the project navigator and select `Show in Finder`.

In the opened Finder’s window, create a new folder in the project’s root folder and name it `lib`.

{% include image.html
img="assets/2017-12-22/image6.png"
title=""
caption="" %}

Go back to the library project in Xcode, find the file `libNetworking.a` in the project navigator, right-click on it, and select `Show in Finder`.

{% include image.html
img="assets/2017-12-22/image7.png"
title=""
caption="" %}

You will find the `libNetworking.a` file and `Networking.swiftmodule` folder there. Copy and paste them both to the `SimpleApplication/lib` folder.

{% include image.html
img="assets/2017-12-22/image8.png"
title=""
caption="" %}

Move back to Xcode and right-click on the project’s name again, select `Add files to "SimpleApplication"`.

In the window that appears, make sure that the `Create Groups` radio button is selected, select `lib` folder, and press `Add`.

{% include image.html
img="assets/2017-12-22/image9.png"
title=""
caption="" %}

Now you have the library for your new project and Xcode even did some integration for you. Now your task is to check if it did it correctly and perform some additional steps.

Select the project’s name in project navigator, then select `General`, and select your’s application target.

The section `Linked Frameworks and Libraries` has to contain a line with `libNetworking.a`. If it does not, press the `+` button and select it manually. Make sure that the `Required` status is selected.

After that, go to the `Build Phases` tab, expand `Link Binary with Libraries`, and make sure that it contains a line with `libNetworking.a`. If it does not, again, add it manually and set it to required.

Finally, we’ve reached the last, and maybe the most sensitive integration step — setting paths. It’s a step where it’s very easy to make a mistake, so be neat and careful.

In the `Build Settings` tab, search `Library Search Path`, add a new one, `$(PROJECT_DIR)/lib`. Do the same for Import Paths.

{% include image.html
img="assets/2017-12-22/image10.png"
title=""
caption="" %}

Then, go to your `ViewController` class and add the code there:

```swift
import UIKit
import Networking // 1

class ViewController: UIViewController {
    let authService = AuthenticationService() // 2

    override func viewDidLoad() {
        super.viewDidLoad()
        let token = authService.login("user", "S7eo#0-2K&b") // 3
        print("token: \(token)") // 4
    }
}
```

Let’s discuss this small piece of code.

- `// 1` — Import Networking library. It will give you the ability to use the code from your library in ViewController.

- `// 2` — Create an instance of a class that belongs to the library.

- `// 3` — Make a call for a method that belongs to the library and save the result to a local variable. For sure, we send some dummy data and receive a dummy response here.

- `// 4` — print the result to the console

# Summary

In this post we have learnt:

- what static libraries are

- what resource bundles are

- why use static libraries

- why use static libraries over dynamic frameworks

- how to build a static library

Reference: [https://medium.com/onfido-tech/reusing-code-and-resources-with-swift-static-libraries-and-resource-bundles-d070e82d3b3d](https://medium.com/onfido-tech/reusing-code-and-resources-with-swift-static-libraries-and-resource-bundles-d070e82d3b3d)
