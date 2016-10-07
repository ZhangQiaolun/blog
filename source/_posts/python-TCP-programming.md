---
title: python TCP programming
date: 2016-08-17 09:47:28
tags:
  - python
  - network
---
# python TCP programming

## TCP大致结构


## 以下是用python3实现的一个TCP服务器和客户端(really simple)
### client
```
from socket import *
serverName = "localhost"
serverPort = "12000"
clientSocket = socket(AF_INET, SOCK_STREAM) # IPV4, TCP socket
# do not need to specify the port number of the client
clientSocket.connect(serverName, serverPort)
sentence = input("Input lover case sentence")
clientSocket.send(sentence.encode("utf-8"))
modifiedMessage = clientSocket.recv(1024)
print("From server:", modifiedMessage.decode('utf-8'))
```
### server
```
from socket import *
serverPort = "12000"
serverSocket = socket(AF_INET, SOCK_STREAM)
serverSocket.bind(('', serverPort)) # need to specify the port number of the client

# 这一行让服务器等待TCP的连接请求，参数1声明了等待的连接数的最大值（最小是1）
serverSocket.listen(1)

print("The server is ready to receive.")

while 1:
  connectionSocket, addr = serverSocket.accept()
  sentence = connectionSocket.recv(1024)
  capitalizedSentence = sentence.upper()
  connectionSocket.send(capitalizedSentence)
  connectionSocket.close()
```

另见[python UDP 编程](http://love.myyoulemei.com/2016/08/14/python-UDP-programming/)

参考

Computer Networking: A Top Down Approach
