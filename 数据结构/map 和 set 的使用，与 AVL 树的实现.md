---
tags:
  - 形态/概念
aliases:
  - AVL树
日期: 2026年10月23日
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

统计一段文本里每个单词出现几次，最好的写法是 `map<string, int>`，循环里一句 `countMap[word]++`

这一行看着像在改一个已有的值，实际上它可能刚刚插入了一个新结点。`operator[]` 把查找、插入、修改三件事压在了一起，用起来省事，但是一不注意一些无关值就会被插入进去


## 一、 map 和 set 的概念

### 1.1 序列式容器与关联式容器

vector、list、deque 都是序列式容器。它们按元素在内存里的位置保存和访问，元素之间没有大小上的约定，交换两个位置也不破坏结构

map 和 set 是关联式容器。它们按关键字保存和访问，元素之间有紧密的大小关系，交换两个位置，整棵树的顺序就乱了

| | 序列式容器 | 关联式容器 |
| --- | --- | --- |
| 存的依据 | 存储位置 | 关键字 |
| 逻辑结构 | 线性序列 | 通常是非线性的树 |
| 典型容器 | vector、list、deque | set、map、multiset、multimap |
| 遍历顺序 | 插入的先后 | 按关键字排序 |

### 1.2 set 与 map 的分工

set 只存关键字，map 存的是键值对。两者都要求关键字支持小于比较，默认用 `less<Key>`；想按降序走，可以传 `greater<Key>` 进去；`greater` 是个仿函数，重载了 `operator()`

底层是红黑树（一种平衡二叉搜索树），所以中序遍历一遍就是有序的，增删查都是 O(log N)。set 和 map 不允许重复关键字，multiset 和 multimap 允许

## 二、 需要注意的接口

### 2.1 迭代器与 key

set 的迭代器不能改值，写 `*it = 1` 会直接报错，VS 下是 C3892，提示不能给常量赋值

```cpp
set<int> s = { 4, 2, 7, 9 };
auto it = s.begin();
// *it = 1;        // error C3892: "it": 不能给常量赋值
cout << *it << endl;
```

迭代器指向的是树里的关键字，改了它就破坏了整棵树的顺序，后面的查找全部作废。set 的 iterator 和 const_iterator 都按只读处理

map 稍微宽松一点。`it->first` 不能改，`it->second` 可以改，因为 value 不参与排序

### 2.2 insert 的返回值

set 的 `insert` 返回 `pair<iterator, bool>`。second 为 true 表示插入成功，false 表示这个关键字已经存在；first 永远指向那个关键字所在的结点，插入失败时也一样

```cpp
set<int> s = { 4, 2, 7, 9 };
auto ret = s.insert(7);
if (!ret.second)
    cout << "7 已经存在" << endl;
```

这个设计让 insert 同时成了查找：拿到返回值就知道在不在，还拿到了结点位置

map 的 insert 只看 key。key 已经存在时直接失败，哪怕 value 不一样也不会更新

```cpp
map<string, string> dict;
dict.insert({ "auto", "自动的" });
dict.insert({ "auto", "另一个意思" });   // 失败，value 不会改
```

### 2.3 erase 的三种重载

erase 有三种用法，返回值不一样

- 按位置删：`erase(pos)`，返回被删结点的下一个位置。删完 pos 就失效了，不能再解引用
- 按值删：`erase(val)`，返回删掉的个数。set 和 map 里只可能是 0 或 1；multiset 和 multimap 里是实际个数，值为 key 的结点会全删掉
- 按区间删：`erase(first, last)`，删掉左闭右开的一段

### 2.4 范围查找的边界

`lower_bound` 返回第一个大于等于 key 的位置，`upper_bound` 返回第一个大于 key 的位置。一个取等号，一个不取，配起来正好是左闭右开

```cpp
set<int> s;
for (int i = 1; i < 10; i++)
    s.insert(i * 10);          // 10 20 30 40 50 60 70 80 90

// 删掉 [30, 60] 这一段
auto itlow = s.lower_bound(30);   // >= 30
auto itup = s.upper_bound(60);    // > 60
s.erase(itlow, itup);
```

只想确认某个值在不在，用 `count` 也行，返回值非 0 就是在。这里要用成员函数 `find`；算法库的 `find` 是从头到尾顺序找，O(N)，成员函数走的是树，O(log N)

### 2.5 operator[] 的三种用途

map 的 `operator[]` 是最坑的接口。因为可以做到插入、查找、修改三件事

