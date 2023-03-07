前言：在写入csv文件中，出现了乱码的问题。

解决：utf-8 改为utf-8-sig

区别如下：

1、”utf-8“ 是以字节为编码单元,它的字节顺序在所有系统中都是一样的,没有字节序问题,因此它不需要BOM,所以当用"utf-8"编码方式读取带有BOM的文件时,它会把BOM当做是文件内容来处理, 也就会发生类似上边的错误.

2、“uft-8-sig"中sig全拼为 signature 也就是"带有签名的utf-8”, 因此"utf-8-sig"读取带有BOM的"utf-8文件时"会把BOM单独处理,与文本内容隔离开,也是我们期望的结果.

```python
with open('data.csv', 'w',encoding='utf_8_sig') as fp:
```

utf-8保存的csv格式文件要让Excel正常打开的话，需要在文件最前面加入BOM(Byte order mark)。如果接收者收到以EF BB BF开头的字节流，就知道这是UTF-8编码了。

所以在write文件的内容数据之前，先write一下BOM。如下面代码

```python
FileOutputStream fos = new FileOutputStream(new File(this.csvFileAbsolutePath)); 
byte [] bs = { (byte)0xEF, (byte)0xBB, (byte)0xBF}; //UTF-8编码
fos.write(bs); 
fos.write(...);
fos.close(); 
```

这样添加了BOM的CSV文件用excel直接打开，是不会出现乱码的。

我当时遇到的问题是这样的。下载CSV文件，用excel打开，中文乱码，用atom，notepad++和 记事本打开，显示正常。 查资料发现是excel不能识别无BOM头的unicode文件问题，就是excel在打开CSV文件时默认用ASNI打开。 所以需要添加BOM头。

**BOM的含义**

　　BOM即Byte Order Mark字节序标记。BOM是为UTF-16和UTF-32准备的，用户标记字节序（byte order）。拿UTF-16来举例，其是以两个字节为编码单元，在解释一个UTF-16文本前，首先要弄清楚每个编码单元的字节序。例如收到一个“奎”的Unicode编码是594E，“乙”的Unicode编码是4E59。如果我们收到UTF-16字节流"594E"，那么这是“奎”还是“乙”？

　　Unicode规范中推荐的标记字节顺序的方法是BOM：在UCS编码中有一个叫做**"ZERO WIDTH NO-BREAK SPACE"（零宽度无间断空间）**的字符，它的编码是**FEFF**。而FEFF在UCS中是不不能再的字符（**即不可见**），所以不应该出现在实际传输中。UCS规范建议我们在传输字节流前，先传输字符"ZERO WIDTH NO-BREAK SPACE"。这样如果接收者接收到FEFF，就表明这个字节流是Big-Endian的；如果收到FFFE，就表明这个字节流是Little-Endian的。因此字符"ZERO WIDTH NO-BREAK SPACE"又被称为BOM。

　　**UTF-8是以字节为编码单元，没有字节序的问题**。

------

延伸一下：

　　UTF-8编码是以1个字节为单位进行处理的，不会受CPU大小端的影响；需要考虑下一位时就地址 + 1。

　　UTF-16、UTF-32是以2个字节和4个字节为单位进行处理的，即1次读取2个字节或4个字节，这样一来，在存储和网络传输时就要考虑1个单位内2个字节或4个字节之间顺序的问题。

------

**UTF-8 BOM**

　　UTF-8 BOM又叫UTF-8 签名，UTF-8不需要BOM来表明字节顺序，但可以用BOM来表明编码方式。当文本程序读取到以 EF BB BF开头的字节流时，就知道这是UTF-8编码了。**Windows就是使用BOM来标记文本文件的编码方式的**。

------

补充：

"ZERO WIDTH NO-BREAK SPACE"字符的UCS编码为FEFF（假设为大端），对应的UTF-8编码为 EF BB BF 

------

 　　即以EF BB BF开头的字节流可表明这是段UTF-8编码的字节流。**但如果文件本身就是UTF-8编码的，EF BB BF这三个字节就毫无用处了。　所以，可以说BOM的存在对于UTF-8本身没有任何作用**。 

**UTF-8文件中包含BOM的坏处**

　　**1、对php的影响**

　　php在设计时就没有考虑BOM的问题，也就是说他不会忽略UTF-8编码的文件开头的那三个EF BB BF字符，直接当做文本进行解析，导致解析错误。

　　**2、在linux上执行SQL脚本报错**

------

　　最近开发过程中遇到，windows下编写的SQL文件，在linux下执行时，总是报错。

　　*在文件的开头，无论是使用中文注释还是英文注释，甚至去掉注释，也会报SP2-0734: unknown command beginning "?<span "="">dec<span "="">lare ..." - rest of line ignored. 的错误。
<span "="">如下是文件开头部分*

```sql
1 --create tablespace
2 declare
3 v_tbs_name varchar2(200):='hytpdtsmsshistorydb';
4 begin
```

　　报错如下：

```sql
1 SP2-0734: unknown command beginning "?--create ..." - rest of line ignored.
2 
3 
4 PL/SQL procedure successfully completed.
```

　　网上没有找到类似问题的解决办法，且文件编码确认已经更改为utf-8，该问题困惑了我很久。
　　最后查看一下BOM与 no BOM的区别，尝试更改为no BOM，居然就没有再出现错误。

　　修改完成后，无论使用中文，还是英文，或者去掉注释，都能正常执行。

------

 **血泪建议：UTF-8最好不要带BOM**

　　UTF-8」和「带 BOM 的 UTF-8」的区别就是有没有 BOM。即文件开头有没有 U+FEFF。

　　1、Linux中查看BOM的方法：使用less命令，其它命令可能看不到效果：

![img](https://img2018.cnblogs.com/blog/763943/201906/763943-20190615200024203-921803934.png)

　　发现词语之前多了一个<U+FEFF>。

　　2、UTF-8去除BOM的方法

　　Linux下：

　　（1）

　　　　1)vim打开文件

　　　　2)执行:set nobomb

　　　　3)保存:wq

　　（2）

　　　　dos2unix filename

　　　　将windows格式文件转为Unix、Linux格式文件。该命令不仅可将windows文件的换行符\r\n转为Unix、Linux文件的换行符\n，还可以将UTF-8 Unicode (with BOM)转换为UTF-8 Unicode.

　　**PS:**

　　**遇到一个比较坑爹的情况，1个UTF-8 Unicode (with BOM)文件中包含两个<U+FEFF>，这是无论使用方法（1）还是方法（2），都要执行两次才能将<U+FEFF>完全去除！！！**

　　（2）Windows下，使用NotePad++打开这个文件，然后选择“编码”，再选择“以UTF-8无BOM格式编码”，最后重新保存文件即可！



"sig" in "utf-8-sig" is the abbreviation of "signature" (i.e. signature utf-8 file). Using utf-8-sig to read a file will treat BOM as file info. instead of a string.



[python - What is the difference between utf-8 and utf-8-sig? - Stack Overflow](https://stackoverflow.com/questions/57152985/what-is-the-difference-between-utf-8-and-utf-8-sig)

