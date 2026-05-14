---
title: MySQL对一张百万级别的表做查询优化
date: 2020-03-30 15:10:00
description: MySQL对一张百万级别的表做查询优化
slug: MySQL-performs-query-optimization-on-a-million-level-table
image:
categories:
    - MySQL
tags: ["MySQL"]

---

一般对于大数据量的表，都使用分页查询

```sql
SELECT COUNT(*) FROM add_test_str_memory; -- 1000004
```

```sql
SELECT * FROM add_test_str_memory ORDER BY id LIMIT 0,10;
SELECT * FROM add_test_str_memory ORDER BY id LIMIT 1000,10;
SELECT * FROM add_test_str_memory ORDER BY id LIMIT 10000,10;
SELECT * FROM add_test_str_memory ORDER BY id LIMIT 100000,10;
SELECT * FROM add_test_str_memory ORDER BY id LIMIT 1000000,10;
```

随着页数增多，查询速度越慢。

<!--more-->

###改进方案一

```sql
-- 改进方案1
SELECT * FROM add_test_str_memory WHERE id > (SELECT id FROM add_test_str_memory ORDER BY id DESC LIMIT 100000,1) ORDER BY id DESC LIMIT 0,10;
```



### 改进方案二

```sql
-- 改进方案2,不适合带有条件的、id不连续的查询
SELECT * FROM add_test_str_memory WHERE id BETWEEN 100000 AND 100010 ORDER BY id DESC;
```



### 改进方案三

当使用方案一和二时，效果都不明显，则可以使用这种方案。

思路：建立一张索引表，将要查询的几个字段分割出去，写入数据时将2张表同步，查询时则可以使用索引表查询

```sql
SELECT str,num FROM add_test_str_memory WHERE id > (SELECT id FROM add_test_str_memory2 ORDER BY id DESC LIMIT 100000,1) ORDER BY id DESC LIMIT 0,10;
```

除了以上以外，还需要了解一些MySQL性能优化的点：

1、不适用函数或者表达式作为查询条件

2、善于使用 `explain`

3、避免 `select *`

4、正确使用索引

5、不要ORDER BY RAND()

效率很低的一种随机查询。

6、使用 NOT NULL

7、只要一行数据时使用`limit 1`

8、使用 ENUM 而不是 VARCHAR

ENUM 类型是非常快和紧凑的。在实际上，其保存的是 TINYINT，但其外表上显示为字符串。
