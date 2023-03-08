---
title: 售货员问题！
date: 2022-12-23 12:23:23
tags:
  - DP
categories:
  - Algorithm
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

## 一、题目

- 一个售货员必须访问n个城市，恰好访问每个城市一次，并最终回到出发城市。
  售货员从城市i到城市j的旅行费用是一个整数，旅行所需的全部费用是他旅行经过的的各边费用之和，而售货员希望使整个旅行费用最低。
- （等价于求图的最短哈密尔顿回路问题）令G=(V, E)是一个带权重的有向图，顶点集V=(v0, v1, ..., vn-1)。从图中任一顶点vi出发，经图中所有其他顶点一次且只有一次，最后回到同一顶点vi的最短路径。

## 二、测试用例

![img](https://img-blog.csdnimg.cn/20190923191204612.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NTU5NjQx,size_16,color_FFFFFF,t_70)

 

其中1,2,3,4,5代表五个城市。此模型可抽象为图，可用邻接矩阵c表示，如下图所示：

![img](https://img-blog.csdnimg.cn/20190917202003190.png)

## 三、动态规划方程

 假设从顶点s出发，令d(i, V)表示从顶点i出发经过V(是一个点的集合)中各个顶点一次且仅一次，最后回到出发点s的最短路径长度。

​    推导：(分情况来讨论)

​    ①当V为空集，那么d()![d(i, V)](https://private.codecogs.com/gif.latex?d%28i%2C%20V%29)，表示直接从i回到s了，此时![d(i,V) = c_{is}](https://private.codecogs.com/gif.latex?d%28i%2CV%29%20%3D%20c_%7Bis%7D) 且 ![(i\neq s)](https://private.codecogs.com/gif.latex?%28i%5Cneq%20s%29)

​    ②如果V不为空，那么就是对子问题的最优求解。你必须在V这个城市集合中，尝试每一个，并求出最优解。

​     ![d(i, V)=min(C_{ik} + d(k, V-(k))](https://private.codecogs.com/gif.latex?d%28i%2C%20V%29%3Dmin%28C_%7Bik%7D%20&plus;%20d%28k%2C%20V-%28k%29%29)

​      注：![c_{ik}](https://private.codecogs.com/gif.latex?c_%7Bik%7D)表示选择的城市和城市i的距离，![d(k, V-(k))](https://private.codecogs.com/gif.latex?d%28k%2C%20V-%28k%29%29)是一个子问题。

​    综上所述，TSP问题的动态规划方程就出来了：

![img](https://img-blog.csdnimg.cn/20190923192452307.png)

## 四、用例分析

 

现在对问题定义中的例子来说明TSP的求解过程。(假设出发城市是 0城市)

![img](https://img-blog.csdnimg.cn/20190923192924546.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NTU5NjQx,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20190923192935826.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NTU5NjQx,size_16,color_FFFFFF,t_70)

这里只画出了d(1,{2,3,4}),由于篇幅有限这里就不画了。

 

①我们要求的最终结果是d(0,{1,2,3,4}),它表示，从城市0开始，经过{1,2,3,4}之中的城市并且只有一次，求出最短路径.。
②d(0,{1,2,3,4})是不能一下子求出来的，那么他的值是怎么得出的呢？看上图的第二层，第二层表明了d(0,{1,2,3,4})所需依赖的值。那么得出：
   ![img](https://img-blog.csdnimg.cn/20190923194300642.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NTU5NjQx,size_16,color_FFFFFF,t_70)
   ③d(1,{2,3,4})，d(2,{1,3,4})，d(3,{1,2,4})，d(4,{1,2,3})同样也不是一步就能求出来的，它们的解一样需要有依赖，就比如说d(1,{2,3,4})
![img](https://img-blog.csdnimg.cn/20190923194321514.png)
   d(2,{1,3,4})，d(3,{1,2,4})，d(4,{1,2,3})同样需要这么求。

  ④按照上面的思路，只有最后一层的，当V为空集时，就可以满足![d(i,V) = c_{is}](https://private.codecogs.com/gif.latex?d%28i%2CV%29%20%3D%20c_%7Bis%7D) 且 ![(i\neq s)](https://private.codecogs.com/gif.latex?%28i%5Cneq%20s%29)该条件，直接求出dp数组部分的值。

## 五、数据结构

由上述动态规划公式d(i,V)表示从顶点i出发经过V(是一个点的集合)中各个顶点一次且仅一次，最后回到出发点s的最短路径长度。根据上述给的测试用例有5个城市编号0,1,2,3,4。那么访问n个城市，恰好访问每个城市一次，并最终回到出发城市的嘴短距离可表示为d(0,{1,2,3,4}),那么问题来了我们用什么数据结构表示d(i,V)，这里我们就可二维数据dp[N][M]来表示，N表示城市的个数，M表示集合的数量，即![M = 2^{N-1}](https://private.codecogs.com/gif.latex?M%20%3D%202%5E%7BN-1%7D),之所以这么表示因为集合V有![2^{N-1}](https://private.codecogs.com/gif.latex?2%5E%7BN-1%7D)个子集。根据测试用例可得出如下dp数组表格：

 

 

![img](https://img-blog.csdnimg.cn/20190923200357348.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NTU5NjQx,size_16,color_FFFFFF,t_70)

那么你们可能就有疑问了，为什么这么表示？这里说明一下比如集合{1,2,3,4}为什么用15表示，**我们可以把集合中元素看成二进制1的位置**（二进制从右开始看），1表示从右开始第一位为1,2表示从又开始第二位为1，所以集合{1,2,3,4}可表示二进制（1111）转化为十进制为15。再举个例子比如集合{1,3}表示为二进制为0101，十进制为5。所以我们求出dp[0][15]（通用表示dp[0][![2^{N-1}-1](https://private.codecogs.com/gif.latex?2%5E%7BN-1%7D-1)]）就是本题的最终解。

注意：

- 对于第y个城市，他的二进制表达为，1<<(y-1)。
- 对于数字x，要看它的第i位是不是1，那么可以通过判断布尔表达式 (((x >> (i - 1) ) & 1) == 1或者（x  & (1<<(i-1))）!= 0的真值来实现。
- 由动态规划公式可知，需要从集合中剔除元素。假如集合用索引x表示，要剔除元素标号为i,我们异或运算实现减法，其运算表示为： x = x ^ (1<<(i - 1))。

## 六、最短路径顶点的计算

我们先计算dp[N][M]数组之后，我可以用dp数组来反向推出其路径。其算法思想如下：

比如在第一步时，我们就知道那个值最小，如下图所示：

![img](https://img-blog.csdnimg.cn/20190923194300642.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NTU5NjQx,size_16,color_FFFFFF,t_70)

因为dp[][]数组我们已经计算出来了，由计算可知C01+d(1,{2,3,4})最小，所以一开始从起始点0出发，经过1。接下来同样计算d(1,{2,3,4})

![img](https://img-blog.csdnimg.cn/20190923194321514.png)

由计算可知C14+d(4,{2,3})所以0--->1---->4，接下来同理求d(4,{2,3})，这里就省略，读者可以自行计算。最终计算出来的路径为：0--->1--->4--->2--->3--->0

```cpp
#include <iostream>
#include <cmath>
#include <cstring>
#include <vector>
 
using namespace std;
 
#define N 5
#define INF 10e7
#define min(a,b) ((a>b)?b:a)
 
static const int M = 1 << (N-1);
//存储城市之间的距离
int g[N][N] = {{0,3,INF,8,9},
               {3,0,3,10,5},
               {INF,3,0,4,3},
               {8,10,4,0,20},
               {9,5,3,20,0}};

//保存顶点i到状态s最后回到起始点的最小距离
int dp[N][M];

//保存路径
vector<int> path;
 
//核心函数，求出动态规划dp数组
void TSP(){
    //初始化dp[i][0]
    for(int i = 0; i < N; i++)
        dp[i][0] = g[i][0];
    
    //求解dp[i][j]
    for(int j = 1; j < M; j++ ) {
        for(int i = 0; i < N; i++ ) {
            dp[i][j] = INF;
            //如果集和j(或状态j)中包含结点i,则不符合条件退出
            if( ((j >> (i-1)) & 1) == 1)
                continue;
            
            for(int k = 1; k < N; k++) {
                if( ((j >> (k-1)) & 1) == 0)
                    continue;
                
                if( dp[i][j] > g[i][k] + dp[k][j^(1<<(k-1))] )
                    dp[i][j] = g[i][k] + dp[k][j^(1<<(k-1))] ;
            }
        }
    }
}

//判断结点是否都以访问,不包括0号结点
bool isVisited(bool visited[]){
    for(int i = 1 ; i<N ;i++)
        if(visited[i] == false)
            return false;
    return true;
}

//获取最优路径，保存在path中,根据动态规划公式反向找出最短路径结点
void getPath(){
    //标记访问数组
    bool visited[N] = {false};
    //前驱节点编号
    int pioneer = 0 ,min = INF, S = M - 1,temp ;
    //把起点结点编号加入容器
    path.push_back(0);
 
    while(!isVisited(visited)){
        for(int i=1; i<N;i++)
            if(visited[i] == false && (S&(1<<(i-1))) != 0)
                if(min > g[i][pioneer] + dp[i][(S^(1<<(i-1)))])
                    min = g[i][pioneer] + dp[i][(S^(1<<(i-1)))] ;
                    temp = i;
            
        pioneer = temp;
        path.push_back(pioneer);
        visited[pioneer] = true;
        S = S ^ (1<<(pioneer - 1));
        min = INF;
    }
}

//输出路径
void printPath(){
    cout<<"最小路径为：";
    vector<int>::iterator  it = path.begin();
    for(it ; it != path.end();it++)
        cout<<*it<<"--->";
    //单独输出起点编号
    cout<<0;
}
 
int main()
{
    TSP();
    cout<<"最小值为："<<dp[0][M-1]<<endl;
    getPath();
    printPath();
    return 0;
}
```

## 八、测试结果及性能分析 

![img](https://img-blog.csdnimg.cn/20190923205046784.png)

时间复杂度：![O(2^{n}n^{2})](https://private.codecogs.com/gif.latex?O%282%5E%7Bn%7Dn%5E%7B2%7D%29)

空间复杂度：![O(2^{n})](https://private.codecogs.com/gif.latex?O%282%5E%7Bn%7D%29)

[旅行商问题（动态规划方法，超级详细的）_仁者乐山智者乐水的博客-CSDN博客_旅行商问题](https://blog.csdn.net/qq_39559641/article/details/101209534)