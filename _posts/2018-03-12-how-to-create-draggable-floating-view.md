---
layout: post
title: "How to Create a Draggable Floating View in Swift"
author: "Linh Vo"
tags: "UIKit"
---

```swift
import UIKit

class ViewController: UIViewController {

    @IBOutlet weak var draggableView: UIView!

    override func viewDidLoad() {
        super.viewDidLoad()
        self.draggableView.addGestureRecognizer(UIPanGestureRecognizer(target: self, action: #selector(handler)))
    }

    @objc func handler(gesture: UIPanGestureRecognizer){
        let location = gesture.location(in: self.view)
        let draggedView = gesture.view
        draggedView?.center = location

        if gesture.state == .ended {
            if self.draggableView.frame.midX >= self.view.layer.frame.width / 2 {
                UIView.animate(withDuration: 0.5, delay: 0, usingSpringWithDamping: 1, initialSpringVelocity: 1, options: .curveEaseIn, animations: {
                    self.draggableView.center.x = self.view.layer.frame.width - 40
                }, completion: nil)
            } else {
                UIView.animate(withDuration: 0.5, delay: 0, usingSpringWithDamping: 1, initialSpringVelocity: 1, options: .curveEaseIn, animations: {
                    self.draggableView.center.x = 40
                }, completion: nil)
            }
        }
    }
}
```

Here’s the logic :

- We check whether the user has stopped dragging, then we do a comparison between the current view’s midX position and its superview’s width divided by half. We do the comparison to determine whether the current view is closer to its superview’s left or right side.

- If the view’s midX is greater than or equal to half the width of its superview (closer to the right side of the screen), we then update its X position as its superview width minus 40. This means that our view’s final location will be located at 40pt from its superview’s trailing.

- If it is closer to the left screen, we then update its X position to 40. This means that our view’s final location will be located 40pt away from its superview trailing.

{% include image.html
img="assets/2018-03-12/image.gif"
title=""
caption=""
max-width="300px" %}
