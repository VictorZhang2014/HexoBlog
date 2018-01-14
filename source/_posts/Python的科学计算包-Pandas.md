---
title: Python的科学计算包- Pandas
date: 2018-01-09 23:18:53
tags: Python, Pandas
categories: AI
---

## Pandas
<a href="https://pandas.pydata.org/">Pandas</a>是Python Data Analysis Library或Pandas是基于NumPy的一种工具，该工具是为了解决数据分析任务而创建的。Pandas纳入了大量的库和一些标准的数据模型，提供了高效地操作大型数据集所需的工具。Pandas提供了大量能使我们快速便捷地处理数据的函数和方法。你很快就会发现，它是Python成为强大而高效的数据分析环境的重要因素之一。<a href="https://baike.baidu.com/item/pandas/17209606?fr=aladdin">百度百科</a>


本篇教程使用的所有资源文件下载地址：
- [ex1.csv](/files/pandas_data_files/ex1.csv)
- [ex2.csv](/files/pandas_data_files/ex2.csv)
- [ex3.csv](/files/pandas_data_files/ex3.csv)
- [ex3.txt](/files/pandas_data_files/ex3.txt)
- [ex4.csv](/files/pandas_data_files/ex4.csv)
- [ex5.csv](/files/pandas_data_files/ex5.csv)
- [ex6.csv](/files/pandas_data_files/ex6.csv)
- [ex7.csv](/files/pandas_data_files/ex7.csv)
- [csv_mindex.csv](/files/pandas_data_files/csv_mindex.csv)
- [frame_pickle](/files/pandas_data_files/frame_pickle)
- [out.csv](/files/pandas_data_files/out.csv)
- [test_file.csv](/files/pandas_data_files/test_file.csv)
- [tseries.csv](/files/pandas_data_files/tseries.csv)




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
```
# coding: utf-8

import numpy as np
import pandas as pd
import sys
from pandas import Series, DataFrame

data = {"state": ["Ohio", "Utah", "California", "Nevada"],
        "year": [2000, 2001, 2002, 2003],
        "pop": [1.5, 1.7, 3.6, 2.4]}

# 由Pandas自动排序列的位置
frame = DataFrame(data)
print frame
'''
   pop       state  year
0  1.5        Ohio  2000
1  1.7        Utah  2001
2  3.6  California  2002
3  2.4      Nevada  2003
'''

# 指定列的位置显示
frame = DataFrame(data, columns=["year", "state", "pop"])
print frame
'''
   year       state  pop
0  2000        Ohio  1.5
1  2001        Utah  1.7
2  2002  California  3.6
3  2003      Nevada  2.4
'''

# 指定索引名称
frame2 = DataFrame(data, columns=["year", "state", "pop", "debt"], index=["one", "two", "three", "four"])
print frame2
'''
       year       state  pop debt
one    2000        Ohio  1.5  NaN
two    2001        Utah  1.7  NaN
three  2002  California  3.6  NaN
four   2003      Nevada  2.4  NaN
'''

print frame2.columns
'''
Index([u'year', u'state', u'pop', u'debt'], dtype='object')
'''

# 指定名称选取列的值
print frame2['state']
'''
one            Ohio
two            Utah
three    California
four         Nevada
Name: state, dtype: object
'''

# 也可以通过名称直接显示
print frame2.year
'''
one      2000
two      2001
three    2002
four     2003
Name: year, dtype: int64
'''

# 修改值
frame2["debt"] = 16.5
print frame2
'''
       year       state  pop  debt
one    2000        Ohio  1.5  16.5
two    2001        Utah  1.7  16.5
three  2002  California  3.6  16.5
four   2003      Nevada  2.4  16.5
'''

# 赋值
frame2["debt"] = np.arange(4.)
print frame2
'''
       year       state  pop  debt
one    2000        Ohio  1.5   0.0
two    2001        Utah  1.7   1.0
three  2002  California  3.6   2.0
four   2003      Nevada  2.4   3.0
'''


# 在数组里指定索引赋值
val = Series([-1.2, -1.5, -1.7], index=["two", "four", "five"])
frame2["debt"] = val
print frame2
'''
       year       state  pop  debt
one    2000        Ohio  1.5   NaN
two    2001        Utah  1.7  -1.2
three  2002  California  3.6   NaN
four   2003      Nevada  2.4  -1.5
'''

# 添加一列并且等于Ohio的值，才是Bool类型
frame2["eastern"] = frame2.state == "Ohio"
print frame2
'''
       year       state  pop  debt  eastern
one    2000        Ohio  1.5   NaN     True
two    2001        Utah  1.7  -1.2    False
three  2002  California  3.6   NaN    False
four   2003      Nevada  2.4  -1.5    False
'''

# 删除列
del frame2["eastern"]
print frame2
'''
       year       state  pop  debt
one    2000        Ohio  1.5   NaN
two    2001        Utah  1.7  -1.2
three  2002  California  3.6   NaN
four   2003      Nevada  2.4  -1.5
'''

# 字典嵌套，自动排序
pop = {"Nevada": {2001: 2.4, 2002: 2.9},
       "Ohio": {2000: 1.5, 2001: 1.7, 2002: 3.6}}
frame3 = DataFrame(pop)
print frame3
'''
      Nevada  Ohio
2000     NaN   1.5
2001     2.4   1.7
2002     2.9   3.6
'''

# 指定索引的排序方式
pop2 = DataFrame(pop, index=[2000, 2001, 2002, 2003])
print pop2
'''
      Nevada  Ohio
2000     NaN   1.5
2001     2.4   1.7
2002     2.9   3.6
2003     NaN   NaN
'''

print frame3.values
'''
[[ nan  1.5]
 [ 2.4  1.7]
 [ 2.9  3.6]]
'''

frame3.index.name = 'year'
frame3.columns.name = 'state'
print frame3
'''
state  Nevada  Ohio
year               
2000      NaN   1.5
2001      2.4   1.7
2002      2.9   3.6
'''


# 索引对象
obj = Series(range(3), index=["a", "b", "c"])
print obj
'''
a    0
b    1
c    2
'''

index = obj.index
print index
'''
Index([u'a', u'b', u'c'], dtype='object')
'''

# 这是不允许的
# index[1] = 'd'


# Index是一个类，有很多方法
index = pd.Index(np.arange(3))
print index
'''
Int64Index([0, 1, 2], dtype='int64')
'''

obj2 = Series([1.5, -2.5, 0], index=index)
print obj2.index is index  # 输出 True

```


