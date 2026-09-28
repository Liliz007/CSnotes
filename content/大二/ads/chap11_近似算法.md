---
weight: 11
---

# 1. Introduction

由于NP-hard类问题的存在，有时我们并不能在多项式时间内找到最优解。我们要么接受指数时间，要么求解特例，要么放弃对最优的执着。

**放宽对“最优”的执着：** 这是我们今天的核心。如果我们不再强求那个遥不可及的最优解，而是转向寻找一个在多项式时间内可以找到的、并且质量有保证的“次优解”或“近似解”，那么问题就变得豁然开朗。

也就是说，我们希望在效率（多项式时间）和效果（接近最优）之间取得最佳平衡。

# 2. Core Concepts

既然是“近似”，就必须有一个精确的数学工具来衡量我们的解到底有多“好”。这个工具就是近似比 (Approximation Ratio)。

## 2.1 近似比(Approximation Ratio)

![11.1](../image/2025-11-24-21-34-18.png)

我们取最大值，是为了忽略掉最大/最小化问题的区别，求同存异。

对不同类型的问题，近似比的意义如下：

![11.2](../image/2025-11-24-21-35-25.png)

**一个算法的近似比为$\rho(n)$，就称他是一个$\rho(n)$-近似算法**

## 2.2 近似方案(Approximation Scheme)

有些算法更加强大，它们允许用户自己决定想要的近似程度。

**def:**  一个近似方案是一种特殊的近似算法。它除了接收问题的实例作为输入外，还接收一个额外的参数$\epsilon$>0，对于任何给定的$\epsilon$，该算法都能沉稳给一个$(1+\epsilon)$-近似算法

从而，我们可以用$\epsilon$来把近似解夹到最优解。$\epsilon$越小，近似解和最优解的差距就越小。

根据时间复杂度对$\epsilon$的依赖程度，又可以把近似方案分成两类：

### 2.2.1 多项式时间近似方案(PTAS)

* PTAS:Polynomial-Time Approximation Scheme ，**$\epsilon$出现在指数**

对于任意fixed ε，算法的运行时间是N的多项式，例如$O(N^{2/\epsilon})$.若ε取得非常小，指数就会很大，复杂度增长就很快（最优解就很难找）。

### 2.2.2 全多项式时间近似方案(FPTAS)

* FPTAS:Fully Polynomial-Time Approximation Scheme.

这是近似算法的圣杯（非常好！）。算法的运行时间同时是N和$1/\epsilon$的多项式，如$O((1/\epsilon)^2N^3)$.这样即使ε很小（精度很高时），算法效率依然保持在多项式级别，复杂度随N的增长不会太快。

# 3. CASES

## 3.1 装箱问题(Bin Packing)

**问题描述：** 给定N个物品，其尺寸分别为$S_1,S_2,...S_N$，其中$0<S_i \le 1$.我们的任务是最小化所用箱子的数目，每个箱子的容量都是单位1.

例如：

![](../image/2025-11-25-14-11-30.png)

这是一个组合优化问题，是NP-hard问题。

>回顾NP-hard:所有的NP都能在多项式时间内规约到问题Q，则称Q是NP-hard的。不要求Q本身是NP的；
>如果Q既是NP-hard，又属于NP，则Q是NPC的

### 3.1.1 Online Algorithm(在线算法)

每次根据现有的数据进行计算，不能undo之前的决策;也看不到未来的数据，因此无法站在全数据集的角度来solve。见下例：

![](../image/2025-11-25-14-35-51.png)

下面展开讲3种on-line算法

##### (1) Next Fit（邻近适应）

最简单、最鼠目寸光。

**规则：** 始终只关注当前正在装填的箱子。当一个新物品到来时，检查它是否能放入当前箱子。如果能，就放进去。如果不能，就**永久关闭**当前箱子，开启一个新箱子来装这个物品。

