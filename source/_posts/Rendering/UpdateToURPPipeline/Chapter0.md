---
title: Chapter0 - URP 管线分析系列概览
date: 2023-4-3 21:32:05
tags:
  - Algorithm
categories:
  - Algorithm
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

# Chapter0 - URP 管线分析系列概览

## 章节安排

- Chapter 1：Unity URP 管线 `自定义Shader` 小节全内容
- Chapter 2：了解 URP 管线下的光与影
- Chapter 3：URP 管线默认 Shader：`Lit.shader` 分析
- Chapter 4：URP 管线下卡通渲染 Shader 的编写
- Chapter 5：

## 步入 URP Pipline 之前

我们已经知道，Unity Shader 入门精要中所使用的渲染管线是 built-in 管线，这种管线庞大、笨重、可定制性差。虽然 Unity 早早地就将 built-in 管线开源，但是这并不能掩盖管线本身的缺点。

于是，Unity 官方推出了 SRP 管线。同时，还给出了 URP 和 HDRP 管线，SRP 管线有很多好处，而现在我们需要做的，就是开始 URP 管线和 SRP 管线的学习。

## 切换到 URP 管线

文档：

> **Activating URP, HDRP, or a custom render pipeline based on SRP**
>
> To set the active render pipeline to one that is based on SRP, tell Unity which Render Pipeline Asset to use as the default active render pipeline, and optionally which Render Pipeline Assets to use for each quality level.
>
> To do this:
>
> 1.  In your Project folder, locate the Render Pipeline Asset(s) that you want to use.
>
> 2.  Set the default render pipeline, which Unity uses when there is no override for a given quality level. If you do not set this, Unity uses the Built-in Render Pipeline when no override applies.
>
> 3.  Select **Edit** > **Project Settings** > **Graphics**.
> 4.  Drag the Render Pipeline Asset on to the **Scriptable Render Pipeline Setting** field.
>
> 5.  Optional
>
> : Set override Render Pipeline Assets for different quality levels.
>
>     1. Select **Edit** > **Project Settings** > **Quality**.
>     2. Drag the Render Pipeline Asset on to the **Render Pipeline** field.

## Series Main Reference：

- [Unity - Manual: How to get, set, and configure the active render pipeline (unity3d.com)](https://docs.unity3d.com/Manual/srp-setting-render-pipeline-asset.html)
- [编写自定义着色器 | Universal RP | 12.1.1 (unity3d.com)](https://docs.unity3d.com/cn/Packages/com.unity.render-pipelines.universal@12.1/manual/writing-custom-shaders-urp.html)
- [编写 URP 着色器 | Alex Technical Art (cuihongzhi1991.github.io)](https://cuihongzhi1991.github.io/blog/2020/06/08/urpshadercode/)
- UnityShader 入门精要-URP 管线 1. 关于 URP - 王子饼干的文章 - 知乎 https://zhuanlan.zhihu.com/p/228582171
- [Unity - Manual: HLSL in Unity (unity3d.com)](https://docs.unity3d.com/Manual/SL-ShaderPrograms.html)
