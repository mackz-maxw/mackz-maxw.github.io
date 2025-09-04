---
title: 操作系统基础 | 5.2 进程管理详解
categories: os basic
lang: zh-CN
---

### **进程创建 (Process Creation)**

Unix 中的进程创建方式是独特的。大多数操作系统实现一种 **spawn 机制**来在新地址空间中创建新进程、读入可执行文件并开始执行。Unix 则采用了一种不寻常的方法，将这些步骤分离成两个不同的函数：`fork()` 和 `exec()`⁷。第一个函数 `fork()` 创建一个作为当前任务副本的**子进程**。它与父进程的区别仅在于其 PID（是唯一的）、其 PPID（父进程的 PID，被设置为原始进程的 PID）以及某些资源和统计信息（如待处理的信号，这些不被继承）。第二个函数 `exec()` 将一个新的可执行文件加载到地址空间中并开始执行。`fork()` 后接 `exec()` 这种组合，类似于大多数操作系统提供的单一函数。

⁷ 这里的 `exec()` 指的是 `exec()` 函数家族中的任何成员。内核实现了 `execve()` 系统调用，基于此实现了 `execlp()`, `execle()`, `execv()`, 和 `execvp()`。

#### **写时复制 (Copy-on-Write)**

传统上，在 `fork()` 时，父进程拥有的所有资源都会被复制，并将副本交给子进程。这种方法很朴素且低效，因为它复制了许多本可以共享的数据。更糟糕的是，如果新进程要立即执行一个新的映像（image），所有这些复制都将被浪费。在 Linux 中，`fork()` 是通过使用**写时复制 (Copy-on-Write 或 COW)** 页来实现的。写时复制是一种延迟或完全防止数据复制的技术。父进程和子进程可以共享一份单一的副本，而不是复制整个进程地址空间。

然而，数据会被标记，如果对其进行写入，就会创建一个副本，每个进程都会收到一个独一无二的副本。因此，资源的复制**仅发生在它们被写入时**；在此之前，它们以只读方式共享。这种技术将地址空间中每一页的复制延迟到实际被写入的时候。如果这些页永远不被写入——例如，如果在 `fork()` 之后立即调用 `exec()`——它们就永远不需要被复制。

`fork()` 产生的唯一开销是复制父进程的**页表 (page tables)** 以及为子进程创建一个唯一的进程描述符。在常见的场景中（进程在 fork 后立即执行一个新的可执行映像），这种优化避免了大量数据（整个地址空间，很容易达到几十兆字节）的浪费性复制。这是一个重要的优化，因为 Unix 哲学鼓励快速的进程执行。

#### **Forking**

Linux 通过 `clone()` 系统调用来实现 `fork()`。这个调用接受一系列标志（flags），用以指定父进程和子进程应共享哪些资源（如果有的话）。（关于这些标志的更多信息，请参阅本章后面的“Linux 的线程实现”一节。）`fork()`, `vfork()`, 和 `__clone()` 库调用都使用必要的标志来调用 `clone()` 系统调用。而 `clone()` 系统调用又会调用 `do_fork()`。

forking 的大部分工作由 `do_fork()` 处理，该函数定义在 `kernel/fork.c` 中。此函数调用 `copy_process()`，然后启动进程运行。有趣的工作是由 `copy_process()` 完成的：

1.  它调用 `dup_task_struct()`，该函数为新进程创建一个新的内核栈、`thread_info` 结构和 `task_struct`。新的值与当前任务的值完全相同。此时，子进程和父进程的描述符是完全相同的。
2.  然后检查确保新的子进程不会超过当前用户所能拥有的进程数量资源限制。
3.  子进程需要与父进程区分开来。进程描述符的各种成员被清除或设置为初始值。不被继承的进程描述符成员主要是统计信息。`task_struct` 中的大部分值保持不变。
4.  子进程的状态被设置为 `TASK_UNINTERRUPTIBLE` 以确保它还不会运行。
5.  `copy_process()` 调用 `copy_flags()` 来更新 `task_struct` 的 `flags` 成员。`PF_SUPERPRIV` 标志（表示任务是否使用了超级用户权限）被清除。`PF_FORKNOEXEC` 标志（表示进程尚未调用 `exec()`）被设置。
6.  它调用 `alloc_pid()` 为新任务分配一个可用的 PID。
7.  根据传递给 `clone()` 的标志，`copy_process()` 要么复制要么共享打开的文件、文件系统信息、信号处理程序、进程地址空间和命名空间。这些资源通常在给定进程的线程之间共享；否则，它们是唯一的，因此在这里被复制。
8.  最后，`copy_process()` 进行清理工作，并向调用者返回一个指向新子进程的指针。

