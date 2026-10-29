---
tags:
  - 形态/概念
  - 主题/并发
aliases:
  - Linux 进程控制
日期: 2026年10月28日
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
  <tr>
    <td align="center">🐧 Linux</td>
    <td align="center"><a href="https://blog.csdn.net/hahaha6565/category_13217143.html">进入专栏 →</a></td>
  </tr>
</table>

<br>

<p align="center">
  🏠 博客主页：<a href="https://blog.csdn.net/hahaha6565?type=blog">暖焰核心</a><br>
  📦 作者仓库：<a href="https://github.com/warm-flame-core">github</a> | <a href="https://gitee.com/warm-flame-core">gitee</a>
</p>


# 引言

在终端里敲完一条命令，紧接着敲 `echo $?`，会得到一个 0 到 255 之间的数字

它从哪来？`main` 最后那句 `return 0` 和这个数字是什么关系？为什么 `return 256` 得到的是 0、`return -1` 得到的是 255？为什么被 `Ctrl+C` 打断的命令，`$?` 是 130 而不是 2？

## 一、 进程退出的三种场景

一个进程结束的方式只有三类：

- 代码跑完了，结果是对的
- 代码跑完了，结果是错的
- 代码没跑完，被强行终止

前两类是进程自己决定退出，第三类是它没得选。这个区分决定了后面该看什么：前两类看退出码，第三类退出码没有意义，要看它收到了哪个信号

所以「退出码」这个词只在正常终止的语境里成立。一个被 `kill -9` 杀掉的进程，它的退出码没有任何含义，只是恰好占着那块内存位置
## 二、 退出码

### 2.1 return 值的去向

`main` 是个函数，它返回的值就是进程的退出码。中间这一步由 C 运行时的启动代码完成：`main` 返回后，启动代码接住 `%eax` 里的值，把它当作参数交给 `exit`

```asm
main:
    ...
    mov    $0x2a,%eax       ; main 的返回值落在 %eax
    pop    %rbp
    ret                     ; 回到启动代码，由它接手
```

启动代码那一层是 `__libc_start_main`，它拿到 `main` 的返回值之后调 `exit`。`return n` 和 `exit(n)` 等价，说的不是语法，是启动代码把两者接上了

### 2.2 低 8 位的截断

退出码在类型上是 `int`，但真正能传出去的只有低 8 位。看下面这组实测：

```text
return 0    -> $? = 0
return 1    -> $? = 1
return 255  -> $? = 255
return 256  -> $? = 0
return 257  -> $? = 1
return 511  -> $? = 255
return 512  -> $? = 0
return -1   -> $? = 255
```

`256` 和 `0` 分不开，`257` 和 `1` 分不开，`-1` 变成 `255`。规则就是取低 8 位，等价于 `n & 0xFF`

`-1` 那一行值得单独看：`-1` 的补码是 32 个 1，低 8 位全是 1，所以是 255。换成 `_exit(-1)`（直接进内核的那个退出函数，第三节细讲）结果一样是 255

### 2.3 从内核到 $?

退出码不是进程直接喊给 shell 的。中间隔了两层：

1. 进程退出时，内核把退出码存进它自己的 `task_struct`，字段是 `exit_code`
2. 父进程调 `wait` / `waitpid`，内核把 `exit_code` 打包成 `status` 交给父进程
3. shell 是父进程，它从 `status` 里解出退出码，存进 `$?`

`task_struct` 里有两个字段管这件事：

```c
int exit_code, exit_signal;
```

`exit_code` 存退出码或信号编号。`exit_signal` 存子进程退出时给父进程发哪个信号，这个字段是给 `wait` 用的唤醒依据

三层换算各有各的编码方式，看图更容易对上：

![[Linux/attachments/退出码来路.png|同一个退出码 10 在进程、task_struct、status、$? 四个位置的不同形态]]

同一个退出码 `10`，走一路变一次形态。这也是后面不能直接拿 `status` 和 `10` 比较的原因

### 2.4 信号终止与 $?

被信号杀死时，退出码没有意义，但 `$?` 依然有值。常见的有：

