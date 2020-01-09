---
title: Algorithms, Part II - Week 4 - Boggle
date: 2017-12-04 16:52:35
categories: MOOC
tags:
    - TST
    - Tries
    - DFS
---

根据规则找出尽可能多的单词，编写 Boggle 游戏的AI部分<!-- more -->

{% asset_img boggle-gui.png %}

使用 DFS 解决

```java
public Iterable<String> getAllValidWords(BoggleBoard board) {
    TreeSet<String> words = new TreeSet<>();
    for (int i = 0; i < board.rows(); i++) {
        for (int j = 0; j < board.cols(); j++) {
            boolean[][] visited = new boolean[board.rows()][board.cols()];
            dfs(board, i, j, words, visited, "");
        }
    }
    return words;
}
```
