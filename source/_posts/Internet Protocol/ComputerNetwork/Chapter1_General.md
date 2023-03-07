# 1. Beginning

## Questions

>1. **什么是计算机网络？它是怎样发展起来的？**
>2. **计算机网络有那些功能？**=> 具体见
>3. **网络上不同的计算机系统如何进行数据交换的？**=> 具体见
>4. **什么是网络协议?**=> 具体见
>5. **如何进行差错控制，流量控制?** => 具体见
>6. **如何进行路由选择?**=> 具体见
>7. **如何保证网络安全？**=> 具体见

## Answers

> 主要回答了问题1，其他内容略有涉及，但主要集中在接下来的其他章节

> **计算机网络是计算机技术与通信技术的产物**

### 定义

笼统定义: 将若干台具有**独立**功能的计算机系统，用某种或多种通信介质连接起来，通过完善的网络协议，在**数据交换**的基础上，实现网络**资源共享**的系统称为计算机网络。 

> - 一个网络中包含了多个独立的计算机系统。 “独立”的含义是指每台计算机都可运行各自独立的操作系统，**各计算机系统之间的地位平等，无主从之分**，任何一台计算机不能干预或强行控制其他计算机的正常运行。
> - **数据交换是网络的最基本功能**，各种资源共享都是建立在数据交换的基础上的。
> - **资源共享是网络最终目的**。

### 发展阶段

**初期；五十年代中期－六十年代中期；远程联机系统**：世界上最早的计算机网络ARPANET（Internet的前身），由美国国防部高级计划研究署研制。ARPANET于1969年开通。最初仅连接美国本土的四个主机系统（加州大学洛杉矶分校，加州大学伯克利分校，斯坦福研究所，犹他大学），随后网络规模不断扩大，连接的主机数目越来越多，并由最初的纯军事网络演变成为面向教育，科研，商业的全球性网络。

![image-20230302193235598](C:\Users\m1518\OneDrive\DOCUMENTS\MARKDOWN DOCUMENTS\Internet Protocol\ComputerNetwork\ImageResources\1_001)

![image-20230302195015269](C:\Users\m1518\OneDrive\DOCUMENTS\MARKDOWN DOCUMENTS\Internet Protocol\ComputerNetwork\ImageResources\1_002)

**中期；六十年代末 - 七十年代末；计算机—计算机网络：**

![image-20230302195429892](C:\Users\m1518\OneDrive\DOCUMENTS\MARKDOWN DOCUMENTS\Internet Protocol\ComputerNetwork\ImageResources\1_003)

![image-20230302195820874](C:\Users\m1518\OneDrive\DOCUMENTS\MARKDOWN DOCUMENTS\Internet Protocol\ComputerNetwork\ImageResources\1_004)

>**计算机—计算机网络**与**远程联机系统**的本质区别：
>
>计—计网络以资源共享为目标，在网络协议的支持下，用户使用远方计算机系统的资源就好像使用本地计算机系统一样方便。几乎觉察不到地理位置的差别。

**后期；八十年代 - 至今；开放式标准化网络**

具有统一的网络体系结构，遵守国际标准化协议，便于网络互连，大规模生产，降低成本 。如：

- OSI参考模型  
- CCITT建议
- TCP/IP协议族

# 2. Homework

## 什么是计算机网络的拓扑结构？按照拓扑结构，计算机网络可以分为哪几种？各有什么特点？

> 还可以按地理范围分类：
>
> - 局域网LAN
> - 城域网MAN
> - 广域网WAN

- 计算机网络的拓扑结构是指：计算机网络拓扑是指通信子网节点间连接结构的拓扑形式，通过结点与线段间的几何关系表示网络结构，反映网络中各实体的结构关系。

- 按照拓扑结构，计算机网络可以分为：

  -  星型网
    - 转输介质从一个中央结点向外辐射连接其他节点。
    - 任何两个结点之间的信息交换必须经过中央结点转发。
    - 中央结点的可靠性十分重要，一旦中央结点发生故障，会引起整个网络瘫痪

  ![image-20230302201845313](C:\Users\m1518\OneDrive\DOCUMENTS\MARKDOWN DOCUMENTS\Internet Protocol\ComputerNetwork\ImageResources\1_005)

  - 环形网
    - 网络上所有的结点通过传输介质连接成一个闭环，任何两个结点的数据交换必须沿环进行。
    - 一旦结点或链路发生故障，则环路断开，导致网络瘫痪

  ![image-20230302201909929](C:\Users\m1518\OneDrive\DOCUMENTS\MARKDOWN DOCUMENTS\Internet Protocol\ComputerNetwork\ImageResources\1_006)

  - 总线网络
    - 一条总线连接所有的结点，任何一个结点发送数据，其他节点都能收到。
    - 共享信道。
    - 任何结点故障都不会影响整个网络正常运行。

  ![image-20230302201931552](C:\Users\m1518\OneDrive\DOCUMENTS\MARKDOWN DOCUMENTS\Internet Protocol\ComputerNetwork\ImageResources\1_007)

  - 不规则型网
    - 每个结点至少要和其他两个结点连接。
    - 可靠性好：任何一个结点或一条链路发生故障都不会影响网络的连通性。
    - 布线灵活，几乎不受任何拓扑结构的约束。

