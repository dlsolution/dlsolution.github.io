---
layout: post
title: "How to create a page scroll using UIPageViewController"
author: "Linh Vo"
tags: "UIKit"
---

Today we are going to learn how to create a page scroll using UIPageViewController.

```swift
import UIKit

class PageViewController: UIViewController, UIPageViewControllerDataSource, UIPageViewControllerDelegate {

    lazy var pageController: UIPageViewController = {
        let pageController = UIPageViewController(transitionStyle: .scroll, navigationOrientation: .horizontal, options: nil)
        pageController.dataSource = self
        pageController.delegate = self
        pageController.view.translatesAutoresizingMaskIntoConstraints = false
        return pageController
    }()

    var controllers = [UIViewController]()

    override func viewDidLoad() {
        super.viewDidLoad()

        addChild(pageController)
        view.addSubview(pageController.view)
        NSLayoutConstraint.activate([
            pageController.view.topAnchor.constraint(equalTo: view.topAnchor),
            pageController.view.bottomAnchor.constraint(equalTo: view.bottomAnchor),
            pageController.view.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            pageController.view.trailingAnchor.constraint(equalTo: view.trailingAnchor)
        ])

        for _ in 1 ... 5 {
            let vc = UIViewController()
            vc.view.backgroundColor = randomColor()
            controllers.append(vc)
        }

        pageController.setViewControllers([controllers[0]], direction: .forward, animated: false)

        let scrollView = pageController.view.subviews.filter { $0 is UIScrollView }.first as! UIScrollView
        scrollView.delegate = self
        //Fix error when you use 2 fingers in order to continue scrolling between pages. After you pass second page, this startOffset should be set to 0 but it's not the case
        scrollView.panGestureRecognizer.maximumNumberOfTouches = 1
    }

    func pageViewController(_ pageViewController: UIPageViewController, viewControllerBefore viewController: UIViewController) -> UIViewController? {
        if let index = controllers.firstIndex(of: viewController), index > 0 {
            return controllers[index - 1]
        }
        return nil
    }

    func pageViewController(_ pageViewController: UIPageViewController, viewControllerAfter viewController: UIViewController) -> UIViewController? {
        if let index = controllers.firstIndex(of: viewController), index < controllers.count  - 1 {
            return controllers[index + 1]
        }
        return nil
    }

    func randomCGFloat() -> CGFloat {
        return CGFloat(arc4random()) / CGFloat(UInt32.max)
    }

    func randomColor() -> UIColor {
        return UIColor(red: randomCGFloat(), green: randomCGFloat(), blue: randomCGFloat(), alpha: 1)
    }
}

extension PageViewController: UIScrollViewDelegate {
    func scrollViewDidScroll(_ scrollView: UIScrollView) {

        let screenWidth = view.frame.size.width
        let xOffset = scrollView.contentOffset.x

        var direction = 0 //scroll stopped

        if screenWidth < xOffset {
            direction = 1 //going right
        } else if screenWidth > xOffset {
            direction = -1 //going left
        }

        let positionFromStartOfCurrentPage = abs(xOffset - screenWidth)
        let percentComplete = positionFromStartOfCurrentPage / screenWidth

        //you can decide what to do with scroll
    }
}
```