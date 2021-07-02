---
layout: post
title: "Fundamentals of Swift Collections"
author: "Linh Vo"
tags: "Data-Structures-Algorithms"
---

The coding interview process is notoriously difficult, and the preparation process isn’t any easier. Developers often spend months preparing for their coding interviews. At most big tech companies, coding problems are the biggest part of the interview process.

[LeetCode](https://leetcode.com/) is a website where people can practice solving coding problems and prepare for technical interviews. The problems focus on algorithms and data structures.

A cheatsheet for Swift. Including examples of common Swift code and short explanations of what it does and how it works. Perfect for practising [LeetCode](https://leetcode.com/).

# Collection Types

Swift provides three primary collection types, known as **arrays**, **sets**, and **dictionaries**, for storing collections of values.

- **Arrays**: Ordered collections of values. Quick lookup by index, slow to lookup by value, slow to insert/delete.
- **Dictionaries**: Unordered collections of key-value associations. Quick lookups by key.
- **Sets**: Unordered collections of unique values. Quick lookup by value, quick to insert/delete.

{% include image.html
img="assets/2021-01-01/image.png" %}

# Arrays

### Loop array

```swift
for value in array
for index in array.indices
for (index, value) in array.enumerated()
```

### Loop array with range

```swift
for i in 0..<array.count // Traditional loop with indices
for v in array[2...] // Skip the first 2 elements
for v in array[..<(array.count-2)] // Skip the last 2 elements
for v in array.prefix(3) // The first 3
for v in array.suffix(3) // The last 3
```

### Special sequences

```swift
let tenOnes = repeatElement(1, count: 10) // Array(tenOnes)

for countdown in stride(from: 3, through: 1, by: -1) {
    print("\(countdown)...")
}
// 3...
// 2...
// 1...

for countdown in stride(from: 3, to: 0, by: -1) {
    print("\(countdown)...")
}
// 3...
// 2...
// 1...
```

### Array CRUD

```swift
// Find
firstIndex(of: x) // O(n)

// Insert
append(x) // O(1)
append(contentsOf: anArray)
insert(x, at: i) // O(n)
insert(contentsOf: anArray, at: i)
```

It is important to note that `append` **O(1)** is much faster than `insert`.

```swift
// Remove
remove(at: i) // O(n)
removeFirst()
removeLast()
removeAll(where: {..})

let numbers = [1, 2, 3, 4, 5]
print(numbers.dropFirst(2))
// Prints "[3, 4, 5]"
print(numbers.dropFirst(10))
// Prints "[]"

let numbers = [1, 2, 3, 4, 5]
print(numbers.dropLast(2))
// Prints "[1, 2, 3]"
print(numbers.dropLast(10))
// Prints "[]"
```

### Mutate

```swift
sorted()
reversed()
shuffled()
swapAt(i, j)
```

There are more ops using your own predicate: `max(by:)`, `sorted(by:)`, `filter()`, `map()`

# Dictionaries

An optimizing trick is to make use of the quick O(1) operation when accessing a dictionary, since they are hashed.

The trade off is that the setup takes O(n), and a space of O(n). And you need the type to be hashable.

### Creating a Dictionary

```swift
var airports = ["YYZ": "Toronto Pearson", "DUB": "Dublin"]

print("The airports dictionary contains \(airports.count) items.")
// Prints "The airports dictionary contains 2 items."
```

### Loop dictionary

```swift
for (key, value) in dictionary
```

### Check if dictionary has key

```swift
dictionary.keys.contains("k")

// Alternative, if the value type is _never_ `Optional`
if dictionary["k"] == nil // "k" is not in keys
```

### Accessing and Modifying a Dictionary

```swift
if airports.isEmpty {
    print("The airports dictionary is empty.")
} else {
    print("The airports dictionary isn't empty.")
}
// Prints "The airports dictionary isn't empty."

airports["LHR"] = "London"
// the airports dictionary now contains 3 items

airports["LHR"] = "London Heathrow"
// the value for "LHR" has been changed to "London Heathrow"

if let oldValue = airports.updateValue("Dublin Airport", forKey: "DUB") {
    print("The old value for DUB was \(oldValue).")
}
// Prints "The old value for DUB was Dublin."

if let airportName = airports["DUB"] {
    print("The name of the airport is \(airportName).")
} else {
    print("That airport isn't in the airports dictionary.")
}
// Prints "The name of the airport is Dublin Airport."

airports["APL"] = "Apple International"
// "Apple International" isn't the real airport for APL, so delete it
airports["APL"] = nil
// APL has now been removed from the dictionary

if let removedValue = airports.removeValue(forKey: "DUB") {
    print("The removed airport's name is \(removedValue).")
} else {
    print("The airports dictionary doesn't contain a value for DUB.")
}
// Prints "The removed airport's name is Dublin Airport."
```

# Sets

Sets in Swift are similar to arrays and dictionaries. Just like arrays and dictionaries, the `Set` type is used to store multiple items of the same type in one collection.

Here's an example:

```swift
var fruit:Set = ["apple", "banana", "strawberry", "jackfruit"]
```

You can add and remove items like this:

```swift
fruit.insert("pineapple")
fruit.remove("banana")
```

Sets are different from arrays and dictionaries, in these ways:

- Sets don't have an order – they're literally unsorted
- Every item in a set needs to be unique
- Sets don't have indices or keys
- Instead, a set's values need to be _hashable_
- Because set items are hashable, you can search sets in _O(1)_ time

Here's how you can quickly search a set:

```swift
let movies:Set = ["Rocky", "The Matrix", "Lord of the Rings"]

if movies.contains("Rocky") {
    print("Rocky is one of your favorite movies!")
}
```

Sets are particularly useful for membership operations, to find out if sets have items in common for example. We can make a union of sets, subtract sets, intersect them, and find their differences.

Consider the following Italian coffees and their ingredients:

```swift
let cappuccino:Set = ["espresso", "milk", "milk foam"]
let americano:Set  = ["espresso", "water"]
let machiato:Set   = ["espresso", "milk foam"]
let latte:Set      = ["espresso", "milk"]
```

Can we find the **union** (add items) of two coffees?

```swift
machiato.union(latte)
// ["espresso", "milk foam", "milk"]
```

Can we **subtract** one coffee from another?

```swift
cappuccino.subtracting(americano)
// ["milk foam", "milk"]
```

Can we find the **intersection** (shared items) of two coffees?

```swift
latte.intersection(cappuccino)
// ["espresso", "milk"]
```

Can we find the **difference** between two coffees?

```swift
latte.symmetricDifference(americano)
// ["milk", "water"]
```

# Comparable, Equatable

To compare and sort, the type must implement the `Comparable` protocol.

```swift
extension MyClass: Comparable {
    static func < (lhs: MyClass, MyClass: Date) -> Bool {
        return lhs.foo < rhs.foo
    }
}
```

[Comparable](https://developer.apple.com/documentation/swift/comparable) inherits [Equatable](https://developer.apple.com/documentation/swift/equatable) protocol, so you might need to implement it too. In cases like struct where Equatable is provided by default, you need not implement it.

```swift
extension MyClass: Equatable {
    public static func == (lhs: MyClass, rhs: MyClass) -> Bool {
        return lhs.foo == rhs.foo
    }
}
```

# Identity Comparison ===

[https://developer.apple.com/documentation/swift/1538988](https://developer.apple.com/documentation/swift/1538988)

Returns a Boolean value indicating whether two references point to the same object instance.

```swift
func === (lhs: AnyObject?, rhs: AnyObject?) -> Bool
```

This operator tests whether two instances have the same identity, not the same value. For value equality, see the equal-to operator (==) and the Equatable protocol.

The following example defines an IntegerRef type, an integer type with reference semantics.

```swift
class IntegerRef: Equatable {
    let value: Int
    init(_ value: Int) {
        self.value = value
    }
}

func ==(lhs: IntegerRef, rhs: IntegerRef) -> Bool {
    return lhs.value == rhs.value
}
```

Because IntegerRef is a class, its instances can be compared using the identical-to operator (===). In addition, because IntegerRef conforms to the Equatable protocol, instances can also be compared using the equal-to operator (==).

```swift
let a = IntegerRef(10)
let b = a
print(a == b)
// Prints "true"
print(a === b)
// Prints "true"
```

The identical-to operator (===) returns false when comparing two references to different object instances, even if the two instances have the same value.

```swift
let c = IntegerRef(10)
print(a == c)
// Prints "true"
print(a === c)
// Prints "false"
```

# Hashable

[Hashable](https://developer.apple.com/documentation/swift/hashable) - A type that can be hashed into a Hasher to produce an integer hash value.

```swift
extension MyClass: Hashable {
    public func hash(into hasher: inout Hasher) {
        hasher.combine(ObjectIdentifier(self).hashValue)
    }
}
```
