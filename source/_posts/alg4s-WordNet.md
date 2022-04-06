---
title: Algorithms, Part II - Week 1 - WordNet
date: 2017-12-04 10:26:07
categories: MOOC
tags:
	- DFS
	- DAG
	- Shortest path
---

突然开课的 Part II，难度大增<!-- more -->

---

# 分析

目标是用有向无环图(DAG)建立一颗词典树，树根是 event 这个单词

{% asset_img wordnet-fig1.png %}

## DeluxeDFS.java

先从SAP入手，新建一个叫DeluxeDFS的类，增加一个`getMark()`方法，得到`boolean[] marked`

```java
public boolean[] getMarked(){
    boolean[] marked = this.marked;
    return marked;  
}
```

## SAP.java

什么是 shortest ancestor path ？给了一个简单的例子。
> For example, in the digraph at left (digraph1.txt), the shortest ancestral path between 3 and 11 has length 4 (with common ancestor 1). In the digraph at right (digraph2.txt), one ancestral path between 1 and 5 has length 4 (with common ancestor 5), but the shortest ancestral path has length 2 (with common ancestor 0).

{% asset_img wordnet-sap.png %}

读入 synsets*.txt 进行测试，第三个字段为词语解释，这个作业不需要
因此，用 DeluxeDFS 访问 boolean[] marked ，修改讲义P20的 relax()

```java
for (int i = 0; i < vMarked.length; i++) {
    if (vMarked[i] && wMarked[i]) {                     // is same ancestral
        currentPath = vBFS.distTo(i) + wBFS.distTo(i);  // update shortest path
        if (currentPath < shortestPath) {
            shortestPath = currentPath;
            shortestAncestor = i;                       // update shortest path
        }
    }
}
```

## WordNet.java

提供测试的 main 方法读入的两个参数分别是同义词和这个词的上位词(hypernyms)
构造函数把同义词加入一个 list，用上位词创建DAG

设计 一个 Hypernym 类，用 ArrayList 来记录上位词(hypernyms)ID

```java
private class Hypernym implements Comparable<Hypernym> {

    private final String nounId;
    private final ArrayList<Integer> hypernymsId;

    public Hypernym(String nounId) {
        if (nounId == null)
            throw new IllegalArgumentException();
        hypernymsId = new ArrayList<>();
        this.nounId = nounId; 
    }

    @Override
    public int compareTo(Hypernym that) {
        return this.nounId.compareTo(that.nounId);
    }

    private void addId(int id) {
        this.hypernymsId.add(id);
    }
}
```

## Outcast.java

> Given a list of wordnet nouns A1, A2, ..., An, which noun is the least related to the others?

找出一组 String[] 中和其它词偏离最远的词

---

参考
http://coursera.cs.princeton.edu/algs4/assignments/wordnet.html
http://coursera.cs.princeton.edu/algs4/checklists/wordnet.html
http://blog.csdn.net/liuweiran900217/article/details/22603325#t5