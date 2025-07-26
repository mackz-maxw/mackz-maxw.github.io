---
title: 操作系统基础 | 系统编程常用设置
categories: os basic
lang: zh-CN
---

## LPI示例程序说明

### 命令行选项与参数

- 本书中的许多示例程序依赖命令行选项和参数来决定其行为。
- 传统 UNIX 命令行选项格式为：一个连字符（-）加一个字母，后面可跟参数。GNU 工具支持扩展格式：两个连字符（--）加选项名和可选参数。
- 示例程序通常使用标准库函数 `getopt()` 解析命令行选项（详见附录B）。
- 只要程序的命令行语法不简单，都会实现一个帮助功能：如果用 `--help` 选项运行，程序会显示用法说明，指明命令行选项和参数的语法。

### 公共函数与头文件

- 大多数示例程序都包含一个公共头文件，定义常用类型和宏，并引用常用的库函数和系统调用声明，使代码更简洁。

#### 公共头文件（lib/tlpi_hdr.h）

- 该头文件包含了许多常用头文件，定义了布尔类型和求最小/最大值的宏。例如：

  ```c
  #ifndef TLPI_HDR_H
  #define TLPI_HDR_H  /* 防止重复包含 */
  #include <sys/types.h>  /* 常用类型定义 */
  #include <stdio.h>      /* 标准I/O函数 */
  #include <stdlib.h>
  #include <unistd.h>
  #include <errno.h>
  #include <string.h>
  #include "get_num.h"    /* 常用数值处理函数声明 */
  #include "error_functions.h"  /* 错误处理函数声明 */
  typedef enum { FALSE, TRUE } Boolean;
  #define min(m,n) ((m) < (n) ? (m) : (n))
  #define max(m,n) ((m) > (n) ? (m) : (n))
  #endif
  ```

#### 错误诊断函数（lib/error_functions.h）

- 为简化错误处理，示例程序使用一组通用的错误诊断函数，其声明如下：

  ```c
  #ifndef ERROR_FUNCTIONS_H
  #define ERROR_FUNCTIONS_H
  void errMsg(const char *format, ...);
  #ifdef __GNUC__
  #define NORETURN __attribute__ ((__noreturn__))
  #else
  #define NORETURN
  #endif
  void errExit(const char *format, ...) NORETURN ;
  void err_exit(const char *format, ...) NORETURN ;
  void errExitEN(int errnum, const char *format, ...) NORETURN ;
  void fatal(const char *format, ...) NORETURN ;
  void usageErr(const char *format, ...) NORETURN ;
  void cmdLineErr(const char *format, ...) NORETURN ;
  #endif
  ```

#### 使用 errMsg()、errExit()、err_exit() 和 errExitEN() 诊断系统调用和库函数错误

```c
#include "tlpi_hdr.h"
void errMsg(const char *format, ...);
void errExit(const char *format, ...);
void err_exit(const char *format, ...);
void errExitEN(int errnum, const char *format, ...);
```

- **errMsg()**  
  在标准错误输出错误信息。参数列表与 printf() 相同，输出末尾自动加换行。会输出当前 errno 对应的错误文本（如错误名 EPERM 和 strerror() 返回的描述），再加上格式化输出内容。

- **errExit()**  
  功能类似 errMsg()，但会终止程序。终止方式为调用 exit()，如果环境变量 EF_DUMPCORE 被设置为非空字符串，则调用 abort() 生成 core dump 文件（用于调试）。

- **err_exit()**  
  与 errExit() 类似，但有两点不同：
  1. 打印错误信息前不会刷新标准输出。
  2. 通过 _exit() 终止进程，而不是 exit()，这样不会刷新 stdio 缓冲区，也不会调用 exit 处理函数。
  这种方式适合在库函数中创建子进程后，因错误需要立即终止子进程时使用，避免影响父进程的缓冲区和退出处理。

- **errExitEN()**  
  与 errExit() 类似，但输出的是参数 errnum 指定的错误号对应的错误文本（EN 代表 Error Number），而不是当前 errno 的内容。主要用于 POSIX 线程 API（pthread）相关的程序。
  - 传统 UNIX 系统调用出错返回 –1，POSIX 线程函数出错则直接返回错误号（正整数），成功返回 0。
  - 示例：
    ```c
    int s;
    s = pthread_create(&thread, NULL, func, &arg);
    if (s != 0)
      errExitEN(s, "pthread_create");
    ```
  - 这样比直接用 errno 更高效，因为在多线程程序中 errno 是一个宏，会展开为函数调用，返回线程私有的存储区域指针。

