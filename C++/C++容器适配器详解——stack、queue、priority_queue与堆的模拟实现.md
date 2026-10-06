---
tags:
  - 编程
  - C++
  - 数据结构
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
  <font color="#00FFFF"><b>作者仓库</b></font><br>
  <a href="https://gitee.com/warm-flame-core">代码仓库_gitee</a>
</p>


# 引言

上一篇我们探究了 `list` 的底层结构和模拟实现。`vector` 和 `list` 都是可以直接使用的容器，它们负责保存数据，也提供了比较完整的操作接口。这一篇要学习的 `stack`、`queue` 和 `priority_queue` 却有一点不一样：它们更像是套在其他容器外面的一层接口。

`stack` 只允许我们访问栈顶，`queue` 按照先进先出的规则访问队头和队尾，`priority_queue` 则保证每次都能取出优先级最高的元素。它们并不需要把底层数据结构重新写一遍，而是复用已经存在的 `deque`、`vector` 或 `list`，只开放自己需要的那部分操作。这种设计叫作**容器适配器（Container Adapter）**。

这篇先介绍三个容器适配器的使用，再观察 STL 源码中适配器的设计方式。之后我会从一个只能比较整数的堆开始模拟实现，遇到“大堆和小堆需要反复改代码”的问题后，再引入仿函数，把比较规则单独封装起来。最后重点分析堆的向上调整和向下调整，并结合 `Date` 对象和 `Date*` 指针说明自定义比较器的使用。

# 一、容器适配器

## 1.1 什么是适配器

适配器的作用，可以理解成把一个已有接口转换成另一个更适合使用的接口。生活中比较常见的例子是电源转换插头：墙上的插座和设备的插头形状可能不同，中间加一个转换头，就能让两边连接起来。

容器适配器也是类似的思路。底层容器已经负责保存数据，适配器不再重新设计一套存储结构，而是根据自己的规则调用底层容器的接口。

```text
+----------------------+
| stack / queue / pq   |
+----------+-----------+
           |
           v
+----------------------+
| container adapter    |
+----------+-----------+
           |
           v
+----------------------+
| deque / vector / list|
+----------------------+
```

注意：`stack`、`queue` 和 `priority_queue` 对外提供的接口不同，但它们都可以把数据保存工作交给底层容器。

## 1.2 适配器的两个特点

第一，适配器会限制底层容器的接口。`deque` 本身可以在两端插入和删除，也有迭代器和随机访问能力，但经过 `stack` 封装后，使用者只能通过 `push`、`pop`、`top` 等接口操作，不能直接遍历 `stack` 的内部数据。

第二，适配器依赖底层容器提供相应的接口。例如，`stack` 需要底层容器支持 `back`、`push_back` 和 `pop_back`；`queue` 需要支持 `front`、`back`、`push_back` 和 `pop_front`。如果底层容器没有这些接口，就不能拿来作为对应适配器的容器。

这也解释了为什么 `stack`、`queue` 没有迭代器：它们的设计目的就是限制访问范围，而不是把底层容器的所有能力暴露出来。

# 二、stack 的介绍与使用

## 2.1 stack 的特点

`stack` 遵循后进先出（LIFO，Last In First Out）规则。最后放进去的元素，最先被取出来。

```text
        push(x)                 pop()
          |                      |
          v                      v
       +------+               +------+
       |  top |               |  top |
       +------+               +------+
       |  ... |
       +------+
       |  ... |
       +------+
```

注意：上图中的两个操作都发生在 `top` 一侧，这正是后进先出的限制。

`stack` 默认使用 `deque<T>` 作为底层容器，也可以根据模板参数换成其他满足要求的容器。

## 2.2 stack 的常用接口

| 函数 | 作用 | 注意事项 |
| --- | --- | --- |
| `stack()` | 构造空栈 | 默认底层容器是 `deque` |
| `empty()` | 判断栈是否为空 | 返回 `true` 或 `false` |
| `size()` | 返回元素个数 | 统计当前有效元素 |
| `top()` | 返回栈顶元素的引用 | 空栈调用会产生未定义行为 |
| `push(x)` | 将元素压入栈顶 | 新元素成为新的栈顶 |
| `pop()` | 删除栈顶元素 | 只删除，不返回被删元素 |

