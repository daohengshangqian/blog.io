---
layout: post
title: hadoop完全分布式安装
date: 2018-11-24
categories: blog
tags: [bigdata]
description: hadoop完全分布式安装
---

# 规划

主机名 | IP地址 | HDFS角色
--- | --- | ---
node1 | 192.168.2.61 | namenode
node2 | 192.168.2.62 | namenode
node3 | 192.168.2.63 | datanode
node4 | 192.168.2.64 | datanode
node5 | 192.168.2.65 | datanode

# 系统配置

## 关闭selinux

- 所有节点均需操作

```sql

[root@node1 ~]# setenforce 0
setenforce: SELinux is disabled
[root@node1 ~]# cat  /etc/selinux/config

# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=disabled
# SELINUXTYPE= can take one of these two values:
#     targeted - Targeted processes are protected,
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted



```

## 关闭防火墙

- 所有节点均需操作

```sql

[root@node1 ~]# service iptables stop
iptables: Setting chains to policy ACCEPT: filter          [  OK  ]
iptables: Flushing firewall rules:                         [  OK  ]
iptables: Unloading modules:                               [  OK  ]

[root@node1 ~]# chkconfig  iptables off
```

## 配置主机名解析

- 所有节点均需操作

```sql

[root@node1 ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.2.61 node1
192.168.2.62 node2
192.168.2.63 node3
192.168.2.64 node4
192.168.2.65 node5


```
## 配置节点互信

- 单一节点操作

```sql

[root@node1 ~]# ./sshUserSetup.sh -user root  -hosts "node1 node2 node3 node4 node5" -noPromptPassphrase -advanced -exverify -confirm

```
##  配置NTP

- ntp服务节点操作

```sql

[root@node1 ~]# cat /etc/ntp.conf
# For more information about this file, see the man pages
# ntp.conf(5), ntp_acc(5), ntp_auth(5), ntp_clock(5), ntp_misc(5), ntp_mon(5).

driftfile /var/lib/ntp/drift

# Permit time synchronization with our time source, but do not
# permit the source to query or modify the service on this system.
restrict default kod nomodify notrap nopeer noquery
restrict -6 default kod nomodify notrap nopeer noquery

# Permit all access over the loopback interface.  This could
# be tightened as well, but to do so would effect some of
# the administrative functions.
restrict 127.0.0.1
restrict -6 ::1
restrict 192.168.2.0 mask 255.255.255.0 notrap nomodify
# Hosts on local network are less restricted.
#restrict 192.168.1.0 mask 255.255.255.0 nomodify notrap
server 127.127.1.0 fuge
127.127.1.0 stratum 8
# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (http://www.pool.ntp.org/join.html).
#server 0.centos.pool.ntp.org iburst
#server 1.centos.pool.ntp.org iburst
#server 2.centos.pool.ntp.org iburst
#server 3.centos.pool.ntp.org iburst

#broadcast 192.168.1.255 autokey        # broadcast server
#broadcastclient                        # broadcast client
#broadcast 224.0.1.1 autokey            # multicast server
#multicastclient 224.0.1.1              # multicast client
#manycastserver 239.255.254.254         # manycast server
#manycastclient 239.255.254.254 autokey # manycast client

broadcastdelay 0.008

# Enable public key cryptography.
#crypto

includefile /etc/ntp/crypto/pw

# Key file containing the keys and key identifiers used when operating
# with symmetric key cryptography.
keys /etc/ntp/keys

# Specify the key identifiers which are trusted.
#trustedkey 4 8 42

# Specify the key identifier to use with the ntpdc utility.
#requestkey 8

# Specify the key identifier to use with the ntpq utility.
#controlkey 8

# Enable writing of statistics records.
#statistics clockstats cryptostats loopstats peerstats


```

# 安装JAVA

-单一节点操作

```sql

[root@node1 ~]# tar -zxf jdk-8u65-linux-x64.tar.gz
[root@node1 ~]# mv jdk1.8.0_65 /java8

```

- 分发java到其他节点

```sql

[root@node1 ~]# scp -r /java8/ node2:/
[root@node1 ~]# scp -r /java8/ node3:/
[root@node1 ~]# scp -r /java8/ node4:/
[root@node1 ~]# scp -r /java8/ node5:/


```

# 安装配置zookeeper 


## 上传安装zookeeper

- 单一节点操作

```sql

[root@node1 ~]# tar -zxf zookeeper-3.4.5.tar.gz
[root@node1 ~]# ll -d zookeeper-3.4.5
drwxr-xr-x. 10 bd games 4096 Nov  5  2012 zookeeper-3.4.5
[root@node1 ~]# mv zookeeper-3.4.5 /zookeeper


```

## 生成修改zookeeper文件


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
# 修改此处配置
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

## 分发zookeeper 

- 单一节点操作

```sql

[root@node1 ~]# scp -r /zookeeper node2:/
[root@node1 ~]# scp -r /zookeeper node3:/
[root@node1 ~]# scp -r /zookeeper node4:/
[root@node1 ~]# scp -r /zookeeper node5:/


```


## 修改zooker myid

 
--每个节点都需要修改

```sql

[root@node1 ~]# ssh node1 cat /zookeeper/data/myid
1
[root@node1 ~]# ssh node2 cat /zookeeper/data/myid
2
[root@node1 ~]# ssh node3 cat /zookeeper/data/myid
3
[root@node1 ~]# ssh node4 cat /zookeeper/data/myid
4
[root@node1 ~]# ssh node5 cat /zookeeper/data/myid
5


```




# 安装Hadoop


## 上传解压hadoop


