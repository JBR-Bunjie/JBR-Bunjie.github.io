---
title: 实现一个视差体积雾吧！
date: 2023-3-2 12:23:23
tags:
  - Unity
  - Shader
categories:
  - Unity
  - Shader
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

# 实现一个视差体积雾吧！

## 原理说明

核心思路：基于一张噪点图，基于切线空间下的视线方向，不断偏移 uv 来取样其中的内容，由于 Texture 下暗处的 rgb 三值都会急速下降并趋于 0，因此让 uv 偏移到暗处即可形成自然的颜色渐变——我们的视差体积雾即以该内容来伪装“高度”。在完成采样后，将最终采样与初始采样做合并来让暗部更暗以增强对比。

## 实现思路

### Vertex Shader

构建 `rotation` 矩阵，并利用该矩阵将 ObjectSpace 下的 viewDir 转换到 Tangent 空间下。在切线空间，顶点和像素成为原点，xy 轴与 uv 同向——此时就可以利用 ViewDir 来代表了一种对 uv 进行抽象的渐进取样方向与步长，当然，这里只有方向是可以直接使用的，对于具体的步长内容，我们应该将它们的控制变量利用属性以暴露出来。

### Fragment Shader

#### 步长

首先我们将 viewDir 分为 xy 和 z 两个维度，分别利用外置暴露的属性来进行控制，这有两个目的：

- xy 控制了对 uv 的最大偏移量
- 而对 z 进行控制，则是为了控制单次偏移的步长

最终计算步长：

```cpp
float3 minOffset = viewDir / (viewDir.z * _Layers)
```

#### 初始采样

之后，我们先利用初始的 uv 来对 texture 进行取样，这是可选的，但我们利用它来作为一个底层，并将之后的计算结果全部都叠加与这之上，这会：

- 加快取样 uv 的偏移过程
- 增加暗部与亮部的对比来增强立体感

#### uv 偏移过程

uv 偏移结束是基于两个内容的：finiNoise 的减小与 minOffset 上 z 分量的逐步叠加，其中 finiNoise 会减小是因为其数据的直接来源——Texture，在黑色上，其 rgb 值是趋于 0 的——这会让 finiNoise 急速衰减，因此，我们实际使用的 Texture 实际上是由要求的——它的黑色部分应该受到相当程度的限制，否则实际上的偏移量会过小导致视差的体积云失效。

```cpp
float finiNoise = SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, uv.xy).r * MainTex.r;  // 只作为加快偏移结束的变量
float3 prev_uv = uv;

while (finiNoise > uv.z) {
    uv += minOffset;
    finiNoise = SAMPLE_TEXTURE2D_LOD(_MainTex, sampler_MainTex, uv.xy, 0).r * MainTex.r;
    // finiNoise = SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, uv.xy).r * MainTex.r;
    // 此处不能使用寻常的SAMPLE_TEXTURE2D函数，原因见`tex2Dlod` 与 `SAMPLE_TEXTURE2D_LOD`一节
}
```

#### 最终取样

在初始 uv 和最终 uv 间进行插值，这样使我们最终获得的结果更加平滑，减少画面的突变

```cpp
// 可选
float d1 = finiNoise - uv.z;
float d2 = finiNoise - prev_uv.z;
float w = d1 / (d1 - d2 + 0.0000001);
uv = lerp(uv, prev_uv, w);

// 利用最终完成变换的uv来进行取样
half4 resultColor = SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, uv.xy) * MainTex;
```

#### 返回结果

暴露\_Alpha 属性来控制云层密度

```cpp
half rangeClt = MainTex.a * resultColor.r + _Alpha * 0.75;
half Alpha = abs(smoothstep(rangeClt, _Alpha, 1.0));
Alpha = pow(Alpha, 5);

// Light light = GetMainLight();

return half4(resultColor.rgb * _Color.rgb, Alpha); //* light.color.rgb
```

## 代码讲解

> 具体代码已在<a href="#code">Reference</a>中给出

### `tex2Dlod` 与 `SAMPLE_TEXTURE2D_LOD`

`tex2Dlod` 与 `SAMPLE_TEXTURE2D_LOD` 是两个同能相同的取样函数，分别对应了 `Built-in` 和 `urp` 两种渲染管线。不过，这并非意味着这两个函数分别对应了 `CG` 与 `HLSL` 中的不同的实现——事实上，`CG` 与 `HLSL` 中所定义的都是 `tex2Dlod`，`urp` 中采用新的 API，大概只是为了与原 `Built-in` 管线作出区分。

那么，首先先来了解 `tex2Dlod` 函数：

