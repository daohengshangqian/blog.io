---
layout: post
title: hive安装
date: 2020-08-20
categories: blog
tags: [大数据]
description: hive安装
---


## 下载并上传hive至服务器

```
[root@node1 ~]# mkdir /hive
[root@node1 ~]# mv apache-hive-2.1.0-bin.tar.gz /hive/
[root@node1 ~]# cd /hive
[root@node1 hive]# tar -zxf apache-hive-2.1.0-bin.tar.gz
[root@node1 hive]# ll
total 146100
drwxr-xr-x 9 root root      4096 Dec 11 05:22 apache-hive-2.1.0-bin


```

# 解压

```sql

[root@node1 ~]# cd /hive
[root@node1 hive]# tar -zxf apache-hive-2.1.0-bin.tar.gz
[root@node1 hive]# ll
total 146100
drwxr-xr-x 9 root root      4096 Dec 11 05:22 apache-hive-2.1.0-bin
-rw-r--r-- 1 root root 149599799 Dec 11 05:18 apache-hive-2.1.0-bin.tar.gz

[root@node1 apache-hive-2.1.0-bin]# ll
total 132
drwxr-xr-x 3 root root  4096 Dec 11 05:22 bin
drwxr-xr-x 2 root root  4096 Dec 11 05:22 conf
drwxr-xr-x 4 root root  4096 Dec 11 05:22 examples
drwxr-xr-x 7 root root  4096 Dec 11 05:22 hcatalog
drwxr-xr-x 2 root root  4096 Dec 11 05:22 jdbc
drwxr-xr-x 4 root root 12288 Dec 11 05:22 lib
-rw-r--r-- 1 root root 29003 Jun  3  2016 LICENSE
-rw-r--r-- 1 root root   513 Jun  3  2016 NOTICE
-rw-r--r-- 1 root root  4122 Jun  3  2016 README.txt
-rw-r--r-- 1 root root 50294 Jun 17  2016 RELEASE_NOTES.txt
drwxr-xr-x 4 root root  4096 Dec 11 05:22 scripts
[root@node1 apache-hive-2.1.0-bin]# pwd
/hive/apache-hive-2.1.0-bin


```

## 修改环境变量

```sql

[root@node1 apache-hive-2.1.0-bin]# vi /etc/profile
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

export HIVE_HOME=/hive/apache-hive-2.1.0-bin
export PATH=$PATH:$HIVE_HOME/bin
unset i
unset -f pathmunge
~
~


```

## 启动mysql,并创建用户
```sql

[mysql@node1 mysql1]$ /usr/local/mysql/bin/mysqld --defaults-file=/mysql1/mysql1.cnf &
[1] 445

mysql> create user hiveuser@'%' identified by 'hiveuser';
Query OK, 0 rows affected (0.01 sec)

mysql> grant  all on *.* to   hiveuser@'%'  ;
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges ;
Query OK, 0 rows affected (0.01 sec)

mysql> create database hive ;
Query OK, 1 row affected (0.00 sec)


```

## 将mysql配置为hive 元数据库

### 复制mysql驱动值hive lib库
```sql

[root@node1 ~]# pwd
/root

[root@node1 ~]# ll mysql-connector-java-5.1.17.jar
-rw-r--r-- 1 root root 787920 Dec 11 05:39 mysql-connector-java-5.1.17.jar

[root@node1 ~]# mv mysql-connector-java-5.1.17.jar  /hive/apache-hive-2.1.0-bin/lib/


```

### 修改配置文件

```sql

[root@node1 ~]# cd /hive/apache-hive-2.1.0-bin/conf/
[root@node1 conf]# ll
total 260
-rw-r--r-- 1 root root   1596 Jun  3  2016 beeline-log4j2.properties.template
-rw-r--r-- 1 root root 225729 Jun 17  2016 hive-default.xml.template
-rw-r--r-- 1 root root   2378 Jun  3  2016 hive-env.sh.template
-rw-r--r-- 1 root root   2299 Jun  3  2016 hive-exec-log4j2.properties.template
-rw-r--r-- 1 root root   2950 Jun  3  2016 hive-log4j2.properties.template
-rw-r--r-- 1 root root   2049 Jun 10  2016 ivysettings.xml
-rw-r--r-- 1 root root   2768 Jun  3  2016 llap-cli-log4j2.properties.template
-rw-r--r-- 1 root root   4241 Jun  3  2016 llap-daemon-log4j2.properties.template
-rw-r--r-- 1 root root   2662 Jun  9  2016 parquet-logging.properties
[root@node1 conf]#
[root@node1 conf]# cp hive-default.xml.template  hive-site.xml
[root@node1 conf]# mkdir /hive/workspace
[root@node1 conf]# mkdir /hive/downloadspace
[root@node1 conf]# mkdir /hive/logspace
[root@node1 conf]# mkdir /hive/querylog


```

# 修改以下内容

