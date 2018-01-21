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

# 创建一个新数组，内存上并不填充任何值
a = np.empty((2,3,2))


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


## 向量相加
一个一维数组就是一个向量，在python中的向量是不可以相加运算，但是numpy就可以
```
a = np.arange(5)
b = np.arange(5)
print a + b
```

# NumPy支持的数据类型
```
int8, unit8
uint16, uint16
int32, uint32
int64, uint64
float16
float32
float64
float128
complex64, complex128, complex256
bool                ?
object              O
string_             S
unicode_            U


# 初始化指定类型
print np.arange(7, dtype=np.uint16)
'''[0 1 2 3 4 5 6]'''


# 数据类型转换
arr = np.array([1,2,3,4])
print arr.dtype    #输出 int64

float_arr = arr.astype(np.float64)
print float_arr   # [ 1.  2.  3.  4.]

str = np.array(['1.2', '-9.79', '42'], dtype=np.string_)
print str   # 输出['1.2' '-9.79' '42']

print str.astype(float)  # 输出[  1.2   -9.79  42.  ]


# 数据类型对象
a = np.array([[1,2],[3,4]])
print a.dtype    # 打印数组元素类型 : int64
print a.dtype.byteorder
print a.dtype.itemsize   # 打印数组元素类型大小 ： 8


# 字符编码
# 如果觉得dtype=np.string_太长的话，也可以写字符编码的简短形式
print np.arange(7, dtype='f')    # 浮点类型
'''
[ 0.  1.  2.  3.  4.  5.  6.]
'''

print np.arange(7, dtype='D')   # 复数类型
'''
[ 0.+0.j  1.+0.j  2.+0.j  3.+0.j  4.+0.j  5.+0.j  6.+0.j]
'''

print np.dtype(float)   # float64
print np.dtype('f')     # float32
print np.dtype('f8')    # float64


# dtype类的属性
t = np.dtype('Float64')
print t.char    # 打印类型，双精度：d
print t.type    # <type 'numpy.float64'>
print t.str     # 打印对应的编码：<f8
```

创建自定义数据类型
```
# 自定义类型的基本结构
# np.dtype([(name, dtype, len),(name, dtype, len),(name, dtype, len)])

# 第一步，这是自定义类型，下面就可以使用t作为数据类型
t = np.dtype([('name', np.str_, 40), ('numitems', np.int32), ('price', np.float32)])
print t
'''
[('name', 'S40'), ('numitems', '<i4'), ('price', '<f4')]
'''

print t['name']

# 使用自定义类型创建对象
itemz = np.array([('Victor', 20, 3.14), ('Buffet', 10, 2.34)], dtype=t)
print itemz
'''
[('Victor', 20,  3.1400001 ) ('Buffet', 10,  2.33999991)]
'''
```


## 数组与标量的运算 
对于NumPy来说可以少写很多for循环，而且可以完成数组运算，这种方式就叫做`向量化编程`

以下说下运算符，假如：a=10, b=20;
- ** + **	  加 - 两个对象相加	                              a + b 输出结果 30
- ** - **	  减 - 得到负数或是一个数减去另一个数	                a - b 输出结果 -10
- ** * **	  乘 - 两个数相乘或是返回一个被重复若干次的字符串	      a * b 输出结果 200
- ** / **	  除 - x除以y	                                   b / a 输出结果 2
- ** % **	  取模 - 返回除法的余数	                           b % a 输出结果 0
- ** ** **	幂 - 返回x的y次幂	                               a**b 为10的20次方， 输出结果 100000000000000000000
- ** // **	取整除 - 返回商的整数部分	                        9//2 输出结果 4 , 9.0//2.0 输出结果 4.0

```
import numpy as np

arr = np.array([[1., 2., 3.], [4., 5., 6.]])
print arr
'''
[[ 1.  2.  3.]
 [ 4.  5.  6.]]
'''

# 数组相加
print arr + arr
'''
[[  2.   4.   6.]
 [  8.  10.  12.]]
'''

# 数组相减
print arr - arr
'''
[[ 0.  0.  0.]
 [ 0.  0.  0.]]
'''

# 用1除以每个数组元素，返回数组
print 1 / arr
'''
[[ 1.          0.5         0.33333333]
 [ 0.25        0.2         0.16666667]]
'''

# 对arr数组里的每个元素都进行0.5次方运算
print arr ** 0.5
'''
[[ 1.          1.41421356  1.73205081]
 [ 2.          2.23606798  2.44948974]]
'''
```


