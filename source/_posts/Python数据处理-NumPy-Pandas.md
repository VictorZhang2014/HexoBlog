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




本篇教程使用的所有资源文件下载地址：
- [catering_sale.xls](/files/data_screening_files/catering_sale.xls)
- [electricity_data.xls](/files/data_screening_files/electricity_data.xls)
- [foods-2011-10-03.json](/files/data_screening_files/foods-2011-10-03.json)
- [macrodata.csv](/files/data_screening_files/macrodata.csv)
- [movies.dat](/files/data_screening_files/movies.dat)
- [normalization_data.xls](/files/data_screening_files/normalization_data.xls)
- [olivier.txt](/files/data_screening_files/olivier.txt)
- [principal_component.xls](/files/data_screening_files/principal_component.xls)






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



### 如果你遇到报错"A value is trying to be set on a copy of a slice from a DataFrame"
### 解决方法是：pd.options.mode.chained_assignment = None 加到顶部声明处 
[解决方法](https://stackoverflow.com/questions/20625582/how-to-deal-with-settingwithcopywarning-in-pandas)

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
pd.options.mode.chained_assignment = None


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

# 如果你遇到报错"A value is trying to be set on a copy of a slice from a DataFrame"
# 解决方法是：pd.options.mode.chained_assignment = None 加到顶部声明处
data[u'销量'][(data[u'销量'] < 400) | (data[u'销量'] > 5000)] = None
print(data)
exit()

# 自定义列向量插值函数
# s为列向量，n为被插值的位置，k为取前后的数据个数，默认为5
def ployinterp_column(s, n, k=5):
    y = s[list(range(n-k, n)) + list(range(n+1, n+1+k))] # 取数
    y = y[y.notnull()] # 剔除空值
    return lagrange(y.index, list(y))(n) # 插值并返回插值结果

# 逐个元素判断是否需要插值
for i in data.columns:
    for j in range(len(data)):
        if (data[i].isnull())[j]: # 如果为空即插入值
            data[i][j] = ployinterp_column(data[i], j)

# 写入到硬盘
data.to_excel(outputfile)
```

对出缺失值的问题，也可以忽略掉不处理，只不过会影响分析结果




## 合并数据
Pandas对象
- Merge方法：根据一个或者多个键将不同DataFrame中的行合并
- Concat方法：沿一条轴将对多个对象堆叠起来

```
# coding: utf-8

from __future__ import division
import numpy as np
import pandas as pd
from pandas import DataFrame



df1 = DataFrame({"key": ['b', 'b', 'a', 'c', 'a', 'a', 'b'],
                 "data1": range(7) })
df2 = DataFrame({"key": ['a', 'b', 'd'],
                 "data2": range(3)})

print df1
'''
   data1 key
0      0   b
1      1   b
2      2   a
3      3   c
4      4   a
5      5   a
6      6   b
'''

print df2
'''
   data2 key
0      0   a
1      1   b
2      2   d
'''

# 合并，重叠的列名当做键
print pd.merge(df1, df2)
'''
   data1 key  data2
0      0   b      1
1      1   b      1
2      6   b      1
3      2   a      0
4      4   a      0
5      5   a      0
'''

# 合并
print pd.merge(df1, df2, on="key")
'''
   data1 key  data2
0      0   b      1
1      1   b      1
2      6   b      1
3      2   a      0
4      4   a      0
5      5   a      0
'''

# 第二个例子
df3 = DataFrame({"lkey": ['b', 'b', 'a', 'c', 'a', 'a', 'b'],
                 "data1": range(7) })
df4 = DataFrame({"rkey": ['a', 'b', 'd'],
                 "data2": range(3)})

print pd.merge(df3, df4, left_on='lkey', right_on='rkey')
'''
   data1 lkey  data2 rkey
0      0    b      1    b
1      1    b      1    b
2      6    b      1    b
3      2    a      0    a
4      4    a      0    a
5      5    a      0    a
'''

# 外链接
print pd.merge(df1, df2, how='outer')
'''
   data1 key  data2
0    0.0   b    1.0
1    1.0   b    1.0
2    6.0   b    1.0
3    2.0   a    0.0
4    4.0   a    0.0
5    5.0   a    0.0
6    3.0   c    NaN
7    NaN   d    2.0
'''

# 左连接
print pd.merge(df1, df2, how='left')
'''
   data1 key  data2
0      0   b    1.0
1      1   b    1.0
2      2   a    0.0
3      3   c    NaN
4      4   a    0.0
5      5   a    0.0
6      6   b    1.0
'''

# 内连接
print pd.merge(df1, df2, how='inner')
'''
   data1 key  data2
0      0   b      1
1      1   b      1
2      6   b      1
3      2   a      0
4      4   a      0
5      5   a      0
'''

# coding: utf-8

import numpy as np
import pandas as pd
from pandas import DataFrame

left = DataFrame({"key": ['a','b','a','a','b','c'],
                  "value": range(6)})
right = DataFrame({"group_val": [3.5, 7]}, index=['a', 'b'])

print left
'''
  key  value
0   a      0
1   b      1
2   a      2
3   a      3
4   b      4
5   c      5
'''

print right
'''
   group_val
a        3.5
b        7.0
'''

print pd.merge(left, right, left_on='key', right_index=True)
'''
  key  value  group_val
0   a      0        3.5
2   a      2        3.5
3   a      3        3.5
1   b      1        7.0
4   b      4        7.0
'''

print pd.merge(left, right, left_on='key', right_index=True, how='outer')
'''
  key  value  group_val
0   a      0        3.5
2   a      2        3.5
3   a      3        3.5
1   b      1        7.0
4   b      4        7.0
5   c      5        NaN
'''

print left.join(right, how='outer')
'''
   key  value  group_val
0    a    0.0        NaN
1    b    1.0        NaN
2    a    2.0        NaN
3    a    3.0        NaN
4    b    4.0        NaN
5    c    5.0        NaN
a  NaN    NaN        3.5
b  NaN    NaN        7.0
'''

```


## 轴向连接
```
# coding: utf-8

import numpy as np
import pandas as pd
from pandas import Series, DataFrame


arr = np.arange(12).reshape((3, 4))

# 轴向连接
print np.concatenate([arr, arr], axis=1)
'''
[[ 0  1  2  3  0  1  2  3]
 [ 4  5  6  7  4  5  6  7]
 [ 8  9 10 11  8  9 10 11]]
'''

s1 = Series([0, 1], index=['a', 'b'])
s2 = Series([2, 3, 4], index=['c', 'd', 'e'])
s3 = Series([5, 6], index=['f', 'g'])

print pd.concat([s1, s2, s3])
'''
a    0
b    1
c    2
d    3
e    4
f    5
g    6
'''

print pd.concat([s1, s2, s3], axis=1)
'''
dtype: int64
     0    1    2
a  0.0  NaN  NaN
b  1.0  NaN  NaN
c  NaN  2.0  NaN
d  NaN  3.0  NaN
e  NaN  4.0  NaN
f  NaN  NaN  5.0
g  NaN  NaN  6.0
'''

print pd.concat([s1 * 5, s3])
'''
     0    1    2
a  0.0  NaN  NaN
b  1.0  NaN  NaN
c  NaN  2.0  NaN
d  NaN  3.0  NaN
e  NaN  4.0  NaN
f  NaN  NaN  5.0
g  NaN  NaN  6.0
a    0
b    5
f    5
g    6
'''

print pd.concat([s1, s2, s3], keys=['one', 'two', 'three'])
'''
one    a    0
       b    1
two    c    2
       d    3
       e    4
three  f    5
       g    6
'''

```


## 合并重叠数据
```
# coding: utf-8

import numpy as np
import pandas as pd
from pandas import Series, DataFrame

a = Series([np.nan, 2.5, np.nan, 3.5, 4.5, np.nan],
           index=['f','e','d','c','b','a'])
b = Series(np.arange(len(a), dtype=np.float64),
           index=['f','e','d','c','b','a'])

print a
'''
f    NaN
e    2.5
d    NaN
c    3.5
b    4.5
a    NaN
dtype: float64
'''

# 将倒数第一个数修改为NaN
b[-1] = np.nan
print b
'''
f    0.0
e    1.0
d    2.0
c    3.0
b    4.0
a    NaN
dtype: float64
'''

# 对a里的值进行isnull操作，如果是null，就使用b，否则使用a
c = np.where(pd.isnull(a), b, a)
print c
'''
[ 0.   2.5  2.   3.5  4.5  nan]
'''

# 如果一个数组里的元素缺失了，就用另一个数组里的值，反之也一样，如果都空了，那就是空的
print a.combine_first(b)
'''
f    0.0
e    2.5
d    2.0
c    3.5
b    4.5
a    NaN
'''
```




## 重塑和轴向旋转
- stack      行转列
- unstack    列转行

```
# coding: utf-8

import numpy as np
import pandas as pd
from pandas import Series, DataFrame

data = DataFrame(np.arange(6).reshape((2, 3)),
                 index=pd.Index(['Ohio', 'Colorado'], name='state'),
                 columns=pd.Index(['one', 'two', 'three'], name='number'))
print data
'''
number    one  two  three
state                    
Ohio        0    1      2
Colorado    3    4      5
'''

# 行转列
result = data.stack()
print result
'''
state     number
Ohio      one       0
          two       1
          three     2
Colorado  one       3
          two       4
          three     5
'''

# 列转行
print result.unstack()
'''
number    one  two  three
state                    
Ohio        0    1      2
Colorado    3    4      5
'''

print result.unstack('state')
'''
state   Ohio  Colorado
number                
one        0         3
two        1         4
three      2         5
'''
```




## 数据转换
- 删除重复数据


```
# coding: utf-8

import numpy as np
import pandas as pd
from pandas import Series, DataFrame

data = DataFrame({"k1": ['one'] * 3 + ['two'] * 4,
                  "k2": [1, 1, 2, 3, 3, 4, 4]})
print data
'''
    k1  k2
0  one   1
1  one   1
2  one   2
3  two   3
4  two   3
5  two   4
6  two   4
'''

# 查看重复数据
print data.duplicated()
'''
0    False
1     True
2    False
3    False
4     True
5    False
6     True
'''

# 删除重复数据
data.drop_duplicates()
print data
'''
    k1  k2
0  one   1
1  one   1
2  one   2
3  two   3
4  two   3
5  two   4
6  two   4
'''

# 添加一列
data['v1'] = range(7)
print data
'''
    k1  k2  v1
0  one   1   0
1  one   1   1
2  one   2   2
3  two   3   3
4  two   3   4
5  two   4   5
6  two   4   6
'''

# 删除一列
data.drop_duplicates(['k1'])
print data
'''
    k1  k2  v1
0  one   1   0
1  one   1   1
2  one   2   2
3  two   3   3
4  two   3   4
5  two   4   5
6  two   4   6
'''
```


## 利用函数或映射进行数据转换
```
# coding: utf-8

import numpy as np
import pandas as pd
from pandas import Series, DataFrame

data = DataFrame({'food': ['bacon', 'pastrami', 'nova lox'],
                  'ounces': [4, 2, 8]})
print data
'''
       food  ounces
0     bacon       4
1  pastrami       2
2  nova lox       8
'''

meat_to_animal = {
    'bacon': 'pig',
    'pastrami': 'cow',
    'nova lox': 'salmon'
}

# 全部转成小写，使用map
data['animal'] = data['food'].map(str.lower)
print data
'''
       food  ounces    animal
0     bacon       4     bacon
1  pastrami       2  pastrami
2  nova lox       8  nova lox
'''

# 然后将对应的类型添加到data中
data['animal'] = data['food'].map(str.lower).map(meat_to_animal)
print data
'''
       food  ounces  animal
0     bacon       4     pig
1  pastrami       2     cow
2  nova lox       8  salmon
'''

# 通过lambda这样的表达式也可以完成和上面一样的效果
data['food'].map(lambda x: meat_to_animal[x.lower()])
print data
'''
       food  ounces  animal
0     bacon       4     pig
1  pastrami       2     cow
2  nova lox       8  salmon
'''

```



## 数据标准化
```
# coding: utf-8

import numpy as np
import pandas as pd
from pandas import Series, DataFrame


datafile = 'data_screening_files/normalization_data.xls'

# 读取数据
data = pd.read_excel(datafile, header=None)
print "原始数据："
print data
'''
原始数据：
     0    1    2     3
0   78  521  602  2863
1  144 -600 -521  2245
2   95 -457  468 -1283
3   69  596  695  1054
4  190  527  691  2051
5  101  403  470  2487
6  146  413  435  2571
'''

# 最小最大规范化
print (data - data.min()) / (data.max() - data.min())
'''
          0         1         2         3
0  0.074380  0.937291  0.923520  1.000000
1  0.619835  0.000000  0.000000  0.850941
2  0.214876  0.119565  0.813322  0.000000
3  0.000000  1.000000  1.000000  0.563676
4  1.000000  0.942308  0.996711  0.804149
5  0.264463  0.838629  0.814967  0.909310
6  0.636364  0.846990  0.786184  0.929571
'''

# 零-均值规范化
print (data - data.mean()) / data.std()
'''
          0         1         2         3
0 -0.905383  0.635863  0.464531  0.798149
1  0.604678 -1.587675 -2.193167  0.369390
2 -0.516428 -1.304030  0.147406 -2.078279
3 -1.111301  0.784628  0.684625 -0.456906
4  1.657146  0.647765  0.675159  0.234796
5 -0.379150  0.401807  0.152139  0.537286
6  0.650438  0.421642  0.069308  0.595564
'''

# 小数定标规范化
print data / 10 ** np.ceil(np.log10(data.abs().max()))
'''
       0      1      2       3
0  0.078  0.521  0.602  0.2863
1  0.144 -0.600 -0.521  0.2245
2  0.095 -0.457  0.468 -0.1283
3  0.069  0.596  0.695  0.1054
4  0.190  0.527  0.691  0.2051
5  0.101  0.403  0.470  0.2487
6  0.146  0.413  0.435  0.2571
'''
```


## 替换值
```
# coding: utf-8

import numpy as np
import pandas as pd
from pandas import Series, DataFrame


data = Series([1., -999., 2., -999., -1000., 3.])
print data
'''
0       1.0
1    -999.0
2       2.0
3    -999.0
4   -1000.0
5       3.0
dtype: float64
'''

# 替换-999的值为NaN
data = data.replace(-999, np.nan)
print data
'''
0       1.0
1       NaN
2       2.0
3       NaN
4   -1000.0
5       3.0
dtype: float64
'''

# 替换两个值都为NaN
data = data.replace([-999, -1000], np.nan)
print data
'''
0    1.0
1    NaN
2    2.0
3    NaN
4    NaN
5    3.0
dtype: float64
'''

# 如果我们需要替换多种值为指定的值
data = data.replace({-999: np.nan, -1000: 0})
print data
'''
0    1.0
1    NaN
2    2.0
3    NaN
4    NaN
5    3.0
dtype: float64
'''
```


## 重命名轴索引
```
# coding: utf-8

import numpy as np
import pandas as pd
from pandas import Series, DataFrame


data = DataFrame(np.arange(12).reshape((3, 4)),
                 index=['Ohio', 'Colorado', 'New York'],
                 columns=['one', 'two', 'three', 'four'])
print data
'''
          one  two  three  four
Ohio        0    1      2     3
Colorado    4    5      6     7
New York    8    9     10    11
'''

# 将索引全部转为大写
print data.index.map(str.upper)
'''
Index([u'OHIO', u'COLORADO', u'NEW YORK'], dtype='object')
'''

# 将data的index里的值都换成大写
data.index = data.index.map(str.upper)
print data
'''
          one  two  three  four
OHIO        0    1      2     3
COLORADO    4    5      6     7
NEW YORK    8    9     10    11
'''

# 通过rename我们也可以完成相同的效果，把title都转成大写
print data.rename(index=str.title, columns=str.upper)
'''
          ONE  TWO  THREE  FOUR
Ohio        0    1      2     3
Colorado    4    5      6     7
New York    8    9     10    11
'''

# 其实rename就是重命名
data.rename(index={'OHIO': 'INDIANA'},
            columns={'three': 'peekaboo'})
print data
'''
          one  two  three  four
OHIO        0    1      2     3
COLORADO    4    5      6     7
NEW YORK    8    9     10    11
'''
```



## 离散化与面元划分
```
# coding: utf-8

import numpy as np
import pandas as pd
from pandas import Series, DataFrame

# 有这些年龄段
ages = [20, 22, 25, 27, 21, 23, 37, 61, 78]

# 根据这个条件分组
groups = [18, 25, 35, 60]

# 对数据进行分组
cats = pd.cut(ages, groups)
print cats
'''
[(18, 25], (18, 25], (18, 25], (25, 35], (18, 25], (18, 25], (35, 60], NaN, NaN]
Categories (3, interval[int64]): [(18, 25] < (25, 35] < (35, 60]]
'''

print cats.labels
'''
[ 0  0  0  1  0  0  2 -1 -1]
'''

# 统计一下个数，每个年龄阶段的个数有多少
print pd.value_counts(cats)
'''
(18, 25]    5
(35, 60]    1
(25, 35]    1
dtype: int64
'''


#group_names = ['Youth', 'YoungAdult', 'MiddleAge', 'Senior']
#print pd.cut(ages, groups, labels=group_names)
'''
报错：'Bin labels must be one fewer than '
ValueError: Bin labels must be one fewer than the number of bin edges
'''

# 长度为20的随机数组
data = np.random.rand(20)
print data
'''
[0.10734531 0.25739656 0.57500249 0.26341356 0.06755529 0.31844072
 0.40825376 0.63144798 0.34930756 0.28772671 0.82629089 0.4465668
 0.4369444  0.01694405 0.98986687 0.4619442  0.58291355 0.82963555
 0.75812264 0.75970419]
'''

# 将2个为一个阶段划分
print pd.cut(data, 4, precision=2)
'''
[(0.77, 0.99], (0.54, 0.77], (0.77, 0.99], (0.32, 0.54], (0.32, 0.54], ..., (0.77, 0.99], (0.77, 0.99], (0.094, 0.32], (0.77, 0.99], (0.77, 0.99]]
Length: 20
Categories (4, interval[float64]): [(0.094, 0.32] < (0.32, 0.54] < (0.54, 0.77] < (0.77, 0.99]]

'''


data = np.random.randn(1000)
cats = pd.qcut(data, 4)
print cats
'''
[(-3.936, -0.682], (-0.682, -0.0449], (-0.0449, 0.637], (-0.682, -0.0449], (-0.682, -0.0449], ..., (0.637, 3.161], (-0.682, -0.0449], (-0.0449, 0.637], (-0.0449, 0.637], (-0.682, -0.0449]]
Length: 1000
Categories (4, interval[float64]): [(-3.936, -0.682] < (-0.682, -0.0449] < (-0.0449, 0.637] <
                                    (0.637, 3.161]]
'''

# 统计一下
print pd.value_counts(cats)
'''
(0.635, 3.356]       250
(-0.0177, 0.635]     250
(-0.697, -0.0177]    250
(-2.877, -0.697]     250
dtype: int64
'''
```


## 检测和过滤异常值
```
# coding: utf-8

import numpy as np
import pandas as pd
from pandas import Series, DataFrame


data = DataFrame(np.random.randn(1000, 4))
print data.describe()
'''
                 0            1            2            3
count  1000.000000  1000.000000  1000.000000  1000.000000
mean      0.033659     0.009555     0.018607     0.000988
std       0.992909     0.990339     0.961525     1.014569
min      -2.900505    -3.101781    -3.571700    -3.743653
25%      -0.615258    -0.615599    -0.626577    -0.693573
50%       0.056842     0.028285    -0.015288    -0.040166
75%       0.721320     0.664719     0.609565     0.689604
max       3.475868     3.099867     3.611240     3.373796
'''

col = data[3]
print col[np.abs(col) > 3]
'''
79    -3.216870
769    3.154073
Name: 3, dtype: float64
'''
```


## 排列和随机抽样
```
# coding: utf-8

import numpy as np
import pandas as pd
from pandas import Series, DataFrame


df = DataFrame(np.arange(5 * 4).reshape((5, 4)))
print df
'''
    0   1   2   3
0   0   1   2   3
1   4   5   6   7
2   8   9  10  11
3  12  13  14  15
4  16  17  18  19
'''

sampler = np.random.permutation(5)
print sampler
'''
[0 3 1 2 4]
'''

# 无范围抽样
print df.take(sampler)
'''
    0   1   2   3
4  16  17  18  19
0   0   1   2   3
1   4   5   6   7
3  12  13  14  15
2   8   9  10  11
'''

# 无范围抽样
print df.take(np.random.permutation(len(df))[:3])
'''
    0   1   2   3
4  16  17  18  19
2   8   9  10  11
3  12  13  14  15
'''

print "-------------------"

bag = np.array([5, 7, -1, 6, 4])
sampler = np.random.randint(0, len(bag), size=10)
print sampler
'''
[3 1 2 2 4 1 3 3 2 2]
'''

# 有范围抽样
draws = bag.take(sampler)
print draws
'''
[ 7 -1  4  6  7  4  6  5  6  4]
'''
```


## 计算指标和哑变量
```
# coding: utf-8

import numpy as np
import pandas as pd
from pandas import Series, DataFrame


df = DataFrame({'key': ['b', 'b', 'a', 'c', 'a', 'b'],
                'data1': range(6)})
print df
'''
   data1 key
0      0   b
1      1   b
2      2   a
3      3   c
4      4   a
5      5   b
'''

print pd.get_dummies(df['key'])
'''
   a  b  c
0  0  1  0
1  0  1  0
2  1  0  0
3  0  0  1
4  1  0  0
5  0  1  0
'''

dummies = pd.get_dummies(df['key'], prefix='key')
df_with_dummy = df[['data1']].join(dummies)
print df_with_dummy
'''
   data1  key_a  key_b  key_c
0      0      0      1      0
1      1      0      1      0
2      2      1      0      0
3      3      0      0      1
4      4      1      0      0
5      5      0      1      0
'''


# 拿电影的数据举例
mnames = ['movie_id', 'title', 'genres']
movies = pd.read_table('data_screening_files/movies.dat', sep='::', header=None, names=mnames)

# 取出前十个
print movies[:10]
'''
   movie_id                               title                        genres
0         1                    Toy Story (1995)   Animation|Children's|Comedy
1         2                      Jumanji (1995)  Adventure|Children's|Fantasy
2         3             Grumpier Old Men (1995)                Comedy|Romance
3         4            Waiting to Exhale (1995)                  Comedy|Drama
4         5  Father of the Bride Part II (1995)                        Comedy
5         6                         Heat (1995)         Action|Crime|Thriller
6         7                      Sabrina (1995)                Comedy|Romance
7         8                 Tom and Huck (1995)          Adventure|Children's
8         9                 Sudden Death (1995)                        Action
9        10                    GoldenEye (1995)     Action|Adventure|Thriller

'''


genre_iter = (set(x.split('|')) for x in movies.genres)
genres = sorted(set.union(*genre_iter))
print genres
'''
['Action', 'Adventure', 'Animation', "Children's", 'Comedy', 'Crime', 'Documentary', 'Drama', 'Fantasy', 'Film-Noir', 'Horror', 'Musical', 'Mystery', 'Romance', 'Sci-Fi', 'Thriller', 'War', 'Western']
'''
```


## 线损率属性构造
```
# coding: utf-8

import numpy as np
import pandas as pd
from pandas import Series, DataFrame


inputfile = 'data_screening_files/electricity_data.xls'
outputfile = 'data_screening_files/electricity_data_out.xls'

data = pd.read_excel(inputfile)
print "元数据："
print data
'''
元数据：
   供入电量  供出电量
0   986   912
1  1208  1083
2  1108   975
3  1082   934
4  1285  1102
'''

data[u'线损率'] = (data[u'供入电量'] - data[u'供出电量']) / data[u'供入电量']
print "结果："
print data
'''
结果：
   供入电量  供出电量       线损率
0   986   912  0.075051
1  1208  1083  0.103477
2  1108   975  0.120036
3  1082   934  0.136784
4  1285  1102  0.142412
'''

data.to_excel(outputfile, index=False)
```



## 字符串操作（内置函数）
- Split
- Strip
- count
- find/rfind
- replace
- lower/upper
- ljust/rjust
等等

```
val = 'a, b, guido'

# 拆分时，前后空格没有清空
a = val.split(',')
print a
'''
['a', ' b', ' guido']
'''

# 拆分时，去掉每个元素的前后空格
pieces = [x.strip() for x in val.split(',')]
print pieces
'''
['a', 'b', 'guido']
'''

# 将数组所有的元素拼接
b = '::'.join(pieces)
print b
'''
a::b::guido
'''

# 查看一个字符串是否在指定字符串里
print 'guido' in val
'''
True
'''

# 查找逗号的第一个索引位置，如果没有找到指定字符串，则报错
print val.index(',')
'''
1
'''

# 查找冒号，-1表示没有找到，这个与index有区别就是，找不到，不会报错
print val.find(':')
'''
-1
'''

# 统计个数
print val.count('a')
'''
a
'''

# 替换指定字符
print val.replace(',', ':')
'''
a: b: guido
'''

# 全部转大写
print val.upper()
'''
A, B, GUIDO
'''
```



## 正则表达式
Python内置，正则表达式在Re模块
- findall, finditer
- match
- search
- split
- sub, subn
等
```
# coding: utf-8

import re

text = "foo     bar\t baz \tqux"

# 描述一个或者多个空格符号，用 \s 表示
print re.split('\s+', text)
'''
['foo', 'bar', 'baz', 'qux']
'''

# 先指定一个正则表达式模式，然后使用这个实例对象去调用
regex = re.compile('\s+')
print regex.split(text)
'''
['foo', 'bar', 'baz', 'qux']
'''

# 返回所有匹配 \s 的项
print regex.findall(text)
'''
['     ', '\t ', ' \t']
'''


email = """
Dave dave@google.com
Steve steve@gmail.com
Victor victorzhangq@qq.com
"""

#  [A-Z0-9._%+-] 表示A-Z，0-9，._%+-都可以
#  +@            表示紧跟一个@符号
#  [A-Z0-9.-]    表示A-Z，0-9，.-都可以
#  +\.           表示紧跟一个.  而表示转义
#  [A-Z]{2,4}    表示A-Z内，只能2-4位字母
pattern = r'[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,4}'

# 创建匹配模式，且忽略大小写
regex1 = re.compile(pattern, flags=re.IGNORECASE)

print regex1.findall(email)
'''
['dave@google.com', 'steve@gmail.com', 'victorzhangq@qq.com']
'''

# m = regex1.search(email)
# print(text[m.start():m.end()])

# n = regex1.match(email)

print(regex1.sub('REDACTED', email))
'''
Dave REDACTED
Steve REDACTED
Victor REDACTED
'''

```


## Pandas中矢量化字符串方法
- cat
- containsp
- count
- endswith, startswith
- findall
- get
- join
- len
- pad
- replace
等等
```
# coding: utf-8

import re
from pandas import Series

data = {"Dave": "dave@google.com", "Steve": "steve@gmail.com", "Victor": "victorzhangq@qq.com"}
data = Series(data)
print data
'''
Dave          dave@google.com
Steve         steve@gmail.com
Victor    victorzhangq@qq.com
dtype: object
'''

print data.isnull()
'''
Dave      False
Steve     False
Victor    False
dtype: bool
'''

print(data.str.contains('gmail'))
'''
Dave      False
Steve      True
Victor    False
dtype: bool
'''

pattern = r'[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,4}'

# 也可以用正则表达式和pandas配合用
print data.str.findall(pattern, flags=re.IGNORECASE)
'''
Dave          [dave@google.com]
Steve         [steve@gmail.com]
Victor    [victorzhangq@qq.com]
dtype: object
'''

matches = data.str.match(pattern, flags=re.IGNORECASE)
print matches
'''
Dave      True
Steve     True
Victor    True
dtype: bool
'''

# 取返回结果索引为1的值
print matches.get(1)
'''
True
'''

# 同上
print matches[1]
'''
True
'''

# 截取前五个字符
print data.str[:5]
'''
Dave      dave@
Steve     steve
Victor    victo
dtype: object
'''
```



##  json数据处理
```
# coding: utf-8

import json
import pandas as pd
from pandas import DataFrame


db = json.load(open('data_screening_files/foods-2011-10-03.json'))

# 有多少条
print len(db)
'''
6636
'''

print db[0].keys()
'''
[u'portions', u'description', u'tags', u'nutrients', u'group', u'id', u'manufacturer']
'''

print db[0]['nutrients'][0]
'''
{u'units': u'g', u'group': u'Composition', u'description': u'Protein', u'value': 25.18}
'''

# 取前面7行数据
nutrients = DataFrame(db[0]['nutrients'])
print nutrients[:7]
'''
                   description        group units    value
0                      Protein  Composition     g    25.18
1            Total lipid (fat)  Composition     g    29.20
2  Carbohydrate, by difference  Composition     g     3.06
3                          Ash        Other     g     3.28
4                       Energy       Energy  kcal   376.00
5                        Water  Composition     g    39.28
6                       Energy       Energy    kJ  1573.00
'''

info_keys = ['description', 'group', 'id', 'manufacturer']
info = DataFrame(db, columns=info_keys)
print info[:5]
'''
                          description                   group    id  \
0                     Cheese, caraway  Dairy and Egg Products  1008   
1                     Cheese, cheddar  Dairy and Egg Products  1009   
2                        Cheese, edam  Dairy and Egg Products  1018   
3                        Cheese, feta  Dairy and Egg Products  1019   
4  Cheese, mozzarella, part skim milk  Dairy and Egg Products  1028   

  manufacturer  
0               
1               
2               
3               
4   
'''


print pd.value_counts(info.group)[:10]
'''
Vegetables and Vegetable Products    812
Beef Products                        618
Baked Products                       496
Breakfast Cereals                    403
Legumes and Legume Products          365
Fast Foods                           365
Lamb, Veal, and Game Products        345
Sweets                               341
Fruits and Fruit Juices              328
Pork Products                        328
Name: group, dtype: int64
'''

nutrients = []
for rec in db:
    fnuts = DataFrame(rec['nutrients'])
    fnuts['id'] = rec['id']
    nutrients.append(fnuts)

nutrients = pd.concat(nutrients, ignore_index=True)
# print nutrients
'''
                               description        group    units     value  \
0                                  Protein  Composition        g    25.180   
1                        Total lipid (fat)  Composition        g    29.200   
2              Carbohydrate, by difference  Composition        g     3.060   
3                                      Ash        Other        g     3.280   
4                                   Energy       Energy     kcal   376.000
......
[389355 rows x 5 columns]
'''

# 去掉重复项，然后统计有多少行
print nutrients.duplicated().sum()
'''
14179
'''

# 删除掉重复项，并统计行数
# print nutrients.drop_duplicated().sum()


# 重命名
col_mapping = {"description": "food",
               "group": "fgroup"}
info = info.rename(columns=col_mapping, copy=False)
print info[:3]
'''
              food                  fgroup    id manufacturer
0  Cheese, caraway  Dairy and Egg Products  1008             
1  Cheese, cheddar  Dairy and Egg Products  1009             
2     Cheese, edam  Dairy and Egg Products  1018  
'''

# 使用外链接，id相等的
ndata = pd.merge(nutrients, info, on='id', how='outer')

# .ix能通过行号和行标签进行取值
# .iloc只能通过行号进行取值
print ndata.ix[100]
'''
description                      Water
group                      Composition
units                                g
value                            39.28
id                                1008
food                   Cheese, caraway
fgroup          Dairy and Egg Products
manufacturer                          
Name: 100, dtype: object
'''
```

