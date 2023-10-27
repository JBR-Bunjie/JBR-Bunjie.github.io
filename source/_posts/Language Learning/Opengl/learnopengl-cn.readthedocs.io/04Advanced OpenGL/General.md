---
title: 高级 OpenGL
date: 2023-3-8 10:33:08
tags:
  - Opengl
  - Shader
categories:
  - Opengl
  - Shader
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

# 高级 OpenGL

先简单回顾一下一般性的渲染管线：

![img](https://u3d-connect-cdn-public-prd.cdn.unity.cn/h1/20220622/p/images/5e689928-1cd2-4a02-9333-dc4870378dbf_image.png)

入门精要里的管线是这样的：

![概念流水线.png-16.9kB](http://static.zybuluo.com/candycat/c0opg7bab4cwzyok5vek52hs/%E6%A6%82%E5%BF%B5%E6%B5%81%E6%B0%B4%E7%BA%BF.png)
渲染流水线中的三个概念阶段

![GPU流水线.png-82.2kB](http://static.zybuluo.com/candycat/jundxsf604yuoy2zr3r1qkzp/GPU%E6%B5%81%E6%B0%B4%E7%BA%BF.png)
GPU 的渲染流水线实现。颜色表示了不同阶段的可配置性或可编程性：绿色表示该流水线阶段是完全可编程控制的，黄色表示该流水线阶段可以配置但不是可编程的，蓝色表示该流水线阶段是由 GPU 固定实现的，开发者没有任何控制权。实线表示该 shader 必须由开发者编程实现，虚线表示该 Shader 是可选的

## 深度测试

> 接下来三节都处于后片元阶段，我想先列举入门精要中的几张图来加强理解：
>
> ![Per-fragment Operations.png-23.1kB](http://static.zybuluo.com/candycat/epejev04t6vudwsyo2el8rp0/Per-fragment%20Operations.png)
> 逐片元操作阶段所做的操作。只有通过了所有的测试后，新生成的片元才能和颜色缓冲区中已经存在的像素颜色进行混合，最后再写入颜色缓冲区中
>
> ![Stencil Test_Depth Test.png-93.5kB](http://static.zybuluo.com/candycat/28t2ora2kenj1uudwfgfig95/Stencil%20Test_Depth%20Test.png)
> 模板测试和深度测试的简化流程图。
>
> ![Blending.png-67.6kB](http://static.zybuluo.com/candycat/k7q79qcgoqkw8myu1lpuy7he/Blending.png)
> 混合操作的简化流程图

### 使用

在[坐标系统](https://learnopengl-cn.github.io/01 Getting started/08 Coordinate Systems/)小节中，我们渲染了一个 3D 箱子，并且运用了深度缓冲(Depth Buffer，或者 Z Buffer)来防止被阻挡的面渲染到其它面的前面。

首先，从渲染管线上来看，深度测试几乎已经是整个流水线上的最后一步了——在模板测试之后，在 blend 操作之前。

此时，GPU 会将会把该片元的深度值和已经存在于深度缓冲区中的深度值进行比较。这个比较过程是接受配置的——我们来控制 OpenGL 什么时候该通过或丢弃一个片段，什么时候去更新深度缓冲。在 OpenGL 中，可以用：`glDepthFunc();` 来设置比较运算符

OpenGL 中，具体配置内容包括：

| 函数        | 描述                                         |
| :---------- | :------------------------------------------- |
| GL_ALWAYS   | 永远通过深度测试                             |
| GL_NEVER    | 永远不通过深度测试                           |
| GL_LESS     | 在片段深度值小于缓冲的深度值时通过测试       |
| GL_EQUAL    | 在片段深度值等于缓冲区的深度值时通过测试     |
| GL_LEQUAL   | 在片段深度值小于等于缓冲区的深度值时通过测试 |
| GL_GREATER  | 在片段深度值大于缓冲区的深度值时通过测试     |
| GL_NOTEQUAL | 在片段深度值不等于缓冲区的深度值时通过测试   |
| GL_GEQUAL   | 在片段深度值大于等于缓冲区的深度值时通过测试 |

在 Unity 中，这个配置项作为 Shaderlab 中的一个 Tag——`ZTest` 存在，可参考：[Unity - Manual: ShaderLab command: ZTest (unity3d.com)](https://docs.unity3d.com/Manual/SL-ZTest.html)，与之相关的还有[Unity - Manual: ShaderLab command: ZWrite (unity3d.com)](https://docs.unity3d.com/Manual/SL-ZWrite.html)

在 OpenGL 中，深度测试默认是禁用的，所以如果要启用深度测试的话，我们需要用 `GL_DEPTH_TEST` 选项来启用它：

```c++
glEnable(GL_DEPTH_TEST);
```

当它启用的时候，如果一个片段通过了深度测试的话，OpenGL 会在深度缓冲中储存该片段的 z 值；如果没有通过深度缓冲，则会丢弃该片段。如果你启用了深度缓冲，你还应该**在每个渲染迭代之前使用 GL_DEPTH_BUFFER_BIT 来清除深度缓冲，否则你会仍在使用上一次渲染迭代中的写入的深度值**：

```c++
glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
```

可以想象，在某些情况下你会需要对所有片段都执行深度测试并丢弃相应的片段，但不希望更新深度缓冲。基本上来说，你在使用一个只读的(Read-only)深度缓冲。OpenGL 允许我们禁用深度缓冲的写入，只需要设置它的**深度掩码**(Depth Mask)设置为`GL_FALSE`就可以了：

```c++
glDepthMask(GL_FALSE);
```

注意这只在深度测试被启用的时候才有效果。

### 运作

> 深度缓冲就像颜色缓冲(Color Buffer)（储存所有的片段颜色：视觉输出）一样，在每个片段中储存了信息，并且（通常）和颜色缓冲有着一样的宽度和高度。深度缓冲是由窗口系统自动创建的，它会以 16、24 或 32 位 float 的形式储存它的深度值。在大部分的系统中，深度缓冲的精度都是 24 位的。

> A _depth buffer_ stores depth information to control which areas of polygons are rendered rather than hidden from view. A _stencil buffer_ is used to mask pixels in an image, to produce special effects, including compositing; decaling; dissolves, fades, and swipes; outlines and silhouettes; and two-sided stencil.
>
> [Depth and stencil buffers - UWP applications | Microsoft Learn](https://learn.microsoft.com/en-us/windows/uwp/graphics-concepts/depth-and-stencil-buffers)

当深度测试(Depth Testing)被启用的时候，OpenGL 会将一个片段的深度值与深度缓冲的内容进行对比，如果通过，片段会来到 Blend 操作前，并且深度缓冲也会更新为新的深度值。如果深度测试失败了，片段将会被丢弃。

### Early Z

虽然一般意义上的深度缓冲的确是要等到在片段着色器及模板测试(Stencil Testing)运行完毕之后，才在屏幕空间中运行的。但这样子实在是太过浪费——这个 frag 已经完成了几乎全部的运算了，现在却要就这样丢弃。因此，我们希望尽可能早地知道哪些片元是需要丢弃的。正好，现在大部分的 GPU 都提供了称为 `Early Z` 或 `Early Depth Testing` 的硬件特性，这种技术能够在片元着色器之前就进行深度测试，以便我们尽可能地减少 GPU 的运算量。

简而言之，early-z 的解决方式非常简单：它直接修改了传统渲染管线，在光栅化和片元阶段中间，加入一个 early-z 阶段。这个阶段进行的操作和原本逐像素处理阶段的 z-test（为了与 early-z 区别，这个阶段也会被成为[late-z](https://www.zhihu.com/search?q=late-z&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A"389396050"})）操作完全一样，现代的 gpu 已经都开始包含这样的硬件设计。但是 early-z 有以下两个主要的缺点：

- 一旦进行了手动写入深度值、开启`alpha test`或者丢弃像素等操作，那么 gpu 就会关闭 early z 直到下次 clear z-buffer 后才会重新开启（不过现在的 gpu 也在逐渐优化，使其更智能开关 early z）。之所以 gpu 会选择关闭 early z 是因为上述那些操作可能会在片元阶段与 late-z 阶段之间修改深度缓存中的深度值，导致提前的 early-z 的结果并不正确。我们也可以在 fragment shader 中使用 layout(early_fragment_tests)来强制打开 early z。值得一提的是，至少在 Unity 中，因为 `alpha test` 所导致的 early z 关闭是 per object 的。[General questions regarding Early-Z - Unity Forum](https://forum.unity.com/threads/general-questions-regarding-early-z.1065779/)

- **early-z 的优化效果并不稳定**，最理想条件下所有绘制顺序都是由近及远，那么 early-z 可以完全避免过度绘制。但是相反的状态下，则会起不到任何效果。所以有些时候为了完全发挥 early-z 的功效，我们会在每帧绘制时对场景的物体按照到摄像机的距离由远及近进行排序。这个操作会在 cpu 端进行，当场景复杂到一定程度，频繁的排序将会占用 cpu 的大量计算资源。

片段着色器通常开销都是很大的，所以我们应该尽可能避免运行它们。当使用提前深度测试时，片段着色器的一个限制是你不能写入片段的深度值。如果一个片段着色器对它的深度值进行了写入，提前深度测试是不可能的。OpenGL 不能提前知道深度值。

### Depth Prepass/Z prepass

> > Regarding Z-Prepass, the idea seems to imply that it renders additional drawcalls (although extremely lightweight). Does Early-Z depend on it in order to work? Could it actually become a bottleneck in your pipeline due to the initial pre-pass costs?
>
> **The idea behind a Z prepass is to fill in the depth buffer with cheap draws so that you avoid over shading** (leveraging early z on the subsequent “full fat” draws). There was a frame breakdown of Cyberpunk 2077 that showed they used a Z prepass that only rendered a handful of visible meshes, presumably only those that are fully opaque and were considered big enough to block a lot. Other renderers, like Unity’s HDRP, ended up adding a Z prepass to make grass rendering more efficient, so they render _everything_ that uses ZWrite into the Z prepass. And yes, there’s no perfect solution. You can eventually spend more time rendering the Z prepass than the savings you get from having one.
>
> GPUs that use tile based deferred renderering, like Mali and Adreno, are basically doing hardware level Z prepass btw. They basically made the choice to always do a Z prepass for everything ... except they don’t because they can also swap to immediate mode rendering per tile if it doesn’t think it’ll be faster to do TBDR for that tile. Like if there’s only one or two triangles visible, or none of the triangles write to depth.

这个做法很简单：第一个 pass 仅写入深度，不做任何复杂的片元计算，不输出任何颜色。第二个 pass 关闭深度写入，并将深度比较函数设为“相等”。这样，无论场景中的物体的绘制顺序是怎样的，我们都可以凭借不大的代价，提前绘制好当前场景的深度缓存。这样，在第二个 pass 时，early-z 就可以用这个深度缓存中的值和当前深度值进行比较，只绘制深度相等的片元并直接丢弃任何其他的片元。并且由于当前的深度缓存已经是完全正确的结果了，第二个 pass 便可以关闭深度写入。

需要注意的是，z-perpass 必须配合 early-z 才能发挥效果，如果没有 early-z 的话，第二个 pass 的深度测试依旧在片元后，因此所有片元都会在片元阶段进行复杂计算。

> z-perpass 的思想和延迟渲染管线有些相似，差别在于：
>
> - 第一，z-perpass 的第一个 pass 只计算深度，并且结果直接存储在深度缓存。而延迟渲染会同时计算更多其他的屏幕空间数据，并将这些数据存储在额外的 framebuffer（GBuffer）中，这需要更大的缓存。
> - 第二，z-perpass 的第二个 pass 依旧需要对全场景的各个物体进行绘制（至少顶点阶段是如此），而延迟渲染的第二个 pass 类似于后处理本质上只绘制了一个屏幕大小的矩形。

另外还有几种改进方法，可参考：渲染杂谈：early-z、z-culling、hi-z、z-perpass 到底是什么？ - 修行哥的文章 - 知乎 https://zhuanlan.zhihu.com/p/389396050

### 深度值精度

简单来说，默认的深度值伴随投影变换产生，其直接关系有：

$$
Depth = 0.5Z_{ndc}+0.5;\\
Z_{ndc} = Z_{clip}/W_{clip}\\
Position_{clip} = M_{proj} * Position_{view}
$$

经过这样的变换，原本线性的 Z 分量就变成了非线性的，一般来说，Depth 最终的取值函数类似于这样：

$$
Depth = \frac{\frac{1}{n} + \frac{1}{z}}{\frac{1}{n}-\frac{1}{f}}
$$

Learning OpenGL 中将它与直觉上的线性的深度值做了对比：

> ... 它做的就是在 z 值很小的时候提供非常高的精度，而在 z 值很远的时候提供更少的精度。花时间想想这个：我们真的需要对 1000 单位远的深度值和只有 1 单位远的充满细节的物体使用相同的精度吗？线性方程并不会考虑这一点。

> 由于非线性方程与 1/z 成正比，在 1.0 和 2.0 之间的 z 值将会变换至 1.0 到 0.5 之间的深度值，这就是一个 float 提供给我们的一半精度了，这在 z 值很小的情况下提供了非常大的精度。在 50.0 和 100.0 之间的 z 值将会只占 2%的 float 精度，这正是我们所需要的。
>
> 如果你不知道这个方程是怎么回事也不用担心。重要的是要记住深度缓冲中的值在屏幕空间中不是线性的（在透视矩阵应用之前在观察空间中是线性的）。深度缓冲中 0.5 的值并不代表着物体的 z 值是位于平截头体的中间了，这个顶点的 z 值实际上非常接近近平面！你可以在下图中看到 z 值和最终的深度缓冲值之间的非线性关系：
>
> ![img](https://learnopengl-cn.github.io/img/04/01/depth_non_linear_graph.png)
>
> 可以看到，深度值很大一部分是由很小的 z 值所决定的，这给了近处的物体很大的深度精度。这个（从观察者的视角）变换 z 值的方程是嵌入在投影矩阵中的，所以当我们想将一个顶点坐标从观察空间至裁剪空间的时候这个非线性方程就被应用了。如果你想深度了解投影矩阵究竟做了什么，我建议阅读[这篇文章](http://www.songho.ca/opengl/gl_projectionmatrix.html)。

可是，这样子的 Buffer 值并不总是符合我们需求的，在开阔的场景中，这可能会导致 Z-Fighting(即下方的深度冲突)的出现：两个物体在 Buffer 中的深度值因为精度问题导致具体的值一致了，这让两个物体的渲染效果不断闪烁

这时候，我们可能会去做一个叫 Reverse-Z 的操作，简单来说，就是反转原始 Proj 矩阵中的 M33 与 M34 位中分母 Near 与 Far 的位置。若一个原始 Proj 如下：

$$
\begin{bmatrix}
 \frac{\cot{\frac{FOV}{2}}}{Aspect} & 0 & 0 & 0\\
 0 & \frac{\cot{FOV}}{2} & 0 & 0\\
 0 & 0 & -\frac{Far + Near}{Far - Near} & -\frac{2*Near*Far}{Far - Near}\\
 0 & 0 & -1 & 0
\end{bmatrix}
$$

则修改后的 Proj 如下：

$$
\begin{bmatrix}
 \frac{\cot{\frac{FOV}{2}}}{Aspect} & 0 & 0 & 0\\
 0 & \frac{\cot{FOV}}{2} & 0 & 0\\
 0 & 0 & -\frac{Far + Near}{Near - Far} & -\frac{2*Near*Far}{Near - Far}\\
 0 & 0 & -1 & 0
\end{bmatrix}
$$

此时的方程为：

$$
Depth = \frac{\frac{1}{f} + \frac{1}{z}}{\frac{1}{f}-\frac{1}{n}}
$$

具体的原理可参考：

- [Reversed-Z 详解 - jackmaxwell - 博客园 (cnblogs.com)](https://www.cnblogs.com/jackmaxwell/p/6851728.html)
- [程序媛转 TA 之面试篇二：z-fighting，以及 z 精度的最佳分辨率函数 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/651462948)

### 深度冲突

**一个很常见的视觉错误会在两个平面或者三角形非常紧密地平行排列在一起时会发生，深度缓冲没有足够的精度来决定两个形状哪个在前面。结果就是这两个形状不断地在切换前后顺序，这会导致很奇怪的花纹。这个现象叫做深度冲突(Z-fighting)，因为它看起来像是这两个形状在争夺(Fight)谁该处于顶端。**

> 在我们一直使用的场景中，有几个地方的深度冲突还是非常明显的。箱子被放置在地板的同一高度上，这也就意味着箱子的底面和地板是共面的(Coplanar)。这两个面的深度值都是一样的，所以深度测试没有办法决定应该显示哪一个。
>
> 如果你将摄像机移动到其中一个箱子的内部，你就能清楚地看到这个效果的，箱子的底部不断地在箱子底面与地板之间切换，形成一个锯齿的花纹：
>
> ![img](https://learnopengl-cn.github.io/img/04/01/depth_testing_z_fighting.png)
>
> 深度冲突是深度缓冲的一个常见问题，当物体在远处时效果会更明显（因为深度缓冲在 z 值比较大的时候有着更小的精度)。深度冲突不能够被完全避免，但一般会**有一些技巧有助于在你的场景中减轻或者完全避免深度冲突**：
>
> - 第一个也是最重要的技巧是**永远不要把多个物体摆得太靠近，以至于它们的一些三角形会重叠**。通过在两个物体之间设置一个用户无法注意到的偏移值，你可以完全避免这两个物体之间的深度冲突。在箱子和地板的例子中，我们可以将箱子沿着正 y 轴稍微移动一点。箱子位置的这点微小改变将不太可能被注意到，但它能够完全减少深度冲突的发生。然而，这需要对每个物体都手动调整，并且需要进行彻底的测试来保证场景中没有物体会产生深度冲突。
> - 第二个技巧是**尽可能将近平面设置远一些**。在前面我们提到了精度在靠近**近**平面时是非常高的，所以如果我们将**近**平面远离观察者，我们将会对整个平截头体有着更大的精度。然而，将近平面设置太远将会导致近处的物体被裁剪掉，所以这通常需要实验和微调来决定最适合你的场景的**近**平面距离。另外，这个在不管反未反转 Depth 都有效，我们总有 Near 值越大，函数曲线越趋于平滑，产生的精度越高；不过，当我们反转之后，各个数值段能获取到的精度已经经过一次平衡了，也就没必要过分纠结 Near 平面值的设置了。
> - 另外一个很好的技巧是牺牲一些性能，**使用更高精度的深度缓冲**。大部分深度缓冲的精度都是 24 位的，但现在大部分的显卡都支持 32 位的深度缓冲，这将会极大地提高精度。所以，牺牲掉一些性能，你就能获得更高精度的深度测试，减少深度冲突。
>
> 我们上面讨论的三个技术是最普遍也是很容易实现的抗深度冲突技术了。还有一些更复杂的技术，但它们**依然不能完全消除深度冲突**。深度冲突是一个常见的问题，但如果你组合使用了上面列举出来的技术，你可能不会再需要处理深度冲突了。

## 模板测试

> 当片段着色器处理完一个片段之后，**模板测试(Stencil Test)**会开始执行，被保留下来的片段才会进入深度测试。和深度测试一样，模版测试也可能会丢弃片段。模板测试是根据又一个缓冲来进行的，它叫做模板缓冲(Stencil Buffer)，我们可以在渲染的时候更新它来获得一些很有意思的效果。

> 一个模板缓冲中，（通常）每个模板值(Stencil Value)是 8 位的。所以每个像素/片段一共能有 256 种不同的模板值。我们可以将这些模板值设置为我们想要的值，然后当某一个片段有某一个模板值的时候，我们就可以选择丢弃或是保留这个片段了。

模板缓冲的一个简单的例子如下：

![img](https://learnopengl-cn.github.io/img/04/02/stencil_buffer.png)

模板缓冲首先会被清除为 0，之后在模板缓冲中使用 1 填充了一个空心矩形。场景中的片段将会只在片段的模板值为 1 的时候会被渲染（其它的都被丢弃了）。

模板缓冲操作允许我们在渲染片段时将模板缓冲设定为一个特定的值。我们有两个函数用来控制模板缓冲：`glStencilFunc` 和 `glStencilOp`。

### 使用模板测试

首先是 `glStencilFunc`，它会决定当前的 Frag 是否通过模板测试。

`glStencilFunc` 一共包含三个参数，最终构成 `glStencilFunc(GLenum func, GLint ref, GLuint mask)`：

- `func`：设置模板测试函数(Stencil Test Function)。这个测试函数将会应用到已储存的模板值上和 `glStencilFunc` 函数的`ref`值上。可用的选项有：GL_NEVER、GL_LESS、GL_LEQUAL、GL_GREATER、GL_GEQUAL、GL_EQUAL、GL_NOTEQUAL 和 GL_ALWAYS。它们的语义和深度缓冲的函数类似。
- `ref`：设置了模板测试的参考值(Reference Value)。模板缓冲的内容将会与这个值进行比较。
- `mask`：设置一个掩码，它将会与参考值和储存的模板值在测试比较它们之前进行与(AND)运算。初始情况下所有位都为 1。

例如，对于上面的例子，我们希望仅输出 stencil value = 1 的 Frag，那么我们可以这么设置：

```c++
glStencilFunc(GL_EQUAL, 1, 0xFF)
```

但是，`glStencilFunc` 仅仅描述了**OpenGL 应该对模板缓冲内容做什么**，而不是我们应该**如何更新缓冲**。这就需要 `glStencilOp` 这个函数了。

`glStencilOp(GLenum sfail, GLenum dpfail, GLenum dppass)` 一共包含三个选项，我们能够设定每个选项应该采取的行为：

- `sfail`：模板测试失败时采取的行为。
- `dpfail`：模板测试通过，但深度测试失败时采取的行为。
- `dppass`：模板测试和深度测试都通过时采取的行为。

每个选项都可以选用以下的其中一种行为：

| 行为           | 描述                                                   |
| :------------- | :----------------------------------------------------- |
| GL_KEEP        | 保持当前储存的模板值                                   |
| GL_ZERO        | 将模板值设置为 0                                       |
| **GL_REPLACE** | **将模板值设置为 glStencilFunc 函数设置的`ref`值**     |
| GL_INCR        | 如果模板值小于最大值则将模板值加 1                     |
| GL_INCR_WRAP   | 与 GL_INCR 一样，但如果模板值超过了最大值则归零        |
| GL_DECR        | 如果模板值大于最小值则将模板值减 1                     |
| GL_DECR_WRAP   | 与 GL_DECR 一样，但如果模板值小于 0 则将其设置为最大值 |
| GL_INVERT      | 按位翻转当前的模板缓冲值                               |

默认情况下 `glStencilOp` 是设置为`(GL_KEEP, GL_KEEP, GL_KEEP)`的，所以不论任何测试的结果是如何，模板缓冲都会保留它的值。默认的行为不会更新模板缓冲，所以如果你想写入模板缓冲的话，你需要至少对其中一个选项设置不同的值。

在有了上面两个函数：`glStencilFunc` 和 `glStencilOp` 之后，我们就可以精确地指定更新模板缓冲的时机与行为了，我们也可以指定什么时候该让模板缓冲通过，即什么时候片段需要被丢弃。

现在，我们可以尝试启用 GL_STENCIL_TEST 来启用模板测试了。在这一行代码之后，所有的渲染调用都会以某种方式影响着模板缓冲。

```c++
glEnable(GL_STENCIL_TEST);
```

需要注意的是，和颜色和深度缓冲一样，你也需要在每次迭代之前清除模板缓冲。所以，我们现在的 glClear 应该长这样：

```c++
glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT | GL_STENCIL_BUFFER_BIT);
```

和深度测试的 glDepthMask 函数一样，模板缓冲也有一个类似的函数。glStencilMask 允许我们设置一个位掩码(Bitmask)，它会与将要写入缓冲的模板值进行与(AND)运算。默认情况下设置的位掩码所有位都为 1，不影响输出，但如果我们将它设置为`0x00`，写入缓冲的所有模板值最后都会变成 0.这与深度测试中的 glDepthMask(GL_FALSE)是等价的。

```c++
glStencilMask(0xFF); // 每一位写入模板缓冲时都保持原样
glStencilMask(0x00); // 每一位在写入模板缓冲时都会变成0（禁用写入）
```

大部分情况下你都只会使用`0x00`或者`0xFF`作为模板掩码(Stencil Mask)，但是知道有选项可以设置自定义的位掩码总是好的。

### 物体轮廓

让我们以一个使用模板测试完成的例子来讲解模板测试可以用来干什么，这个例子是物体轮廓(Object Outlining)。

![img](https://learnopengl-cn.github.io/img/04/02/stencil_object_outlining.png)

物体轮廓所能做的事情正如它名字所描述的那样。我们将会为每个（或者一个）物体在它的周围创建一个很小的有色边框。当你想要在策略游戏中选中一个单位进行操作的，想要告诉玩家选中的是哪个单位的时候，这个效果就非常有用了。为物体创建轮廓的步骤如下：

1. 在绘制（需要添加轮廓的）物体之前，将模板函数设置为 GL_ALWAYS，每当物体的片段被渲染时，将模板缓冲更新为 1。
2. 渲染物体。
3. 禁用模板写入以及深度测试。
4. 将每个物体缩放一点点。
5. 使用一个不同的片段着色器，输出一个单独的（边框）颜色。
6. 再次绘制物体，但只在它们片段的模板值不等于 1 时才绘制。
7. 再次启用模板写入和深度测试。

这个过程将每个物体的片段的模板缓冲设置为 1，当我们想要绘制边框的时候，我们主要绘制放大版本的物体中模板测试通过的部分，也就是物体的边框的位置。我们主要使用模板缓冲丢弃了放大版本中属于原物体片段的部分。

```c++
        glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT | GL_STENCIL_BUFFER_BIT);

        // 前提：我们已经设置好了glStencilOp为(KEEP, KEEP, REPLACE)，并且这之后都没有变过。其实这么设计值并不算正确，或者说，这样子实现的就不是物体的绝对轮廓，而是物体在可视结果下的全部轮廓
        // 此时，经过glClear模板缓冲内全部为0

        // Plane的渲染，模板无关，先渲染，并且使用遮罩防止它修改模板缓冲
        glStencilFunc(GL_ALWAYS, 0, 0xFF);
        glStencilMask(0x00);
        glBindVertexArray(planeVAO);
        glBindTexture(GL_TEXTURE_2D, floorTexture);
        shader.setMat4("model", model);
        glDrawArrays(GL_TRIANGLES, 0, 6);
        glBindVertexArray(0);

        // 此时模板缓冲应该仍然全为0，开始修改Func与Mask，现在，在Cube对应位置的模板缓冲应该为1
        glStencilFunc(GL_ALWAYS, 1, 0xFF);
        glStencilMask(0xFF);
        glBindVertexArray(cubeVAO);
        glBindTexture(GL_TEXTURE_2D, cubeTexture);
        model = glm::mat4(1.0f);
        model = glm::translate(model, glm::vec3(-1.0f, 0.0f, -1.0f));
        shader.setMat4("model", model);
        glDrawArrays(GL_TRIANGLES, 0, 36);
        model = glm::mat4(1.0f);
        model = glm::translate(model, glm::vec3(2.0f, 0.0f, 0.0f));
        shader.setMat4("model", model);
        glDrawArrays(GL_TRIANGLES, 0, 36);

        // 我们将Func修改为(GL_NOTEQUAL, 1, 0xFF)，并且使用的Mask为0x00防止写入(应该没必要,So I comment it out)
        // 只有模板缓冲中值不为1的部分才能通过，并且不会影响已有的模板缓冲，不然会被REPLACE(根据没有修改过的Op)
        glStencilFunc(GL_NOTEQUAL, 1, 0xFF);
        // glStencilMask(0x00);
        glDisable(GL_DEPTH_TEST); //这里关闭了深度缓冲是另一个有意思的点，这样做是因为我们不想让立方体的边框被地板的边框遮挡
        shaderSingleColor.use();
        float scale = 1.1f;
        glBindVertexArray(cubeVAO);
        glBindTexture(GL_TEXTURE_2D, cubeTexture);
        model = glm::mat4(1.0f);
        model = glm::translate(model, glm::vec3(-1.0f, 0.0f, -1.0f));
        model = glm::scale(model, glm::vec3(scale, scale, scale));
        shaderSingleColor.setMat4("model", model);
        glDrawArrays(GL_TRIANGLES, 0, 36);
        model = glm::mat4(1.0f);
        model = glm::translate(model, glm::vec3(2.0f, 0.0f, 0.0f));
        model = glm::scale(model, glm::vec3(scale, scale, scale));
        shaderSingleColor.setMat4("model", model);
        glDrawArrays(GL_TRIANGLES, 0, 36);
        glBindVertexArray(0);

        // Re-Init
        glStencilMask(0xFF);
        glStencilFunc(GL_ALWAYS, 0, 0xFF);
        glEnable(GL_DEPTH_TEST);
```

这个轮廓算法的结果看起来会像是这样的：

![img](https://learnopengl-cn.github.io/img/04/02/stencil_scene_outlined.png)

## 混合

> OpenGL 中，混合(Blending)通常是实现物体透明度(Transparency)的一种技术。透明就是说一个物体（或者其中的一部分）不是纯色(Solid Color)的，它的颜色是物体本身的颜色和它背后其它物体的颜色的不同强度结合。一个有色玻璃窗是一个透明的物体，玻璃有它自己的颜色，但它最终的颜色还包含了玻璃之后所有物体的颜色。这也是混合这一名字的出处，我们混合(Blend)（不同物体的）多种颜色为一种颜色。所以透明度能让我们看穿物体。

事实上，LearnOpenGL 在这一节既介绍了 Blend，又介绍了 Alpha Test 两个东西。我们可以对这两种做法做一个快速的区分：Blend 是利用 Alpha，基于一定的运算方法去混合两种颜色(一般是当前的 FragColor 和 Color Buffer 混合才被视为 Blend)，而 Alpha Test 则是判断对应 Frag 的 Alpha 是否符合要求，不符合直接 Discard;

### Alpha Test

这里主要的操作就集中在 Discard 指令上，Unity 也有这个指令，但是我们一般采用另一个：`clip()`，但实际上，`Clip()` 就是 `Discard` 的调用，因为：

```shaderlab
if (x < 0) discard; // == clip(x)
```

在 OpenGL 中也没有什么不一样的，根据采样贴图得到的 Alpha，设定阈值与对应逻辑即可。

不过在这一小节中，有这么一小段是值得注意的：

> 注意，当采样纹理的边缘的时候，OpenGL 会对边缘的值和纹理下一个重复的值进行插值（因为我们将它的环绕方式设置为了 GL_REPEAT。这通常是没问题的，但是由于我们使用了透明值，纹理图像的顶部将会与底部边缘的纯色值进行插值。这样的结果是一个半透明的有色边框，你可能会看见它环绕着你的纹理四边形。要想避免这个，每当你 alpha 纹理的时候，请将纹理的环绕方式设置为 GL_CLAMP_TO_EDGE：
>
> ```C++
> glTexParameteri( GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE);
> glTexParameteri( GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE);
> ```

### Blend 操作

> [Unity - Manual: ShaderLab command: Blend (unity3d.com)](https://docs.unity3d.com/Manual/SL-Blend.html)

相较于 Alpha Test，Blend 就要复杂的多，但还是让我们一步一步来。

和 OpenGL 大多数的功能一样，我们可以启用 GL_BLEND 来启用混合：

```c++
glEnable(GL_BLEND);
```

启用了混合之后，我们需要告诉 OpenGL 它该**如何**混合。

OpenGL 中的混合是通过下面这个方程来实现的：

$$
\vec{C_{result}}=\vec{C_{source}} * \vec{F_{source}} \space (One \space Calculation \space Method) \space \vec{C_{destination}}*\vec{F_{destination}}
$$

- C_source：源颜色向量。这是源自纹理的颜色向量。
- C_destination：目标颜色向量。这是当前储存在颜色缓冲中的颜色向量。
- F_source：源因子值。指定了 alpha 值对源颜色的影响。
- F_destination：目标因子值。指定了 alpha 值对目标颜色的影响。

**片段着色器运行完成后，并且所有的测试都通过之后，这个混合方程(Blend Equation)才会应用到片段颜色输出与当前颜色缓冲中的值（当前片段之前储存的之前片段的颜色）上。**源颜色和目标颜色将会由 OpenGL 自动设定，但源因子和目标因子的值，即 F_source 和 F_destination 则可以由我们来决定——和之前一样，通过一个 OpenGL 函数，这次是 `glBlendFunc`

`glBlendFunc(GLenum sfactor, GLenum dfactor)` 函数接受两个参数，来设置源和目标因子。OpenGL 为我们定义了很多个选项，下面列出了大部分最常用的选项。注意常数颜色向量 C_constant 可以通过 `glBlendColor` 函数来另外设置。

| 选项                          | 值                                           |
| :---------------------------- | :------------------------------------------- |
| `GL_ZERO`                     | 因子等于 0                                   |
| `GL_ONE`                      | 因子等于 1                                   |
| `GL_SRC_COLOR`                | 因子等于源颜色向量 C_source                  |
| `GL_ONE_MINUS_SRC_COLOR`      | 因子等于 1 - C_source                        |
| `GL_DST_COLOR`                | 因子等于目标颜色向量 C_destination           |
| `GL_ONE_MINUS_DST_COLOR`      | 因子等于 1 - C_destination                   |
| **`GL_SRC_ALPHA`**            | **因子等于 C_source 的 alpha 分量**          |
| **`GL_ONE_MINUS_SRC_ALPHA`**  | **因子等于 1 - C_source 的 alpha 分量**      |
| **`GL_DST_ALPHA`**            | **因子等于 C_destination 的 alpha 分量**     |
| **`GL_ONE_MINUS_DST_ALPHA`**  | **因子等于 1 - C_destination 的 alpha 分量** |
| `GL_CONSTANT_COLOR`           | 因子等于常数颜色向量 C_constant              |
| `GL_ONE_MINUS_CONSTANT_COLOR` | 因子等于 1 - C_constant                      |
| `GL_CONSTANT_ALPHA`           | 因子等于 C_constant 的 alpha 分量            |
| `GL_ONE_MINUS_CONSTANT_ALPHA` | 因子等于 1 - C_constant 的 alpha 分量        |

除了使用 `glBlendFunc` 来设置统一的混合因子外，我们还可以使用 `glBlendFuncSeparate` 为 RGB 和 alpha 通道**分别设置不同的选项**：

```c++
glBlendFuncSeparate(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA, GL_ONE, GL_ZERO);
```

这个函数和我们之前设置的那样设置了 RGB 分量，但这样只能让最终的 alpha 分量被源颜色向量的 alpha 值所影响到，可翻阅：[glBlendFuncSeparate - OpenGL 4 Reference Pages (khronos.org)](https://registry.khronos.org/OpenGL-Refpages/gl4/html/glBlendFuncSeparate.xhtml)

方程中中央的运算符，也是可以指定的：`glBlendEquation(GLenum mode)` 允许我们设置运算符，它提供了三个选项：

- GL_FUNC_ADD：默认选项，将两个分量相加：C_result=Src + Dst。
- GL_FUNC_SUBTRACT：将两个分量相减： C_result=Src - Dst
- GL_FUNC_REVERSE_SUBTRACT：将两个分量相减，但顺序相反：C_result=Dst - Src。

通常我们都可以省略调用 `glBlendEquation`，因为 GL_FUNC_ADD 对大部分的操作来说都是我们希望的混合方程。

当我们开始渲染透明物体时，我们就需要因地制宜地考虑是否采用深度测试以及指定渲染顺序了，Unity 用一个"Queue"的 Tag 来解决了一部分的渲染顺序的问题，至少能让透明物体都在同一个阶段去渲染。而对于深度的问题，问题就开始复杂了起来：

- 如果是多个透明物体间的互相遮挡，那么就必须严格控制渲染顺序，就算是两个透明物体 AB，先渲染谁后渲染谁都会造成完全不一样的结果。
- 如果是单个透明物体，但是我们不希望出现其内部结构的话，入门精要里的有一个巧妙的解决办法：我们在 Shader 中定义两个 Pass，第一个 Pass 只写入深度，第二个 Pass 用来渲染物体，这样就能避免模型内部被暴露。特别的，如果一个模型中存在循环重叠的问题，我们就得考虑拆分模型/分隔网格了。

![render_order_3.png-15.3kB](http://static.zybuluo.com/candycat/sl85989h54upaju75zstyv0y/render_order_3.png)

## 面剔除

> 尝试在脑子中想象一个 3D 立方体，数数你从任意方向最多能同时看到几个面。如果你的想象力不是过于丰富了，你应该能得出最大的面数是 3。你可以从任意位置和任意方向看向这个球体，但你永远不能看到 3 个以上的面。所以我们为什么要浪费时间绘制我们不能看见的那 3 个面呢？如果我们能够以某种方式丢弃这几个看不见的面，我们能省下超过 50%的片段着色器执行数！

现代的图形渲染引擎中，一般都会提供面剔除功能。它发生在光栅化阶段中，体现为：只渲染正面朝向摄像机的面片，而背朝摄像机的则被剔除。当然，这种剔除的方向也是可以指定的：我们可以通过启用 GL_CULL_FACE 来启用面剔除功能

```c++
glEnable(GL_CULL_FACE);
```

启用之后，可以指定提出方向，可以是正向剔除或者背部剔除：

```c++
glCullFace(GL_BACK);
// or:
glCullFace(GL_FRONT);
```

### 面剔除的实现原理

实际上，这种面剔除直接来自于我们为 OpenGL 提供的数据——是我们直接告诉了 OpenGL 哪些面是正向面(Front Face)，哪些面是背向面(Back Face)的——这其中有一个很聪明的技巧，就是顶点数据的环绕顺序(Winding Order)：

![img](https://learnopengl-cn.github.io/img/04/04/faceculling_windingorder.png)

我们已经知道，OpenGL 是右手系。当我们定义一个三角面片时，如果在当前视角下，这个面片的旋转方向是 Counter-clockwise 的，则会被认为是正面的面片，反之如是 Clockwise 的，则被认为是反面的面片。不过，我们也可以人为地去修改这样一个正反向的认定标准：

```c++
glFrontFace(GL_CCW); 	// counter-clockwise为正向
glFrontFace(GL_CW);		// clockwise为正向
```

> 当你定义顶点顺序的时候，你应该想象对应的三角形是面向你的，所以你定义的三角形从正面看去应该是逆时针的。这样定义顶点很棒的一点是，实际的环绕顺序是在光栅化阶段进行的，也就是顶点着色器运行之后。这些顶点就是从**观察者视角**所见的了。

这是因为一个模型间存在各三角形面片间独立的顶点关系，这种关系体现在环绕顺序上虽然是相对的，但是却是稳定的——它不会随着 Transform 的进行而发生变化。这样的话，我们在模型上预定义的环绕关系就可以在面剔除的阶段发挥巨大作用。

## 帧缓冲

> 到目前为止，我们已经使用了很多屏幕缓冲了：用于写入颜色值的**颜色缓冲**、用于写入深度信息的**深度缓冲**和允许我们根据一些条件丢弃特定片段的**模板缓冲**。**这些缓冲结合起来叫做帧缓冲(Framebuffer)**，它被储存在内存(这里应该是指显存，原文：The combination of these buffers is stored somewhere in GPU memory and is called a framebuffer.)中。OpenGL 允许我们定义我们自己的帧缓冲，也就是说我们能够定义我们自己的颜色缓冲，甚至是深度缓冲和模板缓冲。
>
> **我们目前所做的所有操作都是在默认帧缓冲的渲染缓冲上进行的。默认的帧缓冲是在你创建窗口的时候生成和配置的（GLFW 帮我们做了这些）。**有了我们自己的帧缓冲，我们就能够有更多方式来渲染了。
>
> 你可能不能很快理解帧缓冲的应用，但渲染你的场景到不同的帧缓冲能够让我们在场景中加入类似镜子的东西，或者做出很酷的后期处理效果。首先我们会讨论它是如何工作的，之后我们将来实现这些炫酷的后期处理效果。

### 自定义帧缓冲

首先是创建，不过帧缓冲的创建和别的 Buffer 的创建也没什么区别：我们会使用一个叫做 `glGenFramebuffers` 的函数来创建一个帧缓冲对象(Framebuffer Object, FBO)：

```c++
unsigned int fbo;
glGenFramebuffers(1, &fbo);
```

然后就是老一套的绑定激活，做一些操作，之后解绑帧缓冲。我们先用 `glBindFramebuffer` 来绑定帧缓冲。

```c++
glBindFramebuffer(GL_FRAMEBUFFER, fbo);
```

在绑定到 GL_FRAMEBUFFER 目标之后，所有的**读取**和**写入**帧缓冲的操作将会影响当前绑定的帧缓冲。

> 我们也可以使用 GL_READ_FRAMEBUFFER 或 GL_DRAW_FRAMEBUFFER，将一个帧缓冲分别绑定到读取目标或写入目标。绑定到 GL_READ_FRAMEBUFFER 的帧缓冲将会使用在所有像是 `glReadPixels` 的读取操作中，而绑定到 GL_DRAW_FRAMEBUFFER 的帧缓冲将会被用作渲染、清除等写入操作的目标。大部分情况你都不需要区分它们，通常都会使用 GL_FRAMEBUFFER，绑定到两个上。

不过，在正式使用帧缓冲之前，我们还有一些事情要做——这个帧缓冲还不完整，而一个完整的帧缓冲需要满足以下的组件：

- 附加至少一个缓冲（颜色、深度或模板缓冲）。
- 至少有一个颜色附件(Attachment)。
- 所有的附件都必须是完整的（保留了内存）。
- 每个缓冲都应该有相同的样本数。(如果你不知道什么是样本，不要担心，我们将在[之后的](https://learnopengl-cn.github.io/04 Advanced OpenGL/11 Anti Aliasing/)教程中讲到)

### 创建附件

#### 纹理附件

> 附件是一个内存位置，它能够作为帧缓冲的一个缓冲，可以将它想象为一个图像。当创建一个附件的时候我们有两个选项：纹理或渲染缓冲对象(Renderbuffer Object)。

> 当把一个纹理附加到帧缓冲的时候，所有的渲染指令将会写入到这个纹理中，就像它是一个普通的颜色/深度或模板缓冲一样。使用纹理的优点是，所有渲染操作的结果将会被储存在一个纹理图像中，我们之后可以在着色器中很方便地使用它。

为帧缓冲创建一个纹理和创建一个普通的纹理差不多：

```c++
unsigned int texture;
glGenTextures(1, &texture);
glBindTexture(GL_TEXTURE_2D, texture);

glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, 800, 600, 0, GL_RGB, GL_UNSIGNED_BYTE, NULL);

glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
```

主要的区别就是，我们将维度设置为了屏幕大小（尽管这不是必须的），并且我们给纹理的`data`参数传递了`NULL`。**对于这个纹理，我们仅仅分配了内存而没有填充它。**填充这个纹理将会在我们渲染到帧缓冲之后来进行。同样注意我们并不关心环绕方式或多级渐远纹理，我们在大多数情况下都不会需要它们。

如果你想将你的屏幕渲染到一个更小或更大的纹理上，你需要（在渲染到你的帧缓冲之前）再次调用 glViewport，使用纹理的新维度作为参数，否则只有一小部分的纹理或屏幕会被渲染到这个纹理上。

现在。将我们创建好的空纹理附加到帧缓冲上：

```c++
glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, GL_TEXTURE_2D, texture, 0);
```

`glFrameBufferTexture2D` 有以下的参数：

- `target`：帧缓冲的目标（绘制、读取或者两者皆有）
- `attachment`：我们想要附加的**附件类型**。当前我们正在附加一个颜色附件。**注意最后的`0`意味着我们可以附加多个颜色附件。**我们将在之后的教程中提到。
- `textarget`：你希望附加的纹理类型
- `texture`：要附加的纹理本身
- `level`：多级渐远纹理的级别。我们将它保留为 0。

当然，除了颜色外，我们还可以附加更多如深度、模板缓冲等的纹理到帧缓冲对象中：

- 附加深度缓冲时，我们将附件类型设置为 GL_DEPTH_ATTACHMENT。注意纹理的格式(Format)和内部格式(Internalformat)类型将变为 GL_DEPTH_COMPONENT，来反映深度缓冲的储存格式。
- 要附加模板缓冲的话，你要将第二个参数设置为 GL_STENCIL_ATTACHMENT，并将纹理的格式设定为 GL_STENCIL_INDEX。
- 特别的是，我们也可以将深度缓冲和模板缓冲一起，附加为一个纹理。纹理的每 32 位数值中，将包含 24 位的深度信息和 8 位的模板信息。要将深度和模板缓冲附加为一个纹理的话，我们使用 GL_DEPTH_STENCIL_ATTACHMENT 类型，并配置纹理的格式，让它包含合并的深度和模板值：

```c++
glTexImage2D(GL_TEXTURE_2D, 0, GL_DEPTH24_STENCIL8, 800, 600, 0, GL_DEPTH_STENCIL, GL_UNSIGNED_INT_24_8, NULL);

glFramebufferTexture2D(GL_FRAMEBUFFER, GL_DEPTH_STENCIL_ATTACHMENT, GL_TEXTURE_2D, texture, 0);
```

#### 渲染缓冲对象附件

> **渲染缓冲对象(Renderbuffer Object)是在纹理之后引入到 OpenGL 中，作为一个可用的帧缓冲附件类型的，所以在过去纹理是唯一可用的附件。**和纹理图像一样，渲染缓冲对象是一个真正的缓冲，即一系列的字节、整数、像素等。渲染缓冲对象附加的好处是，它会将数据储存为 OpenGL 原生的渲染格式，它是为离屏渲染到帧缓冲优化过的。
>
> 渲染缓冲对象直接将所有的渲染数据储存到它的缓冲中，不会做任何针对纹理格式的转换，让它变为一个更快的可写储存介质。然而，渲染缓冲对象通常都是只写的，所以你不能读取它们（比如使用纹理访问）。当然你仍然还是能够使用 glReadPixels 来读取它，这会从当前绑定的帧缓冲，而不是附件本身，中返回特定区域的像素。
>
> 因为它的数据已经是原生的格式了，当写入或者复制它的数据到其它缓冲中时是非常快的。所以，交换缓冲这样的操作在使用渲染缓冲对象时会非常快。我们在每个渲染迭代最后使用的 `glfwSwapBuffers`，也可以通过渲染缓冲对象实现：只需要写入一个渲染缓冲图像，并在最后交换到另外一个渲染缓冲就可以了。渲染缓冲对象对这种操作非常完美。

创建一个渲染缓冲对象的代码和帧缓冲的代码很类似：

```c++
unsigned int rbo;
glGenRenderbuffers(1, &rbo);
```

类似，我们需要绑定这个渲染缓冲对象，让之后所有的渲染缓冲操作影响当前的 rbo：

```c++
glBindRenderbuffer(GL_RENDERBUFFER, rbo);
```

由于渲染缓冲对象通常都是只写的，它们会经常用于深度和模板附件，因为大部分时间我们都不需要从深度和模板缓冲中读取值，只关心深度和模板测试。我们**需要**深度和模板值用于测试，但不需要对它们进行**采样**，所以渲染缓冲对象非常适合它们。当我们不需要从这些缓冲中采样的时候，通常都会选择渲染缓冲对象，因为它会更优化一点。

调用 `glRenderbufferStorage` 函数来具体设置这个 rbo 的：

```c++
glRenderbufferStorage(GL_RENDERBUFFER, GL_DEPTH24_STENCIL8, 800, 600);
```

创建一个渲染缓冲对象和纹理对象类似，不同的是这个对象是专门被设计作为帧缓冲附件使用的，而不是纹理那样的通用数据缓冲(General Purpose Data Buffer)。这里我们选择 GL_DEPTH24_STENCIL8 作为内部格式，它封装了 24 位的深度和 8 位的模板缓冲。

最后一件事就是附加这个渲染缓冲对象：

```c++
glFramebufferRenderbuffer(GL_FRAMEBUFFER, GL_DEPTH_STENCIL_ATTACHMENT, GL_RENDERBUFFER, rbo);
```

**渲染缓冲对象能为你的帧缓冲对象提供一些优化，但知道什么时候使用渲染缓冲对象，什么时候使用纹理是很重要的。**通常的规则是，如果你不需要从一个缓冲中采样数据，那么对这个缓冲使用渲染缓冲对象会是明智的选择。如果你需要从缓冲中采样颜色或深度值等数据，那么你应该选择纹理附件。性能方面它不会产生非常大的影响的。

### 检测可用性

在完成所有的条件之后，我们可以以 GL_FRAMEBUFFER 为参数调用 `glCheckFramebufferStatus` 来检查帧缓冲是否完整：它将会检测当前绑定的帧缓冲，并返回规范中[这些](https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/glCheckFramebufferStatus.xhtml)值的其中之一。如果它返回的是 GL_FRAMEBUFFER_COMPLETE，帧缓冲就是完整的了。

```c++
if(glCheckFramebufferStatus(GL_FRAMEBUFFER) == GL_FRAMEBUFFER_COMPLETE)
  // 执行胜利的舞蹈
```

之后所有的渲染操作将会渲染到当前绑定帧缓冲的附件中。**由于我们的帧缓冲不是默认帧缓冲，渲染指令将不会对窗口的视觉输出有任何影响。**出于这个原因，渲染到一个不同的帧缓冲被叫做**离屏渲染(Off-screen Rendering)**。要保证所有的渲染操作在主窗口中有视觉效果，我们需要再次激活默认帧缓冲，将它绑定到`0`。

```c++
glBindFramebuffer(GL_FRAMEBUFFER, 0);
```

### 渲染到纹理

既然我们已经知道帧缓冲（大概）是怎么工作的了，是时候实践它们了。**我们将会将场景渲染到一个附加到帧缓冲对象上的颜色纹理中，之后将在一个横跨整个屏幕的四边形上绘制这个纹理。**这样视觉输出和没使用帧缓冲时是完全一样的，但这次是打印到了一个四边形上。这为什么很有用呢？我们会在下一部分中知道原因。

首先要创建一个帧缓冲对象，并绑定它，这些都很直观：

```c++
unsigned int framebuffer;
glGenFramebuffers(1, &framebuffer);
glBindFramebuffer(GL_FRAMEBUFFER, framebuffer);
```

接下来我们需要创建一个纹理图像，我们将它作为一个颜色附件附加到帧缓冲上。我们将纹理的维度设置为窗口的宽度和高度，并且不初始化它的数据：

```c++
// 生成纹理
unsigned int texColorBuffer;
glGenTextures(1, &texColorBuffer);
glBindTexture(GL_TEXTURE_2D, texColorBuffer);
glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, 800, 600, 0, GL_RGB, GL_UNSIGNED_BYTE, NULL);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR );
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
glBindTexture(GL_TEXTURE_2D, 0);

// 将它附加到当前绑定的帧缓冲对象
glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, GL_TEXTURE_2D, texColorBuffer, 0);
```

我们还希望 OpenGL 能够进行深度测试（如果你需要的话还有模板测试），所以我们还需要添加一个深度（和模板）附件到帧缓冲中。由于我们只希望采样颜色缓冲，而不是其它的缓冲，我们可以为它们创建一个渲染缓冲对象。还记得当我们不需要采样缓冲的时候，渲染缓冲对象是更好的选择吗？

创建一个渲染缓冲对象不是非常复杂。我们需要记住的唯一事情是，我们将它创建为一个深度和模板附件渲染缓冲对象。我们将它的**内部**格式设置为 GL_DEPTH24_STENCIL8，对我们来说这个精度已经足够了。

```c++
unsigned int rbo;
glGenRenderbuffers(1, &rbo);
glBindRenderbuffer(GL_RENDERBUFFER, rbo);
glRenderbufferStorage(GL_RENDERBUFFER, GL_DEPTH24_STENCIL8, 800, 600);
glBindRenderbuffer(GL_RENDERBUFFER, 0);
```

当我们为渲染缓冲对象分配了足够的内存之后，我们就可以解绑这个渲染缓冲了。

接下来，作为完成帧缓冲之前的最后一步，我们将渲染缓冲对象**附加到帧缓冲的深度和模板附件**上：

```c++
glFramebufferRenderbuffer(GL_FRAMEBUFFER, GL_DEPTH_STENCIL_ATTACHMENT, GL_RENDERBUFFER, rbo);
```

最后，我们希望检查帧缓冲是否是完整的，如果不是，我们将打印错误信息。

```c++
if(glCheckFramebufferStatus(GL_FRAMEBUFFER) != GL_FRAMEBUFFER_COMPLETE)
    std::cout << "ERROR::FRAMEBUFFER:: Framebuffer is not complete!" << std::endl;
glBindFramebuffer(GL_FRAMEBUFFER, 0);
```

现在这个帧缓冲就完整了，我们只需要绑定这个帧缓冲对象，让渲染到帧缓冲的缓冲中而不是默认的帧缓冲中。之后的渲染指令将会影响当前绑定的帧缓冲。所有的深度和模板操作都会从当前绑定的帧缓冲的深度和模板附件中（如果有的话）读取。如果你忽略了深度缓冲，那么所有的深度测试操作将不再工作，因为当前绑定的帧缓冲中不存在深度缓冲。

所以，要想绘制场景到一个纹理上，我们需要采取以下的步骤：

1. 将新的帧缓冲绑定为激活的帧缓冲，和往常一样渲染场景
2. 绑定默认的帧缓冲
3. **绘制一个横跨整个屏幕的四边形，将帧缓冲的颜色缓冲作为它的纹理。**

为了绘制这个四边形，我们将会新创建一套简单的着色器。我们将不会包含任何花哨的矩阵变换，因为我们提供的是标准化设备坐标的[顶点坐标](https://learnopengl.com/code_viewer.php?code=advanced/framebuffers_quad_vertices)，所以我们可以直接将它们设定为顶点着色器的输出。顶点着色器是这样的：

```c++
#version 330 core
layout (location = 0) in vec2 aPos;
layout (location = 1) in vec2 aTexCoords;

out vec2 TexCoords;

void main() {
    gl_Position = vec4(aPos.x, aPos.y, 0.0, 1.0);
    TexCoords = aTexCoords;
}
```

并没有太复杂的东西。片段着色器会更加基础，我们做的唯一一件事就是从纹理中采样：

```c++
#version 330 core
out vec4 FragColor;

in vec2 TexCoords;

uniform sampler2D screenTexture;

void main() {
    FragColor = texture(screenTexture, TexCoords);
}
```

接着就靠你来为屏幕四边形创建并配置一个 VAO 了。帧缓冲的一个渲染迭代将会有以下的结构：

```c++
// 第一处理阶段(Pass)
glBindFramebuffer(GL_FRAMEBUFFER, framebuffer);
glClearColor(0.1f, 0.1f, 0.1f, 1.0f);
glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT); // 我们现在不使用模板缓冲
glEnable(GL_DEPTH_TEST);
DrawScene();

// 第二处理阶段
glBindFramebuffer(GL_FRAMEBUFFER, 0); // 返回默认
glClearColor(1.0f, 1.0f, 1.0f, 1.0f);
glClear(GL_COLOR_BUFFER_BIT);

screenShader.use();
glBindVertexArray(quadVAO);
glDisable(GL_DEPTH_TEST);
glBindTexture(GL_TEXTURE_2D, textureColorbuffer);
glDrawArrays(GL_TRIANGLES, 0, 6);
```

要注意一些事情。第一，由于我们使用的每个帧缓冲都有它自己一套缓冲，我们希望设置合适的位，调用 glClear，清除这些缓冲。第二，当绘制四边形时，我们将禁用深度测试，因为我们是在绘制一个简单的四边形，并不需要关系深度测试。在绘制普通场景的时候我们将会重新启用深度测试。

有很多步骤都可能会出错，所以如果你没有得到输出的话，尝试调试程序，并重新阅读本节的相关部分。如果所有的东西都能够正常工作，你将会得到下面这样的视觉输出：

![img](https://learnopengl-cn.github.io/img/04/05/framebuffers_screen_texture.png)

左边展示的是视觉输出，它和[深度测试](https://learnopengl-cn.github.io/04 Advanced OpenGL/01 Depth testing/)中是完全一样的，但这次是渲染在一个简单的四边形上。如果我们使用线框模式渲染场景，就会变得很明显，我们在默认的帧缓冲中只绘制了一个简单的四边形。

所以这个有什么用处呢？因为我们能够以一个纹理图像的方式访问已渲染场景中的每个像素，我们可以在片段着色器中创建出非常有趣的效果。这些有趣效果统称为后期处理(Post-processing)效果。

### 基于色彩的后处理效果

由于离屏渲染的采用，我们得到了一张正常渲染结果的 RT，现在可以像 Unity 中的 Blit()调用一样去做出很多有意思的视觉效果，这些统称为后处理。

#### 反相

从屏幕纹理中取颜色值，然后用 1.0 减去它，就能达到反相的效果：

```c++
void main() {
    FragColor = vec4(vec3(1.0 - texture(screenTexture, TexCoords)), 1.0);
}
```

![img](https://learnopengl-cn.github.io/img/04/05/framebuffers_inverse.png)

#### 灰度

另外一个常见的效果是图像灰度化(Grayscale)，它移除了场景中除了黑白灰以外所有的颜色。这实际上是采用一种算法去计算像素亮度，最简单的就是取所有的颜色分量，将它们平均化：

```c++
void main() {
    FragColor = texture(screenTexture, TexCoords);
    float average = (FragColor.r + FragColor.g + FragColor.b) / 3.0;
    FragColor = vec4(average, average, average, 1.0);
}
```

这已经能创造很好的结果了，但人眼会对绿色更加敏感一些，而对蓝色不那么敏感，所以为了获取物理上更精确的效果，我们需要使用加权的(Weighted)通道：

```c++
void main() {
    FragColor = texture(screenTexture, TexCoords);
    float average = 0.2126 * FragColor.r + 0.7152 * FragColor.g + 0.0722 * FragColor.b;
    // 这里实际上是我们一般会采用的亮度的计算公式，你可以参考这篇文章：https://www.cnblogs.com/wildtester/p/15862624.html
    FragColor = vec4(average, average, average, 1.0);
}
```

![img](https://learnopengl-cn.github.io/img/04/05/framebuffers_grayscale.png)

> 你可能不会立刻发现有什么差别，但在更复杂的场景中，这样的加权灰度效果会更真实一点。

### 基于卷积的后处理效果

在一个纹理图像上做后期处理的另外一个好处是，我们可以从纹理的其它地方采样颜色值。比如说我们可以在当前纹理坐标的周围取一小块区域，对当前纹理值周围的多个纹理值进行采样。我们可以结合它们创建出很有意思的效果。

这实际上是卷积操作，而卷积操作的核心就是其中的卷积核(Kernel)。核是一个类矩阵的数值数组，它的中心为当前的像素，它会用它的核值乘以周围的像素值，并将结果相加变成一个值。所以，基本上我们是在对当前像素周围的纹理坐标添加一个小的偏移量，并根据核将结果合并。下面是核的一个例子：

$$
\begin{bmatrix}
2 & 2 & 2\\
2 & -15 & 2\\
2 & 2 & 2
\end{bmatrix}
$$

你在网上找到的大部分核将所有的权重加起来之后都应该会等于 1，如果它们加起来不等于 1，这就意味着最终的纹理颜色将会比原纹理值更亮或者更暗了。而在这个核中，我们共取了 8 个周围像素值，将它们乘以 2，而把当前的像素乘以-15。这个核的例子将周围的像素乘上了一个权重，并将当前像素乘以一个比较大的负权重来平衡结果。

后面你会发现，虽然它们都是被称为卷积的操作，但是对于采用不同的卷积核并辅以不同的计算方式时，我们就能获得不同的执行结果。

#### 锐化

核是后期处理一个非常有用的工具，它们使用和实验起来都很简单，网上也能找到很多例子。我们需要稍微修改一下片段着色器，让它能够支持核。我们假设使用的核都是 3x3 核（实际上大部分核都是）：

```c++
const float offset = 1.0 / 300.0;

void main() {
    vec2 offsets[9] = vec2[](
        vec2(-offset,  offset), // 左上
        vec2( 0.0f,    offset), // 正上
        vec2( offset,  offset), // 右上
        vec2(-offset,  0.0f),   // 左
        vec2( 0.0f,    0.0f),   // 中
        vec2( offset,  0.0f),   // 右
        vec2(-offset, -offset), // 左下
        vec2( 0.0f,   -offset), // 正下
        vec2( offset, -offset)  // 右下
    );

    float kernel[9] = float[](
        -1, -1, -1,
        -1,  9, -1,
        -1, -1, -1
    );

    vec3 sampleTex[9];
    for(int i = 0; i < 9; i++) sampleTex[i] = vec3(texture(screenTexture, TexCoords.st + offsets[i]));
    vec3 col = vec3(0.0);
    for(int i = 0; i < 9; i++) col += sampleTex[i] * kernel[i];

    FragColor = vec4(col, 1.0);
}
```

在片段着色器中，我们首先为周围的纹理坐标创建了一个 9 个`vec2`偏移量的数组。偏移量是一个常量，你可以按照你的喜好自定义它。之后我们定义一个核，在这个例子中是一个锐化(Sharpen)核，它会采样周围的所有像素，锐化每个颜色值。最后，在采样时我们将每个偏移量加到当前纹理坐标上，获取需要采样的纹理，之后将这些纹理值乘以加权的核值，并将它们加到一起。

这个锐化核看起来是这样的：

![img](https://learnopengl-cn.github.io/img/04/05/framebuffers_sharpen.png)

这能创建一些很有趣的效果，比如说你的玩家打了**麻醉剂/兴奋剂**所感受到的效果。

#### 模糊

创建模糊(Blur)效果的核是类似这样的：

$$
\begin{bmatrix}
1 & 2 & 1\\
2 & 4 & 2\\
1 & 2 & 1
\end{bmatrix}
$$

另外值得一提的是，高斯模糊所采用的也是卷积，只是它的卷积核更复杂一些，是使用高斯方程计算出来的：

$$
G(x,y)=\frac{1}{2\pi\sigma^2}e^{-\frac{x^2+y^2}{2\sigma ^ 2}}
$$

其中，x 和 y 分别是当前位置到中心位置的整数距离，σ 是标准方差并且常取 1，并且需要对最终结果做归一化以防止图片亮度发生变化。那么在这里，由于所有值的和是 16，所以直接返回合并的采样颜色将产生非常亮的颜色，所以我们需要将核的每个值都除以 16。最终的核数组将会是：

```c++
float kernel[9] = float[](
    1.0 / 16, 2.0 / 16, 1.0 / 16,
    2.0 / 16, 4.0 / 16, 2.0 / 16,
    1.0 / 16, 2.0 / 16, 1.0 / 16
);
```

通过在片段着色器中改变核的 float 数组，我们完全改变了后期处理效果。它现在看起来是这样子的：

![img](https://learnopengl-cn.github.io/img/04/05/framebuffers_blur.png)

这样的模糊效果创造了很多的可能性。我们可以随着时间修改模糊的量，创造出玩家醉酒时的效果，或者在主角没带眼镜的时候增加模糊。模糊也能够让我们来平滑颜色值，我们将在之后教程中使用到。

你可以看到，只要我们有了这个核的实现，创建炫酷的后期处理特效是非常容易的事。我们再来看最后一个很流行的效果来结束本节的讨论。

#### 边缘检测

下面的边缘检测(Edge-detection)核和锐化核非常相似：

$$
\begin{bmatrix}
1 & 1 & 1\\
1 & -8 & 1\\
1 & 1 & 1
\end{bmatrix}
$$

**这个核高亮了所有的边缘，而暗化了其它部分，在我们只关心图像的边角的时候是非常有用的。**

![img](https://learnopengl-cn.github.io/img/04/05/framebuffers_edge_detection.png)

你可能不会奇怪，像是 Photoshop 这样的图像修改工具/滤镜使用的也是这样的核。因为显卡处理片段的时候有着极强的并行处理能力，我们可以很轻松地在实时的情况下逐像素对图像进行处理。所以图像编辑工具在图像处理的时候会更倾向于使用显卡。

> 译注：
>
> 注意，核在对屏幕纹理的边缘进行采样的时候，由于还会对中心像素周围的 8 个像素进行采样，其实会取到纹理之外的像素。由于**环绕方式**默认是 GL_REPEAT，所以在没有设置的情况下取到的是屏幕另一边的像素，而另一边的像素本不应该对中心像素产生影响，这就可能会在屏幕边缘产生很奇怪的条纹。为了消除这一问题，我们可以将屏幕纹理的环绕方式都设置为 GL_CLAMP_TO_EDGE。这样子在取到纹理外的像素时，就能够重复边缘的像素来更精确地估计最终的值了。

### 终局

在完成所有的帧缓冲操作之后，不要忘记删除这个帧缓冲对象：

```c++
glDeleteFramebuffers(1, &fbo);
```

## 立方体贴图 - CubeMap

我们已经使用 2D 纹理很长时间了，但除此之外仍有更多的纹理类型等着我们探索。在本节中，我们将讨论的是将多个纹理组合起来映射到一张纹理上的一种纹理类型：立方体贴图(Cube Map)。

> 需要说明的是，我们之前所使用的确实是 TEXTURE2D，但是这里的 CUBEMAP 并非就等于 3D TEXTURE，应该说 CubeMap 最多只能算是 3D TEXTURE 的一种，而我们在实际使用中往往将两者区分开来使用，关于 3D TEXTURE，可参考：[Unity - Manual: 3D textures (unity3d.com)](https://docs.unity3d.com/Manual/class-Texture3D.html) 与 [Introduction To Textures in Direct3D 11 - Win32 apps | Microsoft Learn](https://learn.microsoft.com/en-us/windows/win32/direct3d11/overviews-direct3d-11-resources-textures-intro)。另外当然也有 1D TEXTURE，它们通常是以 1 pixel 的 height 已经足够长的 width 出现，虽然它们实际上的形式是 2D 的，但是比起实在的 2D TEXTURE，它们已经少了一个维度(y 轴已经被指定为 1)，比如说一些用于做色带映射的 ToonMap。

简单来说，立方体贴图就是一个包含了 6 个 2D 纹理的纹理，每个 2D 纹理都组成了立方体的一个面：一个有纹理的立方体。你可能会奇怪，这样一个立方体有什么用途呢？为什么要把 6 张纹理合并到一张纹理中，而不是直接使用 6 个单独的纹理呢？立方体贴图有一个非常有用的特性，它可以通过一个方向向量来进行索引/采样。假设我们有一个 1x1x1 的单位立方体，方向向量的原点位于它的中心。使用一个橘黄色的方向向量来从立方体贴图上采样一个纹理值会像是这样：

![img](https://learnopengl-cn.github.io/img/04/06/cubemaps_sampling.png)

![img](https://pic4.zhimg.com/80/v2-8f24b8b97a2bc10c4fc0ba34b07f0037_1440w.webp)
[详解 Cubemap、IBL 与球谐光照 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/463309766)，你可以用这张图去和我们使用的 vertices 做对比

方向向量的大小并不重要，只要提供了方向，OpenGL 就会获取方向向量（最终）所击中的纹素，并返回对应的采样纹理值——这和 Texture2D 确实有所不同。

如果我们假设将这样的立方体贴图应用到一个立方体上，采样立方体贴图所使用的方向向量将和立方体（插值的）顶点位置非常相像。这样子，**只要立方体的中心位于原点，我们就能使用立方体的实际位置向量来对立方体贴图进行采样了。**接下来，我们可以将所有顶点的纹理坐标当做是立方体的顶点位置。最终得到的结果就是可以访问立方体贴图上正确面(Face)纹理的一个纹理坐标。

### 创建立方体贴图

立方体贴图是和其它纹理一样的，所以如果想创建一个立方体贴图的话，我们需要生成一个纹理，并将其绑定到纹理目标上，之后再做其它的纹理操作。这次要绑定到 GL_TEXTURE_CUBE_MAP：

```c++
unsigned int textureID;
glGenTextures(1, &textureID);
glBindTexture(GL_TEXTURE_CUBE_MAP, textureID);
```

因为立方体贴图包含有 6 个纹理，每个面一个，我们需要调用 glTexImage2D 函数 6 次，参数和之前教程中很类似。但这一次我们将纹理目标(**target**)参数设置为立方体贴图的一个特定的面，告诉 OpenGL 我们在对立方体贴图的哪一个面创建纹理。这就意味着我们需要对立方体贴图的每一个面都调用一次 glTexImage2D。

由于我们有 6 个面，OpenGL 给我们提供了 6 个特殊的纹理目标，专门对应立方体贴图的一个面。

| 纹理目标                         | 方位 |
| :------------------------------- | :--- |
| `GL_TEXTURE_CUBE_MAP_POSITIVE_X` | 右   |
| `GL_TEXTURE_CUBE_MAP_NEGATIVE_X` | 左   |
| `GL_TEXTURE_CUBE_MAP_POSITIVE_Y` | 上   |
| `GL_TEXTURE_CUBE_MAP_NEGATIVE_Y` | 下   |
| `GL_TEXTURE_CUBE_MAP_POSITIVE_Z` | 后   |
| `GL_TEXTURE_CUBE_MAP_NEGATIVE_Z` | 前   |

**和 OpenGL 的很多枚举(Enum)一样，它们背后的 int 值是线性递增的**，所以如果我们有一个纹理位置的数组或者 vector，我们就可以从 GL_TEXTURE_CUBE_MAP_POSITIVE_X 开始遍历它们，在每个迭代中对枚举值加 1，遍历了整个纹理目标：

```c++
int width, height, nrChannels;
unsigned char *data;
for(unsigned int i = 0; i < textures_faces.size(); i++) {
    data = stbi_load(textures_faces[i].c_str(), &width, &height, &nrChannels, 0);
    glTexImage2D(GL_TEXTURE_CUBE_MAP_POSITIVE_X + i, 0, GL_RGB, width, height, 0, GL_RGB, GL_UNSIGNED_BYTE, data);
}
```

这里我们有一个叫做 textures_faces 的 vector，它包含了立方体贴图所需的所有纹理路径，并以表中的顺序排列。这将为当前绑定的立方体贴图中的每个面生成一个纹理。

因为立方体贴图和其它纹理没什么不同，我们也需要设定它的环绕和过滤方式：

```c++
glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE);
glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE);
glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_WRAP_R, GL_CLAMP_TO_EDGE);
```

不要被 GL_TEXTURE_WRAP_R 吓到，它仅仅是为纹理的**R**坐标设置了环绕方式，它对应的是纹理的第三个维度（和位置的**z**一样）。我们将环绕方式设置为 GL_CLAMP_TO_EDGE，这是因为正好处于两个面之间的纹理坐标可能不能击中一个面（由于一些硬件限制），所以通过使用 GL_CLAMP_TO_EDGE，OpenGL 将在我们对两个面之间采样的时候，永远返回它们的边界值。

在绘制使用立方体贴图的物体之前，我们要先激活对应的纹理单元，并绑定立方体贴图，这和普通的 2D 纹理没什么区别。

在片段着色器中，我们使用了一个不同类型的采样器，`samplerCube`，我们将使用 texture 函数使用它进行采样，但这次我们将使用一个`vec3`的方向向量而不是`vec2`。使用立方体贴图的片段着色器会像是这样的：

```c++
in vec3 textureDir; // 代表3D纹理坐标的方向向量
uniform samplerCube cubemap; // 立方体贴图的纹理采样器

void main(){
    FragColor = texture(cubemap, textureDir); // == FragColor = texture(skybox, normalize(TexCoords));
}
```

我们可以用立方体贴图来实现很多东西，当然，其中最广为人知的一个技术想必就是创建天空盒(Skybox)了。

### 天空盒

我们很快就能法线，立方体贴图能完美满足天空盒的需求：我们有一个 6 面的立方体，每个面都需要一个纹理。在上面的图片中，他们使用了夜空的几张图片，让玩家产生其位于广袤宇宙中的错觉，但实际上他只是在一个小小的盒子当中。

你可以在网上找到很多像这样的天空盒资源。比如说这个[网站](http://www.custommapmakers.org)就提供了很多天空盒。天空盒图像通常有以下的形式：

![img](https://learnopengl-cn.github.io/img/04/06/cubemaps_skybox.png)

如果你将这六个面折成一个立方体，你就会得到一个完全贴图的立方体，模拟一个巨大的场景。一些资源可能会提供了这样格式的天空盒，你必须手动提取六个面的图像，但在大部分情况下它们都是 6 张单独的纹理图像。

### 加载天空盒

关于天空盒的加载，我觉得有这些东西是需要指出的。

首先是用来读取 Skybox 的模型的定义：观察 LearnOpenGL 网站为我们提供的顶点数据，我们可以发现：这个箱子模型比我们原先所使用的刚好大了一号——它被用来框住我们的摄像机。如果是你，你会怎么去实现这个 Skybox 的功能？LearnOpenGL 给出的解决方案是：直接去定义 Skybox 模型的 View Space 坐标去框住摄像机，并让它的 Front 方向朝内以保证不会被剔除，在此基础上去读取了 CubeMap 的内容。为了实现这个目的，我们需要一个单位矩阵的 Model Matrix，一个只有旋转属性而抛弃了位移的 View Matrix，这样就能保证它所定义的 Vertices 可以直接达成效果。

关于优化的内容：LearnOpenGL 先举了首先渲染天空盒，之后再渲染场景中的其它物体的例子。这样子能够工作，但不是非常高效。基于上方的原因，先渲染天空盒时是不会开启深度测试的，我们不仅会对屏幕上的每一个像素运行一遍片段着色器，后续的不透明物体的渲染也不会享受到任何红利。

而 LearnOpenGL 所谓的“最后渲染天空盒"，其实应该是指在所有不透明物体后，进行 Skybox 的绘制。这样子的话，深度缓冲就会填充满所有物体的深度值了，我们只需要在提前深度测试通过的地方渲染天空盒的片段就可以了，很大程度上减少了片段着色器的调用。

另一个问题是，天空盒很可能会渲染在所有其他对象之上，因为它只是一个 1x1x1 的立方体（译注：意味着距离摄像机的距离也只有 1），会通过大部分的深度测试。如果将 Skybox 的渲染顺序调后的话，禁用深度测试来进行渲染就不是解决方案了——因为天空盒将会复写场景中的其它物体。我们需要欺骗深度缓冲，让它认为天空盒有着最大的深度值 1.0，只要它前面有一个物体，深度测试就会失败。也就是说，我们得手动修改 gl_position 的值：直接将输出位置的 z 分量等于它的 w 分量，让 z 分量永远等于 1.0，这样子的话，当透视除法执行之后，z 分量会变为`w / w = 1.0`。

```c++
void main() {
    TexCoords = aPos;
    vec4 pos = projection * view * vec4(aPos, 1.0);
    gl_Position = pos.xyww;
}
```

最终的**标准化设备坐标**将永远会有一个等于 1.0 的 z 值：最大的深度值。结果就是天空盒只会在没有可见物体的地方渲染了（只有这样才能通过深度测试，其它所有的东西都在天空盒前面）。

我们还要改变一下深度函数，将它从默认的 GL_LESS 改为 GL_LEQUAL。深度缓冲将会填充上天空盒的 1.0 值，所以我们需要保证天空盒在值小于或等于深度缓冲而不是小于时通过深度测试。

### 环境映射——反射与折射

反射这个属性表现为物体（或物体的一部分）反射它周围环境，即根据观察者的视角，物体的颜色或多或少等于它的环境。镜子就是一个反射性物体：它会根据观察者的视角反射它周围的环境。

反射的原理并不难。下面这张图展示了我们如何计算反射向量，并如何使用这个向量来从立方体贴图中采样：

![img](https://learnopengl-cn.github.io/img/04/06/cubemaps_reflection_theory.png)

具体做法可简述为：我们有对某片元的一个观察方向，根据 Normal 算 Reflect Dir，依照它来对 CubeMap 进行采样

而环境映射的另一种形式是折射，它和反射很相似。折射是光线由于传播介质的改变而产生的方向变化。在常见的类水表面上所产生的现象就是折射，光线不是直直地传播，而是弯曲了一点。将你的半只胳膊伸进水里，观察出来的就是这种效果。

折射是通过[斯涅尔定律](https://en.wikipedia.org/wiki/Snell's_law)(Snell’s Law)来描述的，使用环境贴图的话看起来像是这样：

![img](https://learnopengl-cn.github.io/img/04/06/cubemaps_refraction_theory.png)

同样，我们有一个观察向量，一个法向量，与一个与之前稍有不同的折射向量 R。

> 通常来说，我们在得到折射方向后就可以直接用它来对立方体进行采样了，即使这并不符合物理规律——对一个透明物体来说，这应该是存在两次折射的，一次入射一次出射。但是想要在实时渲染中模拟出第二次折射的方向是比较复杂的，而且仅模拟一次得到的效果也挺像那么回事的——那么根据图形学第一准则：“如果它看上去是对的，那么它就是对的”。

折射也可以使用 GLSL 的内建 `refract` 函数来轻松实现，它需要一个法向量、一个观察方向和**两个材质之间的折射率**(Refractive Index)。

折射率决定了材质中光线弯曲的程度，每个材质都有自己的折射率。一些最常见的折射率可以在下表中找到：

| 材质 | 折射率 |
| :--- | :----- |
| 空气 | 1.00   |
| 水   | 1.33   |
| 冰   | 1.309  |
| 玻璃 | 1.52   |
| 钻石 | 2.42   |

我们使用这些折射率就可以来计算光传播的两种材质间的**比值**了。

### 动态环境贴图

**现在我们使用的都是静态图像的组合来作为天空盒，看起来很不错，但它没有在场景中包括可移动的物体。**我们一直都没有注意到这一点，因为我们只使用了一个物体。如果我们有一个镜子一样的物体，周围还有多个物体，镜子中可见的只有天空盒，看起来就像它是场景中唯一一个物体一样。

**通过使用帧缓冲，我们能够为物体的 6 个不同角度创建出场景的纹理，并在每个渲染迭代中将它们储存到一个立方体贴图中。之后我们就可以使用这个（动态生成的）立方体贴图来创建出更真实的，包含其它物体的，反射和折射表面了。这就叫做动态环境映射(Dynamic Environment Mapping)，因为我们动态创建了物体周围的立方体贴图，并将其用作环境贴图。**

虽然它看起来很棒，但它有一个很大的缺点：我们需要为使用环境贴图的物体渲染场景 6 次，这是对程序是非常大的性能开销。现代的程序通常会尽可能使用天空盒，并在可能的时候使用预编译的立方体贴图，只要它们能产生一点动态环境贴图的效果。虽然动态环境贴图是一个很棒的技术，但是要想在不降低性能的情况下让它工作还是需要非常多的技巧的。

## 高级数据

> 对各种 Buffer 数据的再审视

让我们在开始更多新内容前先回忆一下先前的操作。我们的 Application 阶段与 Render 阶段的、最直接的沟通就是各种 Buffer 的创建，而在这些 Buffer 的创建过程中，存在一些范式——它们都以一个 Unsigned Int 为基础，基于不同的 Target 去 GenBuffer，然后是做绑定，分配大小等等诸如此类的东西。现在，我们来关注一下这个真正生效的这一步——正式分配内存的这一步：

> **OpenGL 中的缓冲只是一个管理特定内存块的对象，没有其它更多的功能了。在我们将它绑定到一个缓冲目标(Buffer Target)时，我们才赋予了其意义。**当我们绑定一个缓冲到 GL_ARRAY_BUFFER 时，它就是一个顶点数组缓冲，但我们也可以很容易地将其绑定到 GL_ELEMENT_ARRAY_BUFFER。OpenGL 内部会为每个目标储存一个缓冲，并且会根据目标的不同，以不同的方式处理缓冲。
>
> 到目前为止，我们一直是调用 glBufferData 函数来填充缓冲对象所管理的内存，这个函数会**分配一块内存**，并将数据添加到这块内存中。如果我们将它的`data`参数设置为`NULL`，那么这个函数将只会分配内存，但不进行填充。这在我们需要**预留**(Reserve)特定大小的内存，之后回到这个缓冲一点一点填充的时候会很有用。

除了使用一次函数调用填充整个缓冲之外，我们也可以使用 `glBufferSubData`，填充缓冲的特定区域。这个函数需要一个缓冲目标、一个偏移量、数据的大小和数据本身作为它的参数。这个函数不同的地方在于，我们可以提供一个偏移量，指定从**何处**开始填充这个缓冲。这能够让我们插入或者更新缓冲内存的某一部分。要注意的是，缓冲需要有足够的已分配内存，所以对一个缓冲调用 `glBufferSubData` 之前必须要先调用 `glBufferData`。

```c++
glBufferSubData(GL_ARRAY_BUFFER, 24, sizeof(data), &data); // 范围： [24, 24 + sizeof(data)]
```

### 分批顶点属性

通过使用 `glVertexAttribPointer`，我们能够指定顶点数组缓冲内容的属性布局。在之前对顶点数组缓冲的操作中，我们对属性进行了交错(Interleave)处理，也就是说，**我们将每一个顶点的位置、法线和/或纹理坐标紧密放置在一起。**既然我们现在已经对缓冲有了更多的了解，我们可以采取另一种方式。

我们可以做的是，**将每一种属性类型的向量数据打包(Batch)为一个大的区块，而不是对它们进行交错储存。**与交错布局 123123123123 不同，我们将采用分批(Batched)的方式 111222333。

当从文件中加载顶点数据的时候，你通常获取到的是一个位置数组、一个法线数组和/或一个纹理坐标数组。我们需要花点力气才能将这些数组转化为一个大的交错数据数组。使用分批的方式会是更简单的解决方案，我们可以很容易使用 glBufferSubData 函数实现：

```c++
float positions[] = { ... };
float normals[] = { ... };
float tex[] = { ... };
// 填充缓冲
glBufferSubData(GL_ARRAY_BUFFER, 0, sizeof(positions), &positions);
glBufferSubData(GL_ARRAY_BUFFER, sizeof(positions), sizeof(normals), &normals);
glBufferSubData(GL_ARRAY_BUFFER, sizeof(positions) + sizeof(normals), sizeof(tex), &tex);
```

这样子我们就能直接将属性数组作为一个整体传递给缓冲，而不需要事先处理它们了。我们仍可以将它们合并为一个大的数组，再使用 `glBufferData` 来填充缓冲，但对于这种工作，使用 `glBufferSubData` 会更合适一点。

我们还需要更新顶点属性指针来反映这些改变：

```c++
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), 0);
glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)(sizeof(positions)));
glVertexAttribPointer(2, 2, GL_FLOAT, GL_FALSE, 2 * sizeof(float), (void*)(sizeof(positions) + sizeof(normals)));
```

注意`stride`参数等于顶点属性的大小，因为下一个顶点属性向量能在 3 个（或 2 个）分量之后找到。

这给了我们设置顶点属性的另一种方法。**使用哪种方法都不会对 OpenGL 有什么立刻的好处，它只是设置顶点属性的一种更整洁的方式。**具体使用的方法将完全取决于你的喜好与程序类型。

### 复制缓冲

当你的缓冲已经填充好数据之后，你可能会想与其它的缓冲共享其中的数据，或者想要将缓冲的内容复制到另一个缓冲当中。`glCopyBufferSubData` 能够让我们相对容易地从一个缓冲中复制数据到另一个缓冲中。这个函数的原型如下：

```c++
void glCopyBufferSubData(GLenum readtarget, GLenum writetarget, GLintptr readoffset, GLintptr writeoffset, GLsizeiptr size);
```

`readtarget`和`writetarget`参数需要填入复制源和复制目标的缓冲目标。比如说，我们可以将 VERTEX_ARRAY_BUFFER 缓冲复制到 VERTEX_ELEMENT_ARRAY_BUFFER 缓冲，分别将这些缓冲目标设置为读和写的目标。当前绑定到这些缓冲目标的缓冲将会被影响到。

但如果我们想读写数据的两个不同缓冲都为顶点数组缓冲该怎么办呢？**我们不能同时将两个缓冲绑定到同一个缓冲目标上**。正是出于这个原因，OpenGL 提供给我们另外两个缓冲目标，叫做 GL_COPY_READ_BUFFER 和 GL_COPY_WRITE_BUFFER。我们接下来就可以将需要的缓冲绑定到这两个缓冲目标上，并将这两个目标作为`readtarget`和`writetarget`参数。

接下来 `glCopyBufferSubData` 会从`readtarget`中读取`size`大小的数据，并将其写入`writetarget`缓冲的`writeoffset`偏移量处。下面这个例子展示了如何复制两个顶点数组缓冲：

```c++
float vertexData[] = { ... };
glBindBuffer(GL_COPY_READ_BUFFER, vbo1);
glBindBuffer(GL_COPY_WRITE_BUFFER, vbo2);
glCopyBufferSubData(GL_COPY_READ_BUFFER, GL_COPY_WRITE_BUFFER, 0, 0, sizeof(vertexData));
```

## 高级 GLSL

这一节中，LearnOpenGL 主要讨论了 OpenGL 中的一些系统语义。我们已经见过的 `gl_Position` 就属于语义

### 顶点着色器变量

我们已经见过 `gl_Position` 了，它是顶点着色器的裁剪空间输出位置向量。如果你想在屏幕上显示任何东西，在顶点着色器中设置 `gl_Position` 是必须的步骤。这已经是它的全部功能了。

#### gl_PointSize

我们能够选用的其中一个图元是 GL_POINTS，**如果使用它的话，每一个顶点都是一个图元，都会被渲染为一个点。**我们可以通过 OpenGL 的 glPointSize 函数来设置渲染出来的点的大小，但我们也可以在顶点着色器中修改这个值。

GLSL 定义了一个叫做 gl_PointSize 输出变量，它是一个 float 变量，你可以使用它来设置点的宽高（像素）。在顶点着色器中修改点的大小的话，你就能对每个顶点设置不同的值了。

在顶点着色器中修改点大小的功能默认是禁用的，如果你需要启用它的话，你需要启用 OpenGL 的 GL_PROGRAM_POINT_SIZE：

```c++
glEnable(GL_PROGRAM_POINT_SIZE);
```

一个简单的例子就是将点的大小设置为裁剪空间位置的 z 值，也就是顶点距观察者的距离。点的大小会随着观察者距顶点距离变远而增大。

```c++
void main() {
    gl_Position = projection * view * model * vec4(aPos, 1.0);
    gl_PointSize = gl_Position.z;
}
```

结果就是，当我们远离这些点的时候，它们会变得更大：

![img](https://learnopengl-cn.github.io/img/04/08/advanced_glsl_pointsize.png)

你可以想到，对每个顶点使用不同的点大小，会在粒子生成之类的技术中很有意思。

#### gl_VertexID

gl_Position 和 gl_PointSize 都是**输出变量**，因为它们的值是作为顶点着色器的输出被读取的。我们可以对它们进行写入，来改变结果。顶点着色器还为我们提供了一个有趣的**输入变量**，我们只能对它进行读取，它叫做 gl_VertexID。

整型变量 gl_VertexID 储存了正在绘制顶点的当前 ID。当（使用 glDrawElements）进行索引渲染的时候，这个变量会存储正在绘制顶点的当前索引。当（使用 glDrawArrays）不使用索引进行绘制的时候，这个变量会储存从渲染调用开始的已处理顶点数量。

虽然现在它没有什么具体的用途，但知道我们能够访问这个信息总是好的。

### 片段着色器变量

在片段着色器中，我们也能访问到一些有趣的变量。GLSL 提供给我们两个有趣的输入变量：gl_FragCoord 和 gl_FrontFacing。

#### gl_FragCoord

在讨论深度测试的时候，我们已经见过 gl_FragCoord 很多次了，因为 gl_FragCoord 的 z 分量等于对应片段的深度值。然而，我们也能使用它的 x 和 y 分量来实现一些有趣的效果。

gl_FragCoord 的 x 和 y 分量是片段的窗口空间(Window-space)坐标(即能获取该像素对应的屏幕坐标)，其原点为窗口的左下角。我们已经使用 glViewport 设定了一个 800x600 的窗口了，所以片段窗口空间坐标的 x 分量将在 0 到 800 之间，y 分量在 0 到 600 之间。

通过利用片段着色器，我们可以根据片段的窗口坐标，计算出不同的颜色。gl_FragCoord 的一个常见用处是用于对比不同片段计算的视觉输出效果，这在技术演示中可以经常看到。比如说，我们能够将屏幕分成两部分，在窗口的左侧渲染一种输出，在窗口的右侧渲染另一种输出。下面这个例子片段着色器会根据窗口坐标输出不同的颜色：

```c++
void main()　{
    if(gl_FragCoord.x < 400) FragColor = vec4(1.0, 0.0, 0.0, 1.0);
    else FragColor = vec4(0.0, 1.0, 0.0, 1.0);
}
```

因为窗口的宽度是 800。当一个像素的 x 坐标小于 400 时，它一定在窗口的左侧，所以我们给它一个不同的颜色。

![img](https://learnopengl-cn.github.io/img/04/08/advanced_glsl_fragcoord.png)

我们现在会计算出两个完全不同的片段着色器结果，并将它们显示在窗口的两侧。举例来说，你可以将它用于测试不同的光照技巧。

#### gl_FrontFacing

片段着色器另外一个很有意思的输入变量是 gl_FrontFacing。在[面剔除](https://learnopengl-cn.github.io/04 Advanced OpenGL/04 Face culling/)教程中，我们提到 OpenGL 能够根据顶点的环绕顺序来决定一个面是正向还是背向面。如果我们不使用面剔除，那么 gl_FrontFacing 将会告诉我们当前片段是属于正向面的一部分还是背向面的一部分。举例来说，我们能够对正向面计算出不同的颜色。

gl_FrontFacing 变量是一个 bool，如果当前片段是正向面的一部分那么就是`true`，否则就是`false`。比如说，我们可以这样子创建一个立方体，在内部和外部使用不同的纹理：

```c++
#version 330 core
out vec4 FragColor;

in vec2 TexCoords;

uniform sampler2D frontTexture;
uniform sampler2D backTexture;

void main() {
    if(gl_FrontFacing) FragColor = texture(frontTexture, TexCoords);
    else FragColor = texture(backTexture, TexCoords);
}
```

如果我们往箱子里面看，就能看到使用的是不同的纹理。

![img](https://learnopengl-cn.github.io/img/04/08/advanced_glsl_frontfacing.png)

注意，如果你开启了面剔除，你就看不到箱子内部的面了，所以现在再使用 gl_FrontFacing 就没有意义了。

#### gl_FragDepth

输入变量 gl_FragCoord 能让我们读取**当前片段的窗口空间坐标，并获取它的深度值**，但是它是一个只读(Read-only)变量。我们不能修改片段的窗口空间坐标，但实际上修改片段的深度值还是可能的。GLSL 提供给我们一个叫做 gl_FragDepth 的输出变量，我们可以使用它来在着色器内设置片段的深度值。

要想设置深度值，我们直接写入一个 0.0 到 1.0 之间的 float 值到输出变量就可以了：

```c++
gl_FragDepth = 0.0; // 这个片段现在的深度值为 0.0
```

如果着色器没有写入值到 gl_FragDepth，它会自动取用`gl_FragCoord.z`的值。

然而，由我们自己设置深度值有一个很大的缺点，只要我们在片段着色器中对 gl_FragDepth 进行写入，OpenGL 就会禁用所有的 Early Z——因为 OpenGL 无法在片段着色器运行**之前**得知片段将拥有的深度值，因为片段着色器可能会完全修改这个深度值。

在写入 gl_FragDepth 时，你就需要考虑到它所带来的性能影响。

**然而，从 OpenGL 4.2 起，我们仍可以对两者进行一定的调和**，在片段着色器的顶部使用深度条件(Depth Condition)重新声明 gl_FragDepth 变量：

```c++
layout (depth_<condition>) out float gl_FragDepth;
```

`condition`可以为下面的值：

| 条件        | 描述                                                         |
| :---------- | :----------------------------------------------------------- |
| `any`       | 默认值。提前深度测试是禁用的，你会损失很多性能               |
| `greater`   | 你只能让深度值比`gl_FragCoord.z`更大                         |
| `less`      | 你只能让深度值比`gl_FragCoord.z`更小                         |
| `unchanged` | 如果你要写入`gl_FragDepth`，你将只能写入`gl_FragCoord.z`的值 |

通过将深度条件设置为`greater`或者`less`，OpenGL 就能假设你只会写入比当前片段深度值更大或者更小的值了。这样子的话，当深度值比片段的深度值要小的时候，OpenGL 仍是能够进行提前深度测试的。

下面这个例子中，我们对片段的深度值进行了递增，但仍然也保留了一些提前深度测试：

```c++
#version 420 core // 注意GLSL的版本！
out vec4 FragColor;
layout (depth_greater) out float gl_FragDepth;

void main() {
    FragColor = vec4(1.0);
    gl_FragDepth = gl_FragCoord.z + 0.1;
}
```

### 接口块

到目前为止，每当我们希望**从顶点着色器向片段着色器发送数据**时，我们都声明了几个对应的输入/输出变量。将它们一个一个声明是着色器间发送数据最简单的方式了，但当程序变得更大时，你希望发送的可能就不只是几个变量了，它还可能包括数组和结构体。

为了帮助我们管理这些变量，GLSL 为我们提供了一个叫做接口块(Interface Block)的东西，来方便我们组合这些变量。接口块的声明和 struct 的声明有点相像，不同的是，现在根据它是一个输入还是输出块(Block)，使用 in 或 out 关键字来定义的。

```c++
#version 330 core
layout (location = 0) in vec3 aPos;
layout (location = 1) in vec2 aTexCoords;

uniform mat4 model;
uniform mat4 view;
uniform mat4 projection;

out VS_OUT {
    vec2 TexCoords;
} vs_out;

void main() {
    gl_Position = projection * view * model * vec4(aPos, 1.0);
    vs_out.TexCoords = aTexCoords;
}
```

这次我们声明了一个叫做 VS_OUT 的接口块，对应的实例名为 vs_out，它打包了我们希望发送到下一个着色器中的所有输出变量。这只是一个很简单的例子，但你可以想象一下，它能够帮助你管理着色器的输入和输出。当我们希望将着色器的输入或输出打包为数组时，它也会非常有用。

之后，我们还需要在下一个着色器，即片段着色器，中定义一个输入接口块。块名(Block Name)应该是和着色器中一样的（VS_OUT），但实例名(Instance Name)（顶点着色器中用的是 vs_out）可以是随意的，但要避免使用误导性的名称，比如对实际上包含输入变量的接口块命名为 vs_out。

```c++
#version 330 core
out vec4 FragColor;

in VS_OUT {
    vec2 TexCoords;
} fs_in;

uniform sampler2D texture;

void main() {
    FragColor = texture(texture, fs_in.TexCoords);
}
```

只要两个接口块的名字一样，它们对应的输入和输出将会匹配起来。这是帮助你管理代码的又一个有用特性，它在几何着色器这样穿插特定着色器阶段的场景下会很有用。

### Uniform 缓冲对象

我们已经使用 OpenGL 很长时间了，学会了一些很酷的技巧，但也遇到了一些很麻烦的地方。比如说，当使用多于一个的着色器时，尽管大部分的 uniform 变量都是相同的，我们还是需要不断地设置它们，所以为什么要这么麻烦地重复设置它们呢？

OpenGL 为我们提供了一个叫做 Uniform 缓冲对象(Uniform Buffer Object)的工具，它允许我们定义一系列**在多个着色器中相同的全局 Uniform 变量**。当使用 Uniform 缓冲对象的时候，我们只需要设置相关的 uniform 一次。当然，我们仍需要手动设置每个着色器中不同的 uniform。并且创建和配置 Uniform 缓冲对象会有一点繁琐。

我们还是按照创建缓冲的一般范式来：先调用 glGenBuffers 创建一个缓冲对象冲，绑定到 GL_UNIFORM_BUFFER 目标，并调用 glBufferData 以分配内存。

```c++
unsigned int uboExampleBlock;
glGenBuffers(1, &uboExampleBlock);
glBindBuffer(GL_UNIFORM_BUFFER, uboExampleBlock);
glBufferData(GL_UNIFORM_BUFFER, 152, NULL, GL_STATIC_DRAW); // 分配152字节的内存
glBindBuffer(GL_UNIFORM_BUFFER, 0);
```

在 Uniform 缓冲对象中储存数据是有一些规则的，我们将会在之后继续讨论它。不过现在，我们先改写一个简单的顶点着色器，将 projection 和 view 矩阵存储到所谓的 Uniform 块(Uniform Block)中：

```c++
#version 330 core
layout (location = 0) in vec3 aPos;

layout (std140) uniform Matrices {
    mat4 projection;
    mat4 view;
};

uniform mat4 model;

void main() {
    gl_Position = projection * view * model * vec4(aPos, 1.0);
}
```

在我们大多数的例子中，我们都会在每个渲染迭代中，对每个着色器设置 projection 和 view Uniform 矩阵。这是利用 Uniform 缓冲对象的一个非常完美的例子，因为现在我们只需要存储这些矩阵一次就可以了。

这里，我们声明了一个叫做 Matrices 的 Uniform 块，它储存了两个 4x4 矩阵。Uniform 块中的变量可以直接访问，不需要加块名作为前缀。接下来，我们在 OpenGL 代码中将这些矩阵值存入缓冲中，每个声明了这个 Uniform 块的着色器都能够访问这些矩阵。

你现在可能会在想`layout (std140)`这个语句是什么意思。它的意思是说，当前定义的 Uniform 块对它的内容使用一个特定的内存布局。这个语句设置了 Uniform 块布局(Uniform Block Layout)。

#### Uniform 块布局

Uniform 块的内容是储存在一个缓冲对象中的，它实际上只是一块预留内存。因为这块内存并不会保存它具体保存的是什么类型的数据，我们还需要告诉 OpenGL 内存的哪一部分对应着着色器中的哪一个 uniform 变量。

假设着色器中有以下的这个 Uniform 块：

```c++
layout (std140) uniform ExampleBlock {
    float value;
    vec3  vector;
    mat4  matrix;
    float values[3];
    bool  boolean;
    int   integer;
};
```

我们需要知道的是每个变量的大小（字节）和（从块起始位置的）偏移量，来让我们能够按顺序将它们放进缓冲中。每个元素的大小都是在 OpenGL 中有清楚地声明的，而且直接对应 C++数据类型，其中向量和矩阵都是大的 float 数组。OpenGL 没有声明的是这些变量间的间距(Spacing)。这允许硬件能够在它认为合适的位置放置变量。比如说，一些硬件可能会将一个 vec3 放置在 float 边上。不是所有的硬件都能这样处理，可能会在附加这个 float 之前，先将 vec3 填充(Pad)为一个 4 个 float 的数组。这个特性本身很棒，但是会对我们造成麻烦。

默认情况下，GLSL 会使用一个叫做共享(Shared)布局的 Uniform 内存布局，共享是因为一旦硬件定义了偏移量，它们在多个程序中是**共享**并一致的。使用共享布局时，GLSL 是可以为了优化而对 uniform 变量的位置进行变动的，只要变量的顺序保持不变。因为我们无法知道每个 uniform 变量的偏移量，我们也就不知道如何准确地填充我们的 Uniform 缓冲了。我们能够使用像是 glGetUniformIndices 这样的函数来查询这个信息，但这超出本节的范围了。

虽然共享布局给了我们很多节省空间的优化，但是我们需要查询每个 uniform 变量的偏移量，这会产生非常多的工作量。通常的做法是，不使用共享布局，而是使用 std140 布局。std140 布局声明了每个变量的偏移量都是由一系列规则所决定的，这**显式地**声明了每个变量类型的内存布局。由于这是显式提及的，我们可以手动计算出每个变量的偏移量。

每个变量都有一个基准对齐量(Base Alignment)，它等于一个变量在 Uniform 块中所占据的空间（包括填充量(Padding)），这个基准对齐量是使用 std140 布局的规则计算出来的。接下来，对每个变量，我们再计算它的对齐偏移量(Aligned Offset)，它是一个变量从块起始位置的字节偏移量。一个变量的对齐字节偏移量**必须**等于基准对齐量的倍数。

布局规则的原文可以在 OpenGL 的 Uniform 缓冲规范[这里](http://www.opengl.org/registry/specs/ARB/uniform_buffer_object.txt)找到，但我们将会在下面列出最常见的规则。GLSL 中的每个变量，比如说 int、float 和 bool，都被定义为 4 字节量。每 4 个字节将会用一个`N`来表示。

| 类型                   | 布局规则                                                       |
| :--------------------- | :------------------------------------------------------------- |
| 标量，比如 int 和 bool | 每个标量的基准对齐量为 N。                                     |
| 向量                   | 2N 或者 4N。这意味着 vec3 的基准对齐量为 4N。                  |
| 标量或向量的数组       | 每个元素的基准对齐量与 vec4 的相同。                           |
| 矩阵                   | 储存为列向量的数组，每个向量的基准对齐量与 vec4 的相同。       |
| 结构体                 | 等于所有元素根据规则计算后的大小，但会填充到 vec4 大小的倍数。 |

和 OpenGL 大多数的规范一样，使用例子就能更容易地理解。我们会使用之前引入的那个叫做 ExampleBlock 的 Uniform 块，并使用 std140 布局计算出每个成员的对齐偏移量：

```c++
layout (std140) uniform ExampleBlock {
                     // 基准对齐量       // 对齐偏移量
    float value;     // 4               // 0
    vec3 vector;     // 16              // 16  (必须是16的倍数，所以 4->16)
    mat4 matrix;     // 16              // 32  (列 0)
                     // 16              // 48  (列 1)
                     // 16              // 64  (列 2)
                     // 16              // 80  (列 3)
    float values[3]; // 16              // 96  (values[0])
                     // 16              // 112 (values[1])
                     // 16              // 128 (values[2])
    bool boolean;    // 4               // 144
    int integer;     // 4               // 148
};
```

作为练习，尝试去自己计算一下偏移量，并和表格进行对比。使用计算后的偏移量值，**根据 std140 布局的规则，我们就能使用像是 glBufferSubData 的函数将变量数据按照偏移量填充进缓冲中了。**虽然 std140 布局不是最高效的布局，但它保证了内存布局在每个声明了这个 Uniform 块的程序中是一致的。

通过在 Uniform 块定义之前添加`layout (std140)`语句，我们告诉 OpenGL 这个 Uniform 块使用的是 std140 布局。除此之外还可以选择两个布局，但它们都需要我们在填充缓冲之前先查询每个偏移量。我们已经见过`shared`布局了，剩下的一个布局是`packed`。当使用紧凑(Packed)布局时，是不能保证这个布局在每个程序中保持不变的（即非共享），因为它允许编译器去将 uniform 变量从 Uniform 块中优化掉，这在每个着色器中都可能是不同的。

#### 构造完整的 Uniform 缓冲

我们已经讨论了如何在着色器中定义 Uniform 块，并设定它们的内存布局了，但我们还没有讨论该如何使用它们。

每当我们需要对缓冲更新或者插入数据，我们应该使用 glBufferSubData 来更新 ubo。我们只需要更新这个 Uniform 缓冲一次，所有使用这个缓冲的着色器就都使用的是更新后的数据了。但是，如何才能让 OpenGL 知道哪个 Uniform 缓冲对应的是哪个 Uniform 块呢？

在 OpenGL 上下文中，定义了一些绑定点(Binding Point)，我们可以将一个 Uniform 缓冲链接至它。在创建 Uniform 缓冲之后，我们将它绑定到其中一个绑定点上，并将着色器中的 Uniform 块绑定到相同的绑定点，把它们连接到一起。下面的这个图示展示了这个：

![img](https://learnopengl-cn.github.io/img/04/08/advanced_glsl_binding_points.png)

你可以看到，我们可以绑定多个 Uniform 缓冲到不同的绑定点上。因为着色器 A 和着色器 B 都有一个链接到绑定点 0 的 Uniform 块，它们的 Uniform 块将会共享相同的 uniform 数据，uboMatrices，前提条件是两个着色器都定义了相同的 Matrices Uniform 块。

> 这和我们的 Texture 的绑定是相通的

为了将 Uniform 块绑定到一个特定的绑定点中，我们需要调用 `glUniformBlockBinding` 函数，它的第一个参数是一个程序对象，之后是一个 Uniform 块索引和链接到的绑定点。Uniform 块索引(Uniform Block Index)是着色器中已定义 Uniform 块的位置值索引。这可以通过调用 `glGetUniformBlockIndex` 来获取，它接受一个程序对象和 Uniform 块的名称。我们可以用以下方式将图示中的 Lights Uniform 块链接到绑定点 2：

```c++
unsigned int lights_index = glGetUniformBlockIndex(shaderA.ID, "Lights");
glUniformBlockBinding(shaderA.ID, lights_index, 2);
```

注意我们需要对**每个**着色器重复这一步骤。

> 从 OpenGL 4.2 版本起，你也可以添加一个布局标识符，显式地将 Uniform 块的绑定点储存在着色器中，这样就不用再调用 glGetUniformBlockIndex 和 glUniformBlockBinding 了。下面的代码显式地设置了 Lights Uniform 块的绑定点。
>
> ```opengl
> layout(std140, binding = 2) uniform Lights { ... };
> ```

接下来，我们还需要绑定 Uniform 缓冲对象到相同的绑定点上，这可以使用 `glBindBufferBase` 或 `glBindBufferRange` 来完成。

```c++
glBindBufferBase(GL_UNIFORM_BUFFER, 2, uboExampleBlock);
// 或
glBindBufferRange(GL_UNIFORM_BUFFER, 2, uboExampleBlock, 0, 152);
```

- `glBindbufferBase` 需要一个目标，一个绑定点索引和一个 Uniform 缓冲对象作为它的参数。这个函数将 uboExampleBlock 链接到绑定点 2 上，自此，绑定点的两端都链接上了。
- `glBindBufferRange` 需要一个附加的偏移量和大小参数，这样子你可以*绑定 Uniform 缓冲的特定一部分到绑定点中*。通过使用 `glBindBufferRange` 函数，你可以让多个不同的 Uniform 块绑定到同一个 Uniform 缓冲对象上。

现在，所有的东西都配置完毕了，我们可以开始向 Uniform 缓冲中添加数据了。只要我们需要，就可以使用 `glBufferSubData` 函数，用一个字节数组添加所有的数据，或者更新缓冲的一部分。要想更新 uniform 变量 boolean，我们可以用以下方式更新 Uniform 缓冲对象：

```c++
glBindBuffer(GL_UNIFORM_BUFFER, uboExampleBlock);
int b = true; // GLSL中的bool是4字节的，所以我们将它存为一个integer
glBufferSubData(GL_UNIFORM_BUFFER, 144, 4, &b);
glBindBuffer(GL_UNIFORM_BUFFER, 0);
```

同样的步骤也能应用到 Uniform 块中其它的 uniform 变量上，但需要使用不同的范围参数。

#### 一个简单的例子

所以，我们来展示一个真正使用 Uniform 缓冲对象的例子。如果我们回头看看之前所有的代码例子，我们不断地在使用 3 个矩阵：投影、观察和模型矩阵。在所有的这些矩阵中，只有模型矩阵会频繁变动。如果我们有多个着色器使用了这同一组矩阵，那么使用 Uniform 缓冲对象可能会更好。

我们会将投影和模型矩阵存储到一个叫做 Matrices 的 Uniform 块中。我们不会将模型矩阵存在这里，因为模型矩阵在不同的着色器中会不断改变，所以使用 Uniform 缓冲对象并不会带来什么好处。

```c++
#version 330 core
layout (location = 0) in vec3 aPos;

layout (std140) uniform Matrices {
    mat4 view;
    mat4 projection;
};
uniform mat4 model;

void main() {
    gl_Position = projection * view * model * vec4(aPos, 1.0);
}
```

这里没什么特别的，除了我们现在使用的是一个 std140 布局的 Uniform 块。我们将在例子程序中，显示 4 个立方体，每个立方体都是使用不同的着色器程序渲染的。这 4 个着色器程序将使用相同的顶点着色器，但使用的是不同的片段着色器，每个着色器会输出不同的颜色。

首先，我们将顶点着色器的 Uniform 块设置为绑定点 0。_注意我们需要对每个着色器都设置一遍。_

```c++
unsigned int uniformBlockIndexRed    = glGetUniformBlockIndex(shaderRed.ID, "Matrices");
unsigned int uniformBlockIndexGreen  = glGetUniformBlockIndex(shaderGreen.ID, "Matrices");
unsigned int uniformBlockIndexBlue   = glGetUniformBlockIndex(shaderBlue.ID, "Matrices");
unsigned int uniformBlockIndexYellow = glGetUniformBlockIndex(shaderYellow.ID, "Matrices");

glUniformBlockBinding(shaderRed.ID,    uniformBlockIndexRed, 0);
glUniformBlockBinding(shaderGreen.ID,  uniformBlockIndexGreen, 0);
glUniformBlockBinding(shaderBlue.ID,   uniformBlockIndexBlue, 0);
glUniformBlockBinding(shaderYellow.ID, uniformBlockIndexYellow, 0);
```

接下来，我们创建 Uniform 缓冲对象本身，并将其绑定到绑定点 0：

```c++
unsigned int uboMatrices
glGenBuffers(1, &uboMatrices);

glBindBuffer(GL_UNIFORM_BUFFER, uboMatrices);
glBufferData(GL_UNIFORM_BUFFER, 2 * sizeof(glm::mat4), NULL, GL_STATIC_DRAW);
glBindBuffer(GL_UNIFORM_BUFFER, 0);

glBindBufferRange(GL_UNIFORM_BUFFER, 0, uboMatrices, 0, 2 * sizeof(glm::mat4));
```

首先我们为缓冲分配了足够的内存，它等于 glm::mat4 大小的两倍。GLM 矩阵类型的大小直接对应于 GLSL 中的 mat4。接下来，我们将缓冲中的特定范围（在这里是整个缓冲）链接到绑定点 0。

剩余的就是填充这个缓冲了。因为我们已经为缓冲对象分配了足够的内存，我们可以使用 `glBufferSubData` 来直接在 Render Loop 中去输入我们的矩阵：

```c++
glm::mat4 view = camera.GetViewMatrix();
glm::mat4 projection = glm::perspective(glm::radians(45.0f), (float)width/(float)height, 0.1f, 100.0f);
glBindBuffer(GL_UNIFORM_BUFFER, uboMatrices);
glBufferSubData(GL_UNIFORM_BUFFER, 0, sizeof(glm::mat4), glm::value_ptr(view)); // 先后顺序一定要与Shader中的保持一致
glBufferSubData(GL_UNIFORM_BUFFER, sizeof(glm::mat4), sizeof(glm::mat4), glm::value_ptr(projection));
glBindBuffer(GL_UNIFORM_BUFFER, 0);
```

Uniform 缓冲对象的部分就结束了。每个包含了 Matrices 这个 Uniform 块的顶点着色器将会包含储存在 uboMatrices 中的数据。所以，如果我们现在要用 4 个不同的着色器绘制 4 个立方体，它们的投影和观察矩阵都会是一样的。

```c++
glBindVertexArray(cubeVAO);
shaderRed.use();
glm::mat4 model;
model = glm::translate(model, glm::vec3(-0.75f, 0.75f, 0.0f));  // 移动到左上角
shaderRed.setMat4("model", model);
glDrawArrays(GL_TRIANGLES, 0, 36);
// ... 绘制绿色立方体
// ... 绘制蓝色立方体
// ... 绘制黄色立方体
```

唯一需要设置的 uniform 只剩 model uniform 了。在像这样的场景中使用 Uniform 缓冲对象会让我们在每个着色器中都剩下一些 uniform 调用。最终的结果会是这样的：

![img](https://learnopengl-cn.github.io/img/04/08/advanced_glsl_uniform_buffer_objects.png)

因为修改了模型矩阵，每个立方体都移动到了窗口的一边，并且由于使用了不同的片段着色器，它们的颜色也不同。这只是一个很简单的情景，我们可能会需要使用 Uniform 缓冲对象，但任何大型的渲染程序都可能同时激活有上百个着色器程序，这时候 Uniform 缓冲对象的优势就会很大地体现出来了。

Uniform 缓冲对象比起独立的 uniform 有很多好处:

- 第一，一次设置很多 uniform 会比一个一个设置多个 uniform 要快很多。
- 第二，比起在多个着色器中修改同样的 uniform，在 Uniform 缓冲中修改一次会更容易一些。
- 最后一个好处可能不会立即显现，如果使用 Uniform 缓冲对象的话，你可以在着色器中使用更多的 uniform。OpenGL 限制了它能够处理的 uniform 数量，这可以通过 GL_MAX_VERTEX_UNIFORM_COMPONENTS 来查询。当使用 Uniform 缓冲对象时，最大的数量会更高。所以，当你达到了 uniform 的最大数量时（比如再做骨骼动画(Skeletal Animation)的时候），你总是可以选择使用 Uniform 缓冲对象。

## 几何着色器

在顶点和片段着色器之间有一个可选的几何着色器(Geometry Shader)，几何着色器的输入是一个图元（如点或三角形）的一组顶点。几何着色器可以在顶点发送到下一着色器阶段之前对它们随意变换。然而，几何着色器最有趣的地方在于，它能够将（这一组）顶点变换为完全不同的图元，并且还能生成比原来更多的顶点。

> 如果你不清楚几何着色器处于管线中的什么位置，你可以回到最上方

先看一个几何着色器的例子：

```c++
#version 330 core
layout (points) in;
layout (line_strip, max_vertices = 2) out;

void main() {
    gl_Position = gl_in[0].gl_Position + vec4(-0.1, 0.0, 0.0, 0.0);
    EmitVertex();

    gl_Position = gl_in[0].gl_Position + vec4( 0.1, 0.0, 0.0, 0.0);
    EmitVertex();

    EndPrimitive();
}
```

在几何着色器的顶部，我们需要声明：

- 从顶点着色器输入的图元类型。这需要在 in 关键字前声明一个布局修饰符(Layout Qualifier)。这个输入布局修饰符可以从顶点着色器接收下列任何一个图元值：（括号内的数字表示的是一个图元所包含的最小顶点数）

  - `points`：绘制 GL_POINTS 图元（1）

  - `lines`：绘制 GL_LINES 或 GL_LINE_STRIP 图元（2）

  - `lines_adjacency`：绘制 GL_LINES_ADJACENCY 或 GL_LINE_STRIP_ADJACENCY 图元（4）

  - `triangles`：绘制 GL_TRIANGLES、GL_TRIANGLE_STRIP 或 GL_TRIANGLE_FAN 图元（3）

  - `triangles_adjacency`：绘制 GL_TRIANGLES_ADJACENCY 或 GL_TRIANGLE_STRIP_ADJACENCY 图元（6）

    > 以上是能提供给 glDrawArrays 渲染函数的几乎所有图元了。如果我们想要将顶点绘制为 GL_TRIANGLES，我们就要将输入修饰符设置为`triangles`。

- 接下来，我们还需要指定几何着色器输出的图元类型，这需要在 out 关键字前面加一个布局修饰符。和输入布局修饰符一样，输出布局修饰符也可以接受几个图元值：

  - `points`

  - `line_strip`

  - `triangle_strip`

有了这 3 个输出修饰符，我们就可以使用输入图元创建几乎任意的形状了。要生成一个三角形的话，我们将输出定义为`triangle_strip`，并输出 3 个顶点。

几何着色器同时希望我们设置一个它最大能够输出的顶点数量（如果你超过了这个值，OpenGL 将不会绘制**多出的**顶点），这个也可以在 out 关键字的布局修饰符中设置。在这个例子中，我们将输出一个`line_strip`，并将最大顶点数设置为 2 个。

如果你不知道什么是线条(Line Strip)：线条连接了一组点，形成一条连续的线，它最少要由两个点来组成。在渲染函数中每多加一个点，就会在这个点与前一个点之间形成一条新的线。在下面这张图中，我们有 5 个顶点：

![img](https://learnopengl-cn.github.io/img/04/09/geometry_shader_line_strip.png)

如果使用的是上面定义的着色器，那么这将只能输出一条线段，因为最大顶点数等于 2。

为了生成更有意义的结果，我们需要某种方式来获取前一着色器阶段的输出。GLSL 提供给我们一个内建(Built-in)变量，在内部看起来（可能）是这样的：

```c++
in gl_Vertex {
    vec4  gl_Position;
    float gl_PointSize;
    float gl_ClipDistance[];
} gl_in[];
```

这里，它被声明为一个接口块，它包含了几个很有意思的变量，其中最有趣的一个是 gl_Position，它是和顶点着色器输出非常相似的一个向量。

要注意的是，它被声明为一个数组，因为大多数的渲染图元包含多于 1 个的顶点，而几何着色器的输入是一个图元的**所有**顶点。

有了之前顶点着色器阶段的顶点数据，我们就可以使用 2 个几何着色器函数，EmitVertex 和 EndPrimitive，来生成新的数据了。几何着色器希望你能够生成并输出至少一个定义为输出的图元。在我们的例子中，我们需要至少生成一个线条图元。

```c++
void main() {
    gl_Position = gl_in[0].gl_Position + vec4(-0.1, 0.0, 0.0, 0.0);
    EmitVertex();

    gl_Position = gl_in[0].gl_Position + vec4( 0.1, 0.0, 0.0, 0.0);
    EmitVertex();

    EndPrimitive();
}
```

每次我们调用 EmitVertex 时，gl_Position 中的向量会被添加到图元中来。当 EndPrimitive 被调用时，所有发射出的(Emitted)顶点都会合成为指定的输出渲染图元。在一个或多个 EmitVertex 调用之后重复调用 EndPrimitive 能够生成多个图元。在这个例子中，我们发射了两个顶点，它们从原始顶点位置平移了一段距离，之后调用了 EndPrimitive，将这两个顶点合成为一个包含两个顶点的线条。

现在你（大概）了解了几何着色器的工作方式，你可能已经猜出这个几何着色器是做什么的了。它接受一个点图元作为输入，以这个点为中心，创建一条水平的线图元。如果我们渲染它，看起来会是这样的：

![img](https://learnopengl-cn.github.io/img/04/09/geometry_shader_lines.png)

目前还并没有什么令人惊叹的效果，但考虑到这个输出是通过调用下面的渲染函数来生成的，它还是很有意思的：

```c++
glDrawArrays(GL_POINTS, 0, 4);
```

虽然这是一个比较简单的例子，它的确向你展示了如何能够使用几何着色器来（动态地）生成新的形状。在之后我们会利用几何着色器创建出更有意思的效果，但现在我们仍将从创建一个简单的几何着色器开始。

### 使用几何着色器

为了展示几何着色器的用法，我们将会渲染一个非常简单的场景，我们只会在标准化设备坐标的 z 平面上绘制四个点。这些点的坐标是：

```c++
float points[] = {
    -0.5f,  0.5f, // 左上
     0.5f,  0.5f, // 右上
     0.5f, -0.5f, // 右下
    -0.5f, -0.5f  // 左下
};
```

顶点着色器只需要在 z 平面绘制点就可以了，所以我们将使用一个最基本顶点着色器：

```c++
#version 330 core
layout (location = 0) in vec2 aPos;

void main() {
    gl_Position = vec4(aPos.x, aPos.y, 0.0, 1.0);
}
```

直接在片段着色器中硬编码，将所有的点都输出为绿色：

```c++
#version 330 core
out vec4 FragColor;

void main() {
    FragColor = vec4(0.0, 1.0, 0.0, 1.0);
}
```

为点的顶点数据生成一个 VAO 和一个 VBO，然后使用 glDrawArrays 进行绘制：

```c++
shader.use();
glBindVertexArray(VAO);
glDrawArrays(GL_POINTS, 0, 4);
```

结果是在黑暗的场景中有四个（很难看见的）绿点：

![img](https://learnopengl-cn.github.io/img/04/09/geometry_shader_points.png)

但我们之前不是学过这些吗？是的，但是现在我们将会添加一个几何着色器，为场景添加活力。

出于学习目的，我们将会创建一个传递(Pass-through)几何着色器，它会接收一个点图元，并直接将它**传递**(Pass)到下一个着色器：

```c++
#version 330 core
layout (points) in;
layout (points, max_vertices = 1) out;

void main() {
    gl_Position = gl_in[0].gl_Position;
    EmitVertex();
    EndPrimitive();
}
```

现在这个几何着色器应该很容易理解了，它只是将它接收到的顶点位置不作修改直接发射出去，并生成一个点图元。

和顶点与片段着色器一样，几何着色器也需要编译和链接，但这次在创建着色器时我们将会使用 GL_GEOMETRY_SHADER 作为着色器类型：

```c++
geometryShader = glCreateShader(GL_GEOMETRY_SHADER);
glShaderSource(geometryShader, 1, &gShaderCode, NULL);
glCompileShader(geometryShader);
...
glAttachShader(program, geometryShader);
glLinkProgram(program);
```

着色器编译的代码和顶点与片段着色器代码都是一样的。记得要检查编译和链接错误！

如果你现在编译并运行程序，会看到和下面类似的结果：

![img](https://learnopengl-cn.github.io/img/04/09/geometry_shader_points.png)

这和没使用几何着色器时是完全一样的！我承认这是有点无聊，但既然我们仍然能够绘制这些点，所以几何着色器是正常工作的，现在是时候做点更有趣的东西了！

### 造几个房子

绘制点和线并没有那么有趣，所以我们会使用一点创造力，利用几何着色器在每个点的位置上绘制一个房子。要实现这个，我们可以将几何着色器的输出设置为 triangle_strip，并绘制三个三角形：其中两个组成一个正方形，另一个用作房顶。

OpenGL 中，三角形带(Triangle Strip)是绘制三角形更高效的方式，它使用顶点更少。在第一个三角形绘制完之后，每个后续顶点将会在上一个三角形边上生成另一个三角形：每 3 个临近的顶点将会形成一个三角形。如果我们一共有 6 个构成三角形带的顶点，那么我们会得到这些三角形：(1, 2, 3)、(2, 3, 4)、(3, 4, 5)和(4, 5, 6)，共形成 4 个三角形。一个三角形带至少需要 3 个顶点，并会生成 N-2 个三角形。使用 6 个顶点，我们创建了 6-2 = 4 个三角形。下面这幅图展示了这点：

![img](https://learnopengl-cn.github.io/img/04/09/geometry_shader_triangle_strip.png)

通过使用三角形带作为几何着色器的输出，我们可以很容易创建出需要的房子形状，只需要以正确的顺序生成 3 个相连的三角形就行了。下面这幅图展示了顶点绘制的顺序，蓝点代表的是输入点：

![img](https://learnopengl-cn.github.io/img/04/09/geometry_shader_house.png)

变为几何着色器是这样的：

```c++
#version 330 core
layout (points) in;
layout (triangle_strip, max_vertices = 5) out;

void build_house(vec4 position) {
    gl_Position = position + vec4(-0.2, -0.2, 0.0, 0.0);    // 1:左下
    EmitVertex();
    gl_Position = position + vec4( 0.2, -0.2, 0.0, 0.0);    // 2:右下
    EmitVertex();
    gl_Position = position + vec4(-0.2,  0.2, 0.0, 0.0);    // 3:左上
    EmitVertex();
    gl_Position = position + vec4( 0.2,  0.2, 0.0, 0.0);    // 4:右上
    EmitVertex();
    gl_Position = position + vec4( 0.0,  0.4, 0.0, 0.0);    // 5:顶部
    EmitVertex();
    EndPrimitive();
}

void main() {
    build_house(gl_in[0].gl_Position);
}
```

这个几何着色器生成了 5 个顶点，每个顶点都是原始点的位置加上一个偏移量，来组成一个大的三角形带。最终的图元会被光栅化，然后片段着色器会处理整个三角形带，最终在每个绘制的点处生成一个绿色房子：

![img](https://learnopengl-cn.github.io/img/04/09/geometry_shader_houses.png)

你可以看到，每个房子实际上是由 3 个三角形组成的——全部都是使用空间中一点来绘制的。这些绿房子看起来是有点无聊，所以我们会再给每个房子分配一个不同的颜色。为了实现这个，我们需要在顶点着色器中添加一个额外的顶点属性，表示颜色信息，将它传递至几何着色器，并再次发送到片段着色器中。

下面是更新后的顶点数据：

```c++
float points[] = {
    -0.5f,  0.5f, 1.0f, 0.0f, 0.0f, // 左上
     0.5f,  0.5f, 0.0f, 1.0f, 0.0f, // 右上
     0.5f, -0.5f, 0.0f, 0.0f, 1.0f, // 右下
    -0.5f, -0.5f, 1.0f, 1.0f, 0.0f  // 左下
};
```

然后我们更新顶点着色器，使用一个接口块将颜色属性发送到几何着色器中：

```c++
#version 330 core
layout (location = 0) in vec2 aPos;
layout (location = 1) in vec3 aColor;

out VS_OUT {
    vec3 color;
} vs_out;

void main()
{
    gl_Position = vec4(aPos.x, aPos.y, 0.0, 1.0);
    vs_out.color = aColor;
}
```

接下来我们还需要在几何着色器中声明相同的接口块（使用一个不同的接口名）：

```c++
in VS_OUT {
    vec3 color;
} gs_in[];
```

因为几何着色器是作用于输入的一组顶点的，从顶点着色器发来输入数据总是会以数组的形式表示出来，即便我们现在只有一个顶点。

我们并不是必须要用接口块来向几何着色器传递数据。如果顶点着色器发送的颜色向量是`out vec3 vColor`，我们也可以这样写：

```
in vec3 vColor[];
```

然而，接口块在几何着色器这样的着色器中会更容易处理一点。实际上，几何着色器的输入能够变得非常大，将它们合并为一个大的接口块数组会更符合逻辑一点。

接下来我们还需要为下个片段着色器阶段声明一个输出颜色向量：

```c++
out vec3 fColor;
```

因为片段着色器只需要一个（插值的）颜色，发送多个颜色并没有什么意义。所以，fColor 向量就不是一个数组，而是一个单独的向量。当发射一个顶点的时候，每个顶点将会使用最后在 fColor 中储存的值，来用于片段着色器的运行。对我们的房子来说，我们只需要在第一个顶点发射之前，使用顶点着色器中的颜色填充 fColor 一次就可以了。

```c++
fColor = gs_in[0].color; // gs_in[0] 因为只有一个输入顶点
gl_Position = position + vec4(-0.2, -0.2, 0.0, 0.0);    // 1:左下
EmitVertex();
gl_Position = position + vec4( 0.2, -0.2, 0.0, 0.0);    // 2:右下
EmitVertex();
gl_Position = position + vec4(-0.2,  0.2, 0.0, 0.0);    // 3:左上
EmitVertex();
gl_Position = position + vec4( 0.2,  0.2, 0.0, 0.0);    // 4:右上
EmitVertex();
gl_Position = position + vec4( 0.0,  0.4, 0.0, 0.0);    // 5:顶部
EmitVertex();
EndPrimitive();
```

所有发射出的顶点都将嵌有最后储存在 fColor 中的值，即顶点的颜色属性值。所有的房子都会有它们自己的颜色了：

![img](https://learnopengl-cn.github.io/img/04/09/geometry_shader_houses_colored.png)

仅仅是为了有趣，我们也可以假装这是冬天，将最后一个顶点的颜色设置为白色，给屋顶落上一些雪。

```c++
fColor = gs_in[0].color;
gl_Position = position + vec4(-0.2, -0.2, 0.0, 0.0);    // 1:左下
EmitVertex();
gl_Position = position + vec4( 0.2, -0.2, 0.0, 0.0);    // 2:右下
EmitVertex();
gl_Position = position + vec4(-0.2,  0.2, 0.0, 0.0);    // 3:左上
EmitVertex();
gl_Position = position + vec4( 0.2,  0.2, 0.0, 0.0);    // 4:右上
EmitVertex();
gl_Position = position + vec4( 0.0,  0.4, 0.0, 0.0);    // 5:顶部
fColor = vec3(1.0, 1.0, 1.0);
EmitVertex();
EndPrimitive();
```

最终结果看起来是这样的：

![img](https://learnopengl-cn.github.io/img/04/09/geometry_shader_houses_snow.png)

你可以将你的代码与[这里](https://learnopengl.com/code_viewer_gh.php?code=src/4.advanced_opengl/9.1.geometry_shader_houses/geometry_shader_houses.cpp)的 OpenGL 代码进行比对。

你可以看到，有了几何着色器，你甚至可以将最简单的图元变得十分有创意。因为这些形状是在 GPU 的超快硬件中动态生成的，这会比在顶点缓冲中手动定义图形要高效很多。因此，几何缓冲对简单而且经常重复的形状来说是一个很好的优化工具，比如体素(Voxel)世界中的方块和室外草地的每一根草。

### 法向量可视化

在这一部分中，我们将使用几何着色器来实现一个真正有用的例子：显示任意物体的法向量。当编写光照着色器时，你可能会最终会得到一些奇怪的视觉输出，但又很难确定导致问题的原因。光照错误很常见的原因就是法向量错误，这可能是由于不正确加载顶点数据、错误地将它们定义为顶点属性或在着色器中不正确地管理所导致的。我们想要的是使用某种方式来检测提供的法向量是正确的。检测法向量是否正确的一个很好的方式就是对它们进行可视化，几何着色器正是实现这一目的非常有用的工具。

思路是这样的：我们首先不使用几何着色器正常绘制场景。然后再次绘制场景，但这次只显示通过几何着色器生成法向量。几何着色器接收一个三角形图元，并沿着法向量生成三条线——每个顶点一个法向量。伪代码看起来会像是这样：

```c++
shader.use();
DrawScene();
normalDisplayShader.use();
DrawScene();
```

这次在几何着色器中，我们会使用模型提供的顶点法线，而不是自己生成，为了适配（观察和模型矩阵的）缩放和旋转，我们在将法线变换到观察空间坐标之前，先使用法线矩阵变换一次（几何着色器接受的位置向量是观察空间坐标，所以我们应该将法向量变换到相同的空间中）。这可以在顶点着色器中完成：

```c++
#version 330 core
layout (location = 0) in vec3 aPos;
layout (location = 1) in vec3 aNormal;

out VS_OUT {
    vec3 normal;
} vs_out;

uniform mat4 view;
uniform mat4 model;

void main()
{
    gl_Position = view * model * vec4(aPos, 1.0);
    mat3 normalMatrix = mat3(transpose(inverse(view * model)));
    vs_out.normal = normalize(vec3(vec4(normalMatrix * aNormal, 0.0)));
}
```

变换后的观察空间法向量会以接口块的形式传递到下个着色器阶段。接下来，几何着色器会接收每一个顶点（包括一个位置向量和一个法向量），并在每个位置向量处绘制一个法线向量：

```c++
#version 330 core
layout (triangles) in;
layout (line_strip, max_vertices = 6) out;

in VS_OUT {
    vec3 normal;
} gs_in[];

const float MAGNITUDE = 0.4;

uniform mat4 projection;

void GenerateLine(int index)
{
    gl_Position = projection * gl_in[index].gl_Position;
    EmitVertex();
    gl_Position = projection * (gl_in[index].gl_Position +
                                vec4(gs_in[index].normal, 0.0) * MAGNITUDE);
    EmitVertex();
    EndPrimitive();
}

void main()
{
    GenerateLine(0); // 第一个顶点法线
    GenerateLine(1); // 第二个顶点法线
    GenerateLine(2); // 第三个顶点法线
}
```

像这样的几何着色器应该很容易理解了。注意我们将法向量乘以了一个 MAGNITUDE 向量，来限制显示出的法向量大小（否则它们就有点大了）。

因为法线的可视化通常都是用于调试目的，我们可以使用片段着色器，将它们显示为单色的线（如果你愿意也可以是非常好看的线）：

```c++
#version 330 core
out vec4 FragColor;

void main()
{
    FragColor = vec4(1.0, 1.0, 0.0, 1.0);
}
```

现在，首先使用普通着色器渲染模型，再使用特别的**法线可视化**着色器渲染，你将看到这样的效果：

![img](https://learnopengl-cn.github.io/img/04/09/geometry_shader_normals.png)

尽管我们的纳米装现在看起来像是一个体毛很多而且带着隔热手套的人，它能够很有效地帮助我们判断模型的法线是否正确。你可以想象到，这样的几何着色器也经常用于给物体添加毛发(Fur)。

> 你可能会对 gl_position 开始出现疑问：这到底是不是一个类似于 SV_POSITION 的变量？嗯，实际上应该是的，但这里的问题其实出在，我们没必要一定在 Vertex Shader 阶段就将这个变量变换到 Clip Space 上，我们完全可以保持它不变，然后再于 Geometry Shader 中进行变换——这完全是可行的。并且我们得建立这个认知：几何着色器中的 gl_Position 是从顶点着色器中传过来的。也就是说顶点着色器中算出来的顶点是什么空间的，那么几何着色器中就是哪个空间的。

## 实例化

> 假设你有一个绘制了很多模型的场景，而大部分的模型包含的是同一组顶点数据，只不过进行的是不同的世界空间变换。想象一个充满草的场景：每根草都是一个包含几个三角形的小模型。你可能会需要绘制很多根草，最终在每帧中你可能会需要渲染上千或者上万根草。因为每一根草仅仅是由几个三角形构成，渲染几乎是瞬间完成的，但上千个渲染函数调用却会极大地影响性能。

如果我们需要渲染大量物体时，代码看起来会像这样：

```c++
for(unsigned int i = 0; i < amount_of_models_to_draw; i++) {
    DoSomePreparations(); // 绑定VAO，绑定纹理，设置uniform等
    glDrawArrays(GL_TRIANGLES, 0, amount_of_vertices);
}
```

如果像这样绘制模型的大量实例(Instance)，你很快就会因为绘制调用过多而达到性能瓶颈。

与绘制顶点本身相比，使用 `glDrawArrays` 或 `glDrawElements` 函数告诉 GPU 去绘制你的顶点数据会消耗更多的性能，因为 OpenGL 在绘制顶点数据之前需要做很多准备工作（比如告诉 GPU 该从哪个缓冲读取数据，从哪寻找顶点属性，而且这些都是在相对缓慢的 CPU 到 GPU 总线(CPU to GPU Bus)上进行的）。所以，即便渲染顶点非常快，命令 GPU 去渲染却未必。

**如果我们能够将数据一次性发送给 GPU，然后使用一个绘制函数让 OpenGL 利用这些数据绘制多个物体，就会更方便了。这就是实例化(Instancing)。**

实例化这项技术能够让我们使用一个渲染调用来绘制多个物体，来节省每次绘制物体时 CPU -> GPU 的通信，它只需要一次即可。如果想使用实例化渲染，我们只需要将 `glDrawArrays` 和 `glDrawElements` 的渲染调用分别改为 `glDrawArraysInstanced` 和 `glDrawElementsInstanced` 就可以了。这些渲染函数的**实例化**版本需要一个额外的参数，叫做实例数量(Instance Count)，它能够设置我们需要渲染的实例个数。这样我们只需要将必须的数据发送到 GPU 一次，然后使用一次函数调用告诉 GPU 它应该如何绘制这些实例。GPU 将会直接渲染这些实例，而不用不断地与 CPU 进行通信。

这个函数本身并没有什么用。渲染同一个物体一千次对我们并没有什么用处，每个物体都是完全相同的，而且还在同一个位置。我们只能看见一个物体！处于这个原因，GLSL 在顶点着色器中嵌入了另一个内建变量，gl_InstanceID。

**在使用实例化渲染调用时**，gl_InstanceID 会从 0 开始，在每个实例被渲染时递增 1。比如说，我们正在渲染第 43 个实例，那么顶点着色器中它的 gl_InstanceID 将会是 42。因为每个实例都有唯一的 ID，**我们可以建立一个数组，将 ID 与位置值对应起来，将每个实例放置在世界的不同位置。**

### 如果不用实例化绘制的具体特性

为了体验一下实例化绘制，我们将会在标准化设备坐标系中使用一个渲染调用，绘制 100 个 2D 四边形。我们会索引一个包含 100 个偏移向量的 uniform 数组，将偏移值加到每个实例化的四边形上。最终的结果是一个排列整齐的四边形网格：

![img](https://learnopengl-cn.github.io/img/04/10/instancing_quads.png)

每个四边形由 2 个三角形所组成，一共有 6 个顶点。每个顶点包含一个 2D 的标准化设备坐标位置向量和一个颜色向量。 下面就是这个例子使用的顶点数据，为了大量填充屏幕，每个三角形都很小：

```c++
float quadVertices[] = {
    // 位置          // 颜色
    -0.05f,  0.05f,  1.0f, 0.0f, 0.0f,
     0.05f, -0.05f,  0.0f, 1.0f, 0.0f,
    -0.05f, -0.05f,  0.0f, 0.0f, 1.0f,

    -0.05f,  0.05f,  1.0f, 0.0f, 0.0f,
     0.05f, -0.05f,  0.0f, 1.0f, 0.0f,
     0.05f,  0.05f,  0.0f, 1.0f, 1.0f
};
```

片段着色器会从顶点着色器接受颜色向量，并将其设置为它的颜色输出，来实现四边形的颜色：

```c++
#version 330 core
out vec4 FragColor;

in vec3 fColor;

void main() {
    FragColor = vec4(fColor, 1.0);
}
```

到现在都没有什么新内容，但从顶点着色器开始就变得很有趣了：

```c++
#version 330 core
layout (location = 0) in vec2 aPos;
layout (location = 1) in vec3 aColor;

out vec3 fColor;

uniform vec2 offsets[100];

void main() {
    vec2 offset = offsets[gl_InstanceID];
    gl_Position = vec4(aPos + offset, 0.0, 1.0);
    fColor = aColor;
}
```

这里我们定义了一个叫做 offsets 的数组，它包含 100 个偏移向量。**在顶点着色器中，我们会使用 gl_InstanceID 来索引 offsets 数组，获取每个实例的偏移向量。**如果我们要实例化绘制 100 个四边形，仅使用这个顶点着色器我们就能得到 100 个位于不同位置的四边形。

当前，我们仍要设置这些偏移位置，我们会在进入渲染循环之前使用一个嵌套 for 循环计算：

```c++
glm::vec2 translations[100];
int index = 0;
float offset = 0.1f;
for(int y = -10; y < 10; y += 2) {
    for(int x = -10; x < 10; x += 2) {
        glm::vec2 translation;
        translation.x = (float)x / 10.0f + offset;
        translation.y = (float)y / 10.0f + offset;
        translations[index++] = translation;
    }
}
```

这里，我们创建 100 个位移向量，表示 10x10 网格上的所有位置。除了生成 translations 数组之外，我们还需要将数据转移到顶点着色器的 uniform 数组中：

```c++
shader.use();
for(unsigned int i = 0; i < 100; i++) {
    stringstream ss;
    string index;
    ss << i;
    index = ss.str();
    shader.setVec2(("offsets[" + index + "]").c_str(), translations[i]);
}
```

在这一段代码中，我们将 for 循环的计数器 i 转换为一个 string，我们可以用它来动态创建位置值的字符串，用于 uniform 位置值的索引。接下来，我们会对 offsets uniform 数组中的每一项设置对应的位移向量。

现在所有的准备工作都做完了，我们可以开始渲染四边形了。对于实例化渲染，我们使用 glDrawArraysInstanced 或 glDrawElementsInstanced。因为我们使用的不是索引缓冲，我们会调用 glDrawArrays 版本的函数：

```c++
glBindVertexArray(quadVAO);
glDrawArraysInstanced(GL_TRIANGLES, 0, 6, 100);
```

glDrawArraysInstanced 的参数和 glDrawArrays 完全一样，除了最后多了个参数用来设置需要绘制的实例数量。因为我们想要在 10x10 网格中显示 100 个四边形，我们将它设置为 100.运行代码之后，你应该能得到熟悉的 100 个五彩的四边形。

### 实例化数组

虽然之前的实现在目前的情况下能够正常工作，但是如果我们要渲染远超过 100 个实例的时候（这其实非常普遍），我们最终会超过最大能够发送至着色器的 uniform 数据大小[上限](<http://www.opengl.org/wiki/Uniform_(GLSL)#Implementation_limits>)。它的一个代替方案是实例化数组(Instanced Array)，它被定义为一个顶点属性（能够让我们储存更多的数据），仅在顶点着色器渲染一个新的实例时才会更新。

使用顶点属性时，顶点着色器的每次运行都会让 GLSL 获取新一组适用于当前顶点的属性。而当我们将顶点属性定义为一个实例化数组时，顶点着色器就只需要对每个实例，而不是每个顶点，更新顶点属性的内容了。这允许我们对逐顶点的数据使用普通的顶点属性，而对逐实例的数据使用实例化数组。

为了给你一个实例化数组的例子，我们将使用之前的例子，并将偏移量 uniform 数组设置为一个实例化数组。我们需要在顶点着色器中再添加一个顶点属性：

```c++
#version 330 core
layout (location = 0) in vec2 aPos;
layout (location = 1) in vec3 aColor;
layout (location = 2) in vec2 aOffset;

out vec3 fColor;

void main() {
    gl_Position = vec4(aPos + aOffset, 0.0, 1.0);
    fColor = aColor;
}
```

我们不再使用 gl_InstanceID，现在不需要索引一个 uniform 数组就能够直接使用 offset 属性了。

因为实例化数组和 position 与 color 变量一样，都是顶点属性，我们还需要将它的内容存在顶点缓冲对象中，并且配置它的属性指针。我们首先将（上一部分的）translations 数组存到一个新的缓冲对象中：

```c++
unsigned int instanceVBO;
glGenBuffers(1, &instanceVBO);
glBindBuffer(GL_ARRAY_BUFFER, instanceVBO);
glBufferData(GL_ARRAY_BUFFER, sizeof(glm::vec2) * 100, &translations[0], GL_STATIC_DRAW);
glBindBuffer(GL_ARRAY_BUFFER, 0);
```

之后我们还需要设置它的顶点属性指针，并启用顶点属性：

```c++
glEnableVertexAttribArray(2);
glBindBuffer(GL_ARRAY_BUFFER, instanceVBO);
glVertexAttribPointer(2, 2, GL_FLOAT, GL_FALSE, 2 * sizeof(float), (void*)0);
glBindBuffer(GL_ARRAY_BUFFER, 0);
glVertexAttribDivisor(2, 1);
```

这段代码很有意思的地方在于最后一行，我们调用了 `glVertexAttribDivisor`。这个函数告诉了 OpenGL 该**什么时候**更新顶点属性的内容至新一组数据。它的第一个参数是需要的顶点属性，第二个参数是属性除数(Attribute Divisor)。**默认情况下，属性除数是 0，告诉 OpenGL 我们需要在顶点着色器的*每次迭代时*更新顶点属性。**将它设置为 1 时，我们告诉 OpenGL 我们希望在渲染一个新实例的时候更新顶点属性。而设置为 2 时，我们希望每 2 个实例更新一次属性，以此类推。我们将属性除数设置为 1，是在告诉 OpenGL，处于位置值 2 的顶点属性是一个实例化数组。

> 你可能会对这里的 `glVertexAttribDivisor` 感到疑惑，我们输入的顶点属性不是和顶点一致的吗？怎么又谈到更新的事情？
>
> 让我们仔细对比这里的具体内容：
>
> 对于普通的顶点属性，我们又类似下方的设计：
>
> ```
> float quadVertices[] = {
>  // 位置          // 颜色
>  -0.05f,  0.05f,  1.0f, 0.0f, 0.0f,
>   0.05f, -0.05f,  0.0f, 1.0f, 0.0f,
>  -0.05f, -0.05f,  0.0f, 0.0f, 1.0f,
>
>  -0.05f,  0.05f,  1.0f, 0.0f, 0.0f,
>   0.05f, -0.05f,  0.0f, 1.0f, 0.0f,
>   0.05f,  0.05f,  0.0f, 1.0f, 1.0f
> };
> ```
>
> 可以发现的是，对于普通的顶点属性，首先它的数量是和顶点完全一致的并且是逐顶点细化的，这让我们可以控制每个顶点的渲染效果。
>
> 而对于我们现在新用到的顶点属性的设计，则不是以逐顶点为目的的，这一点从数量上就能反映出来：
>
> ```C++
> glm::vec2 translations[100];
> ```
>
> 这 100 个内容，不是为某个顶点服务的，而是为我们将要先后渲染的 100 的 Instance 服务的。而同时，它还有另一个特点：就是一个模型中，每个顶点所用到的数据都是相同的。也就是说，这种新式的顶点属性的目的是逐模型的，而显而易见的，默认的顶点属性指针是为前者而设计的——它只会在单个模型渲染过程中，依照对应的目的进行移动。而为了达成逐模型渲染的目的，我们必须手动拨动指针进行位移——这就是为什么我们需要修改 `glVertexAttribDivisor`

如果我们现在使用 glDrawArraysInstanced，再次渲染四边形，会得到以下输出：

![img](https://learnopengl-cn.github.io/img/04/10/instancing_quads.png)

这和之前的例子是完全一样的，但这次是使用实例化数组实现的，这让我们能够传递更多的数据到顶点着色器（只要内存允许）来用于实例化绘制。

为了更有趣一点，我们也可以使用 gl_InstanceID，从右上到左下逐渐缩小四边形：

```c++
void main() {
    vec2 pos = aPos * (gl_InstanceID / 100.0);
    gl_Position = vec4(pos + aOffset, 0.0, 1.0);
    fColor = aColor;
}
```

结果就是，第一个四边形的实例会非常小，随着绘制实例的增加，gl_InstanceID 会越来越接近 100，四边形也就越来越接近原始大小。像这样将实例化数组与 gl_InstanceID 结合使用是完全可行的。

![img](https://learnopengl-cn.github.io/img/04/10/instancing_quads_arrays.png)

如果你还是不确定实例化渲染是如何工作的，或者想看看所有代码是如何组合起来的，你可以在[这里](https://learnopengl.com/code_viewer_gh.php?code=src/4.advanced_opengl/10.1.instancing_quads/instancing_quads.cpp)找到程序的源代码。

虽然很有趣，但是这些例子并不是实例化的好例子。是的，它们的确让你知道实例化是怎么工作的，但是我们还没接触到它最有用的一点：绘制巨大数量的相似物体。出于这个原因，我们将会在下一部分进入太空探险，见识实例化渲染真正的威力。

### 小行星带

想象这样一个场景，在宇宙中有一个大的行星，它位于小行星带的中央。这样的小行星带可能包含成千上万的岩块，在很不错的显卡上也很难完成这样的渲染。实例化渲染正是适用于这样的场景，因为所有的小行星都可以使用一个模型来表示。每个小行星可以再使用不同的变换矩阵来进行少许的变化。

为了展示实例化渲染的作用，我们首先会**不使用**实例化渲染，来渲染小行星绕着行星飞行的场景。这个场景将会包含一个大的行星模型，它可以在[这里](https://learnopengl-cn.github.io/data/planet.rar)下载，以及很多环绕着行星的小行星。小行星的岩石模型可以在[这里](https://learnopengl-cn.github.io/data/rock.rar)下载。

在代码例子中，我们将使用在[模型加载](https://learnopengl-cn.github.io/03 Model Loading/01 Assimp/)小节中定义的模型加载器来加载模型。

为了得到想要的效果，我们将会为每个小行星生成一个变换矩阵，用作它们的模型矩阵。变换矩阵首先将小行星位移到小行星带中的某处，我们还会加一个小的随机偏移值到这个偏移量上，让这个圆环看起来更自然一点。接下来，我们应用一个随机的缩放，并且以一个旋转向量为轴进行一个随机的旋转。最终的变换矩阵不仅能将小行星变换到行星的周围，而且会让它看起来更自然，与其它小行星不同。最终的结果是一个布满小行星的圆环，其中每一个小行星都与众不同。

```c++
unsigned int amount = 1000;
glm::mat4 *modelMatrices;
modelMatrices = new glm::mat4[amount];
srand(glfwGetTime()); // 初始化随机种子
float radius = 50.0;
float offset = 2.5f;
for(unsigned int i = 0; i < amount; i++) {
    glm::mat4 model;
    // 1. 位移：分布在半径为 'radius' 的圆形上，偏移的范围是 [-offset, offset]
    float angle = (float)i / (float)amount * 360.0f;
    float displacement = (rand() % (int)(2 * offset * 100)) / 100.0f - offset;
    float x = sin(angle) * radius + displacement;
    displacement = (rand() % (int)(2 * offset * 100)) / 100.0f - offset;
    float y = displacement * 0.4f; // 让行星带的高度比x和z的宽度要小
    displacement = (rand() % (int)(2 * offset * 100)) / 100.0f - offset;
    float z = cos(angle) * radius + displacement;
    model = glm::translate(model, glm::vec3(x, y, z));

    // 2. 缩放：在 0.05 和 0.25f 之间缩放
    float scale = (rand() % 20) / 100.0f + 0.05;
    model = glm::scale(model, glm::vec3(scale));

    // 3. 旋转：绕着一个（半）随机选择的旋转轴向量进行随机的旋转
    float rotAngle = (rand() % 360);
    model = glm::rotate(model, rotAngle, glm::vec3(0.4f, 0.6f, 0.8f));

    // 4. 添加到矩阵的数组中
    modelMatrices[i] = model;
}
```

这段代码看起来可能有点吓人，但我们只是将小行星的`x`和`z`位置变换到了一个半径为 radius 的圆形上，并且在半径的基础上偏移了-offset 到 offset。我们让`y`偏移的影响更小一点，让小行星带更扁平一点。接下来，我们应用了缩放和旋转变换，并将最终的变换矩阵储存在 modelMatrices 中，这个数组的大小是 amount。这里，我们一共生成 1000 个模型矩阵，每个小行星一个。

在加载完行星和岩石模型，并编译完着色器之后，渲染的代码看起来是这样的：

```c++
// 绘制行星
shader.use();
glm::mat4 model;
model = glm::translate(model, glm::vec3(0.0f, -3.0f, 0.0f));
model = glm::scale(model, glm::vec3(4.0f, 4.0f, 4.0f));
shader.setMat4("model", model);
planet.Draw(shader);

// 绘制小行星
for(unsigned int i = 0; i < amount; i++) {
    shader.setMat4("model", modelMatrices[i]);
    rock.Draw(shader);
}
```

我们首先绘制了行星的模型，并对它进行位移和缩放，以适应场景，接下来，我们绘制 amount 数量的岩石模型。在绘制每个岩石之前，我们首先需要在着色器内设置对应的模型变换矩阵。

最终的结果是一个看起来像是太空的场景，环绕着行星的是看起来很自然的小行星带：

![img](https://learnopengl-cn.github.io/img/04/10/instancing_asteroids.png)

这个场景每帧包含 1001 次渲染调用，其中 1000 个是岩石模型。你可以在[这里](https://learnopengl.com/code_viewer_gh.php?code=src/4.advanced_opengl/10.2.asteroids/asteroids.cpp)找到源代码。

当我们开始增加这个数字的时候，你很快就会发现场景不再能够流畅运行了，帧数也下降很厉害。当我们将 amount 设置为 2000 的时候，场景就已经慢到移动都很困难的程度了。

现在，我们来尝试使用实例化渲染来渲染相同的场景。我们首先对顶点着色器进行一点修改：

```c++
#version 330 core
layout (location = 0) in vec3 aPos;
layout (location = 2) in vec2 aTexCoords;
layout (location = 3) in mat4 instanceMatrix;

out vec2 TexCoords;

uniform mat4 projection;
uniform mat4 view;

void main() {
    gl_Position = projection * view * instanceMatrix * vec4(aPos, 1.0);
    TexCoords = aTexCoords;
}
```

我们不再使用模型 uniform 变量，改为一个 mat4 的顶点属性，让我们能够存储一个实例化数组的变换矩阵。然而，当我们顶点属性的类型大于 vec4 时，就要多进行一步处理了。顶点属性最大允许的数据大小等于一个 vec4。因为一个 mat4 本质上是 4 个 vec4，我们需要为这个矩阵预留 4 个顶点属性。因为我们将它的位置值设置为 3，矩阵每一列的顶点属性位置值就是 3、4、5 和 6。

接下来，我们需要为这 4 个顶点属性设置属性指针，并将它们设置为实例化数组：

```c++
// 顶点缓冲对象
unsigned int buffer;
glGenBuffers(1, &buffer);
glBindBuffer(GL_ARRAY_BUFFER, buffer);
glBufferData(GL_ARRAY_BUFFER, amount * sizeof(glm::mat4), &modelMatrices[0], GL_STATIC_DRAW);

for(unsigned int i = 0; i < rock.meshes.size(); i++)
{
    unsigned int VAO = rock.meshes[i].VAO;
    glBindVertexArray(VAO);
    // 顶点属性
    GLsizei vec4Size = sizeof(glm::vec4);
    glEnableVertexAttribArray(3);
    glVertexAttribPointer(3, 4, GL_FLOAT, GL_FALSE, 4 * vec4Size, (void*)0);
    glEnableVertexAttribArray(4);
    glVertexAttribPointer(4, 4, GL_FLOAT, GL_FALSE, 4 * vec4Size, (void*)(vec4Size));
    glEnableVertexAttribArray(5);
    glVertexAttribPointer(5, 4, GL_FLOAT, GL_FALSE, 4 * vec4Size, (void*)(2 * vec4Size));
    glEnableVertexAttribArray(6);
    glVertexAttribPointer(6, 4, GL_FLOAT, GL_FALSE, 4 * vec4Size, (void*)(3 * vec4Size));

    glVertexAttribDivisor(3, 1);
    glVertexAttribDivisor(4, 1);
    glVertexAttribDivisor(5, 1);
    glVertexAttribDivisor(6, 1);

    glBindVertexArray(0);
}
```

注意这里我们将 Mesh 的 VAO 从私有变量改为了公有变量，让我们能够访问它的顶点数组对象。这并不是最好的解决方案，只是为了配合本小节的一个简单的改动。除此之外代码就应该很清楚了。我们告诉了 OpenGL 应该如何解释每个缓冲顶点属性的缓冲，并且告诉它这些顶点属性是实例化数组。

接下来，我们再次使用网格的 VAO，这一次使用 glDrawElementsInstanced 进行绘制：

```c++
// 绘制小行星
instanceShader.use();
for(unsigned int i = 0; i < rock.meshes.size(); i++)
{
    glBindVertexArray(rock.meshes[i].VAO);
    glDrawElementsInstanced(
        GL_TRIANGLES, rock.meshes[i].indices.size(), GL_UNSIGNED_INT, 0, amount
    );
}
```

这里，我们绘制与之前相同数量 amount 的小行星，但是使用的是实例渲染。结果应该是非常相似的，但如果你开始增加 amount 变量，你就能看见实例化渲染的效果了。没有实例化渲染的时候，我们只能流畅渲染 1000 到 1500 个小行星。而使用了实例化渲染之后，我们可以将这个值设置为 100000，每个岩石模型有 576 个顶点，每帧加起来大概要绘制 5700 万个顶点，但性能却没有受到任何影响！

![img](https://learnopengl-cn.github.io/img/04/10/instancing_asteroids_quantity.png)

上面这幅图渲染了 10 万个小行星，半径为`150.0f`，偏移量等于`25.0f`。在某些机器上，10 万个小行星可能会太多了，所以尝试修改这个值，直到达到一个你能接受的帧率。

可以看到，在合适的环境下，实例化渲染能够大大增加显卡的渲染能力。正是出于这个原因，实例化渲染通常会用于渲染草、植被、粒子，以及上面这样的场景，基本上只要场景中有很多重复的形状，都能够使用实例化渲染来提高性能。

## 抗锯齿

在学习渲染的旅途中，你可能会时不时遇到模型边缘有锯齿的情况。这些锯齿边缘(Jagged Edges)的产生和光栅器将顶点数据转化为片段的方式有关。在下面的例子中，你可以看到，我们只是绘制了一个简单的立方体，你就能注意到它存在锯齿边缘了：

![img](https://learnopengl-cn.github.io/img/04/11/anti_aliasing_aliasing.png)

可能不是非常明显，但如果你离近仔细观察立方体的边缘，你就应该能够看到锯齿状的图案。如果放大的话，你会看到下面的图案：

![img](https://learnopengl-cn.github.io/img/04/11/anti_aliasing_zoomed.png)

这很明显不是我们想要在最终程序中所实现的效果。你能够清楚看见形成边缘的像素。这种现象被称之为走样(Aliasing)。有很多种抗锯齿（Anti-aliasing，也被称为反走样）的技术能够帮助我们缓解这种现象，从而产生更**平滑**的边缘。

最开始我们有一种叫做超采样抗锯齿(Super Sample Anti-aliasing, SSAA)的技术，它会使用比正常分辨率更高的分辨率（即超采样）来渲染场景，当图像输出在帧缓冲中更新时，分辨率会被下采样(Downsample)至正常的分辨率。这些**额外的**分辨率会被用来防止锯齿边缘的产生。虽然它确实能够解决走样的问题，但是由于这样比平时要绘制更多的片段，它也会带来很大的性能开销。所以这项技术只拥有了短暂的辉煌。

然而，在这项技术的基础上也诞生了更为现代的技术，叫做多重采样抗锯齿(Multisample Anti-aliasing, MSAA)。它借鉴了 SSAA 背后的理念，但却以更加高效的方式实现了抗锯齿。我们在这一节中会深度讨论 OpenGL 中内建的 MSAA 技术。

### 多重采样

为了理解什么是多重采样(Multisampling)，以及它是如何解决锯齿问题的，我们有必要更加深入地了解 OpenGL 光栅器的工作方式。

光栅器是位于最终处理过的顶点之后到片段着色器之前所经过的所有的算法与过程的总和。光栅器会将一个图元的所有顶点作为输入，并将它转换为一系列的片段。顶点坐标理论上可以取任意值，但片段不行，因为它们受限于你窗口的分辨率。顶点坐标与片段之间几乎永远也不会有一对一的映射，所以光栅器必须以某种方式来决定每个顶点最终所在的片段/屏幕坐标。

![img](https://learnopengl-cn.github.io/img/04/11/anti_aliasing_rasterization.png)

这里我们可以看到一个屏幕像素的网格，每个像素的中心包含有一个**采样点(Sample Point)**，它会被用来决定这个三角形是否遮盖了某个像素。图中红色的采样点被三角形所遮盖，在每一个遮住的像素处都会生成一个片段。虽然三角形边缘的一些部分也遮住了某些屏幕像素，但是这些像素的采样点并没有被三角形**内部**所遮盖，所以它们不会受到片段着色器的影响。

你现在可能已经清楚走样的原因了。完整渲染后的三角形在屏幕上会是这样的：

![img](https://learnopengl-cn.github.io/img/04/11/anti_aliasing_rasterization_filled.png)

由于屏幕像素总量的限制，有些边缘的像素能够被渲染出来，而有些则不会。结果就是我们使用了不光滑的边缘来渲染图元，导致之前讨论到的锯齿边缘。

多重采样所做的正是将单一的采样点变为多个采样点（这也是它名称的由来）。我们不再使用像素中心的单一采样点，取而代之的是以特定图案排列的 4 个子采样点(Subsample)。我们将用这些子采样点来决定像素的遮盖度。当然，这也意味着**颜色缓冲的大小会随着子采样点的增加而增加。**

![img](https://learnopengl-cn.github.io/img/04/11/anti_aliasing_sample_points.png)

上图的左侧展示了正常情况下判定三角形是否遮盖的方式。在例子中的这个像素上不会运行片段着色器（所以它会保持空白）。因为它的采样点并未被三角形所覆盖。上图的右侧展示的是实施多重采样之后的版本，每个像素包含有 4 个采样点。这里，只有两个采样点遮盖住了三角形。

采样点的数量可以是任意的，更多的采样点能带来更精确的遮盖率。

从这里开始多重采样就变得有趣起来了。我们知道三角形只遮盖了 2 个子采样点，所以下一步是决定这个像素的颜色。你的猜想可能是，我们对每个被遮盖住的子采样点运行一次片段着色器，最后将每个像素所有子采样点的颜色平均一下。在这个例子中，我们需要在两个子采样点上对被插值的顶点数据运行两次片段着色器，并将结果的颜色储存在这些采样点中。（幸运的是）这并**不是**它工作的方式，因为这本质上说还是需要运行更多次的片段着色器，会显著地降低性能。

MSAA 真正的工作方式是，无论三角形遮盖了多少个子采样点，（每个图元中）每个像素只运行**一次**片段着色器。片段着色器所使用的顶点数据会插值到每个像素的**中心**，所得到的结果颜色会被储存在每个被遮盖住的子采样点中。当颜色缓冲的子样本被图元的所有颜色填满时，所有的这些颜色将会在每个像素内部平均化。因为上图的 4 个采样点中只有 2 个被遮盖住了，这个像素的颜色将会是三角形颜色与其他两个采样点的颜色（在这里是无色）的平均值，最终形成一种淡蓝色。

这样子做之后，颜色缓冲中所有的图元边缘将会产生一种更平滑的图形。让我们来看看前面三角形的多重采样会是什么样子：

![img](https://learnopengl-cn.github.io/img/04/11/anti_aliasing_rasterization_samples.png)

这里，每个像素包含 4 个子采样点（不相关的采样点都没有标注），蓝色的采样点被三角形所遮盖，而灰色的则没有。对于三角形的内部的像素，片段着色器只会运行一次，颜色输出会被存储到全部的 4 个子样本中。而在三角形的边缘，并不是所有的子采样点都被遮盖，所以片段着色器的结果将只会储存到部分的子样本中。根据被遮盖的子样本的数量，最终的像素颜色将由三角形的颜色与其它子样本中所储存的颜色来决定。

简单来说，一个像素中如果有更多的采样点被三角形遮盖，那么这个像素的颜色就会更接近于三角形的颜色。如果我们给上面的三角形填充颜色，就能得到以下的效果：

![img](https://learnopengl-cn.github.io/img/04/11/anti_aliasing_rasterization_samples_filled.png)

对于每个像素来说，越少的子采样点被三角形所覆盖，那么它受到三角形的影响就越小。三角形的不平滑边缘被稍浅的颜色所包围后，从远处观察时就会显得更加平滑了。

不仅仅是颜色值会受到多重采样的影响，深度和模板测试也能够使用多个采样点。对深度测试来说，每个顶点的深度值会在运行深度测试之前被插值到各个子样本中。对模板测试来说，我们对每个子样本，而不是每个像素，存储一个模板值。当然，这也意味着深度和模板缓冲的大小会乘以子采样点的个数。

我们到目前为止讨论的都是多重采样抗锯齿的背后原理，光栅器背后的实际逻辑比目前讨论的要复杂，但你现在应该已经可以理解多重采样抗锯齿的大体概念和逻辑了。

(译者注： 如果看到这里还是对原理似懂非懂，可以简单看看知乎上[@文刀秋二](https://www.zhihu.com/people/edliu/answers)对抗锯齿技术的[精彩介绍](https://www.zhihu.com/question/20236638/answer/14438218))

### OpenGL 中的 MSAA

如果我们想要在 OpenGL 中使用 MSAA，我们必须要使用一个能在每个像素中存储大于 1 个颜色值的颜色缓冲（因为多重采样需要我们为每个采样点都储存一个颜色）。所以，我们需要一个新的缓冲类型，来存储特定数量的多重采样样本，它叫做多重采样缓冲(Multisample Buffer)。

大多数的窗口系统都应该提供了一个多重采样缓冲，用以代替默认的颜色缓冲。GLFW 同样给了我们这个功能，我们所要做的只是**提示**(Hint) GLFW，我们希望使用一个包含 N 个样本的多重采样缓冲。这可以在创建窗口之前调用 `glfwWindowHint` 来完成。

```c++
glfwWindowHint(GLFW_SAMPLES, 4);
```

现在再调用 `glfwCreateWindow` 创建渲染窗口时，每个屏幕坐标就会使用一个包含 4 个子采样点的颜色缓冲了。GLFW 会自动创建一个每像素 4 个子采样点的深度和样本缓冲。这也意味着所有缓冲的大小都增长了 4 倍。

现在我们已经向 GLFW 请求了多重采样缓冲，我们还需要调用 `glEnable` 并启用 GL_MULTISAMPLE，来启用多重采样。在大多数 OpenGL 的驱动上，多重采样都是默认启用的，所以这个调用可能会有点多余，但显式地调用一下会更保险一点。这样子不论是什么 OpenGL 的实现都能够正常启用多重采样了。

```c++
glEnable(GL_MULTISAMPLE);
```

只要默认的帧缓冲有了多重采样缓冲的附件，我们所要做的只是调用 glEnable 来启用多重采样。因为多重采样的算法都在 OpenGL 驱动的光栅器中实现了，我们不需要再多做什么。如果现在再来渲染本节一开始的那个绿色的立方体，我们应该能看到更平滑的边缘：

![img](https://learnopengl-cn.github.io/img/04/11/anti_aliasing_multisampled.png)

这个箱子看起来的确要平滑多了，如果在场景中有其它的物体，它们也会看起来平滑很多。

### 离屏 MSAA

由于 GLFW 负责了创建多重采样缓冲，启用 MSAA 非常简单。然而，如果我们想要使用我们自己的帧缓冲来进行离屏渲染，那么我们就必须要自己动手生成多重采样缓冲了。

有两种方式可以创建多重采样缓冲，将其作为帧缓冲的附件：纹理附件和渲染缓冲附件，这和在[帧缓冲](https://learnopengl-cn.github.io/04 Advanced OpenGL/05 Framebuffers/)教程中所讨论的普通附件很相似。

#### 多重采样纹理附件

为了创建一个支持储存多个采样点的纹理，我们使用 `glTexImage2DMultisample` 来替代 `glTexImage2D`，它的纹理目标是 GL_TEXTURE_2D_MULTISAPLE。

```c++
glBindTexture(GL_TEXTURE_2D_MULTISAMPLE, tex);
glTexImage2DMultisample(GL_TEXTURE_2D_MULTISAMPLE, samples, GL_RGB, width, height, GL_TRUE);
glBindTexture(GL_TEXTURE_2D_MULTISAMPLE, 0);
```

它的第二个参数设置的是纹理所拥有的样本个数。如果最后一个参数为 GL_TRUE，图像将会对每个纹素使用相同的样本位置以及相同数量的子采样点个数。

我们使用 `glFramebufferTexture2D` 将多重采样纹理附加到帧缓冲上，但这里纹理类型使用的是 GL_TEXTURE_2D_MULTISAMPLE。

```c++
glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, GL_TEXTURE_2D_MULTISAMPLE, tex, 0);
```

当前绑定的帧缓冲现在就有了一个纹理图像形式的多重采样颜色缓冲。

#### 多重采样渲染缓冲对象

和纹理类似，创建一个多重采样渲染缓冲对象并不难。我们所要做的只是在指定（当前绑定的）渲染缓冲的内存存储时，将 `glRenderbufferStorage` 的调用改为`glRenderbufferStorageMultisample` 就可以了。

```c++
glRenderbufferStorageMultisample(GL_RENDERBUFFER, 4, GL_DEPTH24_STENCIL8, width, height);
```

函数中，渲染缓冲对象后的参数我们将设定为样本的数量，在当前的例子中是 4。

#### 渲染到多重采样帧缓冲

渲染到多重采样帧缓冲对象的过程都是自动的。只要我们在帧缓冲绑定时绘制任何东西，光栅器就会负责所有的多重采样运算。我们最终会得到一个多重采样颜色缓冲以及/或深度和模板缓冲。因为多重采样缓冲有一点特别，我们不能直接将它们的缓冲图像用于其他运算，比如在着色器中对它们进行采样。

一个多重采样的图像包含比普通图像更多的信息，我们所要做的是缩小或者还原(Resolve)图像。多重采样帧缓冲的还原通常是通过 `glBlitFramebuffer` 来完成，它能够将一个帧缓冲中的某个区域复制到另一个帧缓冲中，并且将多重采样缓冲还原。

`glBlitFramebuffer` 会将一个用 4 个屏幕空间坐标所定义的源区域复制到一个同样用 4 个屏幕空间坐标所定义的目标区域中。你可能记得在[帧缓冲](https://learnopengl-cn.github.io/04 Advanced OpenGL/05 Framebuffers/)教程中，当我们绑定到 GL_FRAMEBUFFER 时，我们是同时绑定了读取和绘制的帧缓冲目标。我们也可以将帧缓冲分开绑定至 GL_READ_FRAMEBUFFER 与 GL_DRAW_FRAMEBUFFER。`glBlitFramebuffer` 函数会根据这两个目标，决定哪个是源帧缓冲，哪个是目标帧缓冲。接下来，我们可以将图像位块传送(Blit)到默认的帧缓冲中，将多重采样的帧缓冲传送到屏幕上。

```c++
glBindFramebuffer(GL_READ_FRAMEBUFFER, multisampledFBO);
glBindFramebuffer(GL_DRAW_FRAMEBUFFER, 0);
glBlitFramebuffer(0, 0, width, height, 0, 0, width, height, GL_COLOR_BUFFER_BIT, GL_NEAREST);
```

如果现在再来渲染这个程序，我们会得到与之前完全一样的结果：一个使用 MSAA 显示出来的橄榄绿色的立方体，而且锯齿边缘明显减少了：

![img](https://learnopengl-cn.github.io/img/04/11/anti_aliasing_multisampled.png)

但如果我们想要使用多重采样帧缓冲的纹理输出来做像是后期处理这样的事情呢？我们不能直接在片段着色器中使用多重采样的纹理。但我们能做的是将多重采样缓冲位块传送到一个没有使用多重采样纹理附件的 FBO 中。然后用这个普通的颜色附件来做后期处理，从而达到我们的目的。然而，这也意味着我们需要生成一个新的 FBO，作为中介帧缓冲对象，将多重采样缓冲还原为一个能在着色器中使用的普通 2D 纹理。这个过程的伪代码是这样的：

```c++
unsigned int msFBO = CreateFBOWithMultiSampledAttachments();
// 使用普通的纹理颜色附件创建一个新的FBO
...
glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, GL_TEXTURE_2D, screenTexture, 0);
...
while(!glfwWindowShouldClose(window))
{
    ...

    glBindFramebuffer(msFBO);
    ClearFrameBuffer();
    DrawScene();
    // 将多重采样缓冲还原到中介FBO上
    glBindFramebuffer(GL_READ_FRAMEBUFFER, msFBO);
    glBindFramebuffer(GL_DRAW_FRAMEBUFFER, intermediateFBO);
    glBlitFramebuffer(0, 0, width, height, 0, 0, width, height, GL_COLOR_BUFFER_BIT, GL_NEAREST);
    // 现在场景是一个2D纹理缓冲，可以将这个图像用来后期处理
    glBindFramebuffer(GL_FRAMEBUFFER, 0);
    ClearFramebuffer();
    glBindTexture(GL_TEXTURE_2D, screenTexture);
    DrawPostProcessingQuad();

    ...
}
```

如果现在再实现[帧缓冲](https://learnopengl-cn.github.io/04 Advanced OpenGL/05 Framebuffers/)教程中的后期处理效果，我们就能够在一个几乎没有锯齿的场景纹理上进行后期处理了。如果施加模糊的核滤镜，看起来将会是这样：

![img](https://learnopengl-cn.github.io/img/04/11/anti_aliasing_post_processing.png)

因为屏幕纹理又变回了一个只有单一采样点的普通纹理，像是**边缘检测**这样的后期处理滤镜会**重新导致锯齿**。为了补偿这一问题，你可以之后对纹理进行模糊处理，或者想出你自己的抗锯齿算法。

你可以看到，如果将多重采样与离屏渲染结合起来，我们需要自己负责一些额外的细节。但所有的这些细节都是值得额外的努力的，因为多重采样能够显著提升场景的视觉质量。当然，要注意，如果使用的采样点非常多，启用多重采样会显著降低程序的性能。在本节写作时，通常采用的是 4 采样点的 MSAA。

### 自定义抗锯齿算法

将一个多重采样的纹理图像不进行还原直接传入着色器也是可行的。GLSL 提供了这样的选项，让我们能够对纹理图像的每个子样本进行采样，所以我们可以创建我们自己的抗锯齿算法。在大型的图形应用中通常都会这么做。

要想获取每个子样本的颜色值，你需要将纹理 uniform 采样器设置为 sampler2DMS，而不是平常使用的 sampler2D：

```c++
uniform sampler2DMS screenTextureMS;
```

使用 texelFetch 函数就能够获取每个子样本的颜色值了：

```c++
vec4 colorSample = texelFetch(screenTextureMS, TexCoords, 3);  // 第4个子样本
```

我们不会深入探究自定义抗锯齿技术的细节，这里仅仅是给你一点启发。

## References

- [Rendering 学习：Early-Z 与 Z-PrePass - 技术专栏 - Unity 官方开发者社区](https://developer.unity.cn/projects/62b2dcb3edbc2a7848d36e4f)
- [Unity Shader-渲染队列，ZTest，ZWrite，Early-Z-腾讯游戏学堂 (tencent.com)](https://gwb.tencent.com/community/detail/114229)
- [Unity3D 研究院之 URP 下 PrePassZ（一百一十九） | 雨松 MOMO 程序研究院 (xuanyusong.com)](https://www.xuanyusong.com/archives/4759)
- [Rendering 学习：Early-Z 与 Z-PrePass - 技术专栏 - Unity 官方开发者社区](https://developer.unity.cn/projects/62b2dcb3edbc2a7848d36e4f)
- [Reversed-Z 详解 - jackmaxwell - 博客园 (cnblogs.com)](https://www.cnblogs.com/jackmaxwell/p/6851728.html)
- [程序媛转 TA 之面试篇二：z-fighting，以及 z 精度的最佳分辨率函数 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/651462948)
- [glBlendFuncSeparate - OpenGL 4 Reference Pages (khronos.org)](https://registry.khronos.org/OpenGL-Refpages/gl4/html/glBlendFuncSeparate.xhtml)
- [Android OpenGLES2.0（十八）——轻松搞定 Blend 颜色混合\_湖广午王的博客-CSDN 博客](https://blog.csdn.net/junzia/article/details/76580379)
- [彩色转灰度的公式 - 野生特效测试员 - 博客园 (cnblogs.com)](https://www.cnblogs.com/wildtester/p/15862624.html)
- [详解 Cubemap、IBL 与球谐光照 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/463309766)
