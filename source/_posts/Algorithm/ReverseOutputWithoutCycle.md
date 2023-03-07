# ReverseOutputWithoutCycle

描述：不用循环，不逐一赋值地把一个数组逆序输出

循环 -> 递归；即用递归去承担原本循环的工作

即：

```C#
void Print(int[] arr, int len) {
    if (len > 0) {
        Console.printline(arr[len-1] + "\n");
        Print(arr, len - 1);
    }
}
```


