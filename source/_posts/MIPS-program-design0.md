---
title: MIPS汇编程序设计
date: 2017-10-10 15:27:12
categories: MOOC
tags:
	- MIPS
	- 汇编
---
北京大学-计算机组成原理 第二讲 指令系统体系结构 互评作业<!--more-->

使用MARS（MIPS Assembler and Runtime Simulator）集成开发环境作为实验平台
考察MIPS汇编程序设计，要求熟悉MIPS指令系统的常见指令，掌握MARS集成开发环境基本使用方法，完成自编MIPS汇编语言程序的上机调试过程

---

# 第一题：用系统功能调用实现简单输入输出

利用系统功能调用从键盘输入，转换后在屏幕上显示，具体要求如下：

1. 如果输入的是字母（A~Z，区分大小写）或数字（0~9），则将其转换成对应的英文单词后在屏幕上显示，对应关系见下表

2. 若输入的不是字母或数字，则在屏幕上输出字符`*`

3. 每输入一个字符，即时转换并在屏幕上显示

4. 支持反复输入，直到按`?`键结束程序

<table><tbody><tr><td><strong>A</strong></td><td>Alpha</td><td><strong>N</strong></td><td>November</td><td><strong>1</strong></td><td>First</td><td><strong>a</strong></td><td>alpha</td><td><strong>n</strong></td><td>november</td></tr><tr><td><strong>B</strong></td><td>Bravo</td><td><strong>O</strong></td><td>Oscar</td><td><strong>2</strong></td><td>Second</td><td><strong>b</strong></td><td>bravo</td><td><strong>o</strong></td><td>oscar</td></tr><tr><td><strong>C</strong></td><td>China</td><td><strong>P</strong></td><td>Paper</td><td><strong>3</strong></td><td>Third</td><td><strong>c</strong></td><td>china</td><td><strong>p</strong></td><td>paper</td></tr><tr><td><strong>D</strong></td><td>Delta</td><td><strong>Q</strong></td><td>Quebec</td><td><strong>4</strong></td><td>Fourth</td><td><strong>d</strong></td><td>delta</td><td><strong>q</strong></td><td>quebec</td></tr><tr><td><strong>E</strong></td><td>Echo</td><td><strong>R</strong></td><td>Research</td><td><strong>5</strong></td><td>Fifth</td><td><strong>e</strong></td><td>echo</td><td><strong>r</strong></td><td>research</td></tr><tr><td><strong>F</strong></td><td>Foxtrot</td><td><strong>S</strong></td><td>Sierra</td><td><strong>6</strong></td><td>Sixth</td><td><strong>f</strong></td><td>foxtrot</td><td><strong>s</strong></td><td>sierra</td></tr><tr><td><strong>G</strong></td><td>Golf</td><td><strong>T</strong></td><td>Tango</td><td><strong>7</strong></td><td>Seventh</td><td><strong>g</strong></td><td>golf</td><td><strong>t</strong></td><td>tango</td></tr><tr><td><strong>H</strong></td><td>Hotel</td><td><strong>U</strong></td><td>Uniform</td><td><strong>8</strong></td><td>Eighth</td><td><strong>h</strong></td><td>hotel</td><td><strong>u</strong></td><td>uniform</td></tr><tr><td><strong>I</strong></td><td>India</td><td><strong>V</strong></td><td>Victor</td><td><strong>9</strong></td><td>Ninth</td><td><strong>i</strong></td><td>india</td><td><strong>v</strong></td><td>victor</td></tr><tr><td><strong>J</strong></td><td>Juliet</td><td><strong>W</strong></td><td>Whisky</td><td><strong>0</strong></td><td>zero</td><td><strong>j</strong></td><td>juliet</td><td><strong>w</strong></td><td>whisky</td></tr><tr><td><strong>K</strong></td><td>Kilo</td><td><strong>X</strong></td><td>X-ray</td><td> </td><td> </td><td><strong>k</strong></td><td>kilo</td><td><strong>x</strong></td><td>x-ray</td></tr><tr><td><strong>L</strong></td><td>Lima</td><td><strong>Y</strong></td><td>Yankee</td><td> </td><td> </td><td><strong>l</strong></td><td>lima</td><td><strong>y</strong></td><td>yankee</td></tr><tr><td><strong>M</strong></td><td>Mary</td><td><strong>Z</strong></td><td>Zulu</td><td> </td><td> </td><td><strong>m</strong></td><td>mary</td><td><strong>z</strong></td><td>zulu</td></tr></tbody></table>

