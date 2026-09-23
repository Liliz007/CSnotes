---
title: 第二章 · 操作系统结构
weight: 3
---

# 1. OS service

![](../image/2026-09-17-15-44-51.png)

## 面向用户

**用户接口(user interface)：**
* 图形用户界面(graphic user interface, GUI)
* 触摸屏界面(touch-screen interface)
* 命令行界面(command-line interface, CLI)

**程序执行(program execution)：**加载到内存、运行、（正常 / 异常）退出

**I/O 操作(I/O operations)：**针对文件或 I/O 设备

**文件系统操纵(file-system manipulation)：**
* 读写文件 / 目录
* 创建 / 删除 / 查找 / 罗列文件
* 权限管理

**通信(communication)：**
* 包括同一计算机内不同进程的通信，以及不同计算机系统通过网络连接的进程之间的通信
* 实现技术：共享内存(shared memory)、消息传递(message passing)

**错误检测(error detection)：**
* 错误既可能在硬件层面（CPU、内存、I/O 设备）发生，也可能在软件层面发生
* 解决方法：纠正或终止，有时会返回一个错误码 (error code)

## 面向系统本身

for ensuring the efficient operation of the system itself via resource sharing

**资源分配(resource allocation)：**
* 一些资源可能有特殊的分配码 (allocation code)，另一些资源可能有更通用的请求码 (request code) 和释放码 (release code)

**日志(logging/Accounting)：**使用统计，追踪计算资源的使用，便于管理者配置

**保护和安全(protection and security)：**
* 保护：当多个进程并发运行时，进程间不得相互干扰，这就要确保所有对系统资源的访问是受控的
* 安全：要求用户验证身份

# 2. User OS Interface

CLI

GUI

# 3. System Calls

def:调用OS服务的一种接口，也是user请求kernel服务的唯一合法入口。

## 3.1 API,application programing interface

* 通常通过API进行system call。
* 根据一些library(库函数)中的系统调用陷入kernel mode.

eg：
```c
#include <stdio.h>
int main() {
    printf("Greetings");   // 调用标准 C 库函数
    return 0;
}
```

printf()内部会调用write()这个系统调用，从而进入内核态。内核真正把字符串写到终端。

调用过程图：
![](../image/2026-09-20-16-28-32.png)
* 用户程序 → API → 系统调用接口 → 内核中的具体实现 → 返回状态和结果

API的好处：
* 可移植性：同一 API 可跨不同 OS
* 简化编程：隐藏内核实现细节。
* 由运行时库（如 glibc）管理

上面三条，简单来说就是只需要使用一定的语言来写程序，由程序调用库函数，从而调用不同OS上的system call就行了。不需要区分.(见后面的system call type，不同OS上的名称是不一样的)

## 3.2 system call的传参方式

* register：参数少时可用
* 内存块：传出参数多时，参数可以存在mem中的block或table中。并将这块内存的地址通过reg传给OS.(见下图)
![](../image/2026-09-20-16-38-51.png)
* 用stack（不常用）

eg：
```
read(fd,buffer,max);
//buffer在user space中，承接从kernel space传出的数据
```

### trap frame(中断帧)

* user mode和kernel mode看不见彼此的数据（确保隔离性）
* 两者通过trap frame交换数据
* 各进程有各自独立的trap frame
* 希望通过软件指定save哪些register

### system call num & system call table

调用sys call时，kernel怎么辨认和执行正确的kernel routine(实现)？
* 调用号：每个sys call有一个索引编号，也就是调用号(system call number)
* 调用表：内核维护一个函数指针数组，下标就是系统调用号
* 查表`sys_call_table[number]`得到函数地址，然后调用函数

## 3.3 Types of System Calls

* process control，进程控制
* file management，文件管理
* device management，设备管理
* info maintenance，信息维护
* communications，通信
* protection，保护

命令分类对应：
![](../image/2026-09-20-16-49-08.png)

# 4. System programs

def: system program 是运行在用户态，为程序开发和执行**提供了便利环境**的程序。他不是kernel的一部分

它们可分为：

* file manipulation
* status information
* file modification
* programming language support
* program loading and execution
* communications
* application programs

>kernel里的代码不是system program

>大多数用户眼中的“操作系统”其实是系统程序，而不是真正的系统调用。例如shell、文件管理器、编译器，都是系统程序；它们内部通过系统调用请求内核服务。

# 5. OS Design & Implementation

用户目标vs系统目标

