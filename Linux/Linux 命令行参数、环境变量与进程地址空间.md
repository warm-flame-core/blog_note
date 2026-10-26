---
tags:
  - 形态/概念
  - 主题/内存管理
aliases:
  - Linux 环境变量与地址空间
日期: 2026年10月25日
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

在终端里敲 `ls -l /tmp`，`ls` 知道要列 `/tmp`、要长格式；再敲 `env`，屏幕刷出几十行 `KEY=VALUE`。这两样东西都不在 `ls` 的二进制里：它的代码里没有 `/tmp` 这个字符串，也没有 `PATH` 这个变量

地址也一样。`fork` 之后父子进程打印同一个全局变量的地址，地址一模一样，值却不一样

参数、环境变量、地址，看着是三件事，落到内存里其实是同一块地方

## 一、 main 的三个参数

写 C 程序时 main 一般写成 `int main(void)`，其实在 Linux 下它能带三个参数

```c
int main(int argc, char *argv[], char *env[])
```

`argc` 是参数个数，`argv` 是参数字符串数组，第三个 `env` 是环境变量表。标准 C 只保证前两个，第三个是 Linux 这边多给的。写成 `void` 也能跑，是因为启动代码允许你这么声明，不代表没有东西传进来

`argc` 把程序自己的名字也算一个。程序里把 `argc` 和 `argv` 打印出来，带上四个参数跑：

```c
printf("argc = %d\n", argc);
for (int i = 0; i < argc; i++)
    printf("argv[%d] = %-8s 地址 %p\n", i, argv[i], (void *)argv[i]);
```

输出：

```text
argc = 5
argv[0] = ./argv_env 地址 0x7ffdf00dadd6
argv[1] = a          地址 0x7ffdf00dade1
argv[2] = b          地址 0x7ffdf00dade3
argv[3] = c          地址 0x7ffdf00dade5
argv[4] = d          地址 0x7ffdf00dade7
```

五个参数，第一个是程序自己的名字，后面四个才是敲进去的

调用 main 的不是内核。内核把程序加载起来、跳到入口 `_start`，中间那层启动代码做完 libc 初始化、把环境表备好，最后把三个值交给 `main`：

```c
ret = main(argc, argv, env);
```

所以 main 的三个参数不是内核直接塞的，是启动代码按约定传的

命令行参数最常用的地方是让一个程序走不同分支。比如带 `-a`、`-b`、`-c` 分别干三件事：

```c
if (argc != 2)
{
    printf("Usage: %s [-a|-b|-c]\n", argv[0]);
    return 1;
}

if (strcmp(argv[1], "-a") == 0)
    printf("功能 1\n");
else if (strcmp(argv[1], "-b") == 0)
    printf("功能 2\n");
else
    printf("Usage: %s [-a|-b|-c]\n", argv[0]);
```

参数不对就打印用法，参数对了就进对应分支，`ls`、`gcc` 这些命令都是这么做的

## 二、 参数与环境变量的传入

shell 收到一行命令，先按空格切开，再 fork 一个子进程，子进程用 `execve` 把新程序换进来，切开的那些词作为 `argv` 一起交过去。环境变量不在命令行里，是 shell 自己维护的一张表，`execve` 的时候一并交出

内核接到 `execve` 后，给新程序建一个新的 `mm_struct`，在里面放一个临时栈，然后把 `argv`、`envp` 里的字符串一个个拷到这个新栈顶上。拷是从栈顶往下走的，所以字符串在内存里是倒着落下来的

拷完再摆指针，栈的最低处放 `argc`，往上依次是 `argv` 指针数组、一个 `NULL`、`envp` 指针数组、一个 `NULL`，再往上才是那些字符串

```
        +--------------------+
        |  strings           |   参数字符串与环境字符串
        +--------------------+
        |  0                 |
        |  envp[]            |   指向环境字符串
        |  0                 |
        |  argv[]            |   指向参数字符串
        |  argc = 5          |
        +--------------------+
        低地址  <- 栈顶
```

`mm_struct` 里有四个字段专门记这几段的边界：`arg_start`、`arg_end`、`env_start`、`env_end`。`arg_start` 指向第一个参数字符串，`arg_end` 和 `env_start` 是同一个位置（参数区结束、环境区开始），`env_end` 指向环境区末尾

## 三、 环境变量的概念

