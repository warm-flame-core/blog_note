---
tags:
  - 形态/项目
aliases:
  - 自定义shell尝试二
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

自己的shell已经有雏形了，但是最重要的cd命令还是没有完成，内建命令很重要，这样的shell才能使用

上一版的 shell 已经能跑 `ls`、`pwd`、`cat` 这些命令：它读一行、拆成词、`fork` 一个子进程、在子进程里 `execvp` 装进目标程序、父进程 `waitpid` 等它结束。但是 `cd` 这种内建命令还做不到

## 一、 cd 不能交给子进程

`cd` 之所以没反应，是因为它被当成普通命令丢给了子进程，而磁盘上根本没有叫 `cd` 的可执行文件。`execvp("cd")` 找不到它，返回 -1，子进程走到 `exit(1)`，然后什么都没发生

真正执行 `cd` 的是 `chdir` 这个函数。要让 `cd` 生效，就得让 shell 自己去调 `chdir`

原因在 `chdir` 改的是谁的工作目录

工作目录是进程自己的一份属性，记录在 `task_struct` 里。`chdir` 改的永远是**调用它的那个进程**的工作目录

如果真有一个可执行文件叫 `cd`，把它交给子进程去跑，子进程调 `chdir` 改的也是它自己。子进程跑完就退出，它的 `task_struct` 连同工作目录一起被回收，父进程也就是 shell 从头到尾没动过

所以这种事根本不能靠 `exec` 办到。要改 shell 的工作目录，只能由 shell 自己调 `chdir`

要让 `cd` 生效，它必须由 shell 自己执行。这类命令就叫内建命令

外部命令要 fork 一个子进程，命令的副作用留在子进程里；内建命令直接在 shell 进程里跑，副作用留在 shell 里

![[Linux/attachments/内建命令与外部分流.png|内建命令与外部分流：是内建则在 shell 进程里执行，否则 fork 一个子进程]]

## 二、 分流

有了这个认识，主循环里要多一步：在 fork 之前先问一句「这条命令是不是内建的」

```cpp
bool CheckAndExecBuiltin()
{
    std::string cmd = g_argv[0];
    if (cmd == "cd")
    {
        // ...
        return true;
    }
    return false;
}
```

返回 `true` 表示这条命令已经处理完了

`main` 里对应的是：

```cpp
if (CheckAndExecBuiltin())
{
    continue;
}
Execute();
```

`continue` 直接跳回循环开头，`Execute` 就不会被调到。这一步的位置必须在 fork **之前**，否则子进程已经起来了，会拦不住

## 三、 cd 的实现

`cd` 有两种方法：不带参数和带参数

```cpp
bool CheckAndExecBuiltin()
{
    std::string cmd = g_argv[0];
    if (cmd == "cd")
    {
        if (g_argc == 1)
        {
            std::string home = GetHome();
            if (home.empty())
                return true;
            chdir(home.c_str());
        }
        else
        {
            std::string where = g_argv[1];
            chdir(where.c_str());
        }
        return true;
    }
    return false;
}
```

`cd` 后面没跟东西时，按惯例应该回用户的家目录。家目录从环境变量 `HOME` 里取

```cpp
const char *GetHome()
{
    const char *home = getenv("HOME");
    return home == NULL ? "" : home;
}
```

取不到 `HOME` 就直接返回，不 `chdir`。这样写比 `chdir("")` 稳妥，后者会失败而且失败得莫名其妙

带参数时直接把 `g_argv[1]` 交给 `chdir`。`chdir` 自己能处理绝对路径和相对路径，不用自己判断

## 四、 提示符里的路径

`cd` 能用了，但马上会发现一个新的问题

```text
wfc@VM-0-2-ubuntu:/tmp# cd /
wfc@VM-0-2-ubuntu:/tmp# pwd
/
```

`pwd` 已经返回 `/`，说明目录真的换了。但提示符还写着 `/tmp`，它没有跟着变

一开始的写法的提示符是用 `getenv("PWD")` 拼出来的。`PWD` 是 shell 维护的一个环境变量，它只在 shell 主动更新的时候才变。`chdir` 改的是进程的工作目录，它不认识 `PWD` 这个环境变量，也就不会去改它