## 一维数组的索引与切片
```
import numpy as np

a = np.arange(9)

# : 冒号表示从...到...，如果只写了冒号，没有写数字，表示从头到尾
# 取3到7的数
# 第一组
print a[3:7]
'''
[3 4 5 6]
'''

# 取 从0到7 每间隔为2的数
# 第二组
print a[:7:2]
'''
[0 2 4 6]
'''

# 对一维数组进行倒序
# 第三组
print a[::-1]
'''
[8 7 6 5 4 3 2 1 0]
'''

# 跟第一组一样
s = slice(3, 7)
print a[s]

# 跟第二组一样
s = slice(None, 7, 2)
print a[s]

# 跟第三组一样
s = slice(None, None, -1)
print a[s]

```


## 多维数组的切片与索引
```
import numpy as np

b = np.arange(24).reshape(2,3,4)
print b.shape
print b
'''
[[[ 0  1  2  3]
  [ 4  5  6  7]
  [ 8  9 10 11]]

 [[12 13 14 15]
  [16 17 18 19]
  [20 21 22 23]]]
'''

# 取第一维度的第0行第0列的值
print b[0,0,0]

# 取第一维度和第二维度的第0行的第0列的值
print b[:, 0, 0]   # [ 0, 12 ]

# 取第0维度上的所有元素的值
print b[0, :, :]
# 也可以使用... 三个点表示后面的都选上
print b[0, ...]
'''
[[ 0  1  2  3]
 [ 4  5  6  7]
 [ 8  9 10 11]]
'''

# 取第0维度的索引为1的那一行的所有元素的值
print b[0, 1]
'''
[4 5 6 7]
'''

# 取第0维度上的索引为1的那一行上间隔为2的所有值
print b[0, 1, ::2]
'''
[4 6]
'''

# 取第0维度和第1维度上的索引为1的那一列的所有的值
print b[..., 1]
'''
[[ 1  5  9]
 [13 17 21]]
'''

# 取第0维度和第1维度上索引为1的那一行的所有的值
print b[:, 1]
'''
[[ 4  5  6  7]
 [16 17 18 19]]
'''

# 取第0维度上所有行的第索引为1列的所有的值
print b[0, :, 1]
'''
[1 5 9]
'''

# 取第0维度上所有行的倒数第一列的所有的值
print b[0, :, -1]
'''
[ 3  7 11]
'''

# 对第0维度上所有的行倒序，然后再根据倒序的结果取倒数第一列的值
print b[0, ::-1, -1]
'''
[11  7  3]
'''

# 取第0维度上每间隔为2行的数，再取倒数第一列的值
print b[0, ::2, -1]
'''
[ 3 11]
'''

# 对这两个矩阵反转
print b[::-1]
'''
[[[12 13 14 15]
  [16 17 18 19]
  [20 21 22 23]]

 [[ 0  1  2  3]
  [ 4  5  6  7]
  [ 8  9 10 11]]]
'''

# 我们也可以使用slice切片来取值
# 以下是反转两个矩阵，也就是把这两个矩阵换个索引位置
s = slice(None, None, -1)
print b[s]
'''
[[[12 13 14 15]
  [16 17 18 19]
  [20 21 22 23]]

 [[ 0  1  2  3]
  [ 4  5  6  7]
  [ 8  9 10 11]]]
'''

# 如果我们需要:
# 1.把两个矩阵反转
# 2.把每个矩阵的所有行反转
# 3.把每个矩阵的每一行的所有元素反转
print b[s, s, s]
'''
[[[23 22 21 20]
  [19 18 17 16]
  [15 14 13 12]]

 [[11 10  9  8]
  [ 7  6  5  4]
  [ 3  2  1  0]]]
'''
```


## 布尔型索引
```
import numpy as np

names = np.array(['Bob','Joe','Will','Victor','Dwayne'])

# 对names里每个元素和'Bob'比较，并返回一个数组
print names == 'Bob'
'''
[ True False False False False]
'''

print (names == 'Bob') | (names == 'Will')
'''
[ True False  True False False]
'''
```


