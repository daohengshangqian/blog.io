---
layout: post
title: 使用mysqldump在线搭建主从
date: 2018-08-07
categories: blog
tags: [MySQL]
description: 使用mysqldump在线搭建主从
---

## 对主库进行备份

```sql

[mysql@node1 mysql1]$ mysqldump -uroot -proot -S /tmp/mysql1.sock  --master-data=2 --single-transaction -A >/home/mysql/slavedata.sql
mysqldump: [Warning] Using a password on the command line interface can be insecure.


```
## 查看主库配置文件

```sql

[mysql@node1 mysql1]$ cat /mysql1/mysql1.cnf

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


```

## 创建复制用账户


```sql

mysql> GRANT ALL PRIVILEGES ON *.* TO 'dao'@'192.168.2.52' IDENTIFIED BY 'dao' ;
Query OK, 0 rows affected, 1 warning (0.01 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.01 sec)


```

## 对从库进行初始化

```sql
[mysql@node2 ~]$ mysqld --defaults-file=/mysql1/mysql1.cnf --initialize-insecure

```
## 查看从库配置文件

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


```

## 启动从库并修改密码

```sql

[mysql@node2 ~]$ mysqld --defaults-file=/mysql1/mysql1.cnf &
[1] 3108

[mysql@node2 ~]$ mysql -S /tmp/mysql1.sock -uroot
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.21-log MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> alter user root@'localhost' identified by 'root' ;
Query OK, 0 rows affected (0.02 sec)


```

### 复制 

```sql

[root@node1 ~]# scp /home/mysql/slavedata.sql  192.168.2.52:/home/mysql/
The authenticity of host '192.168.2.52 (192.168.2.52)' can't be established.
RSA key fingerprint is bb:87:21:f9:01:9e:c2:de:9c:f8:d9:c7:20:8e:d9:07.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.2.52' (RSA) to the list of known hosts.

root@192.168.2.52's password:

```

## 导入从库

```sql

[mysql@node2 ~]$ mysql -S /tmp/mysql1.sock -uroot -proot
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.21-log MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> source /home/mysql/slavedata.sql


```

## 开启同步

```sql

mysql> change master to master_host='192.168.2.51',master_user='dao',master_password='dao',
    ->         master_log_file='mysql-bin.000002',master_log_pos=989;
Query OK, 0 rows affected, 2 warnings (0.03 sec)

mysql> start slave ;
Query OK, 0 rows affected (0.00 sec)
```

## 查看同步状态

```sql

mysql> show slave status \G ;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.2.51
                  Master_User: dao
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000002
          Read_Master_Log_Pos: 1431
               Relay_Log_File: node2-relay-bin.000002
                Relay_Log_Pos: 762
        Relay_Master_Log_File: mysql-bin.000002
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
          Exec_Master_Log_Pos: 1431
              Relay_Log_Space: 969
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
                  Master_UUID: 887aefe8-955a-11e8-94c9-000c2990d773
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

```
