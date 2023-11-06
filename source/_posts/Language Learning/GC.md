---
title:  垃圾回收 - Garbage Collection
date: 2023-5-6 2:52:23
tags:
  - C#
  - Language Learning
categories:
  - C#
  - Language Learning
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

# 垃圾回收 - Garbage Collection

## 为什么需要GC

1. 方便

2. > - 为何主流游戏引擎的对象需要有GC机制？ - 霍姚远的回答 - 知乎 https://www.zhihu.com/question/518026433/answer/2488730232
   > - 为何主流游戏引擎的对象需要有GC机制？ - DBinary的回答 - 知乎 https://www.zhihu.com/question/518026433/answer/2518074579

## .Net的GC机制

### 特点

- 精准式GC

- 只处理引用类型的变量：值类型的变量会随着离开对应的作用域而自然销毁，而只有引用类型的变量才涉及到使用堆空间。特别的，我们将引用类型的变量称为”根“

- GC并不是实时性的，而且GC的执行会导致进程停滞——这会造成系统性能上的瓶颈和不确定性。所以有了IDisposable接口，IDisposable接口定义了Dispose方法，这个方法用来供程序员显式调用以释放非托管资源。使用using语句可以简化资源管理。

  - 关于GC导致的进程停滞：GC在完成垃圾清理后，会对留存对象进行碎片整理：这会带来内存中对象address变化，如果此时不挂起，那么上一秒还在读取的对象，下一秒写的时候写到另一个对象里去了。因为可能大多数对象已经被move了。这跟线程锁，临界区是同样的原理。

