---
title: 哈工大《操作系统》实验3 - 进程运行轨迹的跟踪与统计
date: 2020-04-05 18:53:50
categories: MOOC
tags:
---

在 Linux0.11 上实现进程运行轨迹的跟踪<!-- more -->

> 进程从创建（Linux 下调用 fork()）到结束的整个过程就是进程的生命期，进程在其生命期中的运行轨迹实际上就表现为进程状态的多次切换，如进程创建以后会成为就绪态；当该进程被调度以后会切换到运行态；在运行的过程中如果启动了一个文件读写操作，操作系统会将该进程切换到阻塞态（等待态）从而让出 CPU；当文件读写完毕以后，操作系统会在将其切换成就绪态，等待进程调度算法来调度该进程执行……

实验的内容其实就是在系统调用进程的相关代码处，添加几条输出语句，实现跟踪进程状态的改变

- 修改的文件

```text
linux-0.11/init/main.c
linux-0.11/kernel/fork.c
linux-0.11/kernel/sched.c
linux-0.11/kernel/exit.c
~/process.c
```

## 编写样本程序(process.c)

这个文件要在0.11下编译后运行，模拟运行一个多进程程序

```c
#include <stdio.h>
#include <unistd.h>
#include <time.h>
#include <sys/times.h>

#define HZ 100

void cpuio_bound(int last, int cpu_time, int io_time);

int main(int argc, char * argv[])
{
    // 占用10秒的CPU时间
    invoke(10, 1, 0);
    // 以I/O为主要任务
    invoke(10, 0, 1);
    // CPU和I/O各1秒钟轮回
    invoke(10, 1, 1);
    // 较多的I/O，较少的CPU：
    // I/O时间是CPU时间的9倍
    invoke(10, 1, 9);

    wait(NULL);
    return 0;
}

/* fork()用法参考自CSAPP */
void invoke(int last, int cpu_time, int io_time)
{
    pid_t pid;
    pid = fork();
    if (pid == 0) /* child */
    {
        cpuio_bound(last, cpu_time, io_time);
        printf("[Child] %d, parent is %d\n", getpid(), getppid());
        exit(0);
    }
    /* parent */
    printf("[Parent] %d\n", getpid());
    wait(NULL);
}

// 模拟一个持续时间last，cpu用时xx，io用时占xx的程序，last>cpu+io则反复
void cpuio_bound(int last, int cpu_time, int io_time)
{
    ...
}
```

一开始不知道怎么写调用，翻CSAPP看了下，按照书中的例子实现了

## log 文件

记录操作系统在处理进程时进程状态改变的轨迹

### 打开 log 文件

> 为了能尽早开始记录，应当在内核启动时就打开 log 文件。

按照实验指示修改`init/main.c`

```c
//……
move_to_user_mode();

/***************添加开始***************/
setup((void *) &drive_info);

// 建立文件描述符0和/dev/tty0的关联
(void) open("/dev/tty0",O_RDWR,0);

//文件描述符1也和/dev/tty0关联
(void) dup(0);

// 文件描述符2也和/dev/tty0关联
(void) dup(0);

(void) open("/var/process.log",O_CREAT|O_TRUNC|O_WRONLY,0666);

/***************添加结束***************/

if (!fork()) {        /* we count on this going ok */
    init();
}
//……
```

### 写 log 文件

实验提供了例子

```c
// 向log文件输出跟踪进程运行轨迹
fprintk(3, "%ld\t%c\t%ld\n", current->pid, 'R', jiffies);
```

修改这个语句，增加到进程管理的代码中，就可以实现写 log 文件了

## 寻找状态切换点

> 总的来说，Linux 0.11 支持四种进程状态的转移：就绪到运行、运行到就绪、运行到睡眠和睡眠到就绪，此外还有新建和退出两种情况。其中就绪与运行间的状态转移是通过 schedule()（它亦是调度算法所在）完成的；运行到睡眠依靠的是 sleep_on() 和 interruptible_sleep_on()，还有进程主动睡觉的系统调用 sys_pause() 和 sys_waitpid()；睡眠到就绪的转移依靠的是 wake_up()。所以只要在这些函数的适当位置插入适当的处理语句就能完成进程运行轨迹的全面跟踪了。

### 进程的开始

`kernel/fork.c`

```c
int copy_process(int nr,long ebp,long edi,long esi,long gs,long none,
                long ebx,long ecx,long edx,
                long fs,long es,long ds,
                long eip,long cs,long eflags,long esp,long ss)
{
    ...
        p->state = TASK_UNINTERRUPTIBLE;
        p->pid = last_pid;
        p->father = current->pid;
        p->counter = p->priority;
        p->signal = 0;
        p->alarm = 0;
        p->leader = 0;          /* process leadership doesn't inherit */
        p->utime = p->stime = 0;
        p->cutime = p->cstime = 0;
        p->start_time = jiffies;
        fprintk(3, "%ld\t%c\t%ld\n", p->pid, 'N', jiffies);
  ...
        p->state = TASK_RUNNING;        /* do this last, just in case */
        fprintk(3, "%ld\t%c\t%ld\n", p->pid, 'J', jiffies);
        return last_pid;
}
```

### 调度算法

`kernel/sched.c`

