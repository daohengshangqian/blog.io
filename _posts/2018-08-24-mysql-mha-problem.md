---
layout: post
title: MySQL MHA常见问题汇总
date: 2018-08-24
categories: blog
tags: [MySQL]
description: MySQL MHA常见问题汇总
---

## mysql command failed with rc 127:0!

### 问题表现

```sql

Fri Aug 24 11:20:37 2018 - [info]   Connecting to root@192.168.2.63(192.168.2.63:22)..
  Checking slave recovery environment settings..
    Opening /mysql1/data/relay-log.info ... ok.
    Relay log found at /mysql1/data, up to node3-relay-bin.000002
    Temporary relay log file is /mysql1/data/node3-relay-bin.000002
    Testing mysql connection and privileges..sh: mysql: command not found
mysql command failed with rc 127:0!
 at /usr/local/bin/apply_diff_relay_logs line 375
        main::check() called at /usr/local/bin/apply_diff_relay_logs line 497
        eval {...} called at /usr/local/bin/apply_diff_relay_logs line 475
        main::main() called at /usr/local/bin/apply_diff_relay_logs line 120
Fri Aug 24 11:20:37 2018 - [error][/usr/local/share/perl5/MHA/MasterMonitor.pm, ln208] Slaves settings check failed!
Fri Aug 24 11:20:37 2018 - [error][/usr/local/share/perl5/MHA/MasterMonitor.pm, ln416] Slave configuration failed.
Fri Aug 24 11:20:37 2018 - [error][/usr/local/share/perl5/MHA/MasterMonitor.pm, ln427] Error happened on checking configurations.  at /etc/mha/mhamanager/bin/masterha_check_repl line 48
Fri Aug 24 11:20:37 2018 - [error][/usr/local/share/perl5/MHA/MasterMonitor.pm, ln525] Error happened on monitoring servers.
Fri Aug 24 11:20:37 2018 - [info] Got exit code 1 (Not master dead).

MySQL Replication Health is NOT OK!


```

### 解决办法

```sql

[root@node3 ~]#  ln -s /usr/local/mysql/bin/mysql /usr/bin/mysql

```

### 验证

```sql


[root@node3 bin]# /etc/mha/mhamanager/bin/masterha_check_repl --conf=/etc/mha/app1/app1.cnf
Fri Aug 24 11:23:29 2018 - [warning] Global configuration file /etc/masterha_default.cnf not found. Skipping.
Fri Aug 24 11:23:29 2018 - [info] Reading application default configuration from /etc/mha/app1/app1.cnf..
Fri Aug 24 11:23:29 2018 - [info] Reading server configuration from /etc/mha/app1/app1.cnf..
Fri Aug 24 11:23:29 2018 - [info] MHA::MasterMonitor version 0.57.
Fri Aug 24 11:23:30 2018 - [info] GTID failover mode = 0
Fri Aug 24 11:23:30 2018 - [info] Dead Servers:
Fri Aug 24 11:23:30 2018 - [info] Alive Servers:
Fri Aug 24 11:23:30 2018 - [info]   192.168.2.61(192.168.2.61:3306)
Fri Aug 24 11:23:30 2018 - [info]   192.168.2.62(192.168.2.62:3306)
Fri Aug 24 11:23:30 2018 - [info]   192.168.2.63(192.168.2.63:3306)
Fri Aug 24 11:23:30 2018 - [info] Alive Slaves:
Fri Aug 24 11:23:30 2018 - [info]   192.168.2.61(192.168.2.61:3306)  Version=5.7.21-log (oldest major version between slaves) log-bin:enabled
Fri Aug 24 11:23:30 2018 - [info]     Replicating from 192.168.2.62(192.168.2.62:3306)
Fri Aug 24 11:23:30 2018 - [info]     Primary candidate for the new Master (candidate_master is set)
Fri Aug 24 11:23:30 2018 - [info]   192.168.2.63(192.168.2.63:3306)  Version=5.7.21-log (oldest major version between slaves) log-bin:enabled
Fri Aug 24 11:23:30 2018 - [info]     Replicating from 192.168.2.62(192.168.2.62:3306)
Fri Aug 24 11:23:30 2018 - [info]     Not candidate for the new Master (no_master is set)
Fri Aug 24 11:23:30 2018 - [info] Current Alive Master: 192.168.2.62(192.168.2.62:3306)
Fri Aug 24 11:23:30 2018 - [info] Checking slave configurations..
Fri Aug 24 11:23:30 2018 - [info]  read_only=1 is not set on slave 192.168.2.61(192.168.2.61:3306).
Fri Aug 24 11:23:30 2018 - [warning]  relay_log_purge=0 is not set on slave 192.168.2.61(192.168.2.61:3306).
Fri Aug 24 11:23:30 2018 - [info]  read_only=1 is not set on slave 192.168.2.63(192.168.2.63:3306).
Fri Aug 24 11:23:30 2018 - [warning]  relay_log_purge=0 is not set on slave 192.168.2.63(192.168.2.63:3306).
Fri Aug 24 11:23:30 2018 - [info] Checking replication filtering settings..
Fri Aug 24 11:23:30 2018 - [info]  binlog_do_db= , binlog_ignore_db=
Fri Aug 24 11:23:30 2018 - [info]  Replication filtering check ok.
Fri Aug 24 11:23:30 2018 - [info] GTID (with auto-pos) is not supported
Fri Aug 24 11:23:30 2018 - [info] Starting SSH connection tests..
Fri Aug 24 11:23:31 2018 - [info] All SSH connection tests passed successfully.
Fri Aug 24 11:23:31 2018 - [info] Checking MHA Node version..
Fri Aug 24 11:23:32 2018 - [info]  Version check ok.
Fri Aug 24 11:23:32 2018 - [info] Checking SSH publickey authentication settings on the current master..
Fri Aug 24 11:23:32 2018 - [info] HealthCheck: SSH to 192.168.2.62 is reachable.
Fri Aug 24 11:23:32 2018 - [info] Master MHA Node version is 0.57.
Fri Aug 24 11:23:32 2018 - [info] Checking recovery script configurations on 192.168.2.62(192.168.2.62:3306)..
Fri Aug 24 11:23:32 2018 - [info]   Executing command: save_binary_logs --command=test --start_pos=4 --binlog_dir=/mysql1/data --output_file=/etc/mha/app1/save_binary_logs_test --manager_version=0.57 --start_file=mysql-bin.000012
Fri Aug 24 11:23:32 2018 - [info]   Connecting to root@192.168.2.62(192.168.2.62:22)..
  Creating /etc/mha/app1 if not exists..    ok.
  Checking output directory is accessible or not..
   ok.
  Binlog found at /mysql1/data, up to mysql-bin.000012
Fri Aug 24 11:23:33 2018 - [info] Binlog setting check done.
Fri Aug 24 11:23:33 2018 - [info] Checking SSH publickey authentication and checking recovery script configurations on all alive slave servers..
Fri Aug 24 11:23:33 2018 - [info]   Executing command : apply_diff_relay_logs --command=test --slave_user='root' --slave_host=192.168.2.61 --slave_ip=192.168.2.61 --slave_port=3306 --workdir=/etc/mha/app1 --target_version=5.7.21-log --manager_version=0.57 --relay_log_info=/mysql1/data/relay-log.info  --relay_dir=/mysql1/data/  --slave_pass=xxx
Fri Aug 24 11:23:33 2018 - [info]   Connecting to root@192.168.2.61(192.168.2.61:22)..
  Checking slave recovery environment settings..
    Opening /mysql1/data/relay-log.info ... ok.
    Relay log found at /mysql1/data, up to node1-relay-bin.000002
    Temporary relay log file is /mysql1/data/node1-relay-bin.000002
    Testing mysql connection and privileges..mysql: [Warning] Using a password on the command line interface can be insecure.
 done.
    Testing mysqlbinlog output.. done.
    Cleaning up test file(s).. done.
Fri Aug 24 11:23:33 2018 - [info]   Executing command : apply_diff_relay_logs --command=test --slave_user='root' --slave_host=192.168.2.63 --slave_ip=192.168.2.63 --slave_port=3306 --workdir=/etc/mha/app1 --target_version=5.7.21-log --manager_version=0.57 --relay_log_info=/mysql1/data/relay-log.info  --relay_dir=/mysql1/data/  --slave_pass=xxx
Fri Aug 24 11:23:33 2018 - [info]   Connecting to root@192.168.2.63(192.168.2.63:22)..
  Checking slave recovery environment settings..
    Opening /mysql1/data/relay-log.info ... ok.
    Relay log found at /mysql1/data, up to node3-relay-bin.000002
    Temporary relay log file is /mysql1/data/node3-relay-bin.000002
    Testing mysql connection and privileges..mysql: [Warning] Using a password on the command line interface can be insecure.
 done.
    Testing mysqlbinlog output.. done.
    Cleaning up test file(s).. done.
Fri Aug 24 11:23:33 2018 - [info] Slaves settings check done.
Fri Aug 24 11:23:33 2018 - [info]
192.168.2.62(192.168.2.62:3306) (current master)
 +--192.168.2.61(192.168.2.61:3306)
 +--192.168.2.63(192.168.2.63:3306)

Fri Aug 24 11:23:33 2018 - [info] Checking replication health on 192.168.2.61..
Fri Aug 24 11:23:33 2018 - [info]  ok.
Fri Aug 24 11:23:33 2018 - [info] Checking replication health on 192.168.2.63..
Fri Aug 24 11:23:33 2018 - [info]  ok.
Fri Aug 24 11:23:33 2018 - [info] Checking master_ip_failover_script status:
Fri Aug 24 11:23:33 2018 - [info]   /etc/mha/master_ip_failover --command=status --ssh_user=root --orig_master_host=192.168.2.62 --orig_master_ip=192.168.2.62 --orig_master_port=3306


IN SCRIPT TEST====/sbin/ifconfig eth0:0 down==/sbin/ifconfig eth0:0 192.168.6.66/24===

Checking the Status of the script.. OK
Fri Aug 24 11:23:33 2018 - [info]  OK.
Fri Aug 24 11:23:33 2018 - [warning] shutdown_script is not defined.
Fri Aug 24 11:23:33 2018 - [info] Got exit code 0 (Not master dead).

MySQL Replication Health is OK.


```


