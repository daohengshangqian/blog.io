---
layout: post
title:  MySQL redo管理实践
date: 2018-07-20
categories: blog
tags: [MySQL]
description:  MySQL redo管理实践
---

## redo 文件的作用

Redo log是一个基于磁盘文件的数据结构,用以在实例恢复时校正由未完成事务写入的数据文件.

当需要改变某个页的数据时,InnoDB先将这个页加载到buffer pool中,在buffer pool中进行数据的修改,并将修改行为记录到redo log buffer中.这一行为我们称之为日志先行(Write-Ahead Logging).

根据参数的不同,Innodb对日志的写入会有不同的写入策略.

最后才将修改过的数据记录到硬盘中.

从性能的角度来说,数据的变更会造成随机I/O,而redo log 的写方式为追加写,此方式为顺序I/O.所以将变更内容先行写入redo ,而将改变的页慢慢的刷回磁盘可显著增加实例处理数据改变的能力.



## redo 相关参数

```sql

mysql> show variables like 'innodb%log%' ;
+----------------------------------+------------+
| Variable_name                    | Value      |
+----------------------------------+------------+
| innodb_api_enable_binlog         | OFF        |
| innodb_flush_log_at_timeout      | 1          |
| innodb_flush_log_at_trx_commit   | 1          |
| innodb_locks_unsafe_for_binlog   | OFF        |
| innodb_log_buffer_size           | 16777216   |
| innodb_log_checksums             | ON         |
| innodb_log_compressed_pages      | ON         |
| innodb_log_file_size             | 50331648   |
| innodb_log_files_in_group        | 2          |
| innodb_log_group_home_dir        | ./         |
| innodb_log_write_ahead_size      | 8192       |
| innodb_max_undo_log_size         | 1073741824 |
| innodb_online_alter_log_max_size | 134217728  |
| innodb_undo_log_truncate         | OFF        |
| innodb_undo_logs                 | 128        |
+----------------------------------+------------+

```
 参数名称 | 含义
 --- | ---
 innodb_flush_log_at_trx_commit   | log file 写入策略
 innodb_log_buffer_size           | redo log buffer 大小
 innodb_log_file_size             | redo 文件大小
 innodb_log_files_in_group        | redo 文件数量
 innodb_log_group_home_dir        | redo 文件所处文件夹相对路径
 innodb_log_write_ahead_size      | 每次追加写到redo日志文件的大小，默认为页的大小。
 innodb_online_alter_log_max_size | 在线DDL时,临时日志尺寸上限
 
 其中最为复杂的参数为
 
 - innodb_flush_log_at_trx_commit
 

参数值 | redo写入 | 系统层刷新redo log | 风险
--- | --- | --- | ---
0 | 每秒写入log file | 每秒同步刷新                  | 实例崩溃,或主机崩溃丢失最多丢失一秒内的事务
1 | 事务完成后立即写入 | 立即刷新redo log file到磁盘  | 主机崩溃,最多丢失一个语句或一个事务
2 | 事务完成后立即写入 | 每秒刷新redo log file到磁盘 | 主机崩溃 最多丢失一秒内的事务


## redo log文件的管理

### 增加redo log

#### 查看当前redo log 数量

```sql

[mysql@bosenrui ~]$ ll /mysql/data/ib_log*
-rw-r----- 1 mysql mysql 50331648 Apr  1 10:35 /mysql/data/ib_logfile0
-rw-r----- 1 mysql mysql 50331648 Apr  1 10:32 /mysql/data/ib_logfile1

```

#### 修改参数文件

```sql

[mysql@bosenrui data]$ cat /etc/my.cnf
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
user=mysql
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
[mysql@bosenrui data]$ cat /mysql/my.cnf
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

#### 重启MySQL


```sql


[mysql@bosenrui ~]$ mysqladmin  -uroot -p shutdown
Enter password:

[mysql@bosenrui ~]$ mysqld  --defaults-file=/mysql/my.cnf &

```


#### 查看datafir

```sql

