---
title: 哈工大《操作系统》实验8 - 进程运行轨迹的跟踪与统计
date: 2020-04-30 18:50:39
categories: MOOC
tags:
---

修改 Linux 0.11 的终端设备处理代码，对键盘输入和字符显示进行非常规的控制<!-- more -->

> 在初始状态，一切如常。用户按一次 F12 后，把应用程序向终端输出所有字母都替换为“*”。用户再按一次 F12，又恢复正常。第三次按 F12，再进行输出替换。依此类推。

以 ls 命令为例：

正常情况：

```text
# ls
hello.c hello.o hello
```

第一次按 F12，然后输入 ls：

```text
# **
*****.* *****.* *****
```

第二次按 F12，然后输入 ls：

```text
# ls
hello.c hello.o hello
```

第三次按 F12，然后输入 ls：

```text
# **
*****.* *****.* *****
```

---

相对简单的一次实验，根据F12键盘中断，设置一个开关，再更改输出即可

- 修改的文件

```text
include/linux/sched.h
kernel/sched.c
kernel/chr_drv/keyboard.S
kernel/chr_drv/console.c
```

## 修改

- sched.h

    添加一个全局变量作为开关

    ```c
    int f12_flag;
    ```

    因为 console.c 引用 sched.h ，所以不能在 sched.c 里定义

- sched.c

    没错又是 sched.c

    ```c
    f12_flag = 0;
    void switch_f12(void)
    {
        if(f12_flag) f12_flag = 0;
        else f12_flag = 1;
    }
    ```

    一个开关函数，课件里有提示了

- keyboard.S

    修改功能键扫描码处理块，参考自《Linux 内核 0.11 详细注释》

    ```as
    /*
    * this routine handles function keys
    */
    func:
        subb $0x3B,%al
        jb end_func
        cmpb $9,%al
        jbe ok_func
        subb $18,%al // 是功能键F11、F12？
        cmpb $10,%al // 是功能键F11？
        jb end_func  // 不是，则不处理，返回
        cmpb $11,%al // 是功能键F12？
        ja end_func  // 不是，则不处理，返回
        call switch_f12
    ```

    增加跳转`switch_f12`的语句

- console.c

    ```c
    void con_write(struct tty_struct * tty)
    {
        int nr;
        char c;

        nr = CHARS(tty->write_q);
        while (nr--) {
            GETCH(tty->write_q,c);
            switch(state) {
                case 0:
                    if (c>31 && c<127) {
                        if (x>=video_num_columns) {
                            x -= video_num_columns;
                            pos -= video_size_row;
                            lf();
                        }
                        /* 只针对字母 */
                        if(f12_flag && ((c>64 && c<91) || (c>96 && c<123)))
                            c='*';
                        __asm__("movb attr,%%ah\n\t"
                            "movw %%ax,%1\n\t"
                            ::"a" (c),"m" (*(short *)pos)
                            );
                        pos += 2;
                        x++;
                    } 
                    ...
                case 1: ...
                case 2: ...
                case 3: ...
                case 4: ...
            }
        }
        set_cursor();
    }
    ```

    在con_write，真正写显示器前根据 flag 改变输出字符

## 效果

{% asset_img switch_f12.jpg %}

## 存在的问题

- F12原有的功能没能去掉

- 当输出星号后，下一行会有一个`L`

---

相关资源

[操作系统 中国大学MOOC(慕课)](https://www.icourse163.org/learn/HIT-1002531008)
[操作系统原理与实践 - 终端设备的控制 - 实验楼](https://www.shiyanlou.com/courses/115/learning/?id=574)
