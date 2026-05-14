---
title: 理解MVCC多版本并发控制
date: 2021-10-08 18:10:00
description: 关于InnoDB的MVCC
slug: mysql-mvcc
image:
categories:
    - MySQL
tags: ["MySQL"]
---

事务有四种隔离级别（读未提交、读已提交、可重复读、串行化）和三种异常问题（脏读、不可重复读、幻读）

### 什么是MVCC?

MVCC是通过数据行的多个版本管理来实现数据库的并发控制

**使用MVCC的好处：** 

1、读写互相不阻塞. 2、降低了死锁的概率。3、解决一致性读的问题。

**快照读：** 读取的是快照数据（历史版本）

**当前读：** 读取最新的数据，加锁的SELECT、或者插入，删除，更新都会进行当前读











![img](https://static001.geekbang.org/resource/image/4f/5a/4f1cb2414cae9216ee6b3a5fa19a855a.jpg)









事务有四种隔离级别（读未提交、读已提交、可重复读、串行化）和三种异常问题（脏读、不可重复读、幻读）