## Can't exec "mysqlbinlog": No such file or directory at

### 问题表现

```sql

Fri Aug 24 11:24:25 2018 - [info]   Connecting to root@192.168.2.63(192.168.2.63:22)..
Can't exec "mysqlbinlog": No such file or directory at /usr/local/share/perl5/MHA/BinlogManager.pm line 106.
mysqlbinlog version command failed with rc 1:0, please verify PATH, LD_LIBRARY_PATH, and client options
 at /usr/local/bin/apply_diff_relay_logs line 493
Fri Aug 24 11:24:25 2018 - [error][/usr/local/share/perl5/MHA/MasterMonitor.pm, ln208] Slaves settings check failed!
Fri Aug 24 11:24:25 2018 - [error][/usr/local/share/perl5/MHA/MasterMonitor.pm, ln416] Slave configuration failed.
Fri Aug 24 11:24:25 2018 - [error][/usr/local/share/perl5/MHA/MasterMonitor.pm, ln427] Error happened on checking configurations.  at /etc/mha/mhamanager/bin/masterha_check_repl line 48
Fri Aug 24 11:24:25 2018 - [error][/usr/local/share/perl5/MHA/MasterMonitor.pm, ln525] Error happened on monitoring servers.
Fri Aug 24 11:24:25 2018 - [info] Got exit code 1 (Not master dead).

MySQL Replication Health is NOT OK!


```


### 解决方法

```sql


[root@node3 ~]# ln -s /usr/local/mysql/bin/mysqlbinlog  /usr/bin/mysqlbinlog

```

### 问题验证

