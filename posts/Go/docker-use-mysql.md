---
title: "Docker中Data Volume练习MySQL"
description: "Docker中Data Volume练习MySQL"
date: 2022-06-10 18:10:00
slug: docker-use-mysql
image:
categories:
    - Go
tags: ["Go", "Docker"]
---



### 创建镜像

```shell
# 查看已经存在的镜像
docker image ls
# 拉取MySQL版本为5.7
docker pull mysql:5.7
# 查看已经存在的镜像
docker image ls
# 显示如下
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
mysql        5.7       efa50097efbd   34 hours ago   462MB
```

### 创建容器

` MYSQL_ROOT_PASSWORD=123456`设置数据库密码为`123456`

`-v mysql-data` 中的`-v`指定volume中的名称为mysql-data

```shell
# 创建容器
docker container run --name some-mysql -e MYSQL_ROOT_PASSWORD=123456 -d -v mysql-data:/var/lib/mysql mysql:5.7
# 查看volume文件
docker volume ls
# 查看mysql-data信息
docker volume inspect mysql-data


[root@iZwz9dqgikj7gb7539qprfZ my-docker]# docker container run --name some-mysql -e MYSQL_ROOT_PASSWORD=123456 -d -v mysql-data:/var/lib/mysql mysql:5.7
828e0fb073febc3498d3a91ff4f98bd8e17f5327440317df9cccaa8eb91c9483
[root@iZwz9dqgikj7gb7539qprfZ my-docker]# docker volume ls
DRIVER    VOLUME NAME
local     mysql-data
[root@iZwz9dqgikj7gb7539qprfZ my-docker]# docker volume inspect mysql-data
[
    {
        "CreatedAt": "2022-06-29T18:46:24+08:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/mysql-data/_data",
        "Name": "mysql-data",
        "Options": null,
        "Scope": "local"
    }
]

```

### 操作Docker中的数据库 

```sh
# 在终端操作mysql, 828为容器id随机前几位数
docker container exec -it 828 sh
# 连接mysql
mysql -u root -p
# 查看数据库
show databases;
# 创建数据库
create database demo1;

# 查看存放数据的目录中，是否存在刚刚创建的数据库
ls /var/lib/docker/volumes/mysql-data/_data
```

显示结果如下：

```sh
[root@iZwz9dqgikj7gb7539qprfZ my-docker]# docker container exec -it 828 sh
# mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.38 MySQL Community Server (GPL)

Copyright (c) 2000, 2022, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)

mysql> create database demo1;
Query OK, 1 row affected (0.00 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| demo1              |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.01 sec)

mysql> exit
Bye
# exit
[root@iZwz9dqgikj7gb7539qprfZ my-docker]# docker volume inspect mysql-data 
[
    {
        "CreatedAt": "2022-06-29T18:48:33+08:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/mysql-data/_data",
        "Name": "mysql-data",
        "Options": null,
        "Scope": "local"
    }
]
[root@iZwz9dqgikj7gb7539qprfZ my-docker]# ls /var/lib/docker/volumes/mysql-data/_data
auto.cnf    ca.pem           client-key.pem  ib_buffer_pool  ib_logfile0  ibtmp1  performance_schema  public_key.pem   server-key.pem
ca-key.pem  client-cert.pem  demo1           ibdata1         ib_logfile1  mysql   private_key.pem     server-cert.pem  sys
```

