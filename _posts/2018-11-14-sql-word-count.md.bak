---
layout: post
title: zookeeper分布式安装
date: 2018-11-13
categories: blog
tags: [bigdata]
description: zookeeper分布式安装
---


## 规划


节点名 | IP |角色 
 --- | --- | ---
node1 | 192.168.2.61 |
node2 | 192.168.2.62 | 
node3 | 192.168.2.63 | 


### 查看/etc/hosts 
- 三个节点都要操作

```sql

[root@node1 ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.2.61 node1
192.168.2.62 node2
192.168.2.63 node3

```

### Zookeeper解压

- 三个节点都要操作

```sql

[root@node1 ~]# tar -zxf zookeeper-3.4.5.tar.gz
[root@node1 ~]# ll -d zookeeper-3.4.5
drwxr-xr-x. 10 bd games 4096 Nov  5  2012 zookeeper-3.4.5
[root@node1 ~]# mv zookeeper-3.4.5 /zookeeper

```
### 编辑环境变量

- 三个节点都要操作

```sql

[root@node1 conf]# cat /etc/profile
# /etc/profile

# System wide environment and startup programs, for login setup
# Functions and aliases go in /etc/bashrc

# It's NOT a good idea to change this file unless you know what you
# are doing. It's much better to create a custom.sh shell script in
# /etc/profile.d/ to make custom changes to your environment, as this
# will prevent the need for merging in future updates.

pathmunge () {
    case ":${PATH}:" in
        *:"$1":*)
            ;;
        *)
            if [ "$2" = "after" ] ; then
                PATH=$PATH:$1
            else
                PATH=$1:$PATH
            fi
    esac
}


if [ -x /usr/bin/id ]; then
    if [ -z "$EUID" ]; then
        # ksh workaround
        EUID=`id -u`
        UID=`id -ru`
    fi
    USER="`id -un`"
    LOGNAME=$USER
    MAIL="/var/spool/mail/$USER"
fi

# Path manipulation
if [ "$EUID" = "0" ]; then
    pathmunge /sbin
    pathmunge /usr/sbin
    pathmunge /usr/local/sbin
else
    pathmunge /usr/local/sbin after
    pathmunge /usr/sbin after
    pathmunge /sbin after
fi

HOSTNAME=`/bin/hostname 2>/dev/null`
HISTSIZE=1000
if [ "$HISTCONTROL" = "ignorespace" ] ; then
    export HISTCONTROL=ignoreboth
else
    export HISTCONTROL=ignoredups
fi

export PATH USER LOGNAME MAIL HOSTNAME HISTSIZE HISTCONTROL

# By default, we want umask to get set. This sets it for login shell
# Current threshold for system reserved uid/gids is 200
# You could check uidgid reservation validity in
# /usr/share/doc/setup-*/uidgid file
if [ $UID -gt 199 ] && [ "`id -gn`" = "`id -un`" ]; then
    umask 002
else
    umask 022
fi

for i in /etc/profile.d/*.sh ; do
    if [ -r "$i" ]; then
        if [ "${-#*i}" != "$-" ]; then
            . "$i"
        else
            . "$i" >/dev/null 2>&1
        fi
    fi
done

export JAVA_HOME=/java8
export PATH=$JAVA_HOME/bin:$PATH:$HOME/bin

export HADOOP_HOME=/hadoop
export PATH=$JAVA_HOME/bin:$PATH:$HOME/bin:$HADOOP_HOME/bin:$JAVA_HOME/bin:$PATH:$HOME/sbin

export ZOOKEEPER_HOME=/zookeeper
export PATH=$PATH:$ZOOKEEPER_HOME/bin
unset i
unset -f pathmunge



```


### 修改配置文件

- 三个节点都要操作

```sql

[root@node1 bin]# cd  /zookeeper/conf/
[root@node1 conf]# ll
total 12
-rw-r--r--. 1 bd games  535 Oct  1  2012 configuration.xsl
-rw-r--r--. 1 bd games 2161 Oct  1  2012 log4j.properties
-rw-r--r--. 1 bd games  808 Oct  1  2012 zoo_sample.cfg
[root@node1 conf]# cp zoo_sample.cfg  zoo.cfg

[root@node1 conf]# vi zoo.cfg
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just
# example sakes.
dataDir=/zookeeper
# the port at which the clients will connect
clientPort=2181
#
# Be sure to read the maintenance section of the
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1
~
~


```


###  启动zookeeper

