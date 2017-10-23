---
title: Classifying Images with Vision and Core ML
date: 2017-10-23 16:45:58
tags: Core ML, Classifying Images
---

If you don't know the basic use of Core ML, see [this page](/2017/10/23/Getting-Started-on-CoreML-by-Three-Steps/) , on the contrary, keep going our tutorial.

Use Vision with Core ML to perform image classification.

## Overview
With the `Core ML` framework, you can use a trained machine learning model to classify input data. The Vision framework works with Core ML to apply classification models to images, and to preprocess those images to make machine learning tasks easier and more reliable.

This tutorial I'm going to show you the open source `MobileNet` model, which identifies an image using 1000 classification categories as seen in the example screenshots below.

![Official Classifying Image](/img/AI/officialClassifyImages.png)
<center>Figure - Official Test Case [Download Sample Code](https://docs-assets.developer.apple.com/published/43bca2cbbd/ClassifyingImageswithVisionandCoreML.zip)</center>

<br/>
<img src="/img/AI/FountainMobileNet.jpeg" alt="" width="320" height="568" /><img src="/img/AI/StrawberryMobileNet.jpeg" alt="" width="320" height="568" />
<img src="/img/AI/LuHanMobileNet.jpeg" alt="" width="320" height="568" /><img src="/img/AI/avatarMobileNet.jpeg" alt="" width="320" height="568" />
<center>Figure - My Test Case</center>

Now, we've seen it's completely identified in official test case, but in my test case, half of them are failed to be identified. So I can sum this example code up,the trained model is unmatured. 

But anyway, this is a good and easy step for us to going.

## Start to my performance
`Core ML` automatically generates a Swift class which is `MobileNet`, that provides easy access to your ML model. To set up a Vision request using the model, create an instance of that class and use its model property to create a `VNCoreMLRequest` object. Use the request object's completion handler to specify a method to receive results from the model after you run the request. 

** Download the Model **
[Download Model](https://docs-assets.developer.apple.com/coreml/models/MobileNet.mlmodel)
After downloaded, dragging it to your project, complete snippet code as seen below.

```
import UIKit
import CoreML
import Vision

class ViewController: UIViewController {
    
    override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
        choosePhotosAndHandle()
    }
    
    //1. Choose an image and handle it
    func choosePhotosAndHandle() {
        //Specific an image
        let imgpath = Bundle.main.path(forResource: "Strawberry", ofType: ".jpeg")
        if imgpath == nil {
            print("You specified image is nonexistent")
            return
        }
        let image = UIImage(contentsOfFile: imgpath!)!
        let orientation = CGImagePropertyOrientation(rawValue: UInt32(image.imageOrientation.rawValue))!
        guard let ciImage = CIImage(image: image) else {
            fatalError("Unable to create \(CIImage.self) from \(image).")
        }
        
        DispatchQueue.global(qos: .userInitiated).async {
            let handler = VNImageRequestHandler(ciImage: ciImage, orientation: orientation)
            do {
                try handler.perform([self.classifyRequest])
            } catch {
                print("Failed to perform classification.\n\(error.localizedDescription)")
            }
        }
    }
    
    //2.Initialized an instance of MobileNet in lazy mode
    lazy var classifyRequest: VNCoreMLRequest = {
        let mlModel = MobileNet().model
        
        do {
            let model = try VNCoreMLModel(for: mlModel)
            let request = VNCoreMLRequest(model: model) { [weak self] request, error in
                self?.processClassifications(for: request, error: error)
            }
            request.imageCropAndScaleOption = .centerCrop
            return request
        } catch {
            fatalError("Failed to load Vision ML model: \(error)")
        }
    }()
    
    // 3.This is an callback
    func processClassifications(for request: VNRequest, error: Error?) {
        DispatchQueue.main.async {
            guard let results = request.results else {
                print("Unable to classify image. \(error!.localizedDescription)")
                return
            }
            
            // The `results` will always be `VNClassificationObservation`s, as specified by the Core ML model in this project.
            let classifications = results as! [VNClassificationObservation]
            
            if classifications.isEmpty {
                print("Nothing recognized")
            } else {
                // Display top classifications ranked by confidence in the UI
                let topClassifications = classifications.prefix(2)
                let descriptions = topClassifications.map({ classification in
                    return String(format: "   (%.2f) %@", classification.confidence, classification.identifier)
                })
                
                //This is the output result
                print("Classification: \n" + descriptions.joined(separator: "\n"))
            }
        }
    }
}
```
What we will always be seen in the console window as shown below:
```
Classification: 
   (1.00) strawberry
   (0.00) trifle
```

The property of `imageCropAndScaleOption` in the class of `VNCoreMLRequest` illustration is an ML model processes input images in a fixed aspect ratio, but input images may have arbitrary aspect ratio, so Vision must scale of crop the image to fit. For best results, set a value to this property that matches the image layout the model was trained with. For the [available classification models](https://developer.apple.com/machine-learning/), the `centerCrop` option is appropriate unless noted otherwise.


[Official Classifying Images Tutorial](https://developer.apple.com/documentation/vision/classifying_images_with_vision_and_core_ml)
<br/>
<br/>

=======================================================================================================
如果你还不知道Core ML的基本使用，那就先看看这个[教程](/2017/10/23/Getting-Started-on-CoreML-by-Three-Steps/) ，相反，就直接往下看吧。

使用`Vision with Core ML`来执行图片分类（`image classification`）。

## 概述
通过`Core ML`framework，你可以通过已经训练的机器模型对输入的数据（提供的图片）进行分类。`Vision`framework与`Core ML`紧密结合工作，应用于分类模型到图片，并且预处理那些图片，使机器学习任务更简单，更可靠。

本章教程，我将展示给你一个开源`MobileNet`模型的使用，这个模型能使用1000中分类类型来识别图片，如下截图所示：

![官方分类后图片效果](/img/AI/officialClassifyImages.png)
<center>图片 - 官方测试案例 [下载样本代码](https://docs-assets.developer.apple.com/published/43bca2cbbd/ClassifyingImageswithVisionandCoreML.zip)</center>

<br/>
<img src="/img/AI/FountainMobileNet.jpeg" alt="" width="320" height="568" /><img src="/img/AI/StrawberryMobileNet.jpeg" alt="" width="320" height="568" />
<img src="/img/AI/LuHanMobileNet.jpeg" alt="" width="320" height="568" /><img src="/img/AI/avatarMobileNet.jpeg" alt="" width="320" height="568" />
<center>图片 - 我的测试案例</center>

现在，我们可以看到，官方的测试案例的图片全部正确识别了，但是在我的案例里，一半被识别出来了，一半失败了。简单的总结，这个模型并不成熟。

但是，不管怎么样，这对我们来说，是一个简单而且很不错的开始。

## 开始我的表演吧😄
`Core ML`自动生成`Swift`类，这个类叫做`MobileNet`，提供了轻松的访问这个ML模型。使用这个模型来创建`VNCoreMLRequest`对象，并且设置相关的属性，最后给这个请求指定一个回调用来接收模型的输出结果。

[下载模型](https://docs-assets.developer.apple.com/coreml/models/MobileNet.mlmodel)
下载后，把这个模型拖拽到你的项目，完整的代码如下显示

```
import UIKit
import CoreML
import Vision

class ViewController: UIViewController {
    
    override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
        choosePhotosAndHandle()
    }
    
    //1. 选择一张图片并且处理它
    func choosePhotosAndHandle() {
        //指定一张图片
        let imgpath = Bundle.main.path(forResource: "Strawberry", ofType: ".jpeg")
        if imgpath == nil {
            print("You specified image is nonexistent")
            return
        }
        let image = UIImage(contentsOfFile: imgpath!)!
        let orientation = CGImagePropertyOrientation(rawValue: UInt32(image.imageOrientation.rawValue))!
        guard let ciImage = CIImage(image: image) else {
            fatalError("Unable to create \(CIImage.self) from \(image).")
        }
        
        DispatchQueue.global(qos: .userInitiated).async {
            let handler = VNImageRequestHandler(ciImage: ciImage, orientation: orientation)
            do {
                try handler.perform([self.classifyRequest])
            } catch {
                print("未能执行分类.\n\(error.localizedDescription)")
            }
        }
    }
    
    //2.懒加载MobileNet
    lazy var classifyRequest: VNCoreMLRequest = {
        let mlModel = MobileNet().model
        
        do {
            let model = try VNCoreMLModel(for: mlModel)
            let request = VNCoreMLRequest(model: model) { [weak self] request, error in
                self?.processClassifications(for: request, error: error)
            }
            request.imageCropAndScaleOption = .centerCrop
            return request
        } catch {
            fatalError("Failed to load Vision ML model: \(error)")
        }
    }()
    
    // 3.这是回调
    func processClassifications(for request: VNRequest, error: Error?) {
        DispatchQueue.main.async {
            guard let results = request.results else {
                print("Unable to classify image. \(error!.localizedDescription)")
                return
            }
            
            // `results`永远都是`VNClassificationObservation`数组，在这个项目中有ML Model特指的
            let classifications = results as! [VNClassificationObservation]
            
            if classifications.isEmpty {
                print("Nothing recognized")
            } else {
                // Display top classifications ranked by confidence in the UI
                let topClassifications = classifications.prefix(2)
                let descriptions = topClassifications.map({ classification in
                    return String(format: "   (%.2f) %@", classification.confidence, classification.identifier)
                })
                
                //这是输出结果
                print("Classification: \n" + descriptions.joined(separator: "\n"))
            }
        }
    }
}
```
我们可以在控制台窗口看到如下输出：
```
Classification: 
   (1.00) strawberry
   (0.00) trifle
```

在类`VNCoreMLRequest`的属性`imageCropAndScaleOption`，解释：一个ML模型处理图片时是基于固定宽高比（fixed aspect ratio）的，但是提供的图片的宽高比可能是任意的，所以`Vision`就必须把图片进行裁切或者规格化后去适配这个模型所需。为了达到最好的效果，我们一般会设置一个值来匹配图片的布局，这样的图片的布局就是模型所需要的。[所有可用的分类模型](https://developer.apple.com/machine-learning/)，参数选项`centerCrop`是较为合适的，除非有特殊说明用别的，否则，就是这个。


[官方图片分类教程](https://developer.apple.com/documentation/vision/classifying_images_with_vision_and_core_ml)
