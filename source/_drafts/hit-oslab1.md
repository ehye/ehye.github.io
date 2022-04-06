---
title: 哈工大《操作系统》实验1 - 操作系统的引导
date: 2020-04-01 15:02:27
categories: MOOC
tags:
---

对计算机和 Linux 0.11的引导过程进行初步的了解<!-- more -->

- 涉及到的所有文件

```text
linux-0.11/boot/bootsect.s
linux-0.11/boot/setup.s
linux-0.11/tools/build.c
```

# 修改

改写 bootsect.s 打印一段提示信息 `Hello OS world, my name is ehye`

`linux-0.11/boot/bootsect.s`

```x86asm
entry _start
_start:
    mov ah,#0x03
    xor bh,bh
    int 0x10
    mov cx,#36
    mov bx,#0x0007
    mov bp,#msg1
    mov ax,#0x07c0
    mov es,ax
    mov ax,#0x1301
    int 0x10
inf_loop:
    jmp inf_loop
msg1:
    .byte   13,10
    .ascii  "Hello OS world, my name is ehye"
    .byte   13,10,13,10
.org 510
boot_flag:
    .word   0xAA55

```

# 编译



# 测试



---

相关资源

[操作系统 中国大学MOOC(慕课)](https://www.icourse163.org/learn/HIT-1002531008)
[操作系统原理与实践 - 操作系统的引导 - 实验楼](https://www.shiyanlou.com/courses/115/learning/?id=568)

