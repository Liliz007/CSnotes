---
weight: 9
---

# 1. 贪心算法的核心思想

什么是贪心算法？它试图解决什么样的问题？

## 1.1 优化问题 (Optimization Problems)

如，如何规划回家的路线才能时间最短？如何在预算内选择商品才能价值最大？

一个标准的优化问题通常包含两个核心要素：
1. 约束 (Constraints)：解决问题时必须满足的条件。例如，你的总花费不能超过预算，你选择的路线必须能从起点到达终点。
2. 优化函数 (Optimization Function)：**一个需要被最大化或最小化的目标。** 例如，最短的时间，最大的价值。

所有满足约束条件的解，我们称之为可行解 (Feasible Solutions)。而在所有可行解中，那个能让优化函数达到最优值（最大或最小）的解，就是我们梦寐以求的最优解 (Optimal Solution)。

## 1.2 贪心方法

“目光短浅，只看眼前”。在每一步决策时都选择当前状态下看起来最好的解（而不考虑整体）。这个“当前最好”的判断标准，我们称之为**贪心准则 (Greedy Criterion)**。

# 2. CASES

## 2.1 活动选择问题

![9-1](../image/2025-11-11-14-24-58.png)

我们希望能最大化可举办的活动数。

### 2.1.1 DP方法

#### DP1

标记每个时间节点，定义
$$
DP[S] = 活动的子集
$$

```c
for each subset S:
    for each activity i not in S:
        DP[S ∪ {i}] = ...
```
复杂度：$O(N2^N)$
* 共有$2^N$个子集，每个最多新加入N个活动

#### DP2

定义：
$$
S_{ij} = 在活动a_i结束之后开始，且在活动a_j开始之前结束的活动集合。
$$

$$
c_{ij}为集合S_{ij}的最优解的大小
$$

在$S_{ij}$中枚举$a_k$（分点），则可以得到递推式：
$$
c_{ij} = \max_{a_k ∈ S_{ij}}\{c_{ik}+c_{kj}+1\},S \ne \Phi
$$

显然当$S_{ij}$ = 空集时，$c_{ij}=0$

>这个也是枚举分点决定左右子问题的，是区间型DP法。
>复杂度：ij组合数为$\frac{N^2}{2}$，分点数为$O(N)$.故总的为$O(N^3)$


此方法复杂度约为$O(N^3)$，类似于矩阵链乘法问题，选择分点的最优解然后对左右子问题递归。

#### DP3

设 $S_i$ 表示活动$a_1,a_2,...,a_i$ 的最大兼容活动集合，其大小记为 $c_i$，那么有

$$
c_i=max\{c_{i-1},c_{k(i)}+1\}
$$

$k(i)$表示在1~i中不与$a_i$冲突的最晚结束的活动。此处我们想考察最后一个元素$a_i$是否在解中，分成“在”与“不在”两种情况考虑。如果**不在**，那么$c_i=c_{i-1}$；如果将$a_i$选入，那么就是前面能够相容的最晚活动集的大小加1，即$c_{k(i)}+1$

>这个是总时间一点点减少，每轮只依赖于更小的下标($c_i$依赖$c_{i-1}$)，是序列型DP

### 2.1.2 Greedy

**选择策略：** 每一步，选择**结束时间最早**的活动。

**steps：** 
   *  将所有活动按结束时间升序排列
   *  将第一个（最早结束的）活动加入解集
   *  从剩下的活动中，剔除与刚选入的活动相冲突的活动
   *  重复步骤2和3，直到没东西可选

#### 这个贪心选择能得到最优解的证明

**1.贪心选择性质**：证明贪心选择不会使结果坏

考虑任意非空子问题 $S_k$，令 $a_m$ 是 $S_k$中结束时间最早的活动，则$a_m$ 在$S_k$ 的某个最大兼容活动子集中（最优子集中）.

![9-2](../image/2025-11-11-15-42-55.png)

>注：$a_m$是全局中最早结束的，所以换到$A_k$中依然是最早的，不会使这个解变坏。

使用 **“交换参数法”** ，即可以假设存在一个最优选择，其中某个元素可能不在贪心选择中，然后通过交换贪心选择和最优选择的元素来构造一个**不可能变差的解**。

**2.最优子结构**：证明全局最优解可以由**贪心选择+剩余子问题的最优解**得到

在活动选择问题中，用贪心策略选择$a_1$ 之后得到子问题$S_1$，那么$a_1$ 和子问题$S_1$ 的最优解合并一定可以得到原问题的一个最优解。

证明：详见wyy讲义。

事实上思路非常简单，就是反证法，如果综合起来不是最优解，那C1也不可能是子问题最优解，因为能找到一个更优的解。事实上以前的动态规划问题的最优子结构也可以采用这样的证明方法，只是动态规划中都认为最优子结构性质太显然所以没有特别强调。

**证明完毕，贪心合理，从而我们有这样的实现：**
每轮选择最早结束的活动，递归地往后解决，**通过迭代消除尾递归。**（尾递归的空间复杂度为$O(1)$，容易通过迭代优化）

