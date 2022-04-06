---
title: LeetCode 344.Reverse String
date: 2016-08-31 21:08:38
categories: Algorithm
tags: 
    - LeetCode
---
Write a function that takes a string as input and returns the string reversed.<!--more-->

**Example:**
Given s = "hello", return "olleh".

> https://leetcode.com/problems/reverse-string/

**答案**
C（用库函数）

```c
    char* reverseString(char* s) {
    int len = strlen(s);
    char temp;
    for(int i=0; i<len/2; i++){
        temp=s[i];
        s[i]=s[len-1-i];
        s[len-1-i]=temp;
    }
        return s;
    }
```

C（不用库函数）

```c
    #include<stdio.h>

    char* reverseString(char* s);
    int strLen(char* s);

    void main()
    {
        char str[100000] = "";

        printf("%s", reverseString(str));
    }

    char* reverseString(char* s) {
        if(s == NULL)
            return 0;

        char *str = s;
        char *begin = s;
        char *end = s + strLen(str);
        char temp;

        while(begin < end) {
            temp = *begin;
            *begin = *end;
            *end = temp;

            begin++;
            end--;
        }

        return str+1;
    }

    int strLen(char* s){
        char *eos = s;
        while(*eos++);
        return(eos-s-1);
    }
```

Java

```java
    class Solution {
        private String s;
            public String reverseString(String s) {
            this.s = s;
            StringBuffer zs = new StringBuffer();
            char rs;
            for (int i=0; i<s.length(); i++ ) {
            rs = s.charAt(s.length()-i-1 );
            zs.append(rs);
            }
            String ds = zs.toString();
            return ds;
        }
    }
```
