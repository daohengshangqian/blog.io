---
layout: post
title: MySQL概念与实战
date: 2018-07-25
categories: blog
tags: [MySQL]
description:MySQL Binlog概念与实战
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
#180725  7:12:48 server id 1  end_log_pos 1693 CRC32 0x451daf93         Xid = 174
COMMIT/*!*/;
# at 1693
#180725  7:12:48 server id 1  end_log_pos 1758 CRC32 0x313af175         Anonymous_GTID  last_committed=6        sequence_number=7       rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 1758
#180725  7:12:48 server id 1  end_log_pos 1829 CRC32 0x3bc028e2         Query   thread_id=4     exec_time=0     error_code=0
SET TIMESTAMP=1532473968/*!*/;
BEGIN
/*!*/;
# at 1829
#180725  7:12:48 server id 1  end_log_pos 1888 CRC32 0x59871149         Table_map: `dao`.`test_binlog` mapped to number 130
# at 1888
#180725  7:12:48 server id 1  end_log_pos 1935 CRC32 0x9ffe1dc4         Write_rows: table id 130 flags: STMT_END_F

BINLOG '
cLJXWxMBAAAAOwAAAGAHAAAAAIIAAAAAAAEAA2RhbwALdGVzdF9iaW5sb2cAAwMPDwQOAA0ABkkR
h1k=
cLJXWx4BAAAALwAAAI8HAAAAAIIAAAAAAAEAAgAD//gFAAAAAzU1NQI1NcQd/p8=
'/*!*/;
# at 1935
#180725  7:12:48 server id 1  end_log_pos 1966 CRC32 0x839617e7         Xid = 175
COMMIT/*!*/;
# at 1966
#180725  7:12:48 server id 1  end_log_pos 2031 CRC32 0xa4bfd637         Anonymous_GTID  last_committed=7        sequence_number=8       rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 2031
#180725  7:12:48 server id 1  end_log_pos 2102 CRC32 0xc029d054         Query   thread_id=4     exec_time=0     error_code=0
SET TIMESTAMP=1532473968/*!*/;
BEGIN
/*!*/;
# at 2102
#180725  7:12:48 server id 1  end_log_pos 2161 CRC32 0x33059961         Table_map: `dao`.`test_binlog` mapped to number 130
# at 2161
#180725  7:12:48 server id 1  end_log_pos 2208 CRC32 0x578fdd0e         Write_rows: table id 130 flags: STMT_END_F

BINLOG '
cLJXWxMBAAAAOwAAAHEIAAAAAIIAAAAAAAEAA2RhbwALdGVzdF9iaW5sb2cAAwMPDwQOAA0ABmGZ
BTM=
cLJXWx4BAAAALwAAAKAIAAAAAIIAAAAAAAEAAgAD//gGAAAAAzY2NgI2Ng7dj1c=
'/*!*/;
# at 2208
#180725  7:12:48 server id 1  end_log_pos 2239 CRC32 0xfac03c4f         Xid = 176
COMMIT/*!*/;
# at 2239
#180725  7:12:48 server id 1  end_log_pos 2304 CRC32 0x5f26bacc         Anonymous_GTID  last_committed=8        sequence_number=9       rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 2304
#180725  7:12:48 server id 1  end_log_pos 2375 CRC32 0xf8d10983         Query   thread_id=4     exec_time=0     error_code=0
SET TIMESTAMP=1532473968/*!*/;
BEGIN
/*!*/;
# at 2375
#180725  7:12:48 server id 1  end_log_pos 2434 CRC32 0x0319d7c9         Table_map: `dao`.`test_binlog` mapped to number 130
# at 2434
#180725  7:12:48 server id 1  end_log_pos 2481 CRC32 0x99169847         Write_rows: table id 130 flags: STMT_END_F

BINLOG '
cLJXWxMBAAAAOwAAAIIJAAAAAIIAAAAAAAEAA2RhbwALdGVzdF9iaW5sb2cAAwMPDwQOAA0ABsnX
GQM=
cLJXWx4BAAAALwAAALEJAAAAAIIAAAAAAAEAAgAD//gHAAAAAzc3NwI3N0eYFpk=
'/*!*/;
# at 2481
#180725  7:12:48 server id 1  end_log_pos 2512 CRC32 0x31b5de8b         Xid = 177
COMMIT/*!*/;
# at 2512
#180725  7:12:48 server id 1  end_log_pos 2577 CRC32 0x94071c8f         Anonymous_GTID  last_committed=9        sequence_number=10      rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 2577
#180725  7:12:48 server id 1  end_log_pos 2648 CRC32 0x68b75392         Query   thread_id=4     exec_time=0     error_code=0
SET TIMESTAMP=1532473968/*!*/;
BEGIN
/*!*/;
# at 2648
#180725  7:12:48 server id 1  end_log_pos 2707 CRC32 0x525f8deb         Table_map: `dao`.`test_binlog` mapped to number 130
# at 2707
#180725  7:12:48 server id 1  end_log_pos 2754 CRC32 0x5ef0a4b6         Write_rows: table id 130 flags: STMT_END_F

BINLOG '
cLJXWxMBAAAAOwAAAJMKAAAAAIIAAAAAAAEAA2RhbwALdGVzdF9iaW5sb2cAAwMPDwQOAA0ABuuN
X1I=
cLJXWx4BAAAALwAAAMIKAAAAAIIAAAAAAAEAAgAD//gIAAAAAzg4OAI4OLak8F4=
'/*!*/;
# at 2754
#180725  7:12:48 server id 1  end_log_pos 2785 CRC32 0x34291064         Xid = 178
COMMIT/*!*/;
# at 2785
#180725  7:12:49 server id 1  end_log_pos 2850 CRC32 0x9506981b         Anonymous_GTID  last_committed=10       sequence_number=11      rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 2850
#180725  7:12:49 server id 1  end_log_pos 2921 CRC32 0xb464039d         Query   thread_id=4     exec_time=0     error_code=0
SET TIMESTAMP=1532473969/*!*/;
BEGIN
/*!*/;
# at 2921
#180725  7:12:49 server id 1  end_log_pos 2980 CRC32 0x7c5f87e0         Table_map: `dao`.`test_binlog` mapped to number 130
# at 2980
#180725  7:12:49 server id 1  end_log_pos 3028 CRC32 0x9c40ac2c         Write_rows: table id 130 flags: STMT_END_F

BINLOG '
cbJXWxMBAAAAOwAAAKQLAAAAAIIAAAAAAAEAA2RhbwALdGVzdF9iaW5sb2cAAwMPDwQOAA0ABuCH
X3w=
cbJXWx4BAAAAMAAAANQLAAAAAIIAAAAAAAEAAgAD//gJAAAAAzk5OQM5OTksrECc
'/*!*/;
# at 3028
#180725  7:12:49 server id 1  end_log_pos 3059 CRC32 0xcc452aff         Xid = 179
COMMIT/*!*/;
# at 3059
#180725  7:15:59 server id 1  end_log_pos 3124 CRC32 0x49a835b2         Anonymous_GTID  last_committed=11       sequence_number=12      rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 3124
#180725  7:15:59 server id 1  end_log_pos 3195 CRC32 0x5f94d226         Query   thread_id=4     exec_time=0     error_code=0
SET TIMESTAMP=1532474159/*!*/;
BEGIN
/*!*/;
# at 3195
#180725  7:15:59 server id 1  end_log_pos 3254 CRC32 0xa527572e         Table_map: `dao`.`test_binlog` mapped to number 130
# at 3254
#180725  7:15:59 server id 1  end_log_pos 3344 CRC32 0xbe153182         Update_rows: table id 130 flags: STMT_END_F

BINLOG '
L7NXWxMBAAAAOwAAALYMAAAAAIIAAAAAAAEAA2RhbwALdGVzdF9iaW5sb2cAAwMPDwQOAA0ABi5X
J6U=
L7NXWx8BAAAAWgAAABANAAAAAIIAAAAAAAEAAgAD///4BgAAAAM2NjYCNjb4BgAAAAY2NjY2NjYC
Njb4BwAAAAM3NzcCNzf4BwAAAAY2NjY2NjYCNzeCMRW+
'/*!*/;
# at 3344
#180725  7:15:59 server id 1  end_log_pos 3375 CRC32 0x8df964f3         Xid = 180
COMMIT/*!*/;
# at 3375
#180725  7:16:11 server id 1  end_log_pos 3440 CRC32 0x16938e4c         Anonymous_GTID  last_committed=12       sequence_number=13      rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 3440
#180725  7:16:11 server id 1  end_log_pos 3511 CRC32 0x70fecce1         Query   thread_id=4     exec_time=0     error_code=0
SET TIMESTAMP=1532474171/*!*/;
BEGIN
/*!*/;
# at 3511
#180725  7:16:11 server id 1  end_log_pos 3570 CRC32 0x2d3a8ccf         Table_map: `dao`.`test_binlog` mapped to number 130
# at 3570
#180725  7:16:11 server id 1  end_log_pos 3645 CRC32 0x74c02b61         Delete_rows: table id 130 flags: STMT_END_F

