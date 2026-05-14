---
title: MySQL相关
date: 2021-06-28 21:10:00
description: MySQL相关
slug:
image:
categories:
    - MySQL
tags: ["MySQL"]

---

思维导图：https://github.com/cystanford/SQL-XMind

-- 统计7 天内的新增用户数有多少，获取当前时间使用NOW()函数

```sql
select * from qy_members where to_days(now()) - to_days(created) <= 7
```

关于 SQL 大小写的问题，我总结了下面两点：

表名、表别名、字段名、字段别名等都小写；
SQL 保留字、函数名、绑定变量等都大写。
数据表的字段名推荐采用下划线命名，比如 role_main 


SQL 语句在 MySQL 中的流程是：SQL 语句→缓存查询→解析器→优化器→执行器。

MySQL 中，8.0 以后的版本不再支持查询缓存

```sql
-- 代表关闭 1 开启
SELECT @@profiling;

-- 查看当前会话所产生的所有 profiles
SHOW profiles;

-- 获取上一次查询的执行时间
-- checking_permissions(权限检查), Opening tables(打开表), init(初始化), 
-- System lock(锁系统), optimizing(优化查询),  preparing(准备), executing(执行)
SHOW profile;
-- 查询指定的query id
SHOW profile FOR QUERY 13
```

DDL数据定义语言（Data Definition Language），它定义了数据库的结构和数据表的结构。

在 DDL 中，常用的功能是增删改，分别对应的命令是 CREATE、DROP 和 ALTER。在执行 DDL 的时候，不需要 COMMIT，就可以完成执行任务。

```sql
-- 创建一个名为testdb的数据库
CREATE DATABASE testdb;
-- 删除一个名为testdb的数据库
DROP DATABASE testdb;

-- 创建数据表
CREATE TABLE table_name

CREATE TABLE table_name {
	id INT(11) NOT NULL AUTO_INCREMENT,
	username VARCHAR(255) NOT NULL
}
```

 DDL 语句，可以使用一些可视化工具来创建和操作数据库和数据表。例如：Navicat。

### 设计表原则

- 1.**数据表的个数越少越好**
- 2.**数据表中的字段个数越少越好**
- 3.**数据表中联合主键的字段个数越少越好**
- 4.**使用主键和外键越多越好**



```sql
-- select后使用单引号说明是个常数，否则为字段名
SELECT '测试' AS test, username  FROM test

-- 去除重复行
SELECT DISTINCT username FROM test
-- DISTINCT 需要放到所有列名的前面
-- DISTINCT 其实是对后面所有列名的组合进行去重
SELECT DISTINCT username,age FROM test

-- 排序
SELECT * FROM test ORDER BY username

-- 约束数量
SELECT * FROM test LIMIT 5
-- 分页查询
SELECT * FROM test LIMIT 2,5
```

SELECT的2个顺序：

```sql
-- 关键字顺序
SELECT ... FROM ... WHERE ... GROUP BY ... HAVING ... ORDER BY ...

-- SELECT 语句的执行顺序
FROM > WHERE > GROUP BY > HAVING > SELECT 的字段 > DISTINCT > ORDER BY > LIMIT

```

### SQL的内置函数

- 算术函数
- 字符串函数
- 日期函数
- 转换函数

#### 算术函数

| 函数名  | 定义                                                         |
| ------- | ------------------------------------------------------------ |
| ABS()   | 取绝对值                                                     |
| MOD()   | 取余                                                         |
| ROUND() | 四舍五入为指定的小数位数，需要2个参数，分别为字段名和小数位数 |

#### 字符串函数

| 函数名 | 定义 |
| ------ | ---- |
|        |      |

























































