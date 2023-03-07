切片：SpringAOP

请不要使用小写加下划线来命名你的变量，使用驼峰规则来命名

将判断用的等式改为调用.equals()，并且在采用.equals()前使用常量，括号里装填变量即：

```Java
"abc".equals(var) ;
// 或者使用 .equals("abc", var);
```

的调用方式，来避免当var.equals("const")如var == null时可能会出现的error

防御性编程

尽量减少代码块的嵌套，把你的代码改简洁，把它们拉平直

注意代码的简洁度，一个function尽力做到在30行内实现你的需求或者完成一个特定的任务

将Mapper的操作从循环写入改为批量写入，以此来

