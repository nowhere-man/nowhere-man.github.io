---
layout: post
title: C++关键字-thread_local
slug: thread_local
categories: [C++总结]
tags: [C++ keywords]
---

## 存储说明符
存储说明符（storage class specifier）是名字的声明语法中的声明说明符序列 的一部分。它与名字的作用域一同控制名字的两个独立性质：它的**存储期**和它的**链接**。

|存储说明符  |说明  |备注  |
|---------|---------|---------|
|auto     |自动存储期|c++11起弃用|
|register     |自动存储期|c++17起弃用|
|static     |静态或者线程存储期，同时指示是内部链接|         |
|extern     |静态或线程存储期，同时指示是外部链接|         |
|thread_local     |线程存储期|         |
## thread_local
有且只有`thread_local`关键字修饰的变量具有线程存储。
> 线程存储期：对象的存储在线程开始时分配，而在线程结束时解分配。每一个线程都拥有一个独立的变量实例。唯有声明为thread_local的对象拥有此存储期。 thread_local能与static或extern一同出现，以指示链接属性。

### 全局thread_local变量
每个线程都会在线程初始化时拷贝一个全部变量的副本，互不影响
### 局部thread_local变量
只在每个线程创建时初始化一次。
### 类的thread_local成员变量
thread_local作为类成员变量时必须是static的。

需要注意的是：如果类的**成员函数内**定义了`thread_local`变量，则对于**同一个线程内**的该类的多个对象都会共享一个变量实例，并且只会在第一次执行这个成员函数时初始化这个变量实例，这一点**跟类的static静态成员变量类似**。

```cpp
class Foo {
public:
    void show() {
        thread_local int x = 0;
        x++;
        cout << x << endl;
    };
    thread_local static int x; // 定义类的静态成员变量
};

int Foo::x = 1; // 初始化类的静态成员变量

int main()
{
    Foo f1;
    f1.show(); // 1
    Foo f2;
    f2.show(); // 2
}
```