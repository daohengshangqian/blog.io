---
layout: post
title:  MySQL5.7 安装
date: 2018-06-29
categories: blog
tags: [MySQL]
description: MySQL5.7安装
---

## 创建文件夹

```sql

[root@dao ~]# mkdir /usr/local/mysql
[root@dao ~]# mkdir -p /data/mysql3306/data
[root@dao ~]# mkdir -p /data/mysql3306/logs

```


## 解压并查看

```SQL
[root@dao ~]# tar -zxf mysql-5.7.21-linux-glibc2.12-x86_64.tar.gz 
[root@dao ~]# ll
total 626888
-rw-------. 1 root root      1679 Nov  5  2015 anaconda-ks.cfg
drwxr-xr-x. 2 root root      4096 Jun 29 07:04 Desktop
drwxr-xr-x. 2 root root      4096 Nov  5  2015 Documents
drwxr-xr-x. 2 root root      4096 Nov  5  2015 Downloads
-rw-r--r--. 1 root root     14540 Nov  5  2012 epel-release-6-8.noarch.rpm
-rw-r--r--. 1 root root     49565 Nov  5  2015 install.log
-rw-r--r--. 1 root root     10033 Nov  5  2015 install.log.syslog
drwxr-xr-x. 2 root root      4096 Nov  5  2015 Music
drwxr-xr-x. 9 root root      4096 Jun 29 07:14 mysql-5.7.21-linux-glibc2.12-x86_64
-rw-r--r--. 1 root root 641798603 Jan 21 06:51 mysql-5.7.21-linux-glibc2.12-x86_64.tar.gz
drwxr-xr-x. 2 root root      4096 Nov  5  2015 Pictures
drwxr-xr-x. 2 root root      4096 Nov  5  2015 Public
drwxr-xr-x. 2 root root      4096 Nov  5  2015 Templates
drwxr-xr-x. 2 root root      4096 Nov  5  2015 Videos

```
## 复制到相关目录

```sql
[root@dao ~]# cp -r mysql-5.7.21-linux-glibc2.12-x86_64/* /usr/local/mysql
```


##  建立用户并修改权限

```sql

[root@dao ~]# useradd  mysql
[root@dao ~]# chown -R mysql:mysql /usr/local/mysql/
[root@dao ~]# chown -R mysql:mysql /data/

```

## 复制参数文件

```sql

[root@dao support-files]# cp my-default.cnf  /etc/my.cnf
[root@dao support-files]# chown -R mysql:mysql /etc/my.cnf 

```

## 编辑参数文件

