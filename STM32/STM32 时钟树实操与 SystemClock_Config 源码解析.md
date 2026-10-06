---
tags:
  - STM32
  - 嵌入式
aliases:
  - RCC实操
日期: 2026年10月2日
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

前面几篇点灯、读按键，代码里一直有一个我从没正经写过的函数：`SystemClock_Config()`。每次新建工程，CubeMX 都替我把它填好，能跑，我就没再管它。

[[RCC 概念篇——单片机的心跳从哪来]] 把时钟树的道理讲完了：四个时钟源、PLL 倍频、AHB 与 APB 逐级分频、外设时钟门控。道理都懂了，但那棵树怎么用代码配出来、参数填错一点会发生什么，不亲手配一次心里还是没底。

## 一、 频率探针 CPU_Delay

延时函数代码如下：

```c
void CPU_Delay(volatile uint32_t cnt)
{
  while (cnt--)
    ;
}
```

参数上的 `volatile` 不是可有可无。这个循环递减变量、不产生任何外部效果，开了优化之后编译器完全有权把它整个删掉，延时直接消失；加上 `volatile`，编译器就不许动它了，每次循环都得老老实实读一次内存、减一次、写回去。代价是延时不精确，好处是它完全跟着 CPU 的节奏走。

这正好是我要的性质。`HAL_Delay()` 靠 SysTick（系统节拍定时器）中断计的是真实毫秒，主频怎么变它都准，反而看不出时钟配置的效果；`CPU_Delay` 数的是 CPU 自己的节拍，主频多高、数得多快。把 LED（发光二极管）接在 `PF8` 上，翻转一次歇一个延时，闪的快慢就成了时钟频率的直观展示：一个不占用任何外设的频率探针。

就像军训喊拍子走正步：拍子喊得越密，走完同样 100 米用时越短。数数快慢看不出来，正步快慢谁都看得见。

## 二、 默认时钟下的第一版

先看什么都不配的情况。STM32F103 复位之后，时钟控制寄存器 `RCC_CR` 的复位值是 `0x0000 0083`，最低两位 `HSION` 和 `HSIRDY` 都是 1：内部 8 MHz 的 `HSI`（High Speed Internal，高速内部 RC 振荡器）已经在跑，而且它直接接在 `SYSCLK`（系统时钟）的选择器上。也就是说，芯片出厂的默认状态就是能跑的，复位值早就替你把第一版配置填好了。

CubeMX 新建工程、什么都不动的时候，生成的代码就维持了这个状态：HSI 开着，SYSCLK 直接选 HSI，没有倍频也没有分频。配上 GPIO（通用输入输出口）翻转和探针，第一版程序就齐了：

```c
int main(void)
{
  HAL_Init();
  SystemClock_Config();   // 此时只把 HSI 打开，8 MHz 直供 SYSCLK
  MX_GPIO_Init();

  while (1)
  {
    HAL_GPIO_TogglePin(GPIOF, GPIO_PIN_8);
    CPU_Delay(654545 * 2);
  }
}
```

框架就这四样：初始化 HAL、配时钟、配 GPIO、死循环里翻转加延时。后面无论怎么改，只需要动 `SystemClock_Config()` 一个函数。`CPU_Delay(654545 * 2)` 这个数是按主频估出来的：数一次几个周期，乘出来零点几秒，让灯闪肉眼可辨，具体多大不重要，重要的是后面几组对比共用同一个数。`HAL`（Hardware Abstraction Layer，硬件抽象层）把外设初始化收拢成固定套路，时钟这一环单独拎出来配。

## 三、 CubeMX 配置与 SWD

用 CubeMX 把工程配到「能上 72 MHz」的状态，一共五步：

1. `SYS` 里把 Debug 选成 `Serial Wire`；
2. `RCC` 里 HSE（High Speed External，高速外部晶振）选 `Crystal/Ceramic Resonator`，意思是板上有 8 MHz 晶体；
3. 时钟树界面里把 HSE 接进 PLL，倍频拉到 ×9，SYSCLK 顶到 72 MHz；
4. `PF8` 配成推挽输出，接灯；
5. 项目管理里 Toolchain 选 `MDK-ARM`，生成代码。

第 1 步非常重要，不能省。默认的 No Debug 不给调试口留任何配置，第一次烧录往往正常，程序跑起来之后调试器就可能再也连不上，第二次烧录就麻烦了。SWD（Serial Wire Debug，串行调试接口）就四根线：`3.3V`、`GND`、`SWDIO`、`SWCLK`，选中 Serial Wire 之后，CubeMX 生成的代码里才有对应的调试口配置，调试通道才算真正可用。

