---
title: HTTP发送请求建立TCP连接需要的三次握手
---
####TCP连接
几乎所有的HTTP通信都是由TCP/IP承载的，TCP/IP是全球计算机及网络设备都在使用的一种常用的分组交换网络分层协议集。客户端应用程序可以打开一条TCP/IP连接，连接可能运行在世界任何地方的服务器应用程序。一旦连接建立起来，在客户端的和服务器端的计算机之间的报文就永远不会丢失、受损、或失序。

HTTP协议是在TCP/IP传输层上的应用层协议，TCP为HTTP提供了一条可靠的比特传输管道。

下图是浏览器发出http请求到响应的一个过程：
![](https://upload-images.jianshu.io/upload_images/5541401-e4bb5a8160fc8a77.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
在HTTP请求过程中会出现网络时延，主要有以下几种原因：
* 客户端首先要根据URI确定服务器的IP地址和端口号（DNS）；
* 然后客户端向服务器发送一条TCP连接请求，并等待服务器响应请求和接受应答；
* 最后服务器会返回HTTP响应依然需要时间；

#####现在我们主要看一下客户端在发送TCP连接请求时的时延：握手时延
建立一条新的TCP连接时，甚至是在发送任意数据之前，TCP软件之间会交换一系列的IP分组，对连接的有关参数进行沟通（如下图）。如果连接只用来传送少量数据，这些交换过程就会严重降低HTTP的性能。
![](https://upload-images.jianshu.io/upload_images/5541401-a9804371fc77564a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

HTTP是没有连接的概念的，只有请求和响应数据，这个数据包的传送是在TCP连接（TCP connection）之上的。在一个TCP连接上是可以发送多个HTTP请求的。
![](https://upload-images.jianshu.io/upload_images/5541401-a47fbefa64c5a34a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
TCP连接握手需要经过三个步骤：（三次网络传输）
* 请求新的TCP连接时，客户端要向服务器发送一个小的TCP分组（通常是40~60个字节）。这个分组中设置了一个特殊的SYN标记，说明这是一个连接请求。如上图（a）。
* 如果服务器接受了连接，就会对一些连接参数进行计算，并向客户端回送一个TCP分组，这个分组的SYN和ACK标记都被置位，说明连接请求已被接受。如上图（b）。
* 最后，客户端向服务器回送一条确认信息，通知他连接已经成功建立，如上图（c）。现代的TCP栈都允许客户端在这个确认分组中发送数据。
![](https://upload-images.jianshu.io/upload_images/5541401-c7a76b16dd21ffc6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5541401-7de08d39d5af73b1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
####符号解释：

（1）seq序号:占32位，用来标识从TCP源端向目的端发送的字节流，发起方发送数据时对此进行标记。它的初始序号是随机的，相对序号/确认序号是和TCP会话的初始序号相关联的。该序号被用来跟踪该端发送的数据量。每一个包中都包含序号，在接收端则通过确认序号用来通知发送端数据成功接收。

（2）确认序号：ack序号，占32位，只有ACK标志位为1时（不要将确认序号ack与标志位中的ACK弄混了），确认序号才有效。确认方ack = 发起方seq +1，两端配对。 

（3）位码即TCP标志位，有6种，具体含义如下：

    SYN(synchronous建立连接) 发起一个新连接

    ACK(acknowledgement 表示响应、确认) 确认序号有效

    PSH(push表示有DATA数据传输) 接收方应尽快将这个报文交给应用层

    FIN(finish关闭连接) 释放一个连接

    RST(reset表示连接重置)

    URG(urgent pointer紧急指针字段值有效)


参考：
《HTTP权威指南》
https://blog.csdn.net/weixin_36794678/article/details/81491121
