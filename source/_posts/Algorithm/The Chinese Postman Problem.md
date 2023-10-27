---
title: 中国邮递员问题
date: 2022-12-23 12:23:23
tags:
  - Shortest Route
  - Algorithm
categories:
  - Algorithm
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

# The Route of the Postman

## Reference

- [The Chinese-Postman-Method (tum.de)](https://algorithms.discrete.ma.tum.de/graph-algorithms/directed-chinese-postman/index_en.html)
- [邮递员问题 - 维基百科，自由的百科全书 (wikipedia.org)](https://zh.wikipedia.org/wiki/邮递员问题)
- [中国邮递员问题 | Junnor.G (cfonheart.github.io)](https://cfonheart.github.io/2018/04/19/中国邮递员问题/)

## Details

>**邮递员问题**（也称**邮路问题**，Route Inspection Problem，或**中国邮路问题**,China Route Inspection Problem，或**中国邮递员问题**Chinese Postman Problem）是一个[图论](https://zh.wikipedia.org/wiki/图论)问题。此问题为在一个连通的无向图中找到一最短的封闭路径，且此路径需通过所有边至少一次。
>
>简单来说，邮递员问题就是在一个已知的地区，[邮差](https://zh.wikipedia.org/wiki/郵差)要设法找到一条最短路径，可以走过此地区所有的街道，且最后要回到出发点。

[一笔画问题 - 维基百科，自由的百科全书 (wikipedia.org)](https://zh.wikipedia.org/wiki/一笔画问题)

###  问题解决

>1. 图本身就是一个欧拉回路图，那么直接走一个欧拉回路就访问了所有路径并回到了起点
>
>   （欧拉回路图的前提条件是每个点的度数都是偶数）
>
>   
>
>2. 图不是一个欧拉回路图，需要对某些边重复走多次回到起点，***等价于添加了一些已经存在的重边***，构建了欧拉回路。
>
>   不是欧拉回路需要通过增加一些边使得图变成一个欧拉回路，并能保证问题的最优解，添加的边权值和一定是尽可能短的。
>
>   - 计算度数为奇数的点
>   - 算出所有点之间的最短路径
>   - 奇度数的点一定为偶数，点与点之间构建二分图，权值为两点之间的最短路，找到一个最小权值匹配集
>   - 对于匹配集，找到每一个匹配集两点之间形成的最短路径，那么这条最短路径就需要加入到额外的边中，且一定能保证这条路径上除了这两个匹配的奇度数点度数加了1变成了偶数，其他所有中间点都是加了2，不影奇偶性。
>   - 所有边都添加好后，就是一个欧拉回路了，计算从起点开始的欧拉回路路径
