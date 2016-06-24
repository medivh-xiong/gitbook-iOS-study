# Socket学习

##一、概念理解

1.**什么是Socket？**

Socket又称为“套接字”，是系统提供的用于网络通信的方法，本质并不是一个协议，没有规定计算机怎么样传递消息，只是给程序员提供一个接口，使用这个接口提供的方法，发送和接收消息。

Socket简化了程序员操作，知道对方的IP和端口号的情况下，就可以给对方发送消息，再有服务端来处理，因此需要服务端和客户端。

2.**Socket的通信过程**

每一个应用或者服务都有一个端口，因此需要包含以下的步骤：

* 服务端利用Socket监听端口；

* 客户端发起连接；

* 服务端返回信息，建立连接，开始通信；

* 客户端，服务端断开连接；

3.**Socket双方如何建立连接**

服务端：
``` c
  intport = 2000;
  IPEndPointServerEP = new IPEndPoint(IPAddress.Any,port);
  Socketserver = new Socket(AddressFamily.InterNetwork, SocketType.Stream,ProtocolType.Tcp);
```