![[STM32/attachments/STM32开发板SWD接口.jpg|SWD 接口与四根连接线，SWDIO 和 SWCLK 各一根]]

HSE 选 Crystal 之后，时钟树界面会直接显示每级的频率，倍频系数一点，72 MHz 就出来了。这张图值得多看两眼：写进代码的每个参数，都能在图上找到自己的位置。

![[STM32/attachments/CubeMX时钟树配置界面.png|CubeMX 时钟树配置界面，HSE 8 MHz 经 PLL 九倍频到 72 MHz]]

## 四、 SystemClock_Config 与 72MHz

第二版把 SYSCLK 拉到 F103 的顶格 72 MHz。代码如下：

```c
void SystemClock_Config(void)
{
  RCC_OscInitTypeDef RCC_OscInitStruct = {0};
  RCC_ClkInitTypeDef RCC_ClkInitStruct = {0};

  RCC_OscInitStruct.OscillatorType = RCC_OSCILLATORTYPE_HSE;  // 要配的振荡器：HSE
  RCC_OscInitStruct.HSEState = RCC_HSE_ON;                    // 打开 HSE
  RCC_OscInitStruct.HSEPredivValue = RCC_HSE_PREDIV_DIV1;     // HSE 进 PLL 前先除以 1
  RCC_OscInitStruct.HSIState = RCC_HSI_ON;                    // HSI 保持开启
  RCC_OscInitStruct.PLL.PLLState = RCC_PLL_ON;                // 打开 PLL
  RCC_OscInitStruct.PLL.PLLSource = RCC_PLLSOURCE_HSE;        // PLL 的源选 HSE
  RCC_OscInitStruct.PLL.PLLMUL = RCC_PLL_MUL9;                // 9 倍频
  if (HAL_RCC_OscConfig(&RCC_OscInitStruct) != HAL_OK)
  {
    Error_Handler();
  }

  RCC_ClkInitStruct.ClockType = RCC_CLOCKTYPE_HCLK | RCC_CLOCKTYPE_SYSCLK
                              | RCC_CLOCKTYPE_PCLK1 | RCC_CLOCKTYPE_PCLK2;
  RCC_ClkInitStruct.SYSCLKSource = RCC_SYSCLKSOURCE_PLLCLK;   // SYSCLK 选 PLL 输出
  RCC_ClkInitStruct.AHBCLKDivider = RCC_SYSCLK_DIV4;          // AHB 四分频
  RCC_ClkInitStruct.APB1CLKDivider = RCC_HCLK_DIV2;           // APB1 再二分频
  RCC_ClkInitStruct.APB2CLKDivider = RCC_HCLK_DIV1;           // APB2 不分频
  if (HAL_RCC_ClockConfig(&RCC_ClkInitStruct, FLASH_LATENCY_2) != HAL_OK)
  {
    Error_Handler();
  }
}
```

函数分成两半。前一半 `HAL_RCC_OscConfig()` 管振荡器：选哪个源、开不开、倍到多高。8 MHz 的 HSE 先过 `PREDIV` 除以 1，再进 PLL（Phase Locked Loop，锁相环）乘 9，得到 72 MHz 的 PLLCLK。后一半 `HAL_RCC_ClockConfig()` 管总线：SYSCLK 的选择器拨到 PLL，然后 AHB、APB1、APB2 各自分频。`ClockType` 那一行是在声明「这次要配哪几个时钟」，四个都选上了。

HSI 为什么还开着？时钟已经切到 PLL，HSI 留着不碍事；真出问题的时候，它还是唯一能立刻顶上的后备源。概念篇里讲过的 CSS（时钟安全系统）管的是 HSE 出事后自动切回 HSI，这里先把 HSI 开着，思路是一样的：不把退路堵死。

AHB 分频我填的却是 ÷4，72 MHz 的 SYSCLK 分完频之后，`HCLK`（AHB 总线时钟，也是 Cortex-M3 内核的自由运行时钟 FCLK，Free running CLocK）只剩 18 MHz。这不是手滑：FCLK 就是 CPU 数数的节拍，故意留的低主频正好给下一节的分频实验当基准。

## 五、 分频实验与 CFGR 位段

同一份代码，改一行分频，灯的快慢肉眼可见。三组配置表格如下：