## 读写文本格式数据
Pandas提供一些用于将表格型数据读取为DataFrame对象的函数
函数：
- read_csv        逗号为分隔符
- read_table      制表符为分割符，如：'\t'
- read_fwf        没有分隔符，定宽列
- read_clipboard  读取剪切板中的数据

函数选项：
- 索引
- 类型推断和数据转换
- 日期解析
- 迭代
- 不规整数据问题

```
# coding: utf-8

import numpy as np
import pandas as pd
from pandas import DataFrame

# 读取csv文件
df = pd.read_csv('pandas_data_files/ex1.csv')
print df
'''
   a   b   c   d message
0  1   2   3   4   hello
1  5   6   7   8   world
2  9  10  11  12     foo
'''

# 需要制定分隔符为逗号
df1 = pd.read_table('pandas_data_files/ex1.csv', sep=',')
print df1
'''
   a   b   c   d message
0  1   2   3   4   hello
1  5   6   7   8   world
2  9  10  11  12     foo
'''

# 读取的表格不想要表头
df2 = pd.read_csv("pandas_data_files/ex2.csv", header=None)
print "df2:"
print df2
'''
df2:
   0   1   2   3      4
0  1   2   3   4  hello
1  5   6   7   8  world
2  9  10  11  12    foo
'''

# 指定列名，并返回
df3 = pd.read_csv("pandas_data_files/ex2.csv", names=['a', 'b', 'c', 'd', 'message'])
print "df3:"
print df3
'''
df3:
   a   b   c   d message
0  1   2   3   4   hello
1  5   6   7   8   world
2  9  10  11  12     foo
'''

# 行索引是message
names = ['a', 'b', 'c', 'd', 'message']
df4 = pd.read_csv('pandas_data_files/ex2.csv', names=names, index_col='message')
print "df4:"
print df4
'''
df4:
         a   b   c   d
message               
hello    1   2   3   4
world    5   6   7   8
foo      9  10  11  12
'''

# 多个列作为索引列
parsed = pd.read_csv('pandas_data_files/csv_mindex.csv', index_col=['key1', 'key2'])
print "parsed:"
print parsed
'''
parsed:
           value1  value2
key1 key2                
one  a          1       2
     b          3       4
     c          5       6
     d          7       8
two  a          9      10
     b         11      12
     c         13      14
     d         15      16
'''

# 原始数据
list = list(open('pandas_data_files/ex3.txt'))
print list
'''
['A         B         C\n', 'aaa -0.264438 -1.026059 -0.619500\n', 'bbb  0.927272  0.302904 -0.032399\n', 'ccc -0.264273 -0.386314 -0.217601\n', 'ddd -0.871858 -0.348382  1.100491\n']
'''

# 指定正则表达式读取
result = pd.read_table('pandas_data_files/ex3.txt', sep="\s+")
print result
'''
            A         B         C
aaa -0.264438 -1.026059 -0.619500
bbb  0.927272  0.302904 -0.032399
ccc -0.264273 -0.386314 -0.217601
ddd -0.871858 -0.348382  1.100491
'''

# 读取数据，并跳过索引为0,2,3的行的数据
result1 = pd.read_csv('pandas_data_files/ex4.csv', skiprows=[0, 2, 3])
print result1
'''
   a   b   c   d message
0  1   2   3   4   hello
1  5   6   7   8   world
2  9  10  11  12     foo
'''

# 如果有缺失值，则显示long类型，并输出NaN
result2 = pd.read_csv('pandas_data_files/ex5.csv')
print result2
'''
  something  a   b     c   d message
0       one  1   2   3.0   4     NaN
1       two  5   6   NaN   8   world
2     three  9  10  11.0  12     foo
'''

# 我们可以通过isnull来检测，是否为null的值
print pd.isnull(result2)
'''
   something      a      b      c      d  message
0      False  False  False  False  False     True
1      False  False  False   True  False    False
2      False  False  False  False  False    Fals
'''

# 如果原始数据里有NULL值，我们也可以标记为缺失值
result3 = pd.read_csv('pandas_data_files/ex5.csv', na_values=['NULL'])
print result3
'''
  something  a   b     c   d message
0       one  1   2   3.0   4     NaN
1       two  5   6   NaN   8   world
2     three  9  10  11.0  12     foo
'''

#  我们还可以对指定值进行填充为缺失值
sentinels = {"message": ["foo", "NA"], "something": ["two"]}
result4 = pd.read_csv('pandas_data_files/ex5.csv', na_values=sentinels)
print result4
'''
  something  a   b     c   d message
0       one  1   2   3.0   4     NaN
1       NaN  5   6   NaN   8   world
2     three  9  10  11.0  12     NaN
'''
```


