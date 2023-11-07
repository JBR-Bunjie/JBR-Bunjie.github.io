---
title: 主流 Anti-Aliasing 算法及原理概览
date: 2023-2-25 12:23:23
tags:
  - Algorithm
  - Rendering
  - AA
categories:
  - Algorithm
  - Rendering
  - AA
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

# 主流 Anti-Aliasing 算法及原理概览

> https://zhuanlan.zhihu.com/p/57503957

## Spatial Anti-Aliasing

### SSAA

SSAA，即 Super sampling anti-aliasing，有时也称为 full-scene anti-aliasing (FSAA)，一般认为是性能消耗很大的一种抗锯齿方案。

基本思路为：我们通过在更高的分辨率，使用更多的采样点来进行渲染，并在完成渲染后对结果进行混合以及下采样得到最终的结果

> Color samples are taken at several instances inside the pixel (not just at the center as normal), and an average color value is calculated. This is achieved by rendering the image at a much higher resolution than the one being displayed, then shrinking it to the desired size, using the extra pixels for calculation. The result is a downsampled image with smoother transitions from one line of pixels to another along the edges of objects. The number of samples determines the quality of the output.
>
> https://en.wikipedia.org/wiki/Supersampling

这里有一篇关于具体步骤的总结：https://zhuanlan.zhihu.com/p/484890144

这个做法的核心思路就是面多加水水多加面：既然锯齿的直接原因是采样不足，那我就增加渲染过程中的采样，以得到更好的画面。但是 SSAA 的做法并不算好：这样做的计算量太大了——甚至是对显卡、显存、带宽全方位的压力。

### MSAA

一般认为，MSAA 是唯一一个被硬件支持、有硬件实现的 AA 算法，曾经业界普遍采用的方案之一。不过在延迟管线逐渐兴起、众多 AA 方案如雨后春笋般冒出的当下，MSAA 的一些局限性也开始暴露出来。

> Super sampling anti-aliasing (SSAA), also called full-scene anti-aliasing (FSAA), is used to avoid aliasing (or "jaggies") on full-screen images. SSAA was the first type of anti-aliasing available with early video cards. But due to its tremendous computational cost and the advent of multisample anti-aliasing (MSAA) support on GPUs, it is no longer widely used in real time applications. MSAA provides somewhat lower graphic quality, but also tremendous savings in computational power.
>
> https://en.wikipedia.org/wiki/Spatial_anti-aliasing

对于 MSAA 的过程可作简述如下：我们保留 SSAA 中的采样点，但我们将会提前——在光栅化阶段就进行覆盖判断。记录覆盖结果，并在之后的着色中，对存在通过判断的采样点的像素进行着色。分别对通过的采样点进行深度测试等，最后将通过这些测试的结果保存至对应缓冲的对应位置。在所有渲染工作完成时，我们会进行一步"Resolve"操作，它会合并所有的 multi-sampled textures 以得到最终结果。

> For DirectX, resolving a texture means blending multi-sampled texture into a non-multisampled one. For simple scenarios this is usually done automatically by the output merger, but is often needed to be done explicitly (e.g. ResolveSubresource) when using multi-sampled render target as an input for a next render pass (e.g. post-processing). Reasons being both performance and lower complexity of the following pass shaders.
>
> https://computergraphics.stackexchange.com/questions/9262/what-does-texture-resolve-mean

下图是 DX11 的光栅化说明文档中的 MSAA 示意图，非常详细地展示了 MSAA 的应用原理。

![image-20231006191532143](../../images\Rendering\image-20231006191532143.png)

> 关于 MSAA，可以参考：https://zhuanlan.zhihu.com/p/415087003

从以上的内容中，你大概可以看出：相对于 SSAA，我们降低了显卡的计算量——一个像素块仍然只需要一次计算就足够了，但显存和带宽上的压力仍在——因为采样点依然存在。重要的是，这些带宽并没有得到有效利用——很多带宽被浪费了——这也就是为什么延迟管线不会采用 MSAA 的原因。

### TAA

接着看 TAA，Temporal Anti-Aliasing

> https://zhuanlan.zhihu.com/p/57503885

