---
title: MySQL中的interactive_timeout和wait_timeout
tags:
  - interactive_timeout
  - mysql
  - mysql_set
  - wait_timeout
id: 584
categories:
  - 学习
date: 2014-11-30 11:15:09
---

在MySQL中执行数据库操作主要有两种方式，interactive（交互式）和non-interactive（非交互式），当然执行命令会有超时时间（可能由于网络原因，数据没传递过来造成客户端超时；或由于服务端处理此操作非常耗时超过了服务端设置的超时时间造成的服务端超时），但是这里说的超时时间是指数据库连接在指定的时间内都没有被使用，过后使用时的超时。

<!--more-->

分为交互式超时——interactive_timeout，即在交互式命令行界面执行sql时数据库连接长时间不用导致超时；非交互式超时——wait_timeout，即如web应用的数据库连接（池）就属于这类，如果长时间都没有使用这条连接，就会报"the last packet successfully received from the server was xxx milliseconds ago"
[![mysql_sysvar_interactive_timeout](/resources/2014/11/mysql_sysvar_interactive_timeout.png)](/resources/2014/11/mysql_sysvar_interactive_timeout.png "http://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_interactive_timeout")
[![mysql_sysvar_wait_timeout](/resources/2014/11/mysql_sysvar_wait_timeout.png)](/resources/2014/11/mysql_sysvar_wait_timeout.png "http://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_wait_timeout")

查看及设置当前设置的超时时间可以执行如下命令

```shell
mysql> show variables like '%timeout%';
+-----------------------------+----------+
| Variable_name               | Value    |
+-----------------------------+----------+
| connect_timeout             | 10       |
| delayed_insert_timeout      | 300      |
| innodb_flush_log_at_timeout | 1        |
| innodb_lock_wait_timeout    | 50       |
| innodb_rollback_on_timeout  | OFF      |
| interactive_timeout         | 10       |
| lock_wait_timeout           | 31536000 |
| net_read_timeout            | 30       |
| net_write_timeout           | 60       |
| rpl_stop_slave_timeout      | 31536000 |
| slave_net_timeout           | 3600     |
| wait_timeout                | 28800    |
+-----------------------------+----------+
12 rows in set (0.21 sec)

mysql> show global variables like '%timeout%';
+-----------------------------+----------+
| Variable_name               | Value    |
+-----------------------------+----------+
| connect_timeout             | 10       |
| delayed_insert_timeout      | 300      |
| innodb_flush_log_at_timeout | 1        |
| innodb_lock_wait_timeout    | 50       |
| innodb_rollback_on_timeout  | OFF      |
| interactive_timeout         | 28800    |
| lock_wait_timeout           | 31536000 |
| net_read_timeout            | 30       |
| net_write_timeout           | 60       |
| rpl_stop_slave_timeout      | 31536000 |
| slave_net_timeout           | 3600     |
| wait_timeout                | 123      |
+-----------------------------+----------+
12 rows in set (0.01 sec)
```

这里要区分global、非global（session，也叫做local）的系统变量（system variables）和用户自定义变量(user variable)
查看系统变量
> SHOW [GLOBAL | SESSION] VARIABLES
> 
> [LIKE 'pattern' | WHERE expr]
设置系统变量
> SET variable_assignment [, variable_assignment] ...
> 
> 
> variable_assignment:
> 
> user_var_name = expr
> 
> | [GLOBAL | SESSION] system_var_name = expr
> 
> | [@@global. | @@session. | @@]system_var_name = expr
设置用户变量
> SET @var_name = expr;
另外还可以通过select方式查看系统变量(@@.前缀)，用户变量(@前缀，和定义一样)，如
> select @@global.sql_mode, @@session.sql_mode, @@sql_mode;
> set @user_variable=123;
> 
> select @user_variable;

```shell
mysql> SELECT @@global.sql_mode, @@session.sql_mode, @@sql_mode;
+------------------------+------------------------+------------------------+
| @@global.sql_mode      | @@session.sql_mode     | @@sql_mode             |
+------------------------+------------------------+------------------------+
| NO_ENGINE_SUBSTITUTION | NO_ENGINE_SUBSTITUTION | NO_ENGINE_SUBSTITUTION |
+------------------------+------------------------+------------------------+
1 row in set (0.00 sec)

mysql> set @user_variable=123;
Query OK, 0 rows affected (0.01 sec)

mysql> select @user_variable;
+----------------+
| @user_variable |
+----------------+
|            123 |
+----------------+
1 row in set (0.03 sec)
```

参考资料
1\. [MySQL : interactive_timeout v/s wait_timeout](http://www.serveridol.com/2012/04/13/mysql-interactive_timeout-vs-wait_timeout/)
2\. [show-variables](http://dev.mysql.com/doc/refman/5.7/en/show-variables.html)
3\. [set-statement](http://dev.mysql.com/doc/refman/5.7/en/set-statement.html)