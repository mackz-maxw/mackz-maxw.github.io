---
title: 操作系统基础 | 5.6 等待子进程
categories: os basic
lang: zh-CN
---

### **等待子进程 (Waiting on a Child Process)**

在许多由父进程创建子进程的应用程序中，父进程能够监视子进程以了解它们于何时以及如何终止是非常有用的。`wait()` 系统调用及一系列相关的系统调用提供了这个功能。

### **`wait()` 系统调用 (The wait() System Call)**

`wait()` 系统调用等待调用进程的任一子进程终止，并在 `status` 参数所指向的缓冲区中返回该子进程的终止状态。

```c
#include <sys/wait.h>
pid_t wait(int *status);
```
**返回值：** 成功则返回终止子进程的进程ID (PID)，出错则返回 -1。

`wait()` 系统调用执行以下操作：

1.  如果调用进程的（先前未被等待的）子进程中尚无一个终止，则该调用会**阻塞**，直到某个子进程终止为止。如果在调用时已有子进程终止，`wait()` 则立即返回。
2.  如果 `status` 参数不是 `NULL`，则关于子进程如何终止的信息会通过 `status` 指针所指向的整数返回。我们将在第 26.1.3 节描述 `status` 返回的信息。
3.  内核会将此子进程的 CPU 时间（第 10.7 节）和资源使用统计信息（第 36.1 节）添加到其父进程所有子进程的运行总计时长中。
4.  作为其函数结果，`wait()` 返回已终止子进程的进程 ID。

出错时，`wait()` 返回 -1。一个可能的错误是调用进程没有（先前未被等待的）子进程，这由 `errno` 值 `ECHILD` 指示。这意味着我们可以使用以下循环来等待调用进程的所有子进程终止：

```c
while ((childPid = wait(NULL)) != -1)
    continue;
if (errno != ECHILD)    /* 发生意外错误... */
    errExit("wait");  
```

以下代码演示了 `wait()` 的用法。该程序创建多个子进程，每个命令行整数参数对应一个子进程。每个子进程休眠其对应命令行参数所指定的秒数，然后退出。与此同时，在创建完所有子进程后，父进程反复调用 `wait()` 来监视其子进程的终止。此循环持续直到 `wait()` 返回 -1。（这不是唯一的方法：我们也可以选择当终止的子进程数量 `numDead` 匹配创建的子进程数量时退出循环。）

**创建并等待多个子进程** `––––––––––––––––––––––––––––––––––––––––––––––––––––– procexec/multi_wait.c`

```c
#include <sys/wait.h>
#include <time.h>
#include "curr_time.h"              /* Declaration of currTime() */
#include "tlpi_hdr.h"

int main(int argc, char *argv[]){
    int numDead;       /* 目前已等待的子进程数量 */
    pid_t childPid;    /* 被等待的子进程的PID */
    int j;

    if (argc < 2 || strcmp(argv[1], "--help") == 0)
        usageErr("%s sleep-time...\n", argv[0]);
    setbuf(stdout, NULL);           /* 禁用 stdout 的缓冲 */

    for (j = 1; j < argc; j++) {    /* 为每个参数创建一个子进程 */
        switch (fork()) {
        case -1:
            errExit("fork");
        case 0:                     /* 子进程：休眠一段时间后退出 */
            printf("[%s] child %d started with PID %ld, sleeping %s "
                    "seconds\n", currTime("%T"), j, (long) getpid(), argv[j]);
            sleep(getInt(argv[j], GN_NONNEG, "sleep-time"));
            _exit(EXIT_SUCCESS);
        default:                    /* 父进程：继续循环 */
            break;
        }
    }

    numDead = 0;
    for (;;) {                      /* 父进程等待每个子进程退出 */
        childPid = wait(NULL);
        if (childPid == -1) {
            if (errno == ECHILD) {
                printf("No more children - bye!\n");
                exit(EXIT_SUCCESS);
            } else {                /* 发生其他（意外）错误 */
                errExit("wait");
            }
        }
        numDead++;
        printf("[%s] wait() returned child PID %ld (numDead=%d)\n",
                currTime("%T"), (long) childPid, numDead);
    }
}
```
下面的 shell 会话日志展示了我们使用该程序创建三个子进程时发生的情况：
```
$ ./multi_wait 7 1 4
[13:41:00] child 1 started with PID 21835, sleeping 7 seconds
[13:41:00] child 2 started with PID 21836, sleeping 1 seconds
[13:41:00] child 3 started with PID 21837, sleeping 4 seconds
[13:41:01] wait() returned child PID 21836 (numDead=1)
[13:41:04] wait() returned child PID 21837 (numDead=2)
[13:41:07] wait() returned child PID 21835 (numDead=3)
No more children - bye!
```
如果在某个特定时刻有多个子进程已终止，SUSv3 (Single UNIX Specification, version 3) 未明确规定一系列 `wait()` 调用回收这些子进程的顺序；也就是说，顺序依赖于实现。即使在不同的 Linux 内核版本之间，该行为也有所不同。

