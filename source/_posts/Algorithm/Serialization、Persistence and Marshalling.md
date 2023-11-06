---
title: Hash Function
date: 2022-12-23 12:23:23
tags:
  - Serialization
  - Algorithm
categories:
  - Algorithm
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

1. 序列化及其逆过程要解决的核心问题是 实现相同的数据在不同格式间的转化；

2. 持久化要解决的则是内存中数据结构到硬盘上数据的转化 ，比如比特流或者xml格式的文件；分布式系统数据层都需要做持久化的工作，要么存到数据库中、要么直接以文件形式保存到硬盘上；

3. marshalling要解决的问题和serialization类似，但它更加关注网络间数据传输、另有种说法认为marshalling包括跟数据转化有关的codebase;



## First，What is Serialization

引用C#文档中对**serialization**的定义：

> Serialization is the process of **converting an object into a stream of bytes** to store the object or transmit it to memory, a database, or a file. <br />**Its main purpose is to save the state of an object in order to be able to recreate it when needed.** <br />The reverse process is called deserialization.
>
> ## How serialization works
>
> This illustration shows the overall process of serialization:
>
> ![Serialization graphic](..\..\images\Algorithm\serialization-process.gif)
>
> The object is serialized to a stream that carries the data. The stream may also have information about the object's type, such as its version, culture, and assembly name. From that stream, the object can be stored in a database, a file, or memory.
>
> ### Uses for serialization
>
> Serialization allows the developer to save the state of an object and re-create it as needed, providing storage of objects as well as data exchange. Through serialization, a developer can perform actions such as:
>
> - Sending the object to a remote application by using a web service
> - Passing an object from one domain to another
> - Passing an object through a firewall as a JSON or XML string
> - Maintaining security or user-specific information across applications

[Serialization (C#) | Microsoft Docs](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/serialization/)



<a href="#whatIsSerialization">序列化(serialization)</a>过程就是复杂数据在空间上的降维，最终结果是将一个数据结构或者包含对象状态的数据转换为一种可以在计算机文件系统或者内存缓存中被存储或者在计算机网络上被传输的数据格式，当它根据序列化格式被解析时，计算机就能生成一个语义上与原始数据相同的克隆。

数据序列化不是数据持久化！数据序列化只是 “可以作为数据持久化的中间过程” 而已

数据复杂度的降维

## 几个数据序列化标准介绍与比较

[【Netty入门】几种序列化协议的介绍_白夜行-CSDN博客_netty序列化协议](https://blog.csdn.net/baiye_xing/article/details/73249819)

## Marshalling



## The difference between Serialization and Marshalling

[terminology - What is the difference between Serialization and Marshaling? - Stack Overflow](https://stackoverflow.com/questions/770474/what-is-the-difference-between-serialization-and-marshaling)



## 补充材料

<span id="marshalling">https://en.wikipedia.org/wiki/Marshalling_(computer_science)</span>

<span id="marshallingCNPage">https://zh.wikipedia.org/wiki/Marshalling_(%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%A7%91%E5%AD%A6)</span>

<span id="rfc2713">[rfc2713 (ietf.org)](https://datatracker.ietf.org/doc/html/rfc2713)</span>

[序列化理解起来很简单 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/40462507)
