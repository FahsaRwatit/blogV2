---
title: Windows安装MongoDB无法启动服务解决方法
date: 2023-05-03 21:10:00
description: Windows安装MongoDB无法启动服务解决方法
slug: windows-start-mongodb
image:
categories:
    - MongoDB
tags: ["MongoDB"]

---







打开 任务管理器>服务 开启MongoDB服务，提示无法启动。

### 配置环境变量

#### 添加环境变量MONGO_HOME

```shell
名称：MONGO_HOME
值：D:\mysoft\mongodb6.0\server # MongoDB的安装目录
```

#### path添加

```shell
%MONGO_HOME%\bin
```

### 创建文件夹和文件

```shell
# data下创建文件夹db
D:\mysoft\mongodb6.0\server\data\db
# data下创建文件夹 logs
D:\mysoft\mongodb6.0\server\data\logs
# logs下创建文件 MongoDB.log
D:\mysoft\mongodb6.0\server\data\logs\MongoDB.log
```

### 执行命令

用管理员打开cmd

```shell
#删除原来的 MongoDB服务
sc delete MongoDB

# 安装新的MongoDB服务，填写自己的db文件夹和log文件路径
mongod --bind_ip 0.0.0.0 --logpath D:\mysoft\mongodb6.0\server\data\logs\MongoDB.log --logappend --dbpath D:\mysoft\mongodb6.0\server\data\db --port 27017 --serviceName "MongoDB" --serviceDisplayName "MongoDB" --install
```

### 重新打开服务

任务管理器>服务中重新开启服务。
