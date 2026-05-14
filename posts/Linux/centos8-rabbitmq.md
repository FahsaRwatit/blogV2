---
title: CentOS8安装RabbitMQ
description: CentOS8安装RabbitMQ
date: 2022-04-10 16:10:00
slug: centos8-rabbitmq
image:
categories:
    - Linux
tags: ["Linux","RabbitMQ"]

---

## yum 安装问题

CentOS8 使用 `yum`安装 软件出现错误：

```sh
错误:为仓库 'appstream' 下载元数据失败 : Cannot prepare internal mirrorlist:
```

原因：

> CentOS Linux 8在2022年12月31日来到生命周期终点（End of Life，EoL）。即CentOS Linux 8操作系统版本结束了生命周期（EOL），Linux社区已不再维护该操作系统版本。所以原来的CentOS Linux 8的yum源也都失效了！最终导致此问题的产生。

**问题解决方法：**更换CentOS Linux 8的yum源

### 1、切换到源目录，备份原来的源

```sh
# 进入源目
cd /etc/yum.repos.d

# 创建文件夹back存放源文件
mkdir back

# 将文件移入back
mv * back

```

### 2、下载新的源文件，并建立起新元数据缓存

```sh
# 下载阿里云源数据文件
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-vault-8.5.2111.repo

# 建立新元数据缓存
yum makecache
```

## 安装RabbitMQ

### 环境准备

- [使用PackageCloud Yum Repository安装](https://www.rabbitmq.com/install-rpm.html#package-cloud)

```sh
# 依次执行
curl -s https://packagecloud.io/install/repositories/rabbitmq/rabbitmq-server/script.rpm.sh | sudo bash

curl -s https://packagecloud.io/install/repositories/rabbitmq/erlang/script.rpm.sh | sudo bash
```

### 安装Erlang和RabbitMQ

```sh
# 安装Erlang
yum install erlang -y
# 安装RabbitMQ
yum install rabbitmq-server -y

# 查看erlang
rpm -qa | grep erlang
#结果：erlang-24.3.4-1.el8.x86_64

# 查看rabbitmq-server
rpm -qa | grep rabbitmq-server
#结果：rabbitmq-server-3.10.1-1.el8.noarch

# 开启RabbitMQ管理平台的插件管理
rabbitmq-plugins enable rabbitmq_management

# systemctl enable rabbitmq-server
# 开机启动
systemctl enable rabbitmq-server.service

# 启动rabbitmq-server
systemctl start rabbitmq-server
# 查看状态
systemctl status rabbitmq-server

# 列出 rabbitmq 用户列表
rabbitmqctl list_users

```

### 配置

```sh
# 添加用户并设置密码
rabbitmqctl add_user admin 123456
# 授权用户权限
rabbitmqctl set_permissions -p "/" admin ".*" ".*" ".*"
# 结果：Setting permissions for user "admin" in vhost "/" ...

# 修改用户角色
set_user_tags admin administrator
# 结果: Setting tags for user "admin" to [administrator] ...

```

### 访问

使用本地电脑访问:

```
http://域名:15672 或 http://服务器公网 IP:15672
```

### 常用命令

```sh
service rabbitmq-server start


[rabbitmq-server]
sudo systemctl start rabbitmq-server
sudo systemctl stop rabbitmq-server
sudo systemctl restart rabbitmq-server
sudo systemctl status rabbitmq-server

```
