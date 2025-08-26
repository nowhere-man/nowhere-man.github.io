---
layout: post
title: C++关键字-ooverride和final
slug: new
categories: [C++总结]
tags: [C++关键字]
---

## override
override关键字确保你在派生类中正确地重写了一个虚函数。

override确保：
1. 基类中存在一个同名的虚函数。
1. 派生类中重写的函数签名（函数名、参数列表和常量性）与基类虚函数完全匹配。

如果不满足上面任何一条，编译器就会报错。

```cpp
class Base {
   virtual void foo() const {
     std::cout << "This is from Base." << std::endl;
   }
 };
 ​
 class Derivied : Base {
   virtual void foo() { // 派生类和基类的函数签名不一样
     std::cout << "This is from Derived." << std::endl;
   }
 };
 ​
 void bar(Base& base) {
   base.foo();
 }
 ​
 int main() {
   Derived d;
   bar(d);  //派生类隐式转换为基类，没有表现多态："This is from Base."
 }
```
使用override：
```cpp
 class Derivied : Base {
   virtual void foo() override { // 派生类和基类的函数签名不一样, 编译器报错
     std::cout << "This is from Derived." << std::endl;
   }
 };
```
派生类重写基类的虚函数时，可以加virtual，但不是必须的，它的virtual属性会自动继承到所有派生类中，建议：
+ 只在基类的虚函数上加virtual关键字，其余的派生类重写虚函数不加virtual，但要统一加上override。


## final

final有2个用法：
+ 防止派生类重写基类的虚函数。
+ 防止类被继承。

防止派生类重写基类的虚函数
```cpp
class Base {
   virtual void foo() {}
   virtual void bar() final {}
 };
 ​
 class Derivied : Base {
    void foo() override {} // ok
    void bar() override {} // 编译器报错
 };

```

防止类被继承
```cpp
class Base final {};

// 编译器报错
class Derived : public Base {};
```
