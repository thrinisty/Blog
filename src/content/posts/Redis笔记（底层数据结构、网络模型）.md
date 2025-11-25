---
title: Redis笔记（底层数据结构、网络模型）
published: 2025-11-25
updated: 2025-11-25
description: '底层数据结构、网络模型'
image: ''
tags: [Redis]
category: ''
draft: false 
---

# Redis笔记

## 数据结构

### 动态字符串

**SDS** 简单动态字符串

Redis保存的Key是字符串，value往往是字符串或者字符串的集合，所以字符串是Redis中最重要的一个部分，它不用C语言的字符串，因为C语言的字符串存在缺陷：1.获取字符串长度需要通过运算，2.非二进制安全，3.不可修改

![421](../images/421.png)

预分配

![422](../images/422.png)



### 整数集合

**IntSet** 

IntSet是Redis的Set集合的一种实现方式，基于整数数组的来实现的，并且具备长度可变，有序等特性，查询采用二分查找的方式

![423](../images/423.png)

升级编码

![424](../images/424.png)



### 字典

**Dict** 

Redis是一个KV类型的数据可，可以根据键快速实现增删改查，而键值之间的映射关系就是通过Dick来实现的

Dict由三部分组成：哈希表（DictHashTabale）、哈希节点（DictEntry）、字典（Dict）

![425](../images/425.png)

单向Dict添加键值对的时候，Redis首先根据key计算出hash值（h），然后利用h&sizemask来计算元素应该存储到数组中的哪个索引位置

Dict结构如下，包含了两个数组

![426](../images/426.png)

当集合中元素较多的时候，哈希冲突会增多，链表增长，查询效率降低

扩张

每次新增时候会检查负载因子used/size，满足负载因子大于等于1（且服务器没有执行BGSAVE和BGREWRITEAOF等后台进行）、负载因子大于5

![427](../images/427.png)

收缩

![428](../images/428.png)



**rehash**

通过另一个dicthashtable的数组创建新的Entry数组，讲各个KV重新指向，最后将第一个dicthashtable重新指向新创建的数组就可以完成扩容

![429](../images/429.png)

![431](../images/431.png)



### 压缩链表

**ZipList**

是一种特殊的“双端链表”，有一系列特殊的编码的连续内存快组成，可以在任意一端进行压入/弹出，并且实践操作复杂度位O（1）

![432](../images/432.png)

![433](../images/433.png)

Entry内部结构

![434](../images/434.png)



### 快速链表

**QuickList**

压缩链表虽然节省内存，但是申请需要连续空间，如果内存的占用比较多，申请效率很低，这个时候可以去申请分片，分为多个ZipList，但是并不利于管理，我们使用到QuickList进行统一管理

![435](../images/435.png)

![436](../images/436.png)



### 跳表

**SkipList**

是链表，但是与传统链表有所差异：

1.按照升序排列存储

2.节点可能包含多个指针，指针跨度不同

![437](../images/437.png)

![438](../images/438.png)



### Redis对象

**RedisObject**

Redis中的任意数据类型的KV都会被封装为RedisObject，称作Redis对象

![439](../images/439.png)



## 数据类型

### String

RAW基于SDS实现，上限为512MB

![440](../images/440.png)

EMBSTR编码：当SDS长度小于44字节，这个时候ObjectHead和SDS是一段连续的内存空间

![441](../images/441.png)

INT编码

![442](../images/442.png)



### List

Redis中的List可以从头尾查找数组，以下是几种实现思路

![443](../images/443.png)

![444](../images/444.png)

![445](../images/445.png)



### Set

Redis中的单列集合，不保证有序性、保证元素唯一、可求并交差集

底层是HashTable，也就是RedisDict，不过Dick是双列集合

HT编码：这里为了查询效率和唯一性，set采用的是HashTable（Dict）Dict中的key用来存储元素，value统一为null

IntSet编码：如果存储所有数据是整数，当元素数量不超过set-max-inset-entries（512）时，Set采用IntSet编码来节省内存



### ZSet

ZSet是SortedSet，其中每一个元素都需要制定一个score和member值

1.可以根据score值排序后

2.member必须唯一

3.可以根据member查询分数

![446](../images/446.png)

![447](../images/447.png)

底层结构

第一种：SkipList 跳表+字典（效率高、内存大）

![448](../images/448.png)



第二种：ZipList 

![449](../images/449.png)

![450](../images/450.png)



### Hash

Hash结构和ZSet非常类似

1.都是键值存储

2.都是根据键获取值

3.键必须唯一

区别是：zset的键是member，值是score；hash的键值都是任意值

zset根据score排序，hash无需排序

![451](../images/451.png)

![452](../images/452.png)



## 网络模型

用户空间和内核空间

![453](../images/453.png)



### 五种IO模型

#### 阻塞IO（BIO）

![454](../images/454.png)

#### 非阻塞IO（NIO）

![455](../images/455.png)

#### IO多路复用

![456](../images/456.png)

![457](../images/457.png)

![458](../images/458.png)

常见的有三种多路复用

![459](../images/459.png)

**select**

每次执行select都需要拷贝fd_set，select无法得知是哪一个fd就绪，需要遍历set，fd_set监听的fd数量不能够超过1024

![460](../images/460.png)

**poll**

对于select进行简单的改进，但是性能不明显

![461](../images/461.png)

**epoll**

![462](../images/462.png)

**总结**

![463](../images/463.png)



**LT模式和ET模式**

当FD有数据可读的时候，我们调用epoll_wait就可以得到通知，但是通知的模式有两种分别为：LevelTriggered，简称LT和EdgeTriggered简称EL

LevelTriggered：当FD有数据可读的时候，会通知多次，直到数据处理完成，是epoll的默认模式。

EdgeTriggered：当FD有数据可读的时候，只会通知一次，不管数据是否处理完成

![464](../images/464.png)
