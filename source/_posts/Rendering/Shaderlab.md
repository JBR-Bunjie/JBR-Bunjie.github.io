---
title: Shaderlab 总览及其基本执行逻辑
date: 2023-5-2 12:23:23
tags:
  - Unity
  - Shader
categories:
  - Unity
  - Shader
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

# Shaderlab 总览及其基本执行逻辑

## Shaderlab 结构总览

Shaderlab 是 Unity 所使用的，独立的着色器语言。这是一种跨平台的语言体系，大体包含四部分：

- ShaderLab Text
- ShaderLab Compiler
- ShaderLab Asset
- ShaderLab Runtime

我们将一一解释：

## Shaderlab Text

ShaderLab Text 指的是在 `.shader` 文件中，由我们所定义的那些代码。它们需要使用或 `CG` 或 `HLSL` 等的语法规则来编写。官方文档可见：<a href="#ShaderlabManual">Reference</a>

而 ShaderLab Text 的基本结构如下：

```yaml
Shader "<name>" {
    <optional: Material properties> # 暴露在外的材质属性
    <One or more SubShader definitions> # Subshader块是必选的，这是Shader中的实在代码
    <optional: fallback> # 回调Shader
    <optional: custom editor> #指定该Shader需要采用的editor，比如Amplify Shader Editor等
}
```

### Properties

我们利用 `Material properties` 可选项来可以定义在应用了当前 `Shader` 的 `Material` 上显示的数据，而中的格式如下：

```yaml
Properties {
    [optional: attribute] name("display text in Inspector", type name) = default value
    [optional: attribute] name("display text in Inspector", type name) = default value
    ......
}
```

在我们日常编写中，我们不太长见到 `attribute` 的使用，但事实上 `Properties` 中是存在很多类型的 `attribute` 的，注明这些 `attribute` 能让我们在脚本中实时控制 `Properties` 更加方便

特别的，如果使用的是 SRP，想使用 **`SRP Batcher compatibility`** 特性，我们像下方这样必须把每个 Properties 里的变量放到在 HLSL 代码中的 **CBUFFER** 中：
特殊的，贴图和采样器本身不需要再 CBUFFER 中声明，但是其相关参数 `Texel_size` 及 `ST` 则需要

```shaderlab
    Properties {
        _Color ("Colot Tint", Color) = (1, 1, 1, 1)
        _MainTex ("Main Texture", 2D) = "white" {}
        _Layers ("Cloud Level", Range(8, 128)) = 16
        _SpeedX ("Speed X", float) = 0.05
        _SpeedY ("Speed Y", float) = 0.05
        _Alpha ("Cloud Alpha Value", Range(0, 1)) = 0.6
        _HeightOffset ("Height Offset", float) = 5
    }

    SubShader {
    	...
        HLSLINCLUDE

        TEXTURE2D(_MainTex);
        SAMPLER(sampler_MainTex);

        CBUFFER_START(UnityPerMaterial)
            half4 _Color;
            float4 _MainTex_ST;
            float _Layers;
            float _SpeedX;
            float _SpeedY;
            half _Alpha;
            half _HeightOffset;
            CBUFFER_END
        ENDHLSL
        ...
    }
```

### SubShader

SubShader 是主要逻辑的实现部分，它又可以再次细分：

```yaml
SubShader {
    <optional: LOD>
    # Level Of Detial，这个值越小，意味着当前Shader的细节越少，也就是说这个Shader越容易被执行，不过当多个SubShader的LOD都比要求的LOD小时，会从上到下地(第一行代码到最后一行地)执行第一个符合条件的SubShader

    <optional: tags>
    # SubShader Tags，形式为键值对，Unity会根据它们决定何时、如何使用SubShader

    <optional: commands>
    # 诸如Blend、ZTest等命令

    <One or more Pass definitions>
}
```

而最后的 Pass 则是我们实在代码中，具体渲染过程所在的部分了。对于 Pass 本身的结构，可以概括为：

```yaml
Pass {
    <optional: name>
    # 给Pass设置一个名字，以后可以用这个名字在别的Shader中使用UsePass Command来调用这个名字的Pass

    <optional: tags>
    # 和SubShader中的tags大同小异，但工作方式并不一样，具体来看就是所管理的键值对并不相同。下方的commands也是如此，请注意，#pragma这样的语句，不是Command

    <optional: commands>

    <optional: shader code>
    # 最终的Shader Code落脚的地方，包裹在HLSLPROGRAM之类的命令中
}
```

## ShaderLab Compiler

> 光有文本是不行的，就如同你写 C++，如果只是写了一堆 CPP 文件，依然是无法被计算机认可并执行的。中间需要有翻译的过程，就是 shader Compiler 的过程。

简单来说，Unity 会在后台提供的一种服务，用来帮助我们去把写好的 ShaderLab 语言翻译为目标机器能够认可并执行的语言

这事实上包含两层：

- 首先，对于我们的电脑本机来说：`HLSL` 及 `CG` 等代码是不能直接运行在 DirectX 设备上的，而 Unity 的编译器会把我们写好的这些代码编译成可执行的机器语言
- 其次，我们所编写的 `HLSL` 等语言是不能直接运行在 `OpenGL`，`Vulkan` 等平台上的，我们得把代码转译为目标平台山的语言

也就是说，ShaderLab Compiler 的实际行为是：把 ShaderLab Text 翻译成最终目标机器上能够认可和执行的语言，对应的编译代码，我们可以通过点击 Inspect 面板下的 `Compile and show code` 来查看

[Unity - Manual: Shader compilation](https://docs.unity.cn/2021.3/Documentation/Manual/shader-compilation.html)

## ShaderLab Asset

我们已经知道，ShaderLab 里面有很多东西都是不能直接使用的，需要进行翻译。而加工之后得到的东西就叫做 ShaderLab Asset。Asset 比较常见的地方有两个：

一个就是由 Shader 打成的 AssetBundle。另一个是我们打出来的包里面的 level 或 sharedassets 包。

## Reference:

- [【Unity 笔记】ShaderLab 与其底层原理浅谈 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/400470713)

- <span id="ShaderlabManual">[Unity - Manual: ShaderLab (unity3d.com)](https://docs.unity3d.com/2021.3/Documentation/Manual/SL-Reference.html)</span>
