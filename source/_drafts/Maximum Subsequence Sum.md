---
title: 数据结构题目集 - Maximum Subsequence Sum
date: 2016-09-19 23:26:13
categories: Algorithm
tags: 
	- PAT
	- Data Structures
---

Given a sequence of**K**integers **{ $N\_1$，$N\_2$, ... , $N\_K$ }**.A continuous subsequence is defined to be **{ $N\_i$，$N\_{i+1}$, ... ,$N\_j$ }** where **1 ≤ i ≤ j ≤ K**. The Maximum Subsequence is the continuous subsequence which has the largest sum of its elements. For example, given sequence **{ -2, 11, -4, 13, -5, -2 }**, its maximum subsequence is **{ 11, -4, 13 }** with the largest sum being 20.
Now you are supposed to find the largest sum, together with the first and the last numbers of the maximum subsequence.
<!-- more -->

**Input Specification:**
Each input file contains one test case. Each case occupies two lines. The first line contains a positive integer **K(≤10000)**. The second line contains **K** numbers, separated by a space.

**Output Specification:**
For each test case, output in one line the largest sum, together with the first and the last numbers of the maximum subsequence. The numbers must be separated by one space, but there must be no extra space at the end of a line. In case that the maximum subsequence is not unique, output the one with the smallest indices **i** and **j** (as shown by the sample case). If all the **K** numbers are negative, then its maximum sum is defined to be 0, and you are supposed to output the first and the last numbers of the whole sequence.

**Sample Input:**
```
10
-10 1 2 3 4 -5 -23 3 7 -21
```

**Sample Output:**
```
10 1 4
```

> https://pta.patest.cn/pta/test/1342/exam/4/question/18204

**答案**
C#
```cs
public class Program
{
	static int MaxL, MaxR;
	static void Main(string[] args)
	{
		int amount = System.Convert.ToInt32(System.Console.ReadLine());
		int[] element = new int[amount];
		string str = System.Console.ReadLine();
		string[] nums = str.Split(' ');

		for (int i = 0; i < element.Length; i++)
			element[i] = int.Parse(nums[i]);
		
		System.Console.WriteLine(MaxSubseqSum(element, amount) + " " + MaxL + " " + MaxR);

		System.Console.ReadKey();
	}

	static int MaxSubseqSum(int[] A, int N)
	{
		int ThisSum, MaxSum = 0;
		int i, j;
		Program.MaxL = 0;
		Program.MaxR = 0;
		for (i = 0; i < N; i++) /* i是子列左端位置 */
		{ 
			ThisSum = 0; /* ThisSum是从A[i]到A[j]的子列和 */
			for (j = i; j < N; j++) /* j是子列右端位置 */
			{
				ThisSum += A[j]; /* 对于相同的i，不同的j，只要在j-1次循环的基础上累加1项即可 */
				if (ThisSum > MaxSum) /* 如果刚得到的这个子列和更大 */
				{
					MaxL = A[i];
					MaxR = A[j];
					MaxSum = ThisSum; /* 则更新结果 */
				}
				if (MaxSum == 0 && i == A.Length - 1)
				{
					MaxL = A[0];
					MaxR = A[A.Length - 1];
				}
			} /* j循环结束 */
		} /* i循环结束 */
		return MaxSum;
	}
}
```