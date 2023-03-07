```cpp
#include <stdio.h>
int main(){
    #if _WIN32
        system("color 0c");
        printf("http://c.biancheng.net\n");
    #elif __linux__
        printf("\033[22;31mhttp://c.biancheng.net\n\033[22;30m");
    #else
        printf("http://c.biancheng.net\n");
    #endif

    return 0;
}
```

能不能用if和else来代替#if和#else？



c++ 对象调用类内的结构体

```cpp
class Outer
{
public:
    struct Inner
    {
        int x, y;
    };
};

int main()
{
    Outer::Inner obj;
    return 0;
}
```

cpp namespace

cpp extern

cpp 

(hear 1022 -30 passto(23,24)) (see 1022 ((ball) 20 -20 1 -2) ((player hfut1 2) 23 45 0.5 1 22 40 ) ((goal r)
 12 20) ((Line r) -60))



char set and string  





substr的第二个参数是步长

```cpp
#include <iostream>
#include <string>
using namespace std;
int main()
{
	
	string str1 = "hear";
    string *str = &str1;
	string str0 = "hear";
	
	string text = "(hear 1022 -30 passto(23,24)) (see 1022 ((ball) 20 -20 1 -2) ((player hfut1 2) 23 45 0.5 1 22 40 ) ((goal r) 12 20) ((Line r) -60))";
	cout << text.substr(31, 3) << endl;
	
	cout << str1.substr(0, 0+4) << str0 << endl;
	if (str1.substr(0, 0+4) != str0)cout << "no" << endl;
	else cout<<"yes"<<endl;
	
	string str2;
	cout << str1.substr(1, 3) << str2 << "die" << endl;
	str->clear();
	cout << str1 << "die" << endl;
	
	cout <<"?"<< '\0' << "?" << endl;
    return 0;
}
```

