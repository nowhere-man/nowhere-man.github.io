---
layout: post
title: C++关键字-alignas和alignof
slug: alignas-alignof
categories: [C++]
---
## alignof
`alignof(type)`，用于获取类型的对齐字节数；返回size_t类型；alignof只能用于类型。
```cpp
#include <iostream>

using namespace std;

struct DemoStruct2 {
    short s1;                  // 2B
    short s2;                  // 2B
                               // Padding 4B(因为sdb1结构体是8字节对齐的, 所以sdb1.sc1相对于首地址的偏移必须是8的倍数，所以在sdb1.sc1之前填充4B)
    struct SubDemoStruct1 {
        char sc1;              // 1B
                               // Padding 7B
        long long int slli1;   // 8B
    } sdb1;
    struct SubDemoStruct2 {
        char sc1;              // 1B
    } sdb2;
                               // Padding 3B(因为sdb3结构体是4字节对齐的, 所以sdb3.si1相对于首地址的偏移必须是4的倍数，所以在sdb3.si1之前填充3B)
    struct SubDemoStruct3 {
        int si1;               // 4B
    } sdb3;
    char c1;                   // 1B
                               // Padding 7B(因为结构体中最大成员的size是8, 所以结构体的总大小必须是8的倍数，所以在结构体最后填充7B)

};

int main()
{
    cout << "alignof(DemoStruct2): " << alignof(DemoStruct2) << endl;
    cout << "alignof(DemoStruct2::SubDemoStruct1): " << alignof(DemoStruct2::SubDemoStruct1) << endl;
    cout << "alignof(DemoStruct2::SubDemoStruct2): " << alignof(DemoStruct2::SubDemoStruct2) << endl;
    cout << "alignof(DemoStruct2::SubDemoStruct3): " << alignof(DemoStruct2::SubDemoStruct3) << endl;
    return 0;
}
```
输出：
```cpp
alignof(DemoStruct2): 8
alignof(DemoStruct2::SubDemoStruct1): 8
alignof(DemoStruct2::SubDemoStruct2): 1
alignof(DemoStruct2::SubDemoStruct3): 4
```

## alignas
`alignas(number) type`，用于改变目标类型的对齐字节数。

### 1. 修改结构体的字节对齐规则
```cpp
struct Test {
    char a;
    char b;
    test()
    {
        cout << "sizeof(Test) =" << sizeof(Test) << endl;    // 2
        cout << "alignof(Test) =" << alignof(Test) << endl;  // 1
    }
};

struct alignas(4) Test {
    char a;
    char b;
    test()
    {
        cout << "sizeof(Test) =" << sizeof(Test) << endl;    // 4
        cout << "alignof(Test) =" << alignof(Test) << endl;  // 4
    }
};
```
### 2. 修改结构体内部变量的对齐规则
```cpp
struct Test
{
    char a;
    char b;
    alignas(2) char c;
    void test()
    {
        cout << "sizeof(Test) =" << sizeof(Test) << endl;   // 8
        cout << "alignof(Test) =" << alignof(Test) << endl; // 4

        cout << "offset of a: " << offsetof(Test, a) << endl; // 0
        cout << "offset of b: " << offsetof(Test, b) << endl; // 1
        cout << "offset of c: " << offsetof(Test, c) << endl; // 2
    }
};
```
注意：
+ alignas设置的是变量的对齐规则，不影响struct内其他同类型的变量
+ alignas不一定生效，alignas会和默认的对齐规则取最大值
+ 如果struct和struct成员变量都设置了alignas，对齐规则取最大值

### 3. 修改普通变量的对齐规则

```cpp
    int a;
    auto b = reinterpret_cast<intptr_t>(&a) % 16;
    cout << b << endl;
```
int默认以4对齐，则a的地址对16取模，可能是0 4 8 12。
```cpp
alignas(16) int a;
auto b = reinterpret_cast<intptr_t>(&a) % 16;
cout << b << endl;
```
设置a16字节对齐，则a的地址对16取模，必定是0。
