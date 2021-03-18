---
layout: post
title: "Drawing using UIBezierPath and CAShapeLayer"
author: "Linh Vo"
tags: "UIKit"
---

Every view in iOS has a `CALayer` with it which can be used to add multiple customised layer inside it. It works similar to any `UIView`, we can have a parent view and add multiple child views inside it. Similarly iOS provides us with a parent layer for every view and we will create separate layers for creating different shapes and then add that layer to the main layer.

# Drawing an Bezier Lines

```swift
//Path part
let path = UIBezierPath()
path.move(to: CGPoint(x: 200, y: 200)) //StartPoint
path.addLine(to: CGPoint(x: 380, y: 380)) //EndPoint of First Line and StartPoint for Second Line
path.addLine(to: CGPoint(x: 20, y: 380)) //EndPoint of Second Line

//Shape part
let shape = CAShapeLayer()
shape.path = path.cgPath
shape.lineWidth = 4.0
shape.fillColor = UIColor.clear.cgColor
shape.strokeColor = UIColor.orange.cgColor
view.layer.addSublayer(shape)
```

Output looks like this:
{% include image.html
img="assets/2018-01-04/image10.png" %}

# Drawing a Rectangle

```swift
let path = UIBezierPath()
path.move(to: CGPoint(x: 0, y: 0))
path.addLine(to: CGPoint(x: 200, y: 0))
path.addLine(to: CGPoint(x: 200, y: 200))
path.addLine(to: CGPoint(x: 0, y: 200))
path.addLine(to: CGPoint(x: 0, y: 0))

let shapeLayer = CAShapeLayer()
shapeLayer.path = path.cgPath
shapeLayer.strokeColor = UIColor.black.cgColor
shapeLayer.fillColor = UIColor.orange.cgColor
shapeLayer.lineWidth = 3

view.layer.addSublayer(shapeLayer)
```

Output looks like this:
{% include image.html
img="assets/2018-01-04/image1.png" %}

# Drawing a Triangle

```swift
let path = UIBezierPath()
path.move(to: CGPoint(x: 0, y: 200))
path.addLine(to: CGPoint(x: 100, y: 0))
path.addLine(to: CGPoint(x: 200, y: 200))
path.addLine(to: CGPoint(x: 0, y: 200))

let shapeLayer = CAShapeLayer()
shapeLayer.path = path.cgPath
shapeLayer.strokeColor = UIColor.red.cgColor
shapeLayer.fillColor = UIColor.green.cgColor
shapeLayer.lineWidth = 3

view.layer.addSublayer(shapeLayer)
```

Output looks like this:
{% include image.html
img="assets/2018-01-04/image2.png" %}

# Drawing an Oval/Circle

```swift
let path = UIBezierPath(ovalIn: containerView.bounds)

let shapeLayer = CAShapeLayer()
shapeLayer.path = path.cgPath
shapeLayer.fillColor = UIColor.orange.cgColor
shapeLayer.lineWidth = 3
shapeLayer.strokeColor = UIColor.black.cgColor

containerView.layer.addSublayer(shapeLayer)
```

Output looks like this:
{% include image.html
img="assets/2018-01-04/image3.png" %}

# Drawing an Arc

```swift
let path = UIBezierPath(arcCenter: CGPoint(x: 100, y: 100), radius: 100, startAngle: 0, endAngle: .pi, clockwise: false)

let shapeLayer = CAShapeLayer()
shapeLayer.path = path.cgPath
shapeLayer.fillColor = UIColor.orange.cgColor
shapeLayer.lineWidth = 3
shapeLayer.strokeColor = UIColor.black.cgColor

containerView.layer.addSublayer(shapeLayer)
```

Output looks like this:
{% include image.html
img="assets/2018-01-04/image4.png" %}

1. `arcCenter` -> This is the point from which the arc is drawn using it as a centre.

2. `radius` -> Radius of the arc

3. `startAngle` -> The angle from which arc starts

4. `endAngle` -> The angle where arc ends

5. `clockwise` -> Whether the arc is drawn clockwise or not

# Drawing an Bezier QuadCurve

If you want to draw a curve with 1 control point you can easily do it with `addQuadCurve` method.

