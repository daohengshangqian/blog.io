---
layout: post
title: MySQL PXC搭建
date: 2018-09-04
categories: blog
tags: [MySQL]
description: MySQL PXC搭建
---

## 系统准备

### 安装EPEL
```sql


[root@node1 ~]# yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-6.noarch.rpm
Loaded plugins: fastestmirror, refresh-packagekit, security
Repository 'server' is missing name in configuration, using id
Loading mirror speeds from cached hostfile
Setting up Install Process

epel-release-latest-6.noarch.rpm                                                                                                     |  14 kB     00:00     
Examining /var/tmp/yum-root-XhhRvZ/epel-release-latest-6.noarch.rpm: epel-release-6-8.noarch
Marking /var/tmp/yum-root-XhhRvZ/epel-release-latest-6.noarch.rpm to be installed
Resolving Dependencies
--> Running transaction check
---> Package epel-release.noarch 0:6-8 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

============================================================================================================================================================
 Package                             Arch                          Version                       Repository                                            Size
============================================================================================================================================================
Installing:
 epel-release                        noarch                        6-8                           /epel-release-latest-6.noarch                         22 k

Transaction Summary
============================================================================================================================================================
Install       1 Package(s)

Total size: 22 k
Installed size: 22 k
Is this ok [y/N]: y
Downloading Packages:
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction


  Installing : epel-release-6-8.noarch [############################################################################################################  ] 1/1
  Installing : epel-release-6-8.noarch                                                                                                                  1/1 

  Verifying  : epel-release-6-8.noarch                                                                                                                  1/1 

Installed:
  epel-release.noarch 0:6-8                                                                                                                                 

```

### 安装socat 

```sql

[root@node1 ~]# yum install socat
Loaded plugins: fastestmirror, refresh-packagekit, security
Repository 'server' is missing name in configuration, using id
Loading mirror speeds from cached hostfile

epel/metalink                                                                                                                        | 8.5 kB     00:00     
 * epel: mirrors.aliyun.com


epel/primary                                                    99% [==================================================== ] 806 kB/s | 3.2 MB     00:00 ETA 

epel/primary                                                                                                                         | 3.2 MB     00:02     

                                                                     ] 232/12521


epel: [###################################################################################################################################### ] 12520/12521
epel                                                                                                                                            12521/12521
Setting up Install Process
Resolving Dependencies
--> Running transaction check
---> Package socat.x86_64 0:1.7.2.3-1.el6 will be installed
--> Processing Dependency: libreadline.so.5()(64bit) for package: socat-1.7.2.3-1.el6.x86_64
--> Running transaction check
---> Package compat-readline5.x86_64 0:5.2-17.1.el6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

============================================================================================================================================================
 Package                                    Arch                             Version                                 Repository                        Size
============================================================================================================================================================
Installing:
 socat                                      x86_64                           1.7.2.3-1.el6                           epel                             246 k
Installing for dependencies:
 compat-readline5                           x86_64                           5.2-17.1.el6                            server                           130 k

Transaction Summary
============================================================================================================================================================
Install       2 Package(s)

Total download size: 375 k
Installed size: 1.2 M
Is this ok [y/N]: y
Downloading Packages:

(2/2): socat-1.7.2.3-1.el6.x86_64.rpm                                                                                                | 246 kB     00:00     
------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                       1.5 MB/s | 375 kB     00:00     
warning: rpmts_HdrFromFdno: Header V3 RSA/SHA256 Signature, key ID 0608b895: NOKEY
Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6
Importing GPG key 0x0608B895:
 Userid : EPEL (6) <epel@fedoraproject.org>
 Package: epel-release-6-8.noarch (@/epel-release-latest-6.noarch)
 From   : /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6
Is this ok [y/N]: y
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction

 
  Installing : compat-readline5-5.2-17.1.el6.x86_64 [################################################################################################ ] 1/2
  Installing : compat-readline5-5.2-17.1.el6.x86_64                                                                                                     1/2 


  Installing : socat-1.7.2.3-1.el6.x86_64 [#########################################################################################################  ] 2/2
  Installing : socat-1.7.2.3-1.el6.x86_64                                                                                                               2/2 

  Verifying  : socat-1.7.2.3-1.el6.x86_64                                                                                                               1/2 

  Verifying  : compat-readline5-5.2-17.1.el6.x86_64                                                                                                     2/2 

Installed:
  socat.x86_64 0:1.7.2.3-1.el6                                                                                                                              

Dependency Installed:
  compat-readline5.x86_64 0:5.2-17.1.el6                                                                                                                    

Complete!

```

