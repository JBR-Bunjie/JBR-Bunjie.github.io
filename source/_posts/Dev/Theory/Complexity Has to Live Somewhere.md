---
title: Complexity Has to Live Somewhere
date: 2022-7-3 12:23:23
tags:
  - Theroy
categories:
  - Theroy
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

# Complexity Has to Live Somewhere

Original Post: [Complexity Has to Live Somewhere (ferd.ca)](https://ferd.ca/complexity-has-to-live-somewhere.html)

Chinese Version:

[Complexity Has to Live Somewhere - Google 文档](https://docs.google.com/document/d/1Qp2foEIk2Tn0x-hwM3-3FeFf3sADnhMabQGOkgE09QI/edit)

[架构设计-复杂度是不灭的 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/410049005)

Some Related Posts: [复杂度是不灭的，只会转移，难道一切都是徒劳的吗？ - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/138145081)

> Complexity has to live somewhere. If you embrace it, give it the place it deserves, design your system and organisation knowing it exists, and focus on adapting, it might just become a strength.

> 很多架构/系统一开始是简单的，这一点都没错，因为他们开始只处理简单问题，只处理几个点，这是正确的。随着系统的不断升级迭代，他们开始把复杂的事情往简单里入侵，于是系统边界开始变得模糊不清，最后崩塌。
>
> 千里之堤毁于蚁穴。

> - if you make the build tool simple, it won't handle all the weird edge cases that exist out there
> - if you want to handle the weird edge cases, you need to deviate from whatever norm you wanted to establish
> - if you want ease of use for common defaults, the rules for common defaults must be shared between the tool and the users, who shape their systems to fit the tool's expectations
> - if you allow configuration or scripting, you give the users a way to specify the rules that must be shared, so the tool fits their systems
> - if you want to keep the tool simple, you have to force your users to only play within the parameters that fit this simplicity
> - if your users' use cases don't map well to your simplicity, they will build shims around your tool to attain their objectives

所以怎么解决复杂？要把复杂交给谁？

是希望用户自定义？还是一站式打包，全部总揽？是只针对小问题破局？还是囊括一切？

层层交叉，复杂自然分散。但是这些分散的复杂是否都经过了稳妥的处理？它们是不是被随意地丢弃在各个角落？

最后：复杂度不会解决，但你所能做的一切都可以促成一个更完善的垃圾堆放处的实现
