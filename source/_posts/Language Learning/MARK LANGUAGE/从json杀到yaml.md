---
title: 从Json杀到Yaml
date: 2021-11-16 16:15:04
tags:
  - Json
  - Yaml
  - Language Learning
categories:
  - Language Learning
  - Json
  - Yaml
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

# 从 Json 杀到 Yaml

## Json 介绍

### 什么是<a href="#Json">Json</a>？

> **JSON** (JavaScript Object Notation) is a lightweight data-interchange format. It is easy for humans to read and write. It is easy for machines to parse and generate. It is based on a subset of the JavaScript Programming Language Standard ECMA-262 3rd Edition - December 1999. JSON is a text format that is completely language independent but uses conventions that are familiar to programmers of the C-family of languages, including C, C++, C#, Java, JavaScript, Perl, Python, and many others. These properties make JSON an ideal data-interchange language.
>
> > Json 是一种轻量级数据交换格式。它有着可读性高，易于机器解析和生成的优点，它是编程语言 Js 第三版 ECMA-262 规范的一个子集。JSON 是一种完全独立于语言的文本格式，但是却使用了 C 系程序员熟悉的相关约定如 C, C++, C#, Java, JavaScript, Perl, Python, and many others。这些特性使 JSON 成为一种理想的数据交换语言。

### Json 语法

Json is built on two structures:

- A collection of name/value pairs. In various languages, this is realized as an _object_, record, struct, dictionary, hash table, keyed list, or associative array.
- An ordered list of values. In most languages, this is realized as an _array_, vector, list, or sequence.

These are universal data structures. Virtually all modern programming languages support them in one form or another. It makes sense that a data format that is interchangeable with programming languages also be based on these structures.

这些是通用的数据结构。几乎所有的现代编程语言都以这样或那样的形式支持它们。所以一个可以融入编程语言的数据格式自然也应当基于这些结构来设计。

```json
{
    "object1": [
    { "key1.1":"value1.1" , "key1.2":"value1.2" },
    { "key2.1":"value2.1" , "key2.2":"value2.2" },
    ...
    ],
    "object2":"valueForObject2",
    "object3": ...
}
```

Json 整体是一个对象 Object，其中内容是**无序**的 name/value 键值对集合，name 以 string 构建，value 可以以`array`, `string`, `number`, `object`, `true`, `false`和`null`中的任意一种来构建。array 中的元素一般为 Object 的对象，value 又可以直接调用 Object，如此便形成嵌套。name 与 value 以 `:`分隔。

当我们引用 Json 数据时，我们对 Object 中的键值使用点计法取名引用，对数组数据用一般引用数组的办法 Array[ n ]来引用

:::warning

原生 Json 并不支持注释，如果一定需要使用注释，可以查阅<a href="#JsonSchema">Json schema</a>或者换用<a href="#Json5">JSON5</a>，虽然 JSON5 是第三方库，但仍然严格符合 JS 标准，并且 JSON5.parse 与 JSON.parse 的解析结果一致。当然，你也可以直接换用换用下面的 Yaml

:::

## Yaml 介绍

> <a href="#Yaml">YAML</a> Ain't Markup Language, YAML is a human friendly data serialization language for all programming languages.

有意思的是，虽然 YAML 是 YAML Ain't Markup Language 的缩写，但是这个缩写中的 YAML 其实是 Yet Another Markup Language 的缩写，即：这是一个标记语言。正式名称改为了“不是标记语言”，这一点颇为有趣，也耐人寻味——强调其以数据作为重心，而非标记语言。

截止 2021.9.27，Yaml 的最新版本为 1.2.1(发布于 2009.10.1)

到 2021.10.1，Yaml 发布了最新版：1.2.2

