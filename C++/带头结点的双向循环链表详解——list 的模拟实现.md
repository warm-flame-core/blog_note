---
tags:
  - 编程
aliases:
日期:
---


<div align="center"> <img src="https://i-blog.csdnimg.cn/direct/78623ff9c20f4ee58344321c4a2ac0f7.jpeg" width="400" alt="头像"/> </div>

<p align="center">
  <font color="#FF69B4"><b>Ciallo(∠・ω< )⌒☆</b></font>
</p>

<p align="center">
  <font color="#00FFFF"><b>C语言专栏</b></font><br>
  <a href="https://blog.csdn.net/hahaha6565/category_13108044.html">C语言博客_CSDN</a>
</p>
<p align="center">
  <font color="#00FFFF"><b>数据结构专栏</b></font><br>
  <a href="https://blog.csdn.net/hahaha6565/category_13156520.html">数据结构博客_CSDN</a>
</p>
<p align="center">
  <font color="#00FFFF"><b>C++专栏</b></font><br>
  <a href="[C++_暖焰核心的博客-CSDN博客](https://blog.csdn.net/hahaha6565/category_13163238.html?spm=1001.2014.3001.5482)">C++博客_CSDN</a>
</p>
<p align="center">
  <font color="#00FFFF"><b>Python专栏</b></font><br>
  <a href="https://blog.csdn.net/hahaha6565/category_13194466.html">Python博客_CSDN</a>
</p>
<p align="center">
  <font color="#00FFFF"><b>作者仓库</b></font><br>
  <a href="https://gitee.com/warm-flame-core">代码仓库_gitee</a>
</p>


# 一、引言
上一篇我们模拟实现了 vector，它底层是一段连续空间，随机访问很高效，但任意位置插入删除需要搬移元素，效率不高。C 语言阶段的链表虽然能很好地解决插入删除的问题，但接口太少、用起来麻烦，而 STL 的 `list` 正是基于链表封装出来、接口完善的容器。这一篇我们就来探究 list 的实现思路——它是怎么把「带头结点的双向循环链表」这个底层结构包装成一个个好用接口的。

# 二、list 的介绍与使用
## 2.1 list 的底层结构
`list` 底层是**带头结点的双向循环链表**。「带头结点」指链表的第一个节点是哨兵位，不存有效数据，只为统一头插、尾插、中间插的边界处理；「循环」指头尾相接。这样无论头插、尾插、还是中间插，定位和串接逻辑都完全一致，不用单独为「在头结点前插入」写特判。

## 2.2 list 的构造
```cpp
list<int> l1;                            // 空 list
list<int> l2(10, 1);                     // 10 个值为 1 的元素
list<int> l3(l2);                        // 拷贝构造
list<int> l4(l2.begin(), l2.end());      // 迭代器区间构造
```

## 2.3 list 的迭代器
可以把 list 的迭代器先理解成「一个指向节点的指针」。正向迭代器 `begin`/`end` 执行 `++` 向后移动；反向迭代器 `rbegin`/`rend` 执行 `++` 向前移动（因为它内部其实就是正向迭代器的 `--`）。

```cpp
list<int> lt;
for (auto e : lt)
{
    cout << e << ' ';
}
```

## 2.4 list 的容量与元素访问
`empty` 判断是否为空，`size` 返回有效节点个数，`front`/`back` 返回首尾节点值的引用。

## 2.5 list 的增删查改
`push_back`/`pop_back`、`push_front`/`pop_front`、`insert`/`erase`、`swap`、`clear` 都是常用接口。这里额外提一下 `emplace_back`，C++11 引入，和 `push_back` 都能尾插，区别是 `emplace_back` 会**直接把构造参数传进去、就地构造**，省掉一次临时对象拷贝，对构造复杂的自定义类型效率更高。

```cpp
struct A
{
    int _a1, _a2;
    A(int a1 = 1, int a2 = 1) : _a1(a1), _a2(a2) {}
};

list<A> lt;
A aa(1, 1);
lt.push_back(aa);       // 需要临时对象
lt.push_back(A(2, 2));  // 需要临时对象
lt.emplace_back(3, 3);  // 直接传构造参数，就地构造
```

