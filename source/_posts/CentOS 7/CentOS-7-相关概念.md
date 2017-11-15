---
title: CentOS_7_相关概念
date: 2017-09-17 22:35:52
category: Linux
tags: CentOS 7
---

<!-- toc -->

---

# 1. 常用目录说明

| 目录 | 说明 | 备注 |
| --- | --- | --- |
| / | 根目录 | |
| /boot | 保存系统运行相关文件 | 系统启动目录 |
| /bin | 命令保存目录 | 普通用户就可以使用的命令 |
| /sbin | 命令保存目录 | 超级用户可以使用的命令 |
| /lib, /lib64 | 系统库文件保存目录 | |
| /usr | 保存相关的应用程序文件 | |
| /usr/bin | 命令保存目录 | 普通用户就可以使用的命令 |
| /usr/sbin | 命令保存目录 | 超级用户可以使用的命令 |
| /usr/lib, /usr/lib64 | 系统库文件保存目录 |
| /usr/local | 主要存放那些手动安装(非包管理器)的软件 | 和 /usr 目录具有相类似的目录结构 |
| /opt | 存放某些可选、大型或者特殊软件的目录 | 安装到 /opt 目录下的程序，它所有的数据、库文件等等都是放在同个目录下面，安装和删除不影响系统其他任何配置 |
| /tmp | 临时文件存放目录 | |
| /var | 保存系统产生的经常变化的文件 | |
| /proc, /sys | 保存系统相关进程和信息的目录 |  这两个目录都是内存的挂载点 |
| /root | 超级用户的家目录 | |
| /home | 普通用户的家目录 | |
| /etc | 配置文件保存目录 | |
| /dev | 设备硬件文件保存目录 | |
| /mnt, /media | 系统挂载目录 | 空目录，通常用来挂载外置存储设备等 |

## 1.1 /bin, /sbin/, /usr/bin, /usr/sbin 简单区分

这些目录都是用来存放系统、应用程序相关命令的。

1. /bin 和 /sbin
	+ 命令功能
		- /sbin 下的命令属于基本的系统命令，主要放置一些系统管理的必备程式，例如：cfdisk、dhcpcd、dump、e2fsck、fdisk、halt、ifconfig、ifup、 ifdown、init、insmod、lilo、lsmod、mke2fs、modprobe、quotacheck、reboot、rmmod、 runlevel、shutdown 等
		- /bin 下存放一些普通的基本命令，主要放置一些系统的必备执行档，例如：cat、cp、chmod、df、dmesg、gzip、kill、ls、mkdir、more、mount、rm、su、tar 等
	+ 用户权限：
		- /sbin 目录下的命令通常只有管理员才可以运行
		- /bin 下的命令管理员和一般的用户都可以使用
2. /usr/bin 和 /usr/sbin
	+ 命令功能
		- /usr/bin 是你在后期安装的一些软件的运行脚本。主要放置一些应用软体工具的必备执行档，例如：c++、g++、gcc、chdrv、diff、dig、du、eject、elm、free、gnome\*、 gzip、htpasswd、kfm、ktop、last、less、locale、m4、make、man、mcopy、ncftp、 newaliases、nslookup passwd、quota、smb\*、wget 等
		- /usr/sbin 放置一些用户安装的系统管理的必备程式，例如：dhcpd、httpd、imap、in.*d、inetd、lpd、named、netconfig、nmbd、samba、sendmail、squid、swap、tcpd、tcpdump 等
	+ 用户权限
		- /usr/sbin 目录下的命令通常只有管理员才可以运行
		- /usr/bin 下的命令管理员和一般的用户都可以使用
3. / 和 /usr 下的 bin/sbin 的区别
	- /bin,/sbin 目录是在系统启动后挂载到根文件系统中的，所以/sbin,/bin 目录必须和根文件系统在同一分区
    - /usr/bin,usr/sbin 可以和根文件系统不在一个分区
    - 从可运行时间角度看，因为 /sbin,/bin 在系统启动后就挂载到根文件系统中，所以能够在挂载其他文件系统前就使用，但 /usr/bin 和 /usr/sbin 不行

## 1.2 关于 /mnt,/media,/misc 挂载目录

我们可以自定义任意目录用于挂载外置存储设备，而这三个目录都是系统提供的用于挂载的空目录，/mnt 是最为常见的挂载目录，是 Linux 系统一直支持的，而 /media 和 /misc 是新增的，并不是所有 Linux 版本都有的，所以通常都会使用 /mnt，在其中自定义目录来挂载不同类别的外置存储设备。

# 2. 文件/目录详细信息相关说明

## 2.1 详细信息概述

显示详细信息命令：ls -li[h]

举例格式：
524414 drwxr-xr-x. 2 root root 4.0K Aug 12  2015 bin

