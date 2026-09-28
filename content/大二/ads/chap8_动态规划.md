---
title: chap8_动态规划
weight: 8
---

# 1. Introduction

由斐波那契数列引入。使用递归计算F(N)，会有指数级的复杂度。

$$
F(N) = F(N – 1) + F(N – 2) 
$$

```c
int  Fib( int N ) 
{ 
    if ( N <= 1 ) 
        return  1; 
    else 
        return  Fib( N - 1 ) + Fib( N - 2 ); 
}
```

**Trouble** : 冗余计算太多了，重复算了很多次$F(2)$等。
**Observation** : 子问题间有递推顺序关系
**Solution** : 可以打表，把计算过的子问题记录下来。此处我们只需计算
$F(N-1)$和$F(N-2)$即可往下迭代，空间复杂度也并不高。从而我们可以转化为自底向上（自小到大）的迭代方式

```c
int  Fibonacci ( int N ) 
{   int  i, Last, NextToLast, Answer; 
    if ( N <= 1 )  return  1; 
    Last = NextToLast = 1;    /* F(0) = F(1) = 1 */
    for ( i = 2; i <= N; i++ ) { 
        Answer = Last + NextToLast;   /* F(i) = F(i-1) + F(i-2) */
        NextToLast = Last; Last = Answer;  /* update F(i-1) and F(i-2) */
    }  /* end-for */
    return  Answer; 
}
```

采用循环的解法，时间复杂度降至$O(N)$，空间复杂度仅为$O(1)$。

# 2. 动态规划

**核心特征**
1. **最优子结构** (Optimal Substructure): 一个问题的最优解包含其子问题的最优解。
2. **重叠子问题** (Overlapping Subproblems): 在求解过程中，某些子问题会被反复计算多次。

# 3. Cases

## 3.1 矩阵乘法

矩阵乘法的复杂度分析：若一个$m * n$的矩阵与一个$n * p$的矩阵相乘，则复杂度为 $m * n * p$ 。即有$m * p$个 $n$ 维向量对应相乘后相加。

计算顺序会很大程度上影响复杂度。那么我们希望能找到一个顺序使得时间复杂度最小。

### 3.1.1 对加括号方式暴力枚举  

设$b_n = $加括号的方式数，n是矩阵数目。则易得$b_2=1,b_3=2...$。从而对$b_n$，我们可以用动态规划的方式计算。

![8-1](../image/image.png)
$M_{ij}$表示从$M_i$乘到$M_j$。把两大段的划分点从第一第二个矩阵之间，一直枚举到最后两个矩阵之间。接着，对划分出来的两大段继续用这个方法，我们其实可以得到一个卷积递推式：
$$
b_n = \Sigma_{k=1}^nb_{k-1}b_{n-k}
$$
k是划分点位置，左半边有k-1个矩阵，右半边有n-k个矩阵。


**结论：** 卡特兰数
$$
b_n = O(4^n/n^{1.5})
$$

**Observation** : 
1. 在枚举时，不同的划分点产生的片段中，有很多会相互重叠。
2. 记m[i][j]为计算矩阵链$M_i...M_j$的最少乘法次数。设矩阵$M_i$的维度为$r_{i-1}*r_i$
   * 若i=j，只有一个矩阵，从而m[i][j]=0.
   * 若i< j，则尝试全部分割点，并取乘法次数最少的那个选择:
     * $m[i][j]=\min_{i \le < j}\{cost(M_{i...k}+cost(M_{k+1...j})+两大段相乘代价)\}$
     * 从而得到$m[i][j]=\min_{i \le k <j}\{m[i][k]+m[k+1][j]+ r_{i-1}*r_k*r_j\}$

即如果要这一步最优，则必须要求所有子问题已经取到最优，否则可以举出反例。

### 3.1.2 动态规划优化

可以使用DP中自底向上+打表的方式来做，先算短的矩阵链，再逐步组合为长度为n的矩阵链

* 长度为1：m[i][i]
* 长度为2：m[i][i+1]
...
* 长度为n：m[1][n]，即为最终答案。

如下图所示：

![](../image/2026-01-05-21-51-43.png)

