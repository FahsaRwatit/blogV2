---
title: 一道MySQL查询题目
date: 2021-10-08 18:10:00
description: 关于MySQL的一道查询题目
slug: A MySQL query question
image:
categories:
    - MySQL
tags: ["MySQL"]

---

题目：有三张表：学生表 student，科目表：course，成绩表：grade。要求展示如下结果：

![image-20211008175900619](https://qnres.fahsa.cn/images/image-20211008175900619.png)

<!--more-->

### course

```sql
CREATE TABLE `course` (
  `course_id` INT(11) NOT NULL AUTO_INCREMENT,
  `c_name` VARCHAR(64) NOT NULL,
  PRIMARY KEY (`course_id`)
)
INSERT INTO `course` VALUES ('1', '语文');
INSERT INTO `course` VALUES ('2', '数学');
INSERT INTO `course` VALUES ('3', '外语');
```

### grade

```sql
CREATE TABLE `grade` (
  `grade_id` INT(11) NOT NULL AUTO_INCREMENT,
  `student_id` INT(11) NOT NULL,
  `course_id` INT(11) NOT NULL,
  `score` DECIMAL(5,2) NOT NULL,
  PRIMARY KEY (`grade_id`)
)

INSERT INTO `grade` VALUES ('1', '1', '1', '83.00');
INSERT INTO `grade` VALUES ('2', '1', '2', '75.00');
INSERT INTO `grade` VALUES ('3', '1', '3', '59.00');
INSERT INTO `grade` VALUES ('4', '2', '1', '76.00');
INSERT INTO `grade` VALUES ('5', '2', '2', '95.00');
INSERT INTO `grade` VALUES ('6', '2', '3', '87.00');
INSERT INTO `grade` VALUES ('7', '3', '1', '89.00');
INSERT INTO `grade` VALUES ('8', '3', '2', '74.00');
INSERT INTO `grade` VALUES ('9', '3', '3', '58.00');
INSERT INTO `grade` VALUES ('10', '4', '1', '95.00');
INSERT INTO `grade` VALUES ('11', '4', '2', '76.00');
INSERT INTO `grade` VALUES ('12', '4', '3', '87.00');
```

### student

```sql
CREATE TABLE `student` (
  `student_id` INT(11) NOT NULL AUTO_INCREMENT,
  `name` VARCHAR(16) NOT NULL,
  `age` INT(11) DEFAULT NULL,
  PRIMARY KEY (`student_id`)
)

INSERT INTO `student` VALUES ('1', '张三', '18');
INSERT INTO `student` VALUES ('2', '李四', '18');
INSERT INTO `student` VALUES ('3', '王五', '18');
INSERT INTO `student` VALUES ('4', '赵柳', '18');
```

首先使用以下语句查出：

```sql
SELECT student.student_id AS '编号',student.name AS '姓名',student.age AS '年龄',course.c_name,grade.score,
SUM(CASE course.c_name WHEN '语文' THEN grade.score ELSE 0 END) AS '语文',
SUM(CASE course.c_name WHEN '数学' THEN grade.score ELSE 0 END) AS '数学',
SUM(CASE course.c_name WHEN '外语' THEN grade.score ELSE 0 END) AS '外语',
CONVERT(SUM(grade.score)/3, DECIMAL(5,2)) AS '平均成绩',
CONVERT(SUM(grade.score),DECIMAL(5,2)) AS 'total'
FROM student,course,grade WHERE student.student_id = grade.student_id AND course.course_id = grade.course_id 
GROUP BY student.student_id
ORDER BY total DESC;
```

```sql
#CONVERT 对数据类型进行转化 convert(value,type)
# case 函数
-- 简单函数
-- CASE [col_name] WHEN [value1] THEN [result1]…ELSE [default] END

-- 搜索函数
-- CASE WHEN [expr] THEN [result1]…ELSE [default] END
```

对于 rows 和 rank 使用以下语句实现：

```sql
SELECT 
@ROWS:=@ROWS+1 AS ROWS,
IF(@gnum=total,@rownum:=@rownum,@rownum:=@rownum+1) AS rank,
@gnum:=total,
message.* FROM(
SELECT student.student_id AS '编号',student.name AS '姓名',student.age AS '年龄',course.c_name,grade.score,
SUM(CASE course.c_name WHEN '语文' THEN grade.score ELSE 0 END) AS '语文',
SUM(CASE course.c_name WHEN '数学' THEN grade.score ELSE 0 END) AS '数学',
SUM(CASE course.c_name WHEN '外语' THEN grade.score ELSE 0 END) AS '外语',
CONVERT(SUM(grade.score)/3, DECIMAL(5,2)) AS '平均成绩',
CONVERT(SUM(grade.score),DECIMAL(5,2)) AS 'total'
FROM student,course,grade WHERE student.student_id = grade.student_id AND course.course_id = grade.course_id 
GROUP BY student.student_id
ORDER BY total DESC
) message,
(SELECT @rownum:=0,@gnum:=0,@ROWS:=0) number;
```

```sql
sql语句中变量用@来表示，赋值用:=来实现

IF(expr1,expr2,expr3) 如果 expr1 是TRUE (expr1 <> 0 and expr1 <> NULL)，则 IF()的返回值为expr2; 否则返回值则为 expr3。IF() 的返回值为数字值或字符串值，具体情况视其所在语境而定。
```
