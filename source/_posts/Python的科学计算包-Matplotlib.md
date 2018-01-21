---
title: Python的科学计算包-Matplotlib
date: 2018-01-20 19:32:58
tags: Matplotlib
categories: AI
---

先来看个简单的例子
```
import matplotlib.pyplot as plt

plt.plot([1, 2, 3, 2, 3, 2, 2, 1])
plt.show()
```
<img src="/img/Python/Matplotlib/matplotlib_1.png" alt="" width="320" height="640" />


```
import matplotlib.pyplot as plt

plt.plot([4, 3, 2, 1], [1, 2, 3, 4])
plt.show()
```
<img src="/img/Python/Matplotlib/matplotlib_2.png" alt="" width="320" height="640" />



```
import matplotlib.pyplot as plt

x = [1, 2, 3, 4]
y = [5, 4, 3, 2]

# 二维平面分成2x3
plt.subplot(2, 3, 1)
plt.plot(x, y)
plt.show()
```
<img src="/img/Python/Matplotlib/matplotlib_3.png" alt="" width="320" height="640" />



```
import matplotlib.pyplot as plt

x = [1, 2, 3, 4]
y = [5, 4, 3, 2]

plt.subplot(232)
plt.bar(x, y)
plt.show()
```
<img src="/img/Python/Matplotlib/matplotlib_4.png" alt="" width="320" height="640" />



```
import matplotlib.pyplot as plt

x = [1, 2, 3, 4]
y = [5, 4, 3, 2]

plt.subplot(233)
plt.barh(x, y)
plt.show()
```
<img src="/img/Python/Matplotlib/matplotlib_5.png" alt="" width="320" height="640" />



```
import matplotlib.pyplot as plt

x = [1, 2, 3, 4]
y = [5, 4, 3, 2]

plt.subplot(234)
plt.bar(x, y)
plt.show()
```
<img src="/img/Python/Matplotlib/matplotlib_6.png" alt="" width="320" height="640" />



```
import matplotlib.pyplot as plt

x = [1, 2, 3, 4]
y = [5, 4, 3, 2]

y1 = [7, 8, 5, 3]
plt.bar(x, y1, bottom=y, color='r')
plt.show()
```
<img src="/img/Python/Matplotlib/matplotlib_7.png" alt="" width="320" height="640" />



```
import matplotlib.pyplot as plt

x = [1, 2, 3, 4]

plt.subplot(235)
plt.boxplot(x)
plt.show()
```
<img src="/img/Python/Matplotlib/matplotlib_8.png" alt="" width="320" height="640" />



```
import matplotlib.pyplot as plt

x = [1, 2, 3, 4]

plt.subplot(236)
plt.scatter(x, y)
plt.show()
```
<img src="/img/Python/Matplotlib/matplotlib_9.png" alt="" width="320" height="640" />



```
import matplotlib.pyplot as plt

x = [1, 2, 3, 4]
y = [5, 4, 3, 2]

# 二维平面分成 2行3列，此图在第一个位置
plt.subplot(2, 3, 1)
# 斜线
plt.plot(x, y)

# 二维平面分成 2行3列，此图在第二个位置
plt.subplot(232)
# 垂直柱状图
plt.bar(x, y)

# 二维平面分成 2行3列，此图在第三个位置
plt.subplot(233)
# 水平柱状图
plt.barh(x, y)

# 二维平面分成 2行3列，此图在第四个位置
plt.subplot(234)
# 水平柱状图
plt.bar(x, y)

# 二维平面分成 2行3列，此图在第四个位置与上一个相叠加
y1 = [7, 8, 5, 3]
# 水平柱状图
plt.bar(x, y1, bottom=y, color='r')

# 二维平面分成 2行3列，此图在第五个位置
plt.subplot(235)
# 盒子图，或箱型图
plt.boxplot(x)

# 二维平面分成 2行3列，此图在第六个位置
plt.subplot(236)
# 分散图
plt.scatter(x, y)

# 绘制
plt.show()

```
<img src="/img/Python/Matplotlib/matplotlib_10.png" alt="" width="320" height="640" />



<br/>

## figure和subplot
figure对象，就是一个窗口
subplot，就是窗口里的一个图像

```
figure2 = plt.figure()
figure2.suptitle(u"This is an example Figure！很简单的呀")
ax1 = figure2.add_subplot(2, 2, 1)
ax2 = figure2.add_subplot(2, 2, 2)
ax3 = figure2.add_subplot(2, 2, 3)
plt.show()
```
<img src="/img/Python/Matplotlib/matplotlib_11.png" alt="" width="320" height="640" />
中文乱码~~~~


