---
title: URL Explanations
date: 2022-12-23 12:23:23
tags:
  - Computer Network
categories:
  - Protocol Theory
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

## General: What is URI? and URL?

[http - What is the difference between a URI, a URL and a URN? - Stack Overflow](https://stackoverflow.com/questions/176264/what-is-the-difference-between-a-uri-a-url-and-a-urn)

>最简单易懂的：
>
>
>URL - http://example.com/some/page.html
>
>URI - /some/page.html

[HTTP 协议中 URI 和 URL 有什么区别？ - 知乎 (zhihu.com)](https://www.zhihu.com/question/21950864)

URL：(Uniform Resource Locator 的缩写，统一资源定位符)

URI：(Uniform Resource Identifier 的缩写，统一资源标识符)

URN：(Uniform Resource Name 的缩写，统一资源标识符)



其实名字就已经非常明显了：

Identifier: 重点是表示特定的资源。

Locator: 通过位置来表示资源。

Name: 通过名字来表示资源。



URI 在于I(Identifier)，是统一资源标示符，可以唯一标识一个资源。

URL在于L(Locater)，一般来说（URL）统一资源定位符，可以提供找到该资源的路径，比如

https://github.com/JBR-Bunjie/JBR-Bunjie/blob/main/back.jpg

但URL又是URI，因为它可以标识一个资源，所以URL又是URI的子集—>URI 属于 URL 更高层次的抽象，一种字符串文本标准。



## encoding and decoding in URL

URL编码通常也被称为百分号编码（percent-encoding），是因为它的编码方式非常简单：

使用%加上两位的字符——0123456789ABCDEF——代表一个字节的十六进制形式。

> [encodeURI() - JavaScript | MDN (mozilla.org)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/encodeURI)
>
> [decodeURI() - JavaScript | MDN (mozilla.org)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/decodeURI)

URL编码要做的，就是将每一个非安全的ASCII字符都被替换为“%xx”格式，而对于非ASCII字符，**RFC文档**建议使用utf-8对其进行编码得到相应的字节，然后对每个字节执行百分号编码。

如 `中文` 使用 `UTF-8` 字符集得到的字节为 `0xE4 0xB8 0xAD 0xE6 0x96 0x87`，经过Url编码之后得到 `%E4%B8%AD%E6%96%87`。

一些常见的特殊字符换成相应的十六进制的值：

```bash
+   %20   
/   %2F   
?   %3F   
%   %25   
#   %23   
&   %26  
```

用python测试编码：

```python
> "中文".encode(encoding="utf8")
b'\xe4\xb8\xad\xe6\x96\x87'

> b"\x2f".decode()
'/'
```

[URL的编码和解码 - 何必等明天 - 博客园 (cnblogs.com)](https://www.cnblogs.com/xzwblog/p/6932870.html)

Every char in the address bar will be encoded in ansi or utf8.

```js
encodeURI('https://mzh.moegirl.org.cn/乐正绫')
decodeURI("https://mzh.moegirl.org.cn/%E4%B9%90%E6%AD%A3%E7%BB%AB")
```

```python
imp
urllib.parse.unquote(a)
```

