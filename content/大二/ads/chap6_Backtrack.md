---
weight: 6
---

# 1.Introduction

暴力搜索方式解决问题，只能在答案有限的情况下进行。但当答案数太多时，我们如何加快搜索呢？

回溯算法，就像一位经验丰富的探险家，它会系统地探索所有可能的路径，但一旦发现某条路是死胡同，它会立刻“回溯”到上一个岔路口，尝试其他选择，而不是一条道走到黑。这种“走不通就退回”的策略，正是回溯算法的精髓。


- x_k是第k步子空间S_k内的元素。
- 检查第i+1步，若满足条件，则扩展部分解至i+1，并继续往下
- 若整个S_i+1都不满足条件，则说明 x_i 往下的路径是死路，我们需要回溯至第 i-1 步
- 这个提前终止岔路探索的步骤，称为剪枝(Pruning)



# 2.Cases

## 2.1 八皇后

在8*8棋盘中让8个皇后和平共处。（攻击范围：横纵+正反对角）

![](../image/2026-01-05-15-34-30.png)

用一维数组表示queen位置。$Q_i$ = $x_j$ 表示第 i 行的皇后的列序号为 j 

### 2.1.1 Constraints

1. **列约束**: 任何两个皇后不能在同一列。

   * 从而x_1到x_8必须互异，为1~8的一个排列。这样一来，答案空间就从$8^8$缩减到 $8!$ 了

2. **对角线约束**: 任何两个皇后不能在同一对角线。


### 2.1.2 简化分析： 4 queens

#### step1 构建game tree

![](../image/2026-01-05-15-37-37.png)

无backtacking，共64步（探索结点的步数，相当于图中的边。4+12+24+24=64）

#### step2 Perform a DFS

**DFS(depth first search)相关知识：**
不剪枝的话，按照上面的game tree，会先一条路走到底，再回溯一层，换个兄弟之后再往下走。相当于以下的循环嵌套，先遍历最底层。
```c
//不考虑不同列的话
for(i=1;i<=4;i++)
  for(j=1;j<=4;j++)
    for(k=1;k<=4;k++)
      for(z=1;z<=4;z++)
        ...
```
对game tree使用DFS

![](../image/2026-01-05-15-51-18.png)

标黑的点即为不满足constraints的路径，直接剪枝，然后回溯到上一步。这样就节省了很多步（底层的）探索！

最后可找到解：（2，4，1，3）（3，1，4，2）

这个过程会持续到遍历完所有可能性。另外，我们可以注意一下棋盘的对称性，x1=1 和 x1=4，x1=2和x1=3是对称等价的。

> Note : 算法实现中，没有真的构建一个树，他只是帮助理解的手段。我们只要在每一步检查constraints即可。

## 2.2 The Turnpike Reconstruction——计费公路重建

**描述：**
- 若公路上有N个收费站$\{x_1,x_2,...,x_N\}(x_1<x_2<...<x_N)$，则有$N(N-1)/2$段（任意两站之间）可计费路段（一站进路一站出）
- 若有$N(N-1)/2$段可收费路径，能否根据距离集，推算出所有收费站的位置？

**任务**：根据距离集D，重建出这N个点的坐标。假设$x_1$ = 0.（原点下标任意，只要坐标集正确就行）

**示例**：D = {1,2,2,2,3,3,3,4,5,5,5,6,7,8,10}

![](../image/2026-01-05-16-05-13.png)

图中颜色的意味：x5 = 8 和 x2 = 2 在同一层，表示处理距离8的两种方法。

step3中：
- **总是尝试用 D 中当前最大的未使用的距离来放置下一个点**。这样，我们每次处理最大距离时，都只能认为它是跟头尾（x1，x6）的距离，否则会跟“余下的最大距离”这一点产生矛盾。
- 从而我们放置顺序是从两边往中间夹逼。这可以指示我们部分解的进行程度
- 设一个新点的坐标时，要计算它与其他所有已确定点的距离，并将其从距离集中删掉（因为距离集中包含的是所有两两配对之间的距离）
- 注意回溯时要把删掉的距离加回D中!!

> 又注意到，若x5 = 8这条路没有解，由对称性，整个问题就没有解了。

由对称性，用10减去每个点的坐标，得到另一个解：(0,2,4,5,7,10)

### Codes

