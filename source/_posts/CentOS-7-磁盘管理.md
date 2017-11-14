---
title: CentOS_7_磁盘管理
date: 2017-09-17 22:58:41
category: Linux
tags: CentOS 7
---

<!-- toc -->

---

# 1. 分区简介

磁盘分区时使用分区编辑器（partition editor）在磁盘上划分几个逻辑部分。磁盘一旦被划分成数个分区（partition），不同类的目录与文件可以存储进不同的分区。

# 2. 设备相关基本信息查询

## 2.1 查看块设备信息

了解新插入的设备的名字，特别是当你在终端上处理磁盘/块设备时。

### 2.1.1 lsblk

lsblk 是一个 Linux 工具，它会显示有关你系统里所有可用块设备的信息。它从 sysfs 文件系统 中获取信息。默认情况下，这个工具将会以树状格式显示（除了内存虚拟磁盘外的）所有块设备。

格式：lsblk [选项]

选项：

- 省略，即默认值，此时以标准的树状格式输出,整齐地显示块设备
- -l 以列表的形式输出块设备信息
- -n 不显示表头
- -m 显示块设备所有者的相关信息（包括文件的所属用户、所属组以及文件系统挂载的模式）
+ -f 在输出结果中增加文件系统列，其他列也会有所改变
    - FSTYPE: 设备对应的文件系统
    - LABEL: 设备的 label 说明
    - UUID: 设备的唯一标识符
- -t 以拓扑的形式输出块设备信息，对应的输出信息于之前不同，具体不再说明

下面以默认无参的命令作简要说明：

```
[root@localhost ~]# lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda               8:0    0    5G  0 disk
├─sda1            8:1    0  200M  0 part /boot
└─sda2            8:2    0  4.8G  0 part
  ├─centos-root 253:0    0  3.8G  0 lvm  /
  └─centos-swap 253:1    0    1G  0 lvm  [SWAP]
sr0              11:0    1  7.7G  0 rom
```

说明：

- NAME: 设备的名称
- MAJ:MIN: Linux 操作系统中的每个设备都以一个文件表示，对块（磁盘）设备来说，这里用主次设备编号来描述设备
- RM: 是否为可移动设备。如果这是一个可移动设备将显示 1，否则显示 0
- SIZE: 设备的容量
- RO: 对于只读文件系统，这里会显示 1，否则显示 0
- TYPE: 设备的类型
- MOUNTPOINT: 设备挂载的位置

### 2.1.2 blkid

blkid 主要用来对系统的块设备（包括交换分区）所使用的 FSTYPE、LABEL、UUID等信息进行查询。

格式：blkid [选项] [参数] [dev]

选项：

- 全部省略，即默认值，输出系统中所有块设备的基本信息
- /dev/sda1: 根据设备名查询其他几项信息
- -s tag [dev]: 查询出[指定/所有]带 tag 的设备名和 tag 值，常见 tag 有：UUID, LABEL, TYPE
- -U UUID: 根据 UUID 查询出对应的设备名
- -L LABEL: 根据 LABEL 查询出对应的设备名
- -po format dev: 按照指定格式（format:value/device/udev/full）输出指定设备的详细信息
- -o format [dev]: 按照指定格式（format:value/device/list/export/full）输出[指定/所有]设备的信息
- -g: 清除设备列表缓存，刷新当前设备列表

## 2.2 查看磁盘分区基本信息

格式：df [选项] [已挂载分区/文件系统名]

选项：

- -l 仅显示本地磁盘（默认）
- -a 显示所有文件系统的磁盘使用情况
- -h 以 1024 进制计算最合适的单位显示磁盘容量
- -H 以 1000 进制计算最合适的单位显示磁盘容量
- -T 显示磁盘分区类型
- -t 后跟文件系统名，显示指定类型文件系统的磁盘分区
- -x 后跟文件系统名，不显示指定类型文件系统的磁盘分区

## 2.3 统计磁盘上文件的大小

格式：du [选项] [文件/目录]

选项：

- -b 以 byte 为单位统计文件文件大小
- -k 以 KB 为单位统计文件大小
- -m 以 MB 为单位统计文件大小
- -h 按照 1024 进制以最合适的单位统计文件大小
- -H 按照 1000 进制以最合适的单位统计文件大小
- -s 指定统计目标，如目录、文件等

# 3. 磁盘的分区、格式化和挂载/卸载

