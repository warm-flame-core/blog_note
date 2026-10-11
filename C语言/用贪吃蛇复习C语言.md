---
tags:
  - 形态/项目
aliases:
  - 用贪吃蛇复习C语言
日期: 2026年4月17日
---

# 引言
作者最近终于是学完C语言基础语言和一点点链表的知识，必须大展身手一下，参照了一些贪吃蛇的开源，自己写了一个简单的贪吃蛇，不仅可以拿来练手，还可以复习之前学过的知识，特此写一篇文章来分享
程序界面如图所示：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/08d9ae53367a4c37b00594bb1d69c6e0.png#pic_center)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/93113e8ff0944ee7909c61c72c24ad28.png#pic_center)
实现贪吃蛇需要以下知识：函数、枚举、结构体、动态内存管理、预处理指令、链表、Win32API、位运算、指针等，不过没懂也没有关系，相关知识在本文也会做简单介绍。
本文会介绍游戏的流程和每个过程是怎么用函数实现的，另外还会介绍宽字符以及C语言的本地化
# 对控制台的深入理解
在我们学习C语言的过程中，结果的呈现总是在一个黑框框里面，这样的黑框框我们称之为`控制台`，本质是一个终端，在Windows里打开cmd，我们可以对这个控制台做各种操作，比如修改控制的标题，例：

> title 贪吃蛇
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/4325534becff4aeebea502f5f02dd144.png#pic_center)

与此同时我们还可以控制窗口的打开，例：

> mode con cols=100 lines=30
> 即结果会把窗口改成100列，30行

以上这些系统命令在C语言里使用需要用system语句，即

> system(" 控制语句 ");

