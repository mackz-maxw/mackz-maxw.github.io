---
title: 设置Ubuntu
lang: zh-CN
---

### 设置apt源

检查Ubuntu的系统版本和代号是否与要安装的源一致
在终端中输入：
```
lsb_release -a
```
source APT源：
```
sudo bash -c 'echo "deb http://mirrors.aliyun.com/ubuntu/ jammy main restricted universe multiverse" > /etc/apt/sources.list'
```
检查source list：
```
cat /etc/apt/sources.list
```
从源获取包信息：
```
sudo apt-get update
```

### vim技巧

虚拟块模式（选择一块代码）：ctrl + q
在虚拟块模式中删除选中：d

### 设置vim

生成并打开设置文件：
```
cp /etc/vim/vimrc ~/.vimrc
cd ~
ls -a
vim .vimrc
```
具体设置更改：
```
syntax on   " 语法高亮
set background=dark
filetype plugin indent on
set showmatch          " Show matching brackets.
set ignorecase         " Do case insensitive matching
set smartcase          " Do smart case matching
set incsearch          " Incremental search
set hidden             " Hide buffers when they are abandoned
```