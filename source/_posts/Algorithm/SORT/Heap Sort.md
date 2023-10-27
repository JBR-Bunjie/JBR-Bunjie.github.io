---
title: Heap Sort - 堆排序
date: 2022-12-23 12:23:23
tags:
  - Sort
  - Algorithm
categories:
  - Algorithm
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

# Heap Sort - 堆排序

## Reference：

- ***[Recommend：堆排序之JAVA实现月光下一只赏月的猪的博客-CSDN博客java 堆排序](https://blog.csdn.net/qq_16403141/article/details/80526313)***
- [（高效率排序算法三）堆排序_送人玫瑰手留余香的博客-CSDN博客_堆排序效率](https://blog.csdn.net/h348592532/article/details/45508715)
- [1.7 堆排序 | 菜鸟教程 (runoob.com)](https://www.runoob.com/w3cnote/heap-sort.html)

## Detail：

1. 两个定义：

> 大顶堆：每个节点的值都大于或等于其子节点的值，在堆排序算法中用于升序排列；
>
> 小顶堆：每个节点的值都小于或等于其子节点的值，在堆排序算法中用于降序排列；

2. 步骤描述：

>1. 创建一个堆 H[0……n-1]；
>2. 把堆首（最大值）和堆尾互换；
>3. 把堆的尺寸缩小 1，并调用 shift_down(0)，目的是把新的数组顶端数据调整到相应位置；
>4. 重复步骤 2，直到堆的尺寸为 1。

3. 分析：

堆排序的平均时间复杂度为 Ο(nlogn)。

## Code：

### Code - JavaScript

```js
var len;    // 因为声明的多个函数都需要数据长度，所以把len设置成为全局变量

function buildMaxHeap(arr) {   // 建立大顶堆
    len = arr.length;
    for (var i = Math.floor(len/2); i >= 0; i--) {
        heapify(arr, i);
    }
}

function heapify(arr, i) {     // 堆调整
    var left = 2 * i + 1,
        right = 2 * i + 2,
        largest = i;

    if (left < len && arr[left] > arr[largest]) {
        largest = left;
    }

    if (right < len && arr[right] > arr[largest]) {
        largest = right;
    }

    if (largest != i) {
        swap(arr, i, largest);
        heapify(arr, largest);
    }
}

function swap(arr, i, j) {
    var temp = arr[i];
    arr[i] = arr[j];
    arr[j] = temp;
}

function heapSort(arr) {
    buildMaxHeap(arr);

    for (var i = arr.length-1; i > 0; i--) {
        swap(arr, 0, i);
        len--;
        heapify(arr, 0);
    }
    return arr;
}
```

### Code - Python

```python
def buildMaxHeap(arr):
    import math
    for i in range(math.floor(len(arr)/2),-1,-1):
        heapify(arr,i)

def heapify(arr, i):
    left = 2*i+1
    right = 2*i+2
    largest = i
    if left < arrLen and arr[left] > arr[largest]:
        largest = left
    if right < arrLen and arr[right] > arr[largest]:
        largest = right

    if largest != i:
        swap(arr, i, largest)
        heapify(arr, largest)

def swap(arr, i, j):
    arr[i], arr[j] = arr[j], arr[i]

def heapSort(arr):
    global arrLen
    arrLen = len(arr)
    buildMaxHeap(arr)
    for i in range(len(arr)-1,0,-1):
        swap(arr,0,i)
        arrLen -=1
        heapify(arr, 0)
    return arr
```

