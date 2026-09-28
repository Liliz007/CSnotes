---
title: 左倾堆 & 斜堆
weight: 4
---

# 左倾堆和斜堆

这一节讲：努力维持不平衡状态的堆。

- 普通的堆：都是完全二叉树，父子之间的关系不仅可以用指针，还可以通过数组索引确定（节点 i 的孩子：2i 和 2i+1 ）。

- 普通的堆做merge：每个节点都要遍历过，一个O(N)和一个O(M)，归并至少要O(M+N)

## 1. Leftist Heaps

目标：在O(N)的基础上加速归并操作

序性质：与普通的堆一样，上小下大或反之

结构特征：非平衡的二叉树

**注：下面我们全部当作小根堆来讨论。**

### 1.1 定义

#### 1.1.1 Npl：空路径长度

1. Npl: Null path length

2. Npl(X)：X到一个**无二孩节点**的最短距离。
   - 特别地，Npl(null) = -1
   - 如果一个结点的左孩子或右孩子为空结点，则该结点的 Npl 为 0(自己到自己)，这种结点被称为外结点；

3. Note: $Npl(X) = min\{Npl(C)+1 | C是X的孩子\}$

#### 1.1.2 左倾堆

堆中的**任何结点**，**其左孩子的Npl大于等于右孩子的Npl**

- 右路径：从根开始，一直沿右孩子走到底的这条路。
- 注意：允许局部左边直观上比右边小个。（所以还是得计算Npl捏）如图

![](../image/2025-11-10-20-52-09.png)



- 如果$Npl(X) = k$（到无二孩结点的最短路径，肯定是右路径），则以X为根的子树至少是一个k+1层的完美二叉树（即1~k全满，k+1层左满，右可缺），如图

![4-1](../image/2025-11-09-12-43-51.png)

- 向左增长的目的：限制右路径的长度。从根一直沿右孩子走到底的【右路径(the right path)】是整棵树中最短的路径

- 我们希望**二堆归并发生在一棵树的右子树**，从而实现每次都是往子树相对小的那一侧塞进东西，从而避免将堆维护成链状结构，保证了堆的（右路径受控）相对平衡性

- 结论：一个右路径长度为$r$的左倾堆。总结点数至少为$2^r - 1$（右路径最长的情况下，此堆为满二叉树。再增大只能增大到左子树）



### 1.2 实现

#### 1.2.1 Declaration

```C
struct LeftistHeapNode {
    ElementType val;
    int Npl;
    LeftistHeapNode * left, * right;
};
```

#### 1.2.2 Merge操作

例：

![](../image/2025-11-10-20-52-26.png)

- 归并后可能右大左小，只需要交换左右子树即可。

- 归并时，H1的10为根的子树不受影响，H2的12为根的子树Npl也不受影响。事实上只有在【右路径】上的结点才需要调整。

以下是调整步骤演示：

![](../image/2025-11-10-20-52-40.png)

- 检查后发现，只有结点3的Npl违反规则，所以交换左右子树即可。

##### （1） 递归版(top down)

- idea：先比较当前两个待合并子树的根结点的键值，选择较小的那个作为根结点，其左子树依然为左子树，右子树更新为「右子树和另一个待合并子树的合并结果」。

在递归地更新完后，我们需要检查左子树和右子树是否满足$Npl(left\_child)>Npl(right\_child) $的性质，如果不满足，我们则需要**交换左右子树**来维持性质。

此处的递归较为特别，不是自身调用自身，而是互相调用，如下：

```C
function1(){
  function2();
}

function2(){
  function1();
}
```

![](../image/2025-11-10-20-52-57.png)

- Merge就是把顶较小的堆排在前面罢了。将大的attach to顶小的

- 递归出口：到叶节点了，再归回去，逐个调整Npl值。

- 复杂度：就是右路径的结点数，即O(logN)

**将merge和merge1合并的写法：**

