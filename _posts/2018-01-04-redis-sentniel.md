---
layout: post
title:  Redis 哨兵搭建与管理
date: 2018-01-04
categories: blog
tags: [Redis]
description: 哨兵搭建与管理
---


# 主节点操作

## 主节点配置文件

```sql
[root@server ~]# cat  /master/master.conf 
daemonize yes
pidfile /master/redis6379.pid
port 6379
unixsocket /master/redis6379.sock  
unixsocketperm 755           
timeout 0
tcp-keepalive 0
loglevel notice
logfile /master/redis6379.log
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
dir /master/

```


## 主节点启动

```sql
[root@server ~]# redis-server  /master/master.conf 
```

## 主节点日志
```sql
[root@server ~]# cat /master/redis6379.log 

8540:M 25 Dec 22:18:33.475 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
8540:M 25 Dec 22:18:33.475 # Server started, Redis version 3.2.9
8540:M 25 Dec 22:18:33.475 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
8540:M 25 Dec 22:18:33.475 * The server is now ready to accept connections on port 6379
8540:M 25 Dec 22:18:33.475 * The server is now ready to accept connections at /master/redis6379.sock

```


# 从节点1

## 从节点配置文件

```sql
daemonize yes
pidfile "/slave1/redis6380.pid"
port 6380
unixsocket "/tmp/redis6380.sock"
unixsocketperm 755
timeout 0
tcp-keepalive 0
loglevel notice
logfile "/slave1/redis6380.log"
syslog-enabled no
syslog-ident "redis"
syslog-facility local0
databases 16
requirepass "etlbi"
#snap

save 900 1
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename "dump6380.rdb"
dir "/slave1"
#REPLICATION

slaveof 127.0.0.1 6379
masterauth "etlbi"
slave-serve-stale-data yes
slave-read-only yes
repl-ping-slave-period 10
repl-timeout 60
repl-disable-tcp-nodelay no
slave-priority 100
```



## 从节点1启动

```sql
[root@server ~]# redis-server  /slave1/slave1.conf

```
## 从节点1日志

```sql

8943:S 25 Dec 22:28:34.184 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
8943:S 25 Dec 22:28:34.184 # Server started, Redis version 3.2.9
8943:S 25 Dec 22:28:34.184 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
8943:S 25 Dec 22:28:34.184 * The server is now ready to accept connections on port 6380
8943:S 25 Dec 22:28:34.184 * The server is now ready to accept connections at /tmp/redis6380.sock
8943:S 25 Dec 22:28:34.184 * Connecting to MASTER 127.0.0.1:6379
8943:S 25 Dec 22:28:34.185 * MASTER <-> SLAVE sync started
8943:S 25 Dec 22:28:34.185 * Non blocking connect for SYNC fired the event.
8943:S 25 Dec 22:28:34.185 * Master replied to PING, replication can continue...
8943:S 25 Dec 22:28:34.185 * Partial resynchronization not possible (no cached master)
8943:S 25 Dec 22:28:34.185 * Full resync from master: 94f36ce34c9bd9c65eb4686702a98bb98316afa5:1
8943:S 25 Dec 22:28:34.290 * MASTER <-> SLAVE sync: receiving 76 bytes from master
8943:S 25 Dec 22:28:34.290 * MASTER <-> SLAVE sync: Flushing old data
8943:S 25 Dec 22:28:34.290 * MASTER <-> SLAVE sync: Loading DB in memory
8943:S 25 Dec 22:28:34.290 * MASTER <-> SLAVE sync: Finished with success

```
# 从节点2 

## 从节点2配置文件

```sql
#general
daemonize yes
pidfile "/slave2/redis6381.pid"
port 6381
unixsocket "/tmp/redis6381.sock"
unixsocketperm 755
timeout 0
tcp-keepalive 0
loglevel notice
logfile "/slave2/redis6381.log"
syslog-enabled no
syslog-ident "redis"
syslog-facility local0
databases 16
requirepass "etlbi"
#snap

save 900 1
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename "dump6380.rdb"
dir "/slave2"
#REPLICATION

slaveof 127.0.0.1 6379
masterauth "etlbi"
slave-serve-stale-data yes
slave-read-only yes
repl-ping-slave-period 10
repl-timeout 60
repl-disable-tcp-nodelay no
slave-priority 100

```