```sql


[root@node1 ~]# tar -zxf hadoop-2.7.3.tar.gz  -C /r
[root@node1 ~]# ll /
total 110
dr-xr-xr-x.   2 root  root   4096 Aug  8 03:25 bin
dr-xr-xr-x.   5 root  root   1024 Nov 29  2015 boot
drwxr-xr-x.  19 root  root   3840 Nov  2 16:38 dev
drwxr-xr-x. 117 root  root  12288 Nov  2 17:58 etc
drwxr-xr-x.   9 root  root   4096 Aug 18  2016 hadoop-2.7.3
drwxr-xr-x.   5 root  root   4096 Nov  2 16:47 home
drwxr-xr-x.   8 uucp    143  4096 Oct  7  2015 java8
dr-xr-xr-x.  11 root  root   4096 Aug  7 22:59 lib
dr-xr-xr-x.   9 root  root  12288 Aug  8 03:25 lib64
drwx------.   2 root  root  16384 Nov 29  2015 lost+found
drwxr-xr-x.   3 root  root   4096 Nov  2 16:39 media
drwxr-xr-x.   2 root  root      0 Nov  2 16:38 misc
drwxr-xr-x.   2 root  root   4096 Sep 23  2011 mnt
drwxr-xr-x.   3 mysql mysql  4096 Sep 21 16:37 mysql1
drwxr-xr-x.   2 root  root      0 Nov  2 16:38 net
drwxr-xr-x.   3 root  root   4096 Nov 29  2015 opt
dr-xr-xr-x. 161 root  root      0 Nov  2 16:38 proc
dr-xr-x---.  31 root  root   4096 Nov  2 18:08 root
dr-xr-xr-x.   2 root  root  12288 Aug  8 03:25 sbin
drwxr-xr-x.   7 root  root      0 Nov  2 16:38 selinux
drwxr-xr-x.   2 root  root   4096 Sep 23  2011 srv
drwxr-xr-x.  13 root  root      0 Nov  2 16:38 sys
drwxrwxrwt.  20 root  root   4096 Nov  2 17:59 tmp
drwxr-xr-x.  13 root  root   4096 Nov 29  2015 usr
drwxr-xr-x.  22 root  root   4096 Nov 29  2015 var
[root@node1 ~]# mv hadoop-2.7.3.tar.gz  hadoop

```

## 对hadoop 进行配置

- 单一节点操作 

###  core-site.xml

```sql

[root@node1 ~]# cd /hadoop/etc/hadoop/
[root@node1 hadoop]# cat core-site.xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!--
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->

<!-- Put site-specific property overrides in this file. -->

<configuration>

<!-- 指定hdfs的nameservices名称为mycluster，与hdfs-site.xml的HA配置相同 -->
  <property>
      <name>fs.defaultFS</name>
      <value>hdfs://mycluster</value>
  </property>

  <!-- 指定缓存文件存储的路径 -->
  <property>
      <name>hadoop.tmp.dir</name>
      <value>/hadoop/tmp/dfs/tmp</value>
  </property>

  <!-- 配置hdfs文件被永久删除前保留的时间（单位：分钟），默认值为0表明垃圾回收站功能关闭 -->
  <property>
      <name>fs.trash.interval</name>
      <value>0</value>
  </property>

  <!-- 指定zookeeper地址，配置HA时需要 -->
  <property>
      <name>ha.zookeeper.quorum</name>
      <value>node1:2181,node2:2181,node3:2181,node4:2181,node5:2181</value>
  </property>
</configuration>


```

### hdfs-site.xml

```sql

[root@node1 hadoop]# cat hdfs-site.xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!--
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->

<!-- Put site-specific property overrides in this file. -->

<configuration>
<!-- 指定hdfs元数据日志文件存储的路径 -->
  <property>
      <name>dfs.namenode.name.dir</name>
      <value>/hadoop/tmp/dfs/namenode</value>
  </property>

  <!-- 指定hdfs数据临时文件存储的路径 -->
  <property>
      <name>dfs.datanode.data.dir</name>
      <value>/hadoop/tmp/dfs/datanode</value>
  </property>

  <!-- 数据备份的个数 -->
  <property>
      <name>dfs.replication</name>
      <value>3</value>
  </property>

  <!-- 关闭权限验证 -->
  <property>
      <name>dfs.permissions.enabled</name>
      <value>false</value>
  </property>

  <!-- 开启WebHDFS功能（基于REST的接口服务） -->
  <property>
      <name>dfs.webhdfs.enabled</name>
      <value>true</value>
  </property>

  <!-- //////////////以下为HDFS HA的配置////////////// -->
  <!-- 指定hdfs的nameservices名称为mycluster -->
  <property>
      <name>dfs.nameservices</name>
      <value>mycluster</value>
  </property>

  <!-- 指定mycluster的两个namenode的名称分别为nn1,nn2 -->
  <property>
      <name>dfs.ha.namenodes.mycluster</name>
      <value>nn1,nn2</value>
  </property>

  <!-- 配置nn1,nn2的rpc通信端口 -->
  <property>
      <name>dfs.namenode.rpc-address.mycluster.nn1</name>
      <value>node1:8020</value>
      </property>

  <property>
      <name>dfs.namenode.rpc-address.mycluster.nn2</name>
      <value>node2:8020</value>
  </property>

  <!-- 配置nn1,nn2的http通信端口 -->
  <property>
      <name>dfs.namenode.http-address.mycluster.nn1</name>
      <value>node1:50070</value>
  </property>

  <property>
      <name>dfs.namenode.http-address.mycluster.nn2</name>
      <value>node2:50070</value>
  </property>

  <!-- 指定namenode元数据存储在journalnode中的路径 -->
  <property>
      <name>dfs.namenode.shared.edits.dir</name>
      <value>qjournal://node1:8485;node2:8485;node3:8485;node4:8485;node5:8485/mycluster</value>
  </property>

  <!-- 指定journalnode日志文件存储的路径 -->
  <property>
      <name>dfs.journalnode.edits.dir</name>
      <value>/hadoop/tmp/dfs/journal</value>
  </property>

  <!-- 指定HDFS客户端连接active namenode的java类 -->
  <property>
      <name>dfs.client.failover.proxy.provider.mycluster</name>
      <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
  </property>

  <!-- 配置隔离机制为ssh -->
  <property>
      <name>dfs.ha.fencing.methods</name>
      <value>sshfence</value>
  </property>

  <!-- 指定秘钥的位置 -->
  <property>
      <name>dfs.ha.fencing.ssh.private-key-files</name>
      <value>/root/.ssh/id_rsa</value>
   </property>

  <!-- 开启自动故障转移 -->
  <property>
      <name>dfs.ha.automatic-failover.enabled</name>
      <value>true</value>
  </property>

</configuration>


```

###  mapred-site.xml

```sql


[root@node1 hadoop]# cat mapred-site.xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!--
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->

<!-- Put site-specific property overrides in this file. -->

<configuration>

<!-- 指定MapReduce计算框架使用YARN -->
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>

  <!-- 指定jobhistory server的rpc地址 -->
  <property>
    <name>mapreduce.jobhistory.address</name>
    <value>name1:10020</value>
  </property>

  <!-- 指定jobhistory server的http地址 -->
  <property>
    <name>mapreduce.jobhistory.webapp.address</name>
    <value>name1:19888</value>
  </property>

  <!-- 开启uber模式（针对小作业的优化） -->
  <property>
    <name>mapreduce.job.ubertask.enable</name>
    <value>true</value>
  </property>

  <!-- 配置启动uber模式的最大map数 -->
  <property>
    <name>mapreduce.job.ubertask.maxmaps</name>
    <value>9</value>
  </property>

  <!-- 配置启动uber模式的最大reduce数 -->
  <property>
    <name>mapreduce.job.ubertask.maxreduces</name>
    <value>1</value>
  </property>

</configuration>


```