Linux 系统中的硬件设备都是以文件的形式存在于根目录下的 dev 目录中，由 Linux 系统自动识别，对于磁盘，其文件命名结构为：sd[a-z]X

- sd：表示接口类型为 SATA 或 SCSI 的硬盘，对于 IDE 接口为 hd
- [a-z]：表示硬盘编号，分别对应第 1 到 26 块硬盘
- X：表示当前硬盘下的分区编号，

虽然对于一块全新的硬盘，Linux 开机后能自动识别但并不能直接使用，必须对硬盘进行分区、格式化和挂载后才能正常使用。

## 3.1 分区

### 3.1.1 MBR 分区

MBR 分区模式特点：

- 一块硬盘的主分区 + 扩展分区最多只能有 4 个，对应的分区编号为 1-4
- 扩展分区其实也是主分区，为了增加分区个数，将一个主分区作为扩展分区使用，但一个硬盘的扩展分区最多只能有 1 个
- 扩展分区不能写入数据，只能包含若干个逻辑分区
- 逻辑分区对应的编号从 5 开始
- 单个分区容量最大为 2TB

使用 Linux 自带的磁盘分区工具：fdisk，命令帮助如下：

```shell
[root@localhost ~]# fdisk
Usage:
 fdisk [options] <disk>    change partition table
 fdisk [options] -l <disk> list partition table(s)
 fdisk -s <partition>      give partition size(s) in blocks

Options:
 -b <size>             sector size (512, 1024, 2048 or 4096)
 -c[=<mode>]           compatible mode: 'dos' or 'nondos' (default)
 -h                    print this help text
 -u[=<unit>]           display units: 'cylinders' or 'sectors' (default)
 -v                    print program version
 -C <number>           specify the number of cylinders
 -H <number>           specify the number of heads
 -S <number>           specify the number of sectors per track
```

使用 `fdisk -l` 命令显示系统当前分区表信息：

```shell
[root@localhost ~]# fdisk -l

Disk /dev/sdb: 5368 MB, 5368709120 bytes, 10485760 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sda: 21.5 GB, 21474836480 bytes, 41943040 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000b9672

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     1026047      512000   83  Linux
/dev/sda2         1026048    26191871    12582912   83  Linux
/dev/sda3        26191872    30386175     2097152   82  Linux swap / Solaris
/dev/sda4        30386176    41943039     5778432    5  Extended
/dev/sda5        30388224    41938943     5775360   83  Linux
```

其中，/dev/sda 为系统的第一块硬盘，是用于安装系统和存储相关数据的主硬盘，/dev/sdb 为后来新增的第二块硬盘，现在需要对此硬盘进行分区操作，具体流程如下：

##### 1. 进入 fdisk 模式

```shell
[root@localhost ~]# fdisk /dev/sdb
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table
Building a new DOS disklabel with disk identifier 0x97759901.

Command (m for help):
```

在 fdisk 的分区模式下，通过输入 m 查看相关的命令帮助，主要的分区命令都可通过帮助信息找到。

```shell
Command (m for help): m
Command action
   a   toggle a bootable flag
   b   edit bsd disklabel
   c   toggle the dos compatibility flag
   d   delete a partition
   g   create a new empty GPT partition table
   G   create an IRIX (SGI) partition table
   l   list known partition types
   m   print this menu
   n   add a new partition
   o   create a new empty DOS partition table
   p   print the partition table
   q   quit without saving changes
   s   create a new empty Sun disklabel
   t   change a partition's system id
   u   change display/entry units
   v   verify the partition table
   w   write table to disk and exit
   x   extra functionality (experts only)
```

##### 2. 新建分区

###### 2.1 新建主分区/扩展分区

```shell
Command (m for help): n
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p):
```

- p：主分区，默认值
- 0 primary, 0 extended, 4 free：当前硬盘的分区情况，0 个主分区、0 个扩展分区，4 个分区可用
- e：扩展分区

```shell
Select (default p):
Using default response p
Partition number (1-4, default 1): 1
```

- Partition number (1-4, default 1)：指定分区号，1-4 是系统预留给主分区和扩展分区使用的

###### 2.2 新建逻辑分区

新建的扩展分区并不能直接使用，需要在此基础上建立逻辑分区，逻辑分区的分区编号是从 5 开始的。当建立了扩展分区后，再次使用 n 命令会出现如下信息：

