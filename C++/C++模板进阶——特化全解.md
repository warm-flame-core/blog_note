---
tags:
  - 形态/概念
aliases:
  - 模板特化/模板进阶
日期: 2026年9月6日
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
  <a href="https://blog.csdn.net/hahaha6565/category_13163238.html?spm=1001.2014.3001.5482">C++博客_CSDN</a>
</p>
<p align="center">
  <font color="#00FFFF"><b>作者仓库</b></font><br>
  <a href="https://gitee.com/warm-flame-core">代码仓库_gitee</a>
</p>


# 引言

在之前介绍过的模板初阶里，我们学会了函数模板和类模板的基本写法。那种模板解决的是"通用逻辑"——不管传进来的是 `int`、`double` 还是 `Date`，走同一套代码。

但通用模板不是万能的。当一个类型"长得比较特殊"（比如是指针、是引用、或者带 `const`）的时候，通用模板往往会给出**错误的结果**。这篇《模板进阶》就来讲讲：**模板特化**——怎么给特殊类型"特殊处理"，以及它背后一堆容易踩的坑。

# 一、特化的动机

先看一个最经典的例子。我写了一个通用的比较函数模板 `LessFunc`：

```cpp
template<class T>
bool LessFunc(const T& left, const T& right)
{
	return left < right;   // 直接比 "<"
}
```

- 传 `int`、`double`：比的是数值，正确。
- 传 `Date` 对象：`Date` 重载了 `operator<`，比的也是日期值，正确。
- 传 `Date*` 指针：**出问题了**。这里比的是 `left < right`，也就是两个**指针的地址**谁大谁小，而不是它们指向的日期谁早谁晚。

```text
  日期对象比较（正确）        Date* 指针比较（出错）
  d1 = 2022-7-7               p1 --> d1 = 2022-7-7
  d2 = 2022-7-8               p2 --> d2 = 2022-7-8
  d1 < d2 比的是日期值        p1 < p2 比的是地址
```

这就是特化的**动机**：对某些特殊类型，通用模板的逻辑不对，需要单独"定制"一份。

# 二、模板特化与重载的对比

## 2.1 const 在 `*` 两侧的四态

这个坑的根源，其实是 **`const` 放在 `*` 的哪一边，锁的完全不是同一个东西**。我用一张表说清楚：

| 写法 | 含义 | 能否改指针 | 能否改指向的值 |
|---|---|---|---|
| `Date*` | 指向 `Date` | 能 | 能 |
| `const Date*` | 指向 `const Date` | 能 | 不能 |
| `Date* const` | const 指针 | 不能 | 能 |
| `const Date* const` | 指向 const 的 const 指针 | 不能 | 不能 |

一句话记：**`const` 在 `*` 左边，锁的是"指向的值"；在 `*` 右边，锁的是"指针本身"**。写代码时可以直接这么记：

```cpp
// const int *  : 不能通过这个指针改指向的值，但能改指针指向别处
// int * const  : 指针本身不能改，但能改它指向的值
```

## 2.2 Date* VS const Date*

明白了 `const` 的位置，再回头看函数模板特化。原模板是：

```cpp
template<class T>
bool LessFunc(const T& left, const T& right)
```

当 `T = Date*` 时，`const T` 其实是 `Date* const`——`const` 在 `*` 的**右边**，锁的是指针。所以正确的特化形参必须是 `Date* const&`：

```cpp
template<>
bool LessFunc<Date*>(Date* const& left, Date* const& right)
{
	return *left < *right;
}
```

而当 `T = const Date*` 时，`const` 在 `*` 的**左边**，锁的是指向的值。特化形参就变成 `const Date* const&`：

```cpp
template<>
bool LessFunc<const Date*>(const Date* const& left, const Date* const& right)
{
	return *left < *right;
}
```

```text
  原模板: bool LessFunc(const T& l, const T& r)
    T=Date*        ==>  Date* const&
    T=const Date*  ==>  const Date* const&

  特化<Date*>       使用 Date* const&          √
  特化<const Date*> 使用 const Date* const&    √
  写错              使用 const Date*&          ✘
```

我一开始就顺着 `const` 字面写成了 `const Date*&`，结果**类型对不上**。因为 `Date*`（指向普通 Date 的指针）和 `const Date*&`（指向 const Date 的指针引用）根本是两种类型，编译器不会把这个特化匹配到 `Date*` 上。

## 2.3 函数模板特化的局限性

由此可见，函数模板特化**对形参类型的要求极为苛刻**，`const` 位置、引用写法差一点就匹配不上，很容易出"看起来对、编译错"的坑。所以通用的建议是：

> 函数模板一般不建议特化，遇到处理不了的类型，直接**写一个普通函数重载**更省心、可读性更高：

```cpp
bool LessFunc(const Date* left, const Date* right)
{
	return *left < *right;
}

bool LessFunc(Date* left, Date* right)
{
	return *left < *right;
}
```

# 三、类模板特化

函数模板特化不推荐，但**类模板特化**是真用得上的，而且能讲出更多名堂。先看一个模板类 `Data`：

```cpp
template<class T1, class T2>
class Data
{
public:
	Data() { cout << "Data<T1, T2>" << endl; }
private:
	T1 _d1;
	T2 _d2;
};
```

## 3.1 全特化

全特化就是`模板参数列表里的所有参数都确定下来了`，单独写一份：

```cpp
template<>
class Data<int, double>
{
public:
	Data() { cout << "Data<int, double>" << endl; }
};
```

## 3.2 偏特化——部分参数特化

偏特化（也叫半特化）是"只把一部分参数固定，剩下的还留成模板参数"：

```cpp
template<class T1>
class Data<T1, double>
{
public:
	Data() { cout << "Data<T1, double>" << endl; }
};
```

