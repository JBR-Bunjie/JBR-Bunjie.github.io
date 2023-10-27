---
title: Regex
date: 2023-3-8 10:22:54
tags:
  - String Processing
  - Regex
  - Algorithm
categories:
  - Algorithm
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

# Regex

## 正则运用

当你使用多个捕获组时，请务必小心NULL！这是某个捕获组未能捕获任何内容的结果！



验证：是否有bug

[在线正则表达式测试 中文 (oschina.net)](https://tool.oschina.net/regex/)

[在线正则验证 英文 regex101: build, test, and debug regex](https://regex101.com/)

[在线验证正则表达式结构：Regexper](https://regexper.com/)

[regex_match - C++ Reference (cplusplus.com)](https://www.cplusplus.com/reference/regex/regex_match/)

[第 6 章 正则表达式 现代 C++ 教程: 高速上手 C++ 11/14/17/20 - Modern C++ Tutorial: C++ 11/14/17/20 On the Fly (changkun.de)](https://changkun.de/modern-cpp/zh-cn/06-regex/index.html#6-2-std-regex-及其相关)

[c++ 正则表达式 高性能 - Bing](https://www.bing.com/search?q=c%2b%2b+正则表达式+高性能&qs=SC&pq=c%2b%2b正则表达式+&sc=3-9&cvid=03D982A0A5084F17BB42132E2F94B06F&FORM=QBRE&sp=1)

[c++ locale - Bing](https://www.bing.com/search?q=c%2B%2B+locale&qs=n&form=QBRE&sp=-1&pq=c%2Blocale&sc=0-8&sk=&cvid=913DBECC60374D3F965B8F9271271682)

```regex
[‐\-a-zA-Z0-9\.]+
```

## 正则提速

1. 让匹配更快失败，尤其是匹配很长的字符串时，匹配失败的位置要比成功的位置多得多。

2. 以简单、必须的字元开始，排除明显不匹配的位置，如锚点(^或$)，特殊字符(x或\u263A)字符类([a-z]或\d之类的速记符)，和单词边界(\b)；尽量避免使用分组、选择、重复量词开头，如/one|two/、\s、\s{1,}等。
3. 使用量词模式时，尽量让重复部分具体化，让字元互斥，如用”[^"\r\n]*”代替”.*?”（这个依赖回溯）。

4. 减少分支数量、缩小分支范围，用字符集和选项组件来减少分支的出现，或把分支在正则上出现的位置推后，把分支中最常出现的情况放在分支的最前面。

   ```regex
   cat|bat -> [cb]at;red|read -> rea?d;red|raw -> r(?:ed|aw); 
   
   (.|\r|\n) -> [\s\S]
   ```

5. 精确匹配需要的文本以减少后续的处理，如果需要引用匹配的一部分，可使用捕获，然后通过反向引用来处理。
6. 暴露必需的字元，用`/^(ab|cd)/`而不是`/(^ab|^cd)/`。
7. 使用合适的量词，基于预期的回溯数量，使用合适的量词类型。
8. 把正则表达式赋值给变量以便复用和提升提升性能，这样可以让正则减少不必要的编译过程。while (/regex1/.test(str1)) {/regex2/.exec(str2);…}用下面的代替上面的   var regex1 = /regex1/,regex2 = /regex2/;while (regex1.test(str1)) {regex2.exec(str2);…}
9. 将复杂的正则表达式拆分成简单的片段，每个正则只在上一个成功的匹配中查找，更高效，而且可以减少回溯。
10. 使用非捕获组，因为捕获组需要消耗时间和内存来记录反向引用，并不断更新，如果不需要反向引用，可用非捕获组(?:…)代替捕获组(…)；当需要全文匹配的反向引用时，可用regex.exec()返回的结果或者在替换字符串是使用$&。   此优化在firefox中效果较小，但其他浏览器中处理长字符串时有较大影响



And More？

[觉得正则表达式太慢？这里有一个提速100倍的方案！ - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/47401769)

项目地址

[vi3k6i5/flashtext: Extract Keywords from sentence or Replace keywords in sentences. (github.com)](https://github.com/vi3k6i5/flashtext)