```swift
//Path part
let path = UIBezierPath()
path.move(to: CGPoint(x: 50, y: 50))
path.addQuadCurve(to: CGPoint(x: 200, y: 50),
                  controlPoint: CGPoint(x: 100, y: 200))

//Shape part
let shape = CAShapeLayer()
shape.path = path.cgPath
shape.lineWidth = 4.0
shape.fillColor = UIColor.clear.cgColor
shape.strokeColor = UIColor.orange.cgColor
view.layer.addSublayer(shape)
```

Output looks like this:
{% include image.html
img="assets/2018-01-04/image7.png" %}

# Drawing an Bezier Curve

If you want to draw a curve with 2 control points you can do it with `addCurve` method. With 2 control points, you can easily build difficult paths with any form as you need.

```swift
//Path Part
let path = UIBezierPath()
path.move(to: CGPoint(x: 50, y: 200))
path.addCurve(to: CGPoint(x: 200, y: 200),
              controlPoint1: CGPoint(x: 80, y: 300),
              controlPoint2: CGPoint(x: 150, y: 0))
//Shape Part
let shape = CAShapeLayer()
shape.path = path.cgPath
shape.lineWidth = 4.0
shape.fillColor = UIColor.clear.cgColor
shape.strokeColor = UIColor.orange.cgColor
view.layer.addSublayer(shape)
```

Output looks like this:
{% include image.html
img="assets/2018-01-04/image8.png" %}

# Adding multiple layers

Recall that every view has a `CALayer` and we can add many sublayers to it. In this section we are going to create a `rectangle` which has `circle` and `triangle` inside it.

Three are four different methods by which we can insert a sublayer on to a layer:

1. `addSublayer(<CALayer>)` -> This simply adds a new layer on the top of the stack of layers

2. `insertSublayer(<CALayer>, at: <UInt32>` -> This method takes an extra parameter at which inserts the layer at the given position inside the stack of layers

3. `insertSublayer(<CALayer>, above: <CALayer?>` -> This method inserts the layer above a particular layer

4. `insertSublayer(<CALayer>, below: <CALayer?>` -> This methods inserts the layer below a particular layer

```swift
private func addMultipleLayers() {

        let path1 = UIBezierPath()
        path1.move(to: CGPoint(x: 0, y: 0))
        path1.addLine(to: CGPoint(x: 200, y: 0))
        path1.addLine(to: CGPoint(x: 200, y: 200))
        path1.addLine(to: CGPoint(x: 0, y: 200))
        path1.addLine(to: CGPoint(x: 0, y: 0))

        let path2 = UIBezierPath()
        path2.move(to: CGPoint(x: 0, y: 200))
        path2.addLine(to: CGPoint(x: 100, y: 0))
        path2.addLine(to: CGPoint(x: 200, y: 200))
        path2.addLine(to: CGPoint(x: 0, y: 200))

        let path3 = UIBezierPath(ovalIn: CGRect(x: 50, y: 100, width: 100, height: 100))

        let shapeLayer1 = CAShapeLayer()
        shapeLayer1.path = path1.cgPath
        shapeLayer1.strokeColor = UIColor.black.cgColor
        shapeLayer1.fillColor = UIColor.orange.cgColor
        shapeLayer1.lineWidth = 3

        let shapeLayer2 = CAShapeLayer()
        shapeLayer2.path = path2.cgPath
        shapeLayer2.strokeColor = UIColor.red.cgColor
        shapeLayer2.fillColor = UIColor.green.cgColor
        shapeLayer2.lineWidth = 3

        let shapeLayer3 = CAShapeLayer()
        shapeLayer3.path = path3.cgPath
        shapeLayer3.fillColor = UIColor.orange.cgColor
        shapeLayer3.lineWidth = 3
        shapeLayer3.strokeColor = UIColor.black.cgColor

        containerView.layer.addSublayer(shapeLayer1)
        containerView.layer.insertSublayer(shapeLayer2, above: shapeLayer1)
        containerView.layer.insertSublayer(shapeLayer3, above: shapeLayer2)
}
```

Output looks like this:
{% include image.html
img="assets/2018-01-04/image5.png"
title=""
caption="" %}

# Shapes Part

Let’s talk a little bit about the shape part. `UIBezierPath` creation isn’t enough. You need to put a path inside `CAShapeLayer`. You can customize (`lineWidth`, `fillColor`, `strokeColor`) your `CAShapeLayer`. `UIBezierPath` can’t be customized because it is just a curve.
