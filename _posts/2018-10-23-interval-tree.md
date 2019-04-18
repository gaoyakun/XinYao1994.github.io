---
layout:     post
title:      线段树
date:       2018-10-23
author:     GaoYakun
category:   数据结构
catalog:    true
mathjax:    true
mermaid:    true
tags:
    - 数据结构
    - 线段树
---

&emsp;&emsp;线段树是在对一个线性数据结构的连续区间进行动态的修改或查询操作时应用很广范的一种数据结构，它巧妙地利用了分治原理将原本的O(N)复杂度降低到O(logN)。本文介绍了线段树的基本原理和实现方法。



- ### 定义

  &emsp;&emsp;一种空间二叉分割树，是将一个一维的线性结构在保持其顺序的情况下不断地进行二分，直到分割为节点只包含该线性结构的一个元素为止，每个节点表示了该线性结构的一个区间，我们称之为“线段”，其左右端点为该线段的起始元素和结束元素的位置（下标）。整个过程类似于“一尺之棰，日取其半，万世不竭“，其效果是将一个线性结构映射为一个二叉树。

  &emsp;&emsp;具体的分割方式为：对于端点为$L$和$R$的线段，我们取它的中点$M=\lfloor (L+R)/2 \rfloor$，左子线段区间为$[L, M]$，右子线段区间为$[M+1, R]$。对于叶子，其左右端点相等。

  &emsp;&emsp;由于每次都从线段的中心分割，得到的是一颗平衡树。

  &emsp;&emsp;例如对含有10个元素的一维数组进行分割，效果如下图所示（方框内的两个数字为线段的两个端点）：

  <div class="mermaid">
  graph TD
  A[1,10] --> B[1,5]
  A[1,10] --> C[6,10]
  B[1,5] --> D[1,3]
  B[1,5] --> E[4,5]
  C[6,10] --> F[6,8]
  C[6,10] --> G[9,10]
  D[1,3] --> H[1,2]
  D[1,3] --> I[3,3]
  E[4,5] --> J[4,4]
  E[4,5] --> K[5,5]
  H[1,2] --> L[1,1]
  H[1,2] --> M[2,2]
  F[6,8] --> N[6,7]
  F[6,8] --> O[8,8]
  N[6,7] --> P[6,6]
  N[6,7] --> Q[7,7]
  G[9,10] --> R[9,9]
  G[9,10] --> S[10,10]
  </div>

- ### 性质

  &emsp;&emsp;可以看出，对于$N$个元素的线性结构，其线段树的叶子数量也为$N$。如果将这个线段树补全成满二叉树的话，根据满二叉树的性质，其叶子数量为不小于$N$的2的整数次幂，即$2^{\lceil\log N\rceil}$，设该满二叉树节点总数量为$K$，则有：$K=\sum_{k=0}^{\lceil\log N\rceil} 2^k=2^{1+\lceil\log N\rceil}-1$，由于$\lceil\log N\rceil<=\log 2N$，可得$K<2^{1+\log 2N}=4N$。**结论：对于具有N个元素的线性结构，其线段树的总节点数小于4N。**

- ### 作用

  线段树是对线性结构的树形表达，其主要作用是将线性结构中某一段区间的运算转换为子区间的运算，利用分治的方式来将$O(N)$的时间复杂度优化为$O(\log N)$，为了实现该优化，我们需要保存区间的运算结果。对区间的运算必须满足该运算结果可以通过子区间的运算结果来获得，例如求和运算，求最大值，最小值，求最大公约数等运算都满足该条件。

- ### 实现

  对于线段树，我们主要需要实现以下功能：

  1. 构造（通过线性结构构造线段树）
  2. 区间修改（通过增加，减少或替换等方式修改某一区间范围内的元素的值）
  3. 区间查询（查询某一区间范围内的运算结果）