BINLOG '
O7NXWxMBAAAAOwAAAPINAAAAAIIAAAAAAAEAA2RhbwALdGVzdF9iaW5sb2cAAwMPDwQOAA0ABs+M
Oi0=
O7NXWyABAAAASwAAAD0OAAAAAIIAAAAAAAEAAgAD//gHAAAABjY2NjY2NgI3N/gIAAAAAzg4OAI4
OPgJAAAAAzk5OQM5OTlhK8B0
'/*!*/;
# at 3645
#180725  7:16:11 server id 1  end_log_pos 3676 CRC32 0x55a25e2d         Xid = 181
COMMIT/*!*/;
SET @@SESSION.GTID_NEXT= 'AUTOMATIC' /* added by mysqlbinlog */ /*!*/;
DELIMITER ;
# End of log file
/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/;

```

#### 使用-v参数 显示SQL语句

```sql


[mysql@bosenrui data]$ mysqlbinlog mysql-bin.000001 -v  > b.txt
[mysql@bosenrui data]$ cat b.txt
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
### INSERT INTO `dao`.`test_binlog`
### SET
###   @1=1
###   @2='111'
###   @3='11'
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
### INSERT INTO `dao`.`test_binlog`
### SET
###   @1=2
###   @2='222'
###   @3='22'
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
### INSERT INTO `dao`.`test_binlog`
### SET
###   @1=3
###   @2='333'
###   @3='33'
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
### INSERT INTO `dao`.`test_binlog`
### SET
###   @1=4
###   @2='444'
###   @3='44'
# at 1662
#180725  7:12:48 server id 1  end_log_pos 1693 CRC32 0x451daf93         Xid = 174
COMMIT/*!*/;
# at 1693
#180725  7:12:48 server id 1  end_log_pos 1758 CRC32 0x313af175         Anonymous_GTID  last_committed=6        sequence_number=7       rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 1758
#180725  7:12:48 server id 1  end_log_pos 1829 CRC32 0x3bc028e2         Query   thread_id=4     exec_time=0     error_code=0
SET TIMESTAMP=1532473968/*!*/;
BEGIN
/*!*/;
# at 1829
#180725  7:12:48 server id 1  end_log_pos 1888 CRC32 0x59871149         Table_map: `dao`.`test_binlog` mapped to number 130
# at 1888
#180725  7:12:48 server id 1  end_log_pos 1935 CRC32 0x9ffe1dc4         Write_rows: table id 130 flags: STMT_END_F

BINLOG '
cLJXWxMBAAAAOwAAAGAHAAAAAIIAAAAAAAEAA2RhbwALdGVzdF9iaW5sb2cAAwMPDwQOAA0ABkkR
h1k=
cLJXWx4BAAAALwAAAI8HAAAAAIIAAAAAAAEAAgAD//gFAAAAAzU1NQI1NcQd/p8=
'/*!*/;
### INSERT INTO `dao`.`test_binlog`
### SET
###   @1=5
###   @2='555'
###   @3='55'
# at 1935
#180725  7:12:48 server id 1  end_log_pos 1966 CRC32 0x839617e7         Xid = 175
COMMIT/*!*/;
# at 1966
#180725  7:12:48 server id 1  end_log_pos 2031 CRC32 0xa4bfd637         Anonymous_GTID  last_committed=7        sequence_number=8       rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 2031
#180725  7:12:48 server id 1  end_log_pos 2102 CRC32 0xc029d054         Query   thread_id=4     exec_time=0     error_code=0
SET TIMESTAMP=1532473968/*!*/;
BEGIN
/*!*/;
# at 2102
#180725  7:12:48 server id 1  end_log_pos 2161 CRC32 0x33059961         Table_map: `dao`.`test_binlog` mapped to number 130
# at 2161
#180725  7:12:48 server id 1  end_log_pos 2208 CRC32 0x578fdd0e         Write_rows: table id 130 flags: STMT_END_F

BINLOG '
cLJXWxMBAAAAOwAAAHEIAAAAAIIAAAAAAAEAA2RhbwALdGVzdF9iaW5sb2cAAwMPDwQOAA0ABmGZ
BTM=
cLJXWx4BAAAALwAAAKAIAAAAAIIAAAAAAAEAAgAD//gGAAAAAzY2NgI2Ng7dj1c=
'/*!*/;
### INSERT INTO `dao`.`test_binlog`
### SET
###   @1=6
###   @2='666'
###   @3='66'
# at 2208
#180725  7:12:48 server id 1  end_log_pos 2239 CRC32 0xfac03c4f         Xid = 176
COMMIT/*!*/;
# at 2239
#180725  7:12:48 server id 1  end_log_pos 2304 CRC32 0x5f26bacc         Anonymous_GTID  last_committed=8        sequence_number=9       rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 2304
#180725  7:12:48 server id 1  end_log_pos 2375 CRC32 0xf8d10983         Query   thread_id=4     exec_time=0     error_code=0
SET TIMESTAMP=1532473968/*!*/;
BEGIN
/*!*/;
# at 2375
#180725  7:12:48 server id 1  end_log_pos 2434 CRC32 0x0319d7c9         Table_map: `dao`.`test_binlog` mapped to number 130
# at 2434
#180725  7:12:48 server id 1  end_log_pos 2481 CRC32 0x99169847         Write_rows: table id 130 flags: STMT_END_F

BINLOG '
cLJXWxMBAAAAOwAAAIIJAAAAAIIAAAAAAAEAA2RhbwALdGVzdF9iaW5sb2cAAwMPDwQOAA0ABsnX
GQM=
cLJXWx4BAAAALwAAALEJAAAAAIIAAAAAAAEAAgAD//gHAAAAAzc3NwI3N0eYFpk=
'/*!*/;
### INSERT INTO `dao`.`test_binlog`
### SET
###   @1=7
###   @2='777'
###   @3='77'
# at 2481
#180725  7:12:48 server id 1  end_log_pos 2512 CRC32 0x31b5de8b         Xid = 177
COMMIT/*!*/;
# at 2512
#180725  7:12:48 server id 1  end_log_pos 2577 CRC32 0x94071c8f         Anonymous_GTID  last_committed=9        sequence_number=10      rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 2577
#180725  7:12:48 server id 1  end_log_pos 2648 CRC32 0x68b75392         Query   thread_id=4     exec_time=0     error_code=0
SET TIMESTAMP=1532473968/*!*/;
BEGIN
/*!*/;
# at 2648
#180725  7:12:48 server id 1  end_log_pos 2707 CRC32 0x525f8deb         Table_map: `dao`.`test_binlog` mapped to number 130
# at 2707
#180725  7:12:48 server id 1  end_log_pos 2754 CRC32 0x5ef0a4b6         Write_rows: table id 130 flags: STMT_END_F

BINLOG '
cLJXWxMBAAAAOwAAAJMKAAAAAIIAAAAAAAEAA2RhbwALdGVzdF9iaW5sb2cAAwMPDwQOAA0ABuuN
X1I=
cLJXWx4BAAAALwAAAMIKAAAAAIIAAAAAAAEAAgAD//gIAAAAAzg4OAI4OLak8F4=
'/*!*/;
### INSERT INTO `dao`.`test_binlog`
### SET
###   @1=8
###   @2='888'
###   @3='88'
# at 2754
#180725  7:12:48 server id 1  end_log_pos 2785 CRC32 0x34291064         Xid = 178
COMMIT/*!*/;
# at 2785
#180725  7:12:49 server id 1  end_log_pos 2850 CRC32 0x9506981b         Anonymous_GTID  last_committed=10       sequence_number=11      rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 2850
#180725  7:12:49 server id 1  end_log_pos 2921 CRC32 0xb464039d         Query   thread_id=4     exec_time=0     error_code=0
SET TIMESTAMP=1532473969/*!*/;
BEGIN
/*!*/;
# at 2921
#180725  7:12:49 server id 1  end_log_pos 2980 CRC32 0x7c5f87e0         Table_map: `dao`.`test_binlog` mapped to number 130
# at 2980
#180725  7:12:49 server id 1  end_log_pos 3028 CRC32 0x9c40ac2c         Write_rows: table id 130 flags: STMT_END_F

