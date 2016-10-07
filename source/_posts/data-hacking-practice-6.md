---
title: data-hacking-practice-6
date: 2016-08-25 07:03:44
tags:
  - python
  - security
---

*加把油，这应该是最后一篇这个项目了*

## Conn.log information
### ASN information
```
conndf[conndf['maxmind_asn'] == 'UNKNOWN']['id.resp_h'].value_counts()
```
首先检查ASN数据库，发现这个数据库有问题，发现数据库对不上，相差非常大。
![](data-hacking-practice-6/conndf_maxmind_1.png)
*原作者的数据*
![](data-hacking-practice-6/conndf_maxmind_2.png)
*我的数据*

然后先接着做做看
```
ax = box_plot_df_setup(conndf[conndf['threat'] == 'APT']['sample'], conndf[conndf['threat'] == 'APT']['maxmind_asn']).T.plot(kind='bar', stacked=True)
pylab.ylabel('Sample Occurrences')
pylab.xlabel('ASN (Autonomous System Number)')
patches, labels = ax.get_legend_handles_labels()
ax.legend(patches, labels, bbox_to_anchor=(1.05, 1), loc=2, borderaxespad=0., title="Sample Name")
```
下面的图当然不对
![](data-hacking-practice-6/bar1.png)
*原作者的图像*
![](data-hacking-practice-6/bar2.png)
*我的图像*
在这里可以看出，相差其实只有一个样本，这个样本对应的区域现在ASN没有收录，就是说我的数据库和作者使用的有部分的出入。

这个地点的ASN(Autonomous System Number)为AS21788 Network Operations Center Lnc.（但仍有点奇怪，这个像是个很有名的地方）

找出一个可疑的样本
```
conndf[conndf['sample'] == "BIN_8202_6d2c12085f0018daeb9c1a53e53fd4d1"][['maxmind_asn','id.resp_h']]
```

### Top-N(it's everybody's favorite)
找出传输了最多数据的样本，然后查看被这个样本所用的端口。
```
conndf['count']
grouped = conndf.groupby(['sample', 'id.resp_p']).sub()
grouped.sort('total_bytes', ascending = 0).head(10)
```
返现1935端口很可疑，进一步查看
```
conndf[conndf['id.resp_p'] == 1935]['id.resp_h', 'proto']
```
以及336端口
```
conndf[conndf['id.resp_p'] == 336][['id.resp_h', 'proto']]
```

## SMTP, because who doesn't love some SPAM
首先看看大致的情况
```
这个在我的版本上没用
smtpdf.sample.value_counts()
```
用另一种
```
smtpdf['sample'].value_counts()
```
### Host aka Open Relays
```
print "Unique Hosts found as the HELO portion of SMTP traffic: %s" % smtpdf.helo.unique()
print ""
print "Some of the exapmples"
print smtpdf.helo.value_counts().head(10)
```
###  Patterns in email
从发送者和接收方寻找一些模式
```
smtpdf['count'] = 1
grouped = smtpdf[smtpdf['from'] != "-"][['from','subject','count']].groupby(['from', 'subject']).sum()
grouped.sort('count', ascending = 0).head(20)
```

## Files
### Summary information
查看那种文件格式最多，以及与它相关的服务类型
```
ax = box_plot_df_setup(filesdf['source'], filesdf['mime_type']).T.plot(kind='bar', stacked=True)
pylab.xlabel('Mime-Type')
pylab.ylabel('Number of Files')
patches, labels = ax.get_legend_handles_labels()
ax.legend(patches, labels, title="Service Type")
```
一些更具体的分析工作
```
filesdf['count'] = 1
filesdf[filesdf['filename'] != '-'][['source', 'mime_type', 'seen_bytes', 'count']].groupby(['source', 'mime_type']).sum().sort('count', ascending=0).head(10)
```
```
filesdf[filesdf['filename'] != '-'][['sample','mime_type','filename','count']].groupby(['sample','mime_type','filename']).sum().sort('count', ascending=0).head(10)
```

## Notice
### What things should we pay attention to
查看note，msg相关的的表格
```
noticedf['count'] = 1
noticedf[['note','msg','count']].groupby(['note','msg']).sum().sort('count', ascending=0)
```
```
noticedf['count'] = 1
noticedf[['note', 'msg', 'count']].groupby(['note','msg']).sum().sort('count', ascending=0)
```
## SSL
### Stats
在前面看到了很多SSL的notice，接下来就看看SSL

```
ssldf['id.resp_p'].value_counts()
```
接下来的作图倒很有意思,字典里一层一层地下去，然后一层一层地yongname和children
```
data = {'name' : 'ssl'}
samples = list(set(ssldf['sample'].tolist()))
data['children'] = list()
sampleindex = 0
for sample in samples:
    data['children'].append({'name' : sample, 'children' : list()})
    ports = set(ssldf[ssldf['sample'] == sample]['id.resp_p'].tolist())
    portindex = 0
    for port in ports:
        data['children'][sampleindex]['children'].append({'name' : str(port), 'children' : list()})
        hostnames = set(list(ssldf.loc[(ssldf['id.resp_p'] == int(port)) & (ssldf['sample'] == sample)]['server_name']))
        for hostname in hostnames:
            data['children'][sampleindex]['children'][portindex]['children'].append({'name' : hostname, 'size' : 1})
        portindex += 1
    sampleindex += 1
json.dump(data, open('ssl.json', 'w'))

```

参考

http://nbviewer.jupyter.org/github/ClickSecurity/data_hacking/blob/master/contagio_traffic_analysis/contagio_traffic_analysis.ipynb
