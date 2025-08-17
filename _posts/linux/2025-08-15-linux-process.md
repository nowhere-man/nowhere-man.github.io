---
layout: post
title:  Linux 进程管理
slug: process-management
categories: [Linux]
tags: [Linux]
---

## 进程和程序
+ 程序：储存在外部存储的一个可执行文件
+ 进程：在内存中运行，处于执行期间的程序

## 进程和线程
+ 进程：是操作系统**资源分配**的基本单位，进程之间的地址空间和资源相互独立，一个进程至少有一个线程。
+ 线程：是处理器**任务调度和执行**的基本单位，同一进程的所有线程共享本进程的地址空间和资源。

## 进程ID
```c
// 获取当前进程ID
pid_t getpid();

// 获取当前进程的父进程ID
pid_t getppid();
```

## 进程创建
### fork() 函数

fork函数从已存在进程中创建一个新进程。新进程为子进程，而原进程为父进程。
```c
#include <unistd.h>

pid_t fork();
```
进程调用fork后：
+ 分配新的内存块和内核数据结构给子进程。
+ 将父进程部分数据结构内容拷贝至子进程。
+ 添加子进程到系统进程列表当中。
+ fork返回，开始调度器调度。

fork函数时调用一次，返回两次。在父进程和子进程中各调用一次。子进程中返回值为0，父进程中返回值为子进程的PID。可以根据返回值的不同让父进程和子进程执行不同的代码。

### vfork()函数
```c
#include <unistd.h>

pid_t vfork();
```
fork创建子进程，把父进程数据空间、堆和栈复制一份；**vfork创建子进程，与父进程内存数据共享**。

vfork先保证子进程先执行，当子进程调用exit()或者exec后，父进程才往下执行。

## 进程等待
### wait()
```c
// 暂停正在调用进程的执行，直到它的一个子进程终止。
// status：获取子进程退出状态，不关心则可以设置成为NULL。
// return：成功时返回终止子进程的进程ID，出错时返回-1。
pid_t wait(int *status);
```

### waitpid()

```c
// 暂停正在调用进程的执行，直到pid参数指定的子进程改变状态。
pid_t waitpid(pid_t pid, int *status, int options);
```

pid:
+ pid = -1，等待任意一个子进程，与wait等效。
+ pid > 0，等待其进程ID与pid相等的子进程，等待指定的子进程。

waitpid()默认仅等待终止的子进程，但可以通过options参数进行修改。
options:
+ 如果设为0：默认，是阻塞式等待，与 wait 等效。
+ 如果设为WNOHANG：是非阻塞等待。

## 进程终止

```c
void exit(int status);

void _exit(int status);
```
+ exit：在进程退出的时候，会进行后续资源处理（比如刷新缓冲区）。
+ _exit：在进程退出的时候，不会进行后续资源处理，直接终止进程。

## Linux 进程间通信
+ 管道
+ FIFO
+ 消息队列
+ 信号量
+ 共享内存区
