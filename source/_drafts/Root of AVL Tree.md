---
title: 数据结构题目集 - Root of AVL Tree
date: 2016-10-11 09:22:03
categories: Algorithm
tags: 
	- PAT
	- Data Structures
---
An AVL tree is a self-balancing binary search tree. In an AVL tree, the heights of the two child subtrees of any node differ by at most one; if at any time they differ by more than one, rebalancing is done to restore this property. Figures 1-4 illustrate the rotation rules.<!-- more -->

Now given a sequence of insertions, you are supposed to tell the root of the resulting AVL tree.

**Input Specification:**
Each input file contains one test case. For each case, the first line contains a positive integer **N(≤20)** which is the total number of keys to be inserted. Then **N** distinct integer keys are given in the next line. All the numbers in a line are separated by a space.

**Output Specification:**
For each test case, print the root of the resulting AVL tree in one line.

**Sample Input 1:**
```
5
88 70 61 96 120
```

**Sample Output 1:**
```
70
```

**Sample Input 2:**
```
7
88 70 61 96 120 90 65
```

**Sample Output 2:**
```
88
```
> https://pta.patest.cn/pta/test/1342/exam/4/question/20492


**答案**
C
```c
#include <stdio.h>
#include <stdlib.h>

#define ElementType int

typedef struct AVLNode *Position;
typedef Position AVLTree; /* AVL树类型 */
typedef struct AVLNode {
	ElementType Data; /* 结点数据 */
	AVLTree Left;     /* 指向左子树 */
	AVLTree Right;    /* 指向右子树 */
	int Height;       /* 树高 */
}AVLNode, Root;

int Max(int a, int b);
int GetHeight(Position);
AVLTree SingleLeftRotation(AVLTree A);
AVLTree DoubleLeftRightRotation(AVLTree A);
AVLTree SingleRightRotation(AVLTree A);
AVLTree DoubleRightLeftRotation(AVLTree A);
AVLTree Insert(AVLTree T, ElementType X);

int main() {
	AVLTree T = NULL;
	int N, key;
	scanf("%d\n", &N);

	for (int i = 0; i<N; i++) {
		scanf("%d", &key);
		T = Insert(T, key);
	}
	printf("%d", T->Data);
	getchar();
	return 0;
}

int Max(int a, int b) {
	return a > b ? a : b;
}

int GetHeight(Position key) {
	if (key == NULL)
		return -1;
	return key->Height;
}

AVLTree SingleLeftRotation(AVLTree A) {
	/* 注意：A必须有一个左子结点B */
	/* 将A与B做左单旋，更新A与B的高度，返回新的根结点B */
	AVLTree B = A->Left;
	A->Left = B->Right;
	B->Right = A;
	A->Height = Max(GetHeight(A->Left), GetHeight(A->Right)) + 1;
	B->Height = Max(GetHeight(B->Left), A->Height) + 1;
	return B;
}

AVLTree DoubleLeftRightRotation(AVLTree A) {
	/* 注意：A必须有一个左子结点B，且B必须有一个右子结点C */
	/* 将A、B与C做两次单旋，返回新的根结点C */

	/* 将B与C做右单旋，C被返回 */
	A->Left = SingleRightRotation(A->Left);
	/* 将A与C做左单旋，C被返回 */
	return SingleLeftRotation(A);
}

AVLTree SingleRightRotation(AVLTree B) {
	AVLTree A = B->Right;
	B->Right = A->Left;
	A->Left = B;
	A->Height = Max(GetHeight(A->Left), GetHeight(A->Right)) + 1;
	B->Height = Max(GetHeight(B->Left), GetHeight(B->Right)) + 1;
	return A;
}

AVLTree DoubleRightLeftRotation(AVLTree B) {
	B->Right = SingleLeftRotation(B->Right);
	return SingleRightRotation(B);
}

AVLTree Insert(AVLTree T, ElementType X) {
	/* 将X插入AVL树T中，并且返回调整后的AVL树 */
	if (!T) { /* 若插入空树，则新建包含一个结点的树 */
		T = (AVLTree)malloc(sizeof(struct AVLNode));
		T->Data = X;
		T->Height = 0;
		T->Left = T->Right = NULL;
	} /* if (插入空树) 结束 */

	else if (X < T->Data) {
		/* 插入T的左子树 */
		T->Left = Insert(T->Left, X);
		/* 如果需要左旋 */
		if (GetHeight(T->Left) - GetHeight(T->Right) == 2)
			if (X < T->Left->Data)
				T = SingleLeftRotation(T);      /* 左单旋 */
			else
				T = DoubleLeftRightRotation(T); /* 左-右双旋 */
	} /* else if (插入左子树) 结束 */

	else if (X > T->Data) {
		/* 插入T的右子树 */
		T->Right = Insert(T->Right, X);
		/* 如果需要右旋 */
		if (GetHeight(T->Left) - GetHeight(T->Right) == -2)
			if (X > T->Right->Data)
				T = SingleRightRotation(T);     /* 右单旋 */
			else
				T = DoubleRightLeftRotation(T); /* 右-左双旋 */
	} /* else if (插入右子树) 结束 */

	  /* else X == T->Data，无须插入 */

	  /* 别忘了更新树高 */
	T->Height = Max(GetHeight(T->Left), GetHeight(T->Right)) + 1;

	return T;
}
```