```c
void NextFit ( )
{   read item1;
    while ( read item2 ) 
    {
        if ( item2 can be packed in the same bin as item1 )
	        place item2 in the bin;
        else
	        create a new bin for item2;
        item1 = item2;
    } /* end-while */
}
```

**定理：** 设最优解为M，这个算法不会使用超过2M-1个箱子。因此$\rho(N)=2$,是一个2-近似算法。

**proof：** 注意到相邻两个箱子的物品体积之和必然大于1，否则他俩就会被装进一个箱子内。 

反证，假设我们在Next Fit下使用了$K \ge 2M$个箱子，分别为$B_1,B_2,...,B_K$.考虑任意的相邻两个箱子，他们所装物品的总尺寸必为$S(B_i)+S(B_{i+1}) > 1$.现在讲这K个箱子两两配对，由于$K \ge 2M$，至少可以配出M对，每一对所装总体积都大于1，则K个箱子所装物品总体积就严格大于M。

但由于最优解是M个箱子，则物品总体积必然小于等于M，这就与上面的结论矛盾了。**这就说明此算法最多使用2M-1个箱子。**

##### (2) First Fit（最先适应）

比Next Fit更有记忆力。

**规则：** 处理每一个新物品时，从第一个箱子开始依次检查，**将它放入第一个能容纳它的已开箱子**。如果所有已开箱子都放不下，才打开一个新箱子。

```c
void FirstFit ( )
{   while ( read item ) {
        scan for the first bin that is large enough for item;
        if ( found )
	place item in that bin;
        else
	create a new bin for item;
    } /* end-while */
}
```

**定理：** 此算法使用的箱子数不会超过$\frac{17M}{10}+2$.(证明略)

这是一个比 Next Fit 更好的近似保证，大约是 1.7-近似。遍历items和箱子要$O(N^2)$通过使用平衡二叉搜索树等数据结构维护箱子的剩余空间，时间复杂度可以优化到$O(NlogN)$

##### (3) Best Fit（最优适应）

更加精打细算一点。

**规则：** 处理每一个新物品时，遍历所有已开箱子，找到一个能放下该物品**且剩余空间最小的箱子**（即最“紧凑”的那个），然后放进去。如果都放不下，才开新箱子。

**定理：** 近似比与First Fit类似，最坏时也是1.7-近似，时间同样可以优化到$O(NlogN)$


**定理：** There are inputs that force any on-line bin-packing algorithm to use at least 5/3 the optimal number of bins.“存在特定的输入，会迫使任何在线装箱算法使用的箱子数至少是最优解的4/3倍（一个更早的结论是5/3）”

### 3.1.2 Off-line Algorithms（离线算法）

先读入所有数据并进行预处理，不急给出答案，具有“上帝视角”。

对于装箱问题，我们分析一下：
* **Trouble maker:** the large items。他们又大又不容易与其他items凑整。因此我们可以考虑优先处理大件。

* **Solution:** 先将 items 降序排列，然后使用First fit 或 Best fit.这样的算法叫做First-fit decreasing(FFD)

![](../image/2025-11-28-17-27-01.png)

**定理：** FFD算法使用的箱子数$N_{FFD}$不会超过$\frac{11}{9}M+\frac{6}{9}$（M是最优解）。这是一个出色的近似比。实践证明，FFD 在绝大多数情况下都表现得极其接近最优。

这有力地说明了：**简单的贪心启发式，** 如果用对地方，就能产生巨大的威力。

## 3.2 背包问题(Knapsack Problem)

### 3.2.1 分数背包:金沙银沙

![](../image/2025-11-28-17-30-22.png)

* weight相当于占背包容量
* $x_i$是percentage

**如果想用贪心：**

我们该对什么greedy?————对价值密度，即$p_i/w_i$

这个版本是很容易得到最优解的。

### 3.2.2 [0-1] 背包：金块银块

把上面那个版本的percentage去掉，要么拿要么不拿。这是背包问题的NP-hard版本。

