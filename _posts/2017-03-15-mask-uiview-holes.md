---
layout: post
title: "Using CAShapeLayer and UIBezierPath to mask UIView holes"
author: "Linh Vo"
tags: "UIKit"
---

The trick is pretty straightforward — basically we need to get the path from our `overlayView` using `UIBezierPath(rect: overlayView.bounds)`, then set `fillRule` from our `CAShapeLayer` to `.evenOdd` and then append the path of the view we want to crop. We could either do point/point…line/line using `path.append` or we can simply add a subvview to the `overlayView` with the size we want to crop and then append its bounds to the path.

{% include image.html
            img="assets/2017-03-15/image1.PNG"
            title=""
            caption="" %}

```swift
import UIKit

class HoleInViewController: UIViewController {

    @IBOutlet weak var overlayView: UIView!

    let backgroundView: UIView = {
        let v = UIView()
        v.translatesAutoresizingMaskIntoConstraints = false
        v.backgroundColor = UIColor.black.withAlphaComponent(0.7)
        return v
    }()

    override func viewDidLoad() {
        super.viewDidLoad()

        view.addSubview(backgroundView)
        NSLayoutConstraint.activate([
            backgroundView.topAnchor.constraint(equalTo: view.topAnchor),
            backgroundView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            backgroundView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
            backgroundView.bottomAnchor.constraint(equalTo: view.bottomAnchor),
        ])

        overlayView.layer.cornerRadius = overlayView.bounds.width / 2
    }

    override func viewDidLayoutSubviews() {
        super.viewDidLayoutSubviews()

        // Create the initial layer from the view bounds.
        let maskLayer = CAShapeLayer()
        maskLayer.frame = backgroundView.bounds
        maskLayer.fillColor = UIColor.green.cgColor
        maskLayer.fillRule = .evenOdd

        // Create the path.
        let path = UIBezierPath(rect: backgroundView.bounds)

        // Append the overlay image to the path so that it is subtracted.
        path.append(UIBezierPath(roundedRect: overlayView.frame, cornerRadius: overlayView.bounds.width / 2))
        maskLayer.path = path.cgPath

        // Set the mask of the view.
        backgroundView.layer.mask = maskLayer
    }
}
```