```sql

<property>
    <name>javax.jdo.option.ConnectionUserName</name>
    <value>hiveuser</value>
    <description>Username to use against metastore database</description>
</property>

<property>
    <name>javax.jdo.option.ConnectionPassword</name>
    <value>hiveuser</value>
    <description>password to use against metastore database</description>
</property>

 <property>
    <name>javax.jdo.option.ConnectionDriverName</name>
    <value>com.mysql.jdbc.Driver</value>
    <description>Driver class name for a JDBC metastore</description>
  </property>
  
  <property>
    <name>javax.jdo.option.ConnectionURL</name>
    <value>jdbc:mysql://192.168.2.61:3306/hive</value>
    <description>
      JDBC connect string for a JDBC metastore.
      To use SSL to encrypt/authenticate the connection, provide database-specific SSL flag in the connection URL.
      For example, jdbc:postgresql://myhost/db?ssl=true for postgres database.
    </description>
  </property>

 <property>
    <name>hive.exec.local.scratchdir</name>
    <value>/hive/workspace</value>
    <description>Local scratch space for Hive jobs</description>
  </property>

  <property>
    <name>hive.downloaded.resources.dir</name>
    <value>/hive/downloadspace</value>
    <description>Temporary local directory for added resources in the remote file system.</description>
  </property>  
  <property>
    <name>hive.querylog.location</name>
    <value>/hive/querylog</value>
    <description>Location of Hive run time structured log file</description>
  </property>
 <property>
    <name>hive.server2.logging.operation.log.location</name>
    <value>/hive/logspace</value>
    <description>Top level directory where operation logs are stored if logging functionality is enabled</description>
 </property>
<property>
    <name>hive.server2.enable.doAs</name>
    <value>false</value>
    <description>
      Setting this property to true will have HiveServer2 execute
      Hive operations as the user making the calls to it.
    </description>
 </property>
  
  
```

初始化hive的元数据



```sql



[root@node1 conf]# /hive/apache-hive-2.1.0-bin/bin/schematool -dbType mysql -initSchema

SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/hive/apache-hive-2.1.0-bin/lib/log4j-slf4j-impl-2.4.1.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/hadoop/share/hadoop/common/lib/slf4j-log4j12-1.7.10.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]
Metastore connection URL:        jdbc:mysql://192.168.2.61:3306/hive
Metastore Connection Driver :    com.mysql.jdbc.Driver
Metastore connection User:       hiveuser
Starting metastore schema initialization to 2.1.0
Initialization script hive-schema-2.1.0.mysql.sql
Initialization script completed
schemaTool completed

```

## 查看数据库

```sql

mysql> show databases ;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| hive               |
| mysql              |
| performance_schema |
| sys                |
| test               |
+--------------------+
6 rows in set (0.01 sec)

mysql> use hive ;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables ;
+---------------------------+
| Tables_in_hive            |
+---------------------------+
| AUX_TABLE                 |
| BUCKETING_COLS            |
| CDS                       |
| COLUMNS_V2                |
| COMPACTION_QUEUE          |
| COMPLETED_COMPACTIONS     |
| COMPLETED_TXN_COMPONENTS  |
| DATABASE_PARAMS           |
| DBS                       |
| DB_PRIVS                  |
| DELEGATION_TOKENS         |
| FUNCS                     |
| FUNC_RU                   |
| GLOBAL_PRIVS              |
| HIVE_LOCKS                |
| IDXS                      |
| INDEX_PARAMS              |
| KEY_CONSTRAINTS           |
| MASTER_KEYS               |
| NEXT_COMPACTION_QUEUE_ID  |
| NEXT_LOCK_ID              |
| NEXT_TXN_ID               |
| NOTIFICATION_LOG          |
| NOTIFICATION_SEQUENCE     |
| NUCLEUS_TABLES            |
| PARTITIONS                |
| PARTITION_EVENTS          |
| PARTITION_KEYS            |
| PARTITION_KEY_VALS        |
| PARTITION_PARAMS          |
| PART_COL_PRIVS            |
| PART_COL_STATS            |
| PART_PRIVS                |
| ROLES                     |
| ROLE_MAP                  |
| SDS                       |
| SD_PARAMS                 |
| SEQUENCE_TABLE            |
| SERDES                    |
| SERDE_PARAMS              |
| SKEWED_COL_NAMES          |
| SKEWED_COL_VALUE_LOC_MAP  |
| SKEWED_STRING_LIST        |
| SKEWED_STRING_LIST_VALUES |
| SKEWED_VALUES             |
| SORT_COLS                 |
| TABLE_PARAMS              |
| TAB_COL_STATS             |
| TBLS                      |
| TBL_COL_PRIVS             |
| TBL_PRIVS                 |
| TXNS                      |
| TXN_COMPONENTS            |
| TYPES                     |
| TYPE_FIELDS               |
| VERSION                   |
| WRITE_SET                 |
+---------------------------+
57 rows in set (0.00 sec)


```


## 进入hive的命令行

```sql


[root@node1 conf]# hive

SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/hive/apache-hive-2.1.0-bin/lib/log4j-slf4j-impl-2.4.1.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/hadoop/share/hadoop/common/lib/slf4j-log4j12-1.7.10.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]

Logging initialized using configuration in jar:file:/hive/apache-hive-2.1.0-bin/lib/hive-common-2.1.0.jar!/hive-log4j2.properties Async: true
Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
hive>


```