### **`waitpid()` 系统调用 (The waitpid() System Call)**

`wait()` 系统调用有一些局限性，`waitpid()` 的设计正是为了应对这些局限性：

*   如果一个父进程创建了多个子进程，使用 `wait()` 无法等待**特定某个子进程**的完成；我们只能等待**下一个终止**的子进程。
*   如果尚无子进程终止，`wait()` **总是会阻塞**。有时，更可取的是执行**非阻塞的等待**，这样如果尚无子进程终止，我们可以立即获得相应的指示。
*   使用 `wait()`，我们只能获知那些**已经终止**的子进程的信息。无法在一个子进程被信号（如 `SIGSTOP` 或 `SIGTTIN`）**停止**时，或在一个被停止的子进程因收到 `SIGCONT` 信号而**恢复**执行时得到通知。

```c
#include <sys/wait.h>
pid_t waitpid(pid_t pid, int *status, int options);
```
**返回值：** 成功则返回状态发生变化的子进程的进程ID (PID)；如果指定了 `WNOHANG` 且未有子进程状态变化则返回 0；出错则返回 -1。

`waitpid()` 的返回值和 `status` 参数与 `wait()` 相同。（关于通过 `status` 返回值的解释，请参见第 26.1.3 节）。`pid` 参数用于选择要等待的子进程，具体如下：

*   如果 `pid` **大于 0**，则等待进程 ID 等于 `pid` 的那个子进程。
*   如果 `pid` **等于 0**，则等待与调用者（父进程）**属于同一进程组**的任何子进程。我们将在第 34.2 节描述进程组。
*   如果 `pid` **小于 -1**，则等待其**进程组标识符**等于 `pid` 绝对值 (`abs(pid)`) 的任何子进程。
*   如果 `pid` **等于 -1**，则等待**任何**子进程。调用 `wait(&status)` 等价于调用 `waitpid(-1, &status, 0)`。

`options` 参数是一个位掩码，可以包含（通过 OR 操作）以下零个或多个标志（所有这些标志都在 SUSv3 中指定）：

*   **`WUNTRACED`**
    除了返回关于已终止子进程的信息外，还会在子进程因收到信号而**停止**时返回其信息。
*   **`WCONTINUED`** (自 Linux 2.6.10 起)
    还会在因收到 `SIGCONT` 信号而**恢复**执行的、之前被停止的子进程的状态信息。
*   **`WNOHANG`**
    如果由 `pid` 指定的子进程尚未改变状态，则立即返回而非阻塞（即执行一次“**轮询**”）。在这种情况下，`waitpid()` 的返回值为 `0`。如果调用进程没有符合 `pid` 指定条件的子进程，则 `waitpid()` 失败并返回错误 `ECHILD`。

我们将在清单 26-3 中演示 `waitpid()` 的用法。

---

**附加说明 (关于 WUNTRACED 名称的由来)：**

在其关于 `waitpid()` 的原理说明中，SUSv3 指出名称 `WUNTRACED` 是该标志源自 BSD 的一个历史产物，在 BSD 中，一个进程可以通过两种方式之一被停止：一种是由于被 `ptrace()` 系统调用**跟踪**的结果，另一种是被信号**停止**（即未被跟踪）。当一个子进程被 `ptrace()` 跟踪时，**任何信号**（除了 `SIGKILL`）的送达都会导致该子进程被停止，并随之向父进程发送一个 `SIGCHLD` 信号。即使子进程**忽略**该信号，此行为也会发生。然而，如果子进程**阻塞**了该信号，则它不会被停止（除非该信号是 `SIGSTOP`，它无法被阻塞）。