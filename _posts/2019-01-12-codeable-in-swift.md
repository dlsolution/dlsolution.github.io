---
layout: post
title: "Codeable in Swift"
author: "Linh Vo"
tags: "UIKit"
---

In this article I want to provide quick code samples to help you solve answer common questions and solve common problems, all using `Codable`.

# Encoding and decoding JSON

Let’s start with the basics: converting some JSON into Swift structs.

First, here’s some JSON to work with:

```swift
let json = """
[
    {
        "name": "Paul",
        "age": 38
    },
    {
        "name": "Andrew",
        "age": 40
    }
]
"""

let data = Data(json.utf8)
```

The last line converts it to a `Data` object because that’s what `Codable` decoders work with.

Next we need to define a Swift struct that will hold our finished data:

```swift
struct User: Codable {
    var name: String
    var age: Int
}
```

Now we can go ahead and perform the decode:

```swift
let decoder = JSONDecoder()

do {
    let decoded = try decoder.decode([User].self, from: data)
    print(decoded[0].name)
} catch {
    print("Failed to decode JSON")
}
```

That will print “Paul”, which is the name of the first user in the JSON.

# Converting case

A common problem with JSON is that it will use different formatting for its key names than we want to use in Swift. For example, you might get “first_name” in your JSON and need to convert that to a `firstName` property.

Now, one obvious solution here is just to change either the JSON or your Swift types so they use the same naming convention, but we’re not going to do that here. Instead, I’m going to assume you have code like this:

```swift
let json = """
[
    {
        "first_name": "Paul",
        "last_name": "Hudson"
    },
    {
        "first_name": "Andrew",
        "last_name": "Carnegie"
    }
]
"""

let data = Data(json.utf8)

struct User: Codable {
    var firstName: String
    var lastName: String
}
```

To make this work we need to change only one property in our JSON decoder:

```swift
let decoder = JSONDecoder()
decoder.keyDecodingStrategy = .convertFromSnakeCase
```

That instructs Swift to map snake case names (names_written_like_this) to camel case names (namesWrittenLikeThis).

# Mapping different key names

If you have JSON keys that are completely different from your Swift properties, you can map them using a `CodingKeys` enum.

Take a look at this JSON:

```swift
let json = """
[
    {
        "user_first_name": "Taylor",
        "user_last_name": "Swift"
        "user_age": 26
    }
]
"""
```

Those key names aren’t great, and really we’d like to convert that data into a struct like this:

```swift
struct User: Codable {
    var firstName: String
    var lastName: String
    var age: Int
}
```

To make that happen we need to declare a `CodingKeys` enum: a mapping that `Codable` can use to convert JSON names into properties for our struct. This is a regular enum that uses strings for its raw values so that we can specify both our property name (the enum case) and the JSON name (the enum value) at the same time. It also needs to conform to the `CodingKey` protocol, which is what makes this work with the `Codable` protocol.

So, add this enum to the struct:

```swift
enum CodingKeys: String, CodingKey {
    case firstName = "user_first_name"
    case lastName = "user_last_name"
    case age
}
```

That will now able to decode the JSON as planned.

Note: The enum is called `CodingKeys` and the protocol is called `CodingKey`.

# Working with ISO-8601 dates

There are many ways of working with dates on the internet, but ISO-8601 is the most common. It encodes the full date in YYYY-MM-DD format, then the letter “T” to signal the start of time information, then the time in HH:MM:SS format, and finally a timezone. The timezone “Z”, short for “Zulu time” is commonly used to mean UTC.

`Codable` is able to handle ISO-8601 with a built-in date converter. So, given this JSON:

```swift
let json = """
[
    {
        "first_name": "Theo",
        "time_of_birth": "1999-04-03T17:30:31Z"
    }
]
"""
```

We can decode it like this:

```swift
struct Baby: Codable {
    var firstName: String
    var timeOfBirth: Date
}

let decoder = JSONDecoder()
decoder.keyDecodingStrategy = .convertFromSnakeCase
decoder.dateDecodingStrategy = .iso8601
```

That enables ISO-8601 date parsing, which converts from 1999-04-03T17:30:31Z to a `Date` instance, while also handling the snake case to camel case conversion.

# Working with other common dates

Swift comes with built-in support for three other important date formats. You use them just like you use ISO-8601 dates as shown above, so I’ll just talk about them briefly:

- The `.deferredToDate` format is Apple’s own date format, and it tracks the number of seconds and milliseconds since January 1st 2001. This isn’t really useful outside of Apple’s platforms.

- The `.millisecondsSince1970` format tracks the number of seconds and milliseconds since January 1st 1970. This is quite common online.

- The `.secondsSince1970` format tracks the number of whole seconds since January 1st 1970. This extremely common online, and is second only to ISO-8601.

# Working with custom dates

If your date format doesn’t match one of the built-in options, don’t despair: Codable can parse custom dates based on a date formatter you create.

For example, this JSON tracks the day a student graduated from university:

```swift
let json = """
[
    {
        "first_name": "Jess",
        "graduation_day": "31-08-2001"
    }
]
"""
```

That uses the date format DD-MM-YYYY, which isn’t one of Swift’s built-in options. Fortunately, you can provide a pre-configured `DateFormatter` instance as a date decoding strategy, like this:

```swift
let formatter = DateFormatter()
formatter.dateFormat = "dd-MM-yyyy"

let decoder = JSONDecoder()
decoder.keyDecodingStrategy = .convertFromSnakeCase
decoder.dateDecodingStrategy = .formatted(formatter)
```

