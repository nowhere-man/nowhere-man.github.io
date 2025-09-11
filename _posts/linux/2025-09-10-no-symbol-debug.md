---
layout: post
title: Release版本程序发生coredump定位思路
slug: linux-debug-no-symobl
categories: [Linux]
tags: [Linux]
---

在一个没有符号的release版本二进制文件发生coredump时，由于缺乏符号信息，无法直接看到函数名和变量名。

## 根据地址偏移定位

1. 收集和分析coredump文件
1. 使用gdb加载core文件
    ```bash
    gdb <可执行文件名> <coredump文件>
    ```
1. bt (backtrace)：bt会显示程序崩溃时的函数调用栈。虽然没有函数名，但你会看到一串地址。这些地址是函数在内存中的位置。
2. 获取可执行文件的基址：你可以使用readelf -h <可执行文件>或者objdump -f <可执行文件>来查看可执行文件的基址。
3. 计算偏移量：在gdb的bt输出中，每个地址都是一个绝对地址。你需要用这个绝对地址减去可执行文件的基址，得到一个相对偏移量。
4. 使用objdump或nm：使用objdump -d <可执行文件> | grep <偏移量>来查找这个偏移量附近的汇编代码。
5. 使用IDA Pro装载程序，定位偏移量，会得到函数名和偏移量，右键Text View根据汇编转C代码后推测源码的位置。

## 使用objcopy
```bash
// release文件：             删除调试信息后的二进制文件
// debug文件：       原始带调试信息的二进制文件

// 使用 objcopy 命令把 调试信息 单独保存为一个文件
objcopy --only-keep-debug debug debug_info

// 使用 objcopy命令把调试信息文件与二进制运行文件关联想来
objcopy --add-gnu-debuglink=debug_info release

// 当产生了关联后，再对 test 进行 gdb 调试，就和带调试的程序的一样的效果了
gdb release core_file
```
