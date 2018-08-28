---
layout: post
title: MySQL MHA工具包汇总
date: 2018-08-25
categories: blog
tags: [MySQL]
description: MySQL MHA工具包汇总
---



## 概述

- Manager工具包

脚本名称 | 作用
--- | ---
 masterha_check_repl     |  主从复制检查  
 masterha_check_ssh      |  互信检查       
 masterha_check_status   |  运行状态检查
 masterha_conf_host      |  手工添加server信息
 masterha_manager        |  MHA启动
 masterha_master_monitor |  监控master是否宕机
 masterha_master_switch  |  手工故障转移
 masterha_secondary_check|  多路由监测Master可用性
 masterha_stop           |  停止MHA    


- Node工具包

- Node工具包由MHA自动调用,本文中并不提供演示实例. 

脚本名称 | 作用
--- | ---
apply_diff_relay_logs | 识别差异的中继日志事件并应用于其它slave
filter_mysqlbinlog    | 去除不必要的ROLLBACK事件
purge_relay_logs      | 清除中继日志(不会阻塞SQL线程)
save_binary_logs      | 保存和复制master的二进制日志。

## Manager工具包


### masterha_check_repl 

```sql


[root@node3 mha]# /etc/mha/mhamanager/bin/masterha_check_repl --conf=/etc/mha/app1/app1.cnf
Fri Aug 24 02:00:33 2018 - [warning] Global configuration file /etc/masterha_default.cnf not found. Skipping.
Fri Aug 24 02:00:33 2018 - [info] Reading application default configuration from /etc/mha/app1/app1.cnf..
Fri Aug 24 02:00:33 2018 - [info] Reading server configuration from /etc/mha/app1/app1.cnf..
Fri Aug 24 02:00:33 2018 - [info] MHA::MasterMonitor version 0.57.
Fri Aug 24 02:00:33 2018 - [info] GTID failover mode = 0
Fri Aug 24 02:00:33 2018 - [info] Dead Servers:
Fri Aug 24 02:00:33 2018 - [info] Alive Servers:
Fri Aug 24 02:00:33 2018 - [info]   192.168.2.61(192.168.2.61:3306)
Fri Aug 24 02:00:33 2018 - [info]   192.168.2.62(192.168.2.62:3306)
Fri Aug 24 02:00:33 2018 - [info]   192.168.2.63(192.168.2.63:3306)
Fri Aug 24 02:00:33 2018 - [info] Alive Slaves:
Fri Aug 24 02:00:33 2018 - [info]   192.168.2.61(192.168.2.61:3306)  Version=5.7.21-log (oldest major version between slaves) log-bin:enabled
Fri Aug 24 02:00:33 2018 - [info]     Replicating from 192.168.2.62(192.168.2.62:3306)
Fri Aug 24 02:00:33 2018 - [info]     Primary candidate for the new Master (candidate_master is set)
Fri Aug 24 02:00:33 2018 - [info]   192.168.2.63(192.168.2.63:3306)  Version=5.7.21-log (oldest major version between slaves) log-bin:enabled
Fri Aug 24 02:00:33 2018 - [info]     Replicating from 192.168.2.62(192.168.2.62:3306)
Fri Aug 24 02:00:33 2018 - [info]     Not candidate for the new Master (no_master is set)
Fri Aug 24 02:00:33 2018 - [info] Current Alive Master: 192.168.2.62(192.168.2.62:3306)
Fri Aug 24 02:00:33 2018 - [info] Checking slave configurations..
Fri Aug 24 02:00:33 2018 - [info]  read_only=1 is not set on slave 192.168.2.61(192.168.2.61:3306).
Fri Aug 24 02:00:33 2018 - [warning]  relay_log_purge=0 is not set on slave 192.168.2.61(192.168.2.61:3306).
Fri Aug 24 02:00:33 2018 - [info]  read_only=1 is not set on slave 192.168.2.63(192.168.2.63:3306).
Fri Aug 24 02:00:33 2018 - [warning]  relay_log_purge=0 is not set on slave 192.168.2.63(192.168.2.63:3306).
Fri Aug 24 02:00:33 2018 - [info] Checking replication filtering settings..
Fri Aug 24 02:00:33 2018 - [info]  binlog_do_db= , binlog_ignore_db= 
Fri Aug 24 02:00:33 2018 - [info]  Replication filtering check ok.
Fri Aug 24 02:00:33 2018 - [info] GTID (with auto-pos) is not supported
Fri Aug 24 02:00:33 2018 - [info] Starting SSH connection tests..
Fri Aug 24 02:00:35 2018 - [info] All SSH connection tests passed successfully.
Fri Aug 24 02:00:35 2018 - [info] Checking MHA Node version..
Fri Aug 24 02:00:36 2018 - [info]  Version check ok.
Fri Aug 24 02:00:36 2018 - [info] Checking SSH publickey authentication settings on the current master..
Fri Aug 24 02:00:36 2018 - [info] HealthCheck: SSH to 192.168.2.62 is reachable.
Fri Aug 24 02:00:37 2018 - [info] Master MHA Node version is 0.57.
Fri Aug 24 02:00:37 2018 - [info] Checking recovery script configurations on 192.168.2.62(192.168.2.62:3306)..
Fri Aug 24 02:00:37 2018 - [info]   Executing command: save_binary_logs --command=test --start_pos=4 --binlog_dir=/mysql1/data --output_file=/etc/mha/app1/save_binary_logs_test --manager_version=0.57 --start_file=mysql-bin.000010 
Fri Aug 24 02:00:37 2018 - [info]   Connecting to root@192.168.2.62(192.168.2.62:22).. 
  Creating /etc/mha/app1 if not exists..    ok.
  Checking output directory is accessible or not..
   ok.
  Binlog found at /mysql1/data, up to mysql-bin.000010
Fri Aug 24 02:00:38 2018 - [info] Binlog setting check done.
Fri Aug 24 02:00:38 2018 - [info] Checking SSH publickey authentication and checking recovery script configurations on all alive slave servers..
Fri Aug 24 02:00:38 2018 - [info]   Executing command : apply_diff_relay_logs --command=test --slave_user='root' --slave_host=192.168.2.61 --slave_ip=192.168.2.61 --slave_port=3306 --workdir=/etc/mha/app1 --target_version=5.7.21-log --manager_version=0.57 --relay_log_info=/mysql1/data/relay-log.info  --relay_dir=/mysql1/data/  --slave_pass=xxx
Fri Aug 24 02:00:38 2018 - [info]   Connecting to root@192.168.2.61(192.168.2.61:22).. 
  Checking slave recovery environment settings..
    Opening /mysql1/data/relay-log.info ... ok.
    Relay log found at /mysql1/data, up to node1-relay-bin.000002
    Temporary relay log file is /mysql1/data/node1-relay-bin.000002
    Testing mysql connection and privileges..mysql: [Warning] Using a password on the command line interface can be insecure.
 done.
    Testing mysqlbinlog output.. done.
    Cleaning up test file(s).. done.
Fri Aug 24 02:00:38 2018 - [info]   Executing command : apply_diff_relay_logs --command=test --slave_user='root' --slave_host=192.168.2.63 --slave_ip=192.168.2.63 --slave_port=3306 --workdir=/etc/mha/app1 --target_version=5.7.21-log --manager_version=0.57 --relay_log_info=/mysql1/data/relay-log.info  --relay_dir=/mysql1/data/  --slave_pass=xxx
Fri Aug 24 02:00:38 2018 - [info]   Connecting to root@192.168.2.63(192.168.2.63:22).. 
  Checking slave recovery environment settings..
    Opening /mysql1/data/relay-log.info ... ok.
    Relay log found at /mysql1/data, up to node3-relay-bin.000010
    Temporary relay log file is /mysql1/data/node3-relay-bin.000010
    Testing mysql connection and privileges..mysql: [Warning] Using a password on the command line interface can be insecure.
 done.
    Testing mysqlbinlog output.. done.
    Cleaning up test file(s).. done.
Fri Aug 24 02:00:38 2018 - [info] Slaves settings check done.
Fri Aug 24 02:00:38 2018 - [info] 
192.168.2.62(192.168.2.62:3306) (current master)
 +--192.168.2.61(192.168.2.61:3306)
 +--192.168.2.63(192.168.2.63:3306)

Fri Aug 24 02:00:38 2018 - [info] Checking replication health on 192.168.2.61..
Fri Aug 24 02:00:38 2018 - [info]  ok.
Fri Aug 24 02:00:38 2018 - [info] Checking replication health on 192.168.2.63..
Fri Aug 24 02:00:38 2018 - [info]  ok.
Fri Aug 24 02:00:38 2018 - [info] Checking master_ip_failover_script status:
Fri Aug 24 02:00:38 2018 - [info]   /etc/mha/master_ip_failover --command=status --ssh_user=root --orig_master_host=192.168.2.62 --orig_master_ip=192.168.2.62 --orig_master_port=3306 


IN SCRIPT TEST====/sbin/ifconfig eth0:0 down==/sbin/ifconfig eth0:0 192.168.6.66/24===

Checking the Status of the script.. OK 
Fri Aug 24 02:00:38 2018 - [info]  OK.
Fri Aug 24 02:00:38 2018 - [warning] shutdown_script is not defined.
Fri Aug 24 02:00:38 2018 - [info] Got exit code 0 (Not master dead).

MySQL Replication Health is OK.

```
### masterha_check_ssh        