[mysql@bosenrui ~]$ cd /mysql/data/
[mysql@bosenrui data]$ ll
total 221216
-rw-r----- 1 mysql mysql       56 Apr  1 10:32 auto.cnf
-rw-r----- 1 mysql mysql      332 Apr  1 11:40 ib_buffer_pool
-rw-r----- 1 mysql mysql 50331648 Apr  1 11:48 ib_logfile0
-rw-r----- 1 mysql mysql 50331648 Apr  1 11:09 ib_logfile1
-rw-r----- 1 mysql mysql 50331648 Apr  1 11:09 ib_logfile2
-rw-r----- 1 mysql mysql 50331648 Apr  1 11:09 ib_logfile3
-rw-r----- 1 mysql mysql 12582912 Apr  1 11:48 ibdata1
-rw-r----- 1 mysql mysql 12582912 Apr  1 11:48 ibtmp1
drwxr-x--- 2 mysql mysql     4096 Apr  1 10:32 mysql
-rw-r----- 1 mysql mysql        5 Apr  1 11:48 mysql.pid
drwxr-x--- 2 mysql mysql     4096 Apr  1 10:32 performance_schema
drwxr-x--- 2 mysql mysql    12288 Apr  1 10:32 sys

```

### 减少redo log file数量


#### 查看当前redo log 数量


```sql

[mysql@bosenrui ~]$ cd /mysql/data/
[mysql@bosenrui data]$ ll
total 221216
-rw-r----- 1 mysql mysql       56 Apr  1 10:32 auto.cnf
-rw-r----- 1 mysql mysql      332 Apr  1 11:40 ib_buffer_pool
-rw-r----- 1 mysql mysql 50331648 Apr  1 11:48 ib_logfile0
-rw-r----- 1 mysql mysql 50331648 Apr  1 11:09 ib_logfile1
-rw-r----- 1 mysql mysql 50331648 Apr  1 11:09 ib_logfile2
-rw-r----- 1 mysql mysql 50331648 Apr  1 11:09 ib_logfile3
-rw-r----- 1 mysql mysql 12582912 Apr  1 11:48 ibdata1
-rw-r----- 1 mysql mysql 12582912 Apr  1 11:48 ibtmp1
drwxr-x--- 2 mysql mysql     4096 Apr  1 10:32 mysql
-rw-r----- 1 mysql mysql        5 Apr  1 11:48 mysql.pid
drwxr-x--- 2 mysql mysql     4096 Apr  1 10:32 performance_schema
drwxr-x--- 2 mysql mysql    12288 Apr  1 10:32 sys

```


#### 修改参数文件

```sql




```

#### 重启Mysql

```sql
[mysql@bosenrui data]$  mysqladmin  -uroot -p shutdown
Enter password:
[mysql@bosenrui data]$ mysqld  --defaults-file=/mysql/my.cnf &
[1] 3431


```

#### 查看redolog 数量


```sql
[mysql@bosenrui data]$ ll
total 172064
-rw-r----- 1 mysql mysql       56 Apr  1 10:32 auto.cnf
-rw-r----- 1 mysql mysql      302 Apr  1 11:57 ib_buffer_pool
-rw-r----- 1 mysql mysql 50331648 Apr  1 11:57 ib_logfile0
-rw-r----- 1 mysql mysql 50331648 Apr  1 11:57 ib_logfile1
-rw-r----- 1 mysql mysql 50331648 Apr  1 11:57 ib_logfile2
-rw-r----- 1 mysql mysql 12582912 Apr  1 11:57 ibdata1
-rw-r----- 1 mysql mysql 12582912 Apr  1 11:57 ibtmp1
drwxr-x--- 2 mysql mysql     4096 Apr  1 10:32 mysql
-rw-r----- 1 mysql mysql        5 Apr  1 11:57 mysql.pid
drwxr-x--- 2 mysql mysql     4096 Apr  1 10:32 performance_schema
drwxr-x--- 2 mysql mysql    12288 Apr  1 10:32 sys

```

### 增加MySQL redo log大小 


#### 查看当前redo log 

```sql


[mysql@bosenrui data]$ ll
total 172064
-rw-r----- 1 mysql mysql       56 Apr  1 10:32 auto.cnf
-rw-r----- 1 mysql mysql      302 Apr  1 11:57 ib_buffer_pool
-rw-r----- 1 mysql mysql 50331648 Apr  1 11:57 ib_logfile0
-rw-r----- 1 mysql mysql 50331648 Apr  1 11:57 ib_logfile1
-rw-r----- 1 mysql mysql 50331648 Apr  1 11:57 ib_logfile2
-rw-r----- 1 mysql mysql 12582912 Apr  1 11:57 ibdata1
-rw-r----- 1 mysql mysql 12582912 Apr  1 11:57 ibtmp1
drwxr-x--- 2 mysql mysql     4096 Apr  1 10:32 mysql
-rw-r----- 1 mysql mysql        5 Apr  1 11:57 mysql.pid
drwxr-x--- 2 mysql mysql     4096 Apr  1 10:32 performance_schema
drwxr-x--- 2 mysql mysql    12288 Apr  1 10:32 sys


