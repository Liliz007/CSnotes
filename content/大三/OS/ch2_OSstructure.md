---
title: 第二章 · 操作系统结构
weight: 3
---

# 1. OS 服务（Operating System Services）

操作系统提供的服务可分为两类：面向用户的，和面向系统本身的。

![](../image/2026-09-17-15-44-51.png)

## 1.1 面向用户

**用户接口（user interface）：**
* 图形用户界面（graphic user interface, GUI）
* 触摸屏界面（touch-screen interface）
* 命令行界面（command-line interface, CLI）

**程序执行（program execution）：** 加载到内存、运行、（正常 / 异常）退出

**I/O 操作（I/O operations）：** 针对文件或 I/O 设备

**文件系统操纵（file-system manipulation）：**
* 读写文件 / 目录
* 创建 / 删除 / 查找 / 罗列文件
* 权限管理

**通信（communication）：**
* 包括同一计算机内不同进程的通信，以及不同计算机系统通过网络连接的进程之间的通信
* 实现技术：共享内存（shared memory）、消息传递（message passing）

**错误检测（error detection）：**
* 错误既可能在硬件层面（CPU、内存、I/O 设备）发生，也可能在软件层面发生
* 解决方法：纠正或终止，有时会返回一个错误码（error code）

{{<collapse summary = "用户接口（王道）">}}
分为两类：命令接口和程序接口
1. 命令接口：就是用命令行写操作命令
   1. 联机控制（交互式）
   2. 脱机控制（批处理）
2. 程序接口：由一组系统调用组成。用户用这些系统调用来请求外部设备、管理内存等服务
{{</collapse>}}

## 1.2 面向系统本身

用于保证系统自身高效运行（通过资源共享）。

**资源分配（resource allocation）：**
* 一些资源可能有特殊的分配码（allocation code），另一些资源可能有更通用的请求码（request code）和释放码（release code）

**日志（logging / accounting）：** 使用统计，追踪计算资源的使用，便于管理者配置

**保护和安全（protection and security）：**
* 保护：当多个进程并发运行时，进程间不得相互干扰，这就要确保所有对系统资源的访问是受控的
* 安全：要求用户验证身份

# 2. System Calls（系统调用）

定义：调用 OS 服务的一种接口，也是 user 请求 kernel 服务的唯一合法入口。

## 2.1 API（application programming interface，应用程序接口）

* 通常通过 API 进行 system call。
* 通过调用库函数（library）中的系统调用，陷入 kernel mode。

eg：
```c
#include <stdio.h>
int main() {
    printf("Greetings");   // 调用标准 C 库函数
    return 0;
}
```

printf() 内部会调用 write() 这个系统调用，从而进入内核态，由内核真正把字符串写到终端。

调用过程图：
![](../image/2026-09-20-16-28-32.png)
* 用户程序 → API → 系统调用接口 → 内核中的具体实现 → 返回状态和结果

API 的好处：
* 可移植性：同一 API 可跨不同 OS
* 简化编程：隐藏内核实现细节
* 由运行时库（如 glibc）管理

上面三条，简单来说就是：只需要用一定的语言写程序，程序调用库函数，从而调用不同 OS 上的 system call 就行了，不需要区分（见后面的 system call type，不同 OS 上的名称不一样）。

## 2.2 System Call 的传参方式

* register：参数少时可用
* 内存块：参数多时，参数可以存在内存中的 block 或 table 中，并把这块内存的地址通过 reg 传给 OS（见下图）
  ![](../image/2026-09-20-16-38-51.png)
* 用 stack（不常用）

eg：
```
read(fd,buffer,max);
//buffer在user space中，承接从kernel space传出的数据
```

## 2.3 trap frame（中断帧）

* user mode 和 kernel mode 看不见彼此的数据（确保隔离性）
* 两者通过 trap frame 交换数据
* 各进程有各自独立的 trap frame
* 希望通过软件指定 save 哪些 register

## 2.4 system call number & system call table

