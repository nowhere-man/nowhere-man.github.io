---
layout: post
title: C++关键字-constexpr
slug: constexpr
categories: [C++总结]
tags: [C++关键字]
---

## const
const常量是固定值，在程序执行期间不会改变。
常量就像是常规的变量，只不过常量的值在定义后不能进行修改。
> const 并**未区分**出编译期常量和运行期常量，而只是保证程序运行时不直接被修改。


## constexpr

### constexpr常量表达式
**常量表达式** 是能在编译时求值的表达式，constexpr 对象不仅值不会改变，而且保证能够**在编译期取得结果**。

> const 和 constexpr 变量之间的主要区别在于：const 变量的初始化可以延迟到运行时，而 constexpr 变量必须在编译时进行初始化。所有 constexpr 变量均为常量，因此必须使用常量表达式初始化。

> constexpr 限定在了编译期常量。所以在 constexpr 引入， const的职责被拆分出来一部分，只作可读**(readonly)**语义的保证，而常量**(const)**语义交给了 constexpr 负责。

### constexpr函数
c++11中的 constexpr指定的函数返回值和参数都必须保证是字面值，而且必须有且只有一行代码。所以通常只能通过return 三目运算符+递归来计算返回的字面值。
```cpp
constexpr int factorial_1(int n)
{
    return n > 0 ? n * factorial_1( n - 1 ) : 1;
}
```
c++11允许构造函数为constexpr，使得可以在编译时创建和初始化类的对象。
```cpp
class Point {
public:
    constexpr Point(double xVal, double yVal) : x(xVal), y(yVal) {}
    constexpr double getX() const { return x; }
    constexpr double getY() const { return y; }
private:
    double x, y;
};

constexpr Point origin(0, 0);
```
c++14中则只要保证返回值和参数是字面值就行，函数的限制被放宽，允许更复杂的逻辑，如局部变量、循环、无返回值等。
```cpp
constexpr int factorial_2(int n)
{
    int result = 1;
    for (int i = 1; i <= n; ++i)
        result *= i;
    return result;
}
```
c++17中lambda表达式可以被声明为constexpr。对于一个lambda而言，只要被捕获的变量是字面量类型（lieteral type），那么整个lambda也将表现为字面量类型。
```cpp
constexpr auto square_lambda = [](int n) { return n * n; };
```
### constexpr if语句
C++17 引入了 `constexpr if`，允许在编译时根据条件进行分支。
```cpp
template<typename T>
constexpr auto get_value(T t) {
    if constexpr (std::is_pointer<T>::value) {
        return *t;
    } else {
        return t;
    }
}
```
