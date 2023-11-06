---
title: Advanced CSharp
date: 2023-3-19 12:23:23
tags:
  - C#
  - Language Learning
categories:
  - C#
  - Language Learning
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

# Advanced CSharp

[Dictionary (microsoft.com)](https://referencesource.microsoft.com/#mscorlib/system/collections/generic/dictionary.cs,d3599058f8d79be0,references)

哈希表、字典、二维数组的区别是什么？ - VTECISBEST的回答 - 知乎
https://www.zhihu.com/question/266414962/answer/308222427

## readonly, const & static

## Virtual, Interface & Abstract

## Delegate, Action, Func & Event

### Delegate

#### Quick Start

用实例快速了解 delegate：

```c#
public class Program {
    public delegate void TestDelegate();

    private static void MyTestDelegateFunction() {
        Console.WriteLine("My Test");
    }

    public static void Main(string[] args) {
        TestDelegate testDelegateFunction;

        testDelegateFunction = MyTestDelegateFunction;

        testDelegateFunction();
    }
}
```

> A [delegate](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/reference-types) is a type that represents references to methods with a particular parameter list and return type. When you instantiate a delegate, you can associate its instance with any method with a compatible signature and return type. You can invoke (or call) the method through the delegate instance.
>
> [Delegates - C# Programming Guide - C# | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/delegates/)

#### Delegates Overview

Delegates have the following properties:

- **Delegates are similar to C++ function pointers**, but delegates are fully object-oriented, and unlike C++ pointers to member functions, delegates encapsulate both an object instance and a method.
- Delegates allow methods to be passed as parameters.
- Delegates can be used to define callback methods.
- Delegates can be chained together; for example, multiple methods can be called on a single event.
- Methods don't have to match the delegate type exactly. For more information, see [Using Variance in Delegates](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/covariance-contravariance/using-variance-in-delegates).
- Lambda expressions are a more concise way of writing inline code blocks. Lambda expressions (in certain contexts) are compiled to delegate types. For more information about lambda expressions, see [Lambda expressions](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/lambda-expressions).

简单来说，Delegate 即委托，可以用于托管任意数量的、和自己定义了一样参数的函数，事实上，和委托形态最接近的数据结构是——链表。你也许会从这下面的例子中找到结论：

```c#
public class Program {
    public delegate void TestDelegate(int a, int b);

    private static void MyTestDelegateFunction1(int i, int t) {
        Console.WriteLine(i + t);
    }

    private static void MyTestDelegateFunction2(int x, int y) {
        Console.WriteLine(x * y);
    }

    public static void Main(string[] args) {
        TestDelegate testDelegateFunction;

        testDelegateFunction = MyTestDelegateFunction1;
        // TestDelegate testDelegateFunction = new TestDelegate(MyTestDelegateFunction1);

        testDelegateFunction += MyTestDelegateFunction2;

        testDelegateFunction(2, 3);

        testDelegateFunction -= MyTestDelegateFunction1;

        testDelegateFunction(2, 3);
    }
}
// Output:
// 5
// 6
// 6
```

当然，我们也可以改写上面这个例子：

```c#
public class Program {
    public delegate void TestDelegate(int a, int b);

    public static void Main(string[] args) {
        TestDelegate testDelegateFunction = (int i, int t) => { Console.WriteLine(i + t); };
        testDelegateFunction += (int x, int y) => { Console.WriteLine(x * y); };

        testDelegateFunction(2, 3);
    }
}
```

但是使用 Lambda 表达的另一个问题是——由于我们没有函数标识，我们无法从 delegate 中移除对应的函数了！

### Action&Func

#### 一些区别

首先注意的是，我们这样去创建一个 Action：

```c#
Action<float> TestAction;
```

注意和 Delegate 的区别——是的，这里没有返回值

让我们转向 Func：

```c#
Func<bool> TestFunc;
```

简直是和 Action 如出一辙！那这两者的功能各自又是什么？

#### 使用

让我们依然使用实例来了解具体细节：

```c#
public class Program {
    public static void Main(string[] args) {
        Action<float, float> testAction;
        Func<bool> testFunc;

        testAction = (float a, float b) => { Console.WriteLine(a * b); };
        testFunc = () => { return false; };

        testAction(3, 4);
        Console.WriteLine(testFunc());
    }
}
```

你可能已经猜到了：Action 是只接受输入的函数委托，Func 则是以输出为主的函数委托。

欸？为什么是“为主”？让我们借用一下这两张图：

![img](https://pic1.zhimg.com/80/v2-7513645d0b2a6d143b0da2f5d7288274_1440w.webp)

![img](https://pic2.zhimg.com/80/v2-eef6f55eca4cc70575cae8e12f2c5f49_1440w.webp)

> Action: [Action Delegate (System) | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/api/system.action?view=net-7.0)
>
> Func: [Func Delegate (System) | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/api/system.func-1?view=net-7.0)

#### 为什么需要 Action&Func？

当我们使用 Delegate 时，我们一共做了两部：先显式定义委托本身，然后再用委托实例去完成过程，但是当使用 Action&Func 时，我们的代码结构就可以得到省略：

```C#
// From：
public class Program {
    public delegate int explicitDelegate(int a, int b);

    public static void Main(string[] args) {
        explicitDelegate a = (int i, int t) => { return i * t; };
        Console.WriteLine(a(3, 4));
    }
}

// To：
public class Program {
    public static void Main(string[] args) {
        Func<int, int, int> a = (int i, int t) => { return i * t; };
        Console.WriteLine(a(3, 4));
    }
}
```

#### 什么时候会用到委托？

让我们看向这样一个例子：

```c#
using UnityEngine;

public class ActionOnTimer : MonoBehaviour{
    private float timer;

    public void SetTimer(float timer) {
        this.timer = timer;
    }

    private void Update() {
        timer -= Time.deltaTime;
    }

    public bool IsTimerComplete() {
        return timer <= Of;
    }
}
```

可以看到，这是一个计时器，timer 是我们的定时。如果我们希望 1s 后做点什么？

```c#
using UnityEngine;

public class Testing : MonoBehaviour{
    [SerializeField] private ActionOnTimer actionOnTimer;
    private bool hasTimerElapsed;

    private void start() {
        actionOnTimer.SetTimer(1f);
    }

    private void Update() {
        if (!hasTimerElapsed && actionOnTimer.IsTimerComplete()) {
            Debug.Log("Timer complete!");
            hasTimerElapsed = true;
        }
    }
}
```

好，好。我们已经在 1s 后做了点事，不过我们的实现确实称不上好...我们将太多的逻辑延申了出来——如果 ActionOnTimer 可以更紧凑就好了。

让我们用 delegate 来改造原始的 ActionOnTimer：

```C#
using UnityEngine;

public class ActionOnTimer : MonoBehaviour{
    private Action timeCallback;
    private float timer;

    public void SetTimer(float timer, Action timeCallback) {
        this.timeCallback = timeCallback;
        this.timer = timer;
    }

    private void Update() {
        if (timer > 0f) {
            timer -= Time.deltaTime;

            if (IsTimerComplete()) timerCallback();
        }
    }

    public bool IsTimerComplete() {
        return timer <= Of;
    }
}
```

那现在？现在我们的调用就可以相当紧凑了：

```c#
using UnityEngine;

public class Testing : MonoBehaviour{
    [SerializeField] private ActionOnTimer actionOnTimer;
    private void start() {
        actionOnTimer.SetTimer(1f, () => { Debug.Log("Timer Complete"); });
    }
}
```

是不是很简单？

特别的，我在这里列举了一些我们在委托中需要注意的事情：

- [多播委托的坑 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/291277405)

另外需要提到的一点是，Unity 里存在一个 UnityAction，你也许会好奇这是什么：

> Actually `UnityAction<T>` and `System.Action<T>` are both the same generic delegate declarations, just seperate reimplementations. AFAIK UnityEvents used System.Actions in the past and just recently switched to their own version. Both are not really serializable since they are just plain C# delegate which are generally hard up to implossible to serialize. So the exact reason why they shipped their own version is kinda unclear. For me the most sound explanation would be the independence from the System namespace / mscorlib implementation. Though that's just speculation.
>
> Be warned that even though both generic delegate are implemented exactly the same, doesn't make them compatible to each other out of the box. A cast should be possible though.
>
> [What's the difference between UnityAction and C# Actions? - Unity Forum](https://forum.unity.com/threads/whats-the-difference-between-unityaction-and-c-actions.1041184/)

具体采用则可以参考：[Why choose UnityEvent over native C# events? - Stack Overflow](https://stackoverflow.com/questions/44734580/why-choose-unityevent-over-native-c-sharp-events)

### Event

那让我们继续，下面是 Event——一种特殊的委托

> Events enable a [class](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/class) or object to notify other classes or objects when something of interest occurs. The class that sends (or _raises_) the event is called the _publisher_ and the classes that receive (or _handle_) the event are called _subscribers_.
>
> [Events - C# Programming Guide - C# | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/events/)

也就是说，Event 下鲜明地存在两种角色：Publishers 和 Subscribers，并且这是一个一对多的关系：由 Publishers 发出事件并且由 Subscribers 接收。有没有想起什么？对，这看上去就和观察者模式很像！

> 随便打开电脑中的一个应用，很有可能它就使用了[MVC 架构](http://en.wikipedia.org/wiki/Model–view–controller)， 而究其根本，是因为观察者模式。 观察者模式应用广泛，Java 甚至将其放到了核心库之中（[`java.util.Observer`](http://docs.oracle.com/javase/7/docs/api/java/util/Observer.html)），而 C#直接将其嵌入了*语法*（[`event`](http://msdn.microsoft.com/en-us/library/8627sbea.aspx)关键字）。
>
> [观察者模式 · Design Patterns Revisited · 游戏设计模式 (tkchu.me)](https://gpp.tkchu.me/observer.html)

但是我们这里是 event 的 part，所以让我们回到 event 本身上来——还是看看代码吧：

```c#
using System;
namespace SimpleEvent {
  /***********发布器类***********/
  public class EventTest {
    private int value;

    public delegate void NumManipulationHandler();
    public event NumManipulationHandler ChangeNum;

    protected virtual void OnNumChanged() {
      if ( ChangeNum != null ) {
        ChangeNum(); /* 事件被触发 */
      } else {
        Console.WriteLine( "event not fire" );
        Console.ReadKey(); /* 回车继续 */
      }
    }

    public EventTest() {
      int n = 5;
      SetValue( n );
    }

    public void SetValue( int n ) {
      if ( value != n ) {
        value = n;
        OnNumChanged();
      }
    }
  }

  /***********订阅器类***********/
  public class subscribEvent {
    public void printf() {
      Console.WriteLine( "event fire" );
      Console.ReadKey(); /* 回车继续 */
    }
  }

  /***********触发***********/
  public class MainClass {
    public static void Main() {
      EventTest e = new EventTest(); /* 实例化Publisher对象 */
      subscribEvent v = new subscribEvent(); /* 实例化Subscriber对象 */
      e.ChangeNum += new EventTest.NumManipulationHandler( v.printf ); /* 注册 */
      e.SetValue( 7 );
      e.SetValue( 11 );
    }
  }
}
```

事实上，event 使用流程都大体如下：

![No alt text provided for this image](https://media.licdn.com/dms/image/C5612AQH4GMt0SAL1dQ/article-inline_image-shrink_1000_1488/0/1624221860036?e=1700697600&v=beta&t=z3qXv19g7LDe7pToTPJCSgeBS6WvvZs3uhwa0YN09Vg)

可以发现，event 是对 delegate 的一层额外包装，它在执行上看仍然可以看作是一个 delegate，发挥着 delegate 的功能。那为什么我们还需要 event？

- [Delegates vs. events - C# | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/csharp/distinguish-delegates-events)

另外，这里是存在一些预制的 delegate 的，比如我们将用到的 eventHandler

> ```c#
> public delegate void EventHandler(object? sender, EventArgs e);
> ```
>
> [EventHandler Delegate (System) | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/api/system.eventhandler?view=net-7.0)

```c#
// Publishers
using System;
using UnityEngine;

public class TestingEvents : MonoBehaviour {
    public event EventHandler OnSpacePressed;

    private void Update() {
        if (Input.GetKeyDown(KeyCode.Space)) {
            OnSpacePressed?.Invoke(this, EventArgs.Empty);
        }
    }
}

// Subscribers
using System;
using UnityEngine;

public class TestingEventSubscriber : MonoBehaviour {
    private void Start() {
        TestingEvents testingEvents = GetComponent<TestingEvents>();
        testingEvents.OnSpacePressed += TestingEvents_OnSpacePressed;
    }

    // We should match the signature of target event
    private void TestingEvents_OnSpacePressed(object sender, EventArgs e) {
        Debug.Log("Space!");
    }
}
```

另外需要介绍的是 EventHandler 支持的泛型：

```c#
// Publishers
using System;
using UnityEngine;

public class TestingEvents : MonoBehaviour {
    public event EventHandler<OnSpacePressedEventArgs> OnSpacePressed;

    public class OnSpacePressedEventArgs : EventArgs {
        public int pressedCount;
    }

    private int pressedCount;

    private void Update() {
        if (Input.GetKeyDown(KeyCode.Space)) {
            pressedCount++;
            OnspacePressed?.Invoke(this, new OnSpacePressedEventArgs{ pressedCount = pressedCount });
        }
    }
}

// Subscribers
using System;
using UnityEngine;

public class TestingEventSubscriber : MonoBehaviour {
    private void Start() {
        TestingEvents testingEvents = GetComponent<TestingEvents>();
        testingEvents.OnSpacePressed += TestingEvents_OnSpacePressed;
    }

    // We should match the signature of target event
    private void TestingEvents_OnSpacePressed(object sender, TestingEvents.OnSpacePressedEventArgs e) {
        Debug.Log("Space!" + e.pressedCount);
    }
}
```

> The EventHandler\<TEventArgs\> delegate is a predefined delegate that represents an event handler method for an event that generates data. The advantage of using EventHandler\<TEventArgs\> is that you do not need to code your own custom delegate if your event generates event data. You simply provide the type of the event data object as the generic parameter.

### 小结

就让我们这样归类这三者吧：

- Delegate：链表形式的、面向对象的函数指针。接收一致参数的函数委托，并同时执行所有受委托的函数输出结果。同时，Delegate 是既接收输入，又能完成输出的类型。
- Action&Func：对 Delegate 在不同方面的“简化”，能让代码逻辑更加清晰。
- Event：观察者模式的直接实现，Delegate 的特化

## Attribute 与 Reflection

### 元数据

有关程序及其类型的数据被称为**元数据（metadata）**，它们保存在程序的程序集中。可以这样来形象了解元数据：

> 我们先假装 C#没有反射吧！我现在要你在 C#上实现一个函数。函数参数是 Object，函数的功能就是执行这个类内所有参数为空的公有成员方法。好了，你写吧！
>
> ...
>
> ...
>
> "这怎么可能写的出来啊！！运行时获取类里面的所有成员方法！类里有啥成员方法只有编译器和写类的程序员才知道吧！！"
>
> 没错...那么如果允许你，在类里加点东西，来实现这个功能呢？
>
> 那好像就简单了...
>
> 我们可以这样:
>
> 1. 把 C#这个函数的参数限定一下，参数类型改为 MyObject
> 2. 写一个 class MyObject，让所有类都继承自 class MyObject;MyObject 内只实现一个方法:public FuncData[] getFuncData();
> 3. 声明 struct FuncData;FuncData 里就几个属性:返回值类型、函数名称、参数列表(参数类型、名称)
> 4. 给每个类都手写一个 FuncData 的表
>
> 4 实现起来好像很麻烦，又臭又长又机械...但是到此为止，你基本上就是实现了一个很简单的反射功能了...
>
> 好了，然后聪明的你应该就知道了，C#默认就实现了以上功能，也就是说 C#编译器默认就会帮你写好这个 FuncData——和 PropData(类内所有属性的数据表)，他们被合称为 MetaData，元数据。
>
> C#反射呢，就是基于 MetaData 实现的，一个能让代码在执行的时候，知道代码(类)们的属性的一个功能。
>
> https://www.zhihu.com/question/308374020/answer/621704342
>
> 另：关于 Type：[Type Class (System) | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/api/system.type?view=net-7.0)

### Attribute

> 类 [Attribute](https://learn.microsoft.com/zh-cn/dotnet/api/system.attribute?view=net-7.0) 将预定义的系统信息或用户定义的自定义信息与目标元素相关联。 目标元素可以是程序集、类、构造函数、委托、枚举、事件、字段、接口、方法、可移植可执行文件模块、参数、属性、返回值、结构或其他属性。
>
> Attribute 提供的信息也称为元数据。 应用程序可以在运行时检查元数据，以控制程序处理数据的方式，或者在运行时之前由外部工具检查元数据，以控制应用程序本身的处理或维护方式。 例如，.NET 预定义并使用属性类型来控制运行时行为，某些编程语言使用属性类型来表示 .NET 通用类型系统不直接支持的语言功能。
>
> [Attribute 类 (System) | Microsoft Learn](https://learn.microsoft.com/zh-cn/dotnet/api/system.attribute?view=net-7.0)

是的，Attribute 是用来添加元数据的，如编译器指令和注释、描述、方法、类等其他信息。.Net 框架提供了两种类型的特性：*预定义特性*和*自定义特性*。

#### 预定义特性

> 为了构造自定义特性，我们得先了解一些预定义的

.Net 框架提供了三种预定义特性：

- AttributeUsage
- Conditional
- Obsolete

这里列举了两个我们可能会经常见到的：

##### AttributeUsage

预定义特性 `AttributeUsage` **描述了如何使用一个自定义特性类**。它规定了特性可应用到的项目的类型。

规定该特性的语法如下：

```c#
[AttributeUsage(
   validon,
   AllowMultiple=allowmultiple,
   Inherited=inherited
)]
```

其中：

- 参数 validon 规定特性可被放置的语言元素。它是枚举器 _AttributeTargets_ 的值的组合。默认值是 _AttributeTargets.All_。
- 参数 _allowmultiple_（可选的）为该特性的 _AllowMultiple_ 属性（property）提供一个布尔值。如果为 true，则该特性是多用的。默认值是 false（单用的）。
- 参数 _inherited_（可选的）为该特性的 _Inherited_ 属性（property）提供一个布尔值。如果为 true，则该特性可被派生类继承。默认值是 false（不被继承）。

我们后方的用例就存在该特性。

##### Obsolete

这是一个我们经常会见到的特性，它标记了不应被使用的程序实体。它可以让您通知编译器丢弃某个特定的目标元素。例如，当一个新方法被用在一个类中，但是您仍然想要保持类中的旧方法，您可以通过显示一个应该使用新方法，而不是旧方法的消息，来把它标记为 obsolete（过时的）。

规定该特性的语法如下：

```c#
[Obsolete(message, iserror=false)]
```

其中：

- 参数 _message_，是一个字符串，描述项目为什么过时以及该替代使用什么。
- 参数 _iserror_，是一个布尔值。如果该值为 true，编译器应把该项目的使用当作一个错误。默认值是 false（编译器生成一个警告）。

下面的实例演示了该特性：

```c#
using System;
public class MyClass {
   [Obsolete("Don't use OldMethod, use NewMethod instead", true)]
   static void OldMethod() {
      Console.WriteLine("It is the old method");
   }
   static void NewMethod() {
      Console.WriteLine("It is the new method");
   }
   public static void Main() {
      OldMethod();
   }
}
```

当尝试编译该程序时，编译器会给出一个错误消息说明：

```c#
 Don't use OldMethod, use NewMethod instead
```

#### 自定义特性

> 在 C# 中，特性是继承自 `Attribute` 基类的类。

通过定义继承自 `Attribute` 基类的新类来创建特性。

```c#
public class MySpecialAttribute : Attribute {}
```

通过上述代码，可在基本代码中的其他位置将 `[MySpecial]`（或 `[MySpecialAttribute]`）用作特性。

```c#
[MySpecial]
public class SomeOtherClass {}
```

我们需要继续充实这个自定义特性，才能得到一个真正可用的自定义特性：

> 让我们构建一个名为 _DeBugInfo_ 的自定义特性，该特性将存储调试程序获得的信息。它存储下面的信息：
>
> - bug 的代码编号
> - 辨认该 bug 的开发人员名字
> - 最后一次审查该代码的日期
> - 一个存储了开发人员标记的字符串消息
>
> 每个特性必须至少有一个构造函数。必需的 `定位(positional)` 参数应通过构造函数传递。下面的代码演示了 _DeBugInfo_ 类：

```c#
// 一个自定义特性 BugFix 被赋给类及其成员
[AttributeUsage(
    AttributeTargets.Class |
    AttributeTargets.Constructor |
    AttributeTargets.Field |
    AttributeTargets.Method |
    AttributeTargets.Property,
    AllowMultiple = true)
]

public class DeBugInfo : Attribute {
    private int bugNo;
    private string developer;
    private string lastReview;
    public string message;

    public DeBugInfo(int bg, string dev, string d) {
        this.bugNo = bg;
        this.developer = dev;
        this.lastReview = d;
    }

    public int BugNo { get { return bugNo; } }
    public string Developer { get { return developer; } }
    public string LastReview { get { return lastReview; } }
    public string Message {
        get { return message; }
        set { message = value; }
	}
}
```

还是一样，我们通过把特性放置在紧接着它的目标之前，来应用该特性：

```c#
[DeBugInfo(45, "Zara Ali", "12/8/2012", Message = "Return type mismatch")]
[DeBugInfo(49, "Nuha Ali", "10/10/2012", Message = "Unused variable")]
class Rectangle {
  // 成员变量
  protected double length;
  protected double width;
  public Rectangle(double l, double w) {
      length = l;
      width = w;
  }
  [DeBugInfo(55, "Zara Ali", "19/10/2012", Message = "Return type mismatch")]
  public double GetArea() {
      return length * width;
  }
  [DeBugInfo(56, "Zara Ali", "19/10/2012")]
  public void Display(){
      Console.WriteLine("Length: {0}", length);
      Console.WriteLine("Width: {0}", width);
      Console.WriteLine("Area: {0}", GetArea());
  }
}
```

可是，我们的特性没有对这段代码产生任何实质性的影响。我们还需要做些什么？

> The fact that you can define custom attributes and place them in your source code would be of little value without some way of retrieving that information and acting on it. By using reflection, you can retrieve the information that was defined with custom attributes. The key method is `GetCustomAttributes`, which returns an array of objects that are the run-time equivalents of the source code attributes.
>
> [Access attributes using reflection - C# | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/reflection-and-attributes/accessing-attributes-by-using-reflection)

### Reflection

> 反射指程序可以访问、检测和修改它本身状态或行为的一种能力。

对于一个特性来说，其具体的执行，例如：

```csharp
[Author("P. Ackerman", Version = 1.1)]
class SampleClass { }
```

在概念上等效于以下代码：

```csharp
var anonymousAuthorObject = new Author("P. Ackerman") {
    Version = 1.1
};
```

但是，在为特性查询 `SampleClass` 之前，代码将不会执行。 而对 `SampleClass` 调用 `GetCustomAttributes` 则会导致构造并初始化一个 `Author` 对象。 如果该类具有其他特性，则将以类似方式构造其他特性对象。 然后 `GetCustomAttributes` 会以数组形式返回 `Author` 对象和任何其他特性对象。 之后你便可以循环访问此数组，根据每个数组元素的类型确定所应用的特性，并从特性对象中提取信息。关于 `GetCustomAttributes`，可见：[MemberInfo.GetCustomAttributes Method (System.Reflection) | Microsoft Learn](<https://learn.microsoft.com/en-us/dotnet/api/system.reflection.memberinfo.getcustomattributes?view=net-7.0#system-reflection-memberinfo-getcustomattributes(system-boolean)>)

下面是完整的示例。 定义自定义特性、将其应用于多个实体，并通过反射对其进行检索：

```c#
// Multiuse attribute.
[System.AttributeUsage(System.AttributeTargets.Class |
                       System.AttributeTargets.Struct,
                       AllowMultiple = true)  // Multiuse attribute.
]
public class AuthorAttribute : System.Attribute {
    string Name;
    public double Version;

    public AuthorAttribute(string name) {
        Name = name;
        Version = 1.0;
    }

    public string GetName() => Name;
}

// Class with the Author attribute.
[Author("P. Ackerman")]
public class FirstClass {
    // ...
}

// Class without the Author attribute.
public class SecondClass {
    // ...
}

// Class with multiple Author attributes.
[Author("P. Ackerman"), Author("R. Koch", Version = 2.0)]
public class ThirdClass {
    // ...
}

class TestAuthorAttribute {
    public static void Main(string[] args) {
        PrintAuthorInfo(typeof(FirstClass));
        PrintAuthorInfo(typeof(SecondClass));
        PrintAuthorInfo(typeof(ThirdClass));
    }

    private static void PrintAuthorInfo(System.Type t) {
        System.Console.WriteLine($"Author information for {t}");

        // Using reflection.
        System.Attribute[] attrs = System.Attribute.GetCustomAttributes(t);  // Reflection.

        // Displaying output.
        foreach (System.Attribute attr in attrs) {
            if (attr is AuthorAttribute a) { // see https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns#declaration-and-type-patterns
                System.Console.WriteLine($"   {a.GetName()}, version {a.Version:f}");
            }
        }
    }
}
/* Output:
    Author information for FirstClass
       P. Ackerman, version 1.00
    Author information for SecondClass
    Author information for ThirdClass
       R. Koch, version 2.00
       P. Ackerman, version 1.00
*/
```

可以看到的是，我们提取出了注解中的全部信息。知道了这一办法后，我们就可以反过来完善 Attribute 的例子了！

```c#
using System;
using System.Reflection;
namespace BugFixApplication{
    // 一个自定义特性 BugFix
    [AttributeUsage(AttributeTargets.Class |
    AttributeTargets.Constructor |
    AttributeTargets.Field |
    AttributeTargets.Method |
    AttributeTargets.Property,
    AllowMultiple = true)]
    public class DeBugInfo : Attribute {
        private int bugNo;
        private string developer;
        private string lastReview;
        public string? message;

        public DeBugInfo(int bg, string dev, string d) {
            this.bugNo = bg;
            this.developer = dev;
            this.lastReview = d;
        }

        public int BugNo { get { return bugNo; } }
        public string Developer { get { return developer; } }
        public string LastReview { get { return lastReview; } }
        public string? Message {
            get { return message; }
            set { message = value; }
        }
	}

   [DeBugInfo(45, "Zara Ali", "12/8/2012", Message = "Return type mismatch")]
   [DeBugInfo(49, "Nuha Ali", "10/10/2012", Message = "Unused variable")]
   class Rectangle {
      // 成员变量
      protected double length;
      protected double width;
      public Rectangle(double l, double w) {
         length = l;
         width = w;
      }
      [DeBugInfo(55, "Zara Ali", "19/10/2012", Message = "Return type mismatch")]
      public double GetArea() {
         return length * width;
      }
      [DeBugInfo(56, "Zara Ali", "19/10/2012")]
      public void Display() {
         Console.WriteLine("Length: {0}", length);
         Console.WriteLine("Width: {0}", width);
         Console.WriteLine("Area: {0}", GetArea());
      }
   }
   
   class ExecuteRectangle {
      static void Main(string[] args) {
         Rectangle r = new Rectangle(4.5, 7.5);
         r.Display();
         Type type = typeof(Rectangle);

         // 遍历 Rectangle 类的特性
         foreach (Object attributes in type.GetCustomAttributes(false)) {
            if (attributes is DeBugInfo dbi) {
               Console.WriteLine("Bug no: {0}", dbi.BugNo);
               Console.WriteLine("Developer: {0}", dbi.Developer);
               Console.WriteLine("Last Reviewed: {0}", dbi.LastReview);
               Console.WriteLine("Remarks: {0}", dbi.Message);
            }
         }
         
         // 遍历方法特性
         foreach (MethodInfo m in type.GetMethods()) {
            foreach (Attribute a in m.GetCustomAttributes(true)) {
               if (a is DeBugInfo dbi) {
                  Console.WriteLine("Bug no: {0}, for Method: {1}", dbi.BugNo, m.Name);
                  Console.WriteLine("Developer: {0}", dbi.Developer);
                  Console.WriteLine("Last Reviewed: {0}", dbi.LastReview);
                  Console.WriteLine("Remarks: {0}", dbi.Message);
               }
            }
         }
      }
   }
}
/* Output:
    Length: 4.5
    Width: 7.5
    Area: 33.75
    Bug No: 49
    Developer: Nuha Ali
    Last Reviewed: 10/10/2012
    Remarks: Unused variable
    Bug No: 45
    Developer: Zara Ali
    Last Reviewed: 12/8/2012
    Remarks: Return type mismatch
    Bug No: 55, for Method: GetArea
    Developer: Zara Ali
    Last Reviewed: 19/10/2012
    Remarks: Return type mismatch
    Bug No: 56, for Method: Display
    Developer: Zara Ali
    Last Reviewed: 19/10/2012
    Remarks:
*/
```

更多可见：

- [教程：定义和读取自定义特性。 - C# | Microsoft Learn](https://learn.microsoft.com/zh-cn/dotnet/csharp/advanced-topics/reflection-and-attributes/attribute-tutorial)
- [C#之玩转反射 - y-z-f - 博客园 (cnblogs.com)](https://www.cnblogs.com/yaozhenfa/p/CSharp_Reflection_1.html)
- [[C#\]理解和入门依赖注入 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/592698341)
- [【C#入门详解 16】-反射、依赖注入 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/407070469)

## ?、??、?.、is

### ??

```c#
value ?? expression
```

`??` 会判断 value 是否为 null，如果是则执行 expression，而不是则返回 value。

另外，`??` 运算符要求两个操作数都具有相同的类型或可以隐式转换为相同的类型。

因此，这样的书写便是不符合规则的

```C#
class Program {
    static A ab;

    public static void NewA() {
        ab = new A();
    }

    public static void Main(string[] args){
        ab = ab ?? NewA();
    }
}
public class A {}
```

应该是这样的：

```c#
class Program {
    static A ab;

    public static A NewA() {
        return new A();
    }

    public static void Main(string[] args) {
        ab = ab ?? NewA();
    }
}
public class A {}
```

### ?.invode

## 多线程

### 线程

Task

### Unity 协程 Coroutine

> Unity 开发不可避免的要用到协程(Coroutine)，协程能在同步代码中做异步任务，避免了异步操作加回调的 _编码_ 方式

- [聊一聊 Unity 协程背后的实现原理 - iwiniwin - 博客园 (cnblogs.com)](https://www.cnblogs.com/iwiniwin/p/14878498.html)
- [C#图解教程之枚举器和迭代器｜ Zhendong's blog (hzd.plus)](https://hzd.plus/2019/08/25/C-图解教程之枚举器和迭代器/)
- [Unity 协程的原理与应用 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/279383752)

## DOTS

DOTS 即 Data Oriented Technology Stack，是 Unity 关于面向数据编程的一系列技术栈

在正式介绍 DOTS 之前，我想你应该先了解一下什么是面向数据编程，你也许需要先读下这篇文章：
