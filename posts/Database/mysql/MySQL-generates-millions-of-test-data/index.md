---
title: MySQL生成百万条测试数据
description: MySQL生成百万条测试数据
date: 2021-09-12 15:10:00
slug: Export MySQL query results as Excel files
image:
categories:
    - MySQL
tags: ["MySQL"]

---

使用 MySQL 时经常会碰到大数据量的一些操作，但有些时候数据库中并没有足够的数据进行测试，所以需要在测试库中自行生成数据。

整体思路：

1、使用 MySQL 自定义函数`rand_string`，生成长度为n的随机字符串。

2、为了使数据插入快些，创建2张表，一张临时内存表，一张普通表。

3、创建存储过程，让操作简单些。

4、调用存储过程。

5、查询内存表中生成的记录

6、将数据从临时内存表插入普通表中

7、查询普通表生成的记录

<!--more-->

## MySQL 自定义函数

### MySQL中创建自定义函数的语法

```mysql
CREATE FUNCTION func_name(param_list) RETURNS TYPE
BEGIN
      -- Todo:function body
END 
```

拓展：

```mysql
# 调用函数
SELECT func_name(param_list);
# 查看函数
SHOW FUNCTION STATUS; 
# 查看函数创建脚本
SHOW CREATE FUNCTION func_name;
# 删除函数
DROP FUNCTION IF EXISTS func_name;
```

### `rand_string`：生成长度为n的随机字符串

```mysql
-- 创建一个可生成长度为n的随机字符串函数
# 如果存在函数 rand_string 则删除
DROP FUNCTION IF EXISTS rand_string;
# 声明结束符为 $
DELIMITER $
SET NAMES utf8 $
CREATE FUNCTION rand_string (n INT) RETURNS VARCHAR(255) CHARSET 'utf8'
BEGIN
     DECLARE char_set VARCHAR(100) DEFAULT 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789';
     DECLARE return_str VARCHAR(255) DEFAULT '';
     DECLARE i INT DEFAULT 0;
     WHILE i < n DO
	SET return_str = CONCAT(return_str,SUBSTRING(char_str, FLOOR(1+RAND()*62 ), 1));
	SET i = i + 1;
     END WHILE;
     RETURN return_str;
END $

-- 改回默认的 MySQL delimiter：';'
DELIMITER ;
```

## 创建临时内存表和普通表

### 临时内存表

```mysql
CREATE TABLE `add_test_str_memory` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `str` varchar(50) NOT NULL DEFAULT '',
  `num` int(10) NOT NULL,
  `create_time` datetime NOT NULL DEFAULT '0000-00-00 00:00:00',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1000005 DEFAULT CHARSET=utf8mb4
```

### 普通表

```mysql
CREATE TABLE `add_test_str` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `str` varchar(50) NOT NULL DEFAULT '',
  `num` int(10) NOT NULL DEFAULT '0',
  `create_time` decimal(10,0) NOT NULL DEFAULT '0',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4

```

## 创建&调用存储过程

![image-20210929193124124](https://qnres.fahsa.cn/images/image-20210929193124124.png)

## 将数据从临时内存表插入普通表中

```mysql
INSERT INTO add_test_str SELECT * FROM add_test_str_memory;
```



## 查询内存表中生成的记录

```mysql
SELECT COUNT(*) FROM add_test_str_memory;

```

## 查询普通表生成的记录

```mysql
SELECT COUNT(*) FROM add_test_str;
```

## 最后

### 关于存储过程和函数的区别

 1、存储过程的关键字为`procedure`，返回值可以有多个，调用时用`call`，一般用于执行比较复杂的过程体、更新、创建等语句。

函数的关键字为`function`，返回值必须有一个，调用时使用`select`，一般用于查询单个值返回。

| 行为     | 存储过程          | 函数       |
| -------- | ----------------- | ---------- |
| 返回值   | 可以有0个或者多个 | 必须有一个 |
| 关键字   | procedure         | function   |
| 调用方式 | call              | select     |

