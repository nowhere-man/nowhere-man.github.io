---
layout: post
title: STL容器：unordered_map核心总结
slug: stl-unordered-map
categories: [C++总结]
tags: [STL]
---

`std::unordered_map`是一种关联容器，可以存储一组键值对。

```cpp
template<
	class Key, class T,
    class Hash = std::hash<Key>,
    class Pred = std::equal_to<Key>,
    class Alloc = std::allocator<std::pair<const Key, T> >
> class unordered_map
```

其中：

+   `Key`：键类型
+   `T`：值类型
+   `Hash`：哈希函数，用于将键映射到一个无符号整数。
+   `Pred`：用与比较两个键是否相等。自定义的键类型中需要重载`operator==`操作符。

## 构造函数

```cpp
unordered_map<int, int> map1;                          // 默认构造
unordered_map<int, int> map2(map1);                    // 拷贝构造
unordered_map<int, int> map3(std::move(map2));         // 移动构造
unordered_map<int, int> map4{{1, 1}, {2, 2}};          // 初始化列表构造

// 设置初始的桶数
unordered_map<int, int> map5(n);
// 设置初始的桶数和哈希函数
unordered_map<int, int> map6(n, std::hash<int>()); 
// 设置初始的桶数和哈希函数和键相等比较函数
unordered_map<int, int> map7(4, std::hash<int>(), std::equal_to<int>()); 
```

## 成员函数

### **访问**

| 成员函数                           | 函数说明                                                     |
| ---------------------------------- | ------------------------------------------------------------ |
| `at(key)`          | **带有边界检查**，返回对键为 `key` 的元素的引用。如果键不存在，则抛出 `std::out_of_range` 异常。 |
| `operator[](key)`  | 如果键 `key` 不存在，则插入一个新元素并返回对它的引用；如果键存在，返回对现有元素的引用。 |
| `find(key)`        | 查找键为 `key` 的元素。如果找到，返回指向该元素的迭代器；否则返回 `end()`。 |
| `count(key)`       | 返回键为 `key` 的元素数量。由于 `unordered_map` 的键唯一，该函数返回值只能是 `0` 或 `1`。 |
| `equal_range(key)` | 返回一个**迭代器对**，表示所有键等于 `key` 的元素范围。由于键唯一，该范围要么为空，要么包含一个元素。 |

### **修改**

| 成员函数                          | 说明                                                      |
| --------------------------------- | --------------------------------------------------------- |
| `insert(const value_type& value)` | 插入一个键值对 `value`。如果键已存在，则插入失败。        |
| `emplace(Args&&... args)`         | 通过就地构造一个新元素，并将其插入到 `unordered_map` 中。 |
| `try_emplace(key, Args&&... args)` | 尝试就地构造，如果`key`不存在，则并将其插入，否则不做任何事情 |
| `erase(const_iterator position)`  | 移除 `position` 处的元素。                                |
| `erase(key)`      | 移除键为 `key` 的所有元素，返回被移除的元素数量。         |
| `clear()`                         | 移除所有元素，使 `unordered_map` 变为空。                 |
| `swap(unordered_map& other)`      | 与另一个 `unordered_map` 交换内容。                       |

### **容量**

| 成员函数     | 说明                                             |
| ------------ | ------------------------------------------------ |
| `empty()`    | 如果 `unordered_map` 中没有元素，则返回 `true`。 |
| `size()`     | 返回 `unordered_map` 中元素的数量。              |
| `max_size()` | 返回 `unordered_map` 可以容纳的最大元素数量。    |

### **桶**

| 成员函数                      | 说明                                              |
| ----------------------------- | ------------------------------------------------- |
| `bucket_count()`              | 返回当前桶的数量。                                |
| `bucket_size(size_type n)`    | 返回第 `n` 个桶中的元素数量。                     |
| `bucket(key)` | 返回键为 `key` 的元素所在的桶的索引。             |


### **哈希**

| 成员函数                   | 说明                                              |
| -------------------------- | ------------------------------------------------- |
| `load_factor()`            | 返回当前的负载因子（`size() / bucket_count()`）。 |
| `max_load_factor(float z)` | 设置或获取最大负载因子。                          |
| `rehash(size_type n)`      | 重新哈希容器，使得其桶数量至少为 `n`。            |
| `hash_function()`          | 返回哈希函数对象。                                |
| `key_eq()`                 | 返回键相等比较函数。                              |

### **迭代器**

| 成员函数   | 说明                                            |
| ---------- | ----------------------------------------------- |
| `begin()`  | 返回指向第一个元素的迭代器。元素的顺序不确定。  |
| `end()`    | 返回指向最后一个元素之后位置的迭代器。          |
| `cbegin()` | 返回指向第一个元素的 `const` 迭代器。           |
| `cend()`   | 返回指向最后一个元素之后位置的 `const` 迭代器。 |

## 实现分析

`unordered_map`的底层实现是哈希表。



## 性能分析

