---
layout: post
title: MySQL Binlog概念与实战
date: 2018-07-25
categories: blog
tags: [MySQL]
description: MySQL Binlog概念与实战
---

## MySQL Binlog的作用

MySQL binlog起始是binary log 的缩写,翻译成中文就是MySQL二进制日志,以二进制形式记录了对于数据库潜在的变更的操作.

binlog的应用主要存在于以下两个方面

- 复制
- 备份恢复


## binlog相关参数的说明

参数名| 作用
--- | ----
log_bin binlog    | 存放路径与binlog文件前缀
log_bin_index     | binlog索引文件路径与文件名
binlog_cache_size |单个线程binlog缓存大小
max_binLog_cache_size | 所有线程binlog cache加起来最大内存值 超出则报错
sql_log_bin       | 开启/关闭当前binlog
sync_binlog       | binlog 落盘策略 详见下表
binlog_format     | binlog 记录格式详见下表


- sync-binlog

这个参数的值不同,含义也会不同,该参数混合了时间与次数.
个人认为:这并不是一种好的参数定义方式

参数值 | 含义
--- |  ---
0   | 由操作系统控制binlog落盘
1   | 每次提交都落盘binlog
2   | 每两次提交落盘一次binlog
100 | 每100次提交落盘一次binlog

- binlog_format

参数值   | 含义
---      |  ---
row      | 记录每一行的变化
statment | 记录变更数据的语句
minxed   | 对于一般操作记录statment格式,特殊操作自动切换到row格式

特殊操作列表

- 使用的函数包括了UUID
- 表中有自增列并且触发器或函数被调用.
- 视图中使用基于行格式的复制
- 调用了UDF(user-defined function)
- FOUND_ROWS() 或者 ROW_COUNT()被使用
- USER(), CURRENT_USER(),CURRENT_USER被使用
- 当语句引用了一个或多个系统变量

但下列变量在会话级别使用时,不引起记录格式转变

1. auto_increment_increment
1. auto_increment_offset
1. character_set_client
1. character_set_connection
1. character_set_database
1. character_set_server
1. collation_connection
1. collation_database
1. collation_server
1. foreign_key_checks
1. identity
1. last_insert_id
1. lc_time_names
1. pseudo_thread_id
1. sql_auto_is_null
1. time_zone
1. timestamp
1. unique_checks

## binlog何时生成新的文件

- 重启数据库
- flush log
- 超过max_binlog_size中的值 这里有一点需要注意,binlog的切换一定是事务完成之后.
- 
## binlog的查看 

### 查看当前有哪些binlog

```sql

mysql> show binary logs;
+------------------+-----------+
| Log_name         | File_size |
+------------------+-----------+
| mysql-bin.000001 |       753 |
| mysql-bin.000002 |       154 |
+------------------+-----------+

```

### show binlog events 

```sql
SHOW BINLOG EVENTS
   [IN 'log_name']             --在指定的binlog文件中查看
   [FROM pos]                  --从固定位置查看
   [LIMIT [offset,] row_count] --跳过多少行,查看多少行
```

#### 查看binlog event
```sql

mysql> show binlog events ;
+------------------+-----+----------------+-----------+-------------+---------------------------------------+
| Log_name         | Pos | Event_type     | Server_id | End_log_pos | Info                                  |
+------------------+-----+----------------+-----------+-------------+---------------------------------------+
| mysql-bin.000001 |   4 | Format_desc    |         1 |         123 | Server ver: 5.7.21-log, Binlog ver: 4 |
| mysql-bin.000001 | 123 | Previous_gtids |         1 |         154 |                                       |
| mysql-bin.000001 | 154 | Anonymous_Gtid |         1 |         219 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'  |
| mysql-bin.000001 | 219 | Query          |         1 |         290 | BEGIN                                 |
| mysql-bin.000001 | 290 | Table_map      |         1 |         355 | table_id: 116 (dao.test_binlog)       |
| mysql-bin.000001 | 355 | Update_rows    |         1 |         675 | table_id: 116 flags: STMT_END_F       |
| mysql-bin.000001 | 675 | Xid            |         1 |         706 | COMMIT /* xid=134 */                  |
| mysql-bin.000001 | 706 | Rotate         |         1 |         753 | mysql-bin.000002;pos=4                |
+------------------+-----+----------------+-----------+-------------+---------------------------------------+

```
#### 查看指定binlog文件中的事件