- ### 代码

  ``` c++
  /**********************************************************
    intervaltree.hpp
  	线段树的代码实现
  **********************************************************/
  #ifndef __INTERVAL_TREE__
  #define __INTERVAL_TREE__

  #include <vector>

  /**
    这里利用Traits模板来自定义线段树的实现。
    combine方法定义了如何合并左右孩子节点
    combineLeft定义了只有左孩子时候的合并方法
    combineRight定义了只有右孩子时候的合并方法
    update定义了当某节点区间内的元素增加时如何更新节点值
  */

  // 求和的Traits模板
  template <class T>
  struct IntervalTreeTraits_Sum {
      T combine (const T &a, const T &b) const {
          return a + b;
      }
      T combineLeft (const T &a) const {
          return a;
      }
      T combineRight (const T &a) const {
          return a;
      }
      T update (const T &a, const T &delta, size_t num) const {
          return a + delta * num;
      }
  };

  // 求最大值的Traits模板
  template <class T>
  struct IntervalTreeTraits_Max {
      T combine (const T &a, const T &b) const {
          return a > b ? a : b;
      }
      T combineLeft (const T &a) const {
          return a;
      }
      T combineRight (const T &a) const {
          return a;
      }
      T update (const T &a, const T &delta, size_t num) const {
          return a + delta;
      }
  };

  // 求最小值的Traits模板
  template <class T>
  struct IntervalTreeTraits_Min {
      T combine (const T &a, const T &b) const {
          return a < b ? a : b;
      }
      T combineLeft (const T &a) const {
          return a;
      }
      T combineRight (const T &a) const {
          return a;
      }
      T update (const T &a, const T &delta, size_t num) const {
          return a + delta;
      }
  };

  // 线段树模板
  template <class T, class Traits = IntervalTreeTraits_Sum<T> >
  class IntervalTree {
      mutable vector<T> C;
      mutable vector<T> f;
      Traits traits;
  public:
      // 初始化线段树为n个零
      void init (size_t n) {
          C.resize (n * 4, 0);
          f.resize (n * 4, 0);
      }
      // 获取元素个数
      size_t count () const {
          return C.size() / 4;
      }
      // 区间修改，给[posStart,posEnd]区间内的元素都增加d
      void rangeAdd (size_t posStart, size_t posEnd, T d) {
          rangeAdd_r (1, 1, C.size()/4, posStart, posEnd, d);
      }
      // 区间查询，获取[posStart,posEnd]区间内的元素之和
      T rangeQuery (size_t posStart, size_t posEnd) const {
          return rangeQuery_r (1, 1, C.size()/4, posStart, posEnd);
      }
  private:
      void rangeAdd_r (size_t node, size_t l, size_t r, size_t L, size_t R, T d) {
          if (L <= l && R >= r) {
              C[node] = traits.update(C[node], d, r - l + 1);
              f[node] += d;
          } else {
              size_t mid = (l + r) / 2;
              if (f[node]) {
                  C[node*2] = traits.update(C[node*2], f[node], mid - l + 1);
                  f[node*2] += f[node];
                  C[node*2+1] = traits.update(C[node*2+1], f[node], r - mid);
                  f[node*2+1] += f[node];
                  f[node] = 0;
              }
              if (L <= mid) {
                  rangeAdd_r (node*2, l, mid, L, R, d);
              }
              if (R > mid) {
                  rangeAdd_r (node*2+1, mid+1, r, L, R, d);
              }
              C[node] = traits.combine(C[node*2], C[node*2+1]);
          }
      }
      T rangeQuery_r (size_t node, size_t l, size_t r, size_t L, size_t R) const {
          if (L <= l && R >= r) {
              return C[node];
          } else {
              size_t mid = (l + r)/2;
              if (f[node]) {
                  C[node*2] = traits.update(C[node*2], f[node], mid - l + 1);
                  f[node*2] += f[node];
                  C[node*2+1] = traits.update(C[node*2+1], f[node], r - mid);
                  f[node*2+1] += f[node];
                  f[node] = 0;
              }
              T left = 0, right = 0;
              bool hasLeft = false, hasRight = false;
              if (L <= mid) {
                  left = rangeQuery_r (node*2, l, mid, L, R);
                  hasLeft = true;
              }
              if (R > mid) {
                  right = rangeQuery_r (node*2+1, mid+1, r, L, R);
                  hasRight = true;
              }
              if (hasLeft && hasRight) {
                  return traits.combine(left, right);
              } else if (hasLeft) {
                  return traits.combineLeft (left);
              } else {
                  return traits.combineRight (right);
              }
          }
      }
  };
  #endif // __INTERVAL_TREE__

  ```

  