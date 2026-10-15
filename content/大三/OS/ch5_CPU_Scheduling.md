---
title: 第五章 · CPU调度
weight: 6
---

# 1.Basic Concepts

为了multiprogramming，让多道程序都能占用CPU。

CPU utilization,CPU利用率

>利用率并非越高/低越好，要看使用目标。效率or响应速度

CPU Burst和I/O burst
* CPU burst：一堆指令连续运行，主要在使用CPU
* I/O burst：主要在进行I/O，并没有运行指令

![](image/2026-10-15-15-35-32.png)

进程的整个生命周期就是这俩交替出现。

由于要分配资源给不同进程的burst，所以我们需要关注burst的时长。
* 一般burst都很短，所以scheduler要频繁运行，把资源从这个burst完的进程调度给其他进程

考虑schedule的时机，有以下情况
* 进程调用syscall，放弃处理器
* 硬件发生时钟中断，当前进程被Interrupt，从running state变为waiting state
* 进程I/O结束，由waiting-->ready state
* 进程terminate了

其中情况1和4是必须进行调度的；2和3在有些系统上可能是不schedule的。只在1和4下调度的系统为非抢占式系统(nonpreemptive)；4种都调度的则为抢占式系统(preemptive)

Dispatch,指派（也就是调度）
* context switch时save state和restore state产生的延时称为dispatch latency(指派延时)

# 2.Scheduling Criteria



# 3.Scheduling Algorithm

# 4.Multiple-Peocessor Scheduling

# 5.Real-Time Scheduling

# 6.Thread Scheduling

# 7.OS eg

# 8.Algorithm Evaluation