```sql

[root@node3 bin]# /etc/mha/mhamanager/bin/masterha_check_repl --conf=/etc/mha/app1/app1.cnf
Fri Aug 24 11:27:28 2018 - [warning] Global configuration file /etc/masterha_default.cnf not found. Skipping.
Fri Aug 24 11:27:28 2018 - [info] Reading application default configuration from /etc/mha/app1/app1.cnf..
Fri Aug 24 11:27:28 2018 - [info] Reading server configuration from /etc/mha/app1/app1.cnf..
Fri Aug 24 11:27:28 2018 - [info] MHA::MasterMonitor version 0.57.
Fri Aug 24 11:27:28 2018 - [info] GTID failover mode = 0
Fri Aug 24 11:27:28 2018 - [info] Dead Servers:
Fri Aug 24 11:27:28 2018 - [info] Alive Servers:
Fri Aug 24 11:27:28 2018 - [info]   192.168.2.61(192.168.2.61:3306)
Fri Aug 24 11:27:28 2018 - [info]   192.168.2.62(192.168.2.62:3306)
Fri Aug 24 11:27:28 2018 - [info]   192.168.2.63(192.168.2.63:3306)
Fri Aug 24 11:27:28 2018 - [info] Alive Slaves:
Fri Aug 24 11:27:28 2018 - [info]   192.168.2.61(192.168.2.61:3306)  Version=5.7.21-log (oldest major version between slaves) log-bin:enabled
Fri Aug 24 11:27:28 2018 - [info]     Replicating from 192.168.2.62(192.168.2.62:3306)
Fri Aug 24 11:27:28 2018 - [info]     Primary candidate for the new Master (candidate_master is set)
Fri Aug 24 11:27:28 2018 - [info]   192.168.2.63(192.168.2.63:3306)  Version=5.7.21-log (oldest major version between slaves) log-bin:enabled
Fri Aug 24 11:27:28 2018 - [info]     Replicating from 192.168.2.62(192.168.2.62:3306)
Fri Aug 24 11:27:28 2018 - [info]     Not candidate for the new Master (no_master is set)
Fri Aug 24 11:27:28 2018 - [info] Current Alive Master: 192.168.2.62(192.168.2.62:3306)
Fri Aug 24 11:27:28 2018 - [info] Checking slave configurations..
Fri Aug 24 11:27:28 2018 - [info]  read_only=1 is not set on slave 192.168.2.61(192.168.2.61:3306).
Fri Aug 24 11:27:28 2018 - [warning]  relay_log_purge=0 is not set on slave 192.168.2.61(192.168.2.61:3306).
Fri Aug 24 11:27:28 2018 - [info]  read_only=1 is not set on slave 192.168.2.63(192.168.2.63:3306).
Fri Aug 24 11:27:28 2018 - [warning]  relay_log_purge=0 is not set on slave 192.168.2.63(192.168.2.63:3306).
Fri Aug 24 11:27:28 2018 - [info] Checking replication filtering settings..
Fri Aug 24 11:27:28 2018 - [info]  binlog_do_db= , binlog_ignore_db=
Fri Aug 24 11:27:28 2018 - [info]  Replication filtering check ok.
Fri Aug 24 11:27:28 2018 - [info] GTID (with auto-pos) is not supported
Fri Aug 24 11:27:28 2018 - [info] Starting SSH connection tests..
Fri Aug 24 11:27:30 2018 - [info] All SSH connection tests passed successfully.
Fri Aug 24 11:27:30 2018 - [info] Checking MHA Node version..
Fri Aug 24 11:27:31 2018 - [info]  Version check ok.
Fri Aug 24 11:27:31 2018 - [info] Checking SSH publickey authentication settings on the current master..
Fri Aug 24 11:27:31 2018 - [info] HealthCheck: SSH to 192.168.2.62 is reachable.
Fri Aug 24 11:27:31 2018 - [info] Master MHA Node version is 0.57.
Fri Aug 24 11:27:31 2018 - [info] Checking recovery script configurations on 192.168.2.62(192.168.2.62:3306)..
Fri Aug 24 11:27:31 2018 - [info]   Executing command: save_binary_logs --command=test --start_pos=4 --binlog_dir=/mysql1/data --output_file=/etc/mha/app1/save_binary_logs_test --manager_version=0.57 --start_file=mysql-bin.000012
Fri Aug 24 11:27:31 2018 - [info]   Connecting to root@192.168.2.62(192.168.2.62:22)..
  Creating /etc/mha/app1 if not exists..    ok.
  Checking output directory is accessible or not..
   ok.
  Binlog found at /mysql1/data, up to mysql-bin.000012
Fri Aug 24 11:27:32 2018 - [info] Binlog setting check done.
Fri Aug 24 11:27:32 2018 - [info] Checking SSH publickey authentication and checking recovery script configurations on all alive slave servers..
Fri Aug 24 11:27:32 2018 - [info]   Executing command : apply_diff_relay_logs --command=test --slave_user='root' --slave_host=192.168.2.61 --slave_ip=192.168.2.61 --slave_port=3306 --workdir=/etc/mha/app1 --target_version=5.7.21-log --manager_version=0.57 --relay_log_info=/mysql1/data/relay-log.info  --relay_dir=/mysql1/data/  --slave_pass=xxx
Fri Aug 24 11:27:32 2018 - [info]   Connecting to root@192.168.2.61(192.168.2.61:22)..
  Checking slave recovery environment settings..
    Opening /mysql1/data/relay-log.info ... ok.
    Relay log found at /mysql1/data, up to node1-relay-bin.000002
    Temporary relay log file is /mysql1/data/node1-relay-bin.000002
    Testing mysql connection and privileges..mysql: [Warning] Using a password on the command line interface can be insecure.
 done.
    Testing mysqlbinlog output.. done.
    Cleaning up test file(s).. done.
Fri Aug 24 11:27:32 2018 - [info]   Executing command : apply_diff_relay_logs --command=test --slave_user='root' --slave_host=192.168.2.63 --slave_ip=192.168.2.63 --slave_port=3306 --workdir=/etc/mha/app1 --target_version=5.7.21-log --manager_version=0.57 --relay_log_info=/mysql1/data/relay-log.info  --relay_dir=/mysql1/data/  --slave_pass=xxx
Fri Aug 24 11:27:32 2018 - [info]   Connecting to root@192.168.2.63(192.168.2.63:22)..
  Checking slave recovery environment settings..
    Opening /mysql1/data/relay-log.info ... ok.
    Relay log found at /mysql1/data, up to node3-relay-bin.000002
    Temporary relay log file is /mysql1/data/node3-relay-bin.000002
    Testing mysql connection and privileges..mysql: [Warning] Using a password on the command line interface can be insecure.
 done.
    Testing mysqlbinlog output.. done.
    Cleaning up test file(s).. done.
Fri Aug 24 11:27:32 2018 - [info] Slaves settings check done.
Fri Aug 24 11:27:32 2018 - [info]
192.168.2.62(192.168.2.62:3306) (current master)
 +--192.168.2.61(192.168.2.61:3306)
 +--192.168.2.63(192.168.2.63:3306)

Fri Aug 24 11:27:32 2018 - [info] Checking replication health on 192.168.2.61..
Fri Aug 24 11:27:32 2018 - [info]  ok.
Fri Aug 24 11:27:32 2018 - [info] Checking replication health on 192.168.2.63..
Fri Aug 24 11:27:32 2018 - [info]  ok.
Fri Aug 24 11:27:32 2018 - [info] Checking master_ip_failover_script status:
Fri Aug 24 11:27:32 2018 - [info]   /etc/mha/master_ip_failover --command=status --ssh_user=root --orig_master_host=192.168.2.62 --orig_master_ip=192.168.2.62 --orig_master_port=3306


IN SCRIPT TEST====/sbin/ifconfig eth0:0 down==/sbin/ifconfig eth0:0 192.168.6.66/24===

Checking the Status of the script.. OK
Fri Aug 24 11:27:32 2018 - [info]  OK.
Fri Aug 24 11:27:32 2018 - [warning] shutdown_script is not defined.
Fri Aug 24 11:27:32 2018 - [info] Got exit code 0 (Not master dead).

MySQL Replication Health is OK.


```

##  User dao1 does not exist or does not have REPLICATION SLAVE privilege! 

### 问题表现

```sql

Fri Aug 24 11:34:15 2018 - [error][/usr/local/share/perl5/MHA/Server.pm, ln398] 192.168.2.61(192.168.2.61:3306): User dao1 does not exist or does not have REPLICATION SLAVE privilege! Other slaves can not start replication from this host.
Fri Aug 24 11:34:15 2018 - [error][/usr/local/share/perl5/MHA/MasterMonitor.pm, ln427] Error happened on checking configurations.  at /usr/local/share/perl5/MHA/ServerManager.pm line 1403
Fri Aug 24 11:34:15 2018 - [error][/usr/local/share/perl5/MHA/MasterMonitor.pm, ln525] Error happened on monitoring servers.
Fri Aug 24 11:34:15 2018 - [info] Got exit code 1 (Not master dead).

MySQL Replication Health is NOT OK!


```
###   解决方法