调用 sys call 时，kernel 怎么辨认并执行正确的 kernel routine（实现）？
* 调用号：每个 sys call 有一个索引编号，也就是调用号（system call number）
* 调用表：内核维护一个函数指针数组，下标就是系统调用号
* 查表 `sys_call_table[number]` 得到函数地址，然后调用函数

## 2.5 Types of System Calls（系统调用的类型）

* process control，进程控制
* file management，文件管理
* device management，设备管理
* info maintenance，信息维护
* communications，通信
* protection，保护

命令分类对应：
![](../image/2026-09-20-16-49-08.png)

# 3. System Programs（系统程序）

定义：system program 是运行在用户态，为程序开发和执行**提供便利环境**的程序。它不是 kernel 的一部分。

它们可分为：
* file manipulation
* status information
* file modification
* programming language support
* program loading and execution
* communications
* application programs

> kernel 里的代码不是 system program

> 大多数用户眼中的"操作系统"其实是系统程序，而不是真正的系统调用。例如 shell、文件管理器、编译器都是系统程序；它们内部通过系统调用请求内核服务。

# 4. OS Design & Implementation（操作系统的设计与实现）

用户目标 vs 系统目标

**机制（mechanism）与策略（policy）的分离**

* Policy：What will be done?（**规则**，做什么事）
* Mechanism：How to do it?（**平台**，用什么实现）

> 需要灵活性的东西作为 policy（放到 config 文件中，方便修改）

{{<collapse summary = "机制与策略：三个例子">}}

{{< tabs >}}
{{< tab "例1 酒店房卡" >}}

* mechanism：磁卡读卡器、远程控制的门锁、与安全服务器的连接（系统中固定的、通用的底层实现部件）
* policy：哪些人应在何时被允许进入哪些门（可变的、具体的业务规则或决策）

{{< /tab >}}
{{< tab "例2 配置文件 vs 软件" >}}

* 配置文件是 policy
* 软件是 mechanism

{{< /tab >}}
{{< tab "例3 ecall" >}}

* ecall 是机制
* ecall 的具体接口是策略（"什么函数做什么" 的规则）

{{< /tab >}}
{{< /tabs >}}

{{</collapse>}}

# 5. OS Structure（操作系统结构）

## kernel（内核）

定义：内核是操作系统中常驻内存、运行在特权态（kernel mode）的核心部分，直接管理硬件资源，并提供系统调用接口。

## 5.1 Simple Structure（简单结构）

MS-DOS
* 未划分为模块
* 接口和功能层次不清晰

![](../image/2026-09-23-14-41-03.png)

## 5.2 Layered OS（分层结构）

* 底层是硬件，顶层是用户接口
* 下层功能比上层简单
* 上层是下层功能的集成
* 下层是上层功能的支持

![](../image/2026-09-20-17-19-29.png)

不希望 N 太大，以免调用关系太复杂。

是一个理论模型。

## 5.3 Monolithic Structure（单体结构 / 巨内核）

* UNIX 中，中间只有一个 kernel 层，其中集成了很多东西，见图
  ![](../image/2026-09-20-17-22-22.png)

内核是一个大整体，包含文件系统、调度、内存管理等。

## 5.4 Microkernel Structure（微内核）

* 中间只保留三个功能：IPC、内存、调度。
* **把很多东西交给 user mode**
* user 使用自己的功能模块时要通过 msg passing，效率不如 func call
* 所以总体而言微内核速度不如巨内核
  ![](../image/2026-09-20-17-23-52.png)

但是微内核内容少，从而比巨内核更稳定，不容易出 bug。

## 5.5 Kernel Modules（内核模块）

内核核心 + 可动态加载模块（有点像库调用）
* 加载到内存中后，与原生的 kernel 融为一体
* 可以按需取用
* 模块间通过已知接口调用

modular kernel 图示：
![](../image/2026-09-20-17-31-04.png)

## 5.6 Other Structures（其他结构）

