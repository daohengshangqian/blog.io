---
layout: post
title:  MySQL5.7 停机迁移
date: 2018-07-24
categories: blog
tags: [MySQL]
description:  MySQL5.7 停机迁移
---

## 查看当前Mysql环境

```sql


mysql> show variables like '%dir%' ;
+-----------------------------------------+----------------------------------+
| Variable_name                           | Value                            |
+-----------------------------------------+----------------------------------+
| basedir                                 | /usr/local/mysql/                |
| binlog_direct_non_transactional_updates | OFF                              |
| character_sets_dir                      | /usr/local/mysql/share/charsets/ |
| datadir                                 | /mysql1/data/                    |
| ignore_db_dirs                          |                                  |
| innodb_data_home_dir                    |                                  |
| innodb_log_group_home_dir               | ./                               |
| innodb_max_dirty_pages_pct              | 75.000000                        |
| innodb_max_dirty_pages_pct_lwm          | 0.000000                         |
| innodb_tmpdir                           |                                  |
| innodb_undo_directory                   | ./                               |
| lc_messages_dir                         | /usr/local/mysql/share/          |
| plugin_dir                              | /usr/local/mysql/lib/plugin/     |
| slave_load_tmpdir                       | /tmp                             |
| tmpdir                                  | /tmp                             |
+-----------------------------------------+----------------------------------+

```
## 查看当前配置文件

```sql


[mysql@bosenrui mysql]$ cat /mysql/my.cnf
[client]
port = 3306
socket = /tmp/mysql1.sock
[mysqld]
port = 3306
socket = /tmp/mysql1.sock
#skip-grant-tables
basedir = /usr/local/mysql
datadir = /mysql1/data
pid-file = /mysql1/mysql.pid
log-error = /mysql1/mysql.log


```
## 停止数据库

```sql
[mysql@bosenrui ~]$ mysqladmin --defaults-file=/mysql1/mysql1.cnf -uroot -proot shutdown
mysqladmin: [Warning] Using a password on the command line interface can be insecure.

```

## 创建新目录

```sql

[root@bosenrui ~]# mkdir /mysql2
[root@bosenrui ~]# chown -R mysql:mysql /mysql2


```



## 复制当前数据库到新目录

```sql
[mysql@bosenrui ~]$ cp -r /mysql1/* /mysql2/

```

## 修改配置文件

```sql
[mysql@bosenrui mysql2]$ mv mysql1.cnf mysql2.cnf
[mysql@bosenrui mysql]$ cat /mysql/my.cnf
[client]
port = 3306
socket = /tmp/mysql2.sock
[mysqld]
port = 3306
socket = /tmp/mysql2.sock
#skip-grant-tables
basedir = /usr/local/mysql
datadir = /mysql2/data
pid-file = /mysql2/mysql.pid
log-error = /mysql2/mysql.log

```

## 启动验证

```sql

[mysql@bosenrui mysql2]$ mysqld --defaults-file=/mysql2/mysql2.cnf  &
[1] 2308

[mysql@bosenrui mysql2]$ mysql --defaults-file=/mysql2/mysql2.cnf -uroot -proot
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.7.21 MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.


mysql> show variables like '%dir%' ;
+-----------------------------------------+----------------------------------+
| Variable_name                           | Value                            |
+-----------------------------------------+----------------------------------+
| basedir                                 | /usr/local/mysql/                |
| binlog_direct_non_transactional_updates | OFF                              |
| character_sets_dir                      | /usr/local/mysql/share/charsets/ |
| datadir                                 | /mysql2/data/                    |
| ignore_db_dirs                          |                                  |
| innodb_data_home_dir                    |                                  |
| innodb_log_group_home_dir               | ./                               |
| innodb_max_dirty_pages_pct              | 75.000000                        |
| innodb_max_dirty_pages_pct_lwm          | 0.000000                         |
| innodb_tmpdir                           |                                  |
| innodb_undo_directory                   | ./                               |
| lc_messages_dir                         | /usr/local/mysql/share/          |
| plugin_dir                              | /usr/local/mysql/lib/plugin/     |
| slave_load_tmpdir                       | /tmp                             |
| tmpdir                                  | /tmp                             |
+-----------------------------------------+----------------------------------+
15 rows in set (0.00 sec)

```
