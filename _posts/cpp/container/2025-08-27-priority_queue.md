---
layout: post
title: STL容器适配器：priority_queue核心总结
slug: std-priority-queue
categories: [C++总结]
tags: [STL]
---

## 自定义priority_queue的比较函数
默认的比较函数为`less`函数。

### 重载操作符

```cpp
#include<queue>
#include <vector>

struct Node {
    int value;
};

bool operator<(const Node &a, const Node &b)
{
    // 按照value从大到小排列,注意这里是小于符号
    return a.value < b.value;
}
std::priority_queue<Node> pq;
```

### 自定义compare函数

```cpp
#include<queue>
#include <vector>

struct Node {
    int value;
};

struct compare{
    bool operator()(const Node &a, const Node &b)
    {
        // 按照value从小到大排列,注意这里是大于符号
        return a.value > b.value;
    }
};

std::priority_queue<Node, std::vector<Node>, compare> pq;
```

## 成员函数

### **访问**

| 成员函数 | 函数说明                                               |
| -------- | ------------------------------------------------------ |
| `top()`  | 返回对队列中**优先级最高**（通常是最大值）元素的引用。 |

### **修改**

| 成员函数                      | 说明                                                        |
| ----------------------------- | ----------------------------------------------------------- |
| `push(const T& value)`        | 将一个新元素 `value` 添加到优先队列中，并保持其优先级顺序。 |
| `emplace(Args&&... args)`     | 通过就地构造一个新元素，并将其添加到优先队列中。            |
| `pop()`                       | 移除队列中优先级最高的元素。                                |
| `swap(priority_queue& other)` | 与另一个 `priority_queue` 交换内容。                        |

### **容量**

| 成员函数  | 说明                                    |
| --------- | --------------------------------------- |
| `empty()` | 如果优先队列中没有元素，则返回 `true`。 |
| `size()`  | 返回优先队列中元素的数量。              |
