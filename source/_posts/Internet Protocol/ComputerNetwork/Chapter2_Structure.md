> ## **课前思考**
>
> 如何在各自独立、相互平等的计算机系统之间有条不紊的通信？
> 				——建立秩序
> 如何在结构相异、标准不同的计算机系统之间进行数据交换和资源共享？
> 				——建立共识

# 1.课后习题

## 1.阐述计算机网络体系结构的基本思想和主要特点。

- 基本思想：将一个庞大、复杂的问题进行分解，产生若干个功能相对单一、结构比较简单、处理更为方便的局部问题，并基于此建立对应多个层级来解决问题。每层在自己下层所提供服务的支持下，通过自身内部功能实现一种或几种特定的服务。
- 主要特点：
  - 耦合度低(独立性强)：每层只需调用下层接口即可获得下层的服务，无需关心下层的具体实现。即在上层看来，下层是具有特定功能的黑箱。　
  - 适应性强：只要每层提供的服务和接口不变，其内部实现细节可以任意改变。　
  - 易于实现和维护：把复杂的系统分解成若干个涉及范围小且功能简单的子系统，使得系统结构更加清晰，系统实现、调试和维护都变得简单和容易。

## 2.结合定义，谈谈你对计算机网络体系结构的认识。

首先，计算机网络体系结构的定义为：层、层间接口及协议的集合被称为计算机网络体系结构。

这个定义与基于分层思想所需要解决的三个问题深度绑定的

而分层思想有三个主要问题：

- 确定分层与功能，即确定网络体系结构应该具哪些层次，以及每层应该具有哪些功能。

- 确定服务与接口，即确定每个层次为上层提供哪些服务，以及上层调用这些服务的接口规格。

- 确定协议，即确定每层的必须遵守的规则，以便确保通信的两方可以达成高度默契。

因此，计算机网络体系结构的设计与划分就必须解决这三方面的内容。即通过对分层结构、每层的功能及服务与层间接口和协议的设计，解决我们的主要问题。基于这些内容，我们自然得出了我们的定义。

 

## <span id="1.3">3.阐述你对OSI/RM、Internet模型和五层结构模型的认识。</span>

- OSI/RM模型
  - OSI/RM体系结构具有概念清楚、理论完整的特点，是一个理论上的国际标准，但却不是事实上的国际标准；
  - OSI/RM模型具体架构：
    - **最高层为应用层，这是计算机用户以及各种应用程序和网络之间的接口，能 <u>*直接向用户提供服务*</u> 并完成用户希望在网络上完成的各种工作，本层为用户的 <u>*应用进程*</u> 提供网络通信服务。**
    - 第二层是表示层，会对上层数据或命令进行解释，以保证一个主机应用层信息可以被另一个主机的应用程序理解，使应用程序能够理解数据报文的真实含义。
    - 第三层是会话层，这是应用程序和网络之间的接口，负责在网络中的两节点之间建立、维持和终止通信向两个实体的表示层提供建立和使用连接的方法。
    - 第四层是传输层，这是**通信子网**与**资源子网**的分界层，同时也是通信子网和资源子网的接口和桥梁。本层向高层屏蔽数据通信的所有细节，为用户提供透明的报文传输，保证报文可靠的、正确的传输。提供端到端的可靠的透明数据传输，使上层服务用户不必关心通信子网的实现细节。
    - 第五层是网络层，这是通信子网中的最高层，在下两层的基础上向资源子网提供服务。在选择路由时，网络层将综合考虑优先权、网络拥塞程度、服务质量以及路由代价等决定从一个节点 到另一个节点的最佳路径。将网络地址翻译成对应的物理地址，并通过路由选择算法为分组通过通信子网选择最适当的路径。
    - 第六层是数据链路层，本层负责建立和管理节点间的链路，控制网络层与物理层之间的通信。通过差错控制、流量控制方法，在不可靠的物理介质上提供可靠的数据传输。
    - 物理层是OSI的最底层，也是真正将数据在传输媒介上进行传输的层次。该层会为上层实现在物理介质上正确、透明的传送比特流。规定激活、维持、关闭通信端点之间的机械特性、电气特性、功能特性以及过程特性，为上层协议提供一个传输数据的物理媒体。在物理介质上正确地、透明地传送比特流。即在源结点将1、0转化为强弱不同的电信号并进行传输,在目的结点，将强弱不同的电信号转化为1、0，也就是数模转换与模数转换。

- Internet模型
  - **因为模型中两个核心协议为TCP和IP协议，Internet模型也称为TCP/IP模型。**
  
  - TCP/IP协议簇是一个四层体系结构，具有简单易用特点，是事实上的国际标准。
  
  - TCP/IP协议标准完全开放，可免费使用，且独立于特定计算机硬件及操作系统。
  
  - 独立于网络硬件系统，可以运行在广域网，更适合于互联网。
  
  - 网络地址统一分配，网络中每一设备和终端都具有一个唯一地址。
  
  - 高层协议标准化，可以提供多种多样可靠网络服务。
  
  - TCP/IP协议具体架构：（以OSI/RM七层模型为标准）
    - 应用层：大体对应OSI参考模型的应用层、表示层和会话层。该层协议主要包括: 
      > - FTP（文件传输协议）
      > - SMTP(简单报文传输协议)
      > - TELNET(远程网络登陆协议)
      > - DNS(域名服务)  
      > - HTTP（超文本传输协议）
      
    - 传输层：大体上对应OSI的传输层。该层协议包括：
    
      > - TCP(传输控制协议Transmission control protocol)：可靠的，面向连接的协议。将报文以字节流形式从源主机进程发到目的主机进程。
      > - UDP(用户数据报协议User Datagram protocol )：不可靠的，无连接的协议。
    
    - 网际层：大体上对应OSI的网络层。该层的协议是
    
      > IP(INTERNET PROTOCOL) 
      > Internet 体系结构的核心协议。把IP分组以数据报方式从源主机发送到目的主机。
    
    - 网络接口层：大体上对应OSI的物理层和链路层。
    
      > **Internet**体系结构的网络接口层也称为子网层，具有开放的特点。子网协议有：
      >
      > - 以太网协议
      > - FDDI
      > - PPP
      > - SLIP
      > - Token Bus (802.4)
      > - 百兆，千兆，万兆以太网
  
