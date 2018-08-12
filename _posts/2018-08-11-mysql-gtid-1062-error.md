---
layout: post
title: MySQL GTID模式下1062错误详解
date: 2018-08-11
categories: blog
tags: [MySQL]
description: MySQL GTID模式下1062错误详解
---


## 错误模拟

### 查看主库从库数据

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
- 从库

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
### 向从库中插入数据

```sql

mysql> insert into test.t1 values (3,3) ;
Query OK, 1 row affected (0.04 sec)

```

### 主库插入重复数据

```sql

mysql> insert into test.t1 values (3,33) ;
Query OK, 1 row affected (0.01 sec)

```

### 查看同步状态

```sql

mysql> show slave status \G ;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.2.61
                  Master_User: dao
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000001
          Read_Master_Log_Pos: 7275
               Relay_Log_File: node2-relay-bin.000006
                Relay_Log_Pos: 996
        Relay_Master_Log_File: mysql-bin.000001
             Slave_IO_Running: Yes
            Slave_SQL_Running: No
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 1062
                   Last_Error: Could not execute Write_rows event on table test.t1; Duplicate entry '3' for key 'PRIMARY', Error_code: 1062; handler error HA_ERR_FOUND_DUPP_KEY; the event's master log mysql-bin.000001, end_log_pos 7244
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 7021
              Relay_Log_Space: 1497
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
               Last_SQL_Errno: 1062
               Last_SQL_Error: Could not execute Write_rows event on table test.t1; Duplicate entry '3' for key 'PRIMARY', Error_code: 1062; handler error HA_ERR_FOUND_DUPP_KEY; the event's master log mysql-bin.000001, end_log_pos 7244
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
     Last_SQL_Error_Timestamp: 180811 19:05:24
               Master_SSL_Crl:
           Master_SSL_Crlpath:
           Retrieved_Gtid_Set: 03fde241-9a55-11e8-aa12-000c291f013a:1-15
            Executed_Gtid_Set: 03fde241-9a55-11e8-aa12-000c291f013a:1-14,
f63b765b-9c1d-11e8-aa1e-005056296858:1-2
                Auto_Position: 1
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
1 row in set (0.00 sec)

ERROR:
No query specified

```


## 错误解决

### 手工同步数据

- 停止同步

```sql

mysql> stop slave ;
Query OK, 0 rows affected (0.00 sec)

```

- 修改数据

```sql

mysql> update test.t1 set c2=33 where c1=3 ;
Query OK, 1 row affected (0.27 sec)
Rows matched: 1  Changed: 1  Warnings: 0

```

- 跳过GTID


```sql

mysql>  SET @@SESSION.GTID_NEXT= '03fde241-9a55-11e8-aa12-000c291f013a:15' ;
Query OK, 0 rows affected (0.00 sec)

mysql> begin; commit;
Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

mysql> SET SESSION GTID_NEXT = AUTOMATIC;
Query OK, 0 rows affected (0.00 sec)

```
- 开启复制

```sql

mysql> start slave ;
Query OK, 0 rows affected (0.01 sec)

```
- 查看复制状态

```sql

mysql> show slave status \G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.2.61
                  Master_User: dao
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000001
          Read_Master_Log_Pos: 7275
               Relay_Log_File: node2-relay-bin.000007
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
          Exec_Master_Log_Pos: 7275
              Relay_Log_Space: 1757
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
           Retrieved_Gtid_Set: 03fde241-9a55-11e8-aa12-000c291f013a:1-15
            Executed_Gtid_Set: 03fde241-9a55-11e8-aa12-000c291f013a:1-15,
f63b765b-9c1d-11e8-aa1e-005056296858:1-3
                Auto_Position: 1
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
1 row in set (0.00 sec)

```

### 使用工具同步数据

- 跳过错误

```sql


mysql> SET @@SESSION.GTID_NEXT= '03fde241-9a55-11e8-aa12-000c291f013a:15' ;
Query OK, 0 rows affected (0.00 sec)

mysql> begin ; commit ;
Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

mysql>  SET SESSION GTID_NEXT = AUTOMATIC;
Query OK, 0 rows affected (0.00 sec)

```
- 启动从库

