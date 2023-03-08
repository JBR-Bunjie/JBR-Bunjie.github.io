---
title: Unity Shader入门精要笔记 - Chapter9
date: 2023-3-8 21:29:05
tags:
  - Unity
  - Shader
categories:
  - Unity
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

# Chapter 9: 更复杂的光照 - more complex lights

## 1.渲染路径



关于书中提及的与编译指令 `pragma multi_compile_fwdbase` 类似的官方文档：[Unity - Manual: Declaring and using shader keywords in HLSL (unity3d.com)](https://docs.unity3d.com/Manual/SL-MultipleProgramVariants.html)

> 关于 `shader variant`，可以参考这篇文章：[unity ShaderVariants处理 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/369425274)
>
> > You can write **shader** snippets that share common code, but have different functionality when a given keyword is enabled or disabled. 
> >
> > When Unity compiles these shader snippets, it creates separate shader programs for the different combinations of enabled and disabled keywords. These individual shader programs are called shader variants.

简单来说，这种操作类似于，我们通过在Shader运行时动态的改变宏(这里是 `Keyword`)的方式来动态得到我们需要的结果。

> 利用Inspector来快速检测以及debug：[[Unity\]各种Debug方法笔记_unity debug_HytMiao的博客-CSDN博客](https://blog.csdn.net/qq_38642203/article/details/80151194)

