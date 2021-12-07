---
layout: post
title: "How to build a modular architecture in iOS"
author: "Linh Vo"
tags: "Clean-Architecture"
---

Over time, any code base grows along with the project evolves and matures. It creates two main constraints for developers: how to have a code well organized while keeping a build time as low as possible. Let’s see how a modular architecture can fix that.

# Modules

Starting with modules, we can represent it as a resource of code isolated from the rest of our main application. This will be then added as dependency to our iOS app.

> Creating modules also drastically increase the testability and reusability of your code.

This dependency can either be a technical aspect of your app, (Network, Storage, …) or a feature (Search, Account, …) to encapsulate the complexity.

Once defined, we can start adding up the code and resources we want to isolate.

There are only two types of way to package code: **dynamic framework** and **static libraries**.

The main difference between both is the way they are imported in the final executable. A static library is included at compile type, making a copy within the executable, where a dynamic framework is included at runtime of the executable, and never copied, making the startup time faster.

# Create a module

Now we know what can be a module, let’s create one. Assuming we create a new app for e-commerce, we need to create a specific dependency that represent the core concept of our app. I’ll call it Core.

First, I create a dynamic framework project.

{% include image.html
img="assets/2019-11-20/image.png" %}

Since it’s an e-commerce app, the core of our application is represented by the product we sell. Let’s create a simple object for that.

```swift
public struct Product {
    let name: String
    let price: Double
}
```

Since our users want to browse products, we need a way to fetch those one. Let’s create a protocol to expose this.

```swift
public protocol ProductServiceProtocol {
    func getAllProducts() -> [Product]
}

public final class ProductService: ProductServiceProtocol {
    public init() { }

    public func getAllProducts() -> [Product] {

        // imagine we fetch products from server
        let products = [Product(name: "shoe", price: 100), Product(name: "t-shirt", price: 30)]

        return products
    }
}
```

Note that we need to define `init` as `public`, otherwise it’s `internal` by default, which doesn’t make it available to use from other imports.

Our module is ready, let’s import it into an app.

# Import a module

Once our dependency is created, we can include it into our app. For this part, I created first a workspace, that makes it easier to handle two projects at once.

I added a single app to the workspace as well as my core module. They are not linked yet.

To import Core framework within the App and be able to use it, I only drag & drop the framework file to the `Linked Framework and Libraries` section of my main app.

{% include image.html
img="assets/2019-11-20/image1.png" %}

If I build the main app, I can see the Core is also part of it. That’s great, I can now use it.

{% include image.html
img="assets/2019-11-20/image2.png" %}

Using a very simple example, let’s see if I can get my products in the main app.

```swift
import Core

class ViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()

        let products = ProductService().getAllProducts()
        print(products)
    }
}
```

No warning, the console log the result as expected.

```swift
[Core.Product(name: "shoe", price: 100.0), Core.Product(name: "t-shirt", price: 30.0)]
```

Wait but I have many dependencies, some linked to each other, how can I handle them?

With more modules and dependencies, the next question is obviously how to manage them. Let’s walk through some dependency managers.

# Dependency Manager

To handle more and more dependencies, we need some ways to group and manage them.

Let’s start naive with a no dependency manager approach, all code in one repo under the same project.

{% include image.html
img="assets/2019-11-20/image3.png" %}

If it can be a good fit for small apps, it quickly become headache if you have more than one or two modules. The folders won’t help to draw a nice separation.

Going further with this approach, the next step would be to separate projects within one workspace. That’s the solution demoed above. It’s a great way to isolate code and understand visibility and responsibility of code.

{% include image.html
img="assets/2019-11-20/image4.png" %}

However, it’s still under one same git repo. When the project is going to scale, the repo can become crowded. Also consider the build time: each dependency is rebuilt with the main app.

Let’s try to separate git repo and use git sub-modules. It’s already better, the code can be reused in different other projects, but we still got the limitation of the build time.

Another angle to handle dependencies is create an umbrella framework to embed each dependency under one package to limit the built and keep a tidy workspace. **Thing is, you probably already do that if you use CocoaPods.**

If you looked at your workspace and explore a bit the Pods project, it’s the way it handles the dependencies. However, the build time can still be a bottleneck.

Finally, the other popular dependency manager is Carthage. The main difference is dependencies are built before being imported. This is the best solution to keep an optimized build.

I didn’t mentioned Swift Package Manager (or SPM) because it’s only available for macOS so far.

They are other emerging solutions for incremental build too, like Buck or Bazel, but this target a continuous integration pipeline first.

In conclusion, we saw how to isolate code into module, making it easily reusable and testable while keeping a tidy project. It’s also a great opportunity to rethink what should be exposed and what shouldn’t. Although, like any problem, there is no silver bullet, just a balance to find depending for your own project.

You can find a sample project with modules [here](https://github.com/popei69/samples).

Original post: [https://benoitpasquier.com/how-build-modular-architecture-ios/](https://benoitpasquier.com/how-build-modular-architecture-ios/)
