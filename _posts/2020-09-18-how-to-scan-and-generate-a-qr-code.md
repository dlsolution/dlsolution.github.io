---
layout: post
title: "How to scan and generate a QR Code"
author: "Linh Vo"
tags: "UIKit"
---

# QR Code

QR codes are a type of barcode that can store a lot of information in a small space. They can be scanned by a smartphone camera, and the information can be used to do things like open a website, make a phone call, or send an email.

QR codes are great for marketing and advertising.

# The Basics

## AVCaptureSession

```swift
class AVCaptureSession : NSObject
```

An object that manages capture activity and coordinates the flow of data from input devices to capture outputs.

To perform real-time capture, you instantiate an AVCaptureSession object and add appropriate inputs and outputs. The following code fragment illustrates how to configure a capture device to record audio.

```swift
// Create the capture session.
let captureSession = AVCaptureSession()

// Find the default audio device.
guard let audioDevice = AVCaptureDevice.default(for: .audio) else { return }

do {
    // Wrap the audio device in a capture device input.
    let audioInput = try AVCaptureDeviceInput(device: audioDevice)
    // If the input can be added, add it to the session.
    if captureSession.canAddInput(audioInput) {
        captureSession.addInput(audioInput)
    }
} catch {
    // Configuration failed. Handle error.
}
```

You invoke `startRunning()` to start the flow of data from the inputs to the outputs, and invoke `stopRunning()` to stop the flow.

> Important: The startRunning() method is a blocking call which can take some time, therefore you should perform session setup on a serial queue so that the main queue isn’t blocked (which keeps the UI responsive). See AVCam: Building a Camera App for an implementation example.

## AVCaptureVideoPreviewLayer

```swift
class AVCaptureVideoPreviewLayer : CALayer
```

A Core Animation layer that displays the video as it’s captured.

AVCaptureVideoPreviewLayer is a subclass of CALayer that you use to display video as it’s captured by an input device.

You use this preview layer in conjunction with a capture session, as shown in the following code fragment.

```swift
// Create a preview layer.
let previewLayer = AVCaptureVideoPreviewLayer()

// Connect the preview layer with the capturing session.
previewLayer.session = captureSession

// Add the preview layer into the view's layer hierarchy.
view.layer.addSublayer(previewLayer)
```

## AVMetadataMachineReadableCodeObject

Barcode information detected by a metadata capture output.

```swift
class AVMetadataMachineReadableCodeObject : AVMetadataObject
```

## AVCaptureDevice

A device that provides input (such as audio or video) for capture sessions and offers controls for hardware-specific capture features.

```swift
class AVCaptureDevice : NSObject
```

## AVCaptureDeviceInput

A capture input that provides media from a capture device to a capture session.

```swift
class AVCaptureDeviceInput : AVCaptureInput
```

# How to scan a QR Code

iOS has built-in support for scanning QR codes using AVFoundation, but the code isn't easy: you need to create a capture session, create a preview layer, handle delegate callbacks, and more. To make it easier for you, I've created a `UIViewController` subclass that does all the hard work for you – you just need to modify the `found(code:)` method to do something more interesting.

{% include image.html
img="assets/2020-09-18/IMG_9369.PNG"
title="Scanning and decoding QR codes"
caption="Scanning and decoding QR codes"
max-width="300px" %}

