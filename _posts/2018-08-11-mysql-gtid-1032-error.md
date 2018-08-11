---
layout: post
title: MySQL GTID模式下1032错误详解
date: 2018-08-11
categories: blog
tags: [MySQL]
description: MySQL GTID模式下1032错误详解
---



## 1032 错误


### 错误模拟

- 主库查看数据

```sql

mysql> select * from test.t1 ;
+----+------+
| c1 | c2   |
+----+------+
|  1 |    1 |
|  2 |    2 |
+----+------+
2 rows in set (0.00 sec)

```

- 从库查看数据

```sql

mysql> select * from test.t1 ;
+----+------+
| c1 | c2   |
+----+------+
|  1 |    1 |
|  2 |    2 |
+----+------+
2 rows in set (0.00 sec)

```

- 从库删除数据


```sql

mysql> delete from test.t1 where c1 =1 ;
Query OK, 1 row affected (0.12 sec)

```
- 主库修改数据

```sql


mysql> update test.t1 set c2=11 where c1 =1 ;
Query OK, 1 row affected (0.12 sec)
Rows matched: 1  Changed: 1  Warnings: 0

```

- 从库查看同步状态

```sql



mysql> show slave status \G ;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.2.61
                  Master_User: dao
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000001  ## 根据此信息找到binlog文件并对其进行解析
          Read_Master_Log_Pos: 422
               Relay_Log_File: node2-relay-bin.000002
                Relay_Log_Pos: 367
        Relay_Master_Log_File: mysql-bin.000001  
             Slave_IO_Running: Yes
            Slave_SQL_Running: No
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 1032
                   Last_Error: Could not execute Update_rows event on table test.t1; Can't find record in 't1', Error_code: 1032; handler error HA_ERR_KEY_NOT_FOUND; the event's master log mysql-bin.000001, end_log_pos 391
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 154
              Relay_Log_Space: 842
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: NULL
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 1032
               Last_SQL_Error: Could not execute Update_rows event on table test.t1; Can't find record in 't1', Error_code: 1032; handler error HA_ERR_KEY_NOT_FOUND; the event's master log mysql-bin.000001, end_log_pos 391
  Replicate_Ignore_Server_Ids:
             Master_Server_Id: 1
                  Master_UUID: 03fde241-9a55-11e8-aa12-000c291f013a
             Master_Info_File: /mysql1/data/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State:
           Master_Retry_Count: 86400
                  Master_Bind:
      Last_IO_Error_Timestamp:
     Last_SQL_Error_Timestamp: 180811 17:18:48
               Master_SSL_Crl:
           Master_SSL_Crlpath:
           Retrieved_Gtid_Set: 03fde241-9a55-11e8-aa12-000c291f013a:1
            Executed_Gtid_Set:
                Auto_Position: 1
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
1 row in set (0.00 sec)


```

### 错误解决

#### 手工修复此错误

- 根据报错找到binlog文件并解析内容

```sql

[mysql@node1 ~]$ mysqlbinlog  /mysql1/data/mysql-bin.000001 -vv >1032.sql

```

- 查看binlog内容

```sql

[mysql@node1 ~]$ cat 1032.sql
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=1*/;
/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;
DELIMITER /*!*/;
# at 4
#180811 17:17:08 server id 1  end_log_pos 123 CRC32 0xabf110f8  Start: binlog v 4, server v 5.7.21-log created 180811 17:17:08 at startup
# Warning: this binlog is either in use or was not closed properly.
ROLLBACK/*!*/;
BINLOG '
lKluWw8BAAAAdwAAAHsAAAABAAQANS43LjIxLWxvZwAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAACUqW5bEzgNAAgAEgAEBAQEEgAAXwAEGggAAAAICAgCAAAACgoKKioAEjQA
AfgQ8as=
'/*!*/;
# at 123
#180811 17:17:08 server id 1  end_log_pos 154 CRC32 0x212da27e  Previous-GTIDs
# [empty]
# at 154                                               ## 从此开始
#180811 17:18:48 server id 1  end_log_pos 219 CRC32 0xcf04baf6  GTID    last_committed=0        sequence_number=1       rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= '03fde241-9a55-11e8-aa12-000c291f013a:1'/*!*/;
# at 219
#180811 17:18:48 server id 1  end_log_pos 291 CRC32 0xf3af79c9  Query   thread_id=2     exec_time=0     error_code=0
SET TIMESTAMP=1533979128/*!*/;
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
#180811 17:18:48 server id 1  end_log_pos 337 CRC32 0x306e7ae8  Table_map: `test`.`t1` mapped to number 109
# at 337
#180811 17:18:48 server id 1  end_log_pos 391 CRC32 0x4f3e4368  Update_rows: table id 109 flags: STMT_END_F

BINLOG '
+KluWxMBAAAALgAAAFEBAAAAAG0AAAAAAAEABHRlc3QAAnQxAAIDAwAC6HpuMA==
+KluWx8BAAAANgAAAIcBAAAAAG0AAAAAAAEAAgAC///8AQAAAAEAAAD8AQAAAAsAAABoQz5P
'/*!*/;
### UPDATE `test`.`t1`
### WHERE
###   @1=1 /* INT meta=0 nullable=0 is_null=0 */
###   @2=1 /* INT meta=0 nullable=1 is_null=0 */
### SET
###   @1=1 /* INT meta=0 nullable=0 is_null=0 */
###   @2=11 /* INT meta=0 nullable=1 is_null=0 */
# at 391
#180811 17:18:48 server id 1  end_log_pos 422 CRC32 0xc02faebb  Xid = 86
COMMIT/*!*/;
SET @@SESSION.GTID_NEXT= 'AUTOMATIC' /* added by mysqlbinlog */ /*!*/;
DELIMITER ;
# End of log file
/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/;



```
- 找到引发报错的日志

