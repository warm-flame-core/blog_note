---
tags:
  - 嵌入式
  - STM32
aliases:
  - GPIO输出与HAL源码解析
日期: 2026年9月19日
---


![博客横幅图](https://i-blog.csdnimg.cn/direct/098d6ab5280c4b48b7b0a324f8dc3ac5.jpeg#pic_center)

<p align="center">
  <font color="#FF69B4"><b>Ciallo(∠・ω< )⌒☆</b></font>
</p>

<br>

<table align="center" width="100%">
  <tr>
    <th align="center">系列专栏</th>
    <th align="center">直达链接</th>
  </tr>
  <tr>
    <td align="center">🔤 C 语言</td>
    <td align="center"><a href="https://blog.csdn.net/hahaha6565/category_13108044.html">进入专栏 →</a></td>
  </tr>
  <tr>
    <td align="center">🧱 数据结构</td>
    <td align="center"><a href="https://blog.csdn.net/hahaha6565/category_13156520.html">进入专栏 →</a></td>
  </tr>
  <tr>
    <td align="center">🔧 C++</td>
    <td align="center"><a href="https://blog.csdn.net/hahaha6565/category_13163238.html">进入专栏 →</a></td>
  </tr>
  <tr>
    <td align="center">🐍 Python</td>
    <td align="center"><a href="https://blog.csdn.net/hahaha6565/category_13194466.html">进入专栏 →</a></td>
  </tr>
  <tr>
    <td align="center">🔌 STM32</td>
    <td align="center"><a href="https://blog.csdn.net/hahaha6565/category_13209892.html">进入专栏 →</a></td>
  </tr>
</table>

<br>

<p align="center">
  🏠 博客主页：<a href="https://blog.csdn.net/hahaha6565?type=blog">暖焰核心</a><br>
  📦 作者仓库：<a href="https://github.com/warm-flame-core">github</a> | <a href="https://gitee.com/warm-flame-core">gitee</a>
</p>


# 引言

`GPIO` 的八种模式：输入四种、输出四种，推挽和开漏差在哪，上下拉什么时候该用。但这些都还停留在纸面上，一个引脚我都没动过。

更让我心虚的是另一件事。`HAL_GPIO_WritePin(GPIOF, GPIO_PIN_8, 0)` 这行代码我敲得出来，灯也确实亮了，可要是有人问我那个 `0` 是什么意思、`GPIO_PIN_8` 为什么长这个样、灯凭什么就亮，我答不上来。代码能跑和知道它为什么能跑，是两件事。

那就从这行代码往回追，看能追到多远。

---

## 一、 板子和它的四个灯

### 1.1 原理图上的接线

写代码之前得先知道自己在控制什么。板子上有四颗贴片 `LED`（`Light Emitting Diode`，发光二极管），它们的接法能从原理图里看明白：

```text
VDD_3.3V --+-- 680R -- LED -- SYS_LED1 -- PF8    20 脚
           |
           +-- 680R -- LED -- SYS_LED2 -- PF9    21 脚
           |
           +-- 680R -- LED -- SYS_LED3 -- PF10   22 脚
           |
           +-- 680R -- LED -- SYS_LED4 -- PF11   49 脚
```

四颗 `LED` 各配一个 680 欧的限流电阻，一端全都接到 `VDD_3.3V`，另一端经网络标签 `SYS_LED1` 到 `SYS_LED4` 回到芯片的 `PF8`、`PF9`、`PF10`、`PF11` 四个引脚上。

![[STM32/attachments/STM32灵犀板LED接线.png|原理图里 LED 那一栏，四颗 LED 各串一个 680 欧电阻接到 3.3V]]

![[STM32/attachments/STM32灵犀板LED引脚对应.png|原理图引脚侧，SYS_LED1 到 SYS_LED4 分别接到 PF8 到 PF11]]

网络标签是个值得单独说一句的东西。原理图内容多，一张 A4 画不下，通常拆成好几页来画：电源一页、`MCU` 一页、外设一页。这些页之间在物理上是连着的，但纸上没法真的画一根线穿过去，于是约定：**两张图上只要出现同名标签，就当它是同一根导线**。

这里 `SYS_LED1` 就出现了两次。一次在 `LED` 那一页，一次在芯片引脚那一页。所以那根线其实是从 `LED` 的阴极一路通到了 `PF8`。

### 1.2 灯为什么会亮

顺着这张图往下想一步。电流要从 `3.3V` 出发，经过 680 欧电阻和 `LED`，最后**流进芯片的引脚**才算形成回路。这就要求 `PF8` 的电平比 `3.3V` 低。

所以：

- `PF8` 输出**低电平**（0 V）时，两端有压差，回路通，灯亮；
- `PF8` 输出**高电平**（3.3 V）时，两端都是 3.3 V，没有压差，灯灭。

这就是接法的脾气：`LED` 的阳极朝电源，阴极朝芯片，芯片负责把它拉低。在这块板子上，看到 `GPIO_PIN_RESET` 就可以把"点亮"接上去。

顺带把上一篇留下的一条线接上。0 V 是被芯片主动拉低的，拉低这件事靠输出级的 `N-MOS` 导通来完成；而 3.3 V 那一端不是芯片给的，是板子上现成的电源经电阻和 `LED` 过来的。这一点在选输出模式的时候会再遇到。

### 1.3 配置里跟灯有关的三项

配置工具里生成引脚配置的时候，跟这四个灯有关的选择就三项。

**输出模式选了开漏。** 平心而论，这块板子上推挽和开漏都能把灯点亮：

| 模式 | 输出低电平 | 输出高电平 |
| --- | --- | --- |
| 推挽 | `N-MOS` 导通，主动拉到 0 V | `P-MOS` 导通，主动输出 3.3 V |
| 开漏 | `N-MOS` 导通，主动拉到 0 V | `N-MOS` 关断，引脚悬空，电平由外部决定 |

点灯只需要"低电平"这一个动作，两种模式都能做到。差别在输出高电平那半边：推挽是芯片自己把电平推到 3.3 V，开漏是撒手不管。

开漏的用处在负载电压和芯片电压不一致的时候。芯片引脚最多只能输出 3.3 V，要是负载挂在 5 V 上，推挽输出的高电平根本盖不住对方；开漏就不一样，它只负责拉低、不负责拉高，高电平交给外部那条通路去定。这个习惯跟手头的板子无关，是通用做法。

顺带澄清一个容易想歪的地方：开漏输出高电平的时候引脚是**高阻态**，既不是 0 V 也不是 3.3 V，是说"我不参与"。这时候引脚上实际是什么电平，完全看外部接了什么东西。这块板子上外部就是那条 `3.3V → 电阻 → LED` 的通路，所以它自然被拉到 3.3 V，灯就灭了。

**初始电平设成了高。** 这个选择的意思是上电那一刻先让四个灯全灭，别一通电就亮着。后面写代码的时候还会有一道保险，两处对上。

**输出速度选了最低档。** 手册里给的是 2 MHz、10 MHz、50 MHz 三档，点个灯用最低的足够。速度这一项控制的是引脚电平能多快地跟着变，手册里按最大频率分档，跟灯闪的快慢是两回事。

---

## 二、 先点亮一个

### 2.1 第一行代码

灯亮了。

```c
HAL_GPIO_WritePin(GPIOF, GPIO_PIN_8, 0);
```

写完这行，四个灯里有一个亮了，任务算是完成了。但盯着它看，三个参数我一个都说不清楚：`GPIOF` 是什么东西，`GPIO_PIN_8` 为什么不是简单的一个 `8`，那个 `0` 又是什么意思。

先把最简单的那个解决掉。翻了头文件，`PinState` 的类型是这么定义的：

```c
typedef enum
{
  GPIO_PIN_RESET = 0u,
  GPIO_PIN_SET
} GPIO_PinState;

```

`GPIO_PIN_RESET` 就是 `0`，`GPIO_PIN_SET` 是它后面的 `1`。所以传 `0` 和传 `GPIO_PIN_RESET` 是一回事，编译能过、灯也亮，只是读代码的人得自己拐个弯才知道那个 `0` 指的是"复位"。后来我把这几处都换成了枚举名。

`GPIOF` 看着简单，代表 `F` 组那十六个引脚，但它背后还压着一串地址，得单开一节来说。眼下最想弄明白的是第二个参数。

### 2.2 引脚是个位

按我原来的想法，`GPIO_PIN_8` 应该就是 `8`。打开定义一看，完全不是这么回事。

先别管库是怎么写的，自己设想一下这套规则该怎么定。

最省事的办法是直接传编号：要动 8 号引脚就传 `8`，要动 9 号就传 `9`。一个编号对一个引脚，很清楚。但只要想同时处理几个引脚，这个办法立刻卡住：一个数只能指一个引脚，想点亮四个灯就得调四次，而且没有任何办法用"一个参数"表达"第 8、9、10、11 号这四个"。

换个思路。**别传编号了，改成传一排开关的状态。**

想象一排 16 个开关，一个开关管一个引脚，1 是合上、0 是断开：

```text
  15 14 13 12 11 10  9  8  7  6  5  4  3  2  1  0     <- 引脚号
   0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0     <- 每一位的值
                        ^
                    第 8 号引脚归这一格管
```

要给 8 号引脚通电，就把第 8 格拨成 1：

```text
  15 14 13 12 11 10  9  8  7  6  5  4  3  2  1  0
   0  0  0  0  0  0  0  1  0  0  0  0  0  0  0  0
```

这串二进制写成十六进制是 `0x0100`。换个角度看，它也就是"把 1 往左移 8 位"：

```text
1 << 0     0000 0000 0000 0001     第 0 号引脚
1 << 1     0000 0000 0000 0010     第 1 号引脚
1 << 2     0000 0000 0000 0100     第 2 号引脚
   ...
1 << 8     0000 0001 0000 0000     第 8 号引脚
   ...
1 << 15    1000 0000 0000 0000     第 15 号引脚
```

库里的宏，就是把这些算好的结果列成了一张表：

```c
#define GPIO_PIN_0                 ((uint16_t)0x0001)  /* Pin 0 selected    */
#define GPIO_PIN_1                 ((uint16_t)0x0002)  /* Pin 1 selected    */
#define GPIO_PIN_2                 ((uint16_t)0x0004)  /* Pin 2 selected    */
/* 中间省略 */
#define GPIO_PIN_8                 ((uint16_t)0x0100)  /* Pin 8 selected    */
#define GPIO_PIN_9                 ((uint16_t)0x0200)  /* Pin 9 selected    */
#define GPIO_PIN_10                ((uint16_t)0x0400)  /* Pin 10 selected   */
#define GPIO_PIN_11                ((uint16_t)0x0800)  /* Pin 11 selected   */
#define GPIO_PIN_All               ((uint16_t)0xFFFF)  /* All pins selected */
```

`GPIO_PIN_8` 是 `0x0100`，`GPIO_PIN_9` 是 `0x0200`，一路到 `GPIO_PIN_15` 是 `0x8000`。**一个引脚占一个比特位，第几位就是几号引脚。** 引脚号不是"值"，是"位置"。

写成算好的 `0x0100` 而不是 `1 << 8`，是为了看的人不用在脑子里做移位。外面那层 `(uint16_t)` 是要个明确宽度：一组端口正好 16 个引脚，16 位刚好装得下，全选就是 `0xFFFF`。

**这么一比，好处就出来了。** 传编号只能指一个引脚；换成开关排，一个数就装下了整个集合。而且集合运算和位运算能直接对上：

- 两组引脚合起来，用按位或；
- 两组引脚里共有的，用按位与；
- 想去掉哪几个，用按位取反再与一下。

四个灯要的就是后四个开关：

```text
GPIO_PIN_8                0000 0001 0000 0000
GPIO_PIN_9                0000 0010 0000 0000
GPIO_PIN_10               0000 0100 0000 0000
GPIO_PIN_11               0000 1000 0000 0000
------------------------------------------------
四个按位或                0000 1111 0000 0000
```

初始化那行代码里就是这么用的：

```c
GPIO_InitStruct.Pin = GPIO_PIN_8 | GPIO_PIN_9 | GPIO_PIN_10 | GPIO_PIN_11;
```

一次配置四个引脚，靠的就是这一个数。要是 `GPIO_PIN_x` 真按 `8`、`9`、`10`、`11` 那样定义，这四个数既或不到一起，也没有办法再拆开。

还有一处能看出这套设计和硬件是打通的：`BSRR` 寄存器本身就是一位管一个引脚，位图里哪一位是 1，`BSRR` 里对应那一位就动作。软件里的"开关排"和寄存器里的"格子"一一对应，中间不需要任何换算。

### 2.3 GPIOF 与统一编址

第一个参数也不简单。`GPIOF` 展开之后是一句强制类型转换：

```c
#define GPIOF               ((GPIO_TypeDef *) GPIOF_BASE)
```

`GPIOF_BASE` 再展开，是一路加法：

```c
#define GPIOF_BASE          (APB2PERIPH_BASE + 0x00001C00UL)
#define APB2PERIPH_BASE     (PERIPH_BASE + 0x00010000UL)
#define PERIPH_BASE         0x40000000UL
```

顺着算下来：

```text
PERIPH_BASE       0x40000000
+ 0x00010000      0x40010000      APB2PERIPH_BASE
+ 0x00001C00      0x40011C00      GPIOF_BASE
```

所以 `GPIOF` 的本质就是"地址 `0x40011C00` 上的一个 `GPIO_TypeDef` 结构体"，一个十六进制的地址。

**寄存器为什么能当内存访问**

`0x40000000` 这个数不是随手挑的。芯片的 32 位地址空间一共 4GB，手册把它切成 8 块，每块 512MB：

```text
0x00000000 - 0x1FFFFFFF   block 0   程序存储器（Flash / 系统存储器）
0x20000000 - 0x3FFFFFFF   block 1   SRAM
0x40000000 - 0x5FFFFFFF   block 2   片上外设          <- 寄存器都在这一段
0x60000000 - 0x7FFFFFFF   block 3   FSMC bank1 / bank2
0x80000000 - 0x9FFFFFFF   block 4   FSMC bank3 / bank4
0xA0000000 - 0xBFFFFFFF   block 5   FSMC 寄存器
0xC0000000 - 0xDFFFFFFF   block 6   未使用
0xE0000000 - 0xFFFFFFFF   block 7   Cortex-M3 内核外设（NVIC、SysTick 等）
```

**外设寄存器没有单独的地址空间，它和内存挤在同一个 4GB 里，用同一套读写指令访问。** 这叫统一编址。写一句 `GPIOF->BSRR = ...`，编译出来就是"往某个地址写 32 位数据"，跟往 `SRAM` 里的变量赋值没有区别，只是那个地址背后接的不是存储单元，而是外设的寄存器。

外设这一块 512MB 里还要再分：

```text
0x40000000 - 0x4000FFFF   APB1 外设（TIM2~7、USART2/3、I2C、SPI2、CAN、USB、PWR、BKP...）
0x40010000 - 0x4001FFFF   APB2 外设（GPIO、AFIO、EXTI、ADC、TIM1、SPI1、USART1...）
0x40020000 - 0x4002FFFF   AHB 外设（DMA、RCC、Flash 接口、CRC）
```

`GPIOF_BASE` 里那个 `0x00010000`，说的就是"从外设区往里走 `0x10000` 字节，进入 APB2 那一堆"；再加上 `0x1C00`，就是在 APB2 里找到 Port F。

`0x1C00` 这截也是有规矩的，几个端口挨着排，每个占 `0x400` 字节：

| 端口 | 起始地址 | 端口 | 起始地址 |
| --- | --- | --- | --- |
| Port A | `0x4001 0800` | Port E | `0x4001 1800` |
| Port B | `0x4001 0C00` | Port F | `0x4001 1C00` |
| Port C | `0x4001 1000` | Port G | `0x4001 2000` |
| Port D | `0x4001 1400` | | |

**结构体是怎么和寄存器对上的**

`GPIOF` 被转成 `GPIO_TypeDef *`。这个结构体的成员顺序，就是各个寄存器在端口里的排列顺序：

```c
typedef struct
{
  __IO uint32_t CRL;     /* 偏移 0x00 */
  __IO uint32_t CRH;     /* 偏移 0x04 */
  __IO uint32_t IDR;     /* 偏移 0x08 */
  __IO uint32_t ODR;     /* 偏移 0x0C */
  __IO uint32_t BSRR;    /* 偏移 0x10 */
  __IO uint32_t BRR;     /* 偏移 0x14 */
  __IO uint32_t LCKR;    /* 偏移 0x18 */
} GPIO_TypeDef;
```

成员全是 `uint32_t`，每个占 4 个字节，偏移量自然一个接一个排下去。所以 `GPIOF->BSRR` 这句，落到机器上就是：

```text
GPIOF_BASE          +   BSRR 的偏移
0x40011C00          +   0x10
= 0x40011C10
```

然后往这个地址写一个字。C 语言里那个箭头，翻译过来就是"基地址加偏移"。

成员前面的 `__IO` 是个宏，展开是 `volatile`。它的作用是告诉编译器：这个地址上的值随时会被硬件改动，别把它缓存进寄存器，也别把这个读写优化掉。

上面那八块的划分，在数据手册里就是一张图：

![[STM32/attachments/STM32存储器映像图.png|STM32F103xE 存储器映像图，4GB 地址空间分成八个 512MB 的块]]

左半边是把 4GB 横切成八块，右半边把 `0x4000 0000` 那一段展开，挨个列了外设的地址范围。Port A 到 Port G 就排在里面：Port A 从 `0x4001 0800` 起，每隔 `0x400` 字节换下一个，Port F 落在 `0x4001 1C00`，正是前面算出来的那个数。


### 2.4 写操作落到 BSRR

弄明白参数是什么意思，接着就想知道它到底干了什么。`HAL_GPIO_WritePin` 的源码只有几行：

```c
void HAL_GPIO_WritePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin, GPIO_PinState PinState)
{
  /* Check the parameters */
  assert_param(IS_GPIO_PIN(GPIO_Pin));
  assert_param(IS_GPIO_PIN_ACTION(PinState));

  if (PinState != GPIO_PIN_RESET)
  {
    GPIOx->BSRR = GPIO_Pin;
  }
  else
  {
    GPIOx->BSRR = (uint32_t)GPIO_Pin << 16u;
  }
}
```

三个参数各是什么：

```text
void HAL_GPIO_WritePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin, GPIO_PinState PinState);

参数说明：

- `GPIOx`：目标端口，取值 GPIOA 到 GPIOG
- `GPIO_Pin`：引脚位图，一个比特位对应一个引脚，多个引脚按位或后可以一次传入
- `PinState`：目标电平，GPIO_PIN_SET 置高、GPIO_PIN_RESET 置低

返回值：无
注意事项：端口时钟没使能的话，写进去不生效
```

两句 `assert_param` 是参数合法性检查，默认配置下不产生实际代码，先不看。真正起作用的只有那个 `if`-`else`，两条路都是在给同一个寄存器 `BSRR` 赋值。

`BSRR` 是 `Bit Set/Reset Register` 的缩写，位设置/清除寄存器，32 位宽，前后两半各管一件事：

| 区域 | 占哪几位 | 写 1 时 | 写 0 时 |
| --- | --- | --- | --- |
| `BRy` 复位区 | 31 ~ 16 | 对应引脚输出低电平 | 什么都不做 |
| `BSy` 置位区 | 15 ~ 0 | 对应引脚输出高电平 | 什么都不做 |

两个区各管一件事，而**写 0 的位一律不动作**。这一条是理解后面所有位运算的钥匙。

手册里对每一位的定义写得很直白：

| 位 | 名称 | 写 0 时 | 写 1 时 |
| --- | --- | --- | --- |
| 31 ~ 16 | `BRy`：复位 y 号引脚 | 对 `ODRx` 的对应位不执行任何操作 | 复位对应位 |
| 15 ~ 0 | `BSy`：置位 y 号引脚 | 对 `ODRx` 的对应位不执行任何操作 | 置位对应位 |

![[STM32/attachments/STM32_GPIOx_BSRR寄存器.png|参考手册里位设置/清除寄存器 GPIOx_BSRR 的位定义]]

名称里的 `y` 是引脚号。两个区编号的方式一模一样，都是"位序号减掉本区的起始位，得到引脚号"：

```text
复位区   bit31 到 bit16   ->   依次对应引脚 15 到引脚 0
置位区   bit15 到 bit0    ->   依次对应引脚 15 到引脚 0
```

同一个引脚在低 16 位和高 16 位里的相对位置完全相同，中间正好差 16 位。所以"把位图左移 16 位"就等于"从置位区搬到复位区"，引脚号一个字都不用改。

还有两个细节。手册特意注明这些位**只能以字（32 位）的形式写入**，不能按字节或者半字写。另外有个只做复位的老寄存器 `BRR`（`Bit Reset Register`），只有低 16 位有效，往里写 `GPIO_Pin` 跟往 `BSRR` 高 16 位写是一回事，功能上少了一半。上面那两个函数统一走 `BSRR`，一个寄存器管两个方向。

按照这个规则再看那两个分支就顺了。想输出高电平，把引脚位图原样写进低 16 位；想输出低电平，把同一个位图左移 16 位挪到高 16 位去：

```text
传 GPIO_PIN_8，走"置位"分支
  GPIOx->BSRR = GPIO_PIN_8

    bit31                           bit16 bit15                           bit0
    +-----------------------------------+-----------------------------------+
    |  0000 0000 0000 0000              |  0000 0001 0000 0000              |
    +-----------------------------------+-----------------------------------+
                                              ^
                                        低 16 位第 8 位
                                        -> PF8 输出高电平，灯灭

传 GPIO_PIN_8，走"复位"分支
  GPIOx->BSRR = (uint32_t)GPIO_PIN_8 << 16

    bit31                           bit16 bit15                           bit0
    +-----------------------------------+-----------------------------------+
    |  0000 0001 0000 0000              |  0000 0000 0000 0000              |
    +-----------------------------------+-----------------------------------+
        ^
    高 16 位第 8 位
    -> PF8 输出低电平，灯亮
```

左移 16 位做的事，就是把"第几号引脚为 1"从置位区搬到复位区，位置本身没变，还是第 8 号，只是换了个区管。

写 `(uint32_t)GPIO_Pin << 16u` 前面的类型转换，在 32 位的 `Cortex-M` 上其实不写也对：`uint16_t` 参与运算前会先做整型提升变成 `int`，而这里的 `int` 正好是 32 位，移位结果一样。写上它是把"这次移位要在 32 位宽度里做"这个意思摆明，不用去猜编译器怎么想。

既然两个分支最后都是写 `BSRR`，那为什么不直接改输出数据寄存器 `ODR`？毕竟要说"某个引脚输出高"，直觉上就该去写那个"输出数据"的寄存器。

原因在"改一位"这件事上。`ODR` 是一根根引脚并排躺着的 16 位，想让第 8 位变成 1，别的位不动，就得走三步：

```text
走 ODR：读 -> 改 -> 写

  1.  temp = GPIOF->ODR          0000 0000 0000 0000
  2.  temp = temp | GPIO_PIN_8   0000 0001 0000 0000
  3.  GPIOF->ODR = temp          0000 0001 0000 0000
```

看起来没问题，问题藏在第 1 步和第 3 步之间。万一这两步中间来了一次中断，中断里也改了同一个 `ODR`，那么第 3 步写回去的就是中断之前读到的旧值，中断那次改动会被悄悄抹掉。这种错很难查，因为它只在极窄的时间窗口里出现。

`BSRR` 没有这个窗口：

```text
走 BSRR：一次写，完事

  GPIOF->BSRR = GPIO_PIN_8

  写 0 的位不动作，所以不用先读回来把别的位拼上，
  一次写只影响写 1 的那几位，没有"读旧值"这一步，
  也就没有被中断插进来的机会。
```

硬件把"读—改—写"这三步压成了一次写操作，这就是 `BSRR` 存在的理由。手册里管这个叫原子操作。

---

## 三、 让它闪起来

### 3.1 四行写烦了

一个灯常亮着没什么意思，想让它闪。

按上一节学到的，自己拼一个闪烁出来并不难：点亮、等 200 毫秒、熄灭、再等 200 毫秒，整套塞进死循环里。写出来是四行，每多一个灯还要再翻一倍。

写之前先翻了翻库函数的清单，`HAL_GPIO_TogglePin` 就排在 `HAL_GPIO_WritePin` 下面，编号挨着。`Toggle` 是"翻转"的意思。

```c
while (1)
{
  HAL_GPIO_TogglePin(GPIOF, GPIO_PIN_8);
  HAL_Delay(200);
}
```

两行就够了，而且不用自己记"现在该亮还是该灭"。当前是高就拉低、当前是低就拉高，这件事交给它判断。

倒是想知道它是怎么判断的，于是又翻进了源码。

### 3.2 翻转藏在哪几刀里

```c
void HAL_GPIO_TogglePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin)
{
  uint32_t odr;

  /* Check the parameters */
  assert_param(IS_GPIO_PIN(GPIO_Pin));

  /* get current Output Data Register value */
  odr = GPIOx->ODR;

  /* Set selected pins that were at low level, and reset ones that were high */
  GPIOx->BSRR = ((odr & GPIO_Pin) << GPIO_NUMBER) | (~odr & GPIO_Pin);
}
```

```text
void HAL_GPIO_TogglePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin)
                                      |               |
                                      |               | GPIO_Pin：引脚位图
                                      |                 可以一次传多个引脚
                                      | GPIOx：目标端口
                                        取值 GPIOA 到 GPIOG
```

真正干活的只有最后一行。前半段把当前的电平状态读进 `odr`，`GPIO_NUMBER` 是库里的一个常量，值等于 16，跟上节课那行 `<< 16u` 是一回事。

这一行的想法是：**既然要把高的变低、低的变高，那就先把这两拨位分开，一拨送去复位区，一拨送去置位区，最后一个"或"合起来一次写进 `BSRR`。**

`odr` 里躺着 16 个引脚的当前电平，某位是 1 就表示那个引脚现在是高。拿它与引脚位图做两次与运算，就把目标引脚分成了两拨：

- `odr & GPIO_Pin`：目标引脚里**当前为高**的；
- `~odr & GPIO_Pin`：目标引脚里**当前为低**的。

第一拨要拉低，左移 16 位送到复位区；第二拨要拉高，原地不动，正好落在置位区。

拿 PF8 走一遍。假设它当前是低，也就是 `odr` 的第 8 位是 0：

```text
  odr                    0000 0000 0000 0000
  ~odr                   1111 1111 1111 1111

  odr & GPIO_Pin         0000 0000 0000 0000    <- 要高变低的：空
  ~odr & GPIO_Pin        0000 0001 0000 0000    <- 要低变高的：第 8 位

  合并后写进 BSRR

    (odr & Pin) << 16    0000 0000 0000 0000 | 0000 0000 0000 0000
    ~odr & Pin                                | 0000 0001 0000 0000
    ----------------------------------------------------------------
    BSRR 的值            0000 0000 0000 0000 | 0000 0001 0000 0000
                                                    ^ 置位区第 8 位
                                                    -> PF8 输出高电平
```

再假设它当前是高，`odr` 的第 8 位是 1：

```text
  odr                    0000 0001 0000 0000
  ~odr                   1111 1110 1111 1111

  odr & GPIO_Pin         0000 0001 0000 0000    <- 要高变低的：第 8 位
  ~odr & GPIO_Pin        0000 0000 0000 0000    <- 要低变高的：空

  合并后写进 BSRR

    (odr & Pin) << 16    0000 0001 0000 0000 | 0000 0000 0000 0000
    ~odr & Pin                                | 0000 0000 0000 0000
    ----------------------------------------------------------------
    BSRR 的值            0000 0001 0000 0000 | 0000 0000 0000 0000
                              ^ 复位区第 8 位
                              -> PF8 输出低电平
```

两次结果反了过来，而表达式本身一个字都没改。

这行代码里有个地方值得多看两眼：它**没有 `if`**。换成人类思路写，多半会是"如果当前是高就拉低，否则拉高"，两个分支。这里不用判断是因为那两刀与运算已经把情况分完了：

- 引脚当前是高，`~odr & GPIO_Pin` 那一侧自然算出 0，只有复位区有值；
- 引脚当前是低，`odr & GPIO_Pin` 那一侧自然算出 0，只有置位区有值。

两拨里总有一拨是空的，而 `BSRR` 里写 0 的位本来就不动作，空的那一拨送进去等于什么都没干。于是两个分支合并成了一个表达式，连"当前是什么电平"都不需要自己知道。

还有个细节。这里读了 `ODR`，和前面说的"不要用 `ODR`"不冲突：那里说的是不要用 `ODR` 去**改**电平，因为"读旧值再写回"会把别人的改动盖掉。这里读出来的值只参与本次计算，写回去的是 `BSRR` 而不是 `ODR`，就算中间被中断打断，最多是这一次翻转的结果不对，不会牵连到别的引脚。

---

## 四、 四个灯一起跑

### 4.1 一个引脚一个引脚地写太啰嗦

一个灯闪得再欢也只是个灯。想把四个都串起来，从左到右依次亮一遍。

第一反应当然是复制。第四颗的时候我已经在改第三遍了，除了引脚号之外，四行代码长得一模一样。既然只有引脚号在变，那就把引脚号收进一个数组：

```c
uint16_t gleds[] = {GPIO_PIN_8, GPIO_PIN_9, GPIO_PIN_10, GPIO_PIN_11};
const uint16_t gleds_num = sizeof(gleds) / sizeof(gleds[0]);
```

第二行那个除法是数元素个数的老办法：`sizeof` 一个数组得到的是整个数组占的字节数，除以单个元素占的字节数，就是元素个数。这样以后往数组里加引脚，个数会自动跟着变，不用回来改两处。

有了数组，跑马灯就是一个循环：

```c
for (int i = 0; i < gleds_num; i++)
{
  HAL_GPIO_WritePin(GPIOF, gleds[i], GPIO_PIN_RESET);
  HAL_Delay(200);
  HAL_GPIO_WritePin(GPIOF, gleds[i], GPIO_PIN_SET);
}
```

点亮当前的、等 200 毫秒、再熄灭它，让下一个去亮。

### 4.2 跑完一圈又跳回开头

这样跑起来是有毛病的：四个灯从右跑到左跑完，下一个亮的直接跳最左边那个，中间没有过渡，看着像被揪回去的。

补一个反向的循环，让它原路折回来：

```c
for (int i = gleds_num - 1; i >= 0; i--)
{
  HAL_GPIO_WritePin(GPIOF, gleds[i], GPIO_PIN_RESET);
  HAL_Delay(200);
  HAL_GPIO_WritePin(GPIOF, gleds[i], GPIO_PIN_SET);
}
```

这个循环有两个地方要留神。

下标从 `gleds_num - 1` 开始，不是 `gleds_num`。数组有四个元素，下标只到 3，写 `gleds_num` 就已经越界了。

循环变量必须是 `int`。如果把 `i` 声明成 `unsigned`，`i >= 0` 这个条件就永远成立：无符号数减到 0 之后再减一次，会回绕成最大值，循环再也出不来。这里刚好踩在正数的一侧，没事，但是个值得记住的形状。

这两个循环接起来还有个副作用：正向跑完的最后一颗和反向开始的第一颗是同一个灯，它刚熄灭就立刻又被点亮，中间没有停顿，看上去那一颗会连闪两下。

### 4.3 一次熄灭所有灯

跑马灯之前的那个准备工作是这样写的：

```c
for (int i = 0; i < gleds_num; i++)
{
  HAL_GPIO_WritePin(GPIOF, gleds[i], GPIO_PIN_SET);
}
```

四个灯挨个熄灭。但这个循环其实是可以省掉的：

```c
HAL_GPIO_WritePin(GPIOF, GPIO_PIN_8 | GPIO_PIN_9 | GPIO_PIN_10 | GPIO_PIN_11, GPIO_PIN_SET);
```

一句话把四个位图或起来，一次性写进 `BSRR`。

这就回到 2.2 节那个问题上：`GPIO_PIN_x` 为什么不是一个简单的编号。因为它不是一个"值"，是十六个格子上画的点，或起来就是"这几号引脚"。要是当初真按 `8`、`9`、`10`、`11` 来定义，这句话根本没法写。

`BSRR` 这边也天然接得住：写 1 的位动作、写 0 的位不动，所以位图里有几个 1，就同时操作几个引脚，互不打扰。

```text
GPIO_PIN_8 | GPIO_PIN_9 | GPIO_PIN_10 | GPIO_PIN_11

  0000 1111 0000 0000
       ^^^^
   第 8、9、10、11 位

写进 BSRR 的低 16 位，这四位对应的引脚一起输出高电平
```

---

## 五、 回头看那份自动生成的初始化

### 5.1 它做了三件事

自己该写的都写完了，可以回头交代一下开头那份初始化代码了。它一直待在那儿，我之前只顾着往 `main` 里加东西，没细看：

```c
static void MX_GPIO_Init(void)
{
  GPIO_InitTypeDef GPIO_InitStruct = {0};

  /* GPIO Ports Clock Enable */
  __HAL_RCC_GPIOF_CLK_ENABLE();
  __HAL_RCC_GPIOA_CLK_ENABLE();

  /*Configure GPIO pin Output Level */
  HAL_GPIO_WritePin(GPIOF, GPIO_PIN_8|GPIO_PIN_9|GPIO_PIN_10|GPIO_PIN_11, GPIO_PIN_SET);

  /*Configure GPIO pins : PF8 PF9 PF10 PF11 */
  GPIO_InitStruct.Pin = GPIO_PIN_8|GPIO_PIN_9|GPIO_PIN_10|GPIO_PIN_11;
  GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_OD;
  GPIO_InitStruct.Pull = GPIO_NOPULL;
  GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
  HAL_GPIO_Init(GPIOF, &GPIO_InitStruct);
}
```

三件事：开时钟、定初始电平、配引脚。

**开时钟**

`RCC` 是 `Reset and Clock Control`，复位和时钟控制。芯片为了省电，各个外设的时钟默认是关着的，关着的时候哪怕往它的寄存器里写值也写不进去，读回来还是复位值。所以用哪个端口，得先把哪个端口的时钟打开。

同一段里还有一句 `__HAL_RCC_GPIOA_CLK_ENABLE()`，但下面并没有配置 `GPIOA` 的引脚。这是配置工具按它自己的默认设定带上的。

**先定电平，后配引脚**

这两句的先后顺序是有意的。

上电之后引脚默认是浮空输入。要是反过来，先把引脚配成输出、再去设电平，那"配成输出"的一瞬间，输出数据寄存器 `ODR` 里的值还是复位值 0，也就是低电平，四个灯会先亮一下再灭。

现在这个顺序就没这个问题：先往 `BSRR` 里写 1，把 `ODR` 置成高。这会儿引脚还是输入模式，写进去的值只是躺在 `ODR` 里不生效；等 `HAL_GPIO_Init` 把它切成输出，生效的已经是设好的高电平。上电到点灯之间，不会闪那一下。

**配引脚**

最后五行把要配置的内容装进一个结构体，交给 `HAL_GPIO_Init` 去办。结构体本身很简单：

```c
typedef struct
{
  uint32_t Pin;       /*!< Specifies the GPIO pins to be configured. */
  uint32_t Mode;      /*!< Specifies the operating mode for the selected pins. */
  uint32_t Pull;      /*!< Specifies the Pull-up or Pull-Down activation for the selected pins. */
  uint32_t Speed;     /*!< Specifies the speed for the selected pins. */
} GPIO_InitTypeDef;
```

| 成员 | 这一段里的取值 | 含义 |
| --- | --- | --- |
| `Pin` | `GPIO_PIN_8 \| GPIO_PIN_9 \| GPIO_PIN_10 \| GPIO_PIN_11` | 要配置哪几个引脚 |
| `Mode` | `GPIO_MODE_OUTPUT_OD` | 通用开漏输出 |
| `Pull` | `GPIO_NOPULL` | 不启用内部上下拉 |
| `Speed` | `GPIO_SPEED_FREQ_LOW` | 输出速度 2 MHz |

这个函数的两个参数：

```text
void HAL_GPIO_Init(GPIO_TypeDef  *GPIOx, GPIO_InitTypeDef *GPIO_Init)
                                  |                        |
                                  |                        |
                                  |                        GPIO_Init：配置结构体指针
                                  |                          成员含义见上表
                                  | GPIOx：目标端口
                                    取值 GPIOA 到 GPIOG
```

### 5.2 从结构体到寄存器

`HAL_GPIO_Init` 是这两个里最长的函数，两百多行，一大半在处理中断相关的配置，跟点灯没关系，先跳过。留下的骨架是这样的：

```c
void HAL_GPIO_Init(GPIO_TypeDef  *GPIOx, GPIO_InitTypeDef *GPIO_Init)
{
  ...
  while (((GPIO_Init->Pin) >> position) != 0x00u)
  {
    ioposition = (0x01uL << position);
    iocurrent = (uint32_t)(GPIO_Init->Pin) & ioposition;

    if (iocurrent == ioposition)
    {
      switch (GPIO_Init->Mode)
      {
        case GPIO_MODE_OUTPUT_OD:
          config = GPIO_Init->Speed + GPIO_CR_CNF_GP_OUTPUT_OD;
          break;
        ...
      }

      configregister = (iocurrent < GPIO_PIN_8) ? &GPIOx->CRL : &GPIOx->CRH;
      registeroffset = (iocurrent < GPIO_PIN_8) ? (position << 2u) : ((position - 8u) << 2u);

      MODIFY_REG((*configregister),
                 ((GPIO_CRL_MODE0 | GPIO_CRL_CNF0) << registeroffset),
                 (config << registeroffset));
    }

    position++;
  }
}
```

**扫位图**

传进来的 `Pin` 是个位图，可能只有一位是 1，也可能好几位。要一个个处理，就得把位图从头到尾扫一遍：

```c
while (((GPIO_Init->Pin) >> position) != 0x00u)
```

把位图右移 `position` 位，如果结果不是 0，说明还有没处理完的高位。

```text
position = 8    位图 >> 8     0000 0000 0000 1111    != 0，继续
position = 12   位图 >> 12    0000 0000 0000 0000    == 0，停下
```

**看这一位在不在**

```c
ioposition = (0x01uL << position);
```

造一个"只有第 `position` 位是 1"的数，也就是"第 `position` 号引脚"单独的位图。然后拿它去和传进来的位图相与：

```text
position = 8

  ioposition   0000 0000 0000 0000 0000 0001 0000 0000
  Pin          0000 0000 0000 0000 0000 1111 0000 0000
  iocurrent    0000 0000 0000 0000 0000 0001 0000 0000
                                                 ^
                                          与 ioposition 相同
                                          -> 第 8 号引脚在名单里，处理它
```

`if (iocurrent == ioposition)` 判断的就是这件事。不在名单里的位相与之后是 0，跟 `ioposition` 对不上，跳过。

**拼出 4 位的配置值**

```c
config = GPIO_Init->Speed + GPIO_CR_CNF_GP_OUTPUT_OD;
```

`CRL` 和 `CRH` 里每个引脚占 4 位，高 2 位是 `CNF`，低 2 位是 `MODE`。这两个宏的值设计得很巧，它们本身就是"已经落在 4 位组里该在位置上的值"：

```text
Speed（GPIO_SPEED_FREQ_LOW）      0x02   0000 0010
CNF（GPIO_CR_CNF_GP_OUTPUT_OD）   0x04   0000 0100
-----------------------------------------------
相加                               0x06   0000 0110

  低 2 位 MODE = 10  -> 输出模式，最大 2 MHz
  高 2 位 CNF  = 01  -> 通用开漏输出
```

两位各占各的位置，所以这里用加法和用按位或是一样的效果。

顺带记一下 `MODE` 那两位的编码，它不是顺着排的：

| `MODE[1:0]` | 含义 |
| --- | --- |
| `00` | 输入模式 |
| `01` | 输出模式，最大 10 MHz |
| `10` | 输出模式，最大 2 MHz |
| `11` | 输出模式，最大 50 MHz |

最低档是 `10` 不是 `01`。所以 `GPIO_SPEED_FREQ_LOW` 的值是 `0x02`。

**决定写哪个寄存器的哪四位**

引脚 0 到 7 的配置在 `GPIOx_CRL`，8 到 15 在 `GPIOx_CRH`：

| 寄存器 | 管哪些引脚 | 每个引脚占几位 |
| --- | --- | --- |
| `GPIOx_CRL` | 引脚 0 ~ 7 | 4 位 |
| `GPIOx_CRH` | 引脚 8 ~ 15 | 4 位 |

手册里这两个寄存器的位定义是下面两张图，每个引脚占的那 4 位都能对上号：

![[STM32/attachments/STM32_GPIOx_CRL寄存器.png|参考手册里端口配置低寄存器 GPIOx_CRL 的位定义]]

![[STM32/attachments/STM32_GPIOx_CRH寄存器.png|参考手册里端口配置高寄存器 GPIOx_CRH 的位定义]]

```c
configregister = (iocurrent < GPIO_PIN_8) ? &GPIOx->CRL : &GPIOx->CRH;
registeroffset = (iocurrent < GPIO_PIN_8) ? (position << 2u) : ((position - 8u) << 2u);
```

`iocurrent` 是刚才那个单独的位图，`GPIO_PIN_8` 的值是 `0x0100`，比它大就说明引脚号在 8 以上，去 `CRH`。

`registeroffset` 是"在寄存器里从第几位开始"。每个引脚占 4 位，第 n 个引脚就从第 4n 位开始，乘 4 写成移位就是左移 2 位。

对我们这颗 `PF8`：`position` 是 8，走 `CRH` 分支，`(8 - 8) << 2` 得 0，也就是 `CRH` 的最低那 4 位。

那这 4 位各自是什么？手册里把它们拆成两个字段：高 2 位叫 `CNFy[1:0]`，低 2 位叫 `MODEy[1:0]`，`y` 是引脚号。`MODE` 管"输入还是输出、输出多快"，`CNF` 管"具体是哪一种"。两个字段一组合，就成了手册表 17 那张端口位配置表：

| `MODE[1:0]` | `CNF[1:0]` | 模式 |
| --- | --- | --- |
| `00` | `00` | 输入 · 模拟 |
| `00` | `01` | 输入 · 浮空 |
| `00` | `10` | 输入 · 上拉或下拉（靠 `ODR` 那一位分） |
| `01` / `10` / `11` | `00` | 通用输出 · 推挽 |
| `01` / `10` / `11` | `01` | 通用输出 · 开漏 |
| `01` / `10` / `11` | `10` | 复用功能输出 · 推挽 |
| `01` / `10` / `11` | `11` | 复用功能输出 · 开漏 |

`MODE` 那三个非零取值对应的速度，手册单独列了一张表：

| `MODE[1:0]` | 最大输出速度 |
| --- | --- |
| `01` | 10 MHz |
| `10` | 2 MHz |
| `11` | 50 MHz |

竖着看能看出这两个字段各管一摊：`MODE` 管速度、`CNF` 管推挽开漏还是复用功能，谁也不碍谁。这也正是 `config = Speed + CNF` 能直接相加的道理，它们本来就在不同的位上。

上一篇讲的那八种模式，到这里才落到具体的地方。当时说的"输入四种、输出四种"，翻到寄存器这一层就是上面那张表。对上拉和下拉的区别也在表里：两者的 `CNF` 都是 `10`，靠 `ODR` 那一位的值分家，置 1 是上拉、置 0 是下拉。库里处理输入模式时还有两行，`GPIOx->BSRR = ioposition` 和 `GPIOx->BRR = ioposition`，做的就是这件事。

对我们这四盏灯来说，`CNF = 01`、`MODE = 10`，拼起来是 `0110`，正是 `config` 算出来的 `0x06`。

顺带看一眼复位值。`CRL` 和 `CRH` 的复位值都是 `0x4444 4444`，每个 `4` 拆成 4 位是 `0100`，也就是 `CNF = 01`、`MODE = 00`。手册 8.1.1 节的原话是"复位期间和刚复位后，复用功能未开启，I/O 端口被配置成浮空输入模式"。所谓"上电之后引脚默认是浮空输入"，在寄存器层就是这一个数。

**读、清、写**

```c
#define WRITE_REG(REG, VAL)   ((REG) = (VAL))
#define READ_REG(REG)         ((REG))
#define MODIFY_REG(REG, CLEARMASK, SETMASK)  \
  WRITE_REG((REG), (((READ_REG(REG)) & (~(CLEARMASK))) | (SETMASK)))
```

把宏展开，是四步：读出原值、把要改的那 4 位清零、把新值填进去、写回。

对 `PF8` 来说，`CLEARMASK` 是 `(GPIO_CRL_MODE0 | GPIO_CRL_CNF0) << 0`，也就是 `0xF`，四个 1 正好盖住一组：

```text
CRH 原值     0000 0000 0000 0000 0000 0000 0101 1010
~CLEARMASK   1111 1111 1111 1111 1111 1111 1111 0000
相与         0000 0000 0000 0000 0000 0000 0101 0000    低 4 位清掉
SETMASK      0000 0000 0000 0000 0000 0000 0000 0110    要填的新值
相或         0000 0000 0000 0000 0000 0000 0101 0110    填好了
```

这里**必须先清空再填**。因为或运算只能把 0 变成 1，变不回 0：万一新值某位是 0、原值那位偏偏是 1，直接或上去还是 1，这组就改不动了。所以先把 4 位整组抹掉，再整体填进去。

到这里，配置里选的"通用开漏、2 MHz"就变成了 `CRH` 里最低那 4 位的 `0110`。要是哪天改回推挽输出，`config` 会变成 `0000 0010`，落进寄存器的这 4 位也跟着变。

---

## 六、 接口清单与容易踩的地方

### 6.1 这篇用到的接口

| 接口 | 作用 | 留意的地方 |
| --- | --- | --- |
| `HAL_GPIO_WritePin` | 把位图上对应的引脚置高或置低 | 可以一次传多个引脚；不看引脚当前是什么电平 |
| `HAL_GPIO_TogglePin` | 翻转位图上对应引脚的电平 | 内部先读 `ODR`、再写 `BSRR` |
| `HAL_GPIO_Init` | 按结构体里的设定配置引脚 | 同样支持一次配置多个引脚 |
| `HAL_Delay` | 毫秒级延时 | 靠 `SysTick` 定时器中断计数，初始化在 `HAL_Init` 里做完了 |
| `GPIO_PIN_x` | 引脚位图 | 表示"第几个位置"，不是引脚编号 |
| `GPIO_PIN_SET` / `GPIO_PIN_RESET` | 目标电平 | 值分别是 1 和 0，直接写数字也能跑，但读起来费劲 |
| `__HAL_RCC_GPIOx_CLK_ENABLE()` | 打开端口时钟 | 不开时钟，端口寄存器写不进去 |

### 6.2 几个记住的点

**低电平点亮。** 灯是共阳极接法，阳极朝电源、阴极朝引脚，所以 `GPIO_PIN_RESET` 是点亮。换一块板子、换一种接法，这个结论就可能反过来，写之前先看接线。

**`BSRR` 里写 0 的位不动作。** 这一条是后面所有位运算的地基。正因为写 0 不影响任何东西，才能用"一次写"代替"读—改—写"。

**位图描述的是位置，不是数值。** `GPIO_PIN_8` 等于 `0x0100`，它说的是"第 8 个格子涂黑"，不是一个能参与加减的数字。

**每个引脚的配置占 4 位，高 2 位是 `CNF`、低 2 位是 `MODE`。** 引脚 0 到 7 在 `CRL`，8 到 15 在 `CRH`，所以第 n 个引脚从第 4n 位开始。

**改寄存器的固定套路是先清后填。** 或运算只能把 0 变 1，清不掉的位会把新值挡住。

**倒着走的循环，变量要用有符号的。** `for (int i = n - 1; i >= 0; i--)` 写成无符号就会绕回去停不下来。

---

## 相关笔记 / 相关链接

- 相关目录：[[STM32目录]]
- 相关笔记：[[STM32/GPIO 概念篇——从端口位电路到八种工作模式]]（推挽与开漏的电路原理、`N-MOS` 与 `P-MOS` 怎么导通）
- 相关笔记：[[C语言/移位操作符、位操作符]]（左移、按位与、按位或的基础）