### yarn-site.xml

```sql

[root@node1 hadoop]# cat yarn-site.xml
<?xml version="1.0"?>
<!--
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->
<configuration>

<!-- Site specific YARN configuration properties -->

<!-- NodeManager上运行的附属服务，需配置成mapreduce_shuffle才可运行MapReduce程序 -->
  <property>
        <name>yarn.nodemanager.aux-services</name>
              <value>mapreduce_shuffle</value>
                </property>

  <!-- 配置Web Application Proxy安全代理（防止yarn被攻击） -->
  <property>
        <name>yarn.web-proxy.address</name>
              <value>node2:8888</value>
                </property>

  <!-- 开启日志 -->
  <property>
        <name>yarn.log-aggregation-enable</name>
              <value>true</value>
                </property>

  <!-- 配置日志删除时间为7天，-1为禁用，单位为秒 -->
  <property>
        <name>yarn.log-aggregation.retain-seconds</name>
              <value>604800</value>
                </property>

  <!-- 修改日志目录 -->
  <property>
        <name>yarn.nodemanager.remote-app-log-dir</name>
              <value>/logs</value>
                </property>

  <!-- 配置nodemanager可用的资源内存 -->
  <property>
        <name>yarn.nodemanager.resource.memory-mb</name>
              <value>2048</value>
                </property>

  <!-- 配置nodemanager可用的资源CPU -->
  <property>
        <name>yarn.nodemanager.resource.cpu-vcores</name>
              <value>2</value>
                </property>

  <!-- //////////////以下为YARN HA的配置////////////// -->
  <!-- 开启YARN HA -->
  <property>
        <name>yarn.resourcemanager.ha.enabled</name>
              <value>true</value>
                </property>

  <!-- 启用自动故障转移 -->
  <property>
        <name>yarn.resourcemanager.ha.automatic-failover.enabled</name>
              <value>true</value>
                </property>

  <!-- 指定YARN HA的名称 -->
  <property>
        <name>yarn.resourcemanager.cluster-id</name>
              <value>yarncluster</value>
                </property>

  <!-- 指定两个resourcemanager的名称 -->
  <property>
        <name>yarn.resourcemanager.ha.rm-ids</name>
              <value>rm1,rm2</value>
                </property>

  <!-- 配置rm1，rm2的主机 -->
  <property>
        <name>yarn.resourcemanager.hostname.rm1</name>
              <value>node1</value>
                </property>

  <property>
        <name>yarn.resourcemanager.hostname.rm2</name>
              <value>node2</value>
                </property>

  <!-- 配置YARN的http端口 -->
  <property>
        <name>yarn.resourcemanager.webapp.address.rm1</name>
              <value>node1:8088</value>
                </property>

  <property>
        <name>yarn.resourcemanager.webapp.address.rm2</name>
              <value>node2:8088</value>
                </property>

  <!-- 配置zookeeper的地址 -->
  <property>
        <name>yarn.resourcemanager.zk-address</name>
              <value>node1:2181,node2:2181,node3:2181,node4:2181,node5:2181</value>
                </property>

  <!-- 配置zookeeper的存储位置 -->
  <property>
        <name>yarn.resourcemanager.zk-state-store.parent-path</name>
              <value>/rmstore</value>
                </property>

  <!-- 开启yarn resourcemanager restart -->
  <property>
        <name>yarn.resourcemanager.recovery.enabled</name>
              <value>true</value>
                </property>

  <!-- 配置resourcemanager的状态存储到zookeeper中 -->
  <property>
        <name>yarn.resourcemanager.store.class</name>
              <value>org.apache.hadoop.yarn.server.resourcemanager.recovery.ZKRMStateStore</value>
                </property>

  <!-- 开启yarn nodemanager restart -->
  <property>
        <name>yarn.nodemanager.recovery.enabled</name>
              <value>true</value>
                </property>

  <!-- 配置nodemanager IPC的通信端口 -->
  <property>
        <name>yarn.nodemanager.address</name>
              <value>0.0.0.0:45454</value>
                </property>

</configuration>


```

### masters

此文件作用为指定secondary datanode 所在节点
 
```sql

[root@node1 hadoop]# cat masters
node2

```
### slaves

此文件作用为指定datanode

```sql

[root@node1 hadoop]# cat slaves
node3
node4
node5


```



### 分发hadoop

```sql

[root@node1 hadoop]# scp -r /hadoop/ node2:/hadoop
[root@node1 hadoop]# scp -r /hadoop/ node3:/hadoop
[root@node1 hadoop]# scp -r /hadoop/ node4:/hadoop
[root@node1 hadoop]# scp -r /hadoop/ node5:/hadoop

```

# 配置分发/etc/profile

## 配置/etc/profile

```sql

[root@node1 hadoop]# cat /etc/profile
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
export PATH=$JAVA_HOME/bin:$PATH:$HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$JAVA_HOME/bin:$PATH:$HOME/sbin

export ZOOKEEPER_HOME=/zookeeper
export PATH=$PATH:$ZOOKEEPER_HOME/bin
unset i
unset -f pathmunge


```

## 分发/etc/profile 

```sql

[root@node1 hadoop]# scp -r /etc/profile node2:/etc/
[root@node1 hadoop]# scp -r /etc/profile node3:/etc/
[root@node1 hadoop]# scp -r /etc/profile node4:/etc/
[root@node1 hadoop]# scp -r /etc/profile node5:/etc/



```

## 使/etc/profile 生效

```sql

[root@node1 hadoop]# ssh node1 source /etc/profile
[root@node1 hadoop]# ssh node2 source /etc/profile
[root@node1 hadoop]# ssh node3 source /etc/profile
[root@node1 hadoop]# ssh node4 source /etc/profile
[root@node1 hadoop]# ssh node5 source /etc/profile


```


#  启动hadoop

## 启动zookeeper 

- node1 
 
```sql

[root@node1 hadoop]# zkServer.sh start
JMX enabled by default
Using config: /zookeeper/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED


```
- node2

```sql

[root@node2 hadoop]# zkServer.sh start
JMX enabled by default
Using config: /zookeeper/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED


```

- node3 

```sql

[root@node3 hadoop]# zkServer.sh start
JMX enabled by default
Using config: /zookeeper/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED


```

- node4

```sql

[root@node4 hadoop]# zkServer.sh start
JMX enabled by default
Using config: /zookeeper/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED


```

- node5 