- 修改配置文件中的账户正确账户 或创建复制用户

修改配置文件

```sql

[root@node3 ~]# vi /etc/mha/app1/app1.cnf

[server default]
# working directory on the manager
manager_workdir=/etc/mha/app1
manager_log=/etc/mha/app1/manager.log
remote_workdir=/etc/mha/app1
# In case of automatic file-over, the following script will be used
master_ip_failover_script=/etc/mha/master_ip_failover
master_ip_online_change_script=/etc/mha/master_ip_online_change

#mha连接到mysql的帐号
user=root
password=root

#mysql主从复制架构的复制用户(此处账户出错应为dao !!! )
repl_user=dao1   
repl_password=dao


ssh_user=root

#心跳
ping_interval=1


secondary_check_script= masterha_secondary_check -s 192.168.2.62 -s 192.168.2.63

[server1]
hostname=192.168.2.61
port=3306
master_binlog_dir="/mysql1/data"
candidate_master=1
ssh_port=22
[server2]
hostname=192.168.2.62
port=3306
master_binlog_dir="/mysql1/data"
candidate_master=1
ssh_port=22
[server3]
hostname=192.168.2.63
port=3306
master_binlog_dir="/mysql1/data"
no_master=1
ssh_port=22


```

- 或创建复制用户

```sql

GRANT ALL PRIVILEGES ON *.* TO 'dao'@'192.168.2.%' IDENTIFIED BY 'dao' ;

```


### 问题验证

-本例中修改了配置文件中的复制用户

```sql

[root@node3 ~]# vi /etc/mha/app1/app1.cnf

[server default]
# working directory on the manager
manager_workdir=/etc/mha/app1
manager_log=/etc/mha/app1/manager.log
remote_workdir=/etc/mha/app1
# In case of automatic file-over, the following script will be used
master_ip_failover_script=/etc/mha/master_ip_failover
master_ip_online_change_script=/etc/mha/master_ip_online_change

#mha连接到mysql的帐号
user=root
password=root

#mysql主从复制架构的复制用户
repl_user=dao
repl_password=dao


ssh_user=root

#心跳
ping_interval=1


secondary_check_script= masterha_secondary_check -s 192.168.2.62 -s 192.168.2.63

[server1]
hostname=192.168.2.61
port=3306
master_binlog_dir="/mysql1/data"
candidate_master=1
ssh_port=22
[server2]
hostname=192.168.2.62
port=3306
master_binlog_dir="/mysql1/data"
candidate_master=1
ssh_port=22
[server3]
hostname=192.168.2.63
port=3306
master_binlog_dir="/mysql1/data"
no_master=1
ssh_port=22




```

```sql

[root@node3 bin]# /etc/mha/mhamanager/bin/masterha_check_repl --conf=/etc/mha/app1/app1.cnf
Fri Aug 24 12:06:26 2018 - [warning] Global configuration file /etc/masterha_default.cnf not found. Skipping.
Fri Aug 24 12:06:26 2018 - [info] Reading application default configuration from /etc/mha/app1/app1.cnf..
Fri Aug 24 12:06:26 2018 - [info] Reading server configuration from /etc/mha/app1/app1.cnf..
Fri Aug 24 12:06:26 2018 - [info] MHA::MasterMonitor version 0.57.
Fri Aug 24 12:06:27 2018 - [info] GTID failover mode = 0
Fri Aug 24 12:06:27 2018 - [info] Dead Servers:
Fri Aug 24 12:06:27 2018 - [info] Alive Servers:
Fri Aug 24 12:06:27 2018 - [info]   192.168.2.61(192.168.2.61:3306)
Fri Aug 24 12:06:27 2018 - [info]   192.168.2.62(192.168.2.62:3306)
Fri Aug 24 12:06:27 2018 - [info]   192.168.2.63(192.168.2.63:3306)
Fri Aug 24 12:06:27 2018 - [info] Alive Slaves:
Fri Aug 24 12:06:27 2018 - [info]   192.168.2.61(192.168.2.61:3306)  Version=5.7.21-log (oldest major version between slaves) log-bin:enabled
Fri Aug 24 12:06:27 2018 - [info]     Replicating from 192.168.2.62(192.168.2.62:3306)
Fri Aug 24 12:06:27 2018 - [info]     Primary candidate for the new Master (candidate_master is set)
Fri Aug 24 12:06:27 2018 - [info]   192.168.2.63(192.168.2.63:3306)  Version=5.7.21-log (oldest major version between slaves) log-bin:enabled
Fri Aug 24 12:06:27 2018 - [info]     Replicating from 192.168.2.62(192.168.2.62:3306)
Fri Aug 24 12:06:27 2018 - [info]     Not candidate for the new Master (no_master is set)
Fri Aug 24 12:06:27 2018 - [info] Current Alive Master: 192.168.2.62(192.168.2.62:3306)
Fri Aug 24 12:06:27 2018 - [info] Checking slave configurations..
Fri Aug 24 12:06:27 2018 - [info]  read_only=1 is not set on slave 192.168.2.61(192.168.2.61:3306).
Fri Aug 24 12:06:27 2018 - [warning]  relay_log_purge=0 is not set on slave 192.168.2.61(192.168.2.61:3306).
Fri Aug 24 12:06:27 2018 - [info]  read_only=1 is not set on slave 192.168.2.63(192.168.2.63:3306).
Fri Aug 24 12:06:27 2018 - [warning]  relay_log_purge=0 is not set on slave 192.168.2.63(192.168.2.63:3306).
Fri Aug 24 12:06:27 2018 - [info] Checking replication filtering settings..
Fri Aug 24 12:06:27 2018 - [info]  binlog_do_db= , binlog_ignore_db=
Fri Aug 24 12:06:27 2018 - [info]  Replication filtering check ok.
Fri Aug 24 12:06:27 2018 - [info] GTID (with auto-pos) is not supported
Fri Aug 24 12:06:27 2018 - [info] Starting SSH connection tests..
Fri Aug 24 12:06:28 2018 - [info] All SSH connection tests passed successfully.
Fri Aug 24 12:06:28 2018 - [info] Checking MHA Node version..
Fri Aug 24 12:06:29 2018 - [info]  Version check ok.
Fri Aug 24 12:06:29 2018 - [info] Checking SSH publickey authentication settings on the current master..
Fri Aug 24 12:06:29 2018 - [info] HealthCheck: SSH to 192.168.2.62 is reachable.
Fri Aug 24 12:06:29 2018 - [info] Master MHA Node version is 0.57.
Fri Aug 24 12:06:29 2018 - [info] Checking recovery script configurations on 192.168.2.62(192.168.2.62:3306)..
Fri Aug 24 12:06:29 2018 - [info]   Executing command: save_binary_logs --command=test --start_pos=4 --binlog_dir=/mysql1/data --output_file=/etc/mha/app1/save_binary_logs_test --manager_version=0.57 --start_file=mysql-bin.000012
Fri Aug 24 12:06:29 2018 - [info]   Connecting to root@192.168.2.62(192.168.2.62:22)..
  Creating /etc/mha/app1 if not exists..    ok.
  Checking output directory is accessible or not..
   ok.
  Binlog found at /mysql1/data, up to mysql-bin.000012
Fri Aug 24 12:06:30 2018 - [info] Binlog setting check done.
Fri Aug 24 12:06:30 2018 - [info] Checking SSH publickey authentication and checking recovery script configurations on all alive slave servers..
Fri Aug 24 12:06:30 2018 - [info]   Executing command : apply_diff_relay_logs --command=test --slave_user='root' --slave_host=192.168.2.61 --slave_ip=192.168.2.61 --slave_port=3306 --workdir=/etc/mha/app1 --target_version=5.7.21-log --manager_version=0.57 --relay_log_info=/mysql1/data/relay-log.info  --relay_dir=/mysql1/data/  --slave_pass=xxx
Fri Aug 24 12:06:30 2018 - [info]   Connecting to root@192.168.2.61(192.168.2.61:22)..
  Checking slave recovery environment settings..
    Opening /mysql1/data/relay-log.info ... ok.
    Relay log found at /mysql1/data, up to node1-relay-bin.000002
    Temporary relay log file is /mysql1/data/node1-relay-bin.000002
    Testing mysql connection and privileges..mysql: [Warning] Using a password on the command line interface can be insecure.
 done.
    Testing mysqlbinlog output.. done.
    Cleaning up test file(s).. done.
Fri Aug 24 12:06:30 2018 - [info]   Executing command : apply_diff_relay_logs --command=test --slave_user='root' --slave_host=192.168.2.63 --slave_ip=192.168.2.63 --slave_port=3306 --workdir=/etc/mha/app1 --target_version=5.7.21-log --manager_version=0.57 --relay_log_info=/mysql1/data/relay-log.info  --relay_dir=/mysql1/data/  --slave_pass=xxx
Fri Aug 24 12:06:30 2018 - [info]   Connecting to root@192.168.2.63(192.168.2.63:22)..
  Checking slave recovery environment settings..
    Opening /mysql1/data/relay-log.info ... ok.
    Relay log found at /mysql1/data, up to node3-relay-bin.000002
    Temporary relay log file is /mysql1/data/node3-relay-bin.000002
    Testing mysql connection and privileges..mysql: [Warning] Using a password on the command line interface can be insecure.
 done.
    Testing mysqlbinlog output.. done.
    Cleaning up test file(s).. done.
Fri Aug 24 12:06:30 2018 - [info] Slaves settings check done.
Fri Aug 24 12:06:30 2018 - [info]
192.168.2.62(192.168.2.62:3306) (current master)
 +--192.168.2.61(192.168.2.61:3306)
 +--192.168.2.63(192.168.2.63:3306)

Fri Aug 24 12:06:30 2018 - [info] Checking replication health on 192.168.2.61..
Fri Aug 24 12:06:30 2018 - [info]  ok.
Fri Aug 24 12:06:30 2018 - [info] Checking replication health on 192.168.2.63..
Fri Aug 24 12:06:30 2018 - [info]  ok.
Fri Aug 24 12:06:30 2018 - [info] Checking master_ip_failover_script status:
Fri Aug 24 12:06:30 2018 - [info]   /etc/mha/master_ip_failover --command=status --ssh_user=root --orig_master_host=192.168.2.62 --orig_master_ip=192.168.2.62 --orig_master_port=3306


IN SCRIPT TEST====/sbin/ifconfig eth0:0 down==/sbin/ifconfig eth0:0 192.168.6.66/24===

Checking the Status of the script.. OK
Fri Aug 24 12:06:30 2018 - [info]  OK.
Fri Aug 24 12:06:30 2018 - [warning] shutdown_script is not defined.
Fri Aug 24 12:06:30 2018 - [info] Got exit code 0 (Not master dead).

MySQL Replication Health is OK.



```

