


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


1.  [引言](#引言)
2.  [一、前置知识](#一前置知识)
    1.  [1.1 auto变量](#11-auto变量)
        1.  [1.1.1 检查变量类型](#111-检查变量类型)
    2.  [1.2 范围 For (Range-based For Loop)](#12-范围-for-range-based-for-loop)
    3.  [1.3 容器 (Container)](#13-容器-container)
    4.  [1.4 迭代器 (Iterator)](#14-迭代器-iterator)
    5.  [1.5 范围 For 和迭代器的关系](#15-范围-for-range-based-for-loop和迭代器-iterator的关系)
3.  [二、String 功能概览](#二-string-功能概览)
    1.  [2.1 构造与初始化](#21-构造与初始化)
    2.  [2.2 常见容量管理 (Capacity)](#22-常见容量管理-capacity)
    3.  [2.3 增删查改](#23-增删查改)
        1.  [2.3.1 方括号重载查找](#231-方括号重载查找)
        2.  [2.3.2 +=重载](#232-重载)
        3.  [2.3.3 push_back()尾插字符](#233-push_back尾插字符)
        4.  [2.3.4 append()附加字符/字符串](#234-append附加字符字符串)
        5.  [2.3.5 insert()插入字符/字符串](#235-insert插入字符字符串)
        6.  [2.3.6 assign()重新分配字符/字符串](#236-assign重新分配字符字符串)
        7.  [2.3.7 erase()删除字符串](#237-erase删除字符串)
        8.  [2.3.7 replace()替换字符/字符串](#237-replace替换字符字符串)
        9.  [2.3.8 find()查找字符返回下标](#238-find查找字符返回下标)
        10. [2.3.9 substr()构建字串](#239-substr构建字串)
        11. [2.3.10 c_str()字符串指针](#2310-c_str字符串指针)
        12. [2.3.11 getline输入字符串](#2311-getline输入字符串)
4.  [三、 LeetCode 实战演练](#三-leetcode-实战演练)
    1.  [3.1 LeetCode 917. 仅仅反转字母](#31-leetcode-917-仅仅反转字母)
    2.  [3.2 LeetCode 387. 字符串中的第一个唯一字符](#32-leetcode-387-字符串中的第一个唯一字符)
5.  [四、 模拟实现 String](#四-模拟实现-string)
    1.  [4.1 成员变量设计](#41-成员变量设计)
    2.  [4.2 获取各种字符串参数和运算符重载的函数](#42-获取各种字符串参数和运算符重载的函数)
    3.  [4.3 迭代器实现](#43-迭代器实现)
    4.  [4.4 构造与析构](#44-构造与析构)
    5.  [4.5 拷贝构造与赋值重载](#45-拷贝构造与赋值重载)
    6.  [4.6 扩容 Reserve 与 Push_Back](#46-扩容-reserve-与-push_back)
    7.  [4.6 其他简单功能函数](#46-其他简单功能函数)
    8.  [4.7 Insert 函数的“无符号整数陷阱”](#47-insert-函数的无符号整数陷阱)
    9.  [4.8 Substr 的溢出陷阱](#48-substr-的溢出陷阱)
    10. [4.9 IO 流重载的优化](#49-io-流重载的优化)
    11. [4.10 find函数](#410-find函数)
    12. [4.11 现代写法的优化](#411-现代写法的优化)
    13. [4.12 swap函数](#412-swap函数)
6.  [五、GCC 与 MSVC 的底层差异及总结](#五gcc-与-msvc-的底层差异及总结)
    1.  [5.1 VS (MSVC) 的短字符串优化 (SSO)](#51-vs-msvc-的短字符串优化-sso)
    2.  [5.2 GCC 的写时拷贝 (COW) vs 深拷贝](#52-gcc-的写时拷贝-cow-vs-深拷贝)
7.  [结语](#结语)
# 引言
C语言中对于字符串的支持很差，字符串函数也没有任何关于限定，越界问题没有预防。C++扩充了结构体的用法，变成了`类和对象`，STL就有了专门为字符串设计的库(***string***)

---

# 一、前置知识
## 1.1 auto变量
在早期C/C++中auto的含义是：使用auto修饰的变量，是具有自动存储器的局部变量，后来这个 不重要了。C++11中，标准委员会变废为宝赋予了auto全新的含义即：auto不再是一个存储类型 指示符，而是作为一个新的类型指示符来指示编译器，auto声明的变量必须由编译器在编译时期推导而得
对于auto类型变量，有以下几个主要注意的地方：
1. `auto` 和`auto*`在我们想表示指针的时候都是对的，前者表示任何变量，编译器推导；后者表示已经是指针了，其指针所属的基类型由编译器推导，但是C++新增的语法`引用`必须用`auto &`
```cpp
	int x = 10;
	auto y = &x;
	auto* z = &x;
	auto& m = x;
```
2. C99之后支持声明和定义写在一起，同时可以多个变量一起定义且是可以分别定义各自的初始值，引入`auto`之后，同一行必须是同一类型
```cpp
auto cc = 3, dd = 4.0;
```
	上面的写法就是错的，推导只能推导一次，直接冲突了会提示：

> error C3318: “auto []”: 数组不能具有其中包含“auto”的元素类型

```cpp
auto aa = 1, bb = 2;
```
这样就是对的
3. `auto`不能作为函数的参数，可以做返回值，但是建议谨慎使用
```cpp
void func2(auto a) {}
```
	这样是错的
```cpp
auto func3() 
{ 
	return 3; 
}
```
	这样就是对的，但是这样别人使用函数的时候需要观察返回值才能懂函数的类型，如果返回值很复杂就不合适了
4. `auto`不能直接声明数组类型
```cpp
auto array[] = { 4, 5, 6 };
```
会报错：
> “auto []”: 数组不能具有其中包含“auto”的元素类型

### 1.1.1 检查变量类型
如果想看看一个auto变量究竟被编译器推导出什么类型了，用`typeid(变量名).name()`就可以知道类型，在后面观察`迭代器`的时候很有用
## 1.2 范围 For (Range-based For Loop)
对于一个有范围的集合而言，由程序员来说明循环的范围是多余的，有时候还会容易犯错误。因此 C++11中引入了基于范围的for循环。for循环后的括号由冒号“ ：”分为两部分：第一部分是范围 内用于迭代的变量，第二部分则表示被迭代的范围，自动迭代，自动取数据，自动判断结束
```cpp
for(auto ch : arr)
{
	//……………………
}
```
对于数组来说可以这样写
```cpp
int arr[5] = {1, 2, 3, 4, 5};
for (auto ch : arr) 
{
    std::cout << ch << ' ';
}
```
相当于编译器自己推导出循环的起始和终止
对于范围 For需要注意的时候，如果循环内修改不希望影响原本的数组，那么使用值传递就可以，就像上例一样
如果需要修改影响原来的数组，那么用引用传递
```cpp
char arr[] = "hello world";
for (auto &ch : arr)
{
	ch+=2;
	cout << ch << " ";
}
cout << endl;
cout << arr << endl;
```
其结果是就是：
```text
j g n n q " y q t n f
jgnnq"yqtnf
```

> `auto` 会忽略引用和顶层 const。如果只想读取不想修改，建议写 `const auto& ch`。

## 1.3 容器 (Container)
C++中的**容器**就是**用来存放数据的“盒子”**。
是 C++ 标准库（STL）中为我们提前写好的一套**数据结构模板**。不需要自己从零去写链表、数组或二叉树，直接拿来用就行，本文我们就是介绍string这个容器
## 1.4 迭代器 (Iterator)

迭代器是 STL 的核心，所有容器都指向自己开始和结束的标识，类似于指针，用于访问容器中的元素。对于 `string` 而言，迭代器本质上就是 `char*` 指针。
<p align="center"><font color="red">当下这个阶段我们不太需要详细了解迭代器细节和底层，到后续学完多态之后才能详细研究，当下只需要记住怎么用就可以了</font></p>

所有容器都有自己的`begin`、`end`、`rbigin`、`rend`、在C++11之后还加入了`cbegin`、`cend`、`crbigin`、`crend`
其迭代器还分为**正向迭代器、反向迭代器、const正向迭代器、const反向迭代器**
对于string来说
- **正向迭代器**：`begin()` 指向首元素，`end()` 指向最后一个元素的下一个位置（`\0`）
	所属类型是`string::iterator`
- **反向迭代器**：`rbegin()` 指向最后一个元素，`rend()` 指向第一个元素的前一个位置。
	所属类型是`string::reverse_iterator`
其begin和end满足“左闭右开”的原则
如果要遍历字符串，就可以这样写
```cpp
string s1("Hello World");
// 正向遍历
string::iterator it = s1.begin();
while (it != s1.end()) 
{
    cout << *it << " ";
    ++it;
}
cout << endl;
// 反向遍历
string::reverse_iterator rit = s1.rbegin();
while (rit != s1.rend()) 
{
    cout << *rit << " ";
    ++rit;
}
```
输出结果就是
```text
H e l l o   W o r l d
d l r o W   o l l e H
```
会发现每次写类型的时候非常繁琐，这个时候就是auto的作用了
`string::iterator it = s1.begin();`和`string::reverse_iterator rit = s1.rbegin();`就分别可以写成`auto it = s1.begin()`和`auto rit = s1.rend()`


> **const 迭代器**：当对象被 `const` 修饰时，必须使用 `const_iterator`或者`const_reverse_iterator`，此时只能读取不能修改内容。

## 1.5 范围 For (Range-based For Loop)和迭代器 (Iterator)的关系
对于这串代码
```cpp
string s1("Hello World");
for (auto ch : s1)
{
	cout << ch;
}
cout << endl;
auto it = s1.begin();
while (it != s1.end())
{
	cout << *it;
	++it;
}
```
（在VS中随便设置一个断点然后调试，右键代码任意区域转到反汇编即可观察汇编代码）
我们观察他们的汇编代码会发现他们本质都是一样的，其实范围 For 的本质就是迭代器

---

# 二、String 功能概览


## 2.1 构造与初始化
### 2.1.1 构造函数
`string` 提供了极其灵活的构造方式：

```cpp
string s1;
string s2("hello world");
string s3(s2, 6, 5);
string s4(s2);
string s5 = s4;
string s7(6, 'c');
```
从上到下分别是：
1. 默认构造，空串
2. C 风格字符串构造
3. 截取构造：从索引 6 开始取 5 个字符 -> "world"
4. 拷贝构造
5. 赋值构造
6. 填充构造：6 个 'c' -> "cccccc"
## 2.2 常见容量管理 (Capacity)
- **`size()` / `length()`**: 返回有效字符个数
- **`capacity()`**: 返回当前分配的总空间大小（不含 `\0`）
- **`reserve(n)`**: 预分配空间。**只改变 capacity，不改变 size**。常用于已知数据量大小时，避免频繁扩容
- **`resize(n, char)`**: 改变有效字符个数。**既改变 size 也可能改变 capacity**。若 n > size，多出的部分用指定字符填充；若 n < size，则截断用`\0`填充
- **`clear()`**: 清空字符串，变成空值
```cpp
string s1("Hello World");
cout << s1.size() << endl;		// 有效字符个数
cout << s1.length() << endl;	// 有效字符个数
cout << s1.capacity() << endl;	// 字符串有效空间
cout << s1.empty() << endl;		// 字符串判空
s1.reserve(15);					// 字符串扩容
s1.resize(3);					// 字符串调整
cout << s1 << endl;
```
输出为：
```text
11
11
15
0
Hel
```
## 2.3 增删查改
在讲增删查改之前，string有一个全局的静态`size_t`类型的变量`npos`，其值为**-1**，隐式类型转换之后就是无符号整型最大值
### 2.3.1 方括号重载查找
string重载了方括号的使用方法，其使用方法和数组下标一样的用法
### 2.3.2 +=重载
string重载了+=，可以像数值运算一样**追加**字符、字符串
```cpp
string s1("hello world");
s1 += 'x';
s1 += "hello yourself";
```
### 2.3.3 push_back()尾插字符
函数可以在字符串后面直接加一个字符串
```cpp
string s1("hello world");
s1.push_back('x');
```
### 2.3.4 append()附加字符/字符串
函数有多种重载
```cpp
string& append (const string& str);
string& append (const string& str, size_t subpos, size_t sublen);
string& append (const char* s);
string& append (const char* s, size_t n);
string& append (size_t n, char c);
```
其功能和`push_back`差不多，不过是支持字符串了，当然常见的还是`append`一个字符串
```cpp
string s1("Hello World");
s1.append(" Hello Myself");	// 拼接字符串
cout << s1 << endl;
```
### 2.3.5 insert()插入字符/字符串
```cpp
string& insert (size_t pos, const string& str);|
string& insert (size_t pos, const string& str, size_t subpos, size_t sublen);
string& insert (size_t pos, const char* s);
string& insert (size_t pos, const char* s, size_t n);
string& insert (size_t pos, size_t n, char c);
```
常见的是插入一个字符串：
```cpp
string s4("Hello World");
s4.insert(0, "1111");
cout << s4 << endl;
```
上例表示在字符串下标0的位置之前插入`1111`
### 2.3.6 assign()重新分配字符/字符串
```cpp
string& assign (const string& str);
string& assign (const string& str, size_t subpos, size_t sublen);
string& assign (const char* s);
string& assign (const char* s, size_t n);
string& assign (size_t n, char c);
```
常见的是替换一个已经存在的字符串
```cpp
string s5("xxxxxxxxx");
s5.assign(4, 'c');
cout << s5 << endl;
```
打印结果就是`cccc`
### 2.3.7 erase()删除字符串
常见的就是删除字符串里的部分
```cpp
string s6("Hello World");
s6.erase(5, 1);	
cout << s6 << endl;	
s6.erase(s6.begin());
cout << s6 << endl;
```
第一次删除表示在下标`5`的位置开始删除`1`个字符,第二次删除表示从迭代器末尾开始删除，且没有显式传参数要删除几个就是删除`npos`个
### 2.3.7 replace()替换字符/字符串
函数可以用来指定字符串的下标，然后替换字符，但是这个函数更多的用处是借助下面要说的`find()`函数一起使用才有价值
### 2.3.8 find()查找字符返回下标
函数可以找到特定字符并返回下标，如果函数找不到对应下标就会返回`npos`
一个实际使用就是把一串字符串的`空格`全部替换为`%%`
```cpp
string sss("hello          world hello xjw");
size_t pos = sss.find(' ');
这样查找字符效率低下
while (pos != string::npos)
{
	sss.replace(pos, 1, "%%");
	cout << sss << endl;

	pos = sss.find(' ', pos+2);
}
cout << sss << endl;
```
首先先找到第一个空格下标，进入循环用`replace`替换，再从后面第二个开始查找下标，直到函数返回`npos`表示全部替换完了
但是这个函数效率低下，每次的一个`空格`变成两个`%%`都需要扩容，每次扩容是繁重的，效率高的做法是用**空间换时间**，先开一个和原字符串一样大小的`string tmp`，然后依次获取原字符串的字符，如果不是`空格`就复制字符到临时字符串，如果是`空格`就往临时字符串写入两个`%%`，避免扩容次数太多
```cpp
string tmp;
tmp.reserve(sss.size());
for (auto ch : sss)
{
	if (ch == ' ')
		tmp += "%%";
	else
		tmp += ch;
}
sss.swap(tmp);
cout << sss << endl;
```
find是从头开始查找，有的使用场景需要我们反着查找，这个时候就需要`rfind`函数了，这个函数返回值和find一样，不过就是倒着找，一个使用场景就是查找一个文件名的扩展名，这个时候就不能从头查找，像这样的文件名`test.txt.zip`就会出问题，所以要从尾开始查找
```cpp
string s8("test.txt.zip");
size_t pos1 = s8.rfind('.');
string suffix = s8.substr(pos1);
cout << suffix << endl;
```
代码中的`substr`函数在下文有介绍
除了`find``rfind`，还有`find_first_of`、`find_last_of`、`find_first_not_of`、`find_last_not_of`，但是这些函数用的很少，本文就不详细介绍了
### 2.3.9 substr()构建字串
这个函数可以从字符串中复制出一部分构建一个新的string
```cpp
void SplitFilename(const std::string& str)
{
	std::cout << "Splitting: " << str << '\n';
	std::size_t found = str.find_last_of("/\\");

	std::cout << " path: " << str.substr(0, found) << '\n';
	std::cout << " file: " << str.substr(found + 1) << '\n';
}
```
这个示例的函数可以把一串文件路径中的路径名和文件名分开，找到的两个串分别构建两个字符串，输出路径名和文件名

### 2.3.10 c_str()字符串指针
为了兼容C语言的字符串指针，这个函数可以返回string中的字符串指针
```cpp
string file;
cin >> file;
FILE* fout = fopen(file.c_str(), "r");
char ch1 = fgetc(fout);
while (ch1 != EOF)
{
	cout << ch1;
	ch1 = fgetc(fout);
}
fclose(fout);
```
获取的字符串在string类中，传入C语言函数就需要使用c_str函数
### 2.3.11 getline输入字符串
由于C++的字符串已经变成了类，C语言系列的输入已经不能很好的使用了这个时候就需要用`getline`函数，其函数有两个重载
```cpp
istream& getline (istream& is, string& str, char delim);
istream& getline (istream& is, string& str);
```
其中如果第三个参数如果没有指定的话默认函数以空格和换行作为结束标识，即从`io流`中读取字符串到`str`中，结束标识是`delim`

# 三、 LeetCode 实战演练

## 3.1 LeetCode 917. 仅仅反转字母

给定一个字符串，将所有非英文字母保留在原有位置，所有英文字母（小写或大写）位置反转。
1. 输入：s = "ab-cd"
	输出："dc-ba"
2. 输入：s = "a-bC-dEf-ghIj"
	输出："j-Ih-gfE-dCba"
这是**典型的**对撞双指针**问题。
3. 定义 `begin` 指向头，`end` 指向尾。
4. `begin` 向右找字母，`end` 向左找字母。
5. 找到后交换，然后继续向中间逼近。
```cpp
class Solution {
public:
    string reverseOnlyLetters(string s) {
        int begin = 0, end = s.size() - 1;
        while (begin < end) {
            // 左边不是字母就往后走
            while (begin < end && !isalpha(s[begin])) {
                begin++;
            }
            // 右边不是字母就往前走
            while (begin < end && !isalpha(s[end])) {
                end--;
            }
            // 交换两个字母
            swap(s[begin++], s[end--]);
        }
        return s;
    }
};
```

## 3.2 LeetCode 387. 字符串中的第一个唯一字符

找到字符串中第一个不重复的字符，并返回它的索引。如果不存在，则返回 -1。
1. 输入: s = "leetcode"
	输出: 0
2. 输入: s = "aabb"
	输出: -1
**解题思路**：**哈希计数法/计数排序**
因为题目提示只包含小写字母，我们可以开一个大小为 26 的数组 `arr` 作为简易哈希表。
第一次遍历：统计每个字符出现的次数。
第二次遍历：按顺序检查字符，如果 `arr[ch - 'a'] == 1`，说明它是唯一的，直接返回索引。
```cpp
class Solution {
public:
    int firstUniqChar(string s) {
        int arr[26] = {0}; // 初始化为0
        // 统计频次
        for (auto ch : s) {
            arr[ch - 'a']++;
        }
        // 寻找第一个频次为1的
        for (int i = 0; i < s.size(); i++) {
            if (arr[s[i] - 'a'] == 1) {
                return i;
            }
        }
        return -1;
    }
};
```

---

# 四、 模拟实现 String 

模拟实现知识学习设计思路和复习，不是为了写的比官方更好和实现官方的全部功能

## 4.1 成员变量设计

```cpp
private:
    char* _str = nullptr;   // 指向动态开辟的字符串
    size_t _size = 0;       // 有效字符长度
    size_t _capacity = 0;   // 总容量（不含\0）
    static const size_t npos; // 表示“未找到”的特殊值
```
其中npos有一个特殊的点：之前的类和对象我们说static成员不能再成员声明的时候给缺省值，但是静态的整形最特殊，可以直接给初始值，但是为了保持风格还是建议类外面写默认值
## 4.2 获取各种字符串参数和运算符重载的函数
```cpp
size_t size() const
{
	return _size;
}

size_t capacity() const
{
	return _capacity;
}
```
## 4.3 迭代器实现
对于string容器，可以认为迭代器就是指针移动，多以只需要对指针重命名就好了
```cpp
typedef char* iterator;
typedef const char* const_iterator;
iterator begin()
{
	return _str;
}
iterator end()
{
	return _str + _size;
}
const_iterator begin() const
{
	return _str;
}
const_iterator end() const
{
	return _str + _size;
}
```
## 4.4 构造与析构

### 构造函数

```cpp
string(const char* str = "") 
{
    _size = strlen(str);
    _capacity = _size;
    _str = new char[_capacity + 1];
    strcpy(_str, str);
}
```

需要注意的是：
1. 参数给缺省值 `""` 而不是 `nullptr`，防止 `strlen(nullptr)` 崩溃。
2. `new` 的时候必须是 `_capacity + 1`，因为 C 风格字符串必须以 `\0` 结尾，且`capacity`和`size`的数值不包括\0，表示有效的空间和字符
### 析构函数

```cpp
~string() {
    delete[] _str;
    _str = nullptr;
    _size = _capacity = 0;
}
```
## 4.5 拷贝构造与赋值重载
### 拷贝构造 (Copy Constructor)
```cpp
string(const string& str)
{
	_str = new char[str._capacity + 1];
	strcpy(_str, str._str);
	_size = str._size;
	_capacity = str._capacity;
}
```
### 赋值运算符重载 (Assignment Operator)
```cpp
string& operator=(const string& str)
		{
			if (this != &str)	// 考虑自己等于自己，不能把自己删除了
			{
				delete[] _str;	
				_str = new char[str._capacity + 1];
				strcpy(_str, str._str);
				_size = str._size;
				_capacity = str._capacity;
				return *this;
			}
		}
}
```
需要注意的是：
1. 需要考虑自己等于自己，不能把自己删除了
2. 要赋值的字符串可能是已经有字符的字符串，需要先清除
## 4.6 扩容 Reserve 与 Push_Back

```cpp
void string::reserve(size_t n) 
{
    if (n > _capacity) 
    {
        char* tmp = new char[n + 1];
        strcpy(tmp, _str); 
        delete[] _str;     
        _str = tmp;
        _capacity = n;
    }
}

void string::push_back(char ch) 
{
    if (_size == _capacity) 
    {
        // 扩容策略：如果是0则开4，否则翻倍
        reserve(_capacity == 0 ? 4 : _capacity * 2);
    }
    _str[_size++] = ch;
    _str[_size] = '\0'; 
}
```
需要注意的是：
1. 扩容要多开一个空间放\0
2. 尾插扩容之前要考虑字符串可能是空串，对`_capacity`要另外处理
## 4.6 其他简单功能函数
```cpp
void swap(string& str)
{
	_str = new char[str.capacity() + 1];
	std::swap(_str, str._str);
	std::swap(_size, str._size);
	std::swap(_capacity, str._capacity);
}
void clear()
{
	_str[0] = '\0';
	_size = 0;
}
char* c_str()	const
{
	return _str;
}
bool operator<(const string& s1, const string& s2)
	{
		return strcmp(s1.c_str(), s2.c_str()) < 0;
	}
bool operator<=(const string& s1, const string& s2)
	{
		return s1 < s2 || s1 == s2;
	}
bool operator>(const string& s1, const string& s2)
	{
		return !(s1 <= s2);
	}
bool operator>=(const string& s1, const string& s2)
	{
		return !(s1 < s2);
	}
bool operator==(const string& s1, const string& s2)
	{
		return strcmp(s1.c_str(), s2.c_str()) == 0;
	}
bool operator!=(const string& s1, const string& s2)
	{
		return !(s1 == s2);
	}
```
## 4.7 Insert 函数的“无符号整数陷阱” 
在 `insert` 函数中移动数据时，极易写出死循环代码。
```cpp
void insert(size_t pos, char ch)
{
	size_t end = _size;
	while (end >= pos) 
	{ 
	    _str[end + 1] = _str[end];
	    end--;
	}
}

```

`end` 是 `size_t` (无符号整数)。当 `end` 减到 0 时，再执行 `end--`，它不会变成 -1，而是变成一个巨大的正整数（如 4294967295）。导致 `end >= pos` 永远为真，程序越界访问内存崩溃。
<br>
***正确写法***
```cpp
void string::insert(size_t pos, char ch)
{
	assert(pos < _size);
	if (_size == _capacity)
	{
		reserve(_capacity == 0 ? 4 : _capacity * 2);
	}
	size_t end = _size + 1;	// end指向\0的下一个位置
	//挪动数据
	while (end > pos)
	{
		_str[end] = _str[end - 1];
		end--;
	}
	_size++;
	_str[pos] = ch;
}
```

或者强制转换为 `int` 进行比较，但保持无符号风格更佳
两种函数重载
```CPP
void string::insert(size_t pos, char ch)
{
	assert(pos < _size);
	if (_size == _capacity)
	{
		reserve(_capacity == 0 ? 4 : _capacity * 2);
	}
	size_t end = _size + 1;	// end指向\0的下一个位置
	//挪动数据
	while (end > pos)
	{
		_str[end] = _str[end - 1];
		end--;
	}
	_size++;
	_str[pos] = ch;
}
	
void string::insert(size_t pos, const char* str)
{
	assert(pos < _size);
	size_t len = strlen(str);
	if (_size + len > _capacity)
	{
		// 大于2倍，需要多少开多少，小于2倍按2倍扩
		reserve(_size + len > 2 * _capacity ? _size + len : 2 * _capacity);
	}
	size_t end = _size + len;
	// 挪动数据
	while (end > pos + len - 1)
	{
		_str[end] = _str[end - len];
		end--;
	}
	_size += len;
	for (int i = 0; i < len; i++)
	{
		_str[pos + i] = str[i];
	}
}
```
插入字符串的重载和插入字符的思路是一样的，计算str的长度然后先移动，但是挪动数据的结束条件是`end > pos + len - 1`，记住**左闭右开相减就是个数，左闭右闭相减+1才是个数**
## 4.8 Substr 的溢出陷阱
```cpp
string string::substr(size_t pos, size_t len)
{
	if (len > _size - pos)
	{
		len = _size - pos;
	}
	string tmp;
	tmp.reserve(len);
	for (size_t i = 0; i < len; i++)
	{
		tmp += _str[pos + i];
	}
	return tmp;
}
```

不过很多人会这样写
```cpp
if (pos + len > _size) ...
```

`len` 的默认值是 `npos`（即 `size_t` 的最大值）。如果用户只传了 `pos`，`pos + len` 会发生加法溢出，导致结果变小，条件判断失效。改为减法数学上等价但是可以避免这个问题
## 4.9 IO 流重载的优化

### `operator>>` 的实现
```cpp
istream& operator>>(istream& in, string& s)
	{
		s.clear();
		char ch = in.get();
		//in >> ch;			in会跳过分隔符（‘ ’ ‘\n’ ，ch就读不到分隔符）
		while (ch != ' ' && ch != '\n')
		{
			s += ch;
			ch = in.get();
		}
		return in;
	}
```
不能用`in >> ch`因为空格和换行符被认为是分隔符会直接略过，循环永远都没有结束条件，需要用`getline`函数
但是这样的写法每插入一个字符就需要考虑扩容，扩容太频繁效率非常低，提升效率的一个方法就是使用缓存，先缓存一部分字符，缓存满了输入到目标字符串里，再接着读字符

```cpp
istream& operator>>(istream& in, string& s) {
    s.clear();
    const int N = 256;
    char buff[N] = { 0 }; // 栈上开辟缓冲区
    char ch = in.get();
    int i = 0;
    while (ch != ' ' && ch != '\n') {
        if (i == N - 1) {
            buff[i] = '\0';
            s += buff; // 攒够一批再追加
            i = 0;
        }
        buff[i++] = ch;
        ch = in.get();
    }
    buff[i] = '\0';
    s += buff;
    return in;
}
```
需要注意while循环结束之后buff里可能还剩字符，需要再追加一次
如果每读一个字符就调用一次 `push_back` 或 `+=`，可能会导致频繁的内存重新分配（扩容）。使用局部数组 `buff` 积攒 256 个字符后再一次性追加，可以显著减少 `new/delete` 的次数，提高 IO 效率
## 4.10 find函数
```CPP
size_t string::find(char ch, size_t pos)
{
	assert(pos < _size);
	for (size_t i = 0; i < _size - pos; i++)
	{
		if (_str[pos + i] == ch)
			return pos + i;
	}
	return npos;
}
```
和标准库一样的风格，没找到返回npos
## 4.11 现代写法的优化
对于复制拷贝构造函数和赋值重载有可以优化的点，在章节**4.6**中实现了交换函数，尽量复用之前写过的代码，常把返回信息更少、功能更单一的函数作为“底层实现”，让返回信息更多、功能更复杂的函数去调用它，可以认为让别人干活
### 拷贝复制构造优化
深拷贝str的字符串的tmp，与全部是缺省值的this进行替换，用别的函数完成业务
```cpp
string(const string& str)
{
	string tmp(str._str);
	swap(tmp);
}
```
因为要拷贝的字符串是空串，交换给临时变量，临时变量出函数作用域就销毁，可以对空指针释放，代码十分简洁
### 赋值运算符重载优化
和上面一样，可以用交换函数
```cpp
string& operator=(const string& str)
	{
		if (this != &str)
		{
			string tmp(str._str);
			swap(tmp);
		}
		return *this;
	}
```
但是这样还是有一点繁琐，进一步简化代码
```cpp
string& operator=(string tmp)
{
	swap(tmp);
	return *this;
}
```
传值调用临时变量会拷贝构造，这样拷贝出来的临时变量进行交换，临时变量出函数作用域依旧销毁，创建一个临时变量的拷贝避免了自己赋值给自己的情况
### swap函数
在C++库中官方对于string的交换函数是全局函数，其本质是一个交换的成员函数再重载成全局
如果只重载成成员函数需要像结构体那样使用，可能会导致一些不太懂的程序员用正常的调用函数方法，结果调用算法库的交换函数，直接交换会造成浅拷贝问题。官方的目的在于实现**泛型编程**，对于每种数据类型都有一样的调用方法


---

# 五、GCC 与 MSVC 的底层差异及总结

## 5.1 VS (MSVC) 的短字符串优化 (SSO)

在 Visual Studio 中，`sizeof(string)` 通常是 **40 字节**（x64 环境）。
VS 采用了 **Small String Optimization (SSO)** 技术。它在 string 对象内部预留了一个约 15-16 字节的栈上缓冲区（Buffer）。

- **当字符串很短时**（如 "hello"）：直接存在这个 Buffer 里，**不需要堆内存分配**，速度极快。
- **当字符串很长时**：才会去堆上 `new` 空间，此时 Buffer 用来存储指针、size 和 capacity。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/62808be771ad4d49bb082b13edb7aa77.png#pic_center)



## 5.2 GCC 的写时拷贝 (COW) vs 深拷贝

### 早期 GCC
采用 **Copy-On-Write (COW)** 技术。即引用计数的写时拷贝，多个 string 对象共享同一块内存，并计数，并遵循以下规则：
- 计数大于1
	- 有对象需要析构，只进行计数减一
	- 有对象赋值一样的字符串，新字符串应用到相同的常量字符串上，计数加一
- 计数等于1
	- 有对象析构，说明没有共享的对象，直接释放内容
	- 有对象赋值一样的字符串，新字符串应用到相同的常量字符串上，计数加一
- 其中的某对象需要修改，进行深拷贝，更改一个对象引用进行修改
其本质是博弈论，只要不修改效率就很高
### 现代 GCC (C++11 后)
已经**废弃了 COW**，转而采用类似 VS 的 SSO 或者纯粹的深拷贝策略，以符合 C++11 标准对右值引用和移动语义的支持。

# 结语
string作为在STL库出现之前就有的，但是其设计思路和STL库非常相似，所以一般来说会被归于一类，但是正是在STL库之前出现，很多函数接口时没必要的，比如`lenth`和`size`，但也是为了兼容做出的让步，后续新出的其他库接口上就很简洁，总之学习第一个STL库，还是需要多加练习

---

## 相关笔记 / 相关链接

- 相关目录：[[C++目录]]
- 相关笔记：[[C++/类和对象全解：你想知道的这里都有，默认成员函数、友元、隐式类型转换、const和static修饰]]（前置：默认成员函数与深浅拷贝）