## 2.6 list 的迭代器失效
vector 因为扩容会让迭代器全部失效，但 list 底层是节点，插入节点不会改变已有节点的地址，所以**插入元素不会导致迭代器失效**。真正会失效的只有删除：被删的那个节点对应的迭代器失效了，其他迭代器不受影响。

```cpp
// 错误写法：erase 之后 it 指向的节点已被删除，it 失效
while (it != l.end())
{
    l.erase(it);
    ++it;
}

// 正确写法：接收 erase 返回值 / 先自增再删除
it = l.erase(it);   // 返回被删节点的下一个
// 或 l.erase(it++);
```

# 三、list 的模拟实现
list的部分实现还是比较简单的，我们学了模板，所以迭代器我们需要细嗦
## 3.1 节点的定义
双向循环链表每个节点要存三个东西：数据、前驱指针、后继指针。

```cpp
template<class T>
struct list_Node
{
    list_Node(const T date = T())
    {
        _date = date;
        _next = _prev = nullptr;
    }
    T _date;
    list_Node<T>* _next;
    list_Node<T>* _prev;
};
```

## 3.2 迭代器的设计——把一个指针封装成一个类
### 为什么不能直接用指针
vector 的迭代器就是裸指针 `T*`，因为数据连续存放，`it++` 就是指针加 1，天然成立。但 list 的数据是一个个节点、彼此不连续，`it++` 根本不知道往哪儿走——它要跳到 `_next` 节点。所以 list 的迭代器**不能是裸指针**，必须把它写成一个类，让 `++`、`--`、`*`、`->` 这些操作符在我们定义的行为下工作。

### `->` 是最容易懵的
先写出一个「正常」的迭代器，`operator*` 简单，返回当前节点的数据引用；可 `operator->` 却让我卡了很久。当时我盯着 `->` 想：它明明和 `*` 一样是「取当前节点的东西」，为什么 `operator*` 返回 `T&`，而 `operator->` 却要返回 `T*`（地址）？

关键就在于 `->` 在 C++ 里是个**特殊**的操作符。你写 `it->data`，编译器并不是直接调用 `operator->` 拿到 data，而是把它拆解成：

```text
it->data  ==>  (it.operator->())->data
```

也就是说，`operator->()` 返回的是**一个指针**，然后编译器拿着这个指针，**再对它做一次 `->`**。所以要想让 `it->data` 拿到节点的数据，`operator->` 必须返回「数据的地址」，这样：

```text
it->data
  => (it.operator->())->data
  => (&_node->_data)->data   // 即 (*it).data
```

这下就通了。`->` 的重载不是「直接返回数据」，而是「返回数据的地址，交给编译器再解引用一次」。这就是它和 `*` 最大的不同。

```cpp
template<class T>
struct list_iterator
{
    typedef list_Node<T> Node;
    Node* _node;

    list_iterator(Node* node) : _node(node) {}

    T& operator*() { return _node->_date; }
    T* operator->() { return &(_node->_date); }

    list_iterator<T>& operator++() { _node = _node->_next; return *this; }
    list_iterator<T>& operator--() { _node = _node->_prev; return *this; }

    bool operator!=(const list_iterator<T>& s) const { return _node != s._node; }
    bool operator==(const list_iterator<T>& s) const { return _node == s._node; }
};
```

### const 的迭代器还得再写一份
`const list<int>` 这种对象只允许读，所以要一个 `operator*` 返回 `const T&`、`operator->` 返回 `const T*` 的迭代器。于是我又复制了一份，改成 const。两个类摆在一起，你会发现除了三处类型，其余全是重复的。

```cpp
template<class T>
struct list_const_iterator
{
    typedef list_Node<T> Node;
    const Node* _node;

    list_const_iterator(const Node* node) : _node(node) {}

    const T& operator*() { return _node->_date; }
    const T* operator->() { return &(_node->_date); }

    list_const_iterator<T>& operator++() { _node = _node->_next; return *this; }
    list_const_iterator<T>& operator--() { _node = _node->_prev; return *this; }

    bool operator!=(const list_const_iterator<T>& s) const { return _node != s._node; }
    bool operator==(const list_const_iterator<T>& s) const { return _node == s._node; }
};
```

