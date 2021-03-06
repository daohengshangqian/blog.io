---
layout: post
title: MySQL MHA使用keepalived漂移IP
date: 2018-08-26
categories: blog
tags: [MySQL]
description: MySQL MHA使用keepalived漂移IP
---



### 架构规划

IP | 主机名 | 角色 |mha角色
--- | --- | --- | ---
192.168.2.61 | node1 | 主库  | node
192.168.2.62 | node2 | 从库1 | node
192.168.2.63 | node3 | 从库2 | node , mha-master


## 安装keepalived(node1,node2)

### 安装keepalived 

```sql

[root@node1 keepalived-1.4.1]# yun -y install keepalived
-bash: yun: command not found
[root@node1 keepalived-1.4.1]# yum -y install keepalived
Loaded plugins: fastestmirror, refresh-packagekit, security
Repository 'Server' is missing name in configuration, using id
Loading mirror speeds from cached hostfile
Setting up Install Process
Resolving Dependencies
--> Running transaction check
---> Package keepalived.x86_64 0:1.2.7-3.el6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

==============================================================================================================================
 Package                        Arch                       Version                           Repository                  Size
==============================================================================================================================
Installing:
 keepalived                     x86_64                     1.2.7-3.el6                       Server                     174 k

Transaction Summary
==============================================================================================================================
Install       1 Package(s)

Total download size: 174 k
Installed size: 526 k
Downloading Packages:
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
  Installing : keepalived-1.2.7-3.el6.x86_64                                                                              1/1
  Verifying  : keepalived-1.2.7-3.el6.x86_64                                                                              1/1

Installed:
  keepalived.x86_64 0:1.2.7-3.el6

Complete!


```


## 配置keepalived 

### node1修改keepalived配置文件

```sql
[root@node1 ~]# cat /etc/keepalived/keepalived.conf
global_defs {
     notification_email {
     saltstack@163.com
   }
   notification_email_from dba@dbserver.com
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id MySQL-HA
}

vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    virtual_router_id 51
    priority 150
    advert_int 1
    nopreempt

    authentication {
    auth_type PASS
    auth_pass 1111
    }

    virtual_ipaddress {
        192.168.6.66/24
    }
}



```


### node2修改keepalived配置文件

```sql



[root@node2 ~]#  cat /etc/keepalived/keepalived.conf
global_defs {
     notification_email {
     saltstack@163.com
   }
   notification_email_from dba@dbserver.com
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id MySQL-HA
}

vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    virtual_router_id 51
    priority 150
    advert_int 1
    nopreempt

    authentication {
    auth_type PASS
    auth_pass 1111
    }

    virtual_ipaddress {
        192.168.6.66/24
    }
}


```

## 修改原有MHA配置

### 修改manager 配置文件

```sql


[root@node3 ~]# cat /etc/mha/app1/app1.cnf

[server default]
# working directory on the manager
manager_workdir=/etc/mha/app1
manager_log=/etc/mha/app1/manager.log
remote_workdir=/etc/mha/app1
# In case of automatic file-over, the following script will be used
## 此处修改文件名加后缀.pl
master_ip_failover_script=/etc/mha/master_ip_failover.pl
master_ip_online_change_script=/etc/mha/master_ip_online_change.pl

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

### master_ip_failover_script.pl

```sql

[root@node3 ~]# cat /etc/mha/master_ip_failover.pl
#!/usr/bin/env perl

use strict;
use warnings FATAL => 'all';

use Getopt::Long;
my (
    $command,          $ssh_user,        $orig_master_host, $orig_master_ip,
    $orig_master_port, $new_master_host, $new_master_ip,    $new_master_port
);

my $vip = '192.168.6.66/24';
my $key = '0';
my $ssh_start_vip = "/sbin/ifconfig eth0:$key $vip";
my $ssh_stop_vip = "/sbin/ifconfig eth0:$key down";

GetOptions(
    'command=s'          => \$command,
    'ssh_user=s'         => \$ssh_user,
    'orig_master_host=s' => \$orig_master_host,
    'orig_master_ip=s'   => \$orig_master_ip,
    'orig_master_port=i' => \$orig_master_port,
    'new_master_host=s'  => \$new_master_host,
    'new_master_ip=s'    => \$new_master_ip,
    'new_master_port=i'  => \$new_master_port,
);

exit &main();

sub main {

    print "\n\nIN SCRIPT TEST====$ssh_stop_vip==$ssh_start_vip===\n\n";

    if ( $command eq "stop" || $command eq "stopssh" ) {

        my $exit_code = 1;
        eval {
            print "Disabling the VIP on old master: $orig_master_host \n";
            &stop_vip();
            $exit_code = 0;
        };
        if ($@) {
            warn "Got Error: $@\n";
            exit $exit_code;
        }
        exit $exit_code;
    }
    elsif ( $command eq "start" ) {

        my $exit_code = 10;
        eval {
            print "Enabling the VIP - $vip on the new master - $new_master_host \n";
            &start_vip();
            $exit_code = 0;
        };
        if ($@) {
            warn $@;
            exit $exit_code;
        }
        exit $exit_code;
    }
    elsif ( $command eq "status" ) {
        print "Checking the Status of the script.. OK \n";
        exit 0;
    }
    else {
        &usage();
        exit 1;
    }
}

sub start_vip() {
    `ssh $ssh_user\@$new_master_host \" $ssh_start_vip \"`;
}
sub stop_vip() {
     return 0  unless  ($ssh_user);
    `ssh $ssh_user\@$orig_master_host \" $ssh_stop_vip \"`;
}

