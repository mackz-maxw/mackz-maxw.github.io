---
title: 操作系统基础 | 5.7 孤儿进程；错误检查
categories: os basic
lang: zh-CN
---

### **26.2 孤儿进程与僵尸进程 (Orphans and Zombies)**

父进程和子进程的生命周期通常并不相同——要么父进程比子进程存活时间长，要么相反。这就引出了两个问题：

**1. 孤儿进程由谁接管？**
当一个进程的父进程终止后，该进程就变成了“孤儿进程”。这个孤儿进程会被 `init` 进程（所有进程的祖先，其进程 ID 为 1）收养。换句话说，在子进程的父进程终止后，调用 `getppid()` 将返回值 1。这可以作为一种判断子进程的真正父进程是否仍存活的方法（前提是该子进程并非由 `init` 进程创建）。
使用 Linux 特有的 `prctl()` 系统调用的 `PR_SET_PDEATHSIG` 操作，可以设置一个进程在成为孤儿时接收到一个特定的信号。

**2. 父进程还未执行 wait() 子进程就已终止，会发生什么？**
关键在于，尽管子进程已经完成了它的工作，但仍应允许父进程在之后的某个时间点执行 `wait()` 来获取子进程的终止状态。内核通过将子进程转变为“僵尸进程”（Zombie）来处理这种情况。这意味着子进程持有的大部分资源会被释放回系统，供其他进程重用。进程唯一保留的部分是内核进程表中的一个条目，该条目记录了子进程的进程 ID、终止状态以及资源使用统计信息（参见 36.1 节）等信息。

关于僵尸进程，UNIX 系统模仿了电影中的设定——僵尸进程无法被信号杀死，即使是（银弹）`SIGKILL` 信号也不行。这确保了父进程最终总是能够执行 `wait()`。

当父进程执行了 `wait()` 后，内核会清除该僵尸进程，因为关于该子进程的最后剩余信息不再需要。另一方面，如果父进程未执行 `wait()` 就终止了，那么 `init` 进程会收养该子进程，并自动执行 `wait()`，从而将僵尸进程从系统中移除。

如果一个父进程创建了子进程，但未能执行 `wait()`，那么内核进程表中将无限期地保留该僵尸子进程的条目。如果创建了大量这样的僵尸子进程，它们最终会填满内核进程表，从而阻止新进程的创建。由于僵尸进程无法用信号杀死，将它们从系统中移除的唯一方法是杀死它们的父进程（或等待其退出），届时 `init` 进程会收养这些僵尸进程并对它们执行 `wait()`，从而将它们从系统中清除。

这些语义对于需要创建大量子进程的长生命周期父进程（如网络服务器和 Shell）的设计具有重要意义。换句话说，在此类应用程序中，父进程应执行 `wait()` 调用，以确保已终止的子进程总是能从系统中被移除，而不是变成长期存在的僵尸进程。如 26.3.1 节所述，父进程可以同步执行这些 `wait()` 调用，也可以异步地（例如响应 `SIGCHLD` 信号的递送）执行。

以下程序演示了僵尸进程的创建以及僵尸进程无法被 `SIGKILL` 杀死的情况。
```c
#include <signal.h>
#include <libgen.h>             /* For basename() declaration */
#include "tlpi_hdr.h"

#define CMD_SIZE 200

int main(int argc, char *argv[]){
    char cmd[CMD_SIZE];
    pid_t childPid;

    setbuf(stdout, NULL);       /* Disable buffering of stdout */
    printf("Parent PID=%ld\n", (long) getpid());

    switch (childPid = fork()) {
    case -1:
        errExit("fork");
    case 0:     /* Child: immediately exits to become zombie */
        printf("Child (PID=%ld) exiting\n", (long) getpid());
        _exit(EXIT_SUCCESS);
    default:    /* Parent */
        sleep(3);               /* Give child a chance to start and exit */
        snprintf(cmd, CMD_SIZE, "ps | grep %s", basename(argv[0]));
        cmd[CMD_SIZE - 1] = '\0';    /* Ensure string is null-terminated */
        system(cmd);            /* View zombie child */

        /* Now send the "sure kill" signal to the zombie */
        if (kill(childPid, SIGKILL) == -1)
            errMsg("kill");
        sleep(3);               /* Give child a chance to react to signal */
        printf("After sending SIGKILL to zombie (PID=%ld):\n", (long) childPid);
        system(cmd);            /* View zombie child again */

        exit(EXIT_SUCCESS);
    }
}
```
当我们运行此程序时，会看到以下输出：
```
$ ./make_zombie
Parent PID=1013
Child (PID=1014) exiting
 1013 pts/4   00:00:00 make_zombie
 1014 pts/4   00:00:00 make_zombie <defunct>       (ps命令的输出)
After sending SIGKILL to zombie (PID=1014):
 1013 pts/4   00:00:00 make_zombie
 1014 pts/4   00:00:00 make_zombie <defunct>       (ps命令的输出)
```
在上面的输出中，我们看到 `ps(1)` 命令显示字符串 `<defunct>` 来表示一个处于僵尸状态的进程。
示例程序使用 `system()` 函数来执行其字符串参数中给出的 Shell 命令。我们将在 27.6 节详细描述 `system()`。

### **Linux 进程源代码指引**

进程和线程是操作系统中的基本抽象概念。为支持进程和线程，操作系统使用了两个关键数据结构：`task_struct` 和 `thread_info`；以及大量用于管理系统中进程和线程的辅助函数。

