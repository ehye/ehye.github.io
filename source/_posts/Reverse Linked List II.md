---
title: LeetCode 92.Reverse Linked List II
date: 2016-09-25 22:49:41
categories: Algorithm
tags: 
	- LeetCode
---
Reverse a linked list from position m to n. Do it in-place and in one-pass.
<!--more-->
For example:
Given 1->2->3->4->5->NULL, m = 2 and n = 4,

return 1->4->3->2->5->NULL.

Note:
Given m, n satisfy the following condition:
1 ≤ m ≤ n ≤ length of list.

> https://leetcode.com/problems/reverse-linked-list-ii/

分为两个步骤，第一步是找到m结点所在位置，第二步就是进行反转直到n结点。反转的方法就是每读到一个结点，把它插入到m结点前面位置，然后m结点接到读到结点的下一个。

**答案**
C
```c
/*
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
struct ListNode* reverseBetween(struct ListNode* head, int m, int n) {
    if(head == NULL)
		return NULL;
	struct ListNode* fuck = (struct ListNode*)malloc(sizeof(struct ListNode*));
	fuck->next = head;
	struct ListNode* preNode = fuck;
	int i = 1;
	while(preNode->next!=NULL && i<m) {
		preNode = preNode->next;
		i++;
	}
	if(i<m)
		return head;
	struct ListNode* suck = preNode->next;
	struct ListNode* temp = suck->next;
	while(temp!=NULL && i<n) {
		struct ListNode* next = temp->next;
		temp->next = preNode->next;
		preNode->next = temp;
		suck->next = next;
		temp = next;
		i++;
	}
	return fuck->next;
}
```