- key 不存在：插入一个 key 和默认构造的 value，返回 value 的引用
- key 存在：返回对应 value 的引用，可以直接改

它的常见实现就是拿 insert 垫的：

```cpp
mapped_type& operator[](const key_type& k)
{
    pair<iterator, bool> ret = insert({ k, mapped_type() });
    return ret.first->second;
}
```

insert 不管成功还是失败，first 都指向 key 所在的结点，所以这里不用判断 second，直接取 first 指向结点的 second 就行

坑在于：如果只是想查一下，写了 `dict[key]`，key 不在就会悄悄插进去一个默认值。查在不在应该用 `find` 或 `count`。另外 `const map` 不能用 `[]`，因为 `[]` 可能插入新结点，而 const 对象不允许修改

multimap 没有 `operator[]`。它允许 key 重复，`[]` 没法确定该返回哪一个 value；如果硬要提供，插入之后也说不清该改哪一个，所以干脆不提供

### 2.6 multiset 与 multimap 的差异

| | set / map | multiset / multimap |
| --- | --- | --- |
| 重复关键字 | 不允许 | 允许 |
| find 返回 | 唯一结点 | 某个等价结点，常见实现返回中序第一个 |
| count | 0 或 1 | 实际个数 |
| erase(val) | 删 0 或 1 个 | 删掉全部等于 key 的结点 |
| operator[] | map 有 | multimap 没有 |

### 2.7 总结一下

| 接口 | 坑 | 做法 |
| --- | --- | --- |
| `*it =` | set 的迭代器只读，编译报 C3892 | 要改就删了重插 |
| `insert` | 失败时不更新 value | 要改 value 用 `[]` 或迭代器 |
| `erase(val)` | multiset 里会删掉全部同值结点 | 只删一个用 `erase(find(val))` |
| `erase(pos)` | 删完 pos 失效 | 用返回值接下一个位置 |
| `lower_bound` / `upper_bound` | 一个取等一个不取 | 区间是 `[lower, upper)` |
| `operator[]` | 只查也会插入 | 查用 `find` / `count` |
| `find` | 算法库版本是 O(N) | 用成员函数 |

## 三、 从二叉搜索树到 AVL 树

### 3.1 退化

map 和 set 底层的红黑树，本质上是一棵会自己保持平衡的二叉搜索树。去掉平衡规则，剩下的就是普通的二叉搜索树

二叉搜索树（BST）的规矩很简单：左子树都比根小，右子树都比根大，左右子树也各是一棵二叉搜索树。查一个值，每比较一次就能排除一整棵子树

它的形状完全由插入顺序决定。同一组数据，先插中间的值，树就比较匀称，高度接近 log N；按从小到大插，每个新值都往右边挂，最后长成一条链

![[C++/attachments/BST完全二叉树与单支树.png|完全二叉树与单支树的对照]]

链的高度等于结点个数，查找、插入、删除全部退化成 O(N)。AVL 树要解决的就是这件事：让树的高度不随插入顺序失控

### 3.2 平衡因子

AVL 树给每个结点加一个平衡因子，记作 bf，定义是右子树高度减左子树高度

平衡的树里，bf 只能是 -1、0、1 三个值。等于 2 说明右边比左边高两层，等于 -2 反过来

为什么要求高度差不超过 1，而不是干脆要求两边一样高？2 个结点只能一左一右，高度差 1；4 个结点无论怎么摆，总有一边多一层，差 0 做不到。1 是能普遍做到的最严标准

bf 不是 AVL 定义里必须有的字段，是本实现为了 O(1) 判断失衡方向而存的。没有它也能靠递归算高度，只是每次插入都要多算一遍

![[数据结构/attachments/AVL平衡因子标注.png|每个结点上方标着平衡因子的 AVL 树]]

### 3.3 结点结构

结点比二叉搜索树多两个成员：一个指向父亲的指针，一个平衡因子

```cpp
template<class K, class V>
struct AVLTreeNode
{
	pair<K, V> _kv;
	AVLTreeNode* _left;
	AVLTreeNode* _right;
	AVLTreeNode* _parent;
	int _bf;

	AVLTreeNode(const pair<K, V>& kv)
		: _kv(kv)
		, _left(nullptr)
		, _right(nullptr)
		, _parent(nullptr)
		, _bf(0)
	{}
};
```