#### code

**伪代码**
```c
GreedyActivity(s,f) //s是排序后的活动开始时间数组，f是排序后的活动结束时间数组
{
    n = s.length  //总活动数

    A = a1 //活动1
    last_finish_time = f[1]

    for i from 2 to n:
        if s[i] ≥ last_finish_time: //开始时间晚于当前的最后结束事件，也就是能兼容
        {
            add a_i into ans set A
            last_finish_time = f[i]
        }
    return A
}

```

**真代码**
```c
#include <stdio.h>
#include <stdlib.h>

// 活动结构体定义
typedef struct {
    int start;
    int end;
} Activity;

// 比较函数：按结束时间升序排序
int compare(const void *a, const void *b) {
    Activity *actA = (Activity*)a;
    Activity *actB = (Activity*)b;
    return actA->end - actB->end;
}

void activitySelection(Activity activities[], int n) 
{
    // 按结束时间升序排序
    qsort(activities, n, sizeof(Activity), compare);
    
    printf("选中活动序列:\n");
    int lastEnd = 0;
    for(int i=0; i<n; i++) {
        if(activities[i].start >= lastEnd) {
            printf("[%d-%d] ", activities[i].start, activities[i].end);
            lastEnd = activities[i].end;
        }
    }
}

int main() {
    Activity acts[] = {{1,3}, {2,5}, {3,7}, {5,9}, {8,10}};
    int n = sizeof(acts)/sizeof(acts[0]);
    activitySelection(acts, n);  // 输出：[1-3] [3-7] [8-10]
    return 0;
}

```

#### 贪心复杂度分析

先预处理，将活动按结束时间升序排列。排序算法最优复杂度为$O(NlogN)$

然后遍历一次活动就可以选完。因为已经排序，所以直接往后线性查询就可以，查询复杂度为$O(N)$.

因此总复杂度就是
$$
O(NlogN)+O(N)=O(NlogN)
$$

### 2.1.3 活动选择的变体

**【变体1】（加权活动选择问题）** 现在的问题不再是希望能包容进来的活动越多越好，而是每个活动都有一个权重$w_i$，希望找到一个兼容活动集合，使得总权重最大（前面讨论的问题实际上只是所有活动权重相等的特例）。

事实上，很容易举出反例说明一般的贪心算法失效：

活动$A[1,100]，w_A$ = 100
活动$B[2,3]，w_B$ = 1

显然我们应该选活动A，但是按照上面的贪心，会选择活动B（结束时间最早）。

所以现在可以转向最开头给出的动态规划算法，只是所有的加1应当改为加权重。即

$$
c_{i} = max\{c_{i-1},c_{k(i)}+w_i\},i > 1
$$


