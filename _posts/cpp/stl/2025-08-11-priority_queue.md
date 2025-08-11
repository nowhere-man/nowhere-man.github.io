---
layout: post
title: priority_queue核心总结
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
