---
title: 哈工大《操作系统》实验2 - 添加系统调用
date: 2020-04-01 16:02:48
categories: MOOC
tags:
---

在 Linux 0.11上添加两个系统调用，并编写两个简单的应用程序测试它们<!-- more -->

---

> 在通常情况下，调用系统调用和调用一个普通的自定义函数在代码上并没有什么区别，但调用后发生的事情有很大不同。调用自定义函数是通过call指令直接跳转到该函数的地址，继续运行。而调用系统调用，是调用系统库中为该系统调用编写的一个接口函数，叫API（Application Programming Interface）。API并不能完成系统调用的真正功能，它要做的是去调用真正的系统调用

- 修改的的文件

```text
linux-0.11/include/linux/sys.h
linux-0.11/kernel/system_call.s
linux-0.11/kernel/who.c
linux-0.11/kernel/Makefile
/usr/root/include/unistd.h
~/iam.c
~/whoami.c
```

## 修改

- 增加系统调用声明

    `linux-0.11/include/linux/sys.h`

    ```h
    extern int sys_whoami();
    extern int sys_iam();
    ```

    然后下方的 sys_call_table 增加 `sys_whoami` 和 `sys_iam`

    ```h
    fn_ptr sys_call_table[] = { sys_setup, sys_exit, sys_fork, sys_read,
    ***
    sys_setreuid,sys_setregid, sys_whoami, sys_iam };
    ```

    `linux-0.11/kernel/system_call.s`

    ```x86asm
    nr_system_calls = 74 #这是系统调用总数。如果增删了系统调用，必须做相应修改
    ```

- 实现系统调用函数

    `linux-0.11/include/kernal/who.c`

    ```c
    #include <asm/segment.h>
    #include <errno.h>
    #include <string.h>

    char _myname[24];

    int sys_iam(const char *name)
    {
        char str[25];
        int i = 0;

        do
        {
            // get char from user input
            str[i] = get_fs_byte(name + i);
        } while (i <= 25 && str[i++] != '\0');

        if (i > 24)
        {
            errno = EINVAL;
            i = -1;
        }
        else
        {
            // copy from user mode to kernel mode
            strcpy(_myname, str);
        }

        return i;
    }

    int sys_whoami(char *name, unsigned int size)
    {
        int length = strlen(_myname);
        printk("%s\n", _myname);

        if (size < length)
        {
            errno = EINVAL;
            length = -1;
        }
        else
        {
            int i = 0;
            for (i = 0; i < length; i++)
            {
                // copy from kernel mode to user mode
                put_fs_byte(_myname[i], name + i);
            }
        }

        return length;
    }
    ```

    关键点在于如何在核心态同用户态进行数据交换，别误以为可以直接访问全局变量`_myname`，这个是核心态的变量，因此要用 `get_fs_byte()` 和 `put_fs_byte()` 进行读写

## 编译

- 修改 kernel/Makefile

    增加 who.o

    ```makefile
    OBJS  = sched.o system_call.o traps.o asm.o fork.o \
        panic.o printk.o vsprintf.o sys.o exit.o \
        signal.o mktime.o who.o
    ```

    添加 who.s who.o: who.c ../include/linux/kernel.h ../include/unistd.h

    ```makefile
    #### Dependencies:
    who.s who.o: who.c ../include/linux/kernel.h ../include/unistd.h
    exit.s exit.o: exit.c ../include/errno.h ../include/signal.h \
    ../include/sys/types.h ../include/sys/wait.h ../include/linux/sched.h \
    ../include/linux/head.h ../include/linux/fs.h ../include/linux/mm.h \
    ../include/linux/kernel.h ../include/linux/tty.h ../include/termios.h \
    ../include/asm/segment.h
    ```

    > Makefile 修改后，和往常一样 make all 就能自动把 who.c 加入到内核中了。

- 增加系统调用号

    `/usr/root/include/unistd.h`

    ```h
    #define __NR_whoami 72
    #define __NR_iam  73
    ```

## 测试

- 编写测试程序

    `~/whoami.c`

    ```c
    /* 有它，_syscall1 等才有效。详见unistd.h */
    #define __LIBRARY__

    /* 有它，编译器才能获知自定义的系统调用的编号 */
    #include "unistd.h"

    #include <stdio.h>
    #include <errno.h>

    /* whoami()在用户空间的接口函数 */
    _syscall2(int, whoami,char*,name,unsigned int,size);

    int main(char *arg) {
        char myname[50];
        whoami(myname, 50);
        return 0;
    }
    ```

    {% note info %}
    myname 长度只要大于 who.c 里 _myname 的长度就行
    {% endnote %}

    `~/iam.c`

    ```c
    /* 有它，_syscall1 等才有效。详见unistd.h */
    #define __LIBRARY__

    /* 有它，编译器才能获知自定义的系统调用的编号 */
    #include "unistd.h"

    #include <stdio.h>
    #include <errno.h>

    /* iam()在用户空间的接口函数 */
    _syscall1(int, iam, const char*, name);

    int main(int argc, char* argv[]) {
        iam(argv[1]);
        return 0;
    }
    ```

    {% note info %}
    注意是取第二个参数
    {% endnote %}

- 编译

    ```shell
    gcc -o iam iam.c
    gcc -o whoami whoami.c
    ```

- 运行测试

    ```shell
    [/usr/root]# ./iam ehye
    [/usr/root]# ./whoami
    ehye
    ```

## 总结

可以总结出向 Linux 0.11 添加一个系统调用 foo() 的步骤

1. 修改 include/linux/sys.h 在`sys_call_table`数组最后加入`sys_foo`，再加上`extern rettype sys_foo()`

2. 在`include/unistd.h`添加宏`#define __NR_foo num`

3. 在`kernel/system_call.s`添加`nr_system_calls = num`

4. 在`kernel/`中添加 foo.c ，实现系统调用`sys_foo()`

5. 修改Makefile，`OBJS`一节添加 foo.c

6. 编写调用程序时要加上

   - #define __LIBRARY__
   - #include <unistd.h>
   - _syscallN()（N视具体参数数量而定）

---

相关资源

[操作系统 中国大学MOOC(慕课)](https://www.icourse163.org/learn/HIT-1002531008)
[操作系统原理与实践 - 系统调用 - 实验楼](https://www.shiyanlou.com/courses/115/learning/?id=569)