```shell
Command (m for help): n
Partition type:
   p   primary (1 primary, 1 extended, 2 free)
   l   logical (numbered from 5)
Select (default p):
```

此时只需要输入 `l` 即可创建逻辑分区，其余操作同创建主分区。

```shell
First sector (2048-10485759, default 2048):
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-10485759, default 10485759): +2G
Partition 1 of type Linux and of size 2 GiB is set
```

- 指定分区的起始位置

```shell
Command (m for help): p

Disk /dev/sdb: 5368 MB, 5368709120 bytes, 10485760 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x97759901

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048     4196351     2097152   83  Linux

Command (m for help):
```

- p 命令：查看当前磁盘下的分区信息

```shell
Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
```

当硬盘分区完成，使用 p 命令查看无误后，使用 w 命令将硬盘的分区信息写入到系统的分区表（partition table）中，此时，使用 `fdisk -l` 命令就可以看到刚才的分区信息了。

### 3.1.2 GPT 分区

GPT 分区模式特点：

- 主分区个数最大为 128 个，由于此分区模式下的分区数量足够使用，此时扩展分区用于增加分区数量的作用相对弱化，故在 pgt 分区模式下，不严格区分主分区和扩展分区
- 单个分区的容量最大为 18EB (1EB = 1024PB = 1024TB)
- GPT 的主分区中不适合安装 X86 架构的系统

fdisk 命令只能用于创建 MBR 分区模式的分区，所以，此处采用系统自带的 parted 分区工具，此命令可以指定分区的模式为 MBR 或 GPT 等。

##### 1. 进入 parted 模式

```shell
[root@localhost ~]# parted
GNU Parted 3.1
Using /dev/sda
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted)
```

输入 help 查看相关的命令帮助，主要的分区命令都可通过帮助信息找到。

```shell
[root@localhost ~]# parted
GNU Parted 3.1
Using /dev/sda
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) help
  align-check TYPE N                        check partition N for TYPE(min|opt)
        alignment
  help [COMMAND]                           print general help, or help on COMMAND
  mklabel,mktable LABEL-TYPE               create a new disklabel (partition
        table)
  mkpart PART-TYPE [FS-TYPE] START END     make a partition
  name NUMBER NAME                         name partition NUMBER as NAME
  print [devices|free|list,all|NUMBER]     display the partition table, available
        devices, free space, all found partitions, or a particular partition
  quit                                     exit program
  rescue START END                         rescue a lost partition near START and
        END
  rm NUMBER                                delete partition NUMBER
  select DEVICE                            choose the device to edit
  disk_set FLAG STATE                      change the FLAG on selected device
  disk_toggle [FLAG]                       toggle the state of FLAG on selected
        device
  set NUMBER FLAG STATE                    change the FLAG on partition NUMBER
  toggle [NUMBER [FLAG]]                   toggle the state of FLAG on partition
        NUMBER
  unit UNIT                                set the default unit to UNIT
  version                                  display the version number and
        copyright information of GNU Parted
(parted)
```

输入 print 查看当前选中硬盘（默认为 sda）的分区相关信息， 输入 print all 查看所有硬盘的分区相关信息。

```shell
(parted) print all
Model: VMware, VMware Virtual S (scsi)
Disk /dev/sda: 21.5GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags:

Number  Start   End     Size    Type      File system     Flags
 1      1049kB  525MB   524MB   primary   ext4            boot
 2      525MB   13.4GB  12.9GB  primary   ext4
 3      13.4GB  15.6GB  2147MB  primary   linux-swap(v1)
 4      15.6GB  21.5GB  5917MB  extended
 5      15.6GB  21.5GB  5914MB  logical   ext4


Model: VMware, VMware Virtual S (scsi)
Disk /dev/sdb: 5369MB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags:

Number  Start   End     Size    Type      File system  Flags
 1      1049kB  2149MB  2147MB  primary
 2      2149MB  5369MB  3220MB  extended
 5      2150MB  3223MB  1074MB  logical
 6      3224MB  5369MB  2144MB  logical


Error: /dev/sdc: unrecognised disk label
Model: VMware, VMware Virtual S (scsi)
Disk /dev/sdc: 5369MB
Sector size (logical/physical): 512B/512B
Partition Table: unknown
Disk Flags:

```

##### 2. 选择需要进行分区的硬盘

```shell
(parted) select /dev/sdc
```

