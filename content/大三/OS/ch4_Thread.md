---
title: 第四章 · 线程
weight: 5
---

# 0. Intro

交互性和处理效率如何兼得？
* 交互性：希望操作是碎片化执行，这样相应才及时
* 效率：大段大段地执行，不要老被打断

——使用多线程。

![](image/2026-10-13-16-54-19.png)
* 每个线程都是一个loop，且互相独立
* 线程可以自行Block到等到完成操作

图示：

![](image/2026-10-13-16-58-08.png)
* multithread中：有独立的上下文、可以独立调用函数，因此各thread有自己的registers和stack
* 全局的数据（对process来说全局）对各thread是共享的

多个线程想要share memory要怎么做？——它们天然就共享内存空间，不用再分配共享空间。如果想分配的话要注意设置为全局，不要在线程内定义。

multithreads的好处：
* Responsiveness,响应性好  
  * 运行长操作时，一部分Block住了，其他的也可以继续运行并响应 
* Resource sharing,资源共享
* Economy,经济性：create一个process是很昂贵的，而分配线程就便宜得多
  * 因为分配process时要分配一堆resource，而create thread时直接复用这些资源就好了（只需要进行context switch和一些进程内分配）
* Utilization of MP Arch,方便多核架构
  * 多线程的concurrency(并发度)更好--（并行是要求更严格的并发，所以这里讲并发就可以了）

回顾：multiprogramming vs. multiprocessing？
* Multiprogramming,多道程序设计：通过并发设计，尽量让CPU不闲着（一般讲单核）
* Multiprocessing,多处理器处理：使用多个CPU并行执行任务

![](image/2026-10-21-16-55-08.png)

Concurrency vs. Parallelism
* 并发性：同时处理多个进程，可以是一个core/多个core，每个进程都可以取得进展，但不在意时间上它们是否会有交叠（即使物理上是单核，也可以通过调度实现并发，而在某一时刻并不一定是多任务）
* 并行性：物理上就是并行的，在任意时刻都在同时执行
* 并行要求更高，必须是多核上

见下图

![](image/2026-10-13-17-14-00.png)

# 1. 两种特权级的Thread

## 1.1 User Threads,用户线程

由U模式线程库(thread library)管理多线程。OS不知道多线程的事.若发生I/O中断，OS会把整个进程block住，从而所有线程都不能运行

>Linux中调用pthread_create，Kernel其实是能看到多个线程的

特点：
- 线程的创建、调度、切换都在用户态完成，不需要陷入内核，效率高。
- 调度器不知道User thread的存在，只知道Process。所以它只会根据Kernel thread来调度，User thread要映射到Kernel thread上才能被调度

## 1.2 Kernel Threads,内核线程

OS知道每个进程里有多个线程。从而调度也由OS直接完成，并发性好，能利用多核；但是线程的创建/切换要进入S-mode，开销比U-mode大

>Kernel Thread是主流实现

# 2 Multithreading Model,多线程模型

是将User thread映射到Kernel thread的模型。
* User thread：从用户程序和线程库的角度看到的线程。
* Kernel Thread：操作系统内核能够管理和调度的线程执行实体。

![](image/2026-10-21-16-58-22.png)

有以下三种模型：
* Many-to-one
* One-to-one
* Many-to-many

形象类比：
![](image/2026-10-13-17-40-03.png)

## 2.1 Many-to-One model

![](image/2026-10-21-16-59-43.png)

内核看不到user thread.好处：
* 适用于老旧系统的多线程，直接用library来操作user thread

某一时刻只有一个user thread占用kernel thread在跑。若此线程发生阻塞式system call，kernel thread就会被挂起，从而整个进程都卡住了

![](image/2026-10-13-17-41-16.png)
* 拿到号码就相当于user thread占用kernel thread来运行

## 2.2 One-to-One model

![](image/2026-10-13-17-29-50.png)

应用中创建任意一个user thread，kernel都会给创建一个对应的kernel thread.

对kernel thread的调度等价于对user thread的调度

一个user挂起了，只有它对应的kernel thread被挂起

![](image/2026-10-13-17-42-13.png)

## 2.3 Many-to-Many model

![](image/2026-10-13-17-32-07.png)

m个user thread，n个kernel thread.一般来说m>n

为了应对user thread太多的情况。这个模型下kernel thread只会创建有限个

![](image/2026-10-21-17-00-24.png)

* LWP：Light weight process.

## 2.4 Two-Level model

![](image/2026-10-13-17-39-13.png)

# 3 Threading Issues，多线程的一些问题

## fork()时，复制父进程的所有、还是仅复制调用进程？

* 如果接下来马上exec()，整个子进程被替换，就只剩新程序的主线程了
* 看具体情况

## Thread cancellation，线程取消

* 异步取消(Asynchronous cancellation)：立即终止
* 同步取消(Deferred cancellation)：线程自己决定何时退出

## Signal Handling,信号处理：信号送给哪个线程？

* 送给信号适用的那个线程（比如除零错误送给出错线程）。
* 送给进程里的每个线程。
* 送给进程里某些特定线程。
* 指定一个专门的线程来接收所有信号。

## Thread pool,线程池：创建一批进程并让它们等待工作

* 对于大数据流应用，request太多，需要创建很多线程，而create的开销不小——所以不如直接预制一些thread，有任务来了直接挑出一个来执行
* 执行完了也不退出，继续放回pool中等待
* 若pool中没有free thread了，新的request就必须等待

## Thread Specific Data,线程特有数据

* 除了它们各自的stack之外，开这个数据区的作用：stack用来放局部变量那些的，但有些数据是想要在整个thread中使用的（如线程编号thread-id）
* 所以设置Thread-Local Storage
  * 访问方式：基址（存在一个reg中）+ offset

## Scheduler Activations,调度器激活：主要用于解决user thread被block的问题

考虑以下问题：
* 由于内核只看得到进程，看不到线程，因此当running的thread被block住的话，整个进程就会卡住
* 而kernel thread block时不影响其他线程，但线程操作总要陷入kernel中，开销较大
* 所以我们希望结合user thread的轻量级和内核层面的调度功能。

LWP,Light Weight Process：轻量级处理器。可以理解为直接分配给User process的一个CPU，而LWP背后绑定的是一个kernel thread，由这个kernel thread真正干活

Upcall机制——maintain the number of kernel thread
* 内核通过upcall向用户级线程库发送信息，然后线程库会在现有的LWP上调度上另一个ready的user thread
* 线程库还可以决定是否向kernel申请一个新的LWP过来
* 当一个user thread准备block的时候，就会触发upcall

Pthreads：同时是user-level和kernel-level library

## Linux Threads

![](image/2026-10-15-15-13-33.png)
* linux中管thread叫task.
* linux中没有严格的线程/进程的区分。由clone()出的task，父子俩共用一块addr space，那么这个孩子就可以算是父亲的thread

![](image/2026-10-15-15-24-01.png)
![](image/2026-10-15-15-25-27.png)

**线程共享资源问题：**
* user spase中一个进程的多个thread共享线程的地址空间，但它们有自己的user stack(图中那些小方块)
* 每个user thread又有自己的kernel stack；当其进入kernel后，用这个stack来存kernel mode的上下文

**kernel space中的线程：**
* 它们有自己的task_struct(PCB)和kernel stack
* Kernel thread：图中的kthrd1~3。只在kernel mode下运行，没有自己的user space对应进程

**访问隔离：**
* User space中，所有进程的虚拟地址空间是隔离的（互相不可见）；
* 而kernel space中，多个进程映射到同一个内核区域（一般code区域是共享的）
* 但kernel stack是隔离的（同代码但不同数据）