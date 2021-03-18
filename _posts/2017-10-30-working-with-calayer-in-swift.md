---
layout: post
title: "Working with CALayer in Swift"
author: "Linh Vo"
tags: "UIKit"
---

# What is CALayer?

`CALayer` is an object responsible for drawing and animations and it is part of Core Animation framework. Each `UIView` backed by `CALayer` class, and you can easily find it in `view.layer` property. This class helps us to modify views and animate them.

You can change the basic properties of UIView like `cornerRadius`, `shadow`, `border`.

```swift
// MARK: Change cornerRadius UIView
view.layer.cornerRadius = 5

// MARK: Set shadow to UIView
view.layer.shadowOffset = CGSize(width: 5, height: 5)
view.layer.shadowOpacity = 0.7
view.layer.shadowRadius = 5
view.layer.shadowColor = UIColor.green.cgColor

// MARK: Set border to UIView
view.layer.borderColor = UIColor.green.cgColor
view.layer.borderWidth = 2.0

// MARK: Set backgroundColor to UIView
view.layer.backgroundColor = UIColor.yellow.cgColor
```

`UIView` has only one layer in `view.layer` property. But, you can easily add sublayer to this property like add subviews to view.

```swift
var sublayers: [CALayer]? - //Array of all sublayers in current layer

var superlayer: CALayer? - //Parent layer if exist

func addSublayer(CALayer) - //Add new sublayer

func removeFromSuperlayer() - //Remove layer from superLayer

func insertSublayer(CALayer, at: UInt32) - //Insert new layer on specific index

func insertSublayer(CALayer, below: CALayer?) - //Insert sublayer bellow specific layer

func insertSublayer(CALayer, above: CALayer?) - //Insert sublayer above specific layer

func replaceSublayer(CALayer, with: CALayer) - //Switch places of 2 sublayers
```

All these methods are very similar to methods for work with UIViews.

We can easily add sublayers to another layer like in code bellow:

```swift
// MARK: Creation of sublayer1
let sublayer1 = CALayer()
sublayer1.frame = CGRect(x: 20, y: 20, width: 30, height: 30)
sublayer1.backgroundColor = UIColor.red.cgColor

// MARK: Creation of sublayer2
let sublayer2 = CALayer()
sublayer2.frame = CGRect(x: 80, y: 80, width: 40, height: 40)
sublayer2.backgroundColor = UIColor.green.cgColor

//Adding of sublayers to view.layer
view.layer.addSublayer(sublayer1)
view.layer.addSublayer(sublayer2)
```

# How to animate properties of CALayer?

`CALayer` can have multiple animations. I will show you how to animate the layer’s properties. Let’s create an object of `CABasicAnimation`.

```swift
// MARK: Position Animation
let positionAnimation = CABasicAnimation(keyPath: #keyPath(CALayer.position))
positionAnimation.fromValue = CGPoint(x: -20, y: 100)
positionAnimation.toValue = CGPoint(x: 280, y: 100)
positionAnimation.duration = globalDuration
positionAnimation.repeatCount = globalRepeatCount
positionAnimation.autoreverses = true

// MARK: Add Animation to Layer
view.layer.add(positionAnimation, forKey: #keyPath(CALayer.position))
```

# What the difference between CALayer animation and UIView.animate way?

Animations of UIViews happens on the main thread and it means it is using CPU power. Layers are drawn directly on the GPU. It happens on a separate thread without burdening the CPU. So `CALayer` animations are more efficient, but we can’t animate layout constraints of view with `CALayer`.

## How to synchronize them?

`CATransaction` is a singleton class that can help to work with animations. All code between `CATransaction.begin()` and `CATransaction.commit()` will be executed at same time. `CATransaction.setCompletionBlock` will help you to handle the end of the animation. So it is easy to synchronize multiple animations.

```swift
CATransaction.begin()  //Start of animation transaction
CATransaction.setCompletionBlock { //Transaction is Ready
   print("READY")
}

heightConstraint.constant = 80.0

UIView.animate(withDuration: 2.0) { // UIView type of animation
   view.layoutIfNeeded()
}

let cornerAnimation = CABasicAnimation(keyPath: #keyPath(CALayer.cornerRadius)) // CALayer type of animation
cornerAnimation.fromValue = 0
cornerAnimation.toValue = 12
view.layer.add(cornerAnimation, forKey: #keyPath(CALayer.cornerRadius))

CATransaction.commit() //End of animation transaction
```

# How to use multiple animations?

```swift
// CornerRadius Animation
let cornerAnimation = CABasicAnimation(keyPath: #keyPath(CALayer.cornerRadius))
...

// Position Animation
let positionAnimation = CABasicAnimation(keyPath: #keyPath(CALayer.position))
...

// Background Animation
let backgroundAnimation = CABasicAnimation(keyPath: #keyPath(CALayer.backgroundColor))
...

// Bounds Animation
let boundsAnimation = CABasicAnimation(keyPath: #keyPath(CALayer.bounds))
...

// Creation of sublayer
let sublayer = CALayer()
...

// Apply all animations to sublayer
CATransaction.begin()
sublayer.add(positionAnimation, forKey: #keyPath(CALayer.position))
sublayer.add(cornerAnimation, forKey: #keyPath(CALayer.cornerRadius))
sublayer.add(backgroundAnimation, forKey: #keyPath(CALayer.backgroundColor))
sublayer.add(boundsAnimation, forKey: #keyPath(CALayer.bounds))
CATransaction.commit()
```

[Here](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CoreAnimation_guide/AnimatableProperties/AnimatableProperties.html) you can find all properties that you can animate in this way.
