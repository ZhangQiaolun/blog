---
title: 'Python: Lambda'
date: 2016-08-19 17:06:22
tags:
  - python
---
本文翻译自[Python:Lambda Function](http://www.secnetix.de/olli/Python/lambda_functions.hawk)

[本文地址](http://love.myyoulemei.com/2016/08/19/Python-Lambda/)

Python可以用“lambda”创建匿名函数。在Python里，lambda和函数型编程语言中并不相同，但是由于它整合进了Python并经常和filter(),map(),reduce()一起使用。

下面的代码展示了一个普通的函数定义("f")和一个lambda函数("g")
```
>>> def f(x): return x**2
...
>>> print f(8)
64
>>> g = lambda x: x**2
>>>
>>> print g(8)
64
```
正如你看到的，f()和g()功能相同，使用方式相同。需要注意：lambda定义中并没有包括"return"声明--它总是包括了一个被返回的表达式。另外你可以把lambda放在任何需要函数的地方，你根本不需要把它赋给一个变量。
```
>>> def make_incrementor (n): return lambda x: x + n
>>>
>>> f = make_incrementor(2)
>>> g = make_incrementor(6)
>>>
>>> print f(42), g(42)
44 48
>>>
>>> print make_incrementor(22)(33)
55
```
上面的代码定义了一个函数"make_incrementor",这个函数创建并返回了一个匿名的函数。这个返回的函数将它的自变量加上声明的值。

现在你可以创建不同的增加函数并给他们赋上不同的值。正如最后一条表达式，你甚至不需要在任何地方定义一个函数--你可以直接使用，并在不需要它的时候忘了它。

下面的代码更加深入
```
>>> foo = [2, 18, 9, 22, 17, 24, 8, 12, 27]
>>>
>>> print filter(lambda x: x % 3 == 0, foo)
[18, 9, 24, 12, 27]
>>>
>>> print map(lambda x: x * 2 + 10, foo)
[14, 46, 28, 54, 44, 58, 26, 34, 64]
>>>
>>> print reduce(lambda x, y: x + y, foo)
139
```
首先我们定一个了一个整型的数组，然后用标准的函数filter(),map()和reduce()对这个数组做不同的操作。这三个函数都需要两个参数：一个函数和一个数组。

当然，我们可以单独定义一个函数并将这个函数名作为filter()等的一个参数。事实上如果我们需要多次用这个函数或者这个函数用一行来写太复杂了，这样做或许会是一个好方法。然而，如果我们只需要一次这个函数并且这个函数非常简单（比如只包含了一个表达式）。用一个lambda结构去表示一个匿名的函数并把这个函数like传递给filter()简便的多。这可以创造出紧凑但是具有可读性的代码。

在第一个函数里，数组里的每一个元素都执行了lambda函数，返回的是一个只包含原来数组中使函数返回值为真的元素。在这个例子中，我们得到了一个所有的元素都是3的倍数的数组。

在第2个例子中，map()函数被用来转化数组。原来数组中的每一个元素都执行给定的函数，返回值是原来数组钟的元素从lambda函数中返回值所构成的数组。在这个例子中，它将每一个元素都执行了2 * X + 10操作。

最后，reduce()是比较特殊的。"worker function"(作为参数传递的那个匿名函数)必须接受两个参数（在这里是x和y）。这个匿名函数首先调用数组里的前两个元素，然后调用返回值和第三个元素，以此类推，知道数组里的元素全部被调用。这意味着如果数组中有n个元素，这个匿名函数就会调用n-1次，reduce()的结果是最后一次调用匿名函数的返回值。在上面的例子中，它只是简单地将所有元素相加，因此我们得到的是所有元素的和。（需要注意的是，python中内置函数sum()比这种方式效率更高）
```
>>> nums = range(2, 50)
>>> for i in range(2, 8):
...     nums = filter(lambda x: x == i or x % i, nums)
...
>>> print nums
[2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47]
```
这是计算素数的方式之一

```
>>> sentence = 'It is raining cats and dogs'
>>> words = sentence.split()
>>> print words
['It', 'is', 'raining', 'cats', 'and', 'dogs']
>>>
>>> lengths = map(lambda word: len(word), words)
>>> print lengths
[2, 2, 7, 4, 3, 4]
```
计算句子中每个单词的长度。

上面的代码可以写得更短
```
>>> print map(lambda w: len(w), 'It is raining cats and dogs'.split())
[2, 2, 7, 4, 3, 4]
```

下面是一个UNIX编程里的例子：我们想要找到文件系统中的挂载点（mount point？）。为了达到目的，我们执行了外界的命令"mount"并解析结果。
```
>>> import commands
>>>
>>> mount = commands.getoutput('mount -v')
>>> lines = mount.splitlines()
>>> points = map(lambda line: line.split()[2], lines)
>>>
>>> print points
['/', '/var', '/usr', '/usr/local', '/tmp', '/proc']
```
commands模块中的getoutput函数(Python标准库的一部分)执行命令并把结果作为一个字符串返回。因此，我们先把它分割成单独的句子。最后我们用带lambda的"map"函数对每一行操作并返回结果中的第三个元素（就是mountpoint）。

同样的，这也可以写成一句。
```
print map(lambda x:x.split()[2], commands.getoutput('mount -v').splitlines())
```

在实际应用的代码中，推荐分割复杂的表达式，应为这样更容易阅读，同时也便于更改。

然而，把一个命令的输出分割成一个由行组成的数组是非常常见的。每次分析外部的命令是你都需要它。因此，通常将分割操作包括到getgoutput里，但是其他的操作另外做。这是紧凑与可读性的协调。
```
>>> lines = commands.getoutput('mount -v').splitlines()
>>>
>>> points = map(lambda line: line.split()[2], lines)
>>> print points
['/', '/var', '/usr', '/usr/local', '/tmp', '/proc']
```
一个更好的方法是为这个操作写一个小函数，将执行命令和分割输出包装到函数中。

```
# 这也是一种方法
>>> lines = commands.getoutput('mount -v').splitlines()
>>>
>>> points = [line.split()[2] for line in lines]
>>> print points
['/', '/var', '/usr', '/usr/local', '/tmp', '/proc']
```

在新版本中，comands.getoutput()函数被subprocess.check_output()代替