- GC并不是能释放所有的资源。它不能自动释放非托管资源。

  - 请注意：`非托管类型` 和 `非托管资源` 是不一样的！

    如果某个类型是以下类型之一，则它是非托管类型 ：

     - `sbyte`、`byte`、`short`、`ushort`、`int`、`uint`、`long`、`ulong`、`nint`、`nuint`、`char`、`float`、`double`、`decimal` 或 `bool`
     - 任何[枚举](https://learn.microsoft.com/zh-cn/dotnet/csharp/language-reference/builtin-types/enum)类型
     - 任何[指针](https://learn.microsoft.com/zh-cn/dotnet/csharp/language-reference/unsafe-code#pointer-types)类型
     - 任何仅包含非托管类型字段的用户定义的[结构](https://learn.microsoft.com/zh-cn/dotnet/csharp/language-reference/builtin-types/struct)类型。

  - 非托管资源只是托管资源的相对概念，即：

    > > Managed resources basically means "managed memory" that is managed by the garbage collector. ... Unmanaged resources are then everything that the garbage collector does not know about. For example:
    > >
    > > - Open files
    > > - Open network connections
    > > - Unmanaged memory
    > > - In XNA: vertex buffers, index buffers, textures, etc.
    >
    > 简单的说，这些 `非托管资源` 仍然是reference，但是它们引用的不是.Net本身能在产生、获取等各种方式下得到的数据，它们是外部资源。它们随程序的运行于managed resources一同进入了内存中，可gc无法在使用完后想正常内容一样去释放它们，in short:
    >
    > > Unmanaged resources are those that run outside the .NET runtime (CLR)(aka non-.NET code.) For example, a call to a DLL in the Win32 API, or a call to a .dll written in C++.
    >
    > 一个简单的例子是，我们对int进行装箱，产生的Object是 `托管资源`，因为它由.Net全权接管。而如果我们open一个file，那么file的内容不在托管堆中
  
  - > 对于应用创建的大多数对象，可以依赖 [.NET 垃圾回收器](https://learn.microsoft.com/zh-cn/dotnet/standard/garbage-collection/)来进行内存管理。 但是，如果创建包含 `非托管资源` 的对象，则当你使用完 `非托管资源` 后，就必须显式释放这些资源。 最常用的 `非托管资源` 类型是包装操作系统资源的对象，如文件、窗口、网络连接或数据库连接。 虽然垃圾回收器可以跟踪封装非托管资源的对象的生存期，但无法了解如何发布并清理这些非托管资源。

### 自动内存管理

> [自动内存管理 | Microsoft Learn](https://learn.microsoft.com/zh-cn/dotnet/standard/automatic-memory-management)
>
> > 初始化新进程时，运行时会为进程保留一个连续的地址空间区域。 这个保留的地址空间被称为托管堆。 托管堆维护着一个指针，用它指向将在堆中分配的下一个对象的地址。 最初，该指针设置为指向托管堆的基址。 托管堆上包含了所有[引用类型](https://learn.microsoft.com/zh-cn/dotnet/standard/base-types/common-type-system)。 应用程序创建第一个引用类型时，将为托管堆的基址中的类型分配内存。 应用程序创建下一个对象时，垃圾回收器在紧接第一个对象后面的地址空间内为它分配内存。 只要地址空间可用，垃圾回收器就会继续以这种方式为新对象分配空间。
>
> > 垃圾回收器的优化引擎根据所执行的分配决定执行回收的最佳时间。 垃圾回收器在执行回收时，会释放应用程序不再使用的对象的内存...垃圾回收器可以访问由[实时 (JIT) 编译器](https://learn.microsoft.com/zh-cn/dotnet/standard/managed-execution-process)和运行时维护的活动根的列表...并在此过程中创建一个图表...不在该图表中的对象将无法从应用程序的根中访问。 垃圾回收器会考虑无法访问的对象垃圾，并释放为它们分配的内存。 在回收中，垃圾回收器检查托管堆，查找无法访问对象所占据的地址空间块。 发现无法访问的对象时，它就使用内存复制功能来压缩内存中可以访问的对象，释放分配给不可访问对象的地址空间块。 在压缩了可访问对象的内存后，垃圾回收器就会做出必要的指针更正，以便应用程序的根指向新地址中的对象。 它还将托管堆指针定位至最后一个可访问对象之后。 **请注意，只有在回收发现大量的无法访问的对象时，才会压缩内存。 如果托管堆中的所有对象均未被回收，则不需要压缩内存。**为了改进性能，运行时为单独堆中的大型对象分配内存。 垃圾回收器会自动释放大型对象的内存。 但是，为了避免移动内存中的大型对象，不会压缩此内存。
>

### 一般执行过程

1. 首先，CLR暂停进程中的所有线程。防止线程在CLR检查期间访问对象并更改其状态。
2. CLR进入GC的标记阶段
   1. CLR遍历堆中某些代下的对象，并全部设为 `不可达`
   2. CLR检查所有活动根，对于任何活动根直接引用的堆上对象，都会设为 `可达`
   3. 检查被标记为可达对象中存在的根，并将这些根所引用的对象也设为 `可达`
3. 标记完毕，对 `不可达` 对象进行清理，同时对留存下来的对象进行碎片整理(GC Compact)
4. 重新计算对象的偏移位置，以及指针位置，并在完成所有操作后，恢复应用程序进程

### .Net GC的代际设计

CLR的GC是`基于代的垃圾回收器`，它假设：

- 压缩托管堆的一部分内存要比压缩整个托管堆速度快——分代
- 较新的对象生存期较短，而较旧的对象生存期则较长——高频处理低代际，低频处理高代际
- 较新的对象趋向于相互关联，并且大致同时由应用程序访问——连续分配内存

托管堆最多支持三代对象：

- 第0代对象：新构造的未被GC检查过的对象
- 第1代对象：被GC检查过1次且保留下来的对象
- 第2代对象：被GC检查大于等于2次且保留下来的对象

第0代回收只会回收第0代对象，第1代回收则会回收第0代和第1代对象，而第2代回收表示完全回收，会回收所有对象。

### 在GC之外 - `非托管资源` 的回收

> 如果你的类型使用非托管资源，则应执行以下操作：
>
> - 实现[清理模式](https://learn.microsoft.com/zh-cn/dotnet/standard/garbage-collection/implementing-dispose)。 这要求你提供 [IDisposable.Dispose](https://learn.microsoft.com/zh-cn/dotnet/api/system.idisposable.dispose) 实现以启用非托管资源的确定性释放。 当不再需要此对象（或其使用的资源）时，类型使用者可调用 [Dispose](https://learn.microsoft.com/zh-cn/dotnet/api/system.idisposable.dispose)。 [Dispose](https://learn.microsoft.com/zh-cn/dotnet/api/system.idisposable.dispose) 方法立即释放非托管资源。
>
> - 在类型使用者忘记调用 [Dispose](https://learn.microsoft.com/zh-cn/dotnet/api/system.idisposable.dispose) 的情况下，请提供一种方法来释放非托管资源。 有两种方法可以实现此目的：
>
>   - 使用安全句柄包装非托管资源。 这是推荐采用的方法。 安全句柄派生自 [System.Runtime.InteropServices.SafeHandle](https://learn.microsoft.com/zh-cn/dotnet/api/system.runtime.interopservices.safehandle) 抽象类，并包含可靠的 [Finalize](https://learn.microsoft.com/zh-cn/dotnet/api/system.object.finalize) 方法。 在使用安全句柄时，只需实现 [IDisposable](https://learn.microsoft.com/zh-cn/dotnet/api/system.idisposable) 接口并在 [Dispose](https://learn.microsoft.com/zh-cn/dotnet/api/system.runtime.interopservices.safehandle.dispose) 实现中调用安全句柄的 [IDisposable.Dispose](https://learn.microsoft.com/zh-cn/dotnet/api/system.idisposable.dispose) 方法。 如果未调用安全句柄的 [Dispose](https://learn.microsoft.com/zh-cn/dotnet/api/system.idisposable.dispose) 方法，则垃圾回收器将自动调用安全句柄的终结器。
>
>     —或—
>
>   - 定义[终结器](https://learn.microsoft.com/zh-cn/dotnet/csharp/programming-guide/classes-and-structs/finalizers)。 当类型使用者无法调用 [IDisposable.Dispose](https://learn.microsoft.com/zh-cn/dotnet/api/system.idisposable.dispose) 以确定性地释放非托管资源时，终止会启用对非托管资源的非确定性释放。

#### 关于IDisposable

首先是对于 `Dispose` 的实现，这是需要在继承IDisposable类后，再于类中实现的。

特别的，当我们实现了 `IDisposable.Dispose` 后：

```C#
public class MyClass:IDisposable {
    public void Dispose(){}
}
```

我们就可以在代码中以using的语法糖调用：

```C#
using (var m = new Myclass()) {}
```

这等于：

```C#
MyClass mc = null;
try {
    mc = new MyClass();
    ...
} finally {
    if (mc != null){
        mc.Dispose();
    }
}
```

让我们先把安全句柄放在一边，如果我们现在要完整地实现剩下两个方法的话，我们需要这些：

1. 实现Dispose方法；
2. 提取一个受保护的Dispose虚方法，在该方法中实现具体的释放资源的逻辑；
3. 添加析构函数；
4. 添加一个私有的bool类型的字段，作为释放资源的标记

我们可以得到这样的一个模板：

```C#
public class MyClass : IDisposable {
    private IntPtr unManagedResource { get; set; } = Marshal.AllocHGlobal(100); // 模拟一个非托管资源 
    public Random ManagedResource { get; set; } = new Random(); // 模拟一个托管资源 

    private bool disposed; // 释放标记，在真正执行释放操作之前，先检查该标记，防止反复执行释放操作导致的错误

    /// <summary> 这是一个析构函数，C#中，析构函数就是一个终结器，它会在没有显式调用Dispose方法时自动执行 </summary>
    ~MyClass() {
        Dispose(false); //必须为false
    }

    /// <summary> 执行与释放或重置非托管资源关联的应用程序定义的任务。</summary>
    public void Dispose() {
        Dispose(true); //必须为true
        GC.SuppressFinalize(this); //通知垃圾回收器不再调用终结器
    }

    /// <summary> 非必需的，只是为了更符合其他语言的规范，如C++、java </summary>
    public void Close() {
        Dispose();
    }

    /// <summary> 非密封类可重写的Dispose方法，方便子类继承时可重写 </summary>
    protected virtual void Dispose(bool disposing) {
        if (disposed) {
            return;
        }

        // 清理托管资源
        if (disposing) {
            if (ManagedResource != null) {
                ManagedResource = null; // 或者是: ManagedResource.Dispose();
            }
        }

        // 清理非托管资源
        if (unManagedResource != IntPtr.Zero) {
            Marshal.FreeHGlobal(unManagedResource);
            unManagedResource = IntPtr.Zero;
        }

        // 告诉自己已经被释放
        disposed = true;
    }
}
```

这个实现的逻辑大体上是这样的：

1. 我们模拟了两种不同类型的资源，它们最终都可以在Dispose中接受处理。对于非托管资源来说，这使得我们清理非托管资源占用的内存；对于托管资源而言，我们避免了GC的介入以避免GC影响性能

2. 定义析构函数，C#中的析构函数就是所谓的"终结器"。**如果我们忘记了显式调用Dispose方法，垃圾回收器在扫描内存的时候，会作为释放资源的一种补救措施。**虽然MS的文档中明确指出不建议我们这样做。

   > 析构方法是在C++中的一种说法，因为终结器和析构方法两者特点很像，为了沿袭C++的叫法，称之为析构方法也没有什么不妥，但它们又不完全一致，所以微软后来又确定它叫终结器。

   基于这样的情况，当我们执行 `Dispose(true)` 后，终结器当然就没有必要再运行了。因此，我们使用 `SuppressFinalize` 方法来阻止GC对该终结器的执行

3. 最后是 `Dispose(bool)` 方法重载。在重载中，`disposing` 参数是一个 `Boolean`，它指示方法调用是来自 `Dispose` 方法（其值为 `true`）还是来自终结器（其值为 `false`）。基于这样的设计，我们需要针对不同的输入执行不同的代码块：

   - 如果对象已释放，则为条件返回块。
   - 释放非托管资源的块。 无论 `disposing` 参数的值如何，都会执行此块。
   - 释放托管资源的条件块。 如果 `disposing` 的值为 `true`，则执行此块。 它释放的托管资源可包括：
     - **实现 [IDisposable](https://learn.microsoft.com/zh-cn/dotnet/api/system.idisposable) 的托管对象。** 可用于调用其 [Dispose](https://learn.microsoft.com/zh-cn/dotnet/api/system.idisposable.dispose) 实现（级联释放）的条件块。 如果你已使用 [System.Runtime.InteropServices.SafeHandle](https://learn.microsoft.com/zh-cn/dotnet/api/system.runtime.interopservices.safehandle) 的派生类来包装非托管资源，则应在此处调用 [SafeHandle.Dispose()](https://learn.microsoft.com/zh-cn/dotnet/api/system.runtime.interopservices.safehandle.dispose#system-runtime-interopservices-safehandle-dispose) 实现。
     - **占用大量内存或使用短缺资源的托管对象。** 将大型托管对象引用分配到 `null`，使它们更有可能无法访问。 相比以非确定性方式回收它们，这样做释放的速度更快。

#### 实现Dispose方法

在上面的模板中，我们注意到了，实现自 `Idisposable` 接口的 `Dispose` 方法并没有做实际的清理工作，而是和 `~MyClass()` 一样，以不同的参数调用了 `protected virtual void Dispose(bool disposing)` 函数。这个虚函数是我们 `Dispose` 方法的核心。

##### 为什么是虚函数？

首先，简单介绍一下虚函数：

一个实例方法声明前带有virtual关键字，那么这个方法就是虚方法。**虚方法与非虚方法的最大不同是，虚方法的实现可以由派生类通过方法的重写所取代。**

[virtual - C# 参考 | Microsoft Learn](https://learn.microsoft.com/zh-cn/dotnet/csharp/language-reference/keywords/virtual)

那么，之所以是虚方法，就是考虑到它如果被其他类继承时，子类也需要实现 `Dispose` 模式，这个虚方法可以提醒子类，清理的时候要注意到父类的清理工作。在子类重载了该方法后，调用 `base.Dispose` 方法继续清理父类中可能存在的非托管资源。

#### 非托管资源的Dispose部分以及Finalize相关

非托管资源的Dispose调用顺序大概如下：

myClass.Dispose() => myClass.Dispose(true) => myUnmanagedResources.Dispose() => myUnmanagedResources.Dispose(true) => 外部资源的清理函数

其中，最后的清理函数由我们在载入dll等资源时获取到，我们只需要调用即可

这其中，我们至少实现了Dispose Pattern两次，分别位于两个类中

特别的，如果我们没有主动地进行最初的 myClass.Dispose() 调用的话，当下次GC启动且对myClass实例内存的引用计数为零时，就会帮我们自动调用 `~myClass` 这个终结器。这里我们可以详细了解一下这个终结器：

> `终结器` 内部大体上以如下方式工作
>
> 1. `new`新对象时，如果该对象的类型定义了`Finalize`方法，那么在该类型的实例构造器被调用之前，会在一个所谓`Finalization Queue` 的队列中存储一个指向该对象的指针，该列表由GC内部控制。
> 2. 当可终结对象被回收时，即，成为垃圾时，这个对象会被分离出来，并将指向它的指针从 `Finalization Queue` 移动到 `freachable Queue` 中，该队列亦由GC内部控制，而这个过程被称为是对象的复生（Resurrection）。
> 3. CLR会启用一个特殊的高优先级线程来专门调用`Finalze`方法。`freachable` 队列为空时，该线程将睡眠；但一旦队列中有记录项出现，线程就会被唤醒，将每一项都从 `freachable` 队列中移除，并调用每个对象的`Finalize`方法。

总之就是，当我们没有显式调用Dispose时，终结器就会被唤醒，执行其中的内容并完成非托管资源的清除。这个过程是很漫长的过程，并且会造成较大的性能开销。因此，主动地去进行Dispose总是更好的。另外，也不要使用空的析构函数。如果类包含析构函数，则 Finalize 队列中则会创建一个项。当调用析构函数时，将调用垃圾回收器（GC）来处理该队列。如果析构函数为空，只会导致不必要的性能损失。

对于这个终结过程，我们有两个常用的控制函数：`ReRegisterForFinalize()` 和 `SuppressFinalize()`，前者适用于当某一个类成为垃圾时，将指向该对象的指针重新存入 `Finialization Queue` 中，后者则用来抑制 `Finalize` 方法的进行，这也就是为什么我们在主动调用的 `Dispose()` 中会主动使用这个方法——因为主动释放之后就没必要再运行 `Finalize`了。

特别的，如果在 `Finalize` 方法中，配合 `ReRegisterForFinalize()` 的使用，会使在 `Finalization Queue` 中的对象可以反复执行复生过程，这样就形成了一个在堆上永远不会死去的对象。

在示例代码中，我们直接就在myClass中调用了外部的内存清理函数，并以 `unManagedResource != IntPtr.Zero` 检验是否完成了目的。

> [c# - Is IntPtr.Zero equivalent to null? - Stack Overflow](https://stackoverflow.com/questions/1456861/is-intptr-zero-equivalent-to-null)

#### 托管资源的Dispose部分

那为什么我们还需要在 `Dispose` 中处理托管资源，GC不是会帮我们完成吗？

是的，C#是没有手段去自行释放托管资源的，我们所谓的对托管资源的主动释放，实际上也就是调用 `gc.collect` 罢了，但是有这些要点是需要强调的：

- 对于C#这种存在GC机制的语言，我们实际上不应该总是焦虑内存释放，除非我们实在需要。默认的GC机制只有一个比较大的弊端，那就是我们不能确定，GC到底会在什么时候工作。那么这就出现了我们实在需要主动要求GC执行的理由：当前场景下内存需求剧烈，我们希望GC现在就执行。
- 当某一个对象不再使用时，可以考虑将对象置为null。这不会加速下一次GC的执行，但是会保证这个资源会因为没有被指向而最终一定导致回收。特别是静态变量，这是我们特别需要注意的：当一个类中存在静态字段或属性时，其实例的静态字段或属性，是不会因为没有对象来指向它就会被GC回收的——静态内容会脱离这个资源，并总是会维持自己的存在，因为类型的静态字段一旦被创建，就被作为“根”存在，基本上不参与GC，所以GC始终不会认为它是个垃圾。清除这些滞留内容的办法，是将对应的内容明确地设置为null，这种行为的典型应用是缓存的过期策略。

#### 番外：Close

> 首先，Dispose和Close基本上应该是一样的。Close是为了那些不熟悉Dispose的开发者设计的。因为基本上所有的developer都知道Close是干吗的(特别是对于那些有C++背景的developer) ... 在.net的framework里，Close()被设计成public的，并且在Close()里面call被隐藏的Dispose()；Dispose()去call另一个virtual的Dispose(bool)函数。所以如果你从这个class继承，你就必须实现Dispose(bool)方法 ... 用着call Close()的时候就会call到你重载的那个Dispose(bool)方法去释放资源。

### SafeHandle

MSDN上对SafeHandle的解释算是比较明确的：

> [SafeHandle](https://learn.microsoft.com/zh-cn/dotnet/api/system.runtime.interopservices.safehandle) 类提供了终结器，因此你无需自行编写。

关于SafeHandle的各种形式，可以查阅：[Microsoft.Win32.SafeHandles 命名空间 | Microsoft Learn](https://learn.microsoft.com/zh-cn/dotnet/api/microsoft.win32.safehandles?view=net-7.0)

>  [IDisposable.Dispose](https://learn.microsoft.com/zh-cn/dotnet/api/system.idisposable.dispose) 的派生类请提供以下内容：
>
> - `protected override void Dispose(bool)` 方法，用于替代基类方法并执行派生类的实际清理。 此方法还必须调用 `base.Dispose(bool)`（Visual Basic 中的 `MyBase.Dispose(bool)`）方法，并将释放状态（`bool disposing` 参数）作为参数传递给它。
> - 从包装非托管资源的 [SafeHandle](https://learn.microsoft.com/zh-cn/dotnet/api/system.runtime.interopservices.safehandle) 派生的类（推荐），或对 [Object.Finalize](https://learn.microsoft.com/zh-cn/dotnet/api/system.object.finalize) 方法的重写。 [SafeHandle](https://learn.microsoft.com/zh-cn/dotnet/api/system.runtime.interopservices.safehandle) 类提供了一个使你无需编写代码的终结器。 如果你提供了终结器，它必须调用具有 `false` 参数的 `Dispose(bool)` 重载。
>
> 以下是一个常规模式的示例，该模式用于实现使用安全句柄的派生类的释放模式：
>
> ```C#
> using Microsoft.Win32.SafeHandles;
> using System;
> using System.Runtime.InteropServices;
> 
> public class DerivedClassWithSafeHandle : BaseClassWithSafeHandle{
>     private bool _disposedValue; // To detect redundant calls
>     private SafeHandle? _safeHandle = new SafeFileHandle(IntPtr.Zero, true); // Instantiate a SafeHandle instance.
> 
>     // Protected implementation of Dispose pattern.
>     protected override void Dispose(bool disposing){
>         if (!_disposedValue){
>             if (disposing){
>                 _safeHandle?.Dispose();
>                 _safeHandle = null;
>             }
> 
>             _disposedValue = true;
>         }
> 
>         // Call base class implementation.
>         base.Dispose(disposing);
>     }
> }
> ```

## Unity GC

> [Unity - Manual: Garbage collector overview (unity3d.com)](https://docs.unity3d.com/Manual/performance-garbage-collector.html)

> 是的，Unity使用了一种与.Net不同的GC机制：保守式的Boehm GC，这里有一段小故事：[c# - Unity's garbage collector - Why non-generational and non-compacting? - Stack Overflow](https://stackoverflow.com/questions/46574407/unitys-garbage-collector-why-non-generational-and-non-compacting)
>
> 简而言之，Unity使用，并且仍在使用Boehm GC是因为当初世界上对C#有两种实现，而unity选择了民间的而非MS的，这在之后成为了一种历史问题。

### Unity Memory 的内容划分

> Unity uses three memory management layers to handle memory in your application:
>
> - [Managed memory](https://docs.unity3d.com/cn/current/Manual/performance-memory-overview.html#managed-memory): A controlled memory layer that uses a managed heap and a [garbage collector](https://en.wikipedia.org/wiki/Garbage_collection_(computer_science)) to automatically allocate and assign memory.
> - [C# unmanaged memory](https://docs.unity3d.com/cn/current/Manual/performance-memory-overview.html#unmanaged-memory): A layer of memory management that you can use in conjunction with the Unity Collections namespace and package. This memory type is called “unmanaged” because it doesn’t use a [garbage collector](https://docs.unity3d.com/cn/current/Manual/performance-garbage-collector.html) to manage unused parts of memory.
> - [Native memory](https://docs.unity3d.com/cn/current/Manual/performance-memory-overview.html#native-memory): C++ memory that Unity uses to run the engine. In most situations, this memory is inaccessible to Unity users, but it’s useful to be aware of it if you want to fine-tune certain aspects of the performance of your application.
>
> Using **managed memory** in Unity is the easiest way to manage the memory in your application; but it has some disadvantages. The garbage collector is convenient to use, but it’s also unpredictable in how it releases and allocates memory, which might lead to performance issues such as stuttering, which happens when the garbage collector has to stop to release and allocate memory. **To work around this unpredictability, you can use the [C# unmanaged memory layer](https://docs.unity3d.com/cn/current/Manual/performance-memory-overview.html#unmanaged-memory).**
>
> The **C# unmanaged memory layer** allows you to access the native memory layer to fine-tune memory allocations, with the convenience of writing C# code.You can use the `Unity.Collections`namespace (including [NativeArray](https://docs.unity3d.com/cn/current/ScriptReference/Unity.Collections.NativeArray_1.html)) in the Unity core API, and the data structures in the [Unity Collections package](https://docs.unity3d.com/Packages/com.unity.collections@latest) to access C# unmanaged memory. If you use Unity’s C# Job system, or Burst, you must use C# unmanaged memory. For more information about this, see the documentation on the [Job system](https://docs.unity3d.com/cn/current/Manual/JobSystem.html) and [Burst](http://docs.unity3d.com/Packages/com.unity.burst@latest).
>
> [Memory in Unity - Unity 手册 (unity3d.com)](https://docs.unity3d.com/cn/current/Manual/performance-memory-overview.html)

> 在Unity中，内存分配的具体过程也是可以定义的，这涉及到多种内存分配算法，但却并非今天的重点。[对应手册的 Memory allocator customization 内容](https://docs.unity3d.com/cn/current/Manual/memory-allocator-customization.html)

### 触发GC工作实例

以下是一个触发GC的具体实例，来自Unity Docs

![A quantity of memory. Marked A on the diagram is some free memory.](https://docs.unity3d.com/cn/current/uploads/Main/managed-heap.jpg)
A quantity of memory. Marked A on the diagram is some free memory.

In the above diagram, the blue box represents a quantity of memory that Unity allocates to the managed heap. The white boxes within it represent data values that Unity stores within the managed heap’s memory space. When additional data values are needed, Unity allocates them free space from the managed heap (annotated **A**).

![A quantity of memory, with some objects released represented by grey dashed lines.](https://docs.unity3d.com/cn/current/uploads/Main/managed-heap-removed-objects.jpg)
A quantity of memory, with some objects released represented by grey dashed lines.

And when Unity releases an object, **the memory that the object occupied is freed up. However, the free space doesn’t become part of a single large pool of “free memory.”**

The objects on either side of the released object might still be in use. Because of this, the freed space is a “gap” between other segments of memory. Unity can only use this gap to store data of identical or lesser size than the released object.

This situation is called **memory fragmentation**(即：**内存碎片**). This happens when there is a large amount of memory available in the heap, but it is only available in the “gaps” between objects. This means that even though there is enough total space for a large memory allocation, the managed heap can’t find a large enough single block of contiguous memory to assign to the allocation.

![The object annotated A, is the new object needed to be added to the heap. The items annotated B are the memory space that the released objects took up, plus the free, unreserved memory. Even though there is enough total free space, because there isnt enough contiguous space, the memory for the new object annotated A cant fit on the heap, and the garbage collector must run.](https://docs.unity3d.com/cn/current/uploads/Main/managed-heap-fragmentation.jpg)
The object annotated A, is the new object needed to be added to the heap. The items annotated B are the memory space that the released objects took up, plus the free, unreserved memory. **Even though there is enough total free space, because there isn’t enough contiguous space, the memory for the new object annotated A can’t fit on the heap, and the garbage collector must run.**

If a large object is allocated and there is insufficient contiguous free space to accommodate it, as illustrated above, the Unity memory manager performs two operations:

- First, the garbage collector runs, if it hasn’t already done so. This attempts to free up enough space to fulfill the allocation request.
- **If, after the garbage collector runs, there is still not enough contiguous space to fit the requested amount of memory, the heap must expand.** The specific amount that the heap expands is platform-dependent; however, on most platforms, when the heap expands, it expands by double the amount of the previous expansion.

### Unity GC的执行方式

> [Managed memory - Unity 手册 (unity3d.com)](https://docs.unity3d.com/cn/current/Manual/performance-managed-memory.html)

和.Net一样，只要当脚本尝试在托管堆上进行分配但又没有足够的可用堆内存来容纳分配时，Unity 就会运行垃圾收集器。当垃圾收集器运行时，它会检查堆中的所有对象，并将应用程序不再引用的任何对象标记为删除。然后 Unity 会删除未引用的对象，从而释放内存。

我们可以通过下面的API来fine-tune control over the automatic garbage collector:

- `System.GC.Collect`: Performs a full, blocking garbage collection.
- [GarbageCollector.Mode.Disabled](https://docs.unity3d.com/cn/current/ScriptReference/Scripting.GarbageCollector.Mode.Disabled.html): Fully disables the garbage collector. Using `System.Gc.Collect` in this mode has no effect.
- [GarbageCollector.Mode.Manual](https://docs.unity3d.com/cn/current/ScriptReference/Scripting.GarbageCollector.Mode.Manual.html): Disables automatic invocations of the garbage collector, but you can still use `System.GC.Collect` to run a full collection.
- `GarbageCollection.CollectIncremental`: Runs the garbage collector [incrementally](https://docs.unity3d.com/cn/current/Manual/performance-incremental-garbage-collection.html).

[Scripting.GarbageCollector - Unity 脚本 API (unity3d.com)](https://docs.unity3d.com/cn/current/ScriptReference/Scripting.GarbageCollector.html)

而前面也已经说过，Unity中的GC是Boehm GC，它有以下特点：

- 实现方式与.Net大体上类似：Unity中，GC是Mark-Sweep方法的实现，它和.Net很像，也使用了类似根的概念。其呈现的大致流程如下：

  > To determine which heap blocks are no longer in use, the garbage collector searches through all active reference variables and marks the blocks of memory that they refer to as “live.” At the end of the search, the garbage collector considers any space between the “live” blocks empty and marks them for use for subsequent allocations. The process of locating and freeing up unused memory is called **garbage collection** (GC).

  但是需要注意的是，它无法分辨一个Root 是一个指针还是非指针，所以会对所有当做一个指针做尝试，所以无法使用Copying算法。在BOEHM GC为了增加分辨能力其会根据已经分配过的内存区间做比较，增加指针黑名单，来增加分辨率。而准确式GC就是GC可以直接分辨出这是个指针还是非指针。而且可以使用Copying算法优化内存的使用。

- 与.Net类似的另一点是，它会在需要进行GC时占用主线程来进行遍历-标记-垃圾回收的过程，然后在归还主线程控制权。这会导致帧数的突然下降，产生卡顿，由于这是占用主线程进行大量工作的行为，所以当我们优化GC时，实际上优化的是CPU时间，即GC占用主线程的时间减少了而非实际使用的内存。而就具体的运行过程而言，它又有三种工作模式：

  - 增量式[Incremental garbage collection](https://docs.unity3d.com/cn/current/Manual/performance-incremental-garbage-collection.html): Enabled by default (**Project Settings > Player > Configuration**), this mode spreads out the process of garbage collection over multiple frames.

    - 这种模式是目前(2021.3)的默认的GC工作模式

      > By default, Unity uses it in incremental mode, which means that the garbage collector splits up its workload over multiple frames, instead of stopping the main CPU thread ([stop-the-world garbage collection](https://en.wikipedia.org/wiki/Tracing_garbage_collection#Stop-the-world_vs._incremental_vs._concurrent)) to process all objects on the managed heap. This means that Unity makes shorter interruptions to your application’s execution, instead of one long interruption to let the garbage collector process the objects on the managed heap.

    - 这种模式并不适用于存在大量引用快速变化的情况：在两次增量式GC之间，如果对象的引用发生变化会导致前一次GC的标记失效，需要重新进行遍历标记，而最糟的情况会退化为每一帧都是普通的非分代GC，而实时上要更糟糕一些，增量gc本就会需要运行一些额外的、用来通知gc重新扫描变化的代码。这时，主动地使用gc的非增量模式会是更好的选择。

  - [Incremental garbage collection disabled](https://docs.unity3d.com/cn/current/Manual/performance-incremental-garbage-collection.html#disabling): If you disable the **Incremental GC** Player Setting, the garbage collector stops running your application to inspect and process objects on the heap.

  - [Disable automatic garbage collection](https://docs.unity3d.com/cn/current/Manual/performance-disabling-garbage-collection.html): Use the [GarbageCollector.GCMode](https://docs.unity3d.com/cn/current/ScriptReference/Scripting.GarbageCollector.GCMode.html) API to take full control of when Unity should run the garbage collector.

    - 需要说明的是，关闭GC这个操作完全是可行的，并且是具有一定意义的，比如：当我们载入了一个关卡之后，我们就可以考虑关闭GC来保证GC不会拖累CPU时间，而当我们在进行Scene Change时，我们又可以手动调用System.GC.Collect来主动回收上一个Level的垃圾。

- 无内存压缩：由于没有内存压缩，程序会有内存碎片的问题，其对内存利用率在比起.Net的实现上是更低的，并且应该会随着持续运行而导致碎片不断增多。但是换个思路，由于这是非压缩式的实现，所以GC在执行时导致的卡顿现象也相对较轻。前者可能导致平均帧率降低，后者也许反而会提升1%Low帧之类的东西。

- 无代际划分，每次GC的执行都是一次Full GC，但存在一种渐进的GC模式即增量式GC模式，具体见上方工作模式。

### Managed heap expansion considerations

The unexpected expansion of the heap can be problematic. Unity’s garbage collection strategy tends to fragment memory more often. You should be aware of the following:

- Unity doesn’t release the memory allocated to the managed heap when it expands regularly; instead, it retains the expanded heap, even if a large section of it is empty. This is to prevent the need to re-expand the heap if further large allocations occur.
- On most platforms, Unity eventually releases the memory that the empty portions of the managed heap uses back to the operating system. The interval at which this happens isn’t guaranteed and is unreliable.

### 查看GC运行状态

> Unity has the following tools to keep track of memory allocations:
>
> - [Unity Profiler’s CPU Usage module](https://docs.unity3d.com/cn/current/Manual/ProfilerCPU.html): Provides details of the **GC Alloc** per frame
> - [Unity Profiler’s Memory module](https://docs.unity3d.com/cn/current/Manual/ProfilerMemory.html): Provides high-level memory usage frame by frame
> - [The Memory Profiler package](https://docs.unity3d.com/Packages/com.unity.memoryprofiler@latest): A separate Unity package which provides detailed information about memory usage during specific frames in your application

### GC的优化之道 - Garbage Collector best practices

> 优化GC的核心不在GC本身——在于**减少垃圾的出现**。

> Automatic memory management allows you to write code quickly and easily, and with few errors. However, this convenience might have performance implications. To optimize your code for performance, you must avoid situations where your application triggers the [garbage collector](https://docs.unity3d.com/cn/current/Manual/performance-garbage-collector.html) a lot. This section outlines some common issues and workflows that affect when your application triggers the garbage collector.
>
> - 临时内存申请 - [Temporary allocations](https://docs.unity3d.com/cn/current/Manual/performance-garbage-collection-best-practices.html#tempalloc)
> - 复用对象池 - [Reusable object pools](https://docs.unity3d.com/cn/current/Manual/performance-garbage-collection-best-practices.html#reusablepools)
> - 重复连接字符串 - [Repeated string concatenation](https://docs.unity3d.com/cn/current/Manual/performance-garbage-collection-best-practices.html#repeatedstring)
> - 数组作方法返回值 - [Method returning an array value](https://docs.unity3d.com/cn/current/Manual/performance-garbage-collection-best-practices.html#methodarray)
> - [Collection and array reuse](https://docs.unity3d.com/cn/current/Manual/performance-garbage-collection-best-practices.html#collectionreuse)
> - 闭包与匿名方法 - [Closures and anonymous methods](https://docs.unity3d.com/cn/current/Manual/performance-garbage-collection-best-practices.html#closures)
> - 装箱 - [Boxing](https://docs.unity3d.com/cn/current/Manual/performance-garbage-collection-best-practices.html#boxing)
> - [Array-valued Unity APIs](https://docs.unity3d.com/cn/current/Manual/performance-garbage-collection-best-practices.html#arrayapis)
> - 空数组复用 - [Empty array reuse](https://docs.unity3d.com/cn/current/Manual/performance-garbage-collection-best-practices.html#emptyarray)

#### Temporary allocations

> It’s common for an application to allocate temporary data to the [managed heap](https://docs.unity3d.com/cn/current/Manual/performance-managed-memory.html#managed-heap) in each frame; however, this can affect the performance of the application. 

> 这实际上是一个积少成多的数学问题，优化GC的第一步是，自己少产生点垃圾。让我们以60帧为游戏的目标帧率，那一分钟内就会 `Update` 3600次。1KB * 3600 ≈ 3.6MB，放在.Net，这已经会引发GC了。

> - **Program Continue to Allocateing Memory:** If a program allocates one kilobyte (1KB) of temporary memory each frame, and it runs at 60 frames per second, then it must allocate 60 kilobytes of temporary memory per second. Over the course of a minute, this adds up to 3.6 megabytes of memory available to the garbage collector.
> - **Invoking the garbage collector:** Invoking the garbage collector once per second has a negative effect on performance. If the garbage collector only runs once per minute, it has to clean up 3.6 megabytes spread across thousands of individual allocations, which might result in significant garbage collection times.
> - **Loading operations:** Loading operations have an impact on performance. If your application generates a lot of temporary objects during a heavy asset-loading operation, and Unity references those objects until the operation completes, then the garbage collector can’t release those temporary objects. This means that the managed heap needs to expand, even though Unity releases a lot of the objects that it contains a short time later.
>
> To get around this, you should try to reduce the amount of frequently managed heap allocations as possible: ideally to 0 bytes per frame, or as close to zero as you can get.

#### Reusable object pools

> There are a lot of cases where you can reduce the number of times that your application creates and destroys objects, to avoid generating garbage. There are certain types of objects in games, such as projectiles, which might appear over and over again even though only a small number are ever in play at once. In cases like this, you can reuse the objects, rather than destroy old ones and replace them with new ones.
>
> 即，创建一个会在游戏中大量刷新的物体的缓冲池，每次需要的时候就可以直接复用这部分区域，这里，Unity官方举了一个枪械开火的示例：
>
> For example, it’s not optimal to instantiate a new projectile object from a Prefab every time one is fired. Instead, you can calculate the maximum number of projectiles that could ever exist simultaneously during gameplay, and instantiate an array of objects of the correct size when the game first enters the gameplay scene. To do this:
>
> - Start with all the projectile GameObjects set to being inactive.
> - When a projectile is fired, search through the array to find the first inactive projectile in the array, move it to the required position and set the GameObject to be active.
> - When the projectile is destroyed, set the GameObject to inactive again.
>
> You can use the [ObjectPool](https://docs.unity3d.com/cn/current/ScriptReference/Pool.ObjectPool_1.html) class, which provides an implementation of this reusable object pool technique.
>
> The code below shows a simple implementation of a stack-based object pool. You might find it useful to refer to if you’re using an older version of Unity which doesn’t contain the ObjectPool API, or if you’d like to see an example of how a custom object pool might be implemented.
>
> ```csharp
> using System.Collections.Generic;
> using UnityEngine;
> 
> public class ExampleObjectPool : MonoBehaviour {
> 
>    public GameObject PrefabToPool;
>    public int MaxPoolSize = 10;
>   
>    private Stack<GameObject> inactiveObjects = new Stack<GameObject>();
>   
>    void Start() {
>        if (PrefabToPool != null) {
>            for (int i = 0; i < MaxPoolSize; ++i) {
>                var newObj = Instantiate(PrefabToPool);
>                newObj.SetActive(false);
>                inactiveObjects.Push(newObj);
>            }
>        }
>    }
> 
>    public GameObject GetObjectFromPool() {
>        while (inactiveObjects.Count > 0) {
>            var obj = inactiveObjects.Pop();
>           
>            if (obj != null) {
>                obj.SetActive(true);
>                return obj;
>            }
>            else {
>                Debug.LogWarning("Found a null object in the pool. Has some code outside the pool destroyed it?");
>            }
>        }
>       
>        Debug.LogError("All pooled objects are already in use or have been destroyed");
>        return null;
>    }
>   
>    public void ReturnObjectToPool(GameObject objectToDeactivate) {
>        if (objectToDeactivate != null) {
>            objectToDeactivate.SetActive(false);
>            inactiveObjects.Push(objectToDeactivate);
>        }
>    }
> }
> ```

#### Repeated string concatenation

> Strings in C# are immutable reference types. A reference type means that Unity allocates them on the managed heap and are subject to garbage collection. Immutable means that once a string has been created, it can’t be changed; **any attempt to modify the string results in an entirely new string.** For this reason, you should avoid creating temporary strings wherever possible.

一个例子是：

```C#
// Bad C# script example: repeated string concatenations create lots of temporary strings.

public class ExampleScript : MonoBehaviour {
    string ConcatExample(string[] stringArray) {
        string result = "";

        for (int i = 0; i < stringArray.Length; i++) {
            result += stringArray[i];
        }
        return result;
    }
}
```

如果输入`stringArray == {'A', 'B', 'C', 'D', 'E'}`，则会出现数个中间的临时的变量：`A, AB, ABC, ABCD`，要是这也是在Update中调用呢？就这里而言，我们可以做的包括：使用 `StringBuilder`，使用全局变量，tostring的对象直接占用独立的Text以避免与前缀进行拼接，采用条件限制字符串拼接的进行等

#### Method returning an array value

> Sometimes it might be convenient to write a method that creates a new array, fills the array with values and then returns it. However, if this method is called repeatedly, then new memory gets allocated each time.

**One way you can avoid allocating memory every time is to make use of the fact that an array is a reference type. You can modify an array that’s passed into a method as a parameter, and the results remain after the method returns.**

```csharp
// Bad C# script example: Every time the RandomList method is called it allocates a new array
public class ExampleScript : MonoBehaviour {
    float[] RandomList(int numElements) {
        var result = new float[numElements];
        for (int i = 0; i < numElements; i++) {
            result[i] = Random.value;
        }
        return result;
    }
}

// Good C# script example: This version of method is passed an array to fill with random values. The array can be cached and re-used to avoid repeated temporary allocations
public class ExampleScript : MonoBehaviour {
    void RandomList(float[] arrayToFill) {
        for (int i = 0; i < arrayToFill.Length; i++) {
            arrayToFill[i] = Random.value;
        }
    }
}
```

在上方，我们用一个数组对象作为参数而非数组长度，这样函数体内便不需要创建新的数组，避免了内存分配与垃圾产生。And the array can then be re-used and re-filled with random numbers the next time this method is called without any new allocations on the managed heap.

#### Collection and array reuse

> When you use arrays or classes from the [`System.Collection`](https://docs.microsoft.com/en-us/dotnet/api/system.collections.generic?view=net-6.0) namespace (for example, Lists or Dictionaries), it’s efficient to reuse or pool the allocated collection or array. Collection classes expose a Clear method, which eliminates a collection’s values but doesn’t release the memory allocated to the collection.

Clear方法消除集合的值，但不会释放分配给集合的内存。我们可以直接复用这部分内存——只要它还是可引用的，即，将List等于 `System.Collection` 的类型的变量提升到包含的类中，这样就不需要每帧分配一个新的 List

#### Closures and anonymous methods

> In general, you should avoid closures in C# whenever possible. You should minimize the use of anonymous methods and method references in performance-sensitive code, and especially in code that executes on a per-frame basis.

我们需要尽可能地避免使用闭包，同时还得小心我们的匿名函数构成了闭包。因为闭包除了函数指针还会将捕获的变量一起包装起来创建到堆上，相当于 new 了个对象，严重的情况下还会导致C#会创建一个匿名class并将其实例化。

> 在 IL2CPP 環境下任何使用匿名方法都會配置 Managed 的記憶體。使用 Mono 則不會發生。 此外，IL2CPP 會因為方法的 指向另一個方法的參數 宣告方法不同而會有不同的 Managed 記憶體配置量。如預期般地用閉包來呼叫會配置最多的記憶體。 
>
> 但違反一般人的直覺地，預先定義的方法在 IL2CPP 環境下作為參數傳遞時，配置的記憶體幾乎和閉包一樣多。而匿名方法在堆積上產生的臨時垃圾最少，少了一個以上的數量級。 
>
> 因此，如果專案打算用 IL2CPP 做為執行環境，有三點建議： 
>
> - 建議程式風格避免傳遞方法作為參數。 
> - 真的無法避免的話，採用匿名方法而非預先定義方法。 
> - 不管 Mono 或 IL2CPP 都避免使用閉包。 

#### Boxing

> **Boxing is one of the most common sources of unintended temporary memory allocations found in Unity projects.** It happens when a value-typed variable gets automatically converted to a reference type. This most often happens when passing primitive value-typed variables (such as int and float) to object-typed methods. You should avoid boxing when writing C# code for Unity.

装箱也会隐式地导致对象的创建。从而产生意想不到的垃圾，而且装箱操作往往很隐蔽，我们需要格外小心，比如：用枚举值当字典的key的时，各种字典操作会调用 Object.getHashCode 获取哈希值 ，该方法会导致装箱。

#### Array-valued Unity APIs

某些Unity内部函数在返回时，返回的对象是原对象的一份副本。例如，以下代码在每次循环迭代时，都不必要地创建了顶点数组的四个副本：每次访问 `.vertices` 属性时，这种复制都会执行。

```csharp
for(int i = 0; i < mesh.vertices.Length; i++){
    float x, y, z;
    x = mesh.vertices[i].x;
    y = mesh.vertices[i].y;
    z = mesh.vertices[i].z;
    // ...
    DoSomething(x, y, z);   
}
```

我们可以通过在第一次就cache返回的数组来解决。当然，如果我们能直接使用不会分配内存的API不是更好吗？

##### Alternative non-allocating APIs

Some Unity APIs have alternative versions that don’t cause memory allocations. You should use these when possible. The following table shows a small selection of common allocating APIs and their non-allocating alternatives. The list isn’t exhaustive, but should indicate the kind of APIs to watch out for.

| **Allocating API**                                           | **Non-allocating API alternative**                           |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [Physics.RaycastAll](https://docs.unity3d.com/cn/current/ScriptReference/Physics.RaycastAll.html) | [Physics.RaycastNonAlloc](https://docs.unity3d.com/cn/current/ScriptReference/Physics.RaycastNonAlloc.html) |
| [Animator.parameters](https://docs.unity3d.com/cn/current/ScriptReference/Animator-parameters.html) | [Animator.parameterCount](https://docs.unity3d.com/cn/current/ScriptReference/Animator-parameterCount.html) and [Animator.GetParameter](https://docs.unity3d.com/cn/current/ScriptReference/Animator.GetParameter.html) |
| [Renderer.sharedMaterials](https://docs.unity3d.com/cn/current/ScriptReference/Renderer-sharedMaterials.html) | [Renderer.GetSharedMaterials](https://docs.unity3d.com/cn/current/ScriptReference/Renderer.GetSharedMaterials.html) |
| ...                                                          | ...                                                          |

#### Empty array reuse

> Some development teams prefer to return empty arrays instead of null when an array-valued method needs to return an empty set. This coding pattern is common in a lot of managed languages, particularly C# and Java.
>
> In general, when returning a zero-length array from a method, it’s more efficient to return a pre-allocated static instance of the zero-length array than to repeatedly create empty arrays.

空数组（长度为0的数组）的创建事实上也会导致堆内存的分配。所以应该将其提前创建出来并复用。

上述问题的原因都是类似的，即大量地创建了短暂使用的对象（垃圾），基本上都可以通过将会反复使用的对象创建为非局部变量来解决（或者更进一步，使用所谓对象池的技术，基本原理是一样的）。有些地方就只能通过避免会造成垃圾产生的接口来解决。总之优化GC，核心在于**消灭垃圾**。

### 同.Net下GC的区别

> "保守式垃圾回收是一种通过近似方式识别和回收垃圾对象的方式。在保守式垃圾回收中，垃圾回收器并不直接访问对象的内部结构和引用关系，而是通过扫描内存中的数据块，识别出可能是指向对象的指针，并将其标记为活动对象。然后，垃圾回收器将从活动对象出发，递归地遍历和标记其他可达的对象，并回收那些未被标记为活动对象的内存。保守式垃圾回收不需要额外的内存开销来维护对象之间的引用关系，但可能会存在一定的误判，即将某些实际上是垃圾的对象错误地标记为活动对象。”
>
> 用大白话讲就是Boehm GC无法区分指针和非指针，这就可能由于误判导致有些已经可以释放的内存无法释放。

## References

- [自动内存管理 | Microsoft Learn](https://learn.microsoft.com/zh-cn/dotnet/standard/automatic-memory-management)
- [How IDisposable, Dispose, and Finalizers work in C# - YouTube](https://www.youtube.com/watch?v=e0G5X3bu6hY)
- [C#托管堆和垃圾回收（GC） - xiaoxiaotank - 博客园 (cnblogs.com)](https://www.cnblogs.com/xiaoxiaotank/p/11193745.html)
  - [[C#学习笔记\]类型对象指针和同步块索引 - knqiufan - 博客园 (cnblogs.com)](https://www.cnblogs.com/knqiufan/p/10475186.html)
- [C#托管堆和垃圾回收 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/141896678)
- [C#技术漫谈之垃圾回收机制(GC)_知识库_博客园 (cnblogs.com)](https://kb.cnblogs.com/page/106720/)
- [非托管类型 - C# 参考 | Microsoft Learn](https://learn.microsoft.com/zh-cn/dotnet/csharp/language-reference/builtin-types/unmanaged-types)
- [清理未托管资源 | Microsoft Learn](https://learn.microsoft.com/zh-cn/dotnet/standard/garbage-collection/unmanaged)
- [c# - What exactly are unmanaged resources? - Stack Overflow](https://stackoverflow.com/questions/3433197/what-exactly-are-unmanaged-resources)
- [温故之.NET托管资源与非托管资源 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/38180717)
- [C#与C++的析构函数 - Justin Liu - 博客园 (cnblogs.com)](https://www.cnblogs.com/liucfy/archive/2009/07/05/1517356.html)
- [深入理解C#中的IDisposable接口 - CharyGao - 博客园 (cnblogs.com)](https://www.cnblogs.com/Chary/p/11351429.html)
- [探讨C#中Dispose方法与Close方法的区别详解 - 董川民 (dongchuanmin.com)](https://www.dongchuanmin.com/csharp/3456.html)
- [Memory in Unity - Unity 手册 (unity3d.com)](https://docs.unity3d.com/cn/current/Manual/performance-memory-overview.html)
- [Unity - Manual: Garbage collector overview (unity3d.com)](https://docs.unity3d.com/Manual/performance-garbage-collector.html)
- [c# - Unity's garbage collector - Why non-generational and non-compacting? - Stack Overflow](https://stackoverflow.com/questions/46574407/unitys-garbage-collector-why-non-generational-and-non-compacting)
- [Unity内存问答笔记 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/61105374)
- [Mono中的BOEHM GC 原理学习（1） - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/49623707)
- [Unity GC 学习总结 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/265217138)
- [Unity Taiwan: 最佳實踐 - 了解Unity效能 - GC和Managed Heap](https://unitytaiwan.blogspot.com/2017/03/unity-gcmanaged-heap.html)
- [Unity 内存优化 | 新诸子 (xinzhuzi.github.io)](https://xinzhuzi.github.io/2020/05/08/Unity/Optimize/Unity内存优化/)

