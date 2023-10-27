---
title: learningOpenGl Chapter 1.7
date: 2023-3-8 10:26:08
tags:
  - Opengl
  - Shader
categories:
  - Opengl
  - Shader
mathjax: true
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

# 1.7 Transform

欢迎！

欢迎来到这里，有关数学的一节。

这里会结合`Games 101`与`Unity Shader入门精要`的相关内容进行。

> Games 101：[Lecture 03 Transformation](https://www.bilibili.com/video/BV1X7411F744?p=3&vd_source=c8eda79dd90c30ff02e09fb39906ac54)

## 1.7.1 GAMES 101

### 1.7.1.1 GAMES 101 Lecture 03 Transformation

线性变换适用范围：缩放、旋转
$$
\begin{bmatrix}
 x'\\
 y'
\end{bmatrix}= 
\begin{bmatrix}
 a & b\\
 c & d
\end{bmatrix}
\begin{bmatrix}
 x \\
 y 
\end{bmatrix}
$$
但当加入平移变换时，线性变换就无能为力：
$$
\begin{bmatrix}
 x'\\
 y'
\end{bmatrix}
=
\begin{bmatrix}
 a & b\\
 c & d
\end{bmatrix}
\begin{bmatrix}
 x \\
 y 
\end{bmatrix}
+
\begin{bmatrix}
t_x\\
t_y
\end{bmatrix}
$$
为了能在一个矩阵中表示这些变换，我们选择引入齐次坐标(`Homogeneous Coordinates`)：
$$
\begin{bmatrix}
 x'\\
 y'\\
 w'
\end{bmatrix}
=
\begin{bmatrix}
 a & b & t_x\\
 c & d & t_y\\
 0 & 0 & 1
\end{bmatrix}
\begin{bmatrix}
 x \\
 y \\
 1
\end{bmatrix}
=
\begin{bmatrix}
ax+by+t_x\\
cx+dy+t_y\\
1
\end{bmatrix}
$$
> 此时：
$$
\displaylines{2D \ Point: \
=
\left( 
x, \ y, \ 1
\right)^T\\
2D \ Vector: \ =\left( 
x, \ y, \ 0
\right)^T
}
$$

> 为什么向量添置的是0而点是1？
>
> - 通过简单的运算我们可以发现，如果w分量是0，则平移变换对该对象不造成任何影响——我们对向量的w分量置0是为了保护向量的“不随“平移变换”的这种性质，而Point的w为1则正是为了采用这种性质
>
> 基于w分量的原理，我们可以得到：

$$
\displaylines{Vector \ + \ Vector = Vector\\
Point \ - \  Point = Vector\\
Point \ + \ Vector = Point\\
Point \ + \  Point = \ ? \\
}
$$

> 当Point + Point时，w分量为2了，这能代表什么？——我们人为地定义w用于均分：
$$
\displaylines{
\begin{pmatrix}
x\\
y\\
w 
\end{pmatrix}
= 
\begin{pmatrix}
x/w \\
y/w \\
1
\end{pmatrix}(w \ne 0)
}
$$

我们把形如上方的运算方式称为`仿射变换`，`Affine Transformations`，它采用齐次坐标

此时，缩放、旋转和平移三种操作会变为：
$$
\displaylines{
Scale(s_x, \ s_y) = 
\begin{pmatrix}
s_x & 0 & 0\\
0 & x_y & 0\\
0 & 0 & 1
\end{pmatrix}\\
Rotation(\alpha) = 
\begin{pmatrix}
\cos\alpha & -\sin\alpha & 0\\
\sin\alpha & \cos\alpha & 0\\
0 & 0 & 1
\end{pmatrix}\\
T(t_x, \ t_y)=
\begin{pmatrix}
1 & 0 & t_x\\
0 & 1 & t_y\\
0 & 0 & 1
\end{pmatrix}
}
$$
但是，矩阵乘法中虽然没有交换律但是有结合律

我们按照顺序：
$$
平移矩阵 \ * \ 旋转矩阵 \ * \ 缩放矩阵 \ * \ (x, y, z)^T
$$
进行运算时，我们是可以将前三个矩阵先计算然后集结合为一个矩阵再与目标左乘的

当然，这个顺序并不是定死的，需要注意，我们只会关于原点的操作：关于原点的平移、旋转与缩放，如需必要，我们得把目标“**先平移到原点上**”

### 1.7.1.2 GAMES 101 Lecture 04 Transformation Count.

对于原Rotation矩阵：
$$
\displaylines{
Rotation(\theta) =
\begin{pmatrix}
\cos\theta & -\sin\theta\\
\sin\theta & \cos\theta
\end{pmatrix}\\
Rotation(-\theta) = 
\begin{pmatrix}
\cos\theta & \sin\theta\\
-\sin\theta & \cos\theta
\end{pmatrix} = 
Rotation(\theta) ^ T\\
}
$$
即对于旋转矩阵，有逆矩阵等于转置矩阵——这是一个`正交矩阵`

> 正交矩阵定义：
>
> 若一个矩阵满足：A^T^A = AA^T^ = E，则A矩阵是正交矩阵
>
> 此时A有如下性质：
> $$
> \displaylines{
> A^TA = E \Rightarrow A^T = A^{-1} \\
> A^TA = E \Rightarrow \begin{vmatrix} A \end{vmatrix} = 1
> }
> $$
> [正交矩阵有哪些性质？ - 知乎 (zhihu.com)](https://www.zhihu.com/question/62155511)

#### 1.7.1.2.1 3D transformations

类同1.7.1.1中的二维变换，只是我们需要将2D变换所用的三维齐次坐标改为四维齐次坐标：
$$
\displaylines{
3D \ Point: = (x, \ y, \ z, \ 1)^T\\
3D \ Vector: = (x, \ y, \ z, \ 0)^T
}
$$
此时的三种变换的矩阵会变为：
$$
\displaylines{
Scale(s_x,\ s_y,\ s_z) = 
\begin{pmatrix}
s_x & 0 & 0 & 0\\
0 & s_y & 0 & 0\\
0 & 0 & s_z & 0\\
0 & 0 & 0 & 1
\end{pmatrix} \\\\
Translation(t_x,\ t_y,\ t_z) = 
\begin{pmatrix}
1 & 0 & 0 & t_x\\
0 & 1 & 0 & t_y\\
0 & 0 & 1 & t_z\\
0 & 0 & 0 & 1
\end{pmatrix} \\
}
$$
而旋转则分为三种，分别应用于绕x, y, z三轴旋转的情况：
$$
\displaylines{
Rotation_x(\theta) = 
\begin{pmatrix}
1 & 0 & 0 & 0\\
0 & \cos\theta & -\sin\theta & 0\\
0 & \sin\theta & \cos\theta & 0\\
0 & 0 & 0 & 1
\end{pmatrix} \Longleftarrow y \times z \\\\
Rotation_y(\theta) = 
\begin{pmatrix}
\cos\theta & 0 & \sin\theta & 0\\
0 & 1 & 0 & 0\\
-\sin\theta & 0 & \cos\theta & 0\\
0 & 0 & 0 & 1
\end{pmatrix} \Longleftarrow z \times x  \\\\
Rotation_z(\theta) = 
\begin{pmatrix}
\cos\theta & -\sin\theta & 0 & 0\\
\sin\theta & \cos\theta & 0 & 0\\
0 & 0 & 1 & 0\\
0 & 0 & 0 & 1
\end{pmatrix} \Longleftarrow x \times y  \\\\
}
$$
有了基础的绕三轴的旋转，我们可以通过组合，将物体旋转到任意方向上去：
$$
R_{xyz}(\alpha,\ \beta,\ \gamma) = R_x(\alpha)R_y(\beta)R_z(\gamma)
$$
例：飞机的运动：roll, yaw, pitch

> 在三维空间中任意的旋转，我们都可以通过将它分解到三个轴上
>
> - 将旋转轴平移到经过原点
> - 分解该旋转过程
>
> 公式：Rodrigues' Rotation Formula: 
>
> Rotation by angle α around axis n
> $$
> R(\vec{n}, \alpha) = \cos(\alpha)\vec{I} + (1 - \cos(\alpha))\vec{n}\vec{n}^T + \sin(\alpha)
> \begin{pmatrix}
> 0 & -n_z & n_y\\
> n_z & 0 & -n_x\\
> -n_y & n_x & 0
> \end{pmatrix}
> $$
>
> > 简明推导过程：
> >
> > 
>
> ~罗德里格斯公式，给个轴和角度就能转~
>
> [向量叉乘与叉乘矩阵 - neu博 - 博客园 (cnblogs.com)](https://www.cnblogs.com/monoSLAM/p/5349497.html#:~:text=由叉乘的规则，a叉乘a的结果为0： 通过对比，可以发现 aH,就是a向量的叉乘矩阵 ，当a为列向量时 为a向量的叉乘矩阵。)



#### 1.7.1.2.2 Viewing transformation

>以拍照片为例：
>
>- Find a good place and arrange people (Model transformation)
>- Find a good "angle" to put the camera (View transformation)
>- Cheese! (Projection transformation)
>
>即MVP变换——模型-视图-投影变换
>
>相应的，我们也有MVP矩阵。当然，这里是View transformation

##### 1.7.1.2.2.1 View/Camera transformation

目的：将摄像头移动至世界中心
过程：做“将原点移动至当前摄像头位置的“的**逆变换**
原因：我们很难直接求出从世界空间转换到摄像机空间的旋转矩阵，那么鉴于

- **旋转矩阵本身的特殊性**
- 旋转后最终结果必定为 *x: (1, 0, 0); y: (0, 1, 0); z: (0, 0, 1)*
- 旋转前(世界空间下的)摄像机的x, y, z轴已知

那么，我们可以利用以上信息构建出**从摄像机空间变换到世界空间**的旋转矩阵，再将该矩阵转置得到目标矩阵

> 利用R_view.rotate，我们将世界空间变换到摄像机空间
>
> 同时我们也可以相反地利用**R^-1^_view.rotate**矩阵实现从摄像机空间到世界空间的逆变换

$$
\displaylines{
摄像机世界空间主轴坐标构成的旋转矩阵：R_{example}^{(-1)} = \begin{pmatrix}
T_x & B_x & N_x\\
T_y & B_y & N_y\\
T_z & B_z & N_z
\end{pmatrix}；\\
摄像机空间下主轴向量：Vector_x\begin{pmatrix}1\\0\\0\end{pmatrix}\\\Downarrow \\
则世界空间下摄像机主轴向量为：V^{'}_x = R_{example} * Vector_x
而上述内容的直接表示形式就是：R^{-1} = R_{ViewtoWorld} = \begin{pmatrix}
| & | & |\\
T & B & N\\
| & | & |
\end{pmatrix}；\\

正因如此，我们可以直接得到：R = R_{WorldToView} = R_{View} = \begin{pmatrix}
- & T & -\\
- & B & -\\
- & N & -
\end{pmatrix}=\begin{pmatrix}
T_x & T_y & T_z\\
B_x & B_y & B_z\\
N_z & N_y & N_z
\end{pmatrix}
}
$$

这样，便有完整的变换如下：
$$
\displaylines{
R_{view.rotate}^{-1} = \begin{pmatrix}
x_{\hat{g}\times\hat{t}} & x_t & x_{-g}\\
y_{\hat{g}\times\hat{t}} & y_t & y_{-g}\\
z_{\hat{g}\times\hat{t}} & z_t & z_{-g}
\end{pmatrix}^{-1} = R_{view.rotate}^T\\\Downarrow\\
R_{view.rotate} = \begin{pmatrix}
x_{\hat{g}\times\hat{t}} & y_{\hat{g}\times\hat{t}} & z_{\hat{g}\times\hat{t}} \\
x_t & y_t & z_t \\
x_{-g} & y_{-g} & z_{-g}
\end{pmatrix}\\\\\Downarrow\\\\
R_{view} = 
\begin{pmatrix}
x_{\hat{g}\times\hat{t}} & y_{\hat{g}\times\hat{t}} & z_{\hat{g}\times\hat{t}} & 0\\
x_t & y_t & z_t & 0\\
x_{-g} & y_{-g} & z_{-g} & 0\\
0 & 0 & 0 & 1
\end{pmatrix}
\begin{pmatrix}
1 & 0 & 0 & -P_{camera_x}\\
0 & 1 & 0 & -P_{camera_y}\\
0 & 0 & 1 & -P_{camera_z}\\
0 & 0 & 0 & 1
\end{pmatrix}
}
$$

##### 1.7.1.2.2.2 Projection transformation - 投影变换

###### 1.7.1.2.2.2.1 Orthographic projection - 正交变换

map a cuboid: \[l, r\], \[b, t\], \[**f**, **n**\] to the "canonical" cube \[-1, 1\]^3^

使用的手段包括：位移、缩放等
$$
M_{ortho} = 
\begin{bmatrix}
\frac{2}{r-l} & 0 & 0 & 0\\
0 & \frac{2}{t-b} & 0 & 0\\
0 & 0 & \frac{2}{n-f} & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
\begin{bmatrix}
1 & 0 & 0 & -\frac{r+l}{2}\\
0 & 1 & 0 & -\frac{t+b}{2}\\
0 & 0 & 1 & -\frac{n+f}{2}\\
0 & 0 & 0 & 1
\end{bmatrix}
$$


###### 1.7.1.2.2.2.2 Perspective projection - 透视变换

GAMES101：

> 将Perspective projection先挤压为Orthographic projection，然后再依照正交变换的思路做projection

$$
M_{persp} = 
\begin{pmatrix}
n & 0 & 0 & 0\\
0 & n & 0 & 0\\
0 & 0 & n+f & -nf\\
0 & 0 & 1 & 0
\end{pmatrix}
$$



## 1.7.2 利用GLM变换

GLM版本：

> 从0.9.9版本开始，GLM创建矩阵的默认值为零矩阵；在此之前的版本则是单位矩阵。使用语句初始化：
>
> ```cpp
> glm::mat4 mat = glm::mat4(1.0f);
> ```

现在的情况：

在CPU侧提前完成变换矩阵的计算，然后将该结果传递到shader处用于position的计算

```cpp
// main.cpp处
// 1. 计算好变换矩阵
glm::mat4 trans = glm::mat4(1.0f);
trans = glm::rorate(trans, glm::radians(90.0f), glm::vec3(0.0f, 0.0f, 1.0f));
trans = glm::scale(trans, glm::vec3(0.5f, 0.5f, 0.5f));

// 2. 传递值 - uniform手段
Gluint transformLoc = glGetUniformLocation(...);
glUniformMatrix4fv(transformLoc, 1, GL_FALSE, trans);
```

```glsl
// vertex shader处
// 3. 实际使用
uniform mat4 transform;
...
gl_Position = transform * vec4(position, 1.0f);
```
