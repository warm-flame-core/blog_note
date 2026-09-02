---
tags:
  - 编程
aliases:
  - CPlusPlus
日期: 2026年5月7日
---


![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/78623ff9c20f4ee58344321c4a2ac0f7.jpeg#pic_center)

<p align="center">
  <font color="#FF69B4"><b>Ciallo(∠・ω< )⌒☆</b></font>
</p>

<p align="center">
  <font color="#00FFFF"><b>C语言专栏</b></font><br>
  <a href="https://blog.csdn.net/hahaha6565/category_13108044.html">C语言博客_CSDN</a>
</p>
<br>
<p align="center">
  <font color="#00FFFF"><b>数据结构专栏</b></font><br>
  <a href="https://blog.csdn.net/hahaha6565/category_13156520.html">数据结构博客_CSDN</a>
</p>
<br>
<p align="center">
  <font color="#00FFFF"><b>C++专栏</b></font><br>
  <a href="[C++_暖焰核心的博客-CSDN博客](https://blog.csdn.net/hahaha6565/category_13163238.html?spm=1001.2014.3001.5482)">C++博客_CSDN</a>
</p>
<br>
<p align="center">
  <font color="#00FFFF"><b>作者仓库</b></font><br>
  <a href="https://gitee.com/warm-flame-core">代码仓库_gitee</a>
</p>


# 引言

最近学完了C语言，开始进行C++学习，大概浏览了一下，很多东西，很有趣但是也很难，让我们开始吧
<center><b style="color: red;">请注意，本系列的C++博客or笔记是在学完C语言的情况下写的，如有看不懂的地方，~可能是读者C语言尚未完全掌握（bushi），也有可能是笔者C语言尚未掌握</b></center>
# 一、 为什么要用C++

在之前的系列中，我们用C语言实现了各种数据结构。虽然C语言强大且高效，但在处理大型项目时，它缺乏对数据和行为的封装能力，在写`贪吃蛇`那个项目时，我们已经感觉到**困难**了。C++在保持C语言高效性的同时，引入了面向对象（OOP）和泛型编程（STL）的特性
## 1.1简要发展史

C++的发展史就是一部“填坑”与“加功能”的历史。从1998年的第一个标准到如今的C++23，每个版本都试图解决特定的问题。

| 版本 | 关键词 | 核心特性/意义 |
| :--- | :--- | :--- |
| **C++98** | 奠基者 | 官方首个标准，引入STL（标准模板库），确立了模板编程的地位。 |
| **C++11** | 革命者 | **面试必考！** 引入了`auto`、`lambda`、智能指针、右值引用等，让C++焕发新生。 |
| **C++14/17** | 完善者 | 对C++11的修补与扩展，如`if constexpr`、结构化绑定，让代码更简洁。 |
| **C++20** | 模块化 | 引入了**Concepts**、**Modules**（模块）、协程，试图解决头文件编译慢等历史顽疾。 |
| **C++23** | 现代化 | 小步快跑，增加了`std::print`、`if consteval`等，进一步提升开发体验。 |
但是令人疑惑不解的是C++到现如今还是没有原生的`网络库`，工程上的网络库都是第三方的，本来在C++23标准制定的时候就有想法要加入网络库，但是由于标准委员会内部对于使用 Asio 还是 Senders/Receivers 方案的争论，最终Networking库遗憾缺席，这件事让人十分恼火，明明是底层的语言，偏偏对网络不能原生支持
## 1.2 方便之处

在C语言中，我们习惯用 `printf` 和 `scanf`。但在C++中，我们有了更安全、更智能的 `cout` 和 `cin`。

**核心差异：**
*   **自动识别类型**：C语言的 `printf` 需要手动指定 `%d` 或 `%f`，容易出错；C++的 `<<` 运算符会自动根据变量类型进行推导。
*   **流的概念**：`cin`（输入流）和 `cout`（输出流）是面向对象的产物，它们不仅仅是函数，更是对象。

### 1.2.1 常用小技巧
在竞赛或高性能场景下，C++的IO默认是兼容C的（为了保证 `printf` 和 `cout` 混用时不乱序），但是这会带来性能损耗，很多OJ题目会超时，我们通常会在代码开头加上：
```cpp
ios_base::sync_with_stdio(false); 
cin.tie(nullptr);
```
这两段话可以让C++的IO只需要照顾C++的方便，极大提高了效率
# 二、 命名空间 (namespace)

在C语言中，如果我们在项目中定义了一个叫 `rand` 的变量，很容易与库函数中的 `rand()` 函数冲突
```c++
#include <stdlib.h>
#include <stdio.h>
int rand = 10;
int main()
{
	printf("%d", rand);
}
```
会提示这样错误
```TEXT
“rand”: 重定义；以前的定义是“函数”
```
为了解决多人创作的时候出现同样的名字，C++引入了 **命名空间（namespace）** 来解决这个问题。
*   **定义**：`namespace bit { ... }`
*   **使用方式**：
    1.  **指定访问**：`bit::rand` （推荐，最安全）
    2.  **部分展开**：`using bit::rand;`
    3.  **全部展开**：`using namespace std;` （日常练习方便，但项目中慎用，容易造成命名污染）

命名空间使得C++允许同样的`变量名`或者`函数名`。而且支持嵌套命名空间
```c++
namespace wfc
{
	namespace xie
	{
		int rand = 10;
		int Add(int a, int b)
		{
			return a + b;
		}
	}
}
using wfc::xie::rand;
int main()
{
	printf("%d\n", rand);
	printf("%d\n", wfc::xie::Add(1, 2));
	return 0;
}
```
- namespace本质是定义出⼀个域，这个域跟全局域各⾃独⽴，不同的域可以定义同名变量，所以下 ⾯的rand不在冲突了。 
- C++中域有函数**局部域，全局域，命名空间域，类域**；域影响的是编译时语法查找⼀个`变量`/`函数`/ `类型出处(声明或定义)`的逻辑，所有有了域隔离，名字冲突就解决了。局部域和全局域除了会影响 编译查找逻辑，还会影响变量的⽣命周期，命名空间域和类域不影响变量⽣命周期
- 项⽬⼯程中多⽂件中定义的同名namespace会认为是⼀个namespace，不会冲突
- C++也怕自己的函数和别人第三方库名字冲突，C++标准库都放在⼀个叫std(standard)的命名空间中
# 三、 核心语法新特性

## 3.1 缺省参数 (Default Arguments)

就像函数参数有了“默认值”。这在写代码时非常有用
```c++
#include <iostream>
using namespace std;
int Add(int a, int b = 10)
{
	return a + b;
}

double Add(double a, double b = 10)
{
	return a + b;
}

int main()
{
	std::cout << Add(1) << std::endl;
}
```
缺省参数是声明或定义函数时为函数的参数指定⼀个缺省值。在调⽤该函数时，如果没有指定实参 则采⽤该形参的缺省值，否则使⽤指定的实参
其中缺省参数还分为**全缺省和半缺省**
- 全缺省就是全部形参给缺省值
	规定半缺省参数必须从右往左 依次连续缺省，不能间隔跳跃给缺省值
```c++
void Func1(int a = 5, int b = 10, int c = 20)
{
	cout << "a = " << a << endl;
	cout << "b = " << b << endl;
	cout << "c = " << c << endl << endl;
}
```
- 半缺省就是部分形参给缺省值
```c++
void Func2(int a, int b = 10, int c = 20)
{
	cout << "a = " << a << endl;
	cout << "b = " << b << endl;
	cout << "c = " << c << endl << endl;
}
```
带缺省参数的函数调⽤，C++规定必须从左到右依次给实参，不能跳跃给实参
函数声明和定义分离时，缺省参数不能在函数声明和定义中同时出现，规定必须函数声明给缺省值

## 3.2 函数重载 (Function Overloading)

C++允许同名函数存在，只要它们的参数列表（个数、类型、顺序）不同
1. 参数类型不同
```c++
int Add(int left, int right)
{
	cout << "int Add(int left, int right)" << endl;
	return left + right;
}
double Add(double left, double right)
{
	cout << "double Add(double left, double right)" << endl;
	return left + right;
}
```
2. 参数个数不同
```c++
void f()
{
	cout << "f()" << endl;
}
void f(int a)
{
	cout << "f(int a)" << endl;
}
```
3. 参数类型顺序不同
```c++
void f()
{
	cout << "f()" << endl;
}
void f(int a)
{
	cout << "f(int a)" << endl;
}
```
不过要注意返回值不同不能作为重载条件，因为只看条用语句无法分辨条用哪个函数
<b style="color: red;">请注意重载函数可以有缺省参数，但是不明确的缺省参数可能会导致歧义</b>
```c++
void f1()
{
	cout << "f1()" << endl;
}
void f1(int a = 10)
{
	cout << "f1()" << endl;
}
```
## 3.3 引用 (Reference) 

引用本质上是**变量的别名**。它和指针很像，但又完全不同
```c++
类型& 引⽤别名=引⽤对象;
```

```c++
int a = 10;
int b = 20;
int& d = a;
++d;
```
- 引⽤在定义时必须初始化
- ⼀个变量可以有多个引⽤
- **引⽤⼀旦引⽤⼀个实体，再不能引⽤其他实体**(这一点和java不一样)
引用在现阶段好用的地方在于很多地方可以不用传指针，非常方便
```cpp
void Swap(int& ra, int& rb)
{
	int tmp = ra;
	ra = rb;
	rb = tmp;
}
int main()
{
	Swap(a, b);
	cout << a << b << endl;
}
```

### 3.3.1 指针与引用的辩证关系

| 特性 | 指针 (Pointer) | 引用 (Reference) |
| :--- | :--- | :--- |
| **概念** | 存储地址的变量 | 变量的别名 |
| **内存** | 占用独立内存空间 | 不占用独立空间（与原变量共用） |
| **初始化** | 非必须（但建议） | 必须在定义时初始化 |
| **改变指向** | 可以改变（`p = &a` -> `p = &b`） | 不可改变（从一而终） |
| **空值** | 有 `nullptr` | 没有“空引用” |

## 3.4 const 引用

`const` 修饰的引用可以接收临时对象（如表达式结果、类型转换中间量），这在C语言中是无法实现的，也是初学者最容易报错的地方
在 C++ 中，引用本质上是**别名**。既然是别名，就必须遵守一个铁律：**你不能通过别名改变原对象的性质（尤其是权限）。**
如果原对象是“只读”的（常量），你给它起的别名就不能是“可读写”的。
```cpp
const int a = 10;    
const int& rra = a; 
```
示例中`a`是常量，不可修改，是只读的，不能放大权限
### 3.4.1 “临时对象”

```cpp
int x = 10;
int& rx = x * 3; // ❌❌❌❌
```

**计算过程**：`x * 3` 的结果是 `30`。
**存储位置**：这个 `30` 是一个**纯右值**。在计算完成后，编译器需要一个地方暂时存放这个结果，这个“地方”就是**临时对象**。
**生命周期**：临时对象的生命周期非常短，通常在表达式结束时（`;` 之前）就会被销毁。
**常性**：C++ 规定，所有临时对象都具有**常性**（就是 `const` 的）。

`x * 3` 产生了一个 `const int` 类型的临时对象。
-   你试图用 `int&`（非常量引用）去绑定它。
-   这既涉及了**权限放大**（临时对象是 const，试图非 const 引用），又涉及了**生命周期风险**（引用了一个马上要销毁的东西）。

**正确写法：**
```cpp
const int& rx = x * 3; 
```
加上 `const` 后，权限匹配了，而且 C++ 有一个特殊的**“临时对象生命周期延长”**规则：只要 `const` 引用还活着，这个临时对象就不会销毁。

## 3.5 内联函数 (inline) 与 nullptr


*   **inline**：用来替代C语言中容易出错的宏函数（Macro）。它建议编译器在调用处直接展开代码，避免了函数调用的压栈开销，同时保留了类型检查的安全性。
	1. inline对于编译器⽽⾔只是⼀个建议，也就是说，加了inline编译器也可以选择在调⽤的地⽅不展开，不同编译器关于inline什么情况展开各不相同，因为C++标准没有规定这个。inline适⽤于频繁 调⽤的短⼩函数，对于递归函数，代码相对多⼀些的函数，加上inline也会被编译器忽略。
	2. C语⾔实现宏函数也会在预处理时替换展开，但是宏函数实现很复杂很容易出错的，且不⽅便调试，C++设计了inline⽬的就是替代C的宏函数。
	3. 所以一般是在**短小且经常调用的函数**前面用inline修饰
*   **nullptr**：C语言中 `NULL` 通常被定义为 `0`，这导致在函数重载时会产生歧义，例如函数参数传入0和NULL指针。C++引入了 `nullptr` 专门表示空指针，完美解决了这个问题。所以今后写代码空指针就用 <b style="color: red;">nullptr </b>

---

## 相关笔记 / 相关链接

- 前置知识：[[C语言/指针]]、[[C语言/数据在内存中的存取]]
- 进阶：[[C++/类和对象全解：你想知道的这里都有，默认成员函数、友元、隐式类型转换、const和static修饰]]、[[C++/string库]]
- 所属目录：[[C++目录]]、[[C语言目录]]