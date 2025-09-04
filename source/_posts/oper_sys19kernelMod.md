---
title: 操作系统基础 | 4.5 内核数据结构-内核模块
categories: os basic
lang: zh-CN
---

### Linux 内核模块开发简介

#### 设备类型分类
| **类型**       | **缩写** | **访问方式**               | **典型设备**               | **特殊文件**         |
|----------------|----------|---------------------------|---------------------------|---------------------|
| **块设备**     | blkdevs  | 按块随机访问（支持寻址）   | 硬盘/SSD/光驱             | `/dev/sda1`         |
| **字符设备**   | cdevs    | 字节流顺序访问             | 键盘/打印机/伪设备        | `/dev/ttyS0`        |
| **网络设备**   | -        | 套接字API（破坏"一切皆文件"原则） | 网卡/无线适配器           | 无设备节点          |
| **混杂设备**   | miscdevs | 字符设备简化形式           | 简单专用设备              | `/dev/random` 等    |

**伪设备示例**：
```bash
/dev/random   # 内核随机数生成器
/dev/null     # 空设备（丢弃所有写入）
/dev/zero     # 零设备（提供无限\0字节）
/dev/full     # 满设备（写入总返回ENOSPC错误）
/dev/mem      # 物理内存访问设备
```

---

### 内核模块开发

#### Hello World 模块示例
```c
#include <linux/init.h>
#include <linux/module.h>
#include <linux/kernel.h>

/* 模块加载时执行 */
static int __init hello_init(void) 
{
    printk(KERN_INFO "I bear a charmed life.\n"); 
    return 0;  // 返回0表示成功
}

/* 模块卸载时执行 */
static void __exit hello_exit(void) 
{
    printk(KERN_INFO "Out, out, brief candle!\n");
}

/* 注册入口/出口函数 */
module_init(hello_init);  // 不是函数调用，而是宏定义
module_exit(hello_exit);

/* 模块元信息 */
MODULE_LICENSE("GPL");                  // 必须声明许可证
MODULE_AUTHOR("Shakespeare");           // 作者信息
MODULE_DESCRIPTION("Hello World Module"); // 模块描述
```

#### 关键机制解析
1. **入口函数**：
   - 形式：`int init_func(void)`
   - 职责：注册资源/初始化硬件/分配数据结构
   - 返回值：0=成功，非0=失败

2. **出口函数**：
   - 形式：`void exit_func(void)`
   - 职责：释放资源/复位硬件/清理状态

3. **许可证声明**：
   ```c
   MODULE_LICENSE("GPL");  // 合法选项：GPL/MIT/BSD等
   ```
   - 非GPL模块会导致内核标记为"tainted"
   - 无法调用GPL-only符号

---

### 模块构建指南

#### 集成到内核源码树（推荐）
1. **选择路径**：
   - 字符设备 → `drivers/char/`
   - 块设备 → `drivers/block/`
   - USB设备 → `drivers/usb/`

2. **修改Makefile**：
   ```makefile
   # drivers/char/Makefile 添加
   obj-$(CONFIG_FISHING_POLE) += fishing/
   
   # drivers/char/fishing/Makefile 内容
   obj-$(CONFIG_FISHING_POLE) += fishing.o
   fishing-objs := main.o line.o  # 多文件模块
   EXTRA_CFLAGS += -DTITANIUM_POLE  # 自定义编译标志
   ```

#### 外部独立构建
1. **Makefile示例**：
   ```makefile
   obj-m := fishing.o
   fishing-objs := fishing-main.o fishing-line.o
   
   all:
       make -C /path/to/kernel/source M=$(PWD) modules
   ```

2. **构建命令**：
   ```bash
   # 在模块目录执行
   make -C /lib/modules/$(uname -r)/build M=$PWD modules
   ```
### 模块管理

#### 模块安装路径规范
```bash
# 标准安装路径模板
/lib/modules/$(uname -r)/kernel/<源码树路径>/<模块名>.ko

# 示例：2.6.34内核的钓鱼竿模块
/lib/modules/2.6.34/kernel/drivers/char/fishing.ko
```

**安装命令**：
```bash
sudo make modules_install  # 需root权限
```

---

### 模块依赖管理
#### 1. 依赖关系生成
```bash
# 生成完整依赖关系
sudo depmod

# 增量更新（仅处理新模块）
sudo depmod -A
```

#### 2. 依赖存储位置
```bash
# 依赖关系文件路径
/lib/modules/$(uname -r)/modules.dep

# 文件内容示例
kernel/drivers/char/fishing.ko: kernel/drivers/net/bait.ko
```

> **智能加载原理**：  
> 当加载 `chum.ko` 时，系统自动解析其依赖并先加载 `bait.ko`

---

### 模块加载与卸载
#### 基础工具（不推荐）
| **操作** | **命令**             | **缺陷**               | **示例**               |
|----------|----------------------|-----------------------|------------------------|
| 加载     | `insmod module.ko`   | 无依赖解析            | `insmod fishing.ko`    |
| 卸载     | `rmmod module_name`  | 不检查依赖            | `rmmod fishing`        |

#### 高级工具（推荐）
**1. 智能加载**：
```bash
# 加载模块（自动处理依赖）
sudo modprobe fishing pole_length=300 material=titanium

# 查看加载的模块
lsmod | grep fishing
```

**2. 安全卸载**：
```bash
# 卸载模块（自动移除未用依赖）
sudo modprobe -r fishing

# 强制卸载（危险！）
sudo modprobe -rf fishing  # 可能破坏依赖树
```

### 内核配置与模块开发高级指南

