关键词：`yeild关键字`，`yeild return`，`生成器函数`，`协程`

## Yeild in JS

[yield - JavaScript | MDN (mozilla.org)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/yield)

[function* - JavaScript | MDN (mozilla.org)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function*)

简单地说，Yeild可以在JS中暂停一个生成器函数，当我们再次呼叫这个生成器函数示例时，我们就可以该示例相关的一次结果

```js
function* foo(index) {
  while (index < 5) {
    yield index;
    index++;
  }
}

const iterator = foo(0);

console.log(iterator.next().value);
// Expected output: 0

console.log(iterator.next().value);
// Expected output: 1

console.log(iterator.next().value); // 2
console.log(iterator.next().value); // 3
console.log(iterator.next().value); // 4
console.log(iterator.next().value); // undefined
```

## Yeild in Python

[如何理解Python中的yield用法? - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/268605982)

事实上，Python中的yield与JS中的yield十分相近：同样是对规则的定义、会暂停执行函数，返回generator对象等等。

不同的是，Python不需要使用类似`function*`这样的单独的定义字符。

不过，在Python中我们可以直接使用for等方法来直接遍历整个生成器对象来快速获得全部的结果。

```python
def fab(max): 
    n, a, b = 0, 0, 1 
    while n < max: 
        yield b      # 使用 yield
        # print b 
        a, b = b, a + b 
        n = n + 1
 
for n in fab(5): 
    print n
```

> 生成器就是一个使用了yield关键字的函数，此函数可返回生成器对象

对了，我们在Python中还可以使用send方法来在实时调用中给生成器函数传入我们需要的值：

```python
import time

def fib(n):
    index = 0
    a = 0
    b = 1

    while index < n:
        sleep = yield b
        print('等待%s秒' %sleep)
        time.sleep(sleep)
        a,b = b, a+b
        index += 1

fib = fib(20)
print(fib.send(None))   # 效果等同于print(next(fib))
print(fib.send(2))
print(fib.send(3))
print(fib.send(4))

# -----output:-----
# 
```



~更多示例：[Python 中 yield 的用法理解 与 send() 函数对生成器赋值_怎样才能回到过去的博客-CSDN博客](https://blog.csdn.net/Z2572862506/article/details/128766574) ~

```python
def test():
    print("--------Test starting------")
    while True:
        print("Stop here, position NO.0")
        a = "a is unassigned!!!"
        print(a)
        print("Stop here, position NO.1")
        a = yield "--------Test over----------\n"
        print(a)
        print("Stop here, position NO.2")


t = test()
print("*" * 20 + "Next 1 Start" + "*" * 20 + "\n")
print(next(t))
print("*" * 20 + "Next 2 Start" + "*" * 20 + "\n")
print(next(t))
print("*" * 20 + "Next 3 Start" + "*" * 20 + "\n")
print(t.send("a is assigned!"))

# ********************Next 1 Start********************
# 
# --------Test starting------
# Stop here, position NO.0
# a is unassigned!!!
# Stop here, position NO.1
# --------Test over----------
# 
# ********************Next 2 Start********************
# 
# None
# Stop here, position NO.2
# Stop here, position NO.0
# a is unassigned!!!
# Stop here, position NO.1
# --------Test over----------
# 
# ********************Next 3 Start********************
#
# a is assigned!
# Stop here, position NO.2
# Stop here, position NO.0
# a is unassigned!!!
# Stop here, position NO.1
# --------Test over----------
#


# 规律总结：
# 1. 生成器函数实例化时，是不会执行的
# 2. 当调用next函数时，我们的函数才会开始执行，每次执行时只会执行到yeild处一次
# 	2.1. 初次执行时，函数是从头开始执行，执行到第一个yeild的地方放回yeild后面的值，如果没有外部传值，则yeild本身返回None
#	2.2. 后续执行时，函数从上一次yeild中断的地方重新开始执行直到下一个yeild出现（然后在那个地方再次中断）
# 3. 我们可以通过send向生成器传值，这是JS所没有的
```



### 小拓展

[深入理解Python 中的 yield from语法 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/267966140)

就Python而言，它还有更多的语法糖：`yeild from`:

>```python
>使用yield
>
># 字符串
>astr='ABC'
># 列表
>alist=[1,2,3]
># 字典
>adict={"name":"wangbm","age":18}
># 生成器
>agen=(i for i in range(4,8))
>
>def gen(*args, **kw):
>    for item in args:
>        for i in item:
>            yield i
>
>new_list=gen(astr, alist, adict, agen)
>print(list(new_list))
># ['A', 'B', 'C', 1, 2, 3, 'name', 'age', 4, 5, 6, 7]
>使用yield from
>
># 字符串
>astr='ABC'
># 列表
>alist=[1,2,3]
># 字典
>adict={"name":"wangbm","age":18}
># 生成器
>agen=(i for i in range(4,8))
>
>def gen(*args, **kw):
>    for item in args:
>        yield from item
>
>new_list=gen(astr, alist, adict, agen)
>print(list(new_list))
># ['A', 'B', 'C', 1, 2, 3, 'name', 'age', 4, 5, 6, 7]
>```

## Yeild in C# and Unity





