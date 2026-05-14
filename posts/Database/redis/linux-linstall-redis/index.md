---
title: 在Linux上安装Redis
description: 记录在Linux下安装Redis
date: 2019-03-12 15:10:00
slug: linux linstall redis
image:
categories:
    - Redis
tags: ["Redis"]
---

今天记录一下自己在Linux下是怎么安装 Redis 的，是一个基础的实操安装过程。

## 安装依赖文件

```linux
yum -y install make zlib zlib-devel gcc-c++ libtool openssl openssl-devel
```

<!--more-->

## 选择/usr/local/src作为安装目录，进入目录并下载redis安装包并解压

```linux
cd /usr/local/src
wget http://download.redis.io/releases/redis-4.0.8.tar.gz
tar -xvf redis-4.0.8.tar.gz
```

## 进入解压目录进行编译安装。

```linux
cd redis-4.0.8/
make # 编译
make install PREFIX=/usr/local/src/redis     #进行安装，prefix指定安装目录为/usr/local/src/redis
```

出现 INSTALL install 代表安装成功

![linux-install-img1](https://qnres.fahsa.cn/images/pic1/linux-install-img1.png)

![linux-install-img2](https://qnres.fahsa.cn/images/pic1/linux-install-img2.png)

## 启动

进入redis安装目录，cd /usr/local/src/redis

进入bin：cd bin 运行 ./redis-server 启动redis服务端如下图，代表启动成功

![linux-install-img3](https://qnres.fahsa.cn/images/pic1/linux-install-img3.png)

但是一般情况下都需要后台启动服务。
ctrl + c 退出服务。可以见：Redis is now ready to exit, bye bye.. （os:挺有趣的）

## 后台启动

进入安装包目录 cd /usr/local/src/redis-4.0.8/，复制redis.conf到安装目录bin下：cp redis.conf /usr/local/src/redis/bin/

进入目录查看是否复制成功，cd /usr/local/src/redis/bin/

![linux-install-img4](https://qnres.fahsa.cn/images/pic1/linux-install-img4.png)

编辑配置文件，vim redis.conf命令行下输入/daemonize 查找 daemonize no ，把no 改为yes，保存并退出

![linux-install-img5](https://qnres.fahsa.cn/images/pic1/linux-install-img5.png)

后台启动：./redis-server redis.conf
然后查看是否启动成功：ps -aux|grep redis如下代表启动成功

![linux-install-img6](https://qnres.fahsa.cn/images/pic1/linux-install-img6.png)

## 总结

好了，虽然过程比较基础，但实操下来相信下次也能记住就OK了。