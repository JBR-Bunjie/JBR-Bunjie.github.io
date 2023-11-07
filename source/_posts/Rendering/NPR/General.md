---
title: 全排列
date: 2022-12-23 12:23:23
tags:
  - Rendering
categories:
  - Rendering
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

## 光照模型特点概括：

首先，分析一下所谓的 `日式卡通渲染` 都有些什么特点：

- 硬过渡亮暗面
- 描边
-

因此，其可采用的具体的表现形式有：

- 裁边漫反射 \- StepDiffuse
- 裁边高光
- 裁边边缘光
- 裁边视角光
- 裁边光源光

### 裁边漫反射

卡通渲染里希望存在明快的色调对比，而不希望存在额外的过渡光照信息：

在实际执行中，我们使用：

```cpp
float NL = dot(N, L);

// 传统Lambert漫反射
// traditionalDiffuse = _ObjectColor.rgb * _ColorTint_rgb * max(0, NL);

// 进行映射以增大光照区域，当然也会方便后续可能的RampTexture取样
float NL01 = NL * 0.5 + 0.5;

float Threshold = step(_LightThreshold, NL01);
diffuse = lerp(DarkSide, BrightSide, Threshold);
```

来改写传统的漫反射光照，这样我们就能得到如下效果：

![image-20230406155635370](../../../images\Rendering\NPR\General\StepDiffuse.png)
裁边漫反射结果示意

当然，我们也可以尝试在明暗交界线上做出特殊效果：

```cpp
NL01 = smoothstep(0, _Smooth, NL01 - _SmoothRange);
// 我们也可以用Step的结果加上traditional diffuse的结果来做出类似smooth step的效果，但是这样就达不到可控交界线位置的效果了
// https://learn.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-smoothstep

float Threshold = step(_LightThreshold, NL01);
```

### 裁边高光

在传统的 BlinPhong 高光中，我们的光照计算公式为：<br />

$$
result = pow(NH, \_Exp) * SpecularScale
$$

而为了裁边，我们会改用这样的计算：<br />

$$
NH = pow(NH, \_Exp)\\\\
result = step(1 - \_StepSpecularWidth * 0.01, NH)*SpecularIntensity;
$$

改写的基本思路为：使用额外的宽度控制参数的情况下，利用 step 函数对整个结果进行裁剪。在完成裁剪后，最后使用一个 Intensity 参数来控制最终 specular 呈现的值，这个值不会改变高光的范围，只会改变高光的具体数值，这会对我们的后续处理(如果存在的话)产生影响。

后面的大多数裁边算法的思路与裁剪高光的思路大体相同。

### 裁边边缘光

传统边缘光使用下方的计算方法，这会存在一个明显的明暗过渡：

$$
float3\space Rim = pow(1-NV,RimExp)*RimIntensity;
$$

因此我们使用一种和裁边高光相似的算法来改写这个过程：

$$
Rim = step(1 - \_RimWidth * 0.01, 1 - NV) * RimIntensity;
$$

当然，我们可能还会对边缘光做很多额外的处理，比如在某种情况下，我们只希望物体上存在亮部相关的边缘光，那么我们可以：

$$
diffuse = dot(N, L)\\\\
Rim = step(1 - \_RimWidth * 0.01, 1 - NV) * RimIntensity;\\\\
BrightSideRim = lerp(0, Rim, diffuse)
$$

这种实现所得到的结果类似于菲涅尔效果，这里也给出菲涅尔的实现（可参考入门精要 10.1.5 节）：

**Schlick 菲涅尔近似等式**：

$$
F_{Schlick}(\overrightarrow v,\space \overrightarrow n) = F_0 + (1 - F_0)(1 - \overrightarrow v \cdot \overrightarrow n)
$$

其中，F~0~是一个反射系数，这回是一个常数，一般我们会将它暴露出来以方便调整；**v**是视角方向；**n**是表面法线

**Empricial 菲涅尔近似等式**：

$$
F_{Empricial}(\overrightarrow v,\space \overrightarrow n) = max(0, min(1,\space bias + scale \times (1- \overrightarrow v \cdot \overrightarrow n)^{power}))
$$

bias，scale，power 都是控制项

从以上不难看出，菲涅尔其实也可以被归为一种边缘光，而边缘光的核心就是：**_1 - NV_**

### 裁边视角光

