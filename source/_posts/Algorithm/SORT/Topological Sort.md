---
title: 拓扑排序
date: 2022-12-23 12:23:23
tags:
  - Sort
categories:
  - Algorithm
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

## 拓扑排序

### Reference：

[拓扑排序（Topological Sorting） | 神奕的博客 (songlee24.github.io)](http://songlee24.github.io/2015/05/07/topological-sorting/)

### Detail：

> 在图论中，**拓扑排序（Topological Sorting）**是一个**有向无环图（DAG, Directed Acyclic Graph）**的所有顶点的线性序列。且该序列必须满足下面两个条件：
>
> 1. 每个顶点出现且只出现一次。
> 2. 若存在一条从顶点 A 到顶点 B 的路径，那么在序列中顶点 A 出现在顶点 B 的前面。
>
> 有向无环图（DAG）才有拓扑排序，非DAG图没有拓扑排序一说。

在sugiyama算法的“分层”步骤中，我们相当于直接用到了这种思路

唯一的不同是：

拓扑排序根据顺序剔除边后入度为零的全部节点生成了一个序列

而sugiyama将当前所有为零的节点并入独立的一整层

### 用途？

>拓扑排序通常用来“排序”具有依赖关系的任务。
>
>比如，如果用一个DAG图来表示一个工程，其中每个顶点表示工程中的一个任务，用有向边 表示在做任务 B 之前必须先完成任务 A。故在这个工程中，任意两个任务要么具有确定的先后关系，要么是没有关系，绝对不存在互相矛盾的关系（即环路）。