## 花式索引
```
# coding: utf8
import numpy as np

# 创建一个8行乘以4列的数组
arr = np.empty((8, 4))
print arr
'''
[[ -3.10503618e+231   3.11107876e+231  -1.07161239e+217   2.19870404e-314]
 [  2.19870333e-314   0.00000000e+000   0.00000000e+000   0.00000000e+000]
 [  1.00253535e-307   2.19378191e-314   2.19435524e-314  -9.27386883e+283]
 [  2.19377210e-314   2.19871256e-314  -6.22760761e-002   2.19439352e-314]
 [  2.19446055e-314   2.09658325e-114   2.19858697e-314   6.92818888e-310]
 [  1.49578392e+091   2.19870406e-314   2.19870337e-314  -1.47948881e-261]
 [  2.19377200e-314   6.92819029e-310  -1.28433547e-117   2.19435531e-314]
 [  2.19434605e-314   8.34595430e-044   2.19870444e-314   2.22507606e-308]]
'''

# 将每一列的值改成0到8
for i in range(8):
    arr[i] = i
print arr
'''
[[ 0.  0.  0.  0.]
 [ 1.  1.  1.  1.]
 [ 2.  2.  2.  2.]
 [ 3.  3.  3.  3.]
 [ 4.  4.  4.  4.]
 [ 5.  5.  5.  5.]
 [ 6.  6.  6.  6.]
 [ 7.  7.  7.  7.]]
'''

# 选择索引为4，3，0，6行的值
print arr[[4, 3, 0, 6]]
'''
[[ 4.  4.  4.  4.]
 [ 3.  3.  3.  3.]
 [ 0.  0.  0.  0.]
 [ 6.  6.  6.  6.]]
'''

# 也可以通过倒序索引的形式选取行的数据
# 取倒序索引的-3, -5, -7的行的值
print arr[[-3, -5, -7]]
'''
[[ 5.  5.  5.  5.]
 [ 3.  3.  3.  3.]
 [ 1.  1.  1.  1.]]
'''

arr = np.arange(32).reshape((8, 4))
print arr
'''
[[ 0  1  2  3]
 [ 4  5  6  7]
 [ 8  9 10 11]
 [12 13 14 15]
 [16 17 18 19]
 [20 21 22 23]
 [24 25 26 27]
 [28 29 30 31]]
'''

# 取 arr[1][0], arr[5][3], arr[7][1], arr[2][2]
print arr[[1, 5, 7, 2], [0, 3, 1, 2]]
'''
[ 4 23 29 10]
'''

# 返回一个区域的值
print arr[[1, 5, 7, 2]][:, [0, 3, 1, 2]]
'''
[[ 4  7  5  6]
 [20 23 21 22]
 [28 31 29 30]
 [ 8 11  9 10]]
'''

# 跟上一个一样
print arr[np.ix_([1, 5, 7, 2], [0, 3, 1, 2])]
'''
[[ 4  7  5  6]
 [20 23 21 22]
 [28 31 29 30]
 [ 8 11  9 10]]
'''
```


## 数组转置
```
# coding: utf8
import numpy as np

arr = np.arange(15).reshape((3, 5))
print arr
'''
[[ 0  1  2  3  4]
 [ 5  6  7  8  9]
 [10 11 12 13 14]]
'''

# 3行5列转成5行三列
print arr.T
'''
[[ 0  5 10]
 [ 1  6 11]
 [ 2  7 12]
 [ 3  8 13]
 [ 4  9 14]]
'''
```


## 改变数组的维度
```
# coding: utf8
import numpy as np

# arange是生成一个一维数组
# reshape就可以改变数组的维度
b = np.arange(24).reshape((2,3,4))
print b
'''
[[[ 0  1  2  3]
  [ 4  5  6  7]
  [ 8  9 10 11]]

 [[12 13 14 15]
  [16 17 18 19]
  [20 21 22 23]]]
'''

# 把多维数组展开 成 一维数组
print b.flatten()
'''
[ 0  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 21 22 23]
'''

print b.ravel()
'''
[ 0  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 21 22 23]
'''

# 还可以将数组的shape改变，就是6行4列
b.shape = (6, 4)
print b
'''
[[ 0  1  2  3]
 [ 4  5  6  7]
 [ 8  9 10 11]
 [12 13 14 15]
 [16 17 18 19]
 [20 21 22 23]]
'''

# 转矩阵，可以把b这个多维数组，向反方向维度的转换，
# 比如: 上面的6行4列，转换后，就是4行6列
print b.transpose()
'''
[[ 0  4  8 12 16 20]
 [ 1  5  9 13 17 21]
 [ 2  6 10 14 18 22]
 [ 3  7 11 15 19 23]]
'''

# resize和reshape的功能是一样的
b.resize((2, 12))
print b
'''
[[ 0  1  2  3  4  5  6  7  8  9 10 11]
 [12 13 14 15 16 17 18 19 20 21 22 23]]
'''

# 进一步开transpose函数的反置换数组
print b.transpose()
'''
[[ 0 12]
 [ 1 13]
 [ 2 14]
 [ 3 15]
 [ 4 16]
 [ 5 17]
 [ 6 18]
 [ 7 19]
 [ 8 20]
 [ 9 21]
 [10 22]
 [11 23]]
'''
```


