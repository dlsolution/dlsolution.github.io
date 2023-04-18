---
layout: post
title: "Distribute your code as XCFramework"
author: "Linh Vo"
tags: "UIKit"
---

## Set up the project

To set up your project for creating an XCFramework, ensure your Xcode project has a scheme that builds only the framework target and its dependencies.

Configure these build settings on your target:

- Set the [Build Libraries for Distribution](https://developer.apple.com/documentation/xcode/build-settings-reference#Build-Libraries-for-Distribution) build setting to Yes. For Swift, this enables support for library evolution and generation of a module interface file.

- Set the [Skip Install](https://developer.apple.com/documentation/xcode/build-settings-reference#Skip-Install) build setting to No. If enabled, the built products aren‘t included in the archives.

- Leave the [Architectures](https://developer.apple.com/documentation/xcode/build-settings-reference#Architectures) build setting unset. The predefined value configures the target to build a universal binary for all the possible architectures the target platform uses.

For more information, see [Customizing the build schemes for a project](https://developer.apple.com/documentation/xcode/customizing-the-build-schemes-for-a-project) and [Configuring the build settings of a target](https://developer.apple.com/documentation/xcode/configuring-the-build-settings-of-a-target).

## Create archives for frameworks or libraries and Generate the XCFramework bundle

Create an archive of your framework or library for each platform you wish to support by running xcodebuild in Terminal using the archive build action.

The following command archives a framework for the iOS platform:

```bash
SCRIPT_DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" >/dev/null && pwd )"

cd "$SCRIPT_DIR"/../

rm -rf MyFramework.xcframework
rm -rf "${SCRIPT_DIR}"/../archives

xcodebuild archive \
    -project MyFramework.xcodeproj \
    -scheme MyFramework \
    -destination "generic/platform=iOS" \
    -archivePath "$PWD/archives/MyFramework" \
    SKIP_INSTALL=NO \
    BUILD_LIBRARY_FOR_DISTRIBUTION=YES | xcbeautify

xcodebuild archive \
    -project MyFramework.xcodeproj \
    -scheme MyFramework \
    -destination "generic/platform=iOS Simulator" \
    -archivePath "$PWD/archives/MyFramework-Simulator" \
    SKIP_INSTALL=NO \
    BUILD_LIBRARY_FOR_DISTRIBUTION=YES | xcbeautify

xcodebuild -create-xcframework \
    -framework "Archives/MyFramework.xcarchive/Products/Library/Frameworks/MyFramework.framework" \
    -debug-symbols "$PWD/archives/MyFramework.xcarchive/dSYMs/MyFramework.framework.dSYM" \
    -framework "Archives/MyFramework-Simulator.xcarchive/Products/Library/Frameworks/MyFramework.framework" \
    -debug-symbols "$PWD/archives/MyFramework-Simulator.xcarchive/dSYMs/MyFramework.framework.dSYM" \
    -output "${SCRIPT_DIR}"/../MyFramework.xcframework

rm -rf "${SCRIPT_DIR}"/../archives
```