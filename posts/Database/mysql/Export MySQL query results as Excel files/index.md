---
title: 将MySQL查询结果导出为excel文件
date: 2020-12-10 18:10:00
description: 将数据库的查询结果导出为Excel
slug: Export MySQL query results as Excel files
image:
categories:
    - MySQL
tags: ["MySQL"]

---

## MySQL查询结果导出Excel的三种方法

参考：https://www.cnblogs.com/qiaoyihang/p/6398673.html

我使用的是第二种，所以 MySQL 语句为：

```mysql
echo "SELECT user_name,mobile,user_title FROM xxx.a_user GROUP BY mobile;" | mysql -u root -p > /data/mysql_data/xxx.xls
```

<!--more--> 

## 数据去重

需求是需要将数据库中的数据去重后导出为`Excel`文件。

首先想到的是用`DISTINCT`进行去重，如下：

```mysql
SELECT DISTINCT mobile FROM a_user;
```

以上可以实现去重，但当需要查询多个字段时，虽然也是根据多个字段进行去重，但是却无法达到所需要的结果。

所以我使用`group by`进行实现：

```mysql
SELECT user_name,mobile,user_title FROM a_user GROUP BY mobile;
```

## MySQL 5.7版本以上 group by 问题

使用以上查询语句报错：

```mysql
Expression #1 of SELECT list is not in GROUP BY clause and contains nonaggregated column 'xx.xx.xx' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by
```

原因是 MySQL 5.7以后默认开启了 `only_full_group_by` SQL模式，所以我们需要将 `only_full_group_by`取消掉就可以了。

解决办法：

1、查询 MySQL 相关的 mode

```mysql
SELECT @@global.sql_mode;
```

显示如下：

```mysql
+-------------------------------------------------------------------------------------------------------------------------------------------+
| @@global.sql_mode                                                                                                                         |
+-------------------------------------------------------------------------------------------------------------------------------------------+
| ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION |
+-------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

2、重新设置模式的值

```mysql
set @@global.sql_mode=`STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE, 
ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION`;
或
SET GLOBAL sql_mode=(SELECT REPLACE(@@sql_mode,'ONLY_FULL_GROUP_BY',''));
```

3、重启 MySQL

```shell
service mysql restart
或
service mysqld restart
```

4、重新连接，查看修改是否成功

```mysql
SELECT @@global.sql_mode;
```

此时发现已经去掉了`ONLY_FULL_GROUP_BY`

然后执行语句：

```mysql
echo "SELECT user_name,mobile,user_title FROM xxx.a_user GROUP BY mobile;" | mysql -u root -p > /data/mysql_data/xxx.xls
```

成功导出 Excel 文件。

## 乱码问题

这里引用原文的解决办法：

将文件下载到本地，打开如果中文乱码，因为office默认的是gb2312编码，服务器端生成的很有可能是utf-8编码，这个时候你有两种选择:
1、在服务器端使用iconv来进行编码转换

```mysql
# type1.xls为输出文件 type.xls为源文件
iconv -futf8 -tgb2312 -otype1.xls type.xls
```

如果转换顺利，那么从server上下载下来就可以使用了。
2、转换如果不顺利，则会提示:

```
iconv: illegal input ``sequence` `at` `position 1841
```

类似错误，如下解决：
   先把type.xls下载下来，这个时候文件是utf-8编码的，用excel打开，乱码。把type.xls以文本方式打开，然后另存为，在编码选择ANSI编码保存。

## 最后

虽然涉及点较为简单，但操作过程中会遇到各种各样问题，这时记住不心急，沉下心来，慢就是快。