```sql
mysql> desc test.t1 ;
+-------+---------+------+-----+---------+-------+
| Field | Type    | Null | Key | Default | Extra |
+-------+---------+------+-----+---------+-------+
| c1    | int(11) | NO   | PRI | NULL    |       |
| c2    | int(11) | YES  |     | NULL    |       |
+-------+---------+------+-----+---------+-------+
2 rows in set (0.01 sec)

mysql> select * from t1 where c1=1 ;
ERROR 1046 (3D000): No database selected
mysql> select * from test.t1 where c1=1 ;
+----+------+
| c1 | c2   |
+----+------+
|  1 |   11 |
+----+------+
1 row in set (0.00 sec)

```
- 手工同步数据

```sql
mysql> insert into test.t1 values (1,11) ;
Query OK, 1 row affected (0.05 sec)


```
- 跳过GTID 

```sql

mysql> stop slave ;
Query OK, 0 rows affected (0.00 sec)
mysql> SET @@SESSION.GTID_NEXT= '03fde241-9a55-11e8-aa12-000c291f013a:1';
Query OK, 0 rows affected (0.00 sec)

mysql> begin ; commit ;
Query OK, 0 rows affected (0.01 sec)

Query OK, 0 rows affected (0.00 sec)

mysql> SET SESSION GTID_NEXT = AUTOMATIC;
Query OK, 0 rows affected (0.00 sec)

```
- 启动同步

```sql

mysql> start slave ;
Query OK, 0 rows affected (0.10 sec)

```
- 查看同步状态

```sql

mysql> show slave status \G ;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.2.61
                  Master_User: dao
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000001
          Read_Master_Log_Pos: 422
               Relay_Log_File: node2-relay-bin.000004
                Relay_Log_Pos: 454
        Relay_Master_Log_File: mysql-bin.000001
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
          Exec_Master_Log_Pos: 422
              Relay_Log_Space: 961
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
           Retrieved_Gtid_Set: 03fde241-9a55-11e8-aa12-000c291f013a:1
            Executed_Gtid_Set: 03fde241-9a55-11e8-aa12-000c291f013a:1
                Auto_Position: 1
         Replicate_Rewrite_DB: 
                 Channel_Name: 
           Master_TLS_Version: 
1 row in set (0.00 sec)
```
#### 使用参数跳过错误

- 此种方式仅仅是跳过错误,两边数据并不同步

- 从库设置skip-slave-errors参数

```sql


```

- 查看同步状态

```sql



```
- 查看主库数据

```sql

mysql> select * from test.t1 ;
+----+------+
| c1 | c2   |
+----+------+
|  1 |   11 |
|  2 |    2 |
+----+------+
2 rows in set (0.00 sec)

```

- 查看从库数据

```sql

mysql> select * from test.t1 ;
+----+------+
| c1 | c2   |
+----+------+
|  2 |    2 |
+----+------+
1 row in set (0.00 sec)


```

### 使用工具同步缺失数据

#### 跳过错误

```sql

mysql> SET @@SESSION.GTID_NEXT= '03fde241-9a55-11e8-aa12-000c291f013a:1';
Query OK, 0 rows affected (0.00 sec)

mysql> begin ; commit ;
Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

mysql>  SET SESSION GTID_NEXT = AUTOMATIC;
Query OK, 0 rows affected (0.00 sec)
```

- 启动同步
```sql

mysql> start slave ;
Query OK, 0 rows affected (0.01 sec)
```
- 查看同步状态
```sql
mysql> show slave status \G ;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.2.61
                  Master_User: dao
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000001
          Read_Master_Log_Pos: 2777
               Relay_Log_File: node2-relay-bin.000004
                Relay_Log_Pos: 454
        Relay_Master_Log_File: mysql-bin.000001
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
          Exec_Master_Log_Pos: 2777
              Relay_Log_Space: 3316
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
           Retrieved_Gtid_Set: 03fde241-9a55-11e8-aa12-000c291f013a:1-6
            Executed_Gtid_Set: 03fde241-9a55-11e8-aa12-000c291f013a:1-6
                Auto_Position: 1
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
1 row in set (0.00 sec)

```

