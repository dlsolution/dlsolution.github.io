---
layout: post
title: "3 Approaches to Applying Blur Effects in iOS"
author: "Linh Vo"
tags: "UIKit"
---

It's a fairly common requirement to apply a blur effect to iOS content. This article introduces three different approaches by using three different iOS frameworks: `UIKit`, `CoreImage`, and `Metal`.

# 1. Using UIBlurEffect to Blur Image

`UIBlurEffect` is a neatly designed UI element in `UIKit`. By using it along with `UIVisualEffectView`, a visual effect view is added on top of the background content.

To add the blur effect, do the following:

```swift
let blurEffect = UIBlurEffect(style: .light)
let blurEffectView = UIVisualEffectView()
blurEffectView.frame = CGRect(x: 0, y: 0, width: imageView.frame.width, height: 400)
blurEffectView.center = imageView.center
self.imageView.addSubview(blurEffectView)
UIView.animate(withDuration: 5) {
    blurEffectView.effect = blurEffect
}
```

Here’s what is happening:

- There were a bunch of blur styles introduced in [UIBlurEffect.Style](https://developer.apple.com/documentation/uikit/uiblureffect/style). We are using `.light` for this example.

- The instance of `UIVisualEffectView` is initialized with the specific height and centered with the background image.

- We add the instance of `UIVisualEffectView` as a subview of the image view.

- We apply the blur effect with animation for a duration of five seconds.

{% include image.html
img="assets/2019-08-15/image1.png"
max-width="700px" %}

`UIVisualEffectView` contains another two private layers (`UIVisualEffectBackdropView` and `UIVisualEffectSubView`) that would be interesting to explore.

{% include image.html
img="assets/2019-08-15/image2.png"
max-width="700px" %}

**Pros**

- `UIBlurEffect` can apply the custom blurred frame on the background content.

- `UIBlurEffect` is at the UIKit level without the need to worry about the lower-level details.

**Cons**

- `UIBlurEffect` only has a bunch of built-in system filters and can't be customized.

- `UIBlurEffect` adds additional layers on top of the background content. It does not change the content itself.

- `UIBlurEffect` is a UIKit framework using CPU computation — no GPU acceleration.

# 2. Apply CIFilter to the Image

CIFilter is the image processer in the `Core Image` framework. It has dozens of built-in image filters and offers the ability to build your own custom filters. `Core Image` can use `OpenGL/OpenGL ES` for high-performance, GPU-based rendering for real-time performance. It can also use CPU-based rendering with Quartz 2D if it doesn't require real-time performance.

To apply the blur effect for this example, we use `CIFilter` with `CIGaussianBlur` type:

```swift
extension UIImage {
    func blurredImage(with context: CIContext, radius: CGFloat, atRect: CGRect) -> UIImage? {
        guard let ciImg = CIImage(image: self) else { return nil }

        let cropedCiImg = ciImg.cropped(to: atRect)
        let blur = CIFilter(name: "CIGaussianBlur")
        blur?.setValue(cropedCiImg, forKey: kCIInputImageKey)
        blur?.setValue(radius, forKey: kCIInputRadiusKey)

        if let ciImgWithBlurredRect = blur?.outputImage?.composited(over: ciImg),
           let outputImg = context.createCGImage(ciImgWithBlurredRect, from: ciImgWithBlurredRect.extent) {
            return UIImage(cgImage: outputImg)
        }
        return nil
    }
}
```

- To be easily reused across the app, we add the `blurredImage` function to the `UIImage` extension.

- `CIContext` is height-weight and expensive. It’s recommended to be initialized only once.

- `radius` is the custom blur effect and can be specified with the key `kCIInputRadiusKey`.

- `atRect` specifies the blurred effect’s location and size.

- We first convert the `UIImage` to `CIImage` to use the `CoreImage` framework.

- We crop the new `CIImage` with the custom frame.

- The gaussian blur filter is initialized and applied with the cropped image as `kCIInputImageKey`.

- `composited` combines the blurred image with the original image into one.

- The `CGImage` is created from `CIImage` and then converted to `UIImage` for UI rendering.

{% include image.html
img="assets/2019-08-15/image3.png"
max-width="700px" %}

**Pros**

- `CIFilter` has dozens of built-in image filters and offers the ability to build your own custom filters.

- `Core Image` can be high-performance with GPU-based rendering.

- `CIFilter` modifies the content itself and can be applied to real-time rendering.

- `CIFilter` has the ability to specify the blurred area of the content.

- `CIFilter` can be subclassed to provide custom and chained filters.

**Cons**

- `CIFilter` exposes the complexity of `Core Image`, which is hard to manage.

- `CIFilter` uses key-value coding. It’s a bit error-prone.

{% include image.html
img="assets/2019-08-15/image4.png"
max-width="700px" %}

# 3. Metal, the GPU Acceleration Way

`Metal` allows us to use GPU directly to perform computing and graphics operations. In doing so, we also free up the CPU so that it is available for other operations. GPU is optimized for highly parallel workflows. It can perform graphics tasks faster and more efficiently than the CPU. A few Apple frameworks, including `Core Image`, use `Metal` under the hood to delegate graphic workloads to the GPU, while `Core ML` uses `Metal` to perform its low-level operations on the GPU.

```swift
// Instance of MTKView
@IBOutlet weak var mtkView: MTKView!

// Metal resources
var device: MTLDevice!
var commandQueue: MTLCommandQueue!
var sourceTexture: MTLTexture!

// Core Image resources
var context: CIContext!
let filter = CIFilter(name: "CIGaussianBlur")!
let colorSpace = CGColorSpaceCreateDeviceRGB()
```

- To use the `Metal` framework, we first initialized the instance of `MTKView` that is for rendering content.

- The instance of `MTLDevice` represents the GPU to use.

- The `MTLCommandQueue` executes the render and computes commands on the GPU.

- The `MTLTexture` object holds a `Metal` texture that contains the image to be processed by the filter.

- `CIContext` is height-weight and expensive. It’s recommended to have one universal instance across the app.

- We apply the `CIGaussianBlur` filter to the image.

- We use `CGColorSpaceCreateDeviceRGB` for the output image color space.

```swift
override func viewDidLoad() {
    super.viewDidLoad()

    device = MTLCreateSystemDefaultDevice()
    commandQueue = device.makeCommandQueue()
    let textureLoader = MTKTextureLoader(device: device)
    sourceTexture = try! textureLoader.newTexture(cgImage: UIImage(named: "street.png")!.cgImage!)
    let view = self.mtkView!
    view.delegate = self
    view.device = device
    view.framebufferOnly = false

    context = CIContext(mtlDevice: device)
}
```

- We initialize the `device` and `commandQueue` at the `viewDidLoad()`.

- `MTKTextureLoader` creates the `Metal` texture from `UIImage`.

- We set up the `MetalKit` view with delegate and device.

- Creating `CIContext` by using the same `Metal` device as the view saves the performance costs of copying data to and from separate CPU or GPU memory buffers.

```swift
extension ViewController: MTKViewDelegate {
    func draw(in view: MTKView) {
        if let currentDrawable = view.currentDrawable,
           let commandBuffer = commandQueue.makeCommandBuffer() {

            let inputImage = CIImage(mtlTexture: sourceTexture)!.oriented(.down)
            filter.setValue(inputImage, forKey: kCIInputImageKey)
            filter.setValue(10.0, forKey: kCIInputRadiusKey)

            context.render(filter.outputImage!,
                to: currentDrawable.texture,
                commandBuffer: commandBuffer,
                bounds:  mtkView.bounds,
                colorSpace: colorSpace)

            commandBuffer.present(currentDrawable)
            commandBuffer.commit()
        }
    }
}
```

- We implement the `draw(in:)` method of the `MTKViewDelegate` to apply the filter to the image.

- `.oriented(.down)` plays an important role here to rotate the image to the right orientation.

- We apply the filter with a radius of 10, but surely it can be customized.

{% include image.html
img="assets/2019-08-15/image5.png"
max-width="700px" %}

**Pros**

- The `Metal` framework is high-performance because it uses GPU parallel computing.

- `Metal` is fully customizable and can apply different user-defined filters to the content.

- `Metal` Texture can be configured with real-time video flow.

- `MTKViewDelegate` performs the draw method up to 60 times per second. We can use this opportunity to vary filter parameters over time to create smooth animation and real-time filters.

- `Metal` modifies the content to apply the filter.

**Cons**

- `Metal` can be complicated when using Metal Shading Language (MSL), which is derived from C++. The `.metal` code runs on the GPU and is referred to as a shader.

- The learning curve is steep when working with the vertex shader and 3D coordinates.

# Conclusion

The article provides three different approaches to adding the blur effect to an image in iOS. These approaches work with applying the blur effect to any instances of `UIView`.

There is a lot more potential to explore. Thank you for reading. Please leave any questions you might have in the comments.

All code mentioned above can be found in this [GitHub repo](https://gist.github.com/ericleiyang/b0414e284db7e17e918d42ce5403e185).

Original post: [https://betterprogramming.pub/three-approaches-to-apply-blur-effect-in-ios-c1c941d862c3](https://betterprogramming.pub/three-approaches-to-apply-blur-effect-in-ios-c1c941d862c3)
