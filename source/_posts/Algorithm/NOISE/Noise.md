---
title: Noise Generate
date: 2022-12-23 12:23:23
tags:
  - Noise
  - Algorithm
categories:
  - Algorithm
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---
# 噪声 - Noise

> Randomness is needed to make things unpredictable, varied, and appear natural.
> ——catlike coding

在任何游戏中，`噪声`都是不可缺少的存在。因此，了解这些噪声的特点、用途以及生成算法都显得十分重要

本文从基于 shader graph 中内置的三种程序化噪声，记录了以下噪声：

- Simple Noise
- Gradient Noise(Perlin Noise)
- Voronoi Noise
- Value Noise

![image-20230325210305648](........\source\images\Algorithm\NOISE\001.png)
_The built-in noise in unity ShaderGraph._

下面，我们试着自己去实现这些噪点算法

## Hashing

> we have a process that for any specific input yields a unique and fixed apparently random output. This is what hash functions are for.
> ——catlike coding

## Hashing Space

## Value Noise

## Perlin / Gradient Noise

## Noise Variants

## Voronoi Noise

## Simplex Noise

> Simplex noise is a type of gradient noise, but we can also create value noise variants of it. We start with those because they are simpler and easier to analyze than the gradient variants.

## Reference

- [Unity Pseudorandom Noise Tutorials (catlikecoding.com)](https://catlikecoding.com/unity/tutorials/pseudorandom-noise/)
- [游戏开发技术杂谈 4：柏林噪声 1 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/354931692)
