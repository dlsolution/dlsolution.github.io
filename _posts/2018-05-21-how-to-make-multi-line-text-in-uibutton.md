---
layout: post
title: "How to make multi-line text in UIButton"
author: "Linh Vo"
tags: "UIKit"
---

The default appearance of `UIButton` is a single line text, but it also supports a multi-line text with some minor tweak.

# Programmatically

To make a multi-line text in UIButton, you insert a new line character (`\n`) wherever you want in button title and set `lineBreakMode` to `byWordWrapping`.

```swift
let buttonCenter = UIButton(type: .system)
buttonCenter.setTitle("Title\nSubtitle", for: .normal)
buttonCenter.titleLabel?.lineBreakMode = .byWordWrapping
buttonCenter.titleLabel?.textAlignment = .center
```

# Storyboard / XIB

You can achieve the same effect on the storyboard. Under **Attributes inspector**, set title with a new line character, and **Line Break** to **Word Wrap**.

> You type a new line character in the storyboard with `Option + Return`

But you can't adjust text alignment with **Plain** title. To change text alignment:

1. Change the title to **Attributed**.
2. Select all the text in the text field.
3. Then select text alignment that you want.

{% include image.html
img="assets/2018-05-21/image.png"
%}
