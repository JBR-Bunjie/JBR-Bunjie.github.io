---
title: Unity Shader入门精要笔记 - Chapter8
date: 2023-3-8 21:28:45
tags:
  - Unity
  - Shader
categories:
  - Unity
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---


## Alpha Test

只要没通过透明度测试，就舍弃该片元

## Important HLSL Functions

### Clip Function

> https://learn.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-clip
> > Discards the current pixel if the specified value is less than zero.
> > ```shaderlab
> > // HLSL
> > clip(x)
> > ```


## SubShader Tags

> [Unity - Manual: ShaderLab: assigning tags to a SubShader (unity3d.com)](https://docs.unity3d.com/2021.3/Documentation/Manual/SL-SubShaderTags.html)

## Pass Command
一个Pass块就已经定义了一次完整的渲染流程。不过有时候在使用Shader时, 会发现只进行一次渲染是不够的：还要在它的基础上在加上一次或者多次才行，这就是多Pass的渲染的由来。

当运行一个SubShader时，Unity会从头开始，顺序地运行所有Pass块

> 但如果Pass的数目过多，往往会造成性能下降。因此，我们应当尽量使用最小数目的Pass

## Alpha Blend

我们利用Alpha Blend来真正实现透明效果