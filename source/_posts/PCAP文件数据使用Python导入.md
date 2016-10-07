---
title: PCAP文件数据使用Python导入
date: 2016-08-17 07:06:06
tags:
  - security
  - python
---
# PCAP文件使用Python导入

首先开始学习的是[data_hacking](http://clicksecurity.github.io/data_hacking/)这个项目里的[PCAP Exploration (BSidesATX 2014)](https://github.com/ClickSecurity/data_hacking/tree/master/contagio_traffic_analysis)

这个项目做的事情主要是根据Bro获取到的PCAP文件进行一些分析工作。

## PCAP文件
PCAP文件是wireshark，Bro等网络嗅探工具针对网络接口、端口和协议的数据包截取的一种保存形式。

## python遍历目录树
使用os.walk函数
使用语法格式如下
```
os.walk(top, topdown=True, onerror=None, followlinks=False)
```
函数的返回值是一个含有三个元素的元组：(dirpath, dirnames, filenames)
eg1:
```
# 得到目录下非目录的文件（除CSV文件外所占的字节数）
import os
from os.path import getsize, join
for root, dirs, files in os.walk(".")
  print root, "consumes",
  # join(root, name)获得完整路径
  print sum(getsize(join(root, name))) for name in files,
  print "bytes in" len(files), "non-directory files"
  if 'CSV' in dirs:
    dirs.remove('CSV')
```
eg2:
```
# 删除一个文件夹下所有文件
# 使用topdown=False让目录遍历从底部开始是因为os.rmdir只能删除空文件夹
for root, dirs, files in os.walk(".", topdown=False)
  for name in files:
    os.remove(os.path.join(root, name))
  for name in dirs:
    os.rmdir(os.path.join(root, name))
```

## 项目分析

参考

https://github.com/ClickSecurity/data_hacking/tree/master/contagio_traffic_analysis
https://github.com/ClickSecurity/data_hacking/tree/master/contagio_traffic_analysis
https://docs.python.org/2/library/os.html
