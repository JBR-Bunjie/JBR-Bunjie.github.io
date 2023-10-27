---
title: URP Chapter2 - 光与影
date: 2023-4-5 21:32:05
tags:
  - Algorithm
categories:
  - Algorithm
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

# Chapter2 - 光与影

这一章节我们开始研究 URP 管线中的光照与阴影函数，让我们从光照开始

## 使用空项目

这一次我们不再使用 Unity 下的示例工程——让我们现从 Hub 重新创建一个 URP Core 的新项目，并以其为基础进行接下来的工作

![image-20230329202320686](..\..\..\images\Dev\Unity\UpdateToURPPipeline\Chapter2_ShadowAndLight\001.png)

_使用 Hub 中的 URP Temple 来创建新项目_

## 从 Lambert 模型开始

> 对应代码可以参考 <a href="#LambertCode">`Reference`</a>

事实上，URP 中已经预定义了多种光照模型，这其中就包括了 Lambert

## 光

### 主光源

设置 LightMode

### 应用材质

解释：

_此属性仅在渲染路径设置为 `Forward` 时可用_

`Depth Priming Mode` 属性决定了 Unity 何时执行 `depth priming` ——它可以通过减少像素着色器执行次数来改善 GPU 帧时间。其带来的性能提升取决于不透明通道中重叠像素的数量以及 Unity 可以通过使用深度优化跳过的像素着色器的复杂度。该功能具有前期内存和性能成本。该功能使用深度预处理来确定哪些像素着色器调用 Unity 可以跳过，并且如果尚不可用，则添加深度预处理。

选项有：

- 禁用：Unity 不执行深度优化。
- 自动：如果有需要深度预处理的渲染通道，则 Unity 执行深度预处理和深度优化。
- 强制：Unity 始终执行深度优化。为此，Unity 还为每个渲染通道执行深度预处理。
  请注意，无论此设置如何，在某些（基于平铺的延迟渲染的）硬件上运行时都会禁用深度优化。

在 Android、iOS 和 Apple TV 上，Unity 仅在强制模式下执行深度优化。在这些平台通用的平铺 GPU 上，深度启动与 MSAA 结合使用时可能会降低性能。

简而言之，“Depth Priming Mode”是一个属性，它决定了 Unity 何时执行深度优化，以改善 GPU 帧时间。

![image-20230329202802662](..\..\..\images\Dev\Unity\UpdateToURPPipeline\Chapter2_ShadowAndLight\002.png)
_Depth Priming Mode_

> This property determines when Unity performs depth priming.
> Depth Priming can improve GPU frame timings by reducing the number of pixel shader executions. The performance improvement depends on the amount of overlapping pixels in the opaque pass and the complexity of the pixel shaders that Unity can skip by using depth priming.
> The feature has an upfront memory and performance cost. The feature uses a depth prepass to determine which pixel shader invocations Unity can skip, and the feature adds the depth prepass if it's not available yet.
> The options are:
>
> - **Disabled**: Unity does not perform depth priming.
> - **Auto**: If there is a Render Pass that requires a depth prepass, Unity performs the depth prepass and depth priming.
> - **Forced**: Unity always performs depth priming. To do this, Unity also performs a depth prepass for every render pass. **Note**: Depth priming is disabled at runtime on certain hardware (Tile Based Deferred Rendering) regardless of this setting.
>
> On Android, iOS, and Apple TV, Unity performs depth priming only in the Forced mode. On tiled GPUs, which are common to those platforms, depth priming might reduce performance when combined with MSAA.
>
> This property is available only if **Rendering Path** is set to **Forward**



## 影

> Modify the alpha value in ShadowCaster, but it seems ShadowCaster only cares if the alpha value exists. If alpha >= 0, the object has shadow. If alpha < 0, the object has no shadow. Alpha's value doesn't change the strength like the Built-in pipeline.
>
> [Question - transparent object's shadow strength - Unity Forum](https://forum.unity.com/threads/transparent-objects-shadow-strength.1156214/)
>
> ？
>
> 关于“实时调变半透明物体阴影”这一问题，看样子是没有能成功的解决办法
> 那么就只能自我开脱：我们到底是需要“阴影”，还是只是需要给当前的下层不透明物体加深颜色？对于水面来说，显然是后者
> 那么我们至于要做到这一点即可——而单纯的混色就已经可以完成这一要求了——没必要再花大力气去做可变阴影

> In Unity’s Universal Render Pipeline (URP), `GrabPass` is not supported. However, there are some alternative methods to achieve similar effects such as refraction. One method is to use `OpaqueTexture` which captures a texture of opaque objects after they have been rendered [1](https://zhidao.baidu.com/question/272034073067464685.html). Another method is to use multiple cameras and layers to control the rendering order of objects and capture the desired texture [2](https://www.jianshu.com/p/a9706ea587c9). You may want to check out these resources for more detailed information on how to implement these methods in URP.



## Reference

- <span id="LambertCode">UnityShader 入门精要-URP 管线 2. 基础光照 - 王子饼干的文章 - 知乎 https://zhuanlan.zhihu.com/p/232193169</span>
- LightMode 设置：
  - [Unity - Manual: ShaderLab: assigning tags to a Pass (unity3d.com)](https://docs.unity3d.com/Manual/SL-PassTags.html)
  - [URP ShaderLab Pass tags | Universal RP | 12.1.10 (unity3d.com)](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@12.1/manual/urp-shaders/urp-shaderlab-pass-tags.html)
  -