`_parent` 是必须的。插入一个新结点后，要沿着父亲一路往上更新 bf；旋转之后，还要把新的子树根接回原来的父亲。没有这个指针，往上走不了

## 四、 插入

### 4.1 BST 插入与 parent 指针

插入的前半段和普通二叉搜索树一样：两个指针往下走，parent 跟在 pcur 后面一步，找到空位置后把新结点挂上去

```cpp
bool Insert(const pair<K, V>& kv)
{
	if (_root == nullptr)
	{
		_root = new Node(kv);
		return true;
	}

	Node* parent = nullptr;
	Node* pcur = _root;
	while (pcur)
	{
		if (pcur->_kv.first < kv.first)
		{
			parent = pcur;
			pcur = pcur->_right;
		}
		else if (pcur->_kv.first > kv.first)
		{
			parent = pcur;
			pcur = pcur->_left;
		}
		else
		{
			return false;
		}
	}

	pcur = new Node(kv);
	if (parent->_kv.first < kv.first)
		parent->_right = pcur;
	else
		parent->_left = pcur;
	pcur->_parent = parent;
	// 后面接平衡因子的更新
}
```

最后一句 `pcur->_parent = parent` 把新结点的父亲补上。漏了的话，这个新结点就没有父亲；后面再往它下面插值，bf 更新到它这里就上不去了，旋转时也接不回原来的父亲

### 4.2 平衡因子更新

新结点插在叶子上，只会影响它到根这条路径上的结点，所以从父亲开始往上更新

规则只有一条：新结点在父亲的左边，父亲 bf 减一；在右边，加一。因为 bf 是右高减左高，左边变高了，差值就变小

更新到某个结点时，只有三种情况：

- bf 变成 0：插入前这个结点一边高一边低，新结点插在矮的那边，两边扯平了。子树高度没变，上面的结点不用动，停
- bf 变成 1 或 -1：插入前两边一样高，现在一边高了一层。子树高度增加了 1，会影响父亲，继续往上
- bf 变成 2 或 -2：插入前就一边高，新结点还插在高的那边，彻底失衡。旋转，转完这棵子树的高度回到插入前，上面的结点不受影响，停

第三种情况里“高度回到插入前”是关键，它决定了旋转之后不用再往上更新

![[数据结构/attachments/AVL平衡因子更新.png|插入 13 后 14 的平衡因子从 0 变成 2]]

### 4.3 旋转占位

先写出不含旋转的更新循环，碰到失衡就退出

```cpp
while (parent)
{
	if (pcur == parent->_left)
		parent->_bf--;
	else
		parent->_bf++;

	if (parent->_bf == 0)
	{
		break;
	}
	else if (parent->_bf == 1 || parent->_bf == -1)
	{
		pcur = parent;
		parent = parent->_parent;
	}
	else if (parent->_bf == 2 || parent->_bf == -2)
	{
		// 旋转先留空
		break;
	}
}
```

这样的配合插入可以通过编译，只是失去平衡会跳出循环

### 4.4 平衡检测的结果

要确认树是不是真的平衡，要有个检测函数。思路是递归算每个结点左右子树的高度，看高度差和结点上记的 bf 对不对得上

```cpp
int _Height(Node* root)
{
	if (root == nullptr)
		return 0;
	int leftHeight = _Height(root->_left);
	int rightHeight = _Height(root->_right);
	return leftHeight > rightHeight ? leftHeight + 1 : rightHeight + 1;
}

bool _IsBalanceTree(Node* root)
{
	if (root == nullptr)
		return true;

	int leftHeight = _Height(root->_left);
	int rightHeight = _Height(root->_right);
	int diff = rightHeight - leftHeight;

	if (abs(diff) >= 2)
	{
		cout << root->_kv.first << "高度差异常" << endl;
		return false;
	}
	if (root->_bf != diff)
	{
		cout << root->_kv.first << "平衡因子异常" << endl;
		return false;
	}
	return _IsBalanceTree(root->_left) && _IsBalanceTree(root->_right);
}
```

用 `{16, 3, 7, 11, 9, 26, 18, 14, 15}` 插进去跑一遍，输出是：

```text
16高度差异常
...
balance: 0
Height: 6
Size: 9
```

插到 7 的时候，16 的左右高度差已经到 2 了。九个结点插完，高度是 6；九个结点的平衡树高度应该是 4

如果在这里再加一个 `else assert(false)` 兜底，bf 涨到 3 时程序会直接断言失败。上面写的这版问题不只是“转不转”，是失衡之后没有处理，后面的插入还会继续把 bf 往上顶