而使用这个语句需要包含头文件`Windows.h`
# win32API简单介绍
## 前言
电脑上的Windows系统提供了各种服务，你可以用这些服务对程序进行控制，下面我们简单介绍几个这次贪吃蛇会用到的API。
## 窗口的坐标类型COORD
**COORD**是win32API里定义的结构体，结构体成员包含窗口的X和Y坐标，其中坐标原点在窗口左上角，X轴正方向水平向右，Y轴正方向竖直向下![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/943ed81390584b6e9e3fc27d16e75be9.png#pic_center)
## 窗口句柄HANDLE

**HANDLE**类型指的是窗口的句柄，可以认为一个窗口对应一个句柄，我们通过句柄可以操控窗口，其本质是**void*指针**
## 获取窗口句柄GetStdHandle
**GetStdHandle函数**可以用来获取当前正在运行的程序的句柄，其返回值也是HANDLE，其函数原型为

> HANDLE GetStdHandle(DWORD nStdHandle);

其中函数参数有以下情况：
- TD_INPUT_HANDLE 
- STD_OUTPUT_HANDLE 
- STD_ERROR_HANDLE 

如果我们只是想获取窗口的句柄，直接这样写就可：

```c
HANDLE hOutput = NULL;
hOutput = GetStdHandle(STD_OUTPUT_HANDLE);
```
将获取的句柄传给HANDLE类型的hOutput
## 控制台光标类型CONSOLE_CURSOR_INFO
这个结构体的成员包括光标的大小和可见性

```c
typedef struct _CONSOLE_CURSOR_INFO {
	DWORD dwSize;
	BOOL  bVisible;
} CONSOLE_CURSOR_INFO, *PCONSOLE_CURSOR_INFO;
```

```c
CursorInfo.bVisible = false; //隐藏控制台光标
```

## 获取控制台光标信息GetConsoleCursorInfo
GetConsoleCursorInfo函数用于获取光标的有关信息，返回值为布尔类型

```c
BOOL WINAPI GetConsoleCursorInfo(
	HANDLE      				hConsoleOutput,
	PCONSOLE_CURSOR_INFO lpConsoleCursorInfo
);
```
函数参数有两个，一个是窗口句柄，一个是光标类型

## 隐藏光标
综上所述，如果需要程序里不出现光标，那么可以这样写

```c
HANDLE hOutput = GetStdHandle(STD_OUTPUT_HANDLE);
//隐藏光标操作
 
CONSOLE_CURSOR_INFO CursorInfo;
GetConsoleCursorInfo(hOutput, &CursorInfo);//获取控制台光标信息
 
CursorInfo.bVisible = false; //隐藏控制台光标
 
SetConsoleCursorInfo(hOutput, &CursorInfo);//设置控制台光标状态
```
## 定位光标位置SetConsoleCursorPosition
SetConsoleCursorPosition函数用来定位光标位置，其函数原型为：

```c
BOOL WINAPI SetConsoleCursorPosition(
HANDLE hConsoleOutput,
COORD  pos
);
```
函数传入提前设定好的pos坐标和窗口句柄，就可以定位光标位置
为了方便使用，我们可以封装一个函数：

```c
void SetPos(short x, short y)
{
	COORD pos = { x, y };
	HANDLE hOutput = NULL;
	hOutput = GetStdHandle(STD_OUTPUT_HANDLE);
	SetConsoleCursorPosition(hOutput, pos);
}
```
## 获取按键情况GetAsyncKeyState
GetAsyncKeyState函数可以用来获取按键是否按下和是否按过

> SHORT GetAsyncKeyState(int vKey);
函数参数为每个键位的虚拟键值，再上一次调用函数开始，返回的short类型里，最高位的1或者0代表这个键按下或者抬起，最低位的1或者0代表这个键位被按过或者没有被按过。我们可以用按位与`&`预算进行检测，按位与的规则是两个数的二进制对应位进行计算：只有两个都是1才会返回1
我们可以将返回的值与`1`进行按位与，这样就能检测最低为是不是1，即检测这个键是否被按下，我们可以定义一个宏方便使用

即：

> "#define KEY_PRESS(VK)  ( (GetAsyncKeyState(VK) & 0x1) ? 1 : 0 )"
如果将这个宏放入循环中，则在循环检测这个是否被按过，即检测这个键

虚拟键值详细见[微软虚拟键值描述](https://learn.microsoft.com/zh-cn/windows/win32/inputdev/virtual-key-codes)
# C语言的本地化以及宽字符
## 本地化
在控制台默认打印字符的时候是一个字符为一个光标宽度，如果我们用默认打印方式会显得游戏很小，而在中文环境下，汉字打印时两个光标宽度，那我们这次的项目就可以按两个光标宽度打印。在C语言中，为了能够使其与国际化接轨，C语言支持修改程序对应的格式，其中修改格式需要包含头文件`locale.h`
头文件提供了控制 C 语言标准库本地化行为的函数和宏，核心是setlocale函数，用于修改程序的本地化环境

> char* setlocale (int category, const char* locale);

函数参数有两个：
1. category：指定要修改的本地化类别
-  LC_ALL：修改所有类别（字符编码、数字、时间、货币等），最常用；
- LC_CTYPE：仅修改字符处理类别（重点，解决宽字符显示）；
2. locale：指定目标本地化环境，常用取值：
- "C"：C 语言默认环境，不支持宽字符，程序启动时默认调用setlocale(LC_ALL, "C")；
- ""：本地环境，自动适配当前操作系统的编码（Windows 为 GBK，支持中文和宽字符）。
- 如果要适配自身系统格式，那么在main函数开头这样写：

```c
#include <locale.h>
int main() 
{
    setlocale(LC_ALL, "");
    return 0;
}
```
## 宽字符输出
wchar_t是 C 语言的宽字符类型，用于存储宽字符
打印宽字符使用`wprint函数`
定义宽字符 / 宽字符串时，必须在字面量前加前缀 L，告诉编译器这是宽字符，否则会被当作普通 ASCII 字符处理，即：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/20e78ccc20fe44759554d00cb1d3b730.png#pic_center)
# 贪吃蛇实现
贪吃蛇的核心逻辑围绕蛇身的移动和增长展开，蛇身的每个节点需要记录坐标，且吃食物后需要动态增长，链表是最适合的存储结构（动态增删、链式存储），同时通过结构体封装蛇的整体信息，枚举定义蛇的方向和游戏状态，让代码逻辑更清晰。
我们用三个文件来管理代码

- `game.c`->游戏的整体逻辑
- `Snake.c`->游戏涉及的函数的实现
- `Snake.h`->游戏涉及的函数的声明
## 定义蛇的方向
既然写了枚举，而且蛇的方向时可以一一列举的，那么为了方便管理，我们用枚举类型列举蛇的方向

```c
enum DIRECTION {
    UP = 1,
    DOWN,
    LEFT,
    RIGHT
};
```
## 定义游戏状态
还是一样，我们对游戏状态也可以一一列举，于此同时通过判断状态来判断游戏是否结束

```c
enum GAME_STATUS {
   	NORMAL,
	KILL_WALL,
	KILL_SELF,
	ENDING
};
```
## 定义蛇的身体
游戏运行时我们需要知道蛇的各个节点的坐标才可以打印蛇，实现移动和增长，而这种已知头就能知道尾巴的数据结构，我们可以用单链表，其单链表本质就是**结构体**其成员包括数据和下一个节点的指针
其一般结构为

```c
struct SeqList {
    Datetype Date; 
    struct SeqList* next;  // 指向下一个节点的指针
};
```

在本文示例中，其数据就为坐标：

```c
typedef struct SnakeNode {
    int x;  // 节点X坐标
    int y;  // 节点Y坐标
    struct SnakeNode* next; 
} SnakeNode, *pSnakeNode;
```
为了方便后续使用，我们可以用typedef语句重新定义变量类型的名字
## 封装蛇的信息
包括蛇身链表的头指针、食物节点指针、当前方向、游戏状态、分数、食物权重、移动速度等，让数据管理更集中。这种思想很想面向对象编程，使其代码更易解读

```c
typedef struct Snake {
    pSnakeNode _pSnake;      // 蛇身链表的头指针（指向蛇头）
    pSnakeNode _pFood;       // 食物节点的指针
    enum DIRECTION _Dir;     // 蛇的当前移动方向
    enum GAME_STATUS _Status;// 游戏当前状态
    int _Socre;              // 当前得分
    int _foodWeight;         // 每个食物的分数（默认10分，加速加分，减速减分）
    int _SleepTime;          // 蛇移动一步的休眠时间（单位：毫秒，时间越短速度越快）
} Snake, *pSnake;
```
## 一些游戏常量
书写代码的时候，有一些常量我们经常用到，这个时候我们就可以提前定义好，这样不仅方便书写，可读性也更高

```c
#define WALL L'■'
#define BODY L'●'
#define FOOD L'★'
#define POS_X 6
#define POS_Y 20
```
其中`POS_X`和`POS_Y`是初始的蛇头坐标
## 游戏逻辑
游戏分为游戏开始：用于游戏的初始化，打印各项信息；游戏进行：移动蛇，随机生成食物，判断是否死亡；游戏结束：回收内存，打印结束信息我们按照大流程再细化一些小流程
### 游戏开始GameStart
其GameStart函数里面包含：设置控制台属性、隐藏光标、打印欢迎信息(WelcomeToGame)、绘制游戏地图(CreateMap)、初始化蛇身(InitSnake)、创建食物(CreateFood)

```c
void GameStart(pSnake ps)
{
	system("mode con cols=100 lines=30");
	system("title 贪吃蛇");
	HANDLE hOutput = GetStdHandle(STD_OUTPUT_HANDLE);
	CONSOLE_CURSOR_INFO CursorInfo;
	GetConsoleCursorInfo(hOutput, &CursorInfo);
	CursorInfo.bVisible = false; 
	SetConsoleCursorInfo(hOutput, &CursorInfo);
	WelcomeToGame();
	CreateMap();
	InitSnake(ps);
	CreateFood(ps);
}
```
### 欢迎界面（WelcomeToGame）
介绍游戏规则

```c
void WelcomeToGame()
{
	SetPos(40, 15);
	printf("欢迎来到贪吃蛇游戏");
	SetPos(40, 25);
	system("pause");
	system("cls");
	SetPos(25, 12);
	SetPos(25, 13);
	printf("用上，下，左，右分别控制蛇的移动，Z为加速，左边ctrl为减速\n");
	printf("加速将能得到更高的分数。\n");
	SetPos(40, 25);
	system("pause");
	system("cls");
	SetPos(60, 15);
}
```
其中系统命令`cls`表示清理屏幕
### 绘制地图（CreateMap）
本文示例的地图为**58列27行**的地图，由于我们使用了宽字符，那么其打印时一步就为2，且使用之前封装的定位坐标的函数
上边界：X 从 0 到 56（步长 2，宽字符），Y=0
下边界：X 从 0 到 56（步长 2），Y=26
左边界：X=0，Y 从 1 到 25
右边界：X=56，Y 从 1 到 25

```c
void CreateMap()
{
	int i = 0;
	SetPos(0, 0);
	for (i = 0; i < 58; i += 2)
	{
		wprintf(L"%c", WALL);
	}
	SetPos(0, 26);
	for (i = 0; i < 58; i += 2)
	{
		wprintf(L"%c", WALL);
	}
	for (i = 1; i < 26; i++)
	{
		SetPos(0, i);
		wprintf(L"%c", WALL);
	}
	for (i = 1; i < 26; i++)
	{
		SetPos(56, i);
		wprintf(L"%c", WALL);
	}
}
```
### 初始化蛇身（InitSnake）
使用头插法进行初始化，其原理是使用malloc语句创建好节点后：新创建的节点作为新的蛇头，指向原来的蛇头，链表头指针始终指向蛇头，即：

```c
pSnakeNode pcur = NULL;
    int i = 0;
    // 1. 循环创建5个蛇身节点，头插法
    for (i = 0; i < 5; i++) 
    {
        // 动态分配节点内存，检测是否分配成功
        pcur = (pSnakeNode)malloc(sizeof(SnakeNode));
        if (cur == NULL) 
        {
            perror("InitSnake malloc error");
			exit(1);
        }
        // 设置节点坐标：X从24开始，每次+2；Y固定为5
        pcur->x = POS_X + i * 2;
        pcur->y = POS_Y;
        pcur->next = NULL;
        // 头插法：新节点作为蛇头
        if (ps->_pSnake == NULL) {
            ps->_pSnake = pcur;  // 第一个节点，头指针指向它
        } 
        else 
        {
            pcur->next = ps->_pSnake;  // 新节点指向原蛇头
            ps->_pSnake = pcur;        // 头指针指向新节点（新蛇头）
        }
    }
```
其中很重要的一点都要对每次申请内存是否成功进行判断，若申请失败会返回空指针，C语言里严禁对空指针解引用的
在创建好蛇后需要打印，并且设置好蛇的其他信息：

```c
while (pcur)
{
	SetPos(pcur->x, pcur->y);
	wprintf(L"%c", BODY);
	pcur = pcur->next;
}
ps->_Dir = RIGHT;
ps->_foodWeight = 10;
ps->_SleepTime = 200;
ps->_Socre = 0;
ps->_Status = NORMAL;
```
### 创建食物（CreateFood）
用随机数生成随机的坐标，由于是宽字符打印，所以必须检测 X 坐标是否为偶数，非偶数则重新生成，且生成的坐标不可以与蛇重合，如果重合，则使用goto语句回去

```c
void CreateFood(pSnake ps)
{
	int x = 0;
	int y = 0;
	again:
	do
	{
		x = rand() % 52 + 2;
		y = rand() % 25 + 1;
	} while (x % 2 != 0);
	pSnakeNode pcur = ps->_pSnake;
	while (pcur)
	{
		if (pcur->x == x && pcur->y == y)
		{
			goto again;
		}
		pcur = pcur->next;
	}
	pSnakeNode pfood = (pSnakeNode)malloc(sizeof(SnakeNode));
	if (pfood == NULL)
	{
		perror("CreateFood malloc error");
		exit(1);
	}
	else 
	{
		pfood->x = x;
		pfood->y = y;
	}
	SetPos(pfood->x, pfood->y);
	wprintf(L"%c", FOOD);
	pfood->next = pfood;
	ps->_pFood = pfood;
}
```
### 游戏运行（GameRun）
游戏的主循环模块，实现以下核心逻辑
1. 打印右侧游戏帮助信息
2. 持续检测按键状态，控制蛇的方向、加速、减速、暂停、退出
3. 根据休眠时间控制蛇的移动速度
4. 调用SnakeMove函数实现蛇身移动
5. 根据游戏状态_Status判断是否继续循环，状态非NORMAL则退出循环，进入游戏结束模块

函数包含打印帮助信息（PrintHelp）、暂停游戏（StopGame）、移动一步（SnakeMove）
要注意判断蛇的方向还要判断没有按下反方向的键，因为游戏里蛇不会原地掉头，例如判断按“上”的时候要判断此时方向不能在“下”
其中减速和加速不能无限制的加减速，要不然程序休眠时间有可能会变成负数
```c
void GameRun(pSnake ps)
{
	PrintHelp();
	do
	{
		SetPos(64, 10);
		printf("当前分数：%2d", ps->_Socre);
		SetPos(64, 11);
		printf("当前食物的权重：%2d", ps->_foodWeight);
		if (KEY_PRESS(VK_UP) && ps->_Dir != DOWN)
		{
			ps->_Dir = UP;
		}
		else if (KEY_PRESS(VK_DOWN) && ps->_Dir != UP)
		{
			ps->_Dir = DOWN;
		}
		else if (KEY_PRESS(VK_LEFT) && ps->_Dir != RIGHT)
		{
			ps->_Dir = LEFT;
		}
		else if (KEY_PRESS(VK_RIGHT) && ps->_Dir != LEFT)
		{
			ps->_Dir = RIGHT;
		}
		else if (KEY_PRESS(VK_ESCAPE))
		{
			ps->_Status = ENDING;
		}
		else if (KEY_PRESS(VK_LCONTROL))
		{
			//减速
			if (ps->_foodWeight > 2)
			{
				ps->_SleepTime += 30;
				ps->_foodWeight -= 2;
			}
		}
		else if (KEY_PRESS(0x5A))
		{
			//加速
			if (ps->_SleepTime > 80)
			{
				ps->_SleepTime -= 30;
				ps->_foodWeight += 2;
			}
		}
		else if (KEY_PRESS(VK_SPACE))
		{
			StopGame();
		}

		SnakeMove(ps);
		Sleep(ps->_SleepTime);
	} while (ps->_Status == NORMAL);
}
```
### 帮助信息（PrintHelp）
帮助信息没什么好说，就是打印信息让用户知道按键信息：

```c
void PrintHelp()
{
	SetPos(64, 14);
	wprintf(L"%ls", L"不能穿墙，不能咬到自己");
	SetPos(64, 15);
	wprintf(L"%ls", L"用 ↑. ↓ . ← . → 来控制蛇的移动");
	SetPos(64, 16);
	wprintf(L"%ls", L"按Z加速，左边ctrl减速");
	SetPos(64, 17);
	wprintf(L"%ls", L"按ESC退出游戏，按空格暂停游戏");
}
```
### 蛇身移动（SnakeMove）
是本文示例最最重要的函数，也是逻辑最复杂的函数
1. 根据蛇的当前方向，计算蛇头下一个节点的坐标
2. 判断下一个节点是否是食物（NextIsFood）
   - 是食物：调用EatFood函数，蛇身增长，加分，重新创建食物
   - 非食物：调用NotEatFood函数，蛇身正常移动，删除蛇尾节点
3. 移动后检测是否撞墙（KillByWall）或撞自身（KillBySelf），修改游戏状态。

蛇身移动是蛇头向前走一步，蛇尾跟进（非食物）或蛇头向前走一步，蛇尾不跟进（吃食物），下一个蛇头节点的坐标计算：
- 向上（UP）：X 不变，Y-1
- 向下（DOWN）：X 不变，Y+1
- 向左（LEFT）：X-2，Y 不变
- 向右（RIGHT）：X+2，Y 不变
 
 其中向左向右+2是因为宽字符，只能偶数移动，如果是奇数移动打印身体会只打印一半
 

```c
void SnakeMove(pSnake ps)
{
	pSnakeNode pnext = (pSnakeNode)malloc(sizeof(SnakeNode));
	switch (ps->_Dir)
	{
	case UP:
		pnext->x = ps->_pSnake->x;
		pnext->y = ps->_pSnake->y - 1;
		break;
	case DOWN:
		pnext->x = ps->_pSnake->x;
		pnext->y = ps->_pSnake->y + 1;
		break;
	case RIGHT:
		pnext->x = ps->_pSnake->x + 2;
		pnext->y = ps->_pSnake->y;
		break;
	case LEFT:
		pnext->x = ps->_pSnake->x - 2;
		pnext->y = ps->_pSnake->y;
		break;
	}
	if (NextIsFood(pnext, ps))
		EatFood(pnext, ps);
	else
		NotEatFood(pnext, ps);
	KillByWall(ps);
	KillBySelf(ps);
}
```
### 判断是否是食物（NextIsFood）
此函数比较简单，直接比较蛇头的坐标与食物的坐标是否相等即可

```c
int NextIsFood(pSnakeNode pnext, pSnake ps)
{
	return (pnext->x == ps->_pFood->x && pnext->y == ps->_pFood->y);
}
```
### 吃食物（EatFood）
- 采用**头插法**将下一个节点加入蛇身链表，成为新的蛇头
- 重新绘制整条蛇身（蛇身增长一节）
- 累加分数（当前食物分数_foodWeight）
- 释放原食物节点的内存，调用CreateFood重新创建食物

```c
void EatFood(pSnakeNode pnext, pSnake ps)
{
	ps->_pFood->next = ps->_pSnake;
	ps->_pSnake = ps->_pFood;
	free(pnext);
	pnext = NULL;
	pSnakeNode pcur = ps->_pSnake;
	while (pcur)
	{
		SetPos(pcur->x, pcur->y);
		wprintf(L"%c", BODY);
		pcur = pcur->next;
	}
	CreateFood(ps);
}
```
### 不是食物（NotEatFood）
- 头插法将下一个节点加入蛇身链表，成为新蛇头
- 绘制蛇身（遍历到倒数第二个节点，不绘制蛇尾）
- 将原蛇尾节点的位置打印为空格（清除蛇尾）
- 释放原蛇尾节点的内存，将倒数第二个节点的next置为NULL（避免野指针）

注意在删除最后一个节点需要将前一个节点的指向下一个节点的指针置空，所以while循环的结束条件是`cur->next != NULL && cur->next->next != NULL`，丙炔要注意释放内存之前要先打印空格在最后一个蛇的节点，不然最后节点不会被覆盖，蛇会被一直“拖长”

```c
void NotEatFood(pSnakeNode pn, pSnake ps)
{
	pn->next = ps->_pSnake;
	ps->_pSnake = pn;

	pSnakeNode cur = ps->_pSnake;
	while (cur->next != NULL && cur->next->next != NULL)
	{
		SetPos(cur->x, cur->y);
		wprintf(L"%lc", BODY);
		cur = cur->next;
	}

	//把最后一个结点打印成空格
	SetPos(cur->next->x, cur->next->y);
	printf("  ");

	//释放最后一个结点
	free(cur->next);

	//把倒数第二个节点的地址置为NULL
	cur->next = NULL;
}
```
### 撞墙检测（KillByWall）与撞自己检测（KillByWall）
这个两个函数比较简单，撞墙就是检查蛇头的坐标是否与墙的坐标重合了，撞自己就是蛇头坐标分别和遍历的蛇身重合

```c
void KillByWall(pSnake ps)
{
	if (ps->_pSnake->x == 0 || ps->_pSnake->x == 56 ||
		ps->_pSnake->y == 0 || ps->_pSnake->y == 26)
		ps->_Status = KILL_WALL;
}

void KillBySelf(pSnake ps)
{
	pSnakeNode pcur = ps->_pSnake->next;
	while (pcur)
	{
		if (ps->_pSnake->x == pcur->x && ps->_pSnake->y == pcur->y)
		{
			ps->_Status = KILL_SELF;
			break;
		}
		pcur = pcur->next;
	}
}
```
### 游戏结束（GameEnd）
- 根据游戏状态_Status，在控制台定点打印游戏结束的原因
- 遍历蛇身链表，释放所有蛇身节点的动态内存，避免内存泄漏

注意释放蛇的节点的时候需要临时节点保存下一个节点，这样节点才不会丢失，否则直接释放节点下一个节点就找不到了

```c
void GameOver(pSnake ps)
{
	SetPos(25, 9);
	if (ps->_Status == KILL_WALL)
		wprintf(L"撞墙了");
	else if (ps->_Status == KILL_SELF)
		wprintf(L"吃到自己了");
	else if (ps->_Status == ENDING)
		wprintf(L"退出了");
	pSnakeNode pcur = ps->_pSnake;
	while (pcur)
	{
		pSnakeNode del = pcur;
		pcur = pcur->next;
		free(del);
	}
	ps->_pSnake = NULL;
}
```
### 逻辑串起来

```c
void Game()
{
	char ch = 0;
	do
	{
		Snake snake = { 0 };
		GameStart(&snake);
		GameRun(&snake);
		GameOver(&snake);
		SetPos(26, 10);
		printf("再来一局吗?(Y/N):");
		ch = getchar();
		while (getchar() != '\n');
	} while (ch == 'Y' || ch == 'y');
}

int main()
{
	srand((unsigned int)time(NULL));
	setlocale(LC_ALL, "");
	Game();
	return 0;
}
```
注意getchar()清除换行符：玩家输入 Y/N 后按回车，输入缓冲区会保留\n，第二次getchar()会直接读取\n，因此需要循环调用getchar()清除
且使用随机数之前要初始化总之，使用srand语句传入时间用于初始化总之，使其不再是假随机数
且本地化要在mian函数开头
# 源码

```c
#define _CRT_SECURE_NO_WARNINGS 1
#include "Snake.h"

void Game()
{
	char ch = 0;
	do
	{
		Snake snake = { 0 };
		GameStart(&snake);
		GameRun(&snake);
		GameOver(&snake);
		SetPos(26, 10);
		printf("再来一局吗?(Y/N):");
		ch = getchar();
		while (getchar() != '\n');
	} while (ch == 'Y' || ch == 'y');
}

int main()
{
	srand((unsigned int)time(NULL));
	setlocale(LC_ALL, "");
	Game();
	return 0;
}
```

```c
#pragma once
#include <stdio.h>
#include <locale.h>
#include <Windows.h>
#include <stdlib.h>
#include <stdbool.h>
#include <time.h>

#define KEY_PRESS(vk)  ((GetAsyncKeyState(vk)&1)?1:0)

#define WALL L'■'
#define BODY L'●'
#define FOOD L'★'
#define POS_X 6
#define POS_Y 20
enum DIRECTION
{
	UP=1,
	DOWN,
	RIGHT,
	LEFT
};
enum GAME_STATUS
{
	NORMAL,
	KILL_WALL,
	KILL_SELF,
	ENDING
};
typedef struct SnakeNode
{
	int x;
	int y;
	struct SnakeNode* next;
}SnakeNode, * pSnakeNode;
typedef struct Snake
{
	pSnakeNode _pSnake;				//蛇头
	pSnakeNode _pFood;				//食物
	enum DIRECTION _Dir;			//方向
	enum GAME_STATUS _Status;		//游戏状态
	int _Socre;						//分数
	int _foodWeight;				//食物权重
	int _SleepTime;					//速度
}Snake, *pSnake;

//设置光标位置
void SetPos(short x, short y);

//游戏初始化
void GameStart(pSnake ps);

//欢迎信息
void WelcomeToGame();

//初始化地图
void CreateMap();

//初始化蛇
void InitSnake(pSnake ps);

//创造食物
void CreateFood(pSnake ps);

//运行游戏
void GameRun(pSnake ps);

//蛇走一步
void SnakeMove(pSnake ps);

//下一步是不是食物
int NextIsFood(pSnakeNode pnext, pSnake ps);

//吃掉食物
void EatFood(pSnakeNode pnext, pSnake ps);

//下一步不是食物
void NotEatFood(pSnakeNode pnext, pSnake ps);

//撞墙了
void KillByWall(pSnake ps);

//撞到自己了
void KillBySelf(pSnake ps);

//游戏结束
void GameOver(pSnake ps);
```

```c
#define _CRT_SECURE_NO_WARNINGS 1
#include "Snake.h"
void SetPos(short x, short y)
{
	COORD pos = { x, y };
	HANDLE hOutput = NULL;
	hOutput = GetStdHandle(STD_OUTPUT_HANDLE);
	SetConsoleCursorPosition(hOutput, pos);
}

void WelcomeToGame()
{
	SetPos(40, 15);
	printf("欢迎来到贪吃蛇游戏");
	SetPos(40, 25);
	system("pause");
	system("cls");
	SetPos(25, 12);
	SetPos(25, 13);
	printf("用上，下，左，右分别控制蛇的移动，Z为加速，左边ctrl为减速\n");
	printf("加速将能得到更高的分数。\n");
	SetPos(40, 25);
	system("pause");
	system("cls");
	SetPos(60, 15);
}

void CreateMap()
{
	int i = 0;
	SetPos(0, 0);
	for (i = 0; i < 58; i += 2)
	{
		wprintf(L"%c", WALL);
	}
	SetPos(0, 26);
	for (i = 0; i < 58; i += 2)
	{
		wprintf(L"%c", WALL);
	}
	for (i = 1; i < 26; i++)
	{
		SetPos(0, i);
		wprintf(L"%c", WALL);
	}
	for (i = 1; i < 26; i++)
	{
		SetPos(56, i);
		wprintf(L"%c", WALL);
	}
}

void InitSnake(pSnake ps)
{
	pSnakeNode pcur = NULL;
	//头插法
	for (int i = 0; i < 5; i++)
	{
		pcur = (pSnakeNode)malloc(sizeof(SnakeNode));
		if (pcur == NULL)
		{
			perror("InitSnake malloc error");
			exit(1);
		}
		pcur->next = NULL;
		pcur->x = POS_X + i * 2;
		pcur->y = POS_Y;
		if (ps->_pSnake == NULL)
		{
			ps->_pSnake = pcur;
		}
		else
		{
			pcur->next = ps->_pSnake;
			ps->_pSnake = pcur;
		}
	}
	pcur = ps->_pSnake;
	while (pcur)
	{
		SetPos(pcur->x, pcur->y);
		wprintf(L"%c", BODY);
		pcur = pcur->next;
	}
	ps->_Dir = RIGHT;
	ps->_foodWeight = 10;
	ps->_SleepTime = 200;
	ps->_Socre = 0;
	ps->_Status = NORMAL;
}

void CreateFood(pSnake ps)
{
	int x = 0;
	int y = 0;
	again:
	do
	{
		x = rand() % 52 + 2;
		y = rand() % 25 + 1;
	} while (x % 2 != 0);
	pSnakeNode pcur = ps->_pSnake;
	while (pcur)
	{
		if (pcur->x == x && pcur->y == y)
		{
			goto again;
		}
		pcur = pcur->next;
	}
	pSnakeNode pfood = (pSnakeNode)malloc(sizeof(SnakeNode));
	if (pfood == NULL)
	{
		perror("CreateFood malloc error");
		exit(1);
	}
	else 
	{
		pfood->x = x;
		pfood->y = y;
	}
	SetPos(pfood->x, pfood->y);
	wprintf(L"%c", FOOD);
	pfood->next = pfood;
	ps->_pFood = pfood;
}

void GameStart(pSnake ps)
{
	system("mode con cols=100 lines=30");
	system("title 贪吃蛇");
	HANDLE hOutput = GetStdHandle(STD_OUTPUT_HANDLE);
	CONSOLE_CURSOR_INFO CursorInfo;
	GetConsoleCursorInfo(hOutput, &CursorInfo);
	CursorInfo.bVisible = false; 
	SetConsoleCursorInfo(hOutput, &CursorInfo);
	WelcomeToGame();
	CreateMap();

	InitSnake(ps);

	CreateFood(ps);

}

void PrintHelp()
{
	SetPos(64, 14);
	wprintf(L"%ls", L"不能穿墙，不能咬到自己");
	SetPos(64, 15);
	wprintf(L"%ls", L"用 ↑. ↓ . ← . → 来控制蛇的移动");
	SetPos(64, 16);
	wprintf(L"%ls", L"按Z加速，左边ctrl减速");
	SetPos(64, 17);
	wprintf(L"%ls", L"按ESC退出游戏，按空格暂停游戏");
}

void StopGame()
{
	while (1)
	{
		Sleep(200);
		if (KEY_PRESS(VK_SPACE))
			break;
	}
}

int NextIsFood(pSnakeNode pnext, pSnake ps)
{
	return (pnext->x == ps->_pFood->x && pnext->y == ps->_pFood->y);
}

void EatFood(pSnakeNode pnext, pSnake ps)
{
	ps->_pFood->next = ps->_pSnake;
	ps->_pSnake = ps->_pFood;
	free(pnext);
	pnext = NULL;
	pSnakeNode pcur = ps->_pSnake;
	while (pcur)
	{
		SetPos(pcur->x, pcur->y);
		wprintf(L"%c", BODY);
		pcur = pcur->next;
	}
	CreateFood(ps);
}

void NotEatFood(pSnakeNode pn, pSnake ps)
{
	pn->next = ps->_pSnake;
	ps->_pSnake = pn;

	pSnakeNode cur = ps->_pSnake;
	while (cur->next != NULL && cur->next->next != NULL)
	{
		SetPos(cur->x, cur->y);
		wprintf(L"%lc", BODY);
		cur = cur->next;
	}

	//把最后一个结点打印成空格
	SetPos(cur->next->x, cur->next->y);
	printf("  ");

	//释放最后一个结点
	free(cur->next);

	//把倒数第二个节点的地址置为NULL
	cur->next = NULL;
}

void KillByWall(pSnake ps)
{
	if (ps->_pSnake->x == 0 || ps->_pSnake->x == 56 ||
		ps->_pSnake->y == 0 || ps->_pSnake->y == 26)
		ps->_Status = KILL_WALL;
}

void KillBySelf(pSnake ps)
{
	pSnakeNode pcur = ps->_pSnake->next;
	while (pcur)
	{
		if (ps->_pSnake->x == pcur->x && ps->_pSnake->y == pcur->y)
		{
			ps->_Status = KILL_SELF;
			break;
		}
		pcur = pcur->next;
	}
}

void SnakeMove(pSnake ps)
{
	pSnakeNode pnext = (pSnakeNode)malloc(sizeof(SnakeNode));
	switch (ps->_Dir)
	{
	case UP:
		pnext->x = ps->_pSnake->x;
		pnext->y = ps->_pSnake->y - 1;
		break;
	case DOWN:
		pnext->x = ps->_pSnake->x;
		pnext->y = ps->_pSnake->y + 1;
		break;
	case RIGHT:
		pnext->x = ps->_pSnake->x + 2;
		pnext->y = ps->_pSnake->y;
		break;
	case LEFT:
		pnext->x = ps->_pSnake->x - 2;
		pnext->y = ps->_pSnake->y;
		break;
	}
	if (NextIsFood(pnext, ps))
		EatFood(pnext, ps);
	else
		NotEatFood(pnext, ps);
	KillByWall(ps);
	KillBySelf(ps);
}

void GameRun(pSnake ps)
{
	PrintHelp();
	do
	{
		SetPos(64, 10);
		printf("当前分数：%2d", ps->_Socre);
		SetPos(64, 11);
		printf("当前食物的权重：%2d", ps->_foodWeight);
		if (KEY_PRESS(VK_UP) && ps->_Dir != DOWN)
		{
			ps->_Dir = UP;
		}
		else if (KEY_PRESS(VK_DOWN) && ps->_Dir != UP)
		{
			ps->_Dir = DOWN;
		}
		else if (KEY_PRESS(VK_LEFT) && ps->_Dir != RIGHT)
		{
			ps->_Dir = LEFT;
		}
		else if (KEY_PRESS(VK_RIGHT) && ps->_Dir != LEFT)
		{
			ps->_Dir = RIGHT;
		}
		else if (KEY_PRESS(VK_ESCAPE))
		{
			ps->_Status = ENDING;
		}
		else if (KEY_PRESS(VK_LCONTROL))
		{
			//减速
			if (ps->_foodWeight > 2)
			{
				ps->_SleepTime += 30;
				ps->_foodWeight -= 2;
			}
		}
		else if (KEY_PRESS(0x5A))
		{
			//加速
			if (ps->_SleepTime > 80)
			{
				ps->_SleepTime -= 30;
				ps->_foodWeight += 2;
			}
		}
		else if (KEY_PRESS(VK_SPACE))
		{
			StopGame();
		}

		SnakeMove(ps);
		Sleep(ps->_SleepTime);
	} while (ps->_Status == NORMAL);
}

void GameOver(pSnake ps)
{
	SetPos(25, 9);
	if (ps->_Status == KILL_WALL)
		wprintf(L"撞墙了");
	else if (ps->_Status == KILL_SELF)
		wprintf(L"吃到自己了");
	else if (ps->_Status == ENDING)
		wprintf(L"退出了");
	pSnakeNode pcur = ps->_pSnake;
	while (pcur)
	{
		pSnakeNode del = pcur;
		pcur = pcur->next;
		free(del);
	}
	ps->_pSnake = NULL;
}
```

# 结语
作者第一次做游戏，还有一些尚未完善的地方，其win32API可能存在一些描述不准确的现象，恳请读者见谅

---

## 相关笔记 / 相关链接

- 相关目录：[[C语言目录]]
