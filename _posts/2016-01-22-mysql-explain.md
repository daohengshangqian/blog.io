---
layout: post
title:  MySQL执行计划解析
date: 2016-01-22
categories: blog
tags: [MySQL]
description:   MySQL执行计划解析
---

## 跳过验证

```sql
[mysql@bosenrui ~]$ cat /mysql/my.cnf
[client]
port = 3306
socket = /tmp/mysql.sock
[mysqld]
port = 3306
socket = /tmp/mysql.sock
skip-grant-tables
basedir = /mysql
datadir = /mysql/data
pid-file = /mysql/data/mysql.pid
log-error = /mysql/mysql.log
innodb_log_files_in_group=4

```

## 启动MySQL

```sql

[mysql@bosenrui ~]$ mysqld --defaults-file=/mysql/my.cnf &

```


## 重置密码


```sql


mysql> update user
    ->    set authentication_string = password('root'),
    ->        password_expired = 'N',
    ->    password_last_changed = now()
    ->  where user = 'root';
Query OK, 1 row affected, 1 warning (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 1

    
```

## 修改参数文件


```sql
[mysql@bosenrui ~]$ cat  /mysql/my.cnf
[client]
port = 3306
socket = /tmp/mysql.sock
[mysqld]
port = 3306
socket = /tmp/mysql.sock
#skip-grant-tables
basedir = /mysql
datadir = /mysql/data
pid-file = /mysql/data/mysql.pid
log-error = /mysql/mysql.log
innodb_log_files_in_group=4


```

## 重启Mysql

```sql


[mysql@bosenrui ~]$ mysqladmin  -uroot -p shutdown
Enter password:

[mysql@bosenrui ~]$ mysqld  --defaults-file=/mysql/my.cnf &

```

## 进入数据库

```sql

[mysql@bosenrui ~]$ mysql -uroot -proot
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 4
Server version: 5.7.19 Source distribution

Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>

 
```
