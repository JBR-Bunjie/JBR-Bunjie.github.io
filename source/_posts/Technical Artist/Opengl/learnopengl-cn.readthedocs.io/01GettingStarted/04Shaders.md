# 1.5 Shaders

> 着色器(Shader)是运行在GPU上的小程序。这些小程序为图形渲染管线的某个特定部分而运行。
>
> 从基本意义上来说，着色器只是一种把输入转化为输出的程序。**着色器也是一种非常独立的程序，因为它们之间不能相互通信；它们之间唯一的沟通只有通过输入和输出。**

## 1.5.1 GLSL

OpenGL中的Shader是用一种叫GLSL的类C语言写成的。GLSL是为图形计算量身定制的，它包含一些针对向量和矩阵操作的有用特性。

着色器的开头总是要声明版本，接着是输入和输出变量、uniform和main函数。每个着色器的入口点都是main函数，在这个函数中我们处理所有的输入变量，并将结果输出到输出变量中。一个典型的着色器有下面的结构：

```glsl
#version version_number

in type in_variable_name;
in type in_variable_name;

out type out_variable_name;

uniform type uniform_name;

int main()
{
  // 处理输入并进行一些图形操作
  ...
  // 输出处理过的结果到输出变量
  out_variable_name = weird_stuff_we_processed;
}
```

特别的，当我们特别谈论到顶点着色器的时候，每个**输入变量**也叫**顶点属性**(Vertex Attribute)。我们能声明的顶点属性是有上限的，它一般由硬件来决定。OpenGL确保至少有16个包含4分量的顶点属性可用，但是有些硬件或许允许更多的顶点属性，你可以查询GL_MAX_VERTEX_ATTRIBS来获取具体的上限：

```
GLint nrAttributes;
glGetIntegerv(GL_MAX_VERTEX_ATTRIBS, &nrAttributes);
std::cout << "Maximum nr of vertex attributes supported: " << nrAttributes << std::endl;
```

通常情况下它至少会返回16个，大部分情况下是够用了。

## 1.5.2 数据类型

GLSL有数据类型可以来指定变量的种类。

GLSL中包含C等其它语言大部分的默认基础数据类型：`int`、`float`、`double`、`uint`和`bool`。

GLSL也有两种特殊的但是**容器**类型，分别是向量(Vector)和矩阵(Matrix)

## 1.5.2.1 向量容器

> 向量是一种灵活的数据类型，我们可以把用在各种输入和输出上。

#### 1.5.2.1.1 向量容器的存在形式

> GLSL中的向量是一个可以包含有1、2、3或者4个分量的容器，分量的类型可以是前面默认基础类型的任意一个。

在GLSL中，向量容器中可以容纳的基本内容可以是：

| 类型    | 含义                            |
| ------- | ------------------------------- |
| `vecn`  | 包含`n`个float分量的默认向量    |
| `bvecn` | 包含`n`个bool分量的向量         |
| `ivecn` | 包含`n`个int分量的向量          |
| `uvecn` | 包含`n`个unsigned int分量的向量 |
| `dvecn` | 包含`n`个double分量的向量       |

#### 1.5.2.1.2 向量容器内容获取

以`vec4`类型的向量"test"为例，想要获取其中的各个分量，可以使用：

`test.x`; `test.y`; `test.z`; `test.w` （位置坐标）

这种分量的获取方式，完全等同于 `rgba`（颜色值） 与 `stpq`（纹理坐标），但是为了维护程序良好的可读性，请务必根据具体数据类型选择分量的获取方式

#### 1.5.2.1.3 Swizzling: 向量容器内容的“重组”

向量这一数据类型也允许一些有趣而灵活的分量选择方式，叫做重组(Swizzling)。重组允许这样的语法：

```glsl
vec2 someVec;
vec4 differentVec = someVec.xyxx;
vec3 anotherVec = differentVec.zyw;
vec4 otherVec = someVec.xxxx + anotherVec.yxzy;
```

你可以使用上面4个字母任意组合来创建一个和原来向量一样长的（同类型）新向量，只要原来向量有那些分量即可；然而，你不允许在一个`vec2`向量中去获取`.z`元素。我们也可以**把一个向量作为一个参数**传给不同的向量构造函数，以减少需求参数的数量：

```glsl
vec2 vect = vec2(0.5f, 0.7f);
vec4 result = vec4(vect, 0.0f, 0.0f);
vec4 otherResult = vec4(result.xyz, 1.0f);
```

### 1.5.2.2 矩阵

