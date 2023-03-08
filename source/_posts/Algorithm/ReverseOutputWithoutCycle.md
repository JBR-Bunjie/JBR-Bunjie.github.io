---
title: 利用递归、无循环地打印数组
date: 2022-12-23 12:23:23
tags:
  - Recursion
categories:
  - Algorithm
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

# ReverseOutputWithoutCycle

描述：不用循环，不逐一赋值地把一个数组逆序输出

循环 -> 递归；即用递归去承担原本循环的工作

即：

```C#
void Print(int[] arr, int len) {
    if (len > 0) {
        Console.printline(arr[len-1] + "\n");
        Print(arr, len - 1);
    }
}
```