```sql

[root@node3 app1]# /etc/mha/mhamanager/bin/masterha_check_ssh --conf=/etc/mha/app1/app1.cnf 
Thu Aug 23 15:14:27 2018 - [warning] Global configuration file /etc/masterha_default.cnf not found. Skipping.
Thu Aug 23 15:14:27 2018 - [info] Reading application default configuration from /etc/mha/app1/app1.cnf..
Thu Aug 23 15:14:27 2018 - [info] Reading server configuration from /etc/mha/app1/app1.cnf..
Thu Aug 23 15:14:27 2018 - [info] Starting SSH connection tests..
Thu Aug 23 15:14:29 2018 - [debug] 
Thu Aug 23 15:14:27 2018 - [debug]  Connecting via SSH from root@192.168.2.61(192.168.2.61:22) to root@192.168.2.62(192.168.2.62:22)..
Thu Aug 23 15:14:29 2018 - [debug]   ok.
Thu Aug 23 15:14:29 2018 - [debug]  Connecting via SSH from root@192.168.2.61(192.168.2.61:22) to root@192.168.2.63(192.168.2.63:22)..
Thu Aug 23 15:14:29 2018 - [debug]   ok.
Thu Aug 23 15:14:29 2018 - [debug] 
Thu Aug 23 15:14:28 2018 - [debug]  Connecting via SSH from root@192.168.2.63(192.168.2.63:22) to root@192.168.2.61(192.168.2.61:22)..
Thu Aug 23 15:14:29 2018 - [debug]   ok.
Thu Aug 23 15:14:29 2018 - [debug]  Connecting via SSH from root@192.168.2.63(192.168.2.63:22) to root@192.168.2.62(192.168.2.62:22)..
Thu Aug 23 15:14:29 2018 - [debug]   ok.
Thu Aug 23 15:14:29 2018 - [debug] 
Thu Aug 23 15:14:28 2018 - [debug]  Connecting via SSH from root@192.168.2.62(192.168.2.62:22) to root@192.168.2.61(192.168.2.61:22)..
Thu Aug 23 15:14:29 2018 - [debug]   ok.
Thu Aug 23 15:14:29 2018 - [debug]  Connecting via SSH from root@192.168.2.62(192.168.2.62:22) to root@192.168.2.63(192.168.2.63:22)..
Thu Aug 23 15:14:29 2018 - [debug]   ok.
Thu Aug 23 15:14:29 2018 - [info] All SSH connection tests passed successfully.

```
### masterha_check_status     

