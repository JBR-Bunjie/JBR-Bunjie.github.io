

```java
module 本模块的名称{
	exports 对外暴露的包路径;
	requires 需要依赖的其他模块名称;
}
```





## 一、什么是Java module？

与Java 中的package有些类似，module引入了Java代码分组的另一个级别。每个这样的分组（module）都包含许多子package包。通过在一个模块的源代码文件package的根部，添加文件module-info.java来声明该文件夹及其子文件夹为一个模块。该文件语法如下：

```java
 module xxx.yyy{
  	....
 }
```

其中xxx.yyy是模块module声明的名称，不是package名称。

## 二、模块导出package

文件module-info.java可以指定该模块下面的哪些package对外可见、可访问。通过一个新的关键字`exports`来实现该功能。

```java
 module xxx.yyy{
 	exports com.zimug.java9;
 }
```

`com.zimug.java9`代表一个package。

> 需要注意的是：即使给定package包中的类是public的，如果未通过'exports'显式导出其程序包，则它们在模块外部也是不可见的（在编译时和运行时都是如此）。

## 三、模块导入package

如果另一个模块想要使用被导出的package包中的类，可以用`requires`关键字在其module-info.java文件中来导入（读取）目标模块的package包。

```java
module def.stu{
	requires xxx.yyy;
}
```

## 四、Java module的意义

在笔者看来，Java 9引入module 模块化管理系统，更多的是从安全性的角度考虑。Java 代码中90%以上的漏洞都是由反射和访问权限控制粒度不足引起的，Java 9的模块化系统正好能解决这个问题。Java 9 module提供另一个级别的Java 代码可见性、可访问性的控制。

比如说：我们都知道当一个class被修饰为private的时候，意味着这个类是内部类。对于顶级类(外部类)来说，只有两种修饰符：public和默认(default)。这也就意味着一个问题，有些public class我们本来是打算在jar包定义的范围内使用的，但是结果却是任何引入了这个jar的项目都可以使用这个jar里面所有的public class代码。

也就是我们的原意是在有限范围内提供公开访问，结果却是无限制的对外公开。在引入Java 9模块化之后，可以实现**有限范围内的代码public访问权限**，将代码公开区分为：**模块外部有限范围的公开访问**和**模块内部的公开访问**。

## 五、实例

在此示例中，我将创建两个模块“ common.widget”和“ data.widget”，并将它们放置在单个文件夹“ modules-examples/src”下。文件“ module-info.java”将放置在每个模块的根文件夹下。
文件及目录格式如下：

```
D:\modules-example>tree /F /A
\---src
    +---common.widget
    |   |   module-info.java
    |   |   
    |   +---com
    |   |   \---zimug
    |   |           RendererSupport.java
    |   |           
    |   \---org
    |       \---jwidgets
    |               SimpleRenderer.java
    |               
    \---data.widget
        |   module-info.java
        |   
        \---com
            \---example
                    Component.java
```

### 第一个模块

本代码文件目录：modules-example/src/common.widget/org/jwidgets/SimpleRenderer.java。这个package在后文中没有被exports。

```
package org.jwidgets;

public class SimpleRenderer {
  public void renderAsString(Object object) {
      System.out.println(object);
  }
}
```

本代码文件目录：modules-example/src/common.widget/com/zimug/RendererSupport.java。这个package在后文中被exports了。

```java
package com.zimug;

import org.jwidgets.SimpleRenderer;

public class RendererSupport {
	public void render(Object object) {
        new SimpleRenderer().renderAsString(object);
  	}
}
```

模块导出，本代码文件目录：modules-example/src/common.widget/module-info.java。只导出`com.zimug`包,没有导出 `org.jwidgets`包。导出的模块名称为`common.widget`

```java
module common.widget{
  	exports com.zimug;
}
```

### 第二个模块

模块导入`common.widget`，本代码文件目录：modules-example/src/data.widget/module-info.java

```java
module data.widget {
	requires common.widget;
}
```

使用导入模块`common.widget`中的package:`com.zimug`。本代码文件路径： modules-example/src/data.widget/com/example/Component.java

```java
package com.example;

import com.zimug.RendererSupport;

public class Component {
  	public static void main(String[] args) {
     	RendererSupport support = new RendererSupport();
      	support.render("Test Object");
  	}
}
```

正常编译执行，结果如下：

```java
Test Object
```

### 尝试使用未被exports的package代码

由于包“ org.jwidgets”尚未通过“ common.widget”模块导出，因此另一个模块“ data.widget”无法使用该package包下的类`SimpleRenderer`。我们做一个反例，看看会发生什么：

```java
package com.example;
import org.jwidgets.SimpleRenderer;

public class Component {
  	public static void main(String[] args) {
    	SimpleRenderer simpleRenderer = new SimpleRenderer(); 
    	simpleRenderer.renderAsString("Test Object");
  	}
}
```

编译报错信息如下：

```java
D:\modules-example\src\data.widget\com\example\Component.java:3: error: package org.jwidgets is not visible
import org.jwidgets.SimpleRenderer;
          ^
  (package org.jwidgets is declared in module common.widget, which does not export it)
1 error
```

正如我们所看到的，未被exports的package下面的class即使是public的也不能被访问。

