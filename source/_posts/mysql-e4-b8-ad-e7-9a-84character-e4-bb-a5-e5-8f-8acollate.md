---
title: mysql中的character以及collate
tags:
  - mysql
id: 566
categories:
  - 学习
date: 2014-11-20 21:12:20
---

mysql中的character指字符编码，collate用于排序，每种character对应多个collate，通过"show collation;"查看支持的character和collate<!--more-->
[toc]

```shell
mysql> show collation;
+--------------------------+----------+-----+---------+----------+---------+
| Collation | Charset | Id | Default | Compiled | Sortlen |
+--------------------------+----------+-----+---------+----------+---------+
......
| utf8_general_ci | utf8 | 33 | Yes | Yes | 1 |
| utf8_bin | utf8 | 83 | | Yes | 1 |
| utf8_unicode_ci | utf8 | 192 | | Yes | 8 |
| utf8_icelandic_ci | utf8 | 193 | | Yes | 8 |
......
+--------------------------+----------+-----+---------+----------+---------+
219 rows in set (0.00 sec)
```

character、collate有四个级别上，分别是全局、数据库、表、字段，后面的级别继承前面的设置
> *   [If both `CHARACTER SET _<code>X`_</code> and `COLLATE _<code>Y`_</code> are specified, character set _`X`_ and collation _`Y`_ are used.](http://dev.mysql.com/doc/refman/5.0/en/charset-database.html)
> *   [If `CHARACTER SET _<code>X`_</code> is specified without `COLLATE`, character set _`X`_ and its default collation are used. To see the default collation for each character set, use the `SHOW COLLATION` statement.](http://dev.mysql.com/doc/refman/5.0/en/charset-database.html)
> *   [If `COLLATE _<code>Y`_</code> is specified without `CHARACTER SET`, the character set associated with _`Y`_ and collation _`Y`_ are used.](http://dev.mysql.com/doc/refman/5.0/en/charset-database.html)
> *   [Otherwise, the server character set and server collation are used.](http://dev.mysql.com/doc/refman/5.0/en/charset-database.html)
下面分别展开这四个级别的查看和修改方法

### 全局

#### 查看方式

```shell
mysql> show variables like '%character%';
+--------------------------+----------------------------------------------+
| Variable_name            | Value                                        |
+--------------------------+----------------------------------------------+
| character_set_client     | utf8                                         |
| character_set_connection | utf8                                         |
| character_set_database   | latin1                                       |
| character_set_filesystem | binary                                       |
| character_set_results    | utf8                                         |
| character_set_server     | latin1                                       |
| character_set_system     | utf8                                         |
| character_sets_dir       | /alidata/server/mysql-5.6.15/share/charsets/ |
+--------------------------+----------------------------------------------+
8 rows in set (0.00 sec)

mysql> show variables like "%collat%";
+----------------------+-----------------+
| Variable_name        | Value           |
+----------------------+-----------------+
| collation_connection | utf8_general_ci |
| collation_database   | utf8_general_ci |
| collation_server     | utf8_general_ci |
+----------------------+-----------------+
3 rows in set (0.00 sec)
```

#### 修改方式

# 修改my.cnf

```shell
# set default charset and collation
# http://dev.mysql.com/doc/refman/5.6/en/charset-applications.html
character-set-server=utf8
collation-server=utf8_general_ci

mysql> show variables like '%character%';
+--------------------------+----------------------------------------------+
| Variable_name            | Value                                        |
+--------------------------+----------------------------------------------+
| character_set_client     | utf8                                         |
| character_set_connection | utf8                                         |
| character_set_database   | utf8                                         |
| character_set_filesystem | binary                                       |
| character_set_results    | utf8                                         |
| character_set_server     | utf8                                         |
| character_set_system     | utf8                                         |
| character_sets_dir       | /alidata/server/mysql-5.6.15/share/charsets/ |
+--------------------------+----------------------------------------------+
8 rows in set (0.01 sec)
```

### 数据库

#### 查看方式

```shell
mysql> select * from information_schema.schemata;
+--------------+--------------------+----------------------------+------------------------+----------+
| CATALOG_NAME | SCHEMA_NAME        | DEFAULT_CHARACTER_SET_NAME | DEFAULT_COLLATION_NAME | SQL_PATH |
+--------------+--------------------+----------------------------+------------------------+----------+
| def          | information_schema | utf8                       | utf8_general_ci        | NULL     |
| def          | dedecms            | latin1                     | latin1_swedish_ci      | NULL     |
| def          | mysql              | utf8                       | utf8_general_ci        | NULL     |
| def          | performance_schema | utf8                       | utf8_general_ci        | NULL     |
| def          | phpwind            | gbk                        | gbk_chinese_ci         | NULL     |
| def          | quickstart         | latin1                     | latin1_swedish_ci      | NULL     |
| def          | simpleweb          | latin1                     | latin1_swedish_ci      | NULL     |
| def          | test               | latin1                     | latin1_swedish_ci      | NULL     |
| def          | wordpress          | latin1                     | latin1_swedish_ci      | NULL     |
+--------------+--------------------+----------------------------+------------------------+----------+
9 rows in set (0.09 sec)
```

#### 修改方式

```shell
mysql> alter database simpleweb default character set utf8 default collate utf8_general_ci;
Query OK, 1 row affected (0.00 sec)

mysql> create database my_tmp default character set utf8 default collate utf8_general_ci;
Query OK, 1 row affected (0.01 sec)

mysql> select * from information_schema.schemata;
+--------------+--------------------+----------------------------+------------------------+----------+
| CATALOG_NAME | SCHEMA_NAME        | DEFAULT_CHARACTER_SET_NAME | DEFAULT_COLLATION_NAME | SQL_PATH |
+--------------+--------------------+----------------------------+------------------------+----------+
| def          | information_schema | utf8                       | utf8_general_ci        | NULL     |
| def          | dedecms            | latin1                     | latin1_swedish_ci      | NULL     |
| def          | my_tmp             | utf8                       | utf8_general_ci        | NULL     |
| def          | mysql              | utf8                       | utf8_general_ci        | NULL     |
| def          | performance_schema | utf8                       | utf8_general_ci        | NULL     |
| def          | phpwind            | gbk                        | gbk_chinese_ci         | NULL     |
| def          | quickstart         | latin1                     | latin1_swedish_ci      | NULL     |
| def          | simpleweb          | utf8                       | utf8_general_ci        | NULL     |
| def          | test               | latin1                     | latin1_swedish_ci      | NULL     |
| def          | wordpress          | latin1                     | latin1_swedish_ci      | NULL     |
+--------------+--------------------+----------------------------+------------------------+----------+
10 rows in set (0.00 sec)
```

### 表

#### 查看方式

```shell
mysql> select ccsa.character_set_name, ccsa.collation_name from information_schema.`tables` t,
    ->        information_schema.`collation_character_set_applicability` ccsa
    -> where ccsa.collation_name = t.table_collation
    ->   and t.table_schema = "simpleweb"
    ->   and t.table_name = "users";
+--------------------+-------------------+
| character_set_name | collation_name    |
+--------------------+-------------------+
| latin1             | latin1_swedish_ci |
+--------------------+-------------------+
1 row in set (0.00 sec)
```

#### 修改方式

```shell
mysql> alter table users default character set utf8 default collate utf8_general_ci;
Query OK, 0 rows affected (0.03 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> select ccsa.character_set_name, ccsa.collation_name from information_schema.`tables` t,  information_schema.`collation_character_set_applicability` ccsa where ccsa.collation_name = t.table_collation   and t.table_schema = "simpleweb"   and t.table_name = "users";
+--------------------+-----------------+
| character_set_name | collation_name  |
+--------------------+-----------------+
| utf8               | utf8_general_ci |
+--------------------+-----------------+
1 row in set (0.00 sec)
```

### 列

#### 查看方式

```shell
mysql> select column_name, character_set_name, collation_name from information_schema.`columns` c
    -> where table_schema = "simpleweb"
    ->   and table_name = "users";
+-------------+--------------------+-------------------+
| column_name | character_set_name | collation_name    |
+-------------+--------------------+-------------------+
| id          | NULL               | NULL              |
| username    | latin1             | latin1_swedish_ci |
| age         | NULL               | NULL              |
| salary      | NULL               | NULL              |
+-------------+--------------------+-------------------+
4 rows in set (0.00 sec)
```

#### 修改方式

```shell
mysql>  alter table users modify username varchar(20) character set utf8 collate utf8_general_ci;
Query OK, 3 rows affected (0.24 sec)
Records: 3  Duplicates: 0  Warnings: 0

mysql> select column_name, character_set_name, collation_name from information_schema.`columns` c where table_schema = "simpleweb"   and table_name = "users";
+-------------+--------------------+-----------------+
| column_name | character_set_name | collation_name  |
+-------------+--------------------+-----------------+
| id          | NULL               | NULL            |
| username    | utf8               | utf8_general_ci |
| age         | NULL               | NULL            |
| salary      | NULL               | NULL            |
+-------------+--------------------+-----------------+
4 rows in set (0.00 sec)
```

### 参考资料

1\. [how-do-i-see-what-character-set-a-database-table-column-is-in-mysql](http://stackoverflow.com/questions/1049728/how-do-i-see-what-character-set-a-database-table-column-is-in-mysql)
2\. [charset-server](http://dev.mysql.com/doc/refman/5.7/en/charset-server.html)
3\. [charset-database](http://dev.mysql.com/doc/refman/5.7/en/charset-database.html)
4\. [charset-table](http://dev.mysql.com/doc/refman/5.7/en/charset-table.html)
5\. [charset-column](http://dev.mysql.com/doc/refman/5.7/en/charset-column.html)
6. [alter-database](http://dev.mysql.com/doc/refman/5.7/en/alter-database.html)
6\. [alter-table](http://dev.mysql.com/doc/refman/5.7/en/alter-table.html)
7. [show-variables](http://dev.mysql.com/doc/refman/5.7/en/show-variables.html)