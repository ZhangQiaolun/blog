---
title: windows下利用Anaconda搭建python2和python3共存
date: 2016-08-16 16:22:07
tags: python
---
# windows下利用Anaconda搭建python2和python3共存
windows下不同的python版本共存一直是一个纠结人的问题。python有些库的安装有比较麻烦，整天折腾工具是一件很浪费时间的事。

而Anaconda整合了很多python的库。在Anaconda里，python和其他的库一样，只是一个插件。搭建不同版本的python就比较简单。

## 官方不同python版本环境的搭建
### 查看已安装版本
```
conda info envs
```
```
python --version #正在使用的版本
```
### 安装不同的版本
```
conda create --name snakes python=3 #安装python
```
这时会发现在Anaconda的安装路径的envs路径下会多出来一个snakes文件夹。
### 使用另一版本的python
linux：
```
source activate snakes
```
```
windows：activate snakes
```
### 回到默认版本的python
```
deactivate
```

## 很便捷的不同python版本环境的搭建

*上面虽然可以创建不同版本的python环境，但安装新包网速着实慢。。。*

首先根据使用最多的python版本，安装对应的Anaconda安装包，然后再把另一个python版本的Anaconda安装到第一个Anaconda安装路径下的envs文件夹即可，详见参考网址。
在envs下的安装目录名就是之后使用它的名字。
eg：
我的安装路径是
```
C:\Anaconda3\envs\py2
```
使用方式：
```
source activate py2
deactivate
```
如果某些包Anaconda没有收录，可转换到对应的python版本后，直接pip install sth，安装后的就是对应版本的python的包。


参考
http://conda.pydata.org/docs/py2or3.html
http://blog.csdn.net/infin1te/article/details/50445217