```sql

[root@node5 hadoop]# zkServer.sh start
JMX enabled by default
Using config: /zookeeper/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED


```

## 格式化namenode

```sql


[root@node1 ~]# hadoop namenode -format
DEPRECATED: Use of this script to execute hdfs command is deprecated.
Instead use the hdfs command for it.

18/11/19 17:15:19 INFO namenode.NameNode: STARTUP_MSG: 
/************************************************************
STARTUP_MSG: Starting NameNode
STARTUP_MSG:   host = node1/192.168.2.61
STARTUP_MSG:   args = [-format]
STARTUP_MSG:   version = 2.7.3
STARTUP_MSG:   classpath = /hadoop/etc/hadoop:/hadoop/share/hadoop/common/lib/commons-beanutils-1.7.0.jar:/hadoop/share/hadoop/common/lib/jetty-6.1.26.jar:/hadoop/share/hadoop/common/lib/commons-lang-2.6.jar:/hadoop/share/hadoop/common/lib/jackson-mapper-asl-1.9.13.jar:/hadoop/share/hadoop/common/lib/jsch-0.1.42.jar:/hadoop/share/hadoop/common/lib/jackson-xc-1.9.13.jar:/hadoop/share/hadoop/common/lib/hamcrest-core-1.3.jar:/hadoop/share/hadoop/common/lib/jsp-api-2.1.jar:/hadoop/share/hadoop/common/lib/httpclient-4.2.5.jar:/hadoop/share/hadoop/common/lib/apacheds-kerberos-codec-2.0.0-M15.jar:/hadoop/share/hadoop/common/lib/commons-net-3.1.jar:/hadoop/share/hadoop/common/lib/commons-math3-3.1.1.jar:/hadoop/share/hadoop/common/lib/curator-framework-2.7.1.jar:/hadoop/share/hadoop/common/lib/jackson-core-asl-1.9.13.jar:/hadoop/share/hadoop/common/lib/snappy-java-1.0.4.1.jar:/hadoop/share/hadoop/common/lib/apacheds-i18n-2.0.0-M15.jar:/hadoop/share/hadoop/common/lib/xz-1.0.jar:/hadoop/share/hadoop/common/lib/commons-collections-3.2.2.jar:/hadoop/share/hadoop/common/lib/commons-digester-1.8.jar:/hadoop/share/hadoop/common/lib/commons-cli-1.2.jar:/hadoop/share/hadoop/common/lib/commons-beanutils-core-1.8.0.jar:/hadoop/share/hadoop/common/lib/jets3t-0.9.0.jar:/hadoop/share/hadoop/common/lib/commons-httpclient-3.1.jar:/hadoop/share/hadoop/common/lib/commons-logging-1.1.3.jar:/hadoop/share/hadoop/common/lib/stax-api-1.0-2.jar:/hadoop/share/hadoop/common/lib/slf4j-log4j12-1.7.10.jar:/hadoop/share/hadoop/common/lib/jackson-jaxrs-1.9.13.jar:/hadoop/share/hadoop/common/lib/java-xmlbuilder-0.4.jar:/hadoop/share/hadoop/common/lib/commons-io-2.4.jar:/hadoop/share/hadoop/common/lib/hadoop-auth-2.7.3.jar:/hadoop/share/hadoop/common/lib/httpcore-4.2.5.jar:/hadoop/share/hadoop/common/lib/htrace-core-3.1.0-incubating.jar:/hadoop/share/hadoop/common/lib/activation-1.1.jar:/hadoop/share/hadoop/common/lib/curator-client-2.7.1.jar:/hadoop/share/hadoop/common/lib/api-asn1-api-1.0.0-M20.jar:/hadoop/share/hadoop/common/lib/commons-configuration-1.6.jar:/hadoop/share/hadoop/common/lib/protobuf-java-2.5.0.jar:/hadoop/share/hadoop/common/lib/jsr305-3.0.0.jar:/hadoop/share/hadoop/common/lib/jaxb-impl-2.2.3-1.jar:/hadoop/share/hadoop/common/lib/jettison-1.1.jar:/hadoop/share/hadoop/common/lib/gson-2.2.4.jar:/hadoop/share/hadoop/common/lib/api-util-1.0.0-M20.jar:/hadoop/share/hadoop/common/lib/asm-3.2.jar:/hadoop/share/hadoop/common/lib/jersey-server-1.9.jar:/hadoop/share/hadoop/common/lib/guava-11.0.2.jar:/hadoop/share/hadoop/common/lib/avro-1.7.4.jar:/hadoop/share/hadoop/common/lib/commons-codec-1.4.jar:/hadoop/share/hadoop/common/lib/paranamer-2.3.jar:/hadoop/share/hadoop/common/lib/jersey-json-1.9.jar:/hadoop/share/hadoop/common/lib/slf4j-api-1.7.10.jar:/hadoop/share/hadoop/common/lib/hadoop-annotations-2.7.3.jar:/hadoop/share/hadoop/common/lib/netty-3.6.2.Final.jar:/hadoop/share/hadoop/common/lib/curator-recipes-2.7.1.jar:/hadoop/share/hadoop/common/lib/servlet-api-2.5.jar:/hadoop/share/hadoop/common/lib/commons-compress-1.4.1.jar:/hadoop/share/hadoop/common/lib/junit-4.11.jar:/hadoop/share/hadoop/common/lib/zookeeper-3.4.6.jar:/hadoop/share/hadoop/common/lib/xmlenc-0.52.jar:/hadoop/share/hadoop/common/lib/jetty-util-6.1.26.jar:/hadoop/share/hadoop/common/lib/log4j-1.2.17.jar:/hadoop/share/hadoop/common/lib/mockito-all-1.8.5.jar:/hadoop/share/hadoop/common/lib/jaxb-api-2.2.2.jar:/hadoop/share/hadoop/common/lib/jersey-core-1.9.jar:/hadoop/share/hadoop/common/hadoop-common-2.7.3.jar:/hadoop/share/hadoop/common/hadoop-nfs-2.7.3.jar:/hadoop/share/hadoop/common/hadoop-common-2.7.3-tests.jar:/hadoop/share/hadoop/hdfs:/hadoop/share/hadoop/hdfs/lib/jetty-6.1.26.jar:/hadoop/share/hadoop/hdfs/lib/commons-lang-2.6.jar:/hadoop/share/hadoop/hdfs/lib/jackson-mapper-asl-1.9.13.jar:/hadoop/share/hadoop/hdfs/lib/jackson-core-asl-1.9.13.jar:/hadoop/share/hadoop/hdfs/lib/commons-cli-1.2.jar:/hadoop/share/hadoop/hdfs/lib/commons-logging-1.1.3.jar:/hadoop/share/hadoop/hdfs/lib/netty-all-4.0.23.Final.jar:/hadoop/share/hadoop/hdfs/lib/commons-io-2.4.jar:/hadoop/share/hadoop/hdfs/lib/htrace-core-3.1.0-incubating.jar:/hadoop/share/hadoop/hdfs/lib/commons-daemon-1.0.13.jar:/hadoop/share/hadoop/hdfs/lib/protobuf-java-2.5.0.jar:/hadoop/share/hadoop/hdfs/lib/jsr305-3.0.0.jar:/hadoop/share/hadoop/hdfs/lib/asm-3.2.jar:/hadoop/share/hadoop/hdfs/lib/jersey-server-1.9.jar:/hadoop/share/hadoop/hdfs/lib/guava-11.0.2.jar:/hadoop/share/hadoop/hdfs/lib/commons-codec-1.4.jar:/hadoop/share/hadoop/hdfs/lib/netty-3.6.2.Final.jar:/hadoop/share/hadoop/hdfs/lib/servlet-api-2.5.jar:/hadoop/share/hadoop/hdfs/lib/xmlenc-0.52.jar:/hadoop/share/hadoop/hdfs/lib/jetty-util-6.1.26.jar:/hadoop/share/hadoop/hdfs/lib/log4j-1.2.17.jar:/hadoop/share/hadoop/hdfs/lib/leveldbjni-all-1.8.jar:/hadoop/share/hadoop/hdfs/lib/xml-apis-1.3.04.jar:/hadoop/share/hadoop/hdfs/lib/jersey-core-1.9.jar:/hadoop/share/hadoop/hdfs/lib/xercesImpl-2.9.1.jar:/hadoop/share/hadoop/hdfs/hadoop-hdfs-nfs-2.7.3.jar:/hadoop/share/hadoop/hdfs/hadoop-hdfs-2.7.3-tests.jar:/hadoop/share/hadoop/hdfs/hadoop-hdfs-2.7.3.jar:/hadoop/share/hadoop/yarn/lib/jetty-6.1.26.jar:/hadoop/share/hadoop/yarn/lib/commons-lang-2.6.jar:/hadoop/share/hadoop/yarn/lib/jackson-mapper-asl-1.9.13.jar:/hadoop/share/hadoop/yarn/lib/jackson-xc-1.9.13.jar:/hadoop/share/hadoop/yarn/lib/zookeeper-3.4.6-tests.jar:/hadoop/share/hadoop/yarn/lib/guice-3.0.jar:/hadoop/share/hadoop/yarn/lib/jackson-core-asl-1.9.13.jar:/hadoop/share/hadoop/yarn/lib/xz-1.0.jar:/hadoop/share/hadoop/yarn/lib/commons-collections-3.2.2.jar:/hadoop/share/hadoop/yarn/lib/commons-cli-1.2.jar:/hadoop/share/hadoop/yarn/lib/commons-logging-1.1.3.jar:/hadoop/share/hadoop/yarn/lib/jersey-guice-1.9.jar:/hadoop/share/hadoop/yarn/lib/javax.inject-1.jar:/hadoop/share/hadoop/yarn/lib/jersey-client-1.9.jar:/hadoop/share/hadoop/yarn/lib/stax-api-1.0-2.jar:/hadoop/share/hadoop/yarn/lib/jackson-jaxrs-1.9.13.jar:/hadoop/share/hadoop/yarn/lib/commons-io-2.4.jar:/hadoop/share/hadoop/yarn/lib/activation-1.1.jar:/hadoop/share/hadoop/yarn/lib/protobuf-java-2.5.0.jar:/hadoop/share/hadoop/yarn/lib/jsr305-3.0.0.jar:/hadoop/share/hadoop/yarn/lib/jaxb-impl-2.2.3-1.jar:/hadoop/share/hadoop/yarn/lib/jettison-1.1.jar:/hadoop/share/hadoop/yarn/lib/asm-3.2.jar:/hadoop/share/hadoop/yarn/lib/jersey-server-1.9.jar:/hadoop/share/hadoop/yarn/lib/guava-11.0.2.jar:/hadoop/share/hadoop/yarn/lib/commons-codec-1.4.jar:/hadoop/share/hadoop/yarn/lib/jersey-json-1.9.jar:/hadoop/share/hadoop/yarn/lib/aopalliance-1.0.jar:/hadoop/share/hadoop/yarn/lib/netty-3.6.2.Final.jar:/hadoop/share/hadoop/yarn/lib/servlet-api-2.5.jar:/hadoop/share/hadoop/yarn/lib/commons-compress-1.4.1.jar:/hadoop/share/hadoop/yarn/lib/zookeeper-3.4.6.jar:/hadoop/share/hadoop/yarn/lib/guice-servlet-3.0.jar:/hadoop/share/hadoop/yarn/lib/jetty-util-6.1.26.jar:/hadoop/share/hadoop/yarn/lib/log4j-1.2.17.jar:/hadoop/share/hadoop/yarn/lib/leveldbjni-all-1.8.jar:/hadoop/share/hadoop/yarn/lib/jaxb-api-2.2.2.jar:/hadoop/share/hadoop/yarn/lib/jersey-core-1.9.jar:/hadoop/share/hadoop/yarn/hadoop-yarn-api-2.7.3.jar:/hadoop/share/hadoop/yarn/hadoop-yarn-server-tests-2.7.3.jar:/hadoop/share/hadoop/yarn/hadoop-yarn-server-common-2.7.3.jar:/hadoop/share/hadoop/yarn/hadoop-yarn-applications-distributedshell-2.7.3.jar:/hadoop/share/hadoop/yarn/hadoop-yarn-server-sharedcachemanager-2.7.3.jar:/hadoop/share/hadoop/yarn/hadoop-yarn-server-applicationhistoryservice-2.7.3.jar:/hadoop/share/hadoop/yarn/hadoop-yarn-server-web-proxy-2.7.3.jar:/hadoop/share/hadoop/yarn/hadoop-yarn-common-2.7.3.jar:/hadoop/share/hadoop/yarn/hadoop-yarn-applications-unmanaged-am-launcher-2.7.3.jar:/hadoop/share/hadoop/yarn/hadoop-yarn-client-2.7.3.jar:/hadoop/share/hadoop/yarn/hadoop-yarn-server-nodemanager-2.7.3.jar:/hadoop/share/hadoop/yarn/hadoop-yarn-registry-2.7.3.jar:/hadoop/share/hadoop/yarn/hadoop-yarn-server-resourcemanager-2.7.3.jar:/hadoop/share/hadoop/mapreduce/lib/jackson-mapper-asl-1.9.13.jar:/hadoop/share/hadoop/mapreduce/lib/hamcrest-core-1.3.jar:/hadoop/share/hadoop/mapreduce/lib/guice-3.0.jar:/hadoop/share/hadoop/mapreduce/lib/jackson-core-asl-1.9.13.jar:/hadoop/share/hadoop/mapreduce/lib/snappy-java-1.0.4.1.jar:/hadoop/share/hadoop/mapreduce/lib/xz-1.0.jar:/hadoop/share/hadoop/mapreduce/lib/jersey-guice-1.9.jar:/hadoop/share/hadoop/mapreduce/lib/javax.inject-1.jar:/hadoop/share/hadoop/mapreduce/lib/commons-io-2.4.jar:/hadoop/share/hadoop/mapreduce/lib/protobuf-java-2.5.0.jar:/hadoop/share/hadoop/mapreduce/lib/asm-3.2.jar:/hadoop/share/hadoop/mapreduce/lib/jersey-server-1.9.jar:/hadoop/share/hadoop/mapreduce/lib/avro-1.7.4.jar:/hadoop/share/hadoop/mapreduce/lib/paranamer-2.3.jar:/hadoop/share/hadoop/mapreduce/lib/aopalliance-1.0.jar:/hadoop/share/hadoop/mapreduce/lib/hadoop-annotations-2.7.3.jar:/hadoop/share/hadoop/mapreduce/lib/netty-3.6.2.Final.jar:/hadoop/share/hadoop/mapreduce/lib/commons-compress-1.4.1.jar:/hadoop/share/hadoop/mapreduce/lib/junit-4.11.jar:/hadoop/share/hadoop/mapreduce/lib/guice-servlet-3.0.jar:/hadoop/share/hadoop/mapreduce/lib/log4j-1.2.17.jar:/hadoop/share/hadoop/mapreduce/lib/leveldbjni-all-1.8.jar:/hadoop/share/hadoop/mapreduce/lib/jersey-core-1.9.jar:/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-client-hs-2.7.3.jar:/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-client-hs-plugins-2.7.3.jar:/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-client-shuffle-2.7.3.jar:/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-client-app-2.7.3.jar:/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-2.7.3.jar:/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-2.7.3-tests.jar:/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-client-common-2.7.3.jar:/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-client-core-2.7.3.jar:/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.3.jar:/hadoop/contrib/capacity-scheduler/*.jar:/hadoop/contrib/capacity-scheduler/*.jar
STARTUP_MSG:   build = https://git-wip-us.apache.org/repos/asf/hadoop.git -r baa91f7c6bc9cb92be5982de4719c1c8af91ccff; compiled by 'root' on 2016-08-18T01:41Z
STARTUP_MSG:   java = 1.8.0_65
************************************************************/
18/11/19 17:15:19 INFO namenode.NameNode: registered UNIX signal handlers for [TERM, HUP, INT]
18/11/19 17:15:19 INFO namenode.NameNode: createNameNode [-format]
18/11/19 17:15:20 WARN common.Util: Path /hadoop/tmp/dfs/namenode should be specified as a URI in configuration files. Please update hdfs configuration.
18/11/19 17:15:20 WARN common.Util: Path /hadoop/tmp/dfs/namenode should be specified as a URI in configuration files. Please update hdfs configuration.
Formatting using clusterid: CID-0fb6ec0a-3ac4-44dd-9c0e-437fc2b49cf9
18/11/19 17:15:20 INFO namenode.FSNamesystem: No KeyProvider found.
18/11/19 17:15:20 INFO namenode.FSNamesystem: fsLock is fair:true
18/11/19 17:15:20 INFO blockmanagement.DatanodeManager: dfs.block.invalidate.limit=1000
18/11/19 17:15:20 INFO blockmanagement.DatanodeManager: dfs.namenode.datanode.registration.ip-hostname-check=true
18/11/19 17:15:20 INFO blockmanagement.BlockManager: dfs.namenode.startup.delay.block.deletion.sec is set to 000:00:00:00.000
18/11/19 17:15:20 INFO blockmanagement.BlockManager: The block deletion will start around 2018 Nov 19 17:15:20
18/11/19 17:15:20 INFO util.GSet: Computing capacity for map BlocksMap
18/11/19 17:15:20 INFO util.GSet: VM type       = 64-bit
18/11/19 17:15:20 INFO util.GSet: 2.0% max memory 966.7 MB = 19.3 MB
18/11/19 17:15:20 INFO util.GSet: capacity      = 2^21 = 2097152 entries
18/11/19 17:15:20 INFO blockmanagement.BlockManager: dfs.block.access.token.enable=false
18/11/19 17:15:20 INFO blockmanagement.BlockManager: defaultReplication         = 3
18/11/19 17:15:20 INFO blockmanagement.BlockManager: maxReplication             = 512
18/11/19 17:15:20 INFO blockmanagement.BlockManager: minReplication             = 1
18/11/19 17:15:20 INFO blockmanagement.BlockManager: maxReplicationStreams      = 2
18/11/19 17:15:20 INFO blockmanagement.BlockManager: replicationRecheckInterval = 3000
18/11/19 17:15:20 INFO blockmanagement.BlockManager: encryptDataTransfer        = false
18/11/19 17:15:20 INFO blockmanagement.BlockManager: maxNumBlocksToLog          = 1000
18/11/19 17:15:20 INFO namenode.FSNamesystem: fsOwner             = root (auth:SIMPLE)
18/11/19 17:15:20 INFO namenode.FSNamesystem: supergroup          = supergroup
18/11/19 17:15:20 INFO namenode.FSNamesystem: isPermissionEnabled = false
18/11/19 17:15:20 INFO namenode.FSNamesystem: Determined nameservice ID: mycluster
18/11/19 17:15:20 INFO namenode.FSNamesystem: HA Enabled: true
18/11/19 17:15:20 INFO namenode.FSNamesystem: Append Enabled: true
18/11/19 17:15:20 INFO util.GSet: Computing capacity for map INodeMap
18/11/19 17:15:20 INFO util.GSet: VM type       = 64-bit
18/11/19 17:15:20 INFO util.GSet: 1.0% max memory 966.7 MB = 9.7 MB
18/11/19 17:15:20 INFO util.GSet: capacity      = 2^20 = 1048576 entries
18/11/19 17:15:20 INFO namenode.FSDirectory: ACLs enabled? false
18/11/19 17:15:20 INFO namenode.FSDirectory: XAttrs enabled? true
18/11/19 17:15:20 INFO namenode.FSDirectory: Maximum size of an xattr: 16384
18/11/19 17:15:20 INFO namenode.NameNode: Caching file names occuring more than 10 times
18/11/19 17:15:20 INFO util.GSet: Computing capacity for map cachedBlocks
18/11/19 17:15:20 INFO util.GSet: VM type       = 64-bit
18/11/19 17:15:20 INFO util.GSet: 0.25% max memory 966.7 MB = 2.4 MB
18/11/19 17:15:20 INFO util.GSet: capacity      = 2^18 = 262144 entries
18/11/19 17:15:20 INFO namenode.FSNamesystem: dfs.namenode.safemode.threshold-pct = 0.9990000128746033
18/11/19 17:15:20 INFO namenode.FSNamesystem: dfs.namenode.safemode.min.datanodes = 0
18/11/19 17:15:20 INFO namenode.FSNamesystem: dfs.namenode.safemode.extension     = 30000
18/11/19 17:15:20 INFO metrics.TopMetrics: NNTop conf: dfs.namenode.top.window.num.buckets = 10
18/11/19 17:15:20 INFO metrics.TopMetrics: NNTop conf: dfs.namenode.top.num.users = 10
18/11/19 17:15:20 INFO metrics.TopMetrics: NNTop conf: dfs.namenode.top.windows.minutes = 1,5,25
18/11/19 17:15:20 INFO namenode.FSNamesystem: Retry cache on namenode is enabled
18/11/19 17:15:20 INFO namenode.FSNamesystem: Retry cache will use 0.03 of total heap and retry cache entry expiry time is 600000 millis
18/11/19 17:15:20 INFO util.GSet: Computing capacity for map NameNodeRetryCache
18/11/19 17:15:20 INFO util.GSet: VM type       = 64-bit
18/11/19 17:15:20 INFO util.GSet: 0.029999999329447746% max memory 966.7 MB = 297.0 KB
18/11/19 17:15:20 INFO util.GSet: capacity      = 2^15 = 32768 entries
Re-format filesystem in Storage Directory /hadoop/tmp/dfs/namenode ? (Y or N) y
18/11/19 17:15:26 INFO ipc.Client: Retrying connect to server: node4/192.168.2.64:8485. Already tried 0 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:26 INFO ipc.Client: Retrying connect to server: node5/192.168.2.65:8485. Already tried 0 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:26 INFO ipc.Client: Retrying connect to server: node1/192.168.2.61:8485. Already tried 0 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:26 INFO ipc.Client: Retrying connect to server: node2/192.168.2.62:8485. Already tried 0 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:26 INFO ipc.Client: Retrying connect to server: node3/192.168.2.63:8485. Already tried 0 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:27 INFO ipc.Client: Retrying connect to server: node1/192.168.2.61:8485. Already tried 1 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:27 INFO ipc.Client: Retrying connect to server: node3/192.168.2.63:8485. Already tried 1 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:27 INFO ipc.Client: Retrying connect to server: node2/192.168.2.62:8485. Already tried 1 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:27 INFO ipc.Client: Retrying connect to server: node5/192.168.2.65:8485. Already tried 1 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:27 INFO ipc.Client: Retrying connect to server: node4/192.168.2.64:8485. Already tried 1 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:28 INFO ipc.Client: Retrying connect to server: node3/192.168.2.63:8485. Already tried 2 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:28 INFO ipc.Client: Retrying connect to server: node2/192.168.2.62:8485. Already tried 2 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:28 INFO ipc.Client: Retrying connect to server: node5/192.168.2.65:8485. Already tried 2 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:28 INFO ipc.Client: Retrying connect to server: node1/192.168.2.61:8485. Already tried 2 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:28 INFO ipc.Client: Retrying connect to server: node4/192.168.2.64:8485. Already tried 2 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:29 INFO ipc.Client: Retrying connect to server: node1/192.168.2.61:8485. Already tried 3 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:29 INFO ipc.Client: Retrying connect to server: node5/192.168.2.65:8485. Already tried 3 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:29 INFO ipc.Client: Retrying connect to server: node2/192.168.2.62:8485. Already tried 3 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:29 INFO ipc.Client: Retrying connect to server: node3/192.168.2.63:8485. Already tried 3 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:29 INFO ipc.Client: Retrying connect to server: node4/192.168.2.64:8485. Already tried 3 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:30 INFO ipc.Client: Retrying connect to server: node5/192.168.2.65:8485. Already tried 4 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:30 INFO ipc.Client: Retrying connect to server: node4/192.168.2.64:8485. Already tried 4 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:30 INFO ipc.Client: Retrying connect to server: node3/192.168.2.63:8485. Already tried 4 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:30 INFO ipc.Client: Retrying connect to server: node2/192.168.2.62:8485. Already tried 4 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:30 INFO ipc.Client: Retrying connect to server: node1/192.168.2.61:8485. Already tried 4 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:31 INFO ipc.Client: Retrying connect to server: node2/192.168.2.62:8485. Already tried 5 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:31 INFO ipc.Client: Retrying connect to server: node1/192.168.2.61:8485. Already tried 5 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:31 INFO ipc.Client: Retrying connect to server: node3/192.168.2.63:8485. Already tried 5 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:31 INFO ipc.Client: Retrying connect to server: node4/192.168.2.64:8485. Already tried 5 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:31 INFO ipc.Client: Retrying connect to server: node5/192.168.2.65:8485. Already tried 5 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:32 INFO ipc.Client: Retrying connect to server: node1/192.168.2.61:8485. Already tried 6 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:32 INFO ipc.Client: Retrying connect to server: node4/192.168.2.64:8485. Already tried 6 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:32 INFO ipc.Client: Retrying connect to server: node3/192.168.2.63:8485. Already tried 6 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:32 INFO ipc.Client: Retrying connect to server: node2/192.168.2.62:8485. Already tried 6 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:32 INFO ipc.Client: Retrying connect to server: node5/192.168.2.65:8485. Already tried 6 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:33 INFO ipc.Client: Retrying connect to server: node1/192.168.2.61:8485. Already tried 7 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:33 INFO ipc.Client: Retrying connect to server: node3/192.168.2.63:8485. Already tried 7 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:33 INFO ipc.Client: Retrying connect to server: node2/192.168.2.62:8485. Already tried 7 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:33 INFO ipc.Client: Retrying connect to server: node4/192.168.2.64:8485. Already tried 7 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:33 INFO ipc.Client: Retrying connect to server: node5/192.168.2.65:8485. Already tried 7 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:34 INFO ipc.Client: Retrying connect to server: node1/192.168.2.61:8485. Already tried 8 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:34 INFO ipc.Client: Retrying connect to server: node4/192.168.2.64:8485. Already tried 8 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:34 INFO ipc.Client: Retrying connect to server: node2/192.168.2.62:8485. Already tried 8 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:34 INFO ipc.Client: Retrying connect to server: node5/192.168.2.65:8485. Already tried 8 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:34 INFO ipc.Client: Retrying connect to server: node3/192.168.2.63:8485. Already tried 8 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:35 INFO ipc.Client: Retrying connect to server: node1/192.168.2.61:8485. Already tried 9 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:35 WARN namenode.NameNode: Encountered exception during format: 
org.apache.hadoop.hdfs.qjournal.client.QuorumException: Unable to check if JNs are ready for formatting. 1 exceptions thrown:
192.168.2.61:8485: Call From node1/192.168.2.61 to node1:8485 failed on connection exception: java.net.ConnectException: Connection refused; For more details see:  http://wiki.apache.org/hadoop/ConnectionRefused
at org.apache.hadoop.hdfs.qjournal.client.QuorumException.create(QuorumException.java:81)
at org.apache.hadoop.hdfs.qjournal.client.QuorumCall.rethrowException(QuorumCall.java:223)
at org.apache.hadoop.hdfs.qjournal.client.QuorumJournalManager.hasSomeData(QuorumJournalManager.java:232)
at org.apache.hadoop.hdfs.server.common.Storage.confirmFormat(Storage.java:901)
at org.apache.hadoop.hdfs.server.namenode.FSImage.confirmFormat(FSImage.java:184)
at org.apache.hadoop.hdfs.server.namenode.NameNode.format(NameNode.java:988)
at org.apache.hadoop.hdfs.server.namenode.NameNode.createNameNode(NameNode.java:1434)
at org.apache.hadoop.hdfs.server.namenode.NameNode.main(NameNode.java:1559)
18/11/19 17:15:35 INFO ipc.Client: Retrying connect to server: node3/192.168.2.63:8485. Already tried 9 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:35 INFO ipc.Client: Retrying connect to server: node2/192.168.2.62:8485. Already tried 9 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:35 INFO ipc.Client: Retrying connect to server: node5/192.168.2.65:8485. Already tried 9 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:35 INFO ipc.Client: Retrying connect to server: node4/192.168.2.64:8485. Already tried 9 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/11/19 17:15:35 ERROR namenode.NameNode: Failed to start namenode.
org.apache.hadoop.hdfs.qjournal.client.QuorumException: Unable to check if JNs are ready for formatting. 1 exceptions thrown:
192.168.2.61:8485: Call From node1/192.168.2.61 to node1:8485 failed on connection exception: java.net.ConnectException: Connection refused; For more details see:  http://wiki.apache.org/hadoop/ConnectionRefused
at org.apache.hadoop.hdfs.qjournal.client.QuorumException.create(QuorumException.java:81)
at org.apache.hadoop.hdfs.qjournal.client.QuorumCall.rethrowException(QuorumCall.java:223)
at org.apache.hadoop.hdfs.qjournal.client.QuorumJournalManager.hasSomeData(QuorumJournalManager.java:232)
at org.apache.hadoop.hdfs.server.common.Storage.confirmFormat(Storage.java:901)
at org.apache.hadoop.hdfs.server.namenode.FSImage.confirmFormat(FSImage.java:184)
at org.apache.hadoop.hdfs.server.namenode.NameNode.format(NameNode.java:988)
at org.apache.hadoop.hdfs.server.namenode.NameNode.createNameNode(NameNode.java:1434)
at org.apache.hadoop.hdfs.server.namenode.NameNode.main(NameNode.java:1559)
18/11/19 17:15:35 INFO util.ExitUtil: Exiting with status 1
18/11/19 17:15:35 INFO namenode.NameNode: SHUTDOWN_MSG: 
/************************************************************
SHUTDOWN_MSG: Shutting down NameNode at node1/192.168.2.61
************************************************************/

```