其他的见讲义[wyyGreedy](https://yhwu-is.github.io/Teach/tcs/ads/notes/lec9.pdf)

## 2.2 调度问题

假设现在有n个任务，每个任务 i 都有一个正的完成用时 $l_i$ 和一个权重 $w_i$ 。假定只能按一定顺序依次执行这些任务，不能并行。**目标是最小化加权完成时间之和**，即

$$
T=\min_\sigma \Sigma_{i=1}^n \omega_i C_i(\sigma)
$$

* σ是某一种调度方式
* 任务 i 的完成时间$C_i(\sigma)$是$\sigma$中从头开始到任务 i 完成的总用时，即前缀和。（不这么理解的话怎么排都没区别了）

显然我们应该先选权重大、时间短的。并且应该计算这个参数：$\frac{w_i}{l_i}$，并降序排序选择。下面证明此参数的合理性。

**1.调度问题的贪心选择性质**

令 i 是当前$w_i/l_i$ **最大**的任务，则在当前问题下，则一定存在将 i 排在首位的最优调度方式.

![9-3](../image/2025-11-12-14-42-15.png)

**2.调度问题的最优子结构**

在调度问题S中，用贪心策略首先选择了最大的$w_i/l_i$ 对应的任务 i 后，剩下的子问题S1（即在除 i 外的任务中寻找一个最小化加权完成时间之和的解）的最优解 C1 和 i 一起一定构成了原问题的一个最优解C。

![9-4](../image/2025-11-12-14-44-29.png)

# 3. 贪心算法的partial summary

设计步骤：

1. 将最优化问题转化为这样的形式：对其做出一次选择后，只剩下一个子问题需要求解；
2. **证明**做出贪心选择后，原问题总是存在最优解，即贪心选择不会使解变坏；
3. **证明**做出贪心选择后，剩余的子问题满足性质：其最优解与贪心选择组合即可得到原问题的最优解，这样就得到了最优子结构

# 4. Huffman Code

哈夫曼编码希望找到一个字母表的期望长度最小（即期望深度最小）（依据字母出现频率）的前缀编码，类似于缩小二叉树的搜索时间之类的。

>前缀编码：任意一个字母编码都不是另一个字母编码的前缀。从而无需分隔符，只能唯一解码。下面会进一步解释

## 4.1 信息熵

对一个离散型随机变量（就是概统里的定义），其信息熵定义为
$$
H(X)=-\Sigma_{i=1}^n p(x_i)\log_2 p(x_i)
$$

* 可以理解为，对于一个随机变量X，我们需要多少bit来表示它（即它有多少种可能取值，若有31种，则用5个bits(2^5=32)来覆盖他），类似于计组的MUX

由于位数多消耗的成本大，那么我们尽可能用较少的位数来编码大概率事件，长位数编码小概率事件。这样平均下来一个取值使用的bit就少一些，从而降低了总开销。

## 4.2 算法描述

我们以二叉树的形式来构建编码树，信息都存储在叶子节点上，非叶子节点不存储信息。从根节点到叶子节点的路径上的 0 和 1 分别代表左右子树。

![9-4](../image/2025-11-12-15-18-27.png)

上图中，a的前缀编码是00，u是01，x是10等。

下面我们按出现频率做一个优化。假如我们要编码aaaxuaxz，按照频率排序为a,x,u,z

所有字符的总编码长度：$\Sigma_{i=1}^n f_i\times d_i$

其中$f_i$是频率，$d_i$是某字符的深度，也就是它的编码长度。我们要将上面那玩意最小化，也就要提升$f_i$大的那些字符。可以构建这样一棵树：

![9-5](../image/2025-11-12-15-23-25.png)

> 注意，如果所有成员出现频率都差不多，那么Huffman方法就节省不了多少空间，甚至因为要存储编码表而耗费了更多空间。

**正确解码Huffman编码的关键**：此编码有一重要性质，就是任意成员的编码都不是其他成员编码的前缀。即不会出现由于断章取义而导致的解码错误。**这个性质被称为Huffman code的前缀性质**。

![9-6](../image/2025-11-12-15-39-00.png)

* 人工地decode一下上面的string，就会发现根本不会有谬误！

**那么如何构建这棵树才能使得编码天然地具有前缀性质呢？**

## 4.3 构造Huffman编码树

**steps：**
1. 统计字符频率
2. 按照频率从小到大，创建优先队列。**堆中每个节点存该字符的频率**
3. 构建Huffman树：
    * 从最小堆中反复取出两个**最小频率**的节点，创建一个新的父节点。这个父节点的频率是两个子节点频率之和。
    * 新的父节点不代表某个具体的字符，而是作为一个中间节点。将新生成的父节点insert回最小堆中。
    * 这个过程不断重复，直到堆中只剩下一个节点为止。最终剩下的节点就是哈夫曼树的根节点。
4. 生成编码：
    一旦哈夫曼树构建完成，就可以为每个字符分配编码。通过从树的根节点出发，沿着每一条路径到达叶子节点来确定编码。通常，沿左边的边分配"0"，沿右边的边分配"1"。
5. 叶子节点所对应的编码即为该字符的Huffman编码
    
### 算法

**伪代码：**

```c
void Huffman(PriorityQueue heap[],int C) //有C个字符
{
    consider the C characters as C single node binary trees,
     and initialize them into a min heap;
     for ( i = 1; i < C; i++ ) 
     { 
        create a new node;
        /* be greedy here */
        delete root from min heap and attach it to left_child of node;
        delete root from min heap and attach it to right_child of node;
        weight of node = sum of weights of its children;
        /* weight of a tree = sum of the frequencies of its leaves */
        insert node into min heap;
   }
}
```

insert复杂度为$O(logC)$，插入C个，故总复杂度为
$$
T=O(ClogC)
$$

**真代码：**

```c
#include <stdio.h>
#include <stdlib.h>
#define MAX_TREE_HT 100

// 霍夫曼树节点
struct MinHeapNode {
    char data;
    unsigned freq;
    struct MinHeapNode *left, *right;
};

// 最小堆结构
struct MinHeap {
    unsigned size;
    unsigned capacity;
    struct MinHeapNode** array;
};

// 创建新节点
struct MinHeapNode* newNode(char data, unsigned freq) {
    struct MinHeapNode* temp = (struct MinHeapNode*)malloc(sizeof(struct MinHeapNode));
    temp->left = temp->right = NULL;
    temp->data = data;
    temp->freq = freq;
    return temp;
}

// 核心构建函数（完整实现需要约150行代码，此处展示核心逻辑）
void buildHuffmanTree(char data[], int freq[], int size) {
    // 1. 创建最小堆并初始化
    // 2. 循环执行以下操作直到堆中只剩一个节点：
    //    a. 提取两个最小频率节点
    //    b. 创建新内部节点，频率为两者之和
    //    c. 将新节点插入堆
    // 3. 剩余节点即为霍夫曼树的根
}
```

### 结论

若有N个字符，则有2N-1个结点。

### eg

![9-7](../image/2025-11-12-16-03-32.png)

注意每次是取出f最小的节点。（no matter 它是中间节点还是真正的叶子节点）

* $cost = \Sigma f_i \times d_i$