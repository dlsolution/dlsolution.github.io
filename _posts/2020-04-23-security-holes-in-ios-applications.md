---
layout: post
title: "Security holes I see in too many iOS applications"
author: "Linh Vo"
tags: "UIKit"
---

My work consists of many things. While working on a project from scratch is great, most of the time I am finishing/fixing work for someone else, consulting on possible next steps, or reviewing the code for performance and security. I want to share the worst developer-inflicted security holes I have seen in the last couple of years with you. And, of course, while we are on it, show how I fixed and resolved them.

# Laziness wins: using UserDefaults for everything

In pretty much all of the projects I open, at least 95% of the user data is stored in UserDefaults. When using UserDefaults in Android (called SharedPreferences), developers quickly learn how to go into the actual plain text files and read and change the preferences. As an iPhone is a little more abstracted, iOS developers feel safer using the UserDefaults. But that is a fallacy. UserDefaults store user information in plain text and can be both read and manipulated. Things I have seen developers store in plain text UserDefaults include:

- Login Information (including passwords). I think this is self-explanatory!

- API access (authentication) tokens. This means someone who reads this token from a phone can talk to the API (could be yours) from anywhere and be authenticated. How about writing a quick Python Program that hits your API with 150 requests per minute, doing something the user in your app is only allowed to do once per hour? Or storing Stripe API access keys?

- Payment details. I think this is again self-explanatory. Instead of never even persisting typed in credit-card details, I have seen credit-card numbers incl. CVC and date stored in plain text. Happy shopping spree!

So, how do we fix it? The answer is simple: first, think about whether you even need to store specific information. If you never store passwords or payment data, you can never lose passwords and payment data and avoid getting yourself onto the cover of the New York Times. If you really need to store some crucial information like API access tokens, you have two options.

1. Generically encrypt all keys and store them in UserDefaults

2. Use the Apple Keychain (possibly with iCloud synchronization)

I don’t want to go into too much detail here, but I can only recommend checking out these libraries on how to use the keychain in 3 lines of code: [keychain-swift](https://github.com/evgenyneu/keychain-swift) or [KeychainAccess](https://github.com/kishikawakatsumi/KeychainAccess)

> And to have this written down somewhere: the argument **“but but but the keychain is much slower than the UserDefaults”** is not really an argument but an excuse for laziness (or a sacrifice of privacy towards performance).

# Bad practice: Data persistence across accounts

When checking through apps, a missing feature I see a lot is the ability to log out of account A and login to account B and have no traces of account A. When a user hits the logout button, it is not enough to delete the authentication key and set the flag isLoggedIn to false! If you are using classes that store data (that might be loaded on launch, it is important to flush or destruct all these classes. You use UserDefaults to set user preferences (like the background color)? You have two options here. Either you set all preferences (that are non-security crucial) to the specific user by using a linked userID key likeuserID + "backgroundColor". Or you have to remove all entries manually! To developers, the logout button has become a little too lush. Most developers assume that a person has a phone, and no one else is using it!

My best practice: logging out destroys all crucial information, and preferences become inaccessible to different accounts. In one case, I implemented a logout flush option. On pressing logout, the user was given a choice to “keep all user-related data and stored/cached files” or destroy everything. The destruction was essentially resetting the app to the download state. And just saying, some people really loved that feature!

# Working fast and insecure

The last bad practice I have seen plenty of times is bypassing security measures for a faster release. The product owner says: the update needs to be ready by the end of the week! So you better start cutting corners.

My favorite cut corner is disabling the **App Transport Security (ATS)**. With iOS9, Apple introduced further measures to force developers to secure their user's data better. With the ATS, Apple disabled any communication to non-secure web servers (HTTP). The good news is, if your API is still not secured with HTTPS, you can disable ATS and add your API as a “ignore, I assure it is secure (enough)” link. My appeal: **please don’t**. Phrasing for you for your product-owner: “I have an impediment, the webserver is non-secure, and Apple doesn’t allow me to use the API. The web guy needs to get this done asap!”.

The second cut corner is integrating shady **third-party libraries** with next to no stars or developers. Especially when they are aiding in encryption, security, networking, or authentication, be super careful! Using those libraries requires two things. First, quick code check for modules or functionalities that do dangerous stuff, and secondly (important) check the network traffic, especially the outgoing one.

Thanks for reading. Feel free to share your experiences and the worst security holes you have seen!

Original Post [https://medium.com/next-level-swift/three-types-of-security-holes-i-see-in-too-many-ios-applications-507812d95b07](https://medium.com/next-level-swift/three-types-of-security-holes-i-see-in-too-many-ios-applications-507812d95b07)
