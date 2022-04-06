---
title: Algorithms, Part II - Week 2 - Seam Carving
date: 2017-12-04 14:17:55
categories: MOOC
mathjax: true
tags:
    - DFS
    - DAG
    - Shortest path
---

这是一个PS里的功能，删去图片中信息量较小的像素，实现改变图片长宽比而不发生扭曲
<!-- more -->

{% asset_img HJoceanSmall_compressed.jpg origin %}
{% asset_img HJoceanSmallShrunk_compressed.jpg shrunk%}

> 是否也可以做一个生成新像素的程序呢

# 分析

分三步

- 计算每个像素能量
- 找出最小能量像素
- 移除最小能量像素

## 能量计算

这点作业写的很详细，一个叫双梯度能量的函数可供计算

对于像素$(x,y)$，计算它的能量公式为
$$
\sqrt{\Delta_x^2(x, y) + \Delta_y^2(x, y)}
$$

其中
$$
\Delta_x^2(x, y) = R_x(x, y)^2 + G_x(x, y)^2 + B_x(x, y)^2
$$
并且规定边缘像素能量为1000

$R_x(x, y),G_x(x, y),B_x(x, y)$分别为像素的RGB值，用java自带的方法就能得到

## 寻找最小能量

按照作业思路，实现一个垂直寻找方法和一个倒置方法，这样一来水平寻找方法就是先倒置进行垂直寻找再倒置回来

## 移除最小能量

同寻找方法思路，实现一个水平移除方法和一个倒置方法
这里要同时移除 color 数据

## 输出图片

不能每移除一次像素就生成一次图片，可以在构造函数获取Color类型的数据，在移除能量的时候一并移除相关Color，最后才生成图片。
写的时候发现不能用泛型数组真的迷茫，好在RGB可以直接用整形存储，再写一个把 int 转成 RGB 的函数就行

```java
private Color getRGB(int rgb) {
    return new Color((rgb >> 16) & 0xFF, (rgb >> 8) & 0xFF, rgb & 0xFF);
}
```

---

参考

http://coursera.cs.princeton.edu/algs4/assignments/seam.html

http://coursera.cs.princeton.edu/algs4/checklists/seam.html

http://www.jianshu.com/p/24cf792dbef8

https://github.com/nastra/AlgorithmsPartII-Princeton/blob/master/src/SeamCarver.java
