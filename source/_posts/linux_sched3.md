---
title: Linux调度算法相关3
---

# 实时进程调度 API

我们现在来看一下组成实时进程调度 API 的各种系统调用。这些系统调用允许我们控制进程的调度策略和优先级。


## 实时优先级范围

`sched_get_priority_min()` 和 `sched_get_priority_max()` 系统调用返回特定调度策略的可用优先级范围。
```c
#include <sched.h>
int sched_get_priority_min(int policy);
int sched_get_priority_max(int policy);
//返回值：成功时返回非负整数优先级，出错时返回 -1。
```
- 对于这两个系统调用，`policy` 指定我们想要获取信息的调度策略。我们可以指定 `SCHED_RR` 或 `SCHED_FIFO`。
- `sched_get_priority_min()` 系统调用返回指定策略的最小优先级，`sched_get_priority_max()` 返回最大优先级。
- 在 Linux 系统中，这两个系统调用分别返回数字 1 和 99，适用于 `SCHED_RR` 和 `SCHED_FIFO` 策略。
- 换句话说，`SCHED_RR` 和 `SCHED_FIFO` 的优先级范围完全相同，`SCHED_RR` 和 `SCHED_FIFO` 的进程在同一优先级时是同样有资格被调度的。首先调度哪个进程取决于它们在该优先级队列中的顺序。

### 实时优先级范围的差异
- 实时优先级范围在不同的 UNIX 实现中有所不同。因此，在应用程序中避免硬编码优先级值时，我们应该根据从这些函数返回的值来指定优先级。
- 例如，最低的 `SCHED_RR` 优先级可以指定为 `sched_get_priority_min(SCHED_RR)`，下一个较高的优先级可以指定为 `sched_get_priority_min(SCHED_RR) + 1`，以此类推。

### SUSv3 和 UNIX 实现的差异
- SUSv3 并不要求 `SCHED_RR` 和 `SCHED_FIFO` 策略使用相同的优先级范围，但在大多数 UNIX 实现中，它们是相同的。
- 例如，在 Solaris 8 中，两个策略的优先级范围为 0 到 59，而在 FreeBSD 6.1 中为 0 到 31。

# 修改和检索调度策略及优先级

## 修改调度策略和优先级
`sched_setscheduler()` 系统调用可以更改指定进程（通过 `pid` 参数指定）的调度策略和优先级。

- **参数说明**：
  - 如果 `pid` 参数被设置为 `0`，则更改的是调用该系统调用的当前进程的属性。

```c
#include <sched.h>
int sched_setscheduler(pid_t pid, int policy, const struct sched_param *param);
```
成功时返回 0。出错时返回 -1。

param：指向如下包含调度优先级信息的结构体的指针。
```c
struct sched_param {
    int sched_priority;  /* 调度优先级 */
};
```
SUSv3 定义了 `param` 参数为一个结构体，以允许实现包括特定于实现的额外字段，这在提供额外的调度策略时可能很有用。然而，与大多数 UNIX 实现一样，Linux 仅提供了 `sched_priority` 字段，用于指定调度优先级。

对于 `SCHED_RR` 和 `SCHED_FIFO` 策略，该字段的值必须位于由 `sched_get_priority_min()` 和 `sched_get_priority_max()` 确定的范围内；对于其他策略，该优先级必须为 0。

`policy` 参数决定了进程的调度策略。它可以是以下策略之一（如表 35-1 所示）：

### 表 35-1：Linux 的实时和非实时调度策略
| 策略         | 描述                                                                 |
|--------------|----------------------------------------------------------------------|
| `SCHED_FIFO` | 实时先进先出（First-in First-out）                                   |
| `SCHED_RR`   | 实时轮转（Round-robin）                                             |
| `SCHED_OTHER`| 标准轮转时间共享                                                    |
| `SCHED_BATCH`| 类似于 `SCHED_OTHER`，但用于批处理（自 Linux 2.6.16 起提供）          |
| `SCHED_IDLE` | 类似于 `SCHED_OTHER`，但优先级比 `nice +19` 更低（自 Linux 2.6.23 起提供）|


### `sched_setscheduler()` 系统调用
- 成功的 `sched_setscheduler()` 调用会将指定的进程（由 `pid` 参数指定）移动到其优先级队列的末尾。

SUSv3 指定，成功的 `sched_setscheduler()` 调用应返回之前的调度策略。然而，Linux 与标准不同，成功调用时返回值为 `0`。  
便携式应用程序应通过检查返回值是否为 `-1` 来判断是否成功。

- 子进程通过 `fork()` 继承其父进程的调度策略和优先级，并在调用 `exec()` 时保留这些属性。

### `sched_setparam()` 系统调用
- `sched_setparam()` 系统调用提供了 `sched_setscheduler()` 功能的子集：
  - 它可以修改进程的调度优先级，但不会更改其调度策略。