## 逐行读取文本文件
比如文件内容很大，我们不可能一下子把所有的内容直接从文件都读取出来，为了速度，我们可以指定多少行、指定多大的方式来读取
```
# coding: utf-8

import pandas as pd
from pandas import Series, DataFrame

# 读取一个大文件
result = pd.read_csv('pandas_data_files/ex6.csv')
print result
'''
           one       two     three      four key
0     0.467976 -0.038649 -0.295344 -1.824726   L
1    -0.358893  1.404453  0.704965 -0.200638   B
...... 这里有一万行略过

[10000 rows x 5 columns]
'''

# 对指定的文件只读取5行
result1 = pd.read_csv('pandas_data_files/ex6.csv', nrows=5)
print result1
'''
        one       two     three      four key
0  0.467976 -0.038649 -0.295344 -1.824726   L
1 -0.358893  1.404453  0.704965 -0.200638   B
2 -0.501840  0.659254 -0.421691 -0.057688   G
3  0.204886  1.074134  1.388361 -0.982404   R
4  0.354628 -0.133116  0.283763 -0.837063   Q
'''

# 读取1000个字节的数据到变量chunker里
chunker = pd.read_csv('pandas_data_files/ex6.csv', chunksize=1000)
print chunker

tot = Series([])
for piece in chunker:
    tot = tot.add(piece['key'].value_counts(), fill_value=0)

tot = tot.order(ascending=False)
print tot[:10]
```
在这里我用的Pandas v0.22.0，这是最新的版本，但是遇到了错误：` AttributeError: 'Series' object has no attribute 'order' ` 我不知道怎么解决，如果有小伙伴知道，请分享以下

