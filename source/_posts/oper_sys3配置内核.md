---
title: 操作系统基础 | 2.3 初试linux内核实验
categories: 
  - os basic
  - lab
lang: zh-CN
---

## 树莓派 Linux 内核源码下载与编译流程

### 1. 下载适用于树莓派的内核源码

一般项目可以直接去 [kernel.org](https://kernel.org) 下载 Linux 源码，但本课程针对树莓派，需要用树莓派官方维护的内核版本（在 [https://github.com/raspberrypi](https://github.com/raspberrypi)）。

在你的 `/project/scratch01/compile/user-name/` 目录下，新建 `linux_source` 文件夹用于存放源码和编译文件：

```sh
mkdir linux_source
cd linux_source
```

下载指定版本的树莓派内核源码（此过程可能需要20-30分钟）：

```sh
wget https://github.com/raspberrypi/linux/archive/raspberrypi-kernel_1.20210527-1.tar.gz
```

解压源码包：

```sh
tar -xzf raspberrypi-kernel_1.20210527-1.tar.gz
```

解压后会得到一个新目录，建议用 `mv` 命令重命名为 `linux`，便于后续操作。解压完成后请删除 `.tar.gz` 文件以节省空间。

进入 `linux` 目录，运行以下命令查看内核版本：

```sh
make kernelversion
```

并用文本编辑器（如 emacs、vim、nano）打开 `Makefile`，查看前几行定义的内核版本常量，记录 `NAME` 常量的值。

---

### 2. 针对树莓派 4/4B 的设备树修改

如果你使用的是 Raspberry Pi 4 或 4B，需要修改设备树文件 `arch/arm/boot/dts/bcm2711.dtsi`，找到 `arm-pmu` 条目，将 `compatible` 行改为：

```c
compatible = "arm,cortex-a72-pmu", "arm,cortex-a15-pmu", "arm,armv8-pmuv3";
```

---

### 3. 配置交叉编译环境

添加交叉编译器和新版 gcc 到 PATH（并将以下两行添加到 `~/.bashrc` 文件末尾，确保下次登录自动生效）：

```sh
module add arm-rpi
module add gcc-8.3.0
```

---

### 4. 配置内核

对于 Raspberry Pi 3B+，运行：

```sh
make -j8 ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- bcm2709_defconfig
```

对于 Raspberry Pi 4/4B，运行：

```sh
make -j8 ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- bcm2711_defconfig
```

这会生成树莓派的默认内核配置。

---

### 5. 自定义内核配置

进入菜单配置界面：

```sh
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- menuconfig
```

- 在 "General setup" -> "Local version" 里，添加你的唯一标识（如 `-v7` 或 `-v7l` 后面加你的名字，无空格）。
- 修改 "Preemption Model" 选项为 "Preemptible Kernel (Low-Latency Desktop)"，以获得更低延迟的抢占模型。
- 启用 ARM 性能监控单元驱动（"Kernel Performance Events and Counters"），并确保 "Profiling support" 也已启用。
- 任选一个有趣的选项，按 `H` 键查看简介，记录该选项的名称、简介和符号（symbol），并简述为何选择 "Preemptible Kernel (Low-Latency Desktop)"（提示：该模式适合需要低延迟响应的场景，如桌面或实时应用）。

保存并退出配置。

---

### 6. 编译内核

记录编译开始和结束时间：

```sh
date>>time.txt; make -j8 ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- zImage modules dtbs; date>>time.txt
```

编译完成后，创建用于存放模块的目录：

```sh
mkdir ../modules
```

安装内核模块：

```sh
make -j8 ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- INSTALL_MOD_PATH=../modules modules_install
```

---

### 7. 回答与说明

- 用 `cat time.txt` 查看编译所用时间。
- 说明为何要用交叉编译器：因为 linuxlab 服务器的架构与树莓派不同，必须用交叉编译器生成适用于 ARM 架构的内核和模块。

---

**总结**  
本流程涵盖了树莓派专用 Linux 内核源码的下载、解压、配置、定制、编译和模块安装，并介绍了如何设置交叉编译环境和设备树修改。通过这些步骤，你可以为树莓派编译和定制属于自己的 Linux 内核。