回到 `do_fork()`，如果 `copy_process()` 成功返回，新的子进程会被唤醒并运行。内核有意地让**子进程先运行**⁸。在子进程通常只是立即调用 `exec()` 的常见情况下，这消除了如果父进程先运行并开始写入地址空间可能发生的任何写时复制开销。

> ⁸ 虽然目标是让子进程先运行，但这目前（指原书写作时）并不能正确运行。

#### **vfork()**

`vfork()` 系统调用与 `fork()` 效果相同，但**不会复制父进程的页表项 (page table entries)**。相反，子进程作为父进程地址空间中的唯一线程执行，并且**父进程被阻塞**，直到子进程调用 `exec()` 或退出。不允许子进程写入地址空间。在引入这个调用的旧版 3BSD 时代，这是一个受欢迎的优化，因为当时还没有使用写时复制页来实现 `fork()`。如今，有了写时复制和子进程优先运行的语义，`vfork()` 的唯一好处就是**不复制父进程的页表项**。

如果 Linux 有一天实现了写时复制页表项，那么 `vfork()` 将不再有任何好处⁹。由于 `vfork()` 的语义很棘手（例如，如果 `exec()` 失败了会发生什么？），理想情况下系统不需要 `vfork()`，内核也不必实现它。完全可以将 `vfork()` 实现为一个普通的 `fork()` —— 这就是 Linux 在 2.2 版本之前所做的。

`vfork()` 系统调用是通过向 `clone()` 系统调用传递一个特殊标志来实现的：

1.  在 `copy_process()` 中，`task_struct` 的成员 `vfork_done` 被设置为 `NULL`。
2.  在 `do_fork()` 中，如果给定了特殊标志，`vfork_done` 会被指向一个特定的地址。
3.  子首次运行后，父进程——不会立即返回——而是等待子进程通过 `vfork_done` 指针向其发出信号。
4.  在 `mm_release()` 函数中（该函数在任务退出内存地址空间时使用），会检查 `vfork_done` 是否为 `NULL`。如果不是，则向父进程发送信号。
5.  回到 `do_fork()`，父进程被唤醒并返回。

如果这一切都按计划进行，那么子进程现在在新的地址空间中执行，而父进程则在其原始地址空间中恢复执行。开销是降低了，但实现并不优雅。

> ⁹ 页表写时复制可作为补丁，已大概率加入主流内核

### **Linux 的线程实现 (The Linux Implementation of Threads)**

线程是一种流行的现代编程抽象。它们在同一程序的共享内存地址空间内提供多个执行线程。它们还可以共享打开的文件和其他资源。线程实现了并发编程，并且在多处理器系统上实现了真正的并行。

Linux 拥有一个独特的线程实现。**对 Linux 内核而言，没有“线程”的概念**。Linux 将所有线程实现为标准进程。Linux 内核并不提供任何特殊的调度语义或数据结构来表示线程。相反，一个线程仅仅是一个与其他进程共享某些资源的进程。每个线程都有一个唯一的 `task_struct`，并且在内核看来就是一个普通的进程——线程只是恰巧与其他进程共享了资源（例如地址空间）。

这种线程实现方法与 Microsoft Windows 或 Sun Solaris 等操作系统形成了巨大对比，这些操作系统在内核中提供了对线程的明确支持（有时称线程为**轻量级进程 (Lightweight Processes)**）。“轻量级进程”这个名字概括了 Linux 与其他系统在哲学上的差异。对这些其他操作系统而言，线程是一种抽象，旨在提供比笨重进程更轻量、更快速的执行单元。而对 Linux 而言，线程仅仅是进程之间共享资源的一种方式（而进程本身已经相当轻量）¹⁰。

例如，假设一个包含四个线程的进程。在具有显式线程支持的系统中，可能会存在一个进程描述符，该描述符又指向四个不同的线程。进程描述符描述共享资源，如地址空间或打开的文件。线程则描述它们独自拥有的资源。相反，在 Linux 中，简单地存在四个进程，因而有四个普通的 `task_struct` 结构。这四个进程被设置为共享某些资源。结果非常优雅。

