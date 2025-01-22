---
title: Linux调度算法相关4
lang: zh-CN
---

### 检索调度策略和优先级
1. **`sched_getscheduler()` 和 `sched_getparam()` 系统调用**：
   - 这两个系统调用可以检索进程的调度策略和优先级。

```c
#include <sched.h>
int sched_getscheduler(pid_t pid);
//返回调度策略，成功时返回调度策略标识符，出错时返回 -1。
int sched_getparam(pid_t pid, struct sched_param *param);
//成功时返回 0，出错时返回 -1。
```
### 调度参数和优先级获取的系统调用说明

对于这两个系统调用（`sched_getscheduler` 和 `sched_getparam`），`pid` 指定了要获取信息的进程 ID。如果 `pid` 为 0，则获取调用进程的信息。  
不论是否具有特权，这两个系统调用都可用于非特权进程，以获取任何进程的信息，无需凭证。

`sched_getparam()` 系统调用在 `sched_param` 结构体中的 `sched_priority` 字段中返回指定进程的实时优先级。

---

### 示例用法

成功执行后，`sched_getscheduler()` 返回表 35-1 中列出的策略之一。  
Listing 35-3 中的程序使用 `sched_getscheduler()` 和 `sched_getparam()` 来获取命令行参数指定的所有进程的调度策略和优先级。  
下面的 shell 会话演示了该程序和 Listing 35-2 中的程序的使用方法：

```bash
$ su  
Password:  
# sleep 100 &    # 创建一个进程
[1] 2006  
# ./sched_view 2006    # 查看 sleep 进程的初始调度策略和优先级
2006: OTHER 0  
# ./sched_set f 25 2006    # 将进程 2006 切换为 SCHED_FIFO 策略，优先级为 25
# ./sched_view 2006        # 验证修改
2006: FIFO 25  
```
```c
#include <sched.h>
#include "tlpi_hdr.h"

int main(int argc, char *argv[])
{
    int j, pol;
    struct sched_param sp;

    for (j = 1; j < argc; j++) {
        pol = sched_getscheduler(getLong(argv[j], 0, "pid")); // 获取调度策略
        if (pol == -1)
            errExit("sched_getscheduler");

        if (sched_getparam(getLong(argv[j], 0, "pid"), &sp) == -1) // 获取调度参数
            errExit("sched_getparam");

        printf("%s: %-5s %2d\n", argv[j],
            (pol == SCHED_OTHER) ? "OTHER" :     // 普通调度策略
            (pol == SCHED_RR) ? "RR" :          // 轮转调度策略
            (pol == SCHED_FIFO) ? "FIFO" :      // 先进先出调度策略
#ifdef SCHED_BATCH                            // Linux特定调度策略
            (pol == SCHED_BATCH) ? "BATCH" :
#endif
#ifdef SCHED_IDLE                             // Linux特定空闲调度策略
            (pol == SCHED_IDLE) ? "IDLE" :
#endif
            "???",                             // 未知调度策略
            sp.sched_priority);                // 优先级
    }

    exit(EXIT_SUCCESS);
}
```
### 防止实时进程锁死系统

由于 **SCHED_RR** 和 **SCHED_FIFO** 进程会抢占任何低优先级的进程（例如运行程序的shell），在开发使用这些策略的应用程序时，我们需要注意一个可能的风险：一个失控的实时进程可能会通过占用CPU导致系统锁死。为避免这种情况，可以采用以下几种方法：

- **设置一个足够低的软CPU时间资源限制（`RLIMIT_CPU`）**  
  使用 `setrlimit()` 函数（参考第36.3节）来设置。如果进程消耗了过多的CPU时间，将会被发送一个 `SIGXCPU` 信号，该信号默认会杀死进程。

- **使用 `alarm()` 设置报警计时器**  
  如果进程运行的实际时间超过了 `alarm()` 调用中指定的秒数，则会被发送一个 `SIGALRM` 信号并被终止。

- **创建一个具有高实时优先级的守护进程**  
  该进程可以循环运行，按照指定的间隔睡眠，然后唤醒并监视其他进程的状态。此类监视可以包括：  
  - 检查每个进程的CPU时间时钟值（参考23.5.3节中的 `clock_getcpuclockid()` 函数）；
  - 使用 `sched_getscheduler()` 和 `sched_getparam()` 检查其调度策略和优先级。  
  如果某个进程被认为行为异常，守护线程可以通过降低其优先级或发送适当的信号终止该进程来控制它。

- **从Linux内核2.6.25开始，提供了非标准的资源限制 `RLIMIT_RTTIME`**  
  该限制用于控制实时调度策略进程在单次连续运行中可以消耗的CPU时间，单位为微秒。  
  - `RLIMIT_RTTIME` 限制了进程在不执行阻塞系统调用时所能消耗的CPU时间总量。  
  - 当进程执行阻塞系统调用时，消耗的CPU时间计数会被重置为0。  
  - 如果进程被更高优先级的进程抢占、由于时间片到期（**SCHED_RR** 进程）或调用 `sched_yield()`（第35.3.3节）而被调度出CPU，则计数也会被重置。  
  - 如果进程达到 `RLIMIT_CPU` 限制，将收到一个 `SIGXCPU` 信号，默认情况下会杀死进程。