如果我们依然用上面的greedy，并不能一定得到最优，例子：

![](../image/2025-11-25-14-55-41.png)

* greedy只得到了35.

**如果我们对“能装下的最大价值物品” 和 “价值密度” greedy呢？**

**-->按密度贪心，然后与能装下的单个最大价值物品比较，取较大者**

**定理：** 上面这种算法近似比为2.证明：

![](../image/2025-11-28-17-41-47.png)

* $P_{opt} \le P_{frac}$是显然的，因为0-1背包的最优解可能依然有空间剩余，而分数背包可以填满这些空隙。
* greedy完之后，对于第一个装不下的$p_k$，分数背包可以将其切一小部分装进去。所以有$P_{frac} \le P_{greedy}+p_k$
* 由于算法的输出是$A=max(P_{greedy},p_{max})$，而$P_{opt} \le P_{greedy}+p_k \le P_{greedy}+p_{max}$，则有$P_{opt} \le 2\max\{P_{greedy},p_{max}\}=2A$. 从而近似比为2.
* 注意，$P_{greedy}$是指先对价值密度greedy，一直到装不下新一个而得到的结果。然后将这个结果与全局价值最大之物进行比较。

### 3.2.3 DP

![](../image/2025-11-25-15-12-54.png)

* 对于无法凑出的p：将$W_{i,p}$设为$\infty$

**时间复杂度：** $P_{total}$是所有物品的总价值。因为p从0到$P_{total}$，每个p都遍历一次物品即（n个），故为双层循环，时间就为$O(N \cdot P_{total})$. 由于$P_{total} \le N \cdot p_{max}$，时间也可以写成$O(N^2p_{max})$

但这其实是一个伪多项式，因为当$p_{max}$非常大时，时间复杂度就依赖于输入数值的大小$p_{max}$，而不是输入的长度N了。

但是我们可以通过压缩一下数据大小，降低价值的范围，从而让DP变得可用。

### 3.2.3 FPTAS：缩放与近似的艺术

1. 回顾FPTAS：完全多项式近似算法，时间是$N$和$1/\epsilon$的多项式。
2. 如果直接普通压缩：会损失精度。我们要用$\epsilon$来控制精度损失量。

控制方式：我们希望最终结果满足$\frac{最优利润-输出利润}{最优利润} \le \epsilon$

**算法步骤：** 设物品 i 原利润为$v_i$，最大单个利润$V_{max}=\max_{1 \le i \le N}v_i$

1. given a 精度参数 $\epsilon$
2. 设$K=\frac{\epsilon V_{max}}{N}$.
3. 将每个原利润缩放到$v'_i=\lfloor{v_i/K}\rfloor$.
4. 对新利润用DP求解，得到物品集合S
5. 输出S及其**原利润** $\Sigma_{i∈S}v_i$.

