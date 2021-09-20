---
layout: post
title: "Unit Testing: Best Practices"
author: "Linh Vo"
tags: "Unit-Test"
---

Testing is about getting feedback on software. This is what gives us confidence in the program that we write. This article is an introduction to Swift unit testing. Let’s learn why to test, what to test, how to test and implement several real-world Swift unit tests.

# What is Unit Test

A unit test is a way of testing a unit - the smallest piece of code that can be logically isolated in a system. In most programming languages, that is a function, a subroutine, a method or property.

# Why to Write Unit Tests

Your first thought might be to skip unit testing, since customer is not directly paying for them. However, Economics of Test Automation demonstrates that the effort spent on tests pays off a lot. The extra cost of developing and maintaining test suite is offset by savings on manual testing, debugging, regression testing, improved code structure.

> We are writing unit tests, because we want our code to work and to keep it working.

# Defining Good Unit Test

A good unit tests possesses following qualities:

- Readable

- Maintainable

- Trustworthy

## Readable

Unit tests are intended to be read very often. When something breaks in your production code, you typically examine failing tests, which is going to happen quite frequently. When unit test fails, you should be able to understand the exact failure reason just by looking at the code. If you can’t understand what is going on, the test is likely to be rewritten or even removed.

Same as the production code, your tests should follow a single coding convention. It includes formatting, naming, preferred patterns and much more. I’ve touched on the subject in Missing Guide on Swift Code Style, where you’ll also find the list of most popular Swift code styles.

Here are the factors which contribute to tests readability:

- Naming unit tests and variables

- Clear assertion messages

- Following Arrange-Act-Assert structure. More on this few paragraphs below.

## Maintainable

When the new features are added to our production code, it should not result in a cascade of changes in the tests. Ideally, we should modify our tests only when the non-functional requirements change. Otherwise the tests can ruin project deliveries and are candidates to be disabled when schedule becomes aggressive.

Here is what makes unit tests maintainable:

- Test only publics and internals. Do not weaken encapsulation just to verify private and fileprivate properties or methods.

- Do not put code into production which is there only to support testing.

- Verify one thing per test.

- Reuse code. If tests contain lots of duplication, it’s time to introduce new abstractions.

- Create and share helper methods for initializations and verifications.

## Trustworthy

A test is trustworthy when it makes clear what’s going on and that you can do something about it. When it passes, you trust such test and that the code works under this scenario. And when the test fails, you don’t tell yourself that this does not mean that the code is not working.

Here is what makes unit tests trustworthy:

- 100% pass rate. Having at least one failing test results in a broken window effect.

- Tests must be reproducible. Any flakiness should not be tolerated. All errors, crashes, freezes and failures that seem to happen ‘randomly’ will sooner or later infect whole test suite. I advert to the topic in Unit Testing Asynchronous Code in Swift.

- Avoid test logic: no `for` or `while` loops, not `if`, `else` or `switch` statements. Not even a `do-catch`.

# Arrange-Act-Assert

Each unit test performs three main actions:

1. **Arrange** objects, setting up the system under test as necessary.

2. **Act** on an object. Here we usually call a method we want to test.

3. **Assert** that the result of the action is as expected.

> Given-When-Then is another name for the same pattern. Each action from Given-When-Then directly maps to Arrange-Act-Assert.

# Control Side-Effects

When we change global app data from our test, this indirectly affects the whole test suite. To prevent this from happening we must cleanup before and after each test. In our `XCTestCase` class, override following methods:

- `setUp()` to reset initial state **before** individual test method. The method has it’s class counterpart which sets up initial state for all test methods from the test case.

- `tearDown()` to cleanup **after** each test method finishes. The class method `tearDown()` performs final cleanup after all test methods finish.

# Do Not Initialize in Setup

By default, `XCTestCase` provides a `setUp()` method as a single entry point for all tests. It’s common to initialize system under test with all dependencies there. I recommend against doing this. Here is why:

- The setup method indirectly couples tests. When tests are coupled, a change in one is likely to affect the others.

- Setup methods tend to grow over time as more tests are added. This makes tests unreadable and hard to manage.

- Tests receive dependencies which they don’t actually need. Such tests can break or raise compilation errors, because of changes in unrelated dependencies.

