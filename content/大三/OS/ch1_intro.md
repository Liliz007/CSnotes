---
title: 第一章 · 概述
weight: 2
---

# 0.目录

overview
* 操作系统干啥的
* 计算机组成
* 计算机体系结构
structure
* OS structure
* OS operations
main parts
* 进程管理
* 内存管理
* 存储管理

# 1.

## 1.1 什么是操作系统

OS的目标：执行用户程序并优化问题处理。

计算机系统的构成：硬件，OS，系统程序和应用程序，user
![](../image/2026-09-15-17-13-43.png)

OS是resource allocator和control program.

OS又叫**kernel**，是指电脑开机后始终运行的那个program.其他program要么是sys program，要么是application program.（通常直接叫kernel了）

功能/特点：abstraction,isolation,sharing

## 1.2 computer startup

boot：计算机启动
bootstrap program：启动程序

# 2. 系统运作

## 2.1 中断

**这块多看一下**

中断
* 发生中断时，跳到interrupt vec指向的代码，即中断服务程序
* 硬件的中断就叫interrupt
* 软件触发的中断称为trap，可能是error也可能是用户故意调用（后者称为system call，系统调用）

不过注意一下RISC-V中的叫法：
所有的中断称作trap，底下分软硬。硬件的还是interrupt，软的是异常/环境调用(ecall)
![](../image/2026-09-15-17-46-08.png)

![](../image/2026-09-22-16-57-39.png)

硬件（如I/O device）触发的中断：异步触发
  * main program并不知道啥时候要进入interrupt handler,是突然收到信号就进了
软件触发的中断：同步触发

![](../image/2026-09-22-17-01-24.png)

## 2.2 I/O structure

两种I/O方法：同步 and 异步，区别于控制何时返回给user program

同步：I/O完成后才把控制权返回给用户程序。程序逻辑是串行的

异步：由于立刻返还控制权，所以要记住所有I/O请求

异步通过device-status table记录I/O请求。完成后通过中断通知系统

# 2. 存储层次

略

multiprogram:把多个程序加载到内存中。（存储角度，关注资源利用率

multitasking:多个用户使用一台机器，但各自都认为自己单独使用整台机器（用户角度，关注交互性
    * timesharing:分时共享

Dual-mode：
* user mode
* kernel mode

mode bit:区分当前是user还是kernel mode

ecall是用户级指令

# Process Management

program是被动实体，process是主动实体

多线程的进程，每个线程有一个程序计数器PC

进程的同步：防止进程访问同一块数据时冲突，将其进行一些同步（？

# Memory management

memory allocation:
* kernel如何把mem分配给每个process？
* 如何管理剩余可用内存？

virtual memory技术

# Storage management

主要讨论文件系统

file system:在一个disk分区上，一批文件及组织它们的目录

# mass storage management

disk scheduling:多个进程同时提出访问请求，如何调度以提效

# I/O subsystem

OS要隐藏hardware的多样性，以便于使用

I/O子系统用处：
* I/O的内存管理，包括缓冲 (在传输数据时临时存储数据) 、缓存 (将部分数据存储在更快的存储中以提高性能) 、假脱机 (一个job的输出与其他job的输入重叠)
* 通用设备驱动程序接口
* 用于特定硬件设备的驱动程序