这里的 `push`、`pop`、`empty` 和 `size`，前面在 `vector`、`list` 中也见过类似用法。不过适配器只保留了符合栈规则的接口。

```cpp
#include <iostream>
#include <stack>
using namespace std;

int main()
{
    stack<int> st;
    st.push(1);
    st.push(2);
    st.push(3);
    st.push(4);

    cout << st.top() << endl;    // 4
    st.pop();                    // 只删除, 不返回
    cout << st.top() << endl;    // 3

    return 0;
}
```

注意：`pop()` 不会把被删除的元素返回出来。如果既要读取栈顶，又要删除栈顶，需要先调用 `top()`，再调用 `pop()`。

```cpp
cout << st.top() << endl;
st.pop();
```

`stack` 没有 `front()` 和 `back()`，也没有迭代器。即使它的底层是 `deque`，也不能因此绕过 `stack` 直接访问内部数据。

# 三、queue 的介绍与使用

## 3.1 queue 的特点

`queue` 遵循先进先出（FIFO，First In First Out）规则。新元素从队尾进入，元素从队头离开。

```text
pop                                      push
 |                                        |
 v                                        v
+------+------+------+------+------+------+
| a1   | a2   | a3   | ...  | an   | new  |
+------+------+------+------+------+------+
 ^                                      ^
 |                                      |
```

注意：左侧的 `a1` 是队头，右侧的 `new` 是队尾新进入的元素。元素从队尾进入，从队头离开。

`queue` 默认使用 `deque<T>` 作为底层容器，也可以使用 `list<T>`。因为队列需要从一端插入、从另一端删除，所以底层容器必须支持 `push_back()` 和 `pop_front()`。

## 3.2 queue 的常用接口

| 函数 | 作用 | 注意事项 |
| --- | --- | --- |
| `queue()` | 构造空队列 | 默认底层容器是 `deque` |
| `empty()` | 判断队列是否为空 | 返回 `true` 或 `false` |
| `size()` | 返回元素个数 | 统计当前有效元素 |
| `front()` | 返回队头元素的引用 | 空队列调用会产生未定义行为 |
| `back()` | 返回队尾元素的引用 | 空队列调用会产生未定义行为 |
| `push(x)` | 在队尾插入元素 | 实际调用底层的 `push_back()` |
| `pop()` | 删除队头元素 | 只删除，不返回被删元素 |

```cpp
#include <iostream>
#include <queue>
using namespace std;

int main()
{
    queue<int> q;
    q.push(1);
    q.push(2);
    q.push(3);

    cout << q.front() << endl;   // 1
    cout << q.back() << endl;    // 3

    q.pop();
    cout << q.front() << endl;   // 2

    return 0;
}
```

注意：`queue` 使用的是 `front()` 和 `back()`，没有 `top()`。我在复现 `queue` 时，一开始把队头接口顺手写成了 `top()`，虽然函数内部取的确实是 `_con.front()`，但接口名称已经和标准 `queue` 对不上了。对照标准接口和老师代码后，这里需要改成 `front()`，并补上 `back()`。

所以，适配器的接口名称不只是换个名字，它也表示了这个适配器允许使用者从哪里观察数据：`stack` 只能看 `top`，`queue` 则区分 `front` 和 `back`。

# 四、priority_queue 的介绍与使用

## 4.1 priority_queue 的特点

`priority_queue` 是优先级队列。它不按照普通队列的先进先出规则取数据，而是每次取出当前优先级最高的元素。

默认情况下，`priority_queue` 使用 `vector` 作为底层容器，使用 `less<T>` 作为比较器，所以默认建立的是大堆，`top()` 返回最大元素。

```text
push(x)
   |
   v
+----------------------+
| priority_queue       |
| top() = best element |
+----------+-----------+
           |
           v
+----------------------+
| vector + heap rule   |
+----------------------+
```

## 4.2 priority_queue 的常用接口

| 函数 | 作用 | 注意事项 |
| --- | --- | --- |
| `priority_queue()` | 构造空优先级队列 | 默认底层是 `vector`，默认大堆 |
| `empty()` | 判断是否为空 | 返回 `true` 或 `false` |
| `size()` | 返回元素个数 | 统计当前有效元素 |
| `top()` | 返回堆顶元素的引用 | 大堆取最大，小堆取最小 |
| `push(x)` | 插入元素 | 插入后需要向上调整 |
| `pop()` | 删除堆顶 | 删除后需要向下调整 |