## 从节点2启动

```sql
[root@server ~]# redis-server /slave2/slave2.conf
```

## 从节点2日志

```sql
9258:S 25 Dec 22:36:06.061 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
9258:S 25 Dec 22:36:06.061 # Server started, Redis version 3.2.9
9258:S 25 Dec 22:36:06.061 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
9258:S 25 Dec 22:36:06.061 * The server is now ready to accept connections on port 6381
9258:S 25 Dec 22:36:06.061 * The server is now ready to accept connections at /tmp/redis6381.sock
9258:S 25 Dec 22:36:06.061 * Connecting to MASTER 127.0.0.1:6379
9258:S 25 Dec 22:36:06.061 * MASTER <-> SLAVE sync started
9258:S 25 Dec 22:36:06.061 * Non blocking connect for SYNC fired the event.
9258:S 25 Dec 22:36:06.061 * Master replied to PING, replication can continue...
9258:S 25 Dec 22:36:06.062 * Partial resynchronization not possible (no cached master)
9258:S 25 Dec 22:36:06.063 * Full resync from master: 94f36ce34c9bd9c65eb4686702a98bb98316afa5:631
9258:S 25 Dec 22:36:06.115 * MASTER <-> SLAVE sync: receiving 76 bytes from master
9258:S 25 Dec 22:36:06.115 * MASTER <-> SLAVE sync: Flushing old data
9258:S 25 Dec 22:36:06.115 * MASTER <-> SLAVE sync: Loading DB in memory
9258:S 25 Dec 22:36:06.115 * MASTER <-> SLAVE sync: Finished with success
```
# 哨兵节点

## 哨兵1 

### 配置文件

```sql

port 26379


dir "/tmp"



sentinel monitor mymaster 127.0.0.1 6381 2

sentinel auth-pass mymaster etlbi

sentinel config-epoch mymaster 1

sentinel leader-epoch mymaster 1

daemonize yes

logfile "/sentinel1/sentinel1.log"

```
### 哨兵1启动


```sql
[root@server ~]# redis-sentinel /sentinel1/sentinel1.conf 

```
### 哨兵1日志
```sql

9981:X 25 Dec 22:53:28.596 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
9981:X 25 Dec 22:53:28.596 # Sentinel ID is c37a2eaaf23e18ab5c564aa749821bd31a869e90
9981:X 25 Dec 22:53:28.596 # +monitor master mymaster 127.0.0.1 6379 quorum 2

```

## 启动后配置文件

```sql
port 26379

dir "/tmp"

sentinel myid c37a2eaaf23e18ab5c564aa749821bd31a869e90

sentinel monitor mymaster 127.0.0.1 6379 2

sentinel auth-pass mymaster etlbi

sentinel config-epoch mymaster 1

sentinel leader-epoch mymaster 1

daemonize yes

logfile "/sentinel1/sentinel1.log"
# Generated by CONFIG REWRITE
sentinel known-slave mymaster 127.0.0.1 6380
sentinel known-slave mymaster 127.0.0.1 6381

```
## 哨兵2 

### 配置文件

```sql
port 26380

dir "/tmp"



sentinel monitor mymaster 127.0.0.1 6381 2

sentinel auth-pass mymaster etlbi

sentinel config-epoch mymaster 1

daemonize yes

logfile "/sentinel2/sentinel2.log"


```


### 哨兵2启动
```sql
[root@server ~]# redis-sentinel /sentinel2/sentinel2.conf 
```

### 哨兵2日志
```sql
10045:X 25 Dec 22:55:09.076 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
10045:X 25 Dec 22:55:09.076 # Sentinel ID is e5d051606237351f06e6ce3efd555345a4aa5e7a
10045:X 25 Dec 22:55:09.076 # +monitor master mymaster 127.0.0.1 6379 quorum 2
10045:X 25 Dec 22:55:09.077 * +slave slave 127.0.0.1:6380 127.0.0.1 6380 @ mymaster 127.0.0.1 6379
10045:X 25 Dec 22:55:09.089 * +slave slave 127.0.0.1:6381 127.0.0.1 6381 @ mymaster 127.0.0.1 6379
10045:X 25 Dec 22:55:09.117 * +sentinel sentinel c37a2eaaf23e18ab5c564aa749821bd31a869e90 127.0.0.1 26379 @ mymaster 127.0.0.1 6379
```
## 哨兵3 

