---
layout: post
title: 智能指针之shared_ptr
slug: smart-pointer-shared-ptr
categories: [C++总结]
tags: [C++内存管理]
---
std::shared_ptr能够**共享对同一资源的所有权**。它允许多个指针指向同一个资源，并通过引用计数来管理资源的生命周期。

当最后一个std::shared_ptr实例被销毁或重置时，它所指向的资源才会被释放。

```cpp
{
    std::shared_ptr<int> sptr = std::make_shared<int>(200);
    // 初始引用计数为1
    assert(sptr.use_count() == 1);

    {
        std::shared_ptr<int> sptr1 = sptr;
        assert(sptr.get() == sptr1.get());
        // sptr和sptr1共享资源，引用计数为2
        assert(sptr.use_count() == 2);
    }
    // sptr1 已经释放
    assert(sptr.use_count() == 1);
}
// sptr离开作用域，use_count为0，自动释放内存。
```

## 实现原理
std::shared_ptr的核心是其**引用计数**机制。一个std::shared_ptr实例实际上包含两个部分：
+ 一个指向被管理对象的原始指针：这是指向实际存储数据的内存地址。
+ 一个指向控制块的指针：这个控制块在堆上分配，包含：
    + 强引用计数：记录有多少个std::shared_ptr实例正在共享该对象（原子操作，线程安全）。
    + 弱引用计数：记录有多少个std::weak_ptr实例正在监视该对象（原子操作，线程安全）。
    + 自定义删除器（deleter）：如果提供，用于在引用计数归零时释放资源。
    + 自定义分配器（allocator）：如果提供，用于分配内存。

```cpp
int main()
{
    std::cout << sizeof(int*) << std::endl;                  // 输出 8
    std::cout << sizeof(std::shared_ptr<int>) << std::endl;  // 输出 16
}
```

## 初始化

```cpp
std::shared_ptr<int> p1;                // 不传入任何实参
std::shared_ptr<int> p2(nullptr);       // 传入空指针nullptr
std::shared_ptr<int> p3(new int(10));   // 传入裸指针

std::shared_ptr<int> p4 = std::make_shared<int>(10);  // 推荐的初始化方式

// 支持拷贝构造
std::shared_ptr<int> p5(p4);
std::shared_ptr<int> p5 = p4;

// 支持移动构造
std::shared_ptr<int> p6(std::move(p5));
std::shared_ptr<int> p6 = std::move(p5);
```

推荐使用`std::make_shared`，而非构造函数：
+ 当使用std::shared_ptr<T> p(new T())时，会进行两次独立的堆内存分配：
    + 第一次分配：为T类型的对象本身分配内存。
    + 第二次分配：为std::shared_ptr的控制块分配内存。
+ 而std::make_shared<T>()只会进行一次堆内存分配。它会一次性分配一块足够大的内存，同时容纳对象本身和控制块。
    + 速度更快：减少了一次系统调用，提高了性能。
    + 缓存友好：对象和控制块位于相邻的内存区域，这有助于CPU缓存的局部性，使得访问速度更快。

并且使用`std::make_shared`，可以避免双重释放的问题：

```cpp
int* p = new int();
// sp和sp2指向同一块内存，导致双重释放
std::shared_ptr<int> sp(p);
std::shared_ptr<int> sp2(p);

// make_shared避免双重释放的可能性
auto sp = std::make_shared<int>();
auto sp2 = sp;
```
## 成员函数

| 成员函数                 | 说明                                                         |
| ------------------------ | ------------------------------------------------------------ |
| p.get()                  | 返回p中保存的裸指针。                                        |
| p.use_count()            | 返回与p共享对象的智能指针数量                                |
| p.unique()               | 若p.use_count()为1，返回true；否则返回false                  |
| p.reset()                | p的引用计数减一。如果减一后变为零，它将销毁p所管理的对象。同时将p置为`nullptr` |
| p.reset(T* q)            | p的引用计数减一。如果减一后变为零，它将销毁p所管理的对象。p指向的新对象q，将其引用计数初始化为 1 |
| p.reset(T* p, deleter d) | 同p.reset(T* q)，只是当p的引用计数归零时，shared_ptr会调用这个自定义删除器，而不是默认的 `delete`。 |
| p.swap(q)或swap(p,q)     | 交换两个智能指针                                             |

## enable_shared_from_this
一个类的成员函数如何获得指向自身（this）的shared_ptr：


