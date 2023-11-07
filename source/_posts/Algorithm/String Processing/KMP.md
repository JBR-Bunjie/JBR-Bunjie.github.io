---
title: KMP
date: 2022-12-23 12:23:23
tags:
  - String Processing
  - Algorithm
categories:
  - Algorithm
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

# KMP算法

## KMP算法作用简介：

在计算机科学中，Knuth-Morris-Pratt字符串查找算法（简称为KMP算法）可在一个字符串S内查找一个词W的出现位置。一个词在不匹配时本身就包含足够的信息来确定下一个匹配可能的开始位置，此算法利用这一特性以避免重新检查先前配对的字符。

## KMP算法原理：

### 1.前缀与后缀

> 几个必须明确的概念：前缀、后缀、相同前缀后缀的最大长度（为表述方便，下文均用公共最大长指代）
>
> - `abcdef`的前缀：`a`、`ab`、`abc`、`abcd`、`abcde`（注意：abcdef不是前缀）
> - `abcdef`的后缀：`f`、`ef`、`def`、`cdef`、`bcdef`（注意：abcdef不是后缀）
> - `abcdef`的公共最大长：0（因为其前缀与后缀没有相同的）
> - `ababa`的前缀：`a`、`ab`、`aba`、`abab`
> - `ababa`的后缀：`a`、`ba`、`aba`、`baba`
> - `ababa`的公共最大长：3（因为他们的公共前缀后缀中最长的为aba，长度3）

### 2.确定PMT (Partial Match Table) 

> "部分匹配值"就是"前缀"和"后缀"的最长的共有元素的长度。以"ABCDABD"为例，
>
> - "A"的前缀和后缀都为空集，共有元素的长度为0；其value为
> - "AB"的前缀为[A]，后缀为[B]，共有元素的长度为0；　　
> - "ABC"的前缀为[A, AB]，后缀为[BC, C]，共有元素的长度0；　　
> - "ABCD"的前缀为[A, AB, ABC]，后缀为[BCD, CD, D]，共有元素的长度为0；　　
> - "ABCDA"的前缀为[A, AB, ABC, ABCD]，后缀为[BCDA, CDA, DA, A]，共有元素为"A"，长度为1；　　
> - "ABCDAB"的前缀为[A, AB, ABC, ABCD, ABCDA]，后缀为[BCDAB, CDAB, DAB, AB, B]，共有元素为"AB"，长度为2；　　
> - "ABCDABD"的前缀为[A, AB, ABC, ABCD, ABCDA, ABCDAB]，后缀为[BCDABD, CDABD, DABD, ABD, BD, D]，共有元素的长度为0。

得表：

![img](../../../images\Algorithm\bg2013050109.png)

## KMP算法的Python实现：

```python
class Solution:
    def strStr(self, haystack: str, needle: str) -> int:
        def KMP(s, p):
            """
            s 为主串
            p 为模式串
            如果 t 里有 p，返回打头下标
            """
            nex = getNext(p)
            i = 0
            j = 0   # 分别是 s 和 p 的指针
            while i < len(s) and j < len(p):
                if j == -1 or s[i] == p[j]: # j == -1 是由于 j = next[j]产生
                    i += 1
                    j += 1
                else:
                    j = nex[j]

            if j == len(p): # j 走到了末尾，说明匹配到了
                return i - j
            else:
                return -1

        def getNext(p):
            """
            p 为模式串
            返回 next 数组，即部分匹配表
            """
            nex = [0] * (len(p) + 1)
            nex[0] = -1
            i = 0
            j = -1
            while i < len(p):
                if j == -1 or p[i] == p[j]:
                    i += 1
                    j += 1
                    nex[i] = j     # 这是最大的不同：记录next[i]
                else:
                    j = nex[j]

            return nex
        
        return KMP(haystack, needle)
```



## Reference

- [字符串匹配的KMP算法 - 阮一峰的网络日志 (ruanyifeng.com)](https://www.ruanyifeng.com/blog/2013/05/Knuth–Morris–Pratt_algorithm.html)
- 如何更好地理解和掌握 KMP 算法? - 白鱼咸蛋笨大的回答 - 知乎 https://www.zhihu.com/question/21923021/answer/642165149
- 如何更好地理解和掌握 KMP 算法? - 海纳的回答 - 知乎 https://www.zhihu.com/question/21923021/answer/281346746 ~(第一个示例比较特殊)~