My suggestion is to use overloaded factory methods which return system under test with different configurations:

```swift
class UserStorageTests: XCTestCase {

    // MARK: - Helpers

    func makeSUT() -> UserStorage {
        let sut = UserStorage(storage: storageMock, secureStorage: secureStorageMock)
        return sut
    }

    func makeSUT(with user: User) -> UserStorage {
        let sut = makeSUT()
        sut.save(user)
        return sut
    }
}
```

# Initialize Test Data in Properties

It’s common to reuse test data in different test methods. So that we do not copy and paste it, we store test data in properties of `XCTestCase` classes.

Say, we are testing `UserStorage` which saves sensitive data to keychain and the rest to user defaults. We inject `UserDefaults` and `Keychain`, so that they can be mocked:

```swift
class UserStorageTests: XCTestCase {

    func testUsernameSavedToStorage() {
        let userDefaultsMock = UserDefaultsMock()
        let keychainMock = KeychainMock()
        let user = User(id: 1, username: "U1", password: "P1")

        let sut = UserStorage(storage: userDefaultsMock, secureStorage: keychainMock)

        sut.save(user)

        // Assert
    }

    func testPasswordSavedToSecureStorage() {
        let userDefaultsMock = UserDefaultsMock()
        let keychainMock = KeychainMock()
        let user = User(id: 1, username: "U1", password: "P1")

        let sut = UserStorage(storage: userDefaultsMock, secureStorage: keychainMock)

        sut.save(user)

        // Assert
    }
}
```

Although we are testing one concern per test, this introduces another problem: the methods have lots of duplicated code. Let’s clean this up by following the best practices that we’ve discussed:

1. Extract the duplicated test data and mocks into properties.

2. Create factory methods to initialize the system under test with different configurations.

```swift
class UserStorageTests: XCTestCase {
    let userDefaults = UserDefaultsMock()
    let keychain = KeychainMock()
    let user = User(id: 1, username: "U1", password: "P1")

    func testUsernameSavedToStorage() {
        makeSUT().save(user)
        XCTAssertNotNil(userDefaults.inputUsername)
    }

    func testPasswordSavedToSecureStorage() {
        makeSUT().save(user)
        XCTAssertNotNil(keychain.inputPassword)
    }

    func makeSUT() -> UserStorage {
        return UserStorage(storage: userDefaults, secureStorage: keychain)
    }
}
```

# Do Not Leak Test Code into Production

Firstly, do not weaken encapsulation for the purpose of testing. If you cannot test something, because it has private access control, do not make it public. It’s either a flaw in the app design or you might be testing too much.

Secondly, do not put logic into production code which is there only to support testing. Changes in your test code should not affect the production.

# System Under Test

> When you test something, you refer to it as the **system under test (SUT)**.

`MusicService` will be our system under test. It is a networking service which searches music via iTunes API:

```swift
struct MusicService {

    func search(_ term: String, completion: @escaping (Result<[Track], Error>) -> Void) {
        URLSession.shared.dataTask(with: .search(term: term)) { data, response, error in
            DispatchQueue.main.async {
                completion(self.parse(data: data, error: error))
            }
        }.resume()
    }

    func parse(data: Data?, error: Error?) -> Result<[Track], Error> {
        if let data = data {
            return Result { try JSONDecoder().decode(SearchMediaResponse.self, from: data).results }
        } else {
            return .failure(error ?? URLError(.badServerResponse))
        }
    }
}
```

It creates a request via:

```swift
extension URLRequest {
    static func search(term: String) -> URLRequest {
        var components = URLComponents(string: "https://itunes.apple.com/search")
        components?.queryItems = [
            .init(name: "media", value: "music"),
            .init(name: "entity", value: "song"),
            .init(name: "term", value: "\(term)")
        ]

        return URLRequest(url: components!.url!)
    }
}
```

`MusicService` uses `URLSession` to fire HTTP requests, and returns an array of `Track`s via the `completion` callback:

```swift
struct SearchMediaResponse: Codable {
    let results: [Track]
}

struct Track: Codable, Equatable {
    let trackName: String?
    let artistName: String?
}
```

When building iOS apps, it is typical to design networking layer using services similar to `MusicService`. Therefore, testing it is a common and relevant task.

# Mocking in Swift

