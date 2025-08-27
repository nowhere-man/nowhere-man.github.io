---
layout: post
title: STL容器：forward_list核心总结
slug: stl-forward-list
categories: [C++总结]
tags: [STL]
---

std::forward_list支持从容器中的任何位置快速插入和移除元素的容器。不支持快速随机访问。

std::forward_list实现为单链表，且实质上与其在C中的实现相比无任何开销。

与std::list相比，此容器在不需要双向迭代时提供更好的存储空间效率。


## 参考
+ [面试题：C++vector的动态扩容，为何是1.5倍或者是2倍](https://blog.csdn.net/qq_44918090/article/details/120583540)
+ [c++ vector的扩容机制](https://blog.csdn.net/qq_41021141/article/details/131329403)