| `$?` | 信号 | 含义 |
| --- | --- | --- |
| 130 | `SIGINT`（2） | `Ctrl+C` |
| 137 | `SIGKILL`（9） | 强制杀死 |
| 139 | `SIGSEGV`（11） | 段错误 |
| 143 | `SIGTERM`（15） | 默认终止信号 |

规律是 `128 + n`，`n` 是信号编号。`128 + n` 很容易被当成内核的行为，其实不是。同一件事在两个视角下是两组数：

| 视角 | 被 `SIGKILL` | 被 `SIGTERM` |
| --- | --- | --- |
| `waitpid` 拿到的 `status` | `0x0009`，`WTERMSIG` = 9 | `0x000f`，`WTERMSIG` = 15 |
| shell 的 `$?` | 137 | 143 |

内核在 `status` 里放的就是信号编号本身（低 7 位），`128 + n` 是 shell 把「被信号终止」编码进 0 到 255 这个区间时自己加的偏移。因为 0 到 128 要留给正常退出码，信号从 129 开始排

### 2.5 退出码与 errno

两者都是整数，都表示「出了问题」，但取值域完全不相干：

- **退出码**是进程和调用者之间的约定，取值范围 0 到 255，含义由程序自己定义
- `errno` 是系统调用失败时内核填的错误码，取值域由系统定义，`ENOENT` 是 2、`EACCES` 是 13、`ENOMEM` 是 12

两者可以对接，前提是你主动去接。下面这段把 `fopen` 失败时的 `errno` 直接当退出码返回：

```c
#include <stdio.h>
#include <errno.h>

int main()
{
    FILE *fp = fopen("log.txt", "r");
    if (fp == NULL)
        return errno;

    fclose(fp);
    return 0;
}
```

文件不存在时 `errno` 是 `ENOENT`（2），`$?` 读回来正好是 2。比起写死 `return 1`，这样调用者能直接从退出码看出是「文件不存在」还是「权限不够」

对接时有两件事要注意。一是 `errno` 的值域远大于 255，超出部分会被截断。二是 `strerror` 只认识 `errno` 的那套编号：

```text
  2 : No such file or directory
 12 : Cannot allocate memory
 24 : Too many open files
 40 : Too many levels of symbolic links
134 : Unknown error 134
200 : Unknown error 200
```

`134` 和 `200` 都返回 `Unknown error`，因为这两个编号不在 `errno` 的名单里。`strerror` 查的是系统错误码，不是退出码，只有退出码恰好等于某个 `errno` 时两边才对得上

## 三、 exit 与 _exit

### 3.1 两条路的分岔

`exit` 和 `_exit` 都能结束进程，签名也长得一样：

```c
#include <stdlib.h>
void exit(int status);

#include <unistd.h>
void _exit(int status);
```

但一个是库函数，一个是系统调用。`exit` 在真正进内核之前还做了一堆事，最后才调 `_exit`。这堆事里最关键的是冲刷缓冲区

两条路的分岔：

![[Linux/attachments/exit与_exit两条路.png|exit 与 _exit 的分岔：exit 先跑 atexit 清理函数并冲刷缓冲区，_exit 直接进内核]]

两条路最后都进内核的 `do_exit`，差别全在路上做没做那几件事

### 3.2 缓冲区的位置

`printf` 不是直接写屏幕的。数据先进 C 库自己维护的缓冲区，攒够了或者遇到特定条件才交给内核。`exit` 会把这个缓冲区刷干净，`_exit` 不会

同一个程序，只换一个退出函数：

```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
    printf("hello");
    exit(0);
}
```

```c
#include <stdio.h>
#include <unistd.h>

int main()
{
    printf("hello");
    _exit(0);
}
```

在终端上跑，输出是这样：

```text
[exit ] 输出: hello
[_exit] 输出: （空）
```

`printf` 里没有 `\n`，这一点是前提。缓冲模式本身不是固定的，由 `isatty` 决定：

| `stdout` 指向 | 缓冲模式 | 什么时候刷 |
| --- | --- | --- |
| 终端（`isatty` 为真） | 行缓冲 | 遇到 `\n`，或缓冲区满 |
| 文件 / 管道（`isatty` 为假） | 全缓冲 | 缓冲区满，或 `fflush`，或 `exit` |

