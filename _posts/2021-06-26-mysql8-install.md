---
layout: post
title: mysql8安装
date: 2020-06-26
categories: blog
tags: [MySQL]
description: hive表相关操作
---
### 创建用户
```
[root@etlbi ~]# useradd mysql8

```
### 创建目录

```

[root@etlbi ~]# mkdir -p /usr/local/mysql8
[root@etlbi ~]# mv /home/mysql8/mysql-8.0.25-linux-glibc2.12-x86_64.tar.xz  /usr/local/mysql8/
[root@etlbi ~]# chown -R mysql8:mysql8  /usr/local/mysql8
[root@etlbi ~]# mkdir -p /data/mysql8/data
[root@etlbi ~]# chown -R mysql8:mysql8 /data/mysql8/data/

```


### 解压

```
[mysql8@etlbi mysql8]$ xz -d mysql-8.0.25-linux-glibc2.12-x86_64.tar.xz 
[mysql8@etlbi mysql8]$ ls
mysql-8.0.25-linux-glibc2.12-x86_64.tar
[mysql8@etlbi mysql8]$ tar -xf mysql-8.0.25-linux-glibc2.12-x86_64.tar 
[mysql8@etlbi mysql8]$ ls
mysql-8.0.25-linux-glibc2.12-x86_64  mysql-8.0.25-linux-glibc2.12-x86_64.tar
[mysql8@etlbi mysql8]$ cd mysql-8.0.25-linux-glibc2.12-x86_64/
[mysql8@etlbi mysql-8.0.25-linux-glibc2.12-x86_64]$ mv * ../
[mysql8@etlbi mysql-8.0.25-linux-glibc2.12-x86_64]$ cd ..
[mysql8@etlbi mysql8]$ ls
bin  docs  include  lib  LICENSE  man  mysql-8.0.25-linux-glibc2.12-x86_64  mysql-8.0.25-linux-glibc2.12-x86_64.tar  README  share  support-files
[mysql8@etlbi mysql8]$ rm -rf mysql-8.0.25-linux-glibc2.12-x86_64.tar 
[mysql8@etlbi mysql8]$ rm -rf mysql-8.0.25-linux-glibc2.12-x86_64
[mysql8@etlbi mysql8]$ ls
bin  docs  include  lib  LICENSE  man  README  share  support-files

```


### 配置环境变量

```
[mysql8@etlbi ~]$ cat .bash_profile  
# .bash_profile

# Get the aliases and functions
if [ -f ~/.bashrc ]; then
        . ~/.bashrc
fi

# User specific environment and startup programs

PATH=/usr/local/mysql8/bin:$PATH:$HOME/.local/bin:$HOME/bin

export PATH
[mysql8@etlbi ~]$ which mysqld
/usr/local/mysql57/bin/mysqld

```


### 编辑配置文件


```

[mysql8@etlbi data]$ cat /home/mysql8/mysql8.cnf 
[mysqld]
#
# Remove leading # and set to the amount of RAM for the most important data
# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
# innodb_buffer_pool_size = 128M
#
# Remove leading # to turn on a very important data integrity option: logging
# changes to the binary log between backups.
# log_bin
log_bin=binlog

# Remove leading # to set options mainly useful for reporting servers.
# The server defaults are faster for transactions and fast SELECTs.
# Adjust sizes as needed, experiment to find the optimal values.
# join_buffer_size = 128M
# sort_buffer_size = 2M
# read_rnd_buffer_size = 2M
datadir=/data/mysql8/data
socket=/tmp/mysql8.sock
server-id=1
port=3366
#explicit_defaults_for_timestamp=true
#lower_case_table_names=1

# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

# Recommended in standard MySQL setup
#sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES

sql_mode=NO_ENGINE_SUBSTITUTION

explicit_defaults_for_timestamp=true
lower_case_table_names=1
character-set-client-handshake = FALSE
character-set-server = utf8mb4
collation-server = utf8mb4_bin
max_connections = 1000
max_allowed_packet = 10240000

[mysqld_safe]
log-error=/data/mysql8/mysqld.log
pid-file=/data/mysql8/mysqld.pid


[client]
default-character-set = utf8mb4

[mysql]
default-character-set = utf8mb4
[mysql8@etlbi data]$ 

```
### 初始化

```

[mysql8@etlbi data]$ mysqld --defaults-file=/home/mysql8/mysql8.cnf --user=mysql --initialize                
2021-06-26T09:14:27.405667Z 0 [Warning] [MY-010139] [Server] Changed limits: max_open_files: 1024 (requested 9010)
2021-06-26T09:14:27.405690Z 0 [Warning] [MY-010141] [Server] Changed limits: max_connections: 214 (requested 1000)
2021-06-26T09:14:27.405700Z 0 [Warning] [MY-010142] [Server] Changed limits: table_open_cache: 400 (requested 4000)
2021-06-26T09:14:27.406326Z 0 [Warning] [MY-011070] [Server] 'Disabling symbolic links using --skip-symbolic-links (or equivalent) is the default. Consider not using this option as it' is deprecated and will be removed in a future release.
2021-06-26T09:14:27.406547Z 0 [System] [MY-013169] [Server] /usr/local/mysql8/bin/mysqld (mysqld 8.0.25) initializing of server in progress as process 23121
2021-06-26T09:14:27.410435Z 0 [Warning] [MY-010122] [Server] One can only use the --user switch if running as root
2021-06-26T09:14:27.450909Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
2021-06-26T09:14:29.109555Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
2021-06-26T09:14:31.326020Z 6 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: 5iui>nwlHijt

```

注意最后一行 给出了临时密码，稍后使用临时密码进行首次登录。

### 启动mysql

```

[mysql8@etlbi ~]$ mysqld_safe --defaults-file=/home/mysql8/mysql8.cnf  &
[1] 26205
[mysql8@etlbi ~]$ 2021-06-26T09:28:38.982757Z mysqld_safe Logging to '/data/mysql8/mysqld.log'.
2021-06-26T09:28:39.047578Z mysqld_safe Starting mysqld daemon with databases from /data/mysql8/data

```
### 登录mysql并修改密码

```
[mysql8@etlbi ~]$ mysql -uroot -S /tmp/mysql8.sock -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 10
Server version: 8.0.25

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
mysql>  alter user user() identified by 'root' ;
Query OK, 0 rows affected (0.02 sec)

```
### 查看版本

```sql

mysql> select version() from dual;
+-----------+
| version() |
+-----------+
| 8.0.25    |
+-----------+
1 row in set (0.00 sec)

```