环境变量是操作系统用来指定运行环境的一些参数。它不属于某个程序，属于这个 shell 会话，登录进来就有，之后启动的每个程序都能看到

常见的几个：

| 变量 | 作用 |
| --- | --- |
| `PATH` | 命令的搜索路径 |
| `HOME` | 用户的主目录 |
| `SHELL` | 当前用的 shell，一般是 `/bin/bash` |
| `USER` | 当前用户名 |
| `PWD` | 当前所在目录 |

查看某个变量的值用 `echo`：

```bash
echo $HOME
```

想看全部就用 `env`，输出是几十行 `KEY=VALUE`：

```text
SHELL=/bin/bash
PWD=/tmp
LOGNAME=wfc
HOME=/home/wfc
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

这些行在内存里排成一张表。环境表是一个字符指针数组，每个指针指向一个以 `\0` 结尾的 `KEY=VALUE` 字符串，数组最后用一个 `NULL` 收尾：

```
        +------------------+        +----------------------+
        |  env[0]          |  --->  | SHELL=/bin/bash\0    |
        +------------------+        +----------------------+
        |  env[1]          |  --->  | PATH=/usr/local/...  |
        +------------------+        +----------------------+
        |  env[2]          |  --->  | HOME=/home/wfc\0     |
        +------------------+        +----------------------+
        |  NULL            |
        +------------------+
```

`NULL` 收尾是这套结构的通用做法，`argv` 也一样，遍历的时候不用额外传个数，走到 `NULL` 就停

## 四、 PATH 与命令查找

`ls` 敲下去就能跑，自己写的程序却要 `./a.out`。区别在于 `ls` 放在 `PATH` 列的某个目录里，而 `./a.out` 在哪儿 `PATH` 不知道

验证一下。把程序编到一个目录里，比如 `/tmp/probe/bin`，直接敲程序名：

```text
$ hello
bash: hello: command not found
```

退出码是 127。把程序所在目录拼进 `PATH` 再敲：

```bash
export PATH=$PATH:/tmp/probe/bin
```

```text
$ hello
hello world!
```

`export PATH=...` 只对当前这个 shell 有效，关掉就没了。想长期生效得写进 `~/.bashrc` 之类的文件

`./a.out` 前面的 `./` 是路径的一部分，意思是当前目录下的 `a.out`，不是命令名。`/bin/ls`、`../test` 同理，带了斜杠就按路径找，不带斜杠才去 `PATH` 里找

## 五、 三种取环境变量的方式

程序里拿环境变量有三种写法

第一种是 main 的第三个参数：

```c
int main(int argc, char *argv[], char *env[])
{
    for (int i = 0; env[i]; i++)
        printf("%s\n", env[i]);
    return 0;
}
```

第二种是 `libc` 里的全局变量 `environ`，它指向同一张环境表，用之前要 `extern` 声明：

```c
extern char **environ;

int main(void)
{
    for (int i = 0; environ[i]; i++)
        printf("%s\n", environ[i]);
    return 0;
}
```

第三种是 `getenv`，按名字取一个值，最常用：

```c
printf("PATH = %s\n", getenv("PATH"));
```

前两种拿到的是整张表，`getenv` 只取一个。三种方式在内存里是同一张表，打印地址能看到 `env` 和 `environ` 是同一个值：

```text
env=0x7ffdf00d9ba8 environ=0x7ffdf00d9ba8 相同? 是
getenv(PATH) 前 40 字符 = /usr/local/sbin:/usr/local/bin:/usr/sbin
```

## 六、 本地变量与环境变量

shell 里的变量分两种。直接赋值的是本地变量，只在当前 shell 里有效：

```bash
i=10
echo $i
sh -c 'echo $i'
```

前一行输出 `10`，后一行输出空，因为 `sh -c` 起的是子进程，看不到父 shell 的本地变量

加 `export` 就不一样了：

```bash
export i
sh -c 'echo $i'
```

这次输出 `10`。`export` 把变量从本地变量变成环境变量，环境变量会跟着 `fork` + `execve` 传给子进程

环境变量是自顶向下继承的。导出一个自己的变量，再让子进程去取：

```bash
export MYENV="hello world"
./getenv_demo
```

```text
getenv(MYENV) = hello world
```

不 `export`，同样的程序取到的是 `NULL`：

```bash
MYENV2="hi"
./getenv_demo
```

```text
getenv(MYENV2) = (NULL)
```

`cd`、`echo`、`export` 这些命令为什么不用 fork 子进程？因为它们是 shell 的内建命令，shell 自己调个函数就做完了，不需要另起进程。像 `ls`、`gcc` 这种才是外部命令，得 fork + `execve`

那环境变量最初从哪来？登录的时候 shell 会读几个文件：`/etc/profile`、`~/.bash_profile`、`~/.bashrc`，里面的 `export` 一条条执行，环境表就这么建起来了

## 七、 程序地址空间

学 C 语言时画过一张内存布局图，从高地址到低地址大致是：命令行参数与环境变量、栈、共享库、堆、BSS、数据段、代码段

```
        +-------------+   高地址
        |  argv/env   |   命令行参数与环境变量
        +-------------+
        |  stack      |   栈，向下增长
        +-------------+
        |     |       |
        |     v       |
        +-------------+
        |  shared lib |   共享库
        +-------------+
        |     ^       |
        |     |       |
        +-------------+
        |  heap       |   堆，向上增长
        +-------------+
        |  bss        |   未初始化数据
        +-------------+
        |  data       |   已初始化数据
        +-------------+
        |  text       |   代码段
        +-------------+   低地址