##  1045:Access denied for user 'root1'@'node3' 

### 问题表现

```sql


[root@node3 bin]# /etc/mha/mhamanager/bin/masterha_check_repl --conf=/etc/mha/app1/app1.cnf
Fri Aug 24 12:26:01 2018 - [warning] Global configuration file /etc/masterha_default.cnf not found. Skipping.
Fri Aug 24 12:26:01 2018 - [info] Reading application default configuration from /etc/mha/app1/app1.cnf..
Fri Aug 24 12:26:01 2018 - [info] Reading server configuration from /etc/mha/app1/app1.cnf..
Fri Aug 24 12:26:01 2018 - [info] MHA::MasterMonitor version 0.57.
Fri Aug 24 12:26:01 2018 - [error][/usr/local/share/perl5/MHA/ServerManager.pm, ln301] Got MySQL error when connecting 192.168.2.63(192.168.2.63:3306) :1045:Access denied for user 'root1'@'node3' (using password: YES), but this is not a MySQL crash. Check MySQL server settings.
 at /usr/local/share/perl5/MHA/ServerManager.pm line 297
Fri Aug 24 12:26:01 2018 - [error][/usr/local/share/perl5/MHA/ServerManager.pm, ln301] Got MySQL error when connecting 192.168.2.61(192.168.2.61:3306) :1045:Access denied for user 'root1'@'node3' (using password: YES), but this is not a MySQL crash. Check MySQL server settings.
 at /usr/local/share/perl5/MHA/ServerManager.pm line 297
Fri Aug 24 12:26:01 2018 - [error][/usr/local/share/perl5/MHA/ServerManager.pm, ln301] Got MySQL error when connecting 192.168.2.62(192.168.2.62:3306) :1045:Access denied for user 'root1'@'node3' (using password: YES), but this is not a MySQL crash. Check MySQL server settings.
 at /usr/local/share/perl5/MHA/ServerManager.pm line 297
Fri Aug 24 12:26:01 2018 - [error][/usr/local/share/perl5/MHA/ServerManager.pm, ln309] Got fatal error, stopping operations
Fri Aug 24 12:26:01 2018 - [error][/usr/local/share/perl5/MHA/MasterMonitor.pm, ln427] Error happened on checking configurations.  at /usr/local/share/perl5/MHA/MasterMonitor.pm line 329
Fri Aug 24 12:26:01 2018 - [error][/usr/local/share/perl5/MHA/MasterMonitor.pm, ln525] Error happened on monitoring servers.
Fri Aug 24 12:26:01 2018 - [info] Got exit code 1 (Not master dead).

MySQL Replication Health is NOT OK!


```

###   解决方法

- 修改配置文件中的用户名 或创建账户 

修改配置文件