可以理解成树能插进去，但形状已经歪了

## 五、 右单旋

### 5.1 触发条件

右单旋处理的是左左：parent 的 bf 是 -2，左边高；左孩子的 bf 是 -1，也是左边高。插入点落在左子树的左侧

旋转要同时做两件事：把子树转平衡，并且把它的高度降回插入前。只平衡不降高度的话，上面的结点还得继续调

### 5.2 抽象形态

把 10 为根的子树拎出来，5 是左孩子，a、b、c 是三棵高度为 h 的子树。插入前 10 的 bf 是 -1，插入发生在 a 里，a 的高度从 h 变成 h+1，10 的 bf 就成 -2

b 这棵子树的值都介于 5 和 10 之间，所以它能挂到 10 的左边；10 挂到 5 的右边；5 成为新的根

![[数据结构/attachments/AVL右单旋抽象.png|右单旋的抽象形态，a、b、c 是三棵高度为 h 的子树]]

中序顺序还是 a、5、b、10、c，搜索树的性质没被破坏。转完这棵树高度回到 h+2，和插入前一样

### 5.3 具体形态

h 等于 0 时，a、b、c 都是空树，b 是空，subLR 就是空指针。这时候改 subLR 的父亲必须先判空

![[数据结构/attachments/AVL右单旋h0.png|h 等于 0 时右单旋的具体形态]]

h 等于 1 时，b 里有一个结点，subLR 不为空，指针要一起改

![[数据结构/attachments/AVL右单旋h1.png|h 等于 1 时右单旋的具体形态]]

这两种情况代码是同一份，差别只在 subLR 可能是空。抽象图里的 a、b、c 代表了所有高度，落到具体形态只是换 h

### 5.4 代码与边界

```cpp
void RotateR(Node* parent)
{
	Node* subL = parent->_left;
	Node* subLR = subL->_right;

	parent->_left = subLR;
	if (subLR)
		subLR->_parent = parent;

	Node* pparent = parent->_parent;
	subL->_right = parent;
	parent->_parent = subL;

	if (pparent == nullptr)
	{
		_root = subL;
		subL->_parent = nullptr;
	}
	else
	{
		if (pparent->_left == parent)
			pparent->_left = subL;
		else
			pparent->_right = subL;
		subL->_parent = pparent;
	}
	parent->_bf = subL->_bf = 0;
}
```

指针一共三组：parent 的左孩子改成 subLR，subLR 的父亲改成 parent；subL 的右孩子改成 parent，parent 的父亲改成 subL；最后把 subL 接到 pparent 上

pparent 必须在改 parent 的父亲之前取。如果先写 `parent->_parent = subL`，再去取 `parent->_parent`，拿到的是 subL，原来的父亲就丢了

根结点要单独处理：pparent 为空时，subL 成为新的 `_root`，父亲置空；否则要判断 parent 是 pparent 的左还是右，把 subL 接到同一边

两个结点的 bf 都归零。右单旋之后，10 和 5 的左右高度都相等

拿 10、5、1 试一下：插入 10，插 5 挂左边，插 1 挂 5 左边，10 的 bf 变成 -2，触发右单旋。转完根是 5，中序是 1、5、10，`IsBalanceTree` 返回 true

## 六、 左单旋

### 6.1 镜像关系

左单旋处理右右：parent 的 bf 是 2，孩子的 bf 是 1。和右单旋完全镜像，把 subL 换成 subR、subLR 换成 subRL，左右方向反过来

![[数据结构/attachments/AVL左单旋抽象.png|左单旋的抽象形态]]

### 6.2 代码

```cpp
void RotateL(Node* parent)
{
	Node* subR = parent->_right;
	Node* subRL = subR->_left;

	parent->_right = subRL;
	if (subRL)
		subRL->_parent = parent;

	Node* pparent = parent->_parent;
	subR->_left = parent;
	parent->_parent = subR;

	if (pparent == nullptr)
	{
		_root = subR;
		subR->_parent = nullptr;
	}
	else
	{
		if (pparent->_left == parent)
			pparent->_left = subR;
		else
			pparent->_right = subR;
		subR->_parent = pparent;
	}
	parent->_bf = subR->_bf = 0;
}
```

和 RotateR 对照着看，只有方向不同，边界处理一模一样

## 七、 双旋

### 7.1 单旋失效的场景

