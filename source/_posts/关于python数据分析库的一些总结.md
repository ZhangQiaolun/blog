---
title: matplotlib与pylab区分及matplotlib入门
date: 2016-08-20 10:57:01
tags:
  - python
---

## matplotlib,pyplot和pylab：他们是怎么联系在一起的？
matplotlib是一个完整（whole）的库，matplotlib.pyplot是matplotlib是matplotlib中的一个库；pylab是安装matplotlib时一起安装的。

pyplot给底层的基于对象的画图库(object-oriented plotting library)提供了一个状态机界面(state-machine interface)。

导入pylab时，会将matplotlib.pyplot(用于作图)和numpy(用于数学和向量)导入到一个作用域里（name space）。

*虽然很多例子用到了pylab，它不再被推荐。对于不用交互式的作图，推荐使用pyplot创建图像然后使用OO界面作图。*
```
#!/usr/bin/env python
# -*- noplot -*-
"""
A pure OO (look Ma, no pylab!) example using the agg backend
"""
from matplotlib.backends.backend_agg import FigureCanvasAgg as FigureCanvas
from matplotlib.figure import Figure

fig = Figure()
canvas = FigureCanvas(fig)
ax = fig.add_subplot(111)
ax.plot([1, 2, 3])
ax.set_title('hi mom')
ax.grid(True)
ax.set_xlabel('time')
ax.set_ylabel('volts')
canvas.print_figure('test')
```
下面是两个例子,在python中可以用两种方式作图，推荐第一种。
```
import matplotlib
matplotlib.pyplot(...)
```
```
# 推荐这一种
import pylab
pylab.plot(...)
```

## matplotlib入门
以下内容翻译自[matplotlib usage](http://matplotlib.org/faq/usage_faq.html#matplotlib-pyplot-and-pylab-how-are-they-related)
### 大体概念
matplotlib因为有着大量的代码库让很多的新手望而生畏。然而掌握了一些相对简单的概念框架和一些要点，matplotlib中的大部分都可以被理解。matplotlib中所有的东西都用层次化来组织。在最高层是matplotlib.pyplot模块提供的状态机环境（state-machine environment)，在这一层，简单地函数被用来向现在的图像(figure)中的axes添加作图成分(线条，图像和文字等)

下一层是面向对象界面，在这里pyplot之辈用于几个简单函数（比如说创建图像），用户显式（explicit）创建并跟踪figure和axes对象。在这一层，用户用pyplot创建figures，通过figures，可以创建一个或多个axes对象。这些axes对象接着被用于大部分的作图操作。


