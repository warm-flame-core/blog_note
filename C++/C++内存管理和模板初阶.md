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
# 一、内存管理
## 1.1 复习一下
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
## 观前提示
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
## 1.2 new关键字
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
## 1.3 delete关键字
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
## 1.4与C语言的比较
1. C++的内存开辟对于类类型有更好的支持，在new对象的时候会自动调用构造函数，同样的delete对象的时候会自动调用析构函数
2. `new`和`delete`本质来也是调用了`malloc`和`free`函数，只不过使用前后还需要析构和构造

## 1.5 operator new与operator delete函数
new和delete是用户进行动态内存申请和释放的操作符，`operator new` 和`operator delete`是系统提供的全局函数，new在底层调用operator new全局函数来申请空间，delete在底层通过 operator delete全局函数来释放空间
观察下面的源码
```cpp
void* __CRTDECL operator new(size_t size) _THROW1(_STD bad_alloc)
{
	// try to allocate size bytes
	void* p;
	while ((p = malloc(size)) == 0)
		if (_callnewh(size) == 0)
		{
			// report no memory
			// 如果申请内存失败了，这里会抛出bad_alloc 类型异常
			static const std::bad_alloc nomem;
			_RAISE(nomem);
		}
	return (p);
}
/*
operator delete: 该函数最终是通过free来释放空间的
*/
void operator delete(void* pUserData)
{
	_CrtMemBlockHeader* pHead;
	RTCCALLBACK(_RTC_Free_hook, (pUserData, 0));
	if (pUserData == NULL)
		return;
	_mlock(_HEAP_LOCK);  /* block other threads */
	__TRY
		/* get a pointer to memory block header */
		pHead = pHdr(pUserData);
	/* verify block type */
	_ASSERTE(_BLOCK_TYPE_IS_VALID(pHead->nBlockUse));
	_free_dbg(pUserData, pHead->nBlockUse);
	__FINALLY
	_munlock(_HEAP_LOCK);  /* release other threads */
	__END_TRY_FINALLY
		return;
}
/*
free的实现
*/
#define free(p) _free_dbg(p, _NORMAL_BLOCK)
```
可以观察到：operator new 实际也是通过malloc来申请空间，如果malloc申请空间成功就直接返回，否则执行用户提供的空间不足应对措施，如果用户提供措施就继续申请，否则就抛异常，**抛异常对于现阶段的我们来说可以先不管，因为设计到多态的知识**
operator delete 最终是通过free来释放空间的。
## 1.6 常见必坑
很多人在从C语言转向C++的时候会犯一些错误，对于内存管理来说，***有以下坑需要注意***
1. 我们可以将C语言的`malloc``calloc``realloc`和`free`函数做一个系列，C++的`new`和`delete`函数做另一个系列，对于同一个变量的开辟和销毁，尽量不混用不同系列函数：因为C系列函数不会调用对应的构造和析构函数，这是一个未定义行为，对于代码审查来说是极度的不规范
2. 连续开辟的变量用连续的销毁，即用了`new[]`那也要用`delete[]`，观察并运行下列代码：
```cpp
class A
{
public:
	A(int a1 = 10, int a2 = 10)
		:_a1(a1)
		, _a2(a2)
	{
		cout << "A():" << this << endl;
	}
	~A()
	{
		cout << "~A():" << this << endl;
		_a1 = 0;
	}
private:
	int _a1;
	int _a2;
};

class B
{
public:
	~B()
	{
		cout << "~B()" << endl;
	}
private:
	int _b1 = 2;
	int _b2 = 2;
};
int main()
{

	B* p2 = new B[10];
	delete p2;
	return 0;
}
```
![[C++/attachments/Pasted image 20260530205943.png]]
会发现本来析构十次的B方法但是析构1次之后程序就崩溃了，如果我们观察以下p2的内存分布，会发现类中显式定义了析构函数，编译器会在实际对象数组的**头部**多开辟 4 个字节的空间，用来记录对象的数量，开辟之后返回变量起始地址，`delete[]`的时候就会向前偏移4个字节找到个数，然后调用对应次数的析构函数，如果用`delete`编译器只会从起始地址开始释放，会造成释放不完全，波坏内存结果，程序崩溃
如果硬要让程序能不崩溃，不显式写析构函数就行，但是这样治标不治本，所以
***严格遵循 `new[]` 配 `delete[]`，`new` 配 `delete`****
# 二、模板
这里讲解的是最基本的用法，后续更高级的用法需要我们学习更多才能了解
## 2.1 烦人的各种类型 
如何实现一个通用的交换函数呢？
```cpp
void Swap(int& left, int& right)
{
	int temp = left;
	left = right;
	right = temp;
}
void Swap(double& left, double& right)
{
	double temp = left;
	left = right;
	right = temp;
}
void Swap(char& left, char& right)
{
	char temp = left;
	left = right;
	right = temp;
}
```
需要对每个类型进行适配，太繁琐了
能否告诉编译器一个模子，让编译器根据不同的类型利用该模子来生成代码呢

