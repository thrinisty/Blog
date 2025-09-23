---
title: Linux笔记（组管理）
published: 2025-09-23
updated: 2025-09-23
description: '组管理和权限管理'
image: ''
tags: [linux]
category: 'OS'
draft: false 
---

# Linux笔记

## 组管理

在linux中每个用户必须属于一个组，不能独立于组外

在linux中文件有所有者、所在组、其他组的概念



### 所有者

一般为文件的创建者，谁创建了该文件，就自然的成为了该文件的所有者

可以通过指令	ls -ahl 	查看文件的所有者

也可以通过指令修改文件的所有者	chown 用户名 文件名



### 所在组 	

当一个用户创建了一个文件，这个文件所在组就是该用户所在的组

可以通过指令	chgrp 组名 文件名	来修改文件的所在组



### 其他组	

除了文件的所有者和所在组的用户外，系统的其他用户都是文件的其他组



有关于组的实用指令

通过 usermod -g  组名  用户名可以改变用户所在组

还可以通过usermod -d  目录名称  用户名 	来改变用户登陆到的初始目录（要求该用户有权限进入目标的目录）





## 权限管理

### 文件类型

第0为指的是文件的类型（d、-、l、c、b）

d为目录	l是链接	c是字符设备	b是块设备	-是普通文件

```
fox@lavm-5q4wdiilrx:~$ ls -al
total 24
drwxr-xr-x  6 root      root      4096 Sep 23 16:53 .
drwxr-xr-x 19 root      root      4096 Sep 19 15:50 ..
drwxr-x---  3 fox       sudo      4096 Sep 23 17:07 fox
drwxr-x---  2 lory      lory      4096 Sep 19 15:12 lory
drwxr-x---  3 test      admin     4096 Sep 19 16:59 test
drwxr-x---  3 thrinisty thrinisty 4096 Sep 19 17:01 thrinisty
```



### 文件权限

1-9为是文件的权限，分别对应着所有者、所在组、其他组对该文件的权限

**修改文件权限** 	chmod （change mode）

可以通过chmod 744 文件名 对文件所有者赋予读写权限（110）	对所在组用户赋予读权限（100）	对其他组赋予读权限（100）

另外一种赋（夺）权方式

chmod u+x 文件	对所有者附加x（执行）权限

chmod g-r  文件	对所有组的用户删除r（读）权限

chmod o+w 文件	对其他组增加w（写）权限

chmod a-x   文件	全部用户删除可执行权限

![34](../images/34.png)