### 图像的构成部分
![](关于python数据分析库的一些总结/figure.png)
### 作图函数的输入方式
所有的作图函数输入格式为np.array或np.ma.masked_array
```
# 转化panda.DataFrame
a = pandas.DataFrame(np.random.rand(4, 5), columns = list('abcde'))
a.asndarray = a.values
```
```
# 转化np.matrix
b = np.matrix([1, 2], [3, 4])
b_asarray = np.asarray(b)
```
### 编程风格
以下是推荐的一种风格：pyplot style
```
import matplotlib.pyplot as plt
import numpy as np
x = np.arange(0, 10, 0.2)
y = np.sin(x)
fig = plt.figure()
ax = fig.add_subplot(111)
ax.plot(x, y)
plt.show()
```
有时人们会发现自己在一次次用不同的数据集做相同的作图操作，下面是推荐的操作。
```
def my_plotter(ax, data1, data2, param_dict):
    """
    A helper function to make a graph

    Parameters
    ----------
    ax : Axes
        The axes to draw to

    data1 : array
       The x data

    data2 : array
       The y data

    param_dict : dict
       Dictionary of kwargs to pass to ax.plot

    Returns
    -------
    out : list
        list of artists added
    """
    out = ax.plot(data1, data2, **param_dict)
    return out
```
然后
```
fig, ax = plt.subplots(1, 1)
my_plotter(ax, data1, data2, {'marker':'x'})
# or if you wanted to have 2 sub-plots:
fig, (ax1, ax2) = plt.subplots(1, 2)
my_plotter(ax1, data1, data2, {'marker':'x'})
my_plotter(ax2, data3, data4, {'marker':'o'})
```
### 什么是backend
matplotlib有不同的用途和输出模式：一些人用matplotlib从python shell里动态输入并让画图窗口在输入时弹出，有些人把它整合进图形界面（wxpython，pygtk等）。为了支持所有的用途，matplotlib可以面向不同的输出，每一种都称为一个backend，frontend称为用户面对的代码（比如画图代码）
### 什么是交互模式
作图窗口是否出现，什么时候出现，一个图像在屏幕上画出来之后一个脚本是否继续，取决于调用的函数和方法，以及一个决定matplotlib是否处于交互模式(interactive mode)的变量。这个布尔型变量可以用matplotlibrc文件来设定。
```
# 打开交互模式
matplotlib.pyplot.ion()
# 第二种打开交互模式的方法
matplotlib.interactive()
# 查询交互模式
matplotlib.is_interactive()
# 关闭交互模式
matplotlib.pyplot.ioff()
```
#### interactive example(交互模式例子)
```
import matplotlib.pyplot as plt
plt.ion()
plt.plot([1.6, 2.7])
```
现在可以看见图像，同时可以添加元素
```
plt.title("interactive test")
plt.xlabel("index")
```
但是下面的代码却没有作用
```
ax = plt.gca()
ax.plot([3.1, 2.2])
```
原因在于Axes方法不会自动调用draw_if_interactive()，这时可以调用draw()来显示图像
```
plt.draw()
```
#### Non-interactive 例子(非交互模式例子)
```
import matplotlib.pyplot as plt
plt.ioff()
plt.plot([1.6, 2.7])
```
现在没有显示图像，需要下面的代码来显示
```
plt.show()
```
这时，终端没有反应，因为只有关闭了显示图像的窗口才能输入其他的命令。同时，这个模式延迟了所有作图操作，直到show()被调用。这比起代码中给一条线增加一个新特性就要重新作图来的有效。
```
# 这是新版本的特性，可以调用多次plt.show()
# 下面代码会创建3个图像，一次一个
import numpy as np
import matplotlib.pyplot as plt
plt.ioff()
for i in range(3):
    plt.plot(np.random.rand(10))
    plt.show()
```
#### 总结
交互模式draw(), 非交互模式show()

### 一个遇到的小问题
- 问题1:fig.add_subplot(111)中111是什么意思
```
import matplotlib.pyplot as plt
x = [1, 2, 3, 4, 5]
y = [1, 4, 9, 16, 25]
fig = plt.figure()
fig.add_subplot(111)
plt.scatter(x, y)
plt.show()
```
上面的代码中
```
fig.add_subplot(111)
```
111表示1x1grid，firstsubplot。更多的：“234”代表"2x3 grid, 4th subplot"。也可以写成`add_subplot(1,1,1)`
```
# 一个四方格的图像
import matplotlib.pyplot as plt
fig = plt.figure()
fig.add_subplot(221)   #top left
fig.add_subplot(222)   #top right
fig.add_subplot(223)   #bottom left
fig.add_subplot(224)   #bottom right
plt.show()
```
```
# 左边两个合在一起
subplot(2,2,[1 3])
subplot(2,2,2)
subplot(2,2,4)
# 上面两个合在一起
subplot(2,2,1:2)
subplot(2,2,3)
subplot(2,2,4)
```
- 问题2：plt.subplots()的用法
```
fig, ax = plt.subplot()
# 等同于
fig = plt.figure()
ax = fig.add_subplot(111)
```
plt.subplot返回一个元组，包含了figure和axes对象。fig接下来可以用于修改图像层面的特性或者是保存图像为图像文件(e.g. with `fig.savefig('youfilename.png')`)

参考

http://stackoverflow.com/questions/12987624/confusion-between-numpy-scipy-matplotlib-and-pylab
http://stackoverflow.com/questions/16849483/which-is-the-recommended-way-to-plot-matplotlib-or-pylab
http://stackoverflow.com/questions/11469336/what-is-the-difference-between-pylab-and-pyplot
http://matplotlib.org/faq/usage_faq.html#matplotlib-pyplot-and-pylab-how-are-they-related
http://matplotlib.org/examples/api/agg_oo.html
http://stackoverflow.com/questions/3584805/in-matplotlib-what-does-the-argument-mean-in-fig-add-subplot111
http://stackoverflow.com/questions/34162443/why-do-many-examples-use-fig-ax-plt-subplots-in-matplotlib-pyplot-python
