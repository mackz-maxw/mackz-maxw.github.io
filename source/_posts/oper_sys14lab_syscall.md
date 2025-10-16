---
title: 操作系统基础 | 3.4 系统调用实验
categories: 
  - os basic
  - lab
lang: zh-CN
---

### **必做练习**
#### **练习1**  
准备好实验报告

#### **练习2**  
我们将通过 libc 包装器发起系统调用（标准用户程序调用内核的方式）。系统调用完整列表可通过 `man 2 syscalls` 查看手册页。  
1. 启动树莓派，在终端中**脱离 Linux 源码目录**新建用户程序目录   
2. 创建文件 `lib_call.c`，编写 C 程序：  
   - 使用库函数 `getuid` 读取并打印用户 ID  
   - 尝试用 `setuid` 将 ID 设为 0（root）并报告是否成功  
   - 再次读取并打印用户 ID  
3. 通过 `man 2 getuid` 和 `man 2 setuid` 查阅：  
   - 函数调用语法和返回值类型  
   - 编译所需的头文件  
4. 错误检查：若 `setuid` 失败，输出错误原因（使用 `strerror(errno)`）。  
   - 相关手册页：`man 3 printf`, `man 3 strerror`, `man 3 errno`  
5. 在实验机上编译运行：  

   ```bash
   gcc lib_call.c -o lib_call && ./lib_call
   ```

#### **练习3**  
从树莓派用户程序目录执行：  
1. 通过 sftp 从 `shell.cec.学校简称.edu` 获取 `lib_call.c`  
2. 在树莓派上编译并运行程序（分别用普通用户和 `sudo` 权限运行）

#### **练习4**  
修改程序使用原生系统调用接口（`man 2 syscall`）：  
1. 复制 `lib_call.c` 为 `native_call.c`  
2. 将 `getuid/setuid` 替换为 `syscall`  
3. 通过 Linux 源码确定 ARM 系统调用号：  
   - 查看 `arch/arm/include/uapi/asm/unistd.h`  
   - 查看生成的 `arch/arm/include/generated/uapi/asm/unistd-common.h`  
4. **使用常量**（如 `__NR_getuid`）而非硬编码数字  
5. 可能需包含头文件 `asm/unistd.h`  
6. 在树莓派上编译运行  

#### **练习5**  
为内核添加两个新系统调用（各打印一条内核日志）：  
**任务清单**  
1. 在内核中声明函数原型  
2. 实现函数逻辑  
3. 分配系统调用号  
4. 更新 ARM 系统调用表  
5. 更新系统调用总数  

**操作步骤**  
1. 登录用于交叉编译的Linux主机 → 进入 Linux 源码目录  
2. **修改前备份文件**（如 `cp syscalls.h syscalls.h.092125`）  

**声明函数原型**（在 `include/linux/syscalls.h` 底部添加）：  

```c
// 092125
asmlinkage long sys_noargs(void); 
asmlinkage long sys_onearg(int arg);
```

#### **练习6**  
**实现系统调用函数**：  
1. 在 `arch/arm/kernel/` 创建文件：  
   - `sys_noargs.c`（无参数函数）  
   - `sys_onearg.c`（带参数函数）  
2. `sys_noargs.c` 内容参考
   
   ```c
   #include <linux/kernel.h>
   #include <linux/init.h>
   #include <linux/sched.h>
   #include <linux/syscalls.h>

   //这个宏定义了一个不携带参数的系统调用，它定义于syscalls.h
   SYSCALL_DEFINE0( noargs ){
   // print out a simple message indicating the function was called, and return SUCCESS
   printk("Someone invoked the sys_noargs system call");
   return 0;  
   }
   ``` 

3. `sys_onearg.c` 模板：  
   ```c
   //这个宏定义了携带一个参数的系统调用，它定义于syscalls.h
   SYSCALL_DEFINE1(onearg, int, arg) {
       printk("Received argument: %d\n", arg); 
       return 0; // 返回适当值
   }
   ```

#### **练习7**  
**添加文件到构建系统**：  
修改 `arch/arm/kernel/Makefile`：  
1. 备份原文件(例如`Makefile.092125`)
2. 在 `obj-y` 列表末尾添加（**不在 `\` 后**）：  
   ```makefile
   sys_noargs.o sys_onearg.o
   ```


#### **练习8**  
**更新系统调用表**：  
修改 `arch/arm/tools/syscall.tbl`：  
1. 备份原文件（ 例如 `syscall.tbl.092125` ）
2. 在末尾添加（分配新调用号）：  
   ```plaintext
   (调用号)  ...  （系统调用名称）    （实际调用的函数名）
   ```

---

#### **练习9**  
**编译并验证新内核**：  
1. 在 Linux 源码根目录执行：  
   ```bash
   make clean
   ```
2. 修改本地版本标识：  
   - 编辑 `.config` 文件  
   - 更新 `CONFIG_LOCALVERSION` 值（如 `-syscallstudio`）  
3. 按实验1/2的步骤编译安装内核  
4. 树莓派上运行：  
   ```bash
   uname -a
   ```

---

#### **练习10**  
**调用新系统调用**：  
1. 通过 sftp 获取 `arch/arm/include/generated/uapi/asm/unistd-common.h`  
2. 复制到 GCC 头文件目录：  
   ```bash
   sudo cp unistd-common.h /usr/include/arm-linux-gnueabihf/asm
   ```
3. 创建 `new_call.c`（基于 `native_call.c`）：  
   - 将 `getuid` 调用号替换为 `__NR_...`  
   - 将 `setuid` 调用号替换为 `__NR_...`  
   - 为单参数调用声明变量（勿直接传常量）  
4. 编译运行后检查内核日志：  
   ```bash
   dmesg
   ```

---

### **可选拓展练习**  
完成任意拓展练习后简述收获（附问题答案）。

**练习1**  
用汇编接口实现系统调用（参考课堂 ARM 调用流程），使用 GCC 内联汇编（`asm` 扩展）。  
> 提示：详细流程见 `man 2 syscall`，内核入口代码见 `arch/arm/kernel/entry-common.S`。

**练习2**  
实现读取周期计数器（CCNT）的系统调用：  
1. 下载[性能监控头文件](https://example.com/perf_regs.h) 至 `arch/arm/include/asm`  
2. 声明函数原型：`asmlinkage long sys_read_ccnt(unsigned long long *val)`  
3. 创建 `sys_read_ccnt.c`（参考[模板](https://example.com/sys_read_ccnt.c)）  
4. 更新 `Makefile` 和 `syscall.tbl`  
5. 编写程序连续调用两次，计算调用耗时（周期数）  

**练习3**  
修复 `read_ccnt` 的安全缺陷：  
1. 使用 `copy_to_user()` 替代直接写入用户指针  
2. 失败时返回 `-EFAULT`  