左高不一定都是左左。还是 10 为根、5 为左孩子，这次插入发生在 5 的右子树 b 里：10 的 bf 是 -2，5 的 bf 却是 +1

这就是一条折线：10 左边高，5 右边高

直接对 10 右单旋，5 变成根，10 挂到 5 右边，可 5 的左边还是矮、右边是 10 这棵高树，5 的 bf 反而变成 +2，还是不平衡

单旋只能把一条直线拉直，折线拉一次会折到另一边

### 7.2 左右双旋

折线要分两步拉。先对 5 左旋，把折线转成直线，5 和 10 变成同向的左边高；再对 10 右旋，把整棵子树转平衡

第一次转的是左孩子 5，第二次转的是原来的根 10。顺序不能反

![[数据结构/attachments/AVL左右双旋h1.png|左右双旋 h 等于 1 时的具体形态]]

### 7.3 平衡因子的三种情况

双旋的 bf 修正比单旋麻烦。单旋之后两个结点都归零，双旋之后三个结点的新 bf 取决于插入位置

把 b 子树拆开看：它的根是 8（也就是 subLR），左边是 e，右边是 f，高度都是 h-1。新结点可能落在 e 里、f 里，或者 8 自己就是新结点

- 新结点在 e：8 的 bf 是 -1。转完 5 的 bf 是 0，8 是 0，10 是 1
- 新结点在 f：8 的 bf 是 1。转完 5 的 bf 是 -1，8 是 0，10 是 0
- b 原本是空树，8 就是刚插进来的结点：8 的 bf 是 0。转完三个都是 0

| subLR 的 bf | subL 的新 bf | subLR 的新 bf | parent 的新 bf |
| --- | --- | --- | --- |
| -1 | 0 | 0 | 1 |
| 1 | -1 | 0 | 0 |
| 0 | 0 | 0 | 0 |

关键是旋转前把 subLR 的 bf 存下来。RotateL 和 RotateR 会把沿途结点的 bf 改成 0，不先存，三分支就没法判断了

这也是双旋最容易写错的地方：单旋的代码直接调两次，bf 全被清零，最后树是平衡的，但每个结点的 bf 都对不上，下一次插入就会按错的方向转

### 7.4 代码

```cpp
void RotateLR(Node* parent)
{
	Node* subL = parent->_left;
	Node* subLR = subL->_right;
	int bf = subLR->_bf;

	RotateL(parent->_left);
	RotateR(parent);

	if (bf == -1)
	{
		subL->_bf = 0;
		subLR->_bf = 0;
		parent->_bf = 1;
	}
	else if (bf == 1)
	{
		subL->_bf = -1;
		subLR->_bf = 0;
		parent->_bf = 0;
	}
	else if (bf == 0)
	{
		subL->_bf = 0;
		subLR->_bf = 0;
		parent->_bf = 0;
	}
	else
	{
		assert(false);
	}
}
```

`int bf = subLR->_bf;` 这一句必须在两次旋转之前。两次旋转之后 subLR 的 bf 已经被清零，再读就晚了

### 7.5 右左双旋

右左双旋是左右双旋的镜像：parent 的 bf 是 2，右孩子的 bf 是 -1，插入点在右子树的左侧，形成一条反方向的折线

先对右孩子右旋，把折线拉直，再对 parent 左旋。bf 的三分支同样由 subRL 旋转前的值决定

| subRL 的 bf | subR 的新 bf | subRL 的新 bf | parent 的新 bf |
| --- | --- | --- | --- |
| -1 | 1 | 0 | 0 |
| 1 | 0 | 0 | -1 |
| 0 | 0 | 0 | 0 |

```cpp
void RotateRL(Node* parent)
{
	Node* subR = parent->_right;
	Node* subRL = subR->_left;
	int bf = subRL->_bf;

	RotateR(parent->_right);
	RotateL(parent);

	if (bf == 0)
	{
		subR->_bf = 0;
		subRL->_bf = 0;
		parent->_bf = 0;
	}
	else if (bf == 1)
	{
		subR->_bf = 0;
		subRL->_bf = 0;
		parent->_bf = -1;
	}
	else if (bf == -1)
	{
		subR->_bf = 1;
		subRL->_bf = 0;
		parent->_bf = 0;
	}
	else
	{
		assert(false);
	}
}
```

两个双旋的 bf 表看着不一样，其实是一回事：新结点落在内侧子树的哪一边，决定了三个结点谁高一层

