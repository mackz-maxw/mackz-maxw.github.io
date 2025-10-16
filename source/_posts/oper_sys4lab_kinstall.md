---
title: 操作系统基础 | 2.4 初试linux内核实验
categories: 
  - os basic
  - lab
lang: zh-CN
---

## 树莓派镜像下载、配置与自定义内核安装流程

### 1. 下载并写入指定树莓派镜像

为避免因购买或借用的树莓派设备时间不同导致的系统版本不一致，建议大家统一下载指定的树莓派镜像作为起点。

- 将 MicroSD 卡插入 USB 读卡器，并连接到你的电脑。
- 下载并安装最新版 Raspberry Pi Imager。
- 下载课程指定的树莓派镜像：[2022-01-28-raspios-bullseye-armhf.zip](https://downloads.raspberrypi.org/raspios_armhf/images/raspios_armhf-2022-01-28/2022-01-28-raspios-bullseye-armhf.zip)（约1.2GB，解压后近4GB）。
- 解压 zip 文件，得到 .img 镜像文件。
- 打开 Raspberry Pi Imager，选择“CHOOSE OS”→“Use custom”，选中刚才解压的 .img 文件。
- 选择“CHOOSE SD CARD”，选中你的 MicroSD 卡（注意不要选错，否则会清空数据）。
- 进入高级设置（齿轮图标，或 Windows 下用 Ctrl+Shift+X），建议修改主机名为唯一值（如包含你的用户名），以免与他人冲突。
- 勾选“Enable SSH”，选择“Use password authentication”，设置用户名和强密码（建议不要用默认密码）。
- 勾选“Set locale settings”，设置时区（如美国中部用 America/Chicago），键盘布局选 us。
- 点击“Save”，然后点击“WRITE”写入镜像。
- 写入完成后，卸载 MicroSD 卡，插入树莓派并开机。


### 2. 首次启动与 SSH 连接

- 树莓派首次启动可能需要几分钟，启动后通过 SSH 连接（如 ssh piuser@pihost，用户名和主机名为你设置的）。
- 若主机名无法解析，可在路由器管理页面查找树莓派的 IP 地址，再用 ssh 连接。

---

### 3. 初始设置与系统升级

- 首次登录会看到“Welcome to the Raspberry Pi”向导，按提示设置国家、语言、时区、键盘等。
- 可能会再次提示设置密码，可直接关闭。
- 为防止欢迎界面反复出现，运行：
  ```sh
  sudo apt purge piwiz
  ```
- 升级系统和驱动：
  ```sh
  sudo apt-get update
  sudo apt-get upgrade
  ```
- 升级完成后重启树莓派。

---

### 4. 网络设置与信息收集

- 连接 WiFi，打开终端，运行：
  ```sh
  ifconfig wlan0
  ```
  记录 MAC 地址（ether 后面的六组十六进制数）。
- 运行：
  ```sh
  hostname -I
  ```
  记录 IP 地址。

---

### 5. 传输并安装自编译内核与模块

- 进入你交叉编译内核的目录：
  ```sh
  cd ~/linux_source
  ```
- 打包模块和内核：
  ```sh
  tar -C modules/lib -czf modules.tgz modules
  tar -C linux/arch/arm -czf boot.tgz boot
  ```
- 在树莓派上新建 linux_source 目录，进入后用 sftp ，或其它方式下载上述两个压缩包：
  ```sh
  sftp [学校统一登陆平台key]@shell.cec.学校曾用简称.edu
  cd /project/scratch01/compile/"your username"/linux_source
  get modules.tgz
  get boot.tgz
  quit
  ```
- 备份 `/usr/lib/modules` 和 `/boot` 目录（或 `/lib/modules`，视系统而定）：
  ```sh
  sudo cp -r /usr/lib/modules ~/Desktop/modules_backup
  sudo cp -r /boot ~/Desktop/boot_backup
  ```
- 解压并安装新内核和模块：
  ```sh
  tar -xzf modules.tgz
  tar -xzf boot.tgz
  cd modules
  sudo cp -rd * /usr/lib/modules   # 或 /lib/modules
  cd ..
  sudo cp boot/dts/*.dtb /boot/
  sudo cp boot/dts/overlays/*.dtb* /boot/overlays
  sudo cp boot/dts/overlays/README /boot/overlays
  ```
- 树莓派 3B+：
  ```sh
  sudo cp boot/zImage /boot/kernel7.img
  ```
- 树莓派 4/4B：
  ```sh
  sudo cp boot/zImage /boot/kernel7l.img
  ```

---

### 6. 验证新内核

- 重启树莓派，运行：
  ```sh
  uname -a
  ```
  检查输出是否包含你设置的本地版本字符串、编译日期等。
- 若未生效，编辑 `/boot/config.txt`，在 `[pi4]` 段落前加一行：
  ```
  arm_64bit=0
  ```
  再重启并用 `uname -a` 检查。


---

### 7. 备份与后续建议

- 建议用 svn、git 等工具备份你的代码，也可在多台树莓派间做冗余，防止系统崩溃或锁死导致数据丢失。
- `sudo passwd root`更新root密码（忘记密码用）
- `sudo raspi-config`中可以更改主机名
- 在树莓派的桌面环境中，可以使用快捷键 Ctrl + Alt + T 快速打开终端

### 用户管理

- `sudo adduser 新用户名`创建新用户
- `usermod -aG sudo username`将username用户加入sudoers组
- 执行visudo命令并在文件中添加`username  ALL=(ALL) NOPASSWD:ALL`-赋予username用户执行所有sudo命令权限，不需要密码提示
- `sudo userdel --remove --force pi`删除默认账号