由前所述，总时间为$O(N \cdot P_{total})$，缩放后时间$O(N \cdot P'_{total})=O(N \cdot \lfloor{P_{total}/K\rfloor})=O(N \cdot \lfloor{N/\epsilon \rfloor})$

**近似比证明：**

>注：v(T)是压缩前T的总价值，v'(T)是压缩后总价值。

![](../image/2025-11-28-20-58-51.png)

>上式表明：缩放后每个物品价值最多少了K

![](../image/2025-11-28-20-59-24.png)

![](../image/2025-11-28-20-59-39.png)

最后是一个$(1-\epsilon)$近似。关键在于证明由缩放带来的总误差$\Sigma K$最多为$NK = \epsilon V_{max} \le \epsilon v(S*)$，$v(S*)$是原问题最优解

## 3.3 K-中心问题(K-center Problem)

![](../image/2025-11-25-15-23-02.png)

**问题描述：** 给出N个站点$s_1,...s_N$和一个整数K。目标是从这些站点中选出K个中心，使得每个站点到离他最近的那个中心的距离最小化
1. 离散型：从给出的站点里面选
2. 连续型：从平面上任意取

![](../image/2025-11-25-15-23-42.png)

* $dist(s_i,C)$是点$s_i$到点集C（选取的中心集）的距离。定义为$s_i$到C中最近点的距离。

把每个点划归到最近的中心。半径取能覆盖旗下所有站点的最小半径。

**目标：** 最小化【所有点到最近中心的距离】中最大的那个。（优化最坏情况）

由于连续空间内点无穷多，因此暴力解法（枚举）是不可接受的。就算是离散，也是$C_N^K$，是很差的复杂度。

### 3.3.1 一个失败的Greedy

一个很自然的想法是：

1. 找到一个能使覆盖半径最小的“最佳”单中心位置。
2. 固定它，然后在此基础上再找第二个中心，使得新的覆盖半径最小。
3. 如此迭代K次，找到K个中心

这个策略是**任意差** 的。例如，若K=2，这个策略可能会把第一个中心放在两大簇点的正中间，导致第二个中心无论放在哪里，覆盖半径都非常大。而最优解是在每个簇里各放一个中心。

![](../image/2026-01-06-20-02-03.png)

### 3.3.2 Greedy——try again

如果我们已知最优解$r(C*) \le r$，则我们可以构建一个近似比为2的解。看下方的discussion：

    Take s to be the center, 
    how can we select r so that 
    s can cover all the sites that are covered by C*?

我们只需要取$r = 2r(C^*)$，即可保证覆盖全部的sites。因为以C*为中心，r(C *)为半径的圆，直径为2r(C *)，在任意边界点取半径为2r(C *)必然能覆盖所有点。（画图是很直观的）

最优覆盖半径大小检测器：

```c
Centers  Greedy-2r ( Sites S[ ], int n, int K, double r )
{   Sites  S’[ ] = S[ ]; /* S’ is the set of the remaining sites */
    Centers  C[ ] = 空集;
    while ( S’[ ] != 空集 ) {
        Select any s from S’ and add it to C;
        Delete all s’ from S’ that are at dist(s’, s) <= 2r;
    } /* end-while */
    if ( |C| <= K ) return C;  //不到K个中心就已经能全覆盖了，那么选K个的最优解必然存在。
    else ERROR(No set of K centers with covering radius at most r);
}
```

上面的算法是检测最优半径大小是否可能为r。即，我们假设最优确实取r，则由上面的discussion的内容可知，我们可以构造一个宽松解，其必然能覆盖所有点。

**定理：** 假设此算法对于半径 r 取了超过K个中心，那么任意大小为K的中心集，覆盖半径都必然严格大于r，也就不可能最优取r了

**最优半径r的取法：** 对$r(C^*)$使用二分查找，在0和$r_{max}$（最远两点之间的距离）之间不断取中间值来逼近。

### 3.3.2 更聪明的做法——选离现有中心最远点作为下一个中心

```c
Centers  Greedy-Kcenter ( Sites S[ ], int n, int K )
{   Centers  C[ ] = 空集;
    Select any s from S and add it to C;
    while ( |C| < K ) {
        Select s from S with maximum dist(s, C);
        Add s to C;
    } /* end-while */
    return C;
}
```

**定理：** 这个算法给出了一个K中心集C，满足$r(C) \le 2r(C*)$，其中$r(C*)$是K中心的最优解。

**证明：** 

![](../image/2025-11-28-22-11-50.png)

>两两之间距离都大于2r*是因为每次都取最远。最后一次都大于2r *了，前面的肯定也大于。

![](../image/2025-11-28-22-12-04.png)

### 3.3.3 近似比的极限

这个 2-近似算法是不是已经足够好了？我们还能找到比如 1.5-近似或者 1.99-近似的算法吗？**答案是不能，除非 P=NP。**

![](../image/2025-11-28-22-15-34.png)