各部分对应作用：
inode 文件类型及权限 硬链接数量 所有者 所属组 文件大小 修改日期 文件名

说明：

- inode
: - inode 译成中文就是索引节点。每个存储设备或存储设备的分区被格式化为文件系统后，应该有两部份，一部份是 inode，另一部份是 Block，Block 是用来存储数据用的，而 inode 就是用来存储这些数据的信息，包括文件大小、属主、归属的用户组、读写权限等。inode 为每个文件进行信息索引，所以就有了 inode 的数值。操作系统根据指令，能通过 inode 值最快的找到相对应的文件。
: - inode 值相同的文件是硬链接文件，也就是说，不同的文件名，inode 可能是相同的，一个 inode 值可以对应多个文件。
- 硬链接数量：1 表示没有硬链接，就是本身

## 2.2 文件类型及权限

格式：drwxr-xr-x.

说明：

- 第一位：文件类型，-：文件，d：目录，l：软链接文件，b：块设备文件
+ rwxr-xr-x：操作权限，每三个为一组，依次对应所有者 user/所属组 group/其他人 other 的权限，其中：
	- r：读，4
	- w：写，2
	- x：执行，1
- 最后一位：.，是在 CentOS 6 之后才有的，未明确定义，估计为 ACL 权限

# 3. 硬链接和软链接

## 3.1 硬链接

特征：

- 拥有与原文件相同的 inode 和 存储 block 块，可以看作是访问原文件的另一个入口
- 可通过 inode 进行识别
- 不能跨分区
- 不能针对目录使用
- 修改原文件或硬链接，另一个都会改变
- 删除原文件或硬链接，另一种方式仍能访问到

## 3.2 软链接

特征：

- 与 Windows 的快捷方式基本一致
- 软链接拥有自己的 inode 和 block 块，但是 block 中保存的是原文件的文件名和 inode 等信息，并没有实际的文件数据
+ 软链接的文件详细信息
	- 软链接的文件类型和操作权限是固定的：lrwxrwxrwx，但对于真正的操作权限是以原文件的操作权限为准的。
	- 软链接详细信息中的文件名后指定了原文件的文件名，如：bin -> usr/bin
- 修改原文件或软链接，另一个都会改变
- 删除原文件，软链接失效，并且再次查看软链接的详细信息时，软链接的文件名以及映射的原文件名都会有颜色变化和闪动，访问该软链接时，会提示 No such file or directory
- 可以跨分区，对目录使用

# 4. 系统运行级别

Linux 默认的运行级别有如下几种：

| 描述 | 值 | 备注 |
| --- | -- | --- |
| 关机 | 0 | Do Not set initdefault to this |
| 单用户 | 1 | 类似于 windows 的安全模式，主要用于系统修复 |
| 不完全多用户命令模式 | 2 | 相比于 3 级别，缺少 NFS 服务，即是一个不完全的标准字符界面 |
| 完全多用户命令模式 | 3 | 标准的字符界面 |
| 未分配 | 4 ||
| 图形界面 | 5 | 使用 `startx` 命令可直接运行 |
| 重启 | 6 | Do Not set initdefault to this |

## 4.1 查看当前运行的级别

```shell
[root@localhost ~]# runlevel
N 3
```

说明：

- `runlevel`：查看系统的运行级别
- `N 3`：上一个运行级别 当前运行级别

## 4.2 修改运行级别

### 4.2.1 临时修改

在 CentOS 6 之前，使用 `init 数值` 来修改对应的运行级别，从 CentOS 6 开始，修改了系统的启动流程，使用 systemd 替代了 sysvinit，使用 target 替代了运行级别，所以，从 CentOS 6 开始，建议使用下面命令来切换运行级别：

- systemctl isolate multi-user.target
- systemctl isolate runlevel5.target

常用 target：

- 第三运行级：runlevel[3/multi-user].target
- 第五运行级：runlevel[5/graphical].target

说明：前者是符号/软链接指向了后面的target

- runlevel3.target -> multi-user.target
- runlevel5.target -> graphical.target

### 4.2.2 开机默认运行级别修改

systemd 使用软链接来指向默认的运行级别，由 /etc/systemd/system/default.target 文件中决定。

#### 4.2.2.1 切换到运行级3

- ln -sf /lib/systemd/system/multi-user.target /etc/systemd/system/default.target
- ln -sf /lib/systemd/system/runlevel3.target /etc/systemd/system/default.target

或

- systemctl set-default multi-user.target

#### 4.2.2.2 切换到运行级5

- ln -sf /lib/systemd/system/graphical.target /etc/systemd/system/default.target
- ln -sf /lib/systemd/system/runlevel5.target /etc/systemd/system/default.target

或

- systemctl set-default graphical.target