The idea of the Mocking pattern is to use special objects called mocks.

> Mock object mimics real objects for testing.

To use mock objects, we must:

- Extract asynchronous work into a new type.

- Delegate asynchronous work to the new type.

- Replace real dependency with a mock during testing.

> Passing dependencies to an object is called dependency injection, which is important to understand when practicing mocking.

Let’s highlight the parts of `MusicService` which perform asynchronous work:

```swift
struct MusicService {
    func search(_ term: String, completion: @escaping (Result<[Track], Error>) -> Void) {
        URLSession.shared.dataTask(with: .search(term: term)) { data, response, error in
            DispatchQueue.main.async {
                ...
            }
        }.resume()
    }
    ...
}
```

The responsibility of the above code is to execute HTTP requests. We extract it into a new type called `HTTPClient`, and use it from `MusicService`:

```swift
protocol HTTPClient {
    func execute(request: URLRequest, completion: @escaping (Result<Data, Error>) -> Void)
}

struct MusicService {
    let httpClient: HTTPClient

    func search(_ term: String, completion: @escaping (Result<[Track], Error>) -> Void) {
        httpClient.execute(request: .search(term: term)) { result in
            completion(self.parse(result))
        }
    }

    private func parse(_ result: Result<Data, Error>) -> Result<[Track], Error> { ... }
}
```

`HTTPClient` will have two implementations. The production one contains all the code from `MusicService` which was responsible for firing HTTP requests:

```swift
class RealHTTPClient: HTTPClient {
    func execute(request: URLRequest, completion: @escaping (Result<Data, Error>) -> Void) {
        URLSession.shared.dataTask(with: request) { data, response, error in
            DispatchQueue.main.async {
                if let data = data {
                    completion(.success(data))
                } else {
                    completion(.failure(error!))
                }
            }
        }.resume()
    }
}
```

The test implementation remembers the last parameter passed to `execute()`, and allows us to pass an arbitrary `result` to the `completion` callback, mimicking network response.

```swift
class MockHTTPClient: HTTPClient {
    var inputRequest: URLRequest?
    var executeCalled = false
    var result: Result<Data, Error>?

    func execute(request: URLRequest, completion: @escaping (Result<Data, Error>) -> Void) {
        executeCalled = true
        inputRequest = request
        result.map(completion)
    }
}
```

Now we are ready to test `MusicService` using a mock object. In the next test we verify that `MusicService` fires a correct HTTP request:

```swift
func testSearch() {
    // 1.
    let httpClient = MockHTTPClient()
    let sut = MusicService(httpClient: httpClient)

    // 2.
    sut.search("A") { _ in }

    // 3.
    XCTAssertTrue(httpClient.executeCalled)
    // 4.
    XCTAssertEqual(httpClient.inputRequest, .search(term: "A"))
}
```

Here is what we are doing:

1. Initialize the system under test with a mock implementation of `HTTPClient`.

2. Run the method `search()`, passing it an arbitrary query.

3. Verify that the method `execute()` has been invoked on the mock.

4. Verify that the correct HTTP request has been passed to the mock.

Next, we verify how `MusicService` handles API response. In order to do this, we mimic a successful response using `MockHTTPClient`:

```swift
func testSearchWithSuccessResponse() throws {
    // 1.
    let expectedTracks = [Track(trackName: "A", artistName: "B")]
    let response = try JSONEncoder().encode(SearchMediaResponse(results: expectedTracks))

    // 2.
    let httpClient = MockHTTPClient()
    httpClient.result = .success(response)

    let sut = MusicService(httpClient: httpClient)

    var result: Result<[Track], Error>?

    // 3.
    sut.search("A") { result = $0 }

    // 4.
    XCTAssertEqual(result?.value, expectedTracks)
}
```

1. Prepare test data.

2. Initialize a mock object, and pass it predefined response data. The mock will return that data in the `execute()` callback.

3. The method `search()` is synchronous since we mocked `HTTPClient`. Therefore, the completion callback is invoked instantly.

4. Verify that the correct result has been received.

We can also verify how `MusicService` handles failed response:

