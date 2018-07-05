---
layout: post
title:  Buffer_pool热数据转储特性
date: 2018-07-03
categories: blog
tags: [MySQL]
description: Buffer_pool热数据转储特性
---



从Mysql 5.6开始，提供InnoDB buffer的预热功能　该特性在5.7中得到加强
下面来看下几个相关参数

参数名 | 作用
--- | ---
innodb_buffer_pool_dump_at_shutdown | 是否在关闭时热数据保存在本地磁盘
innodb_buffer_pool_load_at_startup  |是否在启动时从本地磁盘加载热数据
innodb_buffer_pool_filename         |用以储存热数据文件名
innodb_buffer_pool_dump_now　　　　 |立即保存当前热数据到磁盘
innodb_buffer_pool_load_now         |立即从磁盘中加载热数据文件
innodb_buffer_pool_load_abort       |立即终止加载热数据文件
innodb_buffer_pool_dump_pct 　　　　|转存热数据的百分比




## 相关实验如下

### 查看当前datadir文件

```sql

[mysql@dao data]$ ll
total 1574152
-rw-rw----. 1 mysql mysql         56 Jun 29 05:49 auto.cnf
-rw-rw----. 1 mysql mysql 1073741824 Jun 29 05:57 ibdata1
-rw-rw----. 1 mysql mysql  268435456 Jun 29 05:57 ib_logfile0
-rw-rw----. 1 mysql mysql  268435456 Jun 29 05:49 ib_logfile1
drwx------. 2 mysql mysql       4096 Jun 29 05:54 mysql
-rw-rw----. 1 mysql mysql        120 Jun 29 05:53 mysql-bin.000001
-rw-rw----. 1 mysql mysql        120 Jun 29 05:54 mysql-bin.000002
-rw-rw----. 1 mysql mysql      65402 Jun 29 05:54 mysql-bin.000003
-rw-rw----. 1 mysql mysql    1206067 Jun 29 05:54 mysql-bin.000004
-rw-rw----. 1 mysql mysql       2501 Jun 29 06:00 mysql-bin.000005
-rw-rw----. 1 mysql mysql         95 Jun 29 05:57 mysql-bin.index
-rw-rw----. 1 mysql mysql          6 Jun 29 05:57 mysql.pid
drwx------. 2 mysql mysql       4096 Jun 29 05:54 performance_schema
drwx------. 2 mysql mysql       4096 Jun 29 05:54 test

```

### 手工转储热数据


```sql

mysql> set @@global.innodb_buffer_pool_load_now =1 ;
Query OK, 0 rows affected (0.01 sec)

```

### 查看转储后的文件

```sql

[mysql@dao data]$ ll ib_buffer_pool
-rw-rw----. 1 mysql mysql 884 Jun 29 08:04 ib_buffer_pool

```