```sql
[root@node3 ~]# /etc/mha/mhamanager/bin/masterha_check_status  --conf=/etc/mha/app1/app1.cnf
app1 (pid:14158) is running(0:PING_OK), master:192.168.2.61


```
### masterha_conf_host         

```sql

```
### masterha_manager          

```sql

[root@node3 ~]# /etc/mha/mhamanager/bin/masterha_manager  --conf=/etc/mha/app1/app1.cnf &
[1] 14158
[root@node3 ~]# Fri Aug 24 17:49:08 2018 - [warning] Global configuration file /etc/masterha_default.cnf not found. Skipping.
Fri Aug 24 17:49:08 2018 - [info] Reading application default configuration from /etc/mha/app1/app1.cnf..
Fri Aug 24 17:49:08 2018 - [info] Reading server configuration from /etc/mha/app1/app1.cnf..

```
### masterha_master_monitor      

```sql
[root@node3 ~]# /etc/mha/mhamanager/bin/masterha_master_monitor  --conf=/etc/mha/app1/app1.cnf
Fri Aug 24 17:50:16 2018 - [warning] Global configuration file /etc/masterha_default.cnf not found. Skipping.
Fri Aug 24 17:50:16 2018 - [info] Reading application default configuration from /etc/mha/app1/app1.cnf..
Fri Aug 24 17:50:16 2018 - [info] Reading server configuration from /etc/mha/app1/app1.cnf..

```
### masterha_master_switch       

