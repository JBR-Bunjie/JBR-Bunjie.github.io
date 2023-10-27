---
title: URP源码分析
date: 2023-4-8 21:32:05
tags:
  - Unity
  - URP
  - Shader
categories:
  - Unity
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

# URP源码分析

## 依赖信息：

- Editor版本：2021.21f1 LTS
- Shader版本：URP 12.1.10

## URP Package总览

首先，当我们下载URP资源时，我们的项目的Package中会出现这些内容：

![image-20230408215042373](..\..\..\images\Dev\Unity\SourceCodeAnalysis\URPSourceCode\001.png)
*URP资源结构 - com.unity.render-pipelines.universal Part*

![image-20230408215738417](..\..\..\images\Dev\Unity\SourceCodeAnalysis\URPSourceCode\002.png)
*URP资源结构 - com.unity.render-pipelines.core Part*

这两个Library一起，组成了我们的URP资源，这也是我们在实际使用时引入资源的两个对应的实际位置：

```cpp
#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
#include "Packages/com.unity.render-pipelines.core/ShaderLibrary/Common.hlsl"
```

由于这两个Package实际上是互相引用的，所以我们就先就着 `Universal RP` 来展开：

- Documentation的包含了大量的说明文档，这些都是帮助我们理解URP资源的好资料。这些内容和官方网站上是完全一致的，所以也没必要就着这里面看——[Universal Render Pipeline overview | Universal RP | 12.1.10 (unity3d.com)](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@12.1/manual/index.html)
- Editor
- Runtime：
- Samples：示例文件、资源及工程
- **ShaderLibrary**
- **Shaders**
- Tests
- Textures

## 从Core.hlsl开始

这是我们最早接触的一个hlsl文件。

总的来说，这个文件做了这么三件事：

- 



### 分析

#### 1~2及196

第1~2行与第196行是防止重复引用的判断语句。

#### 4~6

之后的4~5行是注释：

> // *VT is not supported in URP (for now) this ensures any shaders using the VT*
> // *node work by falling to regular texture sampling.*

并在第6行做了定义：\#define FORCE_VIRTUAL_TEXTURING_OFF 1

快速而全面地了解VT，可以看:

