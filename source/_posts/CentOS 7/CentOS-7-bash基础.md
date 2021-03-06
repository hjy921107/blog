---
title: CentOS_7_bash基础
date: 2017-09-17 23:06:00
category: Linux
tags: CentOS 7
---

<!-- toc -->

---

Bash 是 Linux 中最基本和最常见的 Shell，它提供了很多基本功能来帮助我们更方便、高效的使用 Linux 和对编写 Shell 脚本带来帮助。

# 1. 别名

## 1.1 查看系统中别名

格式：alias

在 CentOS 中有如下系统默认的别名

```shell
alias cp='cp -i'
alias egrep='egrep --color=auto'
alias fgrep='fgrep --color=auto'
alias grep='grep --color=auto'
alias l.='ls -d .* --color=auto'
alias ll='ls -l --color=auto'
alias ls='ls --color=auto'
alias mv='mv -i'
alias rm='rm -i'
alias which='alias | /usr/bin/which --tty-only --read-alias --show-dot --show-tilde'
```

## 1.2 设置别名

### 1.2.1 临时设置

格式：alias 别名=完整命令

如：alias la='ls -la --color=auto'

说明：临时设置只对此次登陆有效

### 1.2.2 永久设置

区别于临时设置，永久设置只要没有再次修改此配置，那么此配置将长期有效。永久设置的命令格式同临时设置的完全相同，只是永久设置是将配置写入到系统的环境变量配置文件中，常见的环境变量配置文件为 /当前用户的家目录/.bashrc

如下为系统默认的 /root/.bashrc 文件内容，其中已经指定了几个永久设置的命令别名

```shell
# .bashrc

# User specific aliases and functions

alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'

# Source global definitions
if [ -f /etc/bashrc ]; then
	. /etc/bashrc
fi
```

要想设置永久的命令别名，只需要在对应用户下的 .bashrc 文件中增加 alias 命令即可。不过，修改此配置文件后需要重新登陆才能生效，或者使用 `source /root/.bashrc`命令强制使配置立即生效。

## 1.3 删除别名

格式：unalias 别名

如：unalias la

说明：同样地，此命令只是删除临时的别名，对于写入到系统环境变量配置文件中的别名，需要修改文件内容才，并重新登陆或使用 `source /root/.bashrc` 命令使配置立即生效。

## 1.4 命令的生效顺序

对于命令，除了系统默认的情况，还在多个地方、使用多种方式设置了别名，那么的生效顺序如下：

1. 执行明确指定路径（绝对/相对）的命令
2. 执行别名
3. 执行 Bash 的最原始的内部命令
4. 按照 $PATH 中定义的目录顺序找到的第一个命令

由上述顺序可知：别名的执行优先级要高于原始命令，所以，如非必要，别名尽量不要和原始命令重名，否则原始命令会被覆盖。

# 2. 快捷键

- Ctrl + c 强制终止当前命令
- Ctrl + l 清屏（其实是控制滑轮将当前屏幕的内容隐藏）
- Ctrl + a 光标移到到命令行首
- Ctrl + e 光标移到到命令行尾
- Ctrl + u 从光标所在位置删除到行首
- Ctrl + z 把命令放入后台执行，不建议使用
- Ctrl + r 在执行过的历史命令中搜索

# 3. 历史命令

格式：history [选项] [历史命令保存文件]

选项：

- -c 清空历史命令
- -w 把缓存中的历史命令写入到历史命令保存文件中（每个用户家目录下的 .bash_history）

说明：

- 直接使用 history 命令可以显示当前用户 .bash_history 文件中的所有记录以及此次登陆中已经执行过的所有命令记录
- .bash_history 文件中保存的是当前用户之前执行的所有命令，即默认情况下只有用户正确注销之后，他此次登陆所执行的命令才会保存到此文件中
+ 可以通过上述选项强制清空或保存当前缓存中的命令至指定文件，如果没有指定文件，系统默认为当前用户的 .bash_history 文件
	- 使用强制保存后，会将此次登陆所执行的命令保存到 .bash_history 文件中，而不需要 logout
	- 使用清空命令后，并不会直接清空 .bash_history 文件，还是需要使用 -w 来强制覆盖 .bash\_history 文件，此时，该文件中只剩下 `history -w` 这一条刚刚执行的记录
	- 默认情况下 .bash_history 文件中保存 1000 条记录，可以通过修改环境变量配置文件（/etc/profile）中的 HISTSIZE 的值来自定义记录的保存数量

**历史命令的调用：**

- 使用上、下箭头调用以前的历史命令
- 使用 "!n" 重复执行第 n 条历史命令
- 使用 "!!" 重复执行上一条历史命令
- 使用 "!字符串" 重复执行最后一条以该字符串开头的命令

# 4. 输入/输出及重定向

## 4.1 标准输入/输出

| 设备 | 设备文件名 | 文件描述符 | 类型 |
|:----:|:--------:|:---------:|:---:|
| 键盘 | /dev/stdin | 0 | 标准输入 |
| 显示器 | /dev/stdout | 1 | 标准输出 |
| 显示器 | /dev/stderr | 2 | 标准错误输出 |

## 4.2 标准输出重定向