- 查看从库指向的主库

```sql
mysql> show slave status \G ;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.2.61
                  Master_User: dao
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000005
          Read_Master_Log_Pos: 154
               Relay_Log_File: node3-relay-bin.000002
                Relay_Log_Pos: 320
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
          Exec_Master_Log_Pos: 154
              Relay_Log_Space: 527
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

```

- 停止MHA Mointior

```sql

[root@node3 bin]# masterha_stop --conf=/etc/mha/app1/app1.cnf
Stopped app1 successfully.

```

- 切换主库

```sql

[root@node3 bin]#  masterha_master_switch --master_state=alive --conf=/etc/mha/app1/app1.cnf   --new_master_host=192.168.2.62
Fri Aug 24 18:04:39 2018 - [info] MHA::MasterRotate version 0.57.
Fri Aug 24 18:04:39 2018 - [info] Starting online master switch..
Fri Aug 24 18:04:39 2018 - [info]
Fri Aug 24 18:04:39 2018 - [info] * Phase 1: Configuration Check Phase..
Fri Aug 24 18:04:39 2018 - [info]
Fri Aug 24 18:04:39 2018 - [warning] Global configuration file /etc/masterha_default.cnf not found. Skipping.
Fri Aug 24 18:04:39 2018 - [info] Reading application default configuration from /etc/mha/app1/app1.cnf..
Fri Aug 24 18:04:39 2018 - [info] Reading server configuration from /etc/mha/app1/app1.cnf..
Fri Aug 24 18:04:39 2018 - [info] GTID failover mode = 0
Fri Aug 24 18:04:39 2018 - [info] Current Alive Master: 192.168.2.61(192.168.2.61:3306)
Fri Aug 24 18:04:39 2018 - [info] Alive Slaves:
Fri Aug 24 18:04:39 2018 - [info]   192.168.2.62(192.168.2.62:3306)  Version=5.7.21-log (oldest major version between slaves) log-bin:enabled
Fri Aug 24 18:04:39 2018 - [info]     Replicating from node1(192.168.2.61:3306)
Fri Aug 24 18:04:39 2018 - [info]     Primary candidate for the new Master (candidate_master is set)
Fri Aug 24 18:04:39 2018 - [info]   192.168.2.63(192.168.2.63:3306)  Version=5.7.21-log (oldest major version between slaves) log-bin:enabled
Fri Aug 24 18:04:39 2018 - [info]     Replicating from 192.168.2.61(192.168.2.61:3306)
Fri Aug 24 18:04:39 2018 - [info]     Not candidate for the new Master (no_master is set)

It is better to execute FLUSH NO_WRITE_TO_BINLOG TABLES on the master before switching. Is it ok to execute on 192.168.2.61(192.168.2.61:3306)? (YES/no): yes
Fri Aug 24 18:04:42 2018 - [info] Executing FLUSH NO_WRITE_TO_BINLOG TABLES. This may take long time..
Fri Aug 24 18:04:42 2018 - [info]  ok.
Fri Aug 24 18:04:42 2018 - [info] Checking MHA is not monitoring or doing failover..
Fri Aug 24 18:04:42 2018 - [info] Checking replication health on 192.168.2.62..
Fri Aug 24 18:04:42 2018 - [info]  ok.
Fri Aug 24 18:04:42 2018 - [info] Checking replication health on 192.168.2.63..
Fri Aug 24 18:04:42 2018 - [info]  ok.
Fri Aug 24 18:04:42 2018 - [info] 192.168.2.62 can be new master.
Fri Aug 24 18:04:42 2018 - [info]
From:
192.168.2.61(192.168.2.61:3306) (current master)
 +--192.168.2.62(192.168.2.62:3306)
 +--192.168.2.63(192.168.2.63:3306)

To:
192.168.2.62(192.168.2.62:3306) (new master)
 +--192.168.2.63(192.168.2.63:3306)

Starting master switch from 192.168.2.61(192.168.2.61:3306) to 192.168.2.62(192.168.2.62:3306)? (yes/NO): yes
Fri Aug 24 18:04:45 2018 - [info] Checking whether 192.168.2.62(192.168.2.62:3306) is ok for the new master..
Fri Aug 24 18:04:45 2018 - [info]  ok.
Fri Aug 24 18:04:45 2018 - [info] ** Phase 1: Configuration Check Phase completed.
Fri Aug 24 18:04:45 2018 - [info]
Fri Aug 24 18:04:45 2018 - [info] * Phase 2: Rejecting updates Phase..
Fri Aug 24 18:04:45 2018 - [info]
Fri Aug 24 18:04:45 2018 - [info] Executing master ip online change script to disable write on the current master:
Fri Aug 24 18:04:45 2018 - [info]   /etc/mha/master_ip_online_change --command=stop --orig_master_host=192.168.2.61 --orig_master_ip=192.168.2.61 --orig_master_port=3306 --orig_master_user='root' --new_master_host=192.168.2.62 --new_master_ip=192.168.2.62 --new_master_port=3306 --new_master_user='root' --orig_master_ssh_user=root --new_master_ssh_user=root   --orig_master_password=xxx --new_master_password=xxx



***************************************************************
Disabling the VIP - 192.168.6.66/24 on old master: 192.168.2.61
***************************************************************



Fri Aug 24 18:04:46 2018 - [info]  ok.
Fri Aug 24 18:04:46 2018 - [info] Locking all tables on the orig master to reject updates from everybody (including root):
Fri Aug 24 18:04:46 2018 - [info] Executing FLUSH TABLES WITH READ LOCK..
Fri Aug 24 18:04:46 2018 - [info]  ok.
Fri Aug 24 18:04:46 2018 - [info] Orig master binlog:pos is mysql-bin.000005:154.
Fri Aug 24 18:04:46 2018 - [info]  Waiting to execute all relay logs on 192.168.2.62(192.168.2.62:3306)..
Fri Aug 24 18:04:46 2018 - [info]  master_pos_wait(mysql-bin.000005:154) completed on 192.168.2.62(192.168.2.62:3306). Executed 0 events.
Fri Aug 24 18:04:46 2018 - [info]   done.
Fri Aug 24 18:04:46 2018 - [info] Getting new master's binlog name and position..
Fri Aug 24 18:04:46 2018 - [info]  mysql-bin.000013:234
Fri Aug 24 18:04:46 2018 - [info]  All other slaves should start replication from here. Statement should be: CHANGE MASTER TO MASTER_HOST='192.168.2.62', MASTER_PORT=3306, MASTER_LOG_FILE='mysql-bin.000013', MASTER_LOG_POS=234, MASTER_USER='dao', MASTER_PASSWORD='xxx';
Fri Aug 24 18:04:46 2018 - [info] Executing master ip online change script to allow write on the new master:
Fri Aug 24 18:04:46 2018 - [info]   /etc/mha/master_ip_online_change --command=start --orig_master_host=192.168.2.61 --orig_master_ip=192.168.2.61 --orig_master_port=3306 --orig_master_user='root' --new_master_host=192.168.2.62 --new_master_ip=192.168.2.62 --new_master_port=3306 --new_master_user='root' --orig_master_ssh_user=root --new_master_ssh_user=root   --orig_master_password=xxx --new_master_password=xxx



***************************************************************
Enabling the VIP - 192.168.6.66/24 on new master: 192.168.2.62
***************************************************************



Fri Aug 24 18:04:46 2018 - [info]  ok.
Fri Aug 24 18:04:46 2018 - [info]
Fri Aug 24 18:04:46 2018 - [info] * Switching slaves in parallel..
Fri Aug 24 18:04:46 2018 - [info]
Fri Aug 24 18:04:46 2018 - [info] -- Slave switch on host 192.168.2.63(192.168.2.63:3306) started, pid: 15240
Fri Aug 24 18:04:46 2018 - [info]
Fri Aug 24 18:04:46 2018 - [info] Log messages from 192.168.2.63 ...
Fri Aug 24 18:04:46 2018 - [info]
Fri Aug 24 18:04:46 2018 - [info]  Waiting to execute all relay logs on 192.168.2.63(192.168.2.63:3306)..
Fri Aug 24 18:04:46 2018 - [info]  master_pos_wait(mysql-bin.000005:154) completed on 192.168.2.63(192.168.2.63:3306). Executed 0 events.
Fri Aug 24 18:04:46 2018 - [info]   done.
Fri Aug 24 18:04:46 2018 - [info]  Resetting slave 192.168.2.63(192.168.2.63:3306) and starting replication from the new master 192.168.2.62(192.168.2.62:3306)..
Fri Aug 24 18:04:46 2018 - [info]  Executed CHANGE MASTER.
Fri Aug 24 18:04:46 2018 - [info]  Slave started.
Fri Aug 24 18:04:46 2018 - [info] End of log messages from 192.168.2.63 ...
Fri Aug 24 18:04:46 2018 - [info]
Fri Aug 24 18:04:46 2018 - [info] -- Slave switch on host 192.168.2.63(192.168.2.63:3306) succeeded.
Fri Aug 24 18:04:46 2018 - [info] Unlocking all tables on the orig master:
Fri Aug 24 18:04:46 2018 - [info] Executing UNLOCK TABLES..
Fri Aug 24 18:04:46 2018 - [info]  ok.
Fri Aug 24 18:04:46 2018 - [info] All new slave servers switched successfully.
Fri Aug 24 18:04:46 2018 - [info]
Fri Aug 24 18:04:46 2018 - [info] * Phase 5: New master cleanup phase..
Fri Aug 24 18:04:46 2018 - [info]
Fri Aug 24 18:04:46 2018 - [info]  192.168.2.62: Resetting slave info succeeded.
Fri Aug 24 18:04:46 2018 - [info] Switching master to 192.168.2.62(192.168.2.62:3306) completed successfully.
[root@node3 bin]#


```