##### 3. 指定分区的模式

```shell
(parted) mklabel gpt
```

通过帮助可知：mklabel 和 mktabel 都可以用于指定分区的模式，两者是等价的，后面紧跟分区的模式类型，这是指定为 gpt，如果需要指定分区的模式类型为 MBR，可使用 msdos

##### 4. 创建分区

在 parted 工具中，它提供了两种操作模式：交互和命令。交互模式同 fdisk，通过输入简单命令，然后根据工具提示完成操作；命令模式是直接使用完整命令一次性完成分区创建。

- 交互模式

```shell
(parted) mkpart
Partition name?  []? test1
File system type?  [ext2]?
Start? 1
End? 3000
(parted) print
Model: VMware, VMware Virtual S (scsi)
Disk /dev/sdc: 5369MB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name   Flags
 1      1049kB  3000MB  2999MB               test1
```

- 命令模式

```shell
(parted) mkpart test2 3000 4000
(parted) print
Model: VMware, VMware Virtual S (scsi)
Disk /dev/sdc: 5369MB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name   Flags
 1      1049kB  3000MB  2999MB               test1
 2      3000MB  4000MB  1000MB               test2
```

说明：

- Partition name：分区名，默认为空，命令模式下必须输入分区名
- File system type：文件系统，默认为 ext2
- Start 和 End：起始位置，默认单位为 Mb，区别于 fdisk 中的起始位置，fdisk 中的起始位置输入的是扇区数据块的编号，此处是从 Start Mb 到 End Mb
- 区别于 fdisk，parted 工具分区完成后，不需要额外的保存的命令来使用创建分区生效，它是立即生效的

特殊情况：

```shell
(parted) mkpart test3 3500 5000
Warning: You requested a partition from 3500MB to 5000MB (sectors
6835937..9765625).
The closest location we can manage is 4000MB to 5000MB (sectors
7813120..9765625).
Is this still acceptable to you?
Yes/No? y

(parted) mkpart test4 5300 5000
Error: Can't have the end before the start! (start sector=10351562 length=-585936)

(parted) mkpart test4 5000 6000
Error: The location 6000 is outside of the device /dev/sdc.
```

当输入的分区开始值与原分区结束值重叠时，工具会自动给出解决方案（将新分区的开始值设置为上一分区的结束值），提示是否接受？

当输入的起始值超出硬盘的容量或者结束值小于开始值，工具会自动报错。

##### 5.其他常用命令

- 修改分区起始值的默认单位：`unit GB`
- 删除分区：`rm number`，number 为分区号，可通过 print 命令查看

## 3.2 格式化

通常使用 mkfs 命令进行分区的格式化，有以下两种方式

```shell
[root@localhost ~]# mkfs.ext4 /dev/sdb1

[root@localhost ~]# mkfs -t ext4 /dev/sdb5
```

说明：

- 格式化是针对主分区和逻辑分区而言的，扩展分区不能进行格式化
- 对于已经格式化的分区，可以通过 parted 工具进行查看，如果该分区已经被挂载，也可通过 `df -T` 查看

## 3.3 挂载/卸载

对于一块硬盘，经过分区、格式化后，仍然是无法使用的，必须将分区挂载，才可以正常使用。理论上，分区的挂载目录是任意的，只要该目录存在即可，但系统默认已经提供了 /mnt 目录专门用于挂载新增的设备（随着系统功能的增强和发行版的衍生，也存在 /media, /misc 等目录，视系统版本而定）方便管理，如无特殊需要，建议使用系统提供的目录。

- 挂载：`mount 设备文件名 挂载目录`，如 `mount /dev/sdb1 /mnt/test1`
- 卸载：`umount 挂载目录`，如 `umount /mnt/test1`

详细帮助信息见 `mount --help`

通过命令的方式进行挂载只是临时的，系统重启就会使挂载失效，要想永久有效，需要修改 /etc/fstab 文件，这样每次系统开机时，都会自动挂载.

| file system | mount point | type | options | dump | pass |
| ----------- | ----------- | ---- | ------- |:----:|:----:|
| /dev/sdb1 | /mnt/test1 | ext4 | defaults | 0 | 0 |

## 3.4 swap 分区

为硬盘添加 swap 分区步骤：

1. 建立一个普通的 Linux 分区
2. 修改分区 Id 的 16 进制编码
3. 格式化 swap 分区
4. 启用 swap 分区