所以 `chdir` 之后，真实的工作目录变了，`PWD` 里那个字符串还是旧的

可以把提示符改成用 `getcwd` 直接问内核，这样提示符永远是准的，因为它不看 `PWD`：

```cpp
const char *GetPwd()
{
    static char cwd[COMMAND_SIZE];
    const char *pwd = getcwd(cwd, sizeof(cwd));
    return pwd == NULL ? "None" : pwd;
}
```

也可以把 `PWD` 也一起更新，让它和真实目录同步。这一条更麻烦，但它关系到后面的东西：`PWD` 不只是给提示符看的，子进程通过 `exec` 继承环境表时也会拿到它。很多程序会读 `PWD` 来判断自己在哪

我认为第二种更好，把前一种情况也包含进来了，路径照样由 `getcwd` 取，取到之后顺手把 `PWD` 写回环境表

```cpp
const char *GetPwd()
{
    static char cwd[COMMAND_SIZE];
    static char cwdenv[COMMAND_SIZE];
    const char *pwd = getcwd(cwd, sizeof(cwd));
    if (pwd != NULL)
    {
        snprintf(cwdenv, sizeof(cwdenv), "PWD=%s", cwd);
        putenv(cwdenv);
    }
    return pwd == NULL ? "None" : pwd;
}
```

`putenv` 把一个 `KEY=VALUE` 形式的字符串塞进环境表。传进去的字符串**必须活得比调用长**，因为 `putenv` 不会复制它，只是把指针记下来。所以 `cwdenv` 得是 `static` 的，不能是局部变量

如果 `cwdenv` 是个普通局部数组，函数一返回它就被销毁了，环境表里留下一个悬空指针。之后任何程序去读 `PWD` 都会读到垃圾

## 五、 $? 与退出码

`cd` 完了，但是我们还不知道一个程序运行的结果

真实 shell 里敲 `echo $?` 是能看到的。这个值来自 `waitpid` 的 `status` 参数

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
    if (rid > 0)
    {
        lastcode = WEXITSTATUS(status);
    }
    return 0;
}
```

`status` 不是直接能用的整数，是内核按位拼起来的一张位图。要拿退出码得先过 `WIFEXITED`，确认子进程是正常退出而不是被信号打断，再用 `WEXITSTATUS` 把高 8 位取出来

这里省掉了 `WIFEXITED` 的判断，直接取 `WEXITSTATUS`。子进程正常退出时没问题，但被信号打断时取到的是垃圾值

`lastcode` 是全局变量，`echo $?` 会读它

```cpp
int lastcode = 0;
```

`echo` 也是内建命令，因为它要读 shell 自己的变量，交给子进程读不到

```cpp
void Echo()
{
    if (g_argc == 2)
    {
        std::string opt = g_argv[1];
        if (opt == "$?")
        {
            std::cout << lastcode << std::endl;
            lastcode = 0;
        }
        else if (opt[0] == '$')
        {
            std::string env_name = opt.substr(1);
            const char *env_value = getenv(env_name.c_str());
            if (env_value)
                std::cout << env_value << std::endl;
        }
        else
        {
            std::cout << opt << std::endl;
        }
    }
}
```

`$?` 是个特例，它不对应环境变量，对应的是 `lastcode` 这个全局变量。其余以 `$` 开头的参数按环境变量查

`echo $?` 之后把 `lastcode` 清零，这条清零其实有点多余。真实 shell 里 `echo $?` 之后再敲 `echo $?` 会得到 0，原因是 `echo` 这条命令自己执行成功了，它的退出码是 0，于是新的 `$?` 就是 0，而不是因为「读完就重置」这条规则

## 六、 shell 自己的环境表

到这里 `cd`、`$?` 都能用了。接下来是 `export` 和 `env`，它们要求 shell 自己维护一张环境表

为什么不能直接用 `environ`？因为 `export` 要往环境里加东西，而 `environ` 是父进程传下来的，直接改它不安全。更实际的原因是，`export` 加进去的变量得先落在 shell 自己手里，之后 shell 才能决定把它传给哪些子进程

启动时先拷一份：

```cpp
#define MAX_ENVS 100
char *g_env[MAX_ENVS];
int g_envs = 0;