所以「有换行就会输出」只在终端上成立。把程序重定向到文件，有换行也不刷：

```text
printf("hello\n"); _exit(0);

终端：[hello]
管道：[]
```

终端下行缓冲遇到 `\n` 就把数据交给了内核，`_exit` 来晚了。管道下是全缓冲，`\n` 不算刷新条件，`_exit` 一跳过 `exit`，这行就没了

程序崩溃或者被 `kill` 掉之前打印的东西会丢，也是这个道理：崩溃走信号路径，不经过 `exit`，缓冲区里没刷的内容跟着一起没了。写日志的程序要扛住崩溃，就得自己 `fflush` 或者用不带缓冲的 `write`

### 3.3 exit 的三件事

在调 `_exit` 之前，`exit` 按顺序做这些：

1. 执行 `atexit` 或 `on_exit` 注册的清理函数
2. 关闭所有打开的流，把缓冲区数据全部写出
3. 调用 `_exit`

第 1 条单独验证一下：

```c
#include <stdio.h>
#include <stdlib.h>

void bye()
{
    printf("[atexit] 清理函数跑了\n");
}

int main()
{
    atexit(bye);
    printf("main 结束\n");
    return 0;
}
```

把最后的 `return 0` 换成 `_exit(0)`，输出就少一行：

```text
--- exit 版（终端）---
main 结束
[atexit] 清理函数跑了

--- _exit 版（终端）---
main 结束
```

`atexit` 注册的函数存在进程的退出处理链上，只有走 `exit` 这条路才会被遍历

`_exit` 版还能剩一行，是因为终端是行缓冲，`printf` 里的 `\n` 已经把它送出去了。同样的程序接到管道上，`_exit` 版一行都不剩

### 3.4 fork 与缓冲区复制

缓冲区在用户空间，属于进程的地址空间。`fork` 复制地址空间，缓冲区自然跟着复制一份。如果 `fork` 之前缓冲区里有没刷的数据，父子各带一份，最后两边都刷，屏幕上就看到两遍

```c
#include <stdio.h>
#include <unistd.h>

int main()
{
    printf("这一行没有换行");
    fork();
    return 0;
}
```

```text
终端：这一行没有换行这一行没有换行
管道：这一行没有换行这一行没有换行
```

两种都翻倍，因为 `fork` 复制的是缓冲区里的内容，跟缓冲模式无关。区别出在有换行的时候：

```text
printf("这一行有换行\n"); fork();

终端：这一行有换行          （只一遍）
管道：这一行有换行
      这一行有换行          （两遍）
```

终端下行缓冲已经把数据送走了，`fork` 复制到的是空缓冲区。管道下是全缓冲，数据还在，父子各刷一遍

这个现象在排查「`fork` 之后输出翻倍」时最常撞上。老办法是在 `fork` 之前 `fflush(stdout)`，两种缓冲模式下都对

## 四、 进程等待的必要性

### 4.1 僵尸进程与 kill 命令

子进程退出后，它占的代码和数据可以释放，但 `task_struct` 不能立刻销毁。父进程还要从里面读退出码，这份信息只有这一处存着

于是子进程进入僵尸状态：进程已经死了，`task_struct` 还挂在进程列表里。这时候 `kill -9` 对它无效，因为它不是一个可被调度的实体，没有可以接收信号的东西

实测确认它还占着位置：

```text
    PID    PPID STAT CMD
1862607 1862606 Z    [za] <defunct>
```

`STAT` 是 `Z`，命令名被方括号包住并带 `<defunct>`。父进程一调 `waitpid` 把它回收，这条记录才消失

### 4.2 等待的两件事

父进程通过等待完成两件事：

- 回收子进程的内核资源，让僵尸消失
- 取回子进程的退出信息，判断它干得怎么样

第二件事才是重点。父进程派活给子进程，总得知道结果：跑完了没有，结果对不对，是正常退出还是出了事

### 4.3 多子进程的等待粒度

一个父进程可以有多个子进程，它们的退出顺序不确定。所以「等」这个动作天然有两种粒度：等任意一个，还是等指定的那个。`wait` 和 `waitpid` 的区别就从这里开始

## 五、 wait 与退出信息

### 5.1 wait 的基本用法

