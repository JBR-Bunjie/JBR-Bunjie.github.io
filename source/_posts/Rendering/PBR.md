---
title: PBR
date: 2023-5-1 12:23:23
tags:
  - Unity
  - Shader
categories:
  - Unity
  - Shader
mathjax: true
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

# PBR

> PBR，或者用更通俗一些的称呼是指基于物理的渲染(Physically Based Rendering)，它指的是一些在不同程度上都基于与现实世界的物理原理更相符的基本理论所构成的渲染技术的集合。

## PBR 综述

**PBR 是基于物理的渲染，具体表现为使用一种更符合物理学规律的方式来模拟光线**，这种渲染方式与我们原来的 Phong 或者 Blinn-Phong 光照算法相比总体上看起来要更真实一些。除了看起来更好些以外，由于它与物理性质非常接近，因此我们可以直接以物理参数为依据来编写表面材质，而不必依靠粗劣的修改与调整来让光照效果看上去正常。使用基于物理参数的方法来编写材质还有一个更大的好处，就是不论光照条件如何，这些材质看上去都会是正确的，而在非 PBR 的渲染管线当中有些东西就不会那么真实了。

关于 PBR，有些有趣的东西：[在 PBR 出现之前，3d 界如何做”真实感“渲染的？ - 知乎 (zhihu.com)](https://www.zhihu.com/question/425289490)，其中的这条[回答](https://www.zhihu.com/question/425289490/answer/1522232868)比较有意思：

一个误区是 PBR 一定是真实感的渲染，这当然是不完全正确的：比如迪士尼的电影中大量使用 PBR，但是这些动画电影很难说得上是什么“真实感渲染”的结果

> PBS 是为了对光和材质之间的行为进行更加真实的建模。
>
> ...
>
> 在把 PBS 入当前的游戏项目之前我们需要权衡一下它的优缺点。**需要再次提醒读者的是，PBS 并不意味着游戏画面需要追求和照片一样真实的效果。**事实上，很多游戏都不需要刻意去追求与照片一样的真实感，玩家眼中的真实感大多也并不是如此 PBS 优点在于 我们只需要个万能的 shader 就可以渲染相当一大部分类型的材质而不是使用传统的做法为每种材质写特定的 shader。同时，**PBS 可以保证在各种光照条件下，材质都可以自然地和光源进行交互，而不需要我们反复地调整材质参数。**
>
> 然而，在使用 PBS 时我们也需要考虑到它带来的代价。如上面提到的，PBS 往往需要更复杂的光照配合，例如大量使用光照探针和反射探针等。而且 PBS 需要开启 HDR 以及一些必不可少的屏幕特效，例如抗锯齿、Bloom 和色调映射，如果这些屏幕特效对当前游戏来说需要消耗过多的性能，那么 PBS 就不适合当前的游戏，我们应该使用传统的 shader 来渲染游戏。使用 PBS 对美工人员来说同样是个挑战。美术资源的制作过程和使用传统的 shader 有很大不同，普通的法 线纹理＋高光反射纹理的组合不再适用，我们需要创建更细腻复杂的纹理集，包括金属值纹理、高光反射纹理、粗糙度纹理、遮挡纹理，有些还需要使用额外的细节纹理来给材质添加更多的细节表面。除了使用图片扫描的传统辅助方法外，这些纹理的制作通常还需要更专业的工具来绘制例如 Allegorithmic Substance Painter Quixel Suite
>
> ——Unity Shader 入门精要

不过，即使是“基于物理的渲染”，其本质上仍然只是对基于物理原理的现实世界的一种近似(我们稍后就会法线，PBR 流程中依然存在着很多妥协)，这也就是为什么它被称为**基于**物理的着色(Physically based Shading) 而非物理着色(Physical Shading)的原因。

判断一种光照模型是否是基于物理的，必须满足以下三个条件（不用担心，我们很快就会了解它们的）：

1. 基于微平面(Microfacet)的表面模型。
2. 能量守恒。
3. 应用基于物理的 BRDF。

我们可以反过来说：PBR 并不是某一种特指的算法，而是一类算法的统称。只要一种着色算法满足一定的物理规律（能量守恒等），我们就认为它属于 PBR 算法，反之，则不属于 PBR 算法。

## PBR 理论

### 微平面理论

**所有的 PBR 技术都基于微平面理论。**这项理论认为，达到微观尺度之后任何平面都可以用被称为微平面(Microfacets)的细小镜面来进行描绘。根据平面粗糙程度的不同，这些细小镜面的取向排列可以相当不一致：

![rought_smooth.png-64.6kB](http://static.zybuluo.com/candycat/zmrvmeo27ach7kjpxm1grnwb/rought_smooth.png)
左图：光滑表面的微平面的法线变化较小，反射光线的方向变化也更小
右图：粗糙表面的微平面的法线变化较大，反射光线的方向变化也更大

这些微小镜面这样无序取向排列的影响就是：一个平面越是粗糙，这个平面上的微平面的排列就越混乱，法线变化也就越频繁且混乱，这样一来，当我们特指镜面光/镜面反射时，入射光线更趋向于向完全不同的方向发散(Scatter)开来，进而产生出分布范围更广泛的镜面反射，其结果也就越”钝“；相对的，对于一个光滑的平面，光线大体上会更趋向于向同一个方向反射，造成更小更锐利的反射。

在这样的微观尺度下，我们认为没有任何平面是完全光滑的。然而由于这些微平面已经微小到无法逐像素地继续对其进行区分，因此我们假设一个粗糙度(Roughness)参数，然后用统计学的方法来估计微平面的粗糙程度。我们可以基于一个平面的粗糙度来计算出众多微平面中，朝向方向沿着某个向量 $h$ 方向的比例。这个向量 $h$ 便是位于光线向量 $l$ 和视线向量 $v$ 之间的半程向量(Halfway Vector)。

而微平面的朝向方向与半程向量的方向越是一致，镜面反射的效果就越是强烈越是锐利。通过使用一个介于 0 到 1 之间的粗糙度参数，我们就能概略地估算微平面的取向情况了：

![img](https://learnopengl-cn.github.io/img/07/01/ndf.png)

我们可以看到，较高的粗糙度显示出来的镜面反射的轮廓要更大一些。与之相反，较小的粗糙度显示出的镜面反射轮廓则更小更锐利。

### 能量守恒

**微平面近似法使用了这样一种形式的能量守恒(Energy Conservation)：出射光线的能量永远不能超过入射光线的能量（发光面除外）**

如上图我们可以看到，随着粗糙度的上升，镜面反射区域会增加，但是镜面反射的亮度却会下降。如果每个像素的镜面反射强度都一样（不管反射轮廓的大小），那么粗糙的平面就会放射出过多的能量，而这样就违背了能量守恒定律。这也就是为什么正如我们看到的一样，光滑平面的镜面反射更强烈而粗糙平面的反射更昏暗。

**为了遵守能量守恒定律，我们需要对漫反射光和镜面反射光做出明确的区分。**

_当一束光线碰撞到一个表面的时候，它就会分离成一个折射部分和一个反射部分_：反射部分就是会直接反射开而不进入平面的那部分光线，也就是我们所说的镜面光照。而折射部分就是余下的会进入表面并被吸收的那部分光线，也就是我们所说的漫反射光照。

这里还有一些细节需要处理，因为当光线接触到一个表面的时候折射光是不会立即就被吸收的。通过物理学我们可以得知，光线实际上可以被认为是一束没有耗尽就不停向前运动的能量，而光束是通过碰撞的方式来消耗能量。每一种材料都是由无数微小的粒子所组成，这些粒子都能如下图所示一样与光线发生碰撞。这些粒子在每次的碰撞中都可以吸收光线所携带的一部分或者是全部的能量而后转变成为热量。

![img](https://learnopengl-cn.github.io/img/07/01/surface_reaction.png)

一般来说，并非全部能量都会被吸收，而光线也会继续沿着（基本上）随机的方向发散，然后再和其他的粒子碰撞直至能量完全耗尽或者再次离开这个表面。而光线脱离物体表面后将会协同构成该表面的（漫反射）颜色。不过在基于物理的渲染之中我们进行了简化，假设对平面上的每一点所有的折射光都会被完全吸收而不会散开。而有一些被称为 `次表面散射(Subsurface Scattering)` 技术的着色器技术将这个问题考虑了进去，它们显著地提升了一些诸如皮肤，大理石或者蜡质这样材质的视觉效果，不过伴随而来的代价是性能的下降。

对于金属(Metallic)表面，当讨论到反射与折射的时候还有一个细节需要注意。金属表面对光的反应与非金属（也被称为介电质(Dielectrics)）表面相比是不同的。它们遵从的反射与折射原理是相同的，但是**所有的**折射光都会被直接吸收而不会散开，只留下反射光或者说镜面反射光。亦即是说，金属表面只会显示镜面反射颜色，而不会显示出漫反射颜色。由于金属与电介质之间存在这样明显的区别，因此它们两者在 PBR 渲染管线中被区别处理，而我们将在文章的后面进一步详细探讨这个问题。

反射光与折射光之间的这个区别使我们得到了另一条关于能量守恒的经验结论：反射光与折射光它们二者之间是**互斥**的关系。无论何种光线，其被材质表面所反射的能量将无法再被材质吸收。因此，诸如折射光这样的余下的进入表面之中的能量正好就是我们计算完反射之后余下的能量。

我们按照能量守恒的关系，首先计算镜面反射部分，它的值等于入射光线被反射的能量所占的百分比。然后折射光部分就可以直接由镜面反射部分计算得出：

```c++
float kS = calculateSpecularComponent(...); // 反射/镜面 部分
float kD = 1.0 - ks;                        // 折射/漫反射 部分
```

这样我们就能在遵守能量守恒定律的前提下知道入射光线的反射部分与折射部分所占的总量了。按照这种方法折射/漫反射与反射/镜面反射所占的份额都不会超过 1.0，如此就能保证它们的能量总和永远不会超过入射光线的能量。而这些都是我们在前面的光照教程中没有考虑的问题。

## 反射率方程

在这里我们引入了一种被称为[渲染方程](https://learnopengl.com/wiki-rendereuqation)(Render Equation)的东西。它是某些聪明绝顶的人所构想出来的一个精妙的方程式，是如今我们所拥有的用来模拟光的视觉效果最好的模型。基于物理的渲染所坚定遵循的是一种被称为反射率方程(The Reflectance Equation)的渲染方程的特化版本。要正确地理解 PBR，很重要的一点就是要首先透彻地理解反射率方程

反射率方程简写：

$$
L_0(p,w_0)=\int_Ωf_r(p,w_i,w_0)Li(p,w_i)n \cdot w_idw_i
$$

如果补全绝大多数内容的话，我们能得到一个较为完整的式子：

$$
L_0(p,w_0)=\int_Ω(k_d*\frac{c}{\pi}+k_s*\frac{DGF}{4*(w_0 \cdot n)(w_i \cdot n)}*Li(p,w_i)*n \cdot w_i*dw_i
$$

其中的构建思路

$$
输出颜色=(漫反射比例\frac{纹理颜色}{\pi}+镜面反射比例*\frac{镜面高光\times几何遮蔽\times菲涅尔效应}{4(viewDir\cdot normal)(lightDir\cdot normal)光源颜色})(lightDir\cdot normal)
$$

我们可以逐步细分这个方程：

### 关于能量

### BRDF

![img](https://github.com/QianMo/PBR-White-Paper/raw/master/content/part%201/media/8c60f94b8b6f430fc5dcb41068770454.png)

#### Cook-Torrance 反射率方程

##### D：法线分布函数(Normal Distribution Function，NDF)

> 法线分布函数 `D`，从统计学上近似地表示了与某些（半程）向量 h 取向一致的微平面的比率。举例来说，假设给定向量 h，如果我们的微平面中有 35%与向量 h 取向一致，则法线分布函数或者说 NDF 将会返回 0.35。目前有很多种 NDF 都可以从统计学上来估算微平面的总体取向度，只要给定一些粗糙度的参数。我们马上将要用到的是 Trowbridge-Reitz GGX

若用 h 表示用来与平面上微平面做比较用的半程向量，a 表示表面粗糙度，则可以有 NDF 方程如下：

$$
NDF_{GGXTR}(n, h, \alpha) = \frac{\alpha^{2}}{\pi((n\cdot h)^{2}(\alpha^{2}-1)+1)^2}
$$

![img](https://learnopengl-cn.github.io/img/07/01/ndf.png)

当粗糙度很低（也就是说表面很光滑）的时候，与半程向量取向一致的微平面会高度集中在一个很小的半径范围内。由于这种集中性，NDF 最终会生成一个非常明亮的斑点。但是当表面比较粗糙的时候，微平面的取向方向会更加的随机。你将会发现与 hℎ 向量取向一致的微平面分布在一个大得多的半径范围内，但是同时较低的集中性也会让我们的最终效果显得更加灰暗。

实现：

```glsl
float DistributionGGX(vec3 N, vec3 H, float roughness) {
    float a      = roughness * roughness; // 注意这里，a并非是纯粹的粗糙度
    float a2     = a * a;

    float NdotH  = max(dot(N, H), 0.0);
    float NdotH2 = NdotH*NdotH;

    float num   = a2;
    float denom = (NdotH2 * (a2 - 1.0) + 1.0);
    denom = PI * denom * denom;

    return num / denom; //molecular&denominator
}
```

> 和理论相比，我们直接把粗糙度(roughness)作为参数传给了上述函数；通过这种方式，我们可以针对每一个不同的项对粗糙度做一些修改。\*根据迪士尼公司给出的观察以及后来被 Epic Games 公司采用的光照模型，**在<u>几何遮蔽函数</u>和<u>法线分布函数</u>中采用粗糙度的平方会让光照看起来更加自然。\***

##### G：几何分布函数

> 几何函数从统计学上近似的求得了微平面间相互遮蔽的比率，这种相互遮蔽会损耗光线的能量。
>
> 这种遮蔽存在两种情况，分别是几何遮蔽(Geometry Obstruction)和几何阴影(Geometry Shadowing)，这两种遮蔽各分别由观察方向和光线方向引起。LearnOpenGL 中图示如下；
>
> ![img](https://learnopengl-cn.github.io/img/07/01/geometry_shadowing.png)

与 NDF 类似，几何函数采用一个材料的粗糙度参数作为输入参数，粗糙度较高的表面其微平面间相互遮蔽的概率就越高。我们将要使用的几何函数是 GGX 与 Schlick-Beckmann 近似的结合体，因此又称为 Schlick-GGX：

$$
G_{SchlickGGX}(n, v, k) = \frac{n \cdot v}{(n \cdot v)(1-k) + k}
$$

这里的 k 是 α 的重映射(Remapping)，取决于我们要用的是针对直接光照还是针对 IBL 光照的几何函数。对于 k，有：

$$
k_{direct} = \frac{(\alpha+1)^2}{8}\\
k_{IBL} = \frac{\alpha^2}{2}
$$

而同时，为了有效的估算几何部分，需要将观察方向（几何遮蔽）和光线方向向量（几何阴影）都考虑进去。我们可以使用史密斯法(Smith’s method)来把两者都纳入其中：

$$
G(n, v, l, k) = G_{sub}(n, v, k)G_{sub}(n, l, k)
$$

实现：

```glsl
float GeometrySchlickGGX(float NdotV, float roughness) {
    float r = (roughness + 1.0);
    float k = (r*r) / 8.0;

    float num   = NdotV;
    float denom = NdotV * (1.0 - k) + k;

    return num / denom;
}

float GeometrySmith(vec3 N, vec3 V, vec3 L, float roughness) {
    float NdotV = max(dot(N, V), 0.0);
    float NdotL = max(dot(N, L), 0.0);
    float ggx2  = GeometrySchlickGGX(NdotV, roughness);
    float ggx1  = GeometrySchlickGGX(NdotL, roughness);

    return ggx1 * ggx2;
}
```

##### F：菲涅尔方程

> 菲涅尔（发音为 Freh-nel）方程描述的是**被反射的光线对比光线被折射的部分所占的比率**，这个比率会随着我们观察的角度不同而不同。当光线碰撞到一个表面的时候，菲涅尔方程会根据观察角度告诉我们被反射的光线所占的百分比。利用这个反射比率和能量守恒原则，我们可以直接得出光线被折射的部分以及光线剩余的能量。

菲涅尔效应产生的结果往往是，物体边缘大量光线被反射而泛白，因为在这些边缘上，法线往往能与视角接近 90°；而在物体的中央会呈现自己本身的颜色，而对于透明物体来说就会露出其下层的物体。

![img](https://learnopengl-cn.github.io/img/07/01/fresnel.png)

> 当垂直观察的时候，任何物体或者材质表面都有一个基础反射率(Base Reflectivity)，但是如果以一定的角度往平面上看的时候[所有](http://filmicgames.com/archives/557)反光都会变得明显起来。你可以自己尝试一下，用垂直的视角观察你自己的木制/金属桌面，此时一定只有最基本的反射性。但是如果你从近乎 90 度（译注：应该是指和法线的夹角）的角度观察的话反光就会变得明显的多。如果从理想的 90 度视角观察，所有的平面理论上来说都能完全的反射光线。这种现象因菲涅尔而闻名，并体现在了菲涅尔方程之中。

落实到方程上，真实世界的菲涅尔等式一般比较复杂，我们在实时渲染中，往往采用一些近似等式，这里我们采用第一种，即 Schlick 菲涅尔近似等式：

$$
Schlick菲涅尔近似等式：F_{Schlick}(h, v, F_0) = F_0 + (1 - F_0)(1 - (h \cdot v))^5\\\\
Empricial菲涅尔近似等式：F_{Empricial}(v, n) = max(0, min(1, bias + scale \times (1 - v \cdot n)^{power}))\\\\...
$$

Fresnel-Schlick 近似法接收一个参数`F0`，被称为 0° 入射角的反射率，或者说是直接(垂直)观察表面时有多少光线会被反射。 这个参数`F0`会因为材料不同而不同，而且特别的，对金属材质需要特别处理：

> `F0` 表示平面的基础反射率，它是利用所谓**折射指数**(Indices of Refraction)或者说 IOR 计算得出的。然后正如你可以从球体表面看到的那样，我们越是朝球面掠角的方向上看（此时视线和表面法线的夹角接近 90 度）菲涅尔现象就越明显，反光就越强
>
> 问题就在于，Fresnel-Schlick 近似仅仅对电介质或者说非金属表面有定义。对于导体(Conductor)表面（金属），使用它们的折射指数计算 `F0` 并不能得出正确的结果，这样我们就需要使用一种不同的菲涅尔方程来对导体表面进行计算。由于这样很不方便，所以我们预计算出平面对于法向入射的结果`F0`，然后基于相应观察角的 Fresnel-Schlick 近似对这个值进行插值，用这种方法来进行进一步的估算。这样我们就能对金属和非金属材质使用同一个公式了。

在 PBR 金属流中我们简单地认为大多数的**绝缘体**在`F0`为 0.04 的时候看起来视觉上是正确的，对于**金属表面**我们根据反射率特别地指定`F0`，并对 `F0` 进行插值。这在代码上看起来会像是这样：

```glsl
vec3 F0 = vec3(0.04);
F0      = mix(F0, albedo, metallic);
vec3 F  = fresnelSchlick(max(dot(H, V), 0.0), F0); // 对于金属表面，我们根据初始的 F0 和 金属性 及 反射率 进行线性插值。

...

vec3 fresnelSchlick(float cosTheta, vec3 F0) {
    return F0 + (1.0 - F0) * pow(clamp(1.0 - cosTheta, 0.0, 1.0), 5.0); // 注意这里用的clamp是为了避免黑点
}
```

可以看到，对于非金属表面`F0`始终为 0.04。

> 平面对于法向入射的响应或者说基础反射率可以在一些大型数据库中找到，比如[这个](http://refractiveindex.info/)。下面列举的这一些常见数值就是从 Naty Hoffman 的课程讲义中所得到的：
>
> |       材料        |     F0 (线性)      |     F0 (sRGB)      | 颜色 |
> | :---------------: | :----------------: | :----------------: | :--: |
> |        水         | (0.02, 0.02, 0.02) | (0.15, 0.15, 0.15) |      |
> |  塑料/玻璃（低）  | (0.03, 0.03, 0.03) | (0.21, 0.21, 0.21) |      |
> |    塑料（高）     | (0.05, 0.05, 0.05) | (0.24, 0.24, 0.24) |      |
> | 玻璃（高）/红宝石 | (0.08, 0.08, 0.08) | (0.31, 0.31, 0.31) |      |
> |       钻石        | (0.17, 0.17, 0.17) | (0.45, 0.45, 0.45) |      |
> |        铁         | (0.56, 0.57, 0.58) | (0.77, 0.78, 0.78) |      |
> |        铜         | (0.95, 0.64, 0.54) | (0.98, 0.82, 0.76) |      |
> |        金         | (1.00, 0.71, 0.29) | (1.00, 0.86, 0.57) |      |
> |        铝         | (0.91, 0.92, 0.92) | (0.96, 0.96, 0.97) |      |
> |        银         | (0.95, 0.93, 0.88) | (0.98, 0.97, 0.95) |      |
>
> 这里可以观察到的一个有趣的现象，所有电介质材质表面的基础反射率都不会高于 0.17，这其实是例外而非普遍情况。导体材质表面的基础反射率起点更高一些并且（大多）在 0.5 和 1.0 之间变化。此外，_对于导体或者金属表面而言基础反射率一般是带有色彩的，这也是为什么 `F0` 要用 RGB 三原色来表示的原因（法向入射的反射率可随波长不同而不同）。这种现象我们**只能**在金属表面观察的到。_
>
> 这些金属表面相比于电介质表面所独有的特性引出了所谓的金属工作流的概念。也就是我们需要额外使用一个被称为金属度(Metalness)的参数来参与编写表面材质。金属度用来描述一个材质表面是金属还是非金属的。
>
> 理论上来说，一个表面的金属度应该是二元的：要么是金属要么不是金属，不能两者皆是。但是，大多数的渲染管线都允许在 0.0 至 1.0 之间线性的调配金属度。这主要是由于材质纹理精度不足以描述一个拥有诸如细沙/沙状粒子/刮痕的金属表面。通过对这些小的类非金属粒子/刮痕调整金属度值，我们可以获得非常好看的视觉效果。

另外值得一提的是，当我们采用了菲涅尔方程后，我们会获得一点特殊的便利——我们不再需要再在方程中采用额外的 Ks 了——`F `具有和方程中 Ks 相似的性质。回想一下，`F` 是干什么的？

> 菲涅尔（发音为 Freh-nel）方程描述的是被反射的光线对比光线被折射的部分所占的比率

而 Ks 和 Kd 是干什么的？

> Kd 是入射光线中被折射部分的能量所占的比率，而 Ks 是被反射部分的比率。

也就是说，`F` 已经包含了类似 Ks 的性质，所以我们可以直接将 `F` 用以代替 Ks 的存在，最终我们实际的运算式子如下：

## 编写 PBR 材质

在了解了 PBR 后面的数学模型之后，最后我们将通过说明美术师一般是如何编写一个我们可以直接输入 PBR 的平面物理属性的来结束这部分的讨论。PBR 渲染管线所需要的每一个表面参数都可以用纹理来定义或者建模。使用纹理可以让我们逐个片段的来控制每个表面上特定的点对于光线是如何响应的：不论那个点是不是金属，粗糙或者平滑，也不论表面对于不同波长的光会有如何的反应。

在下面你可以看到在一个 PBR 渲染管线当中经常会碰到的纹理列表，还有将它们输入 PBR 渲染器所能得到的相应的视觉输出：

![img](https://learnopengl-cn.github.io/img/07/01/textures.png)

**反照率**：反照率(Albedo)纹理为每一个金属的纹素(Texel)（纹理像素）_指定表面颜色或者基础反射率_。这和我们之前使用过的漫反射纹理相当类似，不同的是所有光照信息都是由一个纹理中提取的。漫反射纹理的图像当中常常包含一些细小的阴影或者深色的裂纹，而反照率纹理中是不会有这些东西的。它应该只包含表面的颜色（或者折射吸收系数）。

**法线**：法线贴图纹理和我们之前在[法线贴图](https://learnopengl-cn.github.io/05 Advanced Lighting/04 Normal Mapping/)教程中所使用的贴图是完全一样的。法线贴图使我们可以逐片段的指定独特的法线，来为表面制造出起伏不平的假象。

**金属度**：金属(Metallic)贴图逐个纹素的指定该纹素是不是金属质地的。根据 PBR 引擎设置的不同，美术师们既可以将金属度编写为灰度值又可以编写为 1 或 0 这样的二元值。

**粗糙度**：粗糙度(Roughness)贴图可以以纹素为单位指定某个表面有多粗糙。采样得来的粗糙度数值会影响一个表面的微平面统计学上的取向度。一个比较粗糙的表面会得到更宽阔更模糊的镜面反射（高光），而一个比较光滑的表面则会得到集中而清晰的镜面反射。某些 PBR 引擎预设采用的是对某些美术师来说更加直观的光滑度(Smoothness)贴图而非粗糙度贴图，不过这些数值在采样之时就马上用（1.0 – 光滑度）转换成了粗糙度。

**AO**：环境光遮蔽(Ambient Occlusion)贴图或者说 AO 贴图为表面和周围潜在的几何图形指定了一个额外的阴影因子。比如如果我们有一个砖块表面，反照率纹理上的砖块裂缝部分应该没有任何阴影信息。然而 AO 贴图则会把那些光线较难逃逸出来的暗色边缘指定出来。在光照的结尾阶段引入环境遮蔽可以明显的提升你场景的视觉效果。网格/表面的环境遮蔽贴图要么通过手动生成，要么由 3D 建模软件自动生成。

美术师们可以在纹素级别设置或调整这些基于物理的输入值，还可以以现实世界材料的表面物理性质来建立他们的材质数据。这是 PBR 渲染管·线最大的优势之一，因为不论环境或者光照的设置如何改变这些表面的性质是不会改变的，这使得美术师们可以更便捷地获取物理可信的结果。在 PBR 渲染管线中编写的表面可以非常方便的在不同的 PBR 渲染引擎间共享使用，不论处于何种环境中它们看上去都会是正确的，因此看上去也会更自然。

## 结合 IBL

> IBL，即 Image based lighting，可直译为基于图像的光照，它是一类光照技术的集合。**其光源不是可分解的直接光源，而是将周围环境整体视为一个大光源。**IBL 通常使用（取自现实世界或从 3D 场景生成的）环境立方体贴图 (Cubemap) ，我们可以将立方体贴图的每个像素视为光源，在渲染方程中直接使用它。这种方式可以有效地捕捉环境的全局光照和氛围，使物体更好地融入其环境。
>
> 由于基于图像的光照算法会捕捉部分甚至全部的环境光照，通常认为它是一种更精确的环境光照输入格式，甚至也可以说是一种全局光照的粗略近似。基于此特性，IBL 对 PBR 很有意义，因为当我们将环境光纳入计算之后，物体在物理方面看起来会更加准确。

在 IBL 下，有一件事情是很麻烦的：我们无法规避反射率方程中的积分了——直接光源下，积分是可以规避的——因为我们事先已经知道了对积分有贡献的、若干精确的光线方向 w~i~，然而这次，来自周围环境的**每个**方向 w~i~ 的入射光都可能具有一些辐射度，使得解决积分变得不那么简单。这为解决积分提出了两个要求：

- 对于给定的任何方向向量 w~i~，我们都能获取这个方向上场景的辐射度。
- 解决积分需要快速且实时。

现在看，第一个要求相对容易些。我们已经有了一些思路：表示环境或场景辐照度的一种方式是（预处理过的）环境立方体贴图，给定这样的立方体贴图，我们可以将立方体贴图的每个纹素视为一个光源。使用一个方向向量 w~i~ 对此立方体贴图进行采样，我们就可以获取该方向上的场景辐照度。如此，给定方向向量 w~i~ ，获取此方向上场景辐射度的方法就简化为：

```glsl
vec3 radiance =  texture(_cubemapEnvironment, w_i).rgb;
```

为了以更有效的方式解决积分，我们需要对其大部分结果进行预处理——或称预计算。为此，我们必须深入研究反射方程。原来的反射方程有：

$$
L_0(p,w_0)=\int_Ω(k_d*\frac{c}{\pi}+k_s*\frac{DGF}{4*(w_0 \cdot n)(w_i \cdot n)}*Li(p,w_i)*n \cdot w_i*dw_i
$$

我们可以发现，式子中的漫反射项是与积分无关的——这部分是可以拆开的

$$
L_0(p,w_0)=\int_Ω(k_d*\frac{c}{\pi}+k_s*\frac{DGF}{4*(w_0 \cdot n)(w_i \cdot n)}*Li(p,w_i)*n \cdot w_i*dw_i\\
=\int_Ωk_d*\frac{c}{\pi}*Li(p,w_i)*n \cdot w*dw_i + \int_Ωk_s*\frac{DGF}{4*(w_0 \cdot n)(w_i \cdot n)}*L_i(p,w_i)*n \cdot w_i*dw_i
$$

这样，我们就可以分别考虑两部分的预计算方法

先看漫反射项：

### 预计算漫反射项

#### 对方程的一些小改变

让我们先对分离出的原始方程做一些变化：

$$
L_0(p,w_0)=\int_Ωk_d*\frac{c}{\pi}*Li(p,w_i)*n \cdot w*dw_i\\
=k_d\frac{c}{\pi}\int_ΩLi(p,w_i)*n \cdot w*dw_i\\
$$

不难发现，我们所做的内容也就是将常数项提到了积分外部（颜色 c、折射率 k~d~ 和 π 在整个漫反射积分函数中，是常数），这样我们就获得了一个只依赖于 w~i~ 的积分（假设 p 位于环境贴图的中心），这样我们就可以通过卷积，预计算一个新的立方体贴图，它在每个采样方向——也就是纹素——中存储漫反射积分的结果。

>

#### 预计算思路

目前，我们的主要目标是计算所有间接漫反射光的积分，其中光照的辐照度以环境立方体贴图的形式给出。我们已经知道，在方向 w~i~ 上采样 HDR 环境贴图，可以获得场景在此方向上的辐射度 L(p,w~i~) 。虽然如此，要解决积分，我们仍然不能仅从一个方向对环境贴图采样——要从半球 Ω 上所有可能的方向进行采样，这不仅对于片段着色器过于昂贵，并且计算上又不可能从 Ω 的每个可能的方向采样环境光照——理论上可能的方向数量是无限的。后者的解决办法相对容易——我们去对有限数量的方向采样以近似求解，并在半球内以均匀间隔或随机取方向等的方式，可以获得一个相当精确的辐照度近似值，从而离散地计算积分。不过，对于每个片段实时执行此操作仍然太昂贵，因为仍然需要非常大的样本数量才能获得不错的结果，因此我们希望可以**预计算**。

既然明确了需要预计算，那么我们额外需要思考的就是怎么去做预计算。

我们知道的是，我们的计算总是处于一个半球空间中的——即全部从正面来的光——既然半球的朝向决定了我们捕捉辐照度的位置，我们可以预先计算每个可能的半球朝向的辐照度，把结果保存另一个 CubeMap 中，这样我们就几乎达到了我们的目的。

> 下面是一个环境立方体贴图及其生成的辐照度图的示例：
>
> ![img](https://learnopengl-cn.github.io/img/07/03/01/ibl_irradiance.png)

我们可以以这样的手段去描述我们的半球面：

我们用球坐标 θ 和 ϕ 来代替反射方程的积分 ∫ 中的立体角，由于积分是围绕立体角 dw 旋转进行的，那我们用以代替 dw 的球坐标也该定义出相同的性质：我们规定对于围绕半球大圆的航向角 ϕ ，我们在 0 到 2π 内采样，而从半球顶点出发的倾斜角 θ ，采样范围是 0 到 π/2 。于是我们更新一下反射积分方程：

$$
L_0(p, \phi_0, \theta_0)=k_d\frac{c}{\pi}\int_{\phi=0}^{2\pi}\int_{\theta=0}^{\frac{1}{2}\pi}L_i(p, \phi_i, \theta_i)\cos(\theta)\sin(\theta)d\phi d\theta
$$

求解积分需要我们在半球 Ω 内采集固定数量的离散样本并对其结果求平均值。分别给每个球坐标轴指定离散样本数量 n~1~ 和 n~2~ 以求其[黎曼和](https://en.wikipedia.org/wiki/Riemann_sum)，积分式会转换为以下离散版本：

$$
Lo(p,ϕ_0,θ_0)=k_d\frac{c}{π}\frac{1}{n_1n_2}\sum_{\phi=0}^{n_1}\sum_{\theta=0}^{n_2}Li(p,ϕ_i,θ_i)cos(θ)sin(θ)dϕdθ
$$

### 预计算镜面反射

相较于漫反射项，我们的镜面反射显然是一个更加复杂的部分，让我们先做分析：

$$
L_0(p,w_0)=\int_Ωk_s*\frac{DGF}{4*(w_0 \cdot n)(w_i \cdot n)}*L_i(p,w_i)*n \cdot w_i*dw_i
$$

我们可以发现，相较于仅有 w~i~ 一个变量的漫反射，我们的镜面反射部分同时存在 w~i~ 与 w~0~ 两个变量——我们无法用两个方向向量采样预计算的立方体图。要想预计算这部分内容，我们得做更多的处理——这里采用的是 Epic Games 的解决方案：分割求和近似法（split sum approximation）。

分割求和近似将方程的镜面部分分割成两个独立的部分，我们可以单独求卷积，然后在 PBR 着色器中求和，以用于间接镜面反射部分 IBL。分割求和近似法类似于我们之前求辐照图预卷积的方法，需要 HDR 环境贴图作为其卷积输入。在该办法中，我们的公式会变为：

$$
L_0(p,w_0)=\int_\Omega L_i(p, w_i)dw_i * \int_\Omega f_r()
$$

让我们分别考虑这两部分

#### 预滤波环境贴图

预滤波环境贴图的方法与我们对辐射度贴图求卷积的方法非常相似。只是对于卷积的每个粗糙度级别，我们会按顺序把模糊后的结果存储在预滤波贴图的 mipmap 中。

### GGX 重要性采样

有别于均匀或纯随机地（比如蒙特卡洛）在积分半球 Ω 产生采样向量，**我们的采样会根据粗糙度，偏向微表面的半向量的宏观反射方向。**采样过程将与我们之前看到的过程相似：开始一个大循环，生成一个随机（低差异）序列值，用该序列值在切线空间中生成样本向量，将样本向量变换到世界空间并对场景的辐射度采样。不同之处在于，我们现在使用低差异序列值作为输入来生成采样向量：

```c++
const uint SAMPLE_COUNT = 4096u;
for(uint i = 0u; i < SAMPLE_COUNT; ++i)
{
    vec2 Xi = Hammersley(i, SAMPLE_COUNT);
}
```

此外，要构建采样向量，我们需要一些方法定向和偏移采样向量，以使其朝向特定粗糙度的镜面波瓣方向。我们可以如理论教程中所述使用 NDF，并将 GGX NDF 结合到 Epic Games 所述的球形采样向量的处理中：

```c++
vec3 ImportanceSampleGGX(vec2 Xi, vec3 N, float roughness)
{
    float a = roughness*roughness;

    float phi = 2.0 * PI * Xi.x;
    float cosTheta = sqrt((1.0 - Xi.y) / (1.0 + (a*a - 1.0) * Xi.y));
    float sinTheta = sqrt(1.0 - cosTheta*cosTheta);

    // from spherical coordinates to cartesian coordinates
    vec3 H;
    H.x = cos(phi) * sinTheta;
    H.y = sin(phi) * sinTheta;
    H.z = cosTheta;

    // from tangent-space vector to world-space sample vector
    vec3 up        = abs(N.z) < 0.999 ? vec3(0.0, 0.0, 1.0) : vec3(1.0, 0.0, 0.0);
    vec3 tangent   = normalize(cross(up, N));
    vec3 bitangent = cross(N, tangent);

    vec3 sampleVec = tangent * H.x + bitangent * H.y + N * H.z;
    return normalize(sampleVec);
}
```

基于特定的粗糙度输入和低差异序列值 Xi，我们获得了一个采样向量，该向量大体围绕着预估的微表面的半向量。注意，根据迪士尼对 PBR 的研究，Epic Games 使用了平方粗糙度以获得更好的视觉效果。

使用低差异 Hammersley 序列和上述定义的样本生成方法，我们可以最终完成预滤波器卷积着色器：

```c++
#version 330 core
out vec4 FragColor;
in vec3 localPos;

uniform samplerCube environmentMap;
uniform float roughness;

const float PI = 3.14159265359;

float RadicalInverse_VdC(uint bits);
vec2 Hammersley(uint i, uint N);
vec3 ImportanceSampleGGX(vec2 Xi, vec3 N, float roughness);

void main()
{
    vec3 N = normalize(localPos);
    vec3 R = N;
    vec3 V = R;

    const uint SAMPLE_COUNT = 1024u;
    float totalWeight = 0.0;
    vec3 prefilteredColor = vec3(0.0);
    for(uint i = 0u; i < SAMPLE_COUNT; ++i)
    {
        vec2 Xi = Hammersley(i, SAMPLE_COUNT);
        vec3 H  = ImportanceSampleGGX(Xi, N, roughness);
        vec3 L  = normalize(2.0 * dot(V, H) * H - V);

        float NdotL = max(dot(N, L), 0.0);
        if(NdotL > 0.0)
        {
            prefilteredColor += texture(environmentMap, L).rgb * NdotL;
            totalWeight      += NdotL;
        }
    }
    prefilteredColor = prefilteredColor / totalWeight;

    FragColor = vec4(prefilteredColor, 1.0);
}
```

输入的粗糙度随着预过滤的立方体贴图的 mipmap 级别变化（从 0.0 到 1.0），我们根据据粗糙度预过滤环境贴图，把结果存在 prefilteredColor 里。再用 prefilteredColor 除以采样权重总和，其中对最终结果影响较小（NdotL 较小）的采样最终权重也较小。

### 捕获预过滤 mipmap 级别

剩下要做的就是让 OpenGL 在多个 mipmap 级别上以不同的粗糙度值预过滤环境贴图。有了最开始的辐照度教程作为基础，实际上很简单：

```c++
prefilterShader.use();
prefilterShader.setInt("environmentMap", 0);
prefilterShader.setMat4("projection", captureProjection);
glActiveTexture(GL_TEXTURE0);
glBindTexture(GL_TEXTURE_CUBE_MAP, envCubemap);

glBindFramebuffer(GL_FRAMEBUFFER, captureFBO);
unsigned int maxMipLevels = 5;
for (unsigned int mip = 0; mip < maxMipLevels; ++mip)
{
    // reisze framebuffer according to mip-level size.
    unsigned int mipWidth  = 128 * std::pow(0.5, mip);
    unsigned int mipHeight = 128 * std::pow(0.5, mip);
    glBindRenderbuffer(GL_RENDERBUFFER, captureRBO);
    glRenderbufferStorage(GL_RENDERBUFFER, GL_DEPTH_COMPONENT24, mipWidth, mipHeight);
    glViewport(0, 0, mipWidth, mipHeight);

    float roughness = (float)mip / (float)(maxMipLevels - 1);
    prefilterShader.setFloat("roughness", roughness);
    for (unsigned int i = 0; i < 6; ++i)
    {
        prefilterShader.setMat4("view", captureViews[i]);
        glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0,
                               GL_TEXTURE_CUBE_MAP_POSITIVE_X + i, prefilterMap, mip);

        glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
        renderCube();
    }
}
glBindFramebuffer(GL_FRAMEBUFFER, 0);
```

这个过程类似于辐照度贴图卷积，但是这次我们将帧缓冲区缩放到适当的 mipmap 尺寸， mip 级别每增加一级，尺寸缩小为一半。此外，我们在 glFramebufferTexture2D 的最后一个参数中指定要渲染的目标 mip 级别，然后将要预过滤的粗糙度传给预过滤着色器。

这样我们会得到一张经过适当预过滤的环境贴图，访问该贴图时指定的 mip 等级越高，获得的反射就越模糊。如果我们在天空盒着色器中显示这张预过滤的环境立方体贴图，并在其着色器中强制在其第一个 mip 级别以上采样，如下所示：

```c++
vec3 envColor = textureLod(environmentMap, WorldPos, 1.2).rgb;
```

我们得到的结果看起来确实像原始环境的模糊版本：

![img](https://learnopengl-cn.github.io/img/07/03/02/ibl_prefilter_map_sample.png)

如果 HDR 环境贴图的预过滤看起来差不多没问题，尝试一下不同的 mipmap 级别，观察预过滤贴图随着 mip 级别增加，反射逐渐从锐利变模糊的过程。

## 预过滤卷积的伪像

当前的预过滤贴图可以在大多数情况下正常工作，不过你迟早会遇到几个与预过滤卷积直接相关的渲染问题。我将在这里列出最常见的一些问题，以及如何修复它们。

### 高粗糙度的立方体贴图接缝

在具有粗糙表面的表面上对预过滤贴图采样，也就等同于在较低的 mip 级别上对预过滤贴图采样。在对立方体贴图进行采样时，默认情况下，OpenGL 不会在立方体面**之间**进行线性插值。由于较低的 mip 级别具有更低的分辨率，并且预过滤贴图代表了与更大的采样波瓣卷积，因此缺乏**立方体的面和面之间的滤波**的问题就更明显：

![img](https://learnopengl-cn.github.io/img/07/03/02/ibl_prefilter_seams.png)

幸运的是，OpenGL 可以启用 GL_TEXTURE_CUBE_MAP_SEAMLESS，以为我们提供在立方体贴图的面之间进行正确过滤的选项：

```c++
glEnable(GL_TEXTURE_CUBE_MAP_SEAMLESS);
```

### 预过滤卷积的亮点

由于镜面反射中光强度的变化大，高频细节多，所以对镜面反射进行卷积需要大量采样，才能正确反映 HDR 环境反射的混乱变化。我们已经进行了大量的采样，但是在某些环境下，在某些较粗糙的 mip 级别上可能仍然不够，导致明亮区域周围出现点状图案：

![img](https://learnopengl-cn.github.io/img/07/03/02/ibl_prefilter_dots.png)

一种解决方案是进一步增加样本数量，但在某些情况下还是不够。另一种方案如 [Chetan Jags](https://chetanjags.wordpress.com/2015/08/26/image-based-lighting/) 所述，我们可以在预过滤卷积时，不直接采样环境贴图，而是基于积分的 PDF 和粗糙度采样环境贴图的 mipmap ，以减少伪像：

```c++
float D   = DistributionGGX(NdotH, roughness);
float pdf = (D * NdotH / (4.0 * HdotV)) + 0.0001;

float resolution = 512.0; // resolution of source cubemap (per face)
float saTexel  = 4.0 * PI / (6.0 * resolution * resolution);
float saSample = 1.0 / (float(SAMPLE_COUNT) * pdf + 0.0001);

float mipLevel = roughness == 0.0 ? 0.0 : 0.5 * log2(saSample / saTexel);
```

既然要采样 mipmap ，不要忘记在环境贴图上开启三线性过滤：

```c++
glBindTexture(GL_TEXTURE_CUBE_MAP, envCubemap);
glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR);
```

设置立方体贴图的基本纹理后，让 OpenGL 生成 mipmap：

```c++
// convert HDR equirectangular environment map to cubemap equivalent
[...]
// then generate mipmaps
glBindTexture(GL_TEXTURE_CUBE_MAP, envCubemap);
glGenerateMipmap(GL_TEXTURE_CUBE_MAP);
```

这个方法效果非常好，可以去除预过滤贴图中较粗糙表面上的大多数甚至全部亮点。

##### 蒙特卡洛积分

> 为了充分理解重要性采样，我们首先要了解一种数学结构，称为蒙特卡洛积分。蒙特卡洛积分主要是统计和概率理论的组合。其作用很大，比如：_蒙特卡洛可以帮助我们离散地解决人口统计问题，而不必考虑所有人。_
>
> 具体来讲，假设我们得计算一个国家所有公民的平均身高。为了得到结果，我们可以测量每个公民并对他们的身高求平均，这样会得到你需要的确切答案。但是，由于大多数国家人海茫茫，这个方法不现实：需要花费太多精力和时间。
>
> 另一种方法是选择一个小得多的完全随机（无偏）的人口子集，测量他们的身高并对结果求平均。可能只测量 100 人，虽然答案并非绝对精确，但会得到一个相对接近真相的答案，这个理论被称作大数定律。我们的想法是，如果从总人口中测量一组较小的真正随机样本的 N，结果将相对接近真实答案，并随着样本数 N 的增加而愈加接近。
>
> **蒙特卡罗积分建立在大数定律的基础上，并采用相同的方法来求解积分。不为所有可能的（理论上是无限的）样本值 x 求解积分，而是简单地从总体中随机挑选样本 N 生成采样值并求平均。**随着 N 的增加，我们的结果会越来越接近积分的精确结果：
>
> $$
> O=\int_{a}^{b}f(x)dx=\frac{1}{N}\sum_{i=0}^{N-1}\frac{f(x)}{pdf(x)}
> $$
>
> 为了求解这个积分，我们在 $a$ 到 $b$ 上采样 $N$ 个随机样本，将它们加在一起并除以样本总数来取平均。$pdf$ 代表概率密度函数 (probability density function)，它的含义是特定样本在整个样本集上发生的概率。例如，人口身高的 $pdf$ 看起来应该像这样：
>
> ![img](https://learnopengl-cn.github.io/img/07/03/02/ibl_pdf.png)
>
> 从该图中我们可以看出，如果我们对人口任意随机采样，那么挑选身高为 1.70 的人口样本的可能性更高，而样本身高为 1.50 的概率较低。
>
> 当涉及蒙特卡洛积分时，某些样本可能比其他样本具有更高的生成概率。这就是为什么对于任何一般的蒙特卡洛估计，我们都会根据 pdf 将采样值作除以或乘以采样概率。到目前为止，我们每次需要估算积分的时候，生成的样本都是均匀分布的，概率完全相等。到目前为止，我们的估计是无偏的，这意味着随着样本数量的不断增加，我们最终将收敛到积分的**精确**解。
>
> 但是，某些蒙特卡洛估算是有偏的，这意味着生成的样本并不是完全随机的，而是集中于特定的值或方向。这些有偏的蒙特卡洛估算具有更快的收敛速度，它们会以更快的速度向精确解收敛，但是由于其有偏性，可能永远不会收敛到精确解。通常来说，这是一个可以接受的折衷方案，尤其是在计算机图形学中。因为只要结果在视觉上可以接受，解决方案的精确性就不太重要。
>
> 下文我们将会提到一种（有偏的）重要性采样，其生成的样本偏向特定的方向，在这种情况下，我们会将每个样本乘以或除以相应的 pdf 再求和。
>
> **蒙特卡洛积分在计算机图形学中非常普遍，因为它是一种以高效的离散方式对连续的积分求近似而且非常直观的方法：对任何面积/体积进行采样——例如半球 Ω ——在该面积/体积内生成数量 N 的随机采样，权衡每个样本对最终结果的贡献并求和。**
>
> 蒙特卡洛积分是一个庞大的数学主题，在此不再赘述，但有一点需要提到：生成随机样本的方法也多种多样。默认情况下，每次采样都是我们熟悉的完全（伪）随机，不过利用半随机序列的某些属性，我们可以生成虽然是随机样本但具有一些有趣性质的样本向量。例如，我们可以对一种名为低差异序列的东西进行蒙特卡洛积分，该序列生成的仍然是随机样本，但样本分布更均匀：
>
> ![img](https://learnopengl-cn.github.io/img/07/03/02/ibl_low_discrepancy_sequence.png)
>
> 当使用低差异序列生成蒙特卡洛样本向量时，该过程称为拟蒙特卡洛积分。拟蒙特卡洛方法具有更快的收敛速度，这使得它对于性能繁重的应用很有用。
>
> 鉴于我们新获得的有关蒙特卡洛（Monte Carlo）和拟蒙特卡洛（Quasi-Monte Carlo）积分的知识，我们可以使用一个有趣的属性来获得更快的收敛速度，这就是重要性采样。我们在前文已经提到过它，但是在镜面反射的情况下，反射的光向量被限制在镜面波瓣中，波瓣的大小取决于表面的粗糙度。既然镜面波瓣外的任何（拟）随机生成的样本与镜面积分无关，因此将样本集中在镜面波瓣内生成是有意义的，但代价是蒙特卡洛估算会产生偏差。
>
> 本质上来说，这就是重要性采样的核心：只在某些区域生成采样向量，该区域围绕微表面半向量，受粗糙度限制。通过将拟蒙特卡洛采样与低差异序列相结合，并使用重要性采样偏置样本向量的方法，我们可以获得很高的收敛速度。因为我们求解的速度更快，所以要达到足够的近似度，我们所需要的样本更少。因此，这套组合方法甚至可以允许图形应用程序实时求解镜面积分，虽然比预计算结果还是要慢得多。

##### 重要性采样

##### 低差异序列

在本教程中，我们将使用重要性采样来预计算间接反射方程的镜面反射部分，该采样基于拟蒙特卡洛方法给出了随机的低差异序列。我们将使用的序列被称为 Hammersley 序列，[Holger Dammertz](http://holger.dammertz.org/stuff/notes_HammersleyOnHemisphere.html) 曾仔细描述过它。Hammersley 序列是基于 Van Der Corput 序列，该序列是把十进制数字的二进制表示镜像翻转到小数点右边而得。（译注：原文为 Van Der Corpus 疑似笔误，下文各处同）

给出一些巧妙的技巧，我们可以在着色器程序中非常有效地生成 Van Der Corput 序列，我们将用它来获得 Hammersley 序列，设总样本数为 N，样本索引为 i：

```glsl
float RadicalInverse_VdC(uint bits)
{
    bits = (bits << 16u) | (bits >> 16u);
    bits = ((bits & 0x55555555u) << 1u) | ((bits & 0xAAAAAAAAu) >> 1u);
    bits = ((bits & 0x33333333u) << 2u) | ((bits & 0xCCCCCCCCu) >> 2u);
    bits = ((bits & 0x0F0F0F0Fu) << 4u) | ((bits & 0xF0F0F0F0u) >> 4u);
    bits = ((bits & 0x00FF00FFu) << 8u) | ((bits & 0xFF00FF00u) >> 8u);
    return float(bits) * 2.3283064365386963e-10; // / 0x100000000
}
// ----------------------------------------------------------------------------
vec2 Hammersley(uint i, uint N)
{
    return vec2(float(i)/float(N), RadicalInverse_VdC(i));
}
```

GLSL 的 Hammersley 函数可以获取大小为 N 的样本集中的低差异样本 i。

**无需位运算的 Hammersley 序列**

并非所有 OpenGL 相关驱动程序都支持位运算符（例如 WebGL 和 OpenGL ES 2.0），在这种情况下，你可能需要不依赖位运算符的替代版本 Van Der Corput 序列：

```glsl
float VanDerCorpus(uint n, uint base)
{
    float invBase = 1.0 / float(base);
    float denom   = 1.0;
    float result  = 0.0;

    for(uint i = 0u; i < 32u; ++i)
    {
        if(n > 0u)
        {
            denom   = mod(float(n), 2.0);
            result += denom * invBase;
            invBase = invBase / 2.0;
            n       = uint(float(n) / 2.0);
        }
    }

    return result;
}
// ----------------------------------------------------------------------------
vec2 HammersleyNoBitOps(uint i, uint N)
{
    return vec2(float(i)/float(N), VanDerCorpus(i, 2u));
}
```

请注意，由于旧硬件中的 GLSL 循环限制，该序列循环遍历了所有可能的 32 位，性能略差。但是如果你没有位运算符可用的话可以考虑它，它可以在所有硬件上运行。

### 一个完整的 PBR 构建

## 编写 PBR Shader - 渲染方程的实现

### 实现内容的具体简化

### 实现流程

## 最后——关于当前的 PBR 模型

## References

- [重新理解 PBR（1） - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/416112744)
- [理论 - LearnOpenGL CN (learnopengl-cn.github.io)](https://learnopengl-cn.github.io/07 PBR/01 Theory/)
-