```


#### 修改参数文件


```sql

[mysql@bosenrui data]$ cat /mysql/my.cnf
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
innodb_log_files_in_group=3
innodb_log_file_size=100663296

```


#### 重启Mysql
```sql
[mysql@bosenrui data]$  mysqladmin  -uroot -p shutdown
Enter password:
[1]+  Done                    mysqld --defaults-file=/mysql/my.cnf
[mysql@bosenrui data]$ mysqld  --defaults-file=/mysql/my.cnf &
[1] 3543
[mysql@bosenrui data]$ ll

```

#### 查看redo log 大小


```sql
[mysql@bosenrui data]$ ll
total 307228
-rw-r----- 1 mysql mysql        56 Apr  1 10:32 auto.cnf
-rw-r----- 1 mysql mysql       294 Apr  1 12:02 ib_buffer_pool
-rw-r----- 1 mysql mysql 100663296 Apr  1 12:02 ib_logfile1
-rw-r----- 1 mysql mysql 100663296 Apr  1 12:02 ib_logfile101
-rw-r----- 1 mysql mysql 100663296 Apr  1 12:02 ib_logfile2
-rw-r----- 1 mysql mysql  12582912 Apr  1 12:02 ibdata1
drwxr-x--- 2 mysql mysql      4096 Apr  1 10:32 mysql
drwxr-x--- 2 mysql mysql      4096 Apr  1 10:32 performance_schema
drwxr-x--- 2 mysql mysql     12288 Apr  1 10:32 sys


```

### 减小red log大小 
#### 查看当前redo log 大小

```sql

[mysql@bosenrui data]$ ll
total 307228
-rw-r----- 1 mysql mysql        56 Apr  1 10:32 auto.cnf
-rw-r----- 1 mysql mysql       294 Apr  1 12:02 ib_buffer_pool
-rw-r----- 1 mysql mysql 100663296 Apr  1 12:02 ib_logfile1
-rw-r----- 1 mysql mysql 100663296 Apr  1 12:02 ib_logfile101
-rw-r----- 1 mysql mysql 100663296 Apr  1 12:02 ib_logfile2
-rw-r----- 1 mysql mysql  12582912 Apr  1 12:02 ibdata1
drwxr-x--- 2 mysql mysql      4096 Apr  1 10:32 mysql
drwxr-x--- 2 mysql mysql      4096 Apr  1 10:32 performance_schema
drwxr-x--- 2 mysql mysql     12288 Apr  1 10:32 sys
```

#### 修改参数文件

```sql


[mysql@bosenrui data]$ cat  /mysql/my.cnf
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
innodb_log_files_in_group=3
innodb_log_file_size=50331648

```


#### 重启MySQL

```sql

[mysql@bosenrui data]$ mysqladmin -uroot -p shutdown
Enter password:
[1]+  Done                    mysqld --defaults-file=/mysql/my.cnf
[mysql@bosenrui data]$ mysqld --defaults-file=/mysql/my.cnf

```

#### 查看redo log 大下

```sql
[mysql@bosenrui data]$ ll
total 172064
-rw-r----- 1 mysql mysql       56 Apr  1 10:32 auto.cnf
-rw-r----- 1 mysql mysql      290 Apr  1 12:07 ib_buffer_pool
-rw-r----- 1 mysql mysql 50331648 Apr  1 12:08 ib_logfile0
-rw-r----- 1 mysql mysql 50331648 Apr  1 12:08 ib_logfile1
-rw-r----- 1 mysql mysql 50331648 Apr  1 12:08 ib_logfile2
-rw-r----- 1 mysql mysql 12582912 Apr  1 12:08 ibdata1
-rw-r----- 1 mysql mysql 12582912 Apr  1 12:08 ibtmp1
drwxr-x--- 2 mysql mysql     4096 Apr  1 10:32 mysql
-rw-r----- 1 mysql mysql        5 Apr  1 12:08 mysql.pid
drwxr-x--- 2 mysql mysql     4096 Apr  1 10:32 performance_schema
drwxr-x--- 2 mysql mysql    12288 Apr  1 10:32 sys

```