**separation of mechanism(机制) and policy(策略)**
* Policy:What will be done?(**规则**，做什么事)
* mechanism:How to do it?(**平台**，用什么东西实现)

>需要灵活性的东西作为policy.(放到config文件中，方便修改)

eg1：酒店房卡
* mechanism：磁卡读卡器、远程控制的门锁、与安全服务器的连接（系统中固定的、通用的底层实现部件）
* policy：哪些人应在何时被允许进入哪些门（可变的、具体的业务规则或决策）

eg2：
* 配置文件是policy
* 软件是mechanism

eg3：
* ecall是机制
* ecall的具体接口是策略（什么函数做什么 的规则）

# 6. OS Structure

## kernel,内核

def：内核是操作系统中常驻内存、运行在特权态（kernel mode）的核心部分，直接管理硬件资源，并提供系统调用接口。


## 6.1 Simple Structure

MS-DOS
* 未划分为模块
* 接口和功能层次不清晰

![](../image/2026-09-23-14-41-03.png)

## 6.2 Layered OS，分层结构

* 底层是硬件，顶层是用户接口
* 下层功能比上层简单
* 上层是下层功能的集成
* 下层是上层功能的支持

![](../image/2026-09-20-17-19-29.png)

不希望N太大，以免调用关系太复杂

是一个理论模型。

## 6.3 Monolithic Structure(单体结构/巨内核)

* UNIX中，中间只有一个kernel层，其中集成了很多东西，见图
![](../image/2026-09-20-17-22-22.png)

内核是一个大整体，包含文件系统、调度、内存管理等

## 6.4 Microkernel Structure(微内核)

* 中间只保留三个功能：IPC,内存，调度。
* **把很多东西交给user mode**
* user使用自己的功能模块时要通过msg passing，效率不如func call.
* 所以总体而言微内核速度不如巨内核
![](../image/2026-09-20-17-23-52.png)

但是微内核内容少，从而比巨内核更稳定，不容易出bug

## 6.5 kernel modules(内核模块)

内核核心+可动态加载模块（有点像库调用）
* 加载到内存中后，与原生的kernel融为一体
* 可以按需取用
* 模块间通过已知接口调用

modular kernel图示：
![](../image/2026-09-20-17-31-04.png)

## 6.3 other structures

Hybrid（混合结构）：微内核 + 单体/模块混合

exokernel（外核）：内核只负责资源分配，提供低级硬件操作，应用通过定制库使用

unikernel（云服务）:LibOS
* 将应用程序代码和它所依赖的、最小化的操作系统库代码在编译时静态链接在一起，生成一个单一的、自包含的镜像文件
* 适用于云服务，应用程序可在数十毫秒内启动

# 7. Virtual Machines，虚拟机

![](../image/2026-09-20-17-38-32.png)

虚拟机：可以在一台物理机的hardware的基础上创建多台虚拟的计算机，从而运行他们各自的kernel和process

## Hypervisor(虚拟机管理器)

用来隔离各VM使用的处理器和内存

分类：
* Type1：bare-metal hypervisor,裸金属管理系统
* Type2：hosted hypervisor,包了一层hostOS
* 显然，type1更快，但也更贵

![](../image/2026-09-20-17-48-00.png)

container：共享OS和kernel.

# 8. OS Generation

从头开始生成（或构建）一个 OS 的步骤：
1. 编写操作系统源代码（或获取先前编写的源代码）
2. 为将要运行的操作系统配置目标系统
3. 编译操作系统
4. 安装操作系统
5. 启动计算机及其新操作系统

# 9. System Boot

大多数系统的启动进程按如下步骤进行：
1. 上电：CPU从固定内存地址开始执行
2. 固件(Firmware)：BIOS,UEFI,OpenSBI之类运行，进行硬件自检
3. 引导程序
   1. ROM中有一段名为引导程序(bootstrap program) 或引导加载程序(boot loader) 的代码，用于定位内核
   2. 定位内核，加载到内存并启动
4. 内核初始化硬件
5. 根文件系统被挂载

启动流程：

```
上电
  ↓
固件 (BIOS/UEFI/OpenSBI)  [machine mode]
  ↓
Bootstrap loader
  ↓
内核 (Kernel)  [supervisor mode]
  ↓
系统程序 (System Programs)  [user mode]
  ↓
用户应用程序 (Applications)
  ↓
API (printf, open, ...)
  ↓
系统调用 (System Call)  [trap → kernel mode]
  ↓
内核服务 → 硬件
```


