---
layout: post
title: "Adjust Views To Fit the Keyboard"
author: "Linh Vo"
tags: "UIKit"
---

{% include image.html
img="assets/2018-04-10/image1.jpeg"
%}

With a scrollable view — whether that’s a `UITableView`, `UIScrollView`, or even a `UIView` — when the keyboard appears, it takes up space and typically means you have to adjust your view to be dynamically responsive depending on the keyboard’s visibility.

The most common way of handling this is using the `NotificationCenter` to respond to the following events:

```swift
UIResponder.keyboardWillHideNotification
UIResponder.keyboardWillChangeFrameNotification
```

This works, but you have to add code in every class that uses a keyboard to handle it. One option proposed by [Hacking with Swift](https://www.hackingwithswift.com/example-code/uikit/how-to-adjust-a-uiscrollview-to-fit-the-keyboard) is:

```swift
let notificationCenter = NotificationCenter.default
notificationCenter.addObserver(self, selector: #selector(adjustForKeyboard), name: UIResponder.keyboardWillHideNotification, object: nil)
notificationCenter.addObserver(self, selector: #selector(adjustForKeyboard), name: UIResponder.keyboardWillChangeFrameNotification, object: nil)
```

```swift
@objc func adjustForKeyboard(notification: Notification) {
    guard let keyboardValue = notification.userInfo?[UIResponder.keyboardFrameEndUserInfoKey] as? NSValue else { return }

    let keyboardScreenEndFrame = keyboardValue.cgRectValue
    let keyboardViewEndFrame = view.convert(keyboardScreenEndFrame, from: view.window)

    if notification.name == UIResponder.keyboardWillHideNotification {
        yourTextView.contentInset = .zero
    } else {
        yourTextView.contentInset = UIEdgeInsets(top: 0, left: 0, bottom: keyboardViewEndFrame.height - view.safeAreaInsets.bottom, right: 0)
    }

    yourTextView.scrollIndicatorInsets = yourTextView.contentInset

    let selectedRange = yourTextView.selectedRange
    yourTextView.scrollRangeToVisible(selectedRange)
}
```

This works brilliantly, and there is nothing wrong with it. However, let’s see if there is another way of handling it.

My preference is to have my `UIView` and `UIViewController`’s logic as specific as possible to that view and to abstract common logic out where possible. I find that it improves the readability of the code and increases the speed at which you can understand an unfamiliar class.

# Introducing NSLayoutConstraint Subclasses

This is where subclassing an `NSLayoutConstraint` becomes useful. Placing all of the logic for expanding and contracting based on whether the keyboard is showing or hiding allows us to simply set a constraint and not have to worry about handling the logic for it.

If you use `AutoLayout`, then you can simply set the class to be the subclassed variant in your project, as shown below:

{% include image.html
img="assets/2018-04-10/image2.png"
%}

Alternatively, you can create the constraint programmatically, although my preference is to use `AutoLayout` where appropriate.

The only downside to this approach is that if someone were eventually to replace this constraint with a non-subclassed constraint, it would break the view. With that said, something like that should be found by the developer or, at worst, caught with a good testing practice. I believe the upside to this approach far outweighs the downsides.

Below is my variant of the subclass, which is a modification of one created by [Meng To](https://raw.githubusercontent.com/MengTo/Spring/master/Spring/KeyboardLayoutConstraint.swift) that you are more than welcome to use in your projects:

```swift
//
//  KeyboardLayoutConstraint.swift
//  TemplateProject
//
//  Created by Adam Wareing on 12/08/19.
//  Licenced under MIT.
//
//  Based off: https://raw.githubusercontent.com/MengTo/Spring/master/Spring/KeyboardLayoutConstraint.swift
import UIKit

public class KeyboardLayoutConstraint: NSLayoutConstraint {

    /// This offset is added on when the keyboard is collapsed.
    var keyboardDownOffset: CGFloat = 0

    /// Default's to the offset of the constraint so it can be restored when the keyboard hides
    private var offset: CGFloat = 0

    /// The current height of the keyboard. 0 if the keyboard isn't shown
    private var keyboardVisibleHeight: CGFloat = 0

    @available(tvOS, unavailable)
    override public func awakeFromNib() {
        super.awakeFromNib()
        offset = constant

        NotificationCenter.default.addObserver(self, selector: #selector(KeyboardLayoutConstraint.keyboardWillShowNotification(_:)), name: UIResponder.keyboardWillShowNotification, object: nil)
        NotificationCenter.default.addObserver(self, selector: #selector(KeyboardLayoutConstraint.keyboardWillHideNotification(_:)), name: UIResponder.keyboardWillHideNotification, object: nil)
    }

    deinit {
        NotificationCenter.default.removeObserver(self)
    }

    // MARK: Notification

    @objc
    func keyboardWillShowNotification(_ notification: Notification) {
        if let userInfo = notification.userInfo {
            if let frameValue = userInfo[UIResponder.keyboardFrameEndUserInfoKey] as? NSValue {
                let frame = frameValue.cgRectValue
                keyboardVisibleHeight = frame.size.height
            }

            self.updateConstant()
            switch (userInfo[UIResponder.keyboardAnimationDurationUserInfoKey] as? NSNumber, userInfo[UIResponder.keyboardAnimationCurveUserInfoKey] as? NSNumber) {
            case let (.some(duration), .some(curve)):

                let options = UIView.AnimationOptions(rawValue: curve.uintValue)
                UIView.animate(withDuration: TimeInterval(duration.doubleValue), delay: 0, options: options, animations: {
                    UIApplication.shared.keyWindow?.layoutIfNeeded()
                    return
                })
            default:
                break
            }
        }
    }

    @objc
    func keyboardWillHideNotification(_ notification: NSNotification) {
        keyboardVisibleHeight = 0
        self.updateConstant()

        if let userInfo = notification.userInfo {
            switch (userInfo[UIResponder.keyboardAnimationDurationUserInfoKey] as? NSNumber,
                    userInfo[UIResponder.keyboardAnimationCurveUserInfoKey] as? NSNumber) {
            case let (.some(duration), .some(curve)):
                let options = UIView.AnimationOptions(rawValue: curve.uintValue)
                UIView.animate(withDuration: TimeInterval(duration.doubleValue), delay: 0, options: options, animations: {
                    UIApplication.shared.keyWindow?.layoutIfNeeded()
                    return
                })
            default:
                break
            }
        }
    }

    func updateConstant() {
        if keyboardVisibleHeight == 0 {
            self.constant = offset + keyboardDownOffset
            return
        }

        self.constant = offset + keyboardVisibleHeight
    }
}
```

Reference: [Medium](https://betterprogramming.pub/how-to-adjust-views-to-fit-the-keyboard-in-ios-63bcf42163b8)
