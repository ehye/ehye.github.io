---
title: 数据结构题目集 - List Leaves
date: 2016-10-07 13:35:05
categories: Algorithm
tags: 
	- PAT
	- Data Structures
---
Given a tree, you are supposed to list all the leaves in the order of top down, and left to right.<!-- more -->

**Input Specification:**
Each input file contains one test case. For each case, the first line gives a positive integer **N(≤10)** which is the total number of nodes in the tree -- and hence the nodes are numbered from **0** to **N−1**. Then N lines follow, each corresponds to a node, and gives the indices of the left and right children of the node. If the child does not exist, a **"-"** will be put at the position. Any pair of children are separated by a space.

**Output Specification:**
For each test case, print in one line all the leaves' indices in the order of top down, and left to right. There must be exactly one space between any adjacent numbers, and no extra space at the end of the line.

**Sample Input:**
```
8
1 -
- -
0 -
2 7
- -
- -
5 -
4 6
```

**Sample Output:**
```
4 1 5
```
> https://pta.patest.cn/pta/test/1342/exam/4/question/19892

答案
C
```c
#include <stdio.h>

#define MaxSize 10
#define Null -1

typedef int Root;
typedef struct {
	int Data;
	int Left;
	int Right;
}TreeNode;
TreeNode T[MaxSize];
TreeNode Queue[MaxSize];

Root BuildTree();
TreeNode Pop();
void Push(TreeNode treeNode);
void LevelOrderTraversal(Root root);

int n;
int font = -1, rear = -1;
int main() {
	Root R;
	R = BuildTree();
	LevelOrderTraversal(R);
	getchar();
	return 0;
}

Root BuildTree() {
	Root Root;
	char cl, cr;
	int check[MaxSize];
	scanf("%d\n", &n);
	if (n) {
		for (int i = 0; i < n; i++)
			check[i] = 0;
		for (int i = 0; i < n; i++)
		{
			scanf("%c %c", &cl, &cr);
			getchar();
			if (cl != '-') {
				T[i].Left = cl - '0'; //string to int
				T[i].Data  = i;
				check[T[i].Left] = 1;
			}
			else {
				T[i].Data = i;
				T[i].Left = Null;
			}

			if (cr != '-') {
				T[i].Right = cr - '0'; //string to int
				T[i].Data = i;
				check[T[i].Right] = 1;
			}
			else { 
				T[i].Data = i;
				T[i].Right = Null;
			}
		}
		for (int i = 0; i < n; i++)
			if (!check[i]) {
				Root = i; break;
			}
	}
	else
		Root = -1;
	return Root;
}

void Push(TreeNode Node)
{
	Queue[++rear] = Node;
}

TreeNode Pop()
{
	return Queue[++font];
}

void LevelOrderTraversal(Root root) {
	int leaves[MaxSize];
	int k = 0, cnt = 0;
	int first = 1;
	Push(T[root]);
	for (int i = 0; i < n; i++)
	{
		TreeNode node = Pop();

		if (node.Left == -1 && node.Right == -1) {
			leaves[k++] = node.Data;
			cnt++;
		}
		if (node.Left != -1)
			Push(T[node.Left]);
		if (node.Right != -1)
			Push(T[node.Right]);
	}
	for (int i = 0; i < cnt; i++)
	{
		if (first) {
			first = 0;
			printf("%d", leaves[i]);
		}
		else printf(" %d", leaves[i]);
	}
}
```