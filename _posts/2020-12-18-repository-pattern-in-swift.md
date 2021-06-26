---
layout: post
title: "Repository Pattern in Swift"
author: "Linh Vo"
tags: "Clean-Architecture"
---

# Background

All apps developed require data of some description. This data is stored somewhere, could be on the device itself, in a remote database/service or a combination. Let’s take a look at the most common sources of data:

- [JSON](https://www.w3schools.com/whatis/whatis_json.asp) web services
- [UserDefaults](https://developer.apple.com/documentation/foundation/userdefaults)
- [Core Data](https://developer.apple.com/documentation/coredata)
- [Realm](https://docs.mongodb.com/realm-legacy/docs/swift/latest/)
- Other 3rd party system

Each of these methods saves data in a different format. Now I’m sure you will have used at least one of these methods in your apps at some point to retrieve / save data.

When not using the repository pattern it is quite common to access and use these elements directly, either in your ViewController or in some other part of your app depending how it is structured.

# The problem

What’s the problem with this approach? Your app becomes difficult to maintain. Now if you only have a small app with a few screens then this isn’t much of a problem as there are only a few elements to change.

However, what if you are working on a large app with several developers and lots of code? You could have [NSManagedObjects](https://developer.apple.com/documentation/coredata/nsmanagedobject) or [Codable](https://developer.apple.com/documentation/swift/codable) objects littered throughout the codebase for example. What happens if you wish to remove Core Data? Perhaps move to realm? You would need to modify all of the classes in your codebase where you had used your Core Data objects.

Similarly, if you are using Codable objects directly from your JSON response. What happens when your backend team changes the API or you switch to a different API provider? The structure of the data may change which means your Codable objects might change. Again you will need to modify a large number of classes if you are working on a large app.

We can also apply this to the other options such as accessing data from 3rd party frameworks. If we use the objects returned from the framework directly, they will all need changing if we change provider or the SDK changes.

There is also the question of query language. Web services use headers and [URLQueryItem](https://developer.apple.com/documentation/foundation/urlqueryitem), Core Data uses [Predicates](https://developer.apple.com/documentation/foundation/nspredicate) and so on. Every entry point to query the data must know and understand the underlying query language in order to get the information it once. Again, if this changes we need change every query point to the new format.

Let’s have a look at the diagram below:

{% include image.html
img="assets/2020-12-18/image.png" %}

Here we have an app structure that is making use of Core Data. There is an object that is being used to access the stack that returns some data. Let’s say for this example that it is news articles. These new articles must inherit from NSManagedObject to be used in Core Data. Now if our data layer is returning NSManagedObjects to the rest of our app structure we now have a dependency between Core Data and the rest of the files in our app. If we wish to move to Realm for example, or switch to using some other form of data store we would need to modify all the of files in the app. The app in this example is only small, imagine having to do that for a much bigger app!

# Domain Objects and the Repository

This is where Domain Objects come in. Domain Objects are value objects that are defined by your application. Rather than using objects and structures defined outside of the app, we define what we want the objects to look like. It’s then up to the repository to map between the data storage object / structure to these value objects.

When we do this, it means any changes to the data access layer, as we discussed earlier such as data structure changes or changes in provider don’t impact the rest of the app. The only part of the app that needs to be updated is the repository and it’s mapping to the domain objects.

The below quote summarises the idea of the pattern:

> Repositories are classes or components that encapsulate the logic required to access data sources. They centralize common data access functionality, providing better maintainability and decoupling the infrastructure or technology used to access databases from the domain model layer.

Let’s have a look at our previous example but modified to use the a repository and domain objects:

{% include image.html
img="assets/2020-12-18/image1.png" %}

So what is the difference here? As you can see the Core Data stack is still returning NSManagedObjects, however the repository is converting that to a domain object. This object doesn’t inherit from NSManagedObject, also it’s structure and attributes are defined by the app rather than what is in the data store.

Now if we wanted to move away from Core Data to something else the only classes that need to be changed are the Core Data stack and the repository. The rest of the app does not need to be changed as we can map the new data stores type to our domain objects using the repository.

# Example

To show a small working example we are going to use a couple of [Free Public APIs](https://github.com/public-apis/public-apis) (highly recommend this resource if you are looking to build a demo app or experiment). We will use 2 APIs that returns users. However they return them in a different format.

[https://jsonplaceholder.typicode.com/users/1](https://jsonplaceholder.typicode.com/users/1)

[https://randomuser.me/api/](https://randomuser.me/api/)

As we have done in [previous blog posts](https://pyartez.github.io/networking/simple-json-decoder-in-swift-and-combine.html) we are going to use [QuickType](https://app.quicktype.io/) to generate our Codable objects from our JSON response. We will start with our first request.

```swift
// MARK: - User
struct User: Codable {
    let id: Int
    let name, username, email: String
    let address: Address
    let phone, website: String
    let company: Company
}

// MARK: - Address
struct Address: Codable {
    let street, suite, city, zipcode: String
    let geo: Geo
}

// MARK: - Geo
struct Geo: Codable {
    let lat, lng: String
}

// MARK: - Company
struct Company: Codable {
    let name, catchPhrase, bs: String
}
```

This structure will allow us to decode the response from the first request. Let’s make a simple example that takes the response and outputs some data. We will be using code from our [Simple JSON Decoder](https://pyartez.github.io/networking/simple-json-decoder-in-swift-and-combine.html) to process the output so feel free to read up if the code you see doesn’t make sense.

```swift
let url = URL(string: "https://jsonplaceholder.typicode.com/users/1")!
// 1
let task = URLSession.shared.dataTask(with: url, completionHandler: { (user: User?, response, error) in
	// 2
    if let error = error {
        print(error.localizedDescription)
        return
    }

    // 3
    if let user = user {
        print(user.name)
        print(user.address.street)
        print(user.address.city)
        print(user.address.zipcode)
        print(user.address.geo.lat)
        print(user.address.geo.lng)
    }
})
task.resume()
```

So let’s step through what’s happening here:

1. First of all we are making the request using our [Simple JSON Decoder](https://pyartez.github.io/networking/simple-json-decoder-in-swift-and-combine.html) to return our new User type.
2. Output any errors
3. So here we are outputting the name, address and location of the user we get back. Super simple right now.

# Managing change

Now let’s say we change provider. Maybe our backend team changes the API, or we switch data provider or from 2 different data provider SDKs. In our example we will switch from the first url [https://jsonplaceholder.typicode.com/users/1](https://jsonplaceholder.typicode.com/users/1) to the second [https://randomuser.me/api/](https://jsonplaceholder.typicode.com/users/1).

The first thing we will need to do is change all of our codable objects as the structure of the response is different. Let’s use QuickType again to give us the new structure:

```swift
// MARK: - Users
struct Users: Codable {
    let results: [Result]
    let info: Info
}

// MARK: - Info
struct Info: Codable {
    let seed: String
    let results, page: Int
    let version: String
}

// MARK: - Result
struct Result: Codable {
    let gender: String
    let name: Name
    let location: Location
    let email: String
    let login: Login
    let dob, registered: Dob
    let phone, cell: String
    let id: ID
    let picture: Picture
    let nat: String
}

// MARK: - Dob
struct Dob: Codable {
    let date: String
    let age: Int
}

// MARK: - ID
struct ID: Codable {
    let name: String
    let value: String?
}

// MARK: - Location
struct Location: Codable {
    let street: Street
    let city, state, country: String
    let postcode: Int
    let coordinates: Coordinates
    let timezone: Timezone
}

// MARK: - Coordinates
struct Coordinates: Codable {
    let latitude, longitude: String
}

// MARK: - Street
struct Street: Codable {
    let number: Int
    let name: String
}

// MARK: - Timezone
struct Timezone: Codable {
    let offset, timezoneDescription: String

    enum CodingKeys: String, CodingKey {
        case offset
        case timezoneDescription = "description"
    }
}

// MARK: - Login
struct Login: Codable {
    let uuid, username, password, salt: String
    let md5, sha1, sha256: String
}

// MARK: - Name
struct Name: Codable {
    let title, first, last: String
}

// MARK: - Picture
struct Picture: Codable {
    let large, medium, thumbnail: String
}
```

Now this is more complicated that it needs to be for our example but I’m leaving it here as an extreme example of how different things can be. As you can probably tell the structure and types have change dramatically from our first example. So let’s try and output the same data from this example in our previous example. We can ignore the request part and just focus on the data output so we can see the differences:

```swift
// Request 1 output
if let user = user {
    print(user.name)
    print(user.address.street)
    print(user.address.city)
    print(user.address.zipcode)
    print(user.address.geo.lat)
    print(user.address.geo.lng)
}


// Request 2 output
if let user = users?.results.first {
    print("\(user.name.first) \(user.name.last)")
    print(user.location.street.name)
    print(user.location.city)
    print(user.location.postcode)
    print(user.location.coordinates.latitude)
    print(user.location.coordinates.longitude)
}
```

As you can see from even this simple example. We would have to change 7 lines of code, just to produce the same output. Now imagine this change happening on a much bigger project! There could possibly be 100s of lines of code that would need updating, all because the API response has changed.

# Repository Pattern

Here is where the repository pattern comes in. We can create a user repository that fetches the user and converts it to our domain object. That way we don’t need to update the output.

First thing to do is design our domain object that will represent a User in our system. Now all we are doing in this simple example is outputting a few attributes so let’s design our object with just those attributes as we don’t need the rest.

```swift
struct DomainUser {
    let name: String
    let street: String
    let city: String
    let postcode: String
    let latitude: String
    let longitude: String
}
```

Here we have a nice simple representation of our User object. There is no need to consider any of the other possible attributes returned from the API. We aren’t using them in our application and they will just sit around taking up valuable memory. You will also notice that this object doesn’t conform to Codable or subclass NSManagedObject. This is because DomainObject should not contain any knowledge about how they are stored. That is the responsibility of the repository.

To design our repository we can make use of Generics and Protocols to design a repository we can use for anything, not just our DomainUser. Let take a look:

```swift
protocol Repository {
    associatedtype T

    func get(id: Int, completionHandler: (T?, Error?) -> Void)
    func list(completionHandler: ([T]?, Error?) -> Void)
    func add(_ item: T, completionHandler: (Error?) -> Void)
    func delete(_ item: T, completionHandler: (Error?) -> Void)
    func edit(_ item: T, completionHandler: (Error?) -> Void)
}

protocol CombineRepository {
    associatedtype T

    func get(id: Int) -> AnyPublisher<T, Error>
    func list() -> AnyPublisher<[T], Error>
    func add(_ item: T) -> AnyPublisher<Void, Error>
    func delete(_ item: T) -> AnyPublisher<Void, Error>
    func edit(_ item: T) -> AnyPublisher<Void, Error>
}
```

Here we have different functions for all of the operations we can do. What you will notice is that none of these functions specify where or how the data is stored. Remember when we talked about different storage options at the beginning? We could implement a repo that talks to an API (like in our example), one that stores things in Core Data or one that writes to UserDefaults. It’s up to the repository that implements the protocol to decide these details, all we care about is that we can load and save the data from somewhere.

# See it action

Now we have defined what the repository pattern is, let’s create 2 implementations. One for our first request and one for the second. Both should return domain objects, rather than the type returned from the request.

```swift
// 1
enum RepositoryError: Error {
    case notFound
}

struct FirstRequestImp: Repository {
    typealias T = DomainUser

    // 2
    func get(id: Int, completionHandler: @escaping (DomainUser?, Error?) -> Void) {
        let url = URL(string: "https://jsonplaceholder.typicode.com/users/1")!
        let task = URLSession.shared.dataTask(with: url, completionHandler: { (user: User?, response, error) in
            if let error = error {
                completionHandler(nil, error)
                return
            }

            guard let user = user else {
                completionHandler(nil, RepositoryError.notFound)
                return
            }

            // 3
            let domainUser = DomainUser(
                name: user.name,
                street: user.address.street,
                city: user.address.city,
                postcode: user.address.zipcode,
                latitude: user.address.geo.lat,
                longitude: user.address.geo.lng
            )

            completionHandler(domainUser, nil)
        })
        task.resume()
    }

     // 4
    func list(completionHandler: @escaping ([DomainUser]?, Error?) -> Void) {}
    func add(_ item: DomainUser, completionHandler: @escaping (Error?) -> Void) {}
    func delete(_ item: DomainUser, completionHandler: @escaping (Error?) -> Void) {}
    func edit(_ item: DomainUser, completionHandler: @escaping (Error?) -> Void) {}
}

struct SecondRequestImp: Repository {
    typealias T = DomainUser

    func get(id: Int, completionHandler: @escaping (DomainUser?, Error?) -> Void) {
        let url = URL(string: "https://randomuser.me/api/")!
        let task = URLSession.shared.dataTask(with: url, completionHandler: { (users: Users?, response, error) in
            if let error = error {
                completionHandler(nil, error)
                return
            }

            guard let user = users?.results.first else {
                completionHandler(nil, RepositoryError.notFound)
                return
            }

            // 5
            let domainUser = DomainUser(
                name: "\(user.name.first) \(user.name.last)",
                street: user.location.street.name,
                city: user.location.city,
                postcode: "\(user.location.postcode)",
                latitude: user.location.coordinates.latitude,
                longitude: user.location.coordinates.longitude
            )

            completionHandler(domainUser, nil)
        })
        task.resume()
    }

    func list(completionHandler: @escaping ([DomainUser]?, Error?) -> Void) {}
    func add(_ item: DomainUser, completionHandler: @escaping (Error?) -> Void) {}
    func delete(_ item: DomainUser, completionHandler: @escaping (Error?) -> Void) {}
    func edit(_ item: DomainUser, completionHandler: @escaping (Error?) -> Void) {}
}
```

There’s quite a bit of code here so let’s step through it.

1. First of all we have defined a new error to send back if we don’t receive any user info from the API.
2. This is the same call we made in our example before.
3. Now here we are taking the returned Codable User and converting it to your new DomainUser class.
4. We aren’t implementing the other functions in this example so just leaving them empty for now to remove errors.
5. This struct is the second request we are making, and again here we are mapping our Users Codable type from the second request to our DomainUser.

Now that we have made our two repositories, let’s show how we can quickly switch between them without breaking / changing code.

```swift
let repository: FirstRequestImp = FirstRequestImp()
repository.get(id: 1) { (user, error) in
    if let error = error {
        print(error)
    }

    if let user = user {
        print(user.name)
        print(user.street)
        print(user.city)
        print(user.postcode)
        print(user.latitude)
        print(user.longitude)
    }
}
```

Here is our example from earlier in the article but updated to use our new repositories. Here we go and fetch the user and print their details, the same as before. Now below we can switch to our second request and see how that will work.

```swift
let repository: SecondRequestImp = SecondRequestImp()
repository.get(id: 1) { (user, error) in
    if let error = error {
        print(error)
    }

    if let user = user {
        print(user.name)
        print(user.street)
        print(user.city)
        print(user.postcode)
        print(user.latitude)
        print(user.longitude)
    }
}
```

Now notice how the only part we changed was the implementation class? The rest of the code remained the same even though where the data was coming from has changed and is coming back in a completely different structure. Now imagine we are using this repo in many places to fetch user details. We can quickly switch between different data sources without changing the code that uses it. The only changes we have to make are to the repo and to the data mapping code. So only one change rather than a change in every single class that uses these objects.

# Conclusion

So let’s recap what we have discussed here:

- First of all we discussed the problem of using data storage classes throughout your codebase. Especially on large projects if you need to switch data source / structure.
- We then discussed how using the repository pattern and mapping to domain objects rather than using data storage classes can make your code easier to change in the future.
- We worked through some examples of how changing API structures can impact your code.
- We then implemented a basic repository pattern with mapping to domain objects to show how doing this can make updating your project easier.
- Finally let’s discuss the pros and cons of the approach:

### Advantages

- Code is easier to change if you need to switch data source or structure
- Separates concerns of where / how data is stored away from the rest of your app

### Disadvantages

- Adds more code and complexity
- Need to write mappers for each object to domain objects
- Not really needed on smaller solo projects

Feel free to [download the playground](https://github.com/pyartez/blog-samples) and play around with the examples yourself

Original post: [https://pyartez.github.io/architecture/repository-pattern-in-swift-and-combine.html](https://pyartez.github.io/architecture/repository-pattern-in-swift-and-combine.html)