```sql
mysql> show binlog events in  'mysql-bin.000002';
+------------------+-----+----------------+-----------+-------------+---------------------------------------+
| Log_name         | Pos | Event_type     | Server_id | End_log_pos | Info                                  |
+------------------+-----+----------------+-----------+-------------+---------------------------------------+
| mysql-bin.000002 |   4 | Format_desc    |         1 |         123 | Server ver: 5.7.21-log, Binlog ver: 4 |
| mysql-bin.000002 | 123 | Previous_gtids |         1 |         154 |                                       |
+------------------+-----+----------------+-----------+-------------+---------------------------------------+


```

#### 从指定文件指定位置查看事件


```sql


mysql>  show binlog events in  'mysql-bin.000001' from 123 ;
+------------------+-----+----------------+-----------+-------------+--------------------------------------+
| Log_name         | Pos | Event_type     | Server_id | End_log_pos | Info                                 |
+------------------+-----+----------------+-----------+-------------+--------------------------------------+
| mysql-bin.000001 | 123 | Previous_gtids |         1 |         154 |                                      |
| mysql-bin.000001 | 154 | Anonymous_Gtid |         1 |         219 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS' |
| mysql-bin.000001 | 219 | Query          |         1 |         290 | BEGIN                                |
| mysql-bin.000001 | 290 | Table_map      |         1 |         355 | table_id: 116 (dao.test_binlog)      |
| mysql-bin.000001 | 355 | Update_rows    |         1 |         675 | table_id: 116 flags: STMT_END_F      |
| mysql-bin.000001 | 675 | Xid            |         1 |         706 | COMMIT /* xid=134 */                 |
| mysql-bin.000001 | 706 | Rotate         |         1 |         753 | mysql-bin.000002;pos=4               |
+------------------+-----+----------------+-----------+-------------+--------------------------------------+



```
#### 从指定文件指定位置查看事件并限制返回事件数


在文件mysql-bin.000001中查找事件
从123个位置开始查找,跳过找到的前两行后查找三行



```sql


mysql>  show binlog events in  'mysql-bin.000001' from 123 limit 2 ,3 ;
+------------------+-----+-------------+-----------+-------------+---------------------------------+
| Log_name         | Pos | Event_type  | Server_id | End_log_pos | Info                            |
+------------------+-----+-------------+-----------+-------------+---------------------------------+
| mysql-bin.000001 | 219 | Query       |         1 |         290 | BEGIN                           |
| mysql-bin.000001 | 290 | Table_map   |         1 |         355 | table_id: 116 (dao.test_binlog) |
| mysql-bin.000001 | 355 | Update_rows |         1 |         675 | table_id: 116 flags: STMT_END_F |
+------------------+-----+-------------+-----------+-------------+---------------------------------+
3 rows in set (0.00 sec)


```



### mysqlbinlog

mysqlbinlog 命令可以解析binlog,并将解析后的binlog定位到一个文本文件.

#### mysqlbinlog常用参数

参数 | 含义 
--- | ----
--skip-gtids            | 不打印gtid
--verbose, -v           | 以注释的SQL语句形式显示行事件
-vv --verbose --verbose | 注释的SQL语句形式显示行事件并显示数据类型
--start-date            | 起始日期
--stop-datetime         | 结束日期
--start-position        | 起始位置
--stop-position         | 结束位置