![image-20230302201943498](C:\Users\m1518\OneDrive\DOCUMENTS\MARKDOWN DOCUMENTS\Internet Protocol\ComputerNetwork\ImageResources\1_008)

> 小结:
>
> 局域网: 总线型，星型，环型
> 广域网: 不规则型
>
> 点—点通信: 星型、不规则型 独占信道
> 多点通信: 总线型、环型 共享信道



## <span id="2.2">网络中任意两台计算机系统之间可以采取哪些方式进行通信？分别有哪些特点？</span>

> 两个计算机系统进行通信实际上是指两个分别位于不同计算机系统中的程序之间进行通信。

存在两种方式：客户服务器方式和对等方式

- 客户服务器方式：通常称为C/S方式
  -  客户端软件特点：
    - 被用户调用后运行，在打算通信时主动向远地服务器发起通信（服务请求）。因此，客户程序必须知道服务器程序的地址。
    - 不需要特殊的硬件和很复杂的操作系统。 
  - 服务端软件特点：
    - 一种专门用来提供某种服务的程序，可同时处理多个远地或本地客户的请求。
    - 系统启动后即自动调用并一直**不断地运行**着，**被动地等待**并接受来自各地的客户的通信请求。因此，服务器程序不需要知道客户程序的地址。
    - 一般需要强大的硬件和高级的操作系统支持。

- 对等方式：通常称为P2P模式
  - 两个主机在通信时并不区分哪一个是服务请求方还是服务提供方。对等连接方式从本质上看仍然是使用客户服务器方式，只是对等连接中的每一个主机既是客户又同时是服务器。

## <span id="2.3">数据交换交换的作用是什么？按照数据交换形式，计算机网络可以分为哪几种？各有什么特点？</span>

> ![image-20230302203719178](C:\Users\m1518\OneDrive\DOCUMENTS\MARKDOWN DOCUMENTS\Internet Protocol\ComputerNetwork\ImageResources\1_009)

数据交换是在多个数据终端设备之间，为任意两个终端设备建立数据通信临时互连通路的过程。其作用为接收来自源主机系统的数据，并向目的主机系统转发。

数据交换有三种方式——电路交换、报文交换、分组交换

- 电路交换：
  - 电路交换必定是面向连接的。 
  - 电路交换三阶段：Ⅰ.建立连接 Ⅱ.通信 Ⅲ.释放连接
  - 信道资源独占，通信线路的利用率很低
  - 实时性好

![image-20230302204332122](C:\Users\m1518\OneDrive\DOCUMENTS\MARKDOWN DOCUMENTS\Internet Protocol\ComputerNetwork\ImageResources\1_010)![image-20230302204405984](C:\Users\m1518\OneDrive\DOCUMENTS\MARKDOWN DOCUMENTS\Internet Protocol\ComputerNetwork\ImageResources\1_011)

- 分组交换：
  - 优点：
    - 高效：动态分配传输带宽，对通信链路是逐段占用。 
    - 灵活：以分组为传送单位和查找路由。
    - 迅速：不必先建立连接就能向其他主机发送分组。
    - 可靠：保证可靠性的网络协议；分布式的路由选择协议使网络有很好的生存性。  
  - 缺点：
    - 分组在各结点存储转发时需要排队，这就会造成一定的时延。 
    - 分组必须携带的首部（里面有必不可少的控制信息）也造成了一定的开销。 

- 报文交换：
  - 时延较长
  - 基于存储转发原理

![image-20230302205659820](C:\Users\m1518\OneDrive\DOCUMENTS\MARKDOWN DOCUMENTS\Internet Protocol\ComputerNetwork\ImageResources\1_012)

## 阐述分组交换的工作过程，说明其特点。

- 工作过程：
  - 首先，在发送端，我们先把较长的报文划分成较短的、固定长度的数据段。
  - 完成划分后，我们对每一个数据段的段首加入“首部”构成不同的“分组”。
  - 之后分组交换网以“分组”作为数据传输单元，依次把各组发送到接收端。
  - 在接收端，收到分组后会剥去其首部以还原成报文。
  - 最后，在接收端把收到的数据恢复成为原来的报文。

