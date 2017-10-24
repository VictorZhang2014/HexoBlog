---
title: Getting Started on CoreML by Three Steps
date: 2017-10-23 11:47:53
tags: CoreML, Machine Learning, iOS11
categories: AI
---

## Core ML
This tutorial is based on `Core ML` which can be only running on `iOS 11`. 
`Core ML` allows you to integrate machine learning models into your app.
I'll show you `Three Steps` on getting started on CoreML basic use.

<br/>

-------------------------------------------------------------------------
## Overview
With Core ML, you can integrate trained machine learning models into your app. As shown below.
![CoreML framework Procedure](/img/AI/coreml_framework_procedure.png)

So what is `Trained Model`? 
In fact, a `Trained Model` is the result of applying a machine learning algorithm to a set of training data. The model makes predictions based on new input data. For example, a model that's been trained on a region's historical house prices may be able to predict a house's price when given the number of bedrooms and bathrooms.

Core ML is the foundation for `domain-specific` frameworks and funcitonality. Core ML supports Vision for `image analysis`, Foundation for `natural language processing` (for example, the NSLinguisticTagger class), and GameplayKit for evaluating learned `decision trees`. Core ML itself builds on top of low-level primitives like Accelerate and BNNS, as well as Metal Performance Shaders.
![CoreML framework Procedure](/img/AI/CoreML_Foundation_Structure.png)

Core ML is optimized for on-device performance, which minimizes memory footprint and power consumption. Running strictly on the device ensures the privacy of user data and guarantees that your app remains functional and responsive when a network connection is unavailable.

<br/>

-------------------------------------------------------------------------
## Go Ahead!
This example is showing your How to predict houses' price?

## Step 1
First, you need to [download this project](https://docs-assets.developer.apple.com/published/62efb7030a/IntegratingaCoreMLModelintoYourApp.zip) . This is an official demo, if you can understand this project demo, just ignore my anatomy and next steps.

## Step 2
Second, `creating` a Xcode project -> choosing `Swift language` -> `dragging` the file `MarsHabitatPricer.mlmodel` into your project. `MarsHabitatPricer.mlmodel` file located in official demo.

## Step 3
Finally, There is a snnippet code in `ViewController.swift` file as shown below:
```
    //1.Initialized an instance of MarsHabitatPricer
    let model = MarsHabitatPricer()

    override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
        
        //2.Given the input data
        let solarPanels: Double = 2.0 //Range from 1.0 to 5.0
        let greenHouses: Double = 3   //Range from 1 to 5
        let acres: Double = 1500      //Range from 750 to 10,000
        
        //3.Predict a price according by solarPanels, greenHouses and acres
        guard let priceOutput = try? model.prediction(solarPanels: solarPanels, greenhouses: greenHouses, size: acres) else {
            fatalError("Getting an unexpected error.")
        }
        
        //4.Output the predicted price
        let outputPrice = priceOutput.price
        print("house price prediction is \(outputPrice)")
        
    }
```
then, building and running your Core ML app and you'll see there is a result outputed in Console
```
house price prediction is 7264.0491831318
```
which means you get the house price according by your input data(solarPanels, greenHouses and acres)

<br/>

-------------------------------------------------------------------------
## References
[Core ML Documentation](https://developer.apple.com/documentation/coreml)
[Integrating a Core ML Model into your App](https://developer.apple.com/documentation/coreml/integrating_a_core_ml_model_into_your_app)



<br/>

======================================================================================================

## Core ML
这个简单的教程是基于`Core ML`，`Core ML`只能运行在`iOS 11`。
`Core ML`允许你集成机器学习训练模型到你的iOS应用。
我将要通过三步曲来为你展示一下`Core ML`基本的使用。

<br/>

-------------------------------------------------------------------------
## 概述
通过`Core ML`，你可以把机器学习训练的模型集成到你的iOS应用上，如下图所示：
![CoreML framework 流程](/img/AI/coreml_framework_procedure.png)

所以，什么是`训练的模型（Trained Model）`? 
事实上，一个`训练的模型（Trained Model）`就是一个应用机器学习算法到一组训练数据的集合的结果。这个模型可以通过新提供的数据进行预测。 例如，一个模型在一个区域的历史房价上被训练了，那么这个模型可以通过给定的房间数和洗手间数来预测房价。 

`Core ML`是一个特定领域框架（domain-specific）和功能性的基础。`Core ML`支持视觉的图像分析，自然语言处理的基础（例如，`NSLinguisticTagger`类），还有`GameplayKit`用于评估学习`决策树（Decision Trees）`。`Core ML`本身是建立在低层的原语（`low-level primitives`）之上，就像`Accelerate`和`BNNS`，还有`Metal Performance Shaders`。
![CoreML架构](/img/AI/CoreML_Foundation_Structure.png)

`Core ML`是在设备内部做的优化，这种优化可以大大的减小内存占用（`Memory Footprint`）和性能消耗（`Power Consumption`）。
当你的设备网络不可用时，并且在你的设备上运行`Core ML`，它严格的确保了用户数据的隐私性，也保证了设备上的应用的基本操作和响应性。

<br/>

-------------------------------------------------------------------------
## 开始吧！
这个例子是告诉你如何测试房价？

## 步骤一
首先, 你需要[下载此项目](https://docs-assets.developer.apple.com/published/62efb7030a/IntegratingaCoreMLModelintoYourApp.zip) 。这是一个官方的例子，如果你可以理解这个项目，那你就忽略下面两步吧。

## 步骤二
然后, 创建一个项目 -> 选择Swift语言 -> 拖拽`MarsHabitatPricer.mlmodel`文件到你的项目， 这个`MarsHabitatPricer.mlmodel`文件在第一步下载的Demo目录里.

## 步骤三
最后，在`ViewController.swift`文件里，插入如下代码： 
```
    //1.初始化一个MarsHabitatPricer的实例
    let model = MarsHabitatPricer()

    override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
        
        //2.给定输入数据
        let solarPanels: Double = 2.0 //Range from 1.0 to 5.0
        let greenHouses: Double = 3   //Range from 1 to 5
        let acres: Double = 1500      //Range from 750 to 10,000
        
        //3.根据solarPanels、greenHouses和acres来预测房价
        guard let priceOutput = try? model.prediction(solarPanels: solarPanels, greenhouses: greenHouses, size: acres) else {
            fatalError("捕获到未知的错误。")
        }
        
        //4.输出预测的房价
        let outputPrice = priceOutput.price
        print("房价预测的结果为： \(outputPrice)")
        
    }
```
通过`cmd + r` 或者Xcode导航栏左上角的启动按钮，启动你的Xcode项目，在控制台你就能看到输出的结果了
```
房价预测的结果为： 7264.0491831318
```
这个结果的意思是，通过提供的solarPanels、greenHouses和acres输入数据预测出了房价

<br/>

-------------------------------------------------------------------------
## References
[Core ML官方文档](https://developer.apple.com/documentation/coreml)
[集成Core ML模型到你的项目](https://developer.apple.com/documentation/coreml/integrating_a_core_ml_model_into_your_app)

