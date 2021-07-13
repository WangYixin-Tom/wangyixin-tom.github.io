---
title: 数据结构基础
top: false
cover: false
toc: true
mathjax: true
date: 2021-06-09 17:35:27
password:
summary:
tags:
- interview
categories:
- interview
---









## 哈希表

**主要作用**

加快查找速度。时间复杂度可以近似看成O（1）

**缺点**

1.**当更多的数插入时，哈希表冲突的可能性就更大。**对于冲突，哈希表通常有两种解决方案：第一种是线性探索，相当于在冲突的地方后建立一个单链表，这种情况下，插入和查找以及删除操作消耗的时间会达到O(n)，且该哈希表需要更多的空间进行储存。第二种方法是开放寻址，他不需要更多的空间，但是在最坏的情况下（例如所有输入数据都被map到了一个index上）的时间复杂度也会达到O(n)。

2.在决定建立哈希表之前，最好可以估计输入的数据的size。否则，**resize哈希表的过程将会是一个非常消耗时间的过程。**例如，如果现在你的哈希表的长度是100，但是现在有第101个数要插入。这时，不仅哈希表的长度可能要扩展到150，且扩展之后所有的数都需要重新rehash。

3.**哈希表中的元素是没有被排序的。**

## 树

### 二叉查找树（BST）

- 若任意节点的左子树不空，则左子树上所有结点的值均小于它的根结点的值；
- 若任意节点的右子树不空，则右子树上所有结点的值均大于它的根结点的值；
- 任意节点的左、右子树也分别为二叉查找树。
- 没有键值相等的节点（no duplicate nodes）

**缺点**

二叉查找树近似退化为一条链表，这样的二叉查找树的查找时间复杂度顿时变成了` O(n)`

### **平衡二叉树**（AVL）

解决二叉查找树退化成链表。

平衡树具有如下特点：

1、具有二叉查找树的全部特性。

2、每个节点的左子树和右子树的高度差<=1。

#### 怎么插入节点？

比较大小，定位到叶子节点，根据大小插入到叶子节点的左子节点或者右子节点。

#### 问题

因为平衡树要求**每个节点的左子树和右子树的高度差至多等于1**，为了维护绝对平衡，几乎每次插入删除都要进行旋转操作；删除节点的时候，需要维护从被删除节点到根节点这几个节点的平衡，旋转的时间复杂度是O（logn）。

### **红黑树**

解决平衡二叉树插入、删除频繁的场景。

红黑树虽然本质上是一棵二叉查找树，但它在二叉查找树的基础上增加了着色和相关的性质使得红黑树相对平衡，从而保证了红黑树的查找、插入、删除的时间复杂度最坏为O(log n)

#### 性质

红黑树是每个节点都**带有颜色属性的二叉查找树**，颜色或红色或黑色。在二叉查找树强制一般要求以外，对于任何有效的红黑树我们增加了如下的额外要求：

1、节点是红色或黑色

2、根节点是黑色的

3、每个叶节点是黑色的，且为nil

4、每个红色节点的两个子节点都是黑色

5、从任一节点到其每个叶子的所有路径都包含相同数目的黑色节点

#### 好处

与平衡树不同的是，红黑树在插入、删除等操作，**不会像平衡树那样，频繁着破坏红黑树的规则，所以不需要频繁着调整**，这也是我们为什么大多数情况下使用红黑树的原因。

不过，如果你要说，单单在查找方面的效率的话，平衡树比红黑树快。

所以，我们也可以说，**红黑树是一种不大严格的平衡树**。也可以说是一个折中方案。

#### 增加节点

- 首先以二叉查找树的方法增加节点并标记它为红色。
- 设为红色节点后，可能会导致出现两个连续红色节点的冲突，可以通过中心着色和树旋转来调整。 下面要进行什么操作取决于其他临近节点的颜色。

#### 维持平衡

当在对红黑树进行插入和删除等操作时，对树做了修改可能会破坏红黑树的性质。

