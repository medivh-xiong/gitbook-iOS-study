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
  int port = 2000;
  
  IPEndPoint ServerEP = new IPEndPoint(IPAddress.Any,port);
  
  Socket server = new Socket(AddressFamily.InterNetwork, SocketType.Stream,ProtocolType.Tcp);
  
  server.Bind(ServerEP);
  
  server.Listen(0);
```
客户端：
``` obj-c
  int port = 2000;
  
  IPAddress serverip = IPAddress.Parse("192.168.1.53");
  
  IPEndPoint EP = new IPEndPoint(server,port);
  
  Socket server = new Socket(AddressFamily.InterNetwork, SocketType.Stream,ProtocolType.Tcp);
  
  server.Bind(EP);
 
```
当服务端收到客户端连接后，需要新建一个Socket来处理远端消息
``` obj-c
  //应该放在服务端：
  Socket client = server.Accept();
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

![](tcp-三次握手.png)

首先了解一下几个标志，SYN（synchronous），同步标志，ACK (Acknowledgement），即确认标志，seq应该是Sequence Number，序列号的意思，另外还有四次握手的fin，应该是final，表示结束标志。

简单的用英语来表示就是：
客户端：hi,how are you？
服务端：fine，thank you,and you？
客户端：i am fine too.
然后开始通信。

只有进行完三次握手后，才能正式传输数据，理想状态下只要建立起连接，在通信双方主动关闭连接之前，TCP连接将会一直保持下去。断开连接时服务器和客户端均可以主动发起断开TCP连接的请求，断开过程需要经过“四次握手”，在”三次握手“基础上在加一步确认。

2.**HTTP连接**

HTPP协议即超文本传送协议，是建立在TCP协议之上的一种。客户端每次发送的请求都要服务端回送响应，请求结束后，会自动释放连接。

3.**Socekt连接**

概念：Socket是通信的基石，是支持TCP/IP协议的基本操作单元，包含5种信息：连接使用的协议，本机主机IP地址，本地进程的端口号，远程主机IP地址，远程进程的协议端口。

应用层通过传输层进行数据传输时候，可能会遇到同一个TCP协议端口传输好几种数据，可以通过socket来区分不同应用程序或者网络连接。

建立Socket连接的步骤
1. 至少需要1对，一个作用于客户端，一个在服务端；
2. 连接分为三个步骤：服务器监听，客户端请求，连接确认；
3. 服务器监听：并不对应具体的客户端socket，而是处于等待连接状态，实时监听网络状态，等待客户端连接；
4. 客户端请求：客户端的套接字向服务端套接字发起连接请求，因此需要知道服务端的套接字的地址和端口号，而且需要描述他要连接的服务器的套接字；
5. 连接确认：当服务端套接字监听到或者接收到客户端的套接字的连接请求，就响应客户端的套接字，建立一个新的连接，把服务端的套接字的描述发给客户端，一旦确认，双方就正式建立连接。而且服务端的套接字仍在监听状态，继续接受其他客户端的套接字。

**Socket HTTP TCP区别**

Socket连接可以指定传输层协议，可以是TCP或者UDP，当时TCP协议时候就是一个TCP连接。而HTTP连接是请求->响应的方式，在请求时候需要先建立连接，然后客户端向服务器发出请求之后，服务器才能回复数据。而Socket一旦建立连接，服务器可以主动将数据传输给客户端；而HTTP则需要客户端先向服务器发送请求之后才能将数据返回给客户端。但实际上Socket建立之后因为种种原因，会导致断开连接，其中一个原因就是防火墙会断开长时间处于非活跃状态的连接，因此需要轮询高速网络，这个连接是活跃的。


## 三、在iOS里面的使用

iOS提供了Socket网络编程接口CFSocket，tcp和udp的socket是有区别的。

**基于TCP的Socket：**
> ![](tcp_Socket.png)


**基于UDP的Socket**

> ![](udp_Socket.png)








```obj-c
  

```

