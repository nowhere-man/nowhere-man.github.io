---
layout: post
title: C++关键字-new和delete
slug: new-and-delete
categories: [C++总结]
tags: [C++ keywords]
---

## 1. new操作符
new运算符用于在**堆上动态分配**。

如果分配成功，则返回指向分配内存的指针；**如果分配失败，则抛出std::bad_alloc异常**。
### new内置类型

```cpp
// 动态申请一个int类型的空间
int* a = new int;
// 动态申请一个int类型的空间并初始化为11
int* b = new int(11);

// 动态申请3个int类型的空间
int* c = new int[3];
// 动态申请10个int类型的空间，并进行部分初始化
int* d = new int[10]{ 1,2,3 };
```

### new自定义类型
执行过程：
1. 调用**operator new函数**分配内存。
1. 调用类的对应**构造函数**生成对象。
1. 返回对应指针。

```cpp
Foo* f1 = new Foo;
Foo* f2 = new Foo();

Foo* f3 = new Foo[3];
Foo* f4 = new Foo[3]{Foo(), Foo(), Foo()};
```

### new和malloc的区别
+ new操作符内存分配成功时，返回的是对象类型的指针，类型严格与对象匹配，无须进行类型转换；而malloc内存分配成功则是返回void * ，需要强制类型转换
+ new操作符内存分配失败时，会抛出bac_alloc异常，它不会返回NULL；而malloc分配内存失败时返回NULL。
+ new操作符申请内存分配时无须指定内存块的字节大小，编译器会根据类型信息自行计算；而malloc则需要显式地指出所需内存的字节大小。
+ new会调用对象的构造函数以完成对象的构造；而malloc则不会。

## 2. operator new
operator new是系统提供的**全局函数**，在`new`头文件中定义；

operator new执行的操作**只是分配内存**，事实上全局函数`::operator new`只是调用malloc分配内存，并且返回一个void*指针。

而new运算符执行的操作是：1. **调用operator new 分配内存**；2. **调用构造函数生成对象**。

### throw new
在分配失败的情况下，抛出异常`std::bad_alloc`，而不是返回`NULL`。

```cpp
void* operator new  ( std::size_t ) (std::bad_alloc);
void* operator new[]( std::size_t ) (std::bad_alloc);

void* operator new  ( std::size_t, std::align_val_t ) (std::bad_alloc);
void* operator new[]( std::size_t, std::align_val_t ) (std::bad_alloc);
```

### nothrow new
nothrow new在分配失败的情况下，不抛出异常，而是返回`NULL`。

```cpp
void* operator new  ( std::size_t, const std::nothrow_t& );
void* operator new[]( std::size_t, const std::nothrow_t& );

void* operator new  ( std::size_t, std::align_val_t, const std::nothrow_t& ) noexcept;
void* operator new[]( std::size_t, std::align_val_t, const std::nothrow_t& ) noexcept;
```

> 编译器会自动计算所需分配的内存大小，并传递给 operator new。所以不需要显式地传递size参数。

### operator new的重载
如果类中没有重载`operator new`，那么调用的就是全局的`::operator new`来完成堆的分配。
重载operator new时:
+ 返回类型必须声明为`void*`
+ 第一个参数类型必须为表达要求分配空间的大小，类型为`size_t`
+ 可以带其它参数

```cpp
void* operator new  ( std::size_t count, /* args... */ );
void* operator new[]( std::size_t count, /* args... */ );

void* operator new  ( std::size_t count, std::align_val_t al, /* args... */ );
void* operator new[]( std::size_t count, std::align_val_t al, /* args... */ );
```

如何限制对象只能建立在堆上或者栈上：
+ 只能建立在堆上：设置析构函数为`protected`

```cpp
class A {
protected:
    A(){}
    ~A(){}
public:
    static A* create()
    {
        return new A();
    }
    void destory()
    {
        delete this;
    }
};
```
+ 只能建立在栈上：重载`operator new`并设为`private`

```cpp
class A {
private:
    void* operator new(size_t t){}
    void operator delete(void* ptr){}
public:
    A(){}
    ~A(){}
};
```

## 3. placement new

```cpp
void* operator new  ( std::size_t count, void* ptr );
void* operator new[]( std::size_t count, void* ptr );
```

`placement new`**只做对象构造，而不申请内存**。它可以在预先分配好的内存上构造对象。

`placement new`本质上是对operator new的重载，在new头文件中定义。**它不分配内存**，而是调用对应构造函数在`ptr`所指的内存中构造一个对象，之后返回`ptr`。

placement new不可重载。

```cpp
int main()
{
    char mem[100];
    Foo* f = new (mem)Foo;
}
```