```C
LeftistHeapNode *merge(LeftistHeapNode *x,LeftistHeapNode *y)
{
  //递归出口：任意一个遍历完毕（到NULL了），把另一个直接作为子树的根节点
    if(x == NULL) return y;
    if(y == NULL) return x;
  
    if(x->val < y->val)
      swap(x,y);  //总是把x作为根，把y合并到x上
  //将x的右子树与y合并，将合并结果设为x的新右子树
    x->right = merge(x->right,y);
  
  //向上归时，检查左右的Npl，若违反则交换。
  if(x->left == NULL || x->left->Npl < x->right->Npl)
    swap(x->left,x->right);
  
   x->Npl = x->right->Npl + 1;
  
   return x;
}
```

##### （2）迭代版(top down)
  - idea : 维护两个额外的指针，分别指向两棵树还没被合并的子树的根，并不断选择较小的那个合并进去，直到两个指针都为空。

  - 例子：

![](../image/2025-11-10-20-53-27.png)

- step1：3小，3的左子树不动；
- step2: 6比8小，将6插入到3的右子树
- step3：7根和8根子树比，7小，将7接到6的右子树...
- step4：合并完之后，调整Npl. 从根节点一直往右找（因为只有【右路径】上的结点可能违反），违规的交换左右子树即可。

![](../image/2025-11-10-20-53-39.png)


![](../image/2025-11-10-20-53-51.png)



代码：(解析等见修佬笔记)

```C
LeftistHeapNode * merge(LeftistHeapNode * x, LeftistHeapNode * y) 
{
    LeftistHeapNode * tx = x, * ty = y; // `tx` & `ty` 是还没merge的子树的根节点（暂存之）
    
    LeftistHeapNode * res = NULL, * cur = NULL; //res是最终merge完的树的根节点，cur是最后被merge的结点
    
    // Begin merging.
    while (tx != NULL && ty != NULL) 
    {
        if (tx->val > ty->val) 
        {
            swap(tx, ty);  //总是把tx作为较小的那个
        }

        // Specially mark the root on the first merge.
        if (res == NULL) 
        {
            res = tx;
            cur = tx;
        } 
        else //res上已经有东西了，那说明cur上也有东西了
        {
            cur->right = tx;  //取最后merge完的结点的右孩子作为下一个tx
            cur = tx;
        }

        // Go on.
        tx = tx->right;
    }

    // Merge the rest of the tree.
    while (ty != NULL) {
        // Specially mark the root on the first merge. (rarely happens but not impossible)
        if (res == NULL) {
            res = ty;
            cur = ty;
        } else {
            cur->rs = ty;
            cur = cur->rs;
        }

        // Go on.
        ty = ty->rs;
    }

    // Adjust the left and right subtrees of all the nodes according to the properties of `dist`.
    // It does the same work as the adjust part in the recursive version. I ignore it here.
    res = adjust(res);

    return res;
}
```

##### （3）复杂度分析

和参与合并的两个左式堆中节点个数的总和有关。其时间复杂度为 O(log n)，这里的 n 是两个要合并的左式堆节点个数之和。这是因为每次合并操作在比较和调整子树的过程中，最多只会沿着树的高度（而左式堆的高度是和其节点个数的对数相关，由其 `Npl` 等性质决定）进行操作，所以其单次合并操作时间复杂度为 O(log n)。

#### 1.2.3 DeleteMin

- 删除根节点
- 归并左右子树(Merge)
- 复杂度 = O(logN)
- 代码：
```C
//返回删除完毕的树的根结点
LeftistHeapNode *del(LeftistHeapNode *cur,int x)
{
  if(cur->val == x)
    return merge(cur->left,cur->right);
  
  else
  {
    if(cur->val > x) 
      //不在此子树中（小根堆，往下都是比cur->val更大的值了
      return cur;
    
    if(cur->left != NULL)  del(cur->left,x);
    if(cur->right != NULL) del(cur->right,x);
    
    update_Npl(cur); //向上归的过程中更新结点们的Npl值
  }
}
```

## 2. Skew Heaps斜堆

### 2.1 定义与性质

斜堆与左倾堆的关系和AVL树与splay树的关系相似

斜堆是左倾堆的简化版本，只用控制其merge的均摊复杂度。

