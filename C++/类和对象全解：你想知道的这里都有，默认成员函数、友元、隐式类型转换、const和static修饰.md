---
tags:
  - 形态/概念
aliases:
  - 方法
日期: 2026年5月17日
---
类和对象全解，你想知道的这里都有

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
作为C语言的后继，C++扩展了很多语法，写编程的时候就非常方便，其中一大特色就是类和对象，不仅扩展了结构体的语法，这实现了封装，让属性管理更加有道理。
<p align="center" style="color: red; font-weight: bold;">注意：本篇文章有对C++类和对象的解析，需要读者有一定C语言基础和C++入门，且类和对象初学时非常难理解，文章如果有误恳请读者见谅</p>

# 一、语法形式
## 1.1 基本定义
其语法与C语言的结构体`struct`是一样的，`class`为定义类的关键字，后面接的为类的名字，`{}`中为类的主体，注意类定义结束时后⾯分号不能省略，例如：
```cpp
class A
{
	void print()
	{
		cout << _a << endl;
	}
	int _a;
};
```
这个类叫A类或者是A方法，其中`_a`叫成员变量。C++扩展了结构体的用法，类里面也可以写函数了，叫做这个类的成员函数。
为了区分成员变量，⼀般习惯上成员变量会加⼀个特殊标识，如成员变量前⾯或者后⾯加_或者m 开头，注意C++中这个并不是强制的，只是⼀些惯例，具体看公司的要求
需要注意的是：
1. C++中struct也可以定义类，C++兼容C中struct的⽤法，同时struct升级成了类，明显的变化是 struct中可以定义函数，⼀般情况下我们还是推荐⽤class定义类
```cpp
struct A
{
	void print()
	{
		cout << _a << endl;
	}
	int _a;
};
```
2. 定义在类⾯的成员函数默认为inline，也就说在调用处编译器会展开函数
## 1.2 访问限定符
1. C++⼀种实现封装的⽅式，⽤类将对象的属性与⽅法结合在⼀块，让对象更加完善，通过访问权限 选择性的将其接⼝提供给外部的⽤⼾使⽤，其中访问限定符有三种：`public`、`protected`和`private`
2. public修饰的成员在类外可以直接被访问；protected和private修饰的成员在类外不能直接被访 问，protected和private是⼀样的，以后继承章节才能体现出他们的区别，例如对于上例，可以把`_a`藏起来不能随便访问，要访问就要通过类的方法进行访问
<span id="my-point"></span>
```cpp
class A
{
public:
	void print()
	{
		cout << _a << endl;
	}
	int Get()
	{
		return _a;
	}
private:
	int _a;
};
```
这样的好处就是可以封装一些数据，对数据进行规范管理，一般习惯上成员变量都会被限制为private/protected，需要给别⼈使⽤的成员函数会放为public
3. 访问权限作⽤域从该访问限定符出现的位置开始直到下⼀个访问限定符出现时为⽌，如果后⾯没有 访问限定符，作⽤域就到}即类结束，且class定义成员没有被访问限定符修饰时默认为private，struct默认为public
### 1.2.3 类域
当我们定义了一个类，那么和命名空间一样就会产生一个新的作用范围，对于[A类](#my-point)，在类体外定义成员时，需要使⽤::作 ⽤域操作符指明成员属于哪个类域
```cpp
class A
{
public:
	void print()
	{
		cout << _a << endl;
	}
	void free();
	int Get()
	{
		return _a;
	}
private:
	int _a;
};

void A::free()
{
	_a = 0;
}

int main()
{
	A a1;
	a1.free();
	return 0;
}
```
[^1]
## 1.3 实例化
对于一个创建好的类，我们可以创建出很多的变量，例如上例里面A类：
```cpp
A a1, a2;
```
类就像一个设计图纸，根据图纸可以建造出很多房子
### 1.3.1 对象大小
现在的类可以放函数进去，那么就有一个问题，这个类的大小是多少？
类实例化出的每个对象，都有独⽴的数据空间，所以对象中肯定包含 成员变量，那么成员函数是否包含呢？⾸先函数被编译后是⼀段指令，对象中没办法存储，这些指令 存储在⼀个单独的区域(代码段)，那么对象中⾮要存储的话，只能是成员函数的指针。再分析⼀下，对象中是否有存储指针的必要呢，A实例化a1和a2两个对象，d1和d2都有各⾃独⽴的成员变量 `_a`存储各⾃的数据，但是a1和a2的成员函数free/Print指针却是⼀样的，存储在对象中就浪费了。如果⽤A实例化100个对象，那么成员函数指针就重复存储100次，太浪费了。所以对象的大小只需要看成员变量，计算规则和结构体计算规则是一样的

## 1.4 this指针
上面我们知道，对象是不会存函数的，那么函数是怎么知道自己用的是a1的数据还是a2的数据呢？
1. 编译器编译后，类的成员函数默认都会在形参第⼀个位置，增加⼀个当前类类型的指针，叫做this 指针，例如对于A类其两个函数原型print/free应该是`void free(Date* const this)和void print(Date* const this)`
2. 类的成员函数中访问成员变量，本质都是通过this指针访问的，如free函数给_a置于0可以写成`this->_a = 0`
3. C++规定不能在实参和形参的位置显⽰的写this指针(编译时编译器会处理)，但是可以在函数体内显示使⽤this指针
```cpp
#include<iostream>
using namespace std;
class Date
{
public:
	// void Init(Date* const this, int year, int month, int day)
	void Init(int year, int month, int day)
	{
			_year = year;
		this->_month = month;
		this->_day = day;
	}
	void Print()
	{
		cout << _year << "/" << _month << "/" << _day << endl;
	}
private:
	int _year;
	int _month;
	int _day;
};
int main()
{
	Date d1;
	Date d2;
	d1.Init(2024, 3, 31);
	d1.Print();
	d2.Init(2024, 7, 5);
	d2.Print();
	return 0;
}
```
让我们看看下面两段代码：
```cpp
#include<iostream>
using namespace std;
class A
{
public:
	void Print()
	{
		cout << "A::Print()" << endl;
	}
private:
	int _a;
};
int main()
{
	A* p = nullptr;
	p->Print();
	return 0;
}
```
<br>
```cpp
#include<iostream>
using namespace std;
class A
{
public:
	void Print()
	{
		cout << "A::Print()" << endl;
		cout << _a << endl;
	}
private:
	int _a;
};
int main()
{
	A* p = nullptr;
	p->Print();
	return 0;
}
```
其中第一段代码，哪怕把空指针穿给了打印函数，传入的this指针是空指针，但是打印函数没有任何指针解引用的操作，所以是正常运行
对于第二段代码，打印函数的`cout << _a << endl;`实际上是`cout << this->_a << endl;`，this指针是空指针，对空指针进行解引用程序会报错
# 二、六大默认成员函数
在讲解六大默认成员函数之前，我们要知道两个类，一个是**日期类**，一个是**栈类**
```cpp
class Date
{
private:
	int _year;
	int _month;
	int _day;
};
typedef int STDataType;
class Stack
{
private:
	STDataType* _a;
	size_t _capacity;
	size_t _top;
};
```
接下来的所有举例都在这些类上举例
<br>
默认成员函数就是⽤⼾没有显式实现，编译器会⾃动⽣成的成员函数称为默认成员函数。⼀个类，我们不写的情况下编译器会默认⽣成以下6个默认成员函数，需要注意的是这6个中最重要的是前4个，最 后两个取地址重载不重要，我们稍微了解⼀下即可。其次就是C++11以后还会增加两个默认成员函数， 移动构造和移动赋值，这个我们后⾯再讲解。默认成员函数很重要，也⽐较复杂，我们要从两个⽅⾯去学习：
- 我们不写时，编译器默认⽣成的函数⾏为是什么，是否满⾜我们的需求
- 编译器默认⽣成的函数不满⾜我们的需求，我们需要⾃⼰实现，那么如何⾃⼰实现？

| 初始化和清理 | 构造函数主要完成初始化工作                 |
| :----- | :---------------------------- |
|        | 析构函数主要完成清理工作                  |
| 拷贝复制   | 拷贝构造是使用同类对象初始化创建对象            |
|        | 赋值重载主要是把一个对象赋值给另一个对象          |
| 取地址重载  | 主要是普通对象和const对象取地址，这两个很少会自己实现 |
## 2.1 构造函数
构造函数是特殊的成员函数，需要注意的是，构造函数虽然名称叫构造，但是构造函数的主要任务并不是开空间创建对象(我们常使⽤的局部对象是栈帧创建时，空间就开好了)，⽽是对象实例化时初始化对象。构造函数的本质是要替代我们数据结构初阶中栈和队列的Init函数的功能，构造函数⾃动调⽤的特点就完美的替代的了Init
1. 函数名与类名相同 
2. ⽆返回值。(返回值啥都不需要给，也不需要写void，不要纠结，C++规定如此) 
3. 对象实例化时系统会⾃动调⽤对应的构造函数
4. 构造函数可以重载
5. 如果类中没有显式定义构造函数，则C++编译器会⾃动⽣成⼀个⽆参的默认构造函数，⼀旦⽤⼾显 式定义编译器将不再⽣成
对于日期类，我们有一下几种构造函数的写法：
```cpp
// 1.无参构造函数
Date()
{
	_year = 1;
	_month = 1;
	_day = 1;
}
// 2.带参构造函数
Date(int year, int month, int day)
{
	_year = year;
	_month = month;
	_day = day;
}
// 3.全缺省构造函数
Date(int year = 1, int month = 1, int day = 1)
{
	_year = year;
	_month = month;
	_day = day;
}
```
6. **⽆参构造函数、全缺省构造函数、我们不写构造时编译器默认⽣成的构造函数**，都叫做默认构造函数。但是这三个函数有且只有⼀个存在，不能同时存在。⽆参构造函数和全缺省构造函数虽然构成函数重载，但是调⽤时会存在歧义。要注意很多同学会认为默认构造函数是编译器默认⽣成那个叫 默认构造，实际上⽆参构造函数、全缺省构造函数也是默认构造
	-  总结⼀下就是不传实参就可以调⽤的构造就叫默认构造
	  在本例日期类中，`Date()`和`Date(int year = 1, int month = 1, int day = 1)`就不能同时出现
	- 另外注意如果写了**无参数构造函数，那么再实例化对象的时候就不能加括号**（如：`Date d1();`）这样的写法是有歧义的，因为编译会不清楚到底是区分这⾥是函数声明还是实例化对象，会报警告
	  > warning C4930: “Date d1(void)”: 未调⽤原型函数 ( 是否是有意⽤变量定义的 ?)
	
	- 如果上例的日期类的构造只保留第二个带参数的构造函数，那么在实例化对象的时候依旧会出错：`error C2512: “Date”: 没有合适的默认构造函数可用`，因为已经写了构造函数，编译器不会再生成，但是带参数构造函数不是默认构造，而类的实例化会调用默认构造函数，所以会出错
7. 我们不写，编译器默认⽣成的构造，对内置类型成员变量的初始化没有要求，也就是说是是否初始化是不确定的，看编译器。对于⾃定义类型成员变量，要求调⽤这个成员变量的默认构造函数初始 化。如果这个成员变量，没有默认构造函数，那么就会报错，我们要初始化这个成员变量，需要⽤初始化列表才能解决，初始化列表见后文有讲解

## 2.2 析构函数
析构函数与构造函数功能相反，析构函数不是完成对对象本⾝的销毁，⽐如局部对象是存在栈帧的， 函数结束栈帧销毁，他就释放了，不需要我们管，C++规定对象在销毁时会⾃动调⽤析构函数，完成对象中资源的清理释放⼯作。析构函数的功能类⽐我们之前Stack实现的Destroy功能，⽽像Date没有Destroy，其实就是没有资源需要释放，所以严格说Date是不需要析构函数的
1. 析构函数名是在类名前加上字符~。 
2. ⽆参数⽆返回值。(这⾥跟构造类似，也不需要加void) 
3. ⼀个类只能有⼀个析构函数。若未显式定义，系统会⾃动⽣成默认的析构函数。
4. 对象⽣命周期结束时，系统会⾃动调⽤析构函数
	对于`Stack`类
```cpp
Stack(int n = 4)
{
	_a = (STDataType*)malloc(sizeof(STDataType) * n);
	if (nullptr == _a)
	{
		perror("malloc申请空间失败");
		return;
	}
	_capacity = n;
	_top = 0;
}
~Stack()
{
	cout << "~Stack()" << endl;
	free(_a);
	_a = nullptr;
	_top = _capacity = 0;
}
```
5. 跟构造函数类似，我们不写编译器⾃动⽣成的析构函数对内置类型成员不做处理，⾃定类型成员会调⽤他的析构函数。
	- 对于两个栈实现队列，现在知识可以让我们不需要写`MyQueue`的构造函数和析构函数，而且在之后使用的时候，return之前也不需要再管有没有释放，编译器会自动调用对应的析构函数，***这是十分方便的***
```cpp
class MyQueue
{
public:
private:
	Stack pushst;
	Stack popst;
};
int main()
{
	Stack st;
	MyQueue mq;
	return 0;
}
```
6. 还需要注意的是我们显⽰写析构函数，对于⾃定义类型成员也会调⽤他的析构，也就是说⾃定义类 型成员⽆论什么情况都会⾃动调⽤析构函数
## 2.3 拷贝构造函数
前置补充：***C++ 规定⾃定义类型对象进⾏拷⻉⾏为必须调⽤拷⻉构造，传值传参和传值返回也要调用拷贝构造***
如果⼀个构造函数的第⼀个参数是⾃⾝类类型的引⽤，且任何额外的参数都有默认值，则此构造函数也叫做拷⻉构造函数，也就是说拷⻉构造是⼀个特殊的构造函数，本质就是对类变量进行复制
1. 拷⻉构造函数是构造函数的⼀个重载。
2. 拷⻉构造函数的第⼀个参数必须是类类型对象的引⽤，
	- 使⽤传值⽅式编译器直接报错，因为语法逻辑上会引发⽆穷递归调⽤，因为传值会调用构造函数，但是新的构造函数依旧是传值传参，又会调用新的构造，一直这样递归下去
	- 拷⻉构造函数也可以多个参数，但是第⼀个参数必须是类类型对象的引⽤，后⾯的参数必须有缺省值。
	- 对于日期类的拷贝：
```cpp
Date(Date* d) 
{
	_year = d->_year; 
	_month = d->_month; 
	_day = d->_day; 
}
int main()
{
	Date d1(1, 1, 1);
	Date d3(d1);
}
```
3. C++规定⾃定义类型对象进⾏拷⻉⾏为必须调⽤拷⻉构造，所以⾃定义类型传值传参和传值返回都会调⽤拷⻉构造完成 
4. 若未显式定义拷⻉构造，编译器会⽣成⾃动⽣成拷⻉构造函数。⾃动⽣成的拷⻉构造对内置类型成 员变量会完成值拷⻉/浅拷⻉(⼀个字节⼀个字节的拷⻉)[^2]，对⾃定义类型成员变量会调⽤他的拷⻉构造，深拷贝的目的在于在进行指针类型操作的时候需要单独开一块空间，而浅拷贝并不会开空间
对于`Stack`类：
```cpp
typedef int STDataType;
class Stack
{
public:
	Stack(int n = 4)
	{
		_a = (STDataType*)malloc(sizeof(STDataType) * n);
		if (nullptr == _a)
		{
			perror("malloc申请空间失败");
			return;
		}
		_capacity = n;
		_top = 0;
	}
	Stack(const Stack& st)
	{
		_a = (STDataType*)malloc(sizeof(STDataType) * st._capacity);
		if (nullptr == _a)
		{
			perror("malloc申请空间失败!!!");
			return;
		}
		memcpy(_a, st._a, sizeof(STDataType) * st._top);
		_top = st._top;
		_capacity = st._capacity;
	}
	void Push(STDataType x)
	{
		if (_top == _capacity)
		{
			int newcapacity = _capacity * 2;
			STDataType* tmp = (STDataType*)realloc(_a, newcapacity *
				sizeof(STDataType));
			if (tmp == NULL)
			{
				perror("realloc fail");
				return;
			}
			_a = tmp;
			_capacity = newcapacity;
		}
		_a[_top++] = x;
	}
	~Stack()
	{
		cout << "~Stack()" << endl;
		free(_a);
		_a = nullptr;
		_top = _capacity = 0;
	}
private:
	STDataType* _a;
	size_t _capacity;
	size_t _top;
};
class MyQueue
{
public:
private:
	Stack pushst;
	Stack popst;
};
```
如果不显式写stack的拷贝构造调用以下代码
```cpp
Stack st1;
Stack st2 = st1;
st1.Push(1);
st2.Push(2);
```
这段代码在return的时候会崩溃，因为隐式的拷贝构造让st1和st2指向了同一块空间，同一块空间释放了两次
如果显式写stack的拷贝构造调用以下代码
```cpp
Stack st2 = st1;
MyQueue mq1;
MyQueue mq2 = mq1;
```
MyQueue⾃动⽣成的拷⻉构造，会⾃动调⽤Stack拷⻉构造完成pushst / popst的拷⻉，只要Stack拷⻉构造⾃⼰实现了深拷⻉，他就没问题

## 2.4 赋值运算符重载
再讲赋值运算符重载之前，我们要先了解一下运算符重载作为前置知识
### 2.4.1 运算符重载
C++扩展了运算符，支持自定义运算符的逻辑，在C语言我们不可以进行结构体的比较，但是在C++就可以
运算符重载是一个特殊的函数，所以形式和函数一样，有返回值和参数和函数体，但是要注意函数名是operator加上要定义的运算符，比如在日期类中如果要判断两个日期相等：
```cpp
bool operator==(const Date& d1, const Date& d2)
{
	return d1._year == d2._year
		&& d1._month == d2._month
		&& d1._day == d2._day;
}
```
但是对于这个定义在类外面的函数，因为有访问限定符是无法访问的，所以有以下几种解决方法：
1. 成员放公有
2. Date 提供 getxxx 函数
3. 友元函数
4. 重载为成员函数
一般来说会重载成成员函数，写在类里面：
```cpp
bool operator==(const Date& d) 
{ 
	return _year == d._year && _month == d._month && _day == d._day; 
}
```
如果是重载成员函数，因为类的函数的第一个参数默认是this指针，对于单目运算符不需要传参数，对于双目运算符则只需要传一个参数
对于运算符的调用就和内置类型使用运算符是一样的，即：
```cpp
d1 == d2;
d1.operator==(d2);
```
以上这两种调用方式都可以
需要注意的是：
- 如果⼀个重载运算符函数是成员函数，则它的第⼀个运算对象默认传给隐式的this指针，因此运算 符重载作为成员函数时，参数⽐运算对象少⼀个
- 运算符重载以后，其优先级和结合性与对应的内置类型运算符保持⼀致。
- 不能通过连接语法中没有的符号来创建新的操作符：⽐如operator@
- [^3]`.*` `::` `sizeof` `?:`这5个运算符不能重载
- 重载操作符⾄少有⼀个类类型参数，不能通过运算符重载改变内置类型对象的含义，如： 
>operator+(int x, int y)

- 重载++运算符时，有前置++和后置++，运算符重载函数名都是operator++，⽆法很好的区分。 C++规定，后置++重载时，增加⼀个int形参，跟前置++构成函数重载，⽅便区分，对于日期类，来说应该这样写
```cpp
Date& Date::operator++()
{
	*this += 1; 
	return *this; 
}
Date Date::operator++(int)
{
	Date tmp = *this;
	*this+=1;
	return tmp;
}
```
### 2.4.2 赋值运算符重载
赋值运算符重载是⼀个默认成员函数，⽤于完成两个已经存在的对象直接的拷⻉赋值，这⾥要注意跟拷⻉构造区分，拷⻉构造⽤于⼀个对象拷⻉初始化给另⼀个要创建的对象
1. 赋值运算符重载是⼀个运算符重载，规定必须重载为成员函数。赋值运算重载的参数建议写成 const 当前类类型引⽤，否则会传值传参会有拷⻉
2. 有返回值，且建议写成当前类类型引⽤，引⽤返回可以提⾼效率，有返回值⽬的是为了⽀持连续赋值场景。 
3. 没有显式实现时，编译器会⾃动⽣成⼀个默认赋值运算符重载，默认赋值运算符重载⾏为跟默认拷 ⻉构造函数类似，对内置类型成员变量会完成值拷⻉/浅拷⻉(⼀个字节⼀个字节的拷⻉)，对⾃定义类型成员变量会调⽤他的赋值重载函数
对于日期实现重载：
```cpp
Date& operator=(const Date& d) 
{
	_year = d._year; 
	_month = d._month; 
	_day = d._day;
	return *this;
}
```
## 2.5 日期类的实现
对于日期来说，我们要实现比较和赋值，还有前置和后置的加减，且定义和实现分离，参考代码如下：
头文件：
```cpp
#pragma once
#include <iostream>
using namespace std;
class Date
{
    friend ostream& operator<<(ostream& out, const Date& d);
    friend istream& operator>>(istream& out, Date& d);
public:
	Date(int year = 1, int mouth = 1, int day = 1);
	void print();
    int GetMouthDay(int year, int mouth)
    {
        static int arr[13] = { -1,31,28,31,30,31,30,31,31,30,31,30,31 };
        if (mouth == 2 && ((year % 4 == 0 && year % 100 != 0) || year % 400 == 0))
        {
            return 29;
        }
        return arr[mouth];
    }
    bool CheckDate();
    bool operator<(const Date& d) const;
    bool operator<=(const Date& d) const;
    bool operator>(const Date& d) const;
    bool operator>=(const Date& d) const;
    bool operator==(const Date& d) const;
    bool operator!=(const Date& d) const;
    // d1 += 天数

    Date& operator+=(int day);
    Date operator+(int day) const;
    // d1 -= 天数

    Date& operator-=(int day);
    Date operator-(int day) const;
    // d1 - d2
    int operator-(const Date& d) const;
    // ++d1 -> d1.operator++()
    Date& operator++();
    // d1++ -> d1.operator++(0)
    Date operator++(int);
    Date& operator--();
    Date operator--(int);
private:
	int _year;
	int _mouth;
	int _day;
};

ostream& operator<<(ostream& out, const Date& d);
istream& operator>>(istream& out, Date& d);
```
源文件：
```cpp
#define _CRT_SECURE_NO_WARNINGS 1
#include "Date.h"
Date::Date(int year, int mouth, int day)	// 缺省值在定义的时候显式
{
	_year = year;
	_mouth = mouth;
	_day = day;

	if (!(CheckDate()))
	{
		cout << "日期非法：";
		print();
	}
}

void Date::print()
{
	cout << _year << "/" << _mouth << "/" << _day << endl;
}

bool Date::CheckDate()
{
	if (_mouth < 1 || _mouth > 12|| _day < 1 || _day > GetMouthDay(_year, _mouth))
	{
		return false;
	}
	else
	{
		return true;
	}
}
bool Date::operator<(const Date& d) const
{
	if (_year < d._year)
	{
		return true;
	}
	else if (_year == d._year)
	{
		if (_mouth < d._mouth)
			return true;
		else if (_mouth == d._mouth)
		{
			return _day < d._day;
		}
	}
	return false;
}
bool Date::operator<=(const Date& d) const
{
	return *this < d || *this == d;
}
bool Date::operator>(const Date& d) const
{
	return !(*this <= d);
}
bool Date::operator>=(const Date& d) const
{
	return !(*this < d);
}
bool Date::operator==(const Date& d) const
{
	return _year == d._year
		&& _mouth == d._mouth
		&& _day == d._day;
}
bool Date::operator!=(const Date& d) const
{
	return !(*this == d);
}

// 效率更高，让+去复用+=的功能
// 传值返回，因为需要带出tmp，tmp是局部变量出作用域会销毁，需要拷贝构造

//+喜欢克隆，其他运算符重载如果调用+很容易出现局部变量出作用域销毁，尽量少用+去复用其他运算符
Date Date::operator+(int day) const
{
	Date tmp = *this;	// 调用拷贝构造
	return tmp += day;
}
// 传引用返回，因为this指针所指向的是作用域外的变量，传引用返回减少拷贝
Date& Date::operator+=(int day)
{

	_day += day;
	while (_day > Date::GetMouthDay(_year, _mouth))
	{
		_day -= Date::GetMouthDay(_year, _mouth);
		++_mouth;
		if (_mouth == 13)
		{
			_mouth = 1;
			++_year;
		}
	}
	return *this;
}

Date& Date::operator-=(int day)
{
	_day -= day;
	while (_day <= 0)
	{
		--_mouth;
		if (_mouth == 0)
		{
			_mouth = 12;
			--_year;
		}
		_day += Date::GetMouthDay(_year, _mouth);
	}
	return *this;
}
Date Date::operator-(int day) const
{
	Date tmp = *this;
	tmp -= day;
	return tmp;
}

int Date::operator-(const Date& d) const
{
	// 假设法，如果假设正确是大-小，那么返回整数，如果假设错误，返回差值的负数
	int flag = 1;
	Date max = *this;
	Date min = d;
	if (min > max)
	{
		max = d;
		min = *this;
		flag = -1;
	}
	int n = 0;
	while (min != max)
	{
		++n;
		++min;
	}
	return n * flag;
}

Date& Date::operator++()
{
	*this += 1; 
	return *this; 
}
Date Date::operator++(int)
{
	Date tmp = *this;
	*this+=1;
	return tmp;
}

Date& Date::operator--()
{
	*this -= 1;
	return *this;
}
Date Date::operator--(int)
{
	Date tmp = *this;
	*this -= 1;
	return tmp;
}

ostream& operator<<(ostream& out, const Date& d)
{
	cout << d._year << "年" << d._mouth << "月" << d._day << "日" << endl;
	return out;
}
istream& operator>>(istream& in, Date& d)
{
	while (1)
	{
		cout << "请依次输入日期:>";
		in >> d._year >> d._mouth >> d._day;
		if (!(d.CheckDate()))
		{
			cout << "日期无效,重新输入" << endl;
		}
		else
			break;
	}
	return in;
}
```
需要解释一下：
1. 让+去复用+=的功能是为了效率更高，如果是+=去复用+的功能参考代码如下：
```cpp
Date Date::operator+(int day)
{
	Date tmp = *this;	//调用拷贝构造
	tmp._day += day;
	while (tmp._day > Date::GetMouthDay(tmp._year, tmp._mouth))
	{
		tmp._day -= Date::GetMouthDay(tmp._year, tmp._mouth);
		++tmp._mouth;
		if (tmp._mouth == 13)
		{
			tmp._mouth = 1;
			++tmp._year;
		}
	}
	return tmp;		// 传值返回调用拷贝构造
}
Date& Date::operator+=(int day)
{
	*this = *this + day;	// 等于操作调用赋值重载
	return *this;
}
```
这样的写法效率比较低,比让+去复用+=多一次拷贝和赋值。且对于效率高的写法在+的重载中一定是传值返回，因为创建了临时变量
2. 流插入和流提取用到了友元函数，这个知识在后续讲解
## 2.6 取地址运算符重载
在讲解取地址运算符重载之前我们要一点前置知识：对于一个指针，如果我们定义一个指针:
1. 在`*`后面加上const，表明这个指针不能被修改，即不能指向其他东西，如`Date* const this`
2. 在`*`前面加上const，表明这个指针所指向的内容不能被修改，即不能解引用后进行修改，如`const Date* this`
3. 在`*`前后面加上const，表明这个指针是只读的，即不能修改指向也不能修改对应指向的内容，如`const Date* const this`
### 2.6.1 const成员函数
将const修饰的成员函数称之为const成员函数，const修饰成员函数放到成员函数参数列表的后⾯，对于日期类，如果我们想要打印日期：`void Print() const`，实际上函数参数应该是`void Print(const Date* const this) const`
const实际修饰该成员函数隐含的this指针，表明在该成员函数中不能对类的任何成员进⾏修改。 const 修饰Date类的Print成员函数
### 2.6.2 取地址运算符重载
这个的重载分为普通的和const修饰的，也就是说自己显式的时候需要写两个，对于日期类来说：
```cpp
Date* operator&() 
{ 
	return this; // return nullptr; 
} 
const Date* operator&()const 
{ 
	return this; // return nullptr;
}
```
这个重载我们很少自己实现，因为编译器自动生成的也可以帮我们做好这件事，如果是想让同事用你的类的时候一定出错，可以写上面代码块中注释的部分，当调用的时候就会胡乱返回地址
# 三、类和对象详细理解
## 3.1 构造函数深入理解
之前我们实现构造函数时，初始化成员变量主要使⽤函数体内赋值，构造函数初始化还有⼀种⽅ 式，就是初始化列表，初始化列表的使⽤⽅式是以⼀个冒号开始，接着是⼀个以逗号分隔的数据成 员列表，每个"成员变量"后⾯跟⼀个放在括号中的初始值或表达式，对于日期类来说
```cpp
Date(int year = 1, int month = 1, int day = 1)
	:_year(year)
	,_month(month)
	,_day(day)
{}
```
关于初始化列表需要注意以下几点：
- 每个成员变量在初始化列表中只能出现⼀次，语法理解上初始化列表可以认为是每个成员变量定义初始化的地⽅，一个变量需要定义和初始化，可以认为在类里面声明了，就要在类里面初始化
- ***引⽤成员变量，const成员变量，没有默认构造的类类型变量，必须放在初始化列表位置进⾏初始化***，否则会编译报错，如果日期类里面还有一个Time的类、ref的引用，缺少默认构造函数的类，运行会有以下错误
> error C2512: “Time”: 没有合适的默认构造函数可⽤
> error C2530 : “Date::_ref” : 必须初始化引⽤ 
> error C2789 : “Date::_n” : 必须初始化常量限定类型的对象
- C++11⽀持在成员变量声明的位置给缺省值，这个缺省值主要是给没有显⽰在初始化列表初始化的成员使⽤的，对于日期类
```cpp
class Date
{
public:
	Date(int year = 1, int month = 1, int day = 1)
		:_year(year)
		,_month(month)
		,_day(day)
	{
		_year = year;
		_month = month;
		_day = day;
	}
private:
	int _year = 2;
	int _month = 3;
	int _day = 4;
};
```
		a. 其中缺省值和初始化列表的括号中可以是表达式，例如在stack类中可以先给数组开空间
		b. 当有初始化列表时缺省值会被忽略，如果构造函数内部还有赋值操作那么成员的值会覆盖初始化列表的值
- 初始化列表中按照成员变量在类中声明顺序进⾏初始化，跟成员在初始化列表出现的的先后顺序⽆关。建议声明顺序和初始化列表顺序保持⼀致
***总结一下***：
![[C++/attachments/Pasted image 20260518203129.png]]
## 3.2 类的隐式类型转换
C++支持对类类型的隐式类型转换，相当于自动将整形转换为类类型，前提是需要相关内置类型为参数的构造函数
其中如果不希望这样的类型转换，那么在构造函数之前用`explicit`修饰就可以
```cpp
#include<iostream>
using namespace std;
class A
{
public:
	A(int a1)
		:_a1(a1)
	{}
	A(int a1, int a2)
		:_a1(a1)
		, _a2(a2)
	{}
	void Print()
	{
		cout << _a1 << " " << _a2 << endl;
	}
	int Get() const
	{
		return _a1 + _a2;
	}
private:
	int _a1 = 1;
	int _a2 = 2;
};
class B
{
public:
	B(const A& a)
		:_b(a.Get())
	{}
private:
	int _b = 0;
};
int main()
{
	A aa1 = 1;
	aa1.Print();
	const A& aa2 = 1;
	A aa3 = { 2,2 };
	B b = aa3;
	const B& rb = aa3;
	return 0;
}
```
在C++11之后就可以转换多参数，使用的时候加上大括号就行，`A aa3 = { 2,2 };`
## 3.3 static成员函数
工程上我们会使用一些变量是共有，比如几个商品共同属于一个超市，在C++里我们可以用**static**修饰成员变量，这样无论实例化多少个对象，对象内都会有相同的变量，同样的，也可以用static修饰函数
- ⽤static修饰的成员函数，称之为静态成员函数，静态成员函数没有this指针
- 静态成员函数中可以访问其他的静态成员，但是不能访问⾮静态的，因为没有this指针
- ⾮静态的成员函数，可以访问任意的静态成员变量和静态成员函数
- 静态成员也是类的成员，受public、protected、private访问限定符的限制
- 静态成员变量不能在声明位置给缺省值初始化，因为缺省值是个构造函数初始化列表的，静态成员变量不属于某个对象，不⾛构造函数初始化列表
我们用一道题理解一下：
![[C++/attachments/Pasted image 20260518204624.png]]
这道题无法使用递归，因为返回条件判断需要条件判断语句，也不能用加减法，那我们可以这样想：
  利用C++的**类构造函数**，每次创建对象时自动调用构造函数，通过静态变量累加结果。将循环逻辑转化为对象的创建次数（`n`次）。  
1. 定义一个辅助类Sum，包含静态变量i（当前计数）和`ret`（累加结果）。  
2. 构造函数中，将`i`加到`ret`，然后`i`+1。  
3. 在`Sum_Solution`函数中，创建`n`个`Sum`对象，触发`n`次构造函数调用，最终`ret`即为结果。
```cpp
class Sum
{
public:
    Sum()
    {
        ret += i;
        i++;
    }
    static int Get()
    {
        return ret;
    }
private:
    static int i;
    static int ret;
};
int Sum::i = 1;
int Sum::ret = 0;
class Solution {
public:
    int Sum_Solution(int n) {
        Sum arr[n];
        return Sum::Get();
    }
};
```
## 3.4 友元函数
在上文的日期类实现中，我们对流插入和流提取写在了类外面，因为类里面的运算符重载要求第一个参数是隐式的this指针，实现的流插入和流提取使用方法是和内置类型的使用方法是反着来的，所以我们要把函数放外面，但是又要能突破访问限定符的限定，可以在类的任意区域对函数进行声明，并用`friend`修饰，具体参考日期类的实现
- 友元不仅仅是函数，友元的类也可以，等价于把那个类的所有函数都变成了友元函数
- 友元类的关系是单向的，不具有交换性，⽐如A类是B类的友元，但是B类不是A类的友元
- 友元类关系不能传递，如果A是B的友元，B是C的友元，但是A不是C的友元
## 3.5类的内部类
一开始学的时候我们就会想能不能像C语言一样在类里面套一个类，其实是可以的，相当于内部类是外部类的友元类，除此之外内部类还受外部类的域限制和访问限定符限制，其他的性质与正常的类没有差别
## 3.6 匿名对象
和匿名结构体一样，类也有匿名的，其使用方法与匿名结构体方法类似，对于日期类来说：`Date(1,2,3)`这样类的声明周期只有一行，这种写法有很多键盘之处，以现在的知识不好讲解，需要我们今后的学习再去了解
# 四、结语
类和对象是C++的一大特色，由于C++在诞生之初没有其他语言可以抄作业，所以其发展也是有好有坏，笔者认为类和对象的知识有一些是繁重且不必的，比如实例化的时候都是相同大小的空间，内存对齐会浪费很多空间，但是有好有坏，笔者对C++了解不深不对C++做过多评价
类和对象需要读者好好练习，知识点繁重


---
[^1]: 在C++中类的使用可以不带class关键了，这就不需要像之前C语言一样只用`typedef`关键字进行重命名，直接使用类的名字就可以

[^2]: 
1.  ==**浅拷贝**‌==：仅复制对象表面的值，遇到引用类型（如指针、嵌套对象）只拷贝地址。新旧对象内部指向同一块内存，像“给同一个房间多配了一把钥匙”，两把钥匙都能开同一个门 。
2. ==‌**深拷贝**‌==：不仅复制对象本身，还递归复制所有子对象。新对象拥有独立的内存空间，像“重新盖了一间一模一样的房间”，新钥匙开新房间，旧钥匙开旧房间，互不干扰

[^3]: 对于一个定义在类里面的成员函数，先对函数指针重命名`typedef void(A::*PF)();`对成员函数func取地址为`PF pf = &A::func;`，创建一个obj对象，那么调用的时候应该指定类域，即`(obj.*pf)();`
```cpp
class A 
{ 
public: 
	void func() 
	{ 
		`cout << "A::func()" << endl; 
	} 
}; 
typedef void(A::*PF)();  
PF pf = &A::func; A obj;
void main()
{
	(obj.*pf)();
}
```

---

## 相关笔记 / 相关链接

- 相关目录：[[C++目录]]
- 相关笔记：[[C++/C++入门基础]]（前置：引用与命名空间）