### 给自己起个别名 `self`
写到这份上，我发现自己每次都要写一长串 `list_iterator<T>`，尤其返回值、参数、比较，到处都是。一旦后面模板参数变多，写全名更容易写错。于是我就用 `typedef` 给「自己」起个别名 `self`，这样 `operator++` 返回 `self&`、`operator!=` 参数写 `const self&`，简洁多了。

```cpp
typedef list_iterator<T> self;
```

### 既然只差三处，交给模板合并
普通和 const 两个类就差三处：指针是 `Node*` 还是 `const Node*`、`operator*` 返回 `T&` 还是 `const T&`、`operator->` 返回 `T*` 还是 `const T*`。既然「会变的就这几处」，那就把它们抽成模板参数 `Ref` 和 `Ptr`，用 `template<class T, class Ref, class Ptr>` 把两个类合并成一个。`Ref` 管 `operator*` 返回什么、`Ptr` 管 `operator->` 返回什么。普通迭代器实例化成 `list_iterator<T, T&, T*>`，const 迭代器实例化成 `list_iterator<T, const T&, const T*>`。

```cpp
template<class T, class Ref, class Ptr>
struct list_iterator
{
    typedef list_Node<T> Node;
    typedef list_iterator<T, Ref, Ptr> self;
    Node* _node;

    list_iterator(Node* node) : _node(node) {}

    Ref operator*() { return _node->_date; }
    Ptr operator->() { return &_node->_date; }

    self& operator++() { _node = _node->_next; return *this; }
    self& operator--() { _node = _node->_prev; return *this; }

    self operator++(int) { self tmp = *this; _node = _node->_next; return tmp; }
    self operator--(int) { self tmp = *this; _node = _node->_prev; return tmp; }

    bool operator!=(const self& s) const { return _node != s._node; }
    bool operator==(const self& s) const { return _node == s._node; }
};
```

`list` 类里就只要两句 `typedef` 了：
```cpp
typedef list_iterator<T, T&, T*> iterator;
typedef list_iterator<T, const T&, const T*> const_iterator;
```

> 一个巧思：这里 `const` 不是靠 `const Node*` 这种「指针本身只读」来体现的，而是通过**返回值**——`Ref` 是 `const T&`、`Ptr` 是 `const T*`。这样拿到的是「只读」的引用/指针，能改对象的操作自然被拦住了。

### 普通迭代器怎么转成 const 迭代器
平时我们常写 `const list<int>&` 的 `begin()` 返回 `const_iterator`。那想用一个已有的普通 `iterator` 去初始化 `const_iterator`，能不能转？能。**权限只能缩小（普通→const），不能放大（const→普通）**，这才安全。所以给迭代器加一个「模板构造函数」，用普通迭代器构造 const 迭代器，只把 `_node` 拷过来。

```cpp
template<class Ref2, class Ptr2>
list_iterator(const list_iterator<T, Ref2, Ptr2>& lt)
    : _node(lt._node)
{}
```

> 你会发现：一个模板参数 `T` 的类里，还能再嵌套一个「函数模板」，这种「类模板的成员函数依旧是模板」的写法，和 vector 里用「迭代器区间构造」是一个道理，模板参数可以层层叠加。

### `begin()` 的两种返回写法
写 `begin()` 的时候，会发现返回 `_head->_next` 这个 `Node*` 其实可以有两种写法，都能过编译，但背后逻辑值得想一想。

```cpp
iterator begin()
{
    return iterator(_head->_next);   // 方式一：先构造一个临时迭代器对象，再返回
}

iterator begin()
{
    return _head->_next;             // 方式二：直接用 Node* 隐式类型转换返回
}
```

方式一是「显式构造临时对象再返回」，写得很明确；方式二是靠迭代器的构造函数做**隐式类型转换**，更简洁。两者结果一样，方式二更常用。`const` 版本同理：

```cpp
const_iterator begin() const
{
    return _head->_next;
}
```

## 3.3 list 框架与哨兵位
`list` 持有两个成员：`_head`（哨兵位）和 `_size`（节点数）。初始化时先 `empty_init` 把哨兵位弄成自循环的空链表。

```cpp
void empty_init()
{
    _head = new Node;
    _head->_next = _head;
    _head->_prev = _head;
    _size = 0;
}
```