## 思路

由于 alphabet 中的字符串保存的时候将以`\0`结尾，我又在单词后面加了一个`\n`
故可用`=`替换这些字符，将文本保存形似 ` Alpha== Bravo== China` 的计算字符串
以计算空格的总数量来确定每个单词开头字母的位置，从而计算出偏移量

为了算出每个单词的在数组里的位置，使用了Java编写一段用于计算偏移量的程序：

*args[0]：保存计算字符串的文本的路径*

```java
public class Main {

    public static void main(String[] args) throws IOException {
        File f = new File(args[0]);
        InputStreamReader isr = new InputStreamReader(new FileInputStream(f));
        BufferedReader br = new BufferedReader(isr);
        String s = br.readLine();
        char[] chars = s.toCharArray();
        Integer sum = 0;
        System.out.println(s);
        for (char c : chars) {
            sum++;
            if (c == ' ') System.out.print(sum - 1 + ",");
        }
    }

}
```

## 解答
***用Java写汇编算作弊吗 ~(￣▽￣)~***


```mips
.data
alphabetU:
    .asciiz " Alpha\n"," Bravo\n"," China\n"," Delta\n"," Echo\n"," Foxtrot\n"," Golf\n"," Hotel\n",
    " India\n"," Juliet\n"," Kilo\n"," Lima\n"," Mary\n"," November\n"," Oscar\n"," Paper\n"," Quebec\n",
    " Research\n"," Sierra\n"," Tango\n"," Uniform\n"," Victor\n"," Whisky\n"," X-ray\n"," Yankee\n"," Zulu\n"
alphabetL:
    .asciiz " alpha\n"," bravo\n"," china\n"," delta\n"," echo\n"," foxtrot\n"," golf\n"," hotel\n"," india\n",
    " juliet\n"," kilo\n"," lima\n"," mary\n"," november\n"," oscar\n"," paper\n"," quebec\n"," research\n",
    " sierra\n"," tango\n"," uniform\n"," victor\n"," whisky\n"," x-ray\n"," yankee\n"," zulu\n"
al_offset:
    .word 0,8,16,24,32,39,49,56,64,72,81,88,95,102,113,121,129,138,149,158,166,176,185,194,202,211
	# 使用 Java 算出的偏移量
number: 
    .asciiz " Zero\n"," First\n"," Second\n"," Third\n"," Fourth\n"," Fifth\n"," Sixth\n"," Seventh\n"," Eighth\n"," Ninth\n"
n_offset:
    .word 0,7,15,24,32,41,49,57,67,76
exit_str: .asciiz "#Stop program#"

.text
.globl main
main:
    li $v0, 12                # $v0 contains character read
    syscall
    beq $v0, '?', exit        # if (v0 == '?') then exit    

    # is symble ?
    sub $t1, $v0, '0'         # t1 = v0 - '0'
    blt $t1, $zero, symble    # if (t1 < 0) then symble
    
    # is number ?
    sub $t2, $t1, 10          # t2 = t1 - 10
    blt $t2, $zero, getnum    # if (t2 < 0) then getnum

    # is upper case?
    sub $t2, $v0, 91
    slt $s3, $t2, $0         # if (v0 < 'Z') then s3 = 1
    sub $t3, $v0, 64
    sgt $s4, $t3, $0         # if (v0 > 'A') then s4 = 1
    and $s0, $s3, $s4        # s0 = s3 && s4
    bnez $s0, getuword       # if (s0 == 1) then getword

    # is lower case?
    sub $t2, $v0, 123
    slt $s3, $t2, $0         # if (v0 <= 'z') then s3 = 1
    sub $t3, $v0, 96  
    sgt $s4, $t3, $0         # if (v0 >= 'a') then s4 = 1
    and $s0, $s3, $s4        # s0 = s3 && s4
    bnez $s0, getlword       # if (s0 == 1) then getword
    j symble

getnum:
    
    add $t2, $t2, 10        # judge during [is number ?] 补回判断时减去的ASCII
    sll $t2, $t2, 2         # every word use 1 byte (no.9 at address of 9<<2)
    #
    #add $a0, $t2, $zero
    #li $v0, 1
    #syscall
    #
    la $s0, n_offset        # load address to s0
    add $s0, $s0, $t2       # s0 = s0 + offset
    lw $s1, ($s0)           # Load Word at address of s0
    add $a0, $0, '*'        # '*' ascii
    li $v0, 11
    la $a0, number          # load address to a0
    add $a0, $a0, $s1    
    li $v0, 4
    syscall
    j main

getuword:
    # a0: alphabet
    # s1: offset
    sub $t3, $t3, 1
    sll $t3, $t3, 2
    la $s0, al_offset        # load address
    add $s0, $s0, $t3
    lw $s1, ($s0)            # load word
    la $a0, alphabetU        # load address
    add $a0, $s1, $a0
    li $v0, 4                # print null-terminated string string at $a0
    syscall
    j main

getlword:
    # a0: alphabet
    # s1: offset
    sub $t3, $t3, 1
    sll $t3, $t3, 2
    la $s0, al_offset        # load address
    add $s0, $s0, $t3
    lw $s1, ($s0)            # load word
    la $a0, alphabetL        # load address
    add $a0, $s1, $a0
    li $v0, 4                # print null-terminated string string at $a0
    syscall
    j main

symble:
    li  $v0, 11             # $a0 = char to print
    add $a0, $t0, 32        # load value into argument register $a0
    syscall
    add $a0, $a0, 10        # 32 + 10 = '*'
    syscall
    sub $a0, $a0, 32        # '*' - 32 == '\n'
    syscall
    j main

exit:
    add $a0, $0, '\n'
    li $v0, 11
    syscall
    la $a0, exit_str
    li $v0, 4
    syscall

```