## 数组的分割
```
# coding=utf8

import numpy as np

a = np.arange(9).reshape(3, 3)
print a
'''
[[0 1 2]
 [3 4 5]
 [6 7 8]]
'''

b = 2 * a
print b
'''
[[ 0  2  4]
 [ 6  8 10]
 [12 14 16]]
'''

# 水平组合两个数组
print np.hstack((a, b))
'''
[[ 0  1  2  0  2  4]
 [ 3  4  5  6  8 10]
 [ 6  7  8 12 14 16]]
'''

# axis=1 表示水平，和上面的hstack一样
print np.concatenate((a, b), axis=1)
'''
[[ 0  1  2  0  2  4]
 [ 3  4  5  6  8 10]
 [ 6  7  8 12 14 16]]
'''

# 垂直组合两个数组
print np.vstack((a, b))
'''
[[ 0  1  2]
 [ 3  4  5]
 [ 6  7  8]
 [ 0  2  4]
 [ 6  8 10]
 [12 14 16]]
'''

# axis=0表示垂直，和上面的vstack一样
print np.concatenate((a, b), axis=0)
'''
[[ 0  1  2]
 [ 3  4  5]
 [ 6  7  8]
 [ 0  2  4]
 [ 6  8 10]
 [12 14 16]]
'''

# 深度组合，就是将多个矩阵平面点数据沿着纵轴合并（垂直合并）
print np.dstack((a, b))
'''
[[[ 0  0]
  [ 1  2]
  [ 2  4]]

 [[ 3  6]
  [ 4  8]
  [ 5 10]]

 [[ 6 12]
  [ 7 14]
  [ 8 16]]]
'''

# 这个方式和dstack是一样的，就是列垂直组合
one = np.arange(2)
two = one * 2
print np.column_stack((one, two))
'''
[[0 0]
 [1 2]]
'''

# 水平组合
print np.row_stack((one, two))
'''
[[0 1]
 [0 2]]
'''



a = np.arange(9).reshape(3, 3)
print a
'''
[[0 1 2]
 [3 4 5]
 [6 7 8]]
'''

# 水平分割 成 三份相同大小的数组
print np.hsplit(a, 3)
'''
[array([[0],
       [3],
       [6]]), 
 array([[1],
       [4],
       [7]]), 
 array([[2],
       [5],
       [8]])]
'''

# 垂直分割
print np.vsplit(a, 3)
'''
[array([[0, 1, 2]]), array([[3, 4, 5]]), array([[6, 7, 8]])]
'''

# 或者还可以使用split函数，并且指定axis

# 水平分割
print np.split(a, 3, axis=1)
'''
[array([[0],
       [3],
       [6]]), 
 array([[1],
       [4],
       [7]]), 
 array([[2],
       [5],
       [8]])]
'''

# 垂直分割
print np.split(a, 3, axis=0)
'''
[array([[0, 1, 2]]), array([[3, 4, 5]]), array([[6, 7, 8]])]
'''

# 深度分割
c = np.arange(27).reshape((3, 3, 3))
print c
'''
[[[ 0  1  2]
  [ 3  4  5]
  [ 6  7  8]]

 [[ 9 10 11]
  [12 13 14]
  [15 16 17]]

 [[18 19 20]
  [21 22 23]
  [24 25 26]]]
'''

print np.dsplit(c, 3)
'''
[array([[[ 0],
        [ 3],
        [ 6]],

       [[ 9],
        [12],
        [15]],

       [[18],
        [21],
        [24]]]), array([[[ 1],
        [ 4],
        [ 7]],

       [[10],
        [13],
        [16]],

       [[19],
        [22],
        [25]]]), array([[[ 2],
        [ 5],
        [ 8]],

       [[11],
        [14],
        [17]],

       [[20],
        [23],
        [26]]])]
'''
```