### 安装PXC套件 

```sql

yum install Percona-XtraDB-Cluster-*.rpm
Loaded plugins: fastestmirror, refresh-packagekit, security
Repository 'server' is missing name in configuration, using id
Loading mirror speeds from cached hostfile
 * epel: mirrors.yun-idc.com
Setting up Install Process
Examining Percona-XtraDB-Cluster-57-5.7.22-29.26.1.el6.x86_64.rpm: Percona-XtraDB-Cluster-57-5.7.22-29.26.1.el6.x86_64
Marking Percona-XtraDB-Cluster-57-5.7.22-29.26.1.el6.x86_64.rpm to be installed
Examining Percona-XtraDB-Cluster-57-debuginfo-5.7.22-29.26.1.el6.x86_64.rpm: Percona-XtraDB-Cluster-57-debuginfo-5.7.22-29.26.1.el6.x86_64
Marking Percona-XtraDB-Cluster-57-debuginfo-5.7.22-29.26.1.el6.x86_64.rpm to be installed
Examining Percona-XtraDB-Cluster-client-57-5.7.22-29.26.1.el6.x86_64.rpm: Percona-XtraDB-Cluster-client-57-5.7.22-29.26.1.el6.x86_64
Percona-XtraDB-Cluster-client-57-5.7.22-29.26.1.el6.x86_64.rpm: does not update installed package.
Examining Percona-XtraDB-Cluster-devel-57-5.7.22-29.26.1.el6.x86_64.rpm: Percona-XtraDB-Cluster-devel-57-5.7.22-29.26.1.el6.x86_64
Percona-XtraDB-Cluster-devel-57-5.7.22-29.26.1.el6.x86_64.rpm: does not update installed package.
Examining Percona-XtraDB-Cluster-full-57-5.7.22-29.26.1.el6.x86_64.rpm: Percona-XtraDB-Cluster-full-57-5.7.22-29.26.1.el6.x86_64
Marking Percona-XtraDB-Cluster-full-57-5.7.22-29.26.1.el6.x86_64.rpm to be installed
Examining Percona-XtraDB-Cluster-garbd-57-5.7.22-29.26.1.el6.x86_64.rpm: Percona-XtraDB-Cluster-garbd-57-5.7.22-29.26.1.el6.x86_64
Percona-XtraDB-Cluster-garbd-57-5.7.22-29.26.1.el6.x86_64.rpm: does not update installed package.
Examining Percona-XtraDB-Cluster-server-57-5.7.22-29.26.1.el6.x86_64.rpm: Percona-XtraDB-Cluster-server-57-5.7.22-29.26.1.el6.x86_64
Marking Percona-XtraDB-Cluster-server-57-5.7.22-29.26.1.el6.x86_64.rpm to be installed
Examining Percona-XtraDB-Cluster-shared-57-5.7.22-29.26.1.el6.x86_64.rpm: Percona-XtraDB-Cluster-shared-57-5.7.22-29.26.1.el6.x86_64
Percona-XtraDB-Cluster-shared-57-5.7.22-29.26.1.el6.x86_64.rpm: does not update installed package.
Examining Percona-XtraDB-Cluster-test-57-5.7.22-29.26.1.el6.x86_64.rpm: Percona-XtraDB-Cluster-test-57-5.7.22-29.26.1.el6.x86_64
Marking Percona-XtraDB-Cluster-test-57-5.7.22-29.26.1.el6.x86_64.rpm to be installed
Resolving Dependencies
--> Running transaction check
---> Package Percona-XtraDB-Cluster-57.x86_64 0:5.7.22-29.26.1.el6 will be installed
---> Package Percona-XtraDB-Cluster-57-debuginfo.x86_64 0:5.7.22-29.26.1.el6 will be installed
---> Package Percona-XtraDB-Cluster-full-57.x86_64 0:5.7.22-29.26.1.el6 will be installed
---> Package Percona-XtraDB-Cluster-server-57.x86_64 0:5.7.22-29.26.1.el6 will be installed
--> Processing Dependency: percona-xtrabackup-24 >= 2.4.12 for package: Percona-XtraDB-Cluster-server-57-5.7.22-29.26.1.el6.x86_64
--> Processing Dependency: qpress for package: Percona-XtraDB-Cluster-server-57-5.7.22-29.26.1.el6.x86_64
---> Package Percona-XtraDB-Cluster-test-57.x86_64 0:5.7.22-29.26.1.el6 will be installed
--> Processing Dependency: perl(Time::HiRes) for package: Percona-XtraDB-Cluster-test-57-5.7.22-29.26.1.el6.x86_64
--> Running transaction check
---> Package percona-xtrabackup-24.x86_64 0:2.4.8-1.el6 will be updated
---> Package percona-xtrabackup-24.x86_64 0:2.4.12-1.el6 will be an update
---> Package perl-Time-HiRes.x86_64 4:1.9721-136.el6 will be installed
---> Package qpress.x86_64 0:11-1.el6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

============================================================================================================================================================
 Package                                   Arch         Version                  Repository                                                            Size
============================================================================================================================================================
Installing:
 Percona-XtraDB-Cluster-57                 x86_64       5.7.22-29.26.1.el6       /Percona-XtraDB-Cluster-57-5.7.22-29.26.1.el6.x86_64                 0.0  
 Percona-XtraDB-Cluster-57-debuginfo       x86_64       5.7.22-29.26.1.el6       /Percona-XtraDB-Cluster-57-debuginfo-5.7.22-29.26.1.el6.x86_64       743 M
 Percona-XtraDB-Cluster-full-57            x86_64       5.7.22-29.26.1.el6       /Percona-XtraDB-Cluster-full-57-5.7.22-29.26.1.el6.x86_64            0.0  
 Percona-XtraDB-Cluster-server-57          x86_64       5.7.22-29.26.1.el6       /Percona-XtraDB-Cluster-server-57-5.7.22-29.26.1.el6.x86_64          230 M
 Percona-XtraDB-Cluster-test-57            x86_64       5.7.22-29.26.1.el6       /Percona-XtraDB-Cluster-test-57-5.7.22-29.26.1.el6.x86_64            364 M
Installing for dependencies:
 perl-Time-HiRes                           x86_64       4:1.9721-136.el6         server                                                                48 k
 qpress                                    x86_64       11-1.el6                 percona-release-x86_64                                                30 k
Updating for dependencies:
 percona-xtrabackup-24                     x86_64       2.4.12-1.el6             percona-release-x86_64                                               8.1 M

Transaction Summary
============================================================================================================================================================

```

