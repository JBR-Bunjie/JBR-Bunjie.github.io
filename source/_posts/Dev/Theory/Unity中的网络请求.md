---
title: Unity中的网络请求
date: 2023-11-5 23:59:48
tags:
  - Unity
categories:
  - Unity
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

# Unity中的网络请求

## 从网络结构开始：

### 关于网络模型

![image-20231106225117112](..\..\..\images\Dev\image-20231106225117112.png)
其中，TCP/IP模型是实际的使用规则，HTTP协议就运作在应用层里

## 从HTTP协议开始

> **UnityWebRequest**
>
> Provides methods to communicate with web servers.
>
> `UnityWebRequest` handles the flow of HTTP communication with web servers. To download and upload data, use [DownloadHandler](https://docs.unity3d.com/ScriptReference/Networking.DownloadHandler.html) and [UploadHandler](https://docs.unity3d.com/ScriptReference/Networking.UploadHandler.html) respectively.
>
> https://docs.unity3d.com/ScriptReference/Networking.UnityWebRequest.html

### 什么是HTTP

- HTTP 协议构建于 TCP/IP 协议之上，是一个应用层协议，默认端口号是 80
- HTTP 是**无连接无状态**的

### HTTP协议规定的请求报文格式

标准的 HTTP 请求分为三个部分：状态行、请求头、消息主体。类似于下面这样：

```
<method> <request-URL> <version>

<headers>

<entity-body>
```

#### method

HTTP 定义了与服务器交互的不同方法，常用的有 `GET` 和 `POST`，此外其实还有 `PUT`，`DELETE`，`PATCH`等。

#### URL

`URL`全称是统一资源标识符，即 `Uniform Resource Locator`，[统一资源标志符 - 维基百科，自由的百科全书 (wikipedia.org)](https://zh.wikipedia.org/zh-cn/统一资源标志符)

我们可以这样认为：一个`URL`地址会用于唯一标识互联网系统上的某一资源，通过请求这个路径，我们就可以获取到相应的资源。

#### 关于 method 与 url 的组合 - 我们的HTTP请求

##### GET 

`GET` 仅用于信息获取，以下是一个 `GET` 请求示例：

```
192.168.0.104:5000/user_login?acc=BBot&pwd=BBBBB
```

对于`GET`，我们而且应该做到：

- 仅将该操作用于获取信息而非修改源信息，换句话说， 它仅仅是获取资源信息，不会影响资源的状态。
- 对同一 `URL` 的多个请求应该返回同样的结果。

GET 请求报文示例：

```
 GET /books/?sex=man&name=Professional HTTP/1.1
 Host: www.example.com
 User-Agent: Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.7.6)
 Gecko/20050225 Firefox/1.0.1
 Connection: Keep-Alive
```

##### POST

与 `GET` 不同的是，`POST` 是能改变原信息的请求方式：

```
 POST / HTTP/1.1
 Host: www.example.com
 User-Agent: Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.7.6)
 Gecko/20050225 Firefox/1.0.1
 Content-Type: application/x-www-form-urlencoded
 Content-Length: 40
 Connection: Keep-Alive

 sex=man&name=Professional  
```

###### 注意

- `GET` 可提交的数据量受到URL长度的限制，`HTTP` 协议规范没有对 `URL` 长度进行限制。这个限制是特定的浏览器及服务器对它的限制
- 理论上讲，`POST` 是没有大小限制的，`HTTP` 协议规范也没有进行大小限制，出于安全考虑，服务器软件在实现时会做一定限制
- 参考上面的报文示例，可以发现 `GET` 和 `POST` 数据内容是一模一样的，只是位置不同，一个在 `URL` 里，一个在 `HTTP` 包的**包体里**

##### 关于POST携带数据的方式：

> HTTP 协议中规定 POST 提交的数据必须在 body 部分中，但是协议中没有规定数据使用哪种编码方式或者数据格式。实际上，开发者完全可以自己决定消息主体的格式，只要最后发送的 HTTP 请求满足上面的格式就可以——只要数据发送后，服务端能成功解析，我们的数据就是有意义的。

一般服务端语言如 PHP、Python 等，以及它们的 framework，都内置了自动解析常见数据格式的功能。服务端通常是根据**请求头(headers)**中的 `Content-Type` 字段来获知请求中的消息主体是用何种方式编码，再对主体进行解析。所以说到 POST 提交数据方案，包含了 `Content-Type` 和消息主体编码方式两部分。常见的 `Content-Type`: `application/x-www-form-urlencoded`；`multipart/form-data`等

### HTTP协议规定的响应报文格式

当我们的请求被服务器接收后，服务器自然应该作出响应——发送响应报文：

`HTTP` 响应与 `HTTP` 请求相似，也由3个部分构成，分别是：

- 状态行
- 响应头(Response Header)
- 响应正文

状态行由协议版本、数字形式的状态代码、及相应的状态描述，各元素之间以空格分隔。

常见的状态码有如下几种：

- `200 OK` 客户端请求成功
- `301 Moved Permanently` 请求永久重定向
- `302 Moved Temporarily` 请求临时重定向
- `304 Not Modified` 文件未修改，可以直接使用缓存的文件。
- `400 Bad Request` 由于客户端请求有语法错误，不能被服务器所理解。
- `401 Unauthorized` 请求未经授权。这个状态代码必须和WWW-Authenticate报头域一起使用
- `403 Forbidden` 服务器收到请求，但是拒绝提供服务。服务器通常会在响应正文中给出不提供服务的原因
- `404 Not Found` 请求的资源不存在，例如，输入了错误的URL
- `500 Internal Server Error` 服务器发生不可预期的错误，导致无法完成客户端的请求。
- `503 Service Unavailable` 服务器当前不能够处理客户端的请求，在一段时间之后，服务器可能会恢复正常。

下面是一个HTTP响应的例子：

```
HTTP/1.1 200 OK

Server:Apache Tomcat/5.0.12
Date:Mon,6Oct2003 13:23:42 GMT
Content-Length:112

<html>...
```

### 关于HTTPS

HTTPS 即 HTTP over TLS，是一种在加密信道进行 HTTP 内容传输的协议。

具体可参考：https://www.ruanyifeng.com/blog/2014/09/illustration-ssl.html

## HTTP与TCP、UDP

### 关于TCP

- TCP工作在传输层，对HTTP(上层结构)透明
- TCP 提供一种**面向连接的、可靠的**字节流服务
  -  TCP 所谓的“可靠”，是指数据的可靠递送或故障的可靠通知。如果确认数据无法递送，则放弃重传并中断连接并通知用户
  - **Reliable:** Applications don't have to worry about missing packets. If a packet gets lost, TCP will resend it. All data is either transmitted successfully or you get an error and the connection is closed.
  - **Sequenced:** TCP guarantees that every message will arrive in the same order it was sent. If you send "a" then "b" you will receive "a" then "b" on the other side as well.
  - **Connection oriented:** TCP has the concept of a connection. A connection will stay open until either the client or server decides to close it. Both the client and server get notified when the connection ends.
  - **Congestion control:** If a server is being overwhelmed, TCP will throttle the data to avoid congestion collapse.

三次握手：

![image-20231106234220228](..\..\..\images\Dev\image-20231106234220228.png)

四次挥手：

![image-20231106234251959](..\..\..\images\Dev\image-20231106234251959.png)

握手过程可见：https://hit-alibaba.github.io/interview/basic/network/TCP.html

### 关于UDP

与TCP 不同，UDP 是一个简单的传输层协议。和 TCP 相比，UDP 有下面几个显著特性：

- UDP 缺乏可靠性。UDP 本身不提供确认，序列号，超时重传等机制。UDP 数据报可能在网络中被复制，被重新排序。即 UDP 不保证数据报会到达其最终目的地，也不保证各个数据报的先后顺序，也不保证每个数据报只到达一次
- UDP 数据报是有长度的。每个 UDP 数据报都有长度，如果一个数据报正确地到达目的地，那么该数据报的长度将随数据一起传递给接收方。而 TCP 是一个字节流协议，没有任何（协议上的）记录边界。
- UDP 是无连接的。UDP 客户和服务器之前不必存在长期的关系。UDP 发送数据报之前也不需要经过握手创建连接的过程。
- **Low Latency:** UDP is faster because it doesn't need to wait for the remote side to acknowledge packets. Instead, it can send keep sending new data packets one after the other. It is also known as a "scattershot" protocol, since it'll just shoot data at clients with no assurances that they'll actually get the data.
- **Channel support:** Channels allow for different delivery types. Depending on the implementation, one channel can be used for critical data that needs to get to the destination, while a different channel can be used for "send and forget" data transfer without any reliability.
- **Different packet types:** Depending on the implementation on top of the UDP protocol, some transports offer different packet sending methods, such as Reliable Ordered, Reliable Unordered, Unreliable, and more depending on the implementation. Reliable UDP transfer depends on the implementation, but it is usually modelled after TCP's reliability system.

### 短链接，长连接？

#### 短链接

对于HTTP，我们知道 HTTP 协议采用的是「请求-应答」的模式，也就是客户端发起了请求，服务端才会返回响应，一来一回这样子。

![图片](https://s7.51cto.com/oss/202212/02/a75027a69676bf89312000b92804dee24c2462.png)

由于 HTTP 是基于 TCP 传输协议实现的，客户端与服务端要进行 HTTP 通信前，需要先建立 TCP 连接，然后客户端发送 HTTP  请求，服务端收到后就返回响应，至此「请求-应答」的模式就完成了，随后就会释放 TCP 连接。

![图片](https://s6.51cto.com/oss/202212/02/695157c0035704109a25910dc657e6d8da8c1c.png)

如果每次请求都要经历这样的过程：建立 TCP -> 请求资源 -> 响应资源 -> 释放连接，那么此方式就是 HTTP 短连接

很显然，这样子一次连接只能请求一次资源的传输十分低效——这就是为什么我们需要长连接

#### 长连接

所谓长连接，实际上指的是 HTTP 与 TCP 中的 “保活(Keep-Alive)” 机制，不过，他们虽然是共同实现一个目的，但 HTTP 的 Keep-Alive 和 TCP 的 Keep-Alive 其实完全是两样不同东西，实现的层面也不同：

- HTTP 的 Keep-Alive，是由应用层（用户态）实现的，称为 HTTP 长连接；
- TCP 的 Keepalive，是由TCP 层（内核态）实现的，称为 TCP 保活机制；

其核心思路就是：能不能在第一个 HTTP 请求完后，先不断开 TCP 连接，让后续的 HTTP 请求继续使用此连接？

而 HTTP 的 Keep-Alive 就是实现了这个功能，可以使用同一个 TCP 连接来发送和接收多个 HTTP 请求/应答，避免了连接建立和释放的开销，这个方法称为 HTTP 长连接。

![图片](https://s8.51cto.com/oss/202212/02/07ca4851652284415bd56143a6026c65fce801.png)

这样做会产生的结果是：只要任意一端没有明确提出断开连接，则保持 TCP 连接状态。

从 HTTP 1.1 开始， 就默认是开启了 Keep-Alive，如果要关闭 Keep-Alive，需要在 HTTP 请求的包头里添加：

```
Connection:close1.
```

现在大多数浏览器都默认是使用 HTTP/1.1，所以 Keep-Alive 都是默认打开的。一旦客户端和服务端达成协议，那么长连接就建立好了。

那么，既然我们有了保持 TCP 的手段，自然也需要关闭：在使用了 HTTP 长连接的情况下，如果客户端完成一个 HTTP 请求后，就不再发起新的请求，此时这个 TCP 连接一直占用着不是挺浪费资源的吗？

为了应对这种资源浪费的情况，web 服务软件一般都会提供 keepalive_timeout 参数，用来指定 HTTP 长连接的超时时间。

比如设置了 HTTP 长连接的超时时间是 60 秒，web 服务软件就会启动一个定时器，如果客户端在完后一个 HTTP 请求后，在 60 秒内都没有再发起新的请求，定时器的时间一到，就会触发回调函数来释放该连接。

![图片](https://s3.51cto.com/oss/202212/02/71963cb74e7a71af0532326a92a86e517a73e4.png)

### HTTP与UDP

> [(31 封私信 / 82 条消息) HTTP 是基于 TCP 还是 UDP 的？ - 知乎 (zhihu.com)](https://www.zhihu.com/question/20085992)

## 那么KCP呢

>KCP 是一个快速可靠协议，能以比 TCP 浪费 10%-20% 的带宽的代价，换取平均延迟降低 30%-40%，且最大延迟降低三倍的传输效果。纯算法实现，并不负责底层协议（如UDP）的收发，需要使用者自己定义下层数据包的发送方式，以 callback的方式提供给 KCP。连时钟都需要外部传递进来，内部不会有任何一次系统调用。
>
>整个协议只有 ikcp.h, ikcp.c两个源文件，可以方便的集成到用户自己的协议栈中。也许你实现了一个P2P，或者某个基于 UDP的协议，而缺乏一套完善的ARQ可靠协议实现，那么简单的拷贝这两个文件到现有项目中，稍微编写两行代码，即可使用。
>
>https://github.com/skywind3000/kcp

### 网络层级与ARQ协议

协议的可靠需要很多额外的步骤去实现，包括：

- 检验和
- 确认应答
- 超时重传
- 拥塞控制等

TCP能做到可靠，是因为它在请求头中预留了控制字段的位置，自行实现了这些机制，如果我们为了更快的速度选择UDP的话，就不会天然存在这些工具了——我们需要外部的控制手段来实现这些内容——而KCP，正是这样的一个 `ARQ协议`

而 **ARQ(Automatic Repeat-reQuest)**，自动重传请求，正是 OSI模型 中的错误纠正协议之一。

- 通过使用确认和重传这两个机制，在不可靠服务的基础上实现可靠的信息传输。
- 如果发送方在发送后一段时间之内没有收到确认帧，它通常会重新发送。
- 重传的请求是自动进行的，接收方不需要请求发送方重传某个出错的分组
- ARQ包括停止 等待ARQ协议 和 连续ARQ协议

> [在网络中狂奔：KCP协议 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/112442341)

## 游戏中的协议

> **TCP** is the most popular protocols on the internet. TCP core features make it easy for programers to work with the protocol, since a programmer will be able to assure data will be sent from A to B correctly.
>
> However, while this sounds great, it can introduce higher latency when connections are malfunctioning or experiencing severe turbulence.
>
> In a game environment, TCP is better for slower paced games where latency isn't important but the game data is.
>
> **UDP** is used for real time applications where low latency is more important than reliability. 
>
> In a game environment, the raw power of UDP can be harnessed to allow for a greater control of how data is being sent, allowing non-critical data to be sent faster. This in turn makes UDP better for fast paced games where latency between server and client is important and if a few packets are lost, the game can recover.
>
> [TCP and UDP - Mirror (gitbook.io)](https://mirror-networking.gitbook.io/docs/manual/general/tcp-and-udp#transports-1)

> **Transports using the TCP protocol**
>
> - Telepathy
> - WebSockets
>
> **Transports using the UDP Protocol**
>
> - Ignorance
> - KCP
> - LiteNetLib
>
> [TCP and UDP - Mirror (gitbook.io)](https://mirror-networking.gitbook.io/docs/manual/general/tcp-and-udp#transports-1)

## 关于Mirror

Mirror可以参考的设计有很多

- [NetworkBehaviour Callbacks - Mirror (gitbook.io)](https://mirror-networking.gitbook.io/docs/manual/guides/communications/networkbehaviour-callbacks)
- [Timestamp Batching - Mirror (gitbook.io)](https://mirror-networking.gitbook.io/docs/manual/general/timestamp-batching)

## 进行Socket编程

> Socket 是对 TCP/IP 协议族的一种封装，是应用层与TCP/IP协议族通信的中间软件抽象层。从设计模式的角度看来，Socket其实就是一个门面模式，它把复杂的TCP/IP协议族隐藏在Socket接口后面，对用户来说，一组简单的接口就是全部，让Socket去组织数据，以符合指定的协议。
>
> [Socket 编程 · 笔试面试知识整理 (hit-alibaba.github.io)](https://hit-alibaba.github.io/interview/basic/network/Socket-Programming-Basic.html)

- [网络游戏实战:01-网络请求 - CodeFish'Blogs](https://blog.codefish.net/2022/10/17/net-game-netgame-action-01-networkrequest/)

## References

- [HTTP 协议 · 笔试面试知识整理 (hit-alibaba.github.io)](https://hit-alibaba.github.io/interview/basic/network/HTTP.html)
- https://www.ruanyifeng.com/blog/2014/09/illustration-ssl.html
- https://www.51cto.com/article/741263.html
- [在网络中狂奔：KCP协议 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/112442341)
- [Mirror Networking - Mirror (gitbook.io)](https://mirror-networking.gitbook.io/docs/)