> 一个值得记住的点：**`end()` 返回的正是哨兵位 `_head`**。这样 `begin` 到 `end` 遍历正好覆盖所有有效节点一次，而且「插入到 `end()` 就是尾插」逻辑统一，头插尾插中间插全是一样的代码。

## 3.4 insert 与 erase
中间的插入删除是双向链表的常规操作，思路和 C 语言链表一致：插入时新建节点，把当前节点和它的前驱串起来；删除时先接好前后两段再 `delete` 掉当前节点。这里只强调一点——插入返回新节点的迭代器，删除返回被删节点的下一个迭代器。

```cpp
iterator insert(iterator pos, const T& x)
{
    Node* pcur = pos._node;
    Node* prev = pcur->_prev;
    Node* newnode = new Node(x);
    newnode->_next = pcur;
    newnode->_prev = prev;
    pcur->_prev = newnode;
    prev->_next = newnode;
    ++_size;
    return newnode;
}

iterator erase(iterator pos)
{
    assert(pos != end());
    Node* pcur = pos._node->_next;
    Node* prev = pos._node->_prev;
    pcur->_prev = prev;
    prev->_next = pcur;
    delete pos._node;
    --_size;
    return pcur;
}
```

有了 `insert`/`erase`，`push_back`、`push_front`、`pop_back`、`pop_front`、`clear` 都能直接复用，很简洁。

```cpp
void push_back(const T& x) { insert(end(), x); }
void push_front(const T& x) { insert(begin(), x); }
void pop_back() { erase(--end()); }
void pop_front() { erase(begin()); }

void clear()
{
    auto it = begin();
    while (it != end())
    {
        it = erase(it);
    }
}
```

## 3.5 拷贝构造、赋值重载、析构
拷贝构造复用尾插一件件搬进来即可。赋值重载用「现代写法」，参数不引用，传进来的是临时对象，交换后临时对象出作用域自动销毁。

```cpp
list(const list<T>& lt)
{
    empty_init();
    for (auto e : lt)
    {
        push_back(e);
    }
}

list<T>& operator=(list<T> lt)
{
    swap(lt);
    return *this;
}

void swap(list<T>& lt)
{
    std::swap(_head, lt._head);
    std::swap(_size, lt._size);
}

~list()
{
    clear();
    delete _head;
    _head = nullptr;
}
```

# 四、迭代器的分类
写到这里，我想专门把迭代器的**类型**展开聊一聊。因为基础阶段容易把它当成「一个统一的指针」，可实际上迭代器是有强弱之分、有层级关系的，而且**有些接受迭代器的库函数并不支持所有迭代器**，不了解这一点，很容易用错。

迭代器按能力分为以下几类（从弱到强）：

```text
Input Iterator          输入迭代器    只读，只能向前（单遍扫描），比如 istream_iterator
Output Iterator         输出迭代器    只写，只能向前（单遍扫描），比如 ostream_iterator
Forward Iterator        前向迭代器    可读可写，只能向前，可多遍扫描，比如 forward_list
Bidirectional Iterator  双向迭代器    可读可写，既能++也能--，比如 list、set、map
Random Access Iterator  随机访问迭代器 可读可写，支持++/--/+n/-n/下标，比如 vector、deque、string
Contiguous Iterator      连续迭代器    在随机访问基础上，保证内存连续，比如 vector、string（C++17 起）
```

它们之间有明显的继承关系：能力越强，能覆盖的操作越多；越往下，越能当作上面一层的类型使用。可以用一张图来表示：

```text
               Contiguous Iterator
                        |
               Random Access Iterator
                        |
               Bidirectional Iterator
                        |
                Forward Iterator
                       / \
        Input Iterator   Output Iterator
```

也就是说，**Contiguous 属于 Random Access，Random Access 属于 Bidirectional，Bidirectional 属于 Forward**。而 list 的迭代器是 **Bidirectional Iterator**，不是 Random Access。

**这点很关键**。标准库的 `std::sort` 需要随机访问迭代器（因为它要能直接跳位置做分区），所以 `std::sort(lt.begin(), lt.end())` 对 list 是编译不过的。list 只能用自己的成员函数 `lt.sort()` 来排序。反过来，`std::reverse`、`std::find` 这些只要求双向或前向的算法，list 就能直接用。