[mysqlbinlog参数全解](https://dev.mysql.com/doc/refman/5.7/en/mysqlbinlog.html)


#### 建立测试数据

```sql


mysql> use dao ;
Database changed
mysql>  create table test_binlog as select * from dept where 1=2 ;
Query OK, 0 rows affected (0.04 sec)
Records: 0  Duplicates: 0  Warnings: 0


mysql> insert into  test_binlog values(1,'111',11);
Query OK, 1 row affected (0.00 sec)

mysql> insert into  test_binlog values(2,'222',22);
Query OK, 1 row affected (0.01 sec)

mysql> insert into  test_binlog values(3,'333',33);
Query OK, 1 row affected (0.01 sec)

mysql> insert into  test_binlog values(4,'444',44);
Query OK, 1 row affected (0.00 sec)

mysql> insert into  test_binlog values(5,'555',55);
Query OK, 1 row affected (0.01 sec)

mysql> insert into  test_binlog values(6,'666',66);
Query OK, 1 row affected (0.00 sec)

mysql> insert into  test_binlog values(7,'777',77);
Query OK, 1 row affected (0.01 sec)

mysql> insert into  test_binlog values(8,'888',88);
Query OK, 1 row affected (0.00 sec)

mysql> insert into  test_binlog values(9,'999',999);
Query OK, 1 row affected (0.02 sec)

mysql> update test_binlog set dname='666666' where deptno in (6,7);
Query OK, 2 rows affected (0.01 sec)
Rows matched: 2  Changed: 2  Warnings: 0

mysql> delete from test_binlog where deptno>=7 ;
Query OK, 3 rows affected (0.02 sec)


```

### 无参数解析

```sql

[mysql@bosenrui data]$ cat a.txt
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=1*/;
/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;
DELIMITER /*!*/;
# at 4
#180725  6:54:36 server id 1  end_log_pos 123 CRC32 0xe4b827c4  Start: binlog v 4, server v 5.7.21-log created 180725  6:54:36 at startup
# Warning: this binlog is either in use or was not closed properly.
ROLLBACK/*!*/;
BINLOG '
LK5XWw8BAAAAdwAAAHsAAAABAAQANS43LjIxLWxvZwAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAAAsrldbEzgNAAgAEgAEBAQEEgAAXwAEGggAAAAICAgCAAAACgoKKioAEjQA
AcQnuOQ=
'/*!*/;
# at 123
#180725  6:54:36 server id 1  end_log_pos 154 CRC32 0x267ec84d  Previous-GTIDs
# [empty]
# at 154
#180725  7:00:34 server id 1  end_log_pos 219 CRC32 0xad5218df  Anonymous_GTID  last_committed=0        sequence_number=1       rbr_only=no
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 219
#180725  7:00:34 server id 1  end_log_pos 341 CRC32 0xe078aa4b  Query   thread_id=4     exec_time=0     error_code=0
use `dao`/*!*/;
SET TIMESTAMP=1532473234/*!*/;
SET @@session.pseudo_thread_id=4/*!*/;
SET @@session.foreign_key_checks=1, @@session.sql_auto_is_null=0, @@session.unique_checks=1, @@session.autocommit=1/*!*/;
SET @@session.sql_mode=1436549152/*!*/;
SET @@session.auto_increment_increment=1, @@session.auto_increment_offset=1/*!*/;
/*!\C utf8 *//*!*/;
SET @@session.character_set_client=33,@@session.collation_connection=33,@@session.collation_server=8/*!*/;
SET @@session.lc_time_names=0/*!*/;
SET @@session.collation_database=DEFAULT/*!*/;
DROP TABLE `test_binlog` /* generated by server */
/*!*/;
# at 341
#180725  7:00:45 server id 1  end_log_pos 406 CRC32 0x875d390a  Anonymous_GTID  last_committed=1        sequence_number=2       rbr_only=no
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 406
#180725  7:00:45 server id 1  end_log_pos 601 CRC32 0x52d3fb61  Query   thread_id=4     exec_time=0     error_code=0
SET TIMESTAMP=1532473245/*!*/;
CREATE TABLE `test_binlog` (
  `deptNO` int(2) NOT NULL,
  `DNAME` varchar(14) DEFAULT NULL,
  `LOC` varchar(13) DEFAULT NULL
)
/*!*/;
# at 601
#180725  7:12:48 server id 1  end_log_pos 666 CRC32 0x859128e4  Anonymous_GTID  last_committed=2        sequence_number=3       rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 666
#180725  7:12:48 server id 1  end_log_pos 737 CRC32 0xd8234fbe  Query   thread_id=4     exec_time=0     error_code=0
SET TIMESTAMP=1532473968/*!*/;
BEGIN
/*!*/;
# at 737
#180725  7:12:48 server id 1  end_log_pos 796 CRC32 0xd6566c89  Table_map: `dao`.`test_binlog` mapped to number 130
# at 796
#180725  7:12:48 server id 1  end_log_pos 843 CRC32 0x22fae8f0  Write_rows: table id 130 flags: STMT_END_F

BINLOG '
cLJXWxMBAAAAOwAAABwDAAAAAIIAAAAAAAEAA2RhbwALdGVzdF9iaW5sb2cAAwMPDwQOAA0ABols
VtY=
cLJXWx4BAAAALwAAAEsDAAAAAIIAAAAAAAEAAgAD//gBAAAAAzExMQIxMfDo+iI=
'/*!*/;
# at 843
#180725  7:12:48 server id 1  end_log_pos 874 CRC32 0x1c03e483  Xid = 171
COMMIT/*!*/;
# at 874
#180725  7:12:48 server id 1  end_log_pos 939 CRC32 0xc3f11413  Anonymous_GTID  last_committed=3        sequence_number=4       rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 939
#180725  7:12:48 server id 1  end_log_pos 1010 CRC32 0xbe4f9e45         Query   thread_id=4     exec_time=0     error_code=0
SET TIMESTAMP=1532473968/*!*/;
BEGIN
/*!*/;
# at 1010
#180725  7:12:48 server id 1  end_log_pos 1069 CRC32 0xb7c0aa71         Table_map: `dao`.`test_binlog` mapped to number 130
# at 1069
#180725  7:12:48 server id 1  end_log_pos 1116 CRC32 0xed15b79f         Write_rows: table id 130 flags: STMT_END_F

BINLOG '
cLJXWxMBAAAAOwAAAC0EAAAAAIIAAAAAAAEAA2RhbwALdGVzdF9iaW5sb2cAAwMPDwQOAA0ABnGq
wLc=
cLJXWx4BAAAALwAAAFwEAAAAAIIAAAAAAAEAAgAD//gCAAAAAzIyMgIyMp+3Fe0=
'/*!*/;
# at 1116
#180725  7:12:48 server id 1  end_log_pos 1147 CRC32 0x900836d8         Xid = 172
COMMIT/*!*/;
# at 1147
#180725  7:12:48 server id 1  end_log_pos 1212 CRC32 0xea559dd4         Anonymous_GTID  last_committed=4        sequence_number=5       rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 1212
#180725  7:12:48 server id 1  end_log_pos 1283 CRC32 0xf7198b14         Query   thread_id=4     exec_time=0     error_code=0
SET TIMESTAMP=1532473968/*!*/;
BEGIN
/*!*/;
# at 1283
#180725  7:12:48 server id 1  end_log_pos 1342 CRC32 0x74dad8cd         Table_map: `dao`.`test_binlog` mapped to number 130
# at 1342
#180725  7:12:48 server id 1  end_log_pos 1389 CRC32 0xc2548ad2         Write_rows: table id 130 flags: STMT_END_F

BINLOG '
cLJXWxMBAAAAOwAAAD4FAAAAAIIAAAAAAAEAA2RhbwALdGVzdF9iaW5sb2cAAwMPDwQOAA0ABs3Y
2nQ=
cLJXWx4BAAAALwAAAG0FAAAAAIIAAAAAAAEAAgAD//gDAAAAAzMzMwIzM9KKVMI=
'/*!*/;
# at 1389
#180725  7:12:48 server id 1  end_log_pos 1420 CRC32 0x8de519a4         Xid = 173
COMMIT/*!*/;
# at 1420
#180725  7:12:48 server id 1  end_log_pos 1485 CRC32 0x44fdac90         Anonymous_GTID  last_committed=5        sequence_number=6       rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 1485
#180725  7:12:48 server id 1  end_log_pos 1556 CRC32 0x6a529cb5         Query   thread_id=4     exec_time=0     error_code=0
SET TIMESTAMP=1532473968/*!*/;
BEGIN
/*!*/;
# at 1556
#180725  7:12:48 server id 1  end_log_pos 1615 CRC32 0x4f29f58b         Table_map: `dao`.`test_binlog` mapped to number 130
# at 1615
#180725  7:12:48 server id 1  end_log_pos 1662 CRC32 0x99fd3a52         Write_rows: table id 130 flags: STMT_END_F

BINLOG '
cLJXWxMBAAAAOwAAAE8GAAAAAIIAAAAAAAEAA2RhbwALdGVzdF9iaW5sb2cAAwMPDwQOAA0ABov1
KU8=
cLJXWx4BAAAALwAAAH4GAAAAAIIAAAAAAAEAAgAD//gEAAAAAzQ0NAI0NFI6/Zk=
'/*!*/;
# at 1662
#180725  7:12:48 server id 1  end_log_pos 1693 CRC32 0x