```sql

[root@dao ~]# cat /etc/my.cnf 
[mysql]
prompt = [\\u@\\h][\\d]>\\_
socket=/tmp/mysql.sock

[mysqld]
# basic settings #
user = mysql
sql_mode = "STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION,NO_ZERO_DATE,NO_ZERO_IN_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_AUTO_VALUE_ON_ZERO,ONLY_FULL_GROUP_BY"
character_set_server=utf8mb4
collation_server=utf8mb4_bin
default-storage-engine=innodb
pid-file=/data/mysql3306/data/mysql.pid
socket=/tmp/mysql.sock
basedir= /usr/local/mysql
datadir=/data/mysql3306/data
tmpdir=/tmp
transaction-isolation = READ-COMMITTED
explicit_defaults_for_timestamp = 1
lower_case_table_names=1
max_allowed_packet = 32M
slave_pending_jobs_size_max = 32M
event_scheduler = 1
server-id=1383306
port=3306
symbolic-links=0 
sysdate-is-now 
show_compatibility_56=on 
back_log=1000 

# connection #
interactive_timeout = 200
wait_timeout = 200
lock_wait_timeout = 1800
skip_name_resolve = 1
max_connections = 2000
max_connect_errors = 100000

# table cache performance settings #(??)
table_open_cache = 3000
table_definition_cache = 4096
table_open_cache_instances = 16

# session memory settings #(??)
read_buffer_size = 2M
read_rnd_buffer_size = 16M
sort_buffer_size = 4M
tmp_table_size = 100M
max_heap_table_size = 100M
join_buffer_size = 4M
thread_cache_size = 64 
# log settings #
general_log_file=/data/mysql3306/logs/general.log
log-error=/data/mysql3306/logs/error.log
slow_query_log = 1
long_query_time = 1
slow_query_log_file=/data/mysql3306/logs/slow-query.log
log_queries_not_using_indexes = 1
log_slow_admin_statements = 1
log_slow_slave_statements = 1
log_throttle_queries_not_using_indexes = 10
binlog-rows-query-log-events = 1
log-bin=/data/mysql3306/data/mysql-bin
log-bin-index=/data/mysql3306/data/mysql-bin.index
log_bin_trust_function_creators = 1
binlog_cache_size=4M
max_binlog_cache_size = 4096M
max_binlog_size = 1024M
expire-logs-days=3
log-slave-updates = 1
relay-log-index=/data/mysql3306/data/mysql-relay-bin.index
relay-log=/data/mysql3306/data/mysql-relay-bin
log_timestamps=system
#log_warnings =2 #??????,?????log_error_verbosity=2
log_error_verbosity=2

# innodb settings #
innodb_page_size = 16384
innodb_data_home_dir=/data/mysql3306/data
innodb_data_file_path=ibdata1:1024M:autoextend
innodb_buffer_pool_size = 1G  #(??)
innodb_buffer_pool_instances = 1
innodb_buffer_pool_dump_pct = 40
innodb_buffer_pool_load_at_startup = 1
innodb_buffer_pool_dump_at_shutdown = 1
innodb_lru_scan_depth = 4096
innodb_lock_wait_timeout = 50
innodb_rollback_on_timeout=1
innodb_flush_log_at_trx_commit = 1
innodb_io_capacity = 200
innodb_max_dirty_pages_pct =75
innodb_io_capacity_max = 1000
innodb_flush_method = O_DIRECT
innodb_file_format = Barracuda
innodb_file_format_max = Barracuda
innodb_undo_directory=/data/mysql3306/data #??,????
innodb_undo_logs = 128
innodb_undo_tablespaces = 3
innodb_undo_log_truncate = 1 #????/??undo log ??truncate??
innodb_max_undo_log_size = 2G
innodb_purge_threads = 4
innodb_purge_rseg_truncate_frequency = 128 #?????????????,??????
innodb_flush_neighbors = 0 #ssd0,?ssd1
innodb_log_buffer_size = 64M 
innodb_log_file_size = 256M  
innodb_log_files_in_group = 2 
innodb_log_group_home_dir=/data/mysql3306/data
innodb_page_cleaners = 16
innodb_large_prefix = 1
innodb_thread_concurrency = 64
innodb_sync_spin_loops=0 #???????,?????CPU??
innodb_print_all_deadlocks = 1
innodb_strict_mode = 1
innodb_sort_buffer_size = 67108864
innodb_write_io_threads = 16  #(??)
innodb_read_io_threads = 16   #(??)
innodb_file_per_table = 1
innodb_stats_persistent_sample_pages = 64
innodb_autoinc_lock_mode = 2
innodb_online_alter_log_max_size=1G
innodb_open_files=4096
loose_innodb_numa_interleave=1

# about query_cache #
query_cache_type=0
query_cache_size=0

# replication settings #
master_info_repository = TABLE
relay_log_info_repository = TABLE
sync_binlog = 1
sync-relay-log =0 
sync-relay-log-info =0 #??relay_log_info_repository = TABLE,?mysql.slave_relay_log_info?innodb?,???????1,????innodb??N?
gtid_mode = on
enforce_gtid_consistency = 1
log_slave_updates
binlog_format = ROW
binlog_rows_query_log_events = 1
relay_log_recovery = 1
slave_parallel_workers=20
slave-parallel-type = LOGICAL_CLOCK
slave_transaction_retries=128
#slave_skip_errors = ddl_exist_errors
slave-rows-search-algorithms = 'INDEX_SCAN,HASH_SCAN'
log-slave-updates=true
sync-master-info=1  
binlog-checksum=CRC32
master-verify-checksum=1
slave-sql-verify-checksum=1
slave_preserve_commit_order=1 
read_only=0

# GTID settings #
gtid-mode=on
enforce-gtid-consistency=true
binlog_gtid_simple_recovery=1 

# SEMI Replication setting
plugin-load="rpl_semi_sync_master=semisync_master.so;rpl_semi_sync_slave=semisync_slave.so"
rpl_semi_sync_master_enabled =1
rpl_semi_sync_slave_enabled=1
rpl_semi_sync_master_timeout=5000

[mysqldump]
quick
max_allowed_packet = 32M

[mysqld_safe]
user=mysql
open-files-limit=65536
log-error=/data/mysql3306/logs/error.log

```

## 修改环境变量

```sql
[mysql@dao ~]$ vi .bash_profile 
# .bash_profile

# Get the aliases and functions
if [ -f ~/.bashrc ]; then
        . ~/.bashrc
fi

# User specific environment and startup programs
MYSQL_HOME=/usr/local/mysql

PATH=$MYSQL_HOME/bin:$PATH:$HOME/bin

export PATH

```


## 初始化Mysql

```sql
[mysql@dao ~]# /usr/local/mysql/bin/mysqld --defaults-file=/etc/my.cnf --initialize-insecure --basedir=/usr/local/mysql --datadir=/data/mysql3306/data/ --user=mysql

```
## 启动Mysql

```sql
[mysql@dao ~]# /usr/local/mysql/bin/mysqld_safe --defaults-file=/etc/my.cnf  --user=mysql 
2018-06-29T11:31:29.540583Z mysqld_safe Logging to '/data/mysql3306/logs/error.log'.
2018-06-29T11:31:29.583012Z mysqld_safe Starting mysqld daemon with databases from /data/mysql3306/data
```


## 进入Mysql并修改权限

```sql

[mysql@dao ~]# /usr/local/mysql/bin/mysql 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.7.21-log MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

[mysql@localhost][(none)]> 


[mysql@localhost][(none)]> Alter user root@'localhost' identified by 'root' ;
Query OK, 0 rows affected (0.01 sec)


```