**代码：**
```c
/* r contains number of columns for each of the N matrices */ 
/* r[ 0 ] is the number of rows in matrix 1 */ 
/* Minimum number of multiplications is left in M[ 1 ][ N ] */ 
void OptMatrix( const long r[ ], int N, TwoDimArray M ) 
{   int  i, j, k, L; 
    long  ThisM; 
    for( i = 1; i <= N; i++ )  
        M[ i ][ i ] = 0;   //initialize
    for( k = 1; k < N; k++ ) /* k = j - i */ 
        for( i = 1; i <= N - k; i++ ) 
        { /* For each position */ 
	        j = i + k;    
            M[ i ][ j ] = Infinity; 
	        for( L = i; L < j; L++ ) //枚举分点L
            { 
	            ThisM = M[ i ][ L ] + M[ L + 1 ][ j ] + r[ i - 1 ] * r[ L ] * r[ j ]; 
	            if ( ThisM < M[ i ][ j ] )  /* Update min */ 
		            M[ i ][ j ] = ThisM; 
	        }  /* end for-L */
        }  /* end for-Left */
}
```

**复杂度：** 
我们来数一下 DP 计算中要处理的子问题数量.

子问题数量：所有 $(i, j)$ 组合 ≈ $N^2/2$个(只算上i < j的)

每个子问题内部要枚举 k（分割点），最多 N 个。

所以总运算次数：
$$
O(N^2) * O(N) = O(N^3)
$$

从指数级降为多项式级，这是很大的提升！

## 3.2 OBST：最优二叉搜索树

![8-4](../image/image-3.png)

>给定N个已按特定要求排序的单词$w_1...w_N$，以及他们各自被访问的概率$p_i$构建一个BST，使得搜索的期望时间最小。（期望即跟概率挂钩）

$T(N)=\Sigma_{i=1}^Np_i*(1+d_i)$ 这个公式是有N个结点时的总时间。至于+1，其实可以这么想，如果不加1的话根节点（深度d=0）的概率p就乘的是0，就成无关项了，但是访问还是要耗时的，肯定不对。

**def** : 
$$
\begin{align*}
T_{ij} &::= OBST \ for \ w_i , ……, w_j ( i < j ) ，第i到j个词的OBST \\
c_{ij} &::= cost\ of\ T_{ij}  ( c_{ii}= 0\ if\  j < i )，T_{ij}的期望搜索时间。  \\
r_{ij} &::= root\ of\ T_{ij}  \\
w_{ij} &::= weight\ of\ T_{ij} = \Sigma_{k=i}^jp_k\ ( w_{ii} = p_i )，T_{ij}内结点的总概率和
\end{align*}
$$

注意，$w_{ij}$指的是区间概率和，$w_i$指的是第 i 个word
$c_{ij}=\Sigma_{l=i}^jp_l*(1+depth(l))$

**分割方式：**

选择分点k作为根。其余结点深度都+1.

![8-5](../image/image-4.png)

由于考虑$T_{i,k-1}$和$T_{k+1,j}$时，将它们挂到一个根节点$w_k$之下后，左右两子树的所有结点深度都+1，从而根据查询代价的定义（概率*（深度+1）），要补加一个$w_{ij}$ ，并且顺便将还没加进来的根节点权重 $w_{kk}$ 也算上了。

![8-6](../image/image-5.png)
* 表格中，一大格中左下是当前的$c_{ij}$，右下是当前根节点。
* 最下层是完整的树，然后往上检查如何拼接才能使 $c_{ij} = c_{i,l-1}+c_{l+1,j}+w_{ij}$ 最小 
* 对于上例，$c_{1N}$代价最小的根节点是char。由于原本就是排好序的，用DP的话我们希望访问是`左-->根-->右`，对于BST`左 < 根 < 右`，从而根k左子树只能从i..k-1选，根k右子树只能从k+1...j选。
* char为根，左段是break,case。那么我们找这俩拼成的段——在第2行，取出根break，最后，break右边剩下个case，直接拼到它的右子树。
* char的右子树也同理。


## 3.3 最短路径

给定一个带权有向图，对于所有的顶点对，都找出他们之间的最短路径。

### solution1 ：逐对计算

调用|V|次单源最短路径算法。

复杂度：$T=O(|V|^3)$，对于稀疏图较快。

**Observation:** 由于不同点对间的最短路径上的结点可能有重复，并且母路径最短必然要求子路径最短，因此又是可以使用DP的模型。

### solution2 ：DP

**DP状态定义：** 设$D^k[i][j]$为从点$i$到点$j$的最短路径长度，并且路径上所有中间结点（不含起点和终点）的编号都不大于$k$.