```C
bool Reconstruct ( DistType X[ ], DistSet D, int N, int left, int right )

{ /* X[1]...X[left-1] and X[right+1]...X[N] are solved。即两侧已经确定了*/
bool Found = false;
if ( Is_Empty( D ) )
  return true; /* solved */

D_max = Find_Max( D );
/* option 1：X[right] = D_max 即先放右边 */
/* check if |D_max-X[i]|∈D is true for all X[i]’s that have been solved */

OK = Check( D_max, N, left, right ); /* pruning.检查新放的点与部分解的距离是否存在于集合中 */

if ( OK ) 
{ /* add X[right] and update D */
    X[right] = D_max;
    for ( i=1; i<left; i++ ) //删除与左侧点的距离
      Delete( |X[right]-X[i]|, D);
    for ( i=right+1; i<=N; i++ ) //删除与右侧点的距离
      Delete( |X[right]-X[i]|, D);

    Found = Reconstruct ( X, D, N, left, right-1 ); //right位置合法，继续递归缩小范围

    if ( !Found ) 
    { /* if does not work, undo */
      //回溯，把删掉的距离放回D中
      for ( i=1; i<left; i++ ) 
        Insert( |X[right]-X[i]|, D);
      for ( i=right+1; i<=N; i++ ) 
        Insert( |X[right]-X[i]|, D);
    }

}

/* finish checking option 1 */
if ( !Found ) 
{ /* if option 1 does not work */

/* option 2: X[left] = X[N]-D_max */

  OK = Check( X[N]-D_max, N, left, right );
    if ( OK ) 
    {

    X[left] = X[N] – D_max;

    for ( i=1; i<left; i++ ) 
      Delete( |X[left]-X[i]|, D);

    for ( i=right+1; i<=N; i++ ) 
      Delete( |X[left]-X[i]|, D);

    Found = Reconstruct (X, D, N, left+1, right );

    if ( !Found ) 
    {

    for ( i=1; i<left; i++ ) 
      Insert( |X[left]-X[i]|, D);

    for ( i=right+1; i<=N; i++ ) 
      Insert( |X[left]-X[i]|, D);

    }

   }

/* finish checking option 2 */

} /* finish checking all the options */
return Found;
}
```




# 3. 回溯通用模板

```c
bool Backtracking ( int i )
{   Found = false;
    if ( i > N )
        return true; /* solved with (x1, …, xN) */
    for ( each xi ∈ Si ) 
    { 
        /* check if satisfies the restriction R */
        OK = Check((x1, …, xi) , R ); /* pruning */
        if ( OK ) 
        {
            Count xi in;
            Found = Backtracking( i+1 );
            if ( !Found )
                Undo( i ); /* recover to (x1, …, xi-1) */
            //进行下一个，即x_{i-1}
        }
        if ( Found ) break; 
    }
    return Found;
}
```

- 递归出口：i>N，说明N步全部走通(Check OK一直为真)，即找到了可行解法。
- 先Check第 i 步是否合法，若合法，把它加入部分解，然后递归调用Backtracking(i+1)...
- 如果递归进行到底部，即i>N了，Found依然为假，则为not found.回溯

# 4. 算法复杂度分析

每次检查时优先探索解集小的空间，从而可以提高剪枝效率。

# 5. 拓展

## 5.1 井字棋——minimax算法

在两人零和博弈中（一方的收益就是另一方的损失），我们可以使用**Minimax算法**来寻找最优走法。

- **思想**: 假设对手每一步都会选择对我们最不利的走法，而我们则在所有可能的走法中，选择那个在最坏情况下结果最好的走法。

- **实现**:
  - 构建游戏状态树。
  - 定义一个**评估函数 `f(P)`** 来量化一个局面 `P` 对自己的“好坏程度”（效用）。例如：`f(P) = (我方能赢的路线数) - (对方能赢的路线数)`
  - 我方（**MAX**层）的目标是最大化 `f(P)`。
  - 对方（**MIN**层）的目标是最小化 `f(P)`。
  - 从叶子节点（搜索的最大深度）开始，因为叶子节点才有确定数值。逐层看本层想要的是min还是max，然后按要求赋值，一路往上。如下图：

![](../image/2026-01-05-16-32-31.png)

这是一个简单的搜索树，最底下1-8是叶子的分数，max层结点值取儿子中最大那个，min取最小，一路传到根。

## 5.2 优化：α-β pruning

### 5.2.1 定义

* minimax的局限性：必须算出全部孩子的效益才能判断父亲的效益
* 优化：剪掉无用的孩子，仅根据部分孩子就可得到父亲的效益

**定义**
1. MAX结点：
   * α = 遍历到的MAX的孩子的最大值
   * β = 当前MAX结点的左边兄弟的最小值
2. MIN结点：
   * α = 当前MIN结点的左边兄弟的最大值
   * β = 遍历到的MIN的孩子的最小值

* 上面定义是好理解的，对于MAX，它的父亲是MIN，那么我们就关心它左边的最小兄弟是不是比它本身将要取的值小。如果是，那么它就会被父亲嫌弃。
* 对于MIN也同理。下面我们详细看剪枝

>注意，α总是代表最大值。

### α剪枝

α总是代表最大值，故α剪枝对应于：舍弃MAX结点的孩子

![](../image/2026-01-05-16-57-29.png)

### β剪枝

β总是代表最小值，故β剪枝对应于：舍弃MIN结点的孩子

![](../image/2026-01-05-16-59-07.png)

### 5.2.3 复杂度

当结合α-β剪枝时，这个搜索算法复杂度为$O(\sqrt{N})$.其中N是game tree的总结点树。