```cpp
#include <functional>
#include <iostream>
#include <queue>
#include <vector>
using namespace std;

int main()
{
    vector<int> v{ 3, 2, 7, 6, 0, 4, 1, 9, 8, 5 };

    priority_queue<int> big_heap;
    for (auto e : v)
    {
        big_heap.push(e);
    }
    cout << big_heap.top() << endl;       // 9

    priority_queue<int, vector<int>, greater<int>> small_heap;
    for (auto e : v)
    {
        small_heap.push(e);
    }
    cout << small_heap.top() << endl;     // 0

    return 0;
}
```

注意：`priority_queue<int>` 默认是大堆。想要建立小堆，需要显式传入 `greater<int>`：

```cpp
priority_queue<int, vector<int>, greater<int>> pq;
```

`priority_queue` 没有 `front()` 和 `back()`，也没有迭代器。它只保证 `top()` 对应的元素满足优先级规则，不允许我们直接遍历内部数组。

## 4.3 priority_queue 为什么默认使用 vector

堆可以看成一棵完全二叉树，但底层不需要真的创建很多树节点，而是可以把它放进一段连续数组中。对下标为 `parent` 的节点来说，孩子下标为：

```cpp
left  = parent * 2 + 1;
right = parent * 2 + 2;
```

对下标为 `child` 的节点来说，父节点下标为：

```cpp
parent = (child - 1) / 2;
```

因此堆需要底层容器支持随机访问。`vector` 的连续空间和下标访问正好适合这个结构。

`stack` 和 `queue` 默认使用 `deque`，是因为它们需要在一端或两端操作；`priority_queue` 默认使用 `vector`，是因为它需要根据下标快速定位父子节点。

# 五、先看 STL 源码

## 5.1 为什么模拟实现前要看源码

以前我写模拟实现时，通常是先根据接口和自己的理解直接写代码。这次在模拟 `stack`、`queue` 和 `priority_queue` 之前，我想先看看 STL 源码的写法，了解标准库是怎样组织这些容器的。

STL 源码是比较早期的 SGI STL 风格实现，里面有大量的宏、类型定义、迭代器、内存分配和异常处理代码。那些内容有些是为了兼容不同编译器，有些是为了让整个 STL 复用得更方便。以我目前的学习阶段，不需要把这些内容全部弄懂。

这次看源码的目的不是把每个函数都解释清楚，而是先学习它的设计思路：**看别人怎么组织代码，先把能看懂的部分抄下来，再逐步理解为什么这样设计。** 对应到源码中暂时看不懂的内存池、复杂的迭代器和兼容性宏，先略过。

## 5.2 STL 源码中的 stack

在 `stl_stack.h` 中，`stack` 的模板参数可以简化为：

```cpp
template<class T, class Sequence = deque<T>>
class stack
{
protected:
    Sequence c;

public:
    bool empty() const { return c.empty(); }
    size_type size() const { return c.size(); }
    reference top() { return c.back(); }
    void push(const value_type& x) { c.push_back(x); }
    void pop() { c.pop_back(); }
};
```

源码中还有很多 `typedef`，例如 `value_type`、`size_type`、`reference` 和 `const_reference`。这些类型定义的作用是复用底层容器已经提供的类型信息。现在先不展开它们，只看最重要的一句：

```cpp
Sequence c;
```

`stack` 自己没有重新定义数组，也没有重新定义节点。它只保存了一个底层容器对象，再把底层容器的接口包装成 `stack` 需要的接口。

```text
+----------------------+
| stack<T, Sequence>   |
+----------+-----------+
           |
           v
+----------------------+
| Sequence c           |
+----------+-----------+
           |
           +--> push -> push_back
           +--> pop  -> pop_back
           +--> top  -> back
```

这里正好可以看到适配器的本质：**上层接口负责表达使用规则，底层容器负责真正保存数据。**

## 5.3 STL 源码中的 queue

`queue` 的结构和 `stack` 很接近，也保存了一个底层容器对象：

