---
layout:     post
title:      利用DFS对DAG进行拓扑排序
date:       2018-10-28
author:     GaoYakun
category:   算法
catalog:    true
mathjax:    true
mermaid:    true
tags:
    - 算法
    - 图论
    - 拓扑排序
---

&emsp;&emsp;对有向无环图（DAG）进行拓扑排序，除了减治法（逐步删除入度为0的顶点）之外，还可以使用传统DFS方法，实现更为简单，其原理为，用传统DFS方法遍历DAG的过程中，顶点的出栈次序刚好和拓扑次序相反。



&emsp;&emsp;上代码：

``` c++
#include <iostream>
using std::cin;
using std::cout;

#define MAX_VERTICES 100

// 邻接矩阵
int adjMatrix[MAX_VERTICES+1][MAX_VERTICES+1];

// 拓扑次序
int topologicSortArray[MAX_VERTICES];
// 已排序顶点数
int topologicCount = 0;

// 记录顶点是否已访问
int vis[MAX_VERTICES+1];

// 顶点个数 
int numVertices;

/**
 *  拓扑排序的DFS过程
 *  @param vertex 顶点编号
 */
void dfs (int vertex) {
    if (vis[vertex]) {
        return;
    }
    vis[vertex] = 1;
    for (int i = 1; i <= numVertices; i++) {
        if (adjMatrix[vertex][i]) {
            dfs (i);
        }
    }
    topologicSortArray[numVertices-topologicCount-1] = vertex;
    topologicCount++;
}

/**
 *  拓扑排序
 *  @param numVertices 顶点个数
 */
void topologicSort () {
    memset (vis, 0, sizeof(vis));
    for (int i = 1; i <= numVertices; i++) {
        dfs (i);
    }
}

int main () {
    // 读入顶点数
	cin >> numVertices;
    // 读入邻接矩阵
	for (int i = 1; i <= numVertices; i++) {
		for (int j = 1; j <= numVertices; j++) {
			cin >> adjMatrix[i][j];
		}
	}
    
    // 拓扑排序
	topologicSort ();
    
    // 打印拓扑次序
	for (int i = 0; i < topologicCount; i++) {
		cout << topologicSortArray[i] << " ";
	}
	return 0;
}
```

