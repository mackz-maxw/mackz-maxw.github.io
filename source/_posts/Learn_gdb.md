---
title: 学习GCC和GDB
lang: zh-CN
---

### GCC编译

gcc编译hello.c，指定输出为hello：
```
gcc hello.c -o hello
```
运行可执行文件：
```
./hello
```

### 一个简单的Makefile示例

```
hello:hello.c   # 目标文件名:依赖文件列表
	gcc hello.c -o hello	# 用于生成目标文件的命令序列
.PHONY: clean   # 声明伪目标，使用make clean执行clean操作而不是生成clean文件
clean:
	rm hello
```

### debug bash脚本

直接输出：
```
echo "function_name(): value of \\$var is ${var}"
```
可以在脚本shebang列设置需要使用的xtrace选项：
```
#!/bin/bash -x
```
仅在指定列设置xtrace选项：
```
#!/bin/bash
read -p "Path to be added: " $path
set -xv
if [ "$path" = "/home/mike/bin" ]; then
	echo $path >> $PATH
	echo "new path: $PATH"
else
	echo "did not modify PATH"
fi
set +xv
```
使用trap，EXIT模式仅检测退出时状态:
```
#!/bin/bash
trap 'echo score is $score, status is $status' EXIT
if [ -z $1 ]; then
	status="default"
else
	status=$1
fi
score=0
if [ ${USER} = 'superman' ]; then
	score=99
elif [ $# -gt 1 ]; then
	score=$2
fi
```
DEBUG模式可以检测每步状态：
```
trap 'echo "line ${LINENO}: score is $score"' DEBUG
```

### GDB debug

使用友好的GDB debug会话：
```
gcc -ggdb test.c -o test.out
```
设置核心转储：
```
if ! grep -qi 'kernel.core_pattern' /etc/sysctl.conf; then
  sudo sh -c 'echo "kernel.core_pattern=core.%p.%u.%s.%e.%t" >> /etc/sysctl.conf'
  sudo sysctl -p
fi
ulimit -c unlimited # 使得当前会话解除核心文件大小限制
```
永久解除核心文件大小限制：
```
sudo bash -c "cat << EOF > /etc/security/limits.conf
* soft core unlimited
* hard core unlimited
EOF
```
检查核心文件元数据：
```
file core.1341870.1000.8.test.out.1598867712
```
使用GDB分析核心转储：
```
gdb ./test.out ./core.1341870.1000.8.test.out.1598867712
```
在GDB会话中，可以使用以下操作：
```
(gdb) bt	# 回溯
(gdb) f 2	# 转到特定帧
(gdb) list 	# 打印源代码
(gdb) p a/b 	# 打印变量或表达式
```