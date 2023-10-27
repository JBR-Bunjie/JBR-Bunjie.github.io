---
title: URP Chapter1 - 从官方示例开始
date: 2023-4-4 21:32:05
tags:
  - Unity
  - Shader
  - URP Pipeline
categories:
  - Unity
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

# Chapter1 - 从官方示例开始

> 本章涵盖内容：
>
> - 官方 `URP Pipeline Custom Shader` 部分的教程与示例

## 曾今的第一个Shader程序

回想一下，我们在入门精要中的第一个unlit shader是什么？

是不是像下面这样的直接输出颜色的shader？

```cpp
Shader "Example/BeforeURPTutorialShaderBasic"{
    /*一个最基础的着色器*/
    Properties{
        _Color("Color Tint", Color) = (1, 1, 1, 1)
    }
    SubShader{
        pass{
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            
            fixed4 _Color;
            
            struct appdata {
                float4 vertex : POSITION;
            };

            struct v2f {
                float4 pos : SV_POSITION;
            };
            
            float4 vert(app_data o):SV_POSITION{
                v2f v;
                v.pos = UnityObjectToClipPos(o.vertex)
                return v;
            }
            
            fixed4 frag(v2f i):SV_TARGET{
                return _BaseColor;
            }
            ENDCG
        }
    }
}
```

是的，这就是我们曾经使用过的，最最基础的Shader。从大体上看，我们定义了：

- 外部汇入Shader的属性
- CG代码块
  - 自定义vertex和fragment Shader名称
  - 构造输入输出数据结构体
  - 使用CG语义编写具体的vertex和fragment Shader内容

通过实现以上的内容，我们完成了built-in管线下的，Shader的编写。

那么，相对的，我们在URP管线中则应该构造这样子的shader来实现对应功能：

```cpp
Shader "Example/URPTutorialShaderBasic" {
    Properties {
        _Color("Color Tint", Color) = (1, 1, 1, 1)
    }
    SubShader {
        // Place.01
        Tags {
            "RenderPipline"="UniversalPipline"
            "RenderType"="Opaque"
        }

        Pass {
        	// Place.02
            HLSLPROGRAM

            #pragma vertex vert
            #pragma fragment frag
        
        	// Place.03
            #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"

            struct appdata {
                float4 vertex : POSITION;
            };

            struct v2f {
                float4 pos : SV_POSITION;
            };

            sampler2D _MainTex;
            float4 _MainTex_ST;
            half4 _Color;

            v2f vert(appdata v) {
                v2f o;
                        
        		// Place.04
        		o.pos = TransformObjectToHClip(v.vertex.xyz);
                // or: o.pos = mul(UNITY_MATRIX_MVP, v.vertex);
                
                return o;
            }

            half4 frag(v2f i) : SV_Target {
                return _Color;
            }
            ENDHLSL
        }
    }
}
```

可以看到，换用URP管线后，有这些变化：

- Place.01：我们得明确地声明该Shader需要使用 `UniversalPipline` 了
- 原来书写的 `CG` 代码块变成了 `HLSL` 代码块
- 原来引入的 `CG` 头文件也变成了对应的 `HLSL` 文件
- 基于我们的头文件的更替，我们换用了一套和CG几乎完全不同的 `API`。

是的，原先在入门精要里用了一整本书的那些CG函数，现在我们都不能用了——我们得重新熟悉这些“换了个马甲”的HLSL函数。

## 为什么是HLSL

CG已经是一个停止更新很久(2012年Nvidia就已经停止了维护)的语言了，而HLSL则一直更新到了现在(截至3/25/2023)，虽然HLSL本身并非一门跨平台的语言(DirextX专用语言)，但是请注意，我们事实上书写的是Shaderlab语言——只是采用了HLSL语法(标准)

## 绘制纹理

### 代码

> 请注意打上 `// Place.0x` 的全部内容

```cpp
Shader "Example/URPTutorialTextureShader" {
 	// Place.01
    Properties {
        [MainTexture] _MainTex ("Basic Texture", 2D) = "white" {}
    }
    SubShader {
        Tags {
 			// Place.02
            "RenderPipline"="UniversalRenderPipeline"
            "RenderType"="Opaque"
        }

        Pass {
            HLSLPROGRAM

            #pragma vertex vert
            #pragma fragment frag
        
            #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
            
            struct appdata {
                float4 vertex : POSITION;
                float2 uv : TEXCOORD0;
            };

            struct v2f {
                float4 pos : SV_POSITION;
                float2 uv : TEXCOORD0;
            };

 			// Place.03
            TEXTURE2D(_MainTex);
            SAMPLER(sampler_MainTex);

            CBUFFER_START(UnityPerMaterial)
                float4 _MainTex_ST;
            CBUFFER_END
            
            v2f vert(appdata v) {
                v2f o;
                        
        		o.pos = TransformObjectToHClip(v.vertex.xyz);
                o.uv = TRANSFORM_TEX(v.uv, _MainTex);
                
                return o;
            }

            half4 frag(v2f i) : SV_Target {
 				// Place.04
                float4 color = SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv);
                return color;
            }
            ENDHLSL
        }
    }
}
```

