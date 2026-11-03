---
tags:
  - 形态/项目
aliases:
  - 自定义shell尝试一
日期: 2026年10月31日
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

学了Linux一些东西了，我觉得发明shell的人真的是天才，用shell把用户和内核隔离起来，或许可以试试自己来写一个自己的shell

## 一、 程序替换

一个 shell 最基本的动作是「跑一条命令」，而跑命令这件事，内核并没有提供现成的接口

`fork` 之后，子进程执行的是父进程代码的一部分。如果子进程想跑的是磁盘上另一个程序，比如 `ls`，光靠 `fork` 做不到，因为 `fork` 只复制，不换内容。换内容这一步由 `exec` 系列函数完成

### 1.1 换程序不换进程

调用 `exec` 时，发生的事是：当前进程的用户空间代码和数据被完全替换成新程序的，然后从新程序的启动例程开始执行

注意这里没有新进程产生。`exec` 前后 `pid` 不变，进程实体还是那一个，换掉的只是地址空间里的内容。这也是为什么 `exec` 调用成功后**不返回**：函数栈都已经被新程序覆盖了，没有地方可以返回

### 1.2 六个函数的两条命名轴

`exec` 家族有六个函数，名字看起来像乱码，其实拆开看只有两条轴

| 函数 | 参数怎么给 | 要不要写全路径 | 环境表从哪来 |
| --- | --- | --- | --- |
| `execl` | 列表，逐个传 | 要 | 当前环境 |
| `execlp` | 列表 | 不用，自己查 `PATH` | 当前环境 |
| `execle` | 列表 | 要 | 自己传 |
| `execv` | 数组 | 要 | 当前环境 |
| `execvp` | 数组 | 不用，自己查 `PATH` | 当前环境 |
| `execve` | 数组 | 要 | 自己传 |

`l` 和 `v` 决定参数怎么给，`p` 决定要不要自己写全路径，`e` 决定环境表从哪来

这六个里只有 `execve` 是真正的系统调用，其余五个都是库函数，最终都落到它身上。所以 `execve` 的 man 手册在第 2 节，另外五个在第 3 节

### 1.3 成功不返回

因为成功时不返回，`exec` 系列函数**只有出错的返回值，没有成功的返回值**

写代码时这一点会变成一个具体的形状：调用 `execvp` 之后必须再跟一句 `exit`

```cpp
execvp(g_argv[0], g_argv);
exit(1);
```

这两句是子进程那一支要做的事。如果 `execvp` 找到了程序并成功替换，这一行 `exit(1)` 永远走不到。如果它没找到，函数返回 -1，顺着往下执行 `exit(1)`，用一个非 0 的退出码告诉父进程「这条命令没跑起来」

少了这句 `exit`，子进程会在 `execvp` 失败后继续往下跑父进程的代码，结果是同一条命令被执行两遍

至于这两句为什么要放在子进程里、`fork` 之后父子怎么分叉，留到第五章

## 二、 主循环的五步

shell 做的事，其实是重复的

敲一行命令，shell 读进来、拆成词、起一个子进程去跑、等它跑完，然后回到开头等下一行

![[Linux/attachments/shell主循环的五步.png|主循环的五步：打印提示符、读入一行、拆成词、起子进程执行、等待结束，走完回到开头]]

这五步对应四个函数：

| 步骤 | 做什么 | 函数 |
| --- | --- | --- |
| 1 | 打印提示符 | `PrintCommandPrompt` |
| 2 | 读一行 | `GetCommandLine` |
| 3 | 拆成词 | `CommandParse` |
| 4 | 起子进程并替换 | `Execute` |
| 5 | 等子进程结束 | `Execute` 里的 `waitpid` |

## 三、 提示符

提示符长这样：

```text
wfc@VM-0-2-ubuntu:/tmp#
```

三段内容分别是用户名、主机名、当前工作目录。先定格式串，再去取这三段数据

```cpp
#define COMMAND_SIZE 1024
#define FORMAT "[%s@%s %s]# "
```

`FORMAT` 里有三个 `%s`，按顺序对应用户名、主机名、当前目录

