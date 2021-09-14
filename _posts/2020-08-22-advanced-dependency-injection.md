---
layout: post
title: "Advanced Dependency Injection"
author: "Linh Vo"
tags: "Clean-Architecture"
---

Dependency injection is a broad technique which can be implemented differently. In this article let’s learn the core principles of dependency injection and implement commonly used patterns such as dependency injection container, service locator and ambient context.

# Defining Dependency Injection

**Dependency Injection (DI)** is a technique which allows to populate a class with objects, rather than relying on the class to create the objects itself.

To get a better understanding of then notion, let’s see what experts are saying about DI.

Martin Fowler, the originator of the term, defines it as follows [1]:

> The basic idea of the Dependency Injection is to have a separate object, an assembler, that populates a field in the […] class with an appropriate implementation for the interface […].

Robert Martin, the well-known author and speaker, comes with next explanation [2]:

> Dependency Injection is just a special case of Dependency Inversion.

Mark Seemann, the author of Dependency Injection in .NET, has broader definition [3]:

> […] DI is simply a set of patterns and principles that describe how we can write loosely coupled code.

As Robert Martin mentions, dependency injection cannot be considered without [Dependency inversion principle](https://en.wikipedia.org/wiki/Dependency_inversion_principle). The principle states that implementation details should depend on and implement higher level abstractions, rather than the other way around. It is foundational when creating loosely-coupled applications, which aligns with Mark Seemann’s definition.

Martin Fowler describes DI from the implementation standpoint: a class depends on an **interface**, having an **implementation** supplied from the **outside**. This highlights three actors, involved in dependency injection:

- **Injector** – instantiates dependency and wires it with a client.

- **Dependency** – an object, required by a client in order to function correctly.

- **Client** – an object, where dependency is injected.

After understanding the concept of dependency injection, let’s see how it is implemented in Swift.

# Client Patterns

There are four ways how client can receive a dependency:

- Initializer.
- Property.
- Interface.
- Ambient context.

Let’s study how each option is implemented in Swift and which pros and cons does it have.

## Initializer Injection

Description: dependencies are passed via initializer.

When to use: whenever possible. Fits best when the number of dependencies is low or the object needs to be immutable.

Implementation:

```swift
protocol Dependency {
    func foo()
}

struct DependencyImplementation: Dependency {
    func foo() {
        // Does something
    }
}

class Client {
    let dependency: Dependency

    init(dependency: Dependency) {
        self.dependency = dependency
    }

    func foo() {
        dependency.foo()
    }
}

let client = Client(dependency: DependencyImplementation())
client.foo()
```

✓ Pros:

- Provides best encapsulation.

- Ensures that client object is always in a valid state.

✕ Cons:

- Dependencies cannot be changed later.

- Becomes cumbersome with more than 3 dependencies. Consider property injection in this case.

## Property Injection

Description: dependencies are passed via properties.

When to use: dependencies need to be changed later or you do not directly initialize the object. View controllers and NSManagedObject are examples of the latter.

Implementation:

```swift
protocol Dependency {
    func foo()
}

struct DependencyImplementation: Dependency {
    func foo() {
        // Does something
    }
}

class Client {
    var dependency: Dependency!

    func foo() {
        dependency.foo()
    }
}

let client = Client()
client.dependency = DependencyImplementation()
client.foo()
```

✓ Pros:

- Allows to set dependencies later.

- Provides readable way of constructing objects with many dependencies.

✕ Cons:

- Client might be left in inconsistent state if some dependencies are missing.

- Leaks encapsulation.

- Imposes the use of optional or force unwrapped properties.

## Interface Injection

Description: dependency is injected via setter method or passed as a parameter.

When to use: different types of clients need to be handled by an injector. Allows injector to apply policies over the clients.

Implementation:

```swift
protocol Dependency {}

protocol HasDependency {
    func setDependency(_ dependency: Dependency)
}

protocol DoesSomething {
    func doSomething()
}

class Client: HasDependency, DoesSomething {
    private var dependency: Dependency!

    func setDependency(_ dependency: Dependency) {
        self.dependency = dependency
    }

    func doSomething() {
        // Does something with a dependency
    }
}

class Injector {
    typealias Client = HasDependency & DoesSomething
    private var clients: [Client] = []

    func inject(_ client: Client) {
        clients.append(client)
        client.setDependency(SomeDependency())
        // Dependency applies its policies over clients
        client.doSomething()
    }

    // Switch dependencies under certain conditions
    func switchToAnotherDependency() {
        clients.forEach { $0.setDependency(AnotherDependency()) }
    }
}

class SomeDependency: Dependency {}
class AnotherDependency: Dependency {}
```

In the above example `Injector` handles any client, conforming to `HasDependency` and `DoesSomething` protocols. For this pattern to be useful, injector needs to apply certain policies over its clients. In our case, it calls `doSomething()` and is capable of switching dependencies.

✓ Pros:

- Allows to set dependencies later.

- Allows injector to apply its policies over a client.

- Injector can handle any client, which implements the required protocol.

✕ Cons:

- Client becomes a dependency itself, which complicates the flow of control.

## Ambient Context

Description: single globally accessible dependency, exposed via protocol. This allows to substitute implementation if needed, e.g. in tests.

When to use: system-wide dependency, used by dozens of clients. Instead of injecting it to so many clients, we create single and globally accessible dependency.

Implementation:

```swift
protocol DateTimeProvider {
    var now: Date { get }
}

struct SystemDateTimeProvider: DateTimeProvider {
    var now: Date {
        return Date()
    }
}

class DateTime {
    static var provider: DateTimeProvider = SystemDateTimeProvider()

    static var now: Date {
        return provider.now
    }
}
```

This way we can change `DateTimeProvider` to use server time or control the time in test environment.

✓ Pros:

- Dependency is globally available, which reduces the complexity of individual clients.

- Transparent.

✕ Cons:

- Implicit: to understand that ambient context is used by a client, we must manually inspect client’s code.

- Leaks encapsulation.

- Requires thread safety.

# Dependency Injection Patterns

Dependency injection is a broad technique and can be implemented differently. The primary patterns are:

- Factory.

- Service Locator.

- Dependency Injection Container.

Let’s review each of them.

## Factory

I’ll use the word “factory” to mean both **abstract factory** and **factory method** patterns. Conceptually, both these patterns provide a way to encapsulating the instantiation and construction logic, hence can be generalized.

Factories usually act as injectors and wire together clients with their dependencies. The goal of factories is to decouple dependencies from their clients.

```swift
protocol Client {}

enum ClientFactory {

    static func make() -> Client {
        return ClientImplementation(dependency: DependencyImplementation())
    }
}

class ClientImplementation: Client {
    init(dependency: Dependency) {}
}

protocol Dependency { }
struct DependencyImplementation: Dependency {}
```

## Dependency Injection Container

A container takes over some sort of abstractions within its bounds. It serves a wide range of functions:

- Wires dependencies with clients.

- Instantiates objects.

- Manages life cycle of created objects.

- Applies container-specific services to objects.

The core difference from factory is that dependency injection container typically holds a link to created objects, hence the name “container”. The container is especially useful when you need to manage lots of client objects with many dependencies. Factories usually just “forget” about the instantiated objects.

Let’s see how a container can be used to assemble a VIPER module:

```swift
final class Assembly {
    private let view = View()
    private let presenter = Presenter()
    private let interactor = Interactor()
    private let router = Router()

    var input: ModuleInput {
        return presenter
    }

    weak var output: ModuleOutput? {
        didSet {
            presenter.output = output
        }
    }

    init() {
        view.output = presenter
        interactor.output = presenter
        router.output = presenter

        presenter.view = view
        presenter.interactor = interactor
        presenter.router = router
    }
}

class View {
    weak var output: ViewOutput!
}

class Presenter {
    weak var view: ViewInput!
    weak var interactor: InteractorInput!
    weak var router: RouterInput!
    weak var output: ModuleOutput!
}

class Interactor {
    weak var output: InteractorOutput!
}

class Router {
    weak var output: RouterOutput!
}

// Declaration and conformance to input / output protocols is omitted for brevity
```

**Assembly** is a dependency injection container which instantiates, wires together and manages life cycle of VIPER module components. The container exposes module input and output ports, enforcing encapsulation.

## Service Locator

Service Locator is controversial pattern. The idea behind is that instead of instantiating dependencies directly, we must use special locator object, responsible for looking up each dependency (i.e. service, hence the name of the pattern). Locator provides a way to register dependencies and manages their life cycles. It does not instantiate the dependencies.

Service Locator has two common implementations:

- Globally-accessed locator, usually as a singleton or a type with static methods.

- Injected as a dependency.

The former approach violates dependency injection, since DI is an alternative to static and global access. I suggest to follow the second strategy, which is implemented next:

```swift
protocol Locator {
    func resolve<T>() -> T?
}

final class LocatorImpl: Locator {
    private var services: [ObjectIdentifier: Any] = [:]

    func register<T>(_ service: T) {
        services[key(for: T.self)] = service
    }

    func resolve<T>() -> T? {
        return services[key(for: T.self)] as? T
    }

    private func key<T>(for type: T.Type) -> ObjectIdentifier {
        return ObjectIdentifier(T.self)
    }
}

class Client {
    private let locator: Locator

    init(locator: Locator) {
        self.locator = locator
    }

    func doSomething() {
        guard let service: Service = locator.resolve() else { return }
        // do something with service
    }
}

class Service {}
```

# Conclusion

Dependency injection is a powerful technique, which helps to design clean and maintainable applications. It allows to separate the creation of objects from their usage and reduces coupling between components.

**Factory**, **Service Locator** and **Dependency Injection Container** patterns describe different solutions to how dependency can be injected into a client. Our implementations outline their appliance in Swift.

Original post: [https://www.vadimbulavin.com/dependency-injection-in-swift/](https://www.vadimbulavin.com/dependency-injection-in-swift/)
