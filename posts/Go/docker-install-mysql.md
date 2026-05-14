---
title: "docker安装mysql"
description:  "docker安装mysql"
date: 2022-06-08 18:10:00
slug: docker-install-mysql
image:
categories:
    - Go
tags: ["docker"]

---

### # 拉取mysql5.7版本镜像

```shell
# 拉取mysql5.7版本镜像
docker pull mysql:5.7
```

### 通过镜像启动

```shell
docker run -p 3306:3306 --name mymysql -v $PWD/conf:/etc/mysql/conf.d -v $PWD/logs:/logs -v $PWD/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7


Linux虚拟机端口3306->容器映射端口3306

-p 3306:33060 将容器的3306端口映射到主机的3306端口

-v $PWD/conf:/etc/mysql/conf.d 将主机当前目录下的conf/my.cnf 挂载到容器的/etc/mysql/my.cnf

-v $PWD/logs:/logs 将当前目录下的logs挂载到容器的/logs

-v $PWD/data:/var/lib/mysql 将当前目录下的data目录挂载到容器的/var/lib/mysql

-e MYSQL_ROOT_PASSWORD=123456 初始化用户的密码
```

```shell
# 查看容器ID和运行状态，up正在运行，如果为退出状态可以通过 docker logs 容器ID查看日志记录
docker ps -a

# 进入容器
docker exec -it ba6951fbd5fe /bin/bash

# 进入mysql
mysql -uroot -p123456
```

### 建立用户并授权

```shell
# 建立用户并授权
格式：grant 权限 on 数据库名.表名 to 用户@登录主机 identified by "用户密码"；
*.* 代表所有权；

@ 后面是访问MySQL的客户端IP地址（或是 主机名） % 代表任意的客户端，如果填写 localhost 为本地访问（那此用户就不能远程访问该mysql数据库了）。

# 执行下面四条命令进行授权
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;


GRANT ALL PRIVILEGES ON *.* TO 'root'@'127.0.0.1' IDENTIFIED BY '123456' WITH GRANT OPTION;


GRANT ALL PRIVILEGES ON *.* TO 'root'@'localhost' IDENTIFIED BY '123456' WITH GRANT OPTION;

FLUSH PRIVILEGES;
# 最后退出mysql
exit;
# 退出容器
exit
# 关闭Linux防火墙
systemctl stop firewalld
```

最后用`MySQL`客户端工具远程连接`MySQL`。