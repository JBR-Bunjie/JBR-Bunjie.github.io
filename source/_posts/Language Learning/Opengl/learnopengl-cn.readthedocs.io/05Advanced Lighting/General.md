---
title: 高级光照
date: 2023-3-8 10:34:08
tags:
  - Opengl
  - Shader
categories:
  - Opengl
  - Shader
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

# Shadow Mapping

## 基本的 Shadow 算法思想：

阴影就是光所不及的地方，也就是说，我只要能获取到哪些地方是光照不到的，那么就能推断出阴影位置。

而一般来说，物体间产生的遮挡关系就是阴影的直接来源，而这个遮挡关系，正好可以用深度关系推测出来。

也就是说，我们以各个光源为原点建立空间系，生成深度图。一个区域如果能被照亮，那么他一定会被记录深度，而没有被记录深度的地方，要么是被裁剪了，要么是被遮挡了。

## 生成 Shadow Map 的大致流程

从上面可以看出来，Shadow Map 是一种 RT，并且是 Per Light 的，并且这种 RT 的生成还和 Light Type 有一定的关联

### Directional Light 的 Shadow Map

#### 生成

1. 顶点着色器：对于所有的 Directional Light，我们在渲染物体时，我们得建立特殊的 view 及 proj 矩阵：
   - view 和 proj 矩阵不再以摄像机为主，我们会以 Light 为关键建立矩阵，以得到独特的 Light Space
2. Fragment：由于我们已经指定了 Depth 的输出对象是一张 RT，并且又没必要处理仍何颜色计算，我们大可以直接将 Fragment 的处理过程空着。GPU 会默认帮我们在完成 Fragment 后设置深度缓冲，当然我们也可以自己在 Frag 中去设置，但是没必要，还会破坏 Early-z

当这么一个 Prepass 执行完后，我们就可以得到一张 RT 了，直接将这张 RT 汇入下一个 Pass 中，就可以使用了

#### 使用

在使用 Shadow Map 时，也是 Per Light 的，因为我们得重新计算一次顶点的 Light Space 坐标，以确认这个顶点是否被照亮

1. 顶点着色器：在正常的顶点属性设置与输出中，必须要增加一条 Light Space 下 Vertex Pos 的输出。我们在 Vertex Shader 阶段完成坐标系的转换以减少 Fragment 阶段的运算量
2. Fragment：对 Shadow Map 进行采样并与当前的 Light Space 坐标进行比对，只有 Equal 才能通过。
   - 关于取样：可预见的，我们可以使用 Screen Pos 下的 LigthPos.xy 对深度图进行取样，如结果与 LightPos.z 不符则可认为在阴影中
   -

对于我们初步生成的 Shadow Map，我们一般会遇到这些问题：

1. 阴影失真：

   > 我们可以看到地板四边形渲染出很大一块交替黑线。这种阴影贴图的不真实感叫做**阴影失真(Shadow Acne)**
   >
   > ![img](https://learnopengl-cn.github.io/img/05/03/01/shadow_mapping_acne.png)

2. 由于使用 Shadow Bias 来解决 Shadow Acne，导致阴影悬浮

   > 使用阴影偏移的直接导致我们对物体的实际深度应用了平移。如果偏移足够大，就可以看出阴影相对实际物体位置的偏移，你可以从下图看到这个现象（这是一个夸张的偏移值）：
   >
   > ![img](https://learnopengl-cn.github.io/img/05/03/01/shadow_mapping_peter_panning.png)
   >
   > > 使用普通的偏移值通常就能避免 peter panning。

3. 光的视锥不可见的区域的处理：

   这事实上包含两个部分：一个是由于 Shadow Map 的大小限制导致的采样超出问题；另一部分则是生成 Shadow Map 时，由于 Near 与 Far 值的设计导致的视锥大小问题

   > ![img](https://learnopengl-cn.github.io/img/05/03/01/shadow_mapping_outside_frustum.png)
   > 未解决任何问题的情况
   >
   > ![img](https://learnopengl-cn.github.io/img/05/03/01/shadow_mapping_clamp_edge.png)
   > 解决了采样问题，现在如果我们采样深度贴图 0 到 1 坐标范围以外的区域，纹理函数总会返回一个 1.0 的深度值，阴影值为 0.0。但是，仍有一部分是黑暗区域。那里的坐标超出了光的正交视锥的远平面。你可以看到这片黑色区域总是出现在光源视锥的极远处。
   >
   > ![img](https://learnopengl-cn.github.io/img/05/03/01/shadow_mapping_over_sampling_fixed.png)
   > 解决全部问题：检查远平面，并将深度贴图限制为一个手工指定的边界颜色，就能“解决”深度贴图采样超出的问题

4. 阴影的锯齿问题：

   这是一个大问题，让我们从 LearnOpenGL 里所给出的解决方案开始，一步步推进：

   > 阴影现在已经附着到场景中了，不过这仍不是我们想要的。如果你放大看阴影，阴影映射对分辨率的依赖很快变得很明显。
   >
   > ![img](https://learnopengl-cn.github.io/img/05/03/01/shadow_mapping_zoom.png)
   >
   > 因为深度贴图有一个固定的分辨率，多个片段对应于一个纹理像素。结果就是多个片段会从深度贴图的同一个深度值进行采样，这几个片段便得到的是同一个阴影，这就会产生锯齿边。
   >
   > 你可以通过增加深度贴图的分辨率的方式来降低锯齿块，也可以尝试尽可能的让光的视锥接近场景。

   **PCF**

   > ...一个（并不完整的）解决方案叫做 PCF（percentage-closer filtering），这是一种多个不同过滤方式的组合，它产生柔和阴影，使它们出现更少的锯齿块和硬边。核心思想是从深度贴图中多次采样，每一次采样的纹理坐标都稍有不同。每个独立的样本可能在也可能不再阴影中。所有的次生结果接着结合在一起，进行平均化，我们就得到了柔和阴影。
   >
   > 一个简单的 PCF 的实现是简单的从纹理像素四周对深度贴图采样，然后把结果平均起来：

   代码：

   ```glsl
   // 最初的shadow值直接来自于深度取样值：
   float shadow = currentDepth - bias > closestDepth  ? 1.0 : 0.0;
   // 现在，我们在一个frag上，对其对应的贴图纹素及周围8个纹素一同采样，并混合结果：
   float shadow = 0.0;
   vec2 texelSize = 1.0 / textureSize(shadowMap, 0);
   for (int x = -1; x <= 1; ++x) {
       for (int y = -1; y <= 1; ++y) {
           float pcfDepth = texture(shadowMap, projCoords.xy + vec2(x, y) * texelSize).r;
           shadow += currentDepth - bias > pcfDepth ? 1.0 : 0.0;
       }
   }
   shadow /= 9.0;
   ```

### Point Light 的 Shadow Map

#### 生成

#### 使用

## HDR