BINLOG '
cbJXWxMBAAAAOwAAAKQLAAAAAIIAAAAAAAEAA2RhbwALdGVzdF9iaW5sb2cAAwMPDwQOAA0ABuCH
X3w=
cbJXWx4BAAAAMAAAANQLAAAAAIIAAAAAAAEAAgAD//gJAAAAAzk5OQM5OTksrECc
'/*!*/;
### INSERT INTO `dao`.`test_binlog`
### SET
###   @1=9
###   @2='999'
###   @3='999'
# at 3028
#180725  7:12:49 server id 1  end_log_pos 3059 CRC32 0xcc452aff         Xid = 179
COMMIT/*!*/;
# at 3059
#180725  7:15:59 server id 1  end_log_pos 3124 CRC32 0x49a835b2         Anonymous_GTID  last_committed=11       sequence_number=12      rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 3124
#180725  7:15:59 server id 1  end_log_pos 3195 CRC32 0x5f94d226         Query   thread_id=4     exec_time=0     error_code=0
SET TIMESTAMP=1532474159/*!*/;
BEGIN
/*!*/;
# at 3195
#180725  7:15:59 server id 1  end_log_pos 3254 CRC32 0xa527572e         Table_map: `dao`.`test_binlog` mapped to number 130
# at 3254
#180725  7:15:59 server id 1  end_log_pos 3344 CRC32 0xbe153182         Update_rows: table id 130 flags: STMT_END_F

BINLOG '
L7NXWxMBAAAAOwAAALYMAAAAAIIAAAAAAAEAA2RhbwALdGVzdF9iaW5sb2cAAwMPDwQOAA0ABi5X
J6U=
L7NXWx8BAAAAWgAAABANAAAAAIIAAAAAAAEAAgAD///4BgAAAAM2NjYCNjb4BgAAAAY2NjY2NjYC
Njb4BwAAAAM3NzcCNzf4BwAAAAY2NjY2NjYCNzeCMRW+
'/*!*/;
### UPDATE `dao`.`test_binlog`
### WHERE
###   @1=6
###   @2='666'
###   @3='66'
### SET
###   @1=6
###   @2='666666'
###   @3='66'
### UPDATE `dao`.`test_binlog`
### WHERE
###   @1=7
###   @2='777'
###   @3='77'
### SET
###   @1=7
###   @2='666666'
###   @3='77'
# at 3344
#180725  7:15:59 server id 1  end_log_pos 3375 CRC32 0x8df964f3         Xid = 180
COMMIT/*!*/;
# at 3375
#180725  7:16:11 server id 1  end_log_pos 3440 CRC32 0x16938e4c         Anonymous_GTID  last_committed=12       sequence_number=13      rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 3440
#180725  7:16:11 server id 1  end_log_pos 3511 CRC32 0x70fecce1         Query   thread_id=4     exec_time=0     error_code=0
SET TIMESTAMP=1532474171/*!*/;
BEGIN
/*!*/;
# at 3511
#180725  7:16:11 server id 1  end_log_pos 3570 CRC32 0x2d3a8ccf         Table_map: `dao`.`test_binlog` mapped to number 130
# at 3570
#180725  7:16:11 server id 1  end_log_pos 3645 CRC32 0x74c02b61         Delete_rows: table id 130 flags: STMT_END_F

BINLOG '
O7NXWxMBAAAAOwAAAPINAAAAAIIAAAAAAAEAA2RhbwALdGVzdF9iaW5sb2cAAwMPDwQOAA0ABs+M
Oi0=
O7NXWyABAAAASwAAAD0OAAAAAIIAAAAAAAEAAgAD//gHAAAABjY2NjY2NgI3N/gIAAAAAzg4OAI4
OPgJAAAAAzk5OQM5OTlhK8B0
'/*!*/;
### DELETE FROM `dao`.`test_binlog`
### WHERE
###   @1=7
###   @2='666666'
###   @3='77'
### DELETE FROM `dao`.`test_binlog`
### WHERE
###   @1=8
###   @2='888'
###   @3='88'
### DELETE FROM `dao`.`test_binlog`
### WHERE
###   @1=9
###   @2='999'
###   @3='999'
# at 3645
#180725  7:16:11 server id 1  end_log_pos 3676 CRC32 0x55a25e2d         Xid = 181
COMMIT/*!*/;
SET @@SESSION.GTID_NEXT= 'AUTOMATIC' /* added by mysqlbinlog */ /*!*/;
DELIMITER ;
# End of log file
/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/;

```


### 使用-VV参数显示SQL语句与列的数据类型

```sql
[mysql@bosenrui data]$ mysqlbinlog mysql-bin.000001 -vv  > c.txt
[mysql@bosenrui data]$ cat c.txt
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
### INSERT INTO `dao`.`test_binlog`
### SET
###   @1=1 /* INT meta=0 nullable=0 is_null=0 */
###   @2='111' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='11' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
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
### INSERT INTO `dao`.`test_binlog`
### SET
###   @1=2 /* INT meta=0 nullable=0 is_null=0 */
###   @2='222' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='22' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
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
### INSERT INTO `dao`.`test_binlog`
### SET
###   @1=3 /* INT meta=0 nullable=0 is_null=0 */
###   @2='333' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='33' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
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
### INSERT INTO `dao`.`test_binlog`
### SET
###   @1=4 /* INT meta=0 nullable=0 is_null=0 */
###   @2='444' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='44' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
# at 1662
#180725  7:12:48 server id 1  end_log_pos 1693 CRC32 0x451daf93         Xid = 174
COMMIT/*!*/;
# at 1693
#180725  7:12:48 server id 1  end_log_pos 1758 CRC32 0x313af175         Anonymous_GTID  last_committed=6        sequence_number=7       rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 1758
#180725  7:12:48 server id 1  end_log_pos 1829 CRC32 0x3bc028e2         Query   thread_id=4     exec_time=0     error_code=0
SET TIMESTAMP=1532473968/*!*/;
BEGIN
/*!*/;
# at 1829
#180725  7:12:48 server id 1  end_log_pos 1888 CRC32 0x59871149         Table_map: `dao`.`test_binlog` mapped to number 130
# at 1888
#180725  7:12:48 server id 1  end_log_pos 1935 CRC32 0x9ffe1dc4         Write_rows: table id 130 flags: STMT_END_F

BINLOG '
cLJXWxMBAAAAOwAAAGAHAAAAAIIAAAAAAAEAA2RhbwALdGVzdF9iaW5sb2cAAwMPDwQOAA0ABkkR
h1k=
cLJXWx4BAAAALwAAAI8HAAAAAIIAAAAAAAEAAgAD//gFAAAAAzU1NQI1NcQd/p8=
'/*!*/;
### INSERT INTO `dao`.`test_binlog`
### SET
###   @1=5 /* INT meta=0 nullable=0 is_null=0 */
###   @2='555' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='55' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
# at 1935
#180725  7:12:48 server id 1  end_log_pos 1966 CRC32 0x839617e7         Xid = 175
COMMIT/*!*/;
# at 1966
#180725  7:12:48 server id 1  end_log_pos 2031 CRC32 0xa4bfd637         Anonymous_GTID  last_committed=7        sequence_number=8       rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 2031
#180725  7:12:48 server id 1  end_log_pos 2102 CRC32 0xc029d054         Query   thread_id=4     exec_time=0     error_code=0
SET TIMESTAMP=1532473968/*!*/;
BEGIN
/*!*/;
# at 2102
#180725  7:12:48 server id 1  end_log_pos 2161 CRC32 0x33059961         Table_map: `dao`.`test_binlog` mapped to number 130
# at 2161
#180725  7:12:48 server id 1  end_log_pos 2208 CRC32 0x578fdd0e         Write_rows: table id 130 flags: STMT_END_F