`wait` 是最基础的一个，它等任意一个子进程退出，回收并返回它的 `pid`：

```c
#include <sys/types.h>
#include <sys/wait.h>

pid_t wait(int *status);
```

返回值是成功回收的子进程 `pid`，失败返回 -1。参数是个输出型参数，内核往里面填子进程的退出信息

如果暂时不关心退出信息，可以直接传 `NULL`：

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>

int main()
{
    pid_t pid = fork();
    if (pid == 0)
    {
        sleep(2);
        exit(7);
    }

    wait(NULL);
    printf("子进程已回收\n");
    return 0;
}
```

这样就够用了：僵尸被清掉，父进程继续往下走。但子进程到底是正常退出还是被杀了、退出码是几，`NULL` 这一传就全丢了

### 5.2 status 的取法

要拿到退出信息，就得传一个 `int` 的地址进去：

```c
int st = 0;
pid_t rid = wait(&st);
```

这里有个容易踩的地方：`st` 拿到的东西不能当整数用。子进程 `exit(10)`，`st` 的值是 **2560**，不是 10

### 5.3 status 的位图结构

`status` 的低 16 位被分成三段，每段装不同的东西：

![[Linux/attachments/status位图.png|status 低 16 位的三段划分：15 到 8 位是退出码，第 7 位是 core dump 标志，6 到 0 位是终止信号；下附 exit(10) 与 kill -9 两个实例]]

图里下面两条是同一张位图的两种填法。`exit(10)` 把 10 填进高 8 位，得到 `0x0A00`；被 `kill -9` 杀掉的那个正好相反，高 8 位是 0，低 7 位填进 9，得到 `0x0009`

一段占着，另一段就一定是空的。进程要么自己退出（填高 8 位），要么被信号终止（填低 7 位），不会同时发生

### 5.4 手工拆位

知道了位段，可以自己拆：

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>

int main()
{
    pid_t pid = fork();
    if (pid == 0)
    {
        sleep(1);
        exit(10);
    }

    int st = 0;
    wait(&st);

    printf("status = 0x%04x (%d)\n", st, st);
    printf("退出码   = %d\n", (st >> 8) & 0xFF);
    printf("终止信号 = %d\n", st & 0x7F);
    printf("core 位  = %d\n", (st >> 7) & 1);
    return 0;
}
```

```text
status = 0x0a00 (2560)
退出码   = 10
终止信号 = 0
core 位  = 0
```

先看低 7 位是不是 0：是 0 说明进程是自己退出的，高 8 位才是退出码。不是 0 说明被信号杀了，那 7 位就是信号编号，高 8 位的「退出码」是垃圾值

被 `SIGSEGV` 杀的情况能同时看到三段：

```text
status = 0x008b, 位7(core) = 1, 信号 = 11
```

`0x008b` 的低 7 位是 `0x0b` = 11，位 7 是 1，说明段错误并且产生了 core dump

### 5.5 两个宏

手工拆位能用，但每处都写 `(st >> 8) & 0xFF` 太啰嗦，而且容易把判断顺序写错。标准库给了两个宏：

```c
WIFEXITED(status)     // 子进程是否正常退出
WEXITSTATUS(status)   // 正常退出时，取出退出码
```

`WIFEXITED` 判的是「低 7 位是不是 0」，也就是「有没有被信号打断」，**不是**「退出码是不是 0」。这个区别很容易记混：

```c
if (WIFEXITED(st))
    printf("退出码: %d\n", WEXITSTATUS(st));
else
    printf("被信号 %d 终止\n", WTERMSIG(st));
```

`exit(1)` 的进程 `WIFEXITED` 依然为真，只是 `WEXITSTATUS` 取出 1

用宏改写前面的判断，逻辑就清楚了：先判正常还是异常，正常才去取退出码。跳过 `WIFEXITED` 直接取 `WEXITSTATUS`，在异常退出的情况下拿到的是垃圾值

## 六、 waitpid 的控制粒度

### 6.1 wait 的两处不足

`wait` 有两个限制：

- 只能等任意一个子进程，没法指定等哪个
- 只能阻塞，等不到就卡在那儿