## 八、 旋转接进插入

### 8.1 四种旋转的分派

把 parent 的 bf 和孩子 pcur 的 bf 合起来看，四种组合对应四种旋转

| parent 的 bf | pcur 的 bf | 形态 | 旋转 |
| --- | --- | --- | --- |
| -2 | -1 | 左左 | RotateR |
| -2 | 1 | 左右 | RotateLR |
| 2 | 1 | 右右 | RotateL |
| 2 | -1 | 右左 | RotateRL |

### 8.2 完整的 Insert

bf 变成 ±2 时，按 parent 和 pcur 的符号调用对应的旋转

```cpp
else if (parent->_bf == 2 || parent->_bf == -2)
{
	if (parent->_bf == -2 && pcur->_bf == -1)
		RotateR(parent);
	else if (parent->_bf == 2 && pcur->_bf == 1)
		RotateL(parent);
	else if (parent->_bf == -2 && pcur->_bf == 1)
		RotateLR(parent);
	else if (parent->_bf == 2 && pcur->_bf == -1)
		RotateRL(parent);
	else
		assert(false);

	break;
}
```

这里能直接用 pcur 的 bf 判断，是因为往上更新时 pcur 始终是 parent 下面那个结点，它的 bf 在上一步已经更新好了

旋转完直接 break，不用再往上

## 九、 完整实现

### 9.1 类骨架

公开接口保留 Insert、Find、InOrder、Height、Size、IsBalanceTree，四个旋转也放在公开区，方便单独调试验证，递归的部分放在私有

```cpp
template<class K, class V>
class AVLTree
{
	typedef AVLTreeNode<K, V> Node;
public:
	bool Insert(const pair<K, V>& kv);
	Node* Find(const K& key);
	void InOrder();
	int Height();
	int Size();
	bool IsBalanceTree();

	void RotateR(Node* parent);
	void RotateL(Node* parent);
	void RotateLR(Node* parent);
	void RotateRL(Node* parent);

private:
	void _InOrder(Node* root);
	int _Height(Node* root);
	int _Size(Node* root);
	bool _IsBalanceTree(Node* root);

private:
	Node* _root = nullptr;
};
```

### 9.2 完整代码

完整的类如下

