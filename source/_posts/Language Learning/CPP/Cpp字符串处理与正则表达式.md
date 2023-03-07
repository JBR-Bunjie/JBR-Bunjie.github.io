### part0:

默认初始化

```cpp
string s; //s是一个空串
```



### part1

在C语言中，string 是定义一个字符串，存储的是一段如“abcd”的数据，而且最后还有一个结束符'\0';
char 是定义一个字符，存储一个字符，占一个字节。

在C++中，string有两种，一种是字符串`char[]`，另外一种是封装好的字符串类`string`，要区别理解。例如`'a'`是`char`, `"a"`是`char string`，这两者都是普通的字符和字符串，和C语言中没什么不同值得注意的是后者包含两个字符，末尾有一个隐身的`'\0'`
而 `string str = "a"` 是C++ 封装好的`string`。C++中的char string和string不是一回事。当用到了"string"这个关键词，就不是普通的字符串，而是用到了封装后的类。
在C++中，`char`仍然是一个primitive type（原始类型），而`string`已经经过封装，成为了一个class（类）用到它时，我们需要 `#include <string>`，它是C++ Standard Library （C++标准库）的一部分。

示例一：

c中的char* 定义字符串，不能改变字符串内的字符的内容，但却可以把另外一个字符串付给它

```c
#include "stdio.h"

int main()
{
	char* str1 = "hello world\n";
	str1 = "aa";
	str1[1] = "a";	//报错

	printf("%s",str1);

}
```
C++中使用char*定义字符串，同样不能改变字符串内的字符的内容，但却可以把另外一个字符串付给它
```cpp
#include<iostream>

using namespace std;

int main()
{
	char* pstr = "hello world";
	pstr = "aa";
	pstr[1] = "a";	//报错


	cout<<pstr<<endl;
}
```
C++中string的定义字符串，同样不能改变字符串内的字符，但却可以把另外一个字符串付给它

```cpp
#include<iostream>
#include<string>

using namespace std;

int main()
{
	string str1;
	str1= "hello world";
	str1="aa";
	str1[1]="a";
	cout<<str1<<endl;
}
```

### part1.5: about the existion of '\0'

```cpp
#include<iostream>
#include<string>

using namespace std;

int main()
{
	string str1;
	string str2;
	str1 = "hello world";
	str2 = "123\0";
	cout << str1 << endl;
	cout << str1[11] << endl;
	cout << str2[2] << endl;
	cout << str2[3] << endl;
}
```

![image-20211006170004564](./CppString-image-20211006170004564.png)



### part2: Cpp Raw String

用R来定义Raw String，能有效减少转义符`\`的出现

```c++
#include <iostream>
#include <string>

int main() {
    // 一个普通的字符串，'\n'被当作是转义字符，表示一个换行符。
    std::string normal_str = "First line.\nSecond line.\nEnd of message.\n";
    // 一个raw string，'\'不会被转义处理。因此，"\n"表示两个字符：字符反斜杠 和 字母n。
    std::string raw_str = R"(First line.\nSecond line.\nEnd of message.\n)";

    std::cout << normal_str << std::endl;
/*---输出：
First line.
Second line.
End of message.
*/
    std::cout << raw_str << std::endl;
/*---输出：
First line.\nSecond line.\nEnd of message.\n
*/
    std::cout << R"foo(Hello, world!)foo" << std::endl;
/*---输出：
Hello, world!
*/
// raw string可以跨越多行，其中的空白和换行符都属于字符串的一部分。
    std::cout << R"(
               Hello,
               world!
               )" << std::endl;
/*---输出：
               Hello,
               world!
*/
    return 0;
}
```

### part3:c++怎么提取字符串的一部分

```cpp
#include <iostream>
#include <string>
using namespace std;

void main()
{
    string s="ABAB";
    cout << s.substr(2) <<endl ; //输出AB
    cout << s.substr(0,2) <<endl ; //同上
    cout << s.substr(1,2) <<endl ; //输出BA
}
```



