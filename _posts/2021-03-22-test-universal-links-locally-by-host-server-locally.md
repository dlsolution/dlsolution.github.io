---
layout: post
title: "Test Universal Links Locally by host server locally"
author: "Linh Vo"
tags: "UIKit"
---

### Definition

Opening an app from an URL is such a powerful iOS feature. Its drives users to your app, and can create shortcuts to specific features. This week, we’ll dive into deep linking on iOS and how to create an URL scheme for your app.

When we talk about deep linking for mobile app, it means creating a specific URL to open a mobile application. It separate into two formats:

- A custom URL scheme that your app registers: `scheme://videos`
- A universal link that opens your app from a registered domain: `mydomain.com/videos`

> Neither of these options will redirect you to the App Store if the app is not installed. Once the App is installed, iOS will make the connection between these URLs and the app based on the app’s metadata, if the app is not installed, the metadata is here and thus the connection is never created. We will have to work that out ourselves later.

Today, we’ll focus on Universal links.

Universal links allow your users to intelligently follow links to content in your app or your website based on the application and website association to provide a seamless experience to the end-user.

The association between the mobile application and the domain must be valid, so it needs two things.

- An app to have the capability to handle the domain association
- A server hosting the website allows your app to do so via a configuration file.

### App Capability

Visit [developer.apple.com](developer.apple.com)

- Approach 1: In Certificate, Identifiers & Profiles -> Look for you application identifier -> Click on the identifier -> Under Capabilities -> Check the checkbox of Associated Domains

- Approach 2: Select Automatically manage signing. Add the Associated Domain Capabilities into your application.

### HOST SERVER LOCALLY

Step 1: Create a configuration file i.e., `apple-app-site-association`

The following JSON code represents the contents of a simple association file.

```swift
{
  "applinks": {
      "details": [
      {
        "appID": "XYZ12389S.com.takemyexample.com",
        "paths": ["*"]
      }
    ]
  }
}
```

- The key `applinks` indicate we are about to declare universal links configuration.

- An array of `details` objects that define the apps and the universal links they handle for the domain.

- The `appIDs` key specifies the application identifiers for the apps that are available for use on this website.

- `IMPORTANT NOTE:` In the paths key, only add paths and `should not prefix it with the domain name` like testmyexample.com/* or testmyexample.com/assistant.

The following example is the right way of defining the paths,

“paths”: [
“/assistant/link/*”,
“/assistant/link”,
“/stella/*”
]

In case, you want only one path to exclude and all should be supported then just use NOT before the specific path.

“paths”: [
“*”,
“NOT /assistant/link/home”
]

NOTE: Replace `appID` value with your `<Application Identifier Prefix>.<Bundle Identifier>`

Step 2: Open terminal -> Go to some dedicated folder on your machine and run below command to install yarn.

```swift
npm install — global yarn
```

Step 3: Generate a project with

```swift
yarn init
```

Step 4: Add express: It’s an fast, minimalist web framework

```swift
yarn add express
```

It installs the express framework in the current directory to create web application

Step 5: Copy-paste the `apple-app-site-association` file in this project folder.

Step 6: Create the `index.js` file

```swift
var express = require('express');
var server = express();

// This will be call by APPLE TO VERIFY THE APP-SITE-ASSOCIATION 
// Make the 'apple-app-site-association' accessable to apple to verify the association
server.get('/.well-known/apple-app-site-association', function(request, response) {
  response.sendFile(__dirname +  '/apple-app-site-association');
});

// HOME PAGE
server.get('/home', function(request, response) {
  response.sendFile(__dirname +  '/home.html');
});

// ABOUT PAGE
server.get('/about', function(request, response) {
  response.sendFile(__dirname +  '/about.html');
});


server.listen(80);
```

Step 6: Create `home.html`

```swift
<!DOCTYPE html>
  <html>
    <head>
      <title> Univeral linking support </title>
    </head>
  <body>
    <h1>This is a demo for Universal Link feature with apple-app-site-association</h1>
  </body>
</html>
```

Step 7: Create `about.html`

```swift
<!DOCTYPE html>
  <html>
    <body>
      <h2>Universal Link About Page</h2>
      <p>This The html page to demonstrate that it will launch the about html page. If you have universal link support application. It simply opens the application</p>
      <img src="https://www.w3schools.com/html/img_chania.jpg" alt="Flowers in Chania" width="460" height="345">
    </body>
</html>
```

Step 8: Spin our server

```swift
node index.js
```

The local web app is running and accessible over with the localhost. We have to make it accessible publically to test the Universal Link.

So, we started to setup web app accessaible publically from our local machine.

Step 9: Install `ngrok` on local machine.

Ngrok exposed local servers behind NATs and firewalls to the public internet over secure tunnels.

Step 10: Open Terminal -> Go to the ngrok executable binary path -> run

```swift
ngrok http 80
```

You should see something similar to the following console UI in your terminal.

```swift
ngrok
    Session Status                online
    Account                       Linh Vo Duy (Plan: Free)
    Version                       3.1.1
    Region                        Asia Pacific (ap)
    Latency                       71ms
    Web Interface                 http://127.0.0.1:4040
    Forwarding                    https://99a6-113-160-235-251.ap.ngrok.io -> http://80clear:80
    
    Connections                   ttl     opn     rt1     rt5     p50     p90
                                  0       0       0.00    0.00    0.00    0.00
```

Now the local machine is accessible over http/https globally. [https://99a6-113-160-235-251.ap.ngrok.io](https://99a6-113-160-235-251.ap.ngrok.io) yours will be different.

Step 11: Let’s go back to our application which we have created and associated with our site.

Now just go back and add the ngrok domain name with prefix applinks:

```swift
applinks:99a6-113-160-235-251.ap.ngrok.io
```

### TEST THE UNIVERSAL LINKS

- Run the application on the simulator once to make sure the application is installed and running fine.

- Keep the same simulator launched and running.

Go to your simulator browser and open type the ngrok domain address i.e., [https://99a6-113-160-235-251.ap.ngrok.io](https://99a6-113-160-235-251.ap.ngrok.io) in my case.

**************************** OR ****************************

Run the below command in terminal,

```swift
xcrun simctl openurl booted https://99a6-113-160-235-251.ap.ngrok.io
```

It should open the application directly, if application is not installed then it simply redirects to the browser.

To handle different URL path implement the method in SceneDelegate.swift file

```swift
func scene(_ scene: UIScene, continue userActivity: NSUserActivity)

func scene(_ scene: UIScene, didFailToContinueUserActivityWithType userActivityType: String, error: Error)

func scene(_ scene: UIScene, willContinueUserActivityWithType userActivityType: String)
```

Happy Coding!!! 