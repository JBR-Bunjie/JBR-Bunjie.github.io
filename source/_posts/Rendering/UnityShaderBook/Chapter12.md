---
title: Unity Shader入门精要笔记 - Chapter12
date: 2023-3-8 21:29:25
tags:
  - Unity
  - Shader
categories:
  - Unity
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

# Unity Shader入门精要笔记 - Chapter12

> If you are doing a series of post-processing "blits", it's best for performance to get and release a temporary render texture for each blit, instead of getting one or two render textures upfront and reusing them. This is mostly beneficial for mobile (tile-based) and multi-GPU systems: GetTemporary will internally do a [DiscardContents](https://docs.unity3d.com/ScriptReference/RenderTexture.DiscardContents.html) call which helps to avoid costly restore operations on the previous render texture contents.