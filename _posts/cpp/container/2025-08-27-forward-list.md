---
layout: post
title: STL容器：forward_list核心总结
slug: stl-forward-list
categories: [C++总结]
tags: [STL]
---

std::forward_list支持从容器中的任何位置快速插入和移除元素的容器。不支持快速随机访问。

std::forward_list实现为单链表，且实质上与其在C中的实现相比无任何开销。

与std::list相比，std::forward_list在不需要双向迭代时提供更好的存储空间效率。

forward_list的每个节点只包含数据域以及后继指针。

## forward_list的size
forward_list只有一个头结点成员变量。
![](https://i-blog.csdnimg.cn/blog_migrate/f3f3ebb9ff6e27f5c1a0d986051b693a.png)

```cpp
std::forward_list<int> f;

// 8
std::cout << sizeof(f) << std::endl;
```

## forward_list的构造函数
```cpp
forward_list<int> f1;                                  // 无元素
forward_list<int> f2 {1, 2, 3, 4, 5};                  // 列表初始化
forward_list<int> f3(4);                               // 4个元素，初始值为0
forward_list<int> f4(5, 3);                            // 5个元素，初始值为3
forward_list<int> f5(f4);                              // 使用另外一个forward_list拷贝构造
forward_list<int> v6(std::move(f5));                   // 使用另外一个forward_list移动构造
forward_list<int> v7(v6.begin(), v6.end());            // 使用迭代器初始化[左闭,右开）
```

## forward_list的成员函数

**访问**：
+ front()

**修改**:
+ push_front()：在容器头部插入一个元素。
+ emplace_front()
+ pop_front()：删除容器头部的一个元素。
+ erase_after()：删除容器中某个指定位置或区域内的所有元素。
+ insert_after()：在指定位置之后插入一个新元素，并返回一个指向新元素的迭代器。
+ emplace_after()
+ clear()：清除所有元素，clear()完成后size()为0。
+ resize()：调整容器的大小。

**容量**：
+ empty()
+ max_size():容器的理论极限size，和系统或库实现相关。


**操作**
+ merge()：合并两个事先已排好序的forward_list容器，并且合并之后的 forward_list 容器依然是有序的。
+ splice_after()：将某forward_list容器中指定位置或区域内的元素插入到另一个容器的指定位置之后。
+ remove()：删除容器中所有等于val的元素。
+ remove_if()：删除容器中满足条件的元素。
+ reverse()
+ unique()：删除容器中相邻的重复元素，只保留一个。
+ sort()


**迭代器**：
+ begin() 和 end()
+ cbegin() 和 cend()
+ **before_begin()**：返回一个前向迭代器，其指向容器中第一个元素之前的位置。
+ cbefore_begin()

## 计算forward_list成员个数
```cpp
forward_list<int> f{ 1,2,3,4 };
int count = std::distance(std::begin(f), std::end(f));
```

## 遍历forward_list
```cpp
forward_list<int> f{ 1,2,3,4 };

for(auto it = f.begin(); it != f.end(); ++it) {
    cout << *it << " ";
}
```
forward_list<int> f{ 1,2,3,4 };
auto cur = f.begin();
auto prev = f.before_begin();

while (cur != f.end()) {
    if (*cur % 2 == 0) {
        it = f.erase_after(prev);
    } else {
        prev = cur;
        cur++;
    }
}


## 遍历forward_list的同时删除节点

## forward_list的迭代器失效规则
+ 插入：不会导致任何迭代器失效。
+ 删除：仅指向被删除节点的迭代器失效，其余迭代器仍有效。
