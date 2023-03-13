---
layout: post
title: "Xcode deinit breakpoint for UIViewController"
author: "Linh Vo"
tags: "UIKit"
---

## Xcode deinit breakpoint for UIViewController

This breakpoint provides an easy way to track view controller deinitialization (deallocation). This can help finding memory leaks caused by retain cycles preventing view controllers from being deinitialized when dismissed or popped.

From [Cédric Luthi's tweet](https://twitter.com/0xced/status/900692839557992449) in 2017:

> Useful Xcode breakpoint. When you dismiss a controller and you don’t hear the pop sound (or see the log), you probably have a retain cycle.

![](https://pbs.twimg.com/media/DH_nWyhXUAIqByQ?format=png&name=medium)

## Breakpoint Configuration

1. In Xcode, go to the Breakpoint navigator (Cmd+8)
2. At the bottom-left on the screen, tap '+', then select "Symbolic Breakpoint..." from the menu
3. Fill the form:
  - Name: `UIViewController dealloc`
  - Symbol: `-[UIViewController dealloc]`
  - Module: `UIKitCore` (or leave empty)  
  - Condition: `!(BOOL)([[$arg1 description] containsString:@"Input"])` to exclude `UIInputViewController`, `UICompatibilityInputViewController`, etc.
  - Ignore: leave at zero
  - Action: Select "Log Message"
      - Enter: `--- dealloc @(id)[$arg1 description]@ @(id)[$arg1 title]@` (or customize as needed)
      - Select "Log message to console"
  - Next to "Action: Log Message", tap on '+' to add a new Action
  - Select: "Sound" and choose any sound from the list (Submarine, Pop...)
  - Check "Automatically continue after evaluating actions" so Xcode does not stop at the breakpoint