```
# coding: utf-8

import sys
import numpy as np
import pandas as pd
from pandas import Series, DataFrame

data = pd.read_csv('pandas_data_files/ex5.csv')
print data
'''  something  a   b     c   d message
0       one  1   2   3.0   4     NaN
1       two  5   6   NaN   8   world
2     three  9  10  11.0  12     foo
'''

# 将内容转成csv文件，并输出文件
data.to_csv('pandas_data_files/ex5_out.csv')

# 现在读取下这个 ex5_out.csv文件
print pd.read_csv('pandas_data_files/ex5_out.csv')
'''   
Unnamed: 0 something  a   b     c   d message
0           0       one  1   2   3.0   4     NaN
1           1       two  5   6   NaN   8   world
2           2     three  9  10  11.0  12     foo
'''

# 我们也可以保存文件时，使用别的分隔符，比如"|"
# data.to_csv(sys.stdout, index=False, columns=['a', 'b', 'c'])

# 输出日期
dates = pd.date_range('1/1/2000', periods=7)
print dates
'''
DatetimeIndex(['2000-01-01', '2000-01-02', '2000-01-03', '2000-01-04',
               '2000-01-05', '2000-01-06', '2000-01-07'],
              dtype='datetime64[ns]', freq='D')
'''

# 将日期列表转成Series的形式
ts = Series(np.arange(7), index=dates)
print ts
'''
2000-01-01    0
2000-01-02    1
2000-01-03    2
2000-01-04    3
2000-01-05    4
2000-01-06    5
2000-01-07    6
Freq: D, dtype: int64
'''

# 使用Series写入到文件
ts.to_csv('pandas_data_files/test_Series_dates.csv')

# 使用Series读取文件
print Series.from_csv('pandas_data_files/test_Series_dates.csv', parse_dates=True)
'''
2000-01-01    0
2000-01-02    1
2000-01-03    2
2000-01-04    3
2000-01-05    4
2000-01-06    5
2000-01-07    6
dtype: int64
'''
```

## 手工处理分隔符的方式
```
# coding: utf-8

import numpy as np
import pandas as pd
from pandas import Series, DataFrame
import csv

# 读取文件
f = open('pandas_data_files/ex7.csv')
reader = csv.reader(f)
for line in reader:
    print(line)
'''
['a', 'b', 'c']
['1', '2', '3']
['1', '2', '3', '4']
'''

# 另一种读取方式
lines = list(csv.reader(open('pandas_data_files/ex7.csv')))
header, values = lines[0], lines[1:]
data_dict = {h: v for h, v in zip(header, zip(*values))}
print data_dict
'''
{'a': ('1', '1'), 'c': ('3', '3'), 'b': ('2', '2')}
'''

# 自定义类集成子csv.Dialect，来读取文件
class my_dialect(csv.Dialect):
    lineterminator = '\n'     # 行终止符
    delimiter = ';'           # 定界符
    quotechar = '"'           # 引用字符
    quoting = csv.QUOTE_MINIMAL

with open('pandas_data_files/ex7_out.csv', 'w') as f:
    writer = csv.writer(f, dialect=my_dialect)
    writer.writerow(('one', 'two', 'three'))
    writer.writerow(('1', '2', '3'))
    writer.writerow(('4', '5', '6'))
    writer.writerow(('7', '8', '9'))

# 读取看下
print("------")
with open('pandas_data_files/ex7_out.csv', 'r') as f:
    reader = csv.reader(f)
    for line in reader:
        print(line)

```


