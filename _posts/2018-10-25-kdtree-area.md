---
layout:     post
title:      KD-Tree求解矩形面积覆盖问题
date:       2018-10-25
author:     GaoYakun
category:   数据结构
catalog:    true
mathjax:    true
mermaid:    true
tags:
    - 数据结构
    - KD-Tree
---

## <center>KD-Tree求解矩形面积覆盖问题</center>

&emsp;&emsp;求一组矩形覆盖面积是一类很经典的算法问题，题目一般是给定N个大小已知的矩形（可以相互覆盖），求所有矩形所覆盖的总面积。假设所有矩形能覆盖的最大范围为宽w，高h，矩形数量为n，如果使用暴力法对每个面积单位进行判断的话复杂度将达到O(nwh)，显然难以接受。对于此问题，通常使用线段树+扫描线算法，这里我们介绍如何使用kd树在O(nlogn)的时间复杂度下解决此问题。



​&emsp;&emsp;算法的思路是利用矩形的边来将整个平面进行二叉分割，直至该节点所覆盖的部分全部被矩形覆盖。函数伪代码如下：

```
//这个函数递归添加矩形到平面二叉树并返回该矩形对覆盖面积做出的贡献
int addRect (Node, rect) {
    if (Node已经被覆盖) {
        return 0;
    }
    if (Node是叶子节点) {
        if (Node被rect完全包含) {
            标记Node被覆盖;
            return Node的面积;
        } else {
            在矩形rect上选择一条合适的边分割此节点;
        }
    }
    if (rect和Node的左孩子相交) {
        addRect(Node的左孩子, rect);
    }
    if (rect和Node的右孩子相交) {
        addRect(Node的右孩子, rect);
    }
}
//求N个矩形的覆盖面积
int calcArea (RootNode, rect[N]) {
    int result = 0;
    for (int i = 0; i < N; i++) {
        result += addRect(RootNode, rect[i]);
    }
    return result;
}
```

- ### C++实现

  ``` c++
  #include <iostream>
  #include <vector>
  #include <ctime>
  #include <cstdlib>
  
  using std::cin;
  using std::cout;
  using std::endl;
  using std::vector;
  
  #define MAX_RECTS 100
  #define MAX_DIM 10000
  
  /** 定义矩形结构
   */
  struct Rect {
  	int l, t, r, b; //左，上，右，下
  	Rect(int left = 0, int top = 0, int right = 0, int bottom = 0) {
  		l = left;
  		t = top;
  		r = right;
  		b = bottom;
  	}
  	/** 获取面积
  	 */
  	int getArea() const {
  		return (r - l) * (b - t);
  	}
  	/** 是否完全包含矩形other
  	 */
  	bool contains(const Rect &other) const {
  		return l <= other.l && t <= other.t && r >= other.r && b >= other.b;
  	}
  	/** 是否和矩形other相交
  	 */
  	bool intersectedWith(const Rect &other) const {
  		return l < other.r && r > other.l && t < other.b && b > other.t;
  	}
  };
  
  /** 定义平面二叉树节点
   */
  struct Node {
  	Rect rect; //节点的区域
  	int solid; //是否已完全被矩形覆盖
  	int left;  //左孩子索引，0为叶子节点
  	int right; //右孩子索引，0为叶子节点
  	Node (int l = 0, int t = 0, int r = 0, int b = 0)
  		: rect(l, t, r, b) {
  		solid = 0;
  		left = 0;
  		right = 0;
  	}
  };
  
  /** 平面二叉树，以数组形式存储
   */
  vector < Node > nodes;
  
  /** 初始化平面二叉树
  	只添加根节点，子节点将会在添加矩形的时候动态创建出来
   */
  void tree_init() {
  	nodes.resize(0);
  	nodes.push_back(Node(0, 0, MAX_DIM, MAX_DIM));
  }
  
  /** 在指定的位置左右分割二叉树节点
   * @param node 节点索引
   * @param l 相对于节点左侧的分割位置
   */
  void tree_split_h(int node, int l) {
  	nodes[node].left = nodes.size();
  	nodes.push_back(Node());
  	nodes.back().rect = nodes[node].rect;
  	nodes.back().rect.r = l;
  
  	nodes[node].right = nodes.size();
  	nodes.push_back(Node());
  	nodes.back().rect = nodes[node].rect;
  	nodes.back().rect.l = l;
  }
  
  /** 在指定的位置上下分割二叉树节点
   * @param node 节点索引
   * @param t 相对于节点上侧的分割位置
   */
  void tree_split_v(int node, int t) {
  	nodes[node].left = nodes.size();
  	nodes.push_back(Node());
  	nodes.back().rect = nodes[node].rect;
  	nodes.back().rect.b = t;
  
  	nodes[node].right = nodes.size();
  	nodes.push_back(Node());
  	nodes.back().rect = nodes[node].rect;
  	nodes.back().rect.t = t;
  }
  
  /** 递归添加矩形到平面二叉树某个节点
   * @param node 节点索引
   * @param rect 要添加的矩形
   * @return 该矩形对总覆盖面积做的贡献
   */
  int tree_add_rect(int node, const Rect &rect) {
  	if (nodes[node].solid) {
  		//如果该节点已被覆盖，贡献为0，停止递归并返回。
  		return 0;
  	}
  	if (nodes[node].left == 0) {
  		//如果是叶子节点
  		if (rect.contains(nodes[node].rect)) {
  			//如果节点位于矩形内部，标记该节点被覆盖，面积贡献即为该节点的面积，停止递归并返回。
  			nodes[node].solid = 1;
  			return nodes[node].rect.getArea();
  		} else {
  			//分别测试矩形的四条边，选择位于节点内部的任一条边来分割此节点。
  			if (rect.l > nodes[node].rect.l) {
  				tree_split_h(node, rect.l);
  			} else if (rect.r < nodes[node].rect.r) {
  				tree_split_h(node, rect.r);
  			} else if(rect.t > nodes[node].rect.t) {
  				tree_split_v(node, rect.t);
  			} else if (rect.b < nodes[node].rect.b) {
  				tree_split_v(node, rect.b);
  			}
  		}
  	} 
  	//面积贡献为矩形对左右子节点的面积贡献的和
  	int result = 0;
  	if (rect.intersectedWith(nodes[nodes[node].left].rect)) {
  		result += tree_add_rect(nodes[node].left, rect);
  	}
  	if (rect.intersectedWith(nodes[nodes[node].right].rect)) {
  		result += tree_add_rect(nodes[node].right, rect);
  	}
  	return result;
  }
  
  int main() {
  	int total = 0;
  	int n;
  	cin >> n;
  	tree_init();
  	for (int i = 0; i < n; i++) {
  		Rect rect;
  		cin >> rect.l >> rect.t >> rect.r >> rect.b;
  		total += tree_add_rect(0, rect);
  	}
  	cout << total << endl;
  	return 0;
  }
  ```

  