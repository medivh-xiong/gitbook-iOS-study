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
``` obj-c
  intport = 2000;
  
  IPEndPointServerEP = new IPEndPoint(IPAddress.Any,port);
  
  Socketserver = new Socket(AddressFamily.InterNetwork, SocketType.Stream,ProtocolType.Tcp);
  
  server.Bind(ServerEP);
  
  server.Listen(0);
```
客户端：
``` obj-c
  intport = 2000;
  
  IPAddressserverip = IPAddress.Parse("192.168.1.53");
  
  IPEndPointEP = new IPEndPoint(server,port);
  
  Socketserver = new Socket(AddressFamily.InterNetwork, SocketType.Stream,ProtocolType.Tcp);
  
  server.Bind(EP);
 
```
当服务端收到客户端连接后，需要新建一个Socket来处理远端消息
``` obj-c
  //应该放在服务端：
  Socketclient = server.Accept();
```

##二、各协议的区别

OSI模型把网络通信分成7层，由低向高分别是：物理层，数据链路层，网络层，传输层，会话层，表示层和应用层。

我们常用的HTTP协议是对应应用层，TCP协议对应传输层，IP协议对应网络层，HTTP协议是基于TCP连接。

TCP/IP是传输层协议，主要解决数据如何在网络中传输；而HTTP是应用层协议，主要解决如何包装数据。

  **在传输数据时候可以只是用TCP/IP，但是这样没有应用层，无法识别传输的数据类容，这样是没有意义的，如果想使传输的数据有意义，则必须使用应用层协议，HTTP就是一种，WEB使用它，封装HTTP文本信息，然后使用TCP/IP协议传输到网络上**
  
  Socket实际上就是对TCP/IP协议的封装，本身并不是协议，而是调用一个接口（API），通过Socket，我们才能使用TCP/IP协议；

**HTTP和Socket连接的区别**

1.**TCP连接**

Socket本身就是对TCP的封装，就要先明白TCP连接：

建立一次TCP连接需要进行”三次握手“：

1. 客户端发送syn包到服务器，同时进入SYN_SEND状态，等待服务器确认；
2. 服务器收到syn包，必须确认客户的SYN（发送ACK包确认）；同时发送一个SYN包，即SYN+ACK包，同时进入SYN_RECV状态；
3. 客户端收到服务端的SYN+ACK包，向服务端发送确认包ACK，发送完成后，客户端和服务端都进入ESTABLISHED状态，完成三次握手；

只有进行完三次握手后，才能正式传输数据，理想状态下只要建立起连接，在通信双方主动关闭连接之前，TCP连接将会一直保持下去。断开连接时服务器和客户端均可以主动发起断开TCP连接的请求，断开过程需要经过“四次握手”，在”三次握手“基础上在加一步确认。

2.**HTTP连接**