```sql

[root@node1 conf]# zkServer.sh  start
JMX enabled by default
Using config: /zookeeper/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED


```

### 查看zookeeper进程

```sql
[root@node1 conf]# jps
3876 QuorumPeerMain  -- 此进程为zookeeper进程
3894 Jps


```

### 使用客户端连接

```sql


[root@node1 conf]# zkCli.sh -server node1:2181
Connecting to node1:2181
2018-11-03 14:20:09,463 [myid:] - INFO  [main:Environment@100] - Client environment:zookeeper.version=3.4.5-1392090, built on 09/30/2012 17:52 GMT
2018-11-03 14:20:09,466 [myid:] - INFO  [main:Environment@100] - Client environment:host.name=node1
2018-11-03 14:20:09,467 [myid:] - INFO  [main:Environment@100] - Client environment:java.version=1.8.0_65
2018-11-03 14:20:09,467 [myid:] - INFO  [main:Environment@100] - Client environment:java.vendor=Oracle Corporation
2018-11-03 14:20:09,467 [myid:] - INFO  [main:Environment@100] - Client environment:java.home=/java8/jre
2018-11-03 14:20:09,467 [myid:] - INFO  [main:Environment@100] - Client environment:java.class.path=/zookeeper/bin/../build/classes:/zookeeper/bin/../build/lib/*.jar:/zookeeper/bin/../lib/slf4j-log4j12-1.6.1.jar:/zookeeper/bin/../lib/slf4j-api-1.6.1.jar:/zookeeper/bin/../lib/netty-3.2.2.Final.jar:/zookeeper/bin/../lib/log4j-1.2.15.jar:/zookeeper/bin/../lib/jline-0.9.94.jar:/zookeeper/bin/../zookeeper-3.4.5.jar:/zookeeper/bin/../src/java/lib/*.jar:/zookeeper/bin/../conf:
2018-11-03 14:20:09,468 [myid:] - INFO  [main:Environment@100] - Client environment:java.library.path=/usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib
2018-11-03 14:20:09,469 [myid:] - INFO  [main:Environment@100] - Client environment:java.io.tmpdir=/tmp
2018-11-03 14:20:09,472 [myid:] - INFO  [main:Environment@100] - Client environment:java.compiler=<NA>
2018-11-03 14:20:09,473 [myid:] - INFO  [main:Environment@100] - Client environment:os.name=Linux
2018-11-03 14:20:09,473 [myid:] - INFO  [main:Environment@100] - Client environment:os.arch=amd64
2018-11-03 14:20:09,473 [myid:] - INFO  [main:Environment@100] - Client environment:os.version=2.6.32-431.el6.x86_64
2018-11-03 14:20:09,473 [myid:] - INFO  [main:Environment@100] - Client environment:user.name=root
2018-11-03 14:20:09,474 [myid:] - INFO  [main:Environment@100] - Client environment:user.home=/root
2018-11-03 14:20:09,479 [myid:] - INFO  [main:Environment@100] - Client environment:user.dir=/zookeeper/conf
2018-11-03 14:20:09,484 [myid:] - INFO  [main:ZooKeeper@438] - Initiating client connection, connectString=node1:2181 sessionTimeout=30000 watcher=org.apache.zookeeper.ZooKeeperMain$MyWatcher@7aec35a
Welcome to ZooKeeper!
2018-11-03 14:20:09,549 [myid:] - INFO  [main-SendThread(node1:2181):ClientCnxn$SendThread@966] - Opening socket connection to server node1/192.168.2.61:2181. Will not attempt to authenticate using SASL (unknown error)
JLine support is enabled
2018-11-03 14:20:10,002 [myid:] - INFO  [main-SendThread(node1:2181):ClientCnxn$SendThread@849] - Socket connection established to node1/192.168.2.61:2181, initiating session
[zk: node1:2181(CONNECTING) 0] 2018-11-03 14:20:10,309 [myid:] - INFO  [main-SendThread(node1:2181):ClientCnxn$SendThread@1207] - Session establishment complete on server node1/192.168.2.61:2181, sessionid = 0x166dac6dfc00000, negotiated timeout = 30000

WATCHER::

WatchedEvent state:SyncConnected type:None path:null

[zk: node1:2181(CONNECTED) 0]


```

### 获取帮助 

