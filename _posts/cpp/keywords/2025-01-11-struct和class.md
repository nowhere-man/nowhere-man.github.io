---
layout: post
title: C++关键字-struct和class
slug: struct-and-class
categories: [C++总结]
tags: [C++关键字]
---

struct和class有很大的相似性。
但是两者最大的区别就在于思想上，一般来说，struct更适合看成是一个数据结构（纯粹的数据对象，POD结构）的实现体，class更适合看成是一个对象的实现体。
## 1. 默认的访问权限

struct默认的数据访问控制是public的
calss默认的数据访问控制是private的
```cpp
struct Foo {
    int a;
};
int main()
{
    Foo f;
    // correct
    f.a = 10;
}

class Bar {
    int a;
};

int main()
{
    Bar b;
    // error: ‘int Bar::a’ is private within this context
    b.a = 10;
}
```

## 2. 默认的继承访问权限
```cpp
// correct
struct Foo {
    int a;
};
struct Bar: Foo  {
    int b;
};

int main()
{
    Bar bar;
    // correct
    bar.a = 10;
    bar.b = 20;
}
```

```cpp
class Foo {
    int a;
};
class Bar: Foo  {
    int b;
};

int main()
{
    Bar bar;
    // error: ‘int Foo::a’ is private within this context
    bar.a = 10;
    // error: ‘int Bar::b’ is private within this context
    bar.b = 20;
}
```

除此之外，struct可以继承自class，class也同样可以继承自struct，继承访问权限是看子类到底是用的struct还是class
```cpp
struct A
{
    int a;
};

struct B: A   //public继承
{
    int b;
};

class C: A    //private继承
{
    int c;
};
```
## 3. class可用于定义模板参数，但struct不行
在模板定义语法中class与typename的作用完全一样
```cpp
template<class T, class Y>
int Func(const T& t, const Y& y)
{
    //TODO
}

template<typename T, typename Y>
int Func(const T& t, const Y& y)
{
    //TODO
}
```
## 4. class和struct在使用大括号{}初始化上的区别
1. struct在定义时，如果内部**没有构造函数或虚函数**，可以用{}初始化（用initializer list对数据进行按顺序的初始化）
2. class在定义时，如果内部没有构造函数或虚函数，且**所有成员变量全是public**的话，class可以用{}初始化
3. class和struct如果定义了构造函数或虚函数，则都不能用{}进行初始化