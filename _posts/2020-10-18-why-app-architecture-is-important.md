---
layout: post
title: "Why App Architecture is Important"
author: "Linh Vo"
tags: "Clean-Architecture"
---

You wouldn’t build a house on quicksand. Would you build an app without architecture? In this tutorial, we’ll discuss the importance of app architecture and how you can get started with it.

A while back I got an email from an iOS developer. He was tasked with extending an app with new features. The app was built by someone else.

He ran into these issues:

- “When I change one thing in the code, another part breaks!”

- “I can’t find any feature of the app. They are all over the place.”

- “There is no uniform structure for this app, so how can I extend it?”

We emailed back and forth, and came to the conclusion that the app lacked architecture.

Once we had that sorted out, the developer started the tedious process of rebuilding most of the app. He refactored code, made uniform APIs and applied a thoughtful app architecture to the project.

# When Your App Breaks In Random Places

This joke always gets me:

```markdown
99 little bugs in the code
99 little bugs in the code
Take one down, patch it around
127 little bugs in the code…
```

I like to think that the person who wrote this “poem” (by lack of a better word) wrote code without an architecture.

You have a bug. You fix it. You now have more bugs. Code breaking in random places. How is that even possible!?

Technically, apps consist of many smaller components put together. A database, user interfaces, a back-end, and app features like social sharing, user authentication – they are the components that make up an app.

Every one of those components has more smaller components. When you go one level deeper, a database has objects it saves, code to read and write, code to cache data, code to interpret database queries, and so on.

Every one of those components has even more smaller components. The database caching mechanism has code to empty the cache, to add cached objects, to remove them, and code to keep an eye on available memory.

And so on. Components on top of components on top of components, comprising an entire system. This system is the app.

At this point, there aren’t any problems yet. You can very well build an app that uses all these components. Most of the time, you’re only working directly with the high-level components of an app anyway!

Where does it go wrong?

Let’s say you’re building an app that has a list of movies. Your app saves `Movie` objects in the database. Movies have properties: a title, a director, a cast, and so on.

```swift
struct Movie {
    var title:String
    var director:String
    var cast:[String]
}
```

At one point you decide to add TV shows to your app. Because movies and TV shows are so much alike, you decide to use the `Movie` objects to save TV shows.

You use a function called `isShow()` to indicate a TV show. It returns `true` when the movie object is a TV show.

This works OK for some time. After a year a new developer comes along to work on this app. They add a new property to TV shows called `episode` and adjust the `isShow()` function to only return `true` when the episode property contains a value.

Suddenly, the entire database breaks. Objects that are TV shows show up as movies, and objects that are movies don’t show up at all…

# Encapsulation, Abstraction and Documentation

What caused this bug with the `isShow()` function?

Both developers relied on the `isShow()` function to work correctly. Code that depended on the function broke, because its implementation changed. When you change the underlying mechanism of a function, can you still be sure that the code that depends on that function still works OK?

You could argue that properly documenting the dependencies of the `isShow()` function would have prevented this bug. In hindsight, that seems a reasonable solution. If only the new developer knew what `isShow()` was used for.

When building systems, we usually only look in the rearview mirror when things break. How do you look into the future, avoiding this bug before it even happens?

A good app architecture is based on 3 ideas:

- **Encapsulation:** clearly defining the boundaries of a component

- **Abstraction:** hiding complexity from the developer

- **Documentation:** communicating expected input and resulting output

Encapsulation means that you clearly define the boundaries of a component. You make a “capsule” of your code. You can only use the component via formal functions that encapsulate data, like `setTitle(_:)` or `isMovie()`.

The complex logic inside the movie objected is abstracted away or “hidden”. You or another developer don’t have direct access to the object’s data. You can’t change its implementation after it has been finalized.

Abstraction is crucial in software development. With it, you can avoid exposing too much complexity to a developer (or yourself).

This is an advantage. When the “capsule” is finished, you don’t have to worry about its implementation any more. With unit testing you can test the integrity of the capsule and keep it compatible with the rest of your code.

You also want the components you define, and abstract, to be extensible. A smart approach to avoid the issue with `isShow()` is extending or subclassing the `Movie` type with a `TVShow` type. You pass ordinary `Movie` types to existing code, and the new TV show code uses your own new `TVShow` type.

Documenting your project is also important, of course. Properly documenting your code means adding code comments where necessary. You add so-called “Docblocks” to functions, properties and classes. They describe purpose of a component, the expected input, and the resulting output.

It’s common to also describe other aspects of an app codebase, like how to get it up and running, where to find usernames and passwords, and what open source libraries the project depends on.

> It’s important to note here that architectural patterns like MVC, MVVM and VIPER are valuable because they define relationships between components of your app. On a fundamental level they still use the principles of encapsulation and abstraction, among many others.

# How To Manage Dependencies

What about dependency? If your code doesn’t depend on another part of your code, you can avoid bugs altogether. Right?

A commonly heard argument, when things break, is “Why did you add that dependency in the first place!?” A few examples come to mind, such as when a simple JavaScript library broke the NPM ecosystem and when the Facebook SDK for iOS apps caused a ton of crashes.

