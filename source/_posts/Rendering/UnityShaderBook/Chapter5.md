---
title: Unity Shader入门精要笔记 - Chapter5
date: 2023-3-8 21:27:04
tags:
  - Unity
  - Shader
categories:
  - Unity
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

# Unity Shader入门精要笔记 - Chapter5

## 对texcoord的解释

### 什么是TEXCOORD

[What are the TEXCOORDs and how can I get them in a ComputeShader? - Unity Forum](https://forum.unity.com/threads/what-are-the-texcoords-and-how-can-i-get-them-in-a-computeshader.573385/)

### 明明没有输入texcoord，怎么会有初始值？

texcoord和position等一样，是保存于模型顶点(应该不是VAO，但是可用VAO来理解)上的一类数据

[Unity Shader基础篇：浅谈TEXCOORDn | 烟雨迷离半世殇的成长之路 (lfzxb.top)](https://www.lfzxb.top/unity-shader-base-texcoordn/)

>- 简单来说texcoord就是存在顶点里的一组数据，我们可以通过这组数据在渲染的时候进行贴图采样，比如我们常用的`第一套uv作为基础纹理，通常基础纹理我们可以根据需求进行一些区域的uv重用（比如左右脸贴图一样，可以映射到统一贴图区域），第二套uv经常用于光照贴图，光照贴图要求是uv不可以重复，所以通常不能用第一套uv，第三套uv用于更加奇特的需求，以此类推...`
>- texcoord应该是更加标准的名称，不过因为这个坐标系里面用uvw作为三个轴名称，所以美术那边普遍称作uv

## 在Unity Shader中，编写时有以下报错：

### 1. 定义Struct报错

Struct定义好后的结尾应该加上分号:

```glsl
struct a2v {
    ...
}; // ; 必须加上
```

### 2. 定义Properties过程中出错

定义Properties时的初始值中，不应该加上"f":

```glsl
_Color("Color Tint", Color) = (1.0, 1.0, 1.0, 1.0)  // √
_Color("Color Tint", Color) = (1.0f, 1.0f, 1.0f, 1.0f)  // ×
```



## Unity ShaderLab Function

我们在可视化纹理坐标的小数部分时，用到了如下语句：

```glsl
o.color = frac(v.texcoord);
if(any(saturate(v.texcoord)) - v.texcoord) {
    o.color.b = 0.5;
}
```



[2.3 HLSL常用函数介绍 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/366143272)

### frac函数

这是HLSL的内置函数，其作用是去取x的小数部分

逻辑为：

```glsl
float frac(float v) {
  return v - floor(v);
}
```

可以参考：[Shader实验室：frac函数 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/158462351)

> 它在坐标轴上的效果如下：
>
> ![img](https://pic2.zhimg.com/80/v2-bfd711ced58f0334e8303ff820593c1d_720w.webp)
>
> **周期性：而如果我们将对x*2，则可以将该图像的周期翻倍**

#### length

这是上述参考内容中引用的其他HLSL函数，其功能为：

[length - Win32 apps | Microsoft Learn](https://learn.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-length)

>Returns the length of the specified floating-point vector.

示例：

[shader - GLSL - length function - Stack Overflow](https://stackoverflow.com/questions/48370604/glsl-length-function)

### saturate函数

[saturate (HLSL reference) - Win32 apps | Microsoft Learn](https://learn.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-saturate?redirectedfrom=MSDN)

saturate（x） 返回将x钳制到[0,1]范围之间的值；

### any函数

[any - Win32 apps | Microsoft Learn](https://learn.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-any)

> Determines if any components of the specified value are non-zero.
>
> Return **true** if any components of the *x* parameter are non-zero; otherwise, **false**.