### 3.4.1 修改分区 Id 的 16 进制编码

```shell
[root@localhost ~]# fdisk /dev/sdb
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): t
Partition number (1,2,5,6, default 6): 1
Hex code (type L to list all codes): 82
Changed type of partition 'Linux' to 'Linux swap / Solaris'

Command (m for help): p

Disk /dev/sdb: 5368 MB, 5368709120 bytes, 10485760 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x97759901

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048     4196351     2097152   82  Linux swap / Solaris
```

说明：

- 使用 fdisk 工具的 t 命令修改分区的系统 Id，系统 Id 默认为十六进制数，通过 L 查看对应关系，如下：

```shell
Command (m for help): t
Partition number (1,2,5,6, default 6): 1
Hex code (type L to list all codes): L

 0  Empty           24  NEC DOS         81  Minix / old Lin bf  Solaris
 1  FAT12           27  Hidden NTFS Win 82  Linux swap / So c1  DRDOS/sec (FAT-
 2  XENIX root      39  Plan 9          83  Linux           c4  DRDOS/sec (FAT-
 3  XENIX usr       3c  PartitionMagic  84  OS/2 hidden C:  c6  DRDOS/sec (FAT-
 4  FAT16 <32M      40  Venix 80286     85  Linux extended  c7  Syrinx
 5  Extended        41  PPC PReP Boot   86  NTFS volume set da  Non-FS data
 6  FAT16           42  SFS             87  NTFS volume set db  CP/M / CTOS / .
 7  HPFS/NTFS/exFAT 4d  QNX4.x          88  Linux plaintext de  Dell Utility
 8  AIX             4e  QNX4.x 2nd part 8e  Linux LVM       df  BootIt
 9  AIX bootable    4f  QNX4.x 3rd part 93  Amoeba          e1  DOS access
 a  OS/2 Boot Manag 50  OnTrack DM      94  Amoeba BBT      e3  DOS R/O
 b  W95 FAT32       51  OnTrack DM6 Aux 9f  BSD/OS          e4  SpeedStor
 c  W95 FAT32 (LBA) 52  CP/M            a0  IBM Thinkpad hi eb  BeOS fs
 e  W95 FAT16 (LBA) 53  OnTrack DM6 Aux a5  FreeBSD         ee  GPT
 f  W95 Ext'd (LBA) 54  OnTrackDM6      a6  OpenBSD         ef  EFI (FAT-12/16/
10  OPUS            55  EZ-Drive        a7  NeXTSTEP        f0  Linux/PA-RISC b
11  Hidden FAT12    56  Golden Bow      a8  Darwin UFS      f1  SpeedStor
12  Compaq diagnost 5c  Priam Edisk     a9  NetBSD          f4  SpeedStor
14  Hidden FAT16 <3 61  SpeedStor       ab  Darwin boot     f2  DOS secondary
16  Hidden FAT16    63  GNU HURD or Sys af  HFS / HFS+      fb  VMware VMFS
17  Hidden HPFS/NTF 64  Novell Netware  b7  BSDI fs         fc  VMware
18  AST SmartSleep  65  Novell Netware  b8  BSDI swap       fd  Linux raid auto
1b  Hidden W95 FAT3 70  DiskSecure Mult bb  Boot Wizard hid fe  LANstep
1c  Hidden W95 FAT3 75  PC/IX           be  Solaris boot    ff  BBT
1e  Hidden W95 FAT1 80  Old Minix
```

### 3.4.2 格式化 swap 分区

区别于普通分区的格式化命令 mkfs，此处使用 mkswap 命令

```shell
[root@localhost ~]# mkswap /dev/sdb1
mkswap: /dev/sdb1: warning: wiping old ext4 signature.
Setting up swapspace version 1, size = 2097148 KiB
no label, UUID=cedc88d0-5d2b-46bd-af65-38adecd06901
```

### 3.4.3 启用 swap 分区

```shell
[root@localhost ~]# swapon /dev/sdb1
```

### 3.4.4 查看 swap 的加载情况

```shell
[root@localhost ~]# free /dev/sdb1
              total        used        free      shared  buff/cache   available
Mem:        1001332      120416      741332        6832      139584      735092
Swap:       2097148           0     2097148
```

### 3.4.5 关闭 swap 分区

```shell
[root@localhost ~]# swapoff /dev/sdb1
```

## 3.5 LVM

Blockquote