It’s easy to think that depending on a component is at fault here. If your code functions independently, you’re obviously not affected when a library or framework causes crashes. But is that a realistic choice?

Code inevitably builds upon other code, and is consequently dependent on that code. You can’t code in a vacuum with zero dependencies. The real choice here is managing dependencies, and using tools and approaches that limit your code’s exposure.

Give these ideas some thought:

- **Strike a balance** between writing your own implementation (i.e., left pad) and relying on 3rd-party code (i.e., don’t build your own database, use something like Realm)

- Use approaches like dependency injection to **decouple your code** from the dependency. Ask yourself: “How can I make switching to another library in the future as easy as possible?”

- **Create and run unit- and integration tests** to check if a piece of code still functions like it’s supposed to. It’s not a panacea, esp. not for the above 2 points, but it’s a great early-detection system for implementation mistakes and changes.

- **Audit, maintain and document your work.** Make your code’s dependencies explicit, in the code as well as outside it, in the documentation. Investigate what dependencies your code’s dependencies depend on, and if that’s code you want to put in your project.

Avoiding one bug that breaks hundreds of apps isn’t the goal here; the goal is avoiding hundreds of different bugs that break your one app. You build robustness and anti-fragility into your apps by choosing a good approach, not by avoiding dependencies in the first place.

Let’s get back to app architecture.

# Don’t Stick With Your App Architecture

App architecture is in constant flux. For instance, the VIPER app architecture once rose in popularity among iOS developers. Now that there’s SwiftUI, we hear praises of declarative programming and managing state properly.

Model-View-Controller (MVC) has often been scorned by the developer community. It’s easy to create massive classes of code with MVC, if you don’t follow certain guiding principles.

Ironically, the principles of MVC lie at the foundation of so many other architectural patterns. In defense of MVC: don’t judge the architecture for the mistakes of the developer.

As a developer it’s important to master app architecture, design patterns and software engineering principles to a certain degree. You also need to make mistakes now so you can avoid (similar) mistakes in the future. Mistakes lead to mastery, which is why you need to be wary of extremes like “Never use singletons!” or “MVC sucks!” or “Gitflow is The Way!”

When you grasp the foundations that underlie many of these architectures, it’s much easier to pick up a novel architectures like VIPER. It often doesn’t work the other way around. And by making the journey yourself, you gain insights that you miss out on if you only listened to “experts”.

As an iOS developer you can choose from a wide range of architectures, design patterns and engineering principles. It’s too complex to go into all of them at this point. Make your pick!

### Software development principles:

- [SOLID](https://learnappmaking.com/single-responsibility-principle-solid/)
- [DRY](https://learnappmaking.com/single-responsibility-principle-solid/)
- [KISS](https://learnappmaking.com/single-responsibility-principle-solid/)
- [GRASP](<https://en.wikipedia.org/wiki/GRASP_(object-oriented_design)>)

### Architectural patterns:

- [MVC](https://learnappmaking.com/model-view-controller-mvc-swift/)
- [MVVM](https://learnappmaking.com/model-view-controller-mvc-swift/)
- [MVP](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93presenter)
- [VIPER](https://www.objc.io/issues/13-architecture/viper/)

### Design patterns:

- Extension
- Builder and Factory
- Singleton
- Appearance
- Iterator
- Observer-Observable
- Delegation
- Coordinator

### Programming paradigms:

- [Procedural Programming](https://en.wikipedia.org/wiki/Procedural_programming)
- [Object-Oriented Programming](https://en.wikipedia.org/wiki/Object-oriented_programming)
- [Functional Programming](https://en.wikipedia.org/wiki/Functional_programming)
- [Reactive Programming](https://hackernoon.com/functional-reactive-programming-in-swift-f67a0939266b)

# What’s Next?

Back to the apps we’re working on. Here’s what we run into as iOS developers:

- “When I change one thing in the code, another part breaks!”

- “I can’t find any feature of the app. They are all over the place.”

- “There is no uniform structure for this app, so how can I extend it?”

Creating an app with a sensible architecture solves these problems. Your app’s architecture gives its components structure and formalizes how they talk with each other.

As a result you get an app that …

- … doesn’t easily break, has fewer bugs, and is easy to maintain.

- … has a codebase that’s easy to understand and navigate.

- … has reusable code and can be extended with new features.

You could say that we, as iOS developers, have created rules that we all follow and understand. When you take on a project that someone else built, or continue on your own projects, you can rely on the integrity of the work because of these rules.

You’ve got a choice now:

1. Stick with what works for others, and what’s popular now

2. Pick an architecture, learn more about it, and make lots of mistakes

By the way, it’s never too late to architect your app. You want to do it sooner than later, of course.

When you refactor your code you essentially rewrite the code that doesn’t follow your architectural design anymore. Making this your practice is essential!

You wouldn’t build a house on quicksand. You wouldn’t build an app without putting some thought into its architectural design.

Ultimately, you spend less time fixing bugs, more time building great apps, which means happy customers, happy employer, and that means a happy you.

Original Post: [https://learnappmaking.com/why-app-architecture-is-important/](https://learnappmaking.com/why-app-architecture-is-important/)