#### 配置选项管理 (Kconfig)
linux使用kbuild系统，可以通过修改Kconfig文件便捷地管理配置选项
```kconfig
# drivers/char/Kconfig 示例
config FISHING_POLE
    tristate "Fish Master 3000 support"  # 三态选项，表示模块在编译配置中可以内置编译到内核（Y），作为模块编译（M）或者不编译（N）
    default n                            # 默认禁用
    depends on FISH_TANK && !NO_FISHING  # 依赖条件
    select BAIT                          # 强制关联选项
    help                                 # 帮助文档
        Enable support for the Fish Master 3000 computer interface.
        Choose Y to build into kernel, M for module (fishing.ko), or N to disable.
```

**核心指令解析**：
| **指令**       | **功能**                          | **示例**                          |
|----------------|----------------------------------|-----------------------------------|
| `tristate`     | 三态选项 (Y/M/N)                 | 驱动标准配置                      |
| `bool`         | 布尔选项 (Y/N)                   | 特性开关                          |
| `default`      | 默认值                           | `default y` 默认启用             |
| `depends on`   | 依赖关系                         | `depends on NET` 需网络支持       |
| `select`       | 强制启用其他选项                 | `select CRC32` 自动启用CRC校验   |
| `if`           | 条件显示                         | `if EMBEDDED` 嵌入式场景可见      |
| `help`         | 帮助文档                         | 用户配置时的说明文本             |

---

### 模块参数系统
#### 1. 基础参数声明
```c
static int pole_length = 200;  // 默认值
module_param(pole_length, int, 0644);  // 整型参数
MODULE_PARM_DESC(pole_length, "Pole length in cm");
```

#### 2. 高级参数类型
| **类型**      | **说明**                | **示例**                          |
|---------------|------------------------|-----------------------------------|
| `charp`       | 字符串指针             | `module_param(name, charp, 0);`  |
| `bool`        | 布尔值                 | `module_param(enable, bool, 0);` |
| `module_param_string` | 直接复制到数组   | `char target[32]; module_param_string(dest, target, sizeof(target), 0);` |
| `module_param_array`  | 数组参数         | `int ids[5]; int count; module_param_array(ids, int, &count, 0);` |

#### 3. 参数传递方式
```bash
# 加载时指定参数
sudo modprobe fishing pole_length=300 material=carbon

# 查看参数信息
modinfo fishing
parm:           pole_length:Pole length in cm (int)
parm:           material:Construction material (charp)
```

#### 4. sysfs集成
```bash
# 参数自动暴露到sysfs
/sys/module/fishing/parameters/pole_length  # 权限0644=rwr--r--
```

---

### 符号导出机制
#### 1. 基础导出
```c
// 声明可被模块调用的函数
int get_pole_strength(struct pole *p) {
    return p->load_capacity;
}
EXPORT_SYMBOL(get_pole_strength);  // 全局导出
```

#### 2. GPL受限导出
```c
EXPORT_SYMBOL_GPL(calculate_bait_ratio);  // 仅GPL模块可用
```

#### 3. 导出规则
| **导出类型**         | **调用权限**               | **典型场景**              |
|----------------------|--------------------------|--------------------------|
| `EXPORT_SYMBOL`      | 所有模块                  | 通用内核API              |
| `EXPORT_SYMBOL_GPL`  | 仅GPL许可证模块           | 核心子系统接口           |
| 未导出符号           | 仅内核内部使用            | 静态函数/私有实现        |

---

### 配置系统元选项
```kconfig
config EXPERIMENTAL
    bool "Enable experimental features"  # 高风险功能开关
    default n

config DEBUG_KERNEL
    bool "Kernel debugging"  # 调试选项总开关
    default y if DEBUG
```

**关键元选项**：
- `CONFIG_EMBEDDED`：嵌入式系统优化选项
- `CONFIG_BROKEN_ON_SMP`：标记非SMP安全驱动
- `CONFIG_EXPERIMENTAL`：实验性功能入口

---

### 开发工作流示例
#### 1. 添加新驱动
```bash
# 创建Kconfig
# drivers/char/fishing/Kconfig
config FISHING_PRO
    tristate "Professional Fishing Module"
    select FISHING_ADVANCED
    help
      Support for professional-grade fishing equipment

# 修改上级Kconfig
# drivers/char/Kconfig
source "drivers/char/fishing/Kconfig"
```

#### 2. 实现参数化模块
```c
#include <linux/moduleparam.h>

static char material[20] = "fiberglass";
module_param_string(material, material, sizeof(material), 0644);

static int lengths[] = {180, 240};
static int nr_lengths = 2;
module_param_array(lengths, int, &nr_lengths, 0444);

static bool enable_ai;
module_param(enable_ai, bool, 0644);
```

#### 3. 编译验证
```bash
# 配置内核
make menuconfig
# -> Device Drivers -> Character devices -> Professional Fishing Module (M)

# 编译安装
make -j$(nproc) modules
sudo make modules_install
```

---

### 生产环境最佳实践
1. **参数安全**：
   ```c
   static int max_load = 100;
   module_param(max_load, int, 0644);
   
   static int __init init_func(void) {
       if (max_load > MAX_SAFE_LIMIT) {
           pr_warn("Dangerous load limit %d, capping at %d\n", 
                   max_load, MAX_SAFE_LIMIT);
           max_load = MAX_SAFE_LIMIT;
       }
   }
   ```

2. **版本兼容**：
   ```c
   #include <linux/version.h>
   
   #if LINUX_VERSION_CODE >= KERNEL_VERSION(5,15,0)
   // 新版内核API
   #else
   // 旧版兼容实现
   #endif
   ```

3. **错误处理**：
   ```c
   int err = register_device();
   if (err) {
       // 彻底回滚初始化
       unregister_previous();
       return err;
   }
   ```

> **性能提示**：高频访问的模块参数应复制到局部变量，避免频繁查sysfs