## 数组的属性
```
# coding=utf8
import numpy as np

b = np.arange(24).reshape((2, 12))
print b.ndim    # 数组的维度
print b.size    # 数组整个大小
print b.itemsize # 数组的元素大小
print b.nbytes   # 数组总占用字节数

b = np.array([ 1.+1.j, 3.+2.j])
print b.real    # 数组的实步
print b.imag    # 数组的虚步
print b.flat    #  <numpy.flatiter object at 0x7ffa960f4000>
print b.flat[1] # (3+2j)

b.flat[1] = 700   # 通过flat修改元素的值
print b
```


## 数组的转换
```
# coding=utf8
import numpy as np

b = np.array([ 1.+1.j, 3.+2.j ])
print b
'''
[ 1.+1.j  3.+2.j]
'''

# 转成python的list
print b.tolist()
'''
[(1+1j), (3+2j)]
'''

# 打印一堆乱码，因为数组里的元素不是字符串
print b.tostring()

# 使用冒号分割，然后把分割完的值转成int类型
print np.fromstring('20:42:52', sep=':', dtype=int)
'''
[20 42 52]
'''

# 把数组b里的值都转成int类型
print b.astype(int)
'''
[1 3]
'''
```


## NumPy的一些通用函数
通用函数（ufunc）是一种对ndarray中的数据执行元素级运算的函数
- 一元ufunc
  abs, fabs, sqrt, exp, modf等等
- 二元ufunc
  
对于一元函数和二元函数，以下就列举一些例子，它的更多用法就去百度搜一下，有很多文章讲解，也很简单
如果使用google，就搜索：一元函数（unary function），二元函数（binary function）

```
# coding=utf8

import numpy as np
from numpy.random import randn

arr = np.arange(10)
print arr
'''
[0 1 2 3 4 5 6 7 8 9]
'''

# 分别生成八个随机数，返回数组
x = randn(8)
y = randn(8)
print "x=", x
print "y=", y
'''
x= [-0.54176858  2.66447488  0.69771148 -1.62758206 -0.83304403 -0.52295269
  0.57753982 -1.64298273]
y= [ 2.31025497  0.22378718 -1.61274711 -1.04259512  0.67085879 -0.33534232
 -0.66599697  0.331158  ]
'''

# 元素级最大值
print np.maximum(x, y)
'''
[ 2.31025497  2.66447488  0.69771148 -1.04259512  0.67085879 -0.33534232
  0.57753982  0.331158  ]
'''

arr = randn(7) * 5
print arr
'''
[ 3.40391963  1.50692787  5.17600018 -0.89995448 -5.40648018  6.2721975
  0.21843185]
'''

# 将数组的整数部分和小数部分分开来，作为两个数组返回
print np.modf(arr)
'''
(array([ 0.40391963,  0.50692787,  0.17600018, -0.89995448, -0.40648018,
        0.2721975 ,  0.21843185]), array([ 3.,  1.,  5., -0., -5.,  6.,  0.]))
'''
```