```cpp
template<class T, class Sequence = deque<T>>
class queue
{
protected:
    Sequence c;

public:
    bool empty() const { return c.empty(); }
    size_type size() const { return c.size(); }
    reference front() { return c.front(); }
    reference back() { return c.back(); }
    void push(const value_type& x) { c.push_back(x); }
    void pop() { c.pop_front(); }
};
```

这段代码里需要关注的是接口映射：

```text
+----------------------+
| queue<T, Sequence>  |
+----------+-----------+
           |
           v
+----------------------+
| Sequence c           |
+----------+-----------+
           +--> push  -> push_back
           +--> pop   -> pop_front
           +--> front -> front
           +--> back  -> back
```

`queue` 并没有改变底层数据的存储方式，它只是规定了“从队尾进入，从队头出去”。

## 5.4 STL 源码中的 priority_queue

`priority_queue` 比 `stack` 和 `queue` 多了一个比较器：

```cpp
template<class T,
         class Sequence = vector<T>,
         class Compare = less<typename Sequence::value_type>>
class priority_queue
{
protected:
    Sequence c;
    Compare comp;

public:
    const_reference top() const { return c.front(); }

    void push(const value_type& x)
    {
        c.push_back(x);
        push_heap(c.begin(), c.end(), comp);
    }

    void pop()
    {
        pop_heap(c.begin(), c.end(), comp);
        c.pop_back();
    }
};
```

可以把它简化成下面这种关系：

```text
+--------------------------+
| priority_queue           |
+------------+-------------+
             |
     +-------+-------+
     |               |
     v               v
+----------+   +-----------+
| Sequence |   | Compare   |
| vector   |   | comp      |
+----------+   +-----------+
```

`push_heap` 和 `pop_heap` 才是源码中维护堆结构的关键函数。源码里的 `__push_heap` 会不断计算父节点并向上移动，`__adjust_heap` 会从父节点开始选择孩子并向下调整。

源码中还有内存分配、异常处理和迭代器分类等内容，这篇暂时不展开。对现在的我来说，先看清楚下面三件事就已经很有收获了：

1. `stack` 和 `queue` 保存底层容器对象，自己只包装接口；
2. `priority_queue` 还保存一个比较器对象；
3. 堆算法被单独抽出来，适配器只负责调用它。

# 六、仿函数：把比较规则单独拿出来

## 6.1 先写一个只能建立大堆的版本

如果先不考虑小堆，可以直接把比较条件写死：

```cpp
template<class T>
class priority_queue
{
private:
    vector<T> _con;

public:
    void AdjustUp(int child)
    {
        int parent = (child - 1) / 2;
        while (child > 0)
        {
            if (_con[parent] < _con[child])
            {
                swap(_con[parent], _con[child]);
                child = parent;
                parent = (child - 1) / 2;
            }
            else
            {
                break;
            }
        }
    }
};
```

这里的判断条件是：如果父节点小于子节点，就交换。较大的元素会不断向上，所以建立的是大堆。

但是，如果以后想建立小堆，就要把条件改成：

```cpp
_con[parent] > _con[child]
```

如果大堆和小堆都需要，就只能复制一份代码，或者手动修改比较条件。这样做不但重复，而且容易改错。堆的结构和调整过程其实没有改变，真正改变的只是“两个元素如何比较”。

## 6.2 仿函数是什么

仿函数本质上是一个重载了 `operator()` 的类。它的对象可以像函数一样被调用。

```cpp
template<class T>
struct Less
{
    bool operator()(const T& x, const T& y) const
    {
        return x < y;
    }
};

template<class T>
struct Greater
{
    bool operator()(const T& x, const T& y) const
    {
        return x > y;
    }
};
```

使用时，仿函数对象和普通函数很像：

```cpp
Less<int> less_func;
Greater<int> greater_func;

cout << less_func(1, 2) << endl;
cout << greater_func(1, 2) << endl;
```

调用：

```cpp
less_func(1, 2)
```

本质上等价于：

```cpp
less_func.operator()(1, 2)
```

所以，仿函数可以把“比较规则”装进一个对象里。堆只需要调用这个对象，不需要知道具体使用的是小于号还是大于号。

```text
+------+      +-----------+      +------+
| x,y  | ---> | Compare   | ---> | bool |
+------+      +-----------+      +------+
                  |
             true  -> swap
             false -> stop
```