```cpp
class Foo {
public:
    std::shared_ptr<Foo> GetSPtr() {
        return std::shared_ptr<Foo>(this);
    }
};

{
    auto sptr1 = std::make_shared<Foo>(); // 创建了一个新的控制块
    assert(sptr1.use_count() == 1);

    auto sptr2 = sptr1->GetSPtr();        // 创建了另外一个新的控制块

    assert(sptr1.use_count() == 1);
    assert(sptr2.use_count() == 1);
}
```

std::shared_ptr的核心是其**控制块**和引用计数。当第一次通过std::make_shared或std::shared_ptr(T*)创建智能指针时，会生成一个控制块。

如果在GetSPtr()成员函数中再次使用std::shared_ptr(this)，会为同一个原始指针this再创建一个新的、独立的控制块。

此时，sptr1和sptr2拥有独立的引用计数。当它们各自离开作用域时，都会尝试销毁同一个对象，从而**导致双重释放**。

***

enable_shared_from_this允许一个类**安全地**获得一个**指向自身**的std::shared_ptr。

当一个类T继承自std::enable_shared_from_this<T>后，它的成员函数就可以通过调用shared_from_this()方法来生成一个共享所有权的std::shared_ptr，该指针指向当前对象。
```cpp
class Bar : public std::enable_shared_from_this<Bar> {
public:
    std::shared_ptr<Bar> GetSPtr() {
        return shared_from_this();
    }
};

{
    auto sptr1 = std::make_shared<Bar>();
    assert(sptr1.use_count() == 1);

    auto sptr2 = sptr1->GetSPtr();
    assert(sptr1.use_count() == 2);
    assert(sptr2.use_count() == 2);
}
```

继承了std::enable_shared_from_this的子类，**成员变量中增加了一个指向this的weak_ptr**。这个weak_ptr在第一次创建shared_ptr的时候会被初始化，指this。

**继承了std::enable_shared_from_this的类都被强制必须通过shared_ptr进行管理。**

因为只有当调用shared_ptr的构造函数或std::make_shared函数时，会：
1. 创建控制块（包含强引用计数和弱引用计数）。
2. 创建对象本身。
3. 如果发现该对象继承自std::enable_shared_from_this，它会执行一个内部的**连接操作**。将控制块中指向该对象的weak_ptr的所有权信息传递给enable_shared_from_this内部的weak_ptr。

所以如果创建了一个普通的栈对象或裸指针，它并没有关联任何shared_ptr，因此也没有控制块。当你在这种情况下调用shared_from_this()时，内部的weak_ptr是一个无效值，这会导致程序崩溃或产生其他无法预测的未定义行为。

## 缺陷

### 性能开销
shared_ptr相比于裸指针或者unique_ptr来说，会带来一些性能上的开销：
1. **引用计数开销**：每次创建、复制或销毁shared_ptr时，都需要更新引用计数，这涉及到原子操作，影响性能。
2. **控制块开销**：std::shared_ptr内部使用控制块来管理引用计数和对象生命周期，这会带来额外的内存分配和管理开销。

### 循环引用
循环引用是指两个或多个对象相互持有对方的shared_ptr，从而导致引用计数永远不为零，造成内存泄漏。
```cpp
class B;

class A {
public:
    std::shared_ptr<B> b;
};

class B {
public:
    std::shared_ptr<A> a;
};

int main() {
    auto spa = std::make_shared<A>(); // spa引用计数为1
    auto spb = std::make_shared<B>(); // spb引用计数为1
    spa->b = spb; // spb引用计数为2
    spb->a = spa; // spa引用计数为2
}
```
当离开main()函数作用域时，spa和spb被销毁，引用计数各减一，没有降为零，所以spa和spb指向的内存都不会被销毁，造成内存泄漏。

### 线程安全问题
尽管shared_ptr的引用计数操作是线程安全的，但是shared_ptr所指向的对象的读写操作本身并不是线程安全的。

## 注意事项
- 不要使用裸指针初始化多个shared_ptr。

```cpp
int* p1 = new int;
// 由于p1和p2是两个不同对象，但是管理的是同一个指针，这样会造成空悬指针，
std::shared_ptr<int> p2(p1);
std::shared_ptr<int> p3(p1);
```

- 不要用栈中的指针构造shared_ptr对象。

```cpp
int x = 12;
std::shared_ptr<int> ptr(&x);
```

- 不要使用shared_ptr的get()初始化另一个shared_ptr

```cpp
Base *a = new Base();
std::shared_ptr<Base> p1(a);
//p1、p2各自保留了对一段内存的引用计数，其中有一个引用计数耗尽，资源也就释放了,会出现同一块内存重复释放的问题
std::shared_ptr<Base> p2(p1.get());
```