这里第二个参数固定成 `double`，第一个参数 `T1` 还留着。

## 3.3 偏特化——结构限制特化

偏特化不只是"固定部分参数"，还能对参数做**进一步的结构限制**，比如限定参数必须是指针、必须是引用：

```cpp
// 两个参数都限定为指针
template <typename T1, typename T2>
class Data <T1*, T2*>
{
public:
	Data() { cout << "Data<T1*, T2*>" << endl; }
};

// 两个参数都限定为引用
template <typename T1, typename T2>
class Data <T1&, T2&>
{
public:
	Data() { cout << "Data<T1&, T2&>" << endl; }
};

// T1 是引用、T2 是指针
template <typename T1, typename T2>
class Data <T1&, T2*>
{
public:
	Data() { cout << "Data<T1&, T2*>" << endl; }
	void Push(const T1& x) {}
};
```

## 3.4 全特化与偏特化的优先级

这里有个好玩的点。看这几个实例化：

```cpp
Data<int, int>      d1;   // 主模板
Data<int, char>     d2;   // 主模板
Data<int, double>   d3;   // 同时命中 全特化 和 偏特化
Data<char, double>  d4;   // 命中偏特化  Data<T1,double>
Data<char, char>    d5;   // 主模板
Data<char*, char*>  d6;   // Data<T1*,T2*>
Data<int*, char*>   d7;   // Data<T1*,T2*>
Data<int&, char&>   d8;   // Data<T1&,T2&>
Data<int&, double&> d9;   // Data<T1&,T2&>
Data<int&, int*>    d10;  // Data<T1&,T2*>
```

`d3` 是 `Data<int, double>`，它同时能匹配**全特化** `Data<int,double>` 和**偏特化** `Data<T1,double>`（此时 `T1=int`）。两条路都通，编译器选谁？

```text
  Data<int, double> 实例化时, 能同时匹配:
     全特化: Data<int, double>        ★ 优先命中
     偏特化: Data<T1, double> (T1=int)
  → 全特化比偏特化更"具体", 优先选全特化
```

结论：**全特化优先于偏特化，偏特化优先于主模板**。

# 四、偏特化的深层坑

## 4.1 指针偏特化里 const 的吸收

这是最容易绕晕的地方。看 `Data<const int*, char*>`：它匹配结构偏特化 `Data<T1*, T2*>` 时，`T1` 被推成什么？

```cpp
template <typename T1, typename T2>
class Data <T1*, T2*>
{
public:
	Data() { cout << "Data<T1*, T2*>" << endl; }
};
```

```text
  Data<const int*, char*> 匹配 Data<T1*, T2*>
     T1 = const int   ← const 被吸收进 T1
     T2 = char
```

注意：`T1` 是 `const int`，**不是** `int`。本来 `const int*` 是指向 const 的指针，但结构模式 `T1*` 把那个 `const` 吸收成 `T1` 的一部分了。之所以这种结构特化有用，就是因为它能**用 `T1` 吸收 `const` 来区分不同指针**——`const int*` 和 `int* const` 会对应到不同的 `T1`。

## 4.2 引用/指针偏特化里 T1 是底层类型

再看 `Data<int&, int*>`，它匹配 `Data<T1&, T2*>`。这里 `T1` 到底是不是 `int&`？

```cpp
template <typename T1, typename T2>
class Data <T1&, T2*>
{
public:
	Data() { cout << "Data<T1&, T2*>" << endl; }
	void Push(const T1& x) {}
};
```

```text
  Data<int&, int*> 匹配 Data<T1&, T2*>
     T1 = int        ← 引用被吸收, T1 是底层类型
     T2 = int
     真正引用在 T1&  --> int&
```

不是！`T1` 是 `int`，**不是** `int&`。`T1&` 才是 `int&`。也就是说，结构偏特化里的类型变量存的是"去掉结构修饰后的底层类型"，真正的引用/指针/const 都体现在模式上，而不体现在类型变量里。

所以类里写 `Push(const T1& x)`，当 `T1=int` 时其实是 `Push(const int& x)`；而当实参是 `Data<const int&, int*>` 时，`T1=const int`，`Push` 的形参就变成 `const int&`（`const` 会折叠）。这些细节，就是偏特化最磨人的地方。

# 五、非类型模板参数

模板形参不一定是类型，还可以是**常数**。`std::array` 就用它来定数组大小：

```cpp
template<class T, size_t N = 10>
class array
{
public:
	T& operator[](size_t index) { return _array[index]; }
	const T& operator[](size_t index) const { return _array[index]; }
	size_t size() const { return _size; }
	bool empty() const { return 0 == _size; }
private:
	T _array[N];
	size_t _size;
};
```

`array<int, 100>` 就是一个 100 个元素的数组；`N` 是编译期就能确定的常数。非类型模板参数**必须在编译期确定**；浮点数、类对象、字符串在 C++17 之前不能用作非类型模板参数，C++20 开始允许浮点。

# 六、模板的分离编译

模板的声明和定义如果分开放在两个 `.cpp` 里，**链接会失败**。比如 `a.h` 里只声明 `template<class T> T Add(const T&, const T&)`，`a.cpp` 里写定义，`main.cpp` 里调用——因为模板要遇到实际类型才能"实例化"出代码，而 `a.cpp` 编译时不知道 `main` 会实例化 `int` 还是 `double`，所以链接时报"无法解析的外部符号"。

解决办法是**把声明和定义都放进 `.h` / `.hpp`**，或者反过来用"显式实例化"把类型提前告诉编译器。

---

## 相关笔记 / 相关链接

- 相关目录：[[C++目录]]
- 相关笔记：[[C++/C++内存管理和模板初阶]]（前置：模板基础）
