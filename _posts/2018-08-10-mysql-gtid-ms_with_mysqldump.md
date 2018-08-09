---
layout: post
title: 使用MysqlDump搭建GTID主从
date: 2018-08-10
categories: blog
tags: [MySQL]
description: 使用MysqlDump搭建GTID主从
---

## 查看实例uuid


### 主库
```sql

[mysql@node1 ~]$ cat /mysql1/data/auto.cnf
[auto]
server-uuid=03fde241-9a55-11e8-aa12-000c291f013a

```

### 从库
```sql
[mysql@node2 ~]$ cat /mysql1/data/auto.cnf
[auto]
server-uuid=342ad063-9c15-11e8-8eb9-005056296858


```

## 修改参数文件

### 主库参数文件

```sql

[mysql@node1 ~]$ cat /mysql1/mysql1.cnf
[client]
port = 3306
socket = /tmp/mysql1.sock
[mysqld]
port = 3306
socket = /tmp/mysql1.sock
#skip-grant-tables
basedir = /usr/local/mysql
datadir = /mysql1/data
pid-file = /mysql1/mysql1.pid
log-error = /mysql1/mysql1.log
log-bin=/mysql1/data/mysql-bin
server_id = 1

#gtid

gtid_mode=on
log_slave_updates=1
enforce_gtid_consistency=1

```

### 从库参数文件

```sql
[mysql@node2 ~]$ cat /mysql1/mysql1.cnf
[client]
port = 3306
socket = /tmp/mysql1.sock
[mysqld]
port = 3306
socket = /tmp/mysql1.sock
#skip-grant-tables
basedir = /usr/local/mysql
datadir = /mysql1/data
pid-file = /mysql1/mysql1.pid
log-error = /mysql1/mysql1.log
log-bin=/mysql1/data/mysql-bin
server_id = 2
slave-skip-errors=1062,1032
log-slave-updates=1



#gtid

gtid_mode=on
log_slave_updates=1
enforce_gtid_consistency=1
[mysql@node2 ~]$

```

## 启动主库

```sql

[mysql@node1 ~]$  mysqld --defaults-file=/mysql1/mysql1.cnf &                      [1] 3807


```

## 插入一些测试数据

```sql


mysql> use test ;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> select  * from t1 ;
+----+------+
| c1 | c2   |
+----+------+
|  1 |    8 |
|  2 |    3 |
+----+------+
2 rows in set (0.00 sec)

mysql> insert into t1 values (3,4) ;
Query OK, 1 row affected (0.01 sec)

mysql> insert into t1 values (5,6) ;
Query OK, 1 row affected (0.01 sec)

```


## 建立同步用户 

```sql
mysql> GRANT ALL PRIVILEGES ON *.* TO 'dao'@'192.168.2.%' IDENTIFIED BY 'dao' ;
Query OK, 0 rows affected, 1 warning (0.00 sec)


```

## 使用mysqldump备份数据库

```sql


[mysql@node1 ~]$ mysqldump -S /tmp/mysql1.sock -u root -proot --master-data=2 --single-transaction -A > /home/mysql/alldata.sql
mysqldump: [Warning] Using a password on the command line interface can be insecure.
Warning: A partial dump from a server that has GTIDs will by default include the GTIDs of all transactions, even those that changed suppressed parts of the database. If you don't want to restore GTIDs, pass --set-gtid-purged=OFF. To make a complete dump, pass --all-databases --triggers --routines --events.



```

## 继续在主库中添加数据

```sql

mysql> insert into t1 values (7,8) ;
ERROR 1046 (3D000): No database selected
mysql> insert into test.t1 values (7,8) ;
Query OK, 1 row affected (0.22 sec)

mysql> insert into test.t1 values (9,10) ;
Query OK, 1 row affected (0.01 sec)


```

## 查看主库使用的binlog信息


```sql


mysql> show master status ;
+------------------+----------+--------------+------------------+------------------------------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set                        |
+------------------+----------+--------------+------------------+------------------------------------------+
| mysql-bin.000005 |     1467 |              |                  | 03fde241-9a55-11e8-aa12-000c291f013a:1-5 |
+------------------+----------+--------------+------------------+------------------------------------------+
1 row in set (0.00 sec)

```

