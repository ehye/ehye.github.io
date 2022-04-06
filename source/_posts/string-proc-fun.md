---
title: 一波字符串处理函数（C语言）
date: 2017-03-15 00:02:51
categories: Coding
tags:
---

常见的字符串处理函数<!--more-->

---

## 恺撒加密

*恺撒加密法加密规则是：将原来的小写字母用字母表中其后面的第3个字母的大写形式来替换，大写字母按同样规则用小写字母替换，对于字母表中最后的三个字母，可将字母表看成是首未衔接的。*  


**输入样例：**
AMDxyzXYZ

**输出样例：**
dpgABCabc

```c++
char epy(char input)
{
    char epy_mess;
    int input_asc = (int)input;
    if (input_asc >= 97 && input_asc <= 122) {
        switch (input_asc) {
            case 120:
                input_asc = 65;
                break;
            case 121:
                input_asc = 66;
                break;
            case 122:
                input_asc = 67;
                break;
            default:
                input_asc -= 29;
                break;
        }
    }
    else if (input_asc >= 65 && input_asc <= 90) {
        switch (input_asc) {
            case 88:
                input_asc = 97;
                break;
            case 89:
                input_asc = 98;
                break;
            case 90:
                input_asc = 99;
                break;
            default:
                input_asc += 35;
                break;
        }
    }
    else {
        return input;
    }
    return (char)input_asc;
}
```

---

## 矩阵转置

*输入矩阵阶数，然后按行输入所有矩阵元素（整数），将该矩阵转置输出。*  

**输入：**
4
4 6 8 9
2 7 4 5
3 8 16 15
1 5 7 11
**输出：**
4 2 3 1
6 7 8 5
8 4 16 7
9 5 15 11

```c++
void transfer(int(*m)[5][5], int(*t)[5][5], int rank)
{
    for (int i = 0; i < rank; i++) {
        for (int j = 0; j < rank; j++) {
            if (i != j) {
                (*t)[i][j] = (*m)[j][i];
            }
            else {
                (*t)[i][j] = (*m)[i][j];
            }
        }
    }
}
```

---

## 字符串反转函数

*输入一个字符串，将其反序输出*  

**输入样例：**
Hello,everyone

**输出样例：**
enoyreve,olleH

```c++
#include <iostream>
#include<cstring> 
using namespace std;

const int ArrSize = 100;
void mystrrev(char str[]);

int main()
{
    char str[ArrSize] = {};
    cin.getline(str, ArrSize);
    int empty_length = ArrSize - strlen(str);

    mystrrev(str);
    for (int i = empty_length; i < ArrSize; i++) {
        cout << str[i];
    }

    return 0;
}

void mystrrev(char str[])
{
    for (int i = 0; i < ArrSize / 2; i++) {
        char temp = str[ArrSize - 1 - i];
        str[ArrSize - 1 - i] = str[i];
        str[i] = temp;
    }
}
```

---

## 求数组中最大最小元素

**输入样例：**
5
90 89 30 45 55

**输出样例：**
90
30

```c++
int imax(int array[], int count)
{
    int max = -32767;
    for (int i=0; i<count; i++) {
        if (array[i] > max) {
            max = array[i];
        }
    }
    return max;
}

int imin(int array[], int count)
{
    int min = 32767;
    for (int i=0; i<count; i++) {
        if (array[i] < min) {
            min = array[i];
        }
    }
    return min;
}
```

---

## 判断一个整数是否为素数

*1不是素数*  

```c++
int isprime(int a)
{
    if (a <= 1) {
        return 0;
    }
    int Prime = 1;
    
    for ( int k = 2; k < a-1; k++) {
        if ( a % k == 0 ) {
            Prime = 0;
            break;
        }
    }
    return Prime;
}
```

---

## 英文句子中的单词个数

*It's 是一个单词*

```c++
int countWords(char* p)
{
    int count = 0;
    bool isWord = false;

    while(*p)
    {
        char c = *p;

          if ((c >= 'A' && c <= 'Z') ||
              (c >= 'a' && c <= 'z') || 
              (c == '\''))
        {
            isWord = true;
        }
        else if (isWord)
        {
            count++;
            isWord = false;
        }
        p++;
    }
    return count;
}
```

---

## 字母转换

*除字符串中包含的非字母字符(不包括空格)，并将小写字母转换成大写字母*  

**输入样例：**
hAppy new year!

**输出样例：**
HAPPY NEW YEAR

```c++
int main()
{
    char str[200]; char arr[201] = {};
    cin.getline(str, 200);
    myGetchar(str, arr);
    for (int i = 0; i < strlen(arr); i++) {
        cout << arr[i];
    }
    return 0;
}

void myGetchar(char str[200], char (&arr)[201])
{
    int count = 0; int j = 0;
    for (int i = 0; i < 200; i++) {
        char c = str[i];

        // add lowwer case to upper case
        if (c >= 'a' && c <= 'z') { c -= (char)(97 - 65); arr[j++] = c; }

        // add upper case
        else if (c >= 'A'&&c <= 'Z') { arr[j++] = c; }

        // add blankspace
        else if (c == ' ') { arr[j++] = c; continue; }

        // add end of line
        else if (c == '\0') { arr[j] = c; break; }

        // ignore some char between
        else if (c > 'Z' && c < 'a') { continue; }
        else if (c < 'A' || c > 'z') { continue; }
    }
}
```

---

## 插入加密

*插入式加密是在明文字母中按照指定间隔插入另一些字母以形成密文。例如对明文china，在间隔为1的位置插入其它字母序列中的字母a,b,c,d,e，就变成密文cahbicndae；如果其它字母序列的字母取完，则从头再取，要求密文中最后一个字母一定是其它字母序列中的字母。*  

**输入样例：**
china
1

**输出样例：**
cahbicndae

```c++
int main()
{
    int interval;
    char source[30] = {};
    char arr[60] = {};
    cin.getline(source, 60);
    cin >> interval;
    epy(source, arr, interval);
    for (int i = 0; i < strlen(arr); i++)
    {
        cout << arr[i];
    }
    return 0;
}

void epy(char source[], char(&arr)[60], int interval)
{
    int length = strlen(source);
    int k = 0, flag = 0;
    for (int i = 0; i < length; )
    {
        arr[k++] = source[i++];
        flag++;
        if (flag == interval)
        {
            arr[k++] = getSerial();
            flag = 0;
        }
        // when reach the end
        if (source[i] == '\0') {
            // and from the last character to last serial less than one interval
            if (flag > 0) {
                arr[k] = getSerial();
            }
        }
    }
}

char getSerial()
{
    static int index = 0;
    char temp;
    const char serial[] = { 'a','b','c','d','e' };
    temp = serial[index];
    if (index == 4)
    {
        index = 0;
    }
    else
    {
        index++;
    }
    return temp;
}
```

---