#### 使用工具查找缺失数据

- 主库运行数据检查工具
```sql
[mysql@node1 ~]$  pt-table-checksum --host='192.168.2.61' --user=dao --password=dao --create-replicate-table  --nocheck-replication-filters --replicate=test.checksums   --no-check-binlog-format --databases=test
            TS ERRORS  DIFFS     ROWS  CHUNKS SKIPPED    TIME TABLE
08-11T17:30:10      0      1        2       1       0   0.286 test.t1


```

- 打印同步内容

```sql

[mysql@node1 ~]$ pt-table-sync --print --sync-to-master 192.168.2.62 --user=dao --password=dao
REPLACE INTO `test`.`checksums`(`db`, `tbl`, `chunk`, `chunk_time`, `chunk_index`, `lower_boundary`, `upper_boundary`, `this_crc`, `this_cnt`, `master_crc`, `master_cnt`, `ts`) VALUES ('test', 't1', '1', 0.013511, NULL, NULL, NULL, '1d101dc9', '2', '1d101dc9', '2', '2018-08-11 17:30:09') /*percona-toolkit src_db:test src_tbl:checksums src_dsn:P=3306,h=192.168.2.61,p=...,u=dao dst_db:test dst_tbl:checksums dst_dsn:h=192.168.2.62,p=...,u=dao lock:1 transaction:1 changing_src:1 replicate:0 bidirectional:0 pid:2729 user:mysql host:node1*/;
REPLACE INTO `test`.`t1`(`c1`, `c2`) VALUES ('1', '11') /*percona-toolkit src_db:test src_tbl:t1 src_dsn:P=3306,h=192.168.2.61,p=...,u=dao dst_db:test dst_tbl:t1 dst_dsn:h=192.168.2.62,p=...,u=dao lock:1 transaction:1 changing_src:1 replicate:0 bidirectional:0 pid:2729 user:mysql host:node1*/;


```

- 同步数据

```sql

[mysql@node1 ~]$ pt-table-sync --execute  --sync-to-master 192.168.2.62 --user=dao --password=dao

```
#### 核对表中数据

- 主库 

```sql

mysql> select * from test.t1 ;
+----+------+
| c1 | c2   |
+----+------+
|  1 |   11 |
|  2 |    2 |
+----+------+
2 rows in set (0.00 sec)

```

-从库 

```sql

mysql> select * from test.t1 ;
+----+------+
| c1 | c2   |
+----+------+
|  1 |   11 |
|  2 |    2 |
+----+------+
2 rows in set (0.00 sec)

```

### 设置skip-slave-errors 跳过错误

#### 查看从库参数文件

```sql

[mysql@node2 ~]$ cat  /mysql1/mysql1.cnf
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
slave-skip-errors=1032
log-slave-updates=1



#gtid

gtid_mode=on
log_slave_updates=1
enforce_gtid_consistency=1

```

#### 重启从库

````sql

[mysql@node2 ~]$ mysqld --defaults-file=/mysql1/mysql1.cnf &
[1] 2739

```
#### 模拟错误

```sql

mysql> delete from test.t1 where c1=1 ;
Query OK, 1 row affected (0.04 sec)

```

#### 查看同步状态

```sql


mysql> show slave status \G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.2.61
                  Master_User: dao
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000001
          Read_Master_Log_Pos: 6479
               Relay_Log_File: node2-relay-bin.000006
                Relay_Log_Pos: 454
        Relay_Master_Log_File: mysql-bin.000001
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
          Exec_Master_Log_Pos: 6479
              Relay_Log_Space: 701
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
           Retrieved_Gtid_Set: 03fde241-9a55-11e8-aa12-000c291f013a:1-13
            Executed_Gtid_Set: 03fde241-9a55-11e8-aa12-000c291f013a:1-13,
f63b765b-9c1d-11e8-aa1e-005056296858:1
                Auto_Position: 1
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
1 row in set (0.00 sec)

ERROR:
No query specified

```
#### 查看表中数据

两表数据并不相同 ,但是同步并不报错
 
- 主库

```sql


mysql> select * from test.t1 ;
+----+------+
| c1 | c2   |
+----+------+
|  1 |   11 |
|  2 |    2 |
+----+------+
2 rows in set (0.00 sec)

```

#### 查看表中数据

- 从库

```sql

mysql> select * from test.t1 ;
+----+------+
| c1 | c2   |
+----+------+
|  2 |    2 |
+----+------+
1 row in set (0.00 sec)


```