## 对PXC进行配置
- node1 
```sql


[root@node1 ~]# cat /etc/my.cnf

[mysqld]
pid-file=/mysql1/mysql1.pid
log-error=/mysql1/mysql1.log
datadir=/mysql1/data
socket=/mysql1/mysql1.sock
user=root
server_id=613306
wsrep_node_name=node1
wsrep_node_address=192.168.2.61
pxc_strict_mode=DISABLED
# Disabling symbolic-links is recommended to prevent assorted security risks
innodb_force_recovery= 0
symbolic-links=0
binlog_format=ROW
default-storage-engine=innodb
innodb_autoinc_lock_mode=2
innodb_locks_unsafe_for_binlog=1
innodb_log_file_size=100M
query_cache_size=0
query_cache_type=0
bind-address=0.0.0.0
wsrep_provider=/usr/lib64/libgalera_smm.so
wsrep_cluster_name=dao_percona_xtrdb_cluster
wsrep_cluster_address=gcomm://192.168.2.61,192.168.2.62,192.168.2.63
wsrep_slave_threads=1
wsrep_certify_nonPK=1
wsrep_max_ws_rows=131072
wsrep_max_ws_size=1073741824
wsrep_debug=0
wsrep_convert_LOCK_to_trx=0
wsrep_retry_autocommit=1
wsrep_auto_increment_control=1
wsrep_drupal_282555_workaround=0
wsrep_causal_reads=0
wsrep_notify_cmd=
wsrep_sst_method=xtrabackup-v2
wsrep_sst_auth=root:root
[mysqld_safe]
log-error=/mysql1/mysql1.log

pid-file=/mysql1/mysql1.pid

```