# Working with weird dates

Sometimes you’ll get dates so strange that even `DateFormatter` can’t handle them. For example, you might get JSON that stores dates using the number of days that have elapsed since January 1st 1970:

```swift
let json = """
[
    {
        "first_name": "Jess",
        "graduation_day": 10650
    }
]
"""
```

To make that work we need to write a custom decoder for the date. Everything else will still be handled by `Codable` – we’re just providing a custom closure that will process the dates part.

You could try doing some hacky mathematics here such as multiplying the day count by 86400 (the number of seconds in a day), then using the `addTimeInterval()` method of `Date`. However, that won’t take into account daylight savings time and other date problems, so a better solution is to use `DateComponents` and `Calendar` like this:

```swift
let decoder = JSONDecoder()
decoder.keyDecodingStrategy = .convertFromSnakeCase
decoder.dateDecodingStrategy = .custom { decoder in
    // pull out the number of days from Codable
    let container = try decoder.singleValueContainer()
    let numberOfDays = try container.decode(Int.self)

    // create a start date of Jan 1st 1970, then a DateComponents instance for our JSON days
    let startDate = Date(timeIntervalSince1970: 0)
    var components = DateComponents()
    components.day = numberOfDays

    // create a Calendar and use it to measure the difference between the two
    let calendar = Calendar(identifier: .gregorian)
    return calendar.date(byAdding: components, to: startDate) ?? Date()
}
```

_Warning:_ If you have to parse lots of dates, remember that this closure will be run for every single one – make it fast!

# Parsing hierarchical data the easy way

Any non-trivial JSON is likely to have hierarchical data – one collection of data nested inside another. For example:

```swift
let json = """
[
    {
        "name": {
            "first_name": "Taylor",
            "last_name": "Swift"
        },
        "age": 26
    }
]
"""
```

`Codable` is able to handle this just fine, as long as you can describe the relationships clearly.

I find the easiest way to do this is using nested structs, like this:

```swift
struct User: Codable {
    struct Name: Codable {
        var firstName: String
        var lastName: String
    }

    var name: Name
    var age: Int
}
```

The downside is that if you want to read a user’s first name, you need to use `user.name.firstName`, but at least the actual parsing work is trivial – our existing code works already!

# Parsing hierarchical data the hard way

If you want to parse hierarchical data into a flat struct – i.e., you want to be able to write `user.firstName` rather than `user.name.firstName` - then you need to do some parsing yourself. This isn’t too hard, though, and `Codable` makes it beautifully type safe.

First, create the struct you want to end up with:

```swift
struct User: Codable {
    var firstName: String
    var lastName: String
    var age: Int
}
```

Second, we need to define coding keys that describe where data can be found in the hierarchy.

Let’s look at the JSON again:

```swift
let json = """
[
    {
        "name": {
            "first_name": "Taylor",
            "last_name": "Swift"
        },
        "age": 26
    }
]
"""
```

As you can see, at the root there’s a key called “name” and another called “age”, so we need to add that as our root coding keys. Put this inside your struct:

```swift
enum CodingKeys: String, CodingKey {
    case name, age
}
```

Inside “name” were two more keys, “first_name” and “last_name”, so we’re going to create some coding keys for those two. Add this:

```swift
enum NameCodingKeys: String, CodingKey {
    case firstName, lastName
}
```

Now for the hard part: we need to write a custom initializer and custom encode method for our type. Start by adding this empty method to your struct:

```swift
init(from decoder: Decoder) throws {
}
```

Inside there, the first thing we need to do is attempt to pull out a container we can read using the keys of our `CodingKeys` enum, like this:

```swift
let container = try decoder.container(keyedBy: CodingKeys.self)
```

Once that’s done we can attempt to read our `age` property. This is done in a type-safe way: you tell it the type you want to decode (`Int.self` for our age), along with a key name from the `CodingKeys` enum:

```swift
age = try container.decode(Int.self, forKey: .age)
```

Next we need to dig down one level to read our name data. As you saw earlier, “name” is a top-level key inside our `CodingKeys` enum, but it’s actually a nested container of other values that we need to read inside. So, we need to pull out that container:

```swift
let name = try container.nestedContainer(keyedBy: NameCodingKeys.self, forKey: .name)
```

And finally we can read two strings for the `.firstName` and `.lastName` keys:

```swift
firstName = try name.decode(String.self, forKey: .firstName)
lastName = try name.decode(String.self, forKey: .lastName)
```

That finishes the custom initializer, but we still have one more method to write: `encode(to:)`. This is effectively the reverse of the initializer we just wrote, because it’s job is to convert our properties back into nested container as appropriate.

This means creating a container based on our `CodingKeys` enum and writing age there, then creating a nested container based on our `NameCodingKeys` enum, and writing both `firstName` and `lastName` there:

```swift
func encode(to encoder: Encoder) throws {
    var container = encoder.container(keyedBy: CodingKeys.self)
    try container.encode(age, forKey: .age)

    var name = container.nestedContainer(keyedBy: NameCodingKeys.self, forKey: .name)
    try name.encode(firstName, forKey: .firstName)
    try name.encode(lastName, forKey: .lastName)
}
```

That completes all our code. With that in place, you can now read the `firstName` property directly, which is much nicer!

Reference: [https://www.hackingwithswift.com/articles/119/codable-cheat-sheet](https://www.hackingwithswift.com/articles/119/codable-cheat-sheet)
