---
title: CentOS_7_网络管理
date: 2017-09-17 23:01:46
category: Linux
tags: CentOS 7
---

<!-- toc -->

---

# 1. IP

CentOS 7 中默认使用 ip 命令，不在安装 ifconfig。ifconfig 命令属于 net-tools 套件，而 ip 命令属于 iproute 套件。ip 命令和 ifconfig 命令一样，但是功能更加强大，并旨在取代后者。

## 1.1 ip 常见命令

### 1.1.1 设置、查看和删除IP地址

- 设置IP地址：`ip addr add 192.168.1.1/24 dev eth0`
- 查看IP地址：`ip addr show eth0`
- 删除IP地址：`ip addr del 192.168.1.1 dev eth0`

### 1.1.2 查看、修改路由

- 查看路由表：`ip route show`
- 查看路由包来自的接口(本地接口)：`ip route get 123.125.114.144`
- 更改默认路由：`ip route add default via 192.168.1.254`

### 1.1.3 显示网络信息

- 显示网络统计信息：`ip -s link`
- 查看 ARP 条目：`ip neigh(或neighbour)`
- 监控 netlink 消息：`ip monitor all`

### 1.1.4 激活或停止网络接口

- 激活网络接口：`ip link set eth0 up`
- 停止网络接口：`ip link set eth0 down`

## 1.2 绑定静态 IP 地址

通过修改配置文件来修改 ip，配置文件地址为：`/etc/sysconfig/network-scripts/ifcfg-eno16777736` 。如下为常见的需要新增／修改的内容：

```
# 修改如下配置
BOOTPROTO="static"        # 启用静态IP地址，默认为 dhcp
ONBOOT="yes"              # 自动开启网络服务，如果此项为 no，需要改成 yes
# 增加如下配置
IPADDR="192.168.1.10"    # 设置IP地址
PREFIX="24"              # 设置子网掩码，也可以使用 NETMASK="255.255.255.0"
GATEWAY="192.168.1.1"    # 设置网关
BROADCAST="192.168.1.255" # 广播地址，可选，省略即采用默认值
DNS1="8.8.8.8"            # 设置主 DNS 
DNS2="8.8.4.4"          　# 设置备 DNS（可选） 
```

说明：

- 配置文件的文件名可能会因为个人环境不同而有所区别，但命名格式为：`ifcfg-enXXXXX`
- 如果需要配置多个 ip，可以使用 IPADDR0/PREFIX0/GATEWAY0，IPADDR1/PREFIX1/GATEWAY1... 分组的方式进行多个 ip 的配置
- 修改此配置文件前，建议备份原配置文件
- 修改配置文件后，需要重启网络服务使修改生效，命令为：`systemctl restart network`

# 2. 防火墙

区别于之前的版本，从 CentOS 7.0 开始默认使用的是 firewall 作为防火墙工具，对于旧版本的 iptables，虽然 CentOS 7 之后的版本仍然支持，但需要另行安装，此处不再赘述。 

## 2.1 基本命令

### 2.1.1 系统管理命令

- 启动/关闭/重启防火墙：`systemctl start/stop/restart firewalld[.service]`
- 显示防火墙状态：`systemctl status firewalld`
- 开机启动/禁用防火墙：`systemctl enable/disable firewalld`
- 查看防火墙是否开机启动：`systemctl is-enabled firewalld`

### 2.1.2 firewall 命令

+ 开启指定端口：`firewall-cmd --zone=scope --add-port=port/protocol --permanent`
    - `--zone`: 作用域，通常为 public
    - `--add-port`: 端口号/协议，如：8080/tcp, 3306/tcp 等
    - `--permanent`: 可选项，永久生效，没有此参数重启后失效
- 查看已开放端口：`firewall-cmd --list-ports`
- 刷新防火墙配置（在修改防火墙配置后用于使修改生效）：`firewall-cmd --reload`
