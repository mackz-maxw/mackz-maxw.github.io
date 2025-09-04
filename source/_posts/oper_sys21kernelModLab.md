---
title: 操作系统基础 | 4.7 内核模块实验
categories: 
  - os basic
  - lab
lang: zh-CN
---

### 可加载的内核模块

#### 练习 1
准备好实验报告

---

#### 练习 2: 编译内核模块
1.  **步骤**:
    ```bash
    # 在 Linux Lab 集群上
    mkdir -p /project/scratch01/compile/your-username/modules
    cd /project/scratch01/compile/your-username/modules
    # 保存 simple_module.c 和 Makefile (内容: obj-m := simple_module.o)
    module add arm-rpi
    export LINUX_SOURCE=/path/to/your/linux_source/linux # 设置内核源码路径
    make -C $LINUX_SOURCE ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- M=$PWD modules
    ```
2.  **答案**: 提交 `make` 命令的完整输出。

---

#### 练习 3: 加载模块与系统日志
1.  **步骤** (在树莓派上):
    ```bash
    mkdir ~/modules
    # 使用 sftp 将 simple_module.ko 传输到此目录
    sudo dmesg --clear
    sudo insmod ~/modules/simple_module.ko
    dmesg # 查看日志
    ```
2.  **答案**: 提交 `dmesg` 中显示的模块加载信息。

---

#### 练习 4: 验证模块列表与卸载
1.  **步骤**:
    ```bash
    lsmod # 确认 simple_module 在列表中
    sudo rmmod simple_module
    lsmod # 确认已移除
    dmesg # 查看卸载信息
    ```
2.  **答案**: 提交 `lsmod` 的输出（证明已卸载）和 `dmesg` 中显示的模块卸载信息。

---

#### 练习 5: 访问内核变量 jiffies
1.  **步骤**:
    *   复制 `simple_module.c` 为 `jiffies_module.c`。
    *   修改其 init 和 exit 函数，使用 `printk` 打印 `jiffies` 变量（类型 `extern unsigned long volatile jiffies;`）。
    *   更新 Makefile: `obj-m += jiffies_module.o`。
    *   重新编译并传输 `jiffies_module.ko` 到树莓派。
    *   加载并卸载模块，观察 `dmesg`。
2.  **答案**:
    *   提交 `dmesg` 中显示加载和卸载时 `jiffies` 值的日志消息。
    *   计算并说明两条消息之间发生的滴答数（tick count）。

---

### 需提交的内容
请提交一个包含以下内容的压缩包或文档：
1.  **答案文件**: 包含上述所有练习的答案。
2.  **源代码文件**: 你新创建的 `jiffies_module.c` 文件。

---

### 可选拓展练习

#### 练习 6: 模块初始化返回值实验
*   **任务**: 修改 init 函数，使其分别返回正数和负数（错误码，参见 `/include/uapi/asm-generic/errno-base.h`）。
*   **提示**: 描述加载模块时发生的情况及其在系统日志中的表现。

#### 练习 7: 探查导出的内核符号
*   **任务**: 查看 `/proc/kallsyms` 文件（例如 `cat /proc/kallsyms`），了解内核符号表。查找带有 `__kstrtab_` 和 `__ksymtab_` 前缀的符号（这些是可供模块使用的导出符号）。
*   **提示**: 内核符号是 Linux 内核代码中定义的函数、变量、数据结构等的名称标签。它们代表了在内核地址空间中的一个特定内存地址。
可以将内核符号理解为内核的“公共接口”或“入口点”。主要有两种类型：
- 导出的符号：这些是内核明确声明为可以被外部模块使用的符号。例如，printk（内核的 printf）、kmalloc（内核的内存分配函数）等。模块通过使用 EXPORT_SYMBOL() 或 EXPORT_SYMBOL_GPL() 宏来导出它们的符号，以便其他模块可以使用。
- 非导出的符号：这些是内核内部的静态函数或变量，只在它们被定义的文件或内核的特定部分中使用。它们对于内核模块是不可见的，主要用于内核自身的组织
- EXPORT_SYMBOL(printk) 在编译后创建了两个内部符号：__ksymtab_printk 和 __kstrtab_printk。
  - __ksymtab_printk 是一个结构体，包含了 printk 的地址和指向 __kstrtab_printk 的指针。
  - __kstrtab_printk 是一个字符串，存储着 printk 的名字。
- 内核使用这个结构来高效地通过名字查找函数的地址

#### 练习 8: 实现用户态读取 CPU 周期计数器 (CCNT) 的模块
*   **任务** (如果之前在系统调用工作室未完成):
    *   将提供的驱动文件放入你的内核源码树的 `arch/arm/include/asm` 目录。
    *   下载 `enable_ccnt.c` 内核模块代码到你的模块目录。
    *   编译、传输并加载此模块 (`insmod enable_ccnt.ko`)。
    *   使用 `dmesg | tail` 验证加载成功。
*   **任务** (主要部分):
    *   在树莓派上创建一个用户态程序。
    *   该程序应 `#include` 你获取的驱动头文件。
    *   调用 `unsigned long long pmccntr_get(void)` 函数两次。
*   **答案**:
    *   说明运行一次 `pmccntr_get()` 函数大约需要多少 CPU 周期（计算两次调用的差值）。
    *   如果你在系统调用工作室完成过类似的练习，请对比直接调用此函数与通过系统调用获取周期计数所需的周期数。