## 2.2 函数模板
```cpp
template <typename T1, typename T2,......,typename Tn>
```
```cpp
template <class T1, class T2,......,class Tn>
```
函数模板的基本形式，直接看示例
```cpp
template void Swap( T& left,  T& right) 
{ 
	T temp = left; 
	left = right; 
	right = temp; 
}
```
在编译器编译阶段，对于模板函数的使用，编译器需要根据传入的实参类型来推演生成对应类型的函数以供调用。比如：当用double类型使用函数模板时，编译器通过对实参类型的推演， 将T确定为double类型，然后产生一份专门处理double类型的代码，对于字符类型也是如此
### 2.2.1 使用方法
```cpp
template<class T>
T Add(const T& left, const T& right)
{
	return left + right;
}
```
1. 隐式推导实例化
```cpp
int a1 = 10, a2 = 20; 
double d1 = 10.0, d2 = 20.0; 
Add(a1, a2); 
Add(d1, d2);
```
如果两个参数是不同类型的话可以这样
```cpp
Add(a, (int)d);
```
2. 显式推导实例化
```cpp
Add<int>(a, b);
```
当然这样的写法依旧不能很好的支持不同类型，应该这样写
```cpp
template<class T1, class T2>
T1 Add(const T1& left, const T2& right)
{
	return left + right;
}
```
这样的写法就有以下的调用方法
- Add(2.2, 3.3);
- Add<double, double>(2.2, 3.3);
- Add<double, int>(2.2, 3.3);

### 2.2.2 匹配原则
- 一个非模板函数可以和一个同名的函数模板同时存在，而且该函数模板还可以被实例化为这 个非模板函数，即`int Add(int left, int right)`和`template T Add(T left, T right)`可以同时存在
- 对于非模板函数和同名函数模板，如果其他条件都相同，在调动时会优先调用非模板函数而 不会从该模板产生出一个实例。如果模板可以产生一个具有更好匹配的函数， 那么将选择模板
## 2.3 类模板
### 2.3.1 基本形式
```cpp
template<class T1, class T2, ..., class Tn>
class 类模板名
{
	// 类内成员定义
};
```
### 2.3.2 举例
直接看例子
```cpp
template<typename T>
class Stack
{
public:
	Stack(int n = 4)
		:_arr(new T[n])
		,_top(0)
		,_capacity(n)
	{}
	~Stack()
	{
		delete[] _arr;
		_arr = nullptr;
		_top = _capacity = 0;
	}
private:
	T* _arr;
	size_t _top;
	size_t _capacity;
};
```
这是一个通用型的栈结构可以储存各种类型的数据的栈
### 2.3.3 实例化
类模板实例化一定要显式推导类型
```cpp
Stack<int> st1;
Stack<double> st1;
```

---

## 相关笔记 / 相关链接

- 相关目录：[[C++目录]]
- 相关笔记：[[C++/类和对象全解：你想知道的这里都有，默认成员函数、友元、隐式类型转换、const和static修饰]]（前置：构造析构与对象）
