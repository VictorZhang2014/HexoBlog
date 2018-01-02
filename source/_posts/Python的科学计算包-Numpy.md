---
title: Python的科学计算包 - NumPy
date: 2017-12-24 22:15:04
tags: Python, Numpy
categories: AI
---

[NumPy](http://www.numpy.org/) (Numerical Python extensions)是一个第三方的Python包，用于科学计算。这个库的前身是1995年就开始开发的一个用于数组运算的库。经过了长时间的发展，基本上成了绝大部分Python科学计算的基础包，当然也包括所有提供Python接口的深度学习框架。)

NumPy是用Python编写的，用于科学计算的基础包，包含了以下四种特性：
- 对于处理N维数组对象非常强大
- 复杂（广播）函数 sophisticated (broadcasting) functions
- 便于集成C/C++/Fortran代码
- 支持线性代数，博里叶变换，随机数

通过Github直接下载源码：
```
# 对于NumPy
git clone https://github.com/numpy/numpy.git numpy

# 对于SciPy
git clone https://github.com/scipy/scipy.git scipy
```

[下载安装NumPy, SciPy, Pandas](http://www.googleplus.party/2017/12/30/NumPy,%20SciPy%E5%92%8CPandas%E7%9A%84%E5%AE%89%E8%A3%85%E6%95%99%E7%A8%8B/)


<br/>

## 1.基本类型array
array，也就是数组，是numpy中最基础的数据结构，最关键的属性是维度和元素类型，在numpy中，可以非常方便地创建各种不同类型的多维数组，并且执行一些基本基本操作，来看例子：
```
# coding: utf8
import numpy as np

a = [1, 2, 3, 4]     	    # 创建一个Python数组
b = np.array(a)         	# 根据a创建一个array([1, 2, 3, 4])
type(b)                   	# <type 'numpy.ndarray'>

b.shape                   	# (4,)  一维矩阵长度为4
b.argmax()               	# 3     最大索引
b.max()                   	# 4     最大值
b.mean()                  	# 2.5   平均值


c = [[1,2],[3,4]]    # 二维Python数组
d = np.array(c)      # 二维NumPy数组
print d.shape        # (2, 2)
print d.size         # 4
print d.max(axis=0)  # [3 4]
print d.max(axis=1)  # [2 4]
print d.flatten()    # 将二维数组转成一维数组
print np.ravel(d)    # 将二维数组转成一维数组

# 3x3的浮点二维矩阵，并初始化为所有值为1
e = np.ones((3,3), dtype=np.float)
print e
"""
[[ 1.  1.  1.]
 [ 1.  1.  1.]
 [ 1.  1.  1.]]
"""

# 创建一个一维数组，并把元素值3重复4次
f = np.repeat(3,4)
print f
"""
[3 3 3 3]
"""

# 创建2x2x3无符号8位整型的三维数组，并且初始化所有值为0
g = np.zeros((2,2,3), dtype=np.uint8)
print g
"""
[
  [
    [0 0 0]
    [0 0 0]
  ]
  [
    [0 0 0]
    [0 0 0]
  ]
]
"""
print g.shape     # (2, 2, 3)
print g.astype(np.float)  # 将每个元素都转换成float类型


l = np.arange(10)   # 创建NumPy的一维数组，10个长度
print l.shape       # (10,)
m = np.linspace(0,6,5) # 对0到6之间，取5个值
print m             #  [ 0.   1.5  3.   4.5  6. ]


p = np.array([[1,2,3,4],[5,6,7,8]])
np.save('p.npy', p)    #将数组保存成文件
q = np.load('p.npy')   #加载制定文件的数组
print q

```
注意到在导入numpy的时候，我们将np作为numpy的别名。这是一种习惯性的用法，后面的章节中我们也默认这么使用。作为一种多维数组结构，array的数组相关操作是非常丰富的





 



<br/>
<br/>
<br/>

## References
- https://zhuanlan.zhihu.com/p/24309547

