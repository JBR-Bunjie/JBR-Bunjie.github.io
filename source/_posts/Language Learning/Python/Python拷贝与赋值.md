# 深浅拷贝概念与使用

## Reference：

- [彻底理解Python中的"指针"_杰克小麻雀的博客-CSDN博客_python有指针吗](https://blog.csdn.net/yushuaigee/article/details/96745994)
- 

## deep copy and shallow copy

浅拷贝，指的是重新分配一块内存，创建一个新的对象，但里面的元素是原对象中各个子对象的引用

深拷贝，是指重新分配一块内存，创建一个新的对象，并且将原对象中的元素，以递归的方式，通过创建新的子对象拷贝到新对象中

## examples

### shallow copy 

#### code

```python
list1 = [[1, 2], (30, 40)]
list2 = list(list1)

list1.append(100)
print("list1:",list1)  # [[1, 2], (30, 40), 100]
print("list2:",list2)  # [[1, 2], (30, 40)]

list1[0].append(3)
print("list1:",list1)  # [[1, 2, 3], (30, 40), 100]
print("list2:",list2)  # [[1, 2, 3], (30, 40)]

list1[1] += (50, 60)
print("list1:",list1)  # [[1, 2, 3], (30, 40, 50, 60), 100]
print("list2:",list2)  # [[1, 2, 3], (30, 40)]
```

#### explanations

in this program, we initial a list: `list1` first which contains two elements: a list and a tuple.然后对 list1 执行浅拷贝，赋予 list2。因为浅拷贝里的元素是对原对象元素的引用，因此 list2 中的元素和 list1 指向同一个列表和元组对象。

next，`list1.append(100)`. 表示对 list1 的列表新增元素 100。这个操作不会对 list2 产生任何影响，因为 list2 和 list1 作为整体是两个不同的对象，并不共享内存地址。操作过后 list2 不变，list1 会发生改变。

then，`list1[0].append(3)` 表示对 list1 中的第一个列表新增元素 3。因为 list2 是 list1 的浅拷贝，list2 中的第一个元素和 list1 中的第一个元素，共同指向同一个列表，因此 list2 中的第一个列表也会相对应的新增元素 3。

at last,  `list1[1] += (50, 60)`，因为元组是不可变的，这里表示对 list1 中的第二个元组拼接，然后重新创建了一个新元组作为 list1 中的第二个元素，而 list2 中没有引用新元组，因此 list2 并不受影响。

### deep copy

#### code

```python
import copy
list1 = [[1, 2], (30, 40)]
list2 = copy.deepcopy(list1)

list1.append(100)
print("list1:",list1)  # [[1, 2], (30, 40), 100]
print("list2:",list2)  # [[1, 2], (30, 40)]

list1[0].append(3)
print("list1:",list1)  # [[1, 2, 3], (30, 40), 100]
print("list2:",list2)  # [[1, 2], (30, 40)]

list1[1] += (50, 60)
print("list1:",list1)  # [[1, 2, 3], (30, 40, 50, 60), 100]
print("list2:",list2)  # [[1, 2], (30, 40)]
```

#### explanation

just as the example code above, no matter how the list1 changes, list2 remains itself. 因为此时的 list1 和 list2 完全独立，没有任何联系。



## 特例

如果被深拷贝对象中存在指向自身的引用会怎么样？

```python
import copy
list1 = [1]
list1.append(list1)
print(list1)  # [1, [...]]

list2 = copy.deepcopy(list1)
print(list2)  # [1, [...]]
```

此例子中，列表 list1 中有指向自身的引用，因此 list1 是一个无限嵌套的列表。但是当深度拷贝 list1 到 list2 后，程序并没有出现栈溢出的现象。这是为什么呢？

->因为 deepcopy 会自动维护一个字典，记录已经拷贝的对象与其 ID。拷贝过程中，如果字典里已经存储了将要拷贝的对象，则会从字典直接返回。通过查看 deepcopy 函数实现的源码就会明白：



## 赋值与拷贝的关系

### 赋值

只是复制了新对象的引用，不会开辟新的内存空间。

### 拷贝

创建新对象，具体内容视拷贝类型而定（深浅拷贝）

所以赋值并不会产生一个独立的对象单独存在，只是将原有的数据块打上一个新标签，所以当其中一个标签被改变的时候，数据块就会发生变化，另一个标签也会随之改变。

请注意区分浅拷贝与赋值——关键在是否有新对象被创建

### 示例

赋值操作：

```python
a = [1, 2, 3]

b = a
# a.append(4)
# print(b) # [1, 2, 3, 4]
```

三种浅拷贝操作：

切片操作：
```python
lst1 = lst[:] 或者 lst1 = [each for each in lst]
```

工厂函数：
```python
lst1 = list(lst)
```

copy函数：
```python
lst1 = copy.copy(lst)
```

三种，都是浅拷贝

### 连续赋值

在python中是可以使用连续赋值的方式来一次为多个变量进行赋值的(请注意，仍然是“赋值”！)，比如：

```python
a = b = c = 1
a, b, c = 1, 1, 1
```

这些都可以完成变量的赋值，但是就有一个问题了，比如：

```python
a = 3
a, b = 1, a  # a = 1, b = 3
```

如果按照正常的思维逻辑，先进行a = 1，在进行b = a，最后b应该等于1，但是这里b应该等于3，因为**在连续赋值语句中等式右边其实都是局部变量，而不是真正的变量值本身**，因此，上面例子中右边的a，在python解析的时候，只是把变量a的指向的变量3赋给b，而不是a=1之后a的结果。这里有一个Leetcode里链表的例子：

> 假如要对一个链表进行翻转，就比如把1—>2->3->4转化为4->3->2->1

对于这个问题很简单，只要反转指针就可以了，假如链表结构为：

```python
class ListNode:
    def __init__(self, x):
        self.val = x
        self.next = None1234
```

我们可以用很简单的三行代码完成这个过程：

```python
def reverseList(self, head):
        L = ListNode(float("-inf"))
        while head:
            L.next, head.next, head = head, L.next, head.next
        return L.next12345
```

这里的L是指向一个新建的结点，因为python没有指针的概念，所以用一个额外的结点来代替头指针，这里的核心代码就是中间那一行三个变量的连续赋值，如果单独一句句来理解的话，最后肯定是想不通的，在这里，假设head结点是链表串’1->2->3->4’的头结点，先用新的L结点的next指针指向head的第一个结点‘1’，之后将L.next(第一次也就是空)赋给了head的next指针，之后再把head的next指针（注意，这里的next指针还是指向‘2’的，而不是空）赋给head，相当于next向前移一位，这一步相当于一个串变成了两个：

> L：‘-inf’->‘1’
>
> head：‘2’->‘3’->‘4’->‘5’