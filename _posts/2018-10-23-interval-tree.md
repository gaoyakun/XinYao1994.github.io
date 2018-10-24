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

## <center>线段树</center>

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
  /**************************************************************************
  	本代码演示如何利用线段树对线性数组区间修改,区间求和
  	所有数组下标由1开始
  	使用二叉树的数组存储方式来保存线段树的数据,对于线段K,其左右孩子为K*2和K*2+1
  ***************************************************************************/
  #include <stdio.h>
  
  // 元素的数量
  #define N_NUMBERS 1000
  // 线段树节点数组的大小，设为4N
  #define N_NODES (4*N_NUMBERS)
  
  int num[N_NUMBERS+1]; // 存放线性结构数据（非必需，在这里仅对拍用）
  
  /**************************************************************************
  下面是线段树数据，为二叉树的数组存储模式，以下采用多个数组形石，也可使用struct。*/
  int sum[N_NODES];//存放每个线段求和的结果
  int inc[N_NODES];// 存放每个线段的修改增量标记
  /*************************************************************************/
  
  /** 初始化线性数组为1到N_NUMBERS
   */
  void numbers_init () {
      for (int i = 1; i <= N_NUMBERS; i++) {
          num[i] = i;
      }
  }
  
  /** 利用递归的方式创建线段树
   * @param node 要创建的线段，该值为二叉树索引，下面的函数均相同
   * @param l 该线段的左端点，该值为线性数组索引，下面的函数均相同
   * @param r 该线段的右端点，该值为线性数组索引，下面的函数均相同
   */
  void tree_init_recursive (int node, int l, int r) {
      if (l == r) {
          // 左右端点相同的线段为叶子，设置求和结果为该元素的值
          sum[node] = num[l];
      } else {
          // 不是叶子
          // 递归创建左右子线段
          int mid = (l + r) / 2;
          tree_init_recursive (node * 2, l, mid);
          tree_init_recursive (node * 2 + 1, mid + 1, r);
          // 根据子线段的求和结果更新自己的求和结果
          sum[node] = sum[node * 2] + sum[node * 2 + 1];
      }
      // 清除增量标记
      inc[node] = 0;
  }
  
  /** 创建线段树的非递归包装 
   */
  void tree_init () {
      tree_init_recursive (1, 1, N_NUMBERS);
  }
  
  /** 利用递归的方式求某一线段的元素和
   * @param node 同上
   * @param l 同上
   * @param r 同上
   * @param L 要求和的区间的左端点，线性数组索引
   * @param R 要求和的区间的右端点，线性数组索引
   * @return 该线段的元素之和
   */
  int tree_get_sum_recursive (int node, int l, int r, int L, int R) {
      if (L <= l && R >= r) {
          //该线段完全落入区间内，直接返回该线段的和
          return sum[node];
      } else {
          //线段和区间相交，分别递归左右子线段求和
          int mid = (l + r) / 2;
          //注意：先检查本线段的增量标记，如果有增量需要将该增量传播到子线段
          if (inc[node]) {
              // 更新左子线段的和和增量标记
              sum[node * 2] += (mid - l + 1) * inc[node];
              inc[node * 2] += inc[node];
              // 更新右子线段的和和增量标记
              sum[node * 2 + 1] += (r - mid) * inc[node];
              inc[node * 2 + 1] += inc[node];
              // 清除自己的增量标记
              inc[node] = 0;
          }
          //递归求子线段的和
          int result = 0;
          if (L <= mid) {
              // 和左子线段相交，计算之
              result += tree_get_sum_recursive (node * 2, l, mid, L, R);
          }
          if (R > mid) {
              // 和右子线段相交，计算之
              result += tree_get_sum_recursive (node * 2 + 1, mid + 1, r, L, R);
          }
          return result;
      }
  }
  
  /** 递归求和的非递归包装 
   * @param L 同上
   * @param R 同上
   * @return 同上
  */
  int tree_get_sum (int L, int R) {
      return tree_get_sum_recursive (1, 1, N_NUMBERS, L, R);
  }
  
  /** 求和的朴素算法，用于对拍
   * @param L 同上
   * @param R 同上
   * @return 同上
   */
  int brute_force_get_sum (int L, int R) {
      int result = 0;
      for (int i = L; i <= R; i++) {
          result += num[i];
      }
      return result;
  }
  
  /** 利用递归修改区间的值
   * @param node 同上
   * @param l 同上
   * @param r 同上
   * @param L 要修改的区间的左端点，线性数组索引
   * @param R 要修改的区间的右端点，线性数组索引
   * @param value 要修改区间内每个元素的增量
   */
  void tree_inc_recursive(int node, int l, int r, int L, int R, int value) {
      if (L <= l && R >= r) {
          /*  该线段完全落入修改区间之内，根据增量修正区间的和，并记录增量标记
          	之所以要记录标记是因为我们我们的子线段的和并未修正，出于不正确的状态。
          	如果递归更新所有子线段的和就等于又回到了O(N)的复杂度，没有意义。
          	所以我们在这里通过增量标记进行记录，等查询的时候发现了有标记，就根据标记
          	更新子线段并清除标记。
          	下面使用+=进行自增，是为了在本线段原来已经有增量标记的情况下也不出问题。
          */
          sum[node] += (r - l + 1) * value;
          inc[node] += value;
      } else {
          /* 该线段和修改区间相交，需要先把增量标记传播到子线段，再递归进行修改 */
          int mid = (l + r) / 2;
          sum[node * 2] += (mid - l + 1) * inc[node];
          inc[node * 2] += inc[node];
          sum[node * 2 + 1] +=(R - mid) * inc[node];
          inc[node * 2 + 1] += inc[node];
          inc[node] = 0;
          if (L <= mid) {
              //区间和左子线段相交，处理左子线段
              tree_inc_recursive (node * 2, l, mid, L, R, value);
          }
          if (R > mid) {
              //区间和右子线段相交，处理右子线段
              tree_inc_recursive (node * 2 + 1, mid + 1, r, L, R, value);
          }
          //根据左右子线段的和更新自己的和
          sum[node] = sum[node * 2] + sum[node * 2 + 1];
      }
  }
  
  /** 区间修改的非递归包装 
   * @param L 同上
   * @param R 同上
   * @param value 同上
   */
  void tree_inc (int L, int R, int value) {
  	tree_inc_recursive (1, 1, N_NUMBERS, L, R, value);    
  }
  
  /** 区间修改的朴素版本 
   * @param L 同上
   * @param R 同上
   * @param value 同上
   */
  void brute_force_inc (int L, int R, int value) {
      for (int i = L; i <= R; i++) {
          num[i] += value;
      }
  }
  
  /** 下面是个测试函数，用线段树版本和朴素版本对拍来测试算法是否正确
   * 该函数将测试每一种可能的区间分布。
   * @return 返回true表示测试成功，返回false表示测试失败，有BUG
   */
  bool test_get_sum () {
      for (int i = 1; i < N_NUMBERS; i++) {
          for (int j = i + 1; j <= N_NUMBERS; j++) {
              int a = tree_get_sum (i, j);
              int b = brute_force_get_sum (i, j);
              if (a != b) {
                  // 失败
                  printf ("** test_get_sum(%d,%d): expect %d, but got %d\n", i, j, b, a);
                  return false;
              }
          }
      }
      printf ("test_get_sum() test ok\n");
      return true;
  }
  
  int main (int argc, char *argv[]) {
      // 初始化线性表
      numbers_init ();
      // 初始化线段树
      tree_init ();
      // 在未做区间修改的情况下测试求和
      test_get_sum ();
      // 对区间做两次修改再测试求和
      tree_inc (N_NUMBERS/4, N_NUMBERS/2, 37);
      brute_force_inc (N_NUMBERS/4, N_NUMBERS/2, 37);
      tree_inc (N_NUMBERS/6, N_NUMBERS * 5 / 8, -9);
      brute_force_inc (N_NUMBERS/6, N_NUMBERS * 5 / 8, -9);
      test_get_sum ();
      
      return 0;
  }
  ```

  