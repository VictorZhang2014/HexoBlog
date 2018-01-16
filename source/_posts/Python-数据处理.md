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
- 利用函数或映射进行数据转换
- 

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

# 添加一列
data['v1'] = range(7)
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


data = DataFrame(np.arange(12).reshape((3, 4)),
                 index=['Ohio', 'Colorado', 'New York'],
                 columns=['one', 'two', 'three', 'four'])

# 将Index都转成大写
print data.index.map(str.upper)
'''
Index([u'OHIO', u'COLORADO', u'NEW YORK'], dtype='object')
'''

# 转大写
# data.index = data.index.map(str.upper)
# print data
'''
          one  two  three  four
OHIO        0    1      2     3
COLORADO    4    5      6     7
NEW YORK    8    9     10    11
'''

# 转大写
print data.rename(index=str.title, columns=str.upper)
'''
          ONE  TWO  THREE  FOUR
Ohio        0    1      2     3
Colorado    4    5      6     7
New York    8    9     10    11
'''

# 转大写
print data.rename(index={"OHIO": "INDIANA"},
            columns={"three": "peekaboo"})
'''
          one  two  peekaboo  four
Ohio        0    1         2     3
Colorado    4    5         6     7
New York    8    9        10    11
'''

```