```cpp
#pragma once
#include <iostream>
#include <cstdlib>
#include <cassert>
using namespace std;

template<class K, class V>
struct AVLTreeNode
{
	pair<K, V> _kv;
	AVLTreeNode* _left;
	AVLTreeNode* _right;
	AVLTreeNode* _parent;
	int _bf;

	AVLTreeNode(const pair<K, V>& kv)
		: _kv(kv)
		, _left(nullptr)
		, _right(nullptr)
		, _parent(nullptr)
		, _bf(0)
	{}
};

template<class K, class V>
class AVLTree
{
	typedef AVLTreeNode<K, V> Node;

public:
	bool Insert(const pair<K, V>& kv)
	{
		if (_root == nullptr)
		{
			_root = new Node(kv);
			return true;
		}

		Node* parent = nullptr;
		Node* pcur = _root;
		while (pcur)
		{
			if (pcur->_kv.first < kv.first)
			{
				parent = pcur;
				pcur = pcur->_right;
			}
			else if (pcur->_kv.first > kv.first)
			{
				parent = pcur;
				pcur = pcur->_left;
			}
			else
			{
				return false;
			}
		}

		pcur = new Node(kv);
		if (parent->_kv.first < kv.first)
			parent->_right = pcur;
		else
			parent->_left = pcur;
		pcur->_parent = parent;

		while (parent)
		{
			if (pcur == parent->_left)
				parent->_bf--;
			else
				parent->_bf++;

			if (parent->_bf == 0)
			{
				break;
			}
			else if (parent->_bf == 1 || parent->_bf == -1)
			{
				pcur = parent;
				parent = parent->_parent;
			}
			else if (parent->_bf == 2 || parent->_bf == -2)
			{
				if (parent->_bf == -2 && pcur->_bf == -1)
					RotateR(parent);
				else if (parent->_bf == 2 && pcur->_bf == 1)
					RotateL(parent);
				else if (parent->_bf == -2 && pcur->_bf == 1)
					RotateLR(parent);
				else if (parent->_bf == 2 && pcur->_bf == -1)
					RotateRL(parent);
				else
					assert(false);

				break;
			}
			else
			{
				assert(false);
			}
		}
		return true;
	}

	void RotateR(Node* parent)
	{
		Node* subL = parent->_left;
		Node* subLR = subL->_right;

		parent->_left = subLR;
		if (subLR)
			subLR->_parent = parent;

		Node* pparent = parent->_parent;
		subL->_right = parent;
		parent->_parent = subL;

		if (pparent == nullptr)
		{
			_root = subL;
			subL->_parent = nullptr;
		}
		else
		{
			if (pparent->_left == parent)
				pparent->_left = subL;
			else
				pparent->_right = subL;
			subL->_parent = pparent;
		}
		parent->_bf = subL->_bf = 0;
	}

	void RotateL(Node* parent)
	{
		Node* subR = parent->_right;
		Node* subRL = subR->_left;

		parent->_right = subRL;
		if (subRL)
			subRL->_parent = parent;

		Node* pparent = parent->_parent;
		subR->_left = parent;
		parent->_parent = subR;

		if (pparent == nullptr)
		{
			_root = subR;
			subR->_parent = nullptr;
		}
		else
		{
			if (pparent->_left == parent)
				pparent->_left = subR;
			else
				pparent->_right = subR;
			subR->_parent = pparent;
		}
		parent->_bf = subR->_bf = 0;
	}

	void RotateLR(Node* parent)
	{
		Node* subL = parent->_left;
		Node* subLR = subL->_right;
		int bf = subLR->_bf;

		RotateL(parent->_left);
		RotateR(parent);

		if (bf == -1)
		{
			subL->_bf = 0;
			subLR->_bf = 0;
			parent->_bf = 1;
		}
		else if (bf == 1)
		{
			subL->_bf = -1;
			subLR->_bf = 0;
			parent->_bf = 0;
		}
		else if (bf == 0)
		{
			subL->_bf = 0;
			subLR->_bf = 0;
			parent->_bf = 0;
		}
		else
		{
			assert(false);
		}
	}

	void RotateRL(Node* parent)
	{
		Node* subR = parent->_right;
		Node* subRL = subR->_left;
		int bf = subRL->_bf;

		RotateR(parent->_right);
		RotateL(parent);

		if (bf == 0)
		{
			subR->_bf = 0;
			subRL->_bf = 0;
			parent->_bf = 0;
		}
		else if (bf == 1)
		{
			subR->_bf = 0;
			subRL->_bf = 0;
			parent->_bf = -1;
		}
		else if (bf == -1)
		{
			subR->_bf = 1;
			subRL->_bf = 0;
			parent->_bf = 0;
		}
		else
		{
			assert(false);
		}
	}

	Node* Find(const K& key)
	{
		Node* cur = _root;
		while (cur)
		{
			if (cur->_kv.first < key)
				cur = cur->_right;
			else if (cur->_kv.first > key)
				cur = cur->_left;
			else
				return cur;
		}
		return nullptr;
	}

	void InOrder()
	{
		_InOrder(_root);
		cout << endl;
	}

	int Height()
	{
		return _Height(_root);
	}

	int Size()
	{
		return _Size(_root);
	}

	bool IsBalanceTree()
	{
		return _IsBalanceTree(_root);
	}

private:
	void _InOrder(Node* root)
	{
		if (root == nullptr)
			return;
		_InOrder(root->_left);
		cout << root->_kv.first << ":" << root->_kv.second << endl;
		_InOrder(root->_right);
	}

	int _Height(Node* root)
	{
		if (root == nullptr)
			return 0;
		int leftHeight = _Height(root->_left);
		int rightHeight = _Height(root->_right);
		return leftHeight > rightHeight ? leftHeight + 1 : rightHeight + 1;
	}

	int _Size(Node* root)
	{
		if (root == nullptr)
			return 0;
		return _Size(root->_left) + _Size(root->_right) + 1;
	}

	bool _IsBalanceTree(Node* root)
	{
		if (root == nullptr)
			return true;

		int leftHeight = _Height(root->_left);
		int rightHeight = _Height(root->_right);
		int diff = rightHeight - leftHeight;

		if (abs(diff) >= 2)
		{
			cout << root->_kv.first << "高度差异常" << endl;
			return false;
		}
		if (root->_bf != diff)
		{
			cout << root->_kv.first << "平衡因子异常" << endl;
			return false;
		}
		return _IsBalanceTree(root->_left) && _IsBalanceTree(root->_right);
	}

private:
	Node* _root = nullptr;
};
```

## 十、 验证与性能

### 10.1 平衡检测