回想我们读取在 `CGPROGRAM` 中读取 `Texture` 的主要步骤：

- 建立采样器 `sampler2D`
- 获取材质面板上的 `Tiling` 与 `Offset`
- 利用 `tex2D` 来进行最终的取样过程

而我们现在在URP管线的 `HLSLPROGRAM` 中改写的这些内容，就主要集中在以上代码部分中的各个 `Place` 部分：

1. 设置 `[MainTexture]`，这会方便我们在脚本中调用：

  > > **Material.mainTexture**
  > >
  > > By default, Unity considers a texture with the property name `"_MainTex"` to be the main texture. Use the `[MainTexture]` [ShaderLab Properties attribute](https://docs.unity3d.com/Manual/SL-Properties.html) to make Unity consider a texture with a different property name to be the main texture. When the main texture is set using the `[MainTexture]` attribute, it is not visible in the Game view when you use the texture streaming [debugging view mode](https://docs.unity3d.com/Manual/TextureStreaming-API#DebuggingAPI.html) or a custom debug tool.
  >
  > [Unity - Scripting API: Material.mainTexture (unity3d.com)](https://docs.unity3d.com/ScriptReference/Material-mainTexture.html)

2. 改写 `UniversalPipeline` 为 `UniversalRenderPipeline`：事实上这两种写法好像都有效，Unity 2021.3版本的文档也在左右互博：

   > - 在中文URP 12.1.1文档中的 `编写自定义着色器` 一章中，所有对应内容皆为 `UniversalPipeline`
   >   [编写自定义着色器 | Universal RP | 12.1.1 (unity3d.com)](https://docs.unity3d.com/cn/Packages/com.unity.render-pipelines.universal@12.1/manual/writing-custom-shaders-urp.html)
   >
   > - 在英文URP 12.1.10文档中的 `Writing custom shaders` 一章中同样如此
   >   [Writing custom shaders | Universal RP | 12.1.10 (unity3d.com)](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@12.1/manual/writing-custom-shaders-urp.html)
   >
   > - 但是在 `Unity Manual` 中则是这样的：
   >
   >   > **RenderPipeline tag**
   >   >
   >   > The `RenderPipeline` tag tells Unity whether a SubShader is compatible with the Universal Render Pipeline (URP) or the High Definition Render Pipeline (HDRP).
   >   >
   >   > **Syntax and valid values**
   >   >
   >   > | **Signature**               | **Function**                                                 |
   >   > | :-------------------------- | :----------------------------------------------------------- |
   >   > | “RenderPipeline” = “[name]” | Tells Unity whether this SubShader is compatible with URP or HDRP. |
   >   >
   >   > | **Parameter** | **Value**                          | **Function**                                       |
   >   > | :------------ | :--------------------------------- | :------------------------------------------------- |
   >   > | [name]        | UniversalRenderPipeline            | This SubShader is compatible with URP only.        |
   >   > |               | HighDefinitionRenderPipeline       | This SubShader is compatible with HDRP only.       |
   >   > |               | (any other value, or not declared) | This SubShader is not compatible with URP or HDRP. |
   >   >
   >   > [Unity - Manual: ShaderLab: assigning tags to a SubShader (unity3d.com)](https://docs.unity3d.com/2021.3/Documentation/Manual/SL-SubShaderTags.html)
   >
   > 这样明显的左右互搏...所以应该是可以通用的

3. 相较于CG中直接声明sample2D，HLSL中将 `Sampler` 与 `Texture` 区分开来——我们得先声明一个纹理，然后才能为其指定采样器。而之后的，我们在CBUFFER代码块中像 `CGPROGRAM` 中一样，使用 `_ST` 后缀来声明纹理属性

4. API改变——原来的 `tex2D` 变为 `SAMPLE_TEXTURE2D`

### 关于 `HLSLPROGRAM` 中的 `CBuffer` 语句块与采样器





## 获取深度缓冲及重建其世界坐标

> 此示例中的 Unity 着色器使用深度纹理和屏幕空间 UV 坐标来重建像素的世界空间位置。该着色器在网格上绘制棋盘图案，使位置可视化。

> 本节中的代码较官方示例中的代码有所不同：我们额外设置了渲染队列，修改了RenderType

在URP中，我们可以直接在Shader中拿到 `Depth Texture`：通过在 URP 资源的 General 部分中，启用 `Depth Texture`，我们可以直接在ShaderLab中拿到当前的深度缓冲，并利用它来重建世界坐标：

![在 URP 资源中，启用 Depth Texture](..\..\..\images\Dev\Unity\UpdateToURPPipeline\Chapter1_BasicUnlitShader\001.png)
*启用深度缓冲*

具体步骤可以参考以下代码，需要说明的内容已经在注释中注明

```cpp
Shader "Example/URPTutorialReconstructWorldPos" {
    Properties {}
    SubShader {
        Tags {
            "RenderPipeline"="UniversalRenderPipeline"
            "Queue"="Transparent"
            "RenderType"="Transparent"
        }

        Pass {
            HLSLPROGRAM
            
            #pragma vertex vert
            #pragma fragment frag

            #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
            
            // 我们在这里引入了重要的着色器头文件，DeclareDepthTexture.hlsl，它包含了用于对摄像机深度纹理进行采样的函数 SampleSceneDepth
            #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/DeclareDepthTexture.hlsl"
            
            struct appdata {
                float4 vertex : POSITION;
                float2 uv : TEXCOORD0;
            };

            struct v2f {
                float4 pos : SV_POSITION;
            };

            // 可以发现，我们并没有像入门精要那样显式地去声明我们需要使用的深度纹理，因为我们这里并不是后处理步骤
            v2f vert(appdata v) {
                v2f o;
                o.pos = TransformObjectToHClip(v.vertex.xyz);
                
                return o;
            }

            // 我们所需要实现的功能集中在Fragment Shader中
            half4 frag(v2f i) : SV_Target {
                // 由于深度纹理对应的应是屏幕空间，而 i.pos 则对应裁剪空间，那么我们就可以通过继续对 i.pos 进行后续变换来方便我们采样
                // 首先，采样深度缓冲区的 UV 坐标，由像素位置除以渲染目标分辨率 _ScaledScreenParams 得到(透视除法)
                float2 uv = i.pos.xy / _ScaledScreenParams.xy;
                
                // 开始采样，并根据不同的情况调整z值
                #if UNITY_REVERSED_Z
                    real depth = SampleSceneDepth(uv);
                	// 这里我们使用了real类型，这是unity所定义的类型，可参看：https://forum.unity.com/threads/what-does-real-in-shaders-of-lightweight-pipeline-means-it-seems-some-sort-of-number.547402/
                #else
                	// 匹配 opengl 的 NDC
                    real depth = lerp(UNITY_NEAR_CLIP_VALUE, 1, SampleSceneDepth(uv));
                #endif
                //要使重建函数正常工作，深度值必须位于归一化设备坐标 (NDC) 空间中。在 D3D 中，Z 处于 [0,1] 范围内，而在 OpenGL 中，Z 处于 [-1, 1] 范围内

                // 使用 ComputeWorldSpacePosition 函数开始重建，我们呢现在已经具有：NDC 下 xy，NDC 下 z
                float3 worldPos = ComputeWorldSpacePosition(uv, depth, UNITY_MATRIX_I_VP);

                // 可视化世界空间
                uint scale = 10;
                uint3 worldIntPos = uint3(abs(worldPos.xyz * scale));

                bool white = (worldIntPos.x & 1) ^ (worldIntPos.y & 1) ^ (worldIntPos.z & 1);
                // ^ 异或

                half4 color = white ? half4(1, 1, 1, 1) : half4(0, 0, 0, 1);
                
                #if UNITY_REVERSED_Z
                    // 具有 REVERSED_Z 的平台（如 D3D）的情况。
                    if(depth < 0.0001)
                        return half4(0,0,0,1);
                #else
                    // 没有 REVERSED_Z 的平台（如 OpenGL）的情况。
                    if(depth > 0.9999)
                        return half4(0,0,0,1);
                #endif
                
                return color;
            }
            ENDHLSL
        }
    }
}
```

## 小结

In this chapter, we have finished the short official tutorial of Unity,

Thank God, there are not so many challenges and changes. You can see the example below, you can find out that it's almost as same as `CGPROGRAM` we wrote before.

```cpp
Shader "Example/URPTutorialVisualizingNormalShader" {
    Properties {}
    SubShader {
        Tags {
            "RenderPipeline"="UniversalPipeline"
            "RenderType"="Opaque"
        }

        Pass {
            HLSLPROGRAM
            #pragma vertex vert
            #pragma fragment frag

            #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"

            struct appdata {
                float4 vertex : POSITION;
                float3 normal : NORMAL;
            };

            struct v2f {
                float4 pos : SV_POSITION;
                float3 worldNormal : TEXCOORD0;
            };

            v2f vert(appdata v) {
                v2f o;
                o.pos = TransformObjectToHClip(v.vertex);
                o.worldNormal = TransformObjectToWorldNormal(v.normal);
                return o;
            }

            half4 frag(v2f i) : SV_Target {
                return half4(i.worldNormal * 0.5 + 0.5, 1.0);
            }
            ENDHLSL
        }
    }
}
```

