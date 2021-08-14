---
layout: post
title: "Data Structure in Swift: Stack"
author: "Linh Vo"
tags: "Data-Structures-And-Algorithms"
---

# Overview

Stack is abstract data structure that have a similar representation with physical stack in a real world. You are recommended to use stack if you want to minimise access to the data. There are two main operation that stack used, which are **push** and **pop**. **Push** mean you add new data to top the stack and **pop** mean you remove the last data or the top of the stack. In computer science terminology, stack use **LIFO** (Last in First out) which mean the last data you are inserted to stack, would be the first data that will be removed when you pop the stack.

There are some real use case that used stack as a data structure. Stack is used in UINavigationController to push and pop view controller in iOS development. Memory allocation — in high level also use stack to manage allocation resources. If you familiar with Search and Conquer algorithm, it also use stack for provide backtracking implementation.

# Implementation

```swift
public struct Stack<Element> {

  private var storage: [Element] = []

  public init() { }

  public init(_ elements: [Element]) {
    storage = elements
  }

  // 1
  public mutating func push(_ element: Element) {
    storage.append(element)
  }

  // 2
  @discardableResult
  public mutating func pop() -> Element? {
    return storage.popLast()
  }

  // 3
  public func peek() -> Element? {
    return storage.last
  }

  public var isEmpty: Bool {
    return peek() == nil
  }
}
```

1. **Push** - In this implementation, I just used array in order to make our life become more easier. Push operation is we add the new data to the top of the stack. Because we use array, we can use append function and it just take O(1) complexity.

2. **Pop** - Because we use array, remove the last data is easy to implemented. Just call popLast and it also take O(1) complexity.

3. **Peek** - Peek is optional, but it will help a lot for solve complex problem. Basically peek is we get the last data of the stack. We can use last property from storage to get the last data — or the top of the stack. It take O(1) complexity.

# Conclusion

- Stack is **LIFO** (Last in First out) data structure

- Stack is really useful for solve complex problem such finding path in a maze or another search and conquer problem

- Use stack if you want to minimise access to your data

- There are two main operation which are **push** and **pop**