- **lvalue（左值） 说明**  
  lvalue 是指向存储区域的表达式，最常见的是变量名。某些操作符也能产生 lvalue，比如指针解引用 *p。在 POSIX 线程 API 下，errno 被重定义为返回线程私有存储区指针的函数。


#### 诊断其他类型错误的函数

```c
#include "tlpi_hdr.h"
void fatal(const char *format, ...);
void usageErr(const char *format, ...);
void cmdLineErr(const char *format, ...);
```

- **fatal()**  
  用于诊断一般性错误，包括那些不会设置 errno 的库函数错误。参数列表与 printf() 相同，输出自动换行。该函数会将格式化信息输出到标准错误，并像 errExit() 一样终止程序。

- **usageErr()**  
  用于诊断命令行参数用法错误。参数同 printf()，输出以 "Usage:" 开头，后跟格式化内容，输出到标准错误，然后调用 exit() 终止程序。（有些示例程序会用扩展版 usageError()。）

- **cmdLineErr()**  
  类似于 usageErr()，但用于诊断命令行参数本身的错误。输出以 "Command-line usage error:" 开头，后跟格式化内容，输出到标准错误并终止程序。


#### 错误处理函数实现说明

- 错误处理函数的实现会用到 `ename.c.inc` 文件，该文件定义了一个字符串数组 `ename`，用于将 errno 错误号映射为符号名（如 EPERM、EAGAIN/EWOULDBLOCK 等）。
- 这样做的好处是：strerror() 只返回错误描述，不包含符号名，而手册页用符号名描述错误。输出符号名便于查阅手册定位错误原因。
- `ename.c.inc` 文件内容与硬件架构相关，不同平台 errno 值可能不同。可以用书中提供的脚本（lib/Build_ename.sh）为特定平台生成合适的版本。
- `ename` 数组中有些字符串为空，对应未使用的错误号；有些字符串包含两个错误名（如 "EAGAIN/EWOULDBLOCK"），表示这两个符号对应同一个错误号。
- 例如，EAGAIN 和 EWOULDBLOCK 在大多数 UNIX 系统上值相同，分别用于 System V 和 BSD 的不同场景。SUSv3 规范允许非阻塞调用返回这两个错误之一。

#### 解析数字型命令行参数的函数

- 头文件（如清单3-5）声明了两个常用来解析整型命令行参数的函数：`getInt()` 和 `getLong()`。
- 与 `atoi()`、`atol()`、`strtol()` 等标准函数相比，这两个函数的主要优点是能对数字参数进行基本有效性检查。
- `getInt()` 和 `getLong()` 分别将参数 `arg` 指向的字符串转换为 int 或 long 类型。如果 `arg` 不是有效的整数字符串（即只包含数字和 +、- 号），函数会输出错误信息并终止程序。

```c
#include "tlpi_hdr.h"
int getInt(const char *arg, int flags, const char *name);
long getLong(const char *arg, int flags, const char *name);
// 返回 arg 转换后的数值
```

- 如果 `name` 参数非 NULL，应传入一个字符串，用于标识 arg 参数。该字符串会包含在错误信息中，便于定位问题。
- `flags` 参数用于控制 `getInt()` 和 `getLong()` 的行为。默认情况下，这两个函数期望参数为有符号十进制整数。通过将一个或多个 GN_* 常量（见清单3-5）按位或（|）赋给 flags，可以选择不同的进制或限制数值范围（如只允许非负数或大于0）。
- 这两个函数的实现见清单3-6。

- 虽然 flags 参数可以强制范围检查，但在某些示例程序中我们并未启用这些检查。例如，在清单47-1中，未检查信号量初始值参数，用户可以输入负数，导致后续 semctl() 系统调用出错（ERANGE），因为信号量不能为负。省略范围检查有助于实验系统调用和库函数的正确与错误用法，便于学习。实际应用中通常会对命令行参数做更严格的检查。

