# [stderr和stdout详细解说](https://www.cnblogs.com/zhangyabin---acm/p/3203745.html)

今天又查了一下fprintf，其中对第一个参数stderr特别感兴趣。

int fprintf(FILE *stream,char *format,[argument])；

在此之前先区分一下：printf，sprintf，fprintf。

1，printf就是标准输出，在屏幕上打印出一段字符串来。

2，sprintf就是把格式化的数据写入到某个字符串中。返回值字符串的长度。

3，fprintf是用于文件操作。

   原型：int fprintf(FILE *stream,char *format,[argument])；    

   功能：fprintf()函数根据指定的format(格式)发送信息(参数)到由stream(流)指定的文件.因此fprintf()可以使得信息输出到指 定的文件。

   例子:

```c 
char name[20] = "lucy"; 
    FILE *out;
    out = fopen( "output.txt", "w" );
    if( out != NULL )
    fprintf( out, "Hello %s\n", name );
```

  返回值：若成功则返回输出字符数，若输出出错则返回负值。

好了，以上到此为止。

然后深挖stdout，stderr。

stdout, stdin, stderr的中文名字分别是标准输出，标准输入和标准错误。

////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////今天时间仓促，以下摘自

http://blog.sina.com.cn/s/blog_912673ce01013qq9.html（十分感谢）

1，我们知道，标准输出和标准错误默认都是将信息输出到终端上，那么他们有什么区别呢？让我们来看个题目：

问题：下面程序的输出是什么？（intel笔试2011）

```c
int main() {
    fprintf(stdout,"Hello ");
    fprintf(stderr,"World!");
    return0;
}
```

解答：这段代码的输出是什么呢？你可以快速的将代码敲入你电脑上（当然，拷贝更快），然后发现输出是

> World!Hello

这是为什么呢？在默认情况下，stdout是行缓冲的，他的输出会放在一个buffer里面，只有到换行的时候，才会输出到屏幕。而stderr是无缓冲的，会直接输出，举例来说就是printf(stdout, "xxxx") 和 printf(stdout, "xxxx\n")，前者会憋住，直到遇到新行才会一起输出。而printf(stderr, "xxxxx")，不管有么有\n，都输出。

2，

```c
fprintf(stderr, "Can't open it!\n");
fprintf(stdout, "Can't open it!\n");
printf("Can't open it!\n");
```

这3句效果不是一样啊，有什么区别吗？

有区别。
stdout -- 标准输出设备 (printf("..")) 同 stdout。
stderr -- 标准错误输出设备
两者默认向屏幕输出。
但如果用转向标准输出到磁盘文件，则可看出两者区别。stdout输出到磁盘文件，stderr在屏幕。

例如：
my.exe
Can't open it!
Can't open it!
Can't open it!

转向标准输出到磁盘文件tmp.txt
my.exe > tmp.txt
Can't open it!

用TYPE 看 tmp.txt的内容：
TYPE tmp.txt
Can't open it!
Can't open it!

总结：注意1，点，2点即可！