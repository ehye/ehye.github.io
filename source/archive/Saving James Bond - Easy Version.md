---
title: 数据结构题目集 - Saving James Bond(Easy Version)
date: 2016-10-26 17:49:43
categories: Algorithm
tags: 
    - PAT
    - Data Structures
---
This time let us consider the situation in the movie "Live and Let Die"<!--more--> in which James Bond, the world's most famous spy, was captured by a group of drug dealers. He was sent to a small piece of land at the center of a lake filled with crocodiles. There he performed the most daring action to escape -- he jumped onto the head of the nearest crocodile! Before the animal realized what was happening, James jumped again onto the next big head... Finally he reached the bank before the last crocodile could bite him (actually the stunt man was caught by the big mouth and barely escaped with his extra thick boot).

Assume that the lake is a 100 by 100 square one. Assume that the center of the lake is at (0,0) and the northeast corner at (50,50). The central island is a disk centered at (0,0) with the diameter of 15. A number of crocodiles are in the lake at various positions. Given the coordinates of each crocodile and the distance that James could jump, you must tell him whether or not he can escape.

**Input Specification:**
Each input file contains one test case. Each case starts with a line containing two positive integers **N(≤100)**, the number of crocodiles, and **D**, the maximum distance that James could jump. Then **N** lines follow, each containing the **(x, y)** location of a crocodile. Note that no two crocodiles are staying at the same position.

**Output Specification:**
For each test case, print in a line "Yes" if James can escape, or "No" if not.

**Sample Input 1:**
```
14 20
25 -15
-25 28
8 49
29 15
-35 -2
5 28
27 -29
-8 -28
-20 -35
-25 -20
-13 29
-30 15
-35 40
12 12
```
**Sample Output 1:**
```
Yes
```
**Sample Input 2:**
```
4 13
-12 12
12 12
-12 -12
12 -12
```
**Sample Output 2:**
```
No
```

> https://pta.patest.cn/pta/test/16/exam/4/question/672

**答案**
C
```c
#include<stdio.h>
#include<stdbool.h>
#include<math.h>

#define MaxVertexNum 100
#define Diameter 15

typedef int Crocodile;

typedef struct LocOfCro{
    int x;
    int y;
}Location;

int Nc, distance;
bool visited[MaxVertexNum] = { false };
Location crocodiles[MaxVertexNum];

void AddCrocodile();
void Save007(Location Crocodile[]);
int DFS(Crocodile V);
bool IsSafe(Crocodile V);
bool FirstJump(V);
bool Jump(V, W);

int main() {
    AddCrocodile();
    Save007(crocodiles);
    return 0;
}

void AddCrocodile() {
    int v1, v2;
    scanf("%d", &Nc);
    /* CreateLocation */
    for (int i = 0; i < Nc; i++) {
        crocodiles[i].x = crocodiles[i].y = -1;
    }
    scanf("%d", &distance);
    for (int i = 0; i < Nc; i++) {
        scanf("%d %d", &v1, &v2);
        /* AddCrocodile */
        crocodiles[i].x = v1;
        crocodiles[i].y = v2;
    }
}

int DFS(Crocodile i) {
    int answer = 0;
    visited[i] = true;
    if (IsSafe(i))
        answer = 1;
    else {
        for (int j = 0; j < Nc; j++)
            if (!visited[j] && Jump(i,j)) {
                answer = DFS(j);
                if (answer == 1)
                    break;
            }
    }
    return answer;
}

void Save007(Location crocodiles[]) {
    int answer = 0;
    for (int i = 0; i < Nc; i++) {
        if (!visited[i] && FirstJump(i)) {
            answer = DFS(i);
            if (answer == 1) break;
        }
    }
    if (answer == 1) printf("Yes");
    else printf("No");
}

bool IsSafe(Crocodile i) {
    return (abs(crocodiles[i].x) + distance >= 50) || (abs(crocodiles[i].y) + distance >= 50);
}

bool FirstJump(Crocodile i) {
    return sqrt(pow(crocodiles[i].x + 0.0, 2) + pow(crocodiles[i].y + 0.0, 2)) <= (Diameter / 2 + distance);
}

bool Jump(int V, int W) {
    return sqrt((crocodiles[V].x - crocodiles[W].x) * (crocodiles[V].x - crocodiles[W].x)
        + (crocodiles[V].y - crocodiles[W].y) * (crocodiles[V].y - crocodiles[W].y)) <= distance;
}
```