```c
#include <sched.h>
int sched_setparam(pid_t pid, const struct sched_param *param);
```
成功时返回 0。出错时返回 -1。
`pid` 和 `param` 参数的含义与 `sched_setscheduler()` 中的相同。

- **成功的 `sched_setparam()` 调用**：  
  成功调用 `sched_setparam()` 后，指定的进程（通过 `pid` 参数指定）会被移动到其优先级队列的队尾。

- **代码示例**：  
  以下程序使用 `sched_setscheduler()` 来根据命令行参数设置指定进程的调度策略和优先级。
  
  - **参数说明**：
    1. 第一个参数是一个字母，用于指定调度策略（如 `SCHED_RR` 或 `SCHED_FIFO`）。
    2. 第二个参数是一个整数，用于指定调度优先级。
    3. 剩余参数是需要更改调度属性的进程 ID 列表。
```c
#include <sched.h>
#include "tlpi_hdr.h"//自定义的头文件，提供错误处理和辅助函数
int main(int argc, char *argv[]) {
    int j, pol;
    struct sched_param sp;
    if (argc < 3 || strchr("rfobi", argv[1][0]) == NULL)
        usageErr("%s policy priority [pid...]\n"
                 "    policy is 'r' (RR), 'f' (FIFO), "
#ifdef SCHED_BATCH
                 " 'b' (BATCH), " /* Linux-specific */
#endif
#ifdef SCHED_IDLE
                 " 'i' (IDLE), "  /* Linux-specific */
#endif
                 "or 'o' (OTHER)\n", argv[0]);

    // 解析调度策略
    pol = (argv[1][0] == 'r') ? SCHED_RR :
          (argv[1][0] == 'f') ? SCHED_FIFO :
#ifdef SCHED_BATCH
          (argv[1][0] == 'b') ? SCHED_BATCH :
#endif
#ifdef SCHED_IDLE
          (argv[1][0] == 'i') ? SCHED_IDLE :
#endif
          SCHED_OTHER;

    // 解析优先级
    sp.sched_priority = getInt(argv[2], 0, "priority");

    // 设置每个指定进程的调度策略和优先级
    for (j = 3; j < argc; j++)
        if (sched_setscheduler(getLong(argv[j], 0, "pid"), pol, &sp) == -1)
            errExit("sched_setscheduler");

    exit(EXIT_SUCCESS);
}
```
### 权限和资源限制对调度参数更改的影响

#### 1. **2.6.12 版本之前的内核行为**
- 在 Linux 2.6.12 内核之前，进程通常需要具有特权（`CAP_SYS_NICE`）才能更改调度策略和优先级。
- 唯一的例外是：如果调用者的有效用户 ID 与目标进程的真实或有效用户 ID 匹配，则非特权进程可以将目标进程的调度策略更改为 `SCHED_OTHER`。

#### 2. **2.6.12 版本及之后的内核行为**
- 自 Linux 2.6.12 起，引入了一个新的非标准资源限制 `RLIMIT_RTPRIO`，调整了实时调度策略和优先级的设置规则：
  - 拥有特权的进程（`CAP_SYS_NICE`）可以对任意进程的调度策略和优先级进行更改。
  - 非特权进程也可以根据以下规则更改调度策略和优先级：


1. **`RLIMIT_RTPRIO` 的非零软限制**：
   - 如果进程具有非零的 `RLIMIT_RTPRIO` 软限制，则可以任意更改其调度策略和优先级。
   - 但有以下限制：
     - 实时优先级的最大值不能超过当前实时优先级和 `RLIMIT_RTPRIO` 软限制中的较大值。

2. **如果`RLIMIT_RTPRIO` 的软限制为零**：
   - 唯一可以进行的更改是：
     - 将实时优先级降低。
     - 或从实时调度策略切换到非实时调度策略。

3. **`SCHED_IDLE` 策略的特殊性**：
   - 运行在 `SCHED_IDLE` 策略下的进程无法更改其调度策略，无论 `RLIMIT_RTPRIO` 的值如何。

4. **从其他非特权进程进行更改**：
   - 如果调用者的有效用户 ID 与目标进程的真实或有效用户 ID 匹配，则可以更改目标进程的调度策略和优先级。

5. **`RLIMIT_RTPRIO` 软限制的作用**：
   - 仅决定进程对自身或其它非特权进程对该进程调度策略和优先级的更改。
   - 一个非零限制并不赋予非特权进程对其他进程的调度策略和优先级进行更改的能力。

### 特殊说明
- 自 Linux 2.6.25 起，添加了实时调度组（Realtime Scheduling Groups）的概念：
  - 可通过 `CONFIG_RT_GROUP_SCHED` 内核选项进行配置。
  - 这一选项影响设置实时调度策略时可以进行的更改。
  - 详情请参阅内核源码文档 `Documentation/scheduler/sched-rt-group.txt`。
