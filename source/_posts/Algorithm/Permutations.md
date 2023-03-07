# Permutations - 全排列

顾名思义，就是找出当前元素所有可行的序列

一个朴素的思想是递归，这种方式简单但并不高效快捷



以两个示例来进行算法说明：

Example1: 

>实验五     数据输出
>
>实验准备
>理解数据输入、输出的技巧。
>
>实验目的
>理解算法设计的数学基本思想，理解算法程序化实现的技巧。
>
>实验过程
>输出1,2,3,4,5,6这六个元素的所有全排列。

```python
def permutations(arr: list, position: int, end: int):
    if position == end:
        # 完成一次排列，输出结果，返回上层
        print(arr)
    else:
        # 采用递归解决问题
        for index in range(position, end):
            # 每进入一次函数，都会在当前位置山建立循环，这个循环会将所有的锁具都和当前数进行一次交换
            # 由于每单次循环都会将数据列表重置还原，所以不会对下一次交换产生影响导致重复
            # 故从当前的position开始，与end之前的所有数据交换次序，就可以得到所有内容
            arr[index], arr[position] = arr[position], arr[index]
            permutations(arr, position + 1, end)
            arr[index], arr[position] = arr[position], arr[index]  # 还原到交换前的状态，为了进行下一次交换

permutations(arr, 0, len(arr))
print("共720条数据")  # 6*5*4*3*2*1 == 720
```



Example2: [46. 全排列 - 力扣（LeetCode） (leetcode-cn.com)](https://leetcode-cn.com/problems/permutations/)

```python
class Solution:
    def permute(self, nums: List[int]) -> List[List[int]]:
        self.result = []
        self.generatation(nums, 0, len(nums))
        return self.result

    def generatation(self,nums, start, end):
        if start == end:
            # print(nums)
            self.result.append([t for t in nums])
            return
        
        for i in range(start, end):
            nums[i], nums[start] = nums[start], nums[i]
            self.generatation(nums, start+1,end)
            nums[start], nums[i] = nums[i], nums[start]
```



[1.6 字符串的全排列 | 编程之法：面试和算法心得 (gitbooks.io)](https://wizardforcel.gitbooks.io/the-art-of-programming-by-july/content/01.06.html)