```
import matplotlib.pyplot as plt

figure2 = plt.figure()
figure2.suptitle(u"This title is not supported Mandarin.")

ax1 = figure2.add_subplot(2, 2, 1)
ax2 = figure2.add_subplot(2, 2, 2)
ax3 = figure2.add_subplot(2, 2, 3)


from numpy.random import randn

# 因为最后一个图是第三个，所以这个绘图会在最后一个（也就是第三个图）绘制散点图
plt.plot(randn(50).cumsum(), 'k--')
figure2.show()

plt.show()
```
<img src="/img/Python/Matplotlib/matplotlib_12.png" alt="" width="320" height="640" />



```
# coding: utf-8

import matplotlib.pyplot as plt
import numpy as np

figure2 = plt.figure()

ax1 = figure2.add_subplot(2, 2, 1)
ax2 = figure2.add_subplot(2, 2, 2)
ax3 = figure2.add_subplot(2, 2, 3)


from numpy.random import randn

# 因为最后一个图是第三个，所以这个绘图会在最后一个（也就是第三个图）绘制散点图
plt.plot(randn(50).cumsum(), 'k--')

# 也可以对指定的图进行绘制
ax1.hist(randn(100), bins=20, color='k', alpha=0.3)
ax2.scatter(np.arange(30), np.arange(30) + 3 * randn(30))

plt.show()
```
<img src="/img/Python/Matplotlib/matplotlib_13.png" alt="" width="320" height="640" />


如果需要通过代码关闭窗口，则
```
plt.close('all')
```


```
# coding: utf-8

import matplotlib.pyplot as plt

# 我们创建的一个2行3列的矩阵图后，通过返回值，我们可以设置更多的参数等
fig, axes = plt.subplots(2, 3)
print fig
print "-------"
print axes
'''
Figure(640x480)
-------
[[<matplotlib.axes._subplots.AxesSubplot object at 0x1080157d0>
  <matplotlib.axes._subplots.AxesSubplot object at 0x109599390>
  <matplotlib.axes._subplots.AxesSubplot object at 0x1095d9c90>]
 [<matplotlib.axes._subplots.AxesSubplot object at 0x109628650>
  <matplotlib.axes._subplots.AxesSubplot object at 0x10965ec90>
  <matplotlib.axes._subplots.AxesSubplot object at 0x1096ad850>]]
'''

plt.show()
```
<img src="/img/Python/Matplotlib/matplotlib_14.png" alt="" width="320" height="640" />


<br/>
<br/>

pyplot.subplots的参数选项
- nrows             subplot的行数
- ncols             subplot的列数
- sharex            所有subplot应该使用相同的x轴刻度（调节xlim将会影响所有subplot）
- sharey            所有subplot应该使用相同的y轴刻度（调节ylim将会影响所有subplot）           
- subplot_kw        用于创建各subplot的关键字字典
- **fig_kw          创建figure时的其他关键字，如：plt.subplots(2, 2, figsize=(8, 6))

而且它的基本设置有
- 颜色、标记和线型
- 刻度、标签
- 注释
- 图标文件的保存
- Matplotlib配置


来看个调整参数的figure
```
# coding: utf-8

from numpy.random import randn
import matplotlib.pyplot as plt

plt.subplots_adjust(left=None, bottom=None, right=None, top=None, wspace=None, hspace=None)
fig, axes = plt.subplots(2, 2, sharex=True, sharey=True)
for i in range(2):
    for j in range(2):
        axes[i, j].hist(randn(500), bins=50, color='k', alpha=0.5)
plt.subplots_adjust(wspace=0, hspace=0)

plt.show()

```
<img src="/img/Python/Matplotlib/matplotlib_15.png" alt="" width="320" height="640" />



```
import matplotlib.pyplot as plt

x = [1, 2, 3, 4]
y = [5, 4, 3, 2]

# 线的风格是：--  ，线的颜色是：green
plt.plot(x, y, linestyle='--', color='g')
plt.show()

```
<img src="/img/Python/Matplotlib/matplotlib_16.png" alt="" width="320" height="640" />



