@[toc]
~~一个大一新生的浅谈~~
我的[gitee仓库](https://gitee.com/warm-flame-core)记录了我的作业和简单算法，还有博客的markdown原文
# 数据类型
## 综述
用一个列表表示数据的各种类型：

- **内置类型**
	- 字符型
	- 整形
	- 浮点型
	- 布尔类型
- **自定义类型**
	- 数组
	- 结构体
	- 枚举
	- 联合体
## 字符型
<h3  id="1"></h3>
字符的英文单词是`character`，用`char`表示字符串类型

```c
 char    //character
 [signed] char   //有符号的
unsigned char   //⽆符号的
```
## 整形
整形的英文单词是`integer`，用`int`表示整形类型

```c
 //
短整型  
short [int]
 [signed] short [int]
 unsigned short [int]
 //
整型
int
 [signed] int
 unsigned int
 //
⻓整型 
long [int]
 [signed] long [int]
 unsigned long [int]
 //
更⻓的整型
//C99中引⼊
long long [int]
 [signed] long long [int]
 unsigned long long [int]
```
## 浮点型
可以认为是小数，用`float`表示浮点型

```c
 float		//普通的浮点数
 double	//精度更高的浮点数
 long double	//精度还要更高的浮点数
```
## 布尔类型
在C99中引入了布尔类型`_Bool`(`B`是大写)，表示真假。布尔变量的取值只有两种：`true`或者`false`。在条件判断中很有用
使用布尔类型必须包含头文件`<stdbool.h>`

> ~~但是我还没有碰到过…………~~ 

## 各种类型的长度
使用不同类型的数据有不同的长度，储存的变量就有差异，如果看不同数据类型的长度呢，用sizeof关键字。
## `sizeof`
**`sizeof`是一个关键字**，用来查询一个数据类型的长度，如下例：

```c
#include <stdio.h>
int main()
{
	printf("%zd\n", sizeof(char));
	printf("%zd\n", sizeof(_Bool));
	printf("%zd\n", sizeof(short));
	printf("%zd\n", sizeof(int));
	printf("%zd\n", sizeof(long));
	printf("%zd\n", sizeof(long long));
	printf("%zd\n", sizeof(float));
	printf("%zd\n", sizeof(double));
	printf("%zd\n", sizeof(long double));
	return 0;
}
```
运行各种如图：
![sizeof的运行结果](https://i-blog.csdnimg.cn/direct/e54a9235912e43f19baeb36e47c29f4b.png)
这是在我的电脑上运行的结果，表示各个类型的数据长度，单位是字节（B）
<center><font color="red"><strong>注意：在不同编译器的运行结果可能不同

sizeof在后面求数组长度时很有用，**建议掌握**
### sizeof的表达式不会计算
传入给sizeof的参数不会计算，比如`a`是整形变量，`b`是浮点类型变量，将`a=b+1`传入sizeof的时候只与`a`有关，计算式直接会被忽略。其原理是在编译时sizeof就已经传入了，后面的计算式是在程序运行的时候才会计算。
# `signed`和`unsigned`
在实际生活中，有一些数据是有正负的，比如用卡尔文温度表示的温度就只有正数，而用摄氏度表示的温度就有正负。在C语言中，用`signed`表示有符号，用`unsigned`表示无符号,修饰在变量类型的前面。具体写法在本篇前面已经说过<a href="#1" target="_self">点击此处1</>

不再赘述。
## 为什么要这样规定。
整数变量声明为`unsigned`的好处是，同样⻓度的内存能够表⽰的最⼤整数值，增⼤了⼀倍。⽐如，16位的`signed short int`的取值范围是：

> -32768~ 32767
> **最⼤是32767**

⽽·unsigned short int· 的取值范围是：

> 0~65535 
> 最⼤值增⼤到了65535

32位的signed int的取值范围详细可以查看IDE文件中`limits.h` 中给出的定义。
### 怎么查看`limits.h`
下载好IDE之后，建议用everything软件搜索`limits.h`，~~文件资源管理器自带的搜索简直是史山~~ ，如下图所示：
![搜索结果](https://i-blog.csdnimg.cn/direct/ce5dd9db389e4226bb6605f8f4ed7634.png)
打开就可以看到如图所示：
![头文件详细](https://i-blog.csdnimg.cn/direct/8fb070f6390f4dca895aa867fdec03a5.png)
## 书写简便用法
默认情况下，除了char类型，都是带符号的数据类型，
比如`int`和`signed int`是等价，如果想要表示带符号的数据类型可以直接不修饰。
在整型变量`int`里，`unsigned  int`等价于`unsigned`
### 注意事项
在`char`类型中，**<mark style="color: red;">`char`不一定等价于`signed char`</mark>**，这是由系统决定的，不能一概而论
## 变量类型的最值表示
在实际运用中，我们常常需要使用数据类型的最大值or最小值，这个时候需要借助一个常量符号。常见的常量符号如下：

> `SCHAR_MIN` ，`SCHAR_MAX`：signed char的最⼩值和最⼤值。
> `SHRT_MIN`，`SHRT_MAX`：short的最⼩值和最⼤值。
> `INT_MIN`，`INT_MAX`：int的最⼩值和最⼤值。
> `LONG_MIN`，`LONG_MAX`：long的最⼩值和最⼤值。
> `LLONG_MIN`，`LLONG_MAX`：longlong的最⼩值和最⼤值。 
> `UCHAR_MAX`：unsigned char的最⼤值。 
> `USHRT_MAX`：unsigned short的最⼤值。 
> `UINT_MAX`：unsigned int的最⼤值。
> `ULONG_MAX`：unsigned long的最⼤值。 
> `ULLONG_MAX`：unsigned longlong的最⼤值。

由于每一种编译器对于数据类型的长度都不一样，若要查询编译器最值常量大小，`limits.h`⽂件中说明了整型类型的取值范围。`float.h`这个头⽂件中说明浮点型类型的取值范围。
# 变量与常量
## 变量
### 创建变量
在前面已经介绍了关于数据类型，在创建变量的时候需要在变量名前面写明数据类型，如：
```c
char arr;
int a;
float b;
```
其中变量名应该注意以下几点：
• 只能由字⺟（包括⼤写和⼩写）、数字和下划线（`_`）组成。
• 不能以数字开头。
• ⻓度不能超过63个字符。
• 变量名中区分⼤⼩写的。
• 变量名不能使⽤关键字。
### 变量初始化
如果在前面的基础上，对每个字符进行赋值，就是变量的初始化。如下示例：
```c
char arr = { 0 };
int a = 1;
float b = 3.14;
```
### 变量的分类
变量分为全局变量和局部变量，全局变量是写在函数之外的，局部变量是在函数内的。如示例所示：

```c
#include<stdio.h>
int a = 0;	//全局变量
int main()
{
	int b = 1;	//局部变量
	printf("%d\n", a);
	printf("%d\n", b);
	return 0;
}
```
#### 注意事项
当局部变量与全局变量名字重复时，在函数内优先使用局部变量。
而且函数内的变量只能在函数内使用（除了用`static`修饰的变量）
# 各种操作符
## 双目操作符
需要两个操作数的操作符叫`双目操作符`，有`+  -  *  /  %`
### `+  -  *` 操作符
这些操作符都没有什么注意事项，像现实生活中使用就可以了

```c
int main()
{
	int n = 1 + 2;
	int a = 1 - 2;
	int b = 2 * 3;
	printf("%d\n", n);
	printf("%d\n", a);
	printf("%d\n", b);
	return 0;
}
```
### 除法操作符
除法中可能出现小数的商，在C语言里除号的两边至少有一个数据是浮点数时计算才会有小数结果，要不然输出结果小数部分会丢失，如下例：

```c
#include<stdio.h>
int main()
{
	float x = 6 / 4;
	int y = 6 / 4;
	printf("%f\n", x);
	printf("%d\n", y);
	return 0;
}
```
观察输出结果：
![输出结果](https://i-blog.csdnimg.cn/direct/917e5ba1ea574adb924ba60a7e6398c1.png)
哪怕浮点数类型的X用`%f`输出时结果依旧小数部分被丢弃，正是因为x进行的时整数除法。如果需要表示出小数，可以这样写：

```c
#include<stdio.h>
int main()
{
	float x = 6.0 / 4;
	int y = 6 / 4.0;
	printf("%f\n", x);
	printf("%d\n", y);
	return 0;
}
```
输出结果如图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/7a54092d0a47473e82aad1b41560f005.png)
### 求余操作符
`%`是用来求两个数相除的余数的符号，其中余数的正负号由第一个运算数的正负号决定：

```c
#include<stdio.h>
int main()
{
	printf("%d\n", 11 % 5);
	printf("%d\n", -11 % -5);
	printf("%d\n", 11 % -5);
	printf("%d\n", -11 % 5);
	return 0;
}
```
输出结果为：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/a56ed4ab7b774e60ba370fed9225b3f9.png)
## 赋值操作符
`=`为赋值操作符。在前面介绍的变量的初始化就是一种赋值，这里不再赘述。
### 连续赋值
几个变量可以连续赋值，在代码里时从右往左进行赋值。

```c
int a = 0;
int b = 1;
int c = 0;
c = b = a + 3;
```
### 复合赋值符
在实际应用中，我们常常需要对一个变量进行自身的相加减，比如求和的时候。在C语言中支持这样写：

```c
int a = 0;
a = a + 2;
a = a - 2;
```
#### 简便写法
对于上面的示例，我们可以这样简便：

```c
int a = 0;
a += 2;
a -= 2;
```
除了加和减，其他运算也有简便写法也是类似，不再赘述。
## 单目操作符
### 前置/后置++
这是一种用于传递值的简便写法。
#### 前置++
观察如下代码：

```c
#include<stdio.h>
int main()
{
	int a = 0;
	int b = ++a;
	printf("a=%d b=%d", a, b);
	return 0;
}
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/2b37a7a5c52a4e13891354c5fd019af6.png)
其代码详细为先对`a`＋1，再传给`b`，相当于a和b都+1了
口诀为：`加+1，后使用`
#### 后置++
观察代码：

```c
#include<stdio.h>
int main()
{
	int a = 0;
	int b = a++;
	printf("a=%d b=%d", a, b);
	return 0;
}
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/9314935ecc6b42d6811028b80fe619c9.png)
其代码详细为a的值先传给b，a再＋1。
其口诀为：`先使用，后+1`
### 前置/后置--
其原理和口诀与前置/后置++类似，不在赘述。
# 强制类型转换
有的时候我们需要一个结果的整数部分，那么可以对这个结果进行强制类型转换：

```c
int a = (int) 3.14;
```
打印a的结果就时3。但是平常写代码时，我们应该尽量减少使用强制类型转换，因为这个操作会降低精度。