和 SSAA 类似的是，TAA 的每个像素点同样有多个"采样点"。但是不同与 SSAA 的是，TAA 会综合历史帧的数据来实现抗锯齿，这样会将每个像素点的多次采样成本均摊到多个帧中，相对的开销要小得多，同时，由于 TAA 是从前后帧来获取，将每个像素的多次采样分摊到多个帧中来实现的，其实际的执行效率相较于 MSAA 会高上不少。不过，既然是综合历史帧的算法，其效果必然会因为物体运动而收到影响。

> Sampling the pixels at a different position in each frame can be achieved by adding a per-frame "jitter" when rendering the frames. The "jitter" is a 2D offset that shifts the pixel grid, and its X and Y magnitude are between 0 and 1.
>
> When combining pixels sampled in past frames with pixels sampled in the current frame, care needs to be taken to avoid blending pixels that contain different objects, which would produce ghosting or motion-blurring artifacts. Different implementation of TAA have different ways of achieving this. Possible methods include:
>
> Using motion vectors from the game engine to perform motion compensation before blending.
> Limiting (clamping) the final value of a pixel by the values of pixels surrounding it.
>
> https://en.wikipedia.org/wiki/Temporal_anti-aliasing

![img](https://picx.zhimg.com/v2-7585c87dde80b1f19b908d8c74724cfc_720w.jpg?source=d16d100b)
一种可用的采样点序列实例：Halton 采样序列

TAA 的具体流程可参考：https://zhuanlan.zhihu.com/p/425233743

## Morphological Anti-Aliasing

除了增加采样点的数目，另一种常见的抗锯齿方案是通过后处理的方式来完成——例如 FXAA 和 SMAA。

它的思路也很简单：既然大多数锯齿都只出现在物体边缘或者高光变化的部分，那我们通过后处理的方式，检测出图像块之间的边缘，然后根据边缘信息对边缘两侧的图像进行混合处理，就可以达到抗锯齿的效果了。

比如下图中的图像，左边是待处理的图像，中间是找到的边界，右侧是将边界两侧像素混合后得到的抗锯齿效果。

![img](https://pic1.zhimg.com/v2-8f2778c2af4c2811a6017ea04e935ca4_720w.jpg?source=d16d100b)

### FXAA

FXAA，即 Fast approximate anti-aliasing：

> Fast approximate anti-aliasing (FXAA) is a screen-space anti-aliasing algorithm created by Timothy Lottes at Nvidia.
>
> FXAA 3 is released under a public domain license. A later version, FXAA 3.11, is released under a 3-clause BSD license.
>
> https://en.wikipedia.org/wiki/Fast_approximate_anti-aliasing

FXAA3.11 有两个版本：注重抗锯齿质量的 Quality 版本和注重抗锯齿速度的 Console 版本。

> 即使是 FXAA3.11，也已经是很有年头的东西了——这是出现于 2011 年，适用于 GTX4 系的技术。

FXAA 大致步骤如下：

1. 计算亮度与对比度：

   1. 首先是求亮度，这可以使用常用的求亮度公式 L = 0.213 _ R + 0.715 _ G + 0.072 \* B，也可以直接使用 G 分量的颜色值作为亮度值，因为绿色对整体亮度的贡献是最大的。当然，我们也可以直接从采样结果中获取——这需要我们提前将亮度保存在 alpha 通道中

   2. 采样亮度，计算对比度：

      采样的位置是下图所示的点，分别得到中间点 M 和周围四个点 N、E、W、S 的亮度值。

      ![img](https://pic4.zhimg.com/80/v2-f2feb0d89d8040408c8503362d0dd9e7_1440w.webp)

      可以直接用最大亮度和最小亮度的差作为对比度：

      ```c
      float MaxLuma = max(N, E, W, S, M);
      float MinLuma = min(N, E, W, S, M);
      float Contrast =  MaxLuma - MinLuma;
      if(Contrast >= max(_MinThreshold, MaxLuma * _Threshold)) {
      //    ...
      }
      ```

      如果得到的对比度值比较小，可以认为当前的点，不需要进行锯齿处理。

2. 基于亮度的混合系数计算

关于像素边缘采样：

### SMAA

## FSR

> https://zhuanlan.zhihu.com/p/532302889

## TSR

## Deep Learning Anti-Aliasing

### DLSS

## References

- https://zhuanlan.zhihu.com/p/57503957
