---
title: 第八章 · Main Memory
weight: 7
---

# 1. Background

Processing-in-Memory？？

Base and Limit reg:
* 每个进程有自己的地址空间
* 用base和limit register来定义此地址空间的基址和范围；前者指向最低地址，limit代表总大小

## Address Given in Different Ways:

* Symbolic Addr,符号地址：在源代码中的函数、变量等，程序员可读到？？
* Relocatable Addr,可重定位地址：一个模块名字＋一个offset 组成。此模块可移动
* Absolute Addr,绝对地址：在mem中的数字编号地址

## Multistep Processing of a User Program

![](image/2026-10-22-15-49-15.png)

## Binding of Inst&Data to Memory

在这些时期，地址怎么转换？

![](image/2026-10-22-15-50-15.png)

# 2. Swapping

# 3.Contiguous Memory Allocation

# 4. Paging

# 5. Structure of the Page Table

# 6. Segmentation

# 7. eg:The Intel Pentium