```sql

[zk: node1:2181(CONNECTED) 0] help
ZooKeeper -server host:port cmd args
        stat path [watch]
        set path data [version]
        ls path [watch]
        delquota [-n|-b] path
        ls2 path [watch]
        setAcl path acl
        setquota -n|-b val path
        history
        redo cmdno
        printwatches on|off
        delete path [version]
        sync path
        listquota path
        rmr path
        get path [watch]
        create [-s] [-e] path data acl
        addauth scheme auth
        quit
        getAcl path
        close
        connect host:port


```

### 使用ls命令查看路径


```sql

[zk: node1:2181(CONNECTED) 2] ls /
[zookeeper]
[zk: node1:2181(CONNECTED) 3] ls /zookeeper
[quota]
[zk: node1:2181(CONNECTED) 4] ls /zookeeper/quota
[]


```

### 使用create命令创建路径并关联数据

```sql

[zk: node1:2181(CONNECTED) 5] create /dao etlbi.com  
Created /dao
[zk: node1:2181(CONNECTED) 6] ls /
[dao, zookeeper]
[zk: node1:2181(CONNECTED) 7] get /dao
etlbi.com
cZxid = 0x2
ctime = Sat Nov 03 14:31:32 EDT 2018 --创建时间
mZxid = 0x2
mtime = Sat Nov 03 14:31:32 EDT 2018
pZxid = 0x2
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 9  --数据长度
numChildren = 0


```

## zookeeper 分布式安装

### 修改配置文件 

- 三台机器都要执行


```sql


[root@node1 ~]# vi /zookeeper/conf/zoo.cfg
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just
# example sakes.
dataDir=/zookeeper/data
# the port at which the clients will connect
clientPort=2181
#
# Be sure to read the maintenance section of the
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1


server.1=node1:2888:3888
server.2=node2:2888:3888
server.3=node3:2888:3888


```

### 添加myid文件

- 在zoo.cfg 文件中配置的datadi下建立my.id

```sql

[root@node1 data]# cat /zookeeper/data/myid
1

```


### 启动zookeeper并查看状态

- node1 

```sql

[root@node1 data]# zkServer.sh start
JMX enabled by default
Using config: /zookeeper/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED

[root@node1 data]# jps
3059 QuorumPeerMain
3079 Jps

[root@node1 data]# jps
3059 QuorumPeerMain
3079 Jps
[root@node1 data]# zkServer.sh  status
JMX enabled by default
Using config: /zookeeper/bin/../conf/zoo.cfg
Error contacting service. It is probably not running. -- 此处显示连接失败是因为zookeeper要大多数节点可用才对外提供服务

```

- node2 

```sql


[root@node2 zookeeper]# zkServer.sh start
JMX enabled by default
Using config: /zookeeper/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
[root@node2 zookeeper]# jps
5185 Jps
5158 QuorumPeerMain
[root@node2 zookeeper]# zkServer.sh status
JMX enabled by default
Using config: /zookeeper/bin/../conf/zoo.cfg
Mode: leader



```

-node3 

```sql

[root@node3 zookeeper]# zkServer.sh start
JMX enabled by default
Using config: /zookeeper/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
[root@node3 zookeeper]# jps
2464 Jps
2439 QuorumPeerMain
[root@node3 zookeeper]# zkServer.sh status
JMX enabled by default
Using config: /zookeeper/bin/../conf/zoo.cfg
Mode: follower


```

### 高可用实验

- 停掉leader 


```sql

[root@node2 zookeeper]# zkServer.sh status
JMX enabled by default
Using config: /zookeeper/bin/../conf/zoo.cfg
Mode: leader
[root@node2 zookeeper]# zkServer.sh stop
JMX enabled by default
Using config: /zookeeper/bin/../conf/zoo.cfg
Stopping zookeeper ... STOPPED


```

- 到其他节点查询新选出的leader

```sql

[root@node3 zookeeper]# zkServer.sh status
JMX enabled by default
Using config: /zookeeper/bin/../conf/zoo.cfg
Mode: follower
[root@node3 zookeeper]# zkServer.sh status
JMX enabled by default
Using config: /zookeeper/bin/../conf/zoo.cfg
Mode: leader  -- 可以看出node3 已经当选新的leader


```

```sql

[root@node1 data]# zkServer.sh status
JMX enabled by default
Using config: /zookeeper/bin/../conf/zoo.cfg
Mode: follower
[root@node1 data]# zkServer.sh status
JMX enabled by default
Using config: /zookeeper/bin/../conf/zoo.cfg
Mode: follower   -- node1 仍旧是follower


```