第一个限制在有多个子进程时会成问题：想等某个特定的孩子，`wait` 做不到。第二个限制更要命：父进程在等的时候什么都干不了

`waitpid` 把这两件事都打开了：

```c
#include <sys/types.h>
#include <sys/wait.h>

pid_t waitpid(pid_t pid, int *status, int options);
```

比 `wait` 多了一个 `pid` 和一个 `options`

### 6.2 pid 的四种取值

| `pid` | 含义 |
| --- | --- |
| `> 0` | 等 `pid` 等于这个值的那个子进程 |
| `== -1` | 等任意一个子进程，与 `wait` 等效 |
| `== 0` | 等同一个进程组里的任意子进程 |
| `< -1` | 等进程组 ID 等于 `\|pid\|` 的任意子进程 |

日常用得最多的是前两种。`-1` 就是 `wait` 的等价写法：

```c
pid_t rid = waitpid(-1, &st, 0);
```

### 6.3 WNOHANG 与三种返回

`options` 传 0 表示阻塞等待，行为和 `wait` 一样。传 `WNOHANG` 就变成非阻塞：有已经退出的子进程就回收，没有就立刻返回，不等

这时候返回值有三种含义：

| 返回值 | 含义 |
| --- | --- |
| `> 0` | 回收到了一个子进程，值是它的 `pid` |
| `== 0` | 设置了 `WNOHANG`，但没有已退出的子进程 |
| `< 0` | 出错，`errno` 被设置 |

三种都要处理。子进程 `sleep(3)` 后 `exit(7)`，实测：

```text
第 1 轮: waitpid = 0
第 2 轮: waitpid = 0
第 3 轮: waitpid = 0
第 4 轮: waitpid = 1861774
收尾: WIFEXITED=1 WEXITSTATUS=7
```

前三轮返回 0，说明子进程还在跑。第四轮拿到 `pid`，收工。如果根本没有子进程：

```text
无子进程 waitpid = -1, errno=10 (No child processes)
```

`-1` 配 `ECHILD`，这是「等错了」而不是「还没等到」。这两者的区别决定了循环该不该继续

### 6.4 阻塞与不阻塞的代价

阻塞等待到底阻塞多久？子进程 `sleep(3)`，父进程 `waitpid(pid, &st, 0)`：

```text
阻塞等待耗时 3.00 s, WEXITSTATUS=5
```

整整 3 秒，父进程一行代码都没执行

非阻塞把这段时间还给了父进程，代价是要自己写循环。这就是为什么 `WNOHANG` 一定要配循环用：单次调用返回 0 不代表失败，只代表「还没好」

### 6.5 等待期间的并行任务

循环体里可以塞自己的活。下面这个例子在等的时候顺手跑两个函数：

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>

void download()
{
    printf("  跑一个下载任务\n");
}

void flush_log()
{
    printf("  刷一次日志\n");
}

int main()
{
    pid_t pid = fork();
    if (pid == 0)
    {
        sleep(3);
        exit(10);
    }

    int st = 0;
    pid_t rid = 0;
    while ((rid = waitpid(pid, &st, WNOHANG)) == 0)
    {
        download();
        flush_log();
        sleep(1);
    }

    if (rid > 0 && WIFEXITED(st))
        printf("子进程退出码 %d\n", WEXITSTATUS(st));
    return 0;
}
```

`while` 的条件直接写 `waitpid(...) == 0`：只要还是 0 就继续循环，一旦变成 `pid` 或 -1 就退出。循环体里跑的是父进程自己的活

这个结构在真实程序里很常见：主循环要定期处理事件、刷新界面，还得盯着子进程。用阻塞等待就做不到，只能轮询

轮询的代价是响应延迟。`sleep(1)` 意味着子进程退出后最多要等 1 秒父进程才发现。把 `sleep` 去掉会变成忙等，CPU 直接跑满。这两个极端之间的取舍，取决于程序对延迟和 CPU 占用的要求

---

## 相关笔记 / 相关链接

- 相关目录：[[目录/Linux目录]]
- 相关笔记：[[Linux/Linux 进程——从 PCB 到僵尸进程]]（前置：`fork`、写时拷贝、僵尸进程的成因，不懂这些「进程等待的必要性」那节读不下去）
