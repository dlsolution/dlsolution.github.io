---
layout: post
title: "Fundamentals of Swift Collections"
author: "Linh Vo"
tags: "Data-Structures-Algorithms"
---

The coding interview process is notoriously difficult, and the preparation process isn’t any easier. Developers often spend months preparing for their coding interviews. At most big tech companies, coding problems are the biggest part of the interview process.

[LeetCode](https://leetcode.com/) is a website where people can practice solving coding problems and prepare for technical interviews. The problems focus on algorithms and data structures.

A cheatsheet for Swift. Including examples of common Swift code and short explanations of what it does and how it works. Perfect for practising [LeetCode](https://leetcode.com/).

# Loop array

```swift
for value in array
for index in array.indices
for (index, value) in array.enumerated()
```

# Loop array with range

```swift
for i in 0..<array.count // Traditional loop with indices
for v in array[2...] // Skip the first 2 elements
for v in array[..<(array.count-2)] // Skip the last 2 elements
for v in array.prefix(3) // The first 3
for v in array.suffix(3) // The last 3
```

# Loop dictionary

```swift
for (key, value) in dictionary
```

# Check if dictionary has key

```swift
dictionary.keys.contains("k")

// Alternative, if the value type is _never_ `Optional`
if dictionary["k"] == nil // "k" is not in keys
```

# Special sequences

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

# Array CRUD

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

# Mutate

```swift
sorted()
reversed()
shuffled()
swapAt(i, j)
```

There are more ops using your own predicate: `max(by:)`, `sorted(by:)`, `filter()`, `map()`

# Comparable, Equatable

To compare and sort, the type must implement the `Comparable` protocol.

```swift
extension MyClass: Comparable {
    static func < (lhs: MyClass, MyClass: Date) -> Bool {
        return lhs.foo < rhs.foo
    }
}
```

`Comparable` inherits `Equatable` protocol, so you might need to implement it too. In cases like struct where Equatable is provided by default, you need not implement it.

```swift
extension MyClass: Equatable {
    public static func == (lhs: MyClass, rhs: MyClass) -> Bool {
        return lhs.foo == rhs.foo
    }
}
```

# Identity Comparison ===

There is an op `===` for identity comparison eg. are they the same instances?

```swift
// For example, in the above Equatable implementation, we could also compare their identities
return lhs === rhs
```

# Dictionary O(1) access

An optimizing trick is to make use of the quick O(1) operation when accessing a dictionary, since they are hashed.

The trade off is that the setup takes O(n), and a space of O(n). And you need the type to be hashable.

# Hashable

```swift
extension MyClass: Hashable {
    public func hash(into hasher: inout Hasher) {
        hasher.combine(ObjectIdentifier(self).hashValue)
    }
}
```

# Set

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
