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
  <font color="#00FFFF"><b>作者仓库</b></font><br>
  <a href="https://gitee.com/warm-flame-core">代码仓库_gitee</a>
</p>


# 一、引言
上一篇我们探究了string的用法和模拟实现，会发现因为C语言版本的字符串并不好用，而string就能解决。同样的，vector也是如此，C语言版本的数组能提供的接口还是太少了，接下来我们就探究一下vector
# 二、vector的使用
## 2.1 定义一个vector对象
stl库提供了多种定义vector的方法：
```cpp
vector<int> v
vector<int> v1(10, 1);
vector<int> v2(10, 2);
vector<int> v3(v2.begin(), v2.end());
```
常见的就三种构造方法，其中迭代器构造方法是支持其他类型的迭代器，甚至是普通数组
## 2.2 迭代器
对于初学者，使用上所有迭代器我们都要学会怎么使用，但是具体底层实现我们只了解正向的迭代器怎么实现的

| iterator的使用   | 说明                                                                        |
| ------------- | ------------------------------------------------------------------------- |
| begin + end   | 获取第一个数据位置的iterator/const_iterator， 获取最后一个数据的下一个位置的iterator/const_iterator |
| rbegin + rend | 获取最后一个数据位置的reverse_iterator，获取第一个数据前一个位置的reverse_iterator                 |
```cpp
vector<int> v(10, 1);
for (auto e : v)
{
	cout << e << endl;
}
vector<int>::iterator it = v.begin();
while (it != v.end())
{
	cout << *it << " ";
	++it;
}
cout << endl;
```
具体正向迭代器的底层是怎么实现的后面我们模拟实现的时候再探究
## 2.3 扩容问题