void InitEnv()
{
    extern char **environ;
    for (int i = 0; environ[i]; i++)
    {
        g_env[i] = (char *)malloc(strlen(environ[i]) + 1);
        strcpy(g_env[i], environ[i]);
        g_envs++;
    }
    g_env[g_envs] = NULL;
}
```

每个环境字符串都要单独 `malloc` 一块内存再 `strcpy`。不能直接存 `environ[i]` 那个指针，因为那张表属于父进程，随时可能被回收

`export` 往里加一条：

```cpp
void AddEnv(const char *item)
{
    int index = 0;
    while (g_env[index])
    {
        index++;
    }
    g_env[index] = (char *)malloc(strlen(item) + 1);
    strncpy(g_env[index], item, strlen(item) + 1);
    g_env[index + 1] = NULL;
}
```

先找到表尾，在末尾追加，再把新的 `NULL` 补上

有了这张表，`env` 命令就是把整张表打印一遍：

```cpp
void Env()
{
    for (int i = 0; g_env[i]; i++)
    {
        printf("%s\n", g_env[i]);
    }
}
```

`export` 和 `env` 也走内建命令那一套分流，判断逻辑和 `cd` 一样，这里不重复贴。真正用到这张表的地方是执行外部命令时的 `exec`：把上一版 `Execute` 里的 `execvp` 换成 `execvpe`，环境表这一格就换成了 shell 自己的

```cpp
execvpe(g_argv[0], g_argv, g_env);
```

`execvpe` 的 `e` 表示环境表由调用者给，`p` 表示 `PATH` 查找由它来做。这个函数是 GNU 扩展，标准里没有，用它得在编译时定义 `_GNU_SOURCE`

它和 `execvp` 的差别正在这里：`execvp` 传环境表时用的是当前进程的环境，`execvpe` 允许你指定一份。用 `execvpe` 之后，`export` 加进去的变量才会被子进程看到；`execvp` 用的是 `environ`，那张表里没有 `export` 加的东西

所以「`export` 的变量只属于这个 shell」这个说法要改一下：它先落在 shell 自己的表里，再由 shell 决定传给哪个子进程。`export` 出去的东西，子进程当然看得见

## 七、 完整源码

这一版 shell 的全部代码

```cpp
#include <cstdio>
#include <cstdlib>
#include <cstring>
#include <iostream>
#include <string>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>

#define COMMAND_SIZE 1024
#define FORMAT "[%s@%s %s]# "
#define MAXARGV 128
#define MAX_ENVS 100

// 命令行参数表
char *g_argv[MAXARGV];
int g_argc = 0;

// shell 自己的环境表
char *g_env[MAX_ENVS];
int g_envs = 0;

// 上一条命令的退出码
int lastcode = 0;

const char *GetUserName()
{
    const char *name = getenv("USER");
    return name == NULL ? "None" : name;
}

const char *GetHostName()
{
    static char hostname[256];
    if (gethostname(hostname, sizeof(hostname)) != 0)
        return "None";
    return hostname;
}

const char *GetPwd()
{
    static char cwd[COMMAND_SIZE];
    static char cwdenv[COMMAND_SIZE];
    const char *pwd = getcwd(cwd, sizeof(cwd));
    if (pwd != NULL)
    {
        snprintf(cwdenv, sizeof(cwdenv), "PWD=%s", cwd);
        putenv(cwdenv);
    }
    return pwd == NULL ? "None" : pwd;
}

const char *GetHome()
{
    const char *home = getenv("HOME");
    return home == NULL ? "" : home;
}

void MakeCommandLine(char cmd_prompt[], int size)
{
    snprintf(cmd_prompt, size, FORMAT, GetUserName(), GetHostName(), GetPwd());
}

void PrintCommandPrompt()
{
    char prompt[COMMAND_SIZE];
    MakeCommandLine(prompt, sizeof(prompt));
    printf("%s", prompt);
    fflush(stdout);
}

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

