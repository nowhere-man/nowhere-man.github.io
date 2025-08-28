---
layout: post
title: STL容器适配器：priority_queue核心总结
slug: std-priority-queue
categories: [C++总结]
tags: [STL]
---

优先队列`std::priority_queue` 是 一个**容器适配器**。它是一个特殊的队列，具有队列的所有特性。

`std::priority_queue`的特殊之处就是它**自动保持队列中的元素有序**，确保**队首元素始终是最高优先级**的元素。

>   **容器适配器**是一种特殊的类模板，它不直接管理元素，而是**封装**一个已有的底层容器，并提供不同的**接口**来操作这个底层容器。它通过**限制和重塑**底层容器的行为，表现出特定**抽象数据类型**的性质。

## 实现分析

 默认情况下，`std::priority_queue`的实现是**大根堆**，并使用 `std::vector` 作为其底层容器。

时间复杂度分析：

+   **插入**: 时间复杂度为 **O(logN)**。
+   **移除队首元素**: 时间复杂度为 **O(logN)**。
+   **访问队首元素**: 时间复杂度为 **O(1)**。

>   **大根堆**是一种特殊的二叉树，它满足以下性质：
>
>   1.   **堆序性：**树中任意一个父节点的值都**大于或等于**其子节点的值。（树中最大的元素总是在**根节点**上）
>   1.   **完全二叉树**：树的每一层（除了最后一层）都被完全填满，并且最后一层的所有节点都靠左排列。

## 构造函数

`std::priority_queue`的模板格式：`std::priority_queue<T,Container,Compare>::priority_queue`

+   `T`是元素类型
+   `Container`是底层容器，默认为`std::vector<>`，`Container`必须是用**数组实现**的容器。
+   `Compare`是比较函数，默认为`std::less<>`

```cpp
// 默认构造
std::priority_queue<int> pq1;
// 等价于
std::priority_queue<int, std::vector<int>, std::less<int>> pq2;

// 使用迭代器构造
std::vector<int> v{30, 10, 50, 20, 40};
std::priority_queue<int> pq3(v.begin(), v.end());

// 使用自定义底层容器
std::priority_queue<int, std::deque<int>> pq4;

// 使用自定义比较函数
// 使用自定义比较函数，必须也要显式指定底层容器
std::priority_queue<int, std::vector<int>, std::greater<int>> pq5;
```
## 默认Compare函数

为什么`std::less`产生大根堆？

1.   `std::less<T>` 是一个仿函数，它重载了 `operator()`，用于实现**小于**比较。当 `a < b` 成立时，`std::less<T>()(a, b)` 返回 `true`。

2.   为了维持堆的稳定，**每个父节点都必须大于或等于其子节点**。

     +   `std::priority_queue` **期望 `Compare(parent, child)` 返回 `false`**，这样父子关系才正确。

     +   当 `parent < child` 时，`std::less` 返回 `true`，这个父子关系被认为是**错误的**，需要把 `child` 移动到 `parent` 的位置。

     +   当 `parent >= child` 时，`std::less` 返回 `false`，这个父子关系被认为是**正确的**，不需要调整。

### 自定义Compare函数

默认的比较函数为`std::less`函数。

#### 重载比较操作符

```cpp
struct Node {
    int value;
};

bool operator<(const Node &parent, const Node &child)
{
    // 大根堆
    return parent.value < child.value;
}
std::priority_queue<Node> pq;
```

#### 重载函数调用符

```cpp
struct Node {
    int value;
};

struct compare {
    bool operator()(const Node &parent, const Node &child)
    {
        // 小根堆
        return parent.value > child.value;
        // 大根堆
        return parent.value < child.value;
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