* Hybrid（混合结构）：微内核 + 单体 / 模块混合
* exokernel（外核）：内核只负责资源分配，提供低级硬件操作，应用通过定制库使用
* unikernel（云服务）：LibOS
  * 将应用程序代码和它所依赖的、最小化的操作系统库代码在编译时静态链接在一起，生成一个单一的、自包含的镜像文件
  * 适用于云服务，应用程序可在数十毫秒内启动

# 6. Virtual Machines（虚拟机）

![](../image/2026-09-20-17-38-32.png)

虚拟机：可以在一台物理机的 hardware 基础上创建多台虚拟的计算机，从而运行它们各自的 kernel 和 process。

{{<collapse summary = "虚拟机（王道）">}}
没有任何软件支撑的计算机称为裸机；覆盖了软件的机器称为扩充机器或虚拟机
{{</collapse>}}

## Hypervisor（虚拟机管理器）

用来隔离各 VM 使用的处理器和内存。

分类：
* Type 1：bare-metal hypervisor，裸金属管理系统
* Type 2：hosted hypervisor，包了一层 hostOS
* 显然，type 1 更快，但也更贵

![](../image/2026-09-20-17-48-00.png)

container：共享 OS 和 kernel。

# 7. OS Generation（操作系统生成）

从头开始生成（或构建）一个 OS 的步骤：
1. 编写操作系统源代码（或获取先前编写的源代码）
2. 为将要运行的操作系统配置目标系统
3. 编译操作系统
4. 安装操作系统
5. 启动计算机及其新操作系统

# 8. OS Boot（操作系统引导）

大多数系统的启动进程按如下步骤进行：
1. 上电：CPU 从固定内存地址开始执行
2. 固件（Firmware）：BIOS、UEFI、OpenSBI 之类运行，进行硬件自检
3. 引导程序
   1. ROM 中有一段名为引导程序（bootstrap program）或引导加载程序（boot loader）的代码，用于定位内核
   2. 定位内核，加载到内存并启动
4. 内核初始化硬件
5. 根文件系统被挂载

{{<collapse summary = "启动流程from王道">}}

下面是BIOS+MBR架构下的引导过程。总结成一句话就是：
* CPU 醒了找 BIOS ➔ BIOS 自检找硬盘 ➔ 硬盘读 MBR 找分区 ➔ 分区读 PBR 找启动管理器 ➔ 启动管理器加载 OS 内核。

1. 激活CPU：CPU去ROM里找boot程序，此程序让CPU去执行BIOS的代码
2. 硬件自检（POST）：BIOS构建中断向量表，然后通电自检
3. 加载带有OS程序的硬盘：BIOS根据设置好的启动顺序去看存储设备，如果是硬盘先启动，BIOS 就把硬盘的第一个扇区（512 字节）读进内存
4. 加载MBR（主引导记录）：
   1. BIOS 读到的这块硬盘的第一个扇区，叫做 MBR（主引导记录）。
   2. 如果这个扇区最后两个字节是 55 AA（这就是书里说的“特定标识”），说明这是一块系统盘。
   3. MBR里装了一张分区表，它知道硬盘分成了几个区，以及哪个区里装了系统。
5. 扫描硬盘分区表，并加载活动分区：
   1. MBR里有分区表
   2. 如果你的硬盘分了 C、D、E 盘，MBR 会看分区表里哪一个是“活动分区”。找到后，CPU 就准备去读这个活动分区的第一个扇区。
6. 加载PBR（分区引导记录）：活动分区的第一个扇区叫 PBR（Partition Boot Record）。他告诉CPU去这个分区的根目录下，寻找“启动管理器”。
7. 加载启动管理器：PBR 搜索活动分区中的启动管理器，并将其加载到内存中。
   1. 其实这个管理器就是让用户选择操作系统的菜单
8. 加载OS：启动管理器根据用户选择，将OS的内核文件加载进内存，然后开始内核初始化。


{{</collapse>}}