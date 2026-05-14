---
title: Windows安装RabbitMQ
description: Windows安装RabbitMQ
date: 2022-04-10 16:10:00
slug: centos8-rabbitmq
image:
categories:
    - Linux
tags: ["RabbitMQ"]

---

RabbitMQ教程：https://www.rabbitmq.com/getstarted.html

下载 Erlang：https://www.erlang.org/downloads

下载RabbitMQ：https://www.rabbitmq.com/install-windows.html

### 安装Erlang

按平时安装即可

```bash
# 添加配置系统环境变量
ERLANG_HOME
D:\mysoft\erl-24.3.4

# path中添加
%ERLANG_HOME%\bin

# 查看Erlang
erl
# 输出：Eshell V12.3.2  (abort with ^G)
#      1>
```

### RabbitMQ

按平时安装即可

```bash
# 进入 RabbitMQ  安装目录中的sbin目录，
例如 ：RabbitMQ Server\rabbitmq_server-3.10.1\sbin
#  运行 cmd 
# 开启管理平台插件
rabbitmq-plugins enable rabbitmq_management

# 打开sbin 目录 双击 rabbitmq-service.bat
# 等待执行完毕后 打开 http://localhost:15672/

```