# 第二题：字符串查找比较

利用系统功能调用从键盘输入一个字符串，然后输入单个字符，查找该字符串中是否有该字符（区分大小写）。具体要求如下：

1. 如果找到，则在屏幕上显示：

	`Success! Location: X`

	其中，X为该字符在字符串中第一次出现的位置

2. 如果没找到，则在屏幕上显示：

	`Fail!`

3. 输入一个字符串后，可以反复输入希望查询的字符，直到按`?`键结束程序

4. 每个输入字符独占一行，输出查找结果独占一行，位置编码从1开始。

提示：为避免歧义，字符串内不包含`?`符号

格式示例如下：

```
abcdefgh

a

Success! Location: 1

x

Fail!
```

## 思路
因为只是查找一个字符串中的一个字符，因此可以直接遍历每个字符串，进行字符匹配，具体到实现，就是高级语言中的 if-else 以及 for 所实现的功能。

## 解答
```mips
.data
    suces_str: .asciiz "\nSuccess! Location: "
    fail_str: .asciiz "\nFail!"
    buffer: .space 20
    exit_str: .asciiz "\n#Stop program#"
    str1:  .asciiz "Enter string: "
    str2:  .asciiz "Enter char: "

.text
.globl main
main:
    # a0 = input/result string
    # a1 = input string length
    # a2 = input char
    # a3 = loop donefail

    # initialize
    add $a3, $zero, $zero
    add $t0, $zero, $zero    # i = 0
    # take string
    la $a0, buffer           # load byte address
    li $a1, 20               # length of string
    li $v0, 8                # read string at a0
    syscall

takechar:
    li $v0, 12               # char read to v0
    syscall
    beq $v0, '?', exit       # if (v0 == '?') then exit
    move $a2, $v0            # a2 = char
    
search:
    # t0 = int i
    beq $t0, $a1, donefail   # if (i = length) then donefail
    la $a0, buffer           # else load byte
    add $t2, $a0, $t0        # t2 = &str + offset
    lb $s1, ($t2)            # s1 = *t2
    beq $s1, $a2, sucess     # if (this.char = char) then sucess
    add $t0, $t0, 1          # else i++
    j search

fail:
    la $a0, fail_str
    li $v0, 4
    syscall
    li $v0, 11
    add $a0, $zero, '\n'
    syscall
    j takechar
      
sucess:
    la $a0, suces_str
    li $v0, 4
    syscall
    li $v0, 1
    add $a0, $t0, 1          # print index begin from 1    
    syscall
    li $v0, 11
    add $a0, $zero, '\n'
    syscall
    j takechar

donefail:
    add $a3, $zero, 1        # flag donefail = 1
    add $t0, $zero, $zero    # reset int i = 0
    j fail

exit:
    la $a0, exit_str
    li $v0, 4
    syscall
```