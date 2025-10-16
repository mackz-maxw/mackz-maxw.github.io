---
title: 在wsl2安装ros2遇到的一些坑
categories: wsl2
lang: zh-CN
---

### 依赖问题

由于安装的老版本（humble）ros2, 很多必需包直接安装会报依赖小版本号不一致错误，使用apt install解决是很麻烦的， 于是要用一个比较新的依赖处理工具
```bash
sudo apt install aptitude
sudo aptitude install [package]
```
在[package]填出问题的依赖，aptitude会弹出建议的一些解决方案，按`,`和`.`切换到一个降版本的解决方案，yes即可

### swap空间不足

已经到了编译这一步了，但是编到最后总是报错
```
C++: fatal error: Killed signal terminated program cc1plus
```
很搞心态，开新terminal用`htop`命令一查，cpu内存swap全拉满

这个时候就需要给wsl2扩swap空间， 在 c:\Users\用户名 下找一个叫`.wslconfig`的文件，找不到就创建一个，填：
```
[wsl2]
swap=4GB
```
填完`wsl --shutdown`重启一下wsl2， 在wsl的终端`free -h`可以查看swap分区的大小，当然直接`htop`自然是可以的

如果rosdep已经初始化过了，还要删掉初始化文件重新初始化一下。执行这个`sudo rosdep init`会有提示

这个时候再`colcon build --symlink-install`就万事大吉了