> 判断一个算法能不能用，关键看它的迭代器「要求等级」是否 ≤ 你容器迭代器的「等级」。list 是双向迭代器，凡是要随机访问迭代器的算法它都用不了，只能用自己类里提供的成员版本。这也是前面为什么 vector 能直接 `std::sort`，而 list 得 `lt.sort()` 的根本原因。

# 五、list 的反向迭代器
反向迭代器没必要重写一遍。它的 `++` 就是正向迭代器的 `--`，`--` 就是正向迭代器的 `++`，所以反向迭代器内部**持有一个正向迭代器**，把接口包装一下就行。

于是我们写出这样一个类，模板参数 `Iterator` 就是正向迭代器：

```cpp
template<class Iterator>
class ReverseListIterator
{
public:
    typedef typename Iterator::Ref Ref;
    typedef typename Iterator::Ptr Ptr;
    typedef ReverseListIterator<Iterator> Self;
    ...
};
```

有个细节要注意：`operator*` 要解引用「当前的前一个位置」，得先拷贝一份再前移，不能改动自身的 `_it`。

```cpp
Ref operator*()
{
    Iterator temp(_it);
    --temp;
    return *temp;
}
```

还有 `typename` 这个关键字，它是用来告诉编译器 `Ref` 是 `Iterator` 类里的**类型**，而不是静态成员变量——因为模板还没实例化，编译器分不清 `Iterator::Ref` 到底是类型还是变量，`typename` 就是帮它排雷的。这一点和前面 vector 打印函数里 `typename vector<T>::const_iterator it` 是一个道理。正因为 vector 和 list 都能复用这个模板，所以通常会把反向迭代器抽成一个通用的 `reverse_iterator`。

# 六、list 与 vector 的对比
vector 和 list 都是 STL 重要的序列式容器，但底层结构不同，特性也不同，整理成表：

| 对比项 | vector | list |
| --- | --- | --- |
| 底层结构 | 动态顺序表，一段连续空间 | 带头结点的双向循环链表 |
| 随机访问 | 支持，访问某元素 O(1) | 不支持，访问某元素 O(N) |
| 插入和删除 | 任意位置插入删除效率低，要搬移元素，O(N)；插入可能扩容更慢 | 任意位置插入删除效率高，不用搬移，O(1) |
| 空间利用率 | 连续空间，不易碎片，缓存利用率高 | 节点动态开辟，小节点易碎片，缓存利用率低 |
| 迭代器 | 原生态指针 | 对节点指针进行封装 |
| 迭代器失效 | 插入可能扩容使全部失效；删除当前迭代器要重新赋值 | 插入不失效；删除只使被删节点迭代器失效 |
| 使用场景 | 需要高效存储、支持随机访问、不关心插删效率 | 大量插入删除、不关心随机访问 |

**一个实际体感**：list 节点分散、缓存命中率低，所以即便插入删除是 O(1)，真实跑起来未必比 vector 快。测试里把上百万个随机数分别用 `list.sort()` 和「拷贝到 vector 排序再拷回」比较，往往是 vector 那条路更快，这就是缓存的威力。所以选容器时，要看核心操作是「随机访问」还是「大量插入删除」。

# 七、总结
list 的核心是带头结点的双向循环链表，它把「节点如何串联、迭代器如何封装」隐藏起来，暴露出一套统一好用的接口。这一篇我们重点看了模拟实现，尤其是迭代器怎么把裸指针封装成类、普通/const 迭代器怎么用一个类模板合并、以及迭代器分类和 const 权限转换。比起 vector，它牺牲了随机访问和缓存效率，换来了任意位置插入删除的高效。而且通过迭代器一步步写过来的过程，也更能体会到把裸指针封装成类、再用模板合并这层设计思想的巧妙。

---

## 相关笔记 / 相关链接

- 相关目录：[[C++目录]]
- 相关笔记：[[C++/类和对象全解：你想知道的这里都有，默认成员函数、友元、隐式类型转换、const和static修饰]]（前置：迭代器封装要运算符重载）、[[C++/迭代器失效探究之vector详解——vector对于野指针的检查]]（类比：连续存储与链式存储的接口对照）