- 五层结构模型
  - 五层体系结构是结合了OSI/RM和TCP/IP协议簇的优点而提出来的一个五层结构的模型，但其存在意义只是为了学术学习研究，没有具体的实际意义。
  - 五层模型结构从上到下依次为：应用层、传输层、网络层、数据链路层、物理层

# 2.课堂提问

## 1.说说你对计算机网络体系结构的理解

基于1.1及1.2中的认识，我们划分出计算机网络的体系结构

### 补充部分

> 从定义到实现：
>
> 计算机网络体系结构是关于计算机网络及其部件所应完成功能的精确定义。
>
> 实现(implementation)是在遵循这种体系结构的前提下用合适的硬件或软件完成这些功能的过程。
>
> 体系结构是抽象的，实现则是具体的，是真正在运行的计算机硬件和软件。

![image-20230303142731465](C:\Users\m1518\OneDrive\DOCUMENTS\MARKDOWN DOCUMENTS\Internet Protocol\ComputerNetwork\ImageResources\2_001)

- 实体：任何可发送或接收信息的硬件或软件进程。
- 接口：同系统内相邻层之间的交互通道，也称为服务访问点**SAP**（Service Access Point）。
- 协议：**控制对等层实体间进行数据交换的规则、约定和标准。**
- 服务：为保证上层对等实体间的通信，由下层向上层提供的功能。

协议的影响范围是对应的那一层(系统间的对等层)，有“协议是水平的”之说

服务的影响范围上下相邻层，有“服务是垂直的”之说

![image-20230303143700028](C:\Users\m1518\OneDrive\DOCUMENTS\MARKDOWN DOCUMENTS\Internet Protocol\ComputerNetwork\ImageResources\2_002)



## 2.说说你对网络协议的理解

> 网络协议(network protocol，简称协议)，是为进行网络数据交换而建立的规则、标准或约定。 **也就是说，对等实体间的共识就是协议。**

**协议：网络对等实体间为完成数据交换所必须遵循的规则、约定和标准。**

在构建通信协议时，存在三要素：语法、语义和时序（同步）。

- 语法是指协议元素与数据的组合结构，也就是 报文格式。
- 语义是指对协议中各协议元素的含义的解释。
- 时序（也称为同步）是指在通信过程中，通信两方操作的执行顺序与规则。

**语法规定了报文格式，语义赋予了报文特定内涵；时序则通过控制具有特定语义的报文的交换，从而实现了计算机间的通信。**

> 语法是语义的载体，时序又是对语义的有序组织。正是基于这样的关系，计算机在通信时才得以保持高度默契。

在定义好协议后，便可以依据协议内容进行数据(信息报文)的传输：

![image-20230303145337491](C:\Users\m1518\OneDrive\DOCUMENTS\MARKDOWN DOCUMENTS\Internet Protocol\ComputerNetwork\ImageResources\2_003)

>**两个端系统之间的数据通信过程本质上就是PDU的不断封装和解封过程:**
>
>- 在**源系统**，数据在**自上而下**逐层递交的过程就是**PDU不断封装**的过程；
>- 在**目的系统**，当数据到达后**自下而上**递交的过程就是**PDU不断拆封**的过程。
>
>PDU具体变化可参考<a href="#3.1">3.1</a>

## 3.说说你对OSI/RM与TCP/IP两个模型的理解

<a href="#1.3">1.3</a>

## 4.说说你对应用层的认识，举例说说你所知道的应用层协议

<a href="#1.3">1.3</a>

# 3.重点内容补充

## <span id="3.1">3.1 两主机间互传数据过程</span>

> **计算机间通信的本质在于信息报文的交换。**

> 基于五层体系结构模型，注意数据链路层同时加装了首部与尾部，而其他层只有首部；最下层的物理层不会加装任何首、尾部，但会将数据正式转化为比特流

![image-20230303150649221](C:\Users\m1518\OneDrive\DOCUMENTS\MARKDOWN DOCUMENTS\Internet Protocol\ComputerNetwork\ImageResources\2_004)

## <span id="3.2">3.2 基于OSI/RM模型的层次解析图</span>

![image-20230303151058783](C:\Users\m1518\OneDrive\DOCUMENTS\MARKDOWN DOCUMENTS\Internet Protocol\ComputerNetwork\ImageResources\2_005)

## 3.3 三种主要模型比较

![image-20230303152536659](C:\Users\m1518\OneDrive\DOCUMENTS\MARKDOWN DOCUMENTS\Internet Protocol\ComputerNetwork\ImageResources\2_006)

特别的，我们以OSI/RM模型为基准，对照地看待TCP/IP时，有如1.3的内容