## 利用数组进行数据处理
## 向量化的处理
```
# coding=utf8

import numpy as np
from numpy.random import randn

# 从-5到5，每个值间隔0.1
points = np.arange(-5, 5, 0.1)
print points
'''
[ -5.00000000e+00  -4.90000000e+00  -4.80000000e+00  -4.70000000e+00
  -4.60000000e+00  -4.50000000e+00  -4.40000000e+00  -4.30000000e+00
  -4.20000000e+00  -4.10000000e+00  -4.00000000e+00  -3.90000000e+00
  -3.80000000e+00  -3.70000000e+00  -3.60000000e+00  -3.50000000e+00
  -3.40000000e+00  -3.30000000e+00  -3.20000000e+00  -3.10000000e+00
  -3.00000000e+00  -2.90000000e+00  -2.80000000e+00  -2.70000000e+00
  -2.60000000e+00  -2.50000000e+00  -2.40000000e+00  -2.30000000e+00
  -2.20000000e+00  -2.10000000e+00  -2.00000000e+00  -1.90000000e+00
  -1.80000000e+00  -1.70000000e+00  -1.60000000e+00  -1.50000000e+00
  -1.40000000e+00  -1.30000000e+00  -1.20000000e+00  -1.10000000e+00
  -1.00000000e+00  -9.00000000e-01  -8.00000000e-01  -7.00000000e-01
  -6.00000000e-01  -5.00000000e-01  -4.00000000e-01  -3.00000000e-01
  -2.00000000e-01  -1.00000000e-01  -1.77635684e-14   1.00000000e-01
   2.00000000e-01   3.00000000e-01   4.00000000e-01   5.00000000e-01
   6.00000000e-01   7.00000000e-01   8.00000000e-01   9.00000000e-01
   1.00000000e+00   1.10000000e+00   1.20000000e+00   1.30000000e+00
   1.40000000e+00   1.50000000e+00   1.60000000e+00   1.70000000e+00
   1.80000000e+00   1.90000000e+00   2.00000000e+00   2.10000000e+00
   2.20000000e+00   2.30000000e+00   2.40000000e+00   2.50000000e+00
   2.60000000e+00   2.70000000e+00   2.80000000e+00   2.90000000e+00
   3.00000000e+00   3.10000000e+00   3.20000000e+00   3.30000000e+00
   3.40000000e+00   3.50000000e+00   3.60000000e+00   3.70000000e+00
   3.80000000e+00   3.90000000e+00   4.00000000e+00   4.10000000e+00
   4.20000000e+00   4.30000000e+00   4.40000000e+00   4.50000000e+00
   4.60000000e+00   4.70000000e+00   4.80000000e+00   4.90000000e+00]
'''

xs, ys = np.meshgrid(points, points)
print xs
'''
[[-5.  -4.9 -4.8 ...,  4.7  4.8  4.9]
 [-5.  -4.9 -4.8 ...,  4.7  4.8  4.9]
 [-5.  -4.9 -4.8 ...,  4.7  4.8  4.9]
 ..., 
 [-5.  -4.9 -4.8 ...,  4.7  4.8  4.9]
 [-5.  -4.9 -4.8 ...,  4.7  4.8  4.9]
 [-5.  -4.9 -4.8 ...,  4.7  4.8  4.9]]
'''

print ys
'''
[[-5.  -5.  -5.  ..., -5.  -5.  -5. ]
 [-4.9 -4.9 -4.9 ..., -4.9 -4.9 -4.9]
 [-4.8 -4.8 -4.8 ..., -4.8 -4.8 -4.8]
 ..., 
 [ 4.7  4.7  4.7 ...,  4.7  4.7  4.7]
 [ 4.8  4.8  4.8 ...,  4.8  4.8  4.8]
 [ 4.9  4.9  4.9 ...,  4.9  4.9  4.9]]
'''

import matplotlib.pyplot as plt
z = np.sqrt(xs ** 2 + ys ** 2)
print z

plt.imshow(z, cmap=plt.cm.gray)
plt.colorbar()
plt.title("image plot of $\sqrt{x^2 + y^2}$ for a grid of values")
plt.draw()
```


# 将条件逻辑表达为数组运算
```
# coding=utf8

import numpy as np
from numpy.random import randn

xarr = np.array([1.1, 1.2, 1.3, 1.4, 1.5])
yarr = np.array([2.1, 2.2, 2.3, 2.4, 2.5])
cond = np.array([True, False, True, True, False])

# 列表推导式
# 如果c等于True，则执行x，否则执行y
result = [(x if c else y)
 for x, y, c in zip(xarr, yarr, cond)]
# zip函数的意思是将三个参数作为一个数组，元素是tuple类型返回

print result
'''
[1.1000000000000001, 2.2000000000000002, 1.3, 1.3999999999999999, 2.5]
'''

# 上面的列表推导式 当数据量大时，性能不高，因为它是纯python的，所以numpy就有一个函数，高性能的处理这样的问题
result = np.where(cond, xarr, yarr)
print result
'''
[ 1.1  2.2  1.3  1.4  2.5]
'''

# 生成一个4乘以4的随机数数组
arr_rand = randn(4, 4)
print arr_rand
'''
[[-1.75446035  0.55942022  0.03802657  1.17424156]
 [-0.52429608 -0.08689923  0.00603974  1.96522324]
 [ 0.68151676 -0.39275041  0.14206361 -1.24419373]
 [ 1.41926654 -0.52600884 -0.04940465 -0.37086705]]
'''

# 然后我们使用where函数
# 数组里大于0的元素时，填充2，反之，填充-2
print np.where(arr_rand > 0, 2, -2)
'''
[[-2  2  2  2]
 [-2 -2  2  2]
 [ 2 -2  2 -2]
 [ 2 -2 -2 -2]]
'''

# 数组里的元素大于0时，填充2，反之，填充它本身的值
print np.where(arr_rand > 0, 2, arr_rand)
'''
[[-1.75446035  2.          2.          2.        ]
 [-0.52429608 -0.08689923  2.          2.        ]
 [ 2.         -0.39275041  2.         -1.24419373]
 [ 2.         -0.52600884 -0.04940465 -0.37086705]]
'''

# 如果没有where函数的，我们就需要执行for循环，还需要一些判断才行

# 来个复杂点的where使用
# np.where(cond1 & cond2, 0, np.where(cond1, 1, np.where(cond2, 2, 3)))
```