- 再次查看从库指向的主库

```sql



mysql> show slave status \G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.2.62
                  Master_User: dao
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000013
          Read_Master_Log_Pos: 234
               Relay_Log_File: node3-relay-bin.000002
                Relay_Log_Pos: 320
        Relay_Master_Log_File: mysql-bin.000013
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
          Exec_Master_Log_Pos: 234
              Relay_Log_Space: 527
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
             Master_Server_Id: 2
                  Master_UUID: f63b765b-9c1d-11e8-aa1e-005056296858
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
1 row in set (0.11 sec)

```
- 验证IP是否漂移

```sql


[root@node2 ~]# ifconfig
eth0      Link encap:Ethernet  HWaddr 00:50:56:29:68:58
          inet addr:192.168.2.62  Bcast:192.168.2.255  Mask:255.255.255.0
          inet6 addr: fe80::250:56ff:fe29:6858/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:16163 errors:0 dropped:0 overruns:0 frame:0
          TX packets:13587 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:2744029 (2.6 MiB)  TX bytes:2051279 (1.9 MiB)

eth0:0    Link encap:Ethernet  HWaddr 00:50:56:29:68:58
          inet addr:192.168.6.66  Bcast:192.168.6.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:16436  Metric:1
          RX packets:1814 errors:0 dropped:0 overruns:0 frame:0
          TX packets:1814 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:471431 (460.3 KiB)  TX bytes:471431 (460.3 KiB)


```

### masterha_secondary_check      

```sql

[root@node3 ~]# masterha_secondary_check -s 192.168.2.62 -s 192.168.2.63 --user=root --master_host=node1 --master_ip=192.168.2.61 --master_port=3306
Master is reachable from 192.168.2.62!


```
### masterha_stop                           

```sql

[root@node3 bin]# masterha_stop --conf=/etc/mha/app1/app1.cnf
Stopped app1 successfully.


```