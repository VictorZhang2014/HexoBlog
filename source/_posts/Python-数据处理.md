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

```
# coding: utf-8

from __future__ import division
import numpy as np
import os
import matplotlib.pyplot as plt
from scipy.interpolate import lagrange  # 导入拉格朗日插值法

np.random.seed(12345)
plt.rc('figure', figsize=(10, 6))

import pandas as pd
np.set_printoptions(precision=4, threshold=500)
pd.options.display.max_rows = 100


## 缺失值处理 -- 拉格朗日插值法
inputfile = 'data_screening_files/catering_sale.xls' # 销量数据
outputfile = 'data_screening_files/sales.xls'  # 输出数据路径

data = pd.read_excel(inputfile)
print data
'''
            日期      销量
0   2015-03-01    51.0
1   2015-02-28  2618.2
2   2015-02-27  2608.4
3   2015-02-26  2651.9
4   2015-02-25  3442.1
5   2015-02-24  3393.1
......

[201 rows x 2 columns]
'''

# 过滤异常值，将其变为空值
exception_values = data[u'销量'][(data[u'销量'] < 400) | (data[u'销量'] > 5000)]
print(exception_values)
'''
0        51.00
8      6607.40
103      22.00
110      60.00
144    9106.44
Name: 销量, dtype: float64
'''

# 会报错
# data[u'销量'][(data[u'销量'] < 400) | (data[u'销量'] > 5000)] = None
# print(data)


# 自定义列向量插值函数

```











