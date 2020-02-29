---
layout: post
title:  MySQL5.6安装
date: 2018-06-29
categories: blog
tags: [MySQL]
description: MySQL5.6安装
---

## 创建文件夹
```sql
[root@dao ~]# mkdir /usr/local/mysql
[root@dao ~]# mkdir -p /data/mysql3306/data
[root@dao ~]# mkdir -p /data/mysql3306/logs

```

## 安装依赖包

```sql

[root@dao ~]# yum -y groupinstall "Development tools"

```


## 解压

```SQL

[root@dao ~]# tar -zxf mysql-5.6.39-linux-glibc2.12-x86_64.tar.gz 
[root@dao ~]# mv mysql-5.6.39-linux-glibc2.12-x86_64 /usr/local/mysql
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
[root@dao support-files]# cat /etc/my.cnf q
[client]
port            = 3306
socket          = /tmp/mysql3306.sock
secure_auth     = false

[mysqld]
port            = 3306
socket          = /tmp/mysql3306.sock
datadir         = /data/mysql3306/data
#read_only       = on

#--- GLOBAL ---#
character_set_server    = utf8
lower_case_table_names  = 1
log-output              = FILE
log-error               = /data/mysql3306/logs/mysql-error.log
#general_log
general_log_file        = /data/mysql3306/logs/mysql.log
pid-file                = /data/mysql3306/data/mysql.pid
slow-query-log          = 1
slow_query_log_file     = /data/mysql3306/logs/mysql-slow.log
tmpdir                  = /tmp/
long_query_time         = 2
innodb_force_recovery   = 0
#innodb_buffer_pool_dump_at_shutdown = 1
#innodb_buffer_pool_load_at_startup = 1
#--------------#

#thread_concurrency      = 8  
thread_cache_size       = 51  
table_open_cache        = 16384
open_files_limit        = 65535
table_definition_cache  = 16384
sort_buffer_size        = 2M
join_buffer_size        = 2M
read_buffer_size        = 2M
read_rnd_buffer_size    = 8M
key_buffer_size         = 32M
bulk_insert_buffer_size = 16M
myisam_sort_buffer_size = 64M
tmp_table_size          = 32M
max_heap_table_size     = 16M 
query_cache_size       = 32MB

#gtid_mode=on
#log_slave_updates=1
#enforce_gtid_consistency=1

#--- NETWORK ---#
back_log                = 103
max-connections         = 512
max_connect_errors      = 100000
max_allowed_packet      = 32M
interactive_timeout     = 600
wait_timeout            = 600
skip-external-locking
#max_user_connections    = 0
external-locking        = FALSE
#skip-name-resolve

#--- REPL ---#
server-id               = 28703306
sync_binlog             = 1
log-bin                 = mysql-bin
binlog_format           = row
expire_logs_days        = 10
relay-log               = relay-log
replicate-ignore-db     = test
log_slave_updates       =1
#skip-slave-start
binlog_cache_size       =4M
max_binlog_cache_size   =8M
max_binlog_size         =1024M



#--- INNODB ---#
default_storage_engine          = InnoDB
innodb_data_file_path           = ibdata1:1024M:autoextend
innodb_buffer_pool_size         = 1040M
innodb_buffer_pool_instances    = 1
innodb_additional_mem_pool_size = 16M
innodb_log_files_in_group       = 2
innodb_log_file_size            = 256MB
innodb_log_buffer_size          = 16M
innodb_flush_log_at_trx_commit  = 2
innodb_lock_wait_timeout        = 30
innodb_flush_method             = O_DIRECT
innodb_max_dirty_pages_pct      = 75
innodb_io_capacity              = 200
innodb_thread_concurrency       = 32
innodb_open_files               = 65535
innodb_file_per_table           = 1
transaction_isolation           = REPEATABLE-READ
innodb_locks_unsafe_for_binlog  = 0
#innodb_purge_thread            = 4
skip_name_resolve               = 1
[mysqldump]
quick
max_allowed_packet = 32M


[mysql]
auto-rehash
# Remove the next comment character if you are not familiar with SQL
#safe-updates
default_character_set=utf8


[mysqlhotcopy]
interactive-timeout

cat: q: No such file or directory
[root@dao support-files]# cat /etc/my.cnf 
[client]
port            = 3306
socket          = /tmp/mysql3306.sock
secure_auth     = false

[mysqld]
port            = 3306
socket          = /tmp/mysql3306.sock
datadir         = /data/mysql3306/data
#read_only       = on

#--- GLOBAL ---#
character_set_server    = utf8
lower_case_table_names  = 1
log-output              = FILE
log-error               = /data/mysql3306/logs/mysql-error.log
#general_log
general_log_file        = /data/mysql3306/logs/mysql.log
pid-file                = /data/mysql3306/data/mysql.pid
slow-query-log          = 1
slow_query_log_file     = /data/mysql3306/logs/mysql-slow.log
tmpdir                  = /tmp/
long_query_time         = 2
innodb_force_recovery   = 0
#innodb_buffer_pool_dump_at_shutdown = 1
#innodb_buffer_pool_load_at_startup = 1
#--------------#

#thread_concurrency      = 8  
thread_cache_size       = 51  
table_open_cache        = 16384
open_files_limit        = 65535
table_definition_cache  = 16384
sort_buffer_size        = 2M
join_buffer_size        = 2M
read_buffer_size        = 2M
read_rnd_buffer_size    = 8M
key_buffer_size         = 32M
bulk_insert_buffer_size = 16M
myisam_sort_buffer_size = 64M
tmp_table_size          = 32M
max_heap_table_size     = 16M 
query_cache_size       = 32MB

#gtid_mode=on
#log_slave_updates=1
#enforce_gtid_consistency=1

#--- NETWORK ---#
back_log                = 103
max-connections         = 512
max_connect_errors      = 100000
max_allowed_packet      = 32M
interactive_timeout     = 600
wait_timeout            = 600
skip-external-locking
#max_user_connections    = 0
external-locking        = FALSE
#skip-name-resolve

#--- REPL ---#
server-id               = 28703306
sync_binlog             = 1
log-bin                 = mysql-bin
binlog_format           = row
expire_logs_days        = 10
relay-log               = relay-log
replicate-ignore-db     = test
log_slave_updates       =1
#skip-slave-start
binlog_cache_size       =4M
max_binlog_cache_size   =8M
max_binlog_size         =1024M



#--- INNODB ---#
default_storage_engine          = InnoDB
innodb_data_file_path           = ibdata1:1024M:autoextend
innodb_buffer_pool_size         = 1040M
innodb_buffer_pool_instances    = 1
innodb_additional_mem_pool_size = 16M
innodb_log_files_in_group       = 2
innodb_log_file_size            = 256MB
innodb_log_buffer_size          = 16M
innodb_flush_log_at_trx_commit  = 2
innodb_lock_wait_timeout        = 30
innodb_flush_method             = O_DIRECT
innodb_max_dirty_pages_pct      = 75
innodb_io_capacity              = 200
innodb_thread_concurrency       = 32
innodb_open_files               = 65535
innodb_file_per_table           = 1
transaction_isolation           = REPEATABLE-READ
innodb_locks_unsafe_for_binlog  = 0
#innodb_purge_thread            = 4
skip_name_resolve               = 1
[mysqldump]
quick
max_allowed_packet = 32M


[mysql]
auto-rehash
# Remove the next comment character if you are not familiar with SQL
#safe-updates
default_character_set=utf8


[mysqlhotcopy]
interactive-timeout

[root@dao support-files]# clear
[root@dao support-files]# cat /etc/my.cnf 
[client]
port            = 3306
socket          = /tmp/mysql3306.sock
secure_auth     = false

[mysqld]
port            = 3306
socket          = /tmp/mysql3306.sock
datadir         = /data/mysql3306/data
#read_only       = on

#--- GLOBAL ---#
character_set_server    = utf8
lower_case_table_names  = 1
log-output              = FILE
log-error               = /data/mysql3306/logs/mysql-error.log
#general_log
general_log_file        = /data/mysql3306/logs/mysql.log
pid-file                = /data/mysql3306/data/mysql.pid
slow-query-log          = 1
slow_query_log_file     = /data/mysql3306/logs/mysql-slow.log
tmpdir                  = /tmp/
long_query_time         = 2
innodb_force_recovery   = 0
#innodb_buffer_pool_dump_at_shutdown = 1
#innodb_buffer_pool_load_at_startup = 1
#--------------#

#thread_concurrency      = 8  
thread_cache_size       = 51  
table_open_cache        = 16384
open_files_limit        = 65535
table_definition_cache  = 16384
sort_buffer_size        = 2M
join_buffer_size        = 2M
read_buffer_size        = 2M
read_rnd_buffer_size    = 8M
key_buffer_size         = 32M
bulk_insert_buffer_size = 16M
myisam_sort_buffer_size = 64M
tmp_table_size          = 32M
max_heap_table_size     = 16M 
query_cache_size       = 32MB

#gtid_mode=on
#log_slave_updates=1
#enforce_gtid_consistency=1

#--- NETWORK ---#
back_log                = 103
max-connections         = 512
max_connect_errors      = 100000
max_allowed_packet      = 32M
interactive_timeout     = 600
wait_timeout            = 600
skip-external-locking
#max_user_connections    = 0
external-locking        = FALSE
#skip-name-resolve

#--- REPL ---#
server-id               = 28703306
sync_binlog             = 1
log-bin                 = mysql-bin
binlog_format           = row
expire_logs_days        = 10
relay-log               = relay-log
replicate-ignore-db     = test
log_slave_updates       =1
#skip-slave-start
binlog_cache_size       =4M
max_binlog_cache_size   =8M
max_binlog_size         =1024M



#--- INNODB ---#
default_storage_engine          = InnoDB
innodb_data_file_path           = ibdata1:1024M:autoextend
innodb_buffer_pool_size         = 1040M
innodb_buffer_pool_instances    = 1
innodb_additional_mem_pool_size = 16M
innodb_log_files_in_group       = 2
innodb_log_file_size            = 256MB
innodb_log_buffer_size          = 16M
innodb_flush_log_at_trx_commit  = 2
innodb_lock_wait_timeout        = 30
innodb_flush_method             = O_DIRECT
innodb_max_dirty_pages_pct      = 75
innodb_io_capacity              = 200
innodb_thread_concurrency       = 32
innodb_open_files               = 65535
innodb_file_per_table           = 1
transaction_isolation           = REPEATABLE-READ
innodb_locks_unsafe_for_binlog  = 0
#innodb_purge_thread            = 4
skip_name_resolve               = 1
[mysqldump]
quick
max_allowed_packet = 32M


[mysql]
auto-rehash
# Remove the next comment character if you are not familiar with SQL
#safe-updates
default_character_set=utf8


[mysqlhotcopy]
interactive-timeout

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
[mysql@dao ~]$ /usr/local/mysql/scripts/mysql_install_db  --defaults-file=/etc/my.cnf --user=mysql --basedir=/usr/local/mysql/
Installing MySQL system tables...2018-06-29 05:54:08 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2018-06-29 05:54:08 0 [Note] Ignoring --secure-file-priv value as server is running with --bootstrap.
2018-06-29 05:54:08 0 [Note] /usr/local/mysql//bin/mysqld (mysqld 5.6.39-log) starting as process 19612 ...
OK

Filling help tables...2018-06-29 05:54:11 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2018-06-29 05:54:11 0 [Note] Ignoring --secure-file-priv value as server is running with --bootstrap.
2018-06-29 05:54:11 0 [Note] /usr/local/mysql//bin/mysqld (mysqld 5.6.39-log) starting as process 19636 ...
OK

To start mysqld at boot time you have to copy
support-files/mysql.server to the right place for your system

PLEASE REMEMBER TO SET A PASSWORD FOR THE MySQL root USER !
To do so, start the server, then issue the following commands:

  /usr/local/mysql//bin/mysqladmin -u root password 'new-password'
  /usr/local/mysql//bin/mysqladmin -u root -h dao.com password 'new-password'

Alternatively you can run:

  /usr/local/mysql//bin/mysql_secure_installation

which will also give you the option of removing the test
databases and anonymous user created by default.  This is
strongly recommended for production servers.

See the manual for more instructions.

You can start the MySQL daemon with:

  cd . ; /usr/local/mysql//bin/mysqld_safe &

You can test the MySQL daemon with mysql-test-run.pl

  cd mysql-test ; perl mysql-test-run.pl

Please report any problems at http://bugs.mysql.com/

The latest information about MySQL is available on the web at

  http://www.mysql.com

Support MySQL by buying support/licenses at http://shop.mysql.com

New default config file was created as /usr/local/mysql//my.cnf and
will be used by default by the server when you start it.
You may edit this file to change server settings

WARNING: Default config file /etc/my.cnf exists on the system
This file will be read by default by the MySQL server
If you do not want to use this, either remove it, or use the
--defaults-file argument to mysqld_safe when starting the server

```

## 启动Mysql

```sql

[mysql@dao ~]$  /usr/local/mysql//bin/mysqld_safe &
[1] 19665
[mysql@dao ~]$ 180629 05:57:44 mysqld_safe Logging to '/data/mysql3306/logs/mysql-error.log'.
180629 05:57:44 mysqld_safe Starting mysqld daemon with databases from /data/mysql3306/data

```

## 进入Mysql并修改权限

```sql

[mysql@dao ~]$ /usr/local/mysql/bin/mysql
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1
Server version: 5.6.39-log MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> update mysql.user set password=password('root') where user='root';
Query OK, 4 rows affected (0.02 sec)
Rows matched: 4  Changed: 4  Warnings: 0

mysql> delete from mysql.user where host='::1';
Query OK, 1 row affected (0.01 sec)

mysql> delete from mysql.user where user='';
Query OK, 2 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.01 sec)


```
