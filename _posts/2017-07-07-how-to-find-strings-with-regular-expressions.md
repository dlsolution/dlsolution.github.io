---
layout: post
title: "How To Find Strings with Regular Expressions in Swift"
author: "Linh Vo"
tags: "UIKit"
---

```md
"I made this wonderful pic last #christmas... #instagram #nofilter #snow #fun"
```

# How are we going to get those hashtags out?

You do the same thing in code by using a regular expression or “regex”. You use regular expressions to find occurrences of text in a string using a search pattern.

This is our search pattern: `#[a-z0-9]+`

We’re looking for strings that start with `#` followed by one or more alphanumeric characters. The stuff between the `[` and `]` form a range. The `+` means “one or more characters in this range”.

In other words: look for text that starts with `#`, followed by one or more alphanumeric characters.

In Swift we need an instance of `NSRegularExpression` to search the string for hashtags, like this:

```swift
extension String {

    func hashtags() -> [String] {
        if let regex = try? NSRegularExpression(pattern: "#[a-z0-9]+", options: .caseInsensitive) {

            let string = self as NSString

            return regex
                .matches(in: self,
                         options: [],
                         range: NSRange(location: 0, length: string.length))
                .map {
                    return string
                        .substring(with: $0.range)
                        .replacingOccurrences(of: "#", with: "")
                        .lowercased()
                }
        }
        return []
    }

}
```

> `.caseInsensitive` - It will find hashtags with both uppercase and lowercase characters

Here’s the result:

```swift
let text = "A hashtag is a metadata tag that is prefaced by the hash symbol, #.... #instagram #facebook #twitter #telegram #apple"
let hashtags = text.hashtags()
print(hashtags)
// Output: ["instagram", "facebook", "twitter", "telegram", "apple"]
```

> Why use `NSString` and `NSRange`? Because we’re also using `NSRange` and `NSRegularExpression`.
