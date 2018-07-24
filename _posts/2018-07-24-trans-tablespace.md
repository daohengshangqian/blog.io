---
layout: post
title: MySQL5.7传输表空间实战
date: 2018-07-24
categories: blog
tags: [MySQL]
description: MySQL5.7传输表空间实战
---

## 概述 

在MySQL中我们可以使用可传输表空间将表从一个数据库导入到另一个数据库.

## 可传输表空间使用注意事项


- 必须开启 innodb_file_per_table
- 当这个表处于quiesced状态,甚至不能被select
两边实例的page size 一致
-5.7 版本之前，不支持分区表transport
- 外键相关的表，必须设置 foreign_key_checks=0 才能成功
- ALTER TABLE ... IMPORT TABLESPACE 不需要.cfg metadata file . 但是，这样的话， MySQL就不会对schema进行verificate
- 5.6以及更高版本， import&export 版本必须在同一个series
- 在replication环境中， master & slave 都必须开启 innodb_file_per_table
- 对于InnoDB general tablespace，不支持discard & import tablespace
- 如果两边服务器的table row_format设置的不一样，会导致schema mismatch error
- 加密过的InnoDB tablespace 必须要拷贝.cfp 文件

## 实验过程

- 源端建立测试表并插入测试数据 
- 目标端建立同名表
- 源端刷新表并加锁以便导出
- 删除目标端的表空间
- 复制配置文件与数据文件到目标端
- 源端释放表锁
- 目标端导入表空间
- 验证 验证 验证 


### 源端建立测试表并插入测试数据 

```sql

[mysql@bosenrui ~]$ mysql  --defaults-file=/mysql1/mysql1.cnf -uroot -proot
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.21 MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> use dao ;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> create table test_import as select *  from dept ;
Query OK, 4 rows affected (0.10 sec)
Records: 4  Duplicates: 0  Warnings: 0

mysql> select * from test_import ;
+--------+------------+----------+
| deptNO | DNAME      | LOC      |
+--------+------------+----------+
|     10 | ACCOUNTING | NEW YORK |
|     20 | RESEARCH   | DALLAS   |
|     30 | SALES      | CHICAGO  |
|     40 | OPERATIONS | BOSTON   |
+--------+------------+----------+
4 rows in set (0.00 sec)



```


## 目标端建立同名表

```sql

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

mysql> use dao ;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed

mysql> create table test_import as select * from dept where 1=2 ;
Query OK, 0 rows affected (0.02 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> select * from test_import ;
Empty set (0.00 sec)



```


### 源端刷新表并加锁以便导出

```sql

mysql> flush table test_import for export ;
Query OK, 0 rows affected (0.00 sec)


```

- 注意mysql 终端不可退出 否则加锁会释放并且cfg文件会自动删除
```sql

[root@bosenrui ~]# su - mysql
[mysql@bosenrui ~]$ ll /mysql1/data/dao/test*
-rw-r----- 1 mysql mysql   483 Jul 24 20:20 /mysql1/data/dao/test_import.cfg
-rw-r----- 1 mysql mysql  8624 Jul 24 20:00 /mysql1/data/dao/test_import.frm
-rw-r----- 1 mysql mysql 98304 Jul 24 20:00 /mysql1/data/dao/test_import.ibd

```

### 删除目标端的表空间

```sql
[mysql@bosenrui mysql2]$ ll /mysql2/data/dao/test*
-rw-r----- 1 mysql mysql  8624 Jul 24 20:07 /mysql2/data/dao/test_import.frm
-rw-r----- 1 mysql mysql 98304 Jul 24 20:07 /mysql2/data/dao/test_import.ibd
```

```sql

[mysql@bosenrui mysql2]$ mysql --defaults-file=/mysql2/mysql2.cnf -uroot -proot
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 4
Server version: 5.7.21 MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> use dao ;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> alter table test_import discard tablespace ;
Query OK, 0 rows affected (0.01 sec)



```

```sql
[mysql@bosenrui mysql2]$ ll /mysql2/data/dao/test*
-rw-r----- 1 mysql mysql 8624 Jul 24 20:07 /mysql2/data/dao/test_import.frm

```


### 复制配置文件与数据文件到目标端

```sql

[mysql@bosenrui ~]$ cp /mysql1/data/dao/test_import.ibd /mysql2/data/dao/
[mysql@bosenrui ~]$ cp /mysql1/data/dao/test_import.cfg /mysql2/data/dao/

[mysql@bosenrui ~]$ ll /mysql2/data/dao/test*
-rw-r----- 1 mysql mysql   483 Jul 24 20:22 /mysql2/data/dao/test_import.cfg
-rw-r----- 1 mysql mysql  8624 Jul 24 20:07 /mysql2/data/dao/test_import.frm
-rw-r----- 1 mysql mysql 98304 Jul 24 20:22 /mysql2/data/dao/test_import.ibd


```


### 源端释放表锁


```sql

mysql> unlock tables ;
Query OK, 0 rows affected (0.00 sec)



```


### 目标端导入表空间

```sql

[mysql@bosenrui ~]$ mysql --defaults-file=/mysql2/mysql2.cnf -uroot -proot
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 6
Server version: 5.7.21 MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> use dao ;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed, 1 warning


mysql> alter table test_import import tablespace ;
Query OK, 0 rows affected (0.04 sec)

```

### 验证结果

```sql



mysql> select * from test_import ;
+--------+------------+----------+
| deptNO | DNAME      | LOC      |
+--------+------------+----------+
|     10 | ACCOUNTING | NEW YORK |
|     20 | RESEARCH   | DALLAS   |
|     30 | SALES      | CHICAGO  |
|     40 | OPERATIONS | BOSTON   |
+--------+------------+----------+
4 rows in set (0.00 sec)

```