BINLOG '
cLJXWxMBAAAAOwAAAHEIAAAAAIIAAAAAAAEAA2RhbwALdGVzdF9iaW5sb2cAAwMPDwQOAA0ABmGZ
BTM=
cLJXWx4BAAAALwAAAKAIAAAAAIIAAAAAAAEAAgAD//gGAAAAAzY2NgI2Ng7dj1c=
'/*!*/;
### INSERT INTO `dao`.`test_binlog`
### SET
###   @1=6 /* INT meta=0 nullable=0 is_null=0 */
###   @2='666' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='66' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
# at 2208
#180725  7:12:48 server id 1  end_log_pos 2239 CRC32 0xfac03c4f         Xid = 176
COMMIT/*!*/;
# at 2239
#180725  7:12:48 server id 1  end_log_pos 2304 CRC32 0x5f26bacc         Anonymous_GTID  last_committed=8        sequence_number=9       rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 2304
#180725  7:12:48 server id 1  end_log_pos 2375 CRC32 0xf8d10983         Query   thread_id=4     exec_time=0     error_code=0
SET TIMESTAMP=1532473968/*!*/;
BEGIN
/*!*/;
# at 2375
#180725  7:12:48 server id 1  end_log_pos 2434 CRC32 0x0319d7c9         Table_map: `dao`.`test_binlog` mapped to number 130
# at 2434
#180725  7:12:48 server id 1  end_log_pos 2481 CRC32 0x99169847         Write_rows: table id 130 flags: STMT_END_F

BINLOG '
cLJXWxMBAAAAOwAAAIIJAAAAAIIAAAAAAAEAA2RhbwALdGVzdF9iaW5sb2cAAwMPDwQOAA0ABsnX
GQM=
cLJXWx4BAAAALwAAALEJAAAAAIIAAAAAAAEAAgAD//gHAAAAAzc3NwI3N0eYFpk=
'/*!*/;
### INSERT INTO `dao`.`test_binlog`
### SET
###   @1=7 /* INT meta=0 nullable=0 is_null=0 */
###   @2='777' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='77' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
# at 2481
#180725  7:12:48 server id 1  end_log_pos 2512 CRC32 0x31b5de8b         Xid = 177
COMMIT/*!*/;
# at 2512
#180725  7:12:48 server id 1  end_log_pos 2577 CRC32 0x94071c8f         Anonymous_GTID  last_committed=9        sequence_number=10      rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 2577
#180725  7:12:48 server id 1  end_log_pos 2648 CRC32 0x68b75392         Query   thread_id=4     exec_time=0     error_code=0
SET TIMESTAMP=1532473968/*!*/;
BEGIN
/*!*/;
# at 2648
#180725  7:12:48 server id 1  end_log_pos 2707 CRC32 0x525f8deb         Table_map: `dao`.`test_binlog` mapped to number 130
# at 2707
#180725  7:12:48 server id 1  end_log_pos 2754 CRC32 0x5ef0a4b6         Write_rows: table id 130 flags: STMT_END_F

BINLOG '
cLJXWxMBAAAAOwAAAJMKAAAAAIIAAAAAAAEAA2RhbwALdGVzdF9iaW5sb2cAAwMPDwQOAA0ABuuN
X1I=
cLJXWx4BAAAALwAAAMIKAAAAAIIAAAAAAAEAAgAD//gIAAAAAzg4OAI4OLak8F4=
'/*!*/;
### INSERT INTO `dao`.`test_binlog`
### SET
###   @1=8 /* INT meta=0 nullable=0 is_null=0 */
###   @2='888' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='88' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
# at 2754
#180725  7:12:48 server id 1  end_log_pos 2785 CRC32 0x34291064         Xid = 178
COMMIT/*!*/;
# at 2785
#180725  7:12:49 server id 1  end_log_pos 2850 CRC32 0x9506981b         Anonymous_GTID  last_committed=10       sequence_number=11      rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 2850
#180725  7:12:49 server id 1  end_log_pos 2921 CRC32 0xb464039d         Query   thread_id=4     exec_time=0     error_code=0
SET TIMESTAMP=1532473969/*!*/;
BEGIN
/*!*/;
# at 2921
#180725  7:12:49 server id 1  end_log_pos 2980 CRC32 0x7c5f87e0         Table_map: `dao`.`test_binlog` mapped to number 130
# at 2980
#180725  7:12:49 server id 1  end_log_pos 3028 CRC32 0x9c40ac2c         Write_rows: table id 130 flags: STMT_END_F

BINLOG '
cbJXWxMBAAAAOwAAAKQLAAAAAIIAAAAAAAEAA2RhbwALdGVzdF9iaW5sb2cAAwMPDwQOAA0ABuCH
X3w=
cbJXWx4BAAAAMAAAANQLAAAAAIIAAAAAAAEAAgAD//gJAAAAAzk5OQM5OTksrECc
'/*!*/;
### INSERT INTO `dao`.`test_binlog`
### SET
###   @1=9 /* INT meta=0 nullable=0 is_null=0 */
###   @2='999' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='999' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
# at 3028
#180725  7:12:49 server id 1  end_log_pos 3059 CRC32 0xcc452aff         Xid = 179
COMMIT/*!*/;
# at 3059
#180725  7:15:59 server id 1  end_log_pos 3124 CRC32 0x49a835b2         Anonymous_GTID  last_committed=11       sequence_number=12      rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 3124
#180725  7:15:59 server id 1  end_log_pos 3195 CRC32 0x5f94d226         Query   thread_id=4     exec_time=0     error_code=0
SET TIMESTAMP=1532474159/*!*/;
BEGIN
/*!*/;
# at 3195
#180725  7:15:59 server id 1  end_log_pos 3254 CRC32 0xa527572e         Table_map: `dao`.`test_binlog` mapped to number 130
# at 3254
#180725  7:15:59 server id 1  end_log_pos 3344 CRC32 0xbe153182         Update_rows: table id 130 flags: STMT_END_F

BINLOG '
L7NXWxMBAAAAOwAAALYMAAAAAIIAAAAAAAEAA2RhbwALdGVzdF9iaW5sb2cAAwMPDwQOAA0ABi5X
J6U=
L7NXWx8BAAAAWgAAABANAAAAAIIAAAAAAAEAAgAD///4BgAAAAM2NjYCNjb4BgAAAAY2NjY2NjYC
Njb4BwAAAAM3NzcCNzf4BwAAAAY2NjY2NjYCNzeCMRW+
'/*!*/;
### UPDATE `dao`.`test_binlog`
### WHERE
###   @1=6 /* INT meta=0 nullable=0 is_null=0 */
###   @2='666' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='66' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
### SET
###   @1=6 /* INT meta=0 nullable=0 is_null=0 */
###   @2='666666' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='66' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
### UPDATE `dao`.`test_binlog`
### WHERE
###   @1=7 /* INT meta=0 nullable=0 is_null=0 */
###   @2='777' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='77' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
### SET
###   @1=7 /* INT meta=0 nullable=0 is_null=0 */
###   @2='666666' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='77' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
# at 3344
#180725  7:15:59 server id 1  end_log_pos 3375 CRC32 0x8df964f3         Xid = 180
COMMIT/*!*/;
# at 3375
#180725  7:16:11 server id 1  end_log_pos 3440 CRC32 0x16938e4c         Anonymous_GTID  last_committed=12       sequence_number=13      rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 3440
#180725  7:16:11 server id 1  end_log_pos 3511 CRC32 0x70fecce1         Query   thread_id=4     exec_time=0     error_code=0
SET TIMESTAMP=1532474171/*!*/;
BEGIN
/*!*/;
# at 3511
#180725  7:16:11 server id 1  end_log_pos 3570 CRC32 0x2d3a8ccf         Table_map: `dao`.`test_binlog` mapped to number 130
# at 3570
#180725  7:16:11 server id 1  end_log_pos 3645 CRC32 0x74c02b61         Delete_rows: table id 130 flags: STMT_END_F

BINLOG '
O7NXWxMBAAAAOwAAAPINAAAAAIIAAAAAAAEAA2RhbwALdGVzdF9iaW5sb2cAAwMPDwQOAA0ABs+M
Oi0=
O7NXWyABAAAASwAAAD0OAAAAAIIAAAAAAAEAAgAD//gHAAAABjY2NjY2NgI3N/gIAAAAAzg4OAI4
OPgJAAAAAzk5OQM5OTlhK8B0
'/*!*/;
### DELETE FROM `dao`.`test_binlog`
### WHERE
###   @1=7 /* INT meta=0 nullable=0 is_null=0 */
###   @2='666666' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='77' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
### DELETE FROM `dao`.`test_binlog`
### WHERE
###   @1=8 /* INT meta=0 nullable=0 is_null=0 */
###   @2='888' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='88' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
### DELETE FROM `dao`.`test_binlog`
### WHERE
###   @1=9 /* INT meta=0 nullable=0 is_null=0 */
###   @2='999' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='999' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
# at 3645
#180725  7:16:11 server id 1  end_log_pos 3676 CRC32 0x55a25e2d         Xid = 181
COMMIT/*!*/;
SET @@SESSION.GTID_NEXT= 'AUTOMATIC' /* added by mysqlbinlog */ /*!*/;
DELIMITER ;
# End of log file
/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/;