```sql

[root@node3 ~]# vi /etc/mha/app1/app1.cnf

[server default]
# working directory on the manager
manager_workdir=/etc/mha/app1
manager_log=/etc/mha/app1/manager.log
remote_workdir=/etc/mha/app1
# In case of automatic file-over, the following script will be used
master_ip_failover_script=/etc/mha/master_ip_failover
master_ip_online_change_script=/etc/mha/master_ip_online_change

#mha连接到mysql的用户(此处账户出错应为root!!! )
user=root1
password=root

#mysql主从复制架构的复制用户
repl_user=dao   
repl_password=dao


ssh_user=root

#心跳
ping_interval=1


secondary_check_script= masterha_secondary_check -s 192.168.2.62 -s 192.168.2.63

[server1]
hostname=192.168.2.61
port=3306
master_binlog_dir="/mysql1/data"
candidate_master=1
ssh_port=22
[server2]
hostname=192.168.2.62
port=3306
master_binlog_dir="/mysql1/data"
candidate_master=1
ssh_port=22
[server3]
hostname=192.168.2.63
port=3306
master_binlog_dir="/mysql1/data"
no_master=1
ssh_port=22


```

- 或创建用户

```sql

GRANT ALL PRIVILEGES ON *.* TO 'root1'@'192.168.2.%' IDENTIFIED BY 'dao' ;

```

###  问题验证

```sql

[root@node3 bin]# /etc/mha/mhamanager/bin/masterha_check_repl --conf=/etc/mha/app1/app1.cnf
Fri Aug 24 12:29:47 2018 - [warning] Global configuration file /etc/masterha_default.cnf not found. Skipping.
Fri Aug 24 12:29:47 2018 - [info] Reading application default configuration from /etc/mha/app1/app1.cnf..
Fri Aug 24 12:29:47 2018 - [info] Reading server configuration from /etc/mha/app1/app1.cnf..
Fri Aug 24 12:29:47 2018 - [info] MHA::MasterMonitor version 0.57.
Fri Aug 24 12:29:47 2018 - [info] GTID failover mode = 0
Fri Aug 24 12:29:47 2018 - [info] Dead Servers:
Fri Aug 24 12:29:47 2018 - [info] Alive Servers:
Fri Aug 24 12:29:47 2018 - [info]   192.168.2.61(192.168.2.61:3306)
Fri Aug 24 12:29:47 2018 - [info]   192.168.2.62(192.168.2.62:3306)
Fri Aug 24 12:29:47 2018 - [info]   192.168.2.63(192.168.2.63:3306)
Fri Aug 24 12:29:47 2018 - [info] Alive Slaves:
Fri Aug 24 12:29:47 2018 - [info]   192.168.2.61(192.168.2.61:3306)  Version=5.7.21-log (oldest major version between slaves) log-bin:enabled
Fri Aug 24 12:29:47 2018 - [info]     Replicating from 192.168.2.62(192.168.2.62:3306)
Fri Aug 24 12:29:47 2018 - [info]     Primary candidate for the new Master (candidate_master is set)
Fri Aug 24 12:29:47 2018 - [info]   192.168.2.63(192.168.2.63:3306)  Version=5.7.21-log (oldest major version between slaves) log-bin:enabled
Fri Aug 24 12:29:47 2018 - [info]     Replicating from 192.168.2.62(192.168.2.62:3306)
Fri Aug 24 12:29:47 2018 - [info]     Not candidate for the new Master (no_master is set)
Fri Aug 24 12:29:47 2018 - [info] Current Alive Master: 192.168.2.62(192.168.2.62:3306)
Fri Aug 24 12:29:47 2018 - [info] Checking slave configurations..
Fri Aug 24 12:29:47 2018 - [info]  read_only=1 is not set on slave 192.168.2.61(192.168.2.61:3306).
Fri Aug 24 12:29:47 2018 - [warning]  relay_log_purge=0 is not set on slave 192.168.2.61(192.168.2.61:3306).
Fri Aug 24 12:29:47 2018 - [info]  read_only=1 is not set on slave 192.168.2.63(192.168.2.63:3306).
Fri Aug 24 12:29:47 2018 - [warning]  relay_log_purge=0 is not set on slave 192.168.2.63(192.168.2.63:3306).
Fri Aug 24 12:29:47 2018 - [info] Checking replication filtering settings..
Fri Aug 24 12:29:47 2018 - [info]  binlog_do_db= , binlog_ignore_db=
Fri Aug 24 12:29:47 2018 - [info]  Replication filtering check ok.
Fri Aug 24 12:29:47 2018 - [info] GTID (with auto-pos) is not supported
Fri Aug 24 12:29:47 2018 - [info] Starting SSH connection tests..
Fri Aug 24 12:29:48 2018 - [info] All SSH connection tests passed successfully.
Fri Aug 24 12:29:48 2018 - [info] Checking MHA Node version..
Fri Aug 24 12:29:49 2018 - [info]  Version check ok.
Fri Aug 24 12:29:49 2018 - [info] Checking SSH publickey authentication settings on the current master..
Fri Aug 24 12:29:49 2018 - [info] HealthCheck: SSH to 192.168.2.62 is reachable.
Fri Aug 24 12:29:49 2018 - [info] Master MHA Node version is 0.57.
Fri Aug 24 12:29:49 2018 - [info] Checking recovery script configurations on 192.168.2.62(192.168.2.62:3306)..
Fri Aug 24 12:29:49 2018 - [info]   Executing command: save_binary_logs --command=test --start_pos=4 --binlog_dir=/mysql1/data --output_file=/etc/mha/app1/save_binary_logs_test --manager_version=0.57 --start_file=mysql-bin.000012
Fri Aug 24 12:29:49 2018 - [info]   Connecting to root@192.168.2.62(192.168.2.62:22)..
  Creating /etc/mha/app1 if not exists..    ok.
  Checking output directory is accessible or not..
   ok.
  Binlog found at /mysql1/data, up to mysql-bin.000012
Fri Aug 24 12:29:50 2018 - [info] Binlog setting check done.
Fri Aug 24 12:29:50 2018 - [info] Checking SSH publickey authentication and checking recovery script configurations on all alive slave servers..
Fri Aug 24 12:29:50 2018 - [info]   Executing command : apply_diff_relay_logs --command=test --slave_user='root' --slave_host=192.168.2.61 --slave_ip=192.168.2.61 --slave_port=3306 --workdir=/etc/mha/app1 --target_version=5.7.21-log --manager_version=0.57 --relay_log_info=/mysql1/data/relay-log.info  --relay_dir=/mysql1/data/  --slave_pass=xxx
Fri Aug 24 12:29:50 2018 - [info]   Connecting to root@192.168.2.61(192.168.2.61:22)..
  Checking slave recovery environment settings..
    Opening /mysql1/data/relay-log.info ... ok.
    Relay log found at /mysql1/data, up to node1-relay-bin.000002
    Temporary relay log file is /mysql1/data/node1-relay-bin.000002
    Testing mysql connection and privileges..mysql: [Warning] Using a password on the command line interface can be insecure.
 done.
    Testing mysqlbinlog output.. done.
    Cleaning up test file(s).. done.
Fri Aug 24 12:29:50 2018 - [info]   Executing command : apply_diff_relay_logs --command=test --slave_user='root' --slave_host=192.168.2.63 --slave_ip=192.168.2.63 --slave_port=3306 --workdir=/etc/mha/app1 --target_version=5.7.21-log --manager_version=0.57 --relay_log_info=/mysql1/data/relay-log.info  --relay_dir=/mysql1/data/  --slave_pass=xxx
Fri Aug 24 12:29:50 2018 - [info]   Connecting to root@192.168.2.63(192.168.2.63:22)..
  Checking slave recovery environment settings..
    Opening /mysql1/data/relay-log.info ... ok.
    Relay log found at /mysql1/data, up to node3-relay-bin.000002
    Temporary relay log file is /mysql1/data/node3-relay-bin.000002
    Testing mysql connection and privileges..mysql: [Warning] Using a password on the command line interface can be insecure.
 done.
    Testing mysqlbinlog output.. done.
    Cleaning up test file(s).. done.
Fri Aug 24 12:29:50 2018 - [info] Slaves settings check done.
Fri Aug 24 12:29:50 2018 - [info]
192.168.2.62(192.168.2.62:3306) (current master)
 +--192.168.2.61(192.168.2.61:3306)
 +--192.168.2.63(192.168.2.63:3306)

Fri Aug 24 12:29:50 2018 - [info] Checking replication health on 192.168.2.61..
Fri Aug 24 12:29:50 2018 - [info]  ok.
Fri Aug 24 12:29:50 2018 - [info] Checking replication health on 192.168.2.63..
Fri Aug 24 12:29:50 2018 - [info]  ok.
Fri Aug 24 12:29:50 2018 - [info] Checking master_ip_failover_script status:
Fri Aug 24 12:29:50 2018 - [info]   /etc/mha/master_ip_failover --command=status --ssh_user=root --orig_master_host=192.168.2.62 --orig_master_ip=192.168.2.62 --orig_master_port=3306


IN SCRIPT TEST====/sbin/ifconfig eth0:0 down==/sbin/ifconfig eth0:0 192.168.6.66/24===

Checking the Status of the script.. OK
Fri Aug 24 12:29:50 2018 - [info]  OK.
Fri Aug 24 12:29:50 2018 - [warning] shutdown_script is not defined.
Fri Aug 24 12:29:50 2018 - [info] Got exit code 0 (Not master dead).

MySQL Replication Health is OK.


```

