---
title: Python的科学计算包- Pandas
date: 2018-01-09 23:18:53
tags: Python, Pandas
categories: AI
---

## Pandas
<a href="https://pandas.pydata.org/">Pandas</a>是Python Data Analysis Library或Pandas是基于NumPy的一种工具，该工具是为了解决数据分析任务而创建的。Pandas纳入了大量的库和一些标准的数据模型，提供了高效地操作大型数据集所需的工具。Pandas提供了大量能使我们快速便捷地处理数据的函数和方法。你很快就会发现，它是Python成为强大而高效的数据分析环境的重要因素之一。<a href="https://baike.baidu.com/item/pandas/17209606?fr=aladdin">百度百科</a>


## Pandas有两个主要的数据结构
- 1.Series
    - 数组与标签
    - 可以通过标签选取数据
    - 定长的有序字典
- 2.Dataframe
    - 表格型数据结构
    - 行索引、列索引


## Series的基本使用
```
# coding: utf-8

import numpy as np
from pandas import Series

obj = Series([4, 7, -5, 3])
print obj
'''
第一列是索引，第二列是值
0    4
1    7
2   -5
3    3
dtype: int64
'''

# 获取所有的值
print obj.values
'''
[ 4  7 -5  3]
'''

# 获取所有的索引
print obj.index
'''
RangeIndex(start=0, stop=4, step=1)
'''


obj2 = Series([4, 7, -5, 3], index=['d', 'b', 'a', 'c'])
print obj2
'''
第一列是索引，第二列是值
d    4
b    7
a   -5
c    3
dtype: int64
'''

print obj2.index
'''
Index([u'd', u'b', u'a', u'c'], dtype='object')
'''

# 获取索引为a的值
print obj2['a']
'''
-5
'''

# 把索引为d的值改成6
obj2['d'] = 6
print obj2
'''
d    6
b    7
a   -5
c    3
dtype: int64
'''

# 选取指定索引打印它的值
print obj2[['c', 'a', 'd']]
'''
c    3
a   -5
d    6
dtype: int64
'''

# 打印数组里的元素大于0的值
print obj2[obj2 > 0]
'''
d    6
b    7
c    3
dtype: int64
'''

# 直接运算
print obj2 * 2
'''
d    12
b    14
a   -10
c     6
dtype: int64
'''

print np.exp(obj2)
'''
d     403.428793
b    1096.633158
a       0.006738
c      20.085537
dtype: float64
'''

# b这个索引值是否在obj2里
print 'b' in obj2
'''
True
'''


# 字典的操作
sdata = {"Ohio": 35000, "Texas": 7100, "Oregon": 1600, "Utah": 500}
obj3 = Series(sdata)
print obj3
'''
Ohio      35000
Oregon     1600
Texas      7100
Utah        500
dtype: int64
'''

states = ["California", "Ohio", "Oregon", "Texas"]
obj4 = Series(sdata, index=states)
print obj4
'''
California        NaN
Ohio          35000.0
Oregon         1600.0
Texas          7100.0
dtype: float64
'''

# 查看数组的元素哪些是null，哪些不是null
print pd.isnull(obj4)
'''
California     True
Ohio          False
Oregon        False
Texas         False
dtype: bool
'''

# 查看数组的元素哪些不是null，哪些是null
print pd.notnull(obj4)
'''
California    False
Ohio           True
Oregon         True
Texas          True
dtype: bool
'''

print obj3 + obj4
'''
California        NaN
Ohio          70000.0
Oregon         3200.0
Texas         14200.0
Utah              NaN
dtype: float64
'''

obj4.name = 'population'
obj4.index.name = 'state'
print obj4
'''
California        NaN
Ohio          35000.0
Oregon         1600.0
Texas          7100.0
Name: population, dtype: float64
'''

obj.index = ['Bob', 'Steve', 'Jeff', 'Ryan']
print obj
'''
Bob      4
Steve    7
Jeff    -5
Ryan     3
dtype: int64
'''
```


## DataFrame的基本使用
启蒙于R语言，它的同一列必须是同一种类型
可以面向"行"来操作，也可以面向"列"来操作