```


### 指定起止位置解析binlog

解析后只有update 语句和delete 语句 
```sql

[mysql@bosenrui data]$ mysqlbinlog mysql-bin.000001 --start-position=3254 --stop-position=3645   > d.txt
[mysql@bosenrui data]$ cat d.txt
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
# at 3254
#180725  7:15:59 server id 1  end_log_pos 3344 CRC32 0xbe153182         Update_rows: table id 130 flags: STMT_END_F

BINLOG '
L7NXWx8BAAAAWgAAABANAAAAAIIAAAAAAAEAAgAD///4BgAAAAM2NjYCNjb4BgAAAAY2NjY2NjYC
Njb4BwAAAAM3NzcCNzf4BwAAAAY2NjY2NjYCNzeCMRW+
'/*!*/;
# at 3344
#180725  7:15:59 server id 1  end_log_pos 3375 CRC32 0x8df964f3         Xid = 180
COMMIT/*!*/;
# at 3375
#180725  7:16:11 server id 1  end_log_pos 3440 CRC32 0x16938e4c         Anonymous_GTID  last_committed=12       sequence_number=13      rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 3440
#180725  7:16:11 server id 1  end_log_pos 3511 CRC32 0x70fecce1         Query   thread_id=4     exec_time=0     error_code=0
SET TIMESTAMP=1532474171/*!*/;
SET @@session.pseudo_thread_id=4/*!*/;
SET @@session.foreign_key_checks=1, @@session.sql_auto_is_null=0, @@session.unique_checks=1, @@session.autocommit=1/*!*/;
SET @@session.sql_mode=1436549152/*!*/;
SET @@session.auto_increment_increment=1, @@session.auto_increment_offset=1/*!*/;
/*!\C utf8 *//*!*/;
SET @@session.character_set_client=33,@@session.collation_connection=33,@@session.collation_server=8/*!*/;
SET @@session.lc_time_names=0/*!*/;
SET @@session.collation_database=DEFAULT/*!*/;
BEGIN
/*!*/;
# at 3511
#180725  7:16:11 server id 1  end_log_pos 3570 CRC32 0x2d3a8ccf         Table_map: `dao`.`test_binlog` mapped to number 130
# at 3570
#180725  7:16:11 server id 1  end_log_pos 3645 CRC32 0x74c02b61         Delete_rows: table id 130 flags: STMT_END_F

BINLOG '
O7NXWxMBAAAAOwAAAPINAAAAAIIAAAAAAAEAA2RhbwALdGVzdF9iaW5sb2cAAwMPDwQOAA0ABs+M
Oi0=
O7NXWyABAAAASwAAAD0OAAAAAIIAAAAAAAEAAgAD//gHAAAABjY2NjY2NgI3N/gIAAAAAzg4OAI4
OPgJAAAAAzk5OQM5OTlhK8B0
'/*!*/;
ROLLBACK /* added by mysqlbinlog */ /*!*/;
SET @@SESSION.GTID_NEXT= 'AUTOMATIC' /* added by mysqlbinlog */ /*!*/;
DELIMITER ;
# End of log file
/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/;
[mysql@bosenrui data]$ mysqlbinlog mysql-bin.000001 --start-position=3254 --stop-position=3645 -vv  > d.txt
[mysql@bosenrui data]$ cat d.txt
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
# at 3254
#180725  7:15:59 server id 1  end_log_pos 3344 CRC32 0xbe153182         Update_rows: table id 130 flags: STMT_END_F

BINLOG '
L7NXWx8BAAAAWgAAABANAAAAAIIAAAAAAAEAAgAD///4BgAAAAM2NjYCNjb4BgAAAAY2NjY2NjYC
Njb4BwAAAAM3NzcCNzf4BwAAAAY2NjY2NjYCNzeCMRW+
'/*!*/;
### Row event for unknown table #130# at 3344
#180725  7:15:59 server id 1  end_log_pos 3375 CRC32 0x8df964f3         Xid = 180
COMMIT/*!*/;
# at 3375
#180725  7:16:11 server id 1  end_log_pos 3440 CRC32 0x16938e4c         Anonymous_GTID  last_committed=12       sequence_number=13      rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 3440
#180725  7:16:11 server id 1  end_log_pos 3511 CRC32 0x70fecce1         Query   thread_id=4     exec_time=0     error_code=0
SET TIMESTAMP=1532474171/*!*/;
SET @@session.pseudo_thread_id=4/*!*/;
SET @@session.foreign_key_checks=1, @@session.sql_auto_is_null=0, @@session.unique_checks=1, @@session.autocommit=1/*!*/;
SET @@session.sql_mode=1436549152/*!*/;
SET @@session.auto_increment_increment=1, @@session.auto_increment_offset=1/*!*/;
/*!\C utf8 *//*!*/;
SET @@session.character_set_client=33,@@session.collation_connection=33,@@session.collation_server=8/*!*/;
SET @@session.lc_time_names=0/*!*/;
SET @@session.collation_database=DEFAULT/*!*/;
BEGIN
/*!*/;
# at 3511
#180725  7:16:11 server id 1  end_log_pos 3570 CRC32 0x2d3a8ccf         Table_map: `dao`.`test_binlog` mapped to number 130
# at 3570
#180725  7:16:11 server id 1  end_log_pos 3645 CRC32 0x74c02b61         Delete_rows: table id 130 flags: STMT_END_F

BINLOG '
O7NXWxMBAAAAOwAAAPINAAAAAIIAAAAAAAEAA2RhbwALdGVzdF9iaW5sb2cAAwMPDwQOAA0ABs+M
Oi0=
O7NXWyABAAAASwAAAD0OAAAAAIIAAAAAAAEAAgAD//gHAAAABjY2NjY2NgI3N/gIAAAAAzg4OAI4
OPgJAAAAAzk5OQM5OTlhK8B0
'/*!*/;
### DELETE FROM `dao`.`test_binlog`
### WHERE
###   @1=7 /* INT meta=0 nullable=0 is_null=0 */
###   @2='666666' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='77' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
### DELETE FROM `dao`.`test_binlog`
### WHERE
###   @1=8 /* INT meta=0 nullable=0 is_null=0 */
###   @2='888' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='88' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
### DELETE FROM `dao`.`test_binlog`
### WHERE
###   @1=9 /* INT meta=0 nullable=0 is_null=0 */
###   @2='999' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='999' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
ROLLBACK /* added by mysqlbinlog */ /*!*/;
SET @@SESSION.GTID_NEXT= 'AUTOMATIC' /* added by mysqlbinlog */ /*!*/;
DELIMITER ;
# End of log file
/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/;


```

### 恢复实战

本次我们删除test_binlog表 

#### 删除表

```sql

mysql> drop table test_binlog ;
Query OK, 0 rows affected (0.07 sec)

```

#### 解析binlog并输出到文件

```sql

[mysql@bosenrui data]$ mysqlbinlog mysql-bin.000001  --stop-position=3645 -vv  > e.txt
[mysql@bosenrui data]$ cat e.txt
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
### INSERT INTO `dao`.`test_binlog`
### SET
###   @1=1 /* INT meta=0 nullable=0 is_null=0 */
###   @2='111' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='11' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
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
### INSERT INTO `dao`.`test_binlog`
### SET
###   @1=2 /* INT meta=0 nullable=0 is_null=0 */
###   @2='222' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='22' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
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
### INSERT INTO `dao`.`test_binlog`
### SET
###   @1=3 /* INT meta=0 nullable=0 is_null=0 */
###   @2='333' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='33' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
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
### INSERT INTO `dao`.`test_binlog`
### SET
###   @1=4 /* INT meta=0 nullable=0 is_null=0 */
###   @2='444' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='44' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
# at 1662
#180725  7:12:48 server id 1  end_log_pos 1693 CRC32 0x451daf93         Xid = 174
COMMIT/*!*/;
# at 1693
#180725  7:12:48 server id 1  end_log_pos 1758 CRC32 0x313af175         Anonymous_GTID  last_committed=6        sequence_number=7       rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 1758
#180725  7:12:48 server id 1  end_log_pos 1829 CRC32 0x3bc028e2         Query   thread_id=4     exec_time=0     error_code=0
SET TIMESTAMP=1532473968/*!*/;
BEGIN
/*!*/;
# at 1829
#180725  7:12:48 server id 1  end_log_pos 1888 CRC32 0x59871149         Table_map: `dao`.`test_binlog` mapped to number 130
# at 1888
#180725  7:12:48 server id 1  end_log_pos 1935 CRC32 0x9ffe1dc4         Write_rows: table id 130 flags: STMT_END_F