```cpp
const char *GetUserName()
{
    const char *name = getenv("USER");
    return name == NULL ? "None" : name;
}
```

用户名从环境变量 `USER` 里取。这样写比直接写死字符串更通用，换一台机器、换一个登录用户，提示符自己就跟着变了

关于主机名，会下意识照抄用户名那套写法，去 `getenv("HOSTNAME")` 取，但在 Ubuntu 上**取不到**

```cpp
const char *GetHostName()
{
    const char *hostname = getenv("HOSTNAME");
    return hostname == NULL ? "None" : hostname;
}
```

`HOSTNAME` 是 bash 自己维护的 shell 变量，**默认没有被导出到环境变量里**。用 `env` 命令查是查不到的，所以在子进程里 `getenv("HOSTNAME")` 返回 `NULL`，提示符就成了 `wfc@None:/tmp#`

要拿到主机名，得走系统调用：

```cpp
const char *GetHostName()
{
    static char hostname[256];
    if (gethostname(hostname, sizeof(hostname)) != 0)
        return "None";
    return hostname;
}
```

`gethostname` ，不依赖任何环境变量，也就没有导出没导出的问题

第三格是当前工作目录。这里也是同样的选择：`getenv("PWD")` 也能取到，但 `PWD` 是 shell 维护的一个字符串，它可能和真实的工作目录不同步。`getcwd` 是系统调用，永远和进程真实的 `cwd` 一致

```cpp
const char *GetPwd()
{
    static char cwd[COMMAND_SIZE];
    const char *pwd = getcwd(cwd, sizeof(cwd));
    return pwd == NULL ? "None" : pwd;
}
```

拼成一行：

```cpp
void MakeCommandLine(char cmd_prompt[], int size)
{
    snprintf(cmd_prompt, size, FORMAT, GetUserName(), GetHostName(), GetPwd());
}
```

打印完记得 `fflush(stdout)`。提示符末尾没有换行，如果 stdout 是行缓冲，它不会自己刷出去，光标会停在上一次的输出后面，看起来像卡住了

```cpp
void PrintCommandPrompt()
{
    char prompt[COMMAND_SIZE];
    MakeCommandLine(prompt, sizeof(prompt));
    printf("%s", prompt);
    fflush(stdout);
}
```

## 四、 读一行与拆词

### 4.1 读一行

`fgets` 从 `stdin` 读一行，最多读 `size - 1` 个字符，末尾自动补 `\0`。它会把换行符也一起读进来，所以读完要把最后一个字符改成 `\0`

```cpp
bool GetCommandLine(char *out, int size)
{
    char *c = fgets(out, size, stdin);
    if (c == NULL)
        return false;
    out[strlen(out) - 1] = 0;
    if (strlen(out) == 0)
        return false;
    return true;
}
```

这里有两个返回值要处理

第一个是 `fgets` 返回 `NULL`，表示读失败了。最常见的情况是读到文件末尾：按 `Ctrl + D`，或者标准输入被重定向到一个文件、文件读完了

第二个是读完发现是空行，也就是用户直接敲了个回车。空行没有命令可执行，也应该返回 `false`

这两个都返回 `false`，`main` 里对应的是 `continue`：不执行命令，直接回到循环开头重新打提示符

注意 `out[strlen(out) - 1] = 0` 。它假设读进来的内容一定以 `\n` 结尾。输入是终端的时候这个假设成立，但如果输入被重定向，最后一行可能没有换行符，会把最后一个有效字符误删掉

### 4.2 拆词

一行命令 `ls -a -l` 要变成三个独立的字符串 `"ls"`、`"-a"`、`"-l"`，因为 `execvp` 的第二个参数要的正是这样一个字符串数组

`strtok` 可以做到。它的用法有个特点：第一次调用要传字符串首地址，之后每次传 `NULL`

```cpp
bool CommandParse(char *commandline)
{
#define SEP " "
    g_argc = 0;
    g_argv[g_argc++] = strtok(commandline, SEP);
    while ((bool)(g_argv[g_argc++] = strtok(nullptr, SEP))
        ;
    g_argc--;
    return g_argc > 0 ? true : false;
}
```