| 组 | SYSCLK | AHB 分频 | HCLK | APB1 分频 | PCLK1 | APB2 分频 | PCLK2 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 一 | 72 MHz | ÷8 | 9 MHz | ÷2 | 4.5 MHz | ÷1 | 9 MHz |
| 二（我的版本） | 72 MHz | ÷4 | 18 MHz | ÷2 | 9 MHz | ÷1 | 18 MHz |
| 三 | 72 MHz | ÷4 | 18 MHz | ÷8 | 2.25 MHz | ÷16 | 1.125 MHz |

`CPU_Delay` 数数的节奏由 FCLK 决定，FCLK 就是 HCLK。第二组烧进去，同一个 `CPU_Delay(654545 * 2)`，灯翻一次的间隔是 AHB 不分频时的 4 倍，慢下来的幅度肉眼一下就能分辨。第一组 HCLK 只剩 9 MHz，间隔更长；第三组分频看着最狠，但注意看表：它的 HCLK 和第二组一样都是 18 MHz。灯翻的快慢只由 FCLK 决定，APB1 分到 ÷8、APB2 分到 ÷16 都不改变 CPU 数数的速度，受影响的是挂在 APB 上的外设：定时器、串口这些的节奏被放慢了。三组的 SYSCLK 都是 72 MHz，对比下来就是一句话：**倍频决定 SYSCLK 能到多高，AHB 分频决定 CPU 实际跑多快，APB 分频管的是外设那一头**。

这些分频系数落在 `RCC_CFGR` 里。上一篇提到过这个寄存器，它的位段表格如下：

| 位段 | 位号 | 管什么 | 本篇用到的取值 |
| --- | --- | --- | --- |
| `SW[1:0]` | 1:0 | SYSCLK 选谁 | `10` 选 PLL |
| `SWS[1:0]` | 3:2 | 切换状态（硬件回填） | `10` 表示已切到 PLL |
| `HPRE[3:0]` | 7:4 | AHB 分频 | `0xxx` 不分频，`1000` 起 ÷2、÷4、÷8、÷16 |
| `PPRE1[2:0]` | 10:8 | APB1 分频 | `0xx` 不分频，`100` ÷2、`101` ÷4、`110` ÷8、`111` ÷16 |
| `PPRE2[2:0]` | 13:11 | APB2 分频 | 同 PPRE1 编码 |
| `ADCPRE[1:0]` | 15:14 | ADC 分频 | ÷2/÷4/÷6/÷8 |
| `PLLSRC` | 16 | PLL 源 | `1` 选 HSE |
| `PLLMUL[3:0]` | 21:18 | 倍频系数 | `0111` 是 ×9 |

APB1 二分频对应的宏是 `RCC_HCLK_DIV2`，值 `0x00000400`，翻译成二进制正好落在 bit 10：

```text
RCC_HCLK_DIV2 = 0x00000400
              = 0000 0100 0000 0000
                      ^^^
                      bit 10:8 = 100，写进 PPRE1[2:0]，APB1 就是 HCLK / 2
```

注意写进去的手段不是 `|=`。分频位段的值是「编码」不是「开关」：`|=` 只能把位从 0 变 1，碰上已经置 1 的旧值就无能为力。HAL 里用的是 `MODIFY_REG`，先清位段、再填新值：

```c
MODIFY_REG(RCC->CFGR, RCC_CFGR_PPRE1, RCC_ClkInitStruct.APB1CLKDivider);
// 等价于：
RCC->CFGR = (RCC->CFGR & ~RCC_CFGR_PPRE1) | RCC_ClkInitStruct.APB1CLKDivider;
```

`HAL_RCC_ClockConfig()` 把 SYSCLK 切到 PLL 用的也是同样的手法：先写 `SW = 10`，然后轮询 `SWS`。SW 是给选择器的指令，SWS 是硬件回报的实际状态，等 SWS 也变成 `10`，切换才算真正完成。指令和状态分成两位，硬件同步需要时间这件事，寄存器设计里直接体现了出来。

## 六、 从结构体到寄存器位

`HAL_RCC_OscConfig()` 和 `HAL_RCC_ClockConfig()` 只是入口，参数从结构体传到寄存器位，中间还隔着两层。拿「打开 HSE」这一步当例子，图示如下：

