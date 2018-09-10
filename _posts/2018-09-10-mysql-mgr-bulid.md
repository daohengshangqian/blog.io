---
layout: post
title: MySQL MGR搭建
date: 2018-09-10
categories: blog
tags: [MySQL]
description: MySQL MGR搭建
---

## 规划

HOSTNAME | IP  |ROLE
--- | --- | ---
node1 | 192.168.2.61 | first node
node2 | 192.168.2.62 | 
node3 | 192.168.2.63 | 

## 三节点重置主从信息 

```sql


mysql> reset master ;
Query OK, 0 rows affected (0.01 sec)


mysql> stop slave ;
Query OK, 0 rows affected (0.00 sec)

mysql> reset slave all ;
Query OK, 0 rows affected (0.02 sec)


```

##　修改配置文件

```sql

[mysql@node3 ~]$ cat /mysql1/mysql1.cnf
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
server_id = 3

#gtid

gtid_mode=on
log_slave_updates=1
enforce_gtid_consistency=1

#rep
slave-parallel-type=LOGICAL_CLOCK
slave-parallel-workers=16
master_info_repository=TABLE
relay_log_info_repository=TABLE
relay_log_recovery=ON
binlog_checksum=NONE
slave_preserve_commit_order=1


transaction_write_set_extraction    =XXHASH64                    #    off
loose-group_replication_group_name    ="a876d35e-9110-11e6-a365-842b2b5909d6"        #
loose-group_replication_start_on_boot    =off                        #    off
loose-group_replication_local_address    ="192.168.2.63:24900"                #
loose-group_replication_group_seeds    ="192.168.2.61:24900,192.168.2.62:24900,192.168.2.63:24900"
loose-group_replication_bootstrap_group    =off



```

## 启动MGR


### 启动MYSQL实例(三节点)


```sql


[mysql@node1 ~]$ mysqld_safe --defaults-file=/mysql1/mysql1.cnf &
[1] 12148
[mysql@node1 ~]$ 2018-08-08T12:52:47.022449Z mysqld_safe Logging to '/mysql1/mysql1.log'.
2018-08-08T12:52:47.049333Z mysqld_safe Starting mysqld daemon with databases from /mysql1/data


[mysql@node2 ~]$ mysqld_safe --defaults-file=/mysql1/mysql1.cnf &
[1] 12052
[mysql@node2 ~]$ 2018-08-08T19:21:33.338535Z mysqld_safe Logging to '/mysql1/mysql1.log'.
2018-08-08T19:21:33.374104Z mysqld_safe Starting mysqld daemon with databases from /mysql1/data

[mysql@node3 ~]$ clearmysqld_safe --defaults-file=/mysql1/mysql1.cnf &
[1] 5831

```

###　启动第一节点MGR

```sql

mysql> INSTALL PLUGIN group_replication SONAME 'group_replication.so';
Query OK, 0 rows affected (0.19 sec)

mysql> SET GLOBAL group_replication_bootstrap_group=ON;
Query OK, 0 rows affected (0.00 sec)

mysql> start group_replication ;
Query OK, 0 rows affected (3.57 sec)

```

### 启动其他两个节点MGR(命令相同)

```sql

mysql> INSTALL PLUGIN group_replication SONAME 'group_replication.so';
Query OK, 0 rows affected (0.04 sec)

mysql> CHANGE MASTER TO MASTER_USER='dao', MASTER_PASSWORD='dao' FOR CHANNEL 'group_replication_recovery';
Query OK, 0 rows affected, 2 warnings (0.04 sec)

mysql> START GROUP_REPLICATION ;
Query OK, 0 rows affected (7.16 sec)


```

## MGR信息查看


### 查看是否为主节点

- node1

```sql


mysql> SELECT IF((SELECT @@server_uuid) = (SELECT VARIABLE_VALUE FROM performance_schema.global_status WHERE VARIABLE_NAME= 'group_replication_primary_member'), 1, 0) as is_primary_node ,@@server_id;
+-----------------+-------------+
| is_primary_node | @@server_id |
+-----------------+-------------+
|               1 |           1 |
+-----------------+-------------+
1 row in set (0.00 sec)


```

- node2

```sql

mysql> SELECT IF((SELECT @@server_uuid) = (SELECT VARIABLE_VALUE FROM performance_schema.global_status WHERE VARIABLE_NAME= 'group_replication_primary_member'), 1, 0) as is_primary_node ,@@server_id;
+-----------------+-------------+
| is_primary_node | @@server_id |
+-----------------+-------------+
|               0 |           2 |
+-----------------+-------------+
1 row in set (0.00 sec)


```

- node3