- node2
```sql

[root@node2 ~]# cat  /etc/my.cnf

[mysqld]


pid-file=/mysql1/mysql1.pid
log-error=/mysql1/mysql1.log
datadir=/mysql1/data
socket=/mysql1/mysql1.sock
user=root
server_id=613306
wsrep_node_address=192.168.2.62
pxc_strict_mode=DISABLED
# Disabling symbolic-links is recommended to prevent assorted security risks
innodb_force_recovery= 0
symbolic-links=0
binlog_format=ROW
default-storage-engine=innodb
innodb_autoinc_lock_mode=2
innodb_locks_unsafe_for_binlog=1
innodb_log_file_size=100M
query_cache_size=0
query_cache_type=0
bind-address=0.0.0.0
wsrep_provider=/usr/lib64/libgalera_smm.so
wsrep_cluster_name=dao_percona_xtrdb_cluster
wsrep_cluster_address=gcomm://192.168.2.61,192.168.2.62,192.168.2.63
wsrep_slave_threads=1
wsrep_certify_nonPK=1
wsrep_max_ws_rows=131072
wsrep_max_ws_size=1073741824
wsrep_debug=0
wsrep_convert_LOCK_to_trx=0
wsrep_retry_autocommit=1
wsrep_auto_increment_control=1
wsrep_drupal_282555_workaround=0
wsrep_causal_reads=0
wsrep_notify_cmd=
wsrep_sst_method=xtrabackup-v2
wsrep_sst_auth=sstuser:sstuser
[mysqld_safe]
log-error=/mysql1/mysql1.log

pid-file=/mysql1/mysql1.pid


```



- node3 

```sql
[root@node3 ~]# cat /etc/my.cnf
[mysqld]
pid-file=/mysql1/mysql1.pid
log-error=/mysql1/mysql1.log
datadir=/mysql1/data
socket=/mysql1/mysql1.sock
user=mysql
server_id=633306
wsrep_node_address=192.168.2.63

wsrep_node_name=node3
# Disabling symbolic-links is recommended to prevent assorted security risks
innodb_force_recovery= 1
symbolic-links=0
binlog_format=ROW
default-storage-engine=innodb
innodb_autoinc_lock_mode=2
innodb_locks_unsafe_for_binlog=1
innodb_log_file_size=100M
query_cache_size=0
query_cache_type=0
bind-address=0.0.0.0
wsrep_provider=/usr/lib64/libgalera_smm.so
wsrep_cluster_name="dao_percona_xtrdb_cluster"
wsrep_cluster_address=gcomm://192.168.2.61,192.168.2.62,192.168.2.63
wsrep_slave_threads=1
wsrep_certify_nonPK=1
wsrep_max_ws_rows=131072
wsrep_max_ws_size=1073741824
wsrep_debug=0
wsrep_convert_LOCK_to_trx=0
wsrep_retry_autocommit=1
wsrep_auto_increment_control=1
wsrep_drupal_282555_workaround=0
wsrep_causal_reads=0
wsrep_notify_cmd=
wsrep_sst_method=xtrabackup-v2
wsrep_sst_auth=dao:dao
[mysqld_safe]
log-error=/mysql1/mysql1.log
pid-file=/mysql1/mysql1.pid



```


### 在节点1初始化数据库

```sql


[mysql@node1 ~]$ mysql_install_db --defaults-file=/etc/my.cnf --data-dir=/mysql1/data/ --user=mysql 
2018-09-04 22:05:40 [WARNING] mysql_install_db is deprecated. Please consider switching to mysqld --initialize

```


## PXC 维护

### 启动PXC

- node1
```sql

[root@node1 ~]# /etc/init.d/mysql  bootstrap-pxc
Bootstrapping PXC (Percona XtraDB Cluster)MySQL (Percona Xt[  OK  ]ster) running (2042)


```
- node2
```sql


[root@node2 ~]# /etc/init.d/mysql start

Initializing MySQL database:                               [  OK  ]
Starting MySQL (Percona XtraDB Cluster)...State transfer in progress, setting sleep higher
..                                                         [  OK  ]

```
- node3

```sql


[root@node3 ~]#  /etc/init.d/mysql start
Starting MySQL (Percona XtraDB Cluster)...State transfer in progress, setting sleep higher
..                                                         [  OK  ]

```