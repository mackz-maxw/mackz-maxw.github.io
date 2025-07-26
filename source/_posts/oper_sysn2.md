---
title: 操作系统基础 | 系统编程常用设置2
categories: os basic
lang: zh-CN
---

## 可移植性问题（Portability Issues）

### 特性测试宏（Feature Test Macros）

- 系统调用和库函数 API 的行为受多种标准规范（如 The Open Group 的 Single UNIX Specification、BSD、System V Release 4 及其接口定义）约束。
- 为了让头文件只暴露符合某一标准的定义（如常量、函数原型等），可以在编译时定义一个或多个特性测试宏。定义方式有两种：
  1. 在源代码中包含头文件前定义宏：
     ```c
     #define _BSD_SOURCE 1
     ```
  2. 用编译器的 `-D` 选项定义：
     ```sh
     $ cc -D_BSD_SOURCE prog.c
     ```
- “特性测试宏”这个名字的由来是：实现会通过 `#if` 判断这些宏的值，决定头文件中哪些特性对应用可见。

#### 常用特性测试宏

这些宏由相关标准规定，适用于所有支持这些标准的系统：

- `_POSIX_SOURCE`  
  定义后暴露符合 POSIX.1-1990 和 ISO C (1990) 的定义。已被 `_POSIX_C_SOURCE` 取代。
- `_POSIX_C_SOURCE`  
  - 值为 1 时，效果同 `_POSIX_SOURCE`。
  - 值 ≥ 199309 时，暴露 POSIX.1b（实时）定义。
  - 值 ≥ 199506 时，暴露 POSIX.1c（线程）定义。
  - 值为 200112 时，暴露 POSIX.1-2001 基础规范（不含 XSI 扩展）。
  - 值为 200809 时，暴露 POSIX.1-2008 基础规范。
- `_XOPEN_SOURCE`  
  - 定义后暴露 POSIX.1、POSIX.2 和 X/Open (XPG4) 定义。
  - 值 ≥ 500 时，暴露 SUSv2（UNIX 98 和 XPG5）扩展。
  - 值 ≥ 600 时，暴露 SUSv3 XSI（UNIX 03）和 C99 扩展。
  - 值 ≥ 700 时，暴露 SUSv4 XSI 扩展。

#### glibc 特有的特性测试宏

- `_BSD_SOURCE`  
  定义后暴露 BSD 定义，同时定义 `_POSIX_C_SOURCE=199506`。如只定义此宏，部分标准冲突时优先 BSD 定义。
- `_SVID_SOURCE`  
  定义后暴露 System V 接口定义（SVID）。
- `_GNU_SOURCE`  
  定义后暴露所有上述宏的定义及 GNU 扩展。

#### 默认行为与宏组合

- 默认情况下，GNU C 编译器会定义 `_POSIX_SOURCE`、`_POSIX_C_SOURCE=200809`（或更早版本的 200112/199506）、`_BSD_SOURCE` 和 `_SVID_SOURCE`。
- 如果单独定义了某些宏，或用标准模式（如 `cc -ansi` 或 `cc -std=c99`）编译，则只暴露请求的定义。
- 多个宏可以叠加定义。例如：
  ```sh
  $ cc -D_POSIX_SOURCE -D_POSIX_C_SOURCE=199506 -D_BSD_SOURCE -D_SVID_SOURCE prog.c
  ```
- `<features.h>` 头文件和 `feature_test_macros(7)` 手册页有详细说明。

#### POSIX.1/SUS 相关宏

- POSIX.1-2001/SUSv3 只规定了 `_POSIX_C_SOURCE` 和 `_XOPEN_SOURCE` 两个宏，要求值分别为 200112 和 600。
- POSIX.1-2008/SUSv4 要求值分别为 200809 和 700。
- 设置 `_XOPEN_SOURCE=600` 应包含 `_POSIX_C_SOURCE=200112` 的所有特性，SUSv4 也有类似要求。

#### 示例代码与函数原型中的特性测试宏

- 手册页会说明使用某个常量或函数声明时需要定义哪些特性测试宏。
- 本书示例代码可用默认 GNU C 编译器选项或如下方式编译：
  ```sh
  $ cc -std=c99 -D_XOPEN_SOURCE=600
  ```
- 书中每个函数原型都会注明需要定义哪些特性测试宏。
- 手册页有更详细的宏需求说明。


### 系统数据类型（System Data Types）

在 UNIX 系统中，许多实现相关的数据类型（如进程ID、用户ID、文件偏移量等）都用标准 C 类型来表示。虽然可以直接用 int、long 等基本类型声明这些变量，但这样会降低程序的可移植性，原因包括：

- 不同 UNIX 实现中基本类型的大小可能不同（如 long 在某些系统上是4字节，在另一些系统上是8字节），甚至同一系统的不同编译环境也可能不同。
- 不同实现可能用不同类型表示相同的信息。例如，进程ID在某些系统上是 int，在另一些系统上是 long。
- 同一实现的不同版本也可能改变类型定义。例如，Linux 2.2 及以前用户和组ID是16位，2.4及以后是32位。

为避免这些移植性问题，SUSv3（Single UNIX Specification, Version 3）规定了一系列标准系统数据类型，并要求实现时正确使用这些类型。这些类型通常用 C 的 typedef 定义。例如，pid_t 用于表示进程ID，在 Linux/x86-32 上定义为：
```c
typedef int pid_t;
```
大多数标准系统数据类型以 `_t` 结尾，通常声明在 `<sys/types.h>` 头文件中，部分类型在其他头文件中定义。

