---
title: Catalan Number
date: 2022-12-23 12:23:23
tags:
  - String Processing
  - Algorithm
categories:
  - Algorithm
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

wikipedia-en: [Catalan number - Wikipedia](https://en.wikipedia.org/wiki/Catalan_number?wprov=srpw1_0)

wikipedia-cn:[卡塔兰数 - 维基百科，自由的百科全书 (wikipedia.org)](https://zh.wikipedia.org/wiki/卡塔兰数)

强烈推荐这篇博客：<span id="blog1">[卡特兰(Catalan)数入门详解 - Morning_Glory - 博客园 (cnblogs.com)](https://www.cnblogs.com/Morning-Glory/p/11747744.html)</span>



<img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/58374aa2b2e2c016a5b313e2bbd59940a2e1a5f9">

## Catalan Number

> In [combinatorial mathematics](https://en.wikipedia.org/wiki/Combinatorics), the **Catalan numbers** are a [sequence](https://en.wikipedia.org/wiki/Sequence) of [natural numbers](https://en.wikipedia.org/wiki/Natural_number) that occur in various [counting problems](https://en.wikipedia.org/wiki/Enumeration), often involving [recursively](https://en.wikipedia.org/wiki/Recursion) defined objects. They are named after the French-Belgian mathematician [Eugène Charles Catalan](https://en.wikipedia.org/wiki/Eugène_Charles_Catalan) (1814–1894).
>
> 
> The first Catalan numbers for *n* = 0, 1, 2, 3, ... are
>
> ​	1, 1, 2, 5, 14, 42, 132, 429, 1430, 4862, 16796, 58786, ... (sequence [A000108](https://oeis.org/A000108) in the [OEIS](https://en.wikipedia.org/wiki/On-Line_Encyclopedia_of_Integer_Sequences)).

### 意义？

> <a href="#blog1">卡特兰数是一个在组合数学里经常出现的一个数列，它并没有一个具体的意义，却是一个十分常见的数学规律</a>

也就是说：只要我们能在实际解决问题的过程中，发现当前问题符合Catalan Number的定义(公式)，就可以直接利用Cantanlan的相关公式来解决

### 定义：

设h(n)为catalan数的第n项，令h(0)=1,h(1)=1，catalan数满足递推式：

h(n) = h(0) \* h(n-1) + h(1) \* h(n-2) + ... + h(n-1) \* h(0) (n≥2)

### 容易计算的推导公式：

*可以看英文维基的推导过程*

The *n*th Catalan number can be expressed directly in terms of [binomial coefficients](https://en.wikipedia.org/wiki/Binomial_coefficient) by

<img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/58374aa2b2e2c016a5b313e2bbd59940a2e1a5f9">

~~(公式中的括号表达式请勿用组合的方式来计算)~~

An alternative expression for *Cn* is

<img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/e3b75076b78d4c388d623c5111021d713efea031">

## usage

### 例题1：

题目：2N个人排队买电影票，N个人持5元买票，N个人持10元买票。售票处在售票前只有票没有钱，票价5元，问有多少种排队方式能让2N个人顺利买票，并且输出所有排队队列（不会因为找钱问题）

题解：设x为当前已购票人群中持五元的人数，设y为当前已购票人群中持十元的人数

则易知，任何时候都应有x >= y

将本题转换为坐标系上的问题

则有：

![image-20220126215930311](..\..\images\Algorithm\CatalanNumberPic01.png)

易知，所有在直线y = x之下的路径都是合法路径，而所有与y = x + 1有交集的路径都是非法路径

我们所需要做的，只是从所有的可能路径——C(2n, n)中，取出非法路径即可

将所有经过y = x + 1的非法路径(因为所有路径仍然都是要到达(n,n)的) 对直线y = x + 1进行对称

此时所有路径都会到达点 (n - 1, n + 1)

故易知，所有非法路径总数：C(2n, n-1)

故最终结果为：C(2n, n) - C(2n, n-1)



**由此题我们可以看出Catalan Number类题目的相关特征——使用高度相关的两种数据进行先后排序**

### 例题2：

题目：电影院卖电影票，但是没有零钱找，票价一张 5 元，买票的人为 n 个持有 5 元，m个持 有 10 元，求解出可能的买票序列的个数，使得电影院能够将票卖完。

测试数据：

> 1. n=3, m=3 
>
>   输出：180
>
> 2. n=5, m=3
>
>   输出：20160 
>
> 3. n=100, m=100
>
>    输出：7808493736285054490617457563685000616783524531556170923710322111330291583796072702181230534772124989150269427118016226042154879111313238663979471534186434961519434230403597200370267217266558867539125388517366666256080507202260345081955685568391820824161596607976035333269564672318518060023284166918774048734879105185187102720000000000000000000000000000000000000000000000000

解法：和例题一几乎一致，但是具体公式不再能直接套用

我们仍有相同含义的x(x - n - 5元)与y(y - m - 10元)，但是具体公式开始不同

其中，所有可能的顺序为：C(m+n, n)

不合法的路径仍与y = x + 1有关，但是最终的非法内容有：C(m + n, m-1)

故最终有：C(m + n, n) - C(m + n, m - 1)

是不是有点疑惑？怎么算出来和结果不一样，好像多除以了(m! \* n!)？

若是将每个人视为相同的人，是无序的话，结果解释上述表达式，但是实际上每个人都不尽相同，为了保证顺序，还需要乘以(m! * n!) (**组合**方式已定，每组内自由**排列**)，最终可得到答案(m + n)! * (n - m + 1)/(n + 1)，事实上，上一题也应该作这样子的*额外*处理才对

## 更多典例

**1. 出栈次序**
一个栈(无穷大)的进栈序列为1，2，3，…，n，有多少个不同的出栈序列?

**2. 01序列**
给出一个n，要求一个长度为2n的01序列，使得序列的任意前缀中1的个数不少于0的个数， 有多少个不同的01序列?
以下为长度为6的序列:
111000 101100 101010 110010 110100

**3. ‘+1’ ‘-1’序列**

n个+1和n个-1构成的2n项 a1,a2,⋅⋅⋅,a2na1,a2,⋅⋅⋅,a2n，其部分和满足非负性质，即a1+a2+⋅⋅⋅+ak>=0a1+a2+⋅⋅⋅+ak>=0，(k=1,2,···,2n) ，有多少个不同的此序列?

**4. 括号序列**

n对括号有多少种匹配方式？

**5. 找零问题**

2n个人要买票价为五元的电影票，每人只买一张，但是售票员没有钱找零。其中，n个人持有五元，另外n个人持有十元，问在不发生找零困难的情况下，有多少种排队方法？

**6. 矩阵链乘**

P=a1×a2×a3×……×an，依据乘法结合律，不改变其顺序，只用括号表示成对的乘积，试问有几种括号化的方案？

**7. 二叉树计数**

有n个节点构成的二叉树（非叶子节点都有2个儿子），共有多少种情形？
有n+1个叶子的二叉树的个数？

**8. 凸多边形划分**

在一个n边形中，通过不相交于n边形内部的对角线，把n边形拆分为若干个三角形，问有多少种拆分方案？


**9. 圆上n条线段**

在圆上选择2n个点，将这些点成对连接起来使得所得到的n条线段不相交的方法数？

**10. 单调路径**

一位大城市的律师在他住所以北n个街区和以东n个街区处工作，每天他走2n个街区去上班。如果他从不穿越（但可以碰到）从家到办公室的对角线，那么有多少条可能的道路？

**11. 填充阶梯图形**

用n个长方形填充一个高度为n的阶梯状图形的方法个数**？**

**12. 摞碗问题**

饭后，姐姐洗碗，妹妹把姐姐洗过的碗一个一个放进碗橱摞成一摞。一共有n个不同的碗，洗前也是摞成一摞的，也许因为小妹贪玩而使碗拿进碗橱不及时，姐姐则把洗过的碗摞在旁边，问：小妹摞起的碗有多少种可能的方式？

**13. 汽车胡同加油问题**

一个汽车队在狭窄的路面上行驶，不得超车，但可以进入一个死胡同去加油，然后再插队行驶，共有n辆汽车，问共有多少种不同的方式使得车队开出城去？

**14. 还书借书问题**

在图书馆一共2n个人在排队，n个还《面试宝典》一书，n个在借《面试宝典》一书，图书馆此时没有了面试宝典了，求他们排队的总数

**15. 高矮排队问题**

2n个高矮不同的人,排成两排,每排必须是从矮到高排列,而且第二排比对应的第一排的人高,问排列方式有多少种?