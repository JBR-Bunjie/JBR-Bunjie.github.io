---
title: Sugiyama Algorithm
date: 2022-12-23 12:23:23
tags:
  - Auto Layout
  - Algorithm
categories:
  - Algorithm
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

# 划时代的自动布局算法：sugiyama算法

## sugiyama algorithm steps

of the framework is to divide the task of drawing a graph into several subproblems, most of which closely resemble other well known problems within computer science. That way one can use algorithms for the similar problems to solve the Sugiyama subproblems and thereby simplify the graph layout process. The different steps of the method are illustrated in figure 2. They are the following: 

- Cycle removal 

First the possibly cyclic graph must be made acyclic by removing cycles, done by reversing some edges. 

- Layer assignment 

Second, the vertices are assigned to layers and dummy vertices and dummy edges are introduced for every edge that spans over more than two layers so as to create a proper layering [2], i.e. one where every edge has its endpoints in adjacent layers. 

- Vertex ordering 

Third, the vertices are ordered within their layers to minimise edge crossings.

- Coordinate assignment 

Fourth and last, the vertices are assigned coordinates to create a balanced graph.

## sugiyama barycenter算法内容





两层间的情况：



n层间的情况：





## 纯python实现：

```python
```







原论文地址：

[Methods for Visual Understanding of Hierarchical System Structures | IEEE Journals & Magazine | IEEE Xplore](https://ieeexplore.ieee.org/document/4308636)

所实现算法的仓库：



视频教程：

[(40) Hierarchical Drawings: Sugiyama Framework | Visualization of Graphs - Lecture 8 - YouTube](https://www.youtube.com/playlist?list=PLubYOWSl9mIvxe_HwoSyT-oXgkOmB1u3V)

More Resource:

[161388.pdf (chalmers.se)](https://publications.lib.chalmers.se/records/fulltext/161388.pdf)