##  None of slaves can be master. Check failover configuration file or log-bin settings in my.cnf

### 问题表现

```sql

[root@node3 bin]# /etc/mha/mhamanager/bin/masterha_check_repl --conf=/etc/mha/app1/app1.cnf
Fri Aug 24 13:38:34 2018 - [warning] Global configuration file /etc/masterha_default.cnf not found. Skipping.
Fri Aug 24 13:38:34 2018 - [info] Reading application default configuration from /etc/mha/app1/app1.cnf..
Fri Aug 24 13:38:34 2018 - [info] Reading server configuration from /etc/mha/app1/app1.cnf..
Fri Aug 24 13:38:34 2018 - [info] MHA::MasterMonitor version 0.57.
Fri Aug 24 13:38:34 2018 - [info] GTID failover mode = 0
Fri Aug 24 13:38:34 2018 - [info] Dead Servers:
Fri Aug 24 13:38:34 2018 - [info]   192.168.2.62(192.168.2.62:3306)
Fri Aug 24 13:38:34 2018 - [info] Alive Servers:
Fri Aug 24 13:38:34 2018 - [info]   192.168.2.61(192.168.2.61:3306)
Fri Aug 24 13:38:34 2018 - [info]   192.168.2.63(192.168.2.63:3306)
Fri Aug 24 13:38:34 2018 - [info] Alive Slaves:
Fri Aug 24 13:38:34 2018 - [info]   192.168.2.63(192.168.2.63:3306)  Version=5.7.21-log (oldest major version between slaves) log-bin:enabled
Fri Aug 24 13:38:34 2018 - [info]     Replicating from 192.168.2.61(192.168.2.61:3306)
Fri Aug 24 13:38:34 2018 - [info]     Not candidate for the new Master (no_master is set)
Fri Aug 24 13:38:34 2018 - [info] Current Alive Master: 192.168.2.61(192.168.2.61:3306)
Fri Aug 24 13:38:34 2018 - [info] Checking slave configurations..
Fri Aug 24 13:38:34 2018 - [info]  read_only=1 is not set on slave 192.168.2.63(192.168.2.63:3306).
Fri Aug 24 13:38:34 2018 - [warning]  relay_log_purge=0 is not set on slave 192.168.2.63(192.168.2.63:3306).
Fri Aug 24 13:38:34 2018 - [info] Checking replication filtering settings..
Fri Aug 24 13:38:34 2018 - [info]  binlog_do_db= , binlog_ignore_db=
Fri Aug 24 13:38:34 2018 - [info]  Replication filtering check ok.
Fri Aug 24 13:38:34 2018 - [error][/usr/local/share/perl5/MHA/MasterMonitor.pm, ln364] None of slaves can be master. Check failover configuration file or log-bin settings in my.cnf
Fri Aug 24 13:38:34 2018 - [error][/usr/local/share/perl5/MHA/MasterMonitor.pm, ln427] Error happened on checking configurations.  at /etc/mha/mhamanager/bin/masterha_check_repl line 48
Fri Aug 24 13:38:34 2018 - [error][/usr/local/share/perl5/MHA/MasterMonitor.pm, ln525] Error happened on monitoring servers.
Fri Aug 24 13:38:34 2018 - [info] Got exit code 1 (Not master dead).

MySQL Replication Health is NOT OK!


```

### 解决方法 
   
 此报错多在MHA完成master failover后出现 出错原因为没有将主库加回到MHA中.
 
 当主库从node2->node1后 需要手工将node2的主库指向node1
 
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

mysql> change master to master_host='node1' ,master_port=3306,master_user='dao',master_password='dao',master_log_file='mysql-bin.000005',master_log_pos=154 ;
Query OK, 0 rows affected, 2 warnings (0.03 sec)

mysql> start slave ;
Query OK, 0 rows affected (0.00 sec)

 
 ```
 
 ### 问题验证
 
 
 ```sql
 