```text
RCC_OscInitStruct.HSEState = RCC_HSE_ON
        |
        v
HAL_RCC_OscConfig(&RCC_OscInitStruct)          HAL 函数：编排配置流程
        |
        v
__HAL_RCC_HSE_CONFIG(RCC_HSE_ON)               宏：按参数分流
        |
        v
SET_BIT(RCC->CR, RCC_CR_HSEON)                 寄存器位：CR 的 bit16 置 1
```

**第一层，寄存器在哪。** RCC（Reset and Clock Control，复位与时钟控制器）挂在 AHB 总线上，这一段外设地址从 `0x40020000` 开始，RCC 占据其中的 `0x40021000`。库里用结构体描述它的寄存器组：

```c
#define RCC_BASE   (AHBPERIPH_BASE + 0x00001000UL)   // 0x40021000
#define RCC        ((RCC_TypeDef *) RCC_BASE)

typedef struct
{
  __IO uint32_t CR;         /*!< 时钟控制寄存器，HSE/HSI/PLL 的开关与就绪标志 */
  __IO uint32_t CFGR;       /*!< 时钟配置寄存器，SW/HPRE/PPRE1/PPRE2/PLLMUL */
  __IO uint32_t CIR;        /*!< 时钟中断寄存器 */
  __IO uint32_t APB2RSTR;   /*!< APB2 外设复位寄存器 */
  __IO uint32_t APB1RSTR;   /*!< APB1 外设复位寄存器 */
  __IO uint32_t AHBENR;     /*!< AHB 外设时钟使能寄存器 */
  __IO uint32_t APB2ENR;    /*!< APB2 外设时钟使能寄存器 */
  __IO uint32_t APB1ENR;    /*!< APB1 外设时钟使能寄存器 */
  __IO uint32_t BDCR;       /*!< 备份域控制寄存器 */
  __IO uint32_t CSR;        /*!< 控制与状态寄存器 */
} RCC_TypeDef;
```

成员顺序就是寄存器地址顺序，每个 `uint32_t` 占 4 字节。指针一指基地址，`RCC->CR` 就是 `0x40021000`，`RCC->CFGR` 就是 `0x40021004`。GPIO 篇拆 `GPIO_TypeDef` 时用的同一套指针映射把戏，这里不过换了个外设。

**第二层，开时钟的宏。** 点灯前必写的 `__HAL_RCC_GPIOF_CLK_ENABLE()`，展开代码如下：

```c
#define __HAL_RCC_GPIOF_CLK_ENABLE()                        \
  do {                                                      \
    __IO uint32_t tmpreg;                                   \
    SET_BIT(RCC->APB2ENR, RCC_APB2ENR_IOPFEN);              \
    tmpreg = READ_BIT(RCC->APB2ENR, RCC_APB2ENR_IOPFEN);    \
    UNUSED(tmpreg);                                         \
  } while (0)
```

`SET_BIT` 落到实处就是 `RCC->APB2ENR |= RCC_APB2ENR_IOPFEN`，把 `IOPFEN`（bit 7）置 1，GPIOF 的时钟门打开。后面跟着读一次同一个寄存器的 `tmpreg`，读出的值直接丢弃，作用是给硬件留一点时钟稳定的时间，这也是 HAL 代码里反复出现的固定节奏。

时钟没开会发生什么，GPIO 篇提过一句：外设寄存器读永远返回 `0x0`，写也不生效。这次在 RCC 这里看得更清楚，`APB2ENR` 的每一位就是一扇门，门不开，门后的寄存器无论怎么读写都是空气。所以「外设怎么写都没反应」这类怪问题，第一个该查的就是对应 ENR 位。

**第三层，振荡器开关的宏。** `__HAL_RCC_HSE_CONFIG()` 按参数决定置位还是清位，展开代码如下：

```c
#define __HAL_RCC_HSE_CONFIG(__STATE__)       \
  do {                                        \
    if ((__STATE__) == RCC_HSE_ON)            \
    {                                         \
      SET_BIT(RCC->CR, RCC_CR_HSEON);         \
    }                                         \
    else if ((__STATE__) == RCC_HSE_OFF)      \
    {                                         \
      CLEAR_BIT(RCC->CR, RCC_CR_HSEON);       \
    }                                         \
    else if ((__STATE__) == RCC_HSE_BYPASS)   \
    {                                         \
      SET_BIT(RCC->CR, RCC_CR_HSEBYP);        \
      SET_BIT(RCC->CR, RCC_CR_HSEON);         \
    }                                         \
    else                                      \
    {                                         \
      CLEAR_BIT(RCC->CR, RCC_CR_HSEON);       \
      CLEAR_BIT(RCC->CR, RCC_CR_HSEBYP);      \
    }                                         \
  } while (0)
```