**递推关系：**
![8-7](../image/2025-11-05-20-55-05.png)

* $D^{-1}$就是两点直接相连。与$D^0[i][j]$区分（这个可能中间有个编号为0的结点，如果$ij≠0$的话）
* $D^{N-1}$是因为中间结点就从0~N-1取。
* 若结点k不属于 i 到 j 的最短路径，则更新D中结点k后路径长度不变，即$D^{k-1}=D^{k}$
* 若结点k属于 i 到 j 的最短路径，则可以将 i 与 j 间的路径从结点k处破开，从而就有上图中的下方几个式子。（将k作为端点，自然它就不包含在路径中了，故写$D^{k-1}$）

**代码：**
```c
void AllPairs( TwoDimArray A, TwoDimArray D, int N ) 
{   int  i, j, k; 
    for ( i = 0; i < N; i++ )  /* Initialize D */ 
        for( j = 0; j < N; j++ )
	        D[ i ][ j ] = A[ i ][ j ]; 
    for( k = 0; k < N; k++ )  /* add one vertex k into the path */
        for( i = 0; i < N; i++ ) 
	    for( j = 0; j < N; j++ ) 
	        if( D[ i ][ k ] + D[ k ][ j ] < D[ i ][ j ] ) 
		    /* Update shortest path */ 
		    D[ i ][ j ] = D[ i ][ k ] + D[ k ][ j ]; 
}
```

## 3.4 产品流水线

**模型描述：**
1. 有两条加工流水线
2. 两条线上对同一步骤的的加工时间不同
3. 可以中途换线

如何求出最短加工时间？

![8-7](../image/image-6.png)

>Note : 图中，$t_{ij}$中的 i 表示第 i 条流水线，j 表示加工到第 j 步。

**复杂度：** 由于每一步都有两种选择，因此复杂度为
$$
O(2^N)time + O(N)space.
$$

# 4. 如何设计DP算法？

1. 研究问题的子问题：母问题的最优解是否包含子问题的最优解？子问题是否出现重叠？
2. 递推地优化子问题，遍历不同情况。
   1. 明确DP状态，即定义某个区间的最优指标（如上述case中的$D[i][j]$，$c_{ij}$等）
   2. 明确递推方程
   3. 明确边界条件（如OBST中，区间长度为1时$c_{ii} = p_i$）
   4. 明确计算顺序（自底向上，自顶向下。一般都用循环比较好吧！）

# 5. extension:几个模型

## 5.1 数组/序列DP

* 最长上升子序列（LIS）

* 最大子段和（Kadane）

* 打家劫舍（House Robber）

* 爬楼梯（Fibonacci）

```c
dp[i] = 状态i的最优值
dp[i] = max/min/sum(dp[j] + cost[j→i])  // j < i
```

* “第 i 个”决策依赖“前 i−1 个”的最优解。

* 常见技巧：滚动数组、前缀和优化、单调队列优化。

**也就是说，这个不关心区间，只关心长度，只依赖于更小的i**

## 5.2 区间DP

子问题定义在“区间 $[i, j]$”上，常见于括号匹配、矩阵链、合并石子等问题。

* 矩阵连乘最优次序

* 合并石子/戳气球/加括号最小代价

```c
for (len = 2; len <= n; ++len)
  for (i = 1; i + len - 1 <= n; ++i) {
    j = i + len - 1;
    dp[i][j] = INF;
    for (k = i; k < j; ++k)
      dp[i][j] = min(dp[i][j], dp[i][k] + dp[k+1][j] + cost(i,j));
  }
```

状态：$dp[i][j]$ 表示区间 $[i, j]$ 的最优值。

转移：枚举分割点k。此时复杂度为$O(N^3)$

**关心区间，取哪一段，这一段怎样拆**

## 5.3 背包DP

### 0-1背包

每件物品只能选一次

```c
for i=1;i<=n;i++
  for j=m;j>=w[i];j--
    dp[j] = max(dp[j], dp[j-w[i]] + v[i]);
```
注意此处 j 要倒序枚举。防止重复使用第i个物品
>解释：因为j-w[i] < j,而如果正序枚举，小于 j 的数可能已经被i更新过了，那就选了多次 i ，不符合规则。

### 完全背包

每个物品可以选多次

```c
for i=1;i<=n;i++
  for j=w[i];j<=m;j++
    dp[j] = max(dp[j], dp[j-w[i]] + v[i]);
```

此处 j 正序枚举

