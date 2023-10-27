---
title: Markdown
date: 2021-12-23 12:23:23
tags:
  - Markdown
  - Language Learning
categories:
  - Language Learning
  - Markdown
mathjax: true
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

我展示的是一级标题
=================

我展示的是二级标题
-----------------
	一、二级标题上不能有东西

# 一级标题

## 二级标题

###   三级标题

####    四级标题

##### 五级标题

###### 六级标题

*斜体文本* _斜体文本_
**粗体文本** __粗体文本__
***粗斜体文本*** ___粗斜体文本___

**分隔线:**

***

* * *

*****

- - -

----------
**删除线:**
~~BAIDU.COM~~

**下划线:**
<u>带下划线文本</u>

**上标：**

2^3^

**下标：**

H~2~O

**高亮：**

==22==

**内联公式：**

$\LaTeX$

**无序列表：**

* 第一项
* 第二项
* 第三项

+ 第一项
+ 第二项
+ 第三项


- 第一项
- 第二项
- 第三项

**有序列表:**

1. 第一项
2. 第二项
3. 第三项

允许列表嵌套

**区块**

> 区块引用1
> 区块引用2
> 区块引用3
> 最外层
> > 第一层嵌套
> >
> > > 第二层嵌套

**区块中使用列表:**
> 区块中使用列表
> 1. 第一项
> 2. 第二项
> + 第一项
> + 第二项
> + 第三项

**列表中使用区块**
如果要在列表项目内放进区块，
那么就需要在 > 前添加四个空格的缩进。

区块中使用列表实例如下：

* 第一项
    > 区块引用1
    > 区块引用2
* 第二项

**代码**
`printf()` 函数

可以用 ``` 包裹一段代码，
并指定一种语言（也可以不指定）：

```javascript
$(document).ready(function () {
    alert('RUNOOB');
});
```

**链接**

这是一个链接：[google](https://google.com)

或者直接放上地址：<https://google.com>

> 还可以通过变量来设置一个链接，变量赋值在文档末尾进行：
>
> 这个链接用 1 作为网址变量 [Google][1]
>
> 这个链接用 runoob 作为网址变量 [Runoob][runoob]
>
> 然后在文档的结尾为变量赋值（网址）
>
> [1]: http://www.google.com/
> [runoob]: http://www.runoob.com/

**图片**

```markdown
![alt 属性文本](图片地址 "可选标题")
```

![RUNOOB 图标](https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg)

Markdown 还没有办法指定图片的高度与宽度
如果你需要的话，你可以使用普通的 <img> 标签。
<img src="https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg" width="50%">

**表格**

| 左对齐 | 右对齐 | 居中对齐 |
| :-----| ----: | :----: |
| 单元格 | 单元格 | 单元格 |
| 单元格 | 单元格 | 单元格 |

**支持的 HTML 元素**
不在 Markdown 涵盖范围之内的标签
都可以直接在文档里面用 HTML 撰写

目前支持的 HTML 元素有：
<kbd> <b> <i> <em> <sup> <sub> <br>等，可自行尝试

**转义**
md使用了很多特殊符号来表示特定的意义
如果需要显示特定的符号则需要使用转义字符
md使用反斜杠转义特殊字符：

**文本加粗** 

\*\* 正常显示星号 \*\*

md支持以下这些符号前面加上反斜杠
来帮助插入普通的符号：
\   反斜线
`   反引号

*   星号
_   下划线
{}  花括号
[]  方括号
()  小括号
#   井字号
+   加号
-   减号
.   英文句点
!   感叹号

# 数学公式：