[root@node3 bin]# /etc/mha/mhamanager/bin/masterha_check_repl --conf=/etc/mha/app1/app1.cnf
Fri Aug 24 13:48:06 2018 - [warning] Global configuration file /etc/masterha_default.cnf not found. Skipping.
Fri Aug 24 13:48:06 2018 - [info] Reading application default configuration from /etc/mha/app1/app1.cnf..
Fri Aug 24 13:48:06 2018 - [info] Reading server configuration from /etc/mha/app1/app1.cnf..
Fri Aug 24 13:48:06 2018 - [info] MHA::MasterMonitor version 0.57.
Fri Aug 24 13:48:06 2018 - [info] GTID failover mode = 0
Fri Aug 24 13:48:06 2018 - [info] Dead Servers:
Fri Aug 24 13:48:06 2018 - [info] Alive Servers:
Fri Aug 24 13:48:06 2018 - [info]   192.168.2.61(192.168.2.61:3306)
Fri Aug 24 13:48:06 2018 - [info]   192.168.2.62(192.168.2.62:3306)
Fri Aug 24 13:48:06 2018 - [info]   192.168.2.63(192.168.2.63:3306)
Fri Aug 24 13:48:06 2018 - [info] Alive Slaves:
Fri Aug 24 13:48:06 2018 - [info]   192.168.2.62(192.168.2.62:3306)  Version=5.7.21-log (oldest major version between slaves) log-bin:enabled
Fri Aug 24 13:48:06 2018 - [info]     Replicating from node1(192.168.2.61:3306)
Fri Aug 24 13:48:06 2018 - [info]     Primary candidate for the new Master (candidate_master is set)
Fri Aug 24 13:48:06 2018 - [info]   192.168.2.63(192.168.2.63:3306)  Version=5.7.21-log (oldest major version between slaves) log-bin:enabled
Fri Aug 24 13:48:06 2018 - [info]     Replicating from 192.168.2.61(192.168.2.61:3306)
Fri Aug 24 13:48:06 2018 - [info]     Not candidate for the new Master (no_master is set)
Fri Aug 24 13:48:06 2018 - [info] Current Alive Master: 192.168.2.61(192.168.2.61:3306)
Fri Aug 24 13:48:06 2018 - [info] Checking slave configurations..
Fri Aug 24 13:48:06 2018 - [info]  read_only=1 is not set on slave 192.168.2.62(192.168.2.62:3306).
Fri Aug 24 13:48:06 2018 - [warning]  relay_log_purge=0 is not set on slave 192.168.2.62(192.168.2.62:3306).
Fri Aug 24 13:48:06 2018 - [info]  read_only=1 is not set on slave 192.168.2.63(192.168.2.63:3306).
Fri Aug 24 13:48:06 2018 - [warning]  relay_log_purge=0 is not set on slave 192.168.2.63(192.168.2.63:3306).
Fri Aug 24 13:48:06 2018 - [info] Checking replication filtering settings..
Fri Aug 24 13:48:06 2018 - [info]  binlog_do_db= , binlog_ignore_db=
Fri Aug 24 13:48:06 2018 - [info]  Replication filtering check ok.
Fri Aug 24 13:48:06 2018 - [info] GTID (with auto-pos) is not supported
Fri Aug 24 13:48:06 2018 - [info] Starting SSH connection tests..
Fri Aug 24 13:48:08 2018 - [info] All SSH connection tests passed successfully.
Fri Aug 24 13:48:08 2018 - [info] Checking MHA Node version..
Fri Aug 24 13:48:09 2018 - [info]  Version check ok.
Fri Aug 24 13:48:09 2018 - [info] Checking SSH publickey authentication settings on the current master..
Fri Aug 24 13:48:09 2018 - [info] HealthCheck: SSH to 192.168.2.61 is reachable.
Fri Aug 24 13:48:09 2018 - [info] Master MHA Node version is 0.57.
Fri Aug 24 13:48:09 2018 - [info] Checking recovery script configurations on 192.168.2.61(192.168.2.61:3306)..
Fri Aug 24 13:48:09 2018 - [info]   Executing command: save_binary_logs --command=test --start_pos=4 --binlog_dir=/mysql1/data --output_file=/etc/mha/app1/save_binary_logs_test --manager_version=0.57 --start_file=mysql-bin.000005
Fri Aug 24 13:48:09 2018 - [info]   Connecting to root@192.168.2.61(192.168.2.61:22)..
  Creating /etc/mha/app1 if not exists..    ok.
  Checking output directory is accessible or not..
   ok.
  Binlog found at /mysql1/data, up to mysql-bin.000005
Fri Aug 24 13:48:09 2018 - [info] Binlog setting check done.
Fri Aug 24 13:48:09 2018 - [info] Checking SSH publickey authentication and checking recovery script configurations on all alive slave servers..
Fri Aug 24 13:48:09 2018 - [info]   Executing command : apply_diff_relay_logs --command=test --slave_user='root' --slave_host=192.168.2.62 --slave_ip=192.168.2.62 --slave_port=3306 --workdir=/etc/mha/app1 --target_version=5.7.21-log --manager_version=0.57 --relay_log_info=/mysql1/data/relay-log.info  --relay_dir=/mysql1/data/  --slave_pass=xxx
Fri Aug 24 13:48:09 2018 - [info]   Connecting to root@192.168.2.62(192.168.2.62:22)..
  Checking slave recovery environment settings..
    Opening /mysql1/data/relay-log.info ... ok.
    Relay log found at /mysql1/data, up to node2-relay-bin.000002
    Temporary relay log file is /mysql1/data/node2-relay-bin.000002
    Testing mysql connection and privileges..mysql: [Warning] Using a password on the command line interface can be insecure.
 done.
    Testing mysqlbinlog output.. done.
    Cleaning up test file(s).. done.
Fri Aug 24 13:48:10 2018 - [info]   Executing command : apply_diff_relay_logs --command=test --slave_user='root' --slave_host=192.168.2.63 --slave_ip=192.168.2.63 --slave_port=3306 --workdir=/etc/mha/app1 --target_version=5.7.21-log --manager_version=0.57 --relay_log_info=/mysql1/data/relay-log.info  --relay_dir=/mysql1/data/  --slave_pass=xxx
Fri Aug 24 13:48:10 2018 - [info]   Connecting to root@192.168.2.63(192.168.2.63:22)..
  Checking slave recovery environment settings..
    Opening /mysql1/data/relay-log.info ... ok.
    Relay log found at /mysql1/data, up to node3-relay-bin.000002
    Temporary relay log file is /mysql1/data/node3-relay-bin.000002
    Testing mysql connection and privileges..mysql: [Warning] Using a password on the command line interface can be insecure.
 done.
    Testing mysqlbinlog output.. done.
    Cleaning up test file(s).. done.
Fri Aug 24 13:48:10 2018 - [info] Slaves settings check done.
Fri Aug 24 13:48:10 2018 - [info]
192.168.2.61(192.168.2.61:3306) (current master)
 +--192.168.2.62(192.168.2.62:3306)
 +--192.168.2.63(192.168.2.63:3306)

Fri Aug 24 13:48:10 2018 - [info] Checking replication health on 192.168.2.62..
Fri Aug 24 13:48:10 2018 - [info]  ok.
Fri Aug 24 13:48:10 2018 - [info] Checking replication health on 192.168.2.63..
Fri Aug 24 13:48:10 2018 - [info]  ok.
Fri Aug 24 13:48:10 2018 - [info] Checking master_ip_failover_script status:
Fri Aug 24 13:48:10 2018 - [info]   /etc/mha/master_ip_failover --command=status --ssh_user=root --orig_master_host=192.168.2.61 --orig_master_ip=192.168.2.61 --orig_master_port=3306


IN SCRIPT TEST====/sbin/ifconfig eth0:0 down==/sbin/ifconfig eth0:0 192.168.6.66/24===

Checking the Status of the script.. OK
Fri Aug 24 13:48:10 2018 - [info]  OK.
Fri Aug 24 13:48:10 2018 - [warning] shutdown_script is not defined.
Fri Aug 24 13:48:10 2018 - [info] Got exit code 0 (Not master dead).

MySQL Replication Health is OK.

 
 
 ```