```swift
import AVFoundation
import UIKit

class ScannerViewController: UIViewController, AVCaptureMetadataOutputObjectsDelegate {

    let messageView: UIView = {
        let v = UIView()
        v.backgroundColor = .init(white: 1, alpha: 0.3)
        v.translatesAutoresizingMaskIntoConstraints = false
        return v
    }()

    let messageLabel: UILabel = {
        let l = UILabel()
        l.text = "No QR code is detected"
        l.textColor = .white
        l.translatesAutoresizingMaskIntoConstraints = false
        return l
    }()

    let qrCodeFrameView: UIView = {
        let v = UIView()
        v.layer.borderColor = UIColor.green.cgColor
        v.layer.borderWidth = 2
        return v
    }()

    var captureSession: AVCaptureSession!
    var previewLayer: AVCaptureVideoPreviewLayer!

    override func viewDidLoad() {
        super.viewDidLoad()

        view.backgroundColor = UIColor.black

        captureSession = AVCaptureSession()

        guard let videoCaptureDevice = AVCaptureDevice.default(for: .video) else { return }
        let videoInput: AVCaptureDeviceInput

        do {
            videoInput = try AVCaptureDeviceInput(device: videoCaptureDevice)
        } catch {
            return
        }

        if (captureSession.canAddInput(videoInput)) {
            captureSession.addInput(videoInput)
        } else {
            failed()
            return
        }

        let metadataOutput = AVCaptureMetadataOutput()

        if (captureSession.canAddOutput(metadataOutput)) {
            captureSession.addOutput(metadataOutput)

            metadataOutput.setMetadataObjectsDelegate(self, queue: DispatchQueue.main)
            metadataOutput.metadataObjectTypes = [.qr]
        } else {
            failed()
            return
        }

        previewLayer = AVCaptureVideoPreviewLayer(session: captureSession)
        previewLayer.frame = view.layer.bounds
        previewLayer.videoGravity = .resizeAspectFill
        view.layer.addSublayer(previewLayer)

        captureSession.startRunning()

        view.addSubview(qrCodeFrameView)
        view.bringSubviewToFront(qrCodeFrameView)

        view.addSubview(messageView)
        messageView.addSubview(messageLabel)
        NSLayoutConstraint.activate([
            messageView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            messageView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
            messageView.bottomAnchor.constraint(equalTo: view.bottomAnchor),
            messageView.heightAnchor.constraint(equalToConstant: 60),
            messageLabel.centerXAnchor.constraint(equalTo: messageView.centerXAnchor),
            messageLabel.centerYAnchor.constraint(equalTo: messageView.centerYAnchor)
        ])
    }

    func failed() {
        let ac = UIAlertController(title: "Scanning not supported", message: "Your device does not support scanning a code from an item. Please use a device with a camera.", preferredStyle: .alert)
        ac.addAction(UIAlertAction(title: "OK", style: .default))
        present(ac, animated: true)
        captureSession = nil
    }

    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)

        if (captureSession?.isRunning == false) {
            captureSession.startRunning()
        }
    }

    override func viewWillDisappear(_ animated: Bool) {
        super.viewWillDisappear(animated)

        if (captureSession?.isRunning == true) {
            captureSession.stopRunning()
        }
    }

    deinit {
        print("deinit ScannerViewController")
    }

    func metadataOutput(_ output: AVCaptureMetadataOutput, didOutput metadataObjects: [AVMetadataObject], from connection: AVCaptureConnection) {

        // Check if the metadataObjects array is not nil and it contains at least one object.
        if metadataObjects.count == 0 {
            qrCodeFrameView.frame = CGRect.zero
            messageLabel.text = "No QR code is detected"
            return
        }

        if let metadataObject = metadataObjects.first {
            guard let readableObject = metadataObject as? AVMetadataMachineReadableCodeObject else { return }
            guard let stringValue = readableObject.stringValue else { return }
            guard let barCodeObject = previewLayer?.transformedMetadataObject(for: readableObject) else { return }
            AudioServicesPlaySystemSound(SystemSoundID(kSystemSoundID_Vibrate))
            qrCodeFrameView.frame = barCodeObject.bounds
            found(code: stringValue)
        }
    }

    func found(code: String) {
        messageLabel.text = code
    }

    override var prefersStatusBarHidden: Bool {
        return true
    }

    override var supportedInterfaceOrientations: UIInterfaceOrientationMask {
        return .portrait
    }
}
```

# How to generate a QR Code

iOS has a built-in QR code generator, but it's a bit tricksy to use because it's exposed as a `Core Image filter` that needs various settings to be applied. Also, it generates codes where every bit is just one pixel across, which looks terrible if you try to stretch it inside an image view.

{% include image.html
img="assets/2020-09-18/IMG_9370.jpeg"
%}

So, here's a simple function that wraps up QR code generation while also scaling up the QR code so it's a respectable size:

```swift
func generateQRCode(from string: String) -> UIImage? {
    let data = string.data(using: String.Encoding.ascii)

    if let filter = CIFilter(name: "CIQRCodeGenerator") {
        filter.setValue(data, forKey: "inputMessage")
        let transform = CGAffineTransform(scaleX: 3, y: 3)

        if let output = filter.outputImage?.transformed(by: transform) {
            return UIImage(ciImage: output)
        }
    }

    return nil
}
```

To display the QR code (i.e. an `UIImage`), you can add an image view to your storyboard and connect it with the view controller. Then you can create a QR code to the image view for example by:

```swift
let QRimage = generateQRCode(from: "Hello, world!")

QRView.image = QRimage // Replace QRView with your image view!
```

# How to scan a QR Code from Gallery

Now , after scanning and generating the code , sometimes we need to scan the code from the image in our iOS photo library. This feature doesn’t require any third party library as its available in AVFoundation.

AV Foundation is the full featured framework for working with time-based audiovisual media on iOS, macOS, watchOS and tvOS. Using AV Foundation, you can easily play, create, and edit QuickTime movies and MPEG-4 files, play HLS streams, and build powerful media functionality into your apps.

To scan a code from the image available in gallery Follow these steps:-

1. import AVFoundation in your ViewController class
2. Now assign UIImagePickerControllerDelegate and UINavigationControllerDelegate to get the image from gallery.
3. Get your image from the image gallery and detect the QRCode using CIDetecter

> CIDetecter is an image processor that identifies notable features (such as faces and barcodes) in a still image or video.

Your overall code will be:

```swift
extension QRScannerViewController: UIImagePickerControllerDelegate, UINavigationControllerDelegate {
    func imagePickerController(_ picker: UIImagePickerController, didFinishPickingMediaWithInfo info: [UIImagePickerController.InfoKey : Any]) {

        var qrCodeLink = ""

        if let qrcodeImg = info[UIImagePickerController.InfoKey.originalImage] as? UIImage {

            // We have created an instance of CIDetecter of type QRCode
            let detector = CIDetector(ofType: CIDetectorTypeQRCode, context: nil, options: [CIDetectorAccuracy : CIDetectorAccuracyHigh])!

            // Converted the selected image into CIImage
            let ciImage = CIImage(image:qrcodeImg)!

            // Detect the encoded information from the image using the detecter.
            let features=detector.features(in: ciImage)
            for feature in features as! [CIQRCodeFeature] {
                qrCodeLink += feature.messageString!
            }
        }

        if qrCodeLink == "" {
            print("nothing")
        } else {
            print("message: \(qrCodeLink)")
        }

        dismiss(animated: true, completion: nil)
    }
}
```

Now you have the decoded information from the image containing your QRCode.