[LaTeX数学符号大全_LCCFlccf的博客-CSDN博客_latex 数学符号](https://blog.csdn.net/LCCFlccf/article/details/89643585)

[【Markdown笔记】数学公式 三角函数_dadalaohua的博客-CSDN博客_markdown 三角函数](https://blog.csdn.net/u012028275/article/details/119839411)

[markdown最全数学公式速查_博客-CSDN博客_markdown数学公式](https://blog.csdn.net/jyfu2_12/article/details/79207643)

[Markdown中的行列式和矩阵(LaTex) - 简书 (jianshu.com)](https://www.jianshu.com/p/8618faf9b34f)

## 输入公式：

### 最速解决：

[在线LaTeX公式编辑器-编辑器 (latexlive.com)](https://www.latexlive.com/)

### 公式标签：

在行内输入：$ $符号，在这两个符号之间输入LaTex语法，即可实现在行内插入公式。

在行间输入：$$ $$符号，在这两对符号之间输入LaTex语法，即可实现在行间插入公式。

示例：

 $\alpha$、$\beta$、$\chi$、$\Delta$、$\Gamma$、$\Theta$
$$
\alpha
$$


### 公式语法：

#### **换行**：

`\\`

#### 对齐：

##### &对齐

- 一般对齐：

$$
\begin{aligned}
&y=kx+b\\
&y=kx^2+b\\
\end{aligned}
$$

  - 等号对齐

$$
\begin{aligned}
y=&kx+b\\
kx^2+b=&y\\
\end{aligned}
$$

  - 灵活对齐

$$
\begin{aligned}
&y&&=kx+b\\
&kx^2+b&&=y\\
\end{aligned}
$$

##### 中心对齐：

default

#### 分数：

\frac{分子}{分母}

**求和**：

- 参数位于右方：\sum_{i=1}^{n}
- 参数位于上下方：\sum\limits_{i=1}^{n}

**求导**：

- d - 求导：\frac{\mathrm{d} y }{\mathrm{d} x}
- ∂ - 偏导：\frac{\partial f}{\partial x}
- ∇ - 梯度：\nabla f



## 附录：

### 1.字母的LaTex语法

#### 希腊字母

| 希腊字母小写、大写 | LaTeX形式               | 希腊字母小写、大写 | LaTeX形式         |
| ------------------ | ----------------------- | ------------------ | ----------------- |
| **α** **A**        | \alpha A                | **μ** **N**        | \mu N             |
| **β** **B**        | \beta B                 | **ξ** **Ξ**        | \xi \Xi           |
| **γ** **Γ**        | \gamma \Gamma           | **o** **O**        | o O               |
| **δ** **Δ**        | \delta \Delta           | **π** **Π**        | \pi \Pi           |
| **ϵ** **ε** **E**  | \epsilon \varepsilon E  | **ρ** **ϱ** **P**  | \rho \varrho P    |
| **ζ** **Z**        | \zeta Z                 | **σ** **Σ**        | \sigma \Sigma     |
| **η** **H**        | \eta H                  | **τ** **T**        | \tau T            |
| **θ** **ϑ** **Θ**  | \theta \vartheta \Theta | **υ** **Υ**        | \upsilon \Upsilon |
| **ι** **I**        | \iota I                 | **ϕ** **φ** **Φ**  | \phi \varphi \Phi |
| **κ** **K**        | \kappa K                | **χ** **X**        | \chi X            |
| **λ** **Λ**        | \lambda \Lambda         | **ψ** **Ψ**        | \psi \Psi         |
| **μ** **M**        | \mu M                   | **ω** **Ω**        | \omega \Omega     |

### 2.常用运算符

| 运算符        | latex       |
| ------------- | ----------- |
| $\le$         | \le         |
| $\ge$         | \ge         |
| $\neq$        | \neq 或 \ne |
| $\equiv$      | \equiv      |
| $\triangleq$  | \triangleq  |
| $\rightarrow$ | \rightarrow |
| $\infty$      | \infty      |
| $\in$         | \in         |
| $\subset$     | \subset     |
| $\subseteq$   | \subseteq   |
| $\supset$     | \supset     |
| $\supseteq$   | \supseteq   |
| $\times$      | \times      |
| $\cdot$       | \cdot       |
| \sqrt         | \sqrt       |

