---
layout: post
title: "Autoresizing UITextView Like in Messaging Apps"
author: "Linh Vo"
tags: "UIKit"
---

Today we are going to learn how we can resize our UITextView so it fits its content and after it takes a big chunk of text it cannot longer become bigger and it starts scrolling.

{% include image.html
img="assets/2017-05-11/image1.gif"
title=""
caption="" %}

The first thing we need to do is to give our textView a height constraint, you can do it in storyboard but also in code writing the next line:

```swift
textView.heightAnchor.constraint(equalToConstant: 52.0).isActive = true
```

After that we need to confirm to the UITextFieldDelegate by writing the following line:

```swift
textView.delegate = self
```

Now we will create an extension for this delegate. And the function we are going to use is `textViewDidChange(_ textView: UITextView)`

```swift
extension ViewController: UITextViewDelegate {

    func textViewDidChange(_ textView: UITextView) {

        let size = CGSize(width: textView.frame.size.width, height: .infinity)
        let estimatedSize = textView.sizeThatFits(size)

        guard textView.contentSize.height < 100.0 else {
            textView.isScrollEnabled = true
            return
        }

        textView.isScrollEnabled = false
        textView.constraints.forEach { (constraint) in
            if constraint.firstAttribute == .height {
                constraint.constant = estimatedSize.height
            }
        }
    }
}
```
