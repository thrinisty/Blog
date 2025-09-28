---
title: Linux笔记（磁盘分区、进程管理）
published: 2025-09-27
updated: 2025-09-27
description: 'Linux磁盘分区挂载、Linux进程管理'
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