```swift
func testSearchWithFailureResponse() throws {
    // 1.
    let httpClient = MockHTTPClient()
    httpClient.result = .failure(DummyError())

    let sut = MusicService(httpClient: httpClient)

    var result: Result<[Track], Error>?

    // 2.
    sut.search("A") { result = $0 }

    // 4.
    XCTAssertTrue(result?.error is DummyError)
}

struct DummyError: Error {}
```

1. Provide an error response to the mock.

2. Same as before, the method `search()` is synchronous, and the `completion` callback is invoked immediately.

3. Verify that `search()` passed the correct error to the callback.

# Expectations

The Expectations pattern is based on the usage of `XCTestExpectation`.

> [Expectation](https://developer.apple.com/documentation/xctest/xctestexpectation) is an expected outcome in an asynchronous test.

The pattern can be summarized into four steps:

1. Create an instance of `XCTestExpectation`.

2. Fulfill the expectation when async operation has finished.

3. Wait for the expectation to be fulfilled.

4. Assert the expected result.

We are going to test the version of `MusicService` with `HTTPClient`. Let’s recall the implementation:

```swift
protocol HTTPClient {
    func execute(request: URLRequest, completion: @escaping (Result<Data, Error>) -> Void)
}

struct MusicService {
    let httpClient: HTTPClient

    func search(_ term: String, completion: @escaping (Result<[Track], Error>) -> Void) {
        httpClient.execute(request: .search(term: term)) { result in
            completion(self.parse(result))
        }
    }

    private func parse(_ result: Result<Data, Error>) -> Result<[Track], Error> { ... }
}
```

The primary difference is that `HTTPClient` won’t be mocked. As you already might have guessed, we are going to write an integration test:

```swift
func testSearch() {
    // 1.
    let didReceiveResponse = expectation(description: #function)

    // 2.
    let sut = MusicService(httpClient: RealHTTPClient())

    // 3.
    var result: Result<[Track], Error>?

    // 4.
    sut.search("ANYTHING") {
        result = $0
        didReceiveResponse.fulfill()
    }

    // 5.
    wait(for: [didReceiveResponse], timeout: 5)

    // 6.
    XCTAssertNotNil(result?.value)
}
```

1. Create an instance of `XCTestExpectation`.

2. Initialize SUT with `RealHTTPClient`. That is, the one used in production.

3. Declare a variable which will hold a result of the `search()` method once it completes.

4. In the `search()` callback, fulfill the expectation, and store the received result into the variable.

5. Wait for up to 5 seconds for `search()` to fulfill.

6. Assert that a successful request has been received.

## More Use Cases for Expectations

`XCTestExpectation` is a versatile tool and can be applied in a number of scenarios.

**Inverted Expectations** allows us to verify that something did not happen:

```swift
func testInvertedExpectation() {
    // 1.
    let exp = expectation(description: #function)
    exp.isInverted = true

    // 2.
    sut.maybeComplete {
        exp.fulfill()
    }

    // 3.
    wait(for: [exp], timeout: 0.1)
}
```

1. Create an inverted expectation. The test will fail if the inverted expectation is fulfilled.

2. Call a method that may conditionally invoke a callback.

3. Verify that the callback is not invoked.

**Notification Expectation** is fulfilled when an expected Notification is received:

```swift
func testExpectationForNotification() {
    let exp = XCTNSNotificationExpectation(name: .init("MyNotification"), object: nil)

    ...
    sut.postNotification()

    wait(for: [exp], timeout: 1)
}
```

**Assert for over fulfillment** triggers an assertion if the number of calls to fulfill() exceeds expectedFulfillmentCount:

```swift
func testExpectationFulfillmentCount() {
    let exp = expectation(description: #function)
    exp.expectedFulfillmentCount = 3
    exp.assertForOverFulfill = true

    ...
    sut.doSomethingThreeTimes()

    wait(for: [exp], timeout: 1)
}
```

# References

- [https://www.vadimbulavin.com/unit-testing-best-practices-on-ios-with-swift](https://www.vadimbulavin.com/unit-testing-best-practices-on-ios-with-swift)
- [https://www.vadimbulavin.com/real-world-unit-testing-in-swift](https://www.vadimbulavin.com/real-world-unit-testing-in-swift)
- [https://www.vadimbulavin.com/unit-testing-async-code-in-swift](https://www.vadimbulavin.com/unit-testing-async-code-in-swift)