| size     | 有效数据个数     |
| -------- | ---------- |
| capacity | 获取容量大小     |
| empty    | 是否为空       |
| resize   | 调整size     |
| reserve  | 调整capacity |
对于使用上述方法需要注意一些问题：
- reserve只负责开辟空间，如果确定知道需要用多少空间，reserve可以缓解vector增容的代价缺陷问题
- resize在开空间的同时还会进行初始化，影响size。也就是说如果size的参数过于短那么数据就会被截断
- 如果对一个vector不断尾插，会发现不同平台对于capacity扩容是不一样的，msvc是2倍，g++则是1.5倍，下面笔者给出一段代码可以供读者测试自己平台下的扩容机制
```cpp
void TestVectorExpand()
{
    size_t sz;
    vector<int> v;
    sz = v.capacity();
    cout << "making v grow:\n";
    for (int i = 0; i < 100; ++i)
    {
        v.push_back(i);
        if (sz != v.capacity())
        {
            sz = v.capacity();
            cout << "capacity changed: " << sz << '\n';
        }
    }
}
```
## 2.4增删查改
`vector`的增删查改与`string`一样，这里不再赘述，需要注意的是如果要查找`vector`的元素需要用算法库里面的`find`，`vector`并没有关于`find`的成员函数
# 三、迭代器失效
在学习string的时候我们大部分都是用方括号的形式进行访问，但是到了vector，我们要逐渐习惯迭代器方法，迭代器的主要作用就是让算法能够不用关心底层数据结构，其底层实际就是一个指针，或者是对指针进行了封装，比如：vector的迭代器就是原生态指针T* 。因此迭代器失效，实际就是迭代器底层对应指针所指向的空间被销毁了，而使用一块已经被释放的空间，造成的后果是程序崩溃(即 如果继续使用已经失效的迭代器，程序可能会崩溃)。
## 3.1 扩容造成的迭代器失效
```cpp
#include <iostream>
using namespace std;
#include <vector>
int main()
{
    vector<int> v{ 1,2,3,4,5,6 };
    auto it = v.begin();
    // 给vector重新赋值，可能会引起底层容量改变
    v.assign(100, 8);
    
    while (it != v.end())
    {
        cout << *it << " ";
        ++it;
    }
    cout << endl;
    return 0;
}
```
其中`assign`函数是给vector重新赋值，就像`realloc`一样，重新调整大小需要搬运到新的内存空间，而迭代器`it`还指向者原来已经被释放的空间，造成迭代器变成野指针，使用insert也是同理。迭代器使用前应该先更新
## 3.2 删除造成的迭代器实现
```cpp
#include <iostream>
using namespace std;
#include <vector>
int main()
{
    int a[] = { 1, 2, 3, 4 };
    vector<int> v(a, a + sizeof(a) / sizeof(int));
    vector<int>::iterator pos = find(v.begin(), v.end(), 4);
    v.erase(pos);
    cout << *pos << endl;
    return 0;
}
```
erase删除pos位置元素后，pos位置之后的元素会往前搬移，没有导致底层空间的改变，理论上讲迭代器不应该会失效。但是如果pos刚好是最后一个元素，删完之后pos刚好是end的位置，而end位置是没有元素的，那么pos就失效了。因此删除vector中任意位置上元素时，迭代器如果要使用应该先更新
### 3.3.1 删除所有的偶数
如果不更新迭代器，迭代器可能会越过`end()`位置，造成死循环，正确的应该是这样写
```cpp
vector<int> v{ 1, 2, 3, 4 };
auto it = v.begin();
while (it != v.end())
{
    if (*it % 2 == 0)
        it = v.erase(it);
    else
        ++it;
}
```
<center><font color="red">所以：只要是对象被动过，我们都应该认为迭代器已经失效需要重新获取</font></center>
# 四、一道简单的oj题目
在C语言阶段肯定是学过用数组使用杨辉三角，那么现在我们也可以用vector，既然`vector<int>`可以类比成一维数组，那么vector`<vector<int>>`就是二维数组，使用上和普通二位数组没有区别，那么杨辉三角也可以用vector实现：
```cpp
vector<vector<int>> generate(int numRows) 
{
    vector<vector<int>> vv;
    for (int i = 0; i < numRows; i++)
    {
        vector<int> v(i + 1, 1);
        vv.push_back(v);
    }
    for (int i = 2; i < numRows; i++)
    {
        for (int j = 2; j < i; j++)
        {
            vv[i][j] = vv[i - 1][j - 1] + vv[i - 1][j];
        }
    }
    return vv;
}
```
# 五、模拟实现——迭代器思想（基于**SGI STL**库）
对于初学者来说，直接去看源代码学习是非常好的学习思想，但是源码毕竟是别人十几年的心血，各种弯弯绕绕的函数会让你无从下手，所以我们观察可以从以下几个方面入手
1. 一个库一定是含有好几个类，我们就从头开始看，就先观察第一个类的成员。可以看出，第一个类是迭代器相关的类，其成员有
```cpp
iterator start;
iterator finish;
iterator end_of_storage;
```
	这个时候不通过函数我们还不清楚这些元素是什么意思，但是通过名字我们可以猜，分为对应线性表的开始，结束和内存结束的地方，所以这个类应该是给迭代器用的
2. 往下看能看到很多个函数的声明，根据函数名就可以看出各个函数的作用是什么
<br>
**总结一下：vector并没有用指针+size+capacity定义线性表，所以操作都是基于三个指针之间的运算进行操作的，接下来的模拟实现也是通过**

