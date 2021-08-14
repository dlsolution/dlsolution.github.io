---
layout: post
title: "Data Structure in Swift: Queue"
author: "Linh Vo"
tags: "Data-Structures-And-Algorithms"
---

# Overview

Queue is an abstract data structure that represent queue in real world like the picture above. In computer science terminology, queue is **FIFO** (First in First out) which mean the first data that come in to the queue also will be the first data come out from the queue. You might need to use queue if you have to maintain the order of the data.

# Basic Operations

First of all, let’s define basic protocol for queue here

```swift
public protocol Queue {

  associatedtype Element
  mutating func enqueue(_ element: Element) -> Bool
  mutating func dequeue() -> Element?
  var isEmpty: Bool { get }
  var peek: Element? { get }
}
```

There are two main operations in queue data structure which are **enqueue** and **dequeue**. We also define two additional properties in this protocol.

1. **Enqueue** - This operation is add new data to the end of the queue. This function will return true if operation is success.

2. **Dequeue** - This operation is remove the first data of the queue. If the queue is not empty, then it should return the data.

3. **isEmpty** - This property will check if the queue is empty or not. Will return boolean value.

4. **Peek** - This property will get the first data of the queue without removing it.

So how we can create this data structure in Swift? well basically we can create queue with four ways.

1. Array
2. Doubly linked list
3. Ring Buffer
4. Two Stacks

But in this article I will create the queue just using array implementation. Let’s dive to the code.

# Implementation

## QueueArray

```swift
public struct QueueArray<T>: Queue {

  private var array: [T] = []
  public init() {}

  public var isEmpty: Bool {
    return array.isEmpty
  }

  public var peek: T? {
    return array.first
  }

  public mutating func enqueue(_ element: T) -> Bool {
    array.append(element)
    return true
  }

  public mutating func dequeue() -> T? {
    return isEmpty ? nil : array.removeFirst()
  }
}

extension QueueArray: CustomStringConvertible {

  public var description: String {
    return array.description
  }
}
```

## QueueStack

```swift
public struct QueueStack<T> : Queue {

  private var leftStack: [T] = []
  private var rightStack: [T] = []
  public init() {}

  public var isEmpty: Bool {
    return leftStack.isEmpty && rightStack.isEmpty
  }

  public var peek: T? {
    return !leftStack.isEmpty ? leftStack.last : rightStack.first
  }

  public mutating func enqueue(_ element: T) -> Bool {
    rightStack.append(element)
    return true
  }

  public mutating func dequeue() -> T? {
    if leftStack.isEmpty {
      leftStack = rightStack.reversed()
      rightStack.removeAll()
    }
    return leftStack.popLast()
  }
}

extension QueueStack: CustomStringConvertible {

  public var description: String {
    let printList = leftStack + rightStack.reversed()
    return printList.description
  }
}
```

# Conclusion

- Queue is **FIFO** (First in First out) data structure, which mean the first data that was added to the queue will always be the first data removed from the queue.

- Use queue if you need to maintain the order of the data. You just need to insert data to the last and remove the first data. You don’t care the data in between them.

- It not recommend to use array for create queue if you have a huge data set. Because when you call dequeue operation it take O(N) complexity. You can think another option like doubly linked list, ring buffer, or two stacks.
