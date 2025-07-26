---
title: 操作系统基础 | 2. 初试linux内核
categories: os basic
lang: zh-CN
---

## 内核源码相关

### 获取内核源码  
当前的 Linux 源代码总是可以在 [http://www.kernel.org](http://www.kernel.org) 官方网站上以完整的 tarball（用 tar 命令创建的归档文件）和增量补丁的形式获得。
可以利用[Elixir Cross Referencer](https://elixir.bootlin.com/linux/v5.10.17/source)网站在线查看源码

#### 使用 Git  
你可以用 Git 获取 Linus 主线最新的源码树：
```sh
git clone git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux-2.6.git
```
检出后，可以用如下命令更新你的源码树到 Linus 的最新版本：
```sh
git pull
```

#### 安装内核源码  
内核 tarball 以 GNU zip（gzip）和 bzip2 两种格式发布。bzip2 是默认且推荐的格式，因为它通常压缩得更好。bzip2 格式的内核包名为 linux-x.y.z.tar.bz2，其中 x.y.z 是内核版本号。下载源码后，解压和解包很简单。如果你的 tarball 是 bzip2 压缩的，运行：
```sh
tar xvjf linux-x.y.z.tar.bz2
```
如果是 gzip 压缩的，运行：
```sh
tar xvzf linux-x.y.z.tar.gz
```
这会将源码解压到 linux-x.y.z 目录。如果你用 git 获取源码，就不需要下载 tarball，只需运行 git clone，Git 会自动下载并解包最新源码。

#### 源码安装与开发位置  
内核源码通常安装在 `/usr/src/linux`。但你不应该在这个目录下开发，因为你的 C 库可能会链接到这里的内核版本。此外，修改内核源码不应需要 root 权限——建议在你的 home 目录下开发，只在安装新内核时用 root。即使安装新内核，也不要动 `/usr/src/linux`。

#### 使用补丁  
在 Linux 内核社区，补丁是交流的通用语言。你会以补丁的形式分发你的代码更改，也会以补丁的形式接收别人的代码。增量补丁可以让你轻松地从一个内核版本升级到下一个，无需每次都下载完整的大包，只需应用增量补丁即可，节省带宽和时间。要应用增量补丁，在内核源码目录下运行：
```sh
patch -p1 < ../patch-x.y.z
```
通常，补丁是针对前一个版本的源码生成的。

### 内核源码树结构简介
Linux 内核源码树（source tree）被划分为多个目录，每个目录下又包含许多子目录。下表列出了源码树根目录下的主要目录及其说明：

| 目录（Directory） | 说明（Description）                       |
|-------------------|-------------------------------------------|
| arch              | 架构相关源码（不同CPU架构的实现）         |
| block             | 块设备I/O层                               |
| crypto            | 加密API                                   |
| Documentation     | 内核源码文档                              |
| drivers           | 设备驱动                                  |
| firmware          | 某些驱动需要用到的设备固件                |
| fs                | 虚拟文件系统（VFS）及各类文件系统实现      |
| include           | 内核头文件                                |
| init              | 内核启动与初始化代码                      |
| ipc               | 进程间通信代码                            |
| kernel            | 核心子系统（如调度器等）                  |
| lib               | 辅助函数库                                |
| mm                | 内存管理子系统及虚拟内存                  |
| net               | 网络子系统                                |
| samples           | 示例和演示代码                            |
| scripts           | 构建内核用的脚本                          |
| security          | Linux安全模块                             |
| sound             | 声音子系统                                |
| usr               | 早期用户空间代码（如initramfs）           |
| tools             | 内核开发相关工具                          |
| virt              | 虚拟化基础设施                            |

#### 源码树根目录下的一些文件
- **COPYING**：内核许可证（GNU GPL v2）。
- **CREDITS**：内核主要开发者名单。
- **MAINTAINERS**：各子系统和驱动的维护者名单。
- **Makefile**：内核主 Makefile，用于编译和构建整个内核。

## 配置内核（Configuring the Kernel）

因为 Linux 源码是开放的，所以你可以在编译前根据自己的需求进行配置和定制。实际上，你可以只为你需要的功能和驱动编译支持。**配置内核**是编译前的必经步骤。由于内核功能丰富、支持的硬件种类繁多，配置选项也非常多。

### 内核配置选项（Configuration Options）

内核配置通过一系列以 `CONFIG_` 开头的选项控制，例如 `CONFIG_SMP` 控制对称多处理（SMP）支持。设置该选项即启用 SMP，未设置则禁用。配置选项既决定编译哪些文件，也通过预处理指令影响源码。

- **布尔型（Boolean）**：只有 yes 或 no 两种状态。比如 `CONFIG_PREEMPT`。
- **三态（Tristate）**：yes、no 或 module。module 表示编译为可动态加载的模块（.ko 文件）；yes 表示直接编译进内核镜像。
- **字符串或整数**：用于指定某些参数值，比如数组大小，这些不会影响编译流程，而是作为宏被源码访问。

### 发行版内核与自定义内核

各大 Linux 发行版（如 Ubuntu、Fedora）自带的内核都是预编译好的，通常会启用大部分常用功能，并把绝大多数驱动编译为模块，以便支持各种硬件。但如果你想深入学习或开发内核，还是需要自己编译内核，并选择合适的模块。

### 配置工具

内核提供了多种配置工具：

- **文本命令行工具**  
  ```sh
  make config
  ```
  逐项询问每个选项，适合有耐心的用户。

- **基于 ncurses 的图形界面**  
  ```sh
  make menuconfig
  ```
  推荐使用，界面友好，选项分门别类。

- **基于 GTK+ 的图形界面**  
  ```sh
  make gconfig
  ```
  适合喜欢图形界面的用户。

这些工具会把配置选项分为不同类别（如“处理器类型与特性”），你可以浏览、修改各项配置。

### 快速生成默认配置

如果你不想从零开始配置，可以用默认配置：
```sh
make defconfig
```
这会生成一个适合你当前架构的默认配置（比如 i386 下据说是 Linus 的配置），适合新手快速上手。之后可以再根据自己的硬件调整配置。

### 配置文件的位置与管理

所有配置选项最终会保存在源码根目录下的 `.config` 文件中。你也可以直接编辑这个文件（很多内核开发者都这么做），只需搜索并修改对应的配置项即可。

如果你用现有的 `.config` 文件，或者升级到新内核后想沿用旧配置，可以用：
```sh
make oldconfig
```
它会根据新内核的选项补全或更新你的配置。

### 复制当前内核配置

如果当前内核启用了 `CONFIG_IKCONFIG_PROC`，你可以直接从 `/proc/config.gz` 拷贝当前内核的配置：
```sh
zcat /proc/config.gz > .config
make oldconfig
```
这样可以方便地克隆当前系统的内核配置。

## 编译内核

配置好后，只需一条命令即可编译内核：
```sh
make
```
自 2.6 版本起，无需再手动运行 `make dep` 或分别编译 bzImage、modules，默认的 Makefile 规则会自动处理所有依赖和构建流程。


### 降低编译输出噪音（Minimizing Build Noise）

在编译内核时，终端会输出大量信息。为了减少这些“噪音”，但又能看到警告和错误，可以将 `make` 的标准输出重定向到文件：

```sh
make > ../detritus
```

如果需要查看详细输出，可以阅读该文件。由于警告和错误信息会输出到标准错误（stderr），通常你不需要关心标准输出。实际上，你可以直接把输出丢到“黑洞”：

```sh
make > /dev/null
```

这样所有无用的输出都会被丢弃，只在终端显示警告和错误。

---

### 并行编译（Spawning Multiple Build Jobs）

`make` 支持并行编译，可以同时运行多个编译任务，大大加快多核系统上的编译速度。方法如下：

```sh
make -jN
```

其中 `N` 是并行任务数。通常建议每个处理器核心分配1~2个任务。例如，16核机器可以这样：

```sh
make -j32 > /dev/null
```

此外，使用如 `distcc` 或 `ccache` 等工具也能进一步提升编译速度。

---

## 安装新内核（Installing the New Kernel）

内核编译完成后，需要安装。安装方式依赖于你的硬件架构和引导程序（boot loader），请查阅对应引导程序的文档。

**以 x86 + grub 为例：**

1. 将 `arch/i386/boot/bzImage` 复制到 `/boot`，并命名为如 `vmlinuz-version`。
2. 编辑 `/boot/grub/grub.conf`，为新内核添加启动项。

**如果使用 LILO：**

1. 编辑 `/etc/lilo.conf`，添加新内核项。
2. 重新运行 `lilo` 命令。

### 安装模块（Installing Modules）

模块的安装是自动且与架构无关的。只需以 root 权限运行：
```sh
make modules_install
```
这会把所有编译好的模块安装到 `/lib/modules` 下的对应目录。

### System.map 文件

编译过程中还会在源码根目录生成 `System.map` 文件。它是一个符号查找表，用于将内核符号映射到内存地址，在调试时可以把地址转换为函数或变量名。