## Excel文件的读取
因为很多的数据都是从Excel过来的，但是这两个组件需要安装，使用PyCharm如何安装呢？ <a href="http://www.googleplus.party/2017/12/30/NumPy,%20SciPy%E5%92%8CPandas%E7%9A%84%E5%AE%89%E8%A3%85%E6%95%99%E7%A8%8B/">看这篇文章，因为安装的方式是一样的，只不过名称不同</a>
- Excel基本库， 
  - 读取：xlrd
  - 写入：xlwt
- 基本电子表格交互
  - 生成工作簿
  - 从工作簿中读取数据
  - 使用OpenPyxl
  - 使用pandas读写

Excel中的数据类型和Python的数据类型对比
类型                编号        Python类型
XL_CELL_EMPTY        0           空字符串
XL_CELL_TEXT         1           Unicode字符串
XL_CELL_NUMBER       2           Float
XL_CELL_DATA         3           Float
XL_CELL_BOOLEAN      4           Int( 1 = True, 0 = False)           
XL_CELL_ERROR        5           Int表示Excel内部编码
XL_CELL_BLANK        6           空字符串，仅当formatting into = True

实例讲解，读取和写入excel
```
# coding: utf-8

import numpy as np
import pandas as pd
import xlrd, xlwt

path = 'pandas_data_files'

# 生成一个工作簿
wb = xlwt.Workbook()
print wb
'''
我们看到打印的内存地址是0x109a7cb90，也就是说一张空的工作簿在此内存中
<xlwt.Workbook.Workbook object at 0x109a7cb90>
'''

# 添加一张工作表
wb.add_sheet('first_sheet', cell_overwrite_ok=True)
wb.get_active_sheet()

# 获取第一个工作簿
ws_1 = wb.get_sheet(0)
print ws_1
'''
<xlwt.Worksheet.Worksheet object at 0x10095e8d0>
'''

# 添加第二张工作表
ws_2 = wb.add_sheet('second_sheet')

# 生成一个8x8的矩阵
data = np.arange(1, 65).reshape((8, 8))
print data
'''
[[ 1  2  3  4  5  6  7  8]
 [ 9 10 11 12 13 14 15 16]
 [17 18 19 20 21 22 23 24]
 [25 26 27 28 29 30 31 32]
 [33 34 35 36 37 38 39 40]
 [41 42 43 44 45 46 47 48]
 [49 50 51 52 53 54 55 56]
 [57 58 59 60 61 62 63 64]]
'''

# 例如：写入工作表
# 参数1：写入的行索引号
# 参数2：写入的列索引号
# 参数3：写入的值
# ws_1.write(0, 0, 100)

# 通过两个循环，将行和列的每个值写入到工作表
for c in range(data.shape[0]):
    for r in range(data.shape[1]):
        ws_1.write(r, c, data[c, r])
        ws_2.write(r, c, data[c, r])

# 保存到硬盘
wb.save(path + '/workbook.xls')
# 默认保存的是2003版的excel，如果是2007版的话，就是后缀名改下就行
# 现在可以去你的硬盘查看内容，两个工作表的内容都是一样的


# 接下来，从工作簿中读取
book = xlrd.open_workbook(path + '/workbook.xls')
print book
'''
现在已经读取到内存中了
<xlrd.book.Book object at 0x1107bae10>
'''

# 打印看有几个工作表
print book.sheet_names()
'''
[u'first_sheet', u'second_sheet']
'''

# 通过名字读取工作表
sheet_1 = book.sheet_by_name('first_sheet')
# 通过索引获取工作表
sheet_2 = book.sheet_by_index(1)
print sheet_2.name
'''
second_sheet
'''

# 有多少列，有多少行
print sheet_1.ncols, sheet_1.nrows
'''
8 8
'''

# 比如：我们查看第一行，第一列的值，和值类型
c1 = sheet_1.cell(0, 0)
print c1.value
print c1.ctype
'''
1.0
2
'''
# 2表示Float，对于Excel的数据类型和Python的数据类型的关系，详见上面那个表格

# 从第三列开始读，起始行3，结束行7
print sheet_1.col_values(3, start_rowx=3, end_rowx=7)
'''
[28.0, 29.0, 30.0, 31.0]
'''

# 我们用Python的循环完全展现工作表
for c in range(sheet_1.ncols):
    for r in range(sheet_1.nrows):
        print '%i' % sheet_1.cell(r, c).value,
    print
'''
1 2 3 4 5 6 7 8
9 10 11 12 13 14 15 16
17 18 19 20 21 22 23 24
25 26 27 28 29 30 31 32
33 34 35 36 37 38 39 40
41 42 43 44 45 46 47 48
49 50 51 52 53 54 55 56
57 58 59 60 61 62 63 64
'''

# 但是，如果使用pandas，则是相当的简单
xls_file = pd.ExcelFile(path + '/workbook.xls')
table = xls_file.parse('first_sheet')
print table
'''
   1   9   17  25  33  41  49  57
0   2  10  18  26  34  42  50  58
1   3  11  19  27  35  43  51  59
2   4  12  20  28  36  44  52  60
3   5  13  21  29  37  45  53  61
4   6  14  22  30  38  46  54  62
5   7  15  23  31  39  47  55  63
6   8  16  24  32  40  48  56  64
'''
# 我们看到读取出来的DataFrame类型的数据
```