## 6.3 仿函数在大堆和小堆中的作用

在堆的调整代码中，比较条件改成：

```cpp
if (com(_con[parent], _con[child]))
{
    swap(_con[parent], _con[child]);
}
```

当 `com` 是 `Less<int>` 时：

```cpp
com(parent, child) == parent < child
```

父节点小于子节点时交换，较大的元素向上移动，最后得到大堆。

当 `com` 是 `Greater<int>` 时：

```cpp
com(parent, child) == parent > child
```

父节点大于子节点时交换，较小的元素向上移动，最后得到小堆。

因此，`Less` 和 `Greater` 的名字描述的是比较器内部的判断方式，不能简单地把 `Less` 理解成小堆、把 `Greater` 理解成大堆。真正的结果要看比较器和调整算法配合后的效果。

# 七、自定义类型的比较

## 7.1 Date 对象的比较

之前的类和对象笔记中已经实现过 `Date` 类，并重载了日期之间的比较运算符。只要 `Date` 支持比较，就可以把 `Date` 对象放入 `priority_queue`。

```cpp
class Date
{
public:
    Date(int year = 1, int month = 1, int day = 1)
        : _year(year), _month(month), _day(day)
    {}

    bool operator<(const Date& d) const
    {
        if (_year != d._year)
            return _year < d._year;
        if (_month != d._month)
            return _month < d._month;
        return _day < d._day;
    }

private:
    int _year;
    int _month;
    int _day;
};
```

使用默认比较器时：

```cpp
priority_queue<Date> pq;
pq.push(Date(2024, 7, 31));
pq.push(Date(2025, 1, 1));
pq.push(Date(2023, 12, 1));
```

这里比较的是 `Date` 对象本身，默认会调用 `Date::operator<`。建立大堆后，堆顶是日期更大的元素。

## 7.2 Date 指针的比较

如果优先级队列里保存的是 `Date*`，情况就不一样了：

```cpp
priority_queue<Date*> pq;
```

此时元素类型是指针，默认比较的是指针的地址关系，并不会自动解引用后比较日期内容。也就是说，地址大的指针不代表它指向的日期更大。

如果希望按照指针指向的日期比较，需要自己写比较器：

```cpp
struct DatePtrLess
{
    bool operator()(const Date* x, const Date* y) const
    {
        return *x < *y;
    }
};
```

使用时：

```cpp
priority_queue<Date*, vector<Date*>, DatePtrLess> pq;
```

这个比较器的过程可以画成：

```text
Date* x ----+
            |
            v
           *x ----+
                  |
Date* y ----+     v
            |   Date::operator<
            +-------> bool
```

这里的重点是：**比较器可以把容器中的元素转换成我们真正想比较的内容。** 对象直接比较时比较 `Date`，指针比较时先解引用再比较 `Date`。

# 八、priority_queue 的模拟实现

## 8.1 模拟实现的整体结构

根据前面看到的源码和自己的理解，我的模拟实现可以写成下面这样：

```cpp
template<class T,
         class Container = vector<T>,
         class Compare = Less<T>>
class priority_queue
{
private:
    Container _con;
    Compare _comp;

public:
    void push(const T& x)
    {
        _con.push_back(x);
        AdjustUp(static_cast<int>(_con.size()) - 1);
    }

    void pop()
    {
        swap(_con[0], _con[_con.size() - 1]);
        _con.pop_back();
        AdjustDown(0);
    }

    const T& top() const
    {
        return _con[0];
    }

    size_t size() const
    {
        return _con.size();
    }

    bool empty() const
    {
        return _con.empty();
    }
};
```

这里先把三个角色分开：

- `Container _con`：真正保存数据；
- `Compare _comp`：保存比较规则；
- `AdjustUp` 和 `AdjustDown`：维护堆结构。

## 8.2 堆和完全二叉树

以大堆为例，堆顶元素最大，但并不是所有父节点都要比整棵子树的每个元素大。堆只要求每个父节点都满足和自己孩子之间的关系。

```text
             [9]0
            /    \
         [7]1    [8]2
         /  \    /  \
      [3]3 [4]4 [1]5 [6]6
```

数组形式是：

```text
idx: 0  1  2  3  4  5  6
arr: 9  7  8  3  4  1  6
```

