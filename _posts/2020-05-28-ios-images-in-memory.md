---
layout: post
title: "iOS images in memory"
author: "Linh Vo"
tags: "UIKit"
---

# Why memory and CPU matter?

{% include image.html
img="assets/2020-05-28/image1.jpeg" %}

- It is obvious that too much usage of `CPU` has negative impact on `battery life` and `responsiveness`.

- But, if you application consumes too much memory, that causes more `CPU utilization`, which have negative effects on `battery life` and `performance`.

# Basic Concepts

## Buffers

Buffer is contiguous region of memory. It is often viewed as sequence of elements of the same sizes, usually of the same internal construction.

{% include image.html
img="assets/2020-05-28/image2.jpeg" %}

## Image Buffers

One kind of the important buffers is the `Image Buffer`, which holds the `in-memory representation of some image`.

Each element in image buffers describes the color and the transparency of single pixel in our image. Constantly, the buffer size is proportional to image size. For example, in the image with sRBG format, each 32 bits describes the color and transparency of a single pixel. `The buffer size = 4 byte x image_width x image_height`

{% include image.html
img="assets/2020-05-28/image3.jpeg" %}

## Frame buffer

The frame buffer is what holds the actual rendered output of your application.

As your application updates its view hierarchy UIKit will render the application’s `window` and all of its `subviews` into the `frame buffer`. And that frame buffer provides per pixel color information that the `display hardware` will read in order to illuminate the pixels on the display.