```c
void schedule(void)
{
    int i,next,c;
    struct task_struct ** p;

/* check alarm, wake up any interruptible tasks that have got a signal */

    for(p = &LAST_TASK ; p > &FIRST_TASK ; --p)
        if (*p) {
            if ((*p)->alarm && (*p)->alarm < jiffies) {
                    (*p)->signal |= (1<<(SIGALRM-1));
                    (*p)->alarm = 0;
                }
            if (((*p)->signal & ~(_BLOCKABLE & (*p)->blocked)) &&
            (*p)->state==TASK_INTERRUPTIBLE)
                {
                    (*p)->state=TASK_RUNNING;
                    fprintk(3, "%ld\t%c\t%ld\n", (*p)->pid, 'J', jiffies);
                }
        }

/* this is the scheduler proper: */

    while (1) {
        c = -1;
        next = 0;
        i = NR_TASKS;
        p = &task[NR_TASKS];
        while (--i) {
            if (!*--p)
                continue;
            if ((*p)->state == TASK_RUNNING && (*p)->counter > c)
                c = (*p)->counter, next = i;
        }
        if (c) break;
        for(p = &LAST_TASK ; p > &FIRST_TASK ; --p)
            if (*p)
                (*p)->counter = ((*p)->counter >> 1) +
                        (*p)->priority;
    }
    if(current->pid != task[next] ->pid)
    {
        if(current->state == TASK_RUNNING)
            fprintk(3, "%ld\t%c\t%ld\n", current->pid, 'J', jiffies);
        fprintk(3, "%ld\t%c\t%ld\n", task[next]->pid, 'R', jiffies);
    }
    switch_to(next);
}
```

### 进入睡眠

`kernel/sched.c`

```c
int sys_pause(void)
{
    current->state = TASK_INTERRUPTIBLE;
    if(current->pid != 0)
        fprintk(3, "%ld\t%c\t%ld\n", current->pid, 'W', jiffies);
    schedule();
    return 0;
}
```

### 睡眠到就绪

`kernel/sched.c`

```c
void sleep_on(struct task_struct **p)
{
    struct task_struct *tmp;

    if (!p)
        return;
    if (current == &(init_task.task))
        panic("task[0] trying to sleep");
    tmp = *p;
    *p = current;
    current->state = TASK_UNINTERRUPTIBLE;
    fprintk(3, "%ld\t%c\t%ld\n", current->pid, 'W', jiffies);
    schedule();
    if (tmp)
    {
        tmp->state=0;
        fprintk(3, "%ld\t%c\t%ld\n", current->pid, 'J', jiffies);
    }
}

void interruptible_sleep_on(struct task_struct **p)
{
    struct task_struct *tmp;

    if (!p)
        return;
    if (current == &(init_task.task))
        panic("task[0] trying to sleep");
    tmp=*p;
    *p=current;
repeat:    current->state = TASK_INTERRUPTIBLE;
    fprintk(3, "%ld\t%c\t%ld\n", current->pid, 'W', jiffies);
    schedule();
    if (*p && *p != current) {
        (**p).state=0;
        fprintk(3, "%ld\t%c\t%ld\n", (*p)->pid, 'J', jiffies);
        goto repeat;
    }
    *p=NULL;
    if (tmp)
    {
        tmp->state=0;
        fprintk(3, "%ld\t%c\t%ld\n", (*p)->pid, 'J', jiffies);
    }
}

void wake_up(struct task_struct **p)
{
    if (p && *p) {
        fprintk(3, "%ld\t%c\t%ld\n", (*p)->pid, 'J', jiffies);
        *p=NULL;
    }
}

```

### 进程结束

`kernel/exit.c`

```c
int do_exit(long code)
{
    ...
    current->state = TASK_ZOMBIE;
    fprintk(3, "%ld\t%c\t%ld\n", current->pid, 'E', jiffies);
    current->exit_code = code;
    tell_father(current->father);
    schedule();
    return (-1);    /* just to suppress warnings */
}
```

## 数据统计

装载 0.11，执行 stat_log.py

```text
$ python stat_log.py ../oslab/hdc/var/process.log 12 -g | more
(Unit: tick)
Process   Turnaround   Waiting   CPU Burst   I/O Burst
     12         4031         1           6        4024
Average:     4031.00      1.00
Throughout: 0.02/s

Collect statisics by ehye
-----===< COOL GRAPHIC OF SCHEDULER >===-----

             [Symbol]   [Meaning]
         ~~~~~~~~~~~~~~~~~~~~~~~~~~~
             number   PID or tick
              "-"     New or Exit
              "#"       Running
              "|"        Ready
              ":"       Waiting
                    / Running with
              "+" -|     Ready
                    \and/or Waiting

-----===< !!!!!!!!!!!!!!!!!!!!!!!!! >===-----

11667 -12
11668 #12
11669 #
11670 :12
11671 :
11672 :
11673 :
11674 :
11675 :
11676 :
11677 :
11678 :
11679 :
11680 :
...
```

---

相关资源

[操作系统 中国大学MOOC(慕课)](https://www.icourse163.org/learn/HIT-1002531008)
[操作系统原理与实践 - 进程运行轨迹的跟踪与统计 - 实验楼](https://www.shiyanlou.com/courses/115/learning/?id=570)