#### 注意
内核2.6.25中的改动也有助于防止失控的实时进程锁死系统。有关详细信息，请参阅内核源代码文档：`scheduler/sched-rt-group.txt`。

### 防止子进程继承特权调度策略
Linux 2.6.32 引入了 `SCHED_RESET_ON_FORK` 作为可以在调用 `sched_setscheduler()` 时指定的策略值之一。  
这是一个标志值，它会与表35-1中的策略之一进行位或操作。如果设置了此标志，则由此进程通过 `fork()` 创建的子进程不会继承特权调度策略和优先级。规则如下：

- 如果调用进程具有实时调度策略（**SCHED_RR** 或 **SCHED_FIFO**），那么子进程的调度策略将被重置为标准的时间片轮转策略，**SCHED_OTHER**。
- 如果进程具有负的（即较高的）nice 值，则子进程的 nice 值将被重置为0。

### `SCHED_RESET_ON_FORK` 的作用
`SCHED_RESET_ON_FORK` 标志旨在用于媒体播放等应用场景。它允许创建具有实时调度策略的单个进程，而这些策略不会传递给子进程。  
使用 `SCHED_RESET_ON_FORK` 标志可以防止通过创建多个运行在实时调度策略下的子进程来试图规避 **RLIMIT_RTTIME** 资源限制的 fork 炸弹攻击。

一旦为一个进程启用了 `SCHED_RESET_ON_FORK` 标志，只有具备特权的进程（即具有 **CAP_SYS_NICE** 权限的进程）可以禁用它。  此时当创建子进程时，其 reset-on-fork 标志会被禁用。

---

### 让出 CPU

一个实时进程可以通过以下两种方式主动让出 CPU：
1. 调用一个会阻塞进程的系统调用（例如，从终端执行 `read()`）。
2. 调用 `sched_yield()`。
```c
#include <sched.h>
int sched_yield(void);
//成功时返回 0，出错时返回 -1。
```
### `sched_yield()` 的操作

`sched_yield()` 的操作非常简单。如果有其他与调用进程处于相同优先级队列中的可运行进程，则调用进程会被放到队列的末尾，并由队列头部的进程被调度使用 CPU。  
如果在此优先级上没有其他可运行进程排队，那么 `sched_yield()` 什么都不做；调用进程将继续使用 CPU。

尽管 SUSv3 允许 `sched_yield()` 返回可能的错误，但在 Linux 以及许多其他 UNIX 实现中，此系统调用始终成功。  
然而，可移植的应用程序仍应始终检查是否有错误返回。

对于非实时进程使用 `sched_yield()` 的行为是未定义的。

---

### **`SCHED_RR` 时间片**

`sched_rr_get_interval()` 系统调用使我们能够确定为 `SCHED_RR` 进程每次被分配使用 CPU 时的时间片长度。
```c
#include <sched.h>
int sched_rr_get_interval(pid_t pid, struct timespec *tp);
//成功时返回 0，出错时返回 -1。
```
与其他进程调度系统调用一样，`pid` 用于标识我们希望获取信息的进程。指定 `pid` 为 0 表示当前调用进程。  
时间片的长度通过指向 `tp` 的 `timespec` 结构返回：

```c
struct timespec {
    time_t tv_sec;   /* 秒 */
    long   tv_nsec;  /* 纳秒 */
};
```
在最近的 2.6 内核中，实时轮转调度（SCHED_RR）的时间片为 0.1 秒。
### 总结

默认的内核调度算法采用的是轮转时间共享策略。在这种策略下，所有进程默认可以平等地访问 CPU。不过，我们可以通过将进程的 nice 值设置为 -20（高优先级）到 +19（低优先级）的范围内的数字，从而使调度器偏爱或减少对该进程的调度。然而，即使我们将进程设置为最低优先级，它也不会完全失去对 CPU 的访问。

Linux 还实现了 POSIX 实时调度扩展，这允许应用程序精确控制进程对 CPU 的分配。在实时调度的两种策略下运行的进程，**SCHED_RR**（轮转调度）和 **SCHED_FIFO**（先进先出），总是优先于运行在非实时策略下的进程。实时进程的优先级范围是 1（低优先级）到 99（高优先级）。只要高优先级进程可运行，它就会完全排除低优先级进程对 CPU 的访问。

在 **SCHED_FIFO** 策略下运行的进程会独占 CPU，直到进程终止、自愿释放 CPU，或者被一个更高优先级的进程抢占。同样的规则适用于 **SCHED_RR** 策略，不同之处在于，如果多个进程具有相同的优先级，则 CPU 在这些进程之间以轮转方式共享。

此外，可以使用进程的 CPU 亲和性掩码（CPU affinity mask）来限制进程仅运行在多处理器系统的某些 CPU 上。这有助于提高某些应用程序类型的性能。
