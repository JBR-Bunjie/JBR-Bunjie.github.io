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

> Last Changed: 3/9/2023

## 1. 渲染路径

Unity渲染路径文档：

> [Unity - Manual: Rendering paths in the Built-in Render Pipeline (unity3d.com)](https://docs.unity3d.com/Manual/RenderingPaths.html)
>
> 

关于书中提及的与编译指令 `pragma multi_compile_fwdbase` 类似的官方文档：[Unity - Manual: Declaring and using shader keywords in HLSL (unity3d.com)](https://docs.unity3d.com/Manual/SL-MultipleProgramVariants.html)

> 关于 `shader variant`，可以参考这篇文章：[unity ShaderVariants处理 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/369425274)
>
> > You can write **shader** snippets that share common code, but have different functionality when a given keyword is enabled or disabled. 
> >
> > When Unity compiles these shader snippets, it creates separate shader programs for the different combinations of enabled and disabled keywords. These individual shader programs are called shader variants.

简单来说，这种操作类似于，我们通过在Shader运行时动态的改变宏(这里是 `Keyword`)的方式来动态得到我们需要的结果。

> 利用Inspector来快速检测以及debug：[[Unity\]各种Debug方法笔记_unity debug_HytMiao的博客-CSDN博客](https://blog.csdn.net/qq_38642203/article/details/80151194)



### Forward

>[Unity - Manual: Forward rendering path (unity3d.com)](https://docs.unity3d.com/Manual/RenderTech-ForwardRendering.html)
>
>> 

![forward_rendering.png-175.5kB](http://static.zybuluo.com/candycat/575lq2zgnsaop3nw2miyobt3/forward_rendering.png)

### Deferred

> [Unity - Manual: Deferred Shading rendering path (unity3d.com)](https://docs.unity3d.com/Manual/RenderTech-DeferredShading.html)
>
> >## Overview
> >
> >When using deferred shading, there is no limit on the number of lights that can affect a **GameObject**. All lights are evaluated per-pixel, which means that they all interact correctly with **normal maps**, etc. Additionally, all lights can have cookies and shadows.
> >
> >*Deferred shading has the advantage that the processing overhead of lighting is proportional to the number of **pixels** the light shines on. This is determined by the size of the light volume in the **Scene** regardless of how many GameObjects it illuminates.* Therefore, performance can be improved by keeping lights small. Deferred shading also has highly consistent and predictable behaviour. The effect of each light is computed per-pixel, so there are no lighting computations that break down on large triangles.
> >
> >On the downside, deferred shading has no real support for anti-aliasing and can’t handle semi-transparent GameObjects (these are rendered using [forward](https://docs.unity3d.com/Manual/RenderTech-ForwardRendering.html) rendering). There is also no support for the **Mesh** Renderer’s Receive Shadows flag and **culling masks** are only supported in a limited way. You can only use up to four culling masks. That is, your culling **layer mask** must at least contain all layers minus four arbitrary layers, so 28 of the 32 layers must be set. Otherwise you get graphical artifacts.

## 2. 光源种类

> 每一个光源有五个属性：
>
> - 位置
> - 方向
> - 颜色
> - 强度
> - 衰减

### 平行光

- 最简单的光源：
  - 影响范围无限大，因此，平行光光源的位置属性没有意义——它的几何属性只有方向
  - 光照强度没有衰减

在场景中，一般只有太阳会作为平行光存在。Unity中默认场景中的初始光源即为Directional Light

![directional_ligth.png-51.6kB](http://static.zybuluo.com/candycat/uadla1q69533nc71z7g7ep0g/directional_ligth.png)

### 点光源

![point_ligtht.png-89.4kB](http://static.zybuluo.com/candycat/tvbpd08wgc0s1o31v4nw20ad/point_ligtht.png)

> 可以注意到场景中的 `Sun` 变为了一个 `Lamp` 

> 需要开启Scene视图中的光照才能看到效果

### 聚光灯

![spot_light.png-74.5kB](http://static.zybuluo.com/candycat/tx45g2n04xypq5cdlyblecrv/spot_light.png)



### 面光源
