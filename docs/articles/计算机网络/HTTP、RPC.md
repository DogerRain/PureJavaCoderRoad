## 1、两台计算机如何进行网络通信？

网络通信需要做的就是将流从一台计算机传输到另外一台计算机，基于传输协议和网络 IO 来实现。

其中**传输协议**比较出名的有 tcp 、 udp 等等。

**网络 IO** ，主要有 bio 、 nio 、 aio 三种方式，所有的分布式应用通讯都基于这个原理而实现，只是为了应用的易用，各种语言通常都会提供一些更为贴近应用易用的**应用层协议**。

Socket（套接字）是基于TCP UDP协议上简化网络编程开发 通过操作系统提供SOCKET API完成

socket里面已经封装好了udp和tcp/ip协议，直接使用就可以了。这个是操作系统的类库，提供很多API，后来移植到了windows、各种嵌入式系统，应用程序利用套接字，可以设置对端的 IP 地址，端口号，并实现数据的发送与接收。 网络上的两个程序通过一个双向的通信连接实现数据的交换，这个连接的一端称为一个 Socket。

![](https://cdn.jsdelivr.net/gh/DogerRain/image@main/img-20210401/image-20210604110816912.png)

细化三次握手：

![](https://cdn.jsdelivr.net/gh/DogerRain/image@main/img-20210401/image-20210604110851885.png)

![](https://cdn.jsdelivr.net/gh/DogerRain/image@main/img-20210401/image-20210604110841217.png)



## 2、应用级协议

常见的应用级协议有：

**FTP（文件传送协议）、Telnet（远程登录协议）、DNS（域名解析协议）、SMTP（邮件传送协议），POP3协议（邮局协议），HTTP协议（Hyper Text Transfer Protocol）**

所以这RPC并不是一个OSI七层模型里面的某个模块。

**rpc是远端过程调用，其调用协议通常包含传输协议和序列化协议。**

**传输协议**包含: 如著名gRPC 使用的 http2 协议，也有如dubbo一类的自定义报文的tcp协议。

**序列化协议**包含: 如基于文本编码的 xml、json，也有二进制编码的 protobuf 、hessian等。



### 2.1、常见的RPC框架（规范）：

注意我这里说RPC是一个规范，它不是协议， RPC（即 Remote Procedure Call，远程过程调用）

hessian、rmi、dubbo都是rpc的一种实现

rpc基于xdr

rmi是rpc的面向对象版实现（Java引入的）

hessian是基于binary-rpc协议

dubbo是 基于Hessian二进制序列化

#### 2.1.1、Hessian

Hessian是一个轻量级的RPC框架。

它基于HTTP协议传输，使用Hessian二进制序列化，对于数据包比较大的情况比较友好。

它采用的是二进制RPC协议，因为采用的是二进制协议，所以它很适合于发送二进制数据。

*Hessian* 是由 *caucho* 提供的一个基于 *binary-RPC* 实现的远程通讯 *library* 

*Hessian* 通过其自定义的串行化机制将请求信息进行序列化，产生二进制流。（接收再反序列化）



#### 2.1.2、dubbo

dubbo也是 基于Hessian二进制序列化

dubbo共支持如下几种通信协议：

```
dubbo://
rmi://
hessian://
http://
webservice://
thrift://
memcached://
redis://
```

但是dubbo作为一个优秀的RPC框架，优点肯定不是体验在协议上，更多的是适用于业务之间的远程调用的机制，如容错、重试、负载均衡等等，封装了很多东西的~

#### 2.1.3、gRPC

Google 的开源软件，基于最新的 HTTP2.0 协议，并支持常见的众多编程语言。

我们知道 HTTP2.0 是基于二进制的 HTTP 协议升级版本，目前各大浏览器都在快马加鞭的加以支持。

这个 RPC 框架是基于 HTTP 协议实现的，底层使用到了 Netty 框架的支持。



## 3、为什么有了HTTP，还要使用RPC呢？

更准确来说这个RPC和HTTP不能比较，这俩不是一个东西。

首先， http 使用的是 tcp 协议，（虽然不限于TCP，但基本可以认为是基于 TCP）。

为什么要用的其中之一的原因就是：

 **HTTP默认的tcp 协议 和我们自定义的 tcp 协议在报文上的区别。**

啥意思？就是Dubbo、Hessian 这种RPC规范其实它的报文是和http的报文有区别。

http自定义的报文开销主要是在建立连接和断开上，报文格式的话，header部分有很多冗余的部分，使用二进制编码也压缩不了很多，虽然后来Http2.0也做了多优化。

像**dubbo**这种RPC框架，它也 支持HTTP传输，但是在RPC的规范上面说，它还有容错、重试、负载均衡等等不同的功能；而Hessian更轻量级，才有二进制传输大包更高效；这是HTTP无法媲美的，只不过现在的浏览器、大部分的应用都习惯了使用HTTP协议，因为这样很简单就能进行通信。

**有人说dubbo不是基于Netty的吗？**

Netty的底层也是基于TCP的~它更多体现的是在**网络IO**上面的优势。

## 4、总结

RPC不过是把网络协议和序列化方法封装了一下，RPC只是对底层协议的封装，节省了开销。

在宏观上来说，RPC屏蔽了很多细节，大家用起来，就像方法调用一样。其次就是 RPC 框架一般都有注册中心，有丰富的监控管理；发布、下线接口、动态扩展、容错、重试等，对调用方来说是无感知、统一化的操作。