> ¹⁰ 例如，可以对比 benchmark 一下 Linux 的进程创建时间与其他操作系统的进程（甚至线程！）创建时间。结果对 Linux 有利。

#### **创建线程 (Creating Threads)**

线程的创建方式与普通任务相同，区别在于需要向 `clone()` 系统调用传递一些标志（flags），这些标志指定了要共享的特定资源：
```c
clone(CLONE_VM | CLONE_FS | CLONE_FILES | CLONE_SIGHAND, 0);
```
上述代码产生的行为与普通的 `fork()` 相同，但**地址空间、文件系统资源、文件描述符和信号处理程序是共享的**。换句话说，新任务和其父进程就是通常所说的线程。相比之下，一个普通的 `fork()` 可以实现为：
```c
clone(SIGCHLD, 0);
```
而 `vfork()` 可以实现为：
```c
clone(CLONE_VFORK | CLONE_VM | SIGCHLD, 0);
```
提供给 `clone()` 的标志有助于指定新进程的行为，并详细说明父进程和子进程将共享哪些资源。表 3.1 列出了在 `<linux/sched.h>` 中定义的 clone 标志及其作用。

**(表 3.1 Clone 标志及含义)**
| 标志 (Flag)              | 含义 (Meaning)                                                                 |
| :----------------------- | :----------------------------------------------------------------------------- |
| `CLONE_FILES`            | 父子进程共享打开的文件。                                                               |
| `CLONE_FS`               | 父子进程共享文件系统信息（如根目录、当前工作目录）。                                               |
| `CLONE_IDLETASK`         | 将 PID 设置为零（仅由空闲任务使用）。                                                     |
| `CLONE_NEWNS`            | 为子进程创建新的命名空间 (namespace)。                                                   |
| `CLONE_PARENT`           | 子进程与父进程拥有相同的父进程（即调用者的父进程）。                                             |
| `CLONE_PTRACE`           | 继续跟踪子进程。                                                                     |
| `CLONE_SETTID`           | 将 TID（线程 ID）写回用户空间。                                                        |
| `CLONE_SETTLS`           | 为子进程创建新的 TLS（线程本地存储）。                                                    |
| `CLONE_SIGHAND`          | 父子进程共享信号处理程序和阻塞的信号掩码。                                                   |
| `CLONE_SYSVSEM`          | 父子进程共享 System V SEM_UNDO 语义。                                                |
| `CLONE_THREAD`           | 父子进程位于同一个线程组中。这是将新进程标识为线程而非普通进程的关键标志。                                  |
| `CLONE_VFORK`            | 使用了 `vfork()`，父进程将睡眠直到子进程唤醒它。                                            |
| `CLONE_UNTRACED`         | 不允许跟踪进程强制对子进程使用 `CLONE_PTRACE`。                                           |
| `CLONE_STOP`             | 以 `TASK_STOPPED` 状态启动进程。                                                     |
| `CLONE_CHILD_CLEARTID`   | 在子进程中清除 TID。                                                                 |
| `CLONE_CHILD_SETTID`     | 在子进程中设置 TID。                                                                 |
| `CLONE_PARENT_SETTID`    | 在父进程中设置 TID。                                                                 |
| `CLONE_VM`               | 父子进程共享地址空间。这是创建线程的关键标志之一。                                               |

#### **内核线程 (Kernel Threads)**

内核在后台执行某些操作通常很有用。内核通过**内核线程 (kernel threads)** 来实现这一点——内核线程是**仅存在于内核空间的标准进程**。内核线程与普通进程的显著区别在于内核线程**没有地址空间**（它们的 `mm` 指针指向它们的地址空间，为 `NULL`）。它们只在内核空间中运行，不会上下文切换到用户空间。然而，内核线程与普通进程一样，是可调度和可抢占的。

Linux 将一些任务委托给内核线程，最著名的是 `flush` 任务和 `ksoftirqd` 任务。您可以在 Linux 系统上运行 `ps -ef` 命令来查看内核线程。数量很多！

> -ef 是 every full(info) 的简写。在所有进程中，CMD 带 []，TTY 为 ?，PPID 为 2 或 0， UID是0或者root，是内核线程的特征

