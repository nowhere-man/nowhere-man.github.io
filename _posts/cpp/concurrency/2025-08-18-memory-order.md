---
layout: post
title: C++并发编程 6：内存模型
slug: cpp-concurrency-memory-model
categories: [C++总结]
tags: [C++并发编程]
---
## 为什么需要内存模型

开发者编写的代码和最终运行的程序往往会存在较大的差异，而运行结果与开发者预想一致，只是一种假象，之所以会产生差异，原因主要由于：
+ 编译器优化导致的指令重排序
+ CPU乱序执行
+ CPU Cache不一致性

关于CPU Cache不一致性：
现代的主流CPU几乎都会包含多个核以及多级Cache，每个CPU核在运行的时候，都会优先考虑离自己最近的Cache，一旦命中就直接使用Cache中的数据。所有Cache不命中才会取主存的数据。 但是每个核之间的Cache以及每一层之间的Cache，数据常常是不一致的。而同步这些数据是需要消耗时间的。

这就会造成一个问题，那就是：某个CPU核修改了一个数据，没有同步的让其他核知道，于是就存在了Cache不一致的情况。

```cpp
int val;
bool flag {false};

// thread1中执行
void write() {
    val = 1;           // w1
    flag = true;       // w2
}

// thread2中执行。
void read() {
    while(!flag) {;}   // r1
    assert(val == 1);  // r2
}
```
上面代码中，由于上面的原因，w1不一定比w2先执行，r1也不一定比r2先执行，所以断言是不一定成立的。


## 内存模型

### 宽松内存顺序
memory_order_relaxed完全不提供任何的顺序性约束保证，唯一提供的保证就是操作本身是符合原子性的。

### 适用于读取操作的内存顺序

memory_order_consume
+ 本线程：依赖于本次读取操作的值的、其它的相关的读取和写入操作语句的顺序不允许被重新排列到该读取操作前面
+ 其它线程： 对同样的内存位置的变量的写入操作的释放性顺序的效果在本线程中可见（和memory_order_release相互配对使用）

memory_order_acquire
+ 本线程：依赖于同样内存位置的原子量的读写操作不允许被重排于该操作之前
+ 其它线程：写入释放原子量的操作在本线程中可见（和memory_order_release相互配对使用）

### 适用于单写入操作的内存顺序
memory_order_release
+ 当前线程中的其它的依赖于本原子量的读取和写入操作不许重排在本语句之前
+ 本线程中的其它写入操作对其它线程中的需要获取同一个原子变量对应的位置的采用memory_order_acquire的读取操作可见，同时对其它线程中的传递依赖于原子量值的读取操作(memory_order_consumed)可见


## 适用于读取并修改操作的内存顺序
memory_order_acq_rel保证操作本身同时满足acquire和release；同一线程中的该语句前后的读取或者写入操作均不允许被重新排序，而其它线程中的写入并释放原子量的操作对应的修改在本线程的修改之前可见，并且本线程中对原子量的修改操作本身在其它线程的读取操作中也可见

## 适用于所有操作的默认值
memory_order_seq_cst是默认的操作顺序，它提供了顺序一致性顺序保证。
+ 读取操作采用该内存顺序时，等价于acquire操作
+ 写入操作采用该内存顺序是，等价于release操作
+ 读取并修改操作采用该内存顺序时，等同于叠加了acquire/release操作的同时，还额外保证所有的线程都能看到对多个原子量修改的一种全局顺序




## 参考
[现代C++的内存模型和高性能的多线程编程](https://skyscribe.github.io/post/2019/11/04/cpp-memory-model-and-order/)
[浅谈 C++ 内存模型](https://www.bluepuni.com/archives/cpp-memory-model/)
[一文读懂C++11内存模型，多线程编程不再困惑](https://zhuanlan.zhihu.com/p/18297976883)