为了使眼睛看到的部分更亮而被运用的光。传统的视角光的计算是：

$$
ViewLight = pow(NV,\space ViewLightExp)*ViewLightIntensity;
$$

我们仍然采用雷同高光的做法：

$$
ViewLight = step(1 - \_ViewLightWidth,\space NV)*ViewLightIntensity;
$$

### 裁边光源光

裁边光源光对 NL 做 Step，这意味着其结果与视角无关，仅与法线与光方向相关。这种 `光源光` 可以算作 `视角光` 的一种，只是光源方向和视角方向重合了

$$
StepLight = step(1-StepLightWidth,\space NL);
$$

关于裁光边缘的锯齿，我们也需要作出处理：[涂月观 (tuyg.top)](http://tuyg.top/archives/850)

### Outline

#### Outline 的实现方式

Outline 有多种实现方式，入门精要中提到过这么几种：

1. 基于观察角度和表面法线的轮廓线渲染：使用视角方向和表面法线的点乘结果来得到轮廓信息
2. 过程式几何轮廓线渲染：使用两个 Pass 来进行构建：第一个 Pass 用来渲染背面的面片，并利用诸如顶点扩张等手法来使这些面片可见；第二个面片则用来渲染正面。这是大多数情况下的做法
3. 基于图像处理的轮廓线渲染：直接利用后处理来完成
4. 基于轮廓检测的轮廓线渲染：专用于需要精确检测出边缘并直接渲染它们以达到强烈的风格化效果的情况。利用一些特殊的判别式来确定轮廓线

第二类中，我们一般采用背面法线外扩的方式，这是基于模型的：我们一般先沿法线方向挤出顶点。但是这个方法有较大的限制，如果盲目地进行，可能会出现各种问题，比如：转折处大的或者硬表面会经常发生断裂，内凹的模型在描边后正面的面片被背面的面片遮挡等问题。

> 需要明确的是，平滑与平整是完全不同的

还是入门精要，书中共用到了两种描边方式：一种是后处理时，通过边缘检测来描边，另一种是背部法线扩张。事实上，单是这两种方法就能实现很不错的效果。我们逐个开始：

#### 后处理边缘检测

> 核心内容基本对应入门精要 12.3 节

利用基于卷积的边缘检测，可以实现对剧烈变化的图像区块进行描边

> "入门精要：...如果相邻像素之间存在差别明显的颜色、亮度、纹理等属性，我们就会认为他们之间应该有一条边界，这样的相邻像素之间的差值可以用梯度\(Gradient\)来表示..."

> 什么是卷积？
>
> > 3blue1brown: [【官方双语】那么……什么是卷积？\_哔哩哔哩\_bilibili](https://www.bilibili.com/video/BV1Vd4y1e7pj/?vd_source=c8eda79dd90c30ff02e09fb39906ac54)
>
> > 入门精要：
> >
> > 卷积操作是指，使用一个卷积核，对一张图像中的每个像素进行一系列操作。卷积核通常是一个四方形网络结构，该区域对每个方格都有一个权重值，当对图像中的某个像素进行卷积时，我们会把卷积核的中心放置在像素上，并在翻转核之后再依次计算核中每个元素的每个元素和其覆盖的图像像素值的乘积并求和，得到的结果就是该位置的新像素值。

边缘检测的卷积核被称为边缘检测算子，常用的边缘检测算子有：

$$
Roberts:\space
Gx =
\begin{bmatrix}
 -1 & 0\\
 0 & 1
\end{bmatrix};\space
Gy=
\begin{bmatrix}
 -1 & 0\\
 0 & 1
\end{bmatrix} \\\\

Prewitt:\space
Gx =
\begin{bmatrix}
 -1 & 0 & 1\\
 -1 & 0 & 1\\
 -1 & 0 & 1
\end{bmatrix}

\begin{bmatrix}
 -1 & -1 & -1\\
 0 & 0 & 0\\
 1 & 1 & 1
\end{bmatrix} \\\\

Sobel:\space
Gx =
\begin{bmatrix}
 -1 & 0 & 1\\
 -2 & 0 & 2\\
 -1 & 0 & 1
\end{bmatrix}

\begin{bmatrix}
 -1 & -2 & -1\\
 0 & 0 & 0\\
 1 & 2 & 1
\end{bmatrix}
$$

当我们进行卷积运算时，我们需要对每个像素分别使用 Gx 和 Gy 两个卷积核，进行两次卷积操作。之后我们会得到两个方向上的梯度值。再整合这两者就可以得到整体的梯度值：

$$
G=\sqrt[2]{(G_x)^2+(G_y)^2}
$$

这个梯度值越大，说明当前位置像素周围的颜色变化越剧烈，就越有可能是图像的边缘。

在 Built-in 管线中，我们可以直接通过向场景中的摄像机挂载脚本，通过调用`OnRenderImage`方法来实现后处理系统，但是 URP 中，我们则需要使用系统提供的 Render Feature 来完成这件事。具体实现可参考：[unity urp 14 render feature 实现简单后处理系统 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/526234665?utm_id=0)

对新手而言，在 Render Feature 下有几个要点需要指明：

##### 脚本结构：

> 快速创建 Render Feature 的脚本模板：Asserts->Create->Rendering->Universal Render Pipeline->Renderer Feature
>
> [Unity 中 URP 管线实现后处理的几种方式 (068089dy.github.io)](https://068089dy.github.io/2021/06/14/2021-06-14-unity-urp-postprocessing/)

具体流程为，构造一个继承`ScriptableRendererFeature`的主类，并在这之中或者引用别处的自定义`ScriptableRenderPass`，实现其中的部分方法。

```C#
class custom : ScriptableRendererFeature {
    ...
    public override void Create() {
        throw new System.NotImplementedException();
    }
    ...
    public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData renderingData) {
        throw new System.NotImplementedException();
    }

    class RenderOutlinePass : ScriptableRenderPass {
    	public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData) {
            throw new System.NotImplementedException();
        }
    }
}
```

##### 脚本一般流程：

在当前的 Universal Renderer Data 下挂载好了对应的脚本后，我们的脚本就会开始生效。

Create 函数会最先执行，我们在这里初始化我们会用到的全部数据及内容，包括`ScriptableRenderPass`的初始化等

然后会转到 AddRenderPasses 的执行阶段，这里的标准流程是获取某个阶段的摄像机内容，传入`ScriptableRenderPass`实例中并将它重新压入这一帧的渲染队列

等到了`ScriptableRenderPass`执行时，便会按照我们在`ScriptableRenderPass`类中定义的 Execute 方法执行，这一方法的具体内容和 OnRenderImage 函数很像

##### 脚本核心内容：

###### 外部参数

对于一个后处理描边，我们会有如下的控制需求：

- 实际使用的材质（也可以只汇入 Shader，然后再临时创建专用的 Material，但终究是需要一个所专用的 Material 用于最后的函数调用）
- 描边的颜色、描边的程度等
- 用于指定 Render Feature 生效阶段的`renderPassEvent`

###### ScriptableRenderPass

> - https://docs.unity3d.com/Manual/render-pipelines-feature-comparison.html
>   ScriptableRenderPass implements a logical rendering pass that can be used to extend Universal RP renderer.

在由 AddRenderPasses 压入栈之后，我们所定义的`ScriptableRenderPass`就会在对应的阶段被执行，为此，我们需要至少在`ScriptableRenderPass`中，实现这些内容：

- 接收在 Create 函数中汇入的外部参数，包括指定 renderPassEvent、Material 等

- 编写实际执行调用 Material 去渲染的函数 Execute，这会包括：

  - 申请 Command Buffer
  - 获取当前阶段的摄像机内结果
  - （如有多步的 Blit（如高斯模糊）调用）申请临时 RT 作缓冲区
  - 调用执行 Command Buffer
  - 释放所有资源

- 特别的，我们会遇到这些特殊内容：

  - CommandBuffer: https://docs.unity3d.com/2021.3/Documentation/ScriptReference/Rendering.CommandBuffer.html

    - CommandBuffer.GetTemporaryRT: https://docs.unity3d.com/2021.3/Documentation/ScriptReference/Rendering.CommandBuffer.GetTemporaryRT.html

  - RenderTargetIdentifier: https://docs.unity3d.com/2021.3/Documentation/ScriptReference/Rendering.RenderTargetIdentifier.html，我们用此对象来接收从Camera传入的Src RT

  - RenderTargetHandle: RenderTargetHandle.cs，于 namespace UnityEngine.Rendering.Universal 中

  - 获取摄像机输入的 RT：

    ```C#
            /// <summary>
            /// Returns the camera color target for this renderer.
            /// It's only valid to call cameraColorTarget in the scope of <c>ScriptableRenderPass</c>.
            /// <seealso cref="ScriptableRenderPass"/>.
            /// </summary>
            public RenderTargetIdentifier cameraColorTarget {
                get{
                    if (!(m_IsPipelineExecuting || isCameraColorTargetValid)) {
                        Debug.LogWarning("You can only call cameraColorTarget inside the scope of a ScriptableRenderPass. Otherwise the pipeline camera target texture might have not been created or might have already been disposed.");
                        // TODO: Ideally we should return an error texture (BuiltinRenderTextureType.None?)
                        // but this might break some existing content, so we return the pipeline texture in the hope it gives a "soft" upgrade to users.
                    }
                    return m_CameraColorTarget;
                }
            }
    ```

- RenderTextureDescriptor: https://docs.unity3d.com/2021.3/Documentation/ScriptReference/RenderTextureDescriptor.html

#### 更进一步：在后处理的边缘检测中加入深度与法线纹理

> 核心内容基本对应入门精要 13.4 节

> 在 urp 中快速获取深度图：[【转载】Unity URP 获取深度图 - 掘金 (juejin.cn)](https://juejin.cn/post/7239910114949152826)
>
> 在 urp 中自己生成特定情况下的深度图：[Unity3D:URP 下输出深度图以及自定义 ScriptableRenderer - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/351390737)
>
> [基于深度与法线贴图的屏幕后处理——Unity URP“\_CameraDepthTexture”和“\_CameraNormalsTexture”使用讲解 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/575655530)

> [练习项目(八)：在 URP 中显示法线图 - CodeAntenna](https://codeantenna.com/a/ZS067JQogu)

需要说明的是，我们使用了一种特殊的技巧：我们将 Shader 中的 Lightmode 的值，设为了 URP 中不存在的值，这样这个 Pass 一开始不会执行，而我们在 Render Feature 中捕获这个值，并去替换这个 Pass，就可以拿到深度与法线纹理。

> [URP ShaderLab Pass tags | Universal RP | 11.0.0 (unity3d.com)](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@11.0/manual/urp-shaders/urp-shaderlab-pass-tags.html#urp-pass-tags-lightmode)
>
> [Unity - Manual: ShaderLab: Predefined Pass tags in the Built-in Render Pipeline (unity3d.com)](https://docs.unity3d.com/Manual/shader-predefined-pass-tags-built-in.html)

> [Unity - Manual: ShaderLab: assigning tags to a Pass (unity3d.com)](https://docs.unity3d.com/Manual/SL-PassTags.html) "Using Pass tags with C# scripts" Part

#### 更多办法？

很显然，基于后处理的边缘检测所得到的结果不够精确：不只是人物，场景中的所有内容都被一视同仁地进行处理，使结果包含了大量的不必要的描边。

而对于基于深度与法线后处理检测，虽然它能得到比较准确的效果，但是并非是完全适用于人物的描边.

在入门精要中，对这种基于图像处理的轮廓线渲染的优缺点的描述为：

> 这种方法的优点在于，可以适用于任何种类的模型。但它也有自身的局限所在，一些深度和法线变化很小的轮廓无法被检测出来，例如桌子上的纸张。

> 在米哈游 2017 年的分享中，有这样两张 PPT：
>
> ![image-20230729213125901](../../../images\Rendering\NPR\General\007Outline.png)
>
> ![image-20230729213257293](../../../images\Rendering\NPR\General\008Outline.png)
>
> 即，这种 Image Space 的 Outline 更加适用于对场景进行描边，而对于人物，我们会使用基于 Backface 的描边。这些内容与 2019 年的分享内容没有太大变化。

为了得到更加精确的描边，我们可以采用一种基于法线扩张的描边办法：

#### BackFacing 描边

BackFacing 有很多优点，首先就是实现简单，特别是在 Unity Shader 中，我们只需要新开一个 Pass，然后沿法线方向延展顶点就可以了。

另外，由于我们的描边是模型渲染的一个内容，我们可以很方便地控制很多细节，比如描边颜色与描边粗细等。

在入门精要中，给出的描边 Pass 是在 View Space 中进行的，这样子有一个问题：描边的粗细是固定的，这意味着它会受透视效果而改变，即：当摄像机靠近角色时，描边会变得很粗。因此，我们应该在透视变换后再进行顶点扩张。我们需要将法线转换到 NDC Space，并在该空间下进行运算。

> [【01】从零开始的卡通渲染-描边篇 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/109101851)

在 NDC 下变换后，我们可以发现现在描边粗细已经是摄像机无关的了——无论摄像机远近，我们都能保证描边粗细一致。但是还有一个问题：描边会在某些边缘断裂，准确的说：当模型上出现弯折较大的部位的情况下，其对应位置的法线变化会过大，导致的结果就是最终的描边就会像这样裂开。

为了解决这个问题，我们需要进行法线平滑。最简单的是平均法线，我们在脚本中检测每个顶点上的法线、相加，最后求平均。这样得到的结果基本上可以保证法线不会断裂，但是又另一个问题，就是如果直接使用平均法线，可能并不能完全保证法线的原始效果，我们可能需要对着个平滑过程进行加权：

> [Tech-Artist 学习笔记：Smooth Shade 平均法线与加权法线 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/546554527)

最后，当我们计算完平滑的法线后，我们需要将它存储到模型中，如果是使用模型的 Tangent 空间的话，我们可以直接将当前 Object Space 下的计算结果直接覆写 Tangent 中的数据，可是如果我们需要使用原始的 Tangent 数据，那我们就要考虑别的存储位置例如 UV，此时我们需要多做一件事：将法线转换到切线空间再存储：

> [Unity 平滑法线用于卡渲描边 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/643206592)

值得一提的是，这里的加权方式是角度变量加权。不同的平滑思路会造成不同的结果，这种不同甚至可能出现在模型的不同分件中，我们需要在不同的部位，采用不同的平滑方案。

这样子，我们就基本上构建了一个完善的法线平滑脚本了。这是在模型导入后的二次处理，那么可不可以在模型导入时，就直接对其进行操作呢？

> [【Job/Toon Shading Workflow】自动生成硬表面模型 Outline Normal - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/107664564)

#### 一些补充

至此，我们常用的描边办法就全部介绍完了，不过除此之外，还有几个可以补充的要点：

1. 入门精要中还提到过一种可用的轮廓线渲染办法：基于轮廓边检测的轮廓线渲染：Mesh 上的每一条边一定被两个三角形共用，检测 Mesh 上所有像这样相邻的三角面片是否符合：

   $$
   (\vec{n_0} * \vec{v} > 0) \ne (\vec{n_1} * \vec{v} > 0)\\\\
   其中，n_0和n_1分别代表了两个相邻三角面片的法向，v是从视角到该边上任意顶点的方向
   $$

2. [渲染管线中的法线变换矩阵 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/72734738)

### 高光形变

[Hair Rendering and Shading (oregonstate.edu)](https://web.engr.oregonstate.edu/~mjb/cs519/Projects/Papers/HairRendering.pdf)

我们并不希望高光总是⼀个圆形的光斑，因此我们需要对高光的形状也进行风格化处理：对高光点进行形变

1. Kajiya Kay 模型：

   需要注意的是，这里的 T 并非是切线，它代表的是当前模型的副切线

$$
Specular = L_i * k_s * \sqrt{ \overrightarrow T \cdot \overrightarrow H } ^ {strength}
$$

![image-20230501160324069](../../../images\Rendering\NPR\General\004.png)
用 Kajiya Kay 模型实现的环形高光，Strength == 64，没有使用 Shift Map，没有使用衰减。（计算 W 型高光需要使用 Shift Map，可用的处理办法可参考：[基于 Kajiya-Kay 模型的毛发渲染 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/135910659))

2. Marschner 模型：[hair-sg03final.pdf (stanford.edu)](http://graphics.stanford.edu/papers/hair/hair-sg03final.pdf)

   就 Marschner 模型本身而言，这是一个基于物理的模型。[Marschner Hair Model 论文细读与推导 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/372590873)

![img](../../../images\Rendering\NPR\General\005.png)

R：反射光线，T：透射光线。R，TT 和 TRT 是三个对毛发反射率产生明显影响部分。
R： 从头发纤维表面向观察者反弹的光。TT： 折射到头发中并再次向观察者折射的光。TRT：光线折射到头发纤维中，从内表面反射，然后再次向观察者折射。

$$
S=S_R+S_{TT}+S_{TRT} \\
SP=MP⋅NP\\
for \space P = R, TT, TRT\\
$$

[剖析 Unreal Engine 超真实人类的渲染技术 Part 3 - 毛发渲染及其它 - 0 向往 0 - 博客园 (cnblogs.com)](https://www.cnblogs.com/timlly/p/11199385.html)

[Marschner Hair Model 论文细读与推导 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/372590873)

> Kajiya-Kay 模型是一种基于经验的头发渲染模型，它使用头发的切线来模拟平面法线的效果，能够近似地表现出高光的效果，并使用漫反射项来近似头发间的相互散射情况。然而，Kajiya-Kay 模型不是基于物理的，它将头发建模成不透明的圆柱体，因此不能模拟光线可能穿过头发或者在头发中传播的情况，这就导致了其不能模拟出背光以及二次高光等效果
>
> Marschner 模型是一种基于物理的头发渲染模型，它在 Kajiya-Kay 模型的基础上对单根头发的散射情况进行了实验和理论分析，然后建立了一个实用且基于物理的着色模型。Marschner 模型将光照在毛发上的作用分为三个部分：R、TT 和 TRT。这三个部分对毛发反射率产生明显影响

3. 近似的 Marschner Model？

   [【UE4】材质藏宝阁 01\_头发 Kajiya-Kay Shading - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/434058280)，即 Kajiya Kay + Shift Map + 第二层 Base Color 高光

可参考的各向异性高光示例：

[Unity3D ShaderLab 各向异性高光 - 川北 - 博客园 (cnblogs.com)](https://www.cnblogs.com/2Yous/p/4234811.html)

一个风格高光实现：

[涂月观 (tuyg.top)](http://tuyg.top/archives/927)

ATI 示例代码：

![image-20230501191639629](../../../images\Rendering\NPR\General\006.png)

### 小结：

卡通渲染中，最重要的就是：使用 Step 的、强烈的明暗分割

## PBR 下的材质表达

### PBR 材质分析

![image-20230407210359909](../../../images\Rendering\NPR\General\002_PBRInURP.png)
ASE 在 URP 管线下的默认 PBR 材质输出节点

对于 PBR 预制节点所不能表达的效果，我们有两种解决思路：

1. 每种特性单独做一个 Shader。这种方案的优点是 Shader 功能相对确定，GPU 计算效率快，缺点是会增加 DrawCall

2. 使用 UberShader 以包含所有特性，然后再通过 Mask 进行材质分层。这种方案的优点是 DrawCall 少，基本上⼀个通⽤的 Shader 可以满足大多数功能，同时可以减少贴图数量，缺点是会产生许多无用的 GPU 计算。

   产生无用计算的原因：

   ![image-20230407212230706](../../../images\Rendering\NPR\General\003_UselessCalculations.png)
   If 语句会先计算出括号里面的内容，再根据条件值判断是否接受这个值。这种运行方式与 CPU 不同，CPU 是先根据条件值是否为真，再去判断是否要执行括号内的内容。

和大多数最终的解决方案类似的——⼀种成熟的解决方案是 1 与 2 的结合，即在通用的 Shader 中做出通用特性，每个特性不能消耗太多的计算，而特殊的特性用单独的 Shader。

## Texture 介绍

Base Map：基础色

Shadow Map：暗部衰减色，我们

## 具体案例分析

### 罪恶装备 Strive 的渲染分析与复现

#### 部分引用

[【翻译】西川善司「实验做出的游戏图形」「GUILTY GEAR Xrd -SIGN-」中实现的「纯卡通动画的实时 3D 图形」的秘密，前篇（1） - Trace0429 - 博客园 (cnblogs.com)](https://www.cnblogs.com/TracePlus/p/4205798.html)

[【翻译】西川善司「实验做出的游戏图形」「GUILTY GEAR Xrd -SIGN-」中实现的「纯卡通动画的实时 3D 图形」的秘密，前篇（2） - Trace0429 - 博客园 (cnblogs.com)](https://www.cnblogs.com/TracePlus/p/4205834.html)

#### Shader 编写

数据使用：

- BaseMap：基础⾊
- ShadowMap：暗部衰减色，与 BaseMap 相乘构成暗部
- DetailTex：磨损线条
- LightMap.r：高光类型
- LightMap.g：Ramp 偏移值
- LightMap.b：高光强度 mask
- LightMap.a：内勾线的 Mask
- VertexColor.r：AO，常暗部分
- VertexColor.g：用来区分身体的部位
- VertexColor.b：描边遮罩
- VertexColor.a：渲染无用

```cpp

```

### 碧蓝幻想

### 原神

### 破晓传说

### 战双

## Reference

-
- [【01】从零开始的卡通渲染-描边篇 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/109101851)
- [Unity 描边法线平滑工具 x 踩坑记录 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/508319122)
- [渲染管线中的法线变换矩阵 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/72734738)
- [MenuItem is not working properly - Unity Forum](https://forum.unity.com/threads/menuitem-is-not-working-properly.853822/)
- [URP ShaderLab Pass 标签 | Universal RP | 12.1.1 (unity3d.com)](https://docs.unity3d.com/cn/Packages/com.unity.render-pipelines.universal@12.1/manual/urp-shaders/urp-shaderlab-pass-tags.html)
- [【02】卡通渲染基本光照模型的实现 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/95986273)
- [Guilty Gear Strive - Sol Badguy XPS (Updated) by o-DV89-o on DeviantArt](https://www.deviantart.com/o-dv89-o/art/Guilty-Gear-Strive-Sol-Badguy-pack-for-XPS-882551758)
- [ChiliMilk/URP_Toon: A Toon Shader in Unity Universal Render Pipeline. (github.com)](https://github.com/ChiliMilk/URP_Toon)
- [[卡通渲染\]一、罪恶装备角色渲染还原 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/546396053)
- [基于 Kajiya-Kay 模型的毛发渲染 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/135910659)
- [【UE4】材质藏宝阁 01\_头发 Kajiya-Kay Shading - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/434058280)
- [涂月观-吹灯窗更明，月照一天雪 (tuyg.top)](http://tuyg.top/)
- [Hair Rendering and Shading (oregonstate.edu)](https://web.engr.oregonstate.edu/~mjb/cs519/Projects/Papers/HairRendering.pdf)
- [Yoolie on Twitter: "Tutorial on adjusting vertex normals for that 3D Anime look! since a couple of people were asking for it, I decided to make it! Hopefully it's useful to someone! It's the first tutorial I've ever made, so I apologize if it's kind of unclear ;-; #blender #bnpr #3D #BlenderNPR https://t.co/3cctLK8tFD" / Twitter](https://twitter.com/Yoolies/status/1232345380991438855)
- [TA 技术美术-罪恶装备角色还原 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/493802718)
- [灯光技法-伦勃朗光（2） - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/50943366)
- [【02】卡通渲染基本光照模型的实现 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/95986273)

## 既有渲染思路分析

> [ChiliMilk/URP_Toon: A Toon Shader in Unity Universal Render Pipeline. (github.com)](https://github.com/ChiliMilk/URP_Toon)

### 项目结构分析

该项目有三个

共有

### 核心代码分析

> 分析的文件包括：
>
> - ToonShaderGUI.cs
> - Toon.shader
> -

Toon.shader 文件的布局仍然遵循我们所提到的 shaderlab 基本结构：

- Properties：作者在这里声明了所有可用的属性
- SubShader
- CustomEditor：作者的确在项目中定义一个简单的 GUI，虽然没有以窗口的形式呈现，但是材质的面板上确实变得不同了
- FallBack：FallBack 到 Error

#### Properties 部分

Properties 部分存在于 3 ~ 101 行.

其中定义的大部分属性都直接用

#### 定义输入结构

第一个 Pass 位于 109 ~ 130 行。它通过引入了两个外部文件来实现主要功能，它们分别是：`ToonInput.hlsl`，`ToonHairShadowMaskPass.hlsl`

```cpp
        Pass
        {
            Name "HairShadowMask" //
            ZTest Less
            Tags{"LightMode"="HairShadowMask"}
            ZWrite Off
            Cull Back

            HLSLPROGRAM
            #pragma exclude_renderers gles gles3 glcore
            #pragma target 4.5

            #pragma multi_compile_instancing
            #pragma multi_compile_fog

            #pragma vertex Vertex
            #pragma fragment Fragment

            #include "../Include/ToonInput.hlsl"
            #include "../Include/ToonHairShadowMaskPass.hlsl"
            ENDHLSL
        }
```

显而易见的，这两个引入文件分别实现了输入结构的定义与 ShadowMap 的计算

我们先从 `ToonInput.hlsl` 开始：
