---
layout: post
title: MySQL半同步复制
date: 2018-08-09
categories: blog
tags: [MySQL]
description: MySQL半同步复制
---

## 参数说明

参数名 | 说明
--- | ---
plugin_dir  | 插件所在文件夹
plugin_load | 需要装载的插件
loose_rpl_semi_sync_master_enabled  | 主库启动半同步复制
loose_rpl_semi_sync_slave_enabled   | 从库启动半同步复制
loose_rpl_semi_sync_master_timeout  | 半同步超时时间,超时后变为异步复制
rpl_semi_sync_master_wait_for_slave_count | 一主多从半同步复制,最低从库返回确认信息提交数量
rpl_semi_sync_master_wait_point     | 半同步模式 

- rpl_semi_sync_master_wait_point

在5.6中只有AFTER_COMMIT可选
在5.7中可选择AFTER_COMMIT / AFTER_SYNC

## 修改配置文件

```sql

[root@node1 plugin]# cat  /mysql1/mysql1.cnf
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

# semi_sync

plugin_dir=/usr/local/mysql/lib/plugin
plugin_load = "rpl_semi_sync_master=semisync_master.so;rpl_semi_sync_slave=semisync_slave.so"
loose_rpl_semi_sync_master_enabled = 1
loose_rpl_semi_sync_slave_enabled = 1
loose_rpl_semi_sync_master_timeout = 5000
rpl_semi_sync_master_wait_point = AFTER_SYNC
rpl_semi_sync_master_wait_for_slave_count = 1


```

## 启动数据库

```sql

[mysql@node1 ~]$ mysqld --defaults-file=/mysql1/mysql1.cnf &
[1] 2441

[mysql@node2 ~]$ mysqld --defaults-file=/mysql1/mysql1.cnf &
[1] 2311

``` 

## 查看实例日志

### 主库启动日志

```sql
2018-08-09T08:48:40.861651Z 0 [Note] InnoDB: Buffer pool(s) load completed at 180809 16:48:40
2018-08-09T08:48:40.887714Z 0 [Note] Semi-sync replication initialized for transactions.
2018-08-09T08:48:40.887738Z 0 [Note] Semi-sync replication enabled on the master.
2018-08-09T08:48:40.888086Z 0 [Note] Starting ack receiver thread
2018-08-09T08:48:40.932000Z 0 [Note] Recovering after a crash using /mysql1/data/mysql-bin
2018-08-09T08:48:40.932268Z 0 [Note] Starting crash recovery...
2018-08-09T08:48:40.932414Z 0 [Note] Crash recovery finished.
2018-08-09T08:48:41.010476Z 0 [Warning] Failed to set up SSL because of the following SSL library error: SSL context is not usable without certificate and private key
2018-08-09T08:48:41.010559Z 0 [Note] Server hostname (bind-address): '*'; port: 3306
2018-08-09T08:48:41.021671Z 0 [Note] IPv6 is available.
2018-08-09T08:48:41.021707Z 0 [Note]   - '::' resolves to '::';
2018-08-09T08:48:41.021725Z 0 [Note] Server socket created on IP: '::'.
2018-08-09T08:48:41.285062Z 0 [Note] Event Scheduler: Loaded 0 events
2018-08-09T08:48:41.285534Z 0 [Note] mysqld: ready for connections.
Version: '5.7.21-log'  socket: '/tmp/mysql1.sock'  port: 3306  MySQL Community Server (GPL)

```

### 从库启动日志

```sql

2018-08-09T09:17:21.193305Z 0 [Note] Semi-sync replication initialized for transactions.
2018-08-09T09:17:21.193348Z 0 [Note] Semi-sync replication enabled on the master.
2018-08-09T09:17:21.193783Z 0 [Note] InnoDB: Loading buffer pool(s) from /mysql1/data/ib_buffer_pool
2018-08-09T09:17:21.195277Z 0 [Note] InnoDB: Buffer pool(s) load completed at 180809 17:17:21
2018-08-09T09:17:21.195329Z 0 [Note] Starting ack receiver thread
2018-08-09T09:17:21.205577Z 0 [Warning] Failed to set up SSL because of the following SSL library error: SSL context is not usable without certificate and private key
2018-08-09T09:17:21.205605Z 0 [Note] Server hostname (bind-address): '*'; port: 3306
2018-08-09T09:17:21.205676Z 0 [Note] IPv6 is available.
2018-08-09T09:17:21.205684Z 0 [Note]   - '::' resolves to '::';
2018-08-09T09:17:21.205699Z 0 [Note] Server socket created on IP: '::'.
2018-08-09T09:17:21.211372Z 0 [Warning] Neither --relay-log nor --relay-log-index were used; so replication may break when this MySQL server acts as a slave and has his hostname changed!! Please use '--relay-log=node2-relay-bin' to avoid this problem.
2018-08-09T09:17:21.225231Z 1 [Warning] Slave SQL for channel '': If a crash happens this configuration does not guarantee that the relay log info will be consistent, Error_code: 0
2018-08-09T09:17:21.225296Z 1 [Note] Slave SQL thread for channel '' initialized, starting replication in log 'mysql-bin.000004' at position 154, relay log './node2-relay-bin.000012' position: 320
2018-08-09T09:17:21.225647Z 2 [Note] Slave I/O thread: Start semi-sync replication to master 'dao@192.168.2.61:3306' in log 'mysql-bin.000004' at position 154
2018-08-09T09:17:21.225684Z 2 [Warning] Storing MySQL user name or password information in the master info repository is not secure and is therefore not recommended. Please consider using the USER and PASSWORD connection options for START SLAVE; see the 'START SLAVE Syntax' in the MySQL Manual for more information.
2018-08-09T09:17:21.227055Z 0 [Note] Event Scheduler: Loaded 0 events
2018-08-09T09:17:21.227473Z 0 [Note] mysqld: ready for connections.
Version: '5.7.21-log'  socket: '/tmp/mysql1.sock'  port: 3306  MySQL Community Server (GPL)


```
## 查看半同步复制状态