*   **`arch/arm/include/asm/thread_info.h`** 文件声明了 `thread_info` 结构体。
*   **`include/linux/sched.h`** 文件声明了 `task_struct` 结构体以及许多进程管理函数。
*   **`arch/arm/include/asm/switch_to.h`** 文件为 `switch_to` 进程切换例程定义了一个特定于 ARM 架构的宏，而 **`arch/arm/kernel/entry-armv.S`** 文件实现了该宏所使用的汇编级进程切换例程 `__switch_to`。


### **Linux 内核代码错误检查**

在 Linux 用户空间编程中，函数通常返回整数，并约定返回值 `0` 表示函数调用成功，而不同的（通常为负的）非零值用于指示不同的错误。然而，在内核编程中，许多函数可能返回指针而非整数，这使问题变得复杂，因为指针可能使用非零值来编码作为函数成功调用结果而返回的有效内存地址。

Linux 通过利用以下事实来解决这一挑战：
1.  有效（虚拟）地址范围的上半部分未被使用。
2.  使用负值表示错误会使得其高位比特位非零，无论它们是作为指针还是整数返回。
3.  在 **`include/linux/err.h`** 文件中提供有用的宏和内联函数，这些函数可以以跨不同硬件架构的可移植方式处理指针、整数和布尔类型的不同组合：

*   **`IS_ERR_VALUE` 宏**：检查一个（指针或整数）值的高位范围是否非空（即，包含错误值）。
*   **`ERR_PTR` 函数**：将一个（`long` 类型）整数值转换为（`void *` 类型）指针值。
*   **`PTR_ERR` 函数**：将一个（`const void *` 类型）指针值转换为（`long` 类型）整数值。
*   **`IS_ERR` 函数**：将一个（`const void *` 类型）指针值转换为（`unsigned long` 类型）整数值，使用 `IS_ERR_VALUE` 宏检查该值的高位范围是否非空（即，包含错误值），并相应地返回一个 `bool` 值。
*   **`IS_ERR_OR_NULL` 函数**：返回一个 `bool` 值，如果 (1) 传入的（`const void *` 类型）指针值为 `0` **或** (2) 将其转换为（`unsigned long` 类型）整数值后使用 `IS_ERR_VALUE` 宏检查表明该值的高位范围非空（即，包含错误值），则该函数返回 `true`。
*   **`ERR_CAST` 函数**：将（`const void *` 类型）指针转换为（`void *` 类型）指针（去除 `const` 属性）。
*   **`PTR_ERR_OR_ZERO` 函数**：使用 `IS_ERR` 函数检查传入的（`const void *` 类型）指针值是否包含错误，并返回一个（`int` 类型）整数值：如果不包含错误则返回 `0`，如果包含错误则返回通过调用 `PTR_ERR` 函数获取的错误值。


### **`ERR_PTR` 和 `PTR_ERR` 宏**

关于返回值的讨论，你现在明白了内核模块的 `init` 例程必须返回一个整数。但如果你希望返回一个指针呢？ **`ERR_PTR()`** 内联函数为我们提供了解决方案，它允许我们通过将指针类型转换为 `void *` 来返回一个伪装成整数的指针。实际上还有更好的方法：你可以使用 **`IS_ERR()`** 内联函数检查错误（该函数实质上判断值是否在 `[-1, -4095]` 范围内），通过 **`ERR_PTR()`** 内联函数将负的错误值编码到指针中，并使用对应的例程 **`PTR_ERR()`** 从指针中检索出这个错误值。

一个简单的例子，参见下面给出的被调用方代码。这次，我们让（示例）函数 `myfunc()` 返回一个（指向名为 `mystruct` 的结构体的）指针，而不是整数：

```c
struct mystruct * myfunc(void)
{
    struct mystruct *mys = NULL;
    mys = kzalloc(sizeof(struct mystruct), GFP_KERNEL);
    if (!mys)
        return ERR_PTR(-ENOMEM); /* 示例：返回编码为指针的错误码 */
    /* ... 其他操作 ... */
    return mys; /* 返回有效的指针 */
}
```

### **如何使用 IS_ERR 和 PTR_ERR？它们是什么意思？**

根据内核定义，有三个宏：
*   **`IS_ERR`** - 用于检查。如果 `ptr` 是一个错误指针则返回非 `0` 值。否则，如果不是错误则返回 `0`。
*   **`PTR_ERR`** - 用于打印。获取指针中当前（编码的错误）值。
*   **`IS_ERR_VALUE`** - 在此有更详细的解释 (here1)。

我发现这些宏对于内核空间编程非常有用。用法如下 - 如果 `ptr` 是你要检查的指针，则按如下方式使用：
```c
if (IS_ERR(ptr))
     printk("Error here: %ld", PTR_ERR(ptr)); /* 打印出错误码 */
```
它们在内核中的代码定义如下：
```c
#define IS_ERR_VALUE(x) unlikely((x) >= (unsigned long)-MAX_ERRNO)
static inline void * __must_check ERR_PTR(long error){
    return (void *) error;
}
static inline long __must_check PTR_ERR(const void *ptr){
    return (long) ptr;
}
static inline long __must_check IS_ERR(const void *ptr){
    return IS_ERR_VALUE((unsigned long)ptr);
}
static inline long __must_check IS_ERR_OR_NULL(const void *ptr){
    return !ptr || IS_ERR_VALUE((unsigned long)ptr);
}
```