> We are excited to announce the release of [Revision 1.2.2 of the YAML 1.2 Specification](https://yaml.org/spec/1.2.2/). This revision comes 12 years to the day after the [previous revision](https://yaml.org/spec/1.2.1/).

如果你没有耐心啃最新的文档的话，你只需要看一句话：No normative changes from the 1.2.1 revision. YAML 1.2 has not been changed. 就行

### yaml 优点

> The design goals for YAML are, in decreasing priority:
>
> - YAML is easily readable by humans.（易读
> - YAML data is portable between programming languages.（跨语言
> - YAML matches the native data structures of agile languages.（与当前语言的数据结构兼容
> - YAML has a consistent model to support generic tools.（支持通用工具
> - YAML supports one-pass processing.
> - YAML is expressive and extensible.
> - YAML is easy to implement and use.

- YAML 中没有额外的定界符，所以相比 JSON 更轻量级。
- **没有额外定界符**，所以更易读（不过这一点颇为激进，时至今日仍有很多开发者无法接受这种以缩进来界定结构的做法，因为经常出现“一个空格引发的悲剧”）
- YAML 使数据更易于理解，因此常用于配置文件中（很经常看见别人将.properties 改为.yaml）
- YAML 是 JSON 的超集，对于合法的 JSON 代码，同样可以被 YAML 解析，这样对于使用 JSON 和 YAML 的应用来说，可以使用一个解析器完成两种解析。
  然而其并没有如期望中那样受欢迎，具体而言，因为不同的序列化语言都有其特定的适宜语言或者场景（下文可以提到），并且相较于其他广泛使用的序列化语言，YAML 有一些不足。

### YAML 使用场景

对于序列化语言来说，使用场景如下：

- 与服务器之间传输数据
- 使用一个配置文件来配置应用，这些文件声明对应参数和相应取值
- 在同一个应用不同组件之间转换数据
- 中间数据存储：针对此类场景，YAML 有一些明确的优势相比于其他同类语言。也是为什么现在越来越多的开发者使用其的地方。

### YAML 语法：

Yaml 与 Json 一样同时支持对象和数组，但是 Yaml 将其余的所有值全部归为“纯量”

```yaml
# yaml代码示例：
languages:
  - Ruby
  - Perl
  - Python
  # 以 - 开头的行表示构成一个数组
websites:
  YAML: yaml.org
  Ruby: ruby-lang.org
  Python: python.org
  Perl: use.perl.org
  # 对象键值对使用冒号结构表示 key: value，冒号后面要加一个空格。

companies:
  - id: 1
    name: company1
    price: 200W
  - id: 2
    name: company2
    price: 500W
    # 意思是 companies 属性是一个数组，每一个数组元素又是由 id、name、price 三个属性构成。
```

转换为 json 为：

```json
{
  languages: ['Ruby', 'Perl', 'Python'],
  websites: {
    YAML: 'yaml.org',
    Ruby: 'ruby-lang.org',
    Python: 'python.org',
    Perl: 'use.perl.org'
  companies: [{
      	id: 1,
      	name: company1,
      	price: 200W
  	  }, {
    	id: 2,
    	name: company2,
    	price: 500W
	}]
  }
}
```

### yaml 相比 json？

> official document:
>
> Both JSON and YAML aim to be human readable data interchange formats. However, JSON and YAML have different priorities. JSON’s foremost design goal is simplicity and universality. Thus, JSON is trivial to generate and parse, at the cost of reduced human readability. It also uses a lowest common denominator information model, ensuring any JSON data can be easily processed by every modern programming environment.
>
> > Json 和 Yaml 的目标都是人类可读的数据交换格式。但是，Json 和 Yaml 对于目标有不同的优先级。Json 格式设计的首要目标是 simplicity 和 universality。因此 Json 容易生成和解析，代价就是减少可读性。
>
> In contrast, YAML’s foremost design goals are human readability and support for [serializing](https://yaml.org/spec/1.2.1/#serialize//) arbitrary [native data structures](https://yaml.org/spec/1.2.1/#native data structure//). Thus, YAML allows for extremely readable files, but is more complex to generate and parse. In addition, YAML ventures beyond the lowest common denominator data types, requiring more complex processing when crossing between different programming environments.
>
> YAML 中有很多方式来体系化数据层级，因此处理时会相对复杂些。性能上相对于 XML 和 JSON 会有差别。

### yaml 与 yml 区别?

.yaml 与.yml 都是 yaml 格式的拓展名，虽然.yaml 才是官方推荐的拓展格式，但是到目前为止，yml 仍然被广泛使用：GitHub 上 .yml 相关总提交次数为 1 千 3 百万次，而.yaml 总次数仅为 5 百万次

贴一个比较古早的 stackoverflow 的问题链接：[configuration files - Is it .yaml or .yml? - Stack Overflow](https://stackoverflow.com/questions/21059124/is-it-yaml-or-yml)

如果你不太想看的话，以下由简略版本：

> 正方：I like .yml, it's 25% faster to write. ☝😆. Personally, I'm fine with whichever is the "standard"
>
> 反方：Some people like to stick to 3 letter extensions as it was back in the old days when file systems were still limited to short names

但是我仍然推荐使用 yaml，作为官方推荐的拓展格式，它也许可以帮你避免很多意料之外的 error。

[关于配置文件：是.yaml 还是.yml？ | 码农家园 (codenong.com)](https://www.codenong.com/21059124/)

本来找到了原文，但是链接丢了(((φ(◎ ロ ◎;)φ)))，坏欸

---

### 参考资料：

- <span id="Json">[JSON](https://www.json.org/json-en.html)</span>
- <span id="JsonSchema">[JSON Schema | The home of JSON Schema (json-schema.org)](http://json-schema.org/) </span>
- <span id="Json5">[JSON5 | JSON for Humans](https://json5.org/)</span>
- <span id="Json5Document">[The JSON5 Data Interchange Format](https://spec.json5.org/#summary-of-features)</span>
- <span id="Yaml">[The Official YAML Web Site](https://yaml.org/)</span>
- <span id="YamlFormat">[YAML File Format](https://docs.fileformat.com/programming/yaml/)</span>
- [YAML Documentation · YAML](https://yaml.com/doc/)
- <span id="YamlValidator">[YAMLlint - The YAML Validator](http://www.yamllint.com/)</span>
- <span id="YamlGithubPage">[yaml/yaml: YAML language and community information (github.com)](https://github.com/yaml/yaml)</span>

<iframe src="//player.bilibili.com/player.html?aid=802913184&bvid=BV1dy4y1s7rA&cid=332998437&page=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true" width="100%" height="500" scrolling="no" frameborder="0" sandbox="allow-top-navigation allow-same-origin allow-forms allow-scripts"> </iframe>
