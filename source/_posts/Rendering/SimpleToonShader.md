---
title: UnityURPToonLitShaderExample 源码分析
date: 2023-5-3 12:23:23
tags:
  - Unity
  - Shader
categories:
  - Unity
  - Shader
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

# 最简的 Toon Shader 的源码分析 - UnityURPToonLitShaderExample

> **What is included in this "simplified version" toon lit shader repository?**
>
> This repository contains a very simple toon lit shader example, to help people writing their first custom toon lit shader in URP.
> This example shader's default result(without editing material params) = the following picture
>
> ![screenshot](..\..\images\Dev\Unity\SourceCodeAnalysis\SimpleToonShaderCodeAnalysis\001.png)
>
> > **About this repository**
> >
> > This repository is NOT the full version NiloToonURP. This repository only contains a very simple and short URP toon shader example, only for tutorial purposes, it is under MIT license so you can do whatever you want with the code. If you want to keep the current tutorial shader, please fork it or download a copy now since it may be removed in the future.
>
> [ColinLeung-NiloCat/UnityURPToonLitShaderExample: A very simple toon lit shader example, for you to learn writing custom lit shader in Unity URP (github.com)](https://github.com/ColinLeung-NiloCat/UnityURPToonLitShaderExample)

> 如其所说，这实际上是一个过度简化的 Toon Shader，基本不具有太强的可用性，但这也同样意味着，这是一个不错的起点。
>
> ~~虽然我们看到上面的结果还算不错，但是基本上不太可能复现出来~~

## 项目结构：

容易看出，该项目一共仅包含 8 份文件，而实际在项目中生效的文件为：

- **SimpleURPToonLitOutlineExample.shader**
- SimpleURPToonLitOutlineExample_Shared.hlsl
- NiloOutlineUtil.hlsl
- NiloZOffset.hlsl
- NiloInvLerpRemap.hlsl
- SimpleURPToonLitOutlineExample_LightingEquation.hlsl

这是一个组织上还算不赖的项目，不过考虑到未来扩建的可能性，可以考虑将 `SimpleURPToonLitOutlineExample_Shared.hlsl` 进行进一步的拆分：将该文件拆分为 `Input` 和 `SharedFunction` 两个文件

让我们从 `SimpleURPToonLitOutlineExample.shader` 开始进行代码的分析：

## 代码与效果分析：

### shader 文件主体结构

从头看起，首先在 9~14 行的注释中，有提到该 Shader 的 5 个 Passes：

|                    |            |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| ------------------ | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| _ForwardLit_       | _pass_     | _(this pass will always render to the color buffer \_CameraColorTexture)_                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| _Outline_          | _pass_     | _(this pass will always render to the color buffer \_CameraColorTexture)_                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| _ShadowCaster_     | _pass_     | _(only for URP's shadow mapping, this pass won't render at all if your character don't cast shadow)_                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| _DepthOnly_        | _pass_     | _(only for URP's depth texture \_CameraDepthTexture's rendering, this pass won't render at all if your project don't render URP's offscreen depth prepass)_                                                                                                                                                                                                                                                                                                                                                                                |
| ~~_DepthNormals_~~ | ~~_pass_~~ | ~~_(only for URP's normal texture \_CameraNormalsTexture's rendering)_~~<br />实际上项目并没有对这个 Pass 做出实现<br />当 Pass 的**LightMode**为**DepthNormals**时，(且在 Assert 中勾选 Depth Texture 时)Unity 会生成\_CameraDepthTexture 与\_CameraNormalsTexture, 最终传入到 Shader 中的\_CameraDepthNormalsTexture，关于\_CamerDepthTexture，可见：<br />- https://docs.unity3d.com/Manual/SL-CameraDepthTexture.html<br />- https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@12.1/manual/universalrp-asset.html |

[Universal Render Pipeline Asset | Universal RP | 12.1.11 (unity3d.com)](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@12.1/manual/universalrp-asset.html)

我们需要处理的环节：

一个 high level setting：\_IsFace

https://www.bilibili.com/read/cv6504414/

### 渲染流程

#### Shader file

第一个 Pass 从 117 开始，到 171 行结束。这是 Shader 中的第一个 Pass，该 Pass 完成了从间接光到直接光的计算.
这里对间接光的定义是，而直接光则是 MainLight 和 AdditionalLight 的作用集合

先进行光照的运算，这意味着模型的正面最早被渲染，这样可以最大程度地减少 OverDraw 的情况。

整个 Shader file 的四个 Pass 的流程都差不太多，所以针对 Shader 内的各个 Pass 的流程仅作一次说明：

- 在 123 行，定义 Pass 的 Name
- 在 124~131 行，是 Pass Tags，这里定义了 Pass 的 LightMode：`"LightMode" = "UniversalForward"`，这意味这个 Pass 应该：`Render object geometry and evaluate all light contributions`
- 在 135~138 行，设置 Pass State
- 在 140~170 行，HLSLPROGRAM 2 ENDHLSL
  - 142~159 行，设置关键字以获取 Shader 变体
  - 161~162 行，设置 Vertex 和 fragment 函数
  - 168 行，引入实在代码 `SimpleURPToonLitOutlineExample_Shared.hlsl`

深入 `SimpleURPToonLitOutlineExample_Shared.hlsl` 文件：

#### HLSL Entities

##### Vertex Function: VertexShaderWork

`SimpleURPToonLitOutlineExample_Shared.hlsl` 文件的 153~217 行是我们的 Vertex Shader：

而由于在单纯的 UniversalForward 中没有定义任何控制用的宏，所以我们 Vertex Shader 只会执行最基本的代码，内容仅包括：

- 坐标系转换
- uv 计算

去掉注释后的代码如下：

```cpp
Varyings VertexShaderWork(Attributes input) {
    Varyings output;

    VertexPositionInputs vertexInput = GetVertexPositionInputs(input.positionOS);
    VertexNormalInputs vertexNormalInput = GetVertexNormalInputs(input.normalOS, input.tangentOS);

    float3 positionWS = vertexInput.positionWS;

    // Computes fog factor per-vertex.
    float fogFactor = ComputeFogFactor(vertexInput.positionCS.z);

    // TRANSFORM_TEX is the same as the old shader library.
    output.uv = TRANSFORM_TEX(input.uv,_BaseMap);

    // packing positionWS(xyz) & fog(w) into a vector4
    output.positionWSAndFogFactor = float4(positionWS, fogFactor);
    output.normalWS = vertexNormalInput.normalWS; //normlaized already by GetVertexNormalInputs(...)

    output.positionCS = TransformWorldToHClip(positionWS);

    return output;
}
```

###### 坐标转换过程

我们只使用了 vertexInput 中的 positionWS，这里用两个原因：

首先，如果我们对模型的顶点作出修改后，我们应该将这种变化同步到所有别的空间下。而 Fragment Shader 中会同时用到世界坐标与裁剪坐标，这意味着 Vertex Shader 中需要将“最终的世界坐标与裁剪坐标”上载到 v2f 中。为了简化流程，我们直接在世界空间下操作，而后对完成处理的世界坐标再另做一次坐标转换得到裁剪坐标。

###### 雾效

关于 Unity 的内置雾效：

> 雾效(Fog)是游戏里经常使用的一种效果。Unity 内置的雾效可以产生基于距离的线性或指数雾效。然而，要想在自己编写的顶点/片元着色器中实现这些雾效，我们需要在 Shader 中添加 #pragma multi_compile_fog 指令，同时还需要使用相关的内置宏，例如 UNITY_FOG_COORD、UNITY_TRANSFER_FOG 和 UNTTY_APPLY_FOG 等。这种方法的缺点在于，我们不仅需要为场景中所有物体添加相关的渲染代码，而且能够实现的效果也非常有限。**当我们需要对雾效进行一些个性化操作时，例如使用基于高度的雾效等，仅仅使用 Unity 内置的雾效就变得不再可行**

![image-20230418220205577](..\..\images\Dev\Unity\SourceCodeAnalysis\SimpleToonShaderCodeAnalysis\002.png)
Unity Built-in Fog

关于 Unity Built-in Fog，可参考：[Rendering 14 (catlikecoding.com)](https://catlikecoding.com/unity/tutorials/rendering/part-14/)

##### Fragment

等到了 Fragment Shader 中，我们就开始正式计算光源对物体的影响了：

```cpp
half4 ShadeFinalColor(Varyings input) : SV_TARGET {
    // fillin ToonSurfaceData struct:
    ToonSurfaceData surfaceData = InitializeSurfaceData(input);

    // fillin ToonLightingData struct:
    ToonLightingData lightingData = InitializeLightingData(input);

    // apply all lighting calculation
    half3 color = ShadeAllLights(surfaceData, lightingData);

    return half4(color, surfaceData.alpha);
}
```

可以看到的是，Fragment Shader 中首先仍然是对数据的准备：利用我们传入的 Varyings 数据定义了两个结构体： `ToonSurfaceData` 和 `ToonLightingData`：

```cpp
struct ToonSurfaceData {
    half3   albedo;
    half    alpha;
    half3   emission;
    half    occlusion;
};
struct ToonLightingData {
    half3   normalWS;
    float3  positionWS;
    half3   viewDirectionWS;
    float4  shadowCoord;
};
```

它们分别对应了一个初始化函数：

```cpp
ToonSurfaceData InitializeSurfaceData(Varyings input) {
    ToonSurfaceData output;

    // albedo & alpha
    float4 baseColorFinal = GetFinalBaseColor(input);
    output.albedo = baseColorFinal.rgb;
    output.alpha = baseColorFinal.a;
    DoClipTestToTargetAlphaValue(output.alpha);// early exit if possible

    // emission
    output.emission = GetFinalEmissionColor(input);

    // occlusion
    output.occlusion = GetFinalOcculsion(input);

    return output;
}

ToonLightingData InitializeLightingData(Varyings input) {
    ToonLightingData lightingData;
    lightingData.positionWS = input.positionWSAndFogFactor.xyz;
    lightingData.viewDirectionWS = SafeNormalize(GetCameraPositionWS() - lightingData.positionWS);
    lightingData.normalWS = normalize(input.normalWS); //interpolated normal is NOT unit vector, we need to normalize it

    return lightingData;
}
```

在完成初始化后，利用所得的信息，计算光照影响：

```cpp
half3 color = ShadeAllLights(surfaceData, lightingData);
color = ApplyFog(color, input);
```

最后将 color 作为结果进行返回，完成一次渲染

这其中的 ShadeAllLights 必须考虑全部的 Light 的影响，

```cpp
half3 ShadeAllLights(ToonSurfaceData surfaceData, ToonLightingData lightingData)
{
    // Indirect lighting
    half3 indirectResult = ShadeGI(surfaceData, lightingData);

    //////////////////////////////////////////////////////////////////////////////////
    // Light struct is provided by URP to abstract light shader variables.
    // It contains light's
    // - direction
    // - color
    // - distanceAttenuation
    // - shadowAttenuation
    //
    // URP take different shading approaches depending on light and platform.
    // You should never reference light shader variables in your shader, instead use the
    // -GetMainLight()
    // -GetLight()
    // funcitons to fill this Light struct.
    //////////////////////////////////////////////////////////////////////////////////

    //==============================================================================================
    // Main light is the brightest directional light.
    // It is shaded outside the light loop and it has a specific set of variables and shading path
    // so we can be as fast as possible in the case when there's only a single directional light
    // You can pass optionally a shadowCoord. If so, shadowAttenuation will be computed.
    Light mainLight = GetMainLight();

    float3 shadowTestPosWS = lightingData.positionWS + mainLight.direction * (_ReceiveShadowMappingPosOffset + _IsFace);
#ifdef _MAIN_LIGHT_SHADOWS
    // compute the shadow coords in the fragment shader now due to this change
    // https://forum.unity.com/threads/shadow-cascades-weird-since-7-2-0.828453/#post-5516425

    // _ReceiveShadowMappingPosOffset will control the offset the shadow comparsion position,
    // doing this is usually for hide ugly self shadow for shadow sensitive area like face
    float4 shadowCoord = TransformWorldToShadowCoord(shadowTestPosWS);
    mainLight.shadowAttenuation = MainLightRealtimeShadow(shadowCoord);
#endif

    // Main light
    half3 mainLightResult = ShadeSingleLight(surfaceData, lightingData, mainLight, false);

    //==============================================================================================
    // All additional lights

    half3 additionalLightSumResult = 0;

#ifdef _ADDITIONAL_LIGHTS
    // Returns the amount of lights affecting the object being renderer.
    // These lights are culled per-object in the forward renderer of URP.
    int additionalLightsCount = GetAdditionalLightsCount();
    for (int i = 0; i < additionalLightsCount; ++i)
    {
        // Similar to GetMainLight(), but it takes a for-loop index. This figures out the
        // per-object light index and samples the light buffer accordingly to initialized the
        // Light struct. If ADDITIONAL_LIGHT_CALCULATE_SHADOWS is defined it will also compute shadows.
        int perObjectLightIndex = GetPerObjectLightIndex(i);
        Light light = GetAdditionalPerObjectLight(perObjectLightIndex, lightingData.positionWS); // use original positionWS for lighting
        light.shadowAttenuation = AdditionalLightRealtimeShadow(perObjectLightIndex, shadowTestPosWS); // use offseted positionWS for shadow test

        // Different function used to shade additional lights.
        additionalLightSumResult += ShadeSingleLight(surfaceData, lightingData, light, true);
    }
#endif
    //==============================================================================================

    // emission
    half3 emissionResult = ShadeEmission(surfaceData, lightingData);

    return CompositeAllLightResults(indirectResult, mainLightResult, additionalLightSumResult, emissionResult, surfaceData, lightingData);
}
```

#### 小结

### 描边流程

#### 基本思路：

描边的基本思路：顶点扩张。利用

$$
VertexPos\_Outline = VertexPos + NormalDir * ControlVariable
$$

来计算 Outline Pass 下的受到扩张的顶点坐标。再裁剪掉正面的三角形即可。

#### 实现过程：

Shader 的 173 行到 223 行对应了 Outline 的流程，这里做的事情包括：

- 180 行：设置 Pass 的 Name
- 181~196 行：用于设置 Pass Tags，这里的注释指出，我们可以通过为当前 Pass 设置一个我们自定义的 LightMode：`"LightMode"=="YourCustomPassTag"`，通过这种方式，我们可以利用 URP 的 RenderFeature 以及 cmd.DrawRenderers()来控制我们的渲染来最大化利用 SRP 批处理。关于 RenderFeature，可参考：
  [How to create a custom Renderer Feature | Universal RP | 12.1.11 (unity3d.com)](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@12.1/manual/renderer-features/create-custom-renderer-feature.html)
  [Unity - Scripting API: CommandBuffer (unity3d.com)](https://docs.unity3d.com/2021.2/Documentation/ScriptReference/Rendering.CommandBuffer.html)
- 198 行，`Cull Front`
- 200~222 行，HLSLPROGRAM 2 ENDHLSL
  - 202 行到 211 行，继承了 ForwardLit 的关键字
  - 213~214 行，设置 vertex 和 fragment 函数
  - 217 行，定义宏 `ToonShaderIsOutline` 来控制渲染流程——这是我们整个渲染中，相当重要的一环
  - 220 行，引入实在代码 `SimpleURPToonLitOutlineExample_Shared.hlsl`

让我们继续深入 `SimpleURPToonLitOutlineExample_Shared.hlsl` 文件

#### HLSL Entities

##### Vertex

##### Fragment

为什么描边需要 fov 和 Distance？

## Shader 变体：

## 后记

> **What is NOT included in this simplified example shader?**
>
> For simplicity reason, I removed most of the features from the NiloToonURP (deleted 90% of the original shader), else this example shader will be way too complex for reading & learning. The removed features are:
>
> - face anime lighting (auto-fix face ugly lighting due to vertex normal without modifying .fbx, very important)
> - smooth outline normal auto baking (fix ugly outlines without modifying .fbx once you attach a script on character, very important)
> - auto 2D hair shadow on face (very important, it is very difficult to produce good looking shadow result using shadowmap)
> - sharp const width rim light (Blue Protocol / Genshin Impact)
> - tricks to render eye/eyebrow over hair
> - hair "angel ring" reflection
> - PBR specular lighting (GGX)
> - HSV control shadow & outline color
> - 2D mouth renderer
> - almost all the extra texture input options like roughness, specular, normal map, detail map...
> - LOTS of sliders to control lighting, final color & outline
> - per character "dither fadeinout / rim light / tint / lerp..." control script
> - volume override control of global "dither fadeinout / rim light / tint / lerp..."
> - anime postprocessing
> - auto phong tessellation
> - perspective removal per character
> - \*\*\*just too much for me to write all removed feature here, the full / lite version shader is a totally different level product

## Reference

- [ColinLeung-NiloCat/UnityURPToonLitShaderExample: A very simple toon lit shader example, for you to learn writing custom lit shader in Unity URP (github.com)](https://github.com/ColinLeung-NiloCat/UnityURPToonLitShaderExample)
- [【Unity】URP 源码分析(Pass 篇) - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/555291588)
- [Universal Render Pipeline Asset | Universal RP | 12.1.11 (unity3d.com)](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@12.1/manual/universalrp-asset.html)
- [Unity - Manual: Cameras and depth textures (unity3d.com)](https://docs.unity3d.com/Manual/SL-CameraDepthTexture.html)
- [【浅入浅出】Unity 雾效 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/144845123)
