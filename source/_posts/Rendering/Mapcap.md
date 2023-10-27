---
title: Mapcap
date: 2023-4-29 12:23:23
tags:
  - Unity
  - Shader
categories:
  - Unity
  - Shader
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

## What is Mapcap?

> > MatCap is a method of light expression using pre-rendered images. This technique uses a picture of a sphere that represents the material and light to simulate lighting.
> >
> > [Material Capture(MatCap) Settings | Unity Toon Shader | 0.9.4-preview (unity3d.com)](https://docs.unity3d.com/Packages/com.unity.toonshader@0.9/manual/MatCap.html)
>
> > _Material capture ("MatCap") shaders_ are popular effects used in most 3D modeling tools: A material is captured by rendering a sphere into a texture. Models can be shaded by making a simple lookup into the reference texture.
> > This type of shading is very efficient – no complicated computations, only a single texture lookup per pixel. The effect works great for previewing objects in a 3D modeling tool. **However, it is less suited for games: The reference texture only stores one side of a sphere. The effect breaks down, when the camera moves around the object.**
> >
> > [Material Capture (MatCap) Shaders (digitalrune.github.io)](https://digitalrune.github.io/DigitalRune-Documentation/html/9a8c8b37-b996-477a-aeab-5d92714be3ca.htm)
>
> [【Unity Shader 实践】基于 MatCap 实现适于移动平台的“次时代”车漆 Shader - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/25314364)

## 应用

### 真实感渲染

简单来说，我们会用 matcap 贴图来存储很多预制好的光照、色彩等具体效果，这样我们用一张贴图就可以快速得到我们所希望的材质效果。

对于正常的真实感渲染上，matcap 的用法具有较大的局限性——因为真实感渲染下我们往往要求计算过程遵守物理规则，因此一般仅用 matcap 来节约性能开销：我们可以为场景中的不重要的景物、物件等使用 matcap，这样能保证它们具有足够质感的同时，尽可能的节约硬件资源。

这一点好像又和 LOD 等东西也可以结合起来看待，有一种处理方法就是，当离人物距离太远时，比如远方的树木，不再执行完整的渲染，而一种永远正对角色的面片，而这种改用这种 matcap 来实现，或者改用别的节省资源的办法等。

其一般用法可以参看：[(美学阿姨)MatCap 材质原理讲解与 UE5 中的实现方法\_哔哩哔哩\_bilibili](https://www.bilibili.com/video/BV1GR4y1b76n/?spm_id_from=333.337.search-card.all.click&vd_source=c8eda79dd90c30ff02e09fb39906ac54)

但是，当到了 NPR 领域，就完全不是这么一回事了。

### NPR 下的 Matcap

先从 mmd 中说起：

mmd 中一般会有三类贴图，包括：SPA\\SPH、TEX、TOON。后两者很好理解，关键是 SPA\\SPH 贴图。

就简单来说，我们可以参考这个：

- [Spa 及 Sph 的高光貼圖介紹 (nagongze.me)](https://www.nagongze.me/mmd-sphere-texture/)
- [[MMD\] Why model should use Matcap. by MMDLOID on DeviantArt](https://www.deviantart.com/mmdloid/art/MMD-Why-model-should-use-Matcap-908386574)

发现了吗，其实 SPA\\SPH 所产生的效果很相似，都是将提前存储好的内容映射到模型上。就细分而言，SPA\\SPH 是基于球体映射的贴图技术，它将 2D 图像映射到 3D 模型的球体表面，用于模拟光照效果。我们用它来为模型表面创建天使环等效果。而 Matcap 贴图则用于模拟模型表面的材质和光照效果。这么看来 SPA\\SPH 好像可以直接归到 Matcap 的一个子类里。

也就是说，我们可以在 NPR 中使用 matcap 来实现很多特殊的内容，比如 Angle Ring 等

## 实际使用：Matcap 采样 uv 的构造

### 经典 Matcap 采样

### 改进 1：

[MatCap Shader 改进：解决平面渲染和环境反射问题 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/79040521)

## 补充：法线变换

补一个将模型法线转换到视角空间

- 数学推导：
- 结论：[Transformation of normals into eye space. - OpenGL / OpenGL: Advanced Coding - Khronos Forums](https://community.khronos.org/t/transformation-of-normals-into-eye-space/63611)

## 实际使用案例

- [blender 中使用 mmd 中 spa 贴图的节点分享\_哔哩哔哩\_bilibili](https://www.bilibili.com/video/BV12Y4y157sp/?vd_source=c8eda79dd90c30ff02e09fb39906ac54)