- [Unity - Manual: Marking textures as "Virtual Texturing Only" (unity3d.com)](https://docs.unity3d.com/2020.1/Documentation/Manual/svt-marking-textures.html)
- [Official - Feedback Wanted: Streaming Virtual Texturing - Unity Forum](https://forum.unity.com/threads/feedback-wanted-streaming-virtual-texturing.849133/#:~:text=Streaming Virtual Texturing is a texture streaming feature,tiles to GPU memory when they are needed.)

>Streaming Virtual Texturing is a texture streaming feature that reduces GPU memory usage and Texture loading times when you have many high resolution Textures in your scene. It works by splitting Textures into tiles, and progressively uploading these tiles to GPU memory when they are needed.



#### 8~14

第8~14行，我们定义的是集群渲染，这个在过去文档中有过相关描述，论坛中也有相关讨论
[Unity - Manual: Cluster Rendering (unity3d.com)](https://docs.unity3d.com/560/Documentation/Manual/ClusterRendering.html)
[Any info on experimental URP 12 clustered rendering option? - Unity Forum](https://forum.unity.com/threads/any-info-on-experimental-urp-12-clustered-rendering-option.1238071/)





#### 16~19

引入了四个外部文件，来自core和universal两个位置。这些内容主要做了这些事：

- 

```cpp
#include "Packages/com.unity.render-pipelines.core/ShaderLibrary/Common.hlsl"
#include "Packages/com.unity.render-pipelines.core/ShaderLibrary/Packing.hlsl"
#include "Packages/com.unity.render-pipelines.core/ShaderLibrary/Version.hlsl"
#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Input.hlsl"
```



#### 21~46

21~27行检查并定义了宏 `SHADER_HINT_NICE_QUALITY`，如未定义则默认基于可能存在的移动平台宏 `SHADER_API_MOBILE` 和Switch `SHADER_API_SWITCH` 设为0，其他为1。
[ShaderLibrary - DefineSysmbols_装大炮的自行车的博客-CSDN博客](https://blog.csdn.net/kan464872327/article/details/108618500)

这之后的29~42行，则根据 `SHADER_HINT_NICE_QUALITY` 及实际调用的API输出分别输出 `SHADER_QUALITY_HIGH`, `SHADER_QUALITY_MEDIUM`, `SHADER_QUALITY_LOW`三个宏之一

>// Shader Quality Tiers in Universal.
>// SRP doesn't use Graphics Settings Quality Tiers.
>// We should expose shader quality tiers in the pipeline asset.
>
>// Meanwhile, it's forced to be:
>// High Quality: Non-mobile platforms or shader explicit defined `SHADER_HINT_NICE_QUALITY`
>// Medium: Mobile aside from GLES2
>// Low: GLES2

然后在44~46行对 `SHADER_HINT_NICE_QUALITY` 取反得到宏 `BUMP_SCALE_NOT_SUPPORTED`

#### 49~65

根据具体的平台使用(主要是 `UNITY_REVERSED_Z` 和 `UNITY_UV_STARTS_AT_TOP` 宏)，UNITY做了关于裁剪空间的兼容性处理，内容如下：

```cpp
#if UNITY_REVERSED_Z
    // TODO: workaround. There's a bug where SHADER_API_GL_CORE gets erroneously defined on switch.
    #if (defined(SHADER_API_GLCORE) && !defined(SHADER_API_SWITCH)) || defined(SHADER_API_GLES) || defined(SHADER_API_GLES3)
        //GL with reversed z => z clip range is [near, -far] -> remapping to [0, far]
        #define UNITY_Z_0_FAR_FROM_CLIPSPACE(coord) max((coord - _ProjectionParams.y)/(-_ProjectionParams.z-_ProjectionParams.y)*_ProjectionParams.z, 0)
    #else
        //D3d with reversed Z => z clip range is [near, 0] -> remapping to [0, far]
        //max is required to protect ourselves from near plane not being correct/meaningful in case of oblique matrices.
        #define UNITY_Z_0_FAR_FROM_CLIPSPACE(coord) max(((1.0-(coord)/_ProjectionParams.y)*_ProjectionParams.z),0)
    #endif
#elif UNITY_UV_STARTS_AT_TOP
    //D3d without reversed z => z clip range is [0, far] -> nothing to do
    #define UNITY_Z_0_FAR_FROM_CLIPSPACE(coord) (coord)
#else
    //Opengl => z clip range is [-near, far] -> remapping to [0, far]
    #define UNITY_Z_0_FAR_FROM_CLIPSPACE(coord) max(((coord + _ProjectionParams.y)/(_ProjectionParams.z+_ProjectionParams.y))*_ProjectionParams.z, 0)
#endif
```

原理是：

虽然OpenGL和D3D在从View Space变化到Clip Space中的方式都是类似的，但是实际的值域却有所不同：

1. 屏幕空间差异：当我们使用 `渲染到纹理技术` （render Texture）时，就有可能出现问题，这对应了 `UNITY_UV_STARTS_AT_TOP` 宏

   ![image-20230409155215810](..\..\..\images\Dev\Unity\SourceCodeAnalysis\URPSourceCode\003.png)

2. 



#### 68~104

根据宏 `UNITY_STEREO_INSTANCING_ENABLED` 和宏 `UNITY_STEREO_MULTIVIEW_ENABLED` 的是否启用，对 TEXTURE 类函数做了兼容层。







#### 177~190

定义了两个结构体 `VertexPositionInputs` 和 `VertexNormalnputs`，这是两个我们常用的结构体



#### 193~194

从 universal 中引入了两个外置文件：

```cpp
#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/ShaderVariablesFunctions.hlsl"
#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Deprecated.hlsl"
```

让我们先就着 `ShaderVariablesFunctions.hlsl` 继续





## Reference

- [Universal Render Pipeline overview | Universal RP | 12.1.10 (unity3d.com)](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@12.1/manual/index.html)
- [Unity - Manual: Marking textures as "Virtual Texturing Only" (unity3d.com)](https://docs.unity3d.com/2020.1/Documentation/Manual/svt-marking-textures.html)
- [Official - Feedback Wanted: Streaming Virtual Texturing - Unity Forum](https://forum.unity.com/threads/feedback-wanted-streaming-virtual-texturing.849133/#:~:text=Streaming Virtual Texturing is a texture streaming feature,tiles to GPU memory when they are needed.)
- [ShaderLibrary - DefineSysmbols_装大炮的自行车的博客-CSDN博客](https://blog.csdn.net/kan464872327/article/details/108618500)