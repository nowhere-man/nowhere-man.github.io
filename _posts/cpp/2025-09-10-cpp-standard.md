---
layout: post
title: C++14/17/20新特性总结
slug: cpp-standard
categories: [C++总结]
tags: [C++]
---

## C++14
1. std::make_unique
    ```cpp
    // 旧方式 (不安全)
    // std::unique_ptr<Foo> p(new Foo());

    // C++14 方式 (安全、简洁)
    auto p = std::make_unique<Foo>();
    ```
1. auto
    函数返回值类型推导
    ```cpp
    auto add(int a, int b) {
        return a + b;
    }
    ```
    auto支持Lambda表达式
    ```cpp
    auto generic_add = [](auto x, auto y) {
        return x + y;
    };
    ```
1. 二进制字面量
    ```cpp
    int mask = 0b11010010;
    ```
1. 数字分隔符
    ```cpp
    long long large_number = 1'000'000'000;
    int binary_val = 0b1101'0010'1001'0110;
    ```
1. deprecated属性
    ```cpp
    [[deprecated("Use new_function() instead")]]
    void old_function() {
        // ...
    }
    ```
1. std::shared_lock
    通过std::shared_timed_mutex和std::shared_lock来实现读写锁，保证多个线程可以同时读

## C++17
1. std::optional, std::variant, std::any
    + std::optional<T>: 表示一个可能包含也可能不包含 T 类型值的对象。解决了使用特殊值（如 nullptr 或 -1）来表示“无值”状态的问题。
    + std::variant<T, U, ...>: 一个类型安全的联合体（union）。在任何时候，它都只持有其模板参数类型之一的值。
    + std::any: 可以持有任何可拷贝类型的值。
1. string_view:一个轻量级的字符串引用。它指向一个已存在的字符序列（如 std::string 或 C 风格字符串），但本身不分配内存。
    ```cpp
    void print_string(std::string_view sv) {
        std::cout << sv << std::endl;
    }

    std::string s = "A long long string";
    print_string(s); // 无拷贝
    print_string("A C-style literal"); // 无拷贝
    ```
1. 结构化绑定
    ```cpp
    for (const auto& [key, value] : my_map) {
        std::cout << key << ": " << value << std::endl;
    }

    auto [id, score, name] = get_tuple_data();
    ```
1. if语句中初始化
    ```cpp
    if (auto it = my_map.find(1); it != my_map.end()) {
        std::cout << "Found: " << it->second << std::endl;
    } // it在这里销毁
    ```
1. 类模板参数自动推导
    ```cpp
    std::lock_guard<std::mutx> lock(mtx);  // C++17之前
    std::lock_guard lock(mtx);             // C++17自动推导
    ```
1. constexpr
    constexpr Lambda表达式
    ```cpp
    constexpr auto square = [](int n) {
        return n * n;
    };

    static_assert(square(5) == 25);
    ```
    constexpr if表达式
    ```cpp
    template <typename T>
    auto get_value(T t) {
        if constexpr (std::is_pointer_v<T>) {
            return *t;
        } else {
            return t;
        }
    }
    ```
1. filesystem: 提供了一套标准的、跨平台的API来操作文件和目录。
1. 并行算法
    ```cpp
    std::vector<int> v = { /* ... 大量数据 ... */ };
    // 使用并行策略
    std::sort(std::execution::par, v.begin(), v.end());
    ```
1. namespace嵌套
    ```cpp
    namespace A {
        namespace B {
            namespace C {
                void func();
            }
        }
    }
    ```
1. __has_include预处理表达式
1. 新增attribute
    + [[nodiscard]] ：表示修饰的内容不能被忽略，可用于修饰函数，标明返回值一定要被处理
    + [[maybe_unused]] ：提示编译器修饰的内容可能暂时没有使用，避免产生警告
## C++20
1. 协程
1. 模块
1. 并发
    + jthread
    + 信号量
    + barrier
    + latch
    + std::atomic的等待和通知
    + std::atomic_ref
1. 概念库
    concpet是对模板参数的约束，它允许你指定模板参数必须满足哪些要求
    ```cpp
    // 定义一个概念，要求类型T可被相加并返回T类型
    template<typename T>
    concept Addable = requires(T a, T b) {
        { a + b } -> std::same_as<T>;
    };

    // 使用概念约束模板参数
    template<Addable T>
    T sum(T a, T b) {
        return a + b;
    }

    int main() {
        sum(1, 2);      // OK
        sum("a", "b");  // 编译错误，错误信息清晰指出 const char* 不满足 Addable
    }
    ```
1. 范围库
    ```cpp
    int main() {
        std::vector<int> numbers = {1, 2, 3, 4, 5, 6, 7, 8};

        // 找出所有偶数，将它们平方，然后打印
        auto even_squares = numbers
                          | std::views::filter([](int n){ return n % 2 == 0; })
                          | std::views::transform([](int n){ return n * n; });
        // 计算在这里才真正发生
        for (int n : even_squares) {
            std::cout << n << " ";
        }
    }
    ```

1. 格式化库std::fromat

1. std::span