```cpp
template<class T>
class vector
{
private:
	iterator _start = nullptr;
	iterator _finish = nullptr;
	iterator _end_of_storage = nullptr;
};
```
定义出成员，我们先把迭代器部分写好
## 5.1 迭代器模拟
在vector里，我们可以认为迭代器就是一个指针，所以我们可以对指针重命名
```cpp
typedef T* iterator;
typedef const T* const_iterator;
```
其中begin就是返回头指针，end就是返回数据结束尾指针还是非常简单的
```cpp
iterator begin()
{
	return _start;
}

iterator end()
{
	return _finish;
}

const_iterator begin() const
{
	return _start;
}

const_iterator end() const
{
	return _finish;
}
```
## 5.2 返回线性表的参数
关于size和capacity我们就需要用三种指针之间的运算进行操作了，指针相减就是对应类型的个数，其中多余方括号的重载也很简单，我们有可头指针直接像普通数组一样移动就可以了
```cpp
size_t size() const
{
	return _finish - _start;
}

size_t capacity() const
{
	return _end_of_storage - _start;
}

T& operator[](size_t i)
{
	assert(i < size());
	return _start[i];
}

const T& operator[](size_t i) const
{
	assert(i < size());
	return _start[i];
}
```
## 5.3 调整大小
如果我们要写尾插，那么效率高的做法应该是先调整好大小再进行尾插，减少开空间的次数，所以我们要先实现`reserve`
整体思路是这样的，先开辟一个空间，再拷贝数据，清理旧空间
```cpp
void reserve(size_t n)		// 扩容
{
	if (n > capacity())
	{
		T* tmp = new T[n];
		memcpy(tmp, _start, sizeof(T) * size());
		delete[] _start;
		_start = tmp;
		_finish = tmp + size();
		_end_of_storage = tmp + n;
	}
}
```
但是这样的写法有两个严重错误
1. 拷贝使用了memcpy，这样拷贝如果T实例化成自定义类型，直接搬运字节的浅拷贝会使新旧空间指向同一个地方，后续释放`_start`的时候新空间就是野指针，比如说如果T是string，上一章节我们知道定义对象我们只能拿到`string`的指针，直接搬运指针这种浅拷贝到后面释放的时候会变成野指针，所以我们应该使用深拷贝，这样自定义类型会调用自己的拷贝构造函数
2. 对指针重新指向的时候，此时`size()`的返回值是用旧空间的`_finish`减去了新空间的`_start`，这种运算会导致`_finish`无法确定指向何处，正确做法应该先保存旧的数据个数再使用
```cpp
void reserve(size_t n)		// 扩容
{
	if (n > capacity())
	{
		size_t old_size = size();
		T* tmp = new T[n];
		// 不能使用memcpy，如果对象是自定义的类型会造成浅拷贝，释放旧空间变成野指针
		//memcpy(tmp, _start, sizeof(T) * old_size);
		for (size_t i = 0; i < old_size; i++)
		{
			tmp[i] = _start[i];
		}
		delete[] _start;
		_start = tmp;
		_finish = tmp + old_size;
		_end_of_storage = tmp + n;
	}
}
```
## 5.4 构造函数
我希望我的模拟实现能显示出不传参的构造、迭代器构造和用`N`个`X`进行的构造，`N`个`X`进行的构造的构造非常简单
```cpp
vector(size_t n, T val = T())
{
	reserve(n);
	for (size_t i = 0; i < n; i++)
	{
		push_back(val);
	}
}
```
模板里面的成员函数，依旧可以是函数模板，我们可以再用模板这样支持所有类型的迭代器
```cpp
template <class InputIterator>
vector(InputIterator first, InputIterator last)
{
	while (first != last)
	{
		push_back(*first);
		++first;
	}
}
```
不是所有的迭代器都有大小于号的比较，需要用不等于是为了适配多种迭代器
但是如果只有这两个构造，对于`vector<int> v3(10, 1);`这个语句编译器会报错，本意其实我们想要用`N`个`X`进行的构造，但是类型不匹配所以编译器实例化了迭代器，但是int不能解引用，对此我们应该在提供一个int类型的构造
```cpp
vector(int n, T val = T())
{
	reserve(n);
	for (int i = 0; i < n; i++)
	{
		push_back(val);
	}
}
```
但是只有这三个构造还不够，因为这样编译器就不会生成默认构造函数用来初始化不传参的对象，但是自己写也好烦，所以再C++11中新加入一个语法
```cpp
vector() = default;
```
这样写可以强制让编译器生成默认构造函数
## 5.5 其他函数
这些函数很常规，不必细究
```cpp
bool empty()
{
	return _start == _finish;
}
void resize(int n, T val = T())
{
	if (n < size())
		_finish = _start + n;
	else
	{
		reserve(n);
		while (_finish < _start + n)
		{
			*(_finish) = val;
			++_finish;
		}
	}
}
```
## 5.6 插入和删除
由于之前已经踩过野指针和保存原来数值的坑，这个时候我们写插入函数就得心应手
```cpp
iterator insert(iterator pos, const T& x)
{
	if (_finish == _end_of_storage)
	{
		size_t len = pos - begin();
		reserve(capacity() == 0 ? 4 : capacity() * 2);
		pos = begin() + len;
	}
	iterator end = _finish - 1;
	while (end > pos - 1)
	{
		*(end + 1) = *end;
		end--;
	}
	*pos = x;
	++_finish;
	return pos;
}
```
需要注意的是扩容前应该先计算相对位置，因为扩容后pos可能是野指针
<br>
对于删除应该要从后往前挪动数据，非常简单
```cpp
iterator erase(iterator pos)
{
	assert(pos >= _start);
	assert(pos < _finish);
	iterator it = pos + 1;
	while (it != end())
	{
		*(it - 1) = *(it);
		++it;
	}
	--_finish;
	return pos;
}
```
其中尾插和尾删可以复用
```cpp
void push_back(const T& x)		// 尾插
{
	if (_finish == _end_of_storage)
	{
		reserve(capacity() == 0 ? 4 : capacity() * 2);
	}
	*_finish = x;
	_finish++;
}

void pop_back()
{
	assert(!empty());
	_finish--;
}
```
## 5.7 打印任意迭代器的内容
为了支持我们检验结构，我们可以写一个函数模板用来打印结构，也很简单
```cpp
template<class T>
void vector_print(const vector<T>& v)
{
	vector<T>::const_iterator it = v.begin();
	while (it != v.end())
	{
		cout << *it << ' ';
	}
	cout << endl;
	for (auto e : v)
	{
		cout << e << ' ';
	}
	cout << endl;
}
```
但是运行这个函数编译器会报错，试想一下模板再编译时是没有实例化的，所以编译不知道`vector<T>::const_iterator`指的是类型还是变量，编译器分不清所以就报错，如果要解决这个问题需要在未实例化的类型前加上`typename`，即`typename vector<T>::const_iterator it = v.begin();`，或者有更简单的方法用auto自动推导
```cpp
template<class T>
void vector_print(const vector<T>& v)
{
	auto it = v.begin();
	while (it != v.end())
	{
		cout << *it << ' ';
	}
	cout << endl;
	for (auto e : v)
	{
		cout << e << ' ';
	}
	cout << endl;
}
```
## 5.8 拷贝构造，赋值重载，析构
拷贝构造很简单，直接一个一个尾插就好了
```cpp
vector(const vector<T>& v)
{
	reserve(v.size());
	for (auto& e : v)
	{
		push_back(e);
	}
}
```
对于赋值重载，我们在上一章就说过可以用现代写法
```cpp
vector<T>& operator=(vector<T> v)
{
	swap(v);
	return *this;
}

void swap(vector<T>& v)
{
	std::swap(_start, v._start);
	std::swap(_finish, v._finish);
	std::swap(_end_of_storage, v._end_of_storage);
}
```
交换函数直接交换指针就可以，对于拷贝构造参数不引用，构造一个临时对象交换，临时对象和所需对象进行交换，临时对象出作用域就销毁了
析构函数也很简单
```cpp
~vector()
{
	if (_start)
	{
		delete[] _start;
		_start = _finish = _end_of_storage = nullptr;
	}
}
```
# 六、总结
总的来说vector是比较简单的，也挺会到了迭代器的方便和实现，下一章我们来探究list

---

## 相关笔记 / 相关链接

- 相关目录：[[C++目录]]
- 相关笔记：[[C++/string库]]（同类：STL 容器）、[[C++/带头结点的双向循环链表详解——list 的模拟实现]]（对比：连续存储与链式存储）