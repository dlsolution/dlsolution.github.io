---
layout: post
title: "How to add a shadow to a UIView"
author: "Linh Vo"
tags: "UIKit"
---

Here's a simple example to get you started:

```swift
let yourView = UIView()
yourView.layer.shadowColor = UIColor.black.cgColor
yourView.layer.shadowOpacity = 1
yourView.layer.shadowOffset = .zero
yourView.layer.shadowRadius = 10
```

There are four you need to care about:

- `shadowColor` sets the color of the shadow, and needs to be a CGColor.

- `shadowOpacity` sets how transparent the shadow is, where 0 is invisible and 1 is as strong as possible.

- `shadowOffset` sets how far away from the view the shadow should be, to give a 3D offset effect.

- `shadowRadius` sets how wide the shadow should be.

Be warned: generating shadows dynamically is expensive, because iOS has to draw the shadow around the exact shape of your view's contents. If you can, set the `shadowPath` property to a specific value so that iOS doesn't need to calculate transparency dynamically. For example, this creates a shadow path equivalent to the frame of the view:

```swift
yourView.layer.shadowPath = UIBezierPath(rect: yourView.bounds).cgPath
```

## How to solve the masksToBounds problem with the shadow of UIView?

All layers have `masksToBounds` property. If `masksToBounds = true` – It will clip all layers that are bigger than their superlayer. And this is a useful property. As we know the shadow is an element out of view box. And it will be clipped with `masksToBounds = true`. And it is a problem! Another problem is image. We can’t clip image if `masksToBounds = false`. So it is impossible to create an image like bellow with the only `view.layer` property

{% include image.html
img="assets/2017-11-22/image.jpg" %}

It is difficult because of image needs `masksToBounds = true`, and red shadow needs `masksToBounds = false`.

```swift
let rect = CGRect(x: 40, y: 100, width: 300, height: 250)
let view = UIView(frame: rect)
view.backgroundColor = UIColor.systemPink

//Add Shadow
view.layer.shadowColor = UIColor.red.cgColor
view.layer.shadowOffset = CGSize(width: 20, height: 20)
view.layer.shadowRadius = 10.0
view.layer.shadowOpacity = 0.7
view.layer.shadowPath = UIBezierPath(rect: view.bounds).cgPath

//Add cornerRadius
view.layer.cornerRadius = 20.0
view.layer.borderColor = UIColor.gray.cgColor
view.layer.borderWidth = 4.0

//Add image as CALayer
let image = UIImage(named: "image.png")
let imageLayer = CALayer()
imageLayer.contents = image?.cgImage
imageLayer.frame = view.bounds
imageLayer.masksToBounds = true //Only imageLayer will be masked
imageLayer.cornerRadius = 20.0
imageLayer.contentsGravity = .resizeAspectFill
view.layer.addSublayer(imageLayer)
self.view.addSubview(view)
```

In the code above, we have a default layer setup – cornerRadius, shadow, and border. We need `view.layer.masksToBounds = false`, shadow effect will be invisible if `masksToBounds` will be equal to true.

The trick in the image layer. As we can’t set `masksToBounds` for the parent layer, we can create another layer only for image and mask it. And it is a cheap trick to have a few layers with different `masksToBounds` value.