```

把各类变量的地址打印出来，和图上对得上：

```text
code    0x5e6db4ecf189
rodata  0x5e6db4ed0004
data    0x5e6db4ed2010
bss     0x5e6db4ed201c
static  0x5e6db4ed2014
heap    0x5e6dc299a2a0 0x5e6dc299a2c0
stack   0x7ffd19ffe290
argv    0x7ffd19ffe3d8
env     0x7ffd19ffe3e8
```

顺序和图上一致：代码最低，只读串紧跟，然后是已初始化数据、`bss`，堆往上走，栈在最高的 `0x7ffd...` 一带，命令行参数和环境变量还在栈的上面。`static` 变量在 `0x5e6db4ed2014`，正好夹在已初始化和未初始化之间，它俩本来就挨着

具体数值随机器和编译器变，顺序不变

## 八、 fork 之后同址不同值

`fork` 之后父子进程打印同一个全局变量，地址一模一样，值却不一样

```c
int g_val = 100;

int main(void)
{
    pid_t id = fork();
    if (id == 0)
    {
        g_val = 200;
        printf("child  g_val=%d &g_val=%p\n", g_val, (void *)&g_val);
    }
    else
    {
        sleep(1);
        printf("parent g_val=%d &g_val=%p\n", g_val, (void *)&g_val);
    }
    return 0;
}
```

```text
child  g_val=200 &g_val=0x5c28b525e010
parent g_val=100 &g_val=0x5c28b525e010
```

同一个地址，两份不同的值。如果这个地址是物理地址，那父子就是同一块内存，子进程改了父进程不可能看不见。所以打印出来的地址不是物理地址

它是虚拟地址。C、C++ 里能拿到的地址全是虚拟地址，物理地址用户看不到，由操作系统管着

父子能各有一份值，靠的是写时拷贝：`fork` 时父子共享同一批只读的页，谁要写就单独拷一份出来。改的是拷贝出来的新页，原来的页没动，所以父进程读到的还是 100

## 九、 虚拟地址与页表

虚拟地址要落到物理内存上，中间隔着一张页表。页表把虚拟地址的页号翻译成物理页号，CPU 访存时先查页表再拿数据

每个进程有一套自己的页表。所以两个进程可以有相同的虚拟地址，映射到不同的物理页，互不干扰

页表项里还带权限位：可读、可写、可执行。进程地址空间里代码段是只读可执行的，数据段是可读可写的。往字符串常量写会崩，原因就在这儿：那一页在页表里标的是只读，写操作被拦下来了

![[Linux/attachments/虚拟地址到物理页的映射.png|虚拟地址经页表映射到物理页]]

注意：页表不是程序自己建的，是操作系统在进程创建、加载程序、`malloc` 的时候建的，程序完全感觉不到

## 十、 mm_struct 与 VMA

内核用一个结构体描述整个进程地址空间：`mm_struct`，叫内存描述符。每个进程一个，挂在 `task_struct` 里

```c
struct task_struct
{
    /* ... */
    struct mm_struct *mm;         /* 用户进程的地址空间 */
    struct mm_struct *active_mm;  /* 内核线程借用别人的地址空间 */
    /* ... */
};
```

普通用户进程用 `mm`。内核线程没有自己的用户空间，`mm` 是 `NULL`，就借一个 `active_mm` 用，因为所有进程的内核部分映射是一样的

`mm_struct` 里记着地址空间的全部信息，下面是内核 2.6.32 里几个要紧的字段：

```c
struct mm_struct
{
    struct vm_area_struct *mmap;   /* VMA 链表 */
    struct rb_root mm_rb;          /* VMA 红黑树 */
    pgd_t *pgd;                    /* 页表基址 */
    int map_count;                 /* VMA 个数 */

