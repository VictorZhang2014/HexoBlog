---
title: Converting Trained Models to Core ML
date: 2017-10-23 19:43:32
tags: Core ML
---

Convert trained models created with third-party machine learning tools to the Core ML model format.

## Overview
If your model is created and trained using a supported third-party machine learning tool, you can use [Core ML Tools](https://pypi.python.org/pypi/coremltools) to convert it to the Core ML model format. There is `Table` as shown below, that lists the supported models and third-party tools. 

> Note
> Core ML Tools is a Python package(coremltools), hosted at the Python Package Index(PyPI).

> The table shows Models and third-party tools supported by Core ML Tools

Model Type                  |                          Supported Models                          | Supported Tools
----------------------------|--------------------------------------------------------------------|----------------------------------
----------------------------|--------------------------------------------------------------------|----------------------------------
Neural networks             | Feedforward, convolutional, recurrent                              | Caffe v1, Keras 1.2.2+
----------------------------|--------------------------------------------------------------------|----------------------------------
Tree ensembles              | Random forests, boosted trees, decision trees                      | scikit-learn 0.18, XGBoost 0.6
----------------------------|--------------------------------------------------------------------|----------------------------------
Support vector machines     | Scalar regression, multiclass classification                       | scikit-learn 0.18, LIBSVM 3.22
----------------------------|--------------------------------------------------------------------|----------------------------------
Generalized linear models   | Linear regression, logistic regression                             | scikit-learn 0.18
----------------------------|--------------------------------------------------------------------|----------------------------------
Feature engineering         | Sparse vectorization, dense vectorization, categorical processing  | scikit-learn 0.18
----------------------------|--------------------------------------------------------------------|----------------------------------
Pipeline models             | Sequentially chained models                                        | scikit-learn 0.18
----------------------------|--------------------------------------------------------------------|----------------------------------

<br/>

## Convert your Model
Convert your model using the Core ML converter that corresponds to your models third-party tool. Call the converter's convert method and save the resulting model to the Core ML model format(.mlmodel).

For example, if your model was created using Caffe, pass the Caffe mode(.caffemodel) to the `coremltools.converters.caffe.convert` method.

```
> import coremltools
> coreml_model = coremltools.converters.caffe.convert('my_caffe_model.caffemodel')
```

Now save the resulting model in the Core ML model format.

```
> coreml_model.save('my_model.mlmodel')
```

Depending on your model, you might need to update inputs, outputs, and lables, or you might need to declare image names, types, and formats. The conversion tools are bundled with more documentation, as the options available vary by tool. For more information about Core ML Tools, see the [Package Documentation](https://apple.github.io/coremltools/).

<br/>

## Alternatively, write a Custome Conversion Tool
It's possible to create your own conversion tool when you need to convert a model that isn't in a format supported by the tools listed in above table.

Writing your own conversion tool involves translating the representation of your model's input, output, and architecture into the Core ML model format. You do this by defining each layer of the model's architecture and its connectivity with other layers. Use the conversion tools provided by [Core ML Tools](https://pypi.python.org/pypi/coremltools) as examples; they demonstrate how various model types created from third-party are converted to the Core ML model format.

> Note
> The Core ML model format is defined by a set of protocol buffer files and is described in detail in the [Core ML Model Specification](https://developer.apple.com/machine-learning/).

<br/>

[Official Tutorial](https://developer.apple.com/documentation/coreml/converting_trained_models_to_core_ml)



<br/>
<br/>


把第三方机器学习工具创建的训练模型转换到Core ML模型格式

## 概述
如果你的模型是由第三方机器学习工具创建的和训练的，那么你可以使用[Core ML Tools](https://pypi.python.org/pypi/coremltools)工具，该工具可以把这些模型转换成Core ML可用模型。下面有一张表，是[Core ML Tools](https://pypi.python.org/pypi/coremltools)转换工具所支持的。

> 注意
> Core ML Tools是一个Python包，叫做coremltools，该工具是在Python Package Index(PyPI)运行的.

>  下面的表格是[Core ML Tools](https://pypi.python.org/pypi/coremltools)工具所支持的第三方机器学习工具创建的模型

模型类型                     |                          所支持的模型                                |    所支持的工具
----------------------------|--------------------------------------------------------------------|----------------------------------
----------------------------|--------------------------------------------------------------------|----------------------------------
Neural networks             | Feedforward, convolutional, recurrent                              | Caffe v1, Keras 1.2.2+
----------------------------|--------------------------------------------------------------------|----------------------------------
Tree ensembles              | Random forests, boosted trees, decision trees                      | scikit-learn 0.18, XGBoost 0.6
----------------------------|--------------------------------------------------------------------|----------------------------------
Support vector machines     | Scalar regression, multiclass classification                       | scikit-learn 0.18, LIBSVM 3.22
----------------------------|--------------------------------------------------------------------|----------------------------------
Generalized linear models   | Linear regression, logistic regression                             | scikit-learn 0.18
----------------------------|--------------------------------------------------------------------|----------------------------------
Feature engineering         | Sparse vectorization, dense vectorization, categorical processing  | scikit-learn 0.18
----------------------------|--------------------------------------------------------------------|----------------------------------
Pipeline models             | Sequentially chained models                                        | scikit-learn 0.18
----------------------------|--------------------------------------------------------------------|----------------------------------

<br/>

## 转换你可用模型
使用Core ML转换器将第三方工具创建的模型转换成你可用的模型。就调用转换器的`convert`方法，并且保存Core ML模型的结果，以扩展名为.mlmodel。

例如：如果你使用的是Caffe创建的模型，那传递Caffe模型（扩展名为：.caffemodel）到 `coremltools.converters.caffe.convert`方法，如下代码所示：

```
> import coremltools
> coreml_model = coremltools.converters.caffe.convert('my_caffe_model.caffemodel')
```

现在就把结果的模型保存，以Core ML的格式

```
> coreml_model.save('my_model.mlmodel')
```

这取决于你的模型，可能你需要更新输入、输出和标签，或者你可能需要声明image names、types和格式。这个转化工具里有些文档，你可以查阅，由于工具不同所以相应的参数也不一样。更多关于Core ML Tools的信息, 来这里看[Package Documentation](https://apple.github.io/coremltools/).

<br/>

## 这个是可选的，编写一个自定义的转换工具
在上面的列表中，如果没有适合你的工具，那么你也可以创建一个转换工具，转换成你想要的模型。

编写一个转换工具会涉及到你模型的输入、输出和架构的表现，转换到Core ML模型。你需要做的就是，定义模型架构的每一层和每一层的连接性。那我们提供的[Core ML Tools](https://pypi.python.org/pypi/coremltools) 举例，从第三方创建的不同的模型类型转换到Core ML所需要的模型格式。

> 注意
> Core ML格式是由一组协议缓冲文件所定义的，并且在[Core ML Model Specification](https://developer.apple.com/machine-learning/) 有详细描述。

<br/>

[官方教程](https://developer.apple.com/documentation/coreml/converting_trained_models_to_core_ml)





