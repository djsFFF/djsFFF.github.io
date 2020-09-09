---
title: MySQL学习1：SQL语句
date: 2020-08-14 20:15:00
tags: 
	- MySQL
	- 数据库
categories:
	- 数据库
typora-root-url: ..
---

`{}`表示必选项，`[]`表示可选项

<!--more-->

# 创建数据库

`CREATE {DATABASE | SCHEMA} [IF NOT EXISTS] db_name [DEFAULT] CHARACTER SET [=] charset_name`

# 数据类型

整型：TINYINT、SMALLINT、MEDIUMINT、INT、BIGINT

浮点型：FLOAT、DOUBLE

日期时间型：YEAR、TIME、DATE、DATETIME、TIMESTAMP

字符型：CHAR、VARCHAR、TINYTEXT、TEXT、MEDIUMTEXT、LONGTEXT、ENUM('value1','value2',...)、SET('value1','value2',...)

# 数据表

## 创建数据表

`CREATE TABLE [IF NOT EXIST] table_name (column_name data_type,...);`

- NULL，字段允许为空；NOT NULL，字段禁止为空。

- AUTO_INCREMENT：自动编号，必须与主键组合使用，默认情况下起始值为1，每次增量为1。

- PRIMARY：主键，每张数据表只能存在一个主键字段，自动为NOT NULL。

- UNIQUE KEY：唯一约束，每张数据表可以存在多个唯一约束字段，可以为NULL。

- DEFAULT：默认值，插入记录时，如果没有明确赋值，则自动赋予默认值。

- FOREIGN KEY：外键，子表（具有FOREIGN KEY的表）和父表必须使用InnoDB存储引擎；外键列和参照列必须具有相似的数据类型，必须创建索引。

  `FOREIGN KEY (pid) REFERENCES tb_name (id) ON DELETE CASCADE`

  外键约束的参照操作：

  - CASCADE：从父表删除或更新行时自动删除或更新子表对应的行。
  - SET NULL：从父表删除或更新行时设置子表外键列为NULL，必须保证子表外键列没有指定NOT NULL。
  - RESTRICT：拒绝对父表的删除或更新操作。
  - NO ACTION：在MySQL中与RESTRICT相同。

### CREATE ... SELECT ...

`CREATE tb_name (col_name,...) SELECT ...;`

将查询的数据写入到新建的数据表中。

## 修改数据表

添加单列：`ALTER TABLE tb_name ADD (column_name data_type,...);`

删除列：`ALTER TABLE tb_name DROP column_name;`

添加外键：`ALTER TABLE tb_name ADD FOREIGN KEY (pid) REFERENCES tb2_name (id);`

删除主键、添加/删除默认约束、添加/删除唯一约束、删除外键约束等

## 操作数据表中的记录

### INSERT

`INSERT tb_name(col_name,...) VALUES(value1,...);`可以用NULL或DEFAULT占用递增字段。

`INSERT tb_name SET col_name1=value1,...;`，这种方法可以使用子查询。

`INSERT tb_name(col_name,...) SELECT ...`，将查询结果插入表中。

### UPDATE

`UPDATE tb_name SET col_name=value,... [WHERE ...];`

使用**连接**进行多表更新：

`UPDATE tb_name INNER JOIN tb_name2 ON 连接条件 SET col_name=value,...`

### DELETE

`DELETE FROM tb_name [WHERE ...];`

### SELECT

`SELECT expr,... [FROM tb_name [WHERE ...] [GROUP BY ... [ASC|DESC]] [HAVING ...] [ORDER BY ... [ASC|DESC]] [LIMIT row_count OFFSET row_count]]`

字段表达式的顺序会影响结果的顺序

可以在字段前加上表名避免多表连接查询时字段名冲突

可以使用AS赋予别名，会影响结果的字段名，可用于GROUP BY，ORDER BY，HAVING子句。

#### WHERE

WHERE

#### GROUP BY

查询结果分组，查询某一个字段的所有取值。

#### HAVING

分组条件

#### ORDER BY

按一个或多个字段对查询结果进行排序

#### LIMIT

限制查询结果返回的数量

### 子查询

嵌套在查询内部的SELECT子句，必须出现在圆括号内。

子查询的返回值可以是标量、行、列、子查询。

#### 比较子查询

`SELECT * FROM tb_name WHERE col_name > ANY (子查询);`

![image-20200826112320549](/images/MySQL1/image-20200826112320549.png)

#### IN、NOT IN子查询

`IN (子查询)`与`=ANY (子查询)`等效

`NOT IN (子查询)`与`!=ALL (子查询)`、`<>ALL (子查询)`等效

#### EXISTS、NOT EXISTS子查询

子查询语句查询到结果，EXISTS返回TRUE、否则返回FALSE

### 连接

MySQL在SELECT语句、多表更新、多表删除中支持JOIN操作。

使用ON设定连接条件，也可以使用WHERE代替。

#### 内连接INNER JOIN等价于JOIN、CROSS JOIN

`SELECT col_name FROM tb_name INNER JOIN tb_name2 ON 连接条件;`

显示左表（当前查询表）和右表（连接表）符合连接条件的记录（交集）。

![image-20200826224618172](/images/MySQL1/image-20200826224618172.png)

#### 左外连接LEFT JOIN

显示左表的全部记录及右表符合连接条件的记录。

![image-20200826224854158](/images/MySQL1/image-20200826224854158.png)

#### 右外连接RIGHT JOIN

## 函数

### 字符函数

![image-20200826232137034](/images/MySQL1/image-20200826232137034.png)

![image-20200826232205853](/images/MySQL1/image-20200826232205853.png)

SUBSTRING()从1开始计数，可以用负值从末尾开始数。

### 数值运算符与函数

![image-20200826232654739](/images/MySQL1/image-20200826232654739.png)

### 比较运算符与函数

![image-20200826233006754](/images/MySQL1/image-20200826233006754.png)

### 日期时间函数

![image-20200826233244090](/images/MySQL1/image-20200826233244090.png)

### 信息函数

![image-20200826233613821](/images/MySQL1/image-20200826233613821.png)

### 聚合函数

![image-20200826233855766](/images/MySQL1/image-20200826233855766.png)

### 加密函数

![image-20200826234102940](/images/MySQL1/image-20200826234102940.png)

### 自定义函数

`CREATE FUNCTION function_name(参数) RETURNS {STRING|INTEGER|REAL|DECIMAL} 函数体;`

函数体由合法的SQL语句构成，可以使用声明、循环等流程控制。

只能有一个返回值。

## 存储过程

SQL语句和控制语句的预编译集合，可以增强SQL语句的功能和灵活性，实现较快的执行速度，存储过程名字代替SQL语句减少网络流量。

可以返回多个值。

`CREATE PROCEDURE sp_name([IN|OUT|INOUT]参数) 过程体;`

- IN输入类型参数，表示该参数的值必须在调用存储过程时指定。

- OUT输出类型参数，表示该参数的值可以被存储过程改变，并且可以返回。

- INOUT输入&输出类型参数，表示改参数在调用时指定，并且可以被改变和返回。

过程体由合法的SQL语句构成，可以使用声明、循环等流程控制。

调用存储过程：`CALL sp_name();`