BINLOG '
cLJXWxMBAAAAOwAAAGAHAAAAAIIAAAAAAAEAA2RhbwALdGVzdF9iaW5sb2cAAwMPDwQOAA0ABkkR
h1k=
cLJXWx4BAAAALwAAAI8HAAAAAIIAAAAAAAEAAgAD//gFAAAAAzU1NQI1NcQd/p8=
'/*!*/;
### INSERT INTO `dao`.`test_binlog`
### SET
###   @1=5 /* INT meta=0 nullable=0 is_null=0 */
###   @2='555' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='55' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
# at 1935
#180725  7:12:48 server id 1  end_log_pos 1966 CRC32 0x839617e7         Xid = 175
COMMIT/*!*/;
# at 1966
#180725  7:12:48 server id 1  end_log_pos 2031 CRC32 0xa4bfd637         Anonymous_GTID  last_committed=7        sequence_number=8       rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 2031
#180725  7:12:48 server id 1  end_log_pos 2102 CRC32 0xc029d054         Query   thread_id=4     exec_time=0     error_code=0
SET TIMESTAMP=1532473968/*!*/;
BEGIN
/*!*/;
# at 2102
#180725  7:12:48 server id 1  end_log_pos 2161 CRC32 0x33059961         Table_map: `dao`.`test_binlog` mapped to number 130
# at 2161
#180725  7:12:48 server id 1  end_log_pos 2208 CRC32 0x578fdd0e         Write_rows: table id 130 flags: STMT_END_F

BINLOG '
cLJXWxMBAAAAOwAAAHEIAAAAAIIAAAAAAAEAA2RhbwALdGVzdF9iaW5sb2cAAwMPDwQOAA0ABmGZ
BTM=
cLJXWx4BAAAALwAAAKAIAAAAAIIAAAAAAAEAAgAD//gGAAAAAzY2NgI2Ng7dj1c=
'/*!*/;
### INSERT INTO `dao`.`test_binlog`
### SET
###   @1=6 /* INT meta=0 nullable=0 is_null=0 */
###   @2='666' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='66' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
# at 2208
#180725  7:12:48 server id 1  end_log_pos 2239 CRC32 0xfac03c4f         Xid = 176
COMMIT/*!*/;
# at 2239
#180725  7:12:48 server id 1  end_log_pos 2304 CRC32 0x5f26bacc         Anonymous_GTID  last_committed=8        sequence_number=9       rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 2304
#180725  7:12:48 server id 1  end_log_pos 2375 CRC32 0xf8d10983         Query   thread_id=4     exec_time=0     error_code=0
SET TIMESTAMP=1532473968/*!*/;
BEGIN
/*!*/;
# at 2375
#180725  7:12:48 server id 1  end_log_pos 2434 CRC32 0x0319d7c9         Table_map: `dao`.`test_binlog` mapped to number 130
# at 2434
#180725  7:12:48 server id 1  end_log_pos 2481 CRC32 0x99169847         Write_rows: table id 130 flags: STMT_END_F

BINLOG '
cLJXWxMBAAAAOwAAAIIJAAAAAIIAAAAAAAEAA2RhbwALdGVzdF9iaW5sb2cAAwMPDwQOAA0ABsnX
GQM=
cLJXWx4BAAAALwAAALEJAAAAAIIAAAAAAAEAAgAD//gHAAAAAzc3NwI3N0eYFpk=
'/*!*/;
### INSERT INTO `dao`.`test_binlog`
### SET
###   @1=7 /* INT meta=0 nullable=0 is_null=0 */
###   @2='777' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='77' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
# at 2481
#180725  7:12:48 server id 1  end_log_pos 2512 CRC32 0x31b5de8b         Xid = 177
COMMIT/*!*/;
# at 2512
#180725  7:12:48 server id 1  end_log_pos 2577 CRC32 0x94071c8f         Anonymous_GTID  last_committed=9        sequence_number=10      rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 2577
#180725  7:12:48 server id 1  end_log_pos 2648 CRC32 0x68b75392         Query   thread_id=4     exec_time=0     error_code=0
SET TIMESTAMP=1532473968/*!*/;
BEGIN
/*!*/;
# at 2648
#180725  7:12:48 server id 1  end_log_pos 2707 CRC32 0x525f8deb         Table_map: `dao`.`test_binlog` mapped to number 130
# at 2707
#180725  7:12:48 server id 1  end_log_pos 2754 CRC32 0x5ef0a4b6         Write_rows: table id 130 flags: STMT_END_F

BINLOG '
cLJXWxMBAAAAOwAAAJMKAAAAAIIAAAAAAAEAA2RhbwALdGVzdF9iaW5sb2cAAwMPDwQOAA0ABuuN
X1I=
cLJXWx4BAAAALwAAAMIKAAAAAIIAAAAAAAEAAgAD//gIAAAAAzg4OAI4OLak8F4=
'/*!*/;
### INSERT INTO `dao`.`test_binlog`
### SET
###   @1=8 /* INT meta=0 nullable=0 is_null=0 */
###   @2='888' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='88' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
# at 2754
#180725  7:12:48 server id 1  end_log_pos 2785 CRC32 0x34291064         Xid = 178
COMMIT/*!*/;
# at 2785
#180725  7:12:49 server id 1  end_log_pos 2850 CRC32 0x9506981b         Anonymous_GTID  last_committed=10       sequence_number=11      rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 2850
#180725  7:12:49 server id 1  end_log_pos 2921 CRC32 0xb464039d         Query   thread_id=4     exec_time=0     error_code=0
SET TIMESTAMP=1532473969/*!*/;
BEGIN
/*!*/;
# at 2921
#180725  7:12:49 server id 1  end_log_pos 2980 CRC32 0x7c5f87e0         Table_map: `dao`.`test_binlog` mapped to number 130
# at 2980
#180725  7:12:49 server id 1  end_log_pos 3028 CRC32 0x9c40ac2c         Write_rows: table id 130 flags: STMT_END_F

BINLOG '
cbJXWxMBAAAAOwAAAKQLAAAAAIIAAAAAAAEAA2RhbwALdGVzdF9iaW5sb2cAAwMPDwQOAA0ABuCH
X3w=
cbJXWx4BAAAAMAAAANQLAAAAAIIAAAAAAAEAAgAD//gJAAAAAzk5OQM5OTksrECc
'/*!*/;
### INSERT INTO `dao`.`test_binlog`
### SET
###   @1=9 /* INT meta=0 nullable=0 is_null=0 */
###   @2='999' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='999' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
# at 3028
#180725  7:12:49 server id 1  end_log_pos 3059 CRC32 0xcc452aff         Xid = 179
COMMIT/*!*/;
# at 3059
#180725  7:15:59 server id 1  end_log_pos 3124 CRC32 0x49a835b2         Anonymous_GTID  last_committed=11       sequence_number=12      rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 3124
#180725  7:15:59 server id 1  end_log_pos 3195 CRC32 0x5f94d226         Query   thread_id=4     exec_time=0     error_code=0
SET TIMESTAMP=1532474159/*!*/;
BEGIN
/*!*/;
# at 3195
#180725  7:15:59 server id 1  end_log_pos 3254 CRC32 0xa527572e         Table_map: `dao`.`test_binlog` mapped to number 130
# at 3254
#180725  7:15:59 server id 1  end_log_pos 3344 CRC32 0xbe153182         Update_rows: table id 130 flags: STMT_END_F