下标关系：

```cpp
parent = (child - 1) / 2;
left   = parent * 2 + 1;
right  = parent * 2 + 2;
```

这些公式是向上调整和向下调整的基础。

# 九、向上调整：插入元素时上浮

## 9.1 为什么需要向上调整

向堆中插入一个新元素时，不能随便放在数组中间，否则完全二叉树的结构会被破坏。最简单的办法是先把新元素放到数组末尾，也就是完全二叉树的下一个叶子位置。

但是新元素可能比父节点更大，这样就不满足大堆规则。因此要让它不断和父节点比较，必要时交换，直到它到达合适的位置。

这个过程叫**向上调整**，也可以理解成元素不断“上浮”。

## 9.2 向上调整的完整过程

假设原来的大堆是：

```text
idx: 0  1  2  3  4
arr: 8  5  6  3  4
```

对应的树是：

```text
          [8]0
         /    \
      [5]1    [6]2
      /  \
   [3]3  [4]4
```

现在插入 `9`，先放到数组末尾：

```text
idx: 0  1  2  3  4  5
arr: 8  5  6  3  4  9
                         ^ child = 5
```

根据公式：

```cpp
parent = (child - 1) / 2;
parent = (5 - 1) / 2 = 2;
```

父节点是下标 `2`，值为 `6`。因为 `6 < 9`，不满足大堆关系，所以交换：

```text
idx: 0  1  2  3  4  5
arr: 8  5  9  3  4  6
             ^ child = 2
```

现在新元素 `9` 到了下标 `2`，继续计算父节点：

```cpp
parent = (2 - 1) / 2 = 0;
```

父节点是下标 `0`，值为 `8`。因为 `8 < 9`，继续交换：

```text
idx: 0  1  2  3  4  5
arr: 9  5  8  3  4  6
      ^ child = 0
```

此时 `child == 0`，已经没有父节点，向上调整结束。

```text
          [9]0
         /    \
      [5]1    [8]2
      /  \    /
   [3]3 [4]4 [6]5
```

## 9.3 向上调整的代码

```cpp
void AdjustUp(int child)
{
    Compare com;
    int parent = (child - 1) / 2;

    while (child > 0)
    {
        if (com(_con[parent], _con[child]))
        {
            swap(_con[parent], _con[child]);
            child = parent;
            parent = (child - 1) / 2;
        }
        else
        {
            break;
        }
    }
}
```

以 `Less` 建立大堆时：

```cpp
com(_con[parent], _con[child])
```

等价于：

```cpp
_con[parent] < _con[child]
```

只要父节点小于子节点，就交换。交换之后，`child` 移动到原来的父节点位置，继续向上检查。

向上调整的方向可以概括成：

```text
new node
   ^
   |
parent
   ^
   |
parent
```

## 9.4 向上调整的复杂度

每向上移动一层，所在下标都会变成原来父节点的下标。完全二叉树的高度大约是 `log N`，所以一次向上调整的时间复杂度是 `O(log N)`。

# 十、向下调整：删除元素时下沉

## 10.1 为什么删除堆顶后要向下调整

`priority_queue::pop()` 删除的是堆顶元素，也就是数组下标 `0` 的元素。如果直接删除数组第一个元素，后面的元素整体移动，完全二叉树的下标关系就会发生变化。

常见做法是：

1. 交换堆顶和数组末尾；
2. 删除数组末尾；
3. 从堆顶开始向下调整。

这样既删除了原来的堆顶，又保留了完全二叉树的数组结构。

## 10.2 向下调整的完整过程

仍然使用大堆：

```text
idx: 0  1  2  3  4  5
arr: 9  5  8  3  4  6
```

对应的树：

```text
          [9]0
         /    \
      [5]1    [8]2
      /  \    /
   [3]3 [4]4 [6]5
```

先交换堆顶 `9` 和末尾 `6`：

```text
idx: 0  1  2  3  4  5
arr: 6  5  8  3  4  9
```

删除末尾的原堆顶：

```text
idx: 0  1  2  3  4
arr: 6  5  8  3  4
```

现在从下标 `0` 开始向下调整。父节点是 `6`，左孩子为 `5`，右孩子为 `8`：

```text
          [6]0
         /    \
      [5]1    [8]2
      /  \
   [3]3  [4]4
```

