---
title: data_hacking_practice_3
date: 2016-08-19 14:52:52
tags:
  - python
  - pandas
  - security
---

## 这次要分析的代码如下
```
for threat in ['APT', 'CRIME']:
    subset = conndf[conndf['threat'] == threat][['date','sample']]
    subset['count'] = 1
    pivot = pd.pivot_table(subset, values='count', rows=['date'], cols=['sample'], fill_value=0)
    by = lambda x: lambda y: getattr(y, x)
    grouped = pivot.groupby([by('year'),by('month')]).sum()

    ax = grouped.plot()
    pylab.ylabel('Connections')
    pylab.xlabel('Date Recorded')
    patches, labels = ax.get_legend_handles_labels()
    ax.legend(loc='upper center', bbox_to_anchor=(0.5, -0.15), ncol=2, title="Sample Name")
```
代码涉及到
- pandas：groupby, pivot
- pylab
- lambda

由于这些对我来说也全是新内容，所以我还有点一知半解，暂时将我懂得东西记录下来，边做边学。

## 知识梳理
### pandas：groupby
### pandas：pivot
![](data-hacking-practice-3/pivot_table.png)
### pylab入门
### lambda
## 代码分析
```
for threat in ['APT', 'CRIME']:
    subset = conndf[conndf['threat'] == threat][['date','sample']]
    subset['count'] = 1
```
这一部分做的事情是将conndf这个dataFrame中威胁为'APT'和'CRIME'的分别做处理，将其中的date和sample分离出来，得到的subset依次是

图1
![](data-hacking-practice-3/subset1.png)
图2
![](data-hacking-practice-3/subset2.png)
```
# 原作者代码（pandas老版本）
pivot = pd.pivot_table(subset, values='count', rows=['date'], cols=['sample'], fill_value=0)
# 我的代码（pandas新版本）
pivot = pd.pivot_table(subset, values='count', index='date', columns='sample', fill_value=0)
```

```
by = lambda x: lambda y: getattr(y, x)
```
下面是stackoverflow上一个人给我的回答，返回的会是一个函数
by = lambda x: lambda y: getattr(y, x) is equivalent to the following:
```
def by(x):
    def getter(y):
        return getattr(y, x)
    return getter
```

getattr(a, b) gets an attribute with the name b from an object named a.

So by('bar') returns a function that returns the attribute 'bar' from an object.

by('bar')(foo) means getattr(foo, 'bar') which is roughly foo.bar.
```
grouped = pivot.groupby([by('year'),by('month')]).sum()
```
```

ax = grouped.plot()
pylab.ylabel('Connections')
pylab.xlabel('Date Recorded')
patches, labels = ax.get_legend_handles_labels()
ax.legend(loc='upper center', bbox_to_anchor=(0.5, -0.15), ncol=2, title="Sample Name")
```
下面是对这里的一行代码
```
grouped = pivot.groupby([by('year'), by('month')]).sum
```
当一个可调用的传递给`groupby`的时候，作用在`DataFrame`的index上，所以这个代码is grouping by the year and month of a datetimelike index。
下面又是一个例子
```
In [55]: df = pd.DataFrame({'a': 1.0},
                           index=pd.date_range('2014-01-01', periods=13, freq='M'))

In [56]: df.groupby([by('year'), by('month')]).sum()
Out[56]:
           a
2014 1   1.0
     2   1.0
     3   1.0
     4   1.0
     5   1.0
     6   1.0
     7   1.0
     8   1.0
     9   1.0
     10  1.0
     11  1.0
     12  1.0
2015 1   1.0
```
更明确地表示
```
In [57]: df.groupby([df.index.year, df.index.month]).sum()
Out[57]:
           a
2014 1   1.0
     2   1.0
     3   1.0
     4   1.0
     5   1.0
     6   1.0
     7   1.0
     8   1.0
     9   1.0
     10  1.0
     11  1.0
     12  1.0
2015 1   1.0
```

接下来是作图
```
ax = grouped.plot()
pylab.ylabel('Connections')
pylab.xlabel('Date Recorded')
patches, labels = ax.get_legend_handles_labels()
ax.legend(loc='upper center', bbox_to_anchor=(0.5, -0.15), ncol=2, title="Sample Name")
```
这里有一句很有意思
`ax.legend(loc='upper center', bbox_to_anchor=(0.5, -0.15), ncol=2, title="Sample Name")`

loc参数是参考位置，bbox_to_anchors是相对坐标轴位置

## 作图基础(matplotlib)

- legend entry

    A legend is made up of one or more legend entries. An entry is made up of exactly one key and one label.
- legend key

    The colored/patterned marker to the left of each legend label.
- legend label

    The text which describes the handle represented by the key.
- legend handle

    The original object which is used to generate an appropriate entry in the legend.

```
line_up, = plt.plot([1,2,3], label='Line 2')
line_down, = plt.plot([3,2,1], label='Line 1')
plt.legend(handles=[line_up, line_down])
```
在一些情况下，不能为handle设置label，因此将label的数组传递给legend()是可能的。
```
line_up, = plt.plot([1,2,3], label='Line 2')
line_down, = plt.plot([3,2,1], label='Line 1')
plt.legend([line_up, line_down], ['Line Up', 'Line Down'])
```
在上面的代码中 ，有`line_up,=plt.plot`，这其中的逗号（comma）并不是输入错误
是因为
> plt.plot returns a list of the Line2D objects plotted, even if you plot only one line. That comma is unpacking the single value into line.

解决方法有两个
>加逗号
```
line, = ax.plot(x, np.sin(x))
# 加括号也可以
(line,) = ax.plot(x, np.sin(x))
```
>取数组第一个元素
```
line = ax.plot(x, np.sin(x))[0]
```

- ax.get_legend_handles_labels

```
# 返回一个数组
handles, labels = ax.get_legend_handles_labels()
ax.legend(handles, labels)
```
- ax.legend

参考：

http://stackoverflow.com/questions/39031255/what-does-by-lambda-x-lambda-y-getattry-x-mean
http://pbpython.com/pandas-pivot-table-explained.html
http://www.secnetix.de/olli/Python/lambda_functions.hawk
http://stackoverflow.com/questions/39053348/pandas-groupby-with-a-lambda-parameter
http://matplotlib.org/users/legend_guide.html
http://stackoverflow.com/questions/16037494/python-code-is-it-comma-operator