```
import matplotlib.pyplot as plt

# 我们把这个样式合并的写法
# 连接线的风格是：--  ，标记点是：o，线的颜色是：k，表示黑色，
plt.plot(randn(30).cumsum(), 'ko--')

# 如果这样写，效果也是一样的，样式分开写
# plt.plot(randn(30).cumsum(), color='k', linestyle='dashed', marker='o')

plt.show()
```
<img src="/img/Python/Matplotlib/matplotlib_17.png" alt="" width="320" height="640" />



根据读取数据设置线的颜色，线的风格
```
# coding: utf-8

from numpy.random import randn
import matplotlib.pyplot as plt

data = randn(30).cumsum()
plt.plot(data, linestyle='--', label='Default', color='g')
plt.plot(data, linestyle='-', drawstyle='steps-post', label='steps-post', color='r')
plt.legend(loc='best')

plt.show()

```
<img src="/img/Python/Matplotlib/matplotlib_18.png" alt="" width="320" height="640" />




设置标题、轴标签、刻度以及刻度标签
```
# coding: utf-8

from numpy.random import randn
import matplotlib.pyplot as plt


fig = plt.figure()

# 添加一个图在figure上
ax = fig.add_subplot(1, 1, 1)

ax.set_title('My Matplotlib Plot')
ax.set_xlabel('Stages')

# 添加数据点是随机1000以内
ax.plot(randn(1000).cumsum())

# 设置刻度在0到1000
ticks = ax.set_xticks([0, 250, 500, 750, 1000])

# 设置刻度标签
labels = ax.set_xticklabels(['one', 'two', 'three', 'four', 'five'], rotation=30, fontsize='small')

# 绘制
plt.show()
```
<img src="/img/Python/Matplotlib/matplotlib_19.png" alt="" width="320" height="640" />



## 添加图例
```
# coding: utf-8

from numpy.random import randn
import matplotlib.pyplot as plt


fig = plt.figure()

# 在figure窗口，添加一个图
ax = fig.add_subplot(1, 1, 1)

# 值为随机1000以内，添加蓝色场景，标签为：one，线样式：默认
ax.plot(randn(1000).cumsum(), 'b', label='one')

# 值为随机1000以内，添加绿色场景，标签为：two，线样式：--
ax.plot(randn(1000).cumsum(), 'g--', label='two')

# 值为随机1000以内，添加红色场景，标签为：two，线样式：.
ax.plot(randn(1000).cumsum(), 'r.', label='three')

# 让系统在最佳位置添加图例，位置会随着数据不同而显示在不同的位置
ax.legend(loc='best')

plt.show()
```
<img src="/img/Python/Matplotlib/matplotlib_20.png" alt="" width="320" height="640" /> 
<img src="/img/Python/Matplotlib/matplotlib_21.png" alt="" width="320" height="640" />




## 注释以及在subplot上绘图
```
import pandas as pd
import matplotlib.pyplot as plt
from datetime import datetime


fig = plt.figure()

ax = fig.add_subplot(1, 1, 1)

data = pd.read_csv('matplotlib/spx.csv', index_col=0, parse_dates=True)
spx = data['SPX']
print spx
'''
1990-02-01     328.79
1990-02-02     330.92
1990-02-05     331.85
......
Name: SPX, Length: 5472, dtype: float64
'''

spx.plot(ax=ax, style='k--')

# 需要标记出来的日期和label
crisis_data = [
    (datetime(2007, 10, 11), 'Peak of bull market'),
    (datetime(2008, 3, 12), 'Bear Stearns Fails'),
    (datetime(2008, 9, 15), 'Lehman Bankruptcy'),
]

# 通过遍历对crisis_data里匹配到的数据打上标签，并且对这三个标签设置箭头
for date, label in crisis_data:
    ax.annotate(label,
                xy=(date, spx.asof(date) + 50),
                xytext=(date, spx.asof(date) + 200),
                arrowprops=
                dict(facecolor='blue'),
                horizontalalignment='left',
                verticalalignment='top')

# 设置x方向的值，起始位置到结束位置
ax.set_xlim(['1/1/2007', '1/1/2011'])

# 设置y方向的值，起始位置到结束位置
ax.set_ylim([600, 1800])

ax.set_title('Important dates in 2008-2009 financial crisis')

plt.show()
```
<img src="/img/Python/Matplotlib/matplotlib_22.png" alt="" width="640" height="960" />



