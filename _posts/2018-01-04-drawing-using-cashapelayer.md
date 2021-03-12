---
layout: post
title: "Drawing using CAShapeLayer"
author: "Linh Vo"
tags: "UIKit"
---

Every view in iOS has a `CALaye`r with it which can be used to add multiple customised layer inside it. It works similar to any `UIView`, we can have a parent view and add multiple child views inside it. Similarly iOS provides us with a parent layer for every view and we will create separate layers for creating different shapes and then add that layer to the main layer.

# Drawing a Rectangle

```swift
class MyViewController : UIViewController {

    let containerView: UIView = {
        let view = UIView()
        view.backgroundColor = .clear
        return view
    }()

    override func viewDidLoad() {
        super.viewDidLoad()

        containerView.frame = CGRect(x: view.frame.width/2 - 100, y: view.frame.height/2 - 100, width: 200, height: 200)
        view.addSubview(containerView)

        drawRectangle()
    }

    private func drawRectangle() {

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

        containerView.layer.addSublayer(shapeLayer)
    }
}
```

Output looks like this:
{% include image.html
img="assets/2018-01-04/image1.png"
title=""
caption="" %}

Let’s understand what all we did to achieve this.

1. We have created a `containerView` on which we will draw our custom shapes.

2. Inside `drawRectangle` method we first create a path using `UIBezierPath` and then draw all the four side of a rectangle using `addLine` method. Before adding all the lines we have used a method `moveTo` which is used to point at the place where our path will start drawing.

3. After that we have created a new layer of type `CAShapeLayer` whose path is the custom path that we created above. We also assigned some colour properties and line width to make our rectangle look a bit nice.

4. At last we used the `containerView's` layer and added our layer to it as a `subLayer`

# Drawing a Triangle

```swift
private func drawTriangle() {

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

    containerView.layer.addSublayer(shapeLayer)
}
```

Output looks like this:
{% include image.html
img="assets/2018-01-04/image2.png"
title=""
caption="" %}

For creating a triangle there’s no magic. We have simply change the `CGPoint` inside `addLine` method to make it a rectangle.

# Drawing an Oval/Circle

```swift
private func drawOval() {
    let path = UIBezierPath(ovalIn: containerView.bounds)

    let shapeLayer = CAShapeLayer()
    shapeLayer.path = path.cgPath
    shapeLayer.fillColor = UIColor.orange.cgColor
    shapeLayer.lineWidth = 3
    shapeLayer.strokeColor = UIColor.black.cgColor

    containerView.layer.addSublayer(shapeLayer)
}
```

Output looks like this:
{% include image.html
img="assets/2018-01-04/image3.png"
title=""
caption="" %}

In this we didn’t create the path ourself. Swift provides us with some initialisers of `UIBezierPath` which can be used to draw shapes for us. In this case we used `UIBezierPath(ovalIn: <CGRect>)`, what it does is it takes a rectangle as an input and draws the biggest oval possible inside it. The `height` and `width` of our `containerView` is same that is why it created a circle for us, if we increase the `height` or `width` then the resulting figure would be an oval.

# Drawing an Arc

```swift
private func drawArc() {

    let path = UIBezierPath(arcCenter: CGPoint(x: 100, y: 100), radius: 100, startAngle: 0, endAngle: .pi, clockwise: false)

    let shapeLayer = CAShapeLayer()
    shapeLayer.path = path.cgPath
    shapeLayer.fillColor = UIColor.orange.cgColor
    shapeLayer.lineWidth = 3
    shapeLayer.strokeColor = UIColor.black.cgColor

    containerView.layer.addSublayer(shapeLayer)
}
```

Output looks like this:
{% include image.html
img="assets/2018-01-04/image4.png"
title=""
caption="" %}

You will realise that we have used another initialiser for creating an arc using `UIBezierPath`, we can also create a circle using this method. Let’s breakdown the initialiser and understand what are the different parameters that it takes.

`UIBezierPath(arcCenter: <CGPoint>, radius: <CGFloat>, startAngle: <CGFloat>, endAngle: <CGFloat>, clockwise: <Bool>`

1. `arcCenter` -> This is the point from which the arc is drawn using it as a centre.

2. `radius` -> Radius of the arc

3. `startAngle` -> The angle from which arc starts

4. `endAngle` -> The angle where arc ends

5. `clockwise` -> Whether the arc is drawn clockwise or not

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

# Drawing a random shape

Let’s create a random shape using all the things that we have learnt so far.

```swift
private func drawRandomShape() {

        let path = UIBezierPath()
        path.move(to: CGPoint(x: 0, y: 200))

        path.addLine(to: CGPoint(x: 0, y: 125))
        path.addLine(to: CGPoint(x: 50, y: 125))
        path.addLine(to: CGPoint(x: 50, y: 75))
        path.addLine(to: CGPoint(x: 0, y: 75))
        path.addLine(to: CGPoint(x: 0, y: 0))

        path.addLine(to: CGPoint(x: 75, y: 0))
        path.addLine(to: CGPoint(x: 75, y: 50))
        path.addLine(to: CGPoint(x: 125, y: 50))
        path.addLine(to: CGPoint(x: 125, y: 0))
        path.addLine(to: CGPoint(x: 200, y: 0))

        path.addLine(to: CGPoint(x: 200, y: 75))
        path.addLine(to: CGPoint(x: 150, y: 75))
        path.addLine(to: CGPoint(x: 150, y: 125))
        path.addLine(to: CGPoint(x: 200, y: 125))
        path.addLine(to: CGPoint(x: 200, y: 200))

        path.addLine(to: CGPoint(x: 125, y: 200))
        path.addLine(to: CGPoint(x: 125, y: 150))
        path.addLine(to: CGPoint(x: 75, y: 150))
        path.addLine(to: CGPoint(x: 75, y: 200))
        path.addLine(to: CGPoint(x: 0, y: 200))

        let shapeLayer = CAShapeLayer()
        shapeLayer.path = path.cgPath
        shapeLayer.strokeColor = UIColor.black.cgColor
        shapeLayer.lineWidth = 2
        shapeLayer.fillColor = UIColor.green.cgColor

        containerView.layer.addSublayer(shapeLayer)
}
```

Output looks like this:
{% include image.html
img="assets/2018-01-04/image6.png"
title=""
caption="" %}

# Wrapping Up

This was all about getting started with `CAShapeLayer` in iOS. If you have any doubts then feel free to discuss in the comments.

Reference: [https://medium.com/flawless-app-stories/drawing-using-cashapelayer-in-ios-9a6c83de7eb2](https://medium.com/flawless-app-stories/drawing-using-cashapelayer-in-ios-9a6c83de7eb2)