为了让大堆成立，应该先选出两个孩子中较大的那个，也就是 `8`。因为 `6 < 8`，交换父节点和右孩子：

```text
idx: 0  1  2  3  4
arr: 8  5  6  3  4
```

新的父节点下标是 `2`。下标 `2` 已经没有孩子，调整结束。

```text
          [8]0
         /    \
      [5]1    [6]2
      /  \
   [3]3  [4]4
```

## 10.3 为什么必须选择更大的孩子

向下调整时，父节点可能有两个孩子。如果只判断左孩子，不判断右孩子，可能会把父节点和较小的孩子交换，交换后较大的孩子仍然比父节点大，堆就没有调整好。

所以大堆的向下调整必须：

1. 先假设左孩子是较大孩子；
2. 如果右孩子存在，并且右孩子更大，就改选右孩子；
3. 父节点和较大的孩子比较；
4. 如果父节点较小，就交换并继续下沉。

```text
          parent
          /    \
       left    right
          \    /
        choose the larger child
```

## 10.4 向下调整的代码

```cpp
void AdjustDown(int parent)
{
    Compare com;
    int child = parent * 2 + 1;  // 先假设左孩子

    while (child < static_cast<int>(_con.size()))
    {
        if (child + 1 < static_cast<int>(_con.size())
            && com(_con[child], _con[child + 1]))
        {
            ++child;              // 右孩子更符合优先级
        }

        if (com(_con[parent], _con[child]))
        {
            swap(_con[parent], _con[child]);
            parent = child;
            child = parent * 2 + 1;
        }
        else
        {
            break;
        }
    }
}
```

对于大堆，`com` 是 `Less`。如果左孩子小于右孩子，说明右孩子更大，于是选择右孩子。

```cpp
com(_con[child], _con[child + 1])
```

等价于：

```cpp
_con[child] < _con[child + 1]
```

如果比较结果为 `true`，就让 `child` 加 `1`，从左孩子切换到右孩子。

## 10.5 向上和向下的方向不能写反

向上调整时，新元素从叶子节点出发，移动到父节点：

```cpp
child = parent;
parent = (child - 1) / 2;
```

向下调整时，父节点移动到选中的孩子位置，再寻找下一层的孩子：

```cpp
parent = child;
child = parent * 2 + 1;
```

我在自己的 `PriorityQueue.h` 中就踩过这个坑：向下调整时，把向上调整的更新方向顺手复制了过来。这样 `parent` 和 `child` 的计算方向就错了，元素没有真正向下走，堆结构也不能正确恢复。

```text
AdjustUp:    child  -> parent -> parent -> root
AdjustDown:  parent -> child  -> child  -> leaf
```

这两个过程的循环条件和下标公式很像，但移动方向完全相反，写代码时一定要分清楚。

## 10.6 向下调整的复杂度

一次向下调整最多从根节点走到叶子节点，完全二叉树的高度大约是 `log N`，所以时间复杂度也是 `O(log N)`。

# 十一、stack 和 queue 的模拟实现

## 11.1 stack 的模拟实现

`stack` 只需要把底层容器的尾部接口包装起来：

```cpp
template<class T, class Container = deque<T>>
class stack
{
public:
    void push(const T& x)
    {
        _con.push_back(x);
    }

    void pop()
    {
        _con.pop_back();
    }

    const T& top() const
    {
        return _con.back();
    }

    size_t size() const
    {
        return _con.size();
    }

    bool empty() const
    {
        return _con.empty();
    }

private:
    Container _con;
};
```

这里没有重新实现栈的存储结构，所有操作都委托给 `_con`。所以 `stack` 的重点不是写出复杂算法，而是理解如何通过模板参数复用底层容器。

## 11.2 queue 的模拟实现

`queue` 和 `stack` 的区别在于操作的两端：

```cpp
template<class T, class Container = deque<T>>
class queue
{
public:
    void push(const T& x)
    {
        _con.push_back(x);
    }

    void pop()
    {
        _con.pop_front();
    }

    const T& front() const
    {
        return _con.front();
    }

    const T& back() const
    {
        return _con.back();
    }

    size_t size() const
    {
        return _con.size();
    }

    bool empty() const
    {
        return _con.empty();
    }

private:
    Container _con;
};
```