- 目标：M个操作的复杂度为$O(M\log N)$.从而使均摊代价成为$O(\log N)$

**Merge操作：**
总是交换右路径上的节点的左右孩子。

![](../image/2026-01-04-21-19-19.png)

* step1:比较两根。3小，保留3作根，交换它的左右孩子：
![](../image/2026-01-04-21-20-00.png)

* step2:合并6与原来3的右孩子。比较6与原来3的右孩子8，保留6，交换6的左右孩子
![](../image/2026-01-04-21-21-44.png)

* step3:合并6原来的右孩子7和8.保留7，交换7的左右孩子。后续略，总之得到了这个：
![](../image/2026-01-04-21-24-25.png)

* 事实上，由于是递归，所以交换孩子其实发生在往上归的过程。故每次比较的都是原来的右孩子。其实先一直不交换，留到合并完了再逐个交换右路径上的节点的左右孩子也能得到一样的结果。**注意，右路径上最大那个节点依然不用交换孩子**

**斜堆性质**
- 斜堆不维护Npl
- 两右路径所有结点中的**最大的那一个结点**的左右孩子不用做交换。也就是说，递归地往下合并到右路径的最大结点时，我们是将其与一个empty heap进行合并。单堆的合并直接返回其本身即可，不必交换左右孩子
- 斜堆中不保证左倾特性。（右边可能比左边大）
- 递归是自底向上，迭代是自顶向下（迭代更好理解啊感觉）

- 好处：
  - 不用Npl，方便一些
  - 由于没有Npl约束，斜堆的长相比较任意，右路径可以任意长。

>Note：左倾堆和斜堆的右路径期望长度仍是未解之谜

### 2.2 复杂度分析

**结论：** 斜堆的merge操作（插入和删除也算）的均摊复杂度为
$$
T_{amor} = O(\log N)
$$

使用势能分析法（用势能差来刻画实际成本和均摊成本之差）。回顾：

![](../image/2025-11-10-20-51-10.png)

  - 要在任何 i 下都保证均摊和成本大于实际和成本。（不要求每个单独的都大，$\hat{c_i} - c_i$可以有些大于0，有些小于0，最后加和的差大于0，且不要太大即可）

分析：

  - 插入和删除其实都算Merge操作。

  - 实际成本跟右路径结点总数有关，有几个结点就归并几步。

  - 左倾时，右路径长度为O(logN)，但斜堆无法保证左倾，**最坏情况下右路径长度为O(N)。**

  - 不要用右路径结点数作势能函数，因为其是单调的函数。我们希望势能在复杂的步骤能大大降低

  - 引入heavy node.

heavy node def : 

  - heavy node : **右大左小**，即右子树结点数占总结点数一半以上（不含一半）
  - light node : 非重即轻
  - 没错，定义是不对称的，一个结点 i 左右各有一个孩子的话也算轻结点

**我们用整棵树的heavy node数量作势能函数。**



分析：

![](../image/2025-11-10-20-51-29.png)

- 斜堆归并，只有右路径上的结点的轻重情况可能变化。

- 右路径上重结点**一定**变成轻结点

  - 右路径上的轻结点可能会变重（进行交换了），但也可能不变，如图中的结点5.

![](../image/2025-11-10-20-51-41.png)

- $H_i $是堆 i 的右路径长度

- 黑色h表示除右路径外的重结点

- 重结点数上限：归并后所有轻结点都变成重结点，从而可得$T_{amor} - T_{实际} = \phi_{i+1} - \phi_i$（PPT中笔误了！$T_{worst}就是O(N)$）

- 如何保证$T_{amor}$在$O(logN)$内：$T_{amor}$只与右路径上的轻结点数有关。light node是左重右轻，那么其右子树会受到限制。故右路径的总长是$O(logN)$.

均摊分析的一般做法：先猜一个复杂度，再在此基础上定义势能函数。（因为不能把上限猜得太松）



设计势能函数时，要避免使用全局量（如树高之类的）。尽量用有局部保持不变的那种量（如轻重结点数之类）。