## 启动hadoop

```sql


[root@node1 ~]# start-all.sh
This script is Deprecated. Instead use start-dfs.sh and start-yarn.sh
Starting namenodes on [node1 node2]
node1: starting namenode, logging to /hadoop/logs/hadoop-root-namenode-node1.out
node2: starting namenode, logging to /hadoop/logs/hadoop-root-namenode-node2.out
node5: starting datanode, logging to /hadoop/logs/hadoop-root-datanode-node5.out
node4: starting datanode, logging to /hadoop/logs/hadoop-root-datanode-node4.out
node3: starting datanode, logging to /hadoop/logs/hadoop-root-datanode-node3.out
Starting journal nodes [node1 node2 node3 node4 node5]
node1: starting journalnode, logging to /hadoop/logs/hadoop-root-journalnode-node1.out
node2: starting journalnode, logging to /hadoop/logs/hadoop-root-journalnode-node2.out
node4: starting journalnode, logging to /hadoop/logs/hadoop-root-journalnode-node4.out
node3: starting journalnode, logging to /hadoop/logs/hadoop-root-journalnode-node3.out
node5: starting journalnode, logging to /hadoop/logs/hadoop-root-journalnode-node5.out
Starting ZK Failover Controllers on NN hosts [node1 node2]
node1: starting zkfc, logging to /hadoop/logs/hadoop-root-zkfc-node1.out
node2: starting zkfc, logging to /hadoop/logs/hadoop-root-zkfc-node2.out
starting yarn daemons
starting resourcemanager, logging to /hadoop/logs/yarn-root-resourcemanager-node1.out
node4: starting nodemanager, logging to /hadoop/logs/yarn-root-nodemanager-node4.out
node5: starting nodemanager, logging to /hadoop/logs/yarn-root-nodemanager-node5.out
node3: starting nodemanager, logging to /hadoop/logs/yarn-root-nodemanager-node3.out

```

##  查看进程状态

- node1 

```sql

[root@node1 hadoop]# jps
6341 JournalNode
6519 DFSZKFailoverController
4489 QuorumPeerMain
6636 ResourceManager
21181 Jps
6126 NameNode


```


- node2 

```sql

[root@node2 ~]# jps
2451 QuorumPeerMain
11123 Jps
3220 DFSZKFailoverController
3037 NameNode
3133 JournalNode

```

- node3 

```sql


[root@node3 ~]# jps
2808 QuorumPeerMain
3544 NodeManager
3449 JournalNode
7081 Jps
3354 DataNode


```

- node4

``` sql

[root@node4 ~]# jps
6019 NodeManager
5828 DataNode
5925 JournalNode
7174 Jps
5273 QuorumPeerMain


```

- node5 

```sql

[root@node5 ~]# jps
3600 NodeManager
3507 JournalNode
3411 DataNode
7079 Jps
2863 QuorumPeerMain


```