## 使用mysqlbinlog命令解析binlog

```sql

[mysql@node1 ~]$ cd /mysql1/data/
[mysql@node1 data]$ mysqlbinlog -vv mysql-bin.000005 > /home/mysql/binlog.sql

```


## 查看解析后的文本

```sql
[mysql@node1 data]$ cat /home/mysql/binlog.sql
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=1*/;
/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;
DELIMITER /*!*/;
# at 4
#180810  5:05:03 server id 1  end_log_pos 123 CRC32 0x32b53dc9  Start: binlog v 4, server v 5.7.21-log created 180810  5:05:03 at startup
# Warning: this binlog is either in use or was not closed properly.
ROLLBACK/*!*/;
BINLOG '
f6xsWw8BAAAAdwAAAHsAAAABAAQANS43LjIxLWxvZwAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAAB/rGxbEzgNAAgAEgAEBAQEEgAAXwAEGggAAAAICAgCAAAACgoKKioAEjQA
Ack9tTI=
'/*!*/;
# at 123
#180810  5:05:03 server id 1  end_log_pos 154 CRC32 0x8da8e2f7  Previous-GTIDs
# [empty]
# at 154
#180810  5:06:42 server id 1  end_log_pos 219 CRC32 0x58e13cd5  GTID    last_committed=0        sequence_number=1       rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= '03fde241-9a55-11e8-aa12-000c291f013a:1'/*!*/;
# at 219
#180810  5:06:42 server id 1  end_log_pos 291 CRC32 0x05b03cbf  Query   thread_id=2     exec_time=0     error_code=0
SET TIMESTAMP=1533848802/*!*/;
SET @@session.pseudo_thread_id=2/*!*/;
SET @@session.foreign_key_checks=1, @@session.sql_auto_is_null=0, @@session.unique_checks=1, @@session.autocommit=1/*!*/;
SET @@session.sql_mode=1436549152/*!*/;
SET @@session.auto_increment_increment=1, @@session.auto_increment_offset=1/*!*/;
/*!\C utf8 *//*!*/;
SET @@session.character_set_client=33,@@session.collation_connection=33,@@session.collation_server=8/*!*/;
SET @@session.lc_time_names=0/*!*/;
SET @@session.collation_database=DEFAULT/*!*/;
BEGIN
/*!*/;
# at 291
#180810  5:06:42 server id 1  end_log_pos 337 CRC32 0xc892d4c4  Table_map: `test`.`t1` mapped to number 109
# at 337
#180810  5:06:42 server id 1  end_log_pos 381 CRC32 0x144c04f7  Write_rows: table id 109 flags: STMT_END_F

BINLOG '
4qxsWxMBAAAALgAAAFEBAAAAAG0AAAAAAAEABHRlc3QAAnQxAAIDAwACxNSSyA==
4qxsWx4BAAAALAAAAH0BAAAAAG0AAAAAAAEAAgAC//wDAAAABAAAAPcETBQ=
'/*!*/;
### INSERT INTO `test`.`t1`
### SET
###   @1=3 /* INT meta=0 nullable=0 is_null=0 */
###   @2=4 /* INT meta=0 nullable=1 is_null=0 */
# at 381
#180810  5:06:42 server id 1  end_log_pos 412 CRC32 0xa7468b54  Xid = 10
COMMIT/*!*/;
# at 412
#180810  5:06:49 server id 1  end_log_pos 477 CRC32 0x77e9cf62  GTID    last_committed=1        sequence_number=2       rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= '03fde241-9a55-11e8-aa12-000c291f013a:2'/*!*/;
# at 477
#180810  5:06:49 server id 1  end_log_pos 549 CRC32 0xfb1f09f1  Query   thread_id=2     exec_time=0     error_code=0
SET TIMESTAMP=1533848809/*!*/;
BEGIN
/*!*/;
# at 549
#180810  5:06:49 server id 1  end_log_pos 595 CRC32 0x3de3de6f  Table_map: `test`.`t1` mapped to number 109
# at 595
#180810  5:06:49 server id 1  end_log_pos 639 CRC32 0x07aa2ff6  Write_rows: table id 109 flags: STMT_END_F

BINLOG '
6axsWxMBAAAALgAAAFMCAAAAAG0AAAAAAAEABHRlc3QAAnQxAAIDAwACb97jPQ==
6axsWx4BAAAALAAAAH8CAAAAAG0AAAAAAAEAAgAC//wFAAAABgAAAPYvqgc=
'/*!*/;
### INSERT INTO `test`.`t1`
### SET
###   @1=5 /* INT meta=0 nullable=0 is_null=0 */
###   @2=6 /* INT meta=0 nullable=1 is_null=0 */
# at 639
#180810  5:06:49 server id 1  end_log_pos 670 CRC32 0x465ed8e0  Xid = 11
COMMIT/*!*/;
# at 670
#180810  5:14:39 server id 1  end_log_pos 735 CRC32 0xd719f382  GTID    last_committed=2        sequence_number=3       rbr_only=no
SET @@SESSION.GTID_NEXT= '03fde241-9a55-11e8-aa12-000c291f013a:3'/*!*/;
# at 735
#180810  5:14:39 server id 1  end_log_pos 959 CRC32 0x84f225fb  Query   thread_id=4     exec_time=0     error_code=0
SET TIMESTAMP=1533849279/*!*/;
GRANT ALL PRIVILEGES ON *.* TO 'dao'@'192.168.2.%' IDENTIFIED WITH 'mysql_native_password' AS '*1F49BE757695703A65B50F6F6C6129B084B657D6'
/*!*/;
# at 959
#180810  5:17:37 server id 1  end_log_pos 1024 CRC32 0x8aaf2a3d         GTID    last_committed=3        sequence_number=4       rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= '03fde241-9a55-11e8-aa12-000c291f013a:4'/*!*/;
# at 1024
#180810  5:17:37 server id 1  end_log_pos 1092 CRC32 0x519f5c9c         Query   thread_id=6     exec_time=0     error_code=0
SET TIMESTAMP=1533849457/*!*/;
BEGIN
/*!*/;
# at 1092
#180810  5:17:37 server id 1  end_log_pos 1138 CRC32 0x0e274cfc         Table_map: `test`.`t1` mapped to number 175
# at 1138
#180810  5:17:37 server id 1  end_log_pos 1182 CRC32 0xc11b7dbd         Write_rows: table id 175 flags: STMT_END_F

BINLOG '
ca9sWxMBAAAALgAAAHIEAAAAAK8AAAAAAAEABHRlc3QAAnQxAAIDAwAC/EwnDg==
ca9sWx4BAAAALAAAAJ4EAAAAAK8AAAAAAAEAAgAC//wHAAAACAAAAL19G8E=
'/*!*/;
### INSERT INTO `test`.`t1`
### SET
###   @1=7 /* INT meta=0 nullable=0 is_null=0 */
###   @2=8 /* INT meta=0 nullable=1 is_null=0 */
# at 1182
#180810  5:17:37 server id 1  end_log_pos 1213 CRC32 0x29067551         Xid = 933
COMMIT/*!*/;
# at 1213
#180810  5:17:42 server id 1  end_log_pos 1278 CRC32 0x0cbdeb50         GTID    last_committed=4        sequence_number=5       rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= '03fde241-9a55-11e8-aa12-000c291f013a:5'/*!*/;
# at 1278
#180810  5:17:42 server id 1  end_log_pos 1346 CRC32 0x1fd2aaad         Query   thread_id=6     exec_time=0     error_code=0
SET TIMESTAMP=1533849462/*!*/;
BEGIN
/*!*/;
# at 1346
#180810  5:17:42 server id 1  end_log_pos 1392 CRC32 0x18d391a1         Table_map: `test`.`t1` mapped to number 175
# at 1392
#180810  5:17:42 server id 1  end_log_pos 1436 CRC32 0xbea0c446         Write_rows: table id 175 flags: STMT_END_F

BINLOG '
dq9sWxMBAAAALgAAAHAFAAAAAK8AAAAAAAEABHRlc3QAAnQxAAIDAwACoZHTGA==
dq9sWx4BAAAALAAAAJwFAAAAAK8AAAAAAAEAAgAC//wJAAAACgAAAEbEoL4=
'/*!*/;
### INSERT INTO `test`.`t1`
### SET
###   @1=9 /* INT meta=0 nullable=0 is_null=0 */
###   @2=10 /* INT meta=0 nullable=1 is_null=0 */
# at 1436
#180810  5:17:42 server id 1  end_log_pos 1467 CRC32 0x5008a607         Xid = 934
COMMIT/*!*/;
SET @@SESSION.GTID_NEXT= 'AUTOMATIC' /* added by mysqlbinlog */ /*!*/;
DELIMITER ;
# End of log file
/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/;

```