<table>
	<tr>
		<td>类型</td>
		<td>输出重定向命令的具体格式</td>
		<td>作用</td>
	</tr>
	<tr>
		<td rowspan="2">标准输出重定向</td>
		<td>命令 > 文件</td>
		<td>以覆盖的方式将命令正确执行的结果输出到指定文件或设备中</td>
	</tr>
	<tr>
		<td>命令 >> 文件</td>
		<td>以追加的方式将命令的正确执行结果输出到指定文件或设备中</td>
	</tr>
	<tr>
		<td rowspan="2">标准错误输出重定向</td>
		<td>错误命令 2> 文件</td>
		<td>以覆盖的方式将执行错误命令的提示信息输出到指定文件或设备中</td>
	</tr>
	<tr>
		<td>错误命令 2>> 文件</td>
		<td>以追加的方式将执行错误命令的提示信息输出到指定文件或设备中</td>
	</tr>
</table>

上述表格中的命令使用频率并不高，尤其是标准错误输出重定向命令，因为需要先明确执行的命令是否正确，然后才能使用对应的命令，如：对于错误命令，既然知道执行的命令是错误的，谁还会执行。所以，通常使用下面的命令，不用区分命令是否正确，都可以保存。

| 正确输出和错误输出同时保存命令 | 作用 |
| -------------------------- | --- |
| 命令 > 文件 2>&1 | 以覆盖的方式将正确和错误的输出都保存到同一个文件中 |
| 命令 >> 文件 2>&1 | 以追加的方式将正确和错误的输出都保存到同一个文件中 |
| 命令 &> 文件 | 以覆盖的方式将正确和错误的输出都保存到同一个文件中 |
| 命令 &>> 文件 | 以追加的方式将正确和错误的输出都保存到同一个文件中 |
| 命令 >> 文件1 2>> 文件2 | 以追加的方式将正确输出保存到文件1中，将错误输出保存到文件2中 |

说明：

- 输出重定向符号：> 表示输出覆盖；>> 表示输出追加
- 当使用文件描述符（0/1/2）以及 &
 时，文件描述符与输出重定向符之间不能有空格。默认情况下，输出重定向符之间是 1，如 `命令 1> 文件`，将命令的正确执行结果保存到文件中。
- 特殊的输出目的：/dev/null，类似于回收站，如 `ls &> /dev/null`，将 ls 命令的执行结果直接放弃，通常用于对某个命令的执行结果不关心、不需要显示给用户，可通过此命令直接删除命令的执行结果。

## 4.3 标准输入重定向

与标准输出重定向相反，标准输出重定向使用 < 和 << 符号

- 命令 < 内容 表示将内容作为命令的输入，内容可以是键盘录入或者文件等
- 命令 << 标识符 表示将指定标识符之间的内容作为命令的输入，如执行 `wc << abc`，此时 wc 命令会统计键盘录入的内容，直到再次出现 abc 输入结束（abc 不在统计范围之内）。

# 5. 多命令的顺序执行

| 多命令执行符 | 多命令执行格式 | 作用 |
|:----------:|:------------:| ---- |
| ; | 命令1 ; 命令2 | 多命令顺序执行，命令之间没有任何逻辑关系 |
| && | 命令1 && 命令2 | 逻辑与；当 1 正确执行，则 2 执行；否则 2 不执行 |
| &#124;&#124; | 命令1 &#124;&#124; 命令2 | 逻辑或；当 1 执行错误，则 2 执行，否则 2 不执行 |

# 6. 管道符

格式：命令1 | 命令2 [| 命令3 ...]

说明：管道流也是多命令顺序流的一种，只是管道流比上述的基本的多命令顺序流更为特殊，它要求**命令1的正确输出要作为命令2的操作对象**，并非简单的要求命令之间是否存在逻辑关系的问题。

举例：

- ll -ah /etc/ | more 使用 more 命令将 ll 命令的执行结果分买票显示
- netstat -an | grep ESTABLISHED | wc -l 使用 grep 搜索 netstat 命令列出的哪些端口已经正确建立连接（state 为 ESTABLISHED）并将结果交给 wc 命令统计正确建立连接的个数

# 7. 通配符

| 通配符 | 作用 |
|:-----:| ---- |
| ? | 匹配一个任意字符 |
| * | 匹配 0 个或任意多个字符，即可以匹配任意内容 |
| [] | 匹配括号中列出的任意一个字符 |
| [-] | 匹配括号中列出范围内的任意一个字符 |
| [^] | 匹配一个不是括号内列出的或不在列出范围内的字符 |

# 8. Bash 中其他的特殊符号

| 符号 | 作用 |
|:---:| ---- |
| '' | 单引号，被单引号括起来的所有字符都没有特殊含义 |
| "" | 双引号，被双引号括起来的所有字符都没有特殊含义，但 "$"、"`" 和 "\\" 是例外，它们分别拥有 “调用变量的值”、 “引用命令” 和 “转义符” 的特殊含义，这是单双引号的最重要的区别 |
| `` | 反引号，被反引号括起来内容会作为系统命令，在 Bash 先执行。和 \$() 作用一样，但推荐使用后者，因为反引号不易识别 |
| \$() | 用来执行系统命令，与反引号的作用一样，建议使用 \$() |
| # | 在 Shell 中，# 开头的行代表注释，除了 `#!/bin/bash` |
| \$ | 用于调用变量的值 |
| \ | 转义符，在 \ 后面的字符将失去特殊含义，变为普通字符 |
