---
title: data_hacking_practice 2
date: 2016-08-17 14:34:57
tags:
  - python
  - security
---
# data_hacking_practice 2
## 一些帮助函数

### 函数1:将IPv4地址转化为整型
```
def ip2int(addr):
  try:
    return struct.unpack("!I", socket.inet_aton(addr))[0]
  except Exception as e:
    pass
    #print "Error: %s - %s" % (str(e), addr)
  return 0
```
这个函数名其实也挺有意思，ip To int，将ip地址转化为整型。

python的socket模块里有很多对ip地址进行转换的函数。
```
socket.inet_aton(ip_string)
```
将带点号的IPv4地址（eg："123.45.67.89")转化成32-bit packed binary format
```
struct.unpack(fmt, string)
```
struct模块可以对python值，C结构体之间执行转化（具体意思没怎么明白），同时它可以对
binary data进行操作。

上面的函数的作用：将string转化为fmt格式。上面fmt参数中!表示字节顺序（byte order）是
按照network（即小段法big-endian），I表示无符号整型。

返回：一个元组（即使只有一个元素），本例中就只有一个元素。

### 函数2:得到IP地址对应的地理位置
```
def maxmind_lookup(ip):
  if ip in maxcache:
    return maxcache[ip]
  i = ip2int(ip)
  if i == 0:
    return "UNKNOWN"
  results = list(maxmind.loc[(maxmind["low"] < i) & (maxmind("high") > i)]['asn'])
  if len(results) > 0:
    maxcache[ip] = results[0]
    return results[0]
  maxcache[ip] = "UNKNOWN"
  return "UNKNOWN"
```
maxmind是一家公司，这家公司提供了一个免费的CSV文件——GeoIPASNum2.csv，里面包含了IP地址
对应的地理位置。在这之前，已经用以下代码导入数据。
```
maxmind = pd.read_csv("./GeoIPASNum2.csv", sep=',', header=None, names=['low','high','asn'])
maxmind['low'] = maxmind['low'].astype(int)
maxmind['high'] = maxmind['high'].astype(int)
```
maxcache是之前定义的一个字典，key是ip，value是地理位置的字符串。

函数2的功能可以说是查找一个ip地址，如果它在maxcache中存在了，就直接返回地理位置
如果不存在，则将它加入maxcache，然后返回地理位置，如果找不到位置，就直接返回
"UNKNOWN".

这里的有一行代码
```
results = list(maxmind.loc[(maxmind["low"] < i) & (maxmind("high") > i)]['asn'])
```
results中最多只有一个元素，所以下面直接取results[0]作为物理地址的字符串。

### 函数3：
```
def box_plot_df_setup(series_a, series_b):
    # Count up all the times that a category from series_a
    # matches up with a category from series_b. This is
    # basically a gigantic contingency table
    cont_table = collections.defaultdict(lambda : collections.Counter())
    for val_a, val_b in zip(series_a.values, series_b.values):
        cont_table[val_a][val_b] += 1

    # Create a dataframe
    # A dataframe with keys from series_a as the index, series_b_keys
    # as the columns and the counts as the values.
    dataframe = pd.DataFrame(cont_table.values(), index=cont_table.keys())
    dataframe.fillna(0, inplace=True)
    return dataframe
```
相关知识：pandas，collections，lambda，zip
pandas还有点不清楚，看完pandas再更。

### 函数4：判断是否是ip地址
```
def is_ip(ip):
    try:
        socket.inet_aton(ip)
        return True
    except socket.error:
        return False
```


参考

https://docs.python.org/2/library/struct.html
https://docs.python.org/2/library/socket.html
