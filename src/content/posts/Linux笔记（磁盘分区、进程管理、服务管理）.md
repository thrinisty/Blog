---
title: Linux笔记（磁盘分区、进程管理、服务管理）
published: 2025-09-27
updated: 2025-09-27
description: 'Linux磁盘分区挂载、Linux进程管理、服务管理'
image: ''
tags: [linux]
category: 'OS'
draft: false 
---

# Linux笔记

## Linux分区

Linux无论有多少个分区，分给哪一个目录使用，就只有一个根目录，一个独立且唯一的文件结构，Linux中每个分区都是用来组成整个文件系统的一部分

Linux中采用了一种叫做“载入”的处理方法，它的整个文件系统中包含了一整套的文件和目录，且将一个分区和一个目录联系起来，这时要载入的一个分区将使它的存储空间在下一个目录获得



lsblk （list block）

用于**列出块设备**（如磁盘、分区、RAID、LVM 等）信息的实用命令

通过这条指令可以查看挂载分区和文件的关联

```
root@lavm-5q4wdiilrx:/home/wukong# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
vda    252:0    0  180G  0 disk 
└─vda1 252:1    0  180G  0 part /
```



## Linux挂载硬盘

**分区命令** 

fdisk /dev/sdb（硬盘名称）	可以将硬盘分区，之前也提到了挂载的硬盘会被映射到/dev/目录下

**格式化磁盘**

mkfs -t ext4(分区类型) /dev/sdb1 

**挂载分区**

mount /dev/sdb1（分区名称） /home/newdisk/ 将分区挂载到指定的目录

umount  /dev/sdb1 卸载挂载的分区

:::warning
挂载的关系是临时的，当系统重启的时候挂在关系失效，可以通过一些方式实现永久挂载，要修改/etc/fstab文件，再使用mount -a进行挂载启用
:::

**其他实用指令**

df -h 显示磁盘的使用情况

du -h /目录 显示指定目录的磁盘使用情况，还可以-a列出文件所占的大小



## 进程管理

### 基本介绍

在Linux中，每个执行程序都称为一个进程，每一个进程都分配一个pid

每个进程都可能以两种方式存在，前台和后台。前台是在用户屏幕上可供操作的，后台进程则是实际上在操作，但是屏幕上无法看到的进程，通常使用后台方式执行



### ps指令

用于查看目前系统中，有哪些正在执行，以及他们执行的情况。

ps -a	显示当前终端所有的进程信息

ps -u	以用户格式显示进程信息

ps -x	显示后台进程运行的参数

ps -ef 或 ps -aux 显示所有进程



### 终止进程

kill [选项] 进程号	通过该条指令可以终止进程

killall 进程名称		通过该指令可以终止指定名称的进程

-9：表示强迫进程立即停止

**常见案例**

踢掉某个非法登陆的用户：ps -ef | grep sshd查询到用户进程，再通过kill 进程号杀死对应进程

kill终端：ps -ef | grep bash		kill -9 2825 2839（这里需要带上-9强制删除）

```
lory        2438    2431  0 19:35 pts/0    00:00:00 -bash
root        2590    2589  0 19:38 pts/1    00:00:00 bash
test        2825    2799  0 19:43 pts/2    00:00:00 bash
test        2839    2799  0 19:44 pts/3    00:00:00 bash
root        2851    2590  0 19:44 pts/1    00:00:00 grep --color=auto bash
```



### 查看进程树

pstree [选项] 可以更加直观的查看进程信息

-p	显示进程的PID

-u	显示进程的所属用户



## 服务管理

服务本质上就是进程，但是是运行在后台的，通常会监听某个端口，等待其他程序的请求，例如mysqlld，sshd，防火墙，因此又称为守护进程



### service管理指令

service 服务名称 [start | stop | restart | reload | status]

在CentOS7.0后很多服务改用systemctl管理

service 指令管理服务在/etc/init.d查看

ls -l /etc/init.d 这个指令可以查看服务



### 服务的运行级别

0：关机
1：单用户
2：多用户状态没有网络服务
3：多用户状态有网络服务
4：系统未使用保留给用户
5：图形界面
6：系统重启

其中常用的是3和5，也可以指定默认的运行级别

可以通过systemctl set-default 进行设置

```
sudo systemctl set-default TARGET_NAME.target
```

其中 `TARGET_NAME.target` 是你要设为默认的运行目标。

------

**📌 常用 Target 选项**

| **Target**          | **用途**                            | **对应传统运行级别** |
| :------------------ | :---------------------------------- | :------------------- |
| `multi-user.target` | 命令行模式（无图形界面）            | `runlevel 3`         |
| `graphical.target`  | 图形界面模式                        | `runlevel 5`         |
| `rescue.target`     | 单用户救援模式（类似 `runlevel 1`） | `runlevel 1`         |
| `emergency.target`  | 紧急模式（最基础的 shell）          | `runlevel 1` 变种    |



### systemctl管理指令

systemctl  [start | stop | restart | status]  服务名称（这里和service相反）

管理的服务在/usr/lib/systemd/system 查看



systemctl list-unit-files	查看哪一些服务开机自启动

systemctl enable 服务名	给服务设置开机自启动

systemctl disable 服务名   关闭服务设置开机自启动

systemctl is-enabled 服务名 查看服务是否开机自启动



### 动态监控进程

top指令：和ps命令类似，都是用于显示正在执行的进程，不同之处在于top再执行一段时间可以更新正在运行的进程

top [选项]

-d 秒数	指定top每隔几秒更新，默认为3s

-i	使得top不显示闲置或僵死进程

-p	通过指定监控id，来监视某个进程的状态

在运行其中我们可以使用P按照CPU进行排序、M按照内存排序、N以PID排序、q退出



### 监控网络状态

netstat [选项]

-an 按一定顺序排列输出

-p   显示哪个进行调用
