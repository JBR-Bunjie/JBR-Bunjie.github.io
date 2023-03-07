### 变量的定义

var//ES5及以前的标准，一切都通过var来定义
//目前，仍然可以无脑用var
//var是variant的缩写
let//ES6的新标准，定义一个变量，就用 let
const//ES6的新标准，定义一个常量，就用 const

### 变量的赋值:

//变量的声明和赋值，写在一起
var a = 100; // ES5语法

const b = hello; // ES6 语法
let b = world; // ES6 语法

3.声明一个变量并赋值，我们称之为变量的初始化。
推荐初始化

### 声明多个变量

```js
var name = '千古壹号', 
    age = 27, 
    number = 100;
```



### 变量声明的几种特殊情况

**写法1**、先声明，再赋值：（正常）
但是请务必使用且仅使用写法一!!!!!!!!

**写法2**、不声明，只赋值：（正常）
**写法3**、只声明，不赋值：（注意，打印 undefined）
**写法4**、不声明，不赋值，直接使用：（会报错）

### 变量的命名规范

​	1).大小写敏感
​	2).变量名长度不能超过255个字符。
​	3).汉语可以作为变量名。但是不建议使用
​	4).不能以数字开头
​	5).只能由字母(A-Z、a-z)、数字(0-9)、下划线(_)、美元符( $ )组成
​	6).建议用驼峰命名规则

### 标识符、关键字、保留字

**标识符**：在JS中所有的可以由我们**自主命名**的都可以称之为标识符。
**关键字**：是指 JS 本身已经使用了的单词，我们不能再用它们充当变量、函数名等标识符。
**保留字**：实际上就是预留的“关键字”。意思是现在虽然还不是关键字，但是未来可能会成\
为关键字，同样不能使用它们当充当变量名、函数名等标识符。

JS 中的关键字如下：

>break、continue、case、default、if、else、switch、for、in、do、while、try、catch、finally、throw、var、void、function、return、new、this、typeof、instanceof、delete、with、true、false、null、undefined
>

### 变量的数据类型

1.基本数据类型（值类型）
String 字符串、
Number 数值、
Boolean 布尔值、
Null 空值、
Undefined 未定义

2.引用数据类型（引用类型）
Object 对象

3.变量的数据类型转换
显示类型转换
toString()
String()
Number()
parseInt(string)
parseFloat(string)
Boolean()
prompt()//`prompt()`就是专门用来弹出能够让用户输入的对话框。重要的是：用户不管输入什么，都当字符串处理。

Number() 函数