sub usage {
    print
    "Usage: master_ip_failover --command=start|stop|stopssh|status --orig_master_host=host --orig_master_ip=ip
            --orig_master_port=port --new_master_host=host --new_master_ip=ip --new_master_port=port\n";
}


```



### master_ip_online_change_script.pl

```sql

[root@node3 ~]# cat /etc/mha/master_ip_online_change.pl
#!/usr/bin/env perl
use strict;
use warnings FATAL =>'all';

use Getopt::Long;

my $vip = '192.168.6.66/24';  # Virtual IP
my $key = "0";
my $ssh_start_vip = "/sbin/ifconfig eth0:$key $vip";
my $ssh_stop_vip = "/sbin/ifconfig eth0:$key down";
my $exit_code = 0;

my (
  $command,              $orig_master_is_new_slave, $orig_master_host,
  $orig_master_ip,       $orig_master_port,         $orig_master_user,
  $orig_master_password, $orig_master_ssh_user,     $new_master_host,
  $new_master_ip,        $new_master_port,          $new_master_user,
  $new_master_password,  $new_master_ssh_user,
);
GetOptions(
  'command=s'                => \$command,
  'orig_master_is_new_slave' => \$orig_master_is_new_slave,
  'orig_master_host=s'       => \$orig_master_host,
  'orig_master_ip=s'         => \$orig_master_ip,
  'orig_master_port=i'       => \$orig_master_port,
  'orig_master_user=s'       => \$orig_master_user,
  'orig_master_password=s'   => \$orig_master_password,
  'orig_master_ssh_user=s'   => \$orig_master_ssh_user,
  'new_master_host=s'        => \$new_master_host,
  'new_master_ip=s'          => \$new_master_ip,
  'new_master_port=i'        => \$new_master_port,
  'new_master_user=s'        => \$new_master_user,
  'new_master_password=s'    => \$new_master_password,
  'new_master_ssh_user=s'    => \$new_master_ssh_user,
);


exit &main();

sub main {

#print "\n\nIN SCRIPT TEST====$ssh_stop_vip==$ssh_start_vip===\n\n";

if ( $command eq "stop" || $command eq "stopssh" ) {

        # $orig_master_host, $orig_master_ip, $orig_master_port are passed.
        # If you manage master ip address at global catalog database,
        # invalidate orig_master_ip here.
        my $exit_code = 1;
        eval {
            print "\n\n\n***************************************************************\n";
            print "Disabling the VIP - $vip on old master: $orig_master_host\n";
            print "***************************************************************\n\n\n\n";
&stop_vip();
            $exit_code = 0;
        };
        if ($@) {
            warn "Got Error: $@\n";
            exit $exit_code;
        }
        exit $exit_code;
}
elsif ( $command eq "start" ) {

        # all arguments are passed.
        # If you manage master ip address at global catalog database,
        # activate new_master_ip here.
        # You can also grant write access (create user, set read_only=0, etc) here.
my $exit_code = 10;
        eval {
            print "\n\n\n***************************************************************\n";
            print "Enabling the VIP - $vip on new master: $new_master_host \n";
            print "***************************************************************\n\n\n\n";
&start_vip();
            $exit_code = 0;
        };
        if ($@) {
            warn $@;
            exit $exit_code;
        }
        exit $exit_code;
}
elsif ( $command eq "status" ) {
        print "Checking the Status of the script.. OK \n";
        `ssh $orig_master_ssh_user\@$orig_master_host \" $ssh_start_vip \"`;
        exit 0;
}
else {
&usage();
        exit 1;
}
}

# A simple system call that enable the VIP on the new master
sub start_vip() {
`ssh $new_master_ssh_user\@$new_master_host \" $ssh_start_vip \"`;
}
# A simple system call that disable the VIP on the old_master
sub stop_vip() {
`ssh $orig_master_ssh_user\@$orig_master_host \" $ssh_stop_vip \"`;
}

sub usage {
print
"Usage: master_ip_failover -command=start|stop|stopssh|status -orig_master_host=host -orig_master_ip=ip -orig_master_port=po
rt -new_master_host=host -new_master_ip=ip -new_master_port=port\n";
}

```

## 验证

### 找到从库当前指向的主库

```sql

mysql> show slave status \G ;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.2.62
                  Master_User: dao
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000017
          Read_Master_Log_Pos: 234
               Relay_Log_File: node3-relay-bin.000003
                Relay_Log_Pos: 320
        Relay_Master_Log_File: mysql-bin.000017
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 234
              Relay_Log_Space: 693
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 0
               Last_SQL_Error:
  Replicate_Ignore_Server_Ids:
             Master_Server_Id: 2
                  Master_UUID: f63b765b-9c1d-11e8-aa1e-005056296858
             Master_Info_File: /mysql1/data/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind:
      Last_IO_Error_Timestamp:
     Last_SQL_Error_Timestamp:
               Master_SSL_Crl:
           Master_SSL_Crlpath:
           Retrieved_Gtid_Set:
            Executed_Gtid_Set:
                Auto_Position: 0
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
1 row in set (0.03 sec)


```

### 查看主库IP

```sql

