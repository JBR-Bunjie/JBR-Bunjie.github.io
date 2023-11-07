---
title: Merge Sort - 归并排序
date: 2022-12-23 12:23:23
tags:
  - Sort
  - Algorithm
categories:
  - Algorithm
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

## Merge Sort 归并排序

### Reference：

[1.5 归并排序 | 菜鸟教程 (runoob.com)](https://www.runoob.com/w3cnote/merge-sort.html)

### 算法步骤：

- 概述：

> 1. 申请空间，使其大小为两个已经排序序列之和，该空间用来存放合并后的序列；
> 2. 设定两个指针，最初位置分别为两个已经排序序列的起始位置；
> 3. 比较两个指针所指向的元素，选择相对小的元素放入到合并空间，并移动指针到下一位置；
> 4. 重复步骤 3 直到某一指针达到序列尾；
> 5. 将另一序列剩下的所有元素直接复制到合并序列尾。

- 图示：

  > 1:
  >
  > ![img](../../../images\Algorithm\mergeSort.gif)

  > 2:
  >
  > ![img](../../../images\Algorithm\1024555-20161218163120151-452283750.png)



### 实现-迭代法：

```js
function mergeSort(arr) {  // 采用自上而下的递归方法
    var len = arr.length;
    if(len < 2) {
        return arr;
    }
    var middle = Math.floor(len / 2),
        left = arr.slice(0, middle),
        right = arr.slice(middle);
    return merge(mergeSort(left), mergeSort(right));
}

function merge(left, right)
{
    var result = [];

    while (left.length && right.length) {
        if (left[0] <= right[0]) {
            result.push(left.shift());
        } else {
            result.push(right.shift());
        }
    }

    while (left.length)
        result.push(left.shift());

    while (right.length)
        result.push(right.shift());

    return result;
}
```

### 实现-递归法：

```java
import java.util.Arrays;

/**
 * Created by chengxiao on 2016/12/8.
 */
public class MergeSort {
    public static void main(String []args){
        int []arr = {9,8,7,6,5,4,3,2,1};
        sort(arr);
        System.out.println(Arrays.toString(arr));
    }
    public static void sort(int []arr){
        int []temp = new int[arr.length];//在排序前，先建好一个长度等于原数组长度的临时数组，避免递归中频繁开辟空间
        sort(arr,0,arr.length-1,temp);
    }
    private static void sort(int[] arr,int left,int right,int []temp){
        if(left<right){
            int mid = (left+right)/2;
            sort(arr,left,mid,temp);//左边归并排序，使得左子序列有序
            sort(arr,mid+1,right,temp);//右边归并排序，使得右子序列有序
            merge(arr,left,mid,right,temp);//将两个有序子数组合并操作
        }
    }
    private static void merge(int[] arr,int left,int mid,int right,int[] temp){
        int i = left;//左序列指针
        int j = mid+1;//右序列指针
        int t = 0;//临时数组指针
        while (i<=mid && j<=right){
            if(arr[i]<=arr[j]){
                temp[t++] = arr[i++];
            }else {
                temp[t++] = arr[j++];
            }
        }
        while(i<=mid){//将左边剩余元素填充进temp中
            temp[t++] = arr[i++];
        }
        while(j<=right){//将右序列剩余元素填充进temp中
            temp[t++] = arr[j++];
        }
        t = 0;
        //将temp中的元素全部拷贝到原数组中
        while(left <= right){
            arr[left++] = temp[t++];
        }
    }
}
```



## 用例1：

[计算二分图（bipartite graph）交叉点（crossing）的数量_imred的博客-CSDN博客_bipartite graph](https://blog.csdn.net/imred/article/details/82875443)