BINLOG '
L7NXWxMBAAAAOwAAALYMAAAAAIIAAAAAAAEAA2RhbwALdGVzdF9iaW5sb2cAAwMPDwQOAA0ABi5X
J6U=
L7NXWx8BAAAAWgAAABANAAAAAIIAAAAAAAEAAgAD///4BgAAAAM2NjYCNjb4BgAAAAY2NjY2NjYC
Njb4BwAAAAM3NzcCNzf4BwAAAAY2NjY2NjYCNzeCMRW+
'/*!*/;
### UPDATE `dao`.`test_binlog`
### WHERE
###   @1=6 /* INT meta=0 nullable=0 is_null=0 */
###   @2='666' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='66' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
### SET
###   @1=6 /* INT meta=0 nullable=0 is_null=0 */
###   @2='666666' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='66' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
### UPDATE `dao`.`test_binlog`
### WHERE
###   @1=7 /* INT meta=0 nullable=0 is_null=0 */
###   @2='777' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='77' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
### SET
###   @1=7 /* INT meta=0 nullable=0 is_null=0 */
###   @2='666666' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='77' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
# at 3344
#180725  7:15:59 server id 1  end_log_pos 3375 CRC32 0x8df964f3         Xid = 180
COMMIT/*!*/;
# at 3375
#180725  7:16:11 server id 1  end_log_pos 3440 CRC32 0x16938e4c         Anonymous_GTID  last_committed=12       sequence_number=13      rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 3440
#180725  7:16:11 server id 1  end_log_pos 3511 CRC32 0x70fecce1         Query   thread_id=4     exec_time=0     error_code=0
SET TIMESTAMP=1532474171/*!*/;
BEGIN
/*!*/;
# at 3511
#180725  7:16:11 server id 1  end_log_pos 3570 CRC32 0x2d3a8ccf         Table_map: `dao`.`test_binlog` mapped to number 130
# at 3570
#180725  7:16:11 server id 1  end_log_pos 3645 CRC32 0x74c02b61         Delete_rows: table id 130 flags: STMT_END_F

BINLOG '
O7NXWxMBAAAAOwAAAPINAAAAAIIAAAAAAAEAA2RhbwALdGVzdF9iaW5sb2cAAwMPDwQOAA0ABs+M
Oi0=
O7NXWyABAAAASwAAAD0OAAAAAIIAAAAAAAEAAgAD//gHAAAABjY2NjY2NgI3N/gIAAAAAzg4OAI4
OPgJAAAAAzk5OQM5OTlhK8B0
'/*!*/;
### DELETE FROM `dao`.`test_binlog`
### WHERE
###   @1=7 /* INT meta=0 nullable=0 is_null=0 */
###   @2='666666' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='77' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
### DELETE FROM `dao`.`test_binlog`
### WHERE
###   @1=8 /* INT meta=0 nullable=0 is_null=0 */
###   @2='888' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='88' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
### DELETE FROM `dao`.`test_binlog`
### WHERE
###   @1=9 /* INT meta=0 nullable=0 is_null=0 */
###   @2='999' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='999' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
ROLLBACK /* added by mysqlbinlog */ /*!*/;
SET @@SESSION.GTID_NEXT= 'AUTOMATIC' /* added by mysqlbinlog */ /*!*/;
DELIMITER ;
# End of log file
/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/;
[mysql@bosenrui data]$ cat e.txt
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
### INSERT INTO `dao`.`test_binlog`
### SET
###   @1=1 /* INT meta=0 nullable=0 is_null=0 */
###   @2='111' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='11' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
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
### INSERT INTO `dao`.`test_binlog`
### SET
###   @1=2 /* INT meta=0 nullable=0 is_null=0 */
###   @2='222' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='22' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
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
### INSERT INTO `dao`.`test_binlog`
### SET
###   @1=3 /* INT meta=0 nullable=0 is_null=0 */
###   @2='333' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='33' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
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
### INSERT INTO `dao`.`test_binlog`
### SET
###   @1=4 /* INT meta=0 nullable=0 is_null=0 */
###   @2='444' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='44' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
# at 1662
#180725  7:12:48 server id 1  end_log_pos 1693 CRC32 0x451daf93         Xid = 174
COMMIT/*!*/;
# at 1693
#180725  7:12:48 server id 1  end_log_pos 1758 CRC32 0x313af175         Anonymous_GTID  last_committed=6        sequence_number=7       rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 1758
#180725  7:12:48 server id 1  end_log_pos 1829 CRC32 0x3bc028e2         Query   thread_id=4     exec_time=0     error_code=0
SET TIMESTAMP=1532473968/*!*/;
BEGIN
/*!*/;
# at 1829
#180725  7:12:48 server id 1  end_log_pos 1888 CRC32 0x59871149         Table_map: `dao`.`test_binlog` mapped to number 130
# at 1888
#180725  7:12:48 server id 1  end_log_pos 1935 CRC32 0x9ffe1dc4         Write_rows: table id 130 flags: STMT_END_F

BINLOG '
cLJXWxMBAAAAOwAAAGAHAAAAAIIAAAAAAAEAA2RhbwALdGVzdF9iaW5sb2cAAwMPDwQOAA0ABkkR
h1k=
cLJXWx4BAAAALwAAAI8HAAAAAIIAAAAAAAEAAgAD//gFAAAAAzU1NQI1NcQd/p8=
'/*!*/;
### INSERT INTO `dao`.`test_binlog`
### SET
###   @1=5 /* INT meta=0 nullable=0 is_null=0 */
###   @2='555' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='55' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
# at 1935
#180725  7:12:48 server id 1  end_log_pos 1966 CRC32 0x839617e7         Xid = 175
COMMIT/*!*/;
# at 1966
#180725  7:12:48 server id 1  end_log_pos 2031 CRC32 0xa4bfd637         Anonymous_GTID  last_committed=7        sequence_number=8       rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 2031
#180725  7:12:48 server id 1  end_log_pos 2102 CRC32 0xc029d054         Query   thread_id=4     exec_time=0     error_code=0
SET TIMESTAMP=1532473968/*!*/;
BEGIN
/*!*/;
# at 2102
#180725  7:12:48 server id 1  end_log_pos 2161 CRC32 0x33059961         Table_map: `dao`.`test_binlog` mapped to number 130
# at 2161
#180725  7:12:48 server id 1  end_log_pos 2208 CRC32 0x578fdd0e         Write_rows: table id 130 flags: STMT_END_F

BINLOG '
cLJXWxMBAAAAOwAAAHEIAAAAAIIAAAAAAAEAA2RhbwALdGVzdF9iaW5sb2cAAwMPDwQOAA0ABmGZ
BTM=
cLJXWx4BAAAALwAAAKAIAAAAAIIAAAAAAAEAAgAD//gGAAAAAzY2NgI2Ng7dj1c=
'/*!*/;
### INSERT INTO `dao`.`test_binlog`
### SET
###   @1=6 /* INT meta=0 nullable=0 is_null=0 */
###   @2='666' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='66' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
# at 2208
#180725  7:12:48 server id 1  end_log_pos 2239 CRC32 0xfac03c4f         Xid = 176
COMMIT/*!*/;
# at 2239
#180725  7:12:48 server id 1  end_log_pos 2304 CRC32 0x5f26bacc         Anonymous_GTID  last_committed=8        sequence_number=9       rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 2304
#180725  7:12:48 server id 1  end_log_pos 2375 CRC32 0xf8d10983         Query   thread_id=4     exec_time=0     error_code=0
SET TIMESTAMP=1532473968/*!*/;
BEGIN
/*!*/;
# at 2375
#180725  7:12:48 server id 1  end_log_pos 2434 CRC32 0x0319d7c9         Table_map: `dao`.`test_binlog` mapped to number 130
# at 2434
#180725  7:12:48 server id 1  end_log_pos 2481 CRC32 0x99169847         Write_rows: table id 130 flags: STMT_END_F

BINLOG '
cLJXWxMBAAAAOwAAAIIJAAAAAIIAAAAAAAEAA2RhbwALdGVzdF9iaW5sb2cAAwMPDwQOAA0ABsnX
GQM=
cLJXWx4BAAAALwAAALEJAAAAAIIAAAAAAAEAAgAD//gHAAAAAzc3NwI3N0eYFpk=
'/*!*/;
### INSERT INTO `dao`.`test_binlog`
### SET
###   @1=7 /* INT meta=0 nullable=0 is_null=0 */
###   @2='777' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='77' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
# at 2481
#180725  7:12:48 server id 1  end_log_pos 2512 CRC32 0x31b5de8b         Xid = 177
COMMIT/*!*/;
# at 2512
#180725  7:12:48 server id 1  end_log_pos 2577 CRC32 0x94071c8f         Anonymous_GTID  last_committed=9        sequence_number=10      rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 2577
#180725  7:12:48 server id 1  end_log_pos 2648 CRC32 0x68b75392         Query   thread_id=4     exec_time=0     error_code=0
SET TIMESTAMP=1532473968/*!*/;
BEGIN
/*!*/;
# at 2648
#180725  7:12:48 server id 1  end_log_pos 2707 CRC32 0x525f8deb         Table_map: `dao`.`test_binlog` mapped to number 130
# at 2707
#180725  7:12:48 server id 1  end_log_pos 2754 CRC32 0x5ef0a4b6         Write_rows: table id 130 flags: STMT_END_F

