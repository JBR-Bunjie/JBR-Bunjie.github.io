---
title: Unity Shader入门精要笔记 - Chapter7
date: 2023-3-8 21:28:36
tags:
  - Unity
  - Shader
categories:
  - Unity
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

# 纹理

## Albedo Texture
纹理中最常见的类型，我们用来取代单纯的_Diffuse颜色

## Bump Mapping
法线纹理，我们有两种具体的存储方式：
- 高度纹理/高度图
  - 记录相对高度
- 法线纹理(Default)
  - 模型空间存储
    - 绝对法线信息
  - 切线空间存储(Default)
    - 法线扰动(相对法线信息)
      - 直接在切线空间中计算相关结果
      - 转换到世界空间后再计算结果
    - 可以通过高度图转化而来：->Normal Map->Create From GrayScale，之后Unity会根据高度图生成一张在切线空间下的法线纹理
> 在使用法线纹理前，请先将其Texture Type属性设置为Normal Map，这样我们才能调用Unity中的build-in function来快速解析Normal Map(内置函数能帮我们根据不同的压缩模式来针对性地加载贴图数据)
> 另：关于贴图的压缩模式，可见：http://wiki.polycount.com/wiki/Normal_Map_Compression

# 切线空间

>切线空间定义于每一个顶点之上

## 切线与副切线及其计算

### 切线

在一开始仅有Normal既定的情况下，一点上的切线是存在无数条的

在这所有的切线之中，我们选择与当下点的uv坐标中，u轴同向的那条切线作为我们的T轴或x轴

### 副切线

当我们定好了T轴与N轴后，我们就可以通过cross运算得到我们的B轴：Binormal了

但是我们完成cross运算后得到的副切线存在两个可选的方向，这时，利用Unity在tangent的w分量下预留的值来决定

```shaderlab
fixed3 worldNormal = UnityObjectToWorldNormal(v.normal);
fixed3 worldTangent = UnityObjectToWorldDir(v.tangent.xyz);

fixed tangentSign = v.tangent.w; // * unity_WorldTransformParams.w; // Shader入门精要中未采用，但官方示例中存在
// unity_WorldTransformParams.w，定义于unityShaderVariables.cginc中
// 模型的Scale值是三维向量，即vec3，当这三个值中有奇数个值为负时（1个或者3个值全为负时），unity_WorldTransformParams.w = -1，否则为1.

fixed3 worldBinormal = cross(worldNormal, worldTangent) * tangentSign;
```

## TBN矩阵

用于在 `Object Space` 和 `Tangent Space` 间进行转换

这在实际上类似于view矩阵的变换：

>参见文档`Technical Artist\Opengl\learnopengl-cn.readthedocs.io\01GettingStarted\06Transform.md\1.7.1.2.2.1 View/Camera transformation`

结论为：`做将原点移动至当前顶点位置的变换`，其中由于切线空间更多为向量服务，所以我们更关心坐标轴上的rotation，而非原点到顶点上的transform

**注意区分变换的方向，搞明白当前是从切线空间变换到世界空间还是相反**

## 可参考资料：

- [切线空间（Tangent Space）完全解析 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/139593847)
- U3D内建着色器源码剖析 110页下4.2.8中相关内容
- [关于顶点的法线、切线、副切线 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/103546030)
- [Unity - Scripting API: Mesh.tangents (unity3d.com)](https://docs.unity3d.com/ScriptReference/Mesh-tangents.html)
- [CHAI'S BLOG » 切线的tangent.w的值1或-1的意义 (warmcat.org)](http://warmcat.org/chai/blog/?p=2004)

## 番外：

### `DCC Software`

> 所谓DCC，就是Digital Content Creation的缩写，即数字内容创作。DCC的范围包括二维/三维、音频/视频剪辑合成、动态/互动内容创作、图像编辑等。
> 
> [2.1 DCC工具链与引擎工具链 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/506895643)


## 随记 - 计算光方向

我们开始使用如下built-in函数来取代先前章节用来计算光线向量的方法：
```shaderlab
//fixed3 world_light = normalize(_WorldSpaceLightPos0.xyz);
// =>
fixed3 world_light = UnityWorldSpaceLightDir(i.worldPos);
```

查看该函数源码，可以看到：
```shaderlab
inline float3 UnityWorldSpaceLightDir( in float3 worldPos )
{
    #ifndef USING_LIGHT_MULTI_COMPILE
        return _WorldSpaceLightPos0.xyz - worldPos * _WorldSpaceLightPos0.w;
    #else
        #ifndef USING_DIRECTIONAL_LIGHT //务必注意，这里是`#if n def`，不要误以为平行光会进入该分支！
        return _WorldSpaceLightPos0.xyz - worldPos;
        #else //如果是平行光，则直接返回该点位置
        return _WorldSpaceLightPos0.xyz;
        #endif
    #endif
}
```
## 随记 - 遮罩纹理 Mask Texture 

### AO - Ambient Occlusion - 环境光遮蔽


## 随记 - Texture种类

- 用来定义模型反射率的Tex(MainTex) - Albedo
- 用来定义表面法线的Normal Map - Normal
- 用来控制模型漫反射光照色调的渐变纹理：RampTex
  ```shaderlab
  diffuseColor = tex2D(_RampTex, fixed2(halfLambert, halfLambert)).rgb * _Color.rgb 
  ```
- 用来控制模型

## PROBLEM 1:
> None of the overloads accepts 2 arguments

Pay attention to your vars type
```shaderlab
//float4 _MainTex
//// =>
sampler2D _MainTex
```

## 优化

节约插值寄存器，当两个纹理的操作相同时，仅保留其中一个float4 ST向量