```
import matplotlib.pyplot as plt

# 添加一个窗口
fig = plt.figure()

# 添加一个场景
ax = fig.add_subplot(1, 1, 1)

# 添加矩形
rect = plt.Rectangle((0.2, 0.75), 0.4, 0.15, color='k', alpha=0.3)

# 添加圆
circ = plt.Circle((0.7, 0.2), 0.15, color='b', alpha=0.3)

# 添加三角形
pgon = plt.Polygon([[0.15, 0.15], [0.35, 0.4], [0.2, 0.6]], color='g', alpha=0.5)

ax.add_patch(rect)
ax.add_patch(circ)
ax.add_patch(pgon)

plt.show()
```
<img src="/img/Python/Matplotlib/matplotlib_23.png" alt="" width="640" height="960" />




## 图标的保存

```
# 保存到指定路径
# 保存时plt.show()不要调用，保存后可以调用plt.show()
fig.savefig('/Users/victorzhang/Desktop/figpath.png')
fig.savefig('/Users/victorzhang/Desktop/figpath1.png', dpi=400, bbox_inches='tight')
```

保存到内存
```
from io import BytesIO

buffer = BytesIO()
plt.savefig(buffer)
plot_data = buffer.getvalue()
print plot_data
```

## plot的一些设置

```
plt.rc('figure', figsize=(10, 10))
font_options = {
    "family": "Monospace",
    "weight": "bold",
    "size": "20"
}
plt.rc('font', **font_options)

```

纯matplotlib代码编写图形需要设置很多参数，比较麻烦，所以我们有了pandas来一起构建


<br/>
<br/>

## pandas的绘图函数
- label
- ax
- style
- alpha
- kind
- logy
- use_index
- rot
- xticks
- yticks
- xlim
- ylim
- grid
- subplots
- sharex
- sharey
- figsize
- title
- legend          添加图例（默认为True）
- sort_columns

```
import numpy as np
import matplotlib.pyplot as plt
from numpy.random import randn
from pandas import DataFrame, Series


s = Series(randn(10).cumsum(), index=np.arange(0, 100, 10))
s.plot()

plt.show()

```
<img src="/img/Python/Matplotlib/matplotlib_24.png" alt="" width="640" height="960" />


```
import numpy as np
import matplotlib.pyplot as plt
from numpy.random import randn
from pandas import DataFrame, Series

df = DataFrame(np.random.randn(10, 4).cumsum(0),
               columns=['A', 'B', 'C', 'D'],
               index=np.arange(0, 100, 10))
df.plot()

plt.show()
```
<img src="/img/Python/Matplotlib/matplotlib_25.png" alt="" width="640" height="960" />



## 柱状图
```
import matplotlib.pyplot as plt
from numpy.random import randn
from pandas import DataFrame, Series

fig, axes = plt.subplots(2, 1)
data = Series(randn(16), index=list('abcdedghijklmnop'))
data.plot(kind='bar', ax=axes[0], color='k', alpha=0.7)
data.plot(kind='barh', ax=axes[1], color='k', alpha=0.7)

plt.show()
```
<img src="/img/Python/Matplotlib/matplotlib_26.png" alt="" width="640" height="960" />



```

import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from pandas import DataFrame, Series

df = DataFrame(np.random.rand(6, 4),
               index=['one', 'two', 'three', 'four', 'five', 'six'],
               columns=pd.Index(['A', 'B', 'C', 'D'],
               name='Genus'))
print df
'''
Genus         A         B         C         D
one    0.603944  0.634066  0.400164  0.305856
two    0.860118  0.090741  0.159745  0.439690
three  0.083134  0.789508  0.602428  0.197421
four   0.039622  0.458276  0.239215  0.297469
five   0.170553  0.455839  0.901505  0.080372
six    0.681694  0.628299  0.657260  0.644590
'''

df.plot(kind='bar')
plt.show()
```
<img src="/img/Python/Matplotlib/matplotlib_27.png" alt="" width="640" height="960" />


```
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from pandas import DataFrame, Series

df = DataFrame(np.random.rand(6, 4),
               index=['one', 'two', 'three', 'four', 'five', 'six'],
               columns=pd.Index(['A', 'B', 'C', 'D'],
               name='Genus'))
print df

# 每一个类型都有四段，因为stacked=True，所以最后后堆积在一起
df.plot(kind='barh', stacked=True, alpha=0.5)

plt.show()
```
<img src="/img/Python/Matplotlib/matplotlib_28.png" alt="" width="640" height="960" />