## 数学和统计方法
sum    求和
mean   求平均数
std    求标准值
var    求方差
min    求最小值
max    求最大值
argmin 求最小索引
argmax 求最大索引
cumsum 求所有元素的累计和
cumprod 求所有元素的累计积
```

arr = randn(5, 4)
print arr
'''
[[-0.69532575 -0.46587784  0.09154531  0.02780419]
 [ 1.65731101  0.99630076 -0.62135179 -0.73975346]
 [-0.16868977  0.50925681 -0.21328851  0.88459092]
 [-1.23530288  0.15287639 -0.61429954 -0.17145152]
 [-0.04142243  2.03859271  0.4526737   0.42605781]]
'''

print np.mean(arr)
'''
-0.0496905416252
'''

print np.sum(arr)
'''
6.03672937318
'''

print np.min(arr)
'''
-1.03827242655
'''

print np.max(arr)
'''
2.04400453423
'''


arr = np.array([[0,1,2], [3,4,5], [6,7,8]])
print arr.cumsum(0)
'''
[[ 0  1  2]
 [ 3  5  7]
 [ 9 12 15]]
'''

print arr.cumprod(1)
'''
[[  0   0   0]
 [  3  12  60]
 [  6  42 336]]
'''
```


## 用在布尔型数组的方法
```
# coding=utf8

import numpy as np
from numpy.random import randn

arr = randn(100)

# 对大于0的数求和
print (arr > 0).sum()

bools = np.array([False, False, True, False])

# 检查数组里，所有的元素中，只要有一个为False，或者为0，就返回True
print bools.any()

# 检查一个数组里，所有的元素都是True，或者都不等于0的情况下，才返回True
print bools.all()
```


## 排序
```
import numpy as np
from numpy.random import randn

arr = randn(8)
print arr

# 排序数组
print arr.sort()

names = np.array(['Bob', 'Zoe', 'Will', 'Bob', 'Will'])
print names

# 排重数据，然后排序
print sorted(set(names))

# 唯一化，就是去重
print np.unique(names)

# in1d函数意思是，第二个数组里的元素是否在第一个数组里，返回一组数组，True或者False
print np.in1d(names, [ 'Will' ])
'''
[False False  True False  True]
'''
```


## 线性代数
常用的numpy.linalg函数

diag   以一维数组形式返回方阵的对角线（或非对角线）元素，或将一维数组转换为方阵
dot    矩阵乘法
trace  计算对角线元素的和
det    计算矩阵行列式
eig    计算方阵的本征值和本征向量
inv    计算方阵的逆
pinv   计算矩阵的Moore-Penrose伪逆
qr     计算QR分解
svd    计算奇异值分解SVD
solve  解线性方程组Ax = b，其中A为一个方阵
lstsq  计算Ax = b的最小二乘解


## 随机数生成
在numpy.random
seed         确定随机数生成的种子
permutation  返回一个序列的随机排列或返回一个随机排列的范围
shuffle      对一个序列就地随机排列
rand         产生均匀分布的样本值
randint      从给定的上下限范围内随机选取整数
randn        产生正态分布（平均值为0，标准差为1）的样本值，类似于Matlab接口
binomial     产生二项分布的样本值
normal       产生正态（高斯）分布的样本值
beta         产生beta分布的样本值


```
samples = np.random.normal(size=(4, 4))
print samples

import random

# 随机漫步
position = 0
walk = [position]
steps = 1000
for i in xrange(steps):
    step = 1 if random.randint(0, 1) else -1
    position += step
    walk.append(position)
```