    unsigned long start_code, end_code, start_data, end_data;
    unsigned long start_brk, brk, start_stack;
    unsigned long arg_start, arg_end, env_start, env_end;
};
```

后三行就是各段的边界：代码段、数据段、堆（`start_brk` 到 `brk`）、栈，还有前面说的参数区和环境区

地址空间不是一个整块，是分成一段一段的，每段叫一个 VMA（虚拟内存区域），用 `vm_area_struct` 描述：

```c
struct vm_area_struct
{
    unsigned long vm_start;    /* 起始地址 */
    unsigned long vm_end;      /* 结束地址 */
    struct vm_area_struct *vm_next;
    pgprot_t vm_page_prot;     /* 权限 */
    unsigned long vm_flags;
    struct file *vm_file;      /* 映射的文件，匿名为 NULL */
};
```

分成段是因为每段的权限和归属不同：代码段来自可执行文件、只读可执行，数据段来自文件、可读可写，堆和栈是匿名内存

VMA 多了要管得过来。内核用两种结构：数量少的时候用单链表，`mmap` 指向链表头；数量多的时候用红黑树，`mm_rb` 指向树根。还留了个 `mmap_cache` 记住上次查到的 VMA，下次接着用

![[Linux/attachments/mm_struct与VMA.png|task_struct、mm_struct 与 VMA 的关系]]

这些结构在 `/proc/<pid>/maps` 里能直接看到：

```text
00400000-00401000 r--p 00000000 ... /tmp/probe/maps
00401000-00402000 r-xp 00001000 ... /tmp/probe/maps
00402000-00403000 r--p 00002000 ... /tmp/probe/maps
00403000-00404000 r--p 00002000 ... /tmp/probe/maps
00404000-00405000 rw-p 00003000 ... /tmp/probe/maps
33273000-33294000 rw-p 00000000 00:00 0   [heap]
7b0056c00000-7b0056c28000 r--p ... /usr/lib/x86_64-linux-gnu/libc.so.6
7fff32ad4000-7fff32af5000 rw-p 00000000 00:00 0   [stack]
7fff32b63000-7fff32b65000 r-xp 00000000 00:00 0   [vdso]
```

一行就是一个 VMA，`vm_start-vm_end` 加权限位。同一个可执行文件被拆成五段：`r--p` 一段放 ELF 头和程序头，`r-xp` 是代码，再一段 `r--p` 是只读数据，又一段 `r--p` 放异常处理和重定位用的只读表，最后 `rw-p` 是数据段加 `bss`。堆和栈没有文件名，标成 `[heap]`、`[stack]`，是匿名映射

## 十一、 虚拟地址空间的必要性

如果程序直接用物理地址，会有三个问题

第一个是安全。任何进程都能读写任意内存，一个木马改掉系统关键数据，整台机器就完了

第二个是地址不确定。程序加载到内存的哪个位置，取决于当时内存用到哪儿了，同一个程序两次运行的地址可能完全不同，编译时没法把地址写死

第三个是效率。内存不够时要把不用的进程换到磁盘，按物理地址管理就得整块搬走，太慢

有了虚拟地址空间和页表，这三个问题都能解决：

- 每个进程一套页表，只能看到自己的地址空间，碰不到别人的
- 物理内存可以随便找空位加载，虚拟地址是连续的，进程视角里的内存始终有序
- 换出按页做，不用整块搬

顺带还多出两个好处：`malloc` 时只在地址空间里划一块，真正访问那一页才分配物理内存，叫延迟分配；页表项带权限，越权访问直接被拦

进程管理和内存管理也因此解耦了：进程只管自己那张虚拟地址空间，物理内存怎么分配由内存管理模块负责，两边不用互相知道

---

## 相关笔记 / 相关链接

- 相关目录：[[目录/Linux目录]]
- 相关笔记：[[Linux/Linux 进程——从 PCB 到僵尸进程]]（前置：fork、写时拷贝、进程概念）
