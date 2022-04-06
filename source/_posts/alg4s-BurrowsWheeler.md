---
title: Algorithms, Part II - Week 5 - BurrowsWheeler
date: 2017-12-04 17:49:16
categories: MOOC
tags:
	- 7-Zip
	- WinRAR
---

巨长的 specification，没有图片解释，超多的OJ检测点，最后的一次作业甚至涉及到第一次的内容<!-- more -->

# 分析

## CircularSuffixArray.java

<pre> i       Original Suffixes           Sorted Suffixes         index[i]
--    -----------------------     -----------------------    --------
 0    A B R A C A D A B R A !     ! A B R A C A D A B R A    <font color="blue">11</font>
 1    B R A C A D A B R A ! A     A ! A B R A C A D A B R    <font color="blue">10</font>
 2    R A C A D A B R A ! A B     A B R A ! A B R A C A D    <font color="blue">7</font>
 3    A C A D A B R A ! A B R     A B R A C A D A B R A !    <font color="blue">0</font>
 4    C A D A B R A ! A B R A     A C A D A B R A ! A B R    <font color="blue">3</font>
 5    A D A B R A ! A B R A C     A D A B R A ! A B R A C    <font color="blue">5</font>
 6    D A B R A ! A B R A C A     B R A ! A B R A C A D A    <font color="blue">8</font>
 7    A B R A ! A B R A C A D     B R A C A D A B R A ! A    <font color="blue">1</font>
 8    B R A ! A B R A C A D A     C A D A B R A ! A B R A    <font color="blue">4</font>
 9    R A ! A B R A C A D A B     D A B R A ! A B R A C A    <font color="blue">6</font>
10    A ! A B R A C A D A B R     R A ! A B R A C A D A B    <font color="blue">9</font>
11    ! A B R A C A D A B R A     R A C A D A B R A ! A B    <font color="blue">2</font>
</pre>

读入一行字符串"ABRACADABRA!"构造一个后缀数组，再按照第一列进行字母排序，index数组记录排序后的第i行是原数组的第index[i]行

可以不生成排序后的数组，而只排序原序号

```java
public int compare(Integer first, Integer second) {
    // get start indexes of chars to compare
    int firstIndex = first;
    int secondIndex = second;
    // for all characters
    for (int i = 0; i < s.length(); i++) {
    // if out of the last char then start from beginning
    if (firstIndex > s.length() - 1)
        firstIndex = 0;
    if (secondIndex > s.length() - 1)
        secondIndex = 0;
    // if first string > second
    if (s.charAt(firstIndex) < s.charAt(secondIndex))
        return -1;
    else if (s.charAt(firstIndex) > s.charAt(secondIndex))
        return 1;
    // watch next chars
    firstIndex++;
    secondIndex++;
    }
    // equal strings
    return 0;
}
```

## BurrowsWheeler.java

### transform

目的是输出排序后数组的最后一列，以及原数组第一行在排序后数组的行号

<pre> i     Original Suffixes          Sorted Suffixes       t    index[i]			输出
--    -----------------------     -----------------------    --------			3	
 0    A B R A C A D A B R A !     ! A B R A C A D A B R <font color="blue">A</font>    11                  ARD!RCAAAABB
 1    B R A C A D A B R A ! A     A ! A B R A C A D A B <font color="blue">R</font>    10
 2    R A C A D A B R A ! A B     A B R A ! A B R A C A <font color="blue">D</font>    7
<font color="blue">*3</font>    A C A D A B R A ! A B R     A B R A C A D A B R A <font color="blue">!</font>   *0
 4    C A D A B R A ! A B R A     A C A D A B R A ! A B <font color="blue">R</font>    3
 5    A D A B R A ! A B R A C     A D A B R A ! A B R A <font color="blue">C</font>    5
 6    D A B R A ! A B R A C A     B R A ! A B R A C A D <font color="blue">A</font>    8
 7    A B R A ! A B R A C A D     B R A C A D A B R A ! <font color="blue">A</font>    1
 8    B R A ! A B R A C A D A     C A D A B R A ! A B R <font color="blue">A</font>    4
 9    R A ! A B R A C A D A B     D A B R A ! A B R A C <font color="blue">A</font>    6
10    A ! A B R A C A D A B R     R A ! A B R A C A D A <font color="blue">B</font>    9
11    ! A B R A C A D A B R A     R A C A D A B R A ! A <font color="blue">B</font>    2
</pre>

OJ不允许有`CircularSuffixArray.java`的 API 中之外的 **public**  方法，所以不能在`CircularSuffixArray.java`记录排序后的数组然后在这里引用，因此唯一入手点就是之前的`index()`

### inverse transform

<pre> i      Sorted Suffixes     t      next[i]
--    -----------------------      -------
 0    ! ? ? ? ? ? ? ? ? ? ? A        <font color="blue">3</font>
 1    A ? ? ? ? ? ? ? ? ? ? R        <font color="blue">0</font>
 2    A ? ? ? ? ? ? ? ? ? ? D        <font color="blue">6</font>
*3    A ? ? ? ? ? ? ? ? ? ? !        <font color="blue">7</font>
 4    A ? ? ? ? ? ? ? ? ? ? R        <font color="blue">8</font>
 5    A ? ? ? ? ? ? ? ? ? ? C        <font color="blue">9</font>
 6    B ? ? ? ? ? ? ? ? ? ? A       <font color="blue">10</font>
 7    B ? ? ? ? ? ? ? ? ? ? A       <font color="blue">11</font>
 8    C ? ? ? ? ? ? ? ? ? ? A        <font color="blue">5</font>
 9    D ? ? ? ? ? ? ? ? ? ? A        <font color="blue">2</font>
10    R ? ? ? ? ? ? ? ? ? ? B        <font color="blue">1</font>
11    R ? ? ? ? ? ? ? ? ? ? B        <font color="blue">4</font>
</pre>

最难的一部分，checklist 中说代码不超过10行，实际上就是 LSD

```java
public static void inverseTransform() {
    int first = BinaryStdIn.readInt();
    String s = BinaryStdIn.readString();
    int n = s.length();
    int[] next = new int[n];

    /* use LSD algorithm for char */
    // compute frequency counts
    int[] count = new int[R + 1];
    for (int i = 0; i < n; i++)
        count[s.charAt(i) + 1]++;

    // compute cumulates
    for (int r = 0; r < R; r++)
        count[r+1] += count[r];

    // move data
    for (int i = 0; i < n; i++)
        next[count[s.charAt(i)]++] = i;

    // print out
    for (int j = 0, k = next[first]; j < n; j++, k = next[k])
        BinaryStdOut.write(s.charAt(k));
    BinaryStdOut.close();
}
```

## MoveToFront.java

本次作业里最简单的一部分，简单的字符串处理；但实际编程中被 Java 内置的方法搞得有点晕，一开始不确定要用字符串还是字符数组；最后用 StringBuilder 解决

---

# 参考

- http://coursera.cs.princeton.edu/algs4/assignments/burrows.html
- http://coursera.cs.princeton.edu/algs4/checklists/burrows.html
- http://www.cnblogs.com/yunhsiao/p/algorithms_princeton.html