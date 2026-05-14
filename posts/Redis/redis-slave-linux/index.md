---
title: 在一台Linux服务器上部署多个Redis实现master-slave
description: 操作在一台Linux服务器上部署多个Redis实现master-slave
date: 2021-07-13 15:10:00
slug: redis slave linux
image:
categories:
    - Redis
tags: ["Redis"]
---

Linux上安装好redis，我的安装目录是在`/usr/local/src/redis/`下

这里我将新建3个redis服务，端口分别为：6380,6381,6382

其中6380设置为主服务器，其他2个设置为从服务器。

<!--more-->

## 复制redis.conf文件

进入redis安装目录下的bin目录：`cd /usr/local/src/redis/bin/`

复制配置文件：`cp redis.conf redis_6380.conf`， `cp redis.conf redis_6381.conf` `，cp redis.conf redis_6382.conf`

redis.conf 为默认配置文件，默认端口：`6379`。

## 修改配置文件

主要修改以下3个选项：

```php
pidfile /var/run/redis_6379.pid # redis启动后的进程ID保存文件
port 6379	# 端口
dbfilename dump.rdb	# 指定存储数据的文件名
```

这里以端口6380为例进行，其他的都是一样操作就行。

```php
pidfile /var/run/redis_6380.pid
port 6380
dbfilename dump_6380.rdb
```

## 启动服务

接下来启动redis_6380服务：`./redis-server redis_6380.conf`

然后使用命令`ps -aux | grep redis`查看

![image-20210713174555868](https://qnres.fahsa.cn/images/image-20210713174555868.png)

## 配置主从复制

配置主从复制有三种方案：

- 在从服务器的配置文件中添加配置项：`slaveof <masterip> <masterport>`，这种方式需要重启redis从服务器，如果已经启动先`kill`后重启。

- 从服务器客户端中执行命令：`slaveof (masterHost) (masterPort)`即可

- 在redis-server启动命令后加入`–slaveof (masterHost) (masterPort)`即可。

这里在redis_6381中测试第一种方式：

修改 redis_6381.conf 文件：`vim redis_6381.conf` ，添加如下配置项：

```php
slaveof 127.0.0.1 6380
```

此前，因为我启动了redis_6381，所以我先杀掉进程，操作如下：

![image-20210713175825706](https://qnres.fahsa.cn/images/image-20210713175825706.png)

然后重启：./redis-server redis_6381.conf

### 测试

master 主服务器中创建数据：`set name kim`

![image-20210713180214099](https://qnres.fahsa.cn/images/image-20210713180214099.png)

在从服务器中直接获取`name`：

登录6381：`./redis-cli -p 6381`

![image-20210713180335616](https://qnres.fahsa.cn/images/image-20210713180335616.png)

也可以使用 `info replication` 查看复制信息：role角色为slave

![image-20210713180744692](https://qnres.fahsa.cn/images/image-20210713180744692.png)

第二种方式也很简单直接使用命令即可：`slaveof 127.0.0.1 6380`

![image-20210713181054525](https://qnres.fahsa.cn/images/image-20210713181054525.png)

以上就是个人学习 Redis 主从复制的测试。