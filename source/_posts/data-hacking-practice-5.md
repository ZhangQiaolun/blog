---
title: data-hacking-practice-5
date: 2016-08-24 10:19:47
tags:
  - python
  - security
---
## DNS Break
### Query types
```
print dnsdf.qtype_name.value_counts()
```
> qtype_name:在dns.log（DNS query/response details)文件中,表示name of the query type(e.g.A,AAAA,CNAME,PTR)

### Query type information
```
for q in dnsdf['qtype_name'].unique().tolist():
  print "Query Type: %s" % q
  print dnsdf[dnsdf['qtype_name'] == q]['query'].value_counts().head(5)
  print ""
```
打印出每一种DNS记录对应的前5个请求对象
> query(in dns.log): domain name subject of the query

### Response Code information
```
dnsdf['rcode_name'].value_counts()
```
> rcode_name(in dns.log):Descriptive name of the response code(e.g.NOERROR,NXDOMAIN)

DNS Return Message | DNS Response Code | Function
-------------------|-------------------|---------
 NOERROR	         | RCODE:0	         | DNS Query completed successfully
 FORMERR	         | RCODE:1	         | DNS Query Format Error
 SERVFAIL	         | RCODE:2	         | Server failed to complete the DNS request
 NXDOMAIN	         | RCODE:3	         |Domain name does not exist.  For help resolving this error, read here.
 NOTIMP	| RCODE:4	| Function not implemented
 REFUSED | RCODE:5 | The server refused to answer for the query
 YXDOMAIN	| RCODE:6	| Name that should not exist, does exist
 XRRSET	| RCODE:7	| RRset that should not exist, does exist
 NOTAUTH	| RCODE:8	| Server not authoritative for the zone
 NOTZONE	| RCODE:9	| Name not in zone

> DGA: Domain Generation Algorithm,下面的代码用于查找那些用DGA查找C2 domain的样本
```
dnsdf[dnsdf['rcode_name'] == 'NXDOMAIN']['sample'].value_counts().head(10)
```

在http记录里我们会期待在“Host” header里看到一个值，这说明了客户端正在连接一个给定的IP地址。但是有一些很有意思的是在HTTP的“Host” header里可以看到的hostname，在DNS query记录里却查询不到。这相当于说这个样本没有被记录到PCAP，或者说它就没有发生。
```
intersect_hostnames = set(pd.Series(list(set(httpdf['host']).intersection(set(dnsdf['query'])))))
interesting = []
tempdf = pd.DataFrame()
for hn in list(set(httpdf['host'])):
# is_ip函数之前分析过了
  if hn not in intersect_hostnames and not is_ip(hn):
  #print hn
  interesting.append(hn)
  tempdf = tempdf.append(httpdf[httpdf['host']] == hn)
```
接下来做了一些处理，排序，找出可以的样本
```
tempdf['count'] = 1
tempdf[['host', 'id.resp_h', 'sample', 'count']].groupby(['sample', 'host', 'id.resp_h']).sum().sort('count', ascending=0)
```
挑选前几个，测试是否的确没在DNS记录里出现
```
print dnsdf[dnsdf['query'] == "dgyqimolcqm.cm"]
print dnsdf[dnsdf.answers.str.contains('dgyqimolcqm.cm')]
print dnsdf[dnsdf['sample'] == "BIN_ZeroAccess_Sirefef_C2A9CCC8C6A6DF1CA1725F9"]['query'].value_counts().head(50)
```

## HTTP Funtime
总体情况
```
print "%s Unique User-Agents in %s samples." % (httpdf['user_agent'].nunique(), httpdf['sample'].nunique())
```
对客户端做了排序，接着又做了一些其他的图表操作。
```
tempdf = pd.DataFrame(columns=['sample','num_ua'])
for sample in list(set(httpdf['sample'])):
    tempdf = tempdf.append({'sample':sample, 'num_ua':httpdf[httpdf['sample'] == sample]['user_agent'].nunique()}, ignore_index=True)
tempdf.sort('num_ua', ascending=0).head()
```
## Conn.log information
### ASN information
首先查看ASN数据库的覆盖率，也就是IP位置对应的地理位置知道多少
```
conndf[conndf['maxmind_asn'] == "UNKNOWN"]['id.resp_h'].value_counts()
```
在上式中，conndf中的maxmind_asn这一个column来历如下。
```
conndf['maxmind_asn'] = conndf['id.resp_h'].map(maxmind_lookup)
```
>id.resp_h(conn.log):responding endpoint's IP address(AKA RESP)

### APT connections
接下来的代码看的就是那些标记为APT的样本，看这些样本和我们已经有的ASN数据库的对应情况,这里做了一个柱状图。
```
ax = box_plot_df_setup(conndf[conndf['threat'] == 'APT']['sample'], conndf[conndf['threat'] == 'APT']['maxmind_asn']).T.plot(kind='bar', stacked=True)
pylab.ylabel('Sample Occurrences')
pylab.xlabel('ASN (Autonomous System Number)')
patches, labels = ax.get_legend_handles_labels()
ax.legend(patches, labels, bbox_to_anchor=(1.05, 1), loc=2, borderaxespad=0., title="Sample Name")
```
下面是box_plot_df_setup函数的定义
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
> collections.defaultdict([default_factory[,...]]):第一个参数要是callable，如果key不存在，则调用callable，将value赋为返回的值

```
>>> s = 'mississippi'
>>> d = defaultdict(int)
>>> for k in s:
...     d[k] += 1
...
>>> d.items()
[('i', 4), ('p', 2), ('s', 4), ('m', 1)]
```
```
>>> s = [('yellow', 1), ('blue', 2), ('yellow', 3), ('blue', 4), ('red', 1)]
>>> d = defaultdict(list)
>>> for k, v in s:
...     d[k].append(v)
...
>>> d.items()
[('blue', [2, 4]), ('red', [1]), ('yellow', [1, 3])]
```


> zip用法
```
alist = ['a1', 'a2', 'a3']
blist = ['b1', 'b2', 'b3']

for a, b in zip(alist, blist):
    print a, b

>>> a1 b1
>>> a2 b2
>>> a3 b3
```

> collections.Counter

```
>>> a = collections.Counter()
>>> a[1] += 1
>>> a
>>> Counter({'1' : 1})
```

> pandas.DataFrame.T  :  Transpose index and columns



参考

https://docs.python.org/2/library/collections.html#collections.defaultdict
http://stackoverflow.com/questions/5900578/how-does-collections-defaultdict-work