前面 4.4 已经写过 `_Height` 和 `_IsBalanceTree`。两项都要查：高度差超过 1 不是 AVL 树；高度差没超、但结点记的 bf 和实际对不上，也不是，说明旋转时 bf 修错了

把两组用例插进去

```cpp
int a[] = { 16, 3, 7, 11, 9, 26, 18, 14, 15 };
int b[] = { 4, 2, 6, 1, 3, 5, 15, 7, 16, 14 };
```

同样的 `{16, 3, 7, 11, 9, 26, 18, 14, 15}`，4.4 里旋转还没接上，返回 0；现在返回 1。第二组是专门用来覆盖双旋的，也返回 1

### 10.2 性能

验证只能说明这棵树现在是对的，看不出它比普通二叉搜索树强多少。跑一组数据对比

测试代码如下

```cpp
#include <vector>
#include <algorithm>
#include <random>
#include <ctime>

const int N = 1000000;
vector<int> v;
v.reserve(N);
for (int i = 1; i <= N; i++)
	v.push_back(i);
shuffle(v.begin(), v.end(), mt19937(12345));

size_t begin = clock();
AVLTree<int, int> t;
for (auto e : v)
	t.Insert({ e, e });
size_t end = clock();

cout << "Insert: " << end - begin << " ms" << endl;
cout << "Height: " << t.Height() << endl;
cout << "Size: " << t.Size() << endl;
```

这里用 `shuffle` 把 1 到 100 万打乱，保证没有重复 key。如果直接用 `rand()` 取值，重复的 key 会被 insert 拒掉，`Size` 会小于 N；而且 Windows 上 `RAND_MAX` 只有 32767，根本覆盖不到 100 万

`clock()` 在 Windows 下 1 个计数约 1 ms，这段按毫秒解释；换平台要按 `CLOCKS_PER_SEC` 换算

本机 VS 2022，x64，开 `/O2` 优化跑出来的结果

| 数据 | 结构 | 高度 | 插入耗时 |
| --- | --- | --- | --- |
| 100 万随机 | AVL | 24 | 约 0.3 s |
| 100 万有序 | AVL | 20 | 约 55 ms |
| 1 万随机 | AVL | 16 | 约 1 ms |
| 1 万有序 | AVL | 14 | 约 1 ms |
| 1 万随机 | 普通 BST | 29 | — |
| 1 万有序 | 普通 BST | 10000 | — |

连跑五次，100 万随机插入在 294 到 360 ms 之间，100 万次查找在 266 到 287 ms 之间，高度稳定在 24

- 100 万个结点，AVL 高度 24。log₂(1000000) 约 20，AVL 允许的高度上限大约是 1.44 log₂N，24 在范围内
- 有序数据插进 AVL，高度 20，正好是一棵接近满的二叉树。普通 BST 遇到同样的数据，1 万条时高度就是 10000，完全退化成链。AVL 的形状不受插入顺序影响，这是它和 BST 最本质的区别
- 1 万随机数据，AVL 高度 16，普通 BST 29。随机顺序下 BST 不会退化成链，但高度还是比 AVL 高
- 随机插入比有序插入慢，100 万条差了五倍多。更像是访存：有序插入每次都走最右边那条路，结点刚分配、缓存还热；随机插入要在整棵树里跳，cache miss 多
- 100 万次查找约 0.28 s，平均一次 280 ns 左右，和高度 24 对得上。每次查找走一条从根到叶的路径，路径长度就是高度

## 十一、 map/set 与 AVL 的关系

map 和 set 的底层是红黑树，不是 AVL 树。两者都是平衡二叉搜索树，区别在平衡的严格程度

- AVL 要求左右高度差不超过 1，树更矮，查找更快；插入删除要维护这个严格条件，旋转次数更多
- 红黑树用颜色约束，最长路径不超过最短路径的两倍，平衡条件松一些，旋转少，插入删除整体开销小

map 和 set 插入删除也频繁，选了红黑树。AVL 更适合查找远多于插入删除的场景

AVL 的删除这里不展开说。思路和插入一样，删完沿 parent 往上更新 bf，失衡就旋转，只是要处理的情况比插入多，双旋的分支也更细

---

## 相关笔记 / 相关链接

- 相关目录：[[数据结构目录]]
- 相关笔记：[[C++/二叉搜索树——从插入查到key-value改写]]（前置：BST 的插入、中序遍历和退化是第三章的基础）

---
