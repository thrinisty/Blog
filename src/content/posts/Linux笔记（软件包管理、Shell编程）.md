---
title: Linux笔记（软件包管理、Shell编程）
published: 2025-09-30
updated: 2025-09-30
description: 'rpm包管理、Shell编程快速入门'
image: ''
tags: [linux]
category: 'OS'
draft: false 
---

# Linux笔记

## 软件包管理

### rpm指令

rpm -qa 查询当前系统安装的所有rpm软件包

rpm -q 软件包名	查询是否安装对应软件包

rpm -qi 软件包名	查询软件包信息

rpm -ql 软件包名	查询软件包中的文件

rpm -qf 文件全路径名	查询文件所属软件包

rpm -ivh rpm包全路径	i：安装		v：提示		h：进度条

rpm -e  [--nodeps]  rpm包名称	删除软件包	可以加上可选项强制卸载

我使用的Debian 系（Ubuntu、Debian）Ubuntu一般用的是apt来管理软件，rpm一般管理 Red Hat 系（RHEL、CentOS、Fedora）



### yum指令

yum是一个Shell前端软件包管理器，基于RPM包管理，可以从指定的服务器上自动下载rpm包并安装，可以自动处理依赖关系，可以一次性安装所有依赖的软件包

yum list 	显示yum服务器的软件列表，结合grep进行过滤筛选

yum install yum软件包	下载安装对应的软件



### 配置环境变量

/etc/profile 中配置环境变量

```
export JAVA_HOME=/usr/local/java/jdk1.8.0_261
export PATH=$JAVA_HOME/bin:$PATH
```

source /etc/profile使得环境变量生效

之后我们通过echo $JAVA_HOME就可以得到home目录，PATH中也有java jdk的bin了



## Shell编程

Shell是一个命令行解释器，为用户提供了一个向Linux内核发送请求以便运行程序的界面系统级程序，可以通过Shell启动、挂起、停止甚至编写一些程序



### 执行方式

1.脚本以#!/bin/bash开头

2.脚本有可执行权限

通过相对或者绝对路径即可运行其中内容

如果没有执行权限我我们也可以通过sh指令来执行sh脚本文件



### Shell变量

Linux Shell中的变量分为系统变量和用户自定义变量

系统变量

set 指令：显示当前Shell中的所有系统变量

```
$HOME,$PWD,$SHELL,$USER等
```



**基本语法**

撤销变量：unset 变量

定义变量：变量=值

声明静态变量：readonly 变量=值，不可unset

```sh
A=100
echo A=$A
```

```
./test.sh 或 sh test.sh
```

```
A=100
```

将命令的返回值赋给变量

```sh
A=`cal`
A=$(cal)
```

注释

```sh
:<<!
A=`cal`
A=$(cal)
!
```



**命名规范**

1.变量名称可以由字母，数字，下划线组成，不能以数字开头

2.等号两侧不能为空格

3.变量名称一般习惯为大写



### 环境变量

**基本语法**

export 变量名=变量值

source 配置文件

echo $变量名



**位置参数变量**

当我们执行Shell脚本的时候，我们如果希望获取到命令行参数信息，就可以使用到位置参数变量，如./myshell 100 200 在执行的时候就可以在脚本中使用位置参数

**基本语法**

$n

```
$1-$9代表第一到第九个参数，十个以上的参数需要使用大括号包围${10}
```

$* 	代表命令行中所有的参数，把所有参数看做是一个整体

$@	代表命令行中所有参数，把每个参数区分对待

$#	代表命令行中所有的参数个数



### 预定义变量

是Shell设计者事先定义好的变量，可以在Shell脚本中使用

```
$$	显示当前进程的PID
$!	后台运行的最后一个进程的PID
$?	最后一次执行的命令的返回状态，如果变量为0代表上一个命令正确执行
```



### 编程语法

**运算符**

```
$[运算式] 或  $((运算式)) 或  expr m + n
expr \*, /, %	乘除取余
```



**条件语句**

condition前后需要有空格，返回非空返回为true，可以用$?验证，（0为true， >1为false）

```
[ condition ]
```

常用判断条件

1.字符串比较	=

2.整数比较	-lt：小于，-le：小于等于，-eq：等于，-gt：大于，-ge：大于等于，-ne：不等于 

3.按照文件权限进行判断	-r 有读权限，-w 有写权限，-x 有执行权限

4.按照文件类型进行判断	-f 文件存在且是一个常规文件，-e 文件存在， -d 文件存在并且是一个目录

```
[ -d /root ]
```



**流程控制**

if语句

```
if [ condition1 ]
then
	代码1
elif [ condition2 ]
then
	代码2
else
	代码3
fi
```

```
grade=$1
if [ $grade -ge 60 ]
then    
        echo "及格"
elif [ $grade -lt 60 ]
then
        echo "不及格"
fi
```

case语句

```sh
case $1 in
"1")
echo "周一"
;;
"2")
echo "周二"
;;
*)
echo "other"
;;
esac
```



for循环

语法1：有点类似于增强for循环

```
for 变量 in 值1 值2 值3
do
代码
done
```

例如

```sh
for i in $@
do
echo $i
done
```

语法2：

```
for (( 初始值; 循环控制条件; 变量变化 ))
do
代码
done
```

```sh
for (( i=0; i<10; i++ ))
do
        echo $i
done
```



while循环

```
while [ condition ]
do
code
done
```

```sh
sum=0
i=1
while [ $sum -lt 100 ]
do
        sum=$((sum + i))
        i=$((i + 1))
done
echo $sum
```



read输入

基本语法：read(选项)(参数)

-p：指定读取值时的提示符；

-t：指定读取值时等待的时间（s），如果没有能够在指定时间内输入就不再等待

```sh
read -p "请输入数字num1：" NUM1
read -p "请输入数字num2：" -t 10 NUM2
echo $((NUM1 + NUM2))
```



### Shell函数

又分为系统函数和自定义函数

**系统函数**

常见的系统函数：basename、dirname

basename：

basename返回完整路径最后/的部分，如果suffix被指定了，那么basename将会把pathname或string中的suffix删除

```
basename [pathname][suffix]
```

```
basename /home/lory/test.txt .txt
返回test
```

dirname：

回返回完整路径最后的/的前边的部分

```
dirname /home/1.txt
/home
```



**自定义函数**

基本语法

```
定义：
[ function ] funname[()]
{
	Action;
	[return int;]
}

调用：
funname [值]
```

```sh
function getSum() {
        SUM=$(($1 + $2))
        echo "和为:$SUM"
}

n1=10
n2=20
getSum $n1 $n2
```