## JSON数据
JSON数据（JavaScript Object Notation）
数据类型：
- 对象
- 数组
- 字符串
- 数值
- 布尔值
- null

JSON库的loads方法，JSON这个工具是自带的。

```
# coding: utf-8

import json
from pandas import DataFrame

# 伪造一些数据
obj = """
{ "name": "Wes",
  "places_lived": ["United States", "Spain", "Germany"],
  "pet": null,
  "siblings": [{
                 "name": "Scott",
                 "age": "25"
                }]
}
"""

# 加载json数据
result = json.loads(obj)
print result
'''
{u'pet': None, u'siblings': [{u'age': u'25', u'name': u'Scott'}], u'name': u'Wes', u'places_lived': [u'United States', u'Spain', u'Germany']}
'''

# 转成json对象
json_obj = json.dumps(result)

# Pandas也支持json的数据读取，也特别简单
siblings = DataFrame(result['siblings'], columns=['name', 'age'])
print siblings
'''
    name age
0  Scott  25
'''
# 用法比较简单
```


## 二进制数据格式的读取和写入
pickle存储的数据格式是短期的存储，不能永远保存数据永远是正确的，所以仅仅是用于短期和临时的存储
```
# coding: utf-8

import pandas as pd

# 读取文件
frame = pd.read_csv('pandas_data_files/ex1.csv')
print frame
'''
   a   b   c   d message
0  1   2   3   4   hello
1  5   6   7   8   world
2  9  10  11  12     foo
'''

# 写入二进制文件，并输出
frame.to_pickle('pandas_data_files/ex1_pickle_binary_out.csv')

# 读取二进制文件
print pd.read_pickle('pandas_data_files/ex1_pickle_binary_out.csv')
'''
   a   b   c   d message
0  1   2   3   4   hello
1  5   6   7   8   world
2  9  10  11  12     foo
'''
```


## HDF5格式
<a href="http://www.h5py.org">HDF5介绍</a> 是一种数据格式，用于大量的数据进行科学计算，可移植性高，可扩展型高。

