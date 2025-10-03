---
title: Linux笔记（日志管理、面试题）
published: 2025-10-01
updated: 2025-10-01
description: 'Linux日志管理、Linux相关面试题目'
image: ''
tags: [linux]
category: 'OS'
draft: false 
---

# Linux笔记

## 日志管理

### 基本介绍

日志记录很多重要的系统事件，包括用户的登录信息、系统的启动信息、系统的安全信息、邮件的相关信息、各种服务的相关信息

系统的相关日志一般被保存在 /var/log/ 目录下，以CentOS为例![228](../images/228.png)



### 日志管理服务

CentOS7.6的日志服务是rsyslogd相比于国王的syslogd更为强大，在运行的时候rsyslogd程序帮助我们记录日志

查看rsyslogd服务是否启用

```
ps -ef | grep "rsyslog" | grep -v "grep" 
```

查询服务的自启动状态

```
systemctl list-unit-files | grep rsyslog
```



### 日志配置文件

/etc/rsyslog.conf 文件是该程序的配置文件，在这里配置日志记录的位置![229](../images/229.png)

日志级别![230](../images/230.png)



### 日志轮替

我们记录日志不可能一直保留记录的日志，我们需要按照一定策略管理超时的日志，防止日志过大，或者将日志放到别处

通过修改 /etc/logrotate.conf 配置文件来进行全局日志轮替策略的修改

weekly 	保存四份日志文件，当新的日志文件建立的时候，旧的将会被删除

rotate 4	在日志轮替后，创建新的空日志文件

create		使用日期作为日志轮替后的后缀

dateext		日志文件是否压缩，如果取消注释，则日志在转储的同时进行压缩

也可以单独指定一个日志的配置，用大括号进行配置，或者把轮替规则写到/etc/logrotate.d目录下

![231](../images/231.png)



### 内存日志

journalctl	可以查看内存日志，默认查看全部

-n 3	查看最新三条内存日志

--since 19:00 --until 19:10:10	查看起始时间到结束时间的日志，可追加日期

-p err	查看报错日志

-o verbose	日志详细内容

_PID=1245 _COMD=sshd	查看特定程序的日志



## 面试题集

有这么一个t.txt 文件其中有互联网的访问请求，需要对于ip提出并且统计，要求按照从出现次数从大到小排序

```
https://189.0.9.1/index1.html  
https://github.com/index2.html  
https://189.0.9.1/index3.html  # 重复 IP  
https://189.0.9.1/index4.html  # 重复 IP  
https://example.com/index5.html  
https://203.0.113.1/index6.html  
https://203.0.113.1/index7.html  # 重复 IP  
https://192.168.1.1/index8.html  
https://192.168.1.1/index9.html  # 重复 IP  
https://10.0.0.1/index10.html  
https://10.0.0.1/index11.html  # 重复 IP  
https://172.16.0.1/index12.html  
https://172.16.0.1/index13.html  # 重复 IP 
```

```
cat t.txt | cut -d '/' -f 3 | sort | uniq -c | sort -nr
```

cut -d '/' -f 3代表按照/分隔符进行分割 取出第三个部分



我们需要从其中获取到ip，我们需要使用awk进行对于空格的分割

```sh
[root@iv-ye5w83dog0bw80eugddg lory]# netstat -an | grep ESTABLISHED
tcp        0      0 192.168.95.8:22         119.112.204.254:54359   ESTABLISHED
tcp        0      0 192.168.95.8:55054      128.1.164.196:443       ESTABLISHED
tcp        0     36 192.168.95.8:22         223.104.204.225:49676   ESTABLISHED
tcp        0      0 192.168.95.8:39136      100.64.205.163:80       ESTABLISHED
```

最终指令

```sh
netstat -an | grep ESTABLISHED | awk -F " " '{print $5}' | cut -d ':' -f 1
```

```
175.174.139.246
128.1.164.196
223.104.204.225
100.64.205.163
```