## 将备份传到从库

```sql

[mysql@node1 data]$ scp  /home/mysql/alldata.sql  192.168.2.62:/home/mysql/
mysql@192.168.2.62's password:
alldata.sql    

```

## 从库进行恢复

```sql


mysql> source /home/mysql/alldata.sql ;
Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected, 1 warning (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.01 sec)

Query OK, 1 row affected (0.00 sec)

Database changed
Query OK, 0 rows affected (0.01 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.02 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.01 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 2 rows affected (0.00 sec)
Records: 2  Duplicates: 0  Warnings: 0

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.01 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.13 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 2 rows affected (0.00 sec)
Records: 2  Duplicates: 0  Warnings: 0

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.01 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.01 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.01 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.01 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.05 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 40 rows affected (0.00 sec)
Records: 40  Duplicates: 0  Warnings: 0

Query OK, 0 rows affected (0.01 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.09 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 693 rows affected (0.11 sec)
Records: 693  Duplicates: 0  Warnings: 0

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.01 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.05 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 1406 rows affected (0.02 sec)
Records: 1406  Duplicates: 0  Warnings: 0

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.01 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.06 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 637 rows affected (0.80 sec)
Records: 637  Duplicates: 0  Warnings: 0

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.03 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.10 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 21 rows affected (0.01 sec)
Records: 21  Duplicates: 0  Warnings: 0

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.03 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.16 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 4 rows affected (0.02 sec)
Records: 4  Duplicates: 0  Warnings: 0

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.02 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.05 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.12 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.03 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.02 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.02 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 1 row affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.03 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.13 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 6 rows affected (0.02 sec)
Records: 6  Duplicates: 0  Warnings: 0

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.03 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.13 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.03 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.14 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.03 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 2 rows affected (0.00 sec)
Records: 2  Duplicates: 0  Warnings: 0

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.03 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.13 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.03 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.14 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.03 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.01 sec)

Query OK, 0 rows affected (0.14 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.04 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.14 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.04 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.15 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.03 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 5 rows affected (0.00 sec)
Records: 5  Duplicates: 0  Warnings: 0

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 1 row affected (0.00 sec)

Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
Query OK, 0 rows affected (0.08 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.21 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 24 rows affected (0.05 sec)
Records: 24  Duplicates: 0  Warnings: 0

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.10 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.17 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 4 rows affected (0.02 sec)
Records: 4  Duplicates: 0  Warnings: 0

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected, 1 warning (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)


```

