---
title: data-hacking-practice-4
date: 2016-08-20 19:17:36
tags:
  - python
  - security
---
## PCAP文件的深入理解

## 得到大体的数据
```
print "Total Samples:          %s" % conndf['sample'].nunique()
print ""
print "APT Samples:            %s" % conndf[conndf['threat'] == 'APT']['sample'].nunique()
print "Crime Samples:          %s" % conndf[conndf['threat'] == 'CRIME']['sample'].nunique()
print "Metasploit Samples:     %s" % conndf[conndf['threat'] == 'METASPLOIT']['sample'].nunique()
print ""
print "Connection Log Entries: %s" % conndf.shape[0]
print "DNS Log Entries:        %s" % dnsdf.shape[0]
print "HTTP Log Entries:       %s" % httpdf.shape[0]
print "Files Log Entries:      %s" % filesdf.shape[0]
print "SMTP Log Entries:       %s" % smtpdf.shape[0]
print "Weird Log Entries:      %s" % weirddf.shape[0]
print "SSL Log Entries:        %s" % ssldf.shape[0]
print "Notice Log Entries:     %s" % noticedf.shape[0]
print "Tunnel Log Entries:     %s" % tunneldf.shape[0]
print "Signature Log Entries:  %s" % sigdf.shape[0]
```
### pandas.Series.unique和pandas.Series.nunique
- Series.unique()
返回一个对象中unique value组成的向量，包括NA值.
- Series.nunique(dropna=True)
返回对象中unique elements的数量，默认*不*包括NA值.参数dropna（drop NA）默认为true

### 得到行数（get the number of rows of dataframe df with Pandas）
要得到行数，可以有以下方法
- df.count(),但是只会返回每一列不是NA/NaN类型的行数。
- df.shape[0]
- len(df.index)
```
# 以下是性能的测试
In [1]: import numpy as np

In [2]: import pandas as pd

In [3]: df =pd.DataFrame(np.arange(9).reshape(3,3))

In [4]: df
Out[4]:
   0  1  2
0  0  1  2
1  3  4  5
2  6  7  8

In [5]: df.shape
Out[5]: (3, 3)

In [6]: timeit df.shape
1000000 loops, best of 3: 1.17 us per loop

In [7]: timeit df[0].count()
10000 loops, best of 3: 56 us per loop

In [8]: len(df.index)
Out[8]: 3

In [9]: timeit len(df.index)
1000000 loops, best of 3: 381 ns per loop
```
###　得到列数
- len(df.columns)
```
import pandas as pd
df = pd.DataFrame({"pear": [1,2,3], "apple": [2,3,4], "orange": [3,4,5]})

len(df.columns)
3
```
- df.shape[1]

## Security Workflow Breakdown
### 代码1：从signature hints里得到目标地址
```
# Get all the destination addresses from all the signature hits, in this case it's only one.
sig_dst_ips = sigdf['dst_addr'].tolist()
sigdf[['dst_addr', 'dst_port','sig_id','sub_msg','threat','sample']]
```
#### pandas.Series.tolist
```
转化为一个nested的数组
from pandas import *

d = {'one' : Series([1., 2., 3.], index=['a', 'b', 'c']),
    'two' : Series([1., 2., 3., 4.], index=['a', 'b', 'c', 'd'])}

df = DataFrame(d)

#print df

print "DF", type(df['one']), "\n", df['one']

dfList = df['one'].tolist()

print "DF list", dfList, type(dfList)
```

### 代码2

```
for ip in sig_dst_ips:
    print "**** IP: %s ****" %ip
    print "  ** Flow Information **"
    print conndf[conndf['id.resp_h'] == ip][['id.resp_p','proto','service','duration','conn_state','orig_ip_bytes','resp_ip_bytes']]
    print "  ** HTTP Information **"
    print httpdf[httpdf['id.resp_h'] == ip][['method','host','uri','user_agent']]
    files = httpdf[httpdf['id.resp_h'] == ip]['orig_fuids']
    flist = files.append(httpdf[httpdf['id.resp_h'] == ip]['resp_fuids']).tolist()
    # We use SHA1 because that's what gets tossed in the Bro notice.log for the Team Cymru MHR alerts
    print "  ** File SHA1 **"
    for f in flist:
        if f != '-':
            sha1 = filesdf[filesdf['fuid'] == f]['sha1'].tolist()
            for m in sha1:
                print "Sample Hash: %s" % m
                if noticedf[noticedf['sub'].str.contains(m)][['sub','sample']].shape[0] > 0:
                    print noticedf[noticedf['sub'].str.contains(m)][['sub','sample']]
                print "Filename: %s    mime-type: %s" % (filesdf[filesdf['sha1'] == m]['filename'].tolist()[0], filesdf[filesdf['sha1'] == m]['mime_type'].tolist()[0])
                print ""
```

这里的代码就是打印sha1值比较有意思

###  pandas.Series.str.contains

```
df = pd.DataFrame({'vals': [1, 2, 3, 4], 'ids': [u'aball', u'bball', u'cnut', u'fball']})

df[df['ids'].str.contains("ball")]
```
```
In [3]: df[df['ids'].str.contains("ball")]
Out[3]:
     ids  vals
0  aball     1
1  bball     2
3  fball     4
```


参考
http://pandas.pydata.org/pandas-docs/stable/index.html
http://stackoverflow.com/questions/27975069/how-to-filter-rows-containing-a-string-pattern-from-a-pandas-dataframe
http://stackoverflow.com/questions/15943769/how-to-get-row-count-of-pandas-dataframe#comment33352203_15943975
http://stackoverflow.com/questions/22341271/get-list-from-pandas-dataframe-column
