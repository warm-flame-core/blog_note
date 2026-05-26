---
tags:
  - 编程
aliases:
  - 动态开辟
日期: 2026年5月26日
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
在之前C语言学习阶段我们已经学过了动态内存开辟，即`malloc``calloc``realloc`和`free`，但是在C++阶段，这些函数并不能很好的支持类相关的内存开辟，所以接下来介绍一下C++的内存管理
# 复习一下
```cpp
int globalVar = 1;
static int staticGlobalVar = 1;
void Test()
{
	static int staticVar = 1;
	int localVar = 1;
	int num1[10] = { 1, 2, 3, 4 };
	char char2[] = "abcd";
	const char* pChar3 = "abcd";
	int* ptr1 = (int*)malloc(sizeof(int) * 4);
	int* ptr2 = (int*)calloc(4, sizeof(int));
	int* ptr3 = (int*)realloc(ptr2, sizeof(int) * 4);
	free(ptr1);
	free(ptr3);
}
```
A.栈  B.堆  C.数据段(静态区)  D.代码段(常量区)
1. globalVar在哪里？____
2. staticGlobalVar在哪里？____
3. staticVar在哪里？____
4. localVar在哪里？____
5. num1 在哪里？____
6. char2在哪里？____
7. char2在哪里？___
8. pChar3在哪里？____
9. pChar3在哪里？____
10. ptr1在哪里？____
11. ptr1在哪里？____

***需要注意的是指针变量存在栈区，是指针指向的动态开辟的东西存在堆区***，这是很多人会弄错的地方
![[C++/attachments/Pasted image 20260526212901.png]]
数据段--存储全局数据和静态数据
代码段--可执行的代码/只读常量
# 观前提示
本文示例中需要用到一个类
```cpp
class A
{
public:
	A(int a1 = 0,int a2 = 0)
		: _a1(a1)
		, _a2(a2)
	{
		cout << "A():" << this << endl;
	}
	~A()
	{
		cout << "~A():" << this << endl;
	}
private:
	int _a1;
	int _a2;
};
```
# new关键字
C++中`new`取代了C语言中`malloc``calloc``realloc`的功能，其使用方法看示例
1. 开辟内置类型
```cpp
// 动态申请一个int类型的空间 
int* ptr4 = new int; 
// 动态申请一个int类型的空间并初始化为10 
int* ptr5 = new int(10); 
// 动态申请10个int类型的空间 
int* ptr6 = new int[3];
```
2. 开辟自定义类型
```cpp
A* p1 = new A(1);
A* p2 = new A[10];
A* p3 = new A[10](1, 2, 3, 4, 5, 6, 7, 8, 9, 0);
A* p4 = new A[10]({1,1},{2,2}, { 3,3 }, { 4,4 }, { 5,5 }, { 6,6 }, { 7,7 }, { 8,8 }, { 9,9 }, {  });
```
连续初始化的时候如果是传单参数可以用圆括号，在C++11之后多参数可以用花括号进行初始化
# delete关键字
C++中`new`取代了C语言中`free`的功能，其使用方法看示例
1. 释放内置类型
```cpp
int* ptr4 = new int;
int* ptr5 = new int(10);
int* ptr6 = new int[3];
delete ptr4;
delete ptr5;
delete[] ptr6;
```
***需要注意的是连续开辟的销毁需要用`delete[]`***，这非常重要，一定一定不能用`delete`
2. 释放自定义类型
```cpp
A* p2 = new A(1);
delete p2;
int* p4 = new int;
delete p4;
A* p6 = new A[10];
delete[] p6;
```
其使用方法和内置类型一样
# 与C语言的比较
1. C++的内存开辟对于类类型有更好的支持，在new对象的时候会自动调用构造函数，同样的delete对象的时候会自动调用析构函数
2. 对