```
# coding: utf-8

import pandas as pd
import matplotlib.pyplot as plt


tips = pd.read_csv('matplotlib/tips.csv')

# 交叉表
party_counts = pd.crosstab(tips.day, tips['size'])
print party_counts
'''
size  1   2   3   4  5  6
day                      
Fri   1  16   1   1  0  0
Sat   2  53  18  13  1  0
Sun   0  39  15  18  3  1
Thur  1  48   4   5  1  3
'''

party_counts = party_counts.ix[:, 2:5]
print party_counts
'''
size   2   3   4  5
day                
Fri   16   1   1  0
Sat   53  18  13  1
Sun   39  15  18  3
Thur  48   4   5  1
'''

party_pcts = party_counts.div(party_counts.sum(1).astype(float), axis=0)
print party_pcts
'''
size         2         3         4         5
day                                         
Fri   0.888889  0.055556  0.055556  0.000000
Sat   0.623529  0.211765  0.152941  0.011765
Sun   0.520000  0.200000  0.240000  0.040000
Thur  0.827586  0.068966  0.086207  0.017241
'''

party_pcts.plot(kind='bar', stacked=True)

plt.show()
```
<img src="/img/Python/Matplotlib/matplotlib_29.png" alt="" width="640" height="960" />




##  直方图和密度图

两个分散在不同的plot的图
```
# coding: utf-8

import pandas as pd
import matplotlib.pyplot as plt

fig, axes = plt.subplots(2, 1)

tips = pd.read_csv('matplotlib/tips.csv')

tips['tip_pct'] = tips['tip'] / tips['total_bill']

# 直方图
tips['tip_pct'].hist(bins=50, ax=axes[0])

# 密度图
tips['tip_pct'].plot(kind='kde', ax=axes[1])

plt.show()
```
<img src="/img/Python/Matplotlib/matplotlib_30.png" alt="" width="640" height="960" />


两个在一起的plot的图
```
# coding: utf-8

import numpy as np
import matplotlib.pyplot as plt
from pandas import DataFrame, Series

compl1 = np.random.normal(0, 1, size=200)
compl2 = np.random.normal(10, 2, size=200)
values = Series(np.concatenate([compl1, compl2]))
print values
'''
0      -1.080911
1      -0.522424
2      -1.225437
3       0.755538
......
Length: 400, dtype: float64
'''

# 直方图
values.hist(bins=100, alpha=0.3, color='k', normed=True)

# 密度图
values.plot(kind='kde', style='k--')

plt.show()

```
<img src="/img/Python/Matplotlib/matplotlib_31.png" alt="" width="640" height="960" />




## 散点图
```
# coding: utf-8

import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

macro = pd.read_csv('matplotlib/macrodata.csv')
data = macro[['cpi', 'm1', 'tbilrate', 'unemp']]
trans_data = np.log(data).diff().dropna()
print trans_data[-5:]
'''
          cpi        m1  tbilrate     unemp
198 -0.007904  0.045361 -0.396881  0.105361
199 -0.021979  0.066753 -2.277267  0.139762
200  0.002340  0.010286  0.606136  0.160343
201  0.008419  0.037461 -0.200671  0.127339
202  0.008894  0.012202 -0.405465  0.042560
'''

plt.scatter(trans_data['m1'], trans_data['unemp'])
plt.title('Changes in log %s vs. log %s' % ('m1', 'unemp'))

plt.show()
```
<img src="/img/Python/Matplotlib/matplotlib_32.png" alt="" width="640" height="960" />


```
# coding: utf-8

import numpy as np
import pandas as pd
import matplotlib.pyplot as plt


macro = pd.read_csv('matplotlib/macrodata.csv')
data = macro[['cpi', 'm1', 'tbilrate', 'unemp']]
trans_data = np.log(data).diff().dropna()
print trans_data[-5:]
'''
          cpi        m1  tbilrate     unemp
198 -0.007904  0.045361 -0.396881  0.105361
199 -0.021979  0.066753 -2.277267  0.139762
200  0.002340  0.010286  0.606136  0.160343
201  0.008419  0.037461 -0.200671  0.127339
202  0.008894  0.012202 -0.405465  0.042560
'''

pd.plotting.scatter_matrix(trans_data, diagonal='kde', color='k', alpha=0.3)

plt.show()
```
<img src="/img/Python/Matplotlib/matplotlib_33.png" alt="" width="640" height="960" />