- 特点：
  - 高效：动态分配传输带宽，对通信链路是逐段占用。 
  - 灵活：以分组为传送单位和查找路由。
  - 迅速：不必先建立连接就能向其他主机发送分组。
  - 可靠：保证可靠性的网络协议；分布式的路由选择协议使网络有很好的生存性。  
  - 分组在各结点存储转发时需要排队，这就会造成一定的时延。 
  - 分组必须携带的首部（里面有必不可少的控制信息）也造成了一定的开销。

## 请分别阐述你对“带宽”、“速率”、“吞吐量”、“时延”和“信道利用率”的认识，并说明信道利用率并非越高越好的原因。

- 速率：“速率”即数据率或比特率，是计算机网络中最重要的一个性能指标。速率的单位是b/s, 或kb/s, Mb/s等。“速率”往往是指额定速率或标称速率。

- 带宽：“带宽”本来是指信号具有的频带宽度，单位是赫（或千赫、兆赫等），但现在，“带宽”体现了传输管道中可以传递数据的能力，它通常是数组信道所能传送的“最高数据率”的同义语，单位是“比特每秒”(bps, b/s, bit/s)，它还有多个不同大小的单位。

- 吞吐量：“吞吐量”表示在单位时间内通过某个网络（或信道、接口）的数据量，它经常地用于对现实世界中的网络的一种测量，以便知道实际上到底有多少数据量能够通过网络，不过通常也受网络的带宽或网络的额定速率限制。

- 时延：“时延”可分为处理时延、排队时延、传输时延和传播时延。

  - **传输时延（发送时延）**是发送数据时，数据块从结点到进入传输介质所需的时间。

  ![image-20230302211622698](C:\Users\m1518\OneDrive\DOCUMENTS\MARKDOWN DOCUMENTS\Internet Protocol\ComputerNetwork\ImageResources\1_013)

  - **传播时延**是电磁波在信道中需要传播一定的距离而花费的时间。

  ![image-20230302211635670](C:\Users\m1518\OneDrive\DOCUMENTS\MARKDOWN DOCUMENTS\Internet Protocol\ComputerNetwork\ImageResources\1_014)

  - **处理时延**是交换结点为存储转发而进行一些必要的处理所花费的时间。
  - **排队时延**是结点缓存队列中分组排队所经历的时延。
  - **数据经历的总时延**就是发送时延、传播时延、处理时延和排队时延之和: **总时延 = 发送时延 + 传播时延 + 处理时延 + 排队时延**

![image-20230302211812281](C:\Users\m1518\OneDrive\DOCUMENTS\MARKDOWN DOCUMENTS\Internet Protocol\ComputerNetwork\ImageResources\1_015)

> 注意：对于**高速网络链路**，我们提高的仅仅是**数据的发送速率**，**不是比特在链路上的传播速率**。 提高链路带宽**减小了数据的发送时延**。 

- **信道利用率**：信道利用率**指出某信道有百分之几的时间是被利用的**（有数据通过）。完全空闲的信道的利用率是零。

  **网络利用率**：则是全网络的信道利用率的加权平均值。

>信道利用率并非越高越好，理由如下：
>
>> 根据排队论的理论，当某信道的利用率增大时，该信道引起的时延也就迅速增加。
>
>若令 D0 表示网络空闲时的时延，D 表示网络当前的时延，则在适当的假定条件下，可以用以公式：
>$$
>D=D_0/(1-U)
>$$
>来表示 D 和 D0之间的关系，其中 U 是信道利用率，数值在 0 到 1 之间。
>
>由该公式可简单推知，信道利用率过高会导致时延增大，所以信道利用率并非越高越好

![image-20230302212551910](C:\Users\m1518\OneDrive\DOCUMENTS\MARKDOWN DOCUMENTS\Internet Protocol\ComputerNetwork\ImageResources\1_016)

# Quiz

## 什么是计算机网络？从结构、分类等方面说说你对计算机网络的认识



## <a href="#2.2">说说你对C/S模式和P2P模式的认识</a>



## <a href="#2.3">路由器的主要功能是什么？说说你对数据交换方式的认识</a>

![image-20230302203719178](C:\Users\m1518\OneDrive\DOCUMENTS\MARKDOWN DOCUMENTS\Internet Protocol\ComputerNetwork\ImageResources\1_009)

## 为什么说带宽增大速率就会提高？

## 说说你对时延的认识

## 信道利用率是不是越高越好？为什么？

## 说说你对因特网的三级ISP的认识
