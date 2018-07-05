---
layout: post
title:  Redis主从搭建实录
date: 2018-01-03
categories: blog
tags: [Redis]
description: Redis主从搭建实录
---

# 主库端操作

##  主库配置文件

```sql
[root@bosenrui redis]# cat master.conf 
#master
#general
daemonize yes
pidfile /redis/redis6379.pid
port 6379
unixsocket /redis/redis6379.sock  
unixsocketperm 755           
timeout 0
tcp-keepalive 0
loglevel notice
logfile /redis/redis6379.log
syslog-enabled no
syslog-ident redis
syslog-facility local0
databases 16
requirepass etlbi
#snap

save 900 1                     
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename "dump6379.rdb"
dir /redis/

```
## 启动主库

```sql
/redis/bin/redis-server master.conf 
```

从库端操作

## 从库配置文件


```sql
[root@localhost redis]# cat slave.conf 
#general
daemonize yes
pidfile /tmp/redis6379.pid
port 6379
unixsocket /tmp/redis6379.sock
unixsocketperm 755
timeout 0
tcp-keepalive 0
loglevel notice
logfile /redis/redis6379.log
syslog-enabled no
syslog-ident redis
syslog-facility local0
databases 16
requirepass etlbi
#snap

save 900 1
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename "dump6379.rdb"
dir /redis/
#REPLICATION

slaveof 192.168.2.66 6379
masterauth etlbi
slave-serve-stale-data yes
slave-read-only yes
repl-ping-slave-period 10
repl-timeout 60
repl-disable-tcp-nodelay no
slave-priority 100


```
## 启动从库 

```sql

[root@localhost redis]# ./bin/redis-server slave.conf 

```
    
#   验证

## 主库操作

```sql
127.0.0.1:6379> dbsize
(integer) 0
127.0.0.1:6379> set a 100
OK
```

## 从库操作 

```sql
[root@localhost redis]# ./bin/redis-cli -h 127.0.0.1 -p 6379 -a etlbi
127.0.0.1:6379> dbsize
(integer) 1
127.0.0.1:6379> keys *
1) "a"
127.0.0.1:6379> get a
"100"

```