### 主库 

```sql

mysql> show status like 'Rpl_semi_sync_master_status';
+-----------------------------+-------+
| Variable_name               | Value |
+-----------------------------+-------+
| Rpl_semi_sync_master_status | ON    |
+-----------------------------+-------+
1 row in set (0.00 sec)

```

### 从库

```sql

mysql> show status like 'Rpl_semi_sync_master_status';
+-----------------------------+-------+
| Variable_name               | Value |
+-----------------------------+-------+
| Rpl_semi_sync_master_status | ON    |
+-----------------------------+-------+
1 row in set (0.00 sec)

```


```sql

mysql> show slave status \G ;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.2.61
                  Master_User: dao
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000004
          Read_Master_Log_Pos: 154
               Relay_Log_File: node2-relay-bin.000009
                Relay_Log_Pos: 367
        Relay_Master_Log_File: mysql-bin.000004
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
          Exec_Master_Log_Pos: 154
              Relay_Log_Space: 740
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
           Retrieved_Gtid_Set:
            Executed_Gtid_Set:
                Auto_Position: 0
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
1 row in set (0.00 sec)

ERROR:
No query specified

```
### 查看半同步状态



```sql




mysql>  show status like '%semi%' ;
+--------------------------------------------+-------+
| Variable_name                              | Value |
+--------------------------------------------+-------+
| Rpl_semi_sync_master_clients               | 1     |
| Rpl_semi_sync_master_net_avg_wait_time     | 0     |
| Rpl_semi_sync_master_net_wait_time         | 0     |
| Rpl_semi_sync_master_net_waits             | 2     |
| Rpl_semi_sync_master_no_times              | 1     |
| Rpl_semi_sync_master_no_tx                 | 3     |
| Rpl_semi_sync_master_status                | ON    |
| Rpl_semi_sync_master_timefunc_failures     | 0     |
| Rpl_semi_sync_master_tx_avg_wait_time      | 1090  |
| Rpl_semi_sync_master_tx_wait_time          | 2181  |
| Rpl_semi_sync_master_tx_waits              | 2     |
| Rpl_semi_sync_master_wait_pos_backtraverse | 0     |
| Rpl_semi_sync_master_wait_sessions         | 0     |
| Rpl_semi_sync_master_yes_tx                | 2     |
| Rpl_semi_sync_slave_status                 | OFF   |
+--------------------------------------------+-------+
15 rows in set (0.01 sec)



```

### 从库


```sql


mysql> show status like '%semi%' ;
+--------------------------------------------+-------+
| Variable_name                              | Value |
+--------------------------------------------+-------+
| Rpl_semi_sync_master_clients               | 0     |
| Rpl_semi_sync_master_net_avg_wait_time     | 0     |
| Rpl_semi_sync_master_net_wait_time         | 0     |
| Rpl_semi_sync_master_net_waits             | 0     |
| Rpl_semi_sync_master_no_times              | 1     |
| Rpl_semi_sync_master_no_tx                 | 2     |
| Rpl_semi_sync_master_status                | OFF   |
| Rpl_semi_sync_master_timefunc_failures     | 0     |
| Rpl_semi_sync_master_tx_avg_wait_time      | 0     |
| Rpl_semi_sync_master_tx_wait_time          | 0     |
| Rpl_semi_sync_master_tx_waits              | 0     |
| Rpl_semi_sync_master_wait_pos_backtraverse | 0     |
| Rpl_semi_sync_master_wait_sessions         | 0     |
| Rpl_semi_sync_master_yes_tx                | 0     |
| Rpl_semi_sync_slave_status                 | ON    |
+--------------------------------------------+-------+


```



指标名称  | 含义
--- | ---
Rpl_semi_sync_master_status | 主服务器复制模式
Rpl_semi_sync_slave_status  | 从服务器复制模式
Rpl_semi_sync_master_yes_tx | 主服务器确认提交成功的事务数量
Rpl_semi_sync_master_no_tx  | 主服务器确认提交不成功的事务数量
Rpl_semi_sync_master_clients| 半同步从库数量