## 1.5.3 数据输入与输出

> 虽然Shader是各自独立的小程序，但是它们都是渲染管线上的一小部分，出于这样的原因，我们希望每个Shader都有输入和输出，这样才能进行数据交流和传递。GLSL定义了`in`和`out`关键字专门来实现这个目的。每个着色器使用这两个关键字设定输入和输出，只要一个输出变量与下一个着色器阶段的输入匹配，它就会传递下去。但在顶点和片段着色器中会有点不同。

顶点着色器应该接收的是一种特殊形式的输入，否则就会效率低下。顶点着色器的输入特殊在，它从**顶点数据**中直接接收输入。为了定义顶点数据该如何管理，我们使用`location`这一元数据指定输入变量，这样我们才可以在CPU上配置顶点属性。我们已经在前面的教程看过这个了，`layout (location = 0)`。顶点着色器需要为它的输入提供一个额外的`layout`标识，这样我们才能把它链接到顶点数据。

> [Vertex Shader - OpenGL Wiki (khronos.org)](https://www.khronos.org/opengl/wiki/Vertex_Shader)
>
> [Fragment Shader - OpenGL Wiki (khronos.org)](https://www.khronos.org/opengl/wiki/Fragment_Shader#Outputs)
>
> [opengl - How does the fragment shader know what variable to use for the color of a pixel? - Stack Overflow](https://stackoverflow.com/questions/9222217/how-does-the-fragment-shader-know-what-variable-to-use-for-the-color-of-a-pixel)

如果我们打算从一个着色器向另一个着色器发送数据，我们必须在发送方着色器中声明一个输出，在接收方着色器中声明一个类似的输入。当类型和名字都一样的时候，OpenGL就会把两个变量链接到一起，它们之间就能发送数据了（这是在链接程序对象时完成的）。

示例如下：

```glsl
// 顶点着色器
#version 330 core
layout (location = 0) in vec3 position; // position变量的属性位置值为0
out vec4 vertexColor; // 为片段着色器指定一个颜色输出
void main() {
    gl_Position = vec4(position, 1.0); // 注意我们如何把一个vec3作为vec4的构造器的参数
    vertexColor = vec4(0.5f, 0.0f, 0.0f, 1.0f); // 把输出变量设置为暗红色
}

// 片段着色器
#version 330 core
in vec4 vertexColor; // 从顶点着色器传来的输入变量（名称相同、类型相同）
out vec4 color; // 片段着色器输出的变量名可以任意命名，但类型必须是vec4
void main() {
    color = vertexColor;
}
```

## 1.5.4 Uniform

> Uniform是一种**从CPU中的应用向GPU中的着色器发送数据的方式**，但uniform和顶点属性有些不同。首先，uniform是**全局**的(Global)，这意味着它可以被着色器程序的任意着色器在任意阶段访问。在完成一次设置（赋值）后，无论你把uniform值设置成什么，uniform会一直保存它们的数据，直到它们被重置或更新。

我们可以在一个着色器中添加`uniform`关键字至类型和变量名前来声明一个GLSL的uniform。

示例如下：我在片段着色器中声明一个uniform `vec4`的变量，并把片段着色器的输出颜色设置为该变量的值。

```glsl
#version 330 core
out vec4 color;

uniform vec4 ourColor; // 在OpenGL程序代码中设定这个变量

void main() {
    color = ourColor;
}  
```

由于uniform变量是全局变量，因此我们可以在任何shader中定义它们，而无需像`1.5.3`那样用in out作为中介来传递到目标Shader中。因此，既然顶点着色器中不需要这个uniform，那我们便不在那里定义它。

:warning:如果你声明了一个uniform却在GLSL代码中没用过，编译器会静默移除这个变量

而在shader中写下的ourColor，只是一个声明/定义，这个uniform现在还是空的，我们还没有给它添加任何数据。还记得吗：“Uniform是一种从CPU中的应用向GPU中的着色器发送数据的方式”，也就是说，Uniform的具体定义及实现应该在CPU也就是main.c里完成而不是Shader中

我们可以这样：

```c
...(game loop); // 做底色等等

GLint vertexColorLocation = glGetUniformLocation(shaderProgram, "ourColor"); // 在CPU阶段，查找对应Uniform定义
// 拿到该Uniform变量后，我们就可以对其进行处理。

GLfloat timeValue = glfwGetTime(); // 获取时间
GLfloat greenValue = (sin(timeValue) / 2) + 0.5; // 对时间求sin，结果在(-1, 1)，值在(0, 1)波动

glUseProgram(shaderProgram); // 先激活着色器(Use)，再更新Uniform
glUniform4f(vertexColorLocation, 0.0f, greenValue, 0.0f, 1.0f);

...(draw&call); // draw call在这之后
```

## 1.5.5 解决更多属性

> 如有必要，请重新回看1.4部分

在前面的教程中，我们了解了如何填充VBO、配置顶点属性指针以及如何把它们都储存到一个VAO里。这次，我们同样打算把颜色数据加进顶点数据中。我们将把颜色数据添加为3个float值至vertices数组。我们将把三角形的三个角分别指定为红色、绿色和蓝色：

```c
GLfloat vertices[] = {
    // 位置              // 颜色
     0.5f, -0.5f, 0.0f,  1.0f, 0.0f, 0.0f,   // 右下
    -0.5f, -0.5f, 0.0f,  0.0f, 1.0f, 0.0f,   // 左下
     0.0f,  0.5f, 0.0f,  0.0f, 0.0f, 1.0f    // 顶部
};
```

根据我们的数据，我们需要做的修改如下：

- VS，接收新的Color信息
- FS，接受该Color信息
- main.c，对该数据进行解析，配置VAO指针等

特别注意，`glVertexAttribPointer`函数有两处修改：

```c
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(GLfloat), (GLvoid*)0);
glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(GLfloat), (GLvoid*)(3 * sizeof(GLfloat)));
// 1. 修改了步长
// 2. 修改了偏移量，注意偏移量的具体格式与计算思路（对于每个顶点来说，位置顶点属性在前，所以它的偏移量是0。颜色属性紧随位置数据之后，所以偏移量就是3 * sizeof(GLfloat)，用字节来计算就是12字节）
```

这个图片可能不是你所期望的"只有3个颜色"(因为我们只提供了3个颜色)，这是一个大调色板——这是在片段着色器中进行的所谓片段插值(Fragment Interpolation)的结果。

>当渲染一个三角形时，光栅化(Rasterization)阶段通常会造成比原指定顶点更多的片段。光栅会根据每个片段在三角形形状上所处相对位置决定这些片段的位置。基于这些位置，它会插值(Interpolate)所有片段着色器的输入变量。比如说，我们有一个线段，上面的端点是绿色的，下面的端点是蓝色的。如果一个片段着色器在线段的70%的位置运行，它的颜色输入属性就会是一个绿色和蓝色的线性结合；更精确地说就是30%蓝 + 70%绿。
>
>这正是在这个三角形中发生了什么。我们有3个顶点，和相应的3个颜色，从这个三角形的像素来看它可能包含50000左右的片段，片段着色器为这些像素进行插值颜色。如果你仔细看这些颜色就应该能明白了：红首先变成到紫再变为蓝色。片段插值会被应用到片段着色器的所有输入属性上。

## 1.5.6 我们自己的着色器类

>编写、编译、管理着色器是件麻烦事。在着色器主题的最后，我们会写一个类来让我们的生活轻松一点，它可以从硬盘读取着色器，然后编译并链接它们，并对它们进行错误检测，这就变得很好用了。这也会让你了解该如何封装目前所学的知识到一个抽象对象中。
>
>我们会把着色器类全部放在在头文件里，主要是为了学习用途，当然也方便移植。我们先来添加必要的include，并定义类结构：
>
>```
>#ifndef SHADER_H
>#define SHADER_H
>
>#include <string>
>#include <fstream>
>#include <sstream>
>#include <iostream>
>
>#include <GL/glew.h>; // 包含glew来获取所有的必须OpenGL头文件
>
>class Shader
>{
>public:
>    // 程序ID
>    GLuint Program;
>    // 构造器读取并构建着色器
>    Shader(const GLchar* vertexPath, const GLchar* fragmentPath);
>    // 使用程序
>    void Use();
>};
>
>#endif
>```
>
>:warning:在上面，我们在头文件顶部使用了几个预处理指令(Preprocessor Directives)。这些预处理指令会告知你的编译器只在它没被包含过的情况下才包含和编译这个头文件，即使多个文件都包含了这个着色器头文件。它是用来防止链接冲突的。
>
>着色器类储存了着色器程序的ID。它的构造器需要顶点和片段着色器源代码的文件路径，这样我们就可以把源码的文本文件储存在硬盘上了。我们还添加了一个Use函数，它其实不那么重要，但是能够显示这个自造类如何让我们的生活变得轻松（虽然只有一点）。
>
>## 从文件读取
>
>我们使用C++文件流读取着色器内容，储存到几个`string`对象里：
>
>```
>Shader(const GLchar* vertexPath, const GLchar* fragmentPath)
>{
>    // 1. 从文件路径中获取顶点/片段着色器
>    std::string vertexCode;
>    std::string fragmentCode;
>    std::ifstream vShaderFile;
>    std::ifstream fShaderFile;
>    // 保证ifstream对象可以抛出异常：
>    vShaderFile.exceptions(std::ifstream::badbit);
>    fShaderFile.exceptions(std::ifstream::badbit);
>    try 
>    {
>        // 打开文件
>        vShaderFile.open(vertexPath);
>        fShaderFile.open(fragmentPath);
>        std::stringstream vShaderStream, fShaderStream;
>        // 读取文件的缓冲内容到流中
>        vShaderStream << vShaderFile.rdbuf();
>        fShaderStream << fShaderFile.rdbuf();       
>        // 关闭文件
>        vShaderFile.close();
>        fShaderFile.close();
>        // 转换流至GLchar数组
>        vertexCode = vShaderStream.str();
>        fragmentCode = fShaderStream.str();     
>    }
>    catch(std::ifstream::failure e)
>    {
>        std::cout << "ERROR::SHADER::FILE_NOT_SUCCESFULLY_READ" << std::endl;
>    }
>    const GLchar* vShaderCode = vertexCode.c_str();
>    const GLchar* fShaderCode = fragmentCode.c_str();
>    [...]
>```
>
>下一步，我们需要编译和链接着色器。注意，我们也将检查编译/链接是否失败，如果失败则打印编译时错误，调试的时候这些错误输出会及其重要（你总会需要这些错误日志的）：
>
>```
>// 2. 编译着色器
>GLuint vertex, fragment;
>GLint success;
>GLchar infoLog[512];
>
>// 顶点着色器
>vertex = glCreateShader(GL_VERTEX_SHADER);
>glShaderSource(vertex, 1, &vShaderCode, NULL);
>glCompileShader(vertex);
>// 打印编译错误（如果有的话）
>glGetShaderiv(vertex, GL_COMPILE_STATUS, &success);
>if(!success)
>{
>    glGetShaderInfoLog(vertex, 512, NULL, infoLog);
>    std::cout << "ERROR::SHADER::VERTEX::COMPILATION_FAILED\n" << infoLog << std::endl;
>};
>
>// 片段着色器也类似
>[...]
>
>// 着色器程序
>this->Program = glCreateProgram();
>glAttachShader(this->Program, vertex);
>glAttachShader(this->Program, fragment);
>glLinkProgram(this->Program);
>// 打印连接错误（如果有的话）
>glGetProgramiv(this->Program, GL_LINK_STATUS, &success);
>if(!success)
>{
>    glGetProgramInfoLog(this->Program, 512, NULL, infoLog);
>    std::cout << "ERROR::SHADER::PROGRAM::LINKING_FAILED\n" << infoLog << std::endl;
>}
>
>// 删除着色器，它们已经链接到我们的程序中了，已经不再需要了
>glDeleteShader(vertex);
>glDeleteShader(fragment);
>```
>
>最后我们也会实现Use函数：
>
>```
>void Use()
>{
>    glUseProgram(this->Program);
>}
>```
>
>现在我们就写完了一个完整的着色器类。使用这个着色器类很简单；只要创建一个着色器对象，从那一点开始我们就可以开始使用了：
>
>```
>Shader ourShader("path/to/shaders/shader.vs", "path/to/shaders/shader.frag");
>...
>while(...)
>{
>    ourShader.Use();
>    glUniform1f(glGetUniformLocation(ourShader.Program, "someUniform"), 1.0f);
>    DrawStuff();
>}
>```
>
>我们把顶点和片段着色器储存为两个叫做`shader.vs`和`shader.frag`的文件。你可以使用自己喜欢的名字命名着色器文件；我自己觉得用`.vs`和`.frag`作为扩展名很直观。
>
>源码：[使用新着色器类的程序](http://learnopengl.com/code_viewer.php?code=getting-started/shaders-using-object)，[着色器类](http://learnopengl.com/code_viewer.php?type=header&code=shader)，[顶点着色器](http://learnopengl.com/code_viewer.php?type=vertex&code=getting-started/basic)，和[片段着色器](http://learnopengl.com/code_viewer.php?type=fragment&code=getting-started/basic)。