为了继续保持红黑树的性质，可以通过对结点进行**重新着色**，以及对树进行相关的**旋转操作**，即通过修改树中某些结点的颜色及指针结构，来达到对红黑树进行插入或删除结点等操作后继续保持它的性质或平衡的目的

#### 应用场景

适合排序，查找的场景。

- 容器的基本组成，如Java中的HashMap/TreeMap
- Linux内核的完全公平调度器（CFS）：CFS 背后的主要想法是维护为任务提供处理器时间方面的平衡。任务存储在**以时间为顺序的**红黑树中（由 `sched_entity` 对象表示），对处理器需求最多的任务 （最低虚拟执行时vruntime）存储在树的左側，处理器需求最少的任务（最高虚拟执行时）存储在树的右側。 为了公平。调度器然后选取红黑树最左端的节点调度为下一个以便**保持公平性。**
- Linux中epoll机制的实现：epoll 通过 socket 句柄来作为 key，把 socket 保存在红黑树中。如 图2 所示，每个节点中的数字代表着 socket 句柄。把监听的 socket 保存在红黑树中的目的是，为了在修改监听 socket 的读写事件时，能够通过 socket 句柄快速找到对应的 socket 对象。
- 内存管理与红黑树：Linux内存管理模块使用红黑树来提升虚拟内存的查找速度。`vm_area_struct`是描述进程地址空间的基本管理单元，一个进程往往需要多个`vm_area_struct`来描述它的用户空间虚拟地址，需要使用「链表」和「红黑树」来组织各个`vm_area_struct`。

#### 为什么平衡性好

首先红黑树是不符合AVL树的平衡条件的，即每个节点的左子树和右子树的高度最多差1的二叉查找树。但是提出了为节点增加颜色，红黑是用非严格的平衡来换取增删节点时候旋转次数的降低，任何不平衡都会在三次旋转之内解决，而AVL是严格平衡树，因此在增加或者删除节点的时候，根据不同情况，旋转的次数比红黑树要多。所以红黑树的插入效率更高！

#### 红黑树和哈希表比较

1. 红黑树是有序的，Hash是无序的
2. **时间复杂度上**，红黑树的插入、删除、查找性能都是`O(logN)`而哈希表的插入、删除、查找性能理论上都是`O(1)`，他是相对于稳定的，最差情况下都是高效的。哈希表的插入删除操作的理论上时间复杂度是常数时间的，这有个前提就是哈希表不发生数据碰撞。**在发生碰撞的最坏的情况下，哈希表的插入和删除时间复杂度最坏能达到`O(n)`。**
3. 红黑树占用的内存更小（仅需要为其存在的节点分配内存），而Hash事先应该分配足够的内存存储散列表,即使有些槽可能弃用

### Trie树

又叫字典树、前缀树（Prefix Tree）、单词查找树 或 键树，是一种多叉树结构。

#### 基本性质

- 根节点不包含字符，除根节点外的每一个子节点都包含一个字符。
- 从根节点到某一个节点，路径上经过的字符连接起来，为该节点对应的字符串。
- 每个节点的所有子节点包含的字符互不相同。

#### 优缺点

**优点**

- 插入和查询的效率很高，都为O(m)，其中 m 是待插入/查询的字符串的长度。
- Trie树不用求 hash 值，对短字符串有更快的速度。通常，求hash值也是需要遍历字符串的。
- Trie树可以对关键字按字典序排序。

**缺点**

- 空间消耗比较大。

#### 应用

字符串检索、词频统计、字符串排序、前缀匹配

## 跳表

- 跳表是可以实现二分查找的有序链表；

- 每个元素插入时随机生成它的level；

- 最底层包含所有的元素；

- 如果一个元素出现在level(x)，那么它肯定出现在x以下的level中；

- 每个索引节点包含两个指针，一个向下，一个向右；（笔记目前看过的各种跳表源码实现包括Redis 的zset 都没有向下的指针，那怎么从二级索引跳到一级索引呢？留个悬念，看源码吧，文末有跳表实现源码）

- 跳表查询、插入、删除的时间复杂度为O(log n)，与平衡二叉树接近；