## 误差条形图
```
# coding: utf-8

import matplotlib.pyplot as plt
import numpy as np

x = np.arange(0, 10, 1)
y = np.log(x)
xe = 0.1 * np.abs(np.random.randn(len(y)))

plt.bar(x, y, yerr=xe, width=0.4, align='center', ecolor='r', color='cyan', label='experiment #1')

plt.xlabel('# measurement')
plt.ylabel('Measured values')
plt.title('Measurements')
plt.legend(loc='upper left')

plt.show()
```
<img src="/img/Python/Matplotlib/matplotlib_34.png" alt="" width="640" height="960" />




## 饼图
```
# coding: utf-8

import matplotlib.pyplot as plt

plt.figure(1, figsize=(8, 8))
ax = plt.axes([0.1, 0.1, 0.8, 0.8])

labels = 'Spring', 'Summer', 'Autumn', 'Winter'
values = [15, 16, 16, 18]
explode = [0.1, 0.1, 0.1, 0.1]

plt.pie(values, explode=explode, labels=labels, autopct='%1.1f%%', startangle=67)

plt.title('Rainy days by season')

plt.show()
```
<img src="/img/Python/Matplotlib/matplotlib_35.png" alt="" width="640" height="960" />



## 等高线图
```
# coding: utf-8

import matplotlib as mpl
import matplotlib.pyplot as plt
import numpy as np

def process_signals(x, y):
    return (1 - (x ** 2 + y ** 2)) * np.exp(-y ** 3 / 3)

x = np.arange(-1.5, 1.5, 0.1)
y = np.arange(-1.5, 1.5, 0.1)

X, Y = np.meshgrid(x, y)
Z = process_signals(X, Y)

N = np.arange(-1, 1.5, 0.3)

CS = plt.contour(Z, N, linewidths=2, cmap=mpl.cm.jet)
plt.clabel(CS, inline=True, fmt='%1.1f', fontsize=10)
plt.colorbar(CS)

plt.title('My Function: $z=(1-x^2+y^2) e^{-(y^3) / 3}$')

plt.show()
```
<img src="/img/Python/Matplotlib/matplotlib_36.png" alt="" width="640" height="960" />




## 3D图标
- 3D柱状图
- 3D直方图


#### 3D柱状图
```
# coding: utf-8

import matplotlib as mpl
import matplotlib.pyplot as plt
import numpy as np
from mpl_toolkits.mplot3d import Axes3D

mpl.rcParams['font.size'] = 10

fig = plt.figure()
ax = fig.add_subplot(111, projection='3d')

for z in [2011, 2012, 2013, 2014]:
    xs = xrange(1, 13)
    ys = 1000 * np.random.rand(12)

    color = plt.cm.Set2(np.random.choice(xrange(plt.cm.Set2.N)))
    ax.bar(xs, ys, zs=z, zdir='y', color=color, alpha=0.8)

ax.xaxis.set_major_locator(mpl.ticker.FixedLocator(xs))
ax.yaxis.set_major_locator(mpl.ticker.FixedLocator(ys))

ax.set_xlabel('Month')
ax.set_ylabel('Year')
ax.set_zlabel('Sales Net [usd]')

plt.show()
```
<img src="/img/Python/Matplotlib/matplotlib_37.png" alt="" width="640" height="960" />


#### 3D直方图
```
# coding: utf-8

import matplotlib as mpl
import matplotlib.pyplot as plt
import numpy as np
from mpl_toolkits.mplot3d import Axes3D


mpl.rcParams['font.size'] = 10

samples = 25

x = np.random.normal(5, 1, samples)
y = np.random.normal(3, .5, samples)

fig = plt.figure()
ax = fig.add_subplot(211, projection='3d')

hist, xedges, yedges = np.histogram2d(x, y , bins=10)

elements = (len(xedges) - 1) * (len(yedges) - 1)
xpos, ypos = np.meshgrid(xedges[:-1] + .25, yedges[:-1] + .25)

xpos = xpos.flatten()
ypos = ypos.flatten()
zpos = np.zeros(elements)

dx = .1 * np.ones_like(zpos)
dy = dx.copy()

dz = hist.flatten()

ax.bar3d(xpos, ypos, zpos, dx, dy, dz, color='b', alpha=0.4)
ax.set_xlabel('X Axis')
ax.set_ylabel('Y Axis')
ax.set_zlabel('Z Axis')

ax2 = fig.add_subplot(212)
ax2.scatter(x, y)
ax2.set_xlabel('X Axis')
ax2.set_ylabel('Y Axis')

plt.show()
```
<img src="/img/Python/Matplotlib/matplotlib_38.png" alt="" width="640" height="960" />