[mysql@node2 ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue state UNKNOWN
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:50:56:29:68:58 brd ff:ff:ff:ff:ff:ff
    inet 192.168.2.62/24 brd 192.168.2.255 scope global eth0
    inet 192.168.2.66/24 brd 192.168.2.255 scope global secondary eth0:0  ## 66为VIP
    inet6 fe80::250:56ff:fe29:6858/64 scope link
       valid_lft forever preferred_lft forever


```

### 使用MHA脚本检查复制信息

```sql

[root@node3 ~]# masterha_secondary_check -s 192.168.2.62 -s 192.168.2.63 --user=root --master_host=node1 --master_ip=192.168.2.61 --master_port=3306
Master is reachable from 192.168.2.62!
[root@node3 ~]# /etc/mha/mhamanager/bin/masterha_check_repl --conf=/etc/mha/app1/app1.cnf                                     Sat Aug 25 04:52:54 2018 - [warning] Global configuration file /etc/masterha_default.cnf not found. Skipping.
Sat Aug 25 04:52:54 2018 - [info] Reading application default configuration from /etc/mha/app1/app1.cnf..
Sat Aug 25 04:52:54 2018 - [info] Reading server configuration from /etc/mha/app1/app1.cnf..
Sat Aug 25 04:52:54 2018 - [info] MHA::MasterMonitor version 0.57.
Sat Aug 25 04:52:55 2018 - [info] GTID failover mode = 0
Sat Aug 25 04:52:55 2018 - [info] Dead Servers:
Sat Aug 25 04:52:55 2018 - [info] Alive Servers:
Sat Aug 25 04:52:55 2018 - [info]   192.168.2.61(192.168.2.61:3306)
Sat Aug 25 04:52:55 2018 - [info]   192.168.2.62(192.168.2.62:3306)
Sat Aug 25 04:52:55 2018 - [info]   192.168.2.63(192.168.2.63:3306)
Sat Aug 25 04:52:55 2018 - [info] Alive Slaves:
Sat Aug 25 04:52:55 2018 - [info]   192.168.2.61(192.168.2.61:3306)  Version=5.7.21-log (oldest major version between slaves) log-bin:enabled
Sat Aug 25 04:52:55 2018 - [info]     Replicating from node2(192.168.2.62:3306)
Sat Aug 25 04:52:55 2018 - [info]     Primary candidate for the new Master (candidate_master is set)
Sat Aug 25 04:52:55 2018 - [info]   192.168.2.63(192.168.2.63:3306)  Version=5.7.21-log (oldest major version between slaves) log-bin:enabled
Sat Aug 25 04:52:55 2018 - [info]     Replicating from 192.168.2.62(192.168.2.62:3306)
Sat Aug 25 04:52:55 2018 - [info]     Not candidate for the new Master (no_master is set)
Sat Aug 25 04:52:55 2018 - [info] Current Alive Master: 192.168.2.62(192.168.2.62:3306)
Sat Aug 25 04:52:55 2018 - [info] Checking slave configurations..
Sat Aug 25 04:52:55 2018 - [info]  read_only=1 is not set on slave 192.168.2.61(192.168.2.61:3306).
Sat Aug 25 04:52:55 2018 - [warning]  relay_log_purge=0 is not set on slave 192.168.2.61(192.168.2.61:3306).
Sat Aug 25 04:52:55 2018 - [info]  read_only=1 is not set on slave 192.168.2.63(192.168.2.63:3306).
Sat Aug 25 04:52:55 2018 - [warning]  relay_log_purge=0 is not set on slave 192.168.2.63(192.168.2.63:3306).
Sat Aug 25 04:52:55 2018 - [info] Checking replication filtering settings..
Sat Aug 25 04:52:55 2018 - [info]  binlog_do_db= , binlog_ignore_db=
Sat Aug 25 04:52:55 2018 - [info]  Replication filtering check ok.
Sat Aug 25 04:52:55 2018 - [info] GTID (with auto-pos) is not supported
Sat Aug 25 04:52:55 2018 - [info] Starting SSH connection tests..
Sat Aug 25 04:52:57 2018 - [info] All SSH connection tests passed successfully.
Sat Aug 25 04:52:57 2018 - [info] Checking MHA Node version..
Sat Aug 25 04:52:57 2018 - [info]  Version check ok.
Sat Aug 25 04:52:57 2018 - [info] Checking SSH publickey authentication settings on the current master..
Sat Aug 25 04:52:57 2018 - [info] HealthCheck: SSH to 192.168.2.62 is reachable.
Sat Aug 25 04:52:58 2018 - [info] Master MHA Node version is 0.57.
Sat Aug 25 04:52:58 2018 - [info] Checking recovery script configurations on 192.168.2.62(192.168.2.62:3306)..
Sat Aug 25 04:52:58 2018 - [info]   Executing command: save_binary_logs --command=test --start_pos=4 --binlog_dir=/mysql1/data --output_file=/etc/mha/app1/save_binary_logs_test --manager_version=0.57 --start_file=mysql-bin.000017
Sat Aug 25 04:52:58 2018 - [info]   Connecting to root@192.168.2.62(192.168.2.62:22)..
  Creating /etc/mha/app1 if not exists..    ok.
  Checking output directory is accessible or not..
   ok.
  Binlog found at /mysql1/data, up to mysql-bin.000017
Sat Aug 25 04:52:58 2018 - [info] Binlog setting check done.
Sat Aug 25 04:52:58 2018 - [info] Checking SSH publickey authentication and checking recovery script configurations on all alive slave servers..
Sat Aug 25 04:52:58 2018 - [info]   Executing command : apply_diff_relay_logs --command=test --slave_user='root' --slave_host=192.168.2.61 --slave_ip=192.168.2.61 --slave_port=3306 --workdir=/etc/mha/app1 --target_version=5.7.21-log --manager_version=0.57 --relay_log_info=/mysql1/data/relay-log.info  --relay_dir=/mysql1/data/  --slave_pass=xxx
Sat Aug 25 04:52:58 2018 - [info]   Connecting to root@192.168.2.61(192.168.2.61:22)..
  Checking slave recovery environment settings..
    Opening /mysql1/data/relay-log.info ... ok.
    Relay log found at /mysql1/data, up to node1-relay-bin.000002
    Temporary relay log file is /mysql1/data/node1-relay-bin.000002
    Testing mysql connection and privileges..mysql: [Warning] Using a password on the command line interface can be insecure.
 done.
    Testing mysqlbinlog output.. done.
    Cleaning up test file(s).. done.
Sat Aug 25 04:52:59 2018 - [info]   Executing command : apply_diff_relay_logs --command=test --slave_user='root' --slave_host=192.168.2.63 --slave_ip=192.168.2.63 --slave_port=3306 --workdir=/etc/mha/app1 --target_version=5.7.21-log --manager_version=0.57 --relay_log_info=/mysql1/data/relay-log.info  --relay_dir=/mysql1/data/  --slave_pass=xxx
Sat Aug 25 04:52:59 2018 - [info]   Connecting to root@192.168.2.63(192.168.2.63:22)..
  Checking slave recovery environment settings..
    Opening /mysql1/data/relay-log.info ... ok.
    Relay log found at /mysql1/data, up to node3-relay-bin.000003
    Temporary relay log file is /mysql1/data/node3-relay-bin.000003
    Testing mysql connection and privileges..mysql: [Warning] Using a password on the command line interface can be insecure.
 done.
    Testing mysqlbinlog output.. done.
    Cleaning up test file(s).. done.
Sat Aug 25 04:52:59 2018 - [info] Slaves settings check done.
Sat Aug 25 04:52:59 2018 - [info]
192.168.2.62(192.168.2.62:3306) (current master)
 +--192.168.2.61(192.168.2.61:3306)
 +--192.168.2.63(192.168.2.63:3306)

Sat Aug 25 04:52:59 2018 - [info] Checking replication health on 192.168.2.61..
Sat Aug 25 04:52:59 2018 - [info]  ok.
Sat Aug 25 04:52:59 2018 - [info] Checking replication health on 192.168.2.63..
Sat Aug 25 04:52:59 2018 - [info]  ok.
Sat Aug 25 04:52:59 2018 - [info] Checking master_ip_failover_script status:
Sat Aug 25 04:52:59 2018 - [info]   /etc/mha/master_ip_failover.pl --command=status --ssh_user=root --orig_master_host=192.168.2.62 --orig_master_ip=192.168.2.62 --orig_master_port=3306


IN SCRIPT TEST====/sbin/ifconfig eth0:0 down==/sbin/ifconfig eth0:0 192.168.2.66/24===

Checking the Status of the script.. OK
Sat Aug 25 04:52:59 2018 - [info]  OK.
Sat Aug 25 04:52:59 2018 - [warning] shutdown_script is not defined.
Sat Aug 25 04:52:59 2018 - [info] Got exit code 0 (Not master dead).

MySQL Replication Health is OK.


```
### 开启MHA manager

```sql


[root@node3 ~]# nohup /etc/mha/mhamanager/bin/masterha_manager --conf=/etc/mha/app1/app1.cnf --ignore_last_failover >/tmp/mha_manager.log < /dev/null 2>&1 &
[1] 23119


```
### 停掉主库

```sql

[mysql@node2 ~]$ mysqladmin -S /tmp/mysql1.sock -uroot -proot shutdown
mysqladmin: [Warning] Using a password on the command line interface can be insecure.


```
### 查看MHA日志

```sql
[root@node3 ~]# echo "" >/etc/mha/app1/manager.log
[root@node3 ~]# tail -f /etc/mha/app1/manager.log

Sat Aug 25 04:54:03 2018 - [warning] Got error on MySQL select ping: 2006 (MySQL server has gone away)
Sat Aug 25 04:54:03 2018 - [info] Executing secondary network check script: masterha_secondary_check -s 192.168.2.62 -s 192.168.2.63  --user=root  --master_host=192.168.2.62  --master_ip=192.168.2.62  --master_port=3306 --master_user=root --master_password=root --ping_type=SELECT
Sat Aug 25 04:54:03 2018 - [info] Executing SSH check script: save_binary_logs --command=test --start_pos=4 --binlog_dir=/mysql1/data --output_file=/etc/mha/app1/save_binary_logs_test --manager_version=0.57 --binlog_prefix=mysql-bin
Monitoring server 192.168.2.62 is reachable, Master is not reachable from 192.168.2.62. OK.
Sat Aug 25 04:54:03 2018 - [info] HealthCheck: SSH to 192.168.2.62 is reachable.
Monitoring server 192.168.2.63 is reachable, Master is not reachable from 192.168.2.63. OK.
Sat Aug 25 04:54:03 2018 - [info] Master is not reachable from all other monitoring servers. Failover should start.
Sat Aug 25 04:54:04 2018 - [warning] Got error on MySQL connect: 2013 (Lost connection to MySQL server at 'reading initial communication packet', system error: 111)
Sat Aug 25 04:54:04 2018 - [warning] Connection failed 2 time(s)..
Sat Aug 25 04:54:05 2018 - [warning] Got error on MySQL connect: 2013 (Lost connection to MySQL server at 'reading initial communication packet', system error: 111)
Sat Aug 25 04:54:05 2018 - [warning] Connection failed 3 time(s)..
Sat Aug 25 04:54:06 2018 - [warning] Got error on MySQL connect: 2013 (Lost connection to MySQL server at 'reading initial communication packet', system error: 111)
Sat Aug 25 04:54:06 2018 - [warning] Connection failed 4 time(s)..
Sat Aug 25 04:54:06 2018 - [warning] Master is not reachable from health checker!
Sat Aug 25 04:54:06 2018 - [warning] Master 192.168.2.62(192.168.2.62:3306) is not reachable!
Sat Aug 25 04:54:06 2018 - [warning] SSH is reachable.
Sat Aug 25 04:54:06 2018 - [info] Connecting to a master server failed. Reading configuration file /etc/masterha_default.cnf and /etc/mha/app1/app1.cnf again, and trying to connect to all servers to check server status..
Sat Aug 25 04:54:06 2018 - [warning] Global configuration file /etc/masterha_default.cnf not found. Skipping.
Sat Aug 25 04:54:06 2018 - [info] Reading application default configuration from /etc/mha/app1/app1.cnf..
Sat Aug 25 04:54:06 2018 - [info] Reading server configuration from /etc/mha/app1/app1.cnf..
Sat Aug 25 04:54:06 2018 - [info] GTID failover mode = 0
Sat Aug 25 04:54:06 2018 - [info] Dead Servers:
Sat Aug 25 04:54:06 2018 - [info]   192.168.2.62(192.168.2.62:3306)
Sat Aug 25 04:54:06 2018 - [info] Alive Servers:
Sat Aug 25 04:54:06 2018 - [info]   192.168.2.61(192.168.2.61:3306)
Sat Aug 25 04:54:06 2018 - [info]   192.168.2.63(192.168.2.63:3306)
Sat Aug 25 04:54:06 2018 - [info] Alive Slaves:
Sat Aug 25 04:54:06 2018 - [info]   192.168.2.61(192.168.2.61:3306)  Version=5.7.21-log (oldest major version between slaves) log-bin:enabled
Sat Aug 25 04:54:06 2018 - [info]     Replicating from node2(192.168.2.62:3306)
Sat Aug 25 04:54:06 2018 - [info]     Primary candidate for the new Master (candidate_master is set)
Sat Aug 25 04:54:06 2018 - [info]   192.168.2.63(192.168.2.63:3306)  Version=5.7.21-log (oldest major version between slaves) log-bin:enabled
Sat Aug 25 04:54:06 2018 - [info]     Replicating from 192.168.2.62(192.168.2.62:3306)
Sat Aug 25 04:54:06 2018 - [info]     Not candidate for the new Master (no_master is set)
Sat Aug 25 04:54:06 2018 - [info] Checking slave configurations..
Sat Aug 25 04:54:06 2018 - [info]  read_only=1 is not set on slave 192.168.2.61(192.168.2.61:3306).
Sat Aug 25 04:54:06 2018 - [warning]  relay_log_purge=0 is not set on slave 192.168.2.61(192.168.2.61:3306).
Sat Aug 25 04:54:06 2018 - [info]  read_only=1 is not set on slave 192.168.2.63(192.168.2.63:3306).
Sat Aug 25 04:54:06 2018 - [warning]  relay_log_purge=0 is not set on slave 192.168.2.63(192.168.2.63:3306).
Sat Aug 25 04:54:06 2018 - [info] Checking replication filtering settings..
Sat Aug 25 04:54:06 2018 - [info]  Replication filtering check ok.
Sat Aug 25 04:54:06 2018 - [info] Master is down!
Sat Aug 25 04:54:06 2018 - [info] Terminating monitoring script.
Sat Aug 25 04:54:06 2018 - [info] Got exit code 20 (Master dead).
Sat Aug 25 04:54:06 2018 - [info] MHA::MasterFailover version 0.57.
Sat Aug 25 04:54:06 2018 - [info] Starting master failover.
Sat Aug 25 04:54:06 2018 - [info]
Sat Aug 25 04:54:06 2018 - [info] * Phase 1: Configuration Check Phase..
Sat Aug 25 04:54:06 2018 - [info]
Sat Aug 25 04:54:06 2018 - [info] GTID failover mode = 0
Sat Aug 25 04:54:06 2018 - [info] Dead Servers:
Sat Aug 25 04:54:06 2018 - [info]   192.168.2.62(192.168.2.62:3306)
Sat Aug 25 04:54:06 2018 - [info] Checking master reachability via MySQL(double check)...
Sat Aug 25 04:54:06 2018 - [info]  ok.
Sat Aug 25 04:54:06 2018 - [info] Alive Servers:
Sat Aug 25 04:54:06 2018 - [info]   192.168.2.61(192.168.2.61:3306)
Sat Aug 25 04:54:06 2018 - [info]   192.168.2.63(192.168.2.63:3306)
Sat Aug 25 04:54:06 2018 - [info] Alive Slaves:
Sat Aug 25 04:54:06 2018 - [info]   192.168.2.61(192.168.2.61:3306)  Version=5.7.21-log (oldest major version between slaves) log-bin:enabled
Sat Aug 25 04:54:06 2018 - [info]     Replicating from node2(192.168.2.62:3306)
Sat Aug 25 04:54:06 2018 - [info]     Primary candidate for the new Master (candidate_master is set)
Sat Aug 25 04:54:06 2018 - [info]   192.168.2.63(192.168.2.63:3306)  Version=5.7.21-log (oldest major version between slaves) log-bin:enabled
Sat Aug 25 04:54:06 2018 - [info]     Replicating from 192.168.2.62(192.168.2.62:3306)
Sat Aug 25 04:54:06 2018 - [info]     Not candidate for the new Master (no_master is set)
Sat Aug 25 04:54:06 2018 - [info] Starting Non-GTID based failover.
Sat Aug 25 04:54:06 2018 - [info]
Sat Aug 25 04:54:06 2018 - [info] ** Phase 1: Configuration Check Phase completed.
Sat Aug 25 04:54:06 2018 - [info]
Sat Aug 25 04:54:06 2018 - [info] * Phase 2: Dead Master Shutdown Phase..
Sat Aug 25 04:54:06 2018 - [info]
Sat Aug 25 04:54:06 2018 - [info] Forcing shutdown so that applications never connect to the current master..
Sat Aug 25 04:54:06 2018 - [info] Executing master IP deactivation script:
Sat Aug 25 04:54:06 2018 - [info]   /etc/mha/master_ip_failover.pl --orig_master_host=192.168.2.62 --orig_master_ip=192.168.2.62 --orig_master_port=3306 --command=stopssh --ssh_user=root


IN SCRIPT TEST====/sbin/ifconfig eth0:0 down==/sbin/ifconfig eth0:0 192.168.2.66/24===

Disabling the VIP on old master: 192.168.2.62
Sat Aug 25 04:54:06 2018 - [info]  done.
Sat Aug 25 04:54:06 2018 - [warning] shutdown_script is not set. Skipping explicit shutting down of the dead master.
Sat Aug 25 04:54:06 2018 - [info] * Phase 2: Dead Master Shutdown Phase completed.
Sat Aug 25 04:54:06 2018 - [info]
Sat Aug 25 04:54:06 2018 - [info] * Phase 3: Master Recovery Phase..
Sat Aug 25 04:54:06 2018 - [info]
Sat Aug 25 04:54:06 2018 - [info] * Phase 3.1: Getting Latest Slaves Phase..
Sat Aug 25 04:54:06 2018 - [info]
Sat Aug 25 04:54:06 2018 - [info] The latest binary log file/position on all slaves is mysql-bin.000017:234
Sat Aug 25 04:54:06 2018 - [info] Latest slaves (Slaves that received relay log files to the latest):
Sat Aug 25 04:54:06 2018 - [info]   192.168.2.61(192.168.2.61:3306)  Version=5.7.21-log (oldest major version between slaves) log-bin:enabled
Sat Aug 25 04:54:06 2018 - [info]     Replicating from node2(192.168.2.62:3306)
Sat Aug 25 04:54:06 2018 - [info]     Primary candidate for the new Master (candidate_master is set)
Sat Aug 25 04:54:06 2018 - [info]   192.168.2.63(192.168.2.63:3306)  Version=5.7.21-log (oldest major version between slaves) log-bin:enabled
Sat Aug 25 04:54:06 2018 - [info]     Replicating from 192.168.2.62(192.168.2.62:3306)
Sat Aug 25 04:54:06 2018 - [info]     Not candidate for the new Master (no_master is set)
Sat Aug 25 04:54:06 2018 - [info] The oldest binary log file/position on all slaves is mysql-bin.000017:234
Sat Aug 25 04:54:06 2018 - [info] Oldest slaves:
Sat Aug 25 04:54:06 2018 - [info]   192.168.2.61(192.168.2.61:3306)  Version=5.7.21-log (oldest major version between slaves) log-bin:enabled
Sat Aug 25 04:54:06 2018 - [info]     Replicating from node2(192.168.2.62:3306)
Sat Aug 25 04:54:06 2018 - [info]     Primary candidate for the new Master (candidate_master is set)
Sat Aug 25 04:54:06 2018 - [info]   192.168.2.63(192.168.2.63:3306)  Version=5.7.21-log (oldest major version between slaves) log-bin:enabled
Sat Aug 25 04:54:06 2018 - [info]     Replicating from 192.168.2.62(192.168.2.62:3306)
Sat Aug 25 04:54:06 2018 - [info]     Not candidate for the new Master (no_master is set)
Sat Aug 25 04:54:06 2018 - [info]
Sat Aug 25 04:54:06 2018 - [info] * Phase 3.2: Saving Dead Master's Binlog Phase..
Sat Aug 25 04:54:06 2018 - [info]
Sat Aug 25 04:54:07 2018 - [info] Fetching dead master's binary logs..
Sat Aug 25 04:54:07 2018 - [info] Executing command on the dead master 192.168.2.62(192.168.2.62:3306): save_binary_logs --command=save --start_file=mysql-bin.000017  --start_pos=234 --binlog_dir=/mysql1/data --output_file=/etc/mha/app1/saved_master_binlog_from_192.168.2.62_3306_20180825045406.binlog --handle_raw_binlog=1 --disable_log_bin=0 --manager_version=0.57
  Creating /etc/mha/app1 if not exists..    ok.
 Concat binary/relay logs from mysql-bin.000017 pos 234 to mysql-bin.000017 EOF into /etc/mha/app1/saved_master_binlog_from_192.168.2.62_3306_20180825045406.binlog ..
 Binlog Checksum enabled
  Dumping binlog format description event, from position 0 to 234.. ok.
  No need to dump effective binlog data from /mysql1/data/mysql-bin.000017 (pos starts 234, filesize 234). Skipping.
 Binlog Checksum enabled
 /etc/mha/app1/saved_master_binlog_from_192.168.2.62_3306_20180825045406.binlog has no effective data events.
Event not exists.
Sat Aug 25 04:54:07 2018 - [info] Additional events were not found from the orig master. No need to save.
Sat Aug 25 04:54:07 2018 - [info]
Sat Aug 25 04:54:07 2018 - [info] * Phase 3.3: Determining New Master Phase..
Sat Aug 25 04:54:07 2018 - [info]
Sat Aug 25 04:54:07 2018 - [info] Finding the latest slave that has all relay logs for recovering other slaves..
Sat Aug 25 04:54:07 2018 - [info] All slaves received relay logs to the same position. No need to resync each other.
Sat Aug 25 04:54:07 2018 - [info] Searching new master from slaves..
Sat Aug 25 04:54:07 2018 - [info]  Candidate masters from the configuration file:
Sat Aug 25 04:54:07 2018 - [info]   192.168.2.61(192.168.2.61:3306)  Version=5.7.21-log (oldest major version between slaves) log-bin:enabled
Sat Aug 25 04:54:07 2018 - [info]     Replicating from node2(192.168.2.62:3306)
Sat Aug 25 04:54:07 2018 - [info]     Primary candidate for the new Master (candidate_master is set)
Sat Aug 25 04:54:07 2018 - [info]  Non-candidate masters:
Sat Aug 25 04:54:07 2018 - [info]   192.168.2.63(192.168.2.63:3306)  Version=5.7.21-log (oldest major version between slaves) log-bin:enabled
Sat Aug 25 04:54:07 2018 - [info]     Replicating from 192.168.2.62(192.168.2.62:3306)
Sat Aug 25 04:54:07 2018 - [info]     Not candidate for the new Master (no_master is set)
Sat Aug 25 04:54:07 2018 - [info]  Searching from candidate_master slaves which have received the latest relay log events..
Sat Aug 25 04:54:07 2018 - [info] New master is 192.168.2.61(192.168.2.61:3306)
Sat Aug 25 04:54:07 2018 - [info] Starting master failover..
Sat Aug 25 04:54:07 2018 - [info]
From:
192.168.2.62(192.168.2.62:3306) (current master)
 +--192.168.2.61(192.168.2.61:3306)
 +--192.168.2.63(192.168.2.63:3306)

To:
192.168.2.61(192.168.2.61:3306) (new master)
 +--192.168.2.63(192.168.2.63:3306)
Sat Aug 25 04:54:07 2018 - [info]
Sat Aug 25 04:54:07 2018 - [info] * Phase 3.3: New Master Diff Log Generation Phase..
Sat Aug 25 04:54:07 2018 - [info]
Sat Aug 25 04:54:07 2018 - [info]  This server has all relay logs. No need to generate diff files from the latest slave.
Sat Aug 25 04:54:07 2018 - [info]
Sat Aug 25 04:54:07 2018 - [info] * Phase 3.4: Master Log Apply Phase..
Sat Aug 25 04:54:07 2018 - [info]
Sat Aug 25 04:54:07 2018 - [info] *NOTICE: If any error happens from this phase, manual recovery is needed.
Sat Aug 25 04:54:07 2018 - [info] Starting recovery on 192.168.2.61(192.168.2.61:3306)..
Sat Aug 25 04:54:07 2018 - [info]  This server has all relay logs. Waiting all logs to be applied..
Sat Aug 25 04:54:07 2018 - [info]   done.
Sat Aug 25 04:54:07 2018 - [info]  All relay logs were successfully applied.
Sat Aug 25 04:54:07 2018 - [info] Getting new master's binlog name and position..
Sat Aug 25 04:54:07 2018 - [info]  mysql-bin.000007:154
Sat Aug 25 04:54:07 2018 - [info]  All other slaves should start replication from here. Statement should be: CHANGE MASTER TO MASTER_HOST='192.168.2.61', MASTER_PORT=3306, MASTER_LOG_FILE='mysql-bin.000007', MASTER_LOG_POS=154, MASTER_USER='dao', MASTER_PASSWORD='xxx';
Sat Aug 25 04:54:07 2018 - [info] Executing master IP activate script:
Sat Aug 25 04:54:07 2018 - [info]   /etc/mha/master_ip_failover.pl --command=start --ssh_user=root --orig_master_host=192.168.2.62 --orig_master_ip=192.168.2.62 --orig_master_port=3306 --new_master_host=192.168.2.61 --new_master_ip=192.168.2.61 --new_master_port=3306 --new_master_user='root'   --new_master_password=xxx
Unknown option: new_master_user
Unknown option: new_master_password


IN SCRIPT TEST====/sbin/ifconfig eth0:0 down==/sbin/ifconfig eth0:0 192.168.2.66/24===

Enabling the VIP - 192.168.2.66/24 on the new master - 192.168.2.61
Sat Aug 25 04:54:08 2018 - [info]  OK.
Sat Aug 25 04:54:08 2018 - [info] ** Finished master recovery successfully.
Sat Aug 25 04:54:08 2018 - [info] * Phase 3: Master Recovery Phase completed.
Sat Aug 25 04:54:08 2018 - [info]
Sat Aug 25 04:54:08 2018 - [info] * Phase 4: Slaves Recovery Phase..
Sat Aug 25 04:54:08 2018 - [info]
Sat Aug 25 04:54:08 2018 - [info] * Phase 4.1: Starting Parallel Slave Diff Log Generation Phase..
Sat Aug 25 04:54:08 2018 - [info]
Sat Aug 25 04:54:08 2018 - [info] -- Slave diff file generation on host 192.168.2.63(192.168.2.63:3306) started, pid: 23329. Check tmp log /etc/mha/app1/192.168.2.63_3306_20180825045406.log if it takes time..
Sat Aug 25 04:54:08 2018 - [info]
Sat Aug 25 04:54:08 2018 - [info] Log messages from 192.168.2.63 ...
Sat Aug 25 04:54:08 2018 - [info]
Sat Aug 25 04:54:08 2018 - [info]  This server has all relay logs. No need to generate diff files from the latest slave.
Sat Aug 25 04:54:08 2018 - [info] End of log messages from 192.168.2.63.
Sat Aug 25 04:54:08 2018 - [info] -- 192.168.2.63(192.168.2.63:3306) has the latest relay log events.
Sat Aug 25 04:54:08 2018 - [info] Generating relay diff files from the latest slave succeeded.
Sat Aug 25 04:54:08 2018 - [info]
Sat Aug 25 04:54:08 2018 - [info] * Phase 4.2: Starting Parallel Slave Log Apply Phase..
Sat Aug 25 04:54:08 2018 - [info]
Sat Aug 25 04:54:08 2018 - [info] -- Slave recovery on host 192.168.2.63(192.168.2.63:3306) started, pid: 23331. Check tmp log /etc/mha/app1/192.168.2.63_3306_20180825045406.log if it takes time..
Sat Aug 25 04:54:08 2018 - [info]
Sat Aug 25 04:54:08 2018 - [info] Log messages from 192.168.2.63 ...
Sat Aug 25 04:54:08 2018 - [info]
Sat Aug 25 04:54:08 2018 - [info] Starting recovery on 192.168.2.63(192.168.2.63:3306)..
Sat Aug 25 04:54:08 2018 - [info]  This server has all relay logs. Waiting all logs to be applied..
Sat Aug 25 04:54:08 2018 - [info]   done.
Sat Aug 25 04:54:08 2018 - [info]  All relay logs were successfully applied.
Sat Aug 25 04:54:08 2018 - [info]  Resetting slave 192.168.2.63(192.168.2.63:3306) and starting replication from the new master 192.168.2.61(192.168.2.61:3306)..
Sat Aug 25 04:54:08 2018 - [info]  Executed CHANGE MASTER.
Sat Aug 25 04:54:08 2018 - [info]  Slave started.
Sat Aug 25 04:54:08 2018 - [info] End of log messages from 192.168.2.63.
Sat Aug 25 04:54:08 2018 - [info] -- Slave recovery on host 192.168.2.63(192.168.2.63:3306) succeeded.
Sat Aug 25 04:54:08 2018 - [info] All new slave servers recovered successfully.
Sat Aug 25 04:54:08 2018 - [info]
Sat Aug 25 04:54:08 2018 - [info] * Phase 5: New master cleanup phase..
Sat Aug 25 04:54:08 2018 - [info]
Sat Aug 25 04:54:08 2018 - [info] Resetting slave info on the new master..
Sat Aug 25 04:54:08 2018 - [info]  192.168.2.61: Resetting slave info succeeded.
Sat Aug 25 04:54:08 2018 - [info] Master failover to 192.168.2.61(192.168.2.61:3306) completed successfully.
Sat Aug 25 04:54:08 2018 - [info]

----- Failover Report -----

app1: MySQL Master failover 192.168.2.62(192.168.2.62:3306) to 192.168.2.61(192.168.2.61:3306) succeeded

Master 192.168.2.62(192.168.2.62:3306) is down!

Check MHA Manager logs at node3:/etc/mha/app1/manager.log for details.

Started automated(non-interactive) failover.
Invalidated master IP address on 192.168.2.62(192.168.2.62:3306)
The latest slave 192.168.2.61(192.168.2.61:3306) has all relay logs for recovery.
Selected 192.168.2.61(192.168.2.61:3306) as a new master.
192.168.2.61(192.168.2.61:3306): OK: Applying all logs succeeded.
192.168.2.61(192.168.2.61:3306): OK: Activated master IP address.
192.168.2.63(192.168.2.63:3306): This host has the latest relay log events.
Generating relay diff files from the latest slave succeeded.
192.168.2.63(192.168.2.63:3306): OK: Applying all logs succeeded. Slave started, replicating from 192.168.2.61(192.168.2.61:3306)
192.168.2.61(192.168.2.61:3306): Resetting slave info succeeded.
Master failover to 192.168.2.61(192.168.2.61:3306) completed successfully.


```

### 查看从库复制信息


```sql

ysql> show slave status \G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.2.61
                  Master_User: dao
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000007
          Read_Master_Log_Pos: 154
               Relay_Log_File: node3-relay-bin.000002
                Relay_Log_Pos: 320
        Relay_Master_Log_File: mysql-bin.000007
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 154
              Relay_Log_Space: 527
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 0
               Last_SQL_Error:
  Replicate_Ignore_Server_Ids:
             Master_Server_Id: 1
                  Master_UUID: 03fde241-9a55-11e8-aa12-000c291f013a
             Master_Info_File: /mysql1/data/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind:
      Last_IO_Error_Timestamp:
     Last_SQL_Error_Timestamp:
               Master_SSL_Crl:
           Master_SSL_Crlpath:
           Retrieved_Gtid_Set:
            Executed_Gtid_Set:
                Auto_Position: 0
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
1 row in set (0.00 sec)

ERROR:
No query specified


```
### 查看新主库是否有VIP


```sql

[root@node1 ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue state UNKNOWN
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:1f:01:3a brd ff:ff:ff:ff:ff:ff
    inet 192.168.2.61/24 brd 192.168.2.255 scope global eth0
    inet 192.168.2.66/24 brd 192.168.2.255 scope global secondary eth0:0
    inet6 fe80::20c:29ff:fe1f:13a/64 scope link
       valid_lft forever preferred_lft forever


```