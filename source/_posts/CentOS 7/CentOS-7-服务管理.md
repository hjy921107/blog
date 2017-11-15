---
title: CentOS_7_服务管理
date: 2017-09-17 23:02:27
category: Linux
tags: CentOS 7
---

<!-- toc -->

---

前言

传统的 Linux 系统启动过程主要由 init 进程（也被称为 SysV init 启动系统）处理，从 CentOS 6 开始，就已经开始使用 Systemd 来渐渐取代 init，在 CentOS 7 中，与 init 相关的内容已所剩无几，下面所述的服务相关操作也都是使用 Systemd 的相关命令来完成的，所以，对于 Cent OS 7 以下的版本可能并不完成适用。

# 1. 服务的分类

Linux 服务：

- 源码包安装的服务
+ rpm 包默认安装的服务
   - 独立的服务
   - 基于 xinetd 的服务

说明：

- 独立的服务，每个服务独立的占用内存，对于其他程序的访问能立即响应
+ 基于 xinetd 的服务
    - xinetd：是超级守护进程的一种，用于管理其他服务
    - 基于 xinetd 的服务并不单独占用内存，它们由 xinetd 进程调度，只有 xinted 进程占用内存。由于服务是由 xinetd 进行调度，所以对于其他程序的访问，响应相对没有独立服务那么迅速

# 2. 服务端口

每个 IP 地址都有 65536 个（0 - 65535）端口，不同的端口对应该 IP 提供的不同服务。在传输层，通常使用 TCP 和 UDP 两种协议，即每种协议都有 65536 个端口，但是为了便于管理，通常一个服务占用了一种协议的一个端口后，另一种协议的对应端口系统也会预占，这样不会混淆。

由于端口过多，在 CentOS 中，系统提供了一个相对比较全面的服务与端口的映射关系文件，文件为：`/etc/services`，此文件通常情况下不建议修改，仅供参考、查询之用。

# 3. 服务查询

- `systemctl --type=service [list-units]`：列出当前正在运行的服务状态
- `systemd-cgls`：以树形列出正在运行的进程，它可以递归显示控制组内容
- `systemctl status 服务名`：显示一个服务的运行状态
- `systemctl command 服务名`
    - `is-enabled`：查看服务是否开机启动，返回值为：enable、disable或static，其中 static 是指对应的 Unit 文件中没有定义 [Install] 区域，因此无法配置为开机启动服务
    - `is-active`：判断服务当前是否 active
    - `is-failed`：判断服务启动是否失败
- `systemctl --failed`：查看启动失败的服务列表
- `systemctl [--state=状态] list-unit-files`：查看系统中所有/任意状态服务的列表，输出信息仅有服务名和自启动状态两列

# 4. 服务启动与自启动

+ 服务常见位置：
    + rpm 服务
        - `/usr/lib/systemd/system/` 中的 \*.service 文件
        - `/etc/init.d/` 或 `/etc/rc.d/init.d/`
    - 基于 xinetd 服务，详见 [4.3 基于 xinetd 的服务](#xinetd)
    - 源码包服务：详见 [4.4 源码包安装的服务](#src)

## 4.1 启动

在当前系统中让服务运行，并且提供功能。

启动方式：

1. `systemctl start/stop/restart 服务名`
2. 执行服务启动的脚本文件（绝对路径）
3. `service 服务名 start/stop/restart`

说明：

- 1，3 方式是 RedHat 系列 Linux 的快速启动服务方法，并不是所有 Linux 的发行版本都支持
- 2 方式是通用的启动方法，但需要记住服务启动脚本的绝对路径，通常，rpm 安装的脚本都在 `/usr/sbin` 或 `/usr/bin` 下
- 本质上，方式 3 其实还是使用了方式 1 进行服务的管理

## 4.2 自启动

指让服务在系统开机或重启之后，随着系统的启动而自动启动

自启动方式：

- 修改配置文件：`/etc/rc.d/rc.local` 或 `/etc/rc.local`
- 命令方式：`systemctl enable/disable 服务名`

查看服务是否自启动：`systemctl is-enabled 服务名`

说明：

- 配置文件：后者是前者的软链接，此文件为系统启动后默认加载文件，文件中命令都会默认执行，可以在此文件中添加需要自启动服务的启动命令来使该服务自启。
- 命令方式：启用服务就是在 `/etc/systemd/system/multi-user.target.wants/` 目录下建立 需要自启动服务的软链接；禁用服务就是删除此软链接；添加服务就是添加软连接。
- 上述两种方式相互独立，互不影响

## <span id="xinetd">4.3 基于 xinetd 的服务</span>

基于 xinetd 的服务管理已渐渐被淘汰，此处只作简要说明。

+ 安装 xinetd 服务，同时会自动安装一些由 xinetd 管理的服务
    - xinetd 服务：此服务本身就是一个 rpm 服务，相关操作同 rpm 服务
    - 基于 xinetd 的服务：相关操作见下
- 安装完成后，会在 `/etc/xinetd.d/` 下生成默认随 xinetd 安装的服务的对应配置文件，详细配置不再说明
+ 启动与自启动
    - 对于基于 xinetd 的服务而言，启动服务与服务自启动是一致，如果开启该服务，意味着此服务将会自启动；如果设置这个服务自启动，则该服务也会立即启动；关闭同样如此。
    - 启动/关闭服务：`chkconfig 服务名 on/off`
    - 服务自启动：修改 `/etc/xinetd.d/` 下的对应服务的配置文件，其中 `disable=no` 即可

## <span id="src">4.4 源码包安装的服务</span>

所有服务最根本的启动方式都是执行该服务的启动脚本，对于源码包，默认情况下，唯一的启动方式就是执行脚本文件，通常，源码包安装的服务都会在源码包中说明启动方式。

由于源码包的安装位置是由启用自定义的，所以默认情况下，无法使用 rpm 服务的管理命令，但是只要将源码包中的启动脚本文件放置到 `/usr/lib/systemd/system/` 下，就可以使用 rpm 服务的管理命令了，通常的做法是建立软链接，软链接后缀名建议为 service，便于识别和管理。