```sql

mysql> start slave ;
Query OK, 0 rows affected (0.08 sec)

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
          Read_Master_Log_Pos: 7275
               Relay_Log_File: node2-relay-bin.000006
                Relay_Log_Pos: 1250
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
          Exec_Master_Log_Pos: 7275
              Relay_Log_Space: 1497
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
           Retrieved_Gtid_Set: 03fde241-9a55-11e8-aa12-000c291f013a:1-15
            Executed_Gtid_Set: 03fde241-9a55-11e8-aa12-000c291f013a:1-15,
f63b765b-9c1d-11e8-aa1e-005056296858:1-2
                Auto_Position: 1
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
1 row in set (0.00 sec)

ERROR:


```
- 主库检查冲突数据

```sql

[root@node1 ~]# su - mysql
[mysql@node1 ~]$ clear
[mysql@node1 ~]$ pt-table-checksum --host='192.168.2.61' --user=dao --password=dao --create-replicate-table  --nocheck-replication-filters --replicate=test.checksums   --no-check-binlog-format --databases=test
            TS ERRORS  DIFFS     ROWS  CHUNKS SKIPPED    TIME TABLE
08-11T19:21:07      0      1        3       1       0   1.179 test.t1


```
- 同步数据

```sql


[mysql@node1 ~]$ pt-table-sync --execute --sync-to-master 192.168.2.62 --user=dao --password=dao

```
- 查询主库数据

```sql

mysql> select * from test.t1 ;
+----+------+
| c1 | c2   |
+----+------+
|  1 |   11 |
|  2 |    2 |
|  3 |   33 |
+----+------+

```

- 查看从库数据

```sql

mysql> select * from test.t1 ;
+----+------+
| c1 | c2   |
+----+------+
|  1 |   11 |
|  2 |    2 |
|  3 |   33 |
+----+------+

```

### 使用skip-slave-errors跳过错误

- 修改配置文件

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
slave-skip-errors=1032,1062
log-slave-updates=1



#gtid

gtid_mode=on
log_slave_updates=1
enforce_gtid_consistency=1

```

- 重启Mysql

```sql

[mysql@node2 ~]$ mysqld --defaults-file=/mysql1/mysql1.cnf  &
[1] 3161

```



- 从库先插入数据

```sql

mysql> insert into test.t1 values (4,4) ;
Query OK, 1 row affected (0.02 sec)


```
- 主库插入冲突数据

```sql

mysql> insert into test.t1 values (4,44) ;
Query OK, 1 row affected (0.02 sec)

```

- 查看同步状态

此时同步并不报错,但是主从数据不一致

```sql

mysql> show slave status \G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.2.61
                  Master_User: dao
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000001
          Read_Master_Log_Pos: 11739
               Relay_Log_File: node2-relay-bin.000008
                Relay_Log_Pos: 1216
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
          Exec_Master_Log_Pos: 11739
              Relay_Log_Space: 1463
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
           Retrieved_Gtid_Set: 03fde241-9a55-11e8-aa12-000c291f013a:1-25
            Executed_Gtid_Set: 03fde241-9a55-11e8-aa12-000c291f013a:1-25,
f63b765b-9c1d-11e8-aa1e-005056296858:1-3
                Auto_Position: 1
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
1 row in set (0.00 sec)



```

- 查看主库数据

```sql

mysql> select * from test.t1 ;
+----+------+
| c1 | c2   |
+----+------+
|  1 |   11 |
|  2 |    2 |
|  3 |   33 |
|  4 |   44 |
+----+------+
4 rows in set (0.00 sec)


```

- 查看从库数据

```sql


mysql> select * from test.t1
    -> ;
+----+------+
| c1 | c2   |
+----+------+
|  1 |   11 |
|  2 |    2 |
|  3 |   33 |
|  4 |    4 |
+----+------+
4 rows in set (0.00 sec)


```