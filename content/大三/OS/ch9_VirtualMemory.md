# 1. 概述

VM指的是管理virtual addr和physical addr的一种抽象机制，并不是一个实在的物品

VM将逻辑和物理地址分离，让CPU能看到的逻辑地址比实际上的物理地址空间（这里“物理”只是指Main Memory）大得多。

>有点像皇帝出巡，地方故意在皇帝周围做出很丰饶的景象（但实际上没那么好）

还能轻松实现多个进程共享一块空间

能让进程创建更高效。（有些资源可以留到要用时才去加载）

## 1.1 Virtual-addr Space

VAS中的空洞（与ch8的调度相关的不是一个东西）

由于实际上的stack，heap等这些段很小，所以中间会出现巨大的无法合理访问的hole

![](image/2026-11-05-15-18-52.png)

# 2. Demand Paging，请求式调页

## 2.1 Def

![](image/2026-11-05-15-19-40.png)

* 使用一个Lazy swapper：当一页被需要时才被swap入memory中
* 发生page fault时会trap，然后调页要I/O并更新页表的frame number，当前进程会挂起。
* 这些做完了之后中断恢复，进程回到ready queue中
* 此时valid bit的含义可能不同，由OS查询此页情况
  * 情况一：根本没有给进程分配这个page，访存超出了它应有的logical addr space，这是一次非法访问
  * 情况二：只是表示这个page not in memory，让swapper去换进来就好了

>faster response和more users是指？？

![](image/2026-11-05-15-26-15.png)

* 为了方便disk顺序I/O，一般分配/换出连续的Pages

步骤：
![](image/2026-11-05-15-30-04.png)

## 2.2 Performance of Demand Paging

在这种机制下，EAT的算法又要加入一个新的参数：page fault rate（缺页比例）

![](image/2026-11-05-15-33-19.png)
* 其中swap in占大头
* TLB miss penalty那些的全部隐含到memory access中了

不过Demand Paging只是把一开始要干的活儿均摊到后面来了，所以这些代价其实还是可以接受的。

# 3. COW,Copy-on-Write

创建子进程时，子进程直接把父进程的页表复制一份，从而指向同一批pages：

![](image/2026-11-05-15-40-08.png)

一般进程是隔离的，我们不希望p1对共享资源的modify影响到p2，从而让p1复制一份：

![](image/2026-11-05-15-41-15.png)
* 当发现要写的时候，会陷入OS，然后由OS复制一份给这个进程p1

# 4. Page Replacement

如果内存中没有free page了，就选出没在用的page给它换到disk中。

加设一个Dirty bit：有一些page从disk中进到mem之后没被Modifiy过，是clean的，可以直接覆写它。

步骤：
![](image/2026-11-05-15-49-10.png)
* 注意，被覆写/换出的victim在页表中的valid bit要改成invalid

## 4.1 Page Repalcement Algorithms

# 5. Allocation of Frames

# 6. Thrashing

# 7. Memory-Mapped Files

# 8. Allocationg Kernel Memory

# 9. Other Considerations