我自己的 `queue.h` 最开始写成了 `top()`，对照标准接口和老师代码后，这里需要改成 `front()`、`back()`。这件事也提醒我，模拟实现不能只看函数体能不能工作，还要对照原接口的命名和使用方式。

# 十二、priority_queue 的完整模拟实现

前面的各个部分拼起来，可以得到一个简化版的 `priority_queue`：

```cpp
#include <vector>
using namespace std;

template<class T>
struct Less
{
    bool operator()(const T& x, const T& y) const
    {
        return x < y;
    }
};

template<class T>
struct Greater
{
    bool operator()(const T& x, const T& y) const
    {
        return x > y;
    }
};

template<class T,
         class Container = vector<T>,
         class Compare = Less<T>>
class priority_queue
{
private:
    Container _con;

    void AdjustUp(int child)
    {
        Compare com;
        int parent = (child - 1) / 2;

        while (child > 0)
        {
            if (com(_con[parent], _con[child]))
            {
                swap(_con[parent], _con[child]);
                child = parent;
                parent = (child - 1) / 2;
            }
            else
            {
                break;
            }
        }
    }

    void AdjustDown(int parent)
    {
        Compare com;
        int child = parent * 2 + 1;

        while (child < static_cast<int>(_con.size()))
        {
            if (child + 1 < static_cast<int>(_con.size())
                && com(_con[child], _con[child + 1]))
            {
                ++child;
            }

            if (com(_con[parent], _con[child]))
            {
                swap(_con[parent], _con[child]);
                parent = child;
                child = parent * 2 + 1;
            }
            else
            {
                break;
            }
        }
    }

public:
    void push(const T& x)
    {
        _con.push_back(x);
        AdjustUp(static_cast<int>(_con.size()) - 1);
    }

    void pop()
    {
        swap(_con[0], _con[_con.size() - 1]);
        _con.pop_back();
        AdjustDown(0);
    }

    const T& top() const
    {
        return _con[0];
    }

    size_t size() const
    {
        return _con.size();
    }

    bool empty() const
    {
        return _con.empty();
    }
};
```

这里的实现是为了学习适配器、仿函数和堆算法的设计思路，并不是完整替代标准库。比如空容器调用 `top()` 或 `pop()` 的检查、构造函数、迭代器范围构造和异常处理等内容，还可以继续完善。

# 十三、总结

这篇主要学习了三种容器适配器。

`stack` 遵循后进先出规则，只能访问栈顶；`queue` 遵循先进先出规则，从队尾入队、从队头出队；`priority_queue` 按优先级取出元素，默认使用 `vector` 保存堆，默认建立大堆。

适配器的核心思想是复用已有容器：底层容器负责保存数据，适配器负责限制接口和表达使用规则。通过查看 STL 源码，我看到 `stack` 和 `queue` 都只是保存一个底层容器对象，`priority_queue` 则在此基础上增加了比较器和堆算法。

仿函数解决了比较规则变化的问题。把比较条件从堆的主体代码中抽出来后，同一套向上调整和向下调整代码就可以通过 `Less`、`Greater` 或自定义比较器建立不同类型的堆。对于 `Date` 对象，可以直接比较日期；对于 `Date*`，则需要自己解引用后比较对象内容。

堆的两个核心算法也需要分清：插入元素后使用向上调整，让新元素不断和父节点比较；删除堆顶后使用向下调整，选择更符合优先级的孩子继续下沉。向上调整是 `child` 向 `parent` 移动，向下调整是 `parent` 向 `child` 移动，两个方向写反就会导致堆结构恢复失败。

从 `vector`、`list` 到容器适配器，我越来越能体会到 STL 的分层设计：先把底层容器写好，再通过适配器限制接口，最后用仿函数把容易变化的规则抽出来。学习源码也不一定要一开始就全部看懂，先观察结构、复现能理解的部分，再慢慢补上缺少的知识，这也是我这次看 STL 源码最大的收获。

---

## 相关笔记 / 相关链接

- 相关目录：[[C++目录]]
- 相关笔记：[[数据结构/栈]]（前置：栈的操作）、[[数据结构/队列与双栈模拟]]（前置：队列的操作）、[[C++/C++模板进阶——特化全解]]（前置：仿函数）

---
