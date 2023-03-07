# 1.6 Texture

## 1.6.1 纹理坐标与纹理的应用

### 1.6.1.1 定义纹理坐标

>我们已经了解到，我们可以为每个顶点添加颜色来增加图形的细节，从而创建出有趣的图像。但是，如果想让图形看起来更真实，我们就必须有足够多的顶点，从而指定足够多的颜色。这将会产生很多额外开销，因为每个模型都会需求更多的顶点，每个顶点又需求一个颜色属性。
>
>艺术家和程序员更喜欢使用纹理(Texture)。纹理是一个2D图片（不过也有1D&3D的纹理），它可以用来添加物体的细节；你可以想象纹理是一张绘有砖块的纸，无缝折叠贴合到你的3D的房子上，这样你的房子看起来就像有砖墙外表了。因为我们可以在一张图片上插入非常多的细节，这样就可以让物体非常精细而不用指定额外的顶点。

现在我们利用纹理，给三角形贴上一张[砖墙](https://learnopengl-cn.readthedocs.io/zh/latest/img/01/06/wall.jpg)图片。

![img](https://learnopengl-cn.readthedocs.io/zh/latest/img/01/06/textures.png)

为了能够把纹理`映射(Map)`到该三角形上，**我们需要指定三角形的每个顶点各自对应纹理的哪个部分**。这样每个顶点就会关联着一个纹理坐标(Texture Coordinate)，用来标明该从纹理图像的哪个部分采样。之后在图形的其它片段上进行片段插值(Fragment Interpolation)。

纹理坐标在x和y轴上，范围为0到1之间（注意我们使用的是2D纹理图像）。使用纹理坐标获取纹理颜色叫做采样(Sampling)。纹理坐标起始于(0, 0)，也就是**纹理图片的左下角**，终始于(1, 1)，即纹理图片的右上角。下面的图片展示了我们是如何把纹理坐标映射到三角形上的。

![img](https://learnopengl-cn.readthedocs.io/zh/latest/img/01/06/tex_coords.png)

我们为三角形指定了3个纹理坐标点。如上图所示：

- 我们希望三角形的左下角对应纹理的左下角，因此我们把三角形左下角顶点的纹理坐标设置为(0, 0)；
- 三角形的上顶点对应于图片的上中位置所以我们把它的纹理坐标设置为(0.5, 1.0)；
- 同理右下方的顶点设置为(1, 0)。

我们只要给顶点着色器传递这三个纹理坐标就行了，接下来它们会被传片段着色器中，它会为每个片段进行纹理坐标的插值。

纹理坐标看起来就像这样：

```c
GLfloat texCoords[] = {
    0.0f, 0.0f, // 左下角
    1.0f, 0.0f, // 右下角
    0.5f, 1.0f // 上中
};
```

### 1.6.1.2 使用纹理

类似于Shader程序，使用纹理之前我们应该先把它们加载到我们的（main.c CPU部分）应用中。

纹理图像可能被储存为各种各样的格式，每种都有自己特别的数据结构和排列，为了方便地将图像加载到应用中，一个可行的解决方案是选一个需要的文件格式，比如`.PNG`，然后自己写一个图像加载器，把图像转化为字节序列。这里又会涉及到很多问题，比如文件格式的选择甚至设计，图像加载器的性能？多格式支持？而且鉴于纹理使用的广泛程度，这些东西的设计和项目的成败都可以说是息息相关。

不过，这里为避免麻烦，我们选择直接借用参考答案：使用一个支持多种流行格式的图像加载库来为我们解决这个问题。比如：

#### 1.6.1.2.1: Use `stb_image.h` to load image：

> `stb_image.h` is a very popular single header image loading library by [Sean Barrett](https://github.com/nothings) that is able to load most popular file formats and is easy to integrate in your project(s). `stb_image.h` can be downloaded from [here](https://github.com/nothings/stb/blob/master/stb_image.h). Simply download the single header file, add it to your project as `stb_image.h`, and create an additional C++ file with the following code:

在stb_image.h中，可以发现使用该头文件所需要的引用与定义：

```c
#define STB_IMAGE_IMPLEMENTATION
#include "stb_image.h"
```

By defining STB_IMAGE_IMPLEMENTATION the preprocessor modifies the header file such that it only contains the relevant definition source code, effectively turning the header file into a `.cpp` file, and that's about it. Now simply include `stb_image.h` somewhere in your program and compile.

For the following texture sections we're going to use an image of a [wooden container](https://learnopengl.com/img/textures/container.jpg). To load an image using `stb_image.h` we use its stbi_load function:

```c
int width, height, nrChannels;
unsigned char *data = stbi_load("container.jpg", &width, &height, &nrChannels, 0); 
```

The function first takes as input the location of an image file. It then expects you to give three `ints` as its second, third and fourth argument that `stb_image.h` will fill with the resulting image's *width*, *height* and *number* of color channels. We need the image's width and height for generating textures later on.

#### 1.6.1.2.2: Generate Texture

在前面，我们已经完成了对图片数据的读取，此时就应该回到OpenGL上来——开始准备生成纹理了

```c
GLuint texture; // ID
glGenTexture(1, &texture); // The glGenTextures function first takes as input how many textures we want to generate and stores them in a unsigned int array given as its second argument (in our case just a single unsigned int). 

// Just like other objects we need to bind it, so any subsequent texture commands will configure the currently bound texture:
glBindTexture(GL_TEXTURE_2D, texture[0]);

// 为当前绑定的纹理对象设置环绕、过滤方式
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);

// 生成纹理
glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, width, height, 0, GL_RGB, GL_UNSIGNED_BYTE, data);
glGenerateMipmap(GL_TEXTURE_2D); // 多层渐变纹理
```

glTexImage2D参数不少：

- 第一个参数指定了纹理目标(Target)。设置为GL_TEXTURE_2D意味着会生成与当前绑定的纹理对象在同一个目标上的纹理（任何绑定到GL_TEXTURE_1D和GL_TEXTURE_3D的纹理不会受到影响）。
- 第二个参数为纹理指定多级渐远纹理的级别，如果你希望单独手动设置每个多级渐远纹理的级别的话。这里我们填0，也就是基本级别。
- 第三个参数告诉OpenGL我们希望把纹理储存为何种格式。我们的图像只有`RGB`值，因此我们也把纹理储存为`RGB`值。
- 第四个和第五个参数设置最终的纹理的宽度和高度。我们之前加载图像的时候储存了它们，所以我们使用对应的变量。
- 下个参数应该总是被设为`0`（历史遗留问题）。
- 第七第八个参数定义了源图的格式和数据类型。我们使用RGB值加载这个图像，并把它们储存为`char`(**byte**)数组，我们将会传入对应值。
- 最后一个参数是真正的图像数据。

当调用glTexImage2D时，当前绑定的纹理对象就会被附加上纹理图像。然而，目前只有基本级别(Base-level)的纹理图像被加载了，如果要使用多级渐远纹理，我们必须手动设置所有不同的图像（不断递增第二个参数）。或者，直接在生成纹理之后调glGenerateMipmap。这会为当前绑定的纹理自动生成所有需要的多级渐远纹理。

After we're done generating the texture and its corresponding mipmaps, it is good practice to free the image memory:

```c
stbi_image_free(data);
glBindTexture(GL_TEXTURE_2D, 0);
```

## 1.6.2 纹理采样方式

>纹理坐标的范围通常是从(0, 0)到(1, 1)，那如果我们**把纹理坐标设置在范围之外**会发生什么？OpenGL默认的行为是重复这个纹理图像（我们基本上忽略浮点纹理坐标的整数部分），但OpenGL提供了更多的选择：
>
>| 环绕方式(Wrapping) | 描述                                                         |
>| ------------------ | ------------------------------------------------------------ |
>| GL_REPEAT          | 对纹理的**默认**行为。重复纹理图像。                         |
>| GL_MIRRORED_REPEAT | 和GL_REPEAT一样，但每次重复图片是镜像放置的。                |
>| GL_CLAMP_TO_EDGE   | 纹理坐标会被约束在0到1之间，超出的部分会重复纹理坐标的边缘，产生一种边缘被拉伸的效果。 |
>| GL_CLAMP_TO_BORDER | 超出的坐标为用户指定的边缘颜色。                             |
>
>当纹理坐标超出默认范围时，每个选项都有不同的视觉效果输出。我们来看看这些纹理图像的例子：
>
>![img](https://learnopengl-cn.readthedocs.io/zh/latest/img/01/06/texture_wrapping.png)

> [glTexParameter - OpenGL 4 Reference Pages (khronos.org)](https://registry.khronos.org/OpenGL-Refpages/gl4/html/glTexParameter.xhtml)
>
> [opengl - What does changing GL_TEXTURE_WRAP)_(S/T) do? - Game Development Stack Exchange](https://gamedev.stackexchange.com/questions/62548/what-does-changing-gl-texture-wrap-s-t-do)

前面提到的每个选项都可以使用glTexParameter*函数对单独的一个坐标轴设置（`s`、`t`（如果是使用3D纹理那么还有一个`r`）它们和`x`、`y`、`z`是等价的）：

```C
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_MIRRORED_REPEAT);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_MIRRORED_REPEAT);
```

- 第一个参数指定了纹理目标；我们使用的是2D纹理，因此纹理目标是GL_TEXTURE_2D。
- 第二个参数需要我们指定，`设置的选项` 与 `应用的纹理轴`。我们打算配置的是`WRAP`选项，并且指定`S`和`T`轴。
- 最后一个参数需要我们传递一个环绕方式，在这个例子中OpenGL会给当前激活的纹理设定纹理环绕方式为GL_MIRRORED_REPEAT。

>
>如果我们选择GL_CLAMP_TO_BORDER选项，我们需要执行的步骤是这些：指定一个边缘的颜色，使用glTexParameter函数的`fv`后缀形式，用GL_TEXTURE_BORDER_COLOR作为它的选项：
>
>```C
>float borderColor[] = { 1.0f, 1.0f, 0.0f, 1.0f };
>glTexParameterfv(GL_TEXTURE_2D, GL_TEXTURE_BORDER_COLOR, borderColor);
>```

## 1.6.3 纹理过滤

> 插值办法

OpenGL需要知道怎样将纹理像素（Texture Pixel、Texel、纹素）映射到纹理坐标。这在物体较大而纹理的分辨率较低时十分重要。

> Texture Pixel也叫Texel，你可以想象你打开一张`.jpg`格式图片，不断放大你会发现它是由无数像素点组成的，这个点就是纹理像素；注意不要和纹理坐标搞混，纹理坐标是你给模型顶点设置的那个数组，OpenGL以这个顶点的纹理坐标数据去查找纹理图像上的像素，然后进行采样提取纹理像素的颜色。

GL_NEAREST（也叫邻近过滤，Nearest Neighbor Filtering）是OpenGL默认的纹理过滤方式。当设置为GL_NEAREST的时候，OpenGL会选择中心点最接近纹理坐标的那个像素。下图中你可以看到四个像素，加号代表纹理坐标。左上角那个纹理像素的中心距离纹理坐标最近，所以它会被选择为样本颜色：

![img](https://learnopengl-cn.readthedocs.io/zh/latest/img/01/06/filter_nearest.png)

GL_LINEAR（也叫线性过滤，(Bi)linear Filtering）它会基于纹理坐标附近的纹理像素，计算出一个插值，近似出这些纹理像素之间的颜色。一个纹理像素的中心距离纹理坐标越近，那么这个纹理像素的颜色对最终的样本颜色的贡献越大。下图中你可以看到返回的颜色是邻近像素的混合色：

![img](https://learnopengl-cn.readthedocs.io/zh/latest/img/01/06/filter_linear.png)

那么这两种纹理过滤方式有怎样的视觉效果呢？让我们看看在一个很大的物体上应用一张低分辨率的纹理会发生什么吧（纹理被放大了，每个纹理像素都能看到）：

![img](https://learnopengl-cn.readthedocs.io/zh/latest/img/01/06/texture_filtering.png)

GL_NEAREST产生了颗粒状的图案，我们能够清晰看到组成纹理的像素，而GL_LINEAR能够产生更平滑的图案，很难看出单个的纹理像素。GL_LINEAR可以产生更真实的输出，但有些开发者更喜欢8-bit风格，所以他们会用GL_NEAREST选项。

当进行放大(Magnify)和缩小(Minify)操作的时候可以设置纹理过滤的选项，比如你可以在纹理被缩小的时候使用邻近过滤，被放大时使用线性过滤。我们需要使用glTexParameter*函数为放大和缩小指定过滤方式。这段代码看起来会和纹理环绕方式的设置很相似：

```c
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
```

## 1.6.4 多级渐远纹理

想象一下，假设我们有一个包含着上千物体的大房间，每个物体上都有纹理。有些物体会很远，但其纹理会拥有与近处物体同样高的分辨率。**由于远处的物体可能只产生很少的片段，OpenGL从高分辨率纹理中为这些片段获取正确的颜色值就很困难，因为它需要对一个跨过纹理很大部分的片段只拾取一个纹理颜色，这在小物体上这会产生不真实的感觉**，更不用说对它们使用高分辨率纹理浪费内存的问题了。

OpenGL使用一种叫做多级渐远纹理(Mipmap)的概念来解决这个问题，它简单来说就是一系列的纹理图像，后一个纹理图像是前一个的二分之一。多级渐远纹理背后的理念很简单：距观察者的距离超过一定的阈值，OpenGL会使用不同的多级渐远纹理，即最适合物体的距离的那个。由于距离远，解析度不高也不会被用户注意到。同时，多级渐远纹理另一加分之处是它的性能非常好。让我们看一下多级渐远纹理是什么样子的：

![img](https://learnopengl-cn.readthedocs.io/zh/latest/img/01/06/mipmaps.png)

手工为每个纹理图像创建一系列多级渐远纹理很麻烦，幸好OpenGL有一个glGenerateMipmaps函数，在创建完一个纹理后调用它OpenGL就会承担接下来的所有工作了。后面的教程中你会看到该如何使用它。

在渲染中切换多级渐远纹理级别(Level)时，OpenGL在两个不同级别的多级渐远纹理层之间会产生不真实的生硬边界。就像普通的纹理过滤一样，切换多级渐远纹理级别时你也可以在两个不同多级渐远纹理级别之间使用NEAREST和LINEAR过滤。为了指定不同多级渐远纹理级别之间的过滤方式，你可以使用下面四个选项中的一个代替原有的过滤方式：

| 过滤方式                  | 描述                                                         |
| ------------------------- | ------------------------------------------------------------ |
| GL_NEAREST_MIPMAP_NEAREST | 使用最邻近的多级渐远纹理来匹配像素大小，并使用邻近插值进行纹理采样 |
| GL_LINEAR_MIPMAP_NEAREST  | 使用最邻近的多级渐远纹理级别，并使用线性插值进行采样         |
| GL_NEAREST_MIPMAP_LINEAR  | 在两个最匹配像素大小的多级渐远纹理之间进行线性插值，使用邻近插值进行采样 |
| GL_LINEAR_MIPMAP_LINEAR   | 在两个邻近的多级渐远纹理之间使用线性插值，并使用线性插值进行采样 |

就像纹理过滤一样，我们可以使用glTexParameteri将过滤方式设置为前面四种提到的方法之一：

```c
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
```

一个常见的错误是，将放大过滤的选项设置为多级渐远纹理过滤选项之一。这样没有任何效果，因为多级渐远纹理主要是使用在纹理被缩小的情况下的：纹理放大不会使用多级渐远纹理，为放大过滤设置多级渐远纹理的选项会产生一个GL_INVALID_ENUM错误代码。