bool CommandParse(char *commandline)
{
#define SEP " "
    g_argc = 0;
    g_argv[g_argc++] = strtok(commandline, SEP);
    while ((bool)(g_argv[g_argc++] = strtok(nullptr, SEP)))
        ;
    g_argc--;
    return g_argc > 0 ? true : false;
}

void InitEnv()
{
    extern char **environ;
    for (int i = 0; environ[i]; i++)
    {
        g_env[i] = (char *)malloc(strlen(environ[i]) + 1);
        strcpy(g_env[i], environ[i]);
        g_envs++;
    }
    g_env[g_envs] = NULL;
}

void AddEnv(const char *item)
{
    int index = 0;
    while (g_env[index])
    {
        index++;
    }
    g_env[index] = (char *)malloc(strlen(item) + 1);
    strncpy(g_env[index], item, strlen(item) + 1);
    g_env[index + 1] = NULL;
}

void Cd()
{
    if (g_argc == 1)
    {
        std::string home = GetHome();
        if (home.empty())
            return;
        chdir(home.c_str());
    }
    else
    {
        std::string where = g_argv[1];
        chdir(where.c_str());
    }
}

void Echo()
{
    if (g_argc != 2)
        return;
    std::string opt = g_argv[1];
    if (opt == "$?")
    {
        printf("%d\n", lastcode);
    }
    else if (opt[0] == '$')
    {
        std::string env_name = opt.substr(1);
        const char *env_value = getenv(env_name.c_str());
        if (env_value)
            printf("%s\n", env_value);
    }
    else
    {
        printf("%s\n", opt.c_str());
    }
}

void Env()
{
    for (int i = 0; g_env[i]; i++)
    {
        printf("%s\n", g_env[i]);
    }
}

bool CheckAndExecBuiltin()
{
    std::string cmd = g_argv[0];
    if (cmd == "cd")
    {
        Cd();
        return true;
    }
    if (cmd == "echo")
    {
        Echo();
        return true;
    }
    if (cmd == "export")
    {
        if (g_argc == 2)
            AddEnv(g_argv[1]);
        return true;
    }
    if (cmd == "env")
    {
        Env();
        return true;
    }
    return false;
}

int Execute()
{
    pid_t id = fork();
    if (id == 0)
    {
        execvpe(g_argv[0], g_argv, g_env);
        exit(1);
    }
    int status = 0;
    pid_t rid = waitpid(id, &status, 0);
    if (rid > 0 && WIFEXITED(status))
    {
        lastcode = WEXITSTATUS(status);
    }
    return 0;
}

int main()
{
    InitEnv();

    while (true)
    {
        PrintCommandPrompt();

        char commandline[COMMAND_SIZE];
        if (!GetCommandLine(commandline, sizeof(commandline)))
            break;

        if (!CommandParse(commandline))
            continue;

        if (CheckAndExecBuiltin())
            continue;

        Execute();
    }
    return 0;
}
```

编译时 `execvpe` 是 GNU 扩展，要带一个宏：

```bash
g++ -std=c++11 -D_GNU_SOURCE -o myshell myshell.cc
```

跑起来是这样：

```text
wfc@VM-0-2-ubuntu:/tmp/probe-myshell# pwd
/tmp/probe-myshell
wfc@VM-0-2-ubuntu:/tmp/probe-myshell# echo hello
hello
wfc@VM-0-2-ubuntu:/tmp/probe-myshell# cd /tmp
wfc@VM-0-2-ubuntu:/tmp# export MYVAL=123
wfc@VM-0-2-ubuntu:/tmp# echo $MYVAL
123
```

`cd` 之后提示符里的路径跟着变了，`export` 进去的变量也能被子进程读到。这一版的内建命令有四个：`cd`、`echo`、`export`、`env`

---

## 相关笔记 / 相关链接

- 相关目录：[[Linux目录]]
- 相关笔记：[[Linux/Linux 进程控制——退出码与进程等待]]（前置：不懂 `status` 位图与 `waitpid`，本篇 `$?` 那节读不懂）
- 相关笔记：[[Linux/Linux 自定义shell尝试（一）]]（前置：本篇的 `cd` 分流建立在第一篇的主循环之上）
