---
layout: post
title: "Unit Testing: The Right Way"
author: "Linh Vo"
tags: "Unit-Test"
---

**What is Mock: Mock** is used to describe dependencies that are not real in unit tests in order to isolate component and make sure it behaves as expected in all possible cases

In his book **“xUnit test patterns”, Gerard Meszaros** referred to those components as **Test doubles. A test double** can be any component we eventually use to satisfy an object dependencies.

Actually Mock is a part of **test-doubles.** While **test-doubles** are describes the whole family.

Components of test-doubles :-

1. Dummy
2. Stub
3. Spy
4. Mock
5. Fake

**Important terminology :**

**SUT :** Stands for “System under test”, which refers to the component that the unit tests are about, For instance : the class we are testing .

# Dummy

A dummy is a dependency that simply does not matter for the unit test. Usually it’s an object that will have no influence on the test results. It’s actually only used to fill the dependencies. Sometimes passing a null is way enough.

**We use a dummy to make the compiler stop crying about missing dependencies.**

A dummy also allows to clearly see objects and values that are (or not) used during a unit test.

Usually creating a dummy does not require a Framework or tool. since it’s only an empty implementation.

For example you have a dependency that you use to log events. This is something totally irrelevant to the test output. then we can use a dummy logger.

```swift
// Logger interface definition
protocol Logger {
    func log(message: String)
}

// Logger real implementation
class PrintLogger: Logger {

    func log(message: String) {
        print(message)
    }
}

// Dummy logger will only sit there to fill the dependency
class DummyLogger: Logger {

    // Will do nothing.
    func log(message: String) {
        // Nothing ...
    }
}
```

As a general rule, a dummy should do nothing. and return as close to nothing as possible.

# Stub

This is probably the most common case of test doubles. A Stub provides predefined answers to indirect inputs the SUT will need during unit tests.

It can be created with a tool that will generate it based on the class type. Or it can be manually written as a set of hardcoded values that matches your test case.

We know we need a Stub when we need to control the indirect inputs of our SUT, While we don’t want to have any verification logic on it.

Here for example we need to create a test double for a user account manager dependency. which will return us some hardcoded values we will define depending on the test case.

```swift
// The account manager dependency interface.
protocol AccountManager {
    var isUserAuthenticated: Bool { get }
}

class AccountManagerStub: AccountManager {

    // We implement the dependency property.
    var isUserAuthenticated = true
}
```

# Spy

A spy is a special type of test doubles, that remembers how and when it was called. It literally spies all the informations we can’t directly get from the SUT, which allows to capture indirect outputs.

Imp : A Spy can also return some hardcoded values like Stubs, but with a the key difference. It does not perform any verification, it only gets indirect outputs out of the SUT so that we can verify them later.

Usually creating a spy does not require a framework or tool.

```swift
protocol EMailService {
    func send(email: String)
}

class OrderController {

    let emailService: EMailService

    init(emailService: EMailService) {
        self.emailService = emailService
    }

    // Order product method will call a send method wich
    // returns nothing and, hence it will be dificult to test that sent was called.
    func order(product: String) {
        emailService.send(email: "You orderd some \(product)")
    }
}

class EMailServiceSpy: EMailService {

    // In addition to the interface implementation.
    // The spy has some extra properties that
    // will remember how the dependency was called.
    var sendEmailCallCount: Int = 0

    func send(email: String) {
        sendEmailCallCount += sendEmailCallCount
    }
}
```

Here for instance we have a sendEmail method which returns nothing, which is called inside OrderController.

How to make sure that sendEmail was called when testing OrderController? Knowing that sendEmail makes not output.

The answer is to create an EmailServiceSpy. which will record how many times sendEmail was called. so that we can use it as a test double when Testing OrderController.

# Mock

Like a Spy, a Mock allows also to remember things about how it was called. but it has an extra feature which gives it it definition.

A mock knows what is the right behaviour. a mock knows what should take place.

In other words, a spy only gets out the values we need for our tests. A mock also verifies that those values matches what is expected.

Like stubs mocks might be created manually or generated using a tool.

Using the same example above here we used a mock with a verification method, that returns a boolean indicating whether send email was called the right number of time.

```swift
protocol EMailService {
    func send(email: String)
}

class OrderController {

    let emailService: EMailService

    init(emailService: EMailService) {
        self.emailService = emailService
    }

    // Order product method will call a send method wich
    // returns nothing and, hence it will be dificult to test that sent was called.
    func order(product: String) {
        emailService.send(email: "You orderd some \(product)")
    }
}

// A mock knows what is the right behavior that should take place.
class EMailServiceMock: EMailService {

    var sendEmailCallCount: Int = 0

    // It can do a part of the verification that used to be done only inside the test.
    func verifySendEmailWasCalled() -> Bool {
        return sendEmailCallCount > 0
    }

    func send(email: String) {
        sendEmailCallCount += sendEmailCallCount
    }
}
```

**Question :** Mock or Spy ?

It’s important to mention, as a general rule, a **Mock should be the option to go for rather than Spies**. we only use a spy when it’s not possible or complicated to use a mock.

Or for instance your code core needs some refactoring to be cleanly testable. then you can use spies.

Or let’s say you have a function callback as input for your SUT. and it’s not possible to mock it with the framework you are using.

# Fake

A fake is a simplified implementation of a more complex dependency.

We usually use it when the SUT depends on another component which holds states.

A good candidate for a fake is a component that cannot be replaced with only static values. Hence a **Fake should have some logic on it.**

Let’s say you have a database dependency. obviously we will not be using a real database for our unit tests. then we can create a fake database that will store data in an array to simulated a database under unit tests

```swift
class FakeUserDefaults: UserDefaults {

    // A dictionary we will use to store during uni tests
    private var fakeStorage: [String: Any] = [:]

    override func object(forKey defaultName: String) -> Any? {
            return fakeStorage[defaultName]
    }

    // Override the real set method to fake user default behavior
    override func set(_ value: Any?, forKey defaultName: String) {
            fakeStorage[defaultName] = value
    }
}
```

Note that a using a “OnMemory” database for unit tests is a kind of fakes also.

In the above example we did a FakeUserDefaults which is a subclass of the real UserDefaults , it overrides the **set** and **objectForKey** methods by using a dictionary to store and fetch values.

It’s important to note that a Fake logic can grow along with the real object. which may become complicated to maintain at a certain level. This is why it’s advised to use TDD while making fakes to keep them under control.

Thanks for Reading !!

Original Post [https://shobhitgupta-27686.medium.com/unit-testing-the-right-way-49574112040](https://shobhitgupta-27686.medium.com/unit-testing-the-right-way-49574112040)
