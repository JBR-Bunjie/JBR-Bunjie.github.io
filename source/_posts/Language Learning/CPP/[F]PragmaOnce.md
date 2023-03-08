---
title: Cpp `#pragma Once` and `#ifndef`
date: 2022-12-23 12:23:23
tags:
  - CPP
categories:
  - Coding Language
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

# Source

- 在 C++ 中防止头文件被重复包含时为什么同时使用 #ifndef 和 #pragma once？ - 望山的回答 - 知乎 https://www.zhihu.com/question/40990594/answer/1675549910

- [#pragma once - 维基百科，自由的百科全书 (wikipedia.org)](https://zh.m.wikipedia.org/zh-hans/Pragma_once)

# 结论：

请同时使用`#ifdef`与`#pragma once`

解释：如果所用的编译器支持`#pragma once`，则可以加快我们的编译速度；如果不支持，也会有`#ifdef`语句来兜底

>使用`#pragma once`代替include防范将加快编译速度，因为这是一种高阶的机制；[编译器](https://zh.m.wikipedia.org/wiki/編譯器)会自动比对档案名称或[inode](https://zh.m.wikipedia.org/wiki/Inode)而不需要在[标头档](https://zh.m.wikipedia.org/wiki/標頭檔)去判断`#ifndef`和`#endif`。

> 使用#ifndef的话，缺点不单单是需要多写两行代码和想出一个合适的#define名字，更大的缺点是它要求编译器一直扫描到文件尾，找到对应的#endif，才能确定你的意图是整个头文件都要略过。然而这时候已经花费了许多CPU时间（和电能），前面做的那么多都是白忙。