## 分析股票数据的案例
```
# coding: utf8

import sys
import numpy as np

# 读取股票文件
# 第一个参数：文件路径
# 第二个参数：分隔符
# 第三个参数：选取的列，这个例子选中第6列和第7列，返回tuple类型，对应每列
# 第四个参数：如名字unpack，就是解包后是否tranpose，true则是
sixth, seventh = np.loadtxt('Apple_Partion_StockPrices.csv', delimiter=',', usecols=(6, 7), unpack=True)
print seventh

# 计算成交量加权平均价格
vwap = np.average(sixth, weights=seventh)
print vwap
'''
350.589549353
'''

# 算数平均值函数
print "mean =", np.mean(sixth)
'''
mean = 351.037666667
'''

# 时间加权平均价格
t = np.arange(len(sixth))
print np.average(sixth, weights=t)
'''
352.428321839
'''


# 寻找最大值和最小值
h, l = np.loadtxt('Apple_Partion_StockPrices.csv', delimiter=',', usecols=(4,5), unpack=True)
print "max = ", np.max(h)
print "min = ", np.min(l)
print "最高价和最低价中间的值 = ", (np.max(h) + np.min(l)) / 2
'''
max =  364.9
min =  333.53
最高价和最低价中间的值 =  349.215
'''

print "Spread High price = ", np.ptp(h)
print "Spread Low Price = ", np.ptp(l)
'''
Spread High price =  24.86
Spread Low Price =  26.97
'''


# 统计分析
c = np.loadtxt('Apple_Partion_StockPrices.csv', delimiter=',', usecols=(6,), unpack=True)
print "中位值 = ", np.median(c)
'''
median =  352.055
'''

print np.msort(c)
'''
[ 336.1   338.61  339.32  342.62  342.88  343.44  344.32  345.03  346.5
  346.67  348.16  349.31  350.56  351.88  351.99  352.12  352.47  353.21
  354.54  355.2   355.36  355.76  356.85  358.16  358.3   359.18  359.56
  359.9   360.    363.13]
'''

# 方差
print np.mean((c - c.mean()) ** 2)
'''
50.1265178889
'''

# 股票收益率
c = np.loadtxt('Apple_Partion_StockPrices.csv', delimiter=',', usecols=(6,), unpack=True)

# 标准收益
returns = np.diff(c) / c[:-1]
print np.std(returns)
'''
0.0129221344368
'''

# 对数收益
logreturns = np.diff(np.log(c))
print logreturns
'''
[ 0.00953488  0.01668775 -0.00205991 -0.00255903  0.00887039  0.01540739
  0.0093908   0.0082988  -0.01015864  0.00649435  0.00650813  0.00200256
  0.00893468 -0.01339027 -0.02183875 -0.03468287  0.01177296  0.00075857
  0.01528161  0.01440064 -0.011103    0.00801225  0.02090904  0.00122297
 -0.01297267  0.00112499 -0.00929083 -0.01659219  0.01522945]
'''


# 日期分析
# Monday = 0
# Tuesday = 1
# Wednesday = 2
# Thursday = 3
# Friday = 4
# Saturday = 5
# Sunday = 6
from datetime import datetime

def datestr2num(s):
    return datetime.strptime(s, "%d-%m-%Y").date().weekday()

dates, close = np.loadtxt('Apple_Partion_StockPrices.csv', delimiter=',', usecols=(1, 6), converters={1: datestr2num}, unpack=True)
print dates
'''
[ 4.  0.  1.  2.  3.  4.  0.  1.  2.  3.  4.  0.  1.  2.  3.  4.  1.  2.
  3.  4.  0.  1.  2.  3.  4.  0.  1.  2.  3.  4.]
'''

averages = np.zeros(5)

for i in range(5):
    indices = np.where(dates == i)
    prices = np.take(close, indices)
    avg = np.mean(prices)
    print "Day = ", i, ", Prices = ", prices, ", Average = ", avg
'''
Day =  0 , Prices =  [[ 339.32  351.88  359.18  353.21  355.36]] , Average =  351.79
Day =  1 , Prices =  [[ 345.03  355.2   359.9   338.61  349.31  355.76]] , Average =  350.635
Day =  2 , Prices =  [[ 344.32  358.16  363.13  342.62  352.12  352.47]] , Average =  352.136666667
Day =  3 , Prices =  [[ 343.44  354.54  358.3   342.88  359.56  346.67]] , Average =  350.898333333
Day =  4 , Prices =  [[ 336.1   346.5   356.85  350.56  348.16  360.    351.99]] , Average =  350.022857143
'''
```

<a href="/files/csv/Apple_Partion_StockPrices.csv">Apple_Partion_StockPrices.csv 文件下载地址</a>



<br/>
<br/>
<br/>

## References
- http://www.googleplus.party/categories/AI/
- https://zhuanlan.zhihu.com/p/24309547

