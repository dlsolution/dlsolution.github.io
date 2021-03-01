---
layout: post
title: "Useful of Protocol"
author: "Linh Vo"
tags: "UIKit"
---

We might all have gone through the pain of spending a lot of time to implement a function, but then realize that Swift has a built-in function that does the same thing. In this article, I’m going to talk about 5 useful protocols that can save you much time, and take your code to the next level.

# CaseIterable

The `CaseIterable` protocol allows you to get all the values of the type. Let’s take a look at an example:

**Without:**

```swift
enum City {
    case new_york
    case bei_jing
    case vancouver
    ....
}
let cities: [City] = [.new_york, .bei_jing, .vancouver, ....]
```

In this example, when we want to pull out all the cities from the City enum, we would have to manually type them out. Imagine that if this enum has hundreds of thousands of cities, it would be a nightmare.

**With CaseIterable:**

```swift
enum City: CaseIterable {
    case new_york
    case bei_jing
    case vancouver
    ....
}
let cities = City.allCases
```

With the `allCases` property provided by the `CaseIterable` protocol, we are able to get an array of all the cases of City. This can save us a significant amount of time.

# CustomStringConvertible

Not everyone likes the default description for each object, sometimes, we want to give an object a user-friendly description just for reading purposes.

**Without:**

```swift
struct Weather {
    let city: String
    let temperature: Double
}
let weather = Weather(city: "New York", temperature: 97.5)
print("The temperature in \(weather.city) is \(weather.temperature)°F.")
print("The temperature in \(weather.city) is \(weather.temperature)°F.")
print("The temperature in \(weather.city) is \(weather.temperature)°F.")

```

Every time, we want to print a readable message, we have to copy and paste the message over. If at some point we want to modify the message, we will have to go through everything single place which would waste a lot of time.

**With CustomStringConvertible:**

```swift
struct Weather: CustomStringConvertible {
    let city: String
    let temperature: Double
    var description: String {
        return "The temperature in \(city) is \(temperature)°F."
    }
}
let weather = Weather(city: "New York", temperature: 97.6)
print(weather) // The temperature in New York is 97.6°F.
print(weather) // The temperature in New York is 97.6°F.
print(weather) // The temperature in New York is 97.6°F.
```

By inheriting the CustomStringConvertible protocol, we need to provide a value to the description property. Every time, we want to use the object as a String, the program will refer to the description property. Now we can manage our message in one place.

# IteratorProtocol

This protocol provides a .next() function that allows you to access to the next value. Let’s say we have a list of names and we want to iterate the list one by one without the need to store the current index.

**Without:**

```swift
let names = ["Bob", "Amy", "Mike"]
var currentIndex = 0
print("\(names[currentIndex]) did the laundry.")
currentIndex += 1
print("\(names[currentIndex]) did the dishes.")
currentIndex += 1
print("\(names[currentIndex]) did nothing.")
// Outputs
Bob did the laundry.
Amy did the dishes.
Mike did nothing.
```

Every time, we want to print a readable message, we have to copy and paste the message over. If at some point we want to modify the message, we will have to go through everything single place which would waste a lot of time.

**With IteratorProtocol:**

```swift
let names = ["Bob", "Amy", "Mike"]
var nameIterator = names.makeIterator()
print("\(nameIterator.next() ?? "") did the laundry.")
print("\(nameIterator.next() ?? "") did the dishes.")
print("\(nameIterator.next() ?? "") did nothing.")
// Outputs
Bob did the laundry.
Amy did the dishes.
Mike did nothing.
```

The makeIterator() function returns an object that inherits the IteratorProtocol protocol. We can use the next() function to get the next value without the need of tracking the current index. Remember, the .next() function returns an optional value.

# Conclusion

There are tons of useful built-in protocols out there and I can’t list them all in one article. It would really waste you time if you write code from scratch when Swift already provides built-in supports. Using these protocols can also make your code much cleaner and more efficient.
