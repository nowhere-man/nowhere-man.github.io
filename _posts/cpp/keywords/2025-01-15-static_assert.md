---
layout: post
title: C++关键字-static_assert
slug: static-assert
categories: [C++总结]
tags: [C++ keywords]
---

**static_assert 在 c++用来做编译期间的断言**，也叫静态断言。

该关键字是从 c 语言的 assert 中继承过来的，但是**assert 是在运行期间的断言**。

static_assert 的用法：

```c
static_assert(boolean expression, message)    (C++11 起)
static_assert(boolean expression)             (C++17 起)
```

如果布尔常量表达式为真，则 static_assert 无作用；
如果布尔常量表达式为假，则它就会发布一个编译时的错误，错误的提示就是第二个参数，在 c++17 中可以省略该消息。

说明

1. 由于 static_assert 是编译期间断言的，所以**第一个参数必须是常量**，语法中也有说明，而不能是变量。
2. c 语言中 assert 是运行期间断言的，所以在运行过程中反复调用会极大的影响程序的性能，增加额外开销，但是 static_assert 是编译期间断言的，所以不会生成目标代码，不会对最终程序的运行造成任何的性能影响，
3. 如果该常量表达式依赖于某些模板参数，则延迟到模板实例化时再进行演算。

例子：

```cpp
explicit unique_ptr(pointer __p) noexcept : _M_t(__p, deleter_type())
    { static_assert(!is_pointer<deleter_type>::value,
            "constructed with null function pointer deleter"); }

unique_ptr(pointer __p,
    typename remove_reference<deleter_type>::type&& __d) noexcept
    : _M_t(std::move(__p), std::move(__d))
    { static_assert(!std::is_reference<deleter_type>::value,
            "rvalue deleter bound to reference"); }


constexpr unique_ptr() noexcept
    : _M_t()
    { static_assert(!std::is_pointer<deleter_type>::value,
            "constructed with null function pointer deleter"); }

```
