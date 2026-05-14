---
title: 关于MySQL错误1153 - Got a packet bigger than 'max_allowed_packet' bytes
date: 2021-09-28 15:10:00
description: 关于MySQL错误1153 - Got a packet bigger than 'max_allowed_packet' bytes
slug: mysql-error-code-1153
image:
categories:
    - MySQL
tags: ["MySQL"]

---

导入 MySQL 数据转储时遇到以下错误：

```mysql
Error Code: 1153 - Got a packet bigger than 'max_allowed_packet' bytes
```

意思是收到了一个大于 `max_allowed_packet`允许的字节包，显然是插入量比较大，导致的错误。

<!--more--> 

解决办法有：

1、在客户端上可以使用命令指定参数 `max_allowed_packet`的大小：

```mysql
mysql --max_allowed_packet=100M -u root -p database < dump.sql
```

2、更改 mysqld 配置下的 `my.cnf`或`my.ini`文件并设置：

```mysql
max_allowed_packet=100M
```

3、如果你是在本地连接到 MySQL 服务器，则可以在控制台中运行以下命令：

```mysql
set global net_buffer_length=1000000;
set global max_allowed_packet=1000000000;
# 数据包的大小，需要使用非常大的值
```

最后我使用第三种方法，解决问题并成功导入数据。

![image-20210928105649012](https://qnres.fahsa.cn/images/image-20210928105649012.png)

![image-20210928105753610](https://qnres.fahsa.cn/images/image-20210928105753610.png)

