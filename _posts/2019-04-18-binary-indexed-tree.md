---
layout:     post
title:      树状数组(二叉索引树)
date:       2019-04-18
author:     GaoYakun
category:   算法
catalog:    true
mathjax:    true
mermaid:    true
tags:
    - 算法
    - 数据结构
---

&emsp;&emsp;树状数组（Binary Indexed Tree）也叫二叉索引数。它类似于线段树，主要用于查询数组中任意区间的元素和。它的查询和修改的时间复杂度均为$O(\log N)$。它的核心规律为树状数组中第$i$个元素对应原数组中$[i-lowbit(i)+1,i]$这一区间的和。令A为原数组，C为对应的树状数组，则对应的公式为：



$$
lowbit(n)=n\mathbin{\&}-n \tag{1}
$$
$$
C_n=\sum_{k=n-lowbit(n)+1}^n A_k \tag{2}
$$

&emsp;&emsp;由公式可见，元素$C[i]$的前面一个不重叠的区间为$C[i-lowbit(i)]$，所以如果想要求原数组前$n$个元素的和，可以用以下的方式：

```c++
// 求树状数组C的前n个元素和（假设元素起始索引为1）：
int sum (int C[], int n) {
    int result = 0;
    while (n) {
        result += C[n];
        n -= n & -n;
    }
    return result;
}
```
&emsp;&emsp;基本的树状数组支持单点修改。
```c++
// 原数组的第n个元素增加d，total为总的元素个数
void add (int C[], int total, int n, int d) {
    while (n <= total) {
        C[n] += d;
        n += n & -n;
    }
}
```
- ## 增强的树状数组

&emsp;&emsp;基本的树状数组不支持区间修改，但是如果我们用树状数组维护原来数组的差分序列（也就是相邻元素的差），则可以很方便地进行区间修改。令D为差分序列，并且令$A[0]=0$则有：
$$
D_i=A_i-A_{i-1} \tag{3}
$$
$$
C_n=\sum_{k=n-lowbit(n)+1}^n D_k \tag{4}
$$
$$
A_n=\sum_{k=1}^n D_k \tag{5}
$$

&emsp;&emsp;如果我们对区间$[a,b]$之间的元素增加$d$，则需调用两次单点修改，分别对$D[a]$增加$d$，对$D[b+1]$减少$d$（如果b+1未超出范围）。通过差分序列，我们可以方便地进行单点查询，根据公式$5$可知，单点查询就是求差分序列的前缀和。使用差分序列的缺点是无法进行区间查询。如果我们需要进行区间查询则公式如下：
$$
\sum_{k=1}^n A_k=\sum_{k=1}^n \sum_{j=1}^k D_j \tag{6}
$$
&emsp;&emsp;该公式经过变形可得：
$$
\sum_{k=1}^n A_k=(n+1)\sum_{k=1}^n D_k - \sum_{k=1}^n {kD_k} \tag{7}
$$

&emsp;&emsp;根据公式$7$可以看出，我们可以通过两个$D_k$和$kD_k$两个差分序列来方便地求出原数组的前缀和，进而实现区间查询。