**建议：** 应用程序应使用这些类型来声明变量，以保证在所有符合 SUSv3 的系统上都能正确运行。例如：
```c
pid_t mypid;
```

#### 常用系统数据类型举例

| 数据类型      | 类型要求         | 说明                       |
| ------------- | --------------- | -------------------------- |
| pid_t         | 有符号整数      | 进程ID、进程组ID、会话ID   |
| uid_t         | 整数            | 用户ID                     |
| gid_t         | 整数            | 组ID                       |
| size_t        | 无符号整数      | 对象字节大小               |
| ssize_t       | 有符号整数      | 字节计数或错误指示         |
| off_t         | 有符号整数      | 文件偏移量或文件大小       |
| time_t        | 整数或实数      | 自 Epoch 起的秒数           |
| mode_t        | 整数            | 文件权限和类型             |
| dev_t         | 算术类型        | 设备号（主次设备号）        |
| ino_t         | 无符号整数      | 文件 i-node 号             |
| socklen_t     | 至少32位整数    | 套接字地址结构体大小       |


### 打印系统数据类型的数值

- 在用 `printf()` 打印表3-1中这些数值型系统数据类型（如 `pid_t`、`uid_t`）时，要避免实现相关的依赖问题。
- 由于 C 的参数提升规则，`short` 类型会被提升为 `int`，但 `int` 和 `long` 类型保持不变。因此，系统数据类型的底层实现不同，传递给 `printf()` 的参数类型可能是 `int` 或 `long`。
- 由于 `printf()` 在运行时无法判断参数类型，调用者必须用合适的格式说明符（如 `%d` 或 `%ld`）明确指定类型。但直接写死某个说明符会导致实现依赖。
- 通常的解决办法是统一用 `%ld`，并将对应的值强制转换为 `long`，例如：
  ```c
  pid_t mypid;
  mypid = getpid();  /* 获取当前进程ID */
  printf("My PID is %ld\n", (long) mypid);
  ```
- 有一个例外：`off_t` 类型在某些环境下是 `long long`，因此应强制转换为 `long long` 并用 `%lld` 打印（详见5.10节）。

- C99 标准定义了 `z` 长度修饰符，用于 `size_t` 或 `ssize_t` 类型，可以用 `%zd` 替代 `%ld`+强转。但该说明符并非所有 UNIX 实现都支持，所以本书避免使用。
- C99 还定义了 `j` 长度修饰符，指定参数为 `intmax_t`（或 `uintmax_t`），这种类型足够大，可以表示任何整数类型。理论上，使用 `(intmax_t)` 强转加 `%jd` 是最通用的做法，能处理 `long long` 及扩展整数类型（如 `int128_t`）。但由于并非所有 UNIX 实现都支持，本书也避免使用这种方式。

### 其他可移植性问题（Miscellaneous Portability Issues）

#### 结构体的初始化与使用

- 各 UNIX 实现规定了一系列标准结构体，用于系统调用和库函数。例如，`sembuf` 结构体用于信号量操作（semop）：
  ```c
  struct sembuf {
    unsigned short sem_num;  /* 信号量编号 */
    short          sem_op;   /* 要执行的操作 */
    short          sem_flg;  /* 操作标志 */
  };
  ```
- 虽然 SUSv3 规定了这些结构体，但需要注意：
  - 一般来说，结构体成员的顺序未必有标准规定。
  - 某些实现可能会在结构体中添加额外的字段。
- 因此，**不建议**用如下方式初始化结构体（因为不同实现成员顺序可能不同）：
  ```c
  struct sembuf s = { 3, -1, SEM_UNDO };
  ```
  这种写法在 Linux 下可用，但在其他实现中可能出错。**可移植的做法**是用显式赋值：
  ```c
  struct sembuf s;
  s.sem_num = 3;
  s.sem_op  = -1;
  s.sem_flg = SEM_UNDO;
  ```
  如果使用 C99，可以用新的结构体初始化语法：
  ```c
  struct sembuf s = { .sem_num = 3, .sem_op = -1, .sem_flg = SEM_UNDO };
  ```
- 如果要将结构体内容写入文件，也要注意成员顺序。不能直接二进制写入结构体，而应按指定顺序逐个字段写入（最好用文本形式）。

#### 某些宏可能并非所有实现都支持

- 有些宏在所有 UNIX 实现中并不一定存在。例如，`WCOREDUMP()` 宏（用于检测子进程是否产生 core dump 文件）虽然常见，但 SUSv3 并未规定，因此某些系统可能没有。
- 可移植的做法是用 `#ifdef` 判断宏是否存在：
  ```c
  #ifdef WCOREDUMP
    /* 使用 WCOREDUMP() 宏 */
  #endif
  ```

#### 不同实现对头文件的要求不同

- 某些系统调用和库函数所需的头文件在不同 UNIX 实现中可能不同。本书以 Linux 为主，并注明与 SUSv3 的差异。
- 书中部分函数原型会注明某个头文件后加注释 `/* For portability */`，表示该头文件在 Linux 或 SUSv3 下不是必需的，但为了兼容其他（尤其是老旧）实现，建议在可移植程序中包含。
- POSIX.1-1990 要求在包含与某些函数相关的头文件前，先包含 `<sys/types.h>`，但这一要求后来被 SUSv1 移除。尽管如此，为了可移植性，建议将 `<sys/types.h>` 作为首个头文件包含（本书示例为简洁起见省略了它）。

