---
tags:
  - 编程
  - python
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
学校比赛要用py，正好有一些编程的基础，快速入门一下py
# 与C语言不一样的地方
1. 定义变量不需要声明类型，python里的变量都是动态变量
```python
b = 'hello'  
print(type(b))
```
其中type函数用来查看类型
2. py中整数除法会有小数，与C系列语言不一样
```python
print(1 / 3)
```
结果是`0.3333333333333333`
3. 语句结束不需要分号，一行就表示一条语句
4. 单引号和双引号是一样的
```python
b = 'hello'  
print(type(b))  
c = "hello"  
print(len(c))
```
其中len函数可以求字符串长度
3. 字符串可以直接拼接，不需要显式调用函数
```python
b = 'hello'  
print(type(b))  
c = "hello"  
print(len(c))  
# 字符串可以拼接  
print(b + c)
```
3. 布尔类型不再是0和非0
```python
print(type(True))  
print(type(False))
```
其结果为
```text
<class 'bool'>
<class 'bool'>
```
3. 获取输入用input
```python
num = 0  
num = input('请输入一个整数：')  
print(num)
```
**需要注意的是返回值是字符类型，如果需要参与运算需要强制类型转换**
3. 字符串可以直接比较，不需要显式调用函数
```python
a = 'hello'  
b = 'world'  
print(a == b)  
print(a != b)
```
3. 逻辑运算符支持连续比较，C语言不可以连续比较
```python
a = 10  
b = 20  
c = 30  
print(a < b and b < c)  
print(not (a < b and b < c))  
print(not (a < b < c))
```
3. 支持多元赋值
```python
a, b = 10, 20  
print(a)  
print(b)  
a, b = b, a  
print(a)  
print(b)
```
3. py用缩进表示代码块，C语言用大括号
```python
a = int(input("输入一个整数"))  
if a % 2 == 0:  
    print(f'{a}是偶数')  
else:  
    print(f'{a}是奇数')
```
***缩进在py中十分重要，一定要养成良好习惯要么按Tab要么就按空格***
3. 用pass表示空语句
当多分支条件中还有你没想好的结果，可以用pass让语法先过用于测试
```python
a = int(input("请输入一个整数:"))  
if a != 1:  
  pass  
else:  
  print("hello")
```
3. range函数表示迭代，用于for循环
```python
# range函数生成迭代，左闭右开  
for i in range(1, 11):  
  print(i)  
# 可以规定步长  
for i in range(0, 11, 2):  
  print(i)  
# 步长可以是负数  
for i in range(10, 0, -1):  
  print(i)
```
3. 三元表达式与C语言的三目操作数不一样
```python
x = 4  
y = 5  
small = x if x < y else y  
print(small)
```
4. 函数定义用def关键字，且不需要写返回值类型
```python
def year(x):  
  return 2026 - x  
print(year(int(input())))
```
1. 函数的参数有位置参数和关键字参数
- 位置参数
```python
def func(name, age, height, weight):  
  print('name =', name)  
  print('age =', age)  
  print('height =', height)  
  print('weight =', weight)  
func('小贝', 18, '155cm', '40kg')
```
- 关键字参数
```python
def func(name, age, height, weight):  
  print('name =', name)  
  print('age =', age)  
  print('height =', height)  
  print('weight =', weight)  
func(age = 18, weight = '40kg', name = '小贝', height = '155cm')
```
1. 函数参数支持缺省值
```python
def func(name, age, weight, height = '155cm'):  
  print('name =', name)  
  print('age =', age)  
  print('height =', height)  
  print('weight =', weight)  
func('小贝', 18, '40kg')  
func('小贝', 18, '40kg', '158cm')
```
4. 函数支持返回多参数 
```python
def return_menu(food1, food2, food3, food4):  
    return food1, food2, food3, food4  
print(return_menu('蒸羊羔', '蒸熊掌', '蒸鹿尾儿', '烧花鸭'))
```
返回的参数是元组类型，这种类型后面会学到
# 总结
以上是我认为的C语言和py的不同之处

---

## 相关笔记 / 相关链接

- 前置知识：[[C语言/初识C语言]]、[[C语言/printf中的格式输出]]
- 同主题对照：[[C语言/递归思想]]、[[C语言/指针]]
- 所属目录：[[Python目录]]、[[C语言目录]]