---
title: 为wsl2安装中文输入法
lang: zh-CN
---

安装中文语言包和fcitx输入法管理器，其中fcitx-sunpinyin是新安装的输入法
```
sudo apt install language-pack-zh-hans
sudo apt install fcitx dbus-x11 im-config fcitx-sunpinyin
```

编辑/etc/locale.gen文件，找到 # zh_CN.UTF-8 这一行，取消注释
```
zh_CN.UTF-8
```

编辑~/.profile文件，在文件末尾加上：
```
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS=@im=fcitx
export DefaultIMModule=fcitx
fcitx-autostart &>/dev/null
```
以上代码分别：设置了GTK（GIMP Toolkit）和QT程序的输入法模块为fcitx。GTK和QT是Linux系统中用于创建图形用户界面的工具包；设置了X Window System的修饰符（modifiers），以确保fcitx能够正确地拦截键盘事件；设置了默认的输入法模块为fcitx；设置了fcitx的自动启动，并将启动时的输出重定向到/dev/null，这样就不会在终端中显示启动过程的输出信息

刷新~/.profile在shell中的引用
```
source ~/.profile
```

刷新字体缓存
```
fc-cache -f -v
```
重启 wsl2
```
poweroff
```
重启fcitx
```
fcitx-autostart
```

打开fcitx的设置页面
```
fcitx-config-gtk3
```
在输入法中, 选择keyboard_en_us作为首位输入法，之前安装的fcitx-sunpinyin为第二位。
默认的输入法切换键位是ctrl+space，可以在fcitx的设置页面中更改。

这一套完成后，亲测无需将系统语言切换至中文，也可在gedit使用中文输入法。当然如果想在特定编译器中使用中文输入法，或许需要更复杂的配置了。设置IDEA可以参考：[monkeywie的博客:wsl2官方gui安装IDEA踩坑记录](https://monkeywie.cn/2021/09/26/wsl2-gui-idea-config)