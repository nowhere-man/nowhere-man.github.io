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

### **访问**

| 成员函数  | 函数说明               |
| --------- | ---------------------- |
| `front()` | 返回对第一个元素的引用 |

### **修改**

| 成员函数                                                 | 说明                                                         |
| -------------------------------------------------------- | ------------------------------------------------------------ |
| `assign(size_type count, const T& value)`                | 将 `forward_list` 的内容替换为 `count` 个值为 `value` 的元素。 |
| `push_front(const T& value)`                             | 在 `forward_list` 的开头添加一个新元素 `value`。             |
| `pop_front()`                                            | 移除 `forward_list` 的第一个元素。                           |
| `insert_after(const_iterator position, const T& value)`  | 在 `position` 后插入一个新元素。                             |
| `erase_after(const_iterator position)`                   | 移除 `position` 后的元素。                                   |
| `emplace_front(Args&&... args)`                          | 在 `forward_list` 开头通过就地构造插入新元素。               |
| `emplace_after(const_iterator position, Args&&... args)` | 在 `position` 后通过就地构造插入新元素。                     |
| `clear()`                                                | 移除所有元素，使 `forward_list` 变为空。                     |
| `swap(forward_list& other)`                              | 与另一个 `forward_list` 交换内容。                           |
| `resize(size_type count)`                                | 改变 `forward_list` 的大小为 `count`。                       |

### **容量**

| 成员函数     | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| `empty()`    | 如果 `forward_list` 中没有元素，则返回 `true`。              |
| `max_size()` | 返回 `forward_list` 可以容纳的最大元素数量。和系统或库实现相关。 |

### 操作

| 成员函数  | 说明                                            |
| --------- | ----------------------------------------------- |
| `unique()`                                                   | 移除相邻的重复元素。                                         |
| `merge(forward_list& other)`                                 | 将另一个有序的 `forward_list` 合并到当前列表中。             |
| `splice_after(const_iterator position, forward_list& other)` | 在 `position` 后将另一个 `forward_list` 的所有元素移动到当前列表中。 |
| `remove(const T& value)`                                     | 移除所有等于 `value` 的元素。                                |
| `reverse()`                                                  | 反转列表中的元素顺序。                                       |
| `sort()`                                                     | 对列表中的元素进行排序。                                     |

### **迭代器**

| 成员函数          | 说明                                                         |
| ----------------- | ------------------------------------------------------------ |
| `begin()`         | 返回指向 `forward_list` 第一个元素的迭代器。                 |
| `end()`           | 返回指向 `forward_list` 最后一个元素之后位置的迭代器。       |
| `cbegin()`        | 返回指向 `forward_list` 第一个元素的 `const` 迭代器。        |
| `cend()`          | 返回指向 `forward_list` 最后一个元素之后位置的 `const` 迭代器。 |
| `before_begin()`  | 返回一个指向 `forward_list` 第一个元素**之前**的迭代器。     |
| `cbefore_begin()` | 返回一个指向 `forward_list` 第一个元素**之前**的 `const` 迭代器。 |

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
