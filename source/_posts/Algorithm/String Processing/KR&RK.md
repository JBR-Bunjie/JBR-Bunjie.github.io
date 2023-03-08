---
title: R&K Algorithm
date: 2022-12-23 12:23:23
tags:
  - String Processing
categories:
  - Algorithm
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

# Rabin–Karp algorithm - 拉宾-卡普算法

~~拉宾-卡普算法（英語：Rabin–Karp algorithm）或卡普-拉宾算法（Karp–Rabin algorithm）~~

## General

[Rabin-Karp 算法](https://en.wikipedia.org/wiki/Rabin–Karp_algorithm)是由 [Richard M. Karp](https://en.wikipedia.org/wiki/Richard_M._Karp)和 [Michael O. Rabin](https://en.wikipedia.org/wiki/Michael_O._Rabin)创建的字符串搜索算法

KR Algorithm使用散列来查找文本中的一组模式字符串中的任何一个。

## Detail

假设我们有一个文本： **yeminsajid** ，我们想知道文本中是否存在模式 **nsa** 。要计算散列和滚动散列，我们需要使用素数。这可以是任何素数。让我们在这个例子中使用 **prime** = **11** 。我们将使用以下公式确定哈希值：

```placeholder
(1st letter) X (prime) + (2nd letter) X (prime)¹ + (3rd letter) X (prime)² X + ......
```

我们将表示：

```placeholder
a -> 1    g -> 7    m -> 13   s -> 19   y -> 25
b -> 2    h -> 8    n -> 14   t -> 20   z -> 26
c -> 3    i -> 9    o -> 15   u -> 21
d -> 4    j -> 10   p -> 16   v -> 22
e -> 5    k -> 11   q -> 17   w -> 23
f -> 6    l -> 12   r -> 18   x -> 24
```

**nsa** 的哈希值为 ：
$$
HASH(nsa) = 14 * 11^0 + 19 * 11^1 + 1 * 11^2 = 344
$$


现在我们找到文本的滚动哈希值。如果滚动哈希与模式的哈希值匹配，我们将检查字符串是否匹配。因为我们的模式有 **3 个**字母，我们将采取 1 日 **3** 封 **YEM** 从我们的文本，并计算哈希值。我们得到：
$$
HASH(yem) = 25 * 11^0 + 5 * 11^1 + 13 * 11^2 = 1653
$$
此值与我们的模式的哈希值不匹配。所以字符串在这里不存在。现在我们需要考虑下一步。计算下一个字符串 **emi** 的哈希值。我们当然可以直接重新从。但这将是相当微不足道的，并且会花费更多。相反，我们使用另一种技术。

- 我们从当前哈希值中减去 **Previous String** 中的 **第一个字母的** 即 **y** 的值。即：`1653 - 25 = 1628`。
- 再次除以设定的**素数 prime——11**，可得：1628 / 11 = 148`。
- 最后加上**新的字母 \*（素数）\^ m - 1**，可得：`148 + 9 X 11² = 1237`。

新的哈希值不等于我们的模式哈希值。继续前进直到 **a** \(nsa\)，有：

```placeholder
Previous String: ins
First Letter of Previous String: i(9)
New Letter: a(1)
New String: "nsa"
------------------------------
2462 - 9 = 2453
2453 / 11 = 223
223 + 1 X 11² = 344
```

这是一个匹配！ 现由于两个字符串都匹配，因此匹配字符串存在于目标字符串中，现在我们只需要返回目标字符串匹配字段的起始位置即可。

## Code

### KR - JS

```js
/**
 * @param {string} haystack
 * @param {string} needle
 * @return {number}
 */
/*

*/
/**
 * @param {string} haystack
 * @param {string} needle
 * @return {number}
 */
var strStr = function(haystack, needle){
    if(needle.length == 0) return 0;
    let n = haystack.length;
    let m = needle.length;
    let s = ' '+haystack;
    let t = ' '+needle;
    
    //String添加一个重新确定字符值得方法
    String.prototype.newCode = function(){
        return this.charCodeAt()-97+1;
    }
    //确定模数 尽量避免冲突
    let p = 999991, d = 131;
    //计算thash
    let tHash = 0;
    for(let i=1; i<=m; i++){
        tHash = (tHash*d + t[i].newCode()) % p;
    }
    //计算sHash子串的值
    let sHash = new Array(n+1);
    sHash[0] = 0;
    for(let i=1; i<=n; i++){
        sHash[i] = (sHash[i-1]*d + s[i].newCode()) % p;
    }
   

    //hello ll
    for(let i=m; i<=n; i++){
        if(calcHash(i-m, i, m) == tHash){
            return i-m;
        }
    }
    return -1;
    function calcHash(l, r, len){
        //_hello  求ll的hash
        //先求_hell的hash, 再求_he的hash进行补位
        //_hell - _he00 就是ll
        //注意相减的时候为避免负数，可以先加上模数。再取模
        return (sHash[r] - (sHash[l]*myPow(d, len, p))%p + p) % p;
    }
}
function myPow(x,n, m){
    function helper(x,n){
        if(n==0) return 1.0;
        let y = helper(x, n/2 | 0);
        return n%2 ==0 ? (y*y)%m: (y*y*x)%m;
    }
    return n>=0? helper(x, n) : 1.0/helper(x, n);
}
```



## Reference

- [Rabin-Karp 算法简介 | 他山教程，只选择最优质的自学材料 (tastones.com)](http://www.tastones.com/stackoverflow/algorithm/substring-search/introduction_to_rabin-karp_algorithm/)
- [28. 实现 strStr() 题解 - 力扣（LeetCode） (leetcode-cn.com)](https://leetcode-cn.com/problems/implement-strstr/solution/28-shi-xian-strstr-rabin-karp-by-jingyua-w3h1/)