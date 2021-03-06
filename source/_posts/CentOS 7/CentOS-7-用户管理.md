---
title: CentOS_7_用户管理
date: 2017-09-17 22:57:28
category: Linux
tags: CentOS 7
---

<!-- toc -->

---

# 1. 基本概念

- 用户：使用操作系统的人
- 用户组：具有相同系统权限的一组用户

# 2. 与用户/用户组相关的配置文件

- /etc/group：存储当前系统中所有用户组的信息

	说明：

	- 文件内容格式：组名称:组密码占位符:组编号:组中用户名列表
	- 每行代表一个组信息，组信息中用 `:` 分隔
	- 所有的组密码占位符都是 `x`
	- root 组的组编号始终为 `0`
	- 1-499 为系统预留的编号，系统自动分配当前未使用的最小编号给对应的应用程序
	- 用户手动创建的组编号是从 500 开始的，系统自动分配从 500 开始的当前未使用的最小编号给新建的用户组
	- 组中用户名列表为空并不一定是该组中没有任何用户，当组中的用户名同组名时，该用户名是可以在列表中省略的

- /etc/gshadow：存储当前系统中用户组的密码信息

	说明：

	- 此文件中的内容与 /etc/group 文件中的内容一一对应
	- 文件内容格式：组名称:组密码:组管理者:组中的用户名列表
	- 每行代表一个组信息，组信息中用 `:` 分隔
	- 组密码如果为 */!/空 均视为组密码为空
	- 组管理者：表示组内哪些用户可以管理这个组，通常情况下为空，即表示组内所有用户都可以管理这个用户组

- /etc/passwd：存储当前系统中的所有用户的相关信息

	说明：

	- 文件内容格式：用户名:密码占位符:用户编号:用户组编号:用户注释信息:用户主目录:shell 类型
	- 每行代表一个组信息，组信息中用 `:` 分隔
	- 所有的组密码占位符都是 `x`
	- root 用户的用户编号始终为 `0`

- /etc/shadow：存储当前系统中所有用户的密码信息

	说明：

	- 此文件的内容与 /etc/passwd 文件中的内容一一对应
	- 文件内容格式：用户名:密码::::::
	- 每行代表一个用户信息，用户信息中用 `:` 分隔
	- 用户密码如果为 */!/空，均视为该用户密码为空
	- 密码通过一种单向的加密方式保存


# 3. 用户与用户组的相关命令

## 3.1 用户组

### 3.1.1 增加用户组

格式：groupadd [选项] 参数

- `groupadd sexy`：增加用户组，组名为 sexy
- `groupadd -g 888 boss`：创建 group id 为 888、组名为 boss 的用户组

### 3.1.2 修改用户组

格式：groupmod [选项] 参数

- `groupmod -n market sexy`：修改用户组名，新组名 原组名
- `groupmod -g 668 market`：修改指定用户组的 group id

### 3.1.3 删除用户组

格式：groupdel [选项] 参数

- `groupdel market`：删除用户组 market。删除用户组之前，必须先删除该用户组下的所有用户

## 3.2 用户

### 3.2.1 新增用户

格式：useradd [选项] 参数

说明：

- 如果创建用户时未指定该用户所在的组，系统会自动创建与该用户名同名的用户组，并将该用户加入到此用户组
- 如果创建用户时未指定用户的家目录，这属于默认情况，系统会在 /home 目录下自动创建与用户名同名的目录作为该用户的家目录

- `useradd -g sexy lscang`：在指定的用户组下面新增用户
- `useradd -g group1 -G group2,group3 username`：新增用户，指定该用户的主要组 group1，附属组 group2 和 group3
- `useradd -d /home/test test`：新增用户并指定该用户的家目录

### 3.2.2 修改用户

格式：usermod [选项] 参数

- `usermod -c 123 test`：给指定用户添加备注信息
- `usermod -l bdyjy lscang`：修改用户名，新名称 原名称
- `usermod -g sexy test`：修改指定用户所在的用户组，组名 用户名
- `usermod -d /home/test test`：修改用户的家目录，系统不会自动创建该目录，需要手动创建

### 3.2.3 删除用户

格式：userdel [选项] 参数

- `userdel test`：删除指定用户，但并不会删除该用户的家目录
- `userdel -r lscang`：删除指定用户，并删除该用户的家目录

## 3.3 进阶命令

### 3.3.1 passwd 命令

- 锁定帐户：`passwd -l username`
- 解锁帐户：`passwd -uf username`，其中 -f 为 force，不添加会出现警告，导致解锁失败
- `passwd username`：设置指定用户的密码，执行此命令后，系统会提示输入密码和确认密码，只要再次输入相同，即可设置成功
- 清除帐户密码：`passwd -d username`

### 3.3.2 gpasswd 命令

在 Linux 系统中，一个用户是可以属性多个组的，其中一个为主要组，其他的为附属组。默认情况下用户的所有权限、操作等都是基于主要组的。

- `gpasswd -a lscang boss`：为用户添加附属组，可以同时添加多个附属组，组名之间用 `,` 隔开
- `newgrp boss`：用户切换组，如果目标组存在组密码，需要验证密码
- `gpasswd -d lscang boss`：删除用户的附属组
- `gpasswd groupname`：设置指定用户组的密码，执行此命令后，系统会提示输入密码和确认密码，只要再次输入相同，即可设置成功

## 3.4 其他命令

- `touch /etc/nologin`：在 /etc 目录下创建文件 nologin，可以阻止除 root 用户以外的所有用户的登陆，文件内容无关紧要
+ 切换用户（需要用户密码，root 用户切换到其他用户不需要密码）：
	- `su`：直接执行，不加用户名，切换到 root 用户
	- `su username`：切换到指定的用户
+ `id username/groupname`：
	- username：显示指定用户信息，包括用户编号、用户名
	- groupname：显示指定用户组信息，包括主要组编号及名称、附属组列表
- `groups username`：显示该用户所在的所有组
- `chfn username`：修改用户资料，按照系统提示输入即可
- `finger username`：显示用户的详细资料，需要 finger 包的支持
