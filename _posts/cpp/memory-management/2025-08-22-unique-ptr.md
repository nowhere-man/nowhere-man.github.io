---
layout: post
title: 智能指针之unique_ptr
slug: smart-pointer-unique-ptr
categories: [C++总结]
tags: [C++内存管理]
---
std::unique_ptr**独占**所指向的对象。对其持有的堆内存具有**唯一拥有权**。

当我们**独占资源的所有权**的时候，可以使用std::unique_ptr对资源进行管理。当离开unique_ptr对象的作用域时，会自动释放资源。

```cpp
int main()
{
    {
        std::unique_ptr<int> uptr = std::make_unique<int>(200);
    }
    // 离开uptr的作用域,内存自动释放
}
```

## 实现原理

默认情况下unique_ptr被设计成为一个**零额外开销**的智能指针（**前提是使用默认删除器**，自定义删除器**可能**会引入额外开销），相比`new`和`delete`，使用unique_ptr没有额外开销，不管是时间还是空间上。

```cpp
int main()
{
    int* rp = new int();
    std::unique_ptr<int> up(new int());

    std::cout <<"裸指针大小：" << sizeof(rp) << endl;    // 输出8
    std::cout <<"unique_ptr大小："<< sizeof(up) << endl;// 输出8
}
```

## 初始化

```cpp
std::unique_ptr<int> up1 = std::make_unique<int>(123); // 推荐使用

std::unique_ptr<int> up2(new int(123));                // 使用裸指针

std::unique_ptr<int> up3;
up3.reset(new int(123));                               // 默认构造，再使用reset()
```


### 禁止拷贝语义

为了达到唯一拥有权效果，std::unique_ptr类的拷贝构造函数和赋值运算符(operator =)被标记为`delete`。

```cpp
unique_ptr(const unique_ptr&) = delete;
unique_ptr& operator=(const unique_ptr&) = delete;
```

### 支持移动语义
可以通过调用 release 或 reset 将指针所有权从一个（非 const）unique_ptr 转移给另一个 unique_ptr

```cpp

int main()
{
    std::unique_ptr<string> p1 = std::make_unique<string>("hello");

    // 将所有权从p1转移给p2
    std::unique_ptr<string> p2(p1.release()); // release将p1置为空
    assert(p1 == nullptr);

    std::unique_ptr<string> p3;
    // 将所有权从p3转移到p2
    p3.reset(p2.release());  // reset释放了p2原来指向的内存
    assert(p2 == nullptr);
}
```

也可以调用 std::move 移动语义转移 unique_ptr:
```cpp
int main()
{
    std::unique_ptr<int> up1(new int());
    std::cout << "up1 = " << up1.get() << std::endl;

    // 移动构造
    auto up2 = std::unique_ptr<int>(new int());
    std::cout << "up2 = " << up2.get() << std::endl;

    // 移动赋值
    up2 = std::move(up1); // 先delete up2, 再将up1移动到up2，再将up1置为nullptr。
    std::cout << "up1 = " << up1.get() << std::endl;
    std::cout << "up2 = " << up2.get() << std::endl;
}
```

output:

```cpp
up1 = 0x624ed75b82b0
up2 = 0x624ed75b86e0
up1 = 0
up2 = 0x624ed75b82b0
```

### 指向数组

```cpp
{
    std::unique_ptr<int[]> uptr = std::make_unique<int[]>(10);
    for (int i = 0; i < 10; i++) {
        uptr[i] = i * i;
    }
    for (int i = 0; i < 10; i++) {
        std::cout << uptr[i] << std::endl;
    }
}
```


### 支持移动语义

可以通过调用 release 或 reset 将指针所有权从一个（非 const）unique_ptr 转移给另一个 unique_ptr

```cpp
#include <iostream>
#include <memory>
using namespace std;
int main()
{
    unique_ptr<string> p1 = std::make_unique<string>("hello");

    // 将所有权从p1转移给p2
    unique_ptr<string> p2(p1.release());  // release将p1置为空
    assert(p1 == nullptr);

    unique_ptr<string> p3;
    // 将所有权从p3转移到p2
    p3.reset(p2.release());  // reset释放了p2原来指向的内存
    assert(p2 == nullptr);
}
```

也可以调用 std::move 移动语义转移 unique_ptr:

```cpp
{
    std::unique_ptr<int> uptr = std::make_unique<int>(200);
    std::unique_ptr<int> uptr2 = std::move(uptr);
    assert(uptr == nullptr);
}
```

## 自定义deleter

unique_ptr 默认使用`delete`释放它指向的对象，构造函数允许我们可以重载删除器。

```cpp
template< class T, class Deleter = std::default_delete<T> > class unique_ptr;

// 指向类数组时，必须显式定义Deleter
template< class T, class Deleter > class unique_ptr<T[], Deleter>;
```

一般以下情况需要自定义删除器：

+   **非堆内存资源**：例如使用  `fopen` 打开的文件句柄，需要用 `fclose` 关闭；
+   **内存池管理**：如果从一个内存池中分配对象，那么释放时需要将内存归还给内存池，而不是简单地 `delete`。
+   **数组**：`unique_ptr<T[]>` 有一个专门的删除器，它会调用 `delete[]`。但如果你使用 `unique_ptr<T>` 来管理数组，那么就需要自定义一个调用 `delete[]` 的删除器。

上面我们说过使用unique_ptr没有额外开销，但是当我们使用了自定义的deleter后，可能会引入额外的内存开销。

+   **函数对象**：unique_ptr的额外size等于函数对象的成员变量大小

```cpp
struct MyDeleter1 {
    void operator()(int* p) {delete p;}
};

int main() {
    auto b = std::unique_ptr<int, MyDeleter1>(new int());
    std::cout <<"unique_ptr大小："<< sizeof(up) << endl;   // 输出8
}
/****************************************************************/
struct MyDeleter2 {
    int a = 0;
    void operator()(int* p) {delete p;}
};

int main() {
    auto b = std::unique_ptr<int, MyDeleter2>(new int());
    std::cout <<"unique_ptr大小："<< sizeof(up) << endl;   // 输出16
}
```

+   **lambda表达式**：unique_ptr的额外size等于lambda捕获的值的大小

```cpp
int main()
{

    auto deleter1 = [] (int* p) {delete p;};
    auto a = std::unique_ptr<int, decltype(deleter1)> (new int(), deleter1);
    std::cout << "unique_ptr大小：" << sizeof(a) << std::endl; // 输出8

    int x = 10;
    auto deleter2 = [x](int *p) { delete p; };
    auto b = std::unique_ptr<int, decltype(deleter2)>(new int(), deleter2);
    std::cout << "unique_ptr大小：" << sizeof(b) << std::endl; // 输出16
}
```



+   **函数指针**：unique_ptr的额外size增加一个指针长度

```cpp
void deleter(int *p) { delete p; }

int main() {
    auto a = std::unique_ptr<int, decltype(&deleter)>(new int(), &deleter);
    std::cout << "unique_ptr大小：" << sizeof(a) << std::endl; // 输出8
}
```

## 成员函数

| 函数API      | 含义                                           |
| ------------ | ---------------------------------------------- |
| up.get()     | 获取原生指针（不能delete），所有权仍归uptr     |
| up.release() | 释放所有权、并返回原生指针（要自行负责delete） |
| up.reset()   | 和up = nullptr等价，delete堆对象，同时置空指针 |
| up.reset(p)  | 先delete up，再将所有权指向p指针               |
| up.swap(up2) | 交换两个智能指针                               |