内核线程在系统启动时由其他内核线程创建。实际上，内核线程只能由另一个内核线程创建。内核通过从 `kthreadd` 内核进程 **fork** 出所有新的内核线程来自动处理此事。在 `<linux/kthread.h>` 中声明的，用于从现有内核线程生成新内核线程的接口是：
```c
struct task_struct *kthread_create(int (*threadfn)(void *data),
                                   void *data,
                                   const char namefmt[], ...);
```
新任务由 kthread 内核进程通过 `clone()` 系统调用创建。新进程将运行 `threadfn` 函数，该函数接收 `data` 参数。进程将被命名为 `namefmt`，该参数接受在可变参数列表中的 printf 风格格式化参数。进程创建时处于**不可运行状态 (unrunnable state)**；只有在通过 `wake_up_process()` 显式唤醒后，它才会开始运行。可以使用单个函数 `kthread_run()` 来创建并使进程可运行：
```c
struct task_struct *kthread_run(int (*threadfn)(void *data),
                                void *data,
                                const char namefmt[], ...);
```
这个例程（作为宏实现）简单地调用了 `kthread_create()` 和 `wake_up_process()`：
```c
#define kthread_run(threadfn, data, namefmt, ...)                     \
({                                                                    \
    struct task_struct *k;                                        \
                                                                      \
    k = kthread_create(threadfn, data, namefmt, ## __VA_ARGS__);  \
    if (!IS_ERR(k))                                               \
        wake_up_process(k);                                   \
    k;                                                            \
})
```
启动后，内核线程会继续存在，直到它调用 `do_exit()`，或者内核的另一部分调用 `kthread_stop()`（传入由 `kthread_create()` 返回的 `task_struct` 结构体的地址）：
```c
int kthread_stop(struct task_struct *k);
```

### **进程终止 (Process Termination)**

这很遗憾，但进程最终都会消亡。当一个进程终止时，内核会释放该进程所拥有的资源，并通知其父进程关于子进程消亡的消息。

通常，进程销毁是**自我诱导 (self-induced)** 的。当进程调用 `exit()` 系统调用时就会发生，这可以是在它准备终止时显式调用，也可以是在从任何程序的 main 子程序返回时隐式调用（即 C 编译器会在 `main()` 返回后放置一个对 `exit()` 的调用）。进程也可能**非自愿地 (involuntarily)** 终止。这发生在进程收到一个它无法处理或忽略的信号或异常时。

无论进程如何终止，大部分工作都由 `do_exit()` 处理（定义在 `kernel/exit.c` 中），它完成一系列收尾工作：

1.  它在 `task_struct` 的 `flags` 成员中设置 `PF_EXITING` 标志。
2.  它调用 `del_timer_sync()` 来移除任何内核定时器。确保返回时没有定时器在排队，也没有定时器处理程序在运行。
3.  如果启用了 BSD 进程记账 (process accounting)，`do_exit()` 会调用 `acct_update_integrals()` 来写出记账信息。
4.  它调用 `exit_mm()` 来释放该进程持有的 `mm_struct`。如果没有其他进程在使用这个地址空间（即地址空间未被共享），内核就会销毁它。
5.  它调用 `exit_sem()`。如果进程正在排队等待一个 IPC 信号量，它在这里被移出队列。
6.  然后它调用 `exit_files()` 和 `exit_fs()` 来分别递减与文件描述符和文件系统数据相关的对象的引用计数。如果某个引用计数降为零，说明该对象不再被任何进程使用，随即被销毁。
7.  它将任务的退出码（存储在 `task_struct` 的 `exit_code` 成员中）设置为 `exit()` 提供的代码或任何强制终止它的内核机制所提供的代码。退出码存储在这里，供父进程选择性检索。
8.  它调用 `exit_notify()` 来向任务的父进程发送信号，将该任务的任何子进程**重新设定父进程 (reparent)** 给其线程组中的另一个线程或 init 进程，并将任务的退出状态（存储在 `task_struct` 结构的 `exit_state` 中）设置为 `EXIT_ZOMBIE`。
9.  `do_exit()` 调用 `schedule()` 来切换到新进程（参见第 4 章）。因为该进程现在已不可调度，这是该任务将执行的最后代码。`do_exit()` 永不返回。

至此，与任务关联的所有对象（假设该任务是唯一使用者）都已释放。该任务不可运行（并且不再有地址空间可供运行），并处于 `EXIT_ZOMBIE`（僵尸）退出状态。它占用的唯一内存是它的内核栈、`thread_info` 结构和 `task_struct` 结构。该任务存在的唯一目的是向其父进程提供信息。在父进程检索到信息，或通知内核它不感兴趣之后，进程持有的剩余内存将被释放并返回给系统使用。

#### **移除进程描述符 (Removing the Process Descriptor)**

在 `do_exit()` 完成后，已终止进程的进程描述符仍然存在，但该进程已成为僵尸 (zombie) 且无法运行。如前所述，这使得系统能够在子进程终止后获取其信息。因此，清理进程之后和移除其进程描述符是分开的两个步骤。在父进程获取了已终止子进程的信息，或向内核表示不关心之后，子进程的 `task_struct` 才会被释放。

`wait()` 函数族是通过一个单一（且复杂）的系统调用 `wait4()` 实现的。标准行为是**挂起调用任务的执行**，直到它的一个子进程退出，此时函数返回退出子进程的 PID。此外，还提供一个指针给该函数，该指针在返回时持有已终止子进程的退出码。

当最终要释放进程描述符时，会调用 `release_task()`。它执行以下操作：
1.  它调用 `__exit_signal()`，后者又调用 `__unhash_process()`，继而调用 `detach_pid()` 来将进程从 pidhash 中移除，并从任务列表中移除该进程。
2.  `__exit_signal()` 释放这个已死亡进程使用的任何剩余资源，并完成统计和簿记工作。
3.  如果该任务是线程组的最后一个成员，并且领导者 (leader) 是僵尸进程，那么 `release_task()` 会通知僵尸领导者的父进程。
4.  `release_task()` 调用 `put_task_struct()` 来释放包含进程内核栈和 `thread_info` 结构的内存页，并释放包含 `task_struct` 的 slab 缓存。

至此，进程描述符以及仅属于该进程的所有资源都已被释放。

#### **无父任务的困境 (The Dilemma of the Parentless Task)**

如果一个父进程在其子进程之前退出，必须存在某种机制来将任何子任务**重新设定父进程 (reparent)** 给一个新进程，否则，没有父进程的已终止进程将永远保持僵尸状态，浪费系统内存。解决方案是在退出时将一个任务的子进程重新设定父进程给当前线程组中的另一个进程，如果失败，则设定给 init 进程。`do_exit()` 调用 `exit_notify()`，后者调用 `forget_original_parent()`，该函数又调用 `find_new_reaper()` 来执行重新设定父进程的操作：

```c
static struct task_struct *find_new_reaper(struct task_struct *father)
{
    struct pid_namespace *pid_ns = task_active_pid_ns(father);
    struct task_struct *thread;

    thread = father;
    while_each_thread(father, thread) {
        if (thread->flags & PF_EXITING)
            continue;
        if (unlikely(pid_ns->child_reaper == father))
            pid_ns->child_reaper = thread;
        return thread;
    }
    ...
    return pid_ns->child_reaper; // 通常是 init 进程
}
```
（代码已简化，核心逻辑是查找同一线程组内未退出的线程，或最终返回 init 进程）

这段代码尝试在进程的线程组中查找并返回另一个任务。如果线程组中没有其他任务，它就查找并返回 init 进程。找到合适的新父进程后，需要找到每个子进程并将其重新设定父进程给这个新的“收割者”(reaper)：

```c
reaper = find_new_reaper(father);
list_for_each_entry_safe(p, n, &father->children, sibling) {
    p->real_parent = reaper;
    if (p->parent == father) {
        BUG_ON(p->ptrace);
        p->parent = p->real_parent;
    }
    reparent_thread(p, father);
}
```

然后调用 `ptrace_exit_finish()` 做同样的事情，但是针对一个被 ptrace 跟踪 (ptraced) 的子进程列表。

同时拥有一个子进程列表 (child list) 和一个被跟踪子进程列表 (ptraced list) 的基本原理很有趣；这是 2.6 内核的一个新特性。当一个任务被 `ptrace` 跟踪时，它被临时重新设定父进程给调试进程。然而，当该任务的原始父进程退出时，它必须与其兄弟姐妹一起被重新设定父进程。在之前的内核中，这会导致需要循环遍历系统中的每个进程来查找子进程。解决方案就是简单地维护一个进程被 `ptrace` 跟踪的子进程的独立列表——将查找子进程的范围从系统中的每个进程缩小到仅仅两个相对较小的列表。

随着进程成功被重新设定父进程，就不再存在 stray zombie processes（ stray zombie processes）的风险。init 进程会例行地对其子进程调用 `wait()`，清理分配给它的任何僵尸进程。