很大大型机构的数据存储格式都采用了HDF5，比如：NASA的地球观测系统，MATLAB等等。
```
# coding: utf-8

import h5py
import numpy as np


# 数据
imgData = np.arange(24).reshape((2, 3, 4))
print imgData
'''
[[[ 0  1  2  3]
  [ 4  5  6  7]
  [ 8  9 10 11]]

 [[12 13 14 15]
  [16 17 18 19]
  [20 21 22 23]]]
'''

# 写入硬盘
f = h5py.File('pandas_data_files/HDF5_file.h5', 'w')
f['data'] = imgData
f['labels'] = range(24)
f.close()

# 从硬盘读取
f = h5py.File('pandas_data_files/HDF5_file.h5', 'r')

# 查看所有的主键
print f.keys()
'''
[u'data', u'labels']
'''

# 查看主键为data的所有的值
a = f['data'][:]
print a
'''
[[[ 0  1  2  3]
  [ 4  5  6  7]
  [ 8  9 10 11]]

 [[12 13 14 15]
  [16 17 18 19]
  [20 21 22 23]]]
'''

f.close()
```





## 使用HTML和Web API
需要安装request工具
```
# coding: utf-8

import requests
import json
from pandas import DataFrame

# 从网址读取数据
url = 'https://api.github.com/repos/pydata/pandas/milestones/28/labels'
resp = requests.get(url)
print resp
'''
<Response [200]>
'''

# 读取到json对象
data = json.loads(resp.text)
print data
'''
[{u'url': u'https://api.github.com/repos/pandas-dev/pandas/labels/Bug', u'color': u'e10c02', u'default': False, u'id': 76811, ......}]
'''

# 将json对象转成DataFrame
issue_labels = DataFrame(data)
print issue_labels
'''
     color  default        id             name  \
0   e10c02    False     76811              Bug   
1   4E9A06    False     76812      Enhancement   
2   FCE94F    False    127681         Refactor   
3   75507B    False    129350            Build   
4   3465A4    False    134699             Docs   
5   AFEEEE    False    211840       Timeseries   
......   
  
                                                  url  
0   https://api.github.com/repos/pandas-dev/pandas...  
1   https://api.github.com/repos/pandas-dev/pandas...  
2   https://api.github.com/repos/pandas-dev/pandas...  
3   https://api.github.com/repos/pandas-dev/pandas...  
4   https://api.github.com/repos/pandas-dev/pandas...  
5   https://api.github.com/repos/pandas-dev/pandas...  
......
'''
```



## 使用数据库
自带sqlite3
```
# coding: utf-8

import sqlite3
from pandas import DataFrame


# 创建一张表
query = """
CREATE TABLE test 
(a VARCHAR(20), b VARCHAR(20), c REAL, d INTEGER);
"""

# 创建数据库
conn = sqlite3.connect('pandas_data_files/mydb.sqlite')

# 执行sql语句
conn.execute(query)
conn.commit()

# 插入数据
data = [('Atlanta', 'Georgia', 1.25, 6),
        ('Tallahassee', 'Florida', 2.7, 3),
        ('Sacramento', 'California', 1.9, 6)]
stmt = "insert into test values(?,?,?,?);"

# 执行插入命令
conn.executemany(stmt, data)
conn.commit()

# 查询数据
cursor = conn.execute("select * from test")
rows = cursor.fetchall()
print rows
'''
[(u'Atlanta', u'Georgia', 1.25, 6), (u'Tallahassee', u'Florida', 2.7, 3), (u'Sacramento', u'California', 1.9, 6)]
'''

# 查看数据表示几列
print cursor.description
'''
(('a', None, None, None, None, None, None), ('b', None, None, None, None, None, None), ('c', None, None, None, None, None, None), ('d', None, None, None, None, None, None))
'''

# 使用DataFrame读取数据
print DataFrame(rows, columns=zip(*cursor.description)[0])
'''
             a           b     c  d
0      Atlanta     Georgia  1.25  6
1  Tallahassee     Florida  2.70  3
2   Sacramento  California  1.90  6
'''


# 或者pandas有对sql的接口读取
import pandas.io.sql as sql

print sql.read_sql("select * from test", conn)
'''
             a           b     c  d
0      Atlanta     Georgia  1.25  6
1  Tallahassee     Florida  2.70  3
2   Sacramento  California  1.90  6
'''

```












 