传 `NULL` 的意思是「接着上一次的位置继续切」。`strtok` 内部记着一个静态指针，指向上一次切到的位置，所以它不需要每次重新告诉它从哪开始

`g_argv` 是全局的字符串指针数组，`g_argc` 是词的数量

```cpp
#define MAXARGV 128
char *g_argv[MAXARGV];
int g_argc = 0;
```

需要注意的是`while` 的条件里 `g_argc++` 会先自增再判断。最后一次 `strtok` 返回 `NULL`，被存进 `g_argv` 并让 `g_argc` 加了一，所以循环结束后 `g_argc` 比真实词数多 1，最后那句 `g_argc--` 把这个多加的减回去

同时 `g_argv[g_argc]` 那里留着 `NULL`，正好是 `execvp` 要求的数组结尾标记

这样写有两个边界可以不用管。一是 `g_argc` 恒大于 0，因为空输入时 `g_argv[0]` 已经存了 `NULL`、`g_argc` 也已经是 1，所以 `return g_argc > 0` 永远返回真，空命令的边界已经考虑到了。二是 `MAXARGV` 只有 128，参数特别多的命令会写穿数组，循环里没有边界检查，当然只是尝试一下自定义的shell，不必要求太高

## 五、 执行

到这一步，参数表已经齐了，剩下的就是fork一个子进程、让它在里面跑命令、父进程等它结束

![[Linux/attachments/fork-exec-wait时序.png|shell 与子进程两条生命线：fork 之后子进程 execvp 替换自己（进程号不变），父进程 waitpid 阻塞，子进程 exit 后拿到 status]]

```cpp
int Execute()
{
    pid_t id = fork();
    if (id == 0)
    {
        execvp(g_argv[0], g_argv);
        exit(1);
    }
    int status = 0;
    pid_t rid = waitpid(id, &status, 0);
    (void)rid;
    return 0;
}
```

`fork` 一次调用，两个返回值

- 在子进程里返回 0
- 在父进程里返回子进程的 `pid`
- 出错时返回 -1

`id == 0` 这个分支只有子进程会进去。子进程进去之后立刻调 `execvp`，把自己换成 `ls` 或者别的命令

父进程走的是 `fork` 之后的下一行，也就是 `waitpid`。第三个参数传 0 表示阻塞等待，父进程会停在这里，直到子进程结束

`waitpid` 的返回值是子进程的 `pid`，第二个参数 `status` 是输出参数，子进程的退出信息由内核填进去

## 六、 尝试一下

编译出来跑一下

```bash
g++ -std=c++11 -o myshell myshell.cc
```

```text
wfc@VM-0-2-ubuntu:/tmp/probe-myshell# ls -a
.  ..  myshell  myshell.cc
wfc@VM-0-2-ubuntu:/tmp/probe-myshell# pwd
/tmp/probe-myshell
wfc@VM-0-2-ubuntu:/tmp/probe-myshell# echo hello
hello
```

`ls`、`pwd`、`cat` 这些都能跑。原因是它们都是独立的可执行文件，`fork` 出来的子进程调 `execvp` 就能把它们装进来

但有一条命令不行：

```text
wfc@VM-0-2-ubuntu:/tmp/probe-myshell# cd /
wfc@VM-0-2-ubuntu:/tmp/probe-myshell# pwd
/tmp/probe-myshell
```

`cd /` 敲下去没报错，也没换目录。提示符里的路径没变，再敲 `pwd` 也还是老地方

问题出在 `cd` 压根不是一个可执行文件。磁盘上没有叫 `cd` 的程序，`execvp` 找不到它，返回 -1，子进程顺着执行到 `exit(1)`

所以这一版里 `cd` 的真实表现是「什么都没发生」：既没有换目录，也没有报错

要让 `cd` 生效，它必须在 shell 自己的进程里执行。这类必须在 shell 里执行的命令，就叫内建命令

---

## 相关笔记 / 相关链接

- 相关目录：[[Linux目录]]
- 相关笔记：[[Linux/Linux 命令行参数、环境变量与进程地址空间]]（前置：不懂 `argv` 与环境表，本篇 `execvp` 传参与提示符取环境变量那两段读不懂）
