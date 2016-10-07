---
title: python-UDP-programming
date: 2016-08-14 15:05:03
tags:
  - network
  - python
---
# UDP编程
## 以下是用python3实现的一个UDP服务器和客户端
### client
```
from socket import *

serverName = 'localhost'    # 在本地测试
serverPort = 12000
clientSocket = socket(AF_INET, SOCK_DGRAM)
message = input('Input lowercase sentence:')
clientSocket.sendto(message.encode('utf-8'), (serverName, serverPort))    # 见问题2
modifiedMessage, serverAddress = clientSocket.recvfrom(2048)
print(modifiedMessage.decode('utf-8'))
clientSocket.close()
```
### server
```
from socket import *

serverPort = 12000
serverSocket = socket(AF_INET, SOCK_DGRAM)
serverSocket.bind(('', serverPort))
print("The server is ready to receive")

while 1:
    message, clientAddress = serverSocket.recvfrom(2048)
    modifiedMessage = message.upper()
    serverSocket.sendto(modifiedMessage, clientAddress)
```



socket
### 遇到的问题总结
- 从python2转到python3，发现raw_input已经在python3中取消了，转为用input代替。

在python2中，有input和raw_input函数
在python3中，raw_input被重命名为input，如果想要使用原来input的功能，需要使用eval(input())
- sendto在python2和python3中有不同

```
clientSocket.sendto(message.encode('utf-8'), (serverName, serverPort))
```
为什么上面的message需要编码？
如果写成
```
clientSocket.sendto(message, (serverName, serverPort))
```
会出现以下报错
```
TypeError: a bytes-like object is required, not 'str'
```
在python2里这是对的，但是在python3里改为接受byte类型，但是为什么编码为'utf-8'就可以了，还不清楚。
- python socket中AF_INET等的解释

首先说明一下：
```
AF_* => Address Family
PF_* => Protocol Family
```
另外在linux内核中：/include/linux/socket.h中
```
......... some uninteresting details
#define PF_LOCAL        AF_LOCAL
#define PF_INET            AF_INET
#define PF_AX25           AF_AX25
..................
```
原来AF_*和PF_*是相区分的，但现在BSD的man page上说：The protocol family generally is the same as the address family. 所以现在标准上到处都是AF_*
总的来说：对于TCP和UDP，用AF_INET就好。


参考：
http://stackoverflow.com/questions/13274553/python-3-3-socket-typeerror
https://www.quora.com/Whats-the-difference-between-the-AF_PACKET-and-AF_INET-in-python-socket
http://www.tutorialspoint.com/python3/python_networking.htm
https://docs.python.org/3/library/socket.html#socket.AF_INET
Computer Networking: A Top Down Approach
