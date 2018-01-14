---
title: Python - 数据处理
date: 2018-01-13 22:18:46
tags: Python, SciPy, 数据处理
categories: AI
---

# 数据整理与预处理
- 数据清洗
  - 删除重复记录
  - 缺失值处理
  - 数据插补 - 拉格朗日插值法、牛顿插值法
  - 异常值处理

- 合并数据集
- 数据转换
- 重塑和轴向旋转
- 字符串操作
- 示例


## 拉格朗日 插值法 
该算法在SciPy里
拉格朗日插值法和牛顿插值法得到的结果是一样的，只不过他们的过程不一样，在Python的科学计算库中，我们只能使用拉格朗日插值法，因为没有牛顿插值法

因为本节课需要使用到matplotlib，所以，如果读者遇到像我这个提示一样的错误
```
RuntimeError: Python is not installed as a framework. The Mac OS X backend will not be able to function correctly if Python is not installed as a framework.
```
那么解决方法在：https://stackoverflow.com/questions/21784641/installation-issue-with-matplotlib-python












