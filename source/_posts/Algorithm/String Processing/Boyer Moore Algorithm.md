---
title: Boyer Moore Algorithm
date: 2022-12-23 12:23:23
tags:
  - String Processing
categories:
  - Algorithm
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

# Boyer Moore Algorithm

## General

高效，可靠，应用广泛：各种文本编辑器的"查找"功能（Ctrl+F），大多采用[Boyer-Moore算法](https://en.wikipedia.org/wiki/Boyer–Moore_string_search_algorithm)

1977年，德克萨斯大学的Robert S. Boyer教授和J Strother Moore教授发明了这种算法。

## Detail

### "坏字符规则"

> 后移位数 = 坏字符的位置 - 搜索词中的上一次出现位置

如果"坏字符"不包含在搜索词之中，则上一次出现位置为 -1。

### "好后缀规则"

> 后移位数 = 好后缀的\(最后\)位置 - 后缀\(在搜索词中\)的上一次出现位置

**注意点**：

1. "好后缀"的位置以最后一个字符为准。假定"ABCDEF"的"EF"是好后缀，则它的位置以"F"为准，即5（从0开始计算）
2. 如果"好后缀"在搜索词中只出现一次，则它的上一次出现位置为 -1。比如，"EF"在"ABCDEF"之中只出现一次，则它的上一次出现位置为-1（即未出现）
3. 如果"好后缀"有多个，则除了最长的那个"好后缀"，其他"好后缀"的上一次出现位置必须在头部。比如，假定"BABCDAB"的"好后缀"是"DAB"、"AB"、"B"，请问这时"好后缀"的上一次出现位置是什么？回答是，此时采用的好后缀是"B"，它的上一次出现位置是头部，即第0位。这个规则也可以这样表达：如果最长的那个"好后缀"只出现一次，则可以把搜索词改写成如下形式进行位置计算"(DA)BABCDAB"，即虚拟加入最前面的"DA"





## Code

```java
public static int pattern(String pattern, String target) {
    int tLen = target.length();//主串的长度
    int pLen = pattern.length();//模式串的长度

	//如果模式串比主串长，没有可比性，直接返回-1
    if (pLen > tLen) {
        return -1;
    }

    int[] bad_table = build_bad_table(pattern);// 获得坏字符数值的数组，实现看下面
    int[] good_table = build_good_table(pattern);// 获得好后缀数值的数组，实现看下面

    for (int i = pLen - 1, j; i < tLen;) {
        System.out.println("跳跃位置：" + i);
        //这里和上面实现坏字符的时候不一样的地方，我们之前提前求出坏字符以及好后缀
        //对应的数值数组，所以，我们只要在一边循环中进行比较。还要说明的一点是，这里
        //没有使用skip记录跳过的位置，直接针对主串中移动的指针i进行移动
        for (j = pLen - 1; target.charAt(i) == pattern.charAt(j); i--, j--) {
            if (j == 0) {//指向模式串的首字符，说明匹配成功，直接返回就可以了
                System.out.println("匹配成功，位置：" + i);
                //如果你还要匹配不止一个模式串，那么这里直接跳出这个循环，并且让i++
                //因为不能直接跳过整个已经匹配的字符串，这样的话可能会丢失匹配。
//					i++;   // 多次匹配
//					break;
                return i;
            }
        }
        //如果出现坏字符，那么这个时候比较坏字符以及好后缀的数组，哪个大用哪个
        i += Math.max(good_table[pLen - j - 1], bad_table[target.charAt(i)]);
    }
    return -1;
}

//字符信息表
public static int[] build_bad_table(String pattern) {
    final int table_size = 256;//上面已经解释过了，字符的种类
    int[] bad_table = new int[table_size];//创建一个数组，用来记录坏字符出现时，应该跳过的字符数
    int pLen = pattern.length();//模式串的长度

    for (int i = 0; i < bad_table.length; i++) {
        bad_table[i] = pLen;  
        //默认初始化全部为匹配字符串长度,因为当主串中的坏字符在模式串中没有出
        //现时，直接跳过整个模式串的长度就可以了
    }
    for (int i = 0; i < pLen - 1; i++) {
        int k = pattern.charAt(i);//记录下当前的字符ASCII码值
        //这里其实很值得思考一下，bad_table就不多说了，是根据字符的ASCII值存储
        //坏字符出现最右的位置，这在上面实现坏字符的时候也说过了。不过你仔细思考
        //一下，为什么这里存的坏字符数值，是最右的那个坏字符相对于模式串最后一个
        //字符的位置？为什么？首先你要理解i的含义，这个i不是在这里的i，而是在上面
        //那个pattern函数的循环的那个i，为了方便我们称呼为I，这个I很神奇，虽然I是
        //在主串上的指针，但是由于在循环中没有使用skip来记录，直接使用I随着j匹配
        //进行移动，也就意味着，在某种意义上，I也可以直接定位到模式串的相对位置，
        //理解了这一点，就好理解在本循环中，i的行为了。

		//其实仔细去想一想，我们分情况来思考，如果模式串的最
        //后一个字符，也就是匹配开始的第一个字符，出现了坏字符，那么这个时候，直
        //接移动这个数值，那么正好能让最右的那个字符正对坏字符。那么如果不是第一个
        //字符出现坏字符呢？这种情况你仔细想一想，这种情况也就意味着出现了好后缀的
        //情况，假设我们将最右的字符正对坏字符
        bad_table[k] = pLen - 1 - i;
    }
    return bad_table;
}

//匹配偏移表
public static int[] build_good_table(String pattern) {
    int pLen = pattern.length();//模式串长度
    int[] good_table = new int[pLen];//创建一个数组，存好后缀数值
    //用于记录最新前缀的相对位置，初始化为模式串长度，因为意思就是当前后缀字符串为空
    //要明白lastPrefixPosition 的含义
    int lastPrefixPosition = pLen;

    for (int i = pLen - 1; i >= 0; --i) {
        if (isPrefix(pattern, i + 1)) {
        //如果当前的位置存在前缀匹配，那么记录当前位置
            lastPrefixPosition = i + 1;
        }
        good_table[pLen - 1 - i] = lastPrefixPosition - i + pLen - 1;
    }

    for (int i = 0; i < pLen - 1; ++i) {
    //计算出指定位置匹配的后缀的字符串长度
        int slen = suffixLength(pattern, i);
        good_table[slen] = pLen - 1 - i + slen;
    }
    return good_table;
}

//前缀匹配
private static boolean isPrefix(String pattern, int p) {
    int patternLength = pattern.length();//模式串长度
    //这里j从模式串第一个字符开始，i从指定的字符位置开始，通过循环判断当前指定的位置p
    //之后的字符串是否匹配模式串前缀
    for (int i = p, j = 0; i < patternLength; ++i, ++j) {
        if (pattern.charAt(i) != pattern.charAt(j)) {
            return false;
        }
    }
    return true;
}

//后缀匹配
private static int suffixLength(String pattern, int p) {
    int pLen = pattern.length();
    int len = 0;
    for (int i = p, j = pLen - 1; i >= 0 && pattern.charAt(i) == pattern.charAt(j); i--, j--) {
        len += 1;
    }
    return len;
}
```

## Reference

- [字符串匹配的Boyer-Moore算法 - 阮一峰的网络日志 (ruanyifeng.com)](https://www.ruanyifeng.com/blog/2013/05/boyer-moore_string_search_algorithm.html)
- [不用找了，学习BM算法，这篇就够了（思路+详注代码）_BoCong-Deng的博客-CSDN博客_bm算法](https://blog.csdn.net/DBC_121/article/details/105569440)