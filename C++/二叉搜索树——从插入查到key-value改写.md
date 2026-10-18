---
tags:
  - 形态/概念
aliases:
  - 二叉搜索树
日期: 2026年10月18日
---


![博客横幅图](https://i-blog.csdnimg.cn/direct/098d6ab5280c4b48b7b0a324f8dc3ac5.jpeg#pic_center)

<p align="center">
  <font color="#FF69B4"><b>Ciallo(∠・ω< )⌒☆</b></font>
</p>

<br>

<table align="center" width="100%">
  <tr>
    <th align="center">系列专栏</th>
    <th align="center">直达链接</th>
  </tr>
  <tr>
    <td align="center">🔤 C 语言</td>
    <td align="center"><a href="https://blog.csdn.net/hahaha6565/category_13108044.html">进入专栏 →</a></td>
  </tr>
  <tr>
    <td align="center">🧱 数据结构</td>
    <td align="center"><a href="https://blog.csdn.net/hahaha6565/category_13156520.html">进入专栏 →</a></td>
  </tr>
  <tr>
    <td align="center">🔧 C++</td>
    <td align="center"><a href="https://blog.csdn.net/hahaha6565/category_13163238.html">进入专栏 →</a></td>
  </tr>
  <tr>
    <td align="center">🐍 Python</td>
    <td align="center"><a href="https://blog.csdn.net/hahaha6565/category_13194466.html">进入专栏 →</a></td>
  </tr>
  <tr>
    <td align="center">🔌 STM32</td>
    <td align="center"><a href="https://blog.csdn.net/hahaha6565/category_13209892.html">进入专栏 →</a></td>
  </tr>
  <tr>
    <td align="center">🐧 Linux</td>
    <td align="center"><a href="https://blog.csdn.net/hahaha6565/category_13217143.html">进入专栏 →</a></td>
  </tr>
</table>

<br>

<p align="center">
  🏠 博客主页：<a href="https://blog.csdn.net/hahaha6565?type=blog">暖焰核心</a><br>
  📦 作者仓库：<a href="https://github.com/warm-flame-core">github</a> | <a href="https://gitee.com/warm-flame-core">gitee</a>
</p>


# 引言

先看两种现成的查找做法各自的毛病

数组加二分查找，查找是 O(log N)。代价是数据得有序，还得能按下标随机跳。想插一个新值，位置找到之后要把后面所有元素整体后移，有序性才保得住，一次插入就是 O(N)

链表反过来，插在哪里都不用挪数据。可要找一个值，只能从头顺着 `next` 一个个比，还是 O(N)

一个嫌插得慢，一个嫌找得慢。要找得快、插得也快，就不能让数据挤在一块连续的内存里，又得让它有某种能跳着排除的顺序

二叉搜索树就可以做到，数据分散存在一个个结点里，靠左右子树的大小约定自己排出顺序。查找时每比较一次，就能排除掉一整棵子树

## 一、二叉搜索树的概念

### 1.1 先看一棵树长什么样

二叉树的每个位置叫**结点**，一个结点最多往下伸出两个分支：左边那个连着**左子树**，右边那个连着**右子树**。没有的子树就是**空树**，可以理解成「这个位置什么都没有」

拿一组数 `{8, 3, 1, 10, 6, 4, 7, 14, 13}` 依次插进一棵这样的树，会得到下图左边的形状。8 在最上面，叫**根结点**；8 左边那一整坨（3、1、6、4、7）是它的左子树，右边那一坨（10、14、13）是右子树

![[C++/attachments/BST概念与重复值.png|二叉搜索树的两棵样例，右树允许插入相等的值]]

**中序遍历**是二叉树上的一种访问顺序：先走完左子树，再访问根，最后走右子树，每个结点都按这个顺序递归下去。上图左边这棵树中序遍历的结果是 `1 3 4 6 7 8 10 13 14`

这个遍历顺序有个用处：判断一棵二叉树是不是二叉搜索树，中序遍历一遍看结果是不是升序就行。后面写的 `InOrder` 不只是用来打印，它同时也是这棵树的自检工具

### 1.2 三条性质

二叉搜索树又叫二叉排序树，它要么是一棵空树，要么满足三条性质：

- 左子树不为空时，左子树上所有结点的值都小于等于根结点的值
- 右子树不为空时，右子树上所有结点的值都大于等于根结点的值
- 左右子树也各是一棵二叉搜索树

对照上图左边那棵：8 的左子树是 3、1、6、4、7，全都小于 8；右子树是 10、14、13，全都大于 8。3 自己又是一棵二叉搜索树，它的左子树 1 小于它，右子树 6、4、7 大于它，一层层下去都成立

这就是上图中序遍历结果 `1 3 4 6 7 8 10 13 14` 为什么是升序的原因：左子树全部小于根、右子树全部大于根，按「左-根-右」走一遍，自然从小排到大

### 1.3 重复值

性质里写的是「小于等于」「大于等于」，没有排除相等。也就是说二叉搜索树**可以**支持插入相等的值，也可以不支持，取决于拿它做什么

后面要学的 `map` 和 `set` 底层就是二叉搜索树，这两个不允许插入相等的值；`multimap` 和 `multiset` 则允许

允许重复值时必须先确定遇到相等值时往哪边走，要**定死一个方向**。全部往右，或者全部往左，但不能这次往右下次往左。约定不一致的话，同一批数据每次建出来的树可能不一样，调试时没法对照

上面那张图的右树就是允许重复值的版本：3 出现了两次、10 出现了两次

第二个 10 挂在第一个 10 的右边，这是一路往右的结果。第二个 3 却挂在 6 的左边，因为比到 3 相等时往右走到了 6，而 3 比 6 小，接下来还得往左。所以「定死一个方向」约束的只是**比到相等那一刻**怎么走，之后的每一步仍然按大小关系来

## 二、性能分析

### 2.1 两种极端形态

同样是 `{8, 3, 1, 10, 6, 4, 7, 14, 13}` 这一组数据，按不同顺序插进去，长出来的树可能完全不同

先约定一下**高度**：这里按「从根往下走多少层到最深的那个空位置」来数，一层算 1，只有根结点的树高度为 1

最好的是完全二叉树，或者接近完全二叉树的形态。高度是 log₂N，查找、插入、删除走一条从根到叶的路，都是 O(log N)

最差的是单支树。数据本来就有序的时候最容易出这种情况：`1, 3, 4, 6, 7, 8, 10, 13, 14` 依次插入，每个新值都比前面所有值大，一路往右挂，最后长成一条斜着的链

![[C++/attachments/BST完全二叉树与单支树.png|完全二叉树与单支树的对照]]

右图这条链的形状已经不叫二叉树了，它就是一条链表，只是每个结点多了个空的左指针。这时候查找退化成遍历，高度是 N，效率 O(N)

### 2.2 复杂度

单看最优情况，二叉搜索树和二分查找一样快。但数据结构不能只按最好的情况承诺，得按最坏情况算账，所以增删查改的复杂度记的是最差形态下的 O(N)

这不是说二叉搜索树没用。它的价值在于**形状是可以被管理的**：不旋转的版本长成什么样全看插入顺序，而 AVL 树和红黑树会在插入删除之后自己调整形状，把高度压回 log N。到那时增删查的骨架一行都不用改，只是在外面套一层旋转

### 2.3 与二分查找的分工

二分查找并不是不够快，它的问题是**存储形式**绑死了

| | 二分查找 | 二叉搜索树 |
| --- | --- | --- |
| 最优查找 | O(log N) | O(log N) |
| 最差查找 | O(log N) | O(N) |
| 插入的代价 | O(N)，要挪数据 | O(N)，找位置，但不挪数据 |
| 删除的代价 | O(N)，要挪数据 | O(N)，找位置，但不挪数据 |
| 存储要求 | 连续内存、按下标随机访问、有序 | 任意内存、结点分散存放 |

两边最坏都是 O(N)，但 N 前面那个常数差得很远。二分查找挪一次数据要拷贝一整片连续内存，二叉搜索树找到一个位置只改两三个指针；而且树上的结点是分散的，插删不会牵动别的结点。数据量大、改动频繁的时候，后者的花费方式更划算

有了这个对照，AVL 和红黑树要解决的问题也就清楚了：把「找」的那一行从 O(N) 压回 O(log N)，同时保住「只动指针」这一点


## 三、插入

### 3.1 空树与非空树

插入的第一步是判空。树里一个结点都没有时，`new` 一个结点直接交给 `_root` 就行，它就是整棵树的根

树不空的时候，得先按二叉搜索树的方向往下走，找到该插的位置，再把新结点接上去

### 3.2 双指针往下走

往下走需要两个指针，不是一个

`cur` 负责比较，每次比完就移到左孩子或右孩子；`parent` 跟在 `cur` 后面一步，记着 `cur` 是从谁那儿走过来的

为什么非要留 `parent`：`cur` 走到 `nullptr` 的时候，说明找到插入位置了，可这个位置本身是空的，没有一个结点能挂东西上去。真正要改的是**上一层结点的孩子指针**，而那一刻 `cur` 已经指空了，回头找不到上一层是谁。`parent` 就是这条回头的路

插入和删除都需要它，因为这两者最终要改指针。查找不需要，它只读不改，走到就返回、走到空就结束

### 3.3 走的方向与重复值

方向就是按 1.2 那三条性质来：比当前结点大往右，比当前结点小往左

相等的时候分两种处理。不支持重复值的版本直接返回 `false`，表示这个值已经在树里，不重复插入；支持重复值的版本要往预先定死的那个方向继续走，走到空再插

注意这里「返回 `false`」的含义：不是插入失败，是**这个值已经存在**。调用方拿到 `false` 时，树的结构没有任何变化

### 3.4 插入的代码

`Insert` 的完整实现：

```cpp
bool Insert(const K& key)
{
	if (_root == nullptr)
	{
		_root = new Node(key);
		return true;
	}

	Node* parent = nullptr;
	Node* cur = _root;
	while (cur)
	{
		if (cur->_key < key)
		{
			parent = cur;
			cur = cur->_right;
		}
		else if (cur->_key > key)
		{
			parent = cur;
			cur = cur->_left;
		}
		else
		{
			return false;
		}
	}

	cur = new Node(key);
	if (parent->_key < key)
		parent->_right = cur;
	else
		parent->_left = cur;

	return true;
}
```

循环里每一次 `parent = cur` 都紧挨着 `cur` 移动，两者始终保持一前一后。循环结束 `cur` 是 `nullptr`，`parent` 停在新结点的父亲上

出循环后那一句 `cur = new Node(key)`，是把刚申请到的结点地址重新赋给 `cur`。这时候 `cur` 已经从「遍历用的游标」变成了「新结点」，接下来用 `parent` 跟新值的比较决定挂左边还是右边

### 3.5 按这组数据长出来的树

`{8, 3, 1, 10, 6, 4, 7, 14, 13}` 依次插入，每个都从根 8 开始比：

- 3 比 8 小，挂 8 左边
- 1 比 8 小、比 3 小，挂 3 左边
- 10 比 8 大，挂 8 右边
- 6 比 8 小、比 3 大，挂 3 右边
- 4 比 8 小、比 3 大、比 6 小，挂 6 左边
- 7 比 8 小、比 3 大、比 6 大，挂 6 右边
- 14 比 8 大、比 10 大，挂 10 右边
- 13 比 8 大、比 10 大、比 14 小，挂 14 左边

![[C++/attachments/BST插入后的树.png|按 8 3 1 10 6 4 7 14 13 依次插入后的二叉搜索树]]

对照图上一路数下来，可以验证每个结点的值都落在它该在的位置上

再插一个 16：比 8 大往右，比 10 大往右，比 14 大往右，14 的右边是空的，挂在 14 右边。红色标出的就是这一路走过来的比较

![[C++/attachments/BST插入新值的路径.png|插入 16 时经过的结点路径]]

### 3.6 重复值实际往哪走

在刚才那棵树上再插一个 3。比 8 小往左，比 3 相等

这时候分支的处理决定了重复值往哪边走。这一版的实现是直接返回 `false`，树不动。如果换成允许重复值的版本，得先定死一个方向，比如统一往右：碰到相等就当作「比当前结点大」处理，于是继续往右走，比 6 小往左、比 4 小往左，最后挂在 4 左边

![[C++/attachments/BST重复值插入方向.png|插入重复值 3 时的走向]]

红色这条路径就是「相等统一往右」的走法

注意新版 3 落在了 4 的左边，不在右侧某条链上。相等值只是**在比较到相等的那一刻往右走**，之后该怎么走还按大小来：新 3 比 6 小、比 4 小，自然落到左边。所以「统一往右」给的是**相等时的方向约定**，不是把相等的值排成一条右向的链

这个约定真正的作用是让插入过程**可复现**。同样一批数据、同样的插入顺序，每次建出来的树完全一样；换成一会儿往右一会儿往左，同一批数据就可能长出不同的形状，调试时没法对照

## 四、查找

### 4.1 路径唯一

查找比插入更简单，只需要一个指针 `cur`，从根开始按大小往左或往右

之所以不需要 `parent`，是因为查找只读不改。找到值就返回，走到空就说明不存在，整个过程中动过的只有 `cur`

路径是唯一的：每个结点比完，要么命中，要么排除掉一整棵子树。走到空的时候，所有被排除掉的子树合起来就是整棵树，再没有别的地方可能有这个值


### 4.2 最多走高度次

每往下走一层，剩余待查的范围就缩小一整个子树。所以查找的比较次数最多等于树的高度

这里的「高度」又是那个会变的量：完全二叉树时是 log N 次，退化成单支树时是 N 次。查找本身的代码没有变化，快慢全看树的形状——这也是二.2 说「形状可以被管理」的意义

### 4.3 重复值时的约定

这一版是**不支持重复值**的版本，`Find` 遇到相等直接返回 `true`，下面这套约定只有在允许重复值时才用得上，先了解思路即可

支持重复值的版本里，同一个值可能有好几个结点。这时候「找到了」不够精确，得约定返回哪一个，一般的约定是返回**中序第一个**

3.6 那棵树上有两个 3：一个在 8 的左孩子，一个在 4 的左孩子。中序遍历一遍，顺序是 1、上面那个 3、4 左边那个 3、4、6、7、8……，中序第一个 3 是**上面那个**

查找从根往下走，8 比 3 大往左，第一个撞上的也是上面那个 3，恰好对上了

换成相等值统一往左挂呢？新的相等值会插到已有值的左子树里，形成一条向左下延伸的关系。中序遍历要先走完左子树才轮到根，所以这一批相等值里**最深的**那个排在最前，最上面的排在最后

这时候查找遇到相等不能立刻返回，得先记下候选、继续往左找，走到空再返回最后记下的那个

所以查找在相等值上怎么走，取决于插入时定的是哪个方向，两者要配套。写到 `multimap` 那一层才需要把这套补上

### 4.4 查找的代码

`Find` 只剩一个游标：

```cpp
bool Find(const K& key)
{
	Node* cur = _root;
	while (cur)
	{
		if (cur->_key < key)
		{
			cur = cur->_right;
		}
		else if (cur->_key > key)
		{
			cur = cur->_left;
		}
		else
		{
			return true;
		}
	}

	return false;
}
```

三个分支的顺序与插入完全一样，只是相等那个分支的处理从「返回 `false` 拒绝重复」变成了「返回 `true` 表示找到」


## 五、删除

三种操作里删除最麻烦。插入只需要在空位上挂一个结点，查找只需要往下走，删除要处理「把结点摘掉之后，它下面挂的东西怎么办」

### 5.1 四种情况的划分

先按被删结点 N 有几个孩子分类：

1. 左右孩子都为空
2. 左孩子为空，右孩子不为空
3. 右孩子为空，左孩子不为空
4. 左右孩子都不为空

前三种的共同点是 N 的「下面」最多只有一个孩子要安置，改一条指针就能把它接走。第四种两个方向都有东西，N 自己不能直接摘，得找替身

四种情况的划分里，情况 1 常常被单独列一条，同时又附带一句「可以当成 2 或者 3 处理，效果是一样的」。原因很直观：情况 1 里 N 的左右孩子都是空，把它当情况 2 处理（左孩子为空），就是把父亲的指针改成 `N->_right`，而 `N->_right` 本来就是 `nullptr`。结果正好是「父亲那一侧清空」

所以真正需要写的分支只有两个：左为空、右为空，情况 1 被并进这两个里的任意一个

### 5.2 情况 1、2、3：改父亲指针

左孩子为空的情况，父亲的对应孩子指针改成 N 的右孩子

右孩子为空的情况，父亲的对应孩子指针改成 N 的左孩子

这两种都只有一处指针改动，但有两个坑

**根结点没有父亲**

删的恰好是根，`parent` 是 `nullptr`。这时候不能去访问 `parent->_left`，得直接改 `_root`：

```cpp
if (parent == nullptr)
{
	_root = cur->_right;
}
```

漏了这个判断，`parent->_left` 就是对空指针解引用，删根必崩

**先接孩子，再 delete**

```cpp
if (parent->_left == cur)
	parent->_left = cur->_right;
else
	parent->_right = cur->_right;

delete cur;
```

顺序不能反。先 `delete cur` 的话，`cur->_right` 这块内存已经被回收了，再去读它拿到的就是随机值。这就是未定义行为，运行结果没有保证：可能当场崩，也可能悄悄接错指针、直到很久以后才出问题

下面两张图对应前三种情况

![[C++/attachments/BST删除情况1和2.png|删除情况 1 与情况 2 的样例：删 1 与删 10]]

左边红圈是 1，左右都空，属于情况 1；右边红圈是 10，左空右不空，属于情况 2

![[C++/attachments/BST删除情况3.png|删除情况 3 的样例：删 14]]

14 的右孩子为空、左孩子是 13，属于情况 3，把 10 的右指针改成指向 13 即可

### 5.3 情况 4：替换法

左右孩子都在的结点不能直接摘。它在树上占着一个位置，这个位置空掉之后，两边挂着的子树没有地方安放

办法是找一个「可以替代它」的结点，把值换过来，然后转而去删那个替代者。替代者必须满足两个条件：换到 N 的位置上之后，整棵树仍然满足搜索树的性质；它自己必须是好删的（最多一个孩子）

满足这两个条件的人选有两个：

- N 左子树里的最大值，也就是左子树最右边的结点
- N 右子树里的最小值，也就是右子树最左边的结点

这两个都行。拿右子树最小值 R 来说，R 是右子树里最小的，所以右子树里其他所有值都大于 R，N 换到 R 的位置后，右边依然全部大于 N；而 R 又是右子树里的值，本来就大于 N 左子树里的所有值。两个方向都保住了

这里取右子树最小值，从 `cur->_right` 出发一路往左走到底

```cpp
Node* rightMinP = cur;
Node* rightMin = cur->_right;
while (rightMin->_left)
{
	rightMinP = rightMin;
	rightMin = rightMin->_left;
}

cur->_key = rightMin->_key;
// 此处少了 cur->_value = rightMin->_value，见第八节

if (rightMinP->_left == rightMin)
	rightMinP->_left = rightMin->_right;
else
	rightMinP->_right = rightMin->_right;

delete rightMin;
```

`rightMin` 走到最左之后，它一定没有左孩子，所以摘掉它就是 5.2 里的「左空」情形，直接把它的右孩子接到 `rightMinP` 上

**`rightMinP` 的初值必须是 `cur`，不能是 `nullptr`**

这一条在删根 8 的时候才会暴露。8 的右孩子是 10，10 就是右子树最小值，`while` 循环一次都不进，`rightMinP` 从头到尾没被赋值。如果它的初值是 `nullptr`，后面 `rightMinP->_left` 就是空指针解引用

赋成 `cur` 之后，`rightMinP` 指向 8、`rightMin` 指向 10，循环不进去，最后 `rightMinP->_left == rightMin` 成立吗？8 的左边是 3，不是 10，所以走 `else` 分支，把 8 的右指针改成 `rightMin->_right`。这正是删掉 10 该做的事

![[C++/attachments/BST删除情况4.png|删除情况 4 的两个样例：删内部结点 3 与删根 8]]

左边删 3：右子树最左是 4，把 4 换到 3 的位置，转去删原来那个 4（4 没有左孩子，好删）

右边删 8：右子树最左就是 10 本身，循环不进，`rightMinP` 的初值在这时候起作用

### 5.4 四种情况与两个分支

讲四种情况，代码里只写两个分支，中间差了一个分支

双空被并进左空之后，代码是：

```cpp
if (cur->_left == nullptr)
{
	// 只可能是情况 1 或情况 2，两者对父亲指针的改法一致
}
else if (cur->_right == nullptr)
{
	// 只可能是情况 3
}
else
{
	// 情况 4，替换法
}
```

在 `_left == nullptr` 时，情况 1 和情况 2 都会进来。区别只在 `cur->_right` 是 `nullptr` 还是真实结点，而代码对这两种的处理完全一样，都是把父亲的指针指向 `cur->_right`。所以不需要再多一个分支


### 5.5 删除的完整代码

三种情况加替换法写在同一个循环里：

```cpp
bool Erase(const K& key)
{
	Node* parent = nullptr;
	Node* cur = _root;

	while (cur)
	{
		if (cur->_key < key)
		{
			parent = cur;
			cur = cur->_right;
		}
		else if (cur->_key > key)
		{
			parent = cur;
			cur = cur->_left;
		}
		else
		{
			// 左为空
			if (cur->_left == nullptr)
			{
				if (parent == nullptr)
				{
					_root = cur->_right;
				}
				else
				{
					if (parent->_left == cur)
						parent->_left = cur->_right;
					else
						parent->_right = cur->_right;
				}
				delete cur;
				return true;
			}
			else if (cur->_right == nullptr)
			{
				if (parent == nullptr)
				{
					_root = cur->_left;
				}
				else
				{
					if (parent->_left == cur)
						parent->_left = cur->_left;
					else
						parent->_right = cur->_left;
				}
				delete cur;
				return true;
			}
			else
			{
				// 左右都不为空，替换法
				Node* rightMinP = cur;
				Node* rightMin = cur->_right;
				while (rightMin->_left)
				{
					rightMinP = rightMin;
					rightMin = rightMin->_left;
				}

				cur->_key = rightMin->_key;

				if (rightMinP->_left == rightMin)
					rightMinP->_left = rightMin->_right;
				else
					rightMinP->_right = rightMin->_right;

				delete rightMin;
				return true;
			}
		}
	}

	return false;
}
```

查找部分和 `Insert`、`Find` 一模一样，都是双指针往下走；差别在于找到之后不是返回，而是进入四种情况的判断

这份代码按 `{8, 3, 1, 10, 6, 4, 7, 14, 13}` 建树后逐个删，每一步之后中序打印都有序：删 1（双空）、删 10（左空）、删 14（右空）、删 3（两个孩子）、删 8（根，右子树最小值就是 10 本身）全部正确；删一个不存在的值返回 `false` 且树不变


## 六、key 版完整实现

### 6.1 结点结构

```cpp
template<class K>
struct BSTNode
{
	K _key;
	BSTNode<K>* _left;
	BSTNode<K>* _right;

	BSTNode(const K& key)
		:_key(key)
		, _left(nullptr)
		, _right(nullptr)
	{}
};
```

结点用 `struct` 而非 `class`，成员默认公开——树本身要直接访问 `_key`、`_left`、`_right`

构造函数把两个指针初始化成 `nullptr`，这一步不能省。`new` 只负责分配内存，不给内置类型的成员填初值，那两个指针会停在上一次使用留下的内容上；后面判 `_left == nullptr` 就可能误判成「有孩子」

### 6.2 类骨架

```cpp
template<class K>
class BSTree
{
	using Node = BSTNode<K>;
public:
	bool Insert(const K& key);
	bool Find(const K& key);
	bool Erase(const K& key);
	void InOrder();
private:
	void _InOrder(Node* root);
	Node* _root = nullptr;
};
```

`using Node = BSTNode<K>` 是类内的类型别名，后面的成员函数里可以直接写 `Node*`，不用每次把 `BSTNode<K>` 写全

`_root` 给了一个默认成员初始值 `nullptr`。这样即使不写构造函数，`BSTree<int> t;` 造出来的对象里 `_root` 也是空的，`Insert` 第一次进来就能正确走「空树」分支

### 6.3 中序遍历为什么要套一层

类外调用 `t.InOrder()` 时不带参数，因为调用方拿不到 `_root`——它是私有的

但递归遍历必须有「当前访问到哪个结点」这个参数。两个要求冲突，解法是拆成两个函数：公开的 `InOrder` 不带参数，内部调用私有的 `_InOrder(_root)`；`_InOrder` 带结点参数，自己递归自己

```cpp
void InOrder()
{
	_InOrder(_root);
	cout << endl;
}

private:
void _InOrder(Node* root)
{
	if (root == nullptr)
	{
		return;
	}

	_InOrder(root->_left);
	cout << root->_key << " ";
	_InOrder(root->_right);
}
```

这个「公开接口套私有递归」的模式，凡是需要对外隐藏 `_root` 又要递归的操作都要用。`Copy` 和 `Destroy` 也是同样的写法

### 6.4 key 版不写三件套

这个类里没有拷贝构造、赋值运算符、析构函数

`_root` 有默认初始值，构造函数可以不写。但**析构不写就是内存泄露**：`Insert` 里 `new` 出来的结点，没有 `delete` 去回收，树对象销毁时这批结点全部留在堆上

更麻烦的是拷贝。默认拷贝构造逐成员拷贝，两个 `BSTree` 对象的 `_root` 会指向同一批结点。这两个对象谁先销毁，谁就把这批结点 `delete` 一遍；后销毁的再 `delete` 一遍，就是 double free

这两个问题的性质不一样：

- 泄露是「一定发生」的，只要析构不写，结点就一直在堆上
- double free 是「拷贝才发生」的，不拷贝两个对象就不会撞上

这个版本之所以能跑，是因为测试的时候每次只造一个 `BSTree`，既没拷贝、也没人在意那几个结点的回收。**不拷贝，就只有泄露没有 double free**

所以这属于「够用就不写」的取舍，不是正确做法。到了 key/value 版，`V` 很可能是 `string`，结点里有了需要析构的资源，三件套必须补上——那时候泄露的就不只是结点本身，还有结点里 `string` 占的内存

### 6.5 完整代码

结点、类、三个接口和中序遍历拼在一起。这是教学版，按 6.4 的分析没有补三件套：

```cpp
namespace key
{
	template<class K>
	struct BSTNode
	{
		K _key;
		BSTNode<K>* _left;
		BSTNode<K>* _right;

		BSTNode(const K& key)
			:_key(key)
			, _left(nullptr)
			, _right(nullptr)
		{}
	};

	template<class K>
	class BSTree
	{
		using Node = BSTNode<K>;
	public:
		bool Insert(const K& key)
		{
			if (_root == nullptr)
			{
				_root = new Node(key);
				return true;
			}

			Node* parent = nullptr;
			Node* cur = _root;
			while (cur)
			{
				if (cur->_key < key)
				{
					parent = cur;
					cur = cur->_right;
				}
				else if (cur->_key > key)
				{
					parent = cur;
					cur = cur->_left;
				}
				else
				{
					return false;
				}
			}

			cur = new Node(key);
			if (parent->_key < key)
				parent->_right = cur;
			else
				parent->_left = cur;

			return true;
		}

		bool Find(const K& key)
		{
			Node* cur = _root;
			while (cur)
			{
				if (cur->_key < key)
				{
					cur = cur->_right;
				}
				else if (cur->_key > key)
				{
					cur = cur->_left;
				}
				else
				{
					return true;
				}
			}

			return false;
		}

		bool Erase(const K& key)
		{
			Node* parent = nullptr;
			Node* cur = _root;

			while (cur)
			{
				if (cur->_key < key)
				{
					parent = cur;
					cur = cur->_right;
				}
				else if (cur->_key > key)
				{
					parent = cur;
					cur = cur->_left;
				}
				else
				{
					if (cur->_left == nullptr)
					{
						if (parent == nullptr)
						{
							_root = cur->_right;
						}
						else
						{
							if (parent->_left == cur)
								parent->_left = cur->_right;
							else
								parent->_right = cur->_right;
						}
						delete cur;
						return true;
					}
					else if (cur->_right == nullptr)
					{
						if (parent == nullptr)
						{
							_root = cur->_left;
						}
						else
						{
							if (parent->_left == cur)
								parent->_left = cur->_left;
							else
								parent->_right = cur->_left;
						}
						delete cur;
						return true;
					}
					else
					{
						Node* rightMinP = cur;
						Node* rightMin = cur->_right;
						while (rightMin->_left)
						{
							rightMinP = rightMin;
							rightMin = rightMin->_left;
						}

						cur->_key = rightMin->_key;

						if (rightMinP->_left == rightMin)
							rightMinP->_left = rightMin->_right;
						else
							rightMinP->_right = rightMin->_right;

						delete rightMin;
						return true;
					}
				}
			}

			return false;
		}

		void InOrder()
		{
			_InOrder(_root);
			cout << endl;
		}

	private:
		void _InOrder(Node* root)
		{
			if (root == nullptr)
			{
				return;
			}

			_InOrder(root->_left);
			cout << root->_key << " ";
			_InOrder(root->_right);
		}

		Node* _root = nullptr;
	};
}
```


## 七、key 场景与 key/value 场景

### 7.1 两种场景的分界

同样是二叉搜索树，按照「结点里存什么」可以分成两种用法

一种是**只存 key**。key 就是关键码，也是要查找的值本身。结点里没有别的东西，搜索的目的就是判断这个值在不在树里

另一种是**存 key 和 value**。key 依然按搜索树的规则排队、决定往哪走；value 是挂在 key 上的附加信息，随便什么类型都行。搜索时仍以 key 为关键字，找到 key 的同时就拿到了它对应的 value

### 7.2 支持与不支持的操作

这两种场景差的不只是「多存一个字段」，能做的操作范围也不同

| | key 场景 | key/value 场景 |
| --- | --- | --- |
| 结点内容 | 只有 key | key + value |
| 增 | 支持 | 支持 |
| 删 | 支持 | 支持 |
| 查 | 支持 | 支持 |
| 改 key | **不支持** | **不支持** |
| 改 value | 不适用 | **支持** |

两个场景都不能改 key。原因是结构性的：结点在树里的位置是由 key 的大小关系定下来的，改了 key 就等于让这个结点「不在该在的位置上」，整棵树的有序性被破坏，查找会失效。要改 key 只能先删再插

key/value 场景比 key 场景多出来的是**改 value**。这一点很关键：value 不参与排队，改它不影响结构，所以随便改

### 7.3 key 的实际运用

小区无人值守车库。物业把买了车位的业主车牌号录进后台，车辆进小区时扫描车牌，在系统里就抬杆，不在就提示非本小区车辆

这个场景只需要判断「在不在」，不需要任何附加信息，就是纯 key 场景

检查一篇英文文章的单词拼写。把词库里的所有单词放进二叉搜索树，逐个读文章里的单词去查，不在的标红提示

### 7.4 key/value 的实际运用

中英互译字典。结点里存 key（英文单词）和 value（中文释义），输入英文，同时得到中文

商场停车计费。车辆进场时扫车牌，记录「车牌 → 入场时间」；出场时按车牌查出入场时间，用当前时间减去入场时间算出停车时长，再算费用

统计一篇文章里每个单词出现的次数。读一个单词去查：不存在说明第一次出现，插入 `(单词, 1)`；存在就把那个结点里的次数加一


## 八、key/value 版实现

### 8.1 结点多一个 value

```cpp
template<class K, class V>
struct BSTNode
{
	K _key;
	V _value;
	BSTNode<K, V>* _left;
	BSTNode<K, V>* _right;

	BSTNode(const K& key, const V& value)
		:_key(key)
		, _value(value)
		, _left(nullptr)
		, _right(nullptr)
	{}
};
```

模板参数从 `<class K>` 变成 `<class K, class V>`，构造函数多收一个 `value`

### 8.2 Find 必须返回结点指针

这是整个 key/value 版存在的前提，也是两个版本最大的差别

7.4 里统计词频的写法，核心是这一句：

```cpp
auto ret = countTree.Find(str);
if (ret == nullptr)
{
	countTree.Insert(str, 1);
}
else
{
	ret->_value++;
}
```

`ret->_value++` 要求 `ret` 是一个**能访问到 `_value` 的东西**。`Find` 返回 `bool` 的话，拿到的是 `true` 或者 `false`，是值不是一个位置，`_value` 根本无处可取

所以 key/value 版的 `Find` 返回 `Node*`：

```cpp
Node* Find(const K& key)
{
	Node* cur = _root;
	while (cur)
	{
		if (cur->_key < key)
		{
			cur = cur->_right;
		}
		else if (cur->_key > key)
		{
			cur = cur->_left;
		}
		else
		{
			return cur;
		}
	}

	return nullptr;
}
```

找到了返回结点地址，没找到返回 `nullptr`。调用方拿到地址之后，可以读 `_value`，也可以直接改它

**key 版为什么不需要这样**：key 场景只判断存在性，`true`/`false` 的信息量已经够了。没有 value 要改，就不需要拿到结点的地址。这是「返回类型由用途决定」的一个具体例子

### 8.3 拷贝、赋值与析构的特殊之处

拷贝构造、赋值运算符、析构函数，在 `类和对象` 那一篇里是通用知识。

**拷贝：深拷贝，必须递归**

默认拷贝构造逐成员拷贝，`_root` 指针复制过去，两个对象指向同一批结点。要正确拷贝，得按树的结构递归地复制每一个结点：

```cpp
BSTree(const BSTree<K, V>& t)
{
	_root = Copy(t._root);
}

Node* Copy(Node* root)
{
	if (root == nullptr)
		return nullptr;

	Node* newRoot = new Node(root->_key, root->_value);
	newRoot->_left = Copy(root->_left);
	newRoot->_right = Copy(root->_right);
	return newRoot;
}
```

`Copy` 先造当前结点，再递归造左子树和右子树，最后返回。这是**前序**的复制顺序：根先建好，才能把左右孩子的地址填进去

**赋值：拷贝并交换，不逐结点处理**

```cpp
BSTree<K, V>& operator=(BSTree<K, V> t)
{
	swap(_root, t._root);
	return *this;
}
```

形参 `t` 是按值传的，调用 `a = b` 时会先用 `b` 拷贝构造出一个 `t`（走上面那个拷贝构造）。接着 `swap(_root, t._root)` 把 `a` 原来的树换到 `t` 里，`t` 在函数结束时析构，顺手把 `a` 的旧树释放掉

这样做的好处是**不用自己处理「旧树怎么删」「自我赋值怎么办」**，两件事都交给 `t` 的析构和拷贝构造兜住了

**析构：必须后序**

```cpp
~BSTree()
{
	Destroy(_root);
	_root = nullptr;
}

void Destroy(Node* root)
{
	if (root == nullptr)
		return;

	Destroy(root->_left);
	Destroy(root->_right);
	delete root;
}
```

先释放左右子树，最后 `delete` 自己。顺序反了就是先 `delete` 当前结点，再访问它的 `_left`，读的是已回收的内存。这与 `Copy` 的前序正好相反，因为释放和创建的需求是反的

`_root = nullptr` 那一句在析构里没有实际作用，对象马上就销毁了。它真正的用处是被 `Destroy` 这类函数复用时防止悬空指针，写在这里是为了保持一致

### 8.4 替换法要连 value 一起换

替换法里，两个孩子的结点，替代者 `rightMin` 的值要换到 `cur` 上

如果只换 key：

```cpp
cur->_key = rightMin->_key;
```

`cur` 的 key 变成了 `rightMin` 的 key，但 `cur` 的 value 还是原来那个 key 对应的 value。key 和 value 从此对不上

在词频场景里，这个错误的表现是：某个单词的次数跑到了另一个单词头上。比如树里原本有 `("apple", 5)` 和 `("banana", 3)`，删掉某个结点时会变成 `("banana", 5)`，统计结果全错

正确做法是**key 和 value 一起换**：

```cpp
cur->_key = rightMin->_key;
cur->_value = rightMin->_value;
```

单 key 版完全看不出这个问题，因为它只有 `_key` 一个成员，换 key 就等于换了全部内容。这个问题只有在 key/value 版才会暴露

### 8.5 key/value 版其余接口

`Insert` 与 key 版几乎一样，只是多收一个 value，`new Node(key, value)`

`Erase` 与 key 版完全一样，按 key 删。除了 8.4 那一行要补上

```cpp
bool Insert(const K& key, const V& value)
{
	if (_root == nullptr)
	{
		_root = new Node(key, value);
		return true;
	}

	Node* parent = nullptr;
	Node* cur = _root;
	while (cur)
	{
		if (cur->_key < key)
		{
			parent = cur;
			cur = cur->_right;
		}
		else if (cur->_key > key)
		{
			parent = cur;
			cur = cur->_left;
		}
		else
		{
			return false;
		}
	}

	cur = new Node(key, value);
	if (parent->_key < key)
		parent->_right = cur;
	else
		parent->_left = cur;

	return true;
}
```

`Erase` 的替换法部分，把 5.5 里的

```cpp
cur->_key = rightMin->_key;
```

改成

```cpp
cur->_key = rightMin->_key;
cur->_value = rightMin->_value;
```

其余部分不用动

### 8.6 用测试验证一下

```cpp
int main()
{
	string arr[] = { "苹果", "西瓜", "苹果", "西瓜", "苹果", "苹果",
					 "西瓜", "苹果", "香蕉", "苹果", "香蕉" };
	key_value::BSTree<string, int> countTree;

	for (const auto& str : arr)
	{
		auto ret = countTree.Find(str);
		if (ret == nullptr)
		{
			countTree.Insert(str, 1);
		}
		else
		{
			ret->_value++;
		}
	}

	countTree.InOrder();

	key_value::BSTree<string, int> copy = countTree;
	copy.InOrder();

	return 0;
}
```

`InOrder` 需要为 key/value 版改写，输出 key 和 value 两个值：

```cpp
void _InOrder(Node* root)
{
	if (root == nullptr)
	{
		return;
	}

	_InOrder(root->_left);
	cout << root->_key << ":" << root->_value << endl;
	_InOrder(root->_right);
}
```

输出会是按 key 字典序排好的单词和各自的次数。`copy = countTree` 那一行验证拷贝构造：`copy` 打印出来的内容应该与 `countTree` 完全一样，且两者互不影响

### 8.7 拼成完整的类

前面几节给的是片段，这里按 6.2 的骨架凑成一个完整的类。类的外壳与 key 版同构，只是模板参数和几个接口的签名不同：

```cpp
namespace key_value
{
	template<class K, class V>
	class BSTree
	{
		using Node = BSTNode<K, V>;
	public:
		BSTree() = default;

		BSTree(const BSTree& t)
		{
			_root = Copy(t._root);
		}

		BSTree& operator=(BSTree tmp)
		{
			swap(_root, tmp._root);
			return *this;
		}

		~BSTree()
		{
			Destroy(_root);
			_root = nullptr;
		}

		bool Insert(const K& key, const V& value);
		Node* Find(const K& key);
		bool Erase(const K& key);

		void InOrder()
		{
			_InOrder(_root);
			cout << endl;
		}

	private:
		Node* Copy(Node* root);
		void Destroy(Node* root);
		void _InOrder(Node* root);
	private:
		Node* _root = nullptr;
	};
}
```

三个成员函数的实现分别见 8.5（`Insert`）、8.2（`Find`）、5.5 加上 8.4 那一行（`Erase`）。`Copy`、`Destroy`、`_InOrder` 就是 8.3 和 8.6 里那三段，作为类的私有成员写在 `private` 下面

`InOrder` 与 6.3 完全一样，公开接口不带参数、内部调 `_InOrder(_root)`

拼好的这份代码实测能跑通。`{苹果, 西瓜, 苹果, 西瓜, 苹果, 苹果, 西瓜, 苹果, 香蕉, 苹果, 香蕉}` 数出来是苹果 6 次、西瓜 3 次、香蕉 2 次；`copy = countTree` 之后两棵树打印的内容一致，说明拷贝是真的深拷贝

## 九、退化的代价

`1, 3, 4, 6, 7, 8, 10, 13, 14` 依次插入，每个新值都比前面所有值大，一路往右挂。最后长成一条斜链，高度等于结点个数

这时候查找、插入、删除全部退化成 O(N)，效率掉回了链表那一档

这不是二叉树这个结构本身的问题，而是「形状完全由插入顺序决定」这个设计带来的。数据如果本来就接近有序，树就接近退化

有两种做法：

**AVL 树**。在每个结点上记一个平衡因子，插入删除之后如果某个结点的左右子树高度差超过 1，就通过旋转把它压回来。旋转是对称的，四种情况分别处理

**红黑树**。用颜色标记和更宽松的平衡条件，旋转次数比 AVL 少，插入删除的整体开销更小。`map` 和 `set` 的底层用的就是红黑树

---

## 相关笔记 / 相关链接

- 相关目录：[[C++目录]]
- 相关笔记：[[数据结构/二叉树与堆]]（中序遍历与二叉树的递归结构是这篇的前置）、[[C++/类和对象全解：你想知道的这里都有，默认成员函数、友元、隐式类型转换、const和static修饰]]（拷贝构造、赋值、析构的通用规则在这篇，第 8.3 节直接承接）