`RCC_CR_HSEON` 是 `0x00010000`，CR 的 bit 16。结构体里一句 `HSEState = RCC_HSE_ON`，最终就是落到这一个位上。`HAL_RCC_OscConfig()` 把结构体里所有振荡器参数翻译成一串这样的宏调用，然后进入等状态的循环：晶振起振需要时间，HAL 轮询 `HSERDY`（HSE 就绪标志），等不到就超时返回错误。开振荡器和开外设时钟不一样，前者是模拟电路起振，得等；后者是数字门开关，立即生效。

**第四层，汇总。** 结构体是人机接口，参数有名字、有默认值；HAL 函数是流程编排，决定先配谁后配谁、每一步等多久；宏是位操作的缩写，把「置位、清位、改位段」封装成可读的写法；寄存器位是最终生效的一层。四层走完，`SystemClock_Config()` 里每一行对应的硬件动作就都清楚了。

## 七、 频率边界与等待周期

`HAL_RCC_ClockConfig()` 的第二个参数 `FLASH_LATENCY_2` 一直没解释。CPU 从 Flash 取指令，主频上去了，Flash 的读出速度跟不上怎么办？办法是每次取指多等几个周期。F103 的手册按 SYSCLK 划了三档：

| SYSCLK 范围 | 等待周期 | 本篇取值 |
| --- | --- | --- |
| 0 < SYSCLK ≤ 24 MHz | 0 | `FLASH_LATENCY_0` |
| 24 < SYSCLK ≤ 48 MHz | 1 | `FLASH_LATENCY_1` |
| 48 < SYSCLK ≤ 72 MHz | 2 | `FLASH_LATENCY_2` |

72 MHz 落在第三档，所以是 `FLASH_LATENCY_2`。这条是硬件约束，填小了取指出错，不是能省就省的偏好项。还有一个容易想岔的地方：判据是 SYSCLK，不是 HCLK。本篇实验配置里 HCLK 只有 18 MHz，但 SYSCLK 是 72 MHz，CubeMX 照样生成 `FLASH_LATENCY_2`，对照 CubeMX 生成的工程配置一看便知。

把主频降下来反而要跟着改。配一组 48 MHz 的对照：`PLLMUL` 从 ×9 换成 ×6，AHB 不分频，等待周期降到 `FLASH_LATENCY_1`：

```c
RCC_OscInitStruct.PLL.PLLMUL = RCC_PLL_MUL6;        // 8 MHz x 6 = 48 MHz
RCC_ClkInitStruct.AHBCLKDivider = RCC_SYSCLK_DIV1;  // AHB 不分频
RCC_ClkInitStruct.APB1CLKDivider = RCC_HCLK_DIV2;   // APB1 二分频
RCC_ClkInitStruct.APB2CLKDivider = RCC_HCLK_DIV1;   // APB2 不分频
if (HAL_RCC_ClockConfig(&RCC_ClkInitStruct, FLASH_LATENCY_1) != HAL_OK)
{
  Error_Handler();
}
```

48 MHz 落在第二档，等待周期就是 1。同一颗芯片，主频、分频、等待周期三个数绑在一起动，改一个就得核对其余两个。这组配置里 AHB 不分频，HCLK 直接跑 48 MHz，和前面 ÷4 的实验配置正好是两个方向：一个让总线跟满主频，一个故意让 CPU 慢下来。

APB1 还有条上限：这条总线挂的是 TIM2~7、USART2/3、I2C 这类外设，硬件规定它的时钟不得超过 36 MHz。72 MHz 全速时 APB1 至少得二分频；实验里的极端分频反而离上限很远。方向别记反了，「至少 ÷2」说的是必须分，不是只能分到 2。

到这里，`SystemClock_Config()` 已经不是黑盒了：选源、倍频、分频、切换，四步各对应一段寄存器，每一步的参数都有硬件约束兜着。回头看那只灯，它闪的每一次快慢变化，都是这几段寄存器里某一位被改写的直接后果。

## 相关笔记 / 相关链接

- 相关目录：[[STM32目录]]
- 相关笔记：[[STM32/RCC 概念篇——单片机的心跳从哪来]]（前置：时钟树结构、四源与 72 MHz 推导）、[[STM32/GPIO 输出与 HAL 源码解析]]（类比：拆宏拆寄存器的同一套方法）

---