```sql

mysql>  SELECT IF((SELECT @@server_uuid) = (SELECT VARIABLE_VALUE FROM performance_schema.global_status WHERE VARIABLE_NAME= 'group_replication_primary_member'), 1, 0) as is_primary_node ,@@server_id;
+-----------------+-------------+
| is_primary_node | @@server_id |
+-----------------+-------------+
|               0 |           3 |
+-----------------+-------------+
1 row in set (0.00 sec)


```

### 查看主节点信息

```sql

mysql>  SELECT * FROM performance_schema.replication_group_members WHERE MEMBER_ID = ( SELECT VARIABLE_VALUE FROM performance_schema.global_status WHERE VARIABLE_NAME= 'group_replication_primary_member' );
+---------------------------+--------------------------------------+-------------+-------------+--------------+
| CHANNEL_NAME              | MEMBER_ID                            | MEMBER_HOST | MEMBER_PORT | MEMBER_STATE |
+---------------------------+--------------------------------------+-------------+-------------+--------------+
| group_replication_applier | 03fde241-9a55-11e8-aa12-000c291f013a | node1       |        3306 | ONLINE       |
+---------------------------+--------------------------------------+-------------+-------------+--------------+
1 row in set (0.01 sec)


```

### 查看成员状态



```sql

mysql>  select * from performance_schema.replication_group_member_stats \G;
*************************** 1. row ***************************
                      CHANNEL_NAME: group_replication_applier
                           VIEW_ID: 15337333448542910:6
                         MEMBER_ID: 03fde241-9a55-11e8-aa12-000c291f013a
       COUNT_TRANSACTIONS_IN_QUEUE: 0
        COUNT_TRANSACTIONS_CHECKED: 0
          COUNT_CONFLICTS_DETECTED: 0
COUNT_TRANSACTIONS_ROWS_VALIDATING: 0
TRANSACTIONS_COMMITTED_ALL_MEMBERS: a876d35e-9110-11e6-a365-842b2b5909d6:1-3
    LAST_CONFLICT_FREE_TRANSACTION:
1 row in set (0.00 sec)


```

##　将MGR从单主模式变为多主模式

###　停止MGR

- node3 

```sql

mysql> stop group_replication ;
Query OK, 0 rows affected (1.02 sec)

```

- node2

```sql

mysql> stop group_replication ;
Query OK, 0 rows affected (1.02 sec)

```

- node1 

```sql

mysql> stop group_replication ;
Query OK, 0 rows affected (1.02 sec)

```

###　修改参数

- node3 

```sql

mysql> set global group_replication_single_primary_mode =off ;
Query OK, 0 rows affected (0.00 sec)


```

- node2

```sql

mysql> set global group_replication_single_primary_mode =off ;
Query OK, 0 rows affected (0.00 sec)


```

- node1 

```sql

mysql> set global group_replication_single_primary_mode =off ;
Query OK, 0 rows affected (0.00 sec)


```

###　启动MGR

- node1

```sql


mysql> start group_replication ;
Query OK, 0 rows affected (2.16 sec)


```

- node2

```sql

mysql> start group_replication ;
Query OK, 0 rows affected (6.50 sec)

```

- node3 

```sql

mysql> start group_replication ;
Query OK, 0 rows affected (6.63 sec)

```

###　查看各节点是否为主节点


- node1

```sql

mysql>  SELECT IF((SELECT @@server_uuid) = (SELECT VARIABLE_VALUE FROM performance_schema.global_status WHERE VARIABLE_NAME= 'group_replication_primary_member'), 1, 0) as is_primary_node ,@@server_id;
+-----------------+-------------+
| is_primary_node | @@server_id |
+-----------------+-------------+
|               0 |           1 |
+-----------------+-------------+
1 row in set (0.00 sec)


```

- node2

```sql


mysql>  SELECT IF((SELECT @@server_uuid) = (SELECT VARIABLE_VALUE FROM performance_schema.global_status WHERE VARIABLE_NAME= 'group_replication_primary_member'), 1, 0) as is_primary_node ,@@server_id;
+-----------------+-------------+
| is_primary_node | @@server_id |
+-----------------+-------------+
|               0 |           2 |
+-----------------+-------------+
1 row in set (0.00 sec)


```

- node3 

```sql

mysql>  SELECT IF((SELECT @@server_uuid) = (SELECT VARIABLE_VALUE FROM performance_schema.global_status WHERE VARIABLE_NAME= 'group_replication_primary_member'), 1, 0) as is_primary_node ,@@server_id;
+-----------------+-------------+
| is_primary_node | @@server_id |
+-----------------+-------------+
|               0 |           3 |
+-----------------+-------------+


```