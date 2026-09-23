---
title: 第三章 · 进程
weight: 4
---

# 1.Process Concept

一些def:
* 进程 = 正在执行的程序
* 批处理系统中叫 job，分时系统中叫 user program 或 task。
* 进程执行必须顺序推进


process includes:
* program counter
* text section：代码
* stack section：函数参数、局部变量、返回地址
* data section：全局变量
* heap section：动态分配的内存(C中malloc出来的那种)

process in memory belike:

![](../image/2026-09-22-17-21-00.png)
* 最底下是0地址，向上增大.最顶上是maxVA(图中展示的是virtual addr)
* Hole:中间没被占用的VA

![](../image/2026-09-23-15-14-23.png)
若一个进程有多个thread：
* 进程是资源分配单位，线程是调度单位
* 每个thread有自己的PC，以及自己的私有栈（蓝色框）。（私有栈之间怎么分隔？？
* 多个thread共享 text,data,heap sectio
* library mappings：？？
  * 把外部的库调进本进程中

## 补充：kernel code 的两种执行上下文

也就是讨论：process和kernel的关系？

![](../image/2026-09-23-15-24-53.png)
* VAS:virtual addr space

kernel code进入时有两种上下文(context)：
* process context(进程上下文)：进程通过sys call陷入kernel
  * 但是此时并非启动了一个kernel process，依旧属于原来的进程。
  * 只是从user mode切换成了kernel mode
* interrupt context(中断上下文)：由中断触发，不在任何进程上下文中
  * 它由硬件驱动，调用kernel中的ISR(其不属于任何进程)。
  * ISR是Interrupt Service Routine，中断服务例程

中断上下文优先级高于进程上下文，也就是说：
* 当硬件中断发生时，内核会暂停当前正在执行的进程上下文（无论是用户态还是内核态），优先去执行中断上下文（ISR）


进程上下文中还有一种kernel thread，是为了维护kernel正常运作的进程。不是从usermode过来的，而是完全在kernel mode之下

## 1.1 Process State

进程的状态：
* 新建(new)：正在创建进程
* 运行中(running)：正在执行指令
* 等待中(waiting)：进程正在等待某些事件的发生（如等待I/O操作完成）
* 就绪(ready)：等待被分配给一个处理器执行
* 已终止(terminated)：执行完毕

上述状态名称可以是任意的，且不同的 OS 可能有不同的名称，不过上述状态是所有 OS 共有的。
需要注意的是，**一个处理器核心在任意时刻只能运行一个进程**，所以很多进程会处于就绪或等待状态。

状态机：
![](../image/2026-09-22-17-40-16.png)
* running中进程被外部信号中断：进入ready.因为没有等待I/O之类的操作

## 1.2 Process Control Block,PCB

PCB,一种表示进程的数据结构。又称任务控制块 (task control block)。如下图

![](../image/2026-09-23-17-02-24.png)

PCB包含很多关于特定进程的信息片段
* process state
* process number??
* program counter：下一条指令的地址
* CPU registers:
  * 不同CPU架构的reg不同
  * 包括累加器、索引寄存器、栈指针、通用目的寄存器，以及任何的状态编码信息
  * 中断发生时要保存好这些reg的信息，后续用来恢复
* CPU-scheduling info:包括进程优先级、指向调度队列的指针，以及其他任何调度参数
* mem-management info:
  * base addr和limit reg(??)\
  * page table或segment info
* accounting info(日志信息)：包括使用的 CPU 和实际时间、账户号码、作业或进程号等
* I/O status info:分配给进程的I/O设备表、打开文件列表

# 2.Process Scheduling

process scheduler:
* 从一组可用的进程中挑选一个进程，放在某个处理器核心上执行。而 CPU 在一段时间内只能执行一个进程。
* 若进程数多于核心数，多出来的进程需要等到核心空闲后才能被重新调度。
* 当前在内存中的进程数被称为多道程序的度(degree)

进程一般可根据I/O和计算时间占比分为：
* I/O-bound:I/O 密集型进程，相比在计算上的时间，花在 I/O 上的时间更多
* CPU-bound:CPU 密集型进程，生成 I/O 请求的频率较低，并且将更多时间花在计算上

## 2.1 Scheduling Queues,调度队列

ready queue:
* ready状态的进程的队列，以链表形式存储
* 队列头部是header，存储指向第一个PCB的指针
* tail指针是干啥的？？
![](../image/2026-09-23-17-30-18.png)

wait queue:
* wait状态的进程的队列。
![](../image/2026-09-23-17-31-37.png)

# 3.Operations on Processes

# 4.Cooperating Processes

# 5.Interprocess Communication

# 6.Communication in Client-Server Systems