### 配置文件

```sql

port 26381

dir "/tmp"


sentinel monitor mymaster 127.0.0.1 6381 2

sentinel auth-pass mymaster etlbi

sentinel config-epoch mymaster 1

daemonize yes

logfile "/sentinel3/sentinel3.log"
sentinel leader-epoch mymaster 1


```

### 哨兵3启动
```sql
[root@server ~]# redis-sentinel /sentinel3/sentinel3.conf 

```
### 哨兵3日志

```sql
10108:X 25 Dec 22:56:22.493 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
10108:X 25 Dec 22:56:22.493 # Sentinel ID is b4e4bec80853ae4ae75ca372c179dae257d0e9ca
10108:X 25 Dec 22:56:22.493 # +monitor master mymaster 127.0.0.1 6379 quorum 2
10108:X 25 Dec 22:56:22.494 * +slave slave 127.0.0.1:6380 127.0.0.1 6380 @ mymaster 127.0.0.1 6379
10108:X 25 Dec 22:56:22.501 * +slave slave 127.0.0.1:6381 127.0.0.1 6381 @ mymaster 127.0.0.1 6379
10108:X 25 Dec 22:56:22.664 * +sentinel sentinel e5d051606237351f06e6ce3efd555345a4aa5e7a 127.0.0.1 26380 @ mymaster 127.0.0.1 6379
10108:X 25 Dec 22:56:23.022 * +sentinel sentinel c37a2eaaf23e18ab5c564aa749821bd31a869e90 127.0.0.1 26379 @ mymaster 127.0.0.1 6379

```

# 相关操作

## 查看本节点哨兵信息

```sql

127.0.0.1:26379> info sentinel
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=127.0.0.1:6379,slaves=2,sentinels=3


```

## 查看其他哨兵信息

```sql

127.0.0.1:26379> sentinel sentinels mymaster
1)  1) "name"
    2) "e5d051606237351f06e6ce3efd555345a4aa5e7a"
    3) "ip"
    4) "127.0.0.1"
    5) "port"
    6) "26380"
    7) "runid"
    8) "e5d051606237351f06e6ce3efd555345a4aa5e7a"
    9) "flags"
   10) "sentinel"
   11) "link-pending-commands"
   12) "0"
   13) "link-refcount"
   14) "1"
   15) "last-ping-sent"
   16) "0"
   17) "last-ok-ping-reply"
   18) "909"
   19) "last-ping-reply"
   20) "909"
   21) "down-after-milliseconds"
   22) "30000"
   23) "last-hello-message"
   24) "539"
   25) "voted-leader"
   26) "?"
   27) "voted-leader-epoch"
   28) "0"
2)  1) "name"
    2) "b4e4bec80853ae4ae75ca372c179dae257d0e9ca"
    3) "ip"
    4) "127.0.0.1"
    5) "port"
    6) "26381"
    7) "runid"
    8) "b4e4bec80853ae4ae75ca372c179dae257d0e9ca"
    9) "flags"
   10) "sentinel"
   11) "link-pending-commands"
   12) "0"
   13) "link-refcount"
   14) "1"
   15) "last-ping-sent"
   16) "0"
   17) "last-ok-ping-reply"
   18) "909"
   19) "last-ping-reply"
   20) "909"
   21) "down-after-milliseconds"
   22) "30000"
   23) "last-hello-message"
   24) "760"
   25) "voted-leader"
   26) "?"
   27) "voted-leader-epoch"
   28) "0"
   
```


## 显示全部主节点信息

