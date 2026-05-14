---
title: MySQL开启 关闭 重启命令
date: 2020-08-10 15:10:00
description: MySQL开启 关闭 重启命令
slug: MySQL-on-off-restart-command
image:
categories:
    - MySQL
tags: ["MySQL"]

---

## 启动

1、使用 service 启动

```shell
service mysql start
```

2、使用 mysqld 脚本启动

```shell
/etc/inint.d/mysql start
```

3、使用 safe_mysqld 启动

```shell
safe_mysql&
```

<!--more--> 

## 停止

1、使用 service 启动

```shell
service mysql stop
```

2、使用 mysqld 脚本启动

```shell
/etc/inint.d/mysql stop
```

3、mysqladmin shutdown

## 重启

1、使用 service 启动

```shell
service mysql restart
```

2、使用 mysqld 脚本启动

```shell
/etc/inint.d/mysql restart
```