## 从库开启复制

```sql


mysql> change master to master_host='192.168.2.61',master_port=3306,master_user='dao',master_password='dao',master_auto_position=1 ;
Query OK, 0 rows affected, 2 warnings (0.06 sec)

mysql> start slave ;
Query OK, 0 rows affected (0.00 sec)

```

## 查看同步状态


```sql

mysql> show slave status \G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.2.61
                  Master_User: dao
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000005
          Read_Master_Log_Pos: 1467
               Relay_Log_File: node2-relay-bin.000002
                Relay_Log_Pos: 922
        Relay_Master_Log_File: mysql-bin.000005
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 1467
              Relay_Log_Space: 1129
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 0
               Last_SQL_Error:
  Replicate_Ignore_Server_Ids:
             Master_Server_Id: 1
                  Master_UUID: 03fde241-9a55-11e8-aa12-000c291f013a
             Master_Info_File: /mysql1/data/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind:
      Last_IO_Error_Timestamp:
     Last_SQL_Error_Timestamp:
               Master_SSL_Crl:
           Master_SSL_Crlpath:
           Retrieved_Gtid_Set: 03fde241-9a55-11e8-aa12-000c291f013a:4-5
            Executed_Gtid_Set: 03fde241-9a55-11e8-aa12-000c291f013a:1-5
                Auto_Position: 1
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
1 row in set (0.01 sec)

```
