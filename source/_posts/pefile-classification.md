---
title: pefile classification
date: 2016-08-26 09:49:46
tags:
  - python
  - security
---
我接下来的这个项目其实也只是一个项目的分析，会用到一些机器学习的东西,将要做的是PE文件的分类。

将用到的模块有：
- Pandas
- Scikit Learn
- Matplotlib

## PE file


## PE file viewer tools
- Exeinfo PE
一个轻量的更具，可以透露出一些关键的hints，但是不能得出一些更具体的信息。
- PEstudio
有助于快速判断一个样本是否是带有病毒。
- CFF Explorer
可以得到很多信息
- FileAlyzer
这个在了解PE格式后才有用，无法一眼看出
- PEview
比FileAlyzer或许更难看出


## confusion matrix
用于机器学习，尤其是用于归类的时候
## python class