[Related resource: The View Drawing Cycle](https://developer.apple.com/library/archive/documentation/WindowsViews/Conceptual/ViewPG_iPhoneOS/WindowsandViews/WindowsandViews.html#//apple_ref/doc/uid/TP40009503-CH2-SW10)

The pipeline is like this:

{% include image.html
img="assets/2020-05-28/image4.gif" %}

## Data Buffer

a data buffer is just a buffer that contains a sequence of bytes.

If we’ve downloaded images from the network or we’ve `loaded` them from disk. A data buffer that contains an image file, typically, begins with some metadata describing the size of the image that’s stored in that data buffer.

{% include image.html
img="assets/2020-05-28/image5.jpeg" %}

- Store contents of an `image file` in memory. The `size` is the same as that in the image file in the disk.

- Metadata describing dimensions of image

- Image itself `encoded` as JPEG, PNG, or other (usually compressed) form

- Bytes `do not` directly describe pixels

# The Pipeline in Action

{% include image.html
img="assets/2020-05-28/image6.jpeg" %}

# Consequences of Excessive Memory Usage

- Increased fragmentation.

  - `fragment`: The large allocation that is in your application’s address space could force other related content apart from content that it wants to reference.

- Poor locality of reference

- System starts `compressing` memory Process termination

  - Eventually, if your application starts accumulating a lot of memory usage the operating system will step in and start transparently `compressing` the content of physical memory. Now, the CPU needs to be involved in this operation so in addition to any CPU usage in your own application. You could be `increasing global CPU usage` that you have no control over. Eventually, your application could start consuming so much physical memory that the OS needs to start `terminating processes`. And it’ll start with background processes of low priority.

- You app may be terminated by the system

- Cause `CPU peak`, and then has negative impact on battery life and responsiveness.

# Decoding

Decoding is an operation that will convert the JPEG or PNG or other encoded image data into per pixel image information.

- CPU-intensive process

- The memory used for the image buffer is proportional to original image size, not view size.

- There are persistent large memory allocation in decoding.

- The image buffer is retained for repeat rendering by UIImage.

After decoding, UIImage will hang onto that image buffer, so that it only does that work once. Consequently, your application, for every image that gets decoded, could have a persistent and large memory allocation hanging out.

# The memory use of an image using sRGB space

Memory use is related to the dimensions of the images, not the file size.

Take the following image as an instance, its file size is 590KB, with dimension 2048 x 1536 pixel.

{% include image.html
img="assets/2020-05-28/image7.png" %}

## 4 byetes for each pixel in RGBA

By this [article](https://www.objc.io/issues/3-views/moving-pixels-onto-the-screen/), each pixel in sRGB image needs 32bit, 4 bytes, in memory when it’s decoded. Because in sRGB, there are Red, Green, Blue 3 channels and Alpha. The range of each channel value is from 0 to 255, which needs 8 bits to represent the value.

```md
1 A R G B A R G B A R G B  
2 | pixel 0 | pixel 1 | pixel 2  
3 0 233 2 100 4 155 255 7 8 9 10 11
```

## More memory usage when decoding a image

A image have load -> decode -> render these 3 phases.

{% include image.html
img="assets/2020-05-28/image8.png" %}

We only need 590KB to load a image, while we need
`2048 pixels x 1536 pixels x 4 bytes per pixel` = 10MB when decoding

# Image Rendering Pipeline

- `UIImage` is responsible for loading image content. Usually, it is bigger than the ImageView that is going to display it.

- `UIImageView` is responsible for displaying the image. The image which it is going to display will be shrink down to fit its size.

{% include image.html
img="assets/2020-05-28/image9.jpeg" %}

# Downsampling

So, we can use technique, called Downsampling, to `downsample` the images to `the size that they're going to be displayed`. Rather than having these large allocations hanging around, we’re reducing our memory usage.

{% include image.html
img="assets/2020-05-28/image10.jpeg" %}

```swift
// Downsampling large images for display at smaller size
func downsample(imageAt imageURL: URL, to pointSize: CGSize, scale: CGFloat) -> UIImage {
    // ShouldCache flag tells Core Graphics framework that we're just creating an object to
    // represent the infomation stored in the file at this URL
    let imageSourceOptions = [kCGImageSourceShouldCache: false] as CFDictionary
    let imageSource = CGImageSourceCreateWithURL(imageURL as CFURL, imageSourceOptions)!
    let maxDimensionInPixels = max(pointSize.width, pointSize.height) * scale
    let downsampleOptions =
        // Ask the Core Graphics to create the decoded image buffer for me when calling it to create the thumbnail
        [kCGImageSourceCreateThumbnailFromImageAlways: true,
        kCGImageSourceShouldCacheImmediately: true,
        // Should include kCGImageSourceCreateThumbnailWithTransform: true in the options dictionary. Otherwise, the image result will appear rotated when an image is taken from camera in the portrait orientation.
        kCGImageSourceCreateThumbnailWithTransform: true,
        kCGImageSourceThumbnailMaxPixelSize: maxDimensionInPixels] as CFDictionary
    let downsampledImage =
        CGImageSourceCreateThumbnailAtIndex(imageSource, 0, downsampleOptions)!
    return UIImage(cgImage: downsampledImage)
}
```

# Decoding in ScrollViews

## The cause of the ScrollView hitch

{% include image.html
img="assets/2020-05-28/image11.jpeg" %}

When beginning scrolling, we’re about to display another row of images. And we’re about to ask Core Graphics to `decode` those images before we hand the cells back to UICollectionView.
And that could take `a lot of CPU time`. So much so, that we `don't` get around to `re-rendering the frame buffer`. But the `display hardware` is operating on a `fixed interval`. So, from the user’s perspective the application has just `stuttered`. Now, we’re done decoding these images, we’re able to provide those cells back to UICollectionView. And animation continues on, as before. Just saw a `visual hitch`

# Two techniques to smooth out CPU usage

1. prefetching

2. performing work in the background

## Thread Explosion

It is caused by

- More images to decode than available CPUs

- GCD continues creating threads as new work is enqueued

- Each thread gets less time to actually decode images

Now, to avoid deadlock when we dispatch asynchronously to a global queue, GCD is going to create new threads to capture the work we’re asking it to do. And then, the CPUs are going to spend a lot of time moving between those threads to try and make incremental progress on all of the work we asked the operating system to do for us. And switching between those threads, actually, has a pretty significant overhead.

we’re going to serialize some work.

So, rather than simply dispatching work to one of the global asynchronous queues, we’re going to create a serial queue. And inside of our implementation of the prefetch method we’re going to asynchronously dispatch to that queue.

{% include image.html
img="assets/2020-05-28/image12.jpeg" %}

## Image Sources

Images may come from

- Image assets in asset catalog

- Files in application/framework bundle

- Files in Documents and Caches directories

- Data downloaded from network

This session suggests us to use image assets for the following reasons:

- Optimized name- and trait-based lookup

  - It’s faster to look up an image asset in the asset catalog, than it is to search for files on disk that have a certain naming scheme.

- Smarter buffer caching

  - The asset catalog runtime has, also, got some really good smarts in it for managing buffer sizes.

- Per-device thinning

  - your application only downloads image resources that are relevant to the device that it’s going to run on and vector artwork. The

- Vector artwork

## Vector artwork

- Since iOS 11, image assets support “Preserve Vector Data”

- Avoids blurriness and aliasing when drawn larger or smaller than natural size

## Image Rendering Format

### SRGB

- 4 bytes per pixel

- full color images

- most common used

{% include image.html
img="assets/2020-05-28/image13.png" %}

### Wide format

- 8 bytes per pixel

- super accurate colors. Because they use 8bytes, 16 bits, for each channel. In the meantime, double the image size.

- Only useful with wide color display. We don’t want to use it when we don’t need to.

- Wide color capture cameras since iPhone 7

{% include image.html
img="assets/2020-05-28/image14.png" %}

### Luminance and alpha 8 format

This image only holds grayscale value. And the image size is smaller.

- 2 bytes per pixel

- Single-color iamges and alpha

- Most used in Metal shaders, not very common.

{% include image.html
img="assets/2020-05-28/image15.png" %}

### Alpha 8 Format

- 1 byte per pixel

- Userful for monochrome images because it uses less memory.

  - masks

  - Emoji-free text

- 75% smaller than SRGB

{% include image.html
img="assets/2020-05-28/image16.png" %}

We can also change `image.tintColor` in this image without changing its format.

# How do I pick the right format?

`UIGraphicsBeginImageContextWithOptions` always uses SRGB rendering-format, which use 4 bytes per pixel.

while `UIGraphicsImageRenderer`, which introduced in iOS 10 will automatically pick the best graphic format in iOS12. It means, you will save 75% of memory by replacing `UIGraphicsBeginImageContextWithOptions` with `UIGraphicsImageRenderer`

Do ✅

{% include image.html
img="assets/2020-05-28/image17.png" %}

# Use ImageIO to downsample images

`UIImage` is expensive for sizing and to resizing

Will decompress original image into memory
Internal coordinate space transforms are expensive
Don’t ❌

{% include image.html
img="assets/2020-05-28/image18.png" %}

Use ImageIO

- ImageIO can read image sizes and metadata information without dirtying memory.

- ImageIO can resize images at cost of resized image only.

Do ✅

{% include image.html
img="assets/2020-05-28/image19.png" %}

The bad is that you have to specify some options.

# Resize image effectively

This is a function of resizing image `without reading it into the memory`, using `ImageIO`.

```swift
func resize(url: NSURL, maxPixelSize: Int) -> CGImage? {
    let imgSource = CGImageSourceCreateWithURL(url, nil)
    guard let imageSource = imgSource else {
        return nil
    }

    var scaledImage: CGImage?
    let options: [NSString: Any] = [
            // The maximum width and height in pixels of a thumbnail.
            kCGImageSourceThumbnailMaxPixelSize: maxPixelSize,
            kCGImageSourceCreateThumbnailFromImageAlways: true,
            // Should include kCGImageSourceCreateThumbnailWithTransform: true in the options dictionary. Otherwise, the image result will appear rotated when an image is taken from camera in the portrait orientation.
            kCGImageSourceCreateThumbnailWithTransform: true
    ]
    scaledImage = CGImageSourceCreateThumbnailAtIndex(imageSource, 0, options as CFDictionary)

    return scaledImage
}


let filePath = Bundle.main.path(forResource:"large_leaves_70mp", ofType: "jpg")

let url = NSURL(fileURLWithPath: filePath ?? "")

let image = resize(url: url, maxPixelSize: 600)
```

# Optimizing when in the background

## foreground/background

The strategy is simple, when `UIApplicationDidEnterBackground`, we unload images; and when `UIApplicationWillEnterForeground`, load images.

## on-screen/off-screen

unload large resource when off-screen, `viewDidDisappear`; and load large resource when on-screen, `viewWillAppear`.

## Debug

Use memory graphs to further understand and reduce memory footprint

Original post: [https://suelan.github.io/2020/05/03/iOS-images-in-memory/](https://suelan.github.io/2020/05/03/iOS-images-in-memory/)