BINLOG '
cLJXWxMBAAAAOwAAAJMKAAAAAIIAAAAAAAEAA2RhbwALdGVzdF9iaW5sb2cAAwMPDwQOAA0ABuuN
X1I=
cLJXWx4BAAAALwAAAMIKAAAAAIIAAAAAAAEAAgAD//gIAAAAAzg4OAI4OLak8F4=
'/*!*/;
### INSERT INTO `dao`.`test_binlog`
### SET
###   @1=8 /* INT meta=0 nullable=0 is_null=0 */
###   @2='888' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='88' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
# at 2754
#180725  7:12:48 server id 1  end_log_pos 2785 CRC32 0x34291064         Xid = 178
COMMIT/*!*/;
# at 2785
#180725  7:12:49 server id 1  end_log_pos 2850 CRC32 0x9506981b         Anonymous_GTID  last_committed=10       sequence_number=11      rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 2850
#180725  7:12:49 server id 1  end_log_pos 2921 CRC32 0xb464039d         Query   thread_id=4     exec_time=0     error_code=0
SET TIMESTAMP=1532473969/*!*/;
BEGIN
/*!*/;
# at 2921
#180725  7:12:49 server id 1  end_log_pos 2980 CRC32 0x7c5f87e0         Table_map: `dao`.`test_binlog` mapped to number 130
# at 2980
#180725  7:12:49 server id 1  end_log_pos 3028 CRC32 0x9c40ac2c         Write_rows: table id 130 flags: STMT_END_F

BINLOG '
cbJXWxMBAAAAOwAAAKQLAAAAAIIAAAAAAAEAA2RhbwALdGVzdF9iaW5sb2cAAwMPDwQOAA0ABuCH
X3w=
cbJXWx4BAAAAMAAAANQLAAAAAIIAAAAAAAEAAgAD//gJAAAAAzk5OQM5OTksrECc
'/*!*/;
### INSERT INTO `dao`.`test_binlog`
### SET
###   @1=9 /* INT meta=0 nullable=0 is_null=0 */
###   @2='999' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='999' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
# at 3028
#180725  7:12:49 server id 1  end_log_pos 3059 CRC32 0xcc452aff         Xid = 179
COMMIT/*!*/;
# at 3059
#180725  7:15:59 server id 1  end_log_pos 3124 CRC32 0x49a835b2         Anonymous_GTID  last_committed=11       sequence_number=12      rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 3124
#180725  7:15:59 server id 1  end_log_pos 3195 CRC32 0x5f94d226         Query   thread_id=4     exec_time=0     error_code=0
SET TIMESTAMP=1532474159/*!*/;
BEGIN
/*!*/;
# at 3195
#180725  7:15:59 server id 1  end_log_pos 3254 CRC32 0xa527572e         Table_map: `dao`.`test_binlog` mapped to number 130
# at 3254
#180725  7:15:59 server id 1  end_log_pos 3344 CRC32 0xbe153182         Update_rows: table id 130 flags: STMT_END_F

BINLOG '
L7NXWxMBAAAAOwAAALYMAAAAAIIAAAAAAAEAA2RhbwALdGVzdF9iaW5sb2cAAwMPDwQOAA0ABi5X
J6U=
L7NXWx8BAAAAWgAAABANAAAAAIIAAAAAAAEAAgAD///4BgAAAAM2NjYCNjb4BgAAAAY2NjY2NjYC
Njb4BwAAAAM3NzcCNzf4BwAAAAY2NjY2NjYCNzeCMRW+
'/*!*/;
### UPDATE `dao`.`test_binlog`
### WHERE
###   @1=6 /* INT meta=0 nullable=0 is_null=0 */
###   @2='666' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='66' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
### SET
###   @1=6 /* INT meta=0 nullable=0 is_null=0 */
###   @2='666666' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='66' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
### UPDATE `dao`.`test_binlog`
### WHERE
###   @1=7 /* INT meta=0 nullable=0 is_null=0 */
###   @2='777' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='77' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
### SET
###   @1=7 /* INT meta=0 nullable=0 is_null=0 */
###   @2='666666' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='77' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
# at 3344
#180725  7:15:59 server id 1  end_log_pos 3375 CRC32 0x8df964f3         Xid = 180
COMMIT/*!*/;
# at 3375
#180725  7:16:11 server id 1  end_log_pos 3440 CRC32 0x16938e4c         Anonymous_GTID  last_committed=12       sequence_number=13      rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 3440
#180725  7:16:11 server id 1  end_log_pos 3511 CRC32 0x70fecce1         Query   thread_id=4     exec_time=0     error_code=0
SET TIMESTAMP=1532474171/*!*/;
BEGIN
/*!*/;
# at 3511
#180725  7:16:11 server id 1  end_log_pos 3570 CRC32 0x2d3a8ccf         Table_map: `dao`.`test_binlog` mapped to number 130
# at 3570
#180725  7:16:11 server id 1  end_log_pos 3645 CRC32 0x74c02b61         Delete_rows: table id 130 flags: STMT_END_F

BINLOG '
O7NXWxMBAAAAOwAAAPINAAAAAIIAAAAAAAEAA2RhbwALdGVzdF9iaW5sb2cAAwMPDwQOAA0ABs+M
Oi0=
O7NXWyABAAAASwAAAD0OAAAAAIIAAAAAAAEAAgAD//gHAAAABjY2NjY2NgI3N/gIAAAAAzg4OAI4
OPgJAAAAAzk5OQM5OTlhK8B0
'/*!*/;
### DELETE FROM `dao`.`test_binlog`
### WHERE
###   @1=7 /* INT meta=0 nullable=0 is_null=0 */
###   @2='666666' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='77' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
### DELETE FROM `dao`.`test_binlog`
### WHERE
###   @1=8 /* INT meta=0 nullable=0 is_null=0 */
###   @2='888' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='88' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
### DELETE FROM `dao`.`test_binlog`
### WHERE
###   @1=9 /* INT meta=0 nullable=0 is_null=0 */
###   @2='999' /* VARSTRING(14) meta=14 nullable=1 is_null=0 */
###   @3='999' /* VARSTRING(13) meta=13 nullable=1 is_null=0 */
ROLLBACK /* added by mysqlbinlog */ /*!*/;
SET @@SESSION.GTID_NEXT= 'AUTOMATIC' /* added by mysqlbinlog */ /*!*/;
DELIMITER ;
# End of log file
/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/;


```

#### 在binlog中找到建表语句并执行


```sql



mysql> CREATE TABLE `test_binlog` (
    ->   `deptNO` int(2) NOT NULL,
    ->   `DNAME` varchar(14) DEFAULT NULL,
    ->   `LOC` varchar(13) DEFAULT NULL
    -> )
    -> ;
Query OK, 0 rows affected (0.04 sec)


```
#### 使用binlog恢复到update完成,不恢复delete操作

```sql

[mysql@bosenrui data]$  mysql  --defaults-file=/mysql1/mysql1.cnf -uroot -p  < e.txt
Enter password:

```

#### 查看表中的内容


```sql

mysql> CREATE TABLE `test_binlog` (
    ->   `deptNO` int(2) NOT NULL,
    ->   `DNAME` varchar(14) DEFAULT NULL,
    ->   `LOC` varchar(13) DEFAULT NULL
    -> )
    -> ;
Query OK, 0 rows affected (0.04 sec)

mysql> select * from test_binlog ;
+--------+--------+------+
| deptNO | DNAME  | LOC  |
+--------+--------+------+
|      1 | 111    | 11   |
|      2 | 222    | 22   |
|      3 | 333    | 33   |
|      4 | 444    | 44   |
|      5 | 555    | 55   |
|      6 | 666666 | 66   |
|      7 | 666666 | 77   |
|      8 | 888    | 88   |
|      9 | 999    | 999  |
+--------+--------+------+
9 rows in set (0.00 sec)


```