> Samples a 2D texture with mipmaps. The mipmap LOD is specified in t.w.
> [tex2Dlod - Win32 apps | Microsoft Learn](https://learn.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-tex2dlod)

着我我们平常见到的 `tex2D` 函数是不同的——`tex2D` 仅采样当前纹理本身：

> Samples a 2D texture.
> [tex2D (HLSL reference) - Win32 apps | Microsoft Learn](https://learn.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-tex2d)

当然，这也意味着我们使用该函数时，应确保加入的 texture 的 `Generate Mip Maps` 选项是勾选上的(当我们加入一张图片时，Unity 会帮我们默认勾选该内容:)

![image-20230331190104741](..\..\images\Dev\Unity\Archives\VolumeCloud\001.png)
_Generate Mip Maps 选项_

当做好准备后，我们就可以利用 `tex2Dlod` 来对预生成的 `Mip Maps` 进行采样了：

> **ret tex2Dlod(s, t)**
>
> | Name | In/Out | [**Template Type**](https://learn.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-intrinsic-functions) | [**Component Type**](https://learn.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-intrinsic-functions) | Size |
> | :--- | :----- | :--------------------------------------------------------------------------------------------------------------------- | :---------------------------------------------------------------------------------------------------------------------- | :--- |
> | s    | in     | [**object**](https://learn.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-intrinsic-functions)        | [sampler2D](https://learn.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-sampler)                      | 1    |
> | t    | in     | [**vector**](https://learn.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-intrinsic-functions)        | [**float**](https://learn.microsoft.com/en-us/windows/desktop/WinProg/windows-data-types)                               | 4    |
> | ret  | out    | [**vector**](https://learn.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-intrinsic-functions)        | [**float**](https://learn.microsoft.com/en-us/windows/desktop/WinProg/windows-data-types)                               | 4    |
>
> [tex2Dlod - Win32 apps | Microsoft Learn](https://learn.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-tex2dlod)

可以发现，当前的 `tex2Dlod` 函数相较于 `tex2D`，本来为 `float2` 类型的 UV 变为了 `float4` 类型——新增加了 zw 两位，而其中的 w 位代表了当前 LOD 值，而 z 没有实际意义，常保持为 0：

> [tex2Dlod (nvidia.com)](https://developer.download.nvidia.com/cg/tex2Dlod.html)

我们可以进行一些小小的实验，比和上方展示的结果做对比：

![image-20230331194548990](..\..\images\Rendering\Archives\VolumeCloud\002.png)
_t.z\=\=5000, t.w\=\=0，可以发现结果并没有任何变化_

![image-20230331195527727](..\..\images\Rendering\Archives\VolumeCloud\002dot5.png)
_t.z\=\=5000, t.w\=\=4，高度感仍然比较明显，同时锯齿的感觉也有所减缓_

![image-20230331195108808](..\..\images\Rendering\Archives\VolumeCloud\003.png)
_t.z\=\=5000, t.w\=\=7，可以发现云层的高度起伏已经很不明显了_

![image-20230331194932555](..\..\images\Rendering\Archives\VolumeCloud\004.png)
_t.z\=\=5000, t.w\=\=100，可以说，云层完全失去了高度起伏的感觉并且模糊不堪_

通过以上的对比我们可以发现：当我们不断的增大 LOD 时，云层开始变得越来越模糊，并在 LOD 过大时变得缺乏高度感——也就是说，我们。

不过，tex2Dlod 这样的函数，其逻辑本确实特别直观——在 HLSL 文档的最后，有这样的说明：

> Starting with Direct3D 10, you can use new HLSL syntax to access textures and other resources. You can replace intrinsic-style texture lookup functions, such as tex2Dlod, with a more object-oriented style. In this object-oriented style, textures are decoupled from samplers and have methods for loading and sampling.

HLSL 的解决方案是换用面向对象的 `SampleLevel` 函数，而它的使用方法几乎和 URP 下的 `SAMPLE_TEXTURE2D_LOD` 函数一致：

```cpp
#define SAMPLE_TEXTURE2D_LOD(textureName, samplerName, coord2, lod)                      PLATFORM_SAMPLE_TEXTURE2D_LOD(textureName, samplerName, coord2, lod)
```

其区别在于，HLSL 中的 `SampleLevel` 函数是 `Texture2D` 类型变量的属性，而 `SAMPLER_TEXTURE2D_LOD` 则更加类似于定义于 `D3D11.hlsl` 中的 `'static'` 类型的函数。而对于其中的参数，则可以理解为 `tex2Dlod` 中对应的 `float4 t` 被拆分了，而 LOD 作为独立的参数被写进了函数中。

特别的，`tex2Dlod` 和 `SAMPLER_TEXTURE2D_LOD` 会被专门用在循环当中，例如在本案例中，如果使用 `tex2D` 和 `SAMPLER_TEXTURE2D` 则会报错：

![image-20230331201010767](..\..\images\Rendering\Archives\VolumeCloud\005.png)

> 该问题的出现原因可以参考该回答：[unity3d - Why I can't use tex2D inside a loop in Unity ShaderLab? - Stack Overflow](https://stackoverflow.com/questions/57994423/why-i-cant-use-tex2d-inside-a-loop-in-unity-shaderlab)
> 不严谨地简化解释为：循环中的计算单元不能预测采样的 LOD 值导致了单元间独立而庞大的计算，使循环不能及时地退出进而导致错误

因此，我们需要使用 `tex2Dlod` 类函数来显式地对 `Mip Maps` 进行采样以解决此类问题。Unity Manual 中也提及了这种因为导数计算而需要使用 `tex2Dlod` 类函数的情况：[Unity - Manual: Writing shaders for different graphics APIs (unity3d.com)](https://docs.unity3d.com/Manual/SL-PlatformDifferences.html#Direct3D Shader compiler syntax)

## Reference

- 缘起：[【百人计划先行版】5 分钟教你用视差搓体积云\_哔哩哔哩\_bilibili](https://www.bilibili.com/video/BV1yK4y1U7ow/?vd_source=c8eda79dd90c30ff02e09fb39906ac54)
- 原理参考：[视差贴图 - LearnOpenGL CN (learnopengl-cn.github.io)](https://learnopengl-cn.github.io/05 Advanced Lighting/05 Parallax Mapping/)
- [tex2Dlod - Win32 apps | Microsoft Learn](https://learn.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-tex2dlod)
- <span id="code">[Code - VolumeCloud at URP_LearningPath](https://github.com/JBR-Bunjie/UnityShaderLearningPath/tree/URP_LearningPath/Assets/Shaders/VolumeCloud)</span>