```sql
[root@server ~]# redis-cli -h 127.0.0.1 -p 26379
127.0.0.1:26379> sentinel masters
1)  1) "name"
    2) "mymaster"
    3) "ip"
    4) "127.0.0.1"
    5) "port"
    6) "6379"
    7) "runid"
    8) "18f3ef75b63ea5abefb9968b9e3a58ad43e4d416"
    9) "flags"
   10) "master"
   11) "link-pending-commands"
   12) "0"
   13) "link-refcount"
   14) "1"
   15) "last-ping-sent"
   16) "0"
   17) "last-ok-ping-reply"
   18) "142"
   19) "last-ping-reply"
   20) "142"
   21) "down-after-milliseconds"
   22) "30000"
   23) "info-refresh"
   24) "8918"
   25) "role-reported"
   26) "master"
   27) "role-reported-time"
   28) "219872"
   29) "config-epoch"
   30) "1"
   31) "num-slaves"
   32) "2"
   33) "num-other-sentinels"
   34) "2"
   35) "quorum"
   36) "2"
   37) "failover-timeout"
   38) "180000"
   39) "par
```

## 显示单一主节点信息

```sql

127.0.0.1:26379> sentinel master mymaster
 1) "name"
 2) "mymaster"
 3) "ip"
 4) "127.0.0.1"
 5) "port"
 6) "6379"
 7) "runid"
 8) "18f3ef75b63ea5abefb9968b9e3a58ad43e4d416"
 9) "flags"
10) "master"
11) "link-pending-commands"
12) "0"
13) "link-refcount"
14) "1"
15) "last-ping-sent"
16) "0"
17) "last-ok-ping-reply"
18) "727"
19) "last-ping-reply"
20) "727"
21) "down-after-milliseconds"
22) "30000"
23) "info-refresh"
24) "4032"
25) "role-reported"
26) "master"
27) "role-reported-time"
28) "545953"
29) "config-epoch"
30) "1"
31) "num-slaves"
32) "2"
33) "num-other-sentinels"
34) "2"
35) "quorum"
36) "2"
37) "failover-timeout"
38) "180000"
39) "parallel-syncs"
40) "1"

```

## 显示主节点IP和端口

```sql
127.0.0.1:26379> sentinel get-master-addr-by-name mymaster
1) "127.0.0.1"
2) "6379"
```


## 显示从节点信息

```sql
127.0.0.1:26379> sentinel slaves mymaster
1)  1) "name"
    2) "127.0.0.1:6380"
    3) "ip"
    4) "127.0.0.1"
    5) "port"
    6) "6380"
    7) "runid"
    8) "42b9acb4ad96860f3e86ad14cba40422f8a1917e"
    9) "flags"
   10) "slave"
   11) "link-pending-commands"
   12) "0"
   13) "link-refcount"
   14) "1"
   15) "last-ping-sent"
   16) "0"
   17) "last-ok-ping-reply"
   18) "724"
   19) "last-ping-reply"
   20) "724"
   21) "down-after-milliseconds"
   22) "30000"
   23) "info-refresh"
   24) "9127"
   25) "role-reported"
   26) "slave"
   27) "role-reported-time"
   28) "671561"
   29) "master-link-down-time"
   30) "0"
   31) "master-link-status"
   32) "ok"
   33) "master-host"
   34) "127.0.0.1"
   35) "master-port"
   36) "6379"
   37) "slave-priority"
   38) "100"
   39) "slave-repl-offset"
   40) "128005"
2)  1) "name"
    2) "127.0.0.1:6381"
    3) "ip"
    4) "127.0.0.1"
    5) "port"
    6) "6381"
    7) "runid"
    8) "c0515dd42d9a9dfd5ec799de879b16da2df55e87"
    9) "flags"
   10) "slave"
   11) "link-pending-commands"
   12) "0"
   13) "link-refcount"
   14) "1"
   15) "last-ping-sent"
   16) "0"
   17) "last-ok-ping-reply"
   18) "724"
   19) "last-ping-reply"
   20) "724"
   21) "down-after-milliseconds"
   22) "30000"
   23) "info-refresh"
   24) "9126"
   25) "role-reported"
   26) "slave"
   27) "role-reported-time"
   28) "671561"
   29) "master-link-down-time"
   30) "0"
   31) "master-link-status"
   32) "ok"
   33) "master-host"
   34) "127.0.0.1"
   35) "master-port"
   36) "6379"
   37) "slave-priority"
   38) "100"
   39) "slave-repl-offset"
   40) "128005"

```

