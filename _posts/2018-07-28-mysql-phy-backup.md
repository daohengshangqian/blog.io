---
layout: post
title: 使用innobackupex进行Mysql备份恢复-全量备份
date: 2018-07-28
categories: blog
tags: [MySQL]
description: 使用innobackupex进行Mysql备份恢复-全量备份 
---
## 综述
Xtrabackup 是Percona 公司推出的备份恢复工具.，它能对InnoDB数据库和XtraDB存储引擎的数据库非阻塞地备份（对于MyISAM的备份同样需要加表锁);


[最新版本下载地址](https://www.percona.com/downloads/XtraBackup/LATEST/)


Xtrabackup 安装成功后包含四个文件

- xtrabackup    专门用来备份InnoDB表
- innobackupex  支持同时备份innodb和myisam，但在对myisam备份时需要加一个全局的读锁 
- xbcrypt       备份文件加密解密工具    
- xbstream      流式输出工具

## 备份过程

- 开始记录xtrbackup_log
- 复制共享表空间文件与其他数据文件
- 给myisam表加锁
- 复制frm,myd,myi,等文件
- 获取binlog postition
- 解锁表
- 停止并复制 xtrbackup_log


## 使用innobackupex 备份MySQL数据库

```sql

[mysql@bosenrui ~]$ innobackupex --defaults-file=/mysql1/mysql1.cnf --no-timestamp  -u root -p root --port 3306 /home/mysql/mysql_backup
180626 01:31:55 innobackupex: Starting the backup operation

IMPORTANT: Please check that the backup run completes successfully.
           At the end of a successful backup run innobackupex
           prints "completed OK!".

180626 01:31:55  version_check Connecting to MySQL server with DSN 'dbi:mysql:;mysql_read_default_group=xtrabackup;port=3306;mysql_socket=/tmp/mysql1.sock' as 'root'  (using password: YES).
180626 01:31:55  version_check Connected to MySQL server
180626 01:31:55  version_check Executing a version check against the server...
180626 01:31:55  version_check Done.
180626 01:31:55 Connecting to MySQL server host: localhost, user: root, password: set, port: 3306, socket: /tmp/mysql1.sock
Using server version 5.7.19-log
innobackupex version 2.4.8 based on MySQL server 5.7.13 Linux (x86_64) (revision id: 97330f7)
xtrabackup: uses posix_fadvise().
xtrabackup: cd to /mysql1/data
xtrabackup: open files limit requested 0, set to 1024
xtrabackup: using the following InnoDB configuration:
xtrabackup:   innodb_data_home_dir = .
xtrabackup:   innodb_data_file_path = ibdata1:12M:autoextend
xtrabackup:   innodb_log_group_home_dir = ./
xtrabackup:   innodb_log_files_in_group = 2
xtrabackup:   innodb_log_file_size = 50331648
InnoDB: Number of pools: 1
180626 01:31:55 >> log scanned up to (46296959)
xtrabackup: Generating a list of tablespaces
InnoDB: Allocated tablespace ID 8 for mysql/time_zone_name, old maximum was 0
180626 01:31:55 [01] Copying ./ibdata1 to /home/mysql/mysql_backup/ibdata1
180626 01:31:56 >> log scanned up to (46296959)
180626 01:31:56 [01]        ...done
180626 01:31:56 [01] Copying ./mysql/time_zone_name.ibd to /home/mysql/mysql_backup/mysql/time_zone_name.ibd
180626 01:31:56 [01]        ...done
180626 01:31:56 [01] Copying ./mysql/engine_cost.ibd to /home/mysql/mysql_backup/mysql/engine_cost.ibd
180626 01:31:56 [01]        ...done
180626 01:31:56 [01] Copying ./mysql/help_topic.ibd to /home/mysql/mysql_backup/mysql/help_topic.ibd
180626 01:31:56 [01]        ...done
180626 01:31:56 [01] Copying ./mysql/server_cost.ibd to /home/mysql/mysql_backup/mysql/server_cost.ibd
180626 01:31:56 [01]        ...done
180626 01:31:56 [01] Copying ./mysql/help_category.ibd to /home/mysql/mysql_backup/mysql/help_category.ibd
180626 01:31:56 [01]        ...done
180626 01:31:57 [01] Copying ./mysql/innodb_index_stats.ibd to /home/mysql/mysql_backup/mysql/innodb_index_stats.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./mysql/innodb_table_stats.ibd to /home/mysql/mysql_backup/mysql/innodb_table_stats.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./mysql/time_zone_transition_type.ibd to /home/mysql/mysql_backup/mysql/time_zone_transition_type.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./mysql/slave_worker_info.ibd to /home/mysql/mysql_backup/mysql/slave_worker_info.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./mysql/time_zone_transition.ibd to /home/mysql/mysql_backup/mysql/time_zone_transition.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./mysql/time_zone_leap_second.ibd to /home/mysql/mysql_backup/mysql/time_zone_leap_second.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./mysql/servers.ibd to /home/mysql/mysql_backup/mysql/servers.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./mysql/gtid_executed.ibd to /home/mysql/mysql_backup/mysql/gtid_executed.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./mysql/time_zone.ibd to /home/mysql/mysql_backup/mysql/time_zone.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./mysql/help_relation.ibd to /home/mysql/mysql_backup/mysql/help_relation.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./mysql/plugin.ibd to /home/mysql/mysql_backup/mysql/plugin.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./mysql/slave_master_info.ibd to /home/mysql/mysql_backup/mysql/slave_master_info.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./mysql/help_keyword.ibd to /home/mysql/mysql_backup/mysql/help_keyword.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./mysql/slave_relay_log_info.ibd to /home/mysql/mysql_backup/mysql/slave_relay_log_info.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./dao/emp.ibd to /home/mysql/mysql_backup/dao/emp.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./dao/matches.ibd to /home/mysql/mysql_backup/dao/matches.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./dao/players.ibd to /home/mysql/mysql_backup/dao/players.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./dao/penalties.ibd to /home/mysql/mysql_backup/dao/penalties.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./dao/committee_members.ibd to /home/mysql/mysql_backup/dao/committee_members.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./dao/teams.ibd to /home/mysql/mysql_backup/dao/teams.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./dao/t1.ibd to /home/mysql/mysql_backup/dao/t1.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./dao/dept.ibd to /home/mysql/mysql_backup/dao/dept.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/lepus_status.ibd to /home/mysql/mysql_backup/lepus/lepus_status.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/alarm_temp.ibd to /home/mysql/mysql_backup/lepus/alarm_temp.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/sqlserver_status_history.ibd to /home/mysql/mysql_backup/lepus/sqlserver_status_history.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/mysql_connected.ibd to /home/mysql/mysql_backup/lepus/mysql_connected.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/mysql_slow_query_review_history.ibd to /home/mysql/mysql_backup/lepus/mysql_slow_query_review_history.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/redis_replication.ibd to /home/mysql/mysql_backup/lepus/redis_replication.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/admin_menu.ibd to /home/mysql/mysql_backup/lepus/admin_menu.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/admin_privilege.ibd to /home/mysql/mysql_backup/lepus/admin_privilege.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/os_net.ibd to /home/mysql/mysql_backup/lepus/os_net.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/admin_user.ibd to /home/mysql/mysql_backup/lepus/admin_user.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/oracle_tablespace_history.ibd to /home/mysql/mysql_backup/lepus/oracle_tablespace_history.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/admin_role_privilege.ibd to /home/mysql/mysql_backup/lepus/admin_role_privilege.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/mysql_slow_query_review.ibd to /home/mysql/mysql_backup/lepus/mysql_slow_query_review.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/os_diskio.ibd to /home/mysql/mysql_backup/lepus/os_diskio.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/admin_user1.ibd to /home/mysql/mysql_backup/lepus/admin_user1.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/redis_status.ibd to /home/mysql/mysql_backup/lepus/redis_status.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/mongodb_status_history.ibd to /home/mysql/mysql_backup/lepus/mongodb_status_history.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/sqlserver_status.ibd to /home/mysql/mysql_backup/lepus/sqlserver_status.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/admin_log.ibd to /home/mysql/mysql_backup/lepus/admin_log.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/mysql_bigtable_history.ibd to /home/mysql/mysql_backup/lepus/mysql_bigtable_history.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/os_disk_history.ibd to /home/mysql/mysql_backup/lepus/os_disk_history.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/os_status.ibd to /home/mysql/mysql_backup/lepus/os_status.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/options.ibd to /home/mysql/mysql_backup/lepus/options.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/mysql_bigtable.ibd to /home/mysql/mysql_backup/lepus/mysql_bigtable.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/alarm.ibd to /home/mysql/mysql_backup/lepus/alarm.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/os_net_history.ibd to /home/mysql/mysql_backup/lepus/os_net_history.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/db_servers_mysql.ibd to /home/mysql/mysql_backup/lepus/db_servers_mysql.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/os_status_history.ibd to /home/mysql/mysql_backup/lepus/os_status_history.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/redis_replication_history.ibd to /home/mysql/mysql_backup/lepus/redis_replication_history.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/os_disk.ibd to /home/mysql/mysql_backup/lepus/os_disk.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/oracle_status_history.ibd to /home/mysql/mysql_backup/lepus/oracle_status_history.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/mysql_processlist.ibd to /home/mysql/mysql_backup/lepus/mysql_processlist.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/db_servers_os.ibd to /home/mysql/mysql_backup/lepus/db_servers_os.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/mysql_slow_query_sendmail_log.ibd to /home/mysql/mysql_backup/lepus/mysql_slow_query_sendmail_log.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/alarm_history.ibd to /home/mysql/mysql_backup/lepus/alarm_history.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/mongodb_status.ibd to /home/mysql/mysql_backup/lepus/mongodb_status.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/oracle_status.ibd to /home/mysql/mysql_backup/lepus/oracle_status.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/db_servers_redis.ibd to /home/mysql/mysql_backup/lepus/db_servers_redis.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/admin_role.ibd to /home/mysql/mysql_backup/lepus/admin_role.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/mysql_replication_history.ibd to /home/mysql/mysql_backup/lepus/mysql_replication_history.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 >> log scanned up to (46296959)
180626 01:31:57 [01] Copying ./lepus/db_servers_oracle.ibd to /home/mysql/mysql_backup/lepus/db_servers_oracle.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/admin_user_role.ibd to /home/mysql/mysql_backup/lepus/admin_user_role.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/mysql_replication.ibd to /home/mysql/mysql_backup/lepus/mysql_replication.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/oracle_tablespace.ibd to /home/mysql/mysql_backup/lepus/oracle_tablespace.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/mysql_status.ibd to /home/mysql/mysql_backup/lepus/mysql_status.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/db_status.ibd to /home/mysql/mysql_backup/lepus/db_status.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/os_diskio_history.ibd to /home/mysql/mysql_backup/lepus/os_diskio_history.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/db_servers_sqlserver.ibd to /home/mysql/mysql_backup/lepus/db_servers_sqlserver.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/mysql_status_history.ibd to /home/mysql/mysql_backup/lepus/mysql_status_history.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/db_servers_mongodb.ibd to /home/mysql/mysql_backup/lepus/db_servers_mongodb.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./lepus/redis_status_history.ibd to /home/mysql/mysql_backup/lepus/redis_status_history.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 [01] Copying ./sys/sys_config.ibd to /home/mysql/mysql_backup/sys/sys_config.ibd
180626 01:31:57 [01]        ...done
180626 01:31:57 Executing FLUSH NO_WRITE_TO_BINLOG TABLES...
180626 01:31:58 Executing FLUSH TABLES WITH READ LOCK...
180626 01:31:58 Starting to backup non-InnoDB tables and files
180626 01:31:58 [01] Copying ./performance_schema/replication_applier_status.frm to /home/mysql/mysql_backup/performance_schema/replication_applier_status.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/file_instances.frm to /home/mysql/mysql_backup/performance_schema/file_instances.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/metadata_locks.frm to /home/mysql/mysql_backup/performance_schema/metadata_locks.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/memory_summary_by_account_by_event_name.frm to /home/mysql/mysql_backup/performance_schema/memory_summary_by_account_by_event_name.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/threads.frm to /home/mysql/mysql_backup/performance_schema/threads.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/events_waits_summary_by_user_by_event_name.frm to /home/mysql/mysql_backup/performance_schema/events_waits_summary_by_user_by_event_name.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/events_stages_summary_by_account_by_event_name.frm to /home/mysql/mysql_backup/performance_schema/events_stages_summary_by_account_by_event_name.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/db.opt to /home/mysql/mysql_backup/performance_schema/db.opt
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/table_io_waits_summary_by_table.frm to /home/mysql/mysql_backup/performance_schema/table_io_waits_summary_by_table.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/memory_summary_global_by_event_name.frm to /home/mysql/mysql_backup/performance_schema/memory_summary_global_by_event_name.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/events_stages_history_long.frm to /home/mysql/mysql_backup/performance_schema/events_stages_history_long.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/events_statements_summary_by_program.frm to /home/mysql/mysql_backup/performance_schema/events_statements_summary_by_program.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/events_statements_summary_by_user_by_event_name.frm to /home/mysql/mysql_backup/performance_schema/events_statements_summary_by_user_by_event_name.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/session_account_connect_attrs.frm to /home/mysql/mysql_backup/performance_schema/session_account_connect_attrs.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/session_connect_attrs.frm to /home/mysql/mysql_backup/performance_schema/session_connect_attrs.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/events_statements_summary_by_account_by_event_name.frm to /home/mysql/mysql_backup/performance_schema/events_statements_summary_by_account_by_event_name.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/events_statements_summary_by_thread_by_event_name.frm to /home/mysql/mysql_backup/performance_schema/events_statements_summary_by_thread_by_event_name.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/events_transactions_history.frm to /home/mysql/mysql_backup/performance_schema/events_transactions_history.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/memory_summary_by_user_by_event_name.frm to /home/mysql/mysql_backup/performance_schema/memory_summary_by_user_by_event_name.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/events_statements_summary_global_by_event_name.frm to /home/mysql/mysql_backup/performance_schema/events_statements_summary_global_by_event_name.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/performance_timers.frm to /home/mysql/mysql_backup/performance_schema/performance_timers.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/setup_consumers.frm to /home/mysql/mysql_backup/performance_schema/setup_consumers.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/events_stages_summary_by_host_by_event_name.frm to /home/mysql/mysql_backup/performance_schema/events_stages_summary_by_host_by_event_name.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/events_transactions_history_long.frm to /home/mysql/mysql_backup/performance_schema/events_transactions_history_long.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/replication_connection_status.frm to /home/mysql/mysql_backup/performance_schema/replication_connection_status.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/replication_group_members.frm to /home/mysql/mysql_backup/performance_schema/replication_group_members.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/objects_summary_global_by_type.frm to /home/mysql/mysql_backup/performance_schema/objects_summary_global_by_type.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/rwlock_instances.frm to /home/mysql/mysql_backup/performance_schema/rwlock_instances.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/user_variables_by_thread.frm to /home/mysql/mysql_backup/performance_schema/user_variables_by_thread.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/events_waits_current.frm to /home/mysql/mysql_backup/performance_schema/events_waits_current.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/file_summary_by_event_name.frm to /home/mysql/mysql_backup/performance_schema/file_summary_by_event_name.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/users.frm to /home/mysql/mysql_backup/performance_schema/users.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/replication_applier_status_by_worker.frm to /home/mysql/mysql_backup/performance_schema/replication_applier_status_by_worker.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/socket_instances.frm to /home/mysql/mysql_backup/performance_schema/socket_instances.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/events_stages_current.frm to /home/mysql/mysql_backup/performance_schema/events_stages_current.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/global_status.frm to /home/mysql/mysql_backup/performance_schema/global_status.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/setup_timers.frm to /home/mysql/mysql_backup/performance_schema/setup_timers.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/events_waits_history.frm to /home/mysql/mysql_backup/performance_schema/events_waits_history.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/table_handles.frm to /home/mysql/mysql_backup/performance_schema/table_handles.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/events_statements_current.frm to /home/mysql/mysql_backup/performance_schema/events_statements_current.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/events_transactions_current.frm to /home/mysql/mysql_backup/performance_schema/events_transactions_current.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/events_waits_summary_global_by_event_name.frm to /home/mysql/mysql_backup/performance_schema/events_waits_summary_global_by_event_name.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/replication_applier_status_by_coordinator.frm to /home/mysql/mysql_backup/performance_schema/replication_applier_status_by_coordinator.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/events_statements_summary_by_host_by_event_name.frm to /home/mysql/mysql_backup/performance_schema/events_statements_summary_by_host_by_event_name.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/events_stages_history.frm to /home/mysql/mysql_backup/performance_schema/events_stages_history.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/events_statements_summary_by_digest.frm to /home/mysql/mysql_backup/performance_schema/events_statements_summary_by_digest.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/accounts.frm to /home/mysql/mysql_backup/performance_schema/accounts.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/replication_applier_configuration.frm to /home/mysql/mysql_backup/performance_schema/replication_applier_configuration.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/replication_connection_configuration.frm to /home/mysql/mysql_backup/performance_schema/replication_connection_configuration.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/status_by_user.frm to /home/mysql/mysql_backup/performance_schema/status_by_user.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/events_transactions_summary_global_by_event_name.frm to /home/mysql/mysql_backup/performance_schema/events_transactions_summary_global_by_event_name.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/events_transactions_summary_by_account_by_event_name.frm to /home/mysql/mysql_backup/performance_schema/events_transactions_summary_by_account_by_event_name.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/events_stages_summary_global_by_event_name.frm to /home/mysql/mysql_backup/performance_schema/events_stages_summary_global_by_event_name.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/socket_summary_by_event_name.frm to /home/mysql/mysql_backup/performance_schema/socket_summary_by_event_name.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/memory_summary_by_thread_by_event_name.frm to /home/mysql/mysql_backup/performance_schema/memory_summary_by_thread_by_event_name.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/status_by_thread.frm to /home/mysql/mysql_backup/performance_schema/status_by_thread.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/events_waits_summary_by_instance.frm to /home/mysql/mysql_backup/performance_schema/events_waits_summary_by_instance.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/events_transactions_summary_by_thread_by_event_name.frm to /home/mysql/mysql_backup/performance_schema/events_transactions_summary_by_thread_by_event_name.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/hosts.frm to /home/mysql/mysql_backup/performance_schema/hosts.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/cond_instances.frm to /home/mysql/mysql_backup/performance_schema/cond_instances.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/memory_summary_by_host_by_event_name.frm to /home/mysql/mysql_backup/performance_schema/memory_summary_by_host_by_event_name.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/status_by_account.frm to /home/mysql/mysql_backup/performance_schema/status_by_account.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/socket_summary_by_instance.frm to /home/mysql/mysql_backup/performance_schema/socket_summary_by_instance.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/events_transactions_summary_by_user_by_event_name.frm to /home/mysql/mysql_backup/performance_schema/events_transactions_summary_by_user_by_event_name.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/status_by_host.frm to /home/mysql/mysql_backup/performance_schema/status_by_host.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/events_stages_summary_by_user_by_event_name.frm to /home/mysql/mysql_backup/performance_schema/events_stages_summary_by_user_by_event_name.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/global_variables.frm to /home/mysql/mysql_backup/performance_schema/global_variables.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/setup_instruments.frm to /home/mysql/mysql_backup/performance_schema/setup_instruments.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/events_waits_summary_by_thread_by_event_name.frm to /home/mysql/mysql_backup/performance_schema/events_waits_summary_by_thread_by_event_name.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/replication_group_member_stats.frm to /home/mysql/mysql_backup/performance_schema/replication_group_member_stats.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/host_cache.frm to /home/mysql/mysql_backup/performance_schema/host_cache.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/setup_objects.frm to /home/mysql/mysql_backup/performance_schema/setup_objects.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/events_waits_history_long.frm to /home/mysql/mysql_backup/performance_schema/events_waits_history_long.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/session_status.frm to /home/mysql/mysql_backup/performance_schema/session_status.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/events_transactions_summary_by_host_by_event_name.frm to /home/mysql/mysql_backup/performance_schema/events_transactions_summary_by_host_by_event_name.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/prepared_statements_instances.frm to /home/mysql/mysql_backup/performance_schema/prepared_statements_instances.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/setup_actors.frm to /home/mysql/mysql_backup/performance_schema/setup_actors.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/mutex_instances.frm to /home/mysql/mysql_backup/performance_schema/mutex_instances.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/table_lock_waits_summary_by_table.frm to /home/mysql/mysql_backup/performance_schema/table_lock_waits_summary_by_table.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/variables_by_thread.frm to /home/mysql/mysql_backup/performance_schema/variables_by_thread.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/events_waits_summary_by_host_by_event_name.frm to /home/mysql/mysql_backup/performance_schema/events_waits_summary_by_host_by_event_name.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/events_statements_history_long.frm to /home/mysql/mysql_backup/performance_schema/events_statements_history_long.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/events_waits_summary_by_account_by_event_name.frm to /home/mysql/mysql_backup/performance_schema/events_waits_summary_by_account_by_event_name.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/session_variables.frm to /home/mysql/mysql_backup/performance_schema/session_variables.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/events_stages_summary_by_thread_by_event_name.frm to /home/mysql/mysql_backup/performance_schema/events_stages_summary_by_thread_by_event_name.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/file_summary_by_instance.frm to /home/mysql/mysql_backup/performance_schema/file_summary_by_instance.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./performance_schema/events_statements_history.frm to /home/mysql/mysql_backup/performance_schema/events_statements_history.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 >> log scanned up to (46296959)
180626 01:31:58 [01] Copying ./performance_schema/table_io_waits_summary_by_index_usage.frm to /home/mysql/mysql_backup/performance_schema/table_io_waits_summary_by_index_usage.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./mysql/user.frm to /home/mysql/mysql_backup/mysql/user.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./mysql/func.MYI to /home/mysql/mysql_backup/mysql/func.MYI
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./mysql/help_topic.frm to /home/mysql/mysql_backup/mysql/help_topic.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./mysql/proxies_priv.MYI to /home/mysql/mysql_backup/mysql/proxies_priv.MYI
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./mysql/help_keyword.frm to /home/mysql/mysql_backup/mysql/help_keyword.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./mysql/func.frm to /home/mysql/mysql_backup/mysql/func.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./mysql/ndb_binlog_index.MYD to /home/mysql/mysql_backup/mysql/ndb_binlog_index.MYD
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./mysql/tables_priv.MYI to /home/mysql/mysql_backup/mysql/tables_priv.MYI
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./mysql/db.opt to /home/mysql/mysql_backup/mysql/db.opt
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./mysql/db.MYD to /home/mysql/mysql_backup/mysql/db.MYD
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./mysql/engine_cost.frm to /home/mysql/mysql_backup/mysql/engine_cost.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./mysql/ndb_binlog_index.MYI to /home/mysql/mysql_backup/mysql/ndb_binlog_index.MYI
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./mysql/procs_priv.MYI to /home/mysql/mysql_backup/mysql/procs_priv.MYI
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./mysql/help_category.frm to /home/mysql/mysql_backup/mysql/help_category.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./mysql/ndb_binlog_index.frm to /home/mysql/mysql_backup/mysql/ndb_binlog_index.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./mysql/user.MYI to /home/mysql/mysql_backup/mysql/user.MYI
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./mysql/event.MYI to /home/mysql/mysql_backup/mysql/event.MYI
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./mysql/tables_priv.frm to /home/mysql/mysql_backup/mysql/tables_priv.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./mysql/slave_master_info.frm to /home/mysql/mysql_backup/mysql/slave_master_info.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./mysql/time_zone_transition_type.frm to /home/mysql/mysql_backup/mysql/time_zone_transition_type.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./mysql/columns_priv.MYD to /home/mysql/mysql_backup/mysql/columns_priv.MYD
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./mysql/tables_priv.MYD to /home/mysql/mysql_backup/mysql/tables_priv.MYD
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./mysql/proxies_priv.frm to /home/mysql/mysql_backup/mysql/proxies_priv.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./mysql/event.frm to /home/mysql/mysql_backup/mysql/event.frm
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./mysql/slow_log.CSV to /home/mysql/mysql_backup/mysql/slow_log.CSV
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./mysql/proc.MYI to /home/mysql/mysql_backup/mysql/proc.MYI
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./mysql/func.MYD to /home/mysql/mysql_backup/mysql/func.MYD
180626 01:31:58 [01]        ...done
180626 01:31:58 [01] Copying ./mysql/proc.frm to /home/mysql/mysql_backup/mysql/proc.frm
180626 01:31:58 [01]        ...done
180626 01:31:59 [01] Copying ./mysql/slave_relay_log_info.frm to /home/mysql/mysql_backup/mysql/slave_relay_log_info.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./mysql/time_zone.frm to /home/mysql/mysql_backup/mysql/time_zone.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./mysql/db.MYI to /home/mysql/mysql_backup/mysql/db.MYI
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./mysql/gtid_executed.frm to /home/mysql/mysql_backup/mysql/gtid_executed.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./mysql/general_log.CSM to /home/mysql/mysql_backup/mysql/general_log.CSM
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./mysql/db.frm to /home/mysql/mysql_backup/mysql/db.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./mysql/plugin.frm to /home/mysql/mysql_backup/mysql/plugin.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./mysql/time_zone_leap_second.frm to /home/mysql/mysql_backup/mysql/time_zone_leap_second.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./mysql/general_log.frm to /home/mysql/mysql_backup/mysql/general_log.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./mysql/server_cost.frm to /home/mysql/mysql_backup/mysql/server_cost.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./mysql/servers.frm to /home/mysql/mysql_backup/mysql/servers.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./mysql/innodb_table_stats.frm to /home/mysql/mysql_backup/mysql/innodb_table_stats.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./mysql/columns_priv.frm to /home/mysql/mysql_backup/mysql/columns_priv.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./mysql/user.MYD to /home/mysql/mysql_backup/mysql/user.MYD
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./mysql/slave_worker_info.frm to /home/mysql/mysql_backup/mysql/slave_worker_info.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./mysql/time_zone_transition.frm to /home/mysql/mysql_backup/mysql/time_zone_transition.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./mysql/time_zone_name.frm to /home/mysql/mysql_backup/mysql/time_zone_name.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./mysql/slow_log.frm to /home/mysql/mysql_backup/mysql/slow_log.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./mysql/help_relation.frm to /home/mysql/mysql_backup/mysql/help_relation.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./mysql/proxies_priv.MYD to /home/mysql/mysql_backup/mysql/proxies_priv.MYD
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./mysql/event.MYD to /home/mysql/mysql_backup/mysql/event.MYD
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./mysql/slow_log.CSM to /home/mysql/mysql_backup/mysql/slow_log.CSM
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./mysql/innodb_index_stats.frm to /home/mysql/mysql_backup/mysql/innodb_index_stats.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./mysql/general_log.CSV to /home/mysql/mysql_backup/mysql/general_log.CSV
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./mysql/proc.MYD to /home/mysql/mysql_backup/mysql/proc.MYD
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./mysql/procs_priv.frm to /home/mysql/mysql_backup/mysql/procs_priv.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./mysql/procs_priv.MYD to /home/mysql/mysql_backup/mysql/procs_priv.MYD
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./mysql/columns_priv.MYI to /home/mysql/mysql_backup/mysql/columns_priv.MYI
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./dao/db.opt to /home/mysql/mysql_backup/dao/db.opt
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./dao/penalties.frm to /home/mysql/mysql_backup/dao/penalties.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./dao/dept.frm to /home/mysql/mysql_backup/dao/dept.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./dao/matches.frm to /home/mysql/mysql_backup/dao/matches.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./dao/teams.frm to /home/mysql/mysql_backup/dao/teams.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./dao/committee_members.frm to /home/mysql/mysql_backup/dao/committee_members.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./dao/t1.frm to /home/mysql/mysql_backup/dao/t1.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./dao/players.frm to /home/mysql/mysql_backup/dao/players.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./dao/emp.frm to /home/mysql/mysql_backup/dao/emp.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/oracle_status.frm to /home/mysql/mysql_backup/lepus/oracle_status.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/redis_replication.frm to /home/mysql/mysql_backup/lepus/redis_replication.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/alarm_temp.frm to /home/mysql/mysql_backup/lepus/alarm_temp.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/db.opt to /home/mysql/mysql_backup/lepus/db.opt
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/db_servers_os.frm to /home/mysql/mysql_backup/lepus/db_servers_os.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/os_net.frm to /home/mysql/mysql_backup/lepus/os_net.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/os_diskio.frm to /home/mysql/mysql_backup/lepus/os_diskio.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/admin_role.frm to /home/mysql/mysql_backup/lepus/admin_role.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/options.frm to /home/mysql/mysql_backup/lepus/options.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/mysql_connected.frm to /home/mysql/mysql_backup/lepus/mysql_connected.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/mysql_status.frm to /home/mysql/mysql_backup/lepus/mysql_status.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/redis_status.frm to /home/mysql/mysql_backup/lepus/redis_status.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/mysql_slow_query_review_history.frm to /home/mysql/mysql_backup/lepus/mysql_slow_query_review_history.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/sqlserver_status.frm to /home/mysql/mysql_backup/lepus/sqlserver_status.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/os_disk_history.frm to /home/mysql/mysql_backup/lepus/os_disk_history.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/db_status.frm to /home/mysql/mysql_backup/lepus/db_status.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/db_servers_oracle.frm to /home/mysql/mysql_backup/lepus/db_servers_oracle.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/admin_role_privilege.frm to /home/mysql/mysql_backup/lepus/admin_role_privilege.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/db_servers_mysql.frm to /home/mysql/mysql_backup/lepus/db_servers_mysql.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/redis_status_history.frm to /home/mysql/mysql_backup/lepus/redis_status_history.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/db_servers_mongodb.frm to /home/mysql/mysql_backup/lepus/db_servers_mongodb.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/oracle_tablespace.frm to /home/mysql/mysql_backup/lepus/oracle_tablespace.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/admin_menu.frm to /home/mysql/mysql_backup/lepus/admin_menu.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/mongodb_status.frm to /home/mysql/mysql_backup/lepus/mongodb_status.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/os_net_history.frm to /home/mysql/mysql_backup/lepus/os_net_history.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/mysql_replication_history.frm to /home/mysql/mysql_backup/lepus/mysql_replication_history.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/mongodb_status_history.frm to /home/mysql/mysql_backup/lepus/mongodb_status_history.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/admin_user.frm to /home/mysql/mysql_backup/lepus/admin_user.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/oracle_status_history.frm to /home/mysql/mysql_backup/lepus/oracle_status_history.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/oracle_tablespace_history.frm to /home/mysql/mysql_backup/lepus/oracle_tablespace_history.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/sqlserver_status_history.frm to /home/mysql/mysql_backup/lepus/sqlserver_status_history.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/mysql_bigtable_history.frm to /home/mysql/mysql_backup/lepus/mysql_bigtable_history.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/mysql_processlist.frm to /home/mysql/mysql_backup/lepus/mysql_processlist.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/lepus_status.frm to /home/mysql/mysql_backup/lepus/lepus_status.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/db_servers_sqlserver.frm to /home/mysql/mysql_backup/lepus/db_servers_sqlserver.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/mysql_bigtable.frm to /home/mysql/mysql_backup/lepus/mysql_bigtable.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/os_diskio_history.frm to /home/mysql/mysql_backup/lepus/os_diskio_history.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/mysql_replication.frm to /home/mysql/mysql_backup/lepus/mysql_replication.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/mysql_slow_query_review.frm to /home/mysql/mysql_backup/lepus/mysql_slow_query_review.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/alarm_history.frm to /home/mysql/mysql_backup/lepus/alarm_history.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/redis_replication_history.frm to /home/mysql/mysql_backup/lepus/redis_replication_history.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/os_disk.frm to /home/mysql/mysql_backup/lepus/os_disk.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/mysql_slow_query_sendmail_log.frm to /home/mysql/mysql_backup/lepus/mysql_slow_query_sendmail_log.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/admin_log.frm to /home/mysql/mysql_backup/lepus/admin_log.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/admin_user_role.frm to /home/mysql/mysql_backup/lepus/admin_user_role.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/alarm.frm to /home/mysql/mysql_backup/lepus/alarm.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/admin_user1.frm to /home/mysql/mysql_backup/lepus/admin_user1.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/os_status_history.frm to /home/mysql/mysql_backup/lepus/os_status_history.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/db_servers_redis.frm to /home/mysql/mysql_backup/lepus/db_servers_redis.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/mysql_status_history.frm to /home/mysql/mysql_backup/lepus/mysql_status_history.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/os_status.frm to /home/mysql/mysql_backup/lepus/os_status.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./lepus/admin_privilege.frm to /home/mysql/mysql_backup/lepus/admin_privilege.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./sys/x@0024innodb_buffer_stats_by_schema.frm to /home/mysql/mysql_backup/sys/x@0024innodb_buffer_stats_by_schema.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 >> log scanned up to (46296959)
180626 01:31:59 [01] Copying ./sys/x@0024statement_analysis.frm to /home/mysql/mysql_backup/sys/x@0024statement_analysis.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./sys/x@0024user_summary.frm to /home/mysql/mysql_backup/sys/x@0024user_summary.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./sys/x@0024schema_index_statistics.frm to /home/mysql/mysql_backup/sys/x@0024schema_index_statistics.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./sys/x@0024statements_with_runtimes_in_95th_percentile.frm to /home/mysql/mysql_backup/sys/x@0024statements_with_runtimes_in_95th_percentile.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./sys/x@0024innodb_buffer_stats_by_table.frm to /home/mysql/mysql_backup/sys/x@0024innodb_buffer_stats_by_table.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./sys/db.opt to /home/mysql/mysql_backup/sys/db.opt
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./sys/host_summary_by_statement_type.frm to /home/mysql/mysql_backup/sys/host_summary_by_statement_type.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./sys/sys_config_insert_set_user.TRN to /home/mysql/mysql_backup/sys/sys_config_insert_set_user.TRN
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./sys/statement_analysis.frm to /home/mysql/mysql_backup/sys/statement_analysis.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./sys/version.frm to /home/mysql/mysql_backup/sys/version.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./sys/host_summary.frm to /home/mysql/mysql_backup/sys/host_summary.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./sys/x@0024io_global_by_wait_by_latency.frm to /home/mysql/mysql_backup/sys/x@0024io_global_by_wait_by_latency.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./sys/statements_with_sorting.frm to /home/mysql/mysql_backup/sys/statements_with_sorting.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./sys/sys_config.frm to /home/mysql/mysql_backup/sys/sys_config.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./sys/x@0024io_by_thread_by_latency.frm to /home/mysql/mysql_backup/sys/x@0024io_by_thread_by_latency.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./sys/memory_by_host_by_current_bytes.frm to /home/mysql/mysql_backup/sys/memory_by_host_by_current_bytes.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./sys/x@0024memory_by_thread_by_current_bytes.frm to /home/mysql/mysql_backup/sys/x@0024memory_by_thread_by_current_bytes.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./sys/x@0024schema_flattened_keys.frm to /home/mysql/mysql_backup/sys/x@0024schema_flattened_keys.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./sys/x@0024memory_by_host_by_current_bytes.frm to /home/mysql/mysql_backup/sys/x@0024memory_by_host_by_current_bytes.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./sys/sys_config.TRG to /home/mysql/mysql_backup/sys/sys_config.TRG
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./sys/user_summary_by_file_io.frm to /home/mysql/mysql_backup/sys/user_summary_by_file_io.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./sys/x@0024latest_file_io.frm to /home/mysql/mysql_backup/sys/x@0024latest_file_io.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./sys/schema_object_overview.frm to /home/mysql/mysql_backup/sys/schema_object_overview.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./sys/schema_unused_indexes.frm to /home/mysql/mysql_backup/sys/schema_unused_indexes.frm
180626 01:31:59 [01]        ...done
180626 01:31:59 [01] Copying ./sys/x@0024processlist.frm to /home/mysql/mysql_backup/sys/x@0024processlist.frm
180626 01:31:59 [01]        ...done
180626 01:32:00 [01] Copying ./sys/statements_with_full_table_scans.frm to /home/mysql/mysql_backup/sys/statements_with_full_table_scans.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/x@0024host_summary_by_file_io.frm to /home/mysql/mysql_backup/sys/x@0024host_summary_by_file_io.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/x@0024host_summary.frm to /home/mysql/mysql_backup/sys/x@0024host_summary.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/memory_by_user_by_current_bytes.frm to /home/mysql/mysql_backup/sys/memory_by_user_by_current_bytes.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/wait_classes_global_by_latency.frm to /home/mysql/mysql_backup/sys/wait_classes_global_by_latency.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/x@0024io_global_by_wait_by_bytes.frm to /home/mysql/mysql_backup/sys/x@0024io_global_by_wait_by_bytes.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/x@0024ps_schema_table_statistics_io.frm to /home/mysql/mysql_backup/sys/x@0024ps_schema_table_statistics_io.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/x@0024schema_table_statistics_with_buffer.frm to /home/mysql/mysql_backup/sys/x@0024schema_table_statistics_with_buffer.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/session_ssl_status.frm to /home/mysql/mysql_backup/sys/session_ssl_status.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/innodb_buffer_stats_by_schema.frm to /home/mysql/mysql_backup/sys/innodb_buffer_stats_by_schema.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/x@0024waits_by_user_by_latency.frm to /home/mysql/mysql_backup/sys/x@0024waits_by_user_by_latency.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/x@0024host_summary_by_stages.frm to /home/mysql/mysql_backup/sys/x@0024host_summary_by_stages.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/x@0024waits_global_by_latency.frm to /home/mysql/mysql_backup/sys/x@0024waits_global_by_latency.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/schema_tables_with_full_table_scans.frm to /home/mysql/mysql_backup/sys/schema_tables_with_full_table_scans.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/x@0024schema_table_lock_waits.frm to /home/mysql/mysql_backup/sys/x@0024schema_table_lock_waits.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/x@0024memory_by_user_by_current_bytes.frm to /home/mysql/mysql_backup/sys/x@0024memory_by_user_by_current_bytes.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/innodb_lock_waits.frm to /home/mysql/mysql_backup/sys/innodb_lock_waits.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/x@0024ps_digest_avg_latency_distribution.frm to /home/mysql/mysql_backup/sys/x@0024ps_digest_avg_latency_distribution.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/x@0024schema_table_statistics.frm to /home/mysql/mysql_backup/sys/x@0024schema_table_statistics.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/x@0024host_summary_by_file_io_type.frm to /home/mysql/mysql_backup/sys/x@0024host_summary_by_file_io_type.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/latest_file_io.frm to /home/mysql/mysql_backup/sys/latest_file_io.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/schema_redundant_indexes.frm to /home/mysql/mysql_backup/sys/schema_redundant_indexes.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/memory_global_by_current_bytes.frm to /home/mysql/mysql_backup/sys/memory_global_by_current_bytes.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/x@0024memory_global_by_current_bytes.frm to /home/mysql/mysql_backup/sys/x@0024memory_global_by_current_bytes.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/statements_with_errors_or_warnings.frm to /home/mysql/mysql_backup/sys/statements_with_errors_or_warnings.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/x@0024wait_classes_global_by_avg_latency.frm to /home/mysql/mysql_backup/sys/x@0024wait_classes_global_by_avg_latency.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/x@0024wait_classes_global_by_latency.frm to /home/mysql/mysql_backup/sys/x@0024wait_classes_global_by_latency.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/x@0024session.frm to /home/mysql/mysql_backup/sys/x@0024session.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/x@0024schema_tables_with_full_table_scans.frm to /home/mysql/mysql_backup/sys/x@0024schema_tables_with_full_table_scans.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/host_summary_by_file_io_type.frm to /home/mysql/mysql_backup/sys/host_summary_by_file_io_type.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/user_summary.frm to /home/mysql/mysql_backup/sys/user_summary.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/x@0024user_summary_by_stages.frm to /home/mysql/mysql_backup/sys/x@0024user_summary_by_stages.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/user_summary_by_file_io_type.frm to /home/mysql/mysql_backup/sys/user_summary_by_file_io_type.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/schema_table_statistics.frm to /home/mysql/mysql_backup/sys/schema_table_statistics.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/waits_by_user_by_latency.frm to /home/mysql/mysql_backup/sys/waits_by_user_by_latency.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/io_global_by_wait_by_bytes.frm to /home/mysql/mysql_backup/sys/io_global_by_wait_by_bytes.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/user_summary_by_statement_type.frm to /home/mysql/mysql_backup/sys/user_summary_by_statement_type.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/x@0024statements_with_sorting.frm to /home/mysql/mysql_backup/sys/x@0024statements_with_sorting.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/user_summary_by_statement_latency.frm to /home/mysql/mysql_backup/sys/user_summary_by_statement_latency.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/x@0024user_summary_by_statement_latency.frm to /home/mysql/mysql_backup/sys/x@0024user_summary_by_statement_latency.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/session.frm to /home/mysql/mysql_backup/sys/session.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/schema_table_statistics_with_buffer.frm to /home/mysql/mysql_backup/sys/schema_table_statistics_with_buffer.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/host_summary_by_file_io.frm to /home/mysql/mysql_backup/sys/host_summary_by_file_io.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/memory_by_thread_by_current_bytes.frm to /home/mysql/mysql_backup/sys/memory_by_thread_by_current_bytes.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/x@0024waits_by_host_by_latency.frm to /home/mysql/mysql_backup/sys/x@0024waits_by_host_by_latency.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/schema_auto_increment_columns.frm to /home/mysql/mysql_backup/sys/schema_auto_increment_columns.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/x@0024ps_digest_95th_percentile_by_avg_us.frm to /home/mysql/mysql_backup/sys/x@0024ps_digest_95th_percentile_by_avg_us.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/waits_by_host_by_latency.frm to /home/mysql/mysql_backup/sys/waits_by_host_by_latency.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/host_summary_by_statement_latency.frm to /home/mysql/mysql_backup/sys/host_summary_by_statement_latency.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/metrics.frm to /home/mysql/mysql_backup/sys/metrics.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/x@0024user_summary_by_file_io_type.frm to /home/mysql/mysql_backup/sys/x@0024user_summary_by_file_io_type.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/io_global_by_file_by_bytes.frm to /home/mysql/mysql_backup/sys/io_global_by_file_by_bytes.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/io_by_thread_by_latency.frm to /home/mysql/mysql_backup/sys/io_by_thread_by_latency.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/processlist.frm to /home/mysql/mysql_backup/sys/processlist.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/statements_with_runtimes_in_95th_percentile.frm to /home/mysql/mysql_backup/sys/statements_with_runtimes_in_95th_percentile.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/schema_table_lock_waits.frm to /home/mysql/mysql_backup/sys/schema_table_lock_waits.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/wait_classes_global_by_avg_latency.frm to /home/mysql/mysql_backup/sys/wait_classes_global_by_avg_latency.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/user_summary_by_stages.frm to /home/mysql/mysql_backup/sys/user_summary_by_stages.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/io_global_by_file_by_latency.frm to /home/mysql/mysql_backup/sys/io_global_by_file_by_latency.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/x@0024host_summary_by_statement_type.frm to /home/mysql/mysql_backup/sys/x@0024host_summary_by_statement_type.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/x@0024innodb_lock_waits.frm to /home/mysql/mysql_backup/sys/x@0024innodb_lock_waits.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/ps_check_lost_instrumentation.frm to /home/mysql/mysql_backup/sys/ps_check_lost_instrumentation.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/waits_global_by_latency.frm to /home/mysql/mysql_backup/sys/waits_global_by_latency.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/sys_config_update_set_user.TRN to /home/mysql/mysql_backup/sys/sys_config_update_set_user.TRN
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/innodb_buffer_stats_by_table.frm to /home/mysql/mysql_backup/sys/innodb_buffer_stats_by_table.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/x@0024statements_with_temp_tables.frm to /home/mysql/mysql_backup/sys/x@0024statements_with_temp_tables.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/io_global_by_wait_by_latency.frm to /home/mysql/mysql_backup/sys/io_global_by_wait_by_latency.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/host_summary_by_stages.frm to /home/mysql/mysql_backup/sys/host_summary_by_stages.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/statements_with_temp_tables.frm to /home/mysql/mysql_backup/sys/statements_with_temp_tables.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/x@0024user_summary_by_file_io.frm to /home/mysql/mysql_backup/sys/x@0024user_summary_by_file_io.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/schema_index_statistics.frm to /home/mysql/mysql_backup/sys/schema_index_statistics.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/x@0024host_summary_by_statement_latency.frm to /home/mysql/mysql_backup/sys/x@0024host_summary_by_statement_latency.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/x@0024user_summary_by_statement_type.frm to /home/mysql/mysql_backup/sys/x@0024user_summary_by_statement_type.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/x@0024statements_with_errors_or_warnings.frm to /home/mysql/mysql_backup/sys/x@0024statements_with_errors_or_warnings.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/x@0024io_global_by_file_by_latency.frm to /home/mysql/mysql_backup/sys/x@0024io_global_by_file_by_latency.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/x@0024io_global_by_file_by_bytes.frm to /home/mysql/mysql_backup/sys/x@0024io_global_by_file_by_bytes.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/x@0024memory_global_total.frm to /home/mysql/mysql_backup/sys/x@0024memory_global_total.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/memory_global_total.frm to /home/mysql/mysql_backup/sys/memory_global_total.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 [01] Copying ./sys/x@0024statements_with_full_table_scans.frm to /home/mysql/mysql_backup/sys/x@0024statements_with_full_table_scans.frm
180626 01:32:00 [01]        ...done
180626 01:32:00 Finished backing up non-InnoDB tables and files
180626 01:32:00 [00] Writing /home/mysql/mysql_backup/xtrabackup_binlog_info
180626 01:32:00 [00]        ...done
180626 01:32:00 Executing FLUSH NO_WRITE_TO_BINLOG ENGINE LOGS...
xtrabackup: The latest check point (for incremental): '46296950'
xtrabackup: Stopping log copying thread.
.180626 01:32:00 >> log scanned up to (46296959)

180626 01:32:00 Executing UNLOCK TABLES
180626 01:32:00 All tables unlocked
180626 01:32:00 [00] Copying ib_buffer_pool to /home/mysql/mysql_backup/ib_buffer_pool
180626 01:32:00 [00]        ...done
180626 01:32:00 Backup created in directory '/home/mysql/mysql_backup/'
MySQL binlog position: filename 'mysql-bin.000001', position '5262531'
180626 01:32:00 [00] Writing /home/mysql/mysql_backup/backup-my.cnf
180626 01:32:00 [00]        ...done
180626 01:32:00 [00] Writing /home/mysql/mysql_backup/xtrabackup_info
180626 01:32:00 [00]        ...done
xtrabackup: Transaction log of lsn (46296950) to (46296959) was copied.
180626 01:32:01 completed OK!

```


### 查看备份后的文件

```sql

[mysql@bosenrui mysql_backup]$ ll
total 196660
-rw-r----- 1 mysql mysql      424 Jun 26 01:32 backup-my.cnf
drwxr-x--- 2 mysql mysql     4096 Jun 26 01:31 dao
-rw-r----- 1 mysql mysql      969 Jun 26 01:32 ib_buffer_pool
-rw-r----- 1 mysql mysql 79691776 Jun 26 03:09 ibdata1
-rw-r----- 1 mysql mysql 50331648 Jun 26 03:09 ib_logfile0
-rw-r----- 1 mysql mysql 50331648 Jun 26 03:09 ib_logfile1
-rw-r----- 1 mysql mysql 12582912 Jun 26 03:09 ibtmp1
drwxr-x--- 2 mysql mysql     4096 Jun 26 01:31 lepus
drwxr-x--- 2 mysql mysql     4096 Jun 26 01:31 mysql
drwxr-x--- 2 mysql mysql     4096 Jun 26 01:31 performance_schema
drwxr-x--- 2 mysql mysql    12288 Jun 26 01:32 sys
-rw-r----- 1 mysql mysql       25 Jun 26 01:32 xtrabackup_binlog_info
-rw-rw-r-- 1 mysql mysql       25 Jun 26 03:09 xtrabackup_binlog_pos_innodb
-rw-r----- 1 mysql mysql      115 Jun 26 03:09 xtrabackup_checkpoints
-rw-r----- 1 mysql mysql      536 Jun 26 01:32 xtrabackup_info
-rw-r----- 1 mysql mysql  8388608 Jun 26 03:09 xtrabackup_logfile


```

- backup-my.cnf

此文件记录了备份时使用的相关参数

```sql


[mysql@bosenrui mysql_backup]$ cat backup-my.cnf
# This MySQL options file was generated by innobackupex.

# The MySQL server
[mysqld]
innodb_checksum_algorithm=crc32
innodb_log_checksum_algorithm=strict_crc32
innodb_data_file_path=ibdata1:12M:autoextend
innodb_log_files_in_group=2
innodb_log_file_size=50331648
innodb_fast_checksum=false
innodb_page_size=16384
innodb_log_block_size=512
innodb_undo_directory=./
innodb_undo_tablespaces=0
server_id=1

redo_log_version=1


```

- xtrabackup_info

记录了备份相关的信息 比如何时备份 

```sql

[mysql@bosenrui mysql_backup]$ cat xtrabackup_info
uuid = ab075a3b-789d-11e8-b549-000c2916156f
name =
tool_name = innobackupex
tool_command = --defaults-file=/mysql1/mysql1.cnf --no-timestamp -u root -p root --port 3306 /home/mysql/mysql_backup
tool_version = 2.4.8
ibbackup_version = 2.4.8
server_version = 5.7.19-log
start_time = 2018-06-26 01:31:55
end_time = 2018-06-26 01:32:00
lock_time = 0
binlog_pos = filename 'mysql-bin.000001', position '5262531'
innodb_from_lsn = 0
innodb_to_lsn = 46296950
partial = N
incremental = N
format = file
compact = N
compressed = N
encrypted = N

```

- xtrabackup_binlog_pos_innodb 

备份完成时innodb 的lsn
```sql
[mysql@bosenrui mysql_backup]$ cat xtrabackup_binlog_pos_innodb
mysql-bin.000001        5262531



```

- xtrabackup_binlog_info
 
备份完成时整个mysql server  的lsn
如果数据库只使用innodb引擎则两个文件内容应相同.反之,则xtrabackup_binlog_info中lsn应大于xtrabackup_binlog_pos_innodb

```sql
[mysql@bosenrui mysql_backup]$ cat xtrabackup_binlog_info
mysql-bin.000001        5262531

```


##  恢复

###  prepare 

```sql



[mysql@bosenrui mysql1]$ innobackupex --defaults-file=/mysql1/mysql1.cnf --user root --port 3306 --apply-log /home/mysql/mysql_backup
180626 03:09:36 innobackupex: Starting the apply-log operation

IMPORTANT: Please check that the apply-log run completes successfully.
           At the end of a successful apply-log run innobackupex
           prints "completed OK!".

innobackupex version 2.4.8 based on MySQL server 5.7.13 Linux (x86_64) (revision id: 97330f7)
xtrabackup: cd to /home/mysql/mysql_backup/
xtrabackup: This target seems to be not prepared yet.
InnoDB: Number of pools: 1
xtrabackup: xtrabackup_logfile detected: size=8388608, start_lsn=(46296950)
xtrabackup: using the following InnoDB configuration for recovery:
xtrabackup:   innodb_data_home_dir = .
xtrabackup:   innodb_data_file_path = ibdata1:12M:autoextend
xtrabackup:   innodb_log_group_home_dir = .
xtrabackup:   innodb_log_files_in_group = 1
xtrabackup:   innodb_log_file_size = 8388608
xtrabackup: using the following InnoDB configuration for recovery:
xtrabackup:   innodb_data_home_dir = .
xtrabackup:   innodb_data_file_path = ibdata1:12M:autoextend
xtrabackup:   innodb_log_group_home_dir = .
xtrabackup:   innodb_log_files_in_group = 1
xtrabackup:   innodb_log_file_size = 8388608
xtrabackup: Starting InnoDB instance for recovery.
xtrabackup: Using 104857600 bytes for buffer pool (set by --use-memory parameter)
InnoDB: PUNCH HOLE support available
InnoDB: Mutexes and rw_locks use GCC atomic builtins
InnoDB: Uses event mutexes
InnoDB: GCC builtin __sync_synchronize() is used for memory barrier
InnoDB: Compressed tables use zlib 1.2.3
InnoDB: Number of pools: 1
InnoDB: Using CPU crc32 instructions
InnoDB: Initializing buffer pool, total size = 100M, instances = 1, chunk size = 100M
InnoDB: Completed initialization of buffer pool
InnoDB: If the mysqld execution user is authorized, page cleaner thread priority can be changed. See the man page of setpriority().
InnoDB: Highest supported file format is Barracuda.
InnoDB: Log scan progressed past the checkpoint lsn 46296950
InnoDB: Doing recovery: scanned up to log sequence number 46296959 (0%)
InnoDB: Doing recovery: scanned up to log sequence number 46296959 (0%)
InnoDB: Database was not shutdown normally!
InnoDB: Starting crash recovery.
InnoDB: xtrabackup: Last MySQL binlog file position 5262531, file name mysql-bin.000001
InnoDB: Creating shared tablespace for temporary tables
InnoDB: Setting file './ibtmp1' size to 12 MB. Physically writing the file full; Please wait ...
InnoDB: File './ibtmp1' size is now 12 MB.
InnoDB: 96 redo rollback segment(s) found. 1 redo rollback segment(s) are active.
InnoDB: 32 non-redo rollback segment(s) are active.
InnoDB: Waiting for purge to start
InnoDB: 5.7.13 started; log sequence number 46296959
InnoDB: xtrabackup: Last MySQL binlog file position 5262531, file name mysql-bin.000001

xtrabackup: starting shutdown with innodb_fast_shutdown = 1
InnoDB: FTS optimize thread exiting.
InnoDB: Starting shutdown...
InnoDB: Shutdown completed; log sequence number 46297106
InnoDB: Number of pools: 1
xtrabackup: using the following InnoDB configuration for recovery:
xtrabackup:   innodb_data_home_dir = .
xtrabackup:   innodb_data_file_path = ibdata1:12M:autoextend
xtrabackup:   innodb_log_group_home_dir = .
xtrabackup:   innodb_log_files_in_group = 2
xtrabackup:   innodb_log_file_size = 50331648
InnoDB: PUNCH HOLE support available
InnoDB: Mutexes and rw_locks use GCC atomic builtins
InnoDB: Uses event mutexes
InnoDB: GCC builtin __sync_synchronize() is used for memory barrier
InnoDB: Compressed tables use zlib 1.2.3
InnoDB: Number of pools: 1
InnoDB: Using CPU crc32 instructions
InnoDB: Initializing buffer pool, total size = 100M, instances = 1, chunk size = 100M
InnoDB: Completed initialization of buffer pool
InnoDB: If the mysqld execution user is authorized, page cleaner thread priority can be changed. See the man page of setpriority().
InnoDB: Setting log file ./ib_logfile101 size to 48 MB
InnoDB: Setting log file ./ib_logfile1 size to 48 MB
InnoDB: Renaming log file ./ib_logfile101 to ./ib_logfile0
InnoDB: New log files created, LSN=46297106
InnoDB: Highest supported file format is Barracuda.
InnoDB: Log scan progressed past the checkpoint lsn 46297612
InnoDB: Doing recovery: scanned up to log sequence number 46297621 (0%)
InnoDB: Doing recovery: scanned up to log sequence number 46297621 (0%)
InnoDB: Database was not shutdown normally!
InnoDB: Starting crash recovery.
InnoDB: xtrabackup: Last MySQL binlog file position 5262531, file name mysql-bin.000001
InnoDB: Removed temporary tablespace data file: "ibtmp1"
InnoDB: Creating shared tablespace for temporary tables
InnoDB: Setting file './ibtmp1' size to 12 MB. Physically writing the file full; Please wait ...
InnoDB: File './ibtmp1' size is now 12 MB.
InnoDB: 96 redo rollback segment(s) found. 1 redo rollback segment(s) are active.
InnoDB: 32 non-redo rollback segment(s) are active.
InnoDB: Waiting for purge to start
InnoDB: 5.7.13 started; log sequence number 46297621
xtrabackup: starting shutdown with innodb_fast_shutdown = 1
InnoDB: FTS optimize thread exiting.
InnoDB: Starting shutdown...
InnoDB: Shutdown completed; log sequence number 46297640
180626 03:09:40 completed OK!

```



### 恢复备份文件到datadir

```sql

[mysql@bosenrui mysql1]$ innobackupex --defaults-file=/mysql1/mysql1.cnf --copy-back /home/mysql/mysql_backup
180626 03:15:06 innobackupex: Starting the copy-back operation

IMPORTANT: Please check that the copy-back run completes successfully.
           At the end of a successful copy-back run innobackupex
           prints "completed OK!".

innobackupex version 2.4.8 based on MySQL server 5.7.13 Linux (x86_64) (revision id: 97330f7)
180626 03:15:06 [01] Copying ib_logfile0 to /mysql1/data/ib_logfile0
180626 03:15:06 [01]        ...done
180626 03:15:06 [01] Copying ib_logfile1 to /mysql1/data/ib_logfile1
180626 03:15:07 [01]        ...done
180626 03:15:07 [01] Copying ibdata1 to /mysql1/data/ibdata1
180626 03:15:07 [01]        ...done
180626 03:15:07 [01] Copying ./performance_schema/replication_applier_status.frm to /mysql1/data/performance_schema/replication_applier_status.frm
180626 03:15:07 [01]        ...done
180626 03:15:07 [01] Copying ./performance_schema/file_instances.frm to /mysql1/data/performance_schema/file_instances.frm
180626 03:15:07 [01]        ...done
180626 03:15:07 [01] Copying ./performance_schema/metadata_locks.frm to /mysql1/data/performance_schema/metadata_locks.frm
180626 03:15:07 [01]        ...done
180626 03:15:07 [01] Copying ./performance_schema/memory_summary_by_account_by_event_name.frm to /mysql1/data/performance_schema/memory_summary_by_account_by_event_name.frm
180626 03:15:07 [01]        ...done
180626 03:15:07 [01] Copying ./performance_schema/threads.frm to /mysql1/data/performance_schema/threads.frm
180626 03:15:07 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/events_waits_summary_by_user_by_event_name.frm to /mysql1/data/performance_schema/events_waits_summary_by_user_by_event_name.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/events_stages_summary_by_account_by_event_name.frm to /mysql1/data/performance_schema/events_stages_summary_by_account_by_event_name.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/db.opt to /mysql1/data/performance_schema/db.opt
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/table_io_waits_summary_by_table.frm to /mysql1/data/performance_schema/table_io_waits_summary_by_table.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/memory_summary_global_by_event_name.frm to /mysql1/data/performance_schema/memory_summary_global_by_event_name.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/events_stages_history_long.frm to /mysql1/data/performance_schema/events_stages_history_long.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/events_statements_summary_by_program.frm to /mysql1/data/performance_schema/events_statements_summary_by_program.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/events_statements_summary_by_user_by_event_name.frm to /mysql1/data/performance_schema/events_statements_summary_by_user_by_event_name.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/session_account_connect_attrs.frm to /mysql1/data/performance_schema/session_account_connect_attrs.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/session_connect_attrs.frm to /mysql1/data/performance_schema/session_connect_attrs.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/events_statements_summary_by_account_by_event_name.frm to /mysql1/data/performance_schema/events_statements_summary_by_account_by_event_name.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/events_statements_summary_by_thread_by_event_name.frm to /mysql1/data/performance_schema/events_statements_summary_by_thread_by_event_name.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/events_transactions_history.frm to /mysql1/data/performance_schema/events_transactions_history.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/memory_summary_by_user_by_event_name.frm to /mysql1/data/performance_schema/memory_summary_by_user_by_event_name.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/events_statements_summary_global_by_event_name.frm to /mysql1/data/performance_schema/events_statements_summary_global_by_event_name.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/performance_timers.frm to /mysql1/data/performance_schema/performance_timers.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/setup_consumers.frm to /mysql1/data/performance_schema/setup_consumers.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/events_stages_summary_by_host_by_event_name.frm to /mysql1/data/performance_schema/events_stages_summary_by_host_by_event_name.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/events_transactions_history_long.frm to /mysql1/data/performance_schema/events_transactions_history_long.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/replication_connection_status.frm to /mysql1/data/performance_schema/replication_connection_status.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/replication_group_members.frm to /mysql1/data/performance_schema/replication_group_members.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/objects_summary_global_by_type.frm to /mysql1/data/performance_schema/objects_summary_global_by_type.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/rwlock_instances.frm to /mysql1/data/performance_schema/rwlock_instances.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/user_variables_by_thread.frm to /mysql1/data/performance_schema/user_variables_by_thread.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/events_waits_current.frm to /mysql1/data/performance_schema/events_waits_current.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/file_summary_by_event_name.frm to /mysql1/data/performance_schema/file_summary_by_event_name.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/users.frm to /mysql1/data/performance_schema/users.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/replication_applier_status_by_worker.frm to /mysql1/data/performance_schema/replication_applier_status_by_worker.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/socket_instances.frm to /mysql1/data/performance_schema/socket_instances.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/events_stages_current.frm to /mysql1/data/performance_schema/events_stages_current.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/global_status.frm to /mysql1/data/performance_schema/global_status.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/setup_timers.frm to /mysql1/data/performance_schema/setup_timers.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/events_waits_history.frm to /mysql1/data/performance_schema/events_waits_history.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/table_handles.frm to /mysql1/data/performance_schema/table_handles.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/events_statements_current.frm to /mysql1/data/performance_schema/events_statements_current.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/events_transactions_current.frm to /mysql1/data/performance_schema/events_transactions_current.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/events_waits_summary_global_by_event_name.frm to /mysql1/data/performance_schema/events_waits_summary_global_by_event_name.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/replication_applier_status_by_coordinator.frm to /mysql1/data/performance_schema/replication_applier_status_by_coordinator.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/events_statements_summary_by_host_by_event_name.frm to /mysql1/data/performance_schema/events_statements_summary_by_host_by_event_name.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/events_stages_history.frm to /mysql1/data/performance_schema/events_stages_history.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/events_statements_summary_by_digest.frm to /mysql1/data/performance_schema/events_statements_summary_by_digest.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/accounts.frm to /mysql1/data/performance_schema/accounts.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/replication_applier_configuration.frm to /mysql1/data/performance_schema/replication_applier_configuration.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/replication_connection_configuration.frm to /mysql1/data/performance_schema/replication_connection_configuration.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/status_by_user.frm to /mysql1/data/performance_schema/status_by_user.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/events_transactions_summary_global_by_event_name.frm to /mysql1/data/performance_schema/events_transactions_summary_global_by_event_name.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/events_transactions_summary_by_account_by_event_name.frm to /mysql1/data/performance_schema/events_transactions_summary_by_account_by_event_name.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/events_stages_summary_global_by_event_name.frm to /mysql1/data/performance_schema/events_stages_summary_global_by_event_name.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/socket_summary_by_event_name.frm to /mysql1/data/performance_schema/socket_summary_by_event_name.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/memory_summary_by_thread_by_event_name.frm to /mysql1/data/performance_schema/memory_summary_by_thread_by_event_name.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/status_by_thread.frm to /mysql1/data/performance_schema/status_by_thread.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/events_waits_summary_by_instance.frm to /mysql1/data/performance_schema/events_waits_summary_by_instance.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/events_transactions_summary_by_thread_by_event_name.frm to /mysql1/data/performance_schema/events_transactions_summary_by_thread_by_event_name.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/hosts.frm to /mysql1/data/performance_schema/hosts.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/cond_instances.frm to /mysql1/data/performance_schema/cond_instances.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/memory_summary_by_host_by_event_name.frm to /mysql1/data/performance_schema/memory_summary_by_host_by_event_name.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/status_by_account.frm to /mysql1/data/performance_schema/status_by_account.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/socket_summary_by_instance.frm to /mysql1/data/performance_schema/socket_summary_by_instance.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/events_transactions_summary_by_user_by_event_name.frm to /mysql1/data/performance_schema/events_transactions_summary_by_user_by_event_name.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/status_by_host.frm to /mysql1/data/performance_schema/status_by_host.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/events_stages_summary_by_user_by_event_name.frm to /mysql1/data/performance_schema/events_stages_summary_by_user_by_event_name.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/global_variables.frm to /mysql1/data/performance_schema/global_variables.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/setup_instruments.frm to /mysql1/data/performance_schema/setup_instruments.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/events_waits_summary_by_thread_by_event_name.frm to /mysql1/data/performance_schema/events_waits_summary_by_thread_by_event_name.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/replication_group_member_stats.frm to /mysql1/data/performance_schema/replication_group_member_stats.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/host_cache.frm to /mysql1/data/performance_schema/host_cache.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/setup_objects.frm to /mysql1/data/performance_schema/setup_objects.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/events_waits_history_long.frm to /mysql1/data/performance_schema/events_waits_history_long.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/session_status.frm to /mysql1/data/performance_schema/session_status.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/events_transactions_summary_by_host_by_event_name.frm to /mysql1/data/performance_schema/events_transactions_summary_by_host_by_event_name.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/prepared_statements_instances.frm to /mysql1/data/performance_schema/prepared_statements_instances.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/setup_actors.frm to /mysql1/data/performance_schema/setup_actors.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/mutex_instances.frm to /mysql1/data/performance_schema/mutex_instances.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/table_lock_waits_summary_by_table.frm to /mysql1/data/performance_schema/table_lock_waits_summary_by_table.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/variables_by_thread.frm to /mysql1/data/performance_schema/variables_by_thread.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/events_waits_summary_by_host_by_event_name.frm to /mysql1/data/performance_schema/events_waits_summary_by_host_by_event_name.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/events_statements_history_long.frm to /mysql1/data/performance_schema/events_statements_history_long.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/events_waits_summary_by_account_by_event_name.frm to /mysql1/data/performance_schema/events_waits_summary_by_account_by_event_name.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/session_variables.frm to /mysql1/data/performance_schema/session_variables.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/events_stages_summary_by_thread_by_event_name.frm to /mysql1/data/performance_schema/events_stages_summary_by_thread_by_event_name.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/file_summary_by_instance.frm to /mysql1/data/performance_schema/file_summary_by_instance.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/events_statements_history.frm to /mysql1/data/performance_schema/events_statements_history.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./performance_schema/table_io_waits_summary_by_index_usage.frm to /mysql1/data/performance_schema/table_io_waits_summary_by_index_usage.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./xtrabackup_binlog_pos_innodb to /mysql1/data/xtrabackup_binlog_pos_innodb
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./mysql/user.frm to /mysql1/data/mysql/user.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./mysql/func.MYI to /mysql1/data/mysql/func.MYI
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./mysql/help_topic.frm to /mysql1/data/mysql/help_topic.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./mysql/proxies_priv.MYI to /mysql1/data/mysql/proxies_priv.MYI
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./mysql/help_keyword.frm to /mysql1/data/mysql/help_keyword.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./mysql/func.frm to /mysql1/data/mysql/func.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./mysql/ndb_binlog_index.MYD to /mysql1/data/mysql/ndb_binlog_index.MYD
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./mysql/tables_priv.MYI to /mysql1/data/mysql/tables_priv.MYI
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./mysql/time_zone_name.ibd to /mysql1/data/mysql/time_zone_name.ibd
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./mysql/db.opt to /mysql1/data/mysql/db.opt
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./mysql/db.MYD to /mysql1/data/mysql/db.MYD
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./mysql/engine_cost.ibd to /mysql1/data/mysql/engine_cost.ibd
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./mysql/engine_cost.frm to /mysql1/data/mysql/engine_cost.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./mysql/ndb_binlog_index.MYI to /mysql1/data/mysql/ndb_binlog_index.MYI
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./mysql/procs_priv.MYI to /mysql1/data/mysql/procs_priv.MYI
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./mysql/help_category.frm to /mysql1/data/mysql/help_category.frm
180626 03:15:08 [01]        ...done
180626 03:15:08 [01] Copying ./mysql/ndb_binlog_index.frm to /mysql1/data/mysql/ndb_binlog_index.frm
180626 03:15:08 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/user.MYI to /mysql1/data/mysql/user.MYI
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/event.MYI to /mysql1/data/mysql/event.MYI
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/help_topic.ibd to /mysql1/data/mysql/help_topic.ibd
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/tables_priv.frm to /mysql1/data/mysql/tables_priv.frm
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/slave_master_info.frm to /mysql1/data/mysql/slave_master_info.frm
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/time_zone_transition_type.frm to /mysql1/data/mysql/time_zone_transition_type.frm
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/columns_priv.MYD to /mysql1/data/mysql/columns_priv.MYD
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/server_cost.ibd to /mysql1/data/mysql/server_cost.ibd
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/tables_priv.MYD to /mysql1/data/mysql/tables_priv.MYD
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/help_category.ibd to /mysql1/data/mysql/help_category.ibd
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/innodb_index_stats.ibd to /mysql1/data/mysql/innodb_index_stats.ibd
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/proxies_priv.frm to /mysql1/data/mysql/proxies_priv.frm
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/event.frm to /mysql1/data/mysql/event.frm
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/slow_log.CSV to /mysql1/data/mysql/slow_log.CSV
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/proc.MYI to /mysql1/data/mysql/proc.MYI
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/innodb_table_stats.ibd to /mysql1/data/mysql/innodb_table_stats.ibd
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/func.MYD to /mysql1/data/mysql/func.MYD
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/time_zone_transition_type.ibd to /mysql1/data/mysql/time_zone_transition_type.ibd
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/proc.frm to /mysql1/data/mysql/proc.frm
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/slave_relay_log_info.frm to /mysql1/data/mysql/slave_relay_log_info.frm
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/time_zone.frm to /mysql1/data/mysql/time_zone.frm
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/db.MYI to /mysql1/data/mysql/db.MYI
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/gtid_executed.frm to /mysql1/data/mysql/gtid_executed.frm
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/general_log.CSM to /mysql1/data/mysql/general_log.CSM
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/db.frm to /mysql1/data/mysql/db.frm
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/plugin.frm to /mysql1/data/mysql/plugin.frm
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/time_zone_leap_second.frm to /mysql1/data/mysql/time_zone_leap_second.frm
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/general_log.frm to /mysql1/data/mysql/general_log.frm
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/server_cost.frm to /mysql1/data/mysql/server_cost.frm
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/slave_worker_info.ibd to /mysql1/data/mysql/slave_worker_info.ibd
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/time_zone_transition.ibd to /mysql1/data/mysql/time_zone_transition.ibd
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/time_zone_leap_second.ibd to /mysql1/data/mysql/time_zone_leap_second.ibd
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/servers.frm to /mysql1/data/mysql/servers.frm
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/innodb_table_stats.frm to /mysql1/data/mysql/innodb_table_stats.frm
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/columns_priv.frm to /mysql1/data/mysql/columns_priv.frm
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/user.MYD to /mysql1/data/mysql/user.MYD
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/slave_worker_info.frm to /mysql1/data/mysql/slave_worker_info.frm
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/time_zone_transition.frm to /mysql1/data/mysql/time_zone_transition.frm
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/time_zone_name.frm to /mysql1/data/mysql/time_zone_name.frm
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/slow_log.frm to /mysql1/data/mysql/slow_log.frm
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/help_relation.frm to /mysql1/data/mysql/help_relation.frm
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/servers.ibd to /mysql1/data/mysql/servers.ibd
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/gtid_executed.ibd to /mysql1/data/mysql/gtid_executed.ibd
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/time_zone.ibd to /mysql1/data/mysql/time_zone.ibd
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/proxies_priv.MYD to /mysql1/data/mysql/proxies_priv.MYD
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/event.MYD to /mysql1/data/mysql/event.MYD
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/help_relation.ibd to /mysql1/data/mysql/help_relation.ibd
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/slow_log.CSM to /mysql1/data/mysql/slow_log.CSM
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/innodb_index_stats.frm to /mysql1/data/mysql/innodb_index_stats.frm
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/general_log.CSV to /mysql1/data/mysql/general_log.CSV
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/plugin.ibd to /mysql1/data/mysql/plugin.ibd
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/proc.MYD to /mysql1/data/mysql/proc.MYD
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/slave_master_info.ibd to /mysql1/data/mysql/slave_master_info.ibd
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/procs_priv.frm to /mysql1/data/mysql/procs_priv.frm
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/help_keyword.ibd to /mysql1/data/mysql/help_keyword.ibd
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/procs_priv.MYD to /mysql1/data/mysql/procs_priv.MYD
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/slave_relay_log_info.ibd to /mysql1/data/mysql/slave_relay_log_info.ibd
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./mysql/columns_priv.MYI to /mysql1/data/mysql/columns_priv.MYI
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./xtrabackup_info to /mysql1/data/xtrabackup_info
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./dao/db.opt to /mysql1/data/dao/db.opt
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./dao/penalties.frm to /mysql1/data/dao/penalties.frm
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./dao/emp.ibd to /mysql1/data/dao/emp.ibd
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./dao/dept.frm to /mysql1/data/dao/dept.frm
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./dao/matches.frm to /mysql1/data/dao/matches.frm
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./dao/matches.ibd to /mysql1/data/dao/matches.ibd
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./dao/teams.frm to /mysql1/data/dao/teams.frm
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./dao/players.ibd to /mysql1/data/dao/players.ibd
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./dao/penalties.ibd to /mysql1/data/dao/penalties.ibd
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./dao/committee_members.ibd to /mysql1/data/dao/committee_members.ibd
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./dao/teams.ibd to /mysql1/data/dao/teams.ibd
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./dao/t1.ibd to /mysql1/data/dao/t1.ibd
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./dao/committee_members.frm to /mysql1/data/dao/committee_members.frm
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./dao/dept.ibd to /mysql1/data/dao/dept.ibd
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./dao/t1.frm to /mysql1/data/dao/t1.frm
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./dao/players.frm to /mysql1/data/dao/players.frm
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./dao/emp.frm to /mysql1/data/dao/emp.frm
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./lepus/lepus_status.ibd to /mysql1/data/lepus/lepus_status.ibd
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./lepus/alarm_temp.ibd to /mysql1/data/lepus/alarm_temp.ibd
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./lepus/sqlserver_status_history.ibd to /mysql1/data/lepus/sqlserver_status_history.ibd
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./lepus/mysql_connected.ibd to /mysql1/data/lepus/mysql_connected.ibd
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./lepus/mysql_slow_query_review_history.ibd to /mysql1/data/lepus/mysql_slow_query_review_history.ibd
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./lepus/redis_replication.ibd to /mysql1/data/lepus/redis_replication.ibd
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./lepus/admin_menu.ibd to /mysql1/data/lepus/admin_menu.ibd
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./lepus/admin_privilege.ibd to /mysql1/data/lepus/admin_privilege.ibd
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./lepus/oracle_status.frm to /mysql1/data/lepus/oracle_status.frm
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./lepus/os_net.ibd to /mysql1/data/lepus/os_net.ibd
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./lepus/redis_replication.frm to /mysql1/data/lepus/redis_replication.frm
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./lepus/alarm_temp.frm to /mysql1/data/lepus/alarm_temp.frm
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./lepus/db.opt to /mysql1/data/lepus/db.opt
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./lepus/admin_user.ibd to /mysql1/data/lepus/admin_user.ibd
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./lepus/oracle_tablespace_history.ibd to /mysql1/data/lepus/oracle_tablespace_history.ibd
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./lepus/admin_role_privilege.ibd to /mysql1/data/lepus/admin_role_privilege.ibd
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./lepus/db_servers_os.frm to /mysql1/data/lepus/db_servers_os.frm
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./lepus/os_net.frm to /mysql1/data/lepus/os_net.frm
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./lepus/os_diskio.frm to /mysql1/data/lepus/os_diskio.frm
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./lepus/admin_role.frm to /mysql1/data/lepus/admin_role.frm
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./lepus/mysql_slow_query_review.ibd to /mysql1/data/lepus/mysql_slow_query_review.ibd
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./lepus/options.frm to /mysql1/data/lepus/options.frm
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./lepus/mysql_connected.frm to /mysql1/data/lepus/mysql_connected.frm
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./lepus/mysql_status.frm to /mysql1/data/lepus/mysql_status.frm
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./lepus/os_diskio.ibd to /mysql1/data/lepus/os_diskio.ibd
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./lepus/redis_status.frm to /mysql1/data/lepus/redis_status.frm
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./lepus/admin_user1.ibd to /mysql1/data/lepus/admin_user1.ibd
180626 03:15:09 [01]        ...done
180626 03:15:09 [01] Copying ./lepus/redis_status.ibd to /mysql1/data/lepus/redis_status.ibd
180626 03:15:09 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/mysql_slow_query_review_history.frm to /mysql1/data/lepus/mysql_slow_query_review_history.frm
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/sqlserver_status.frm to /mysql1/data/lepus/sqlserver_status.frm
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/mongodb_status_history.ibd to /mysql1/data/lepus/mongodb_status_history.ibd
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/sqlserver_status.ibd to /mysql1/data/lepus/sqlserver_status.ibd
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/os_disk_history.frm to /mysql1/data/lepus/os_disk_history.frm
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/admin_log.ibd to /mysql1/data/lepus/admin_log.ibd
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/db_status.frm to /mysql1/data/lepus/db_status.frm
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/mysql_bigtable_history.ibd to /mysql1/data/lepus/mysql_bigtable_history.ibd
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/db_servers_oracle.frm to /mysql1/data/lepus/db_servers_oracle.frm
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/os_disk_history.ibd to /mysql1/data/lepus/os_disk_history.ibd
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/admin_role_privilege.frm to /mysql1/data/lepus/admin_role_privilege.frm
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/os_status.ibd to /mysql1/data/lepus/os_status.ibd
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/db_servers_mysql.frm to /mysql1/data/lepus/db_servers_mysql.frm
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/options.ibd to /mysql1/data/lepus/options.ibd
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/redis_status_history.frm to /mysql1/data/lepus/redis_status_history.frm
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/db_servers_mongodb.frm to /mysql1/data/lepus/db_servers_mongodb.frm
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/mysql_bigtable.ibd to /mysql1/data/lepus/mysql_bigtable.ibd
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/oracle_tablespace.frm to /mysql1/data/lepus/oracle_tablespace.frm
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/admin_menu.frm to /mysql1/data/lepus/admin_menu.frm
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/alarm.ibd to /mysql1/data/lepus/alarm.ibd
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/os_net_history.ibd to /mysql1/data/lepus/os_net_history.ibd
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/db_servers_mysql.ibd to /mysql1/data/lepus/db_servers_mysql.ibd
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/mongodb_status.frm to /mysql1/data/lepus/mongodb_status.frm
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/os_status_history.ibd to /mysql1/data/lepus/os_status_history.ibd
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/os_net_history.frm to /mysql1/data/lepus/os_net_history.frm
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/mysql_replication_history.frm to /mysql1/data/lepus/mysql_replication_history.frm
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/redis_replication_history.ibd to /mysql1/data/lepus/redis_replication_history.ibd
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/mongodb_status_history.frm to /mysql1/data/lepus/mongodb_status_history.frm
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/os_disk.ibd to /mysql1/data/lepus/os_disk.ibd
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/admin_user.frm to /mysql1/data/lepus/admin_user.frm
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/oracle_status_history.frm to /mysql1/data/lepus/oracle_status_history.frm
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/oracle_status_history.ibd to /mysql1/data/lepus/oracle_status_history.ibd
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/oracle_tablespace_history.frm to /mysql1/data/lepus/oracle_tablespace_history.frm
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/mysql_processlist.ibd to /mysql1/data/lepus/mysql_processlist.ibd
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/sqlserver_status_history.frm to /mysql1/data/lepus/sqlserver_status_history.frm
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/mysql_bigtable_history.frm to /mysql1/data/lepus/mysql_bigtable_history.frm
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/mysql_processlist.frm to /mysql1/data/lepus/mysql_processlist.frm
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/lepus_status.frm to /mysql1/data/lepus/lepus_status.frm
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/db_servers_sqlserver.frm to /mysql1/data/lepus/db_servers_sqlserver.frm
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/mysql_bigtable.frm to /mysql1/data/lepus/mysql_bigtable.frm
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/os_diskio_history.frm to /mysql1/data/lepus/os_diskio_history.frm
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/db_servers_os.ibd to /mysql1/data/lepus/db_servers_os.ibd
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/mysql_replication.frm to /mysql1/data/lepus/mysql_replication.frm
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/mysql_slow_query_review.frm to /mysql1/data/lepus/mysql_slow_query_review.frm
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/alarm_history.frm to /mysql1/data/lepus/alarm_history.frm
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/mysql_slow_query_sendmail_log.ibd to /mysql1/data/lepus/mysql_slow_query_sendmail_log.ibd
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/redis_replication_history.frm to /mysql1/data/lepus/redis_replication_history.frm
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/alarm_history.ibd to /mysql1/data/lepus/alarm_history.ibd
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/mongodb_status.ibd to /mysql1/data/lepus/mongodb_status.ibd
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/oracle_status.ibd to /mysql1/data/lepus/oracle_status.ibd
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/db_servers_redis.ibd to /mysql1/data/lepus/db_servers_redis.ibd
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/admin_role.ibd to /mysql1/data/lepus/admin_role.ibd
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/mysql_replication_history.ibd to /mysql1/data/lepus/mysql_replication_history.ibd
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/os_disk.frm to /mysql1/data/lepus/os_disk.frm
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/mysql_slow_query_sendmail_log.frm to /mysql1/data/lepus/mysql_slow_query_sendmail_log.frm
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/admin_log.frm to /mysql1/data/lepus/admin_log.frm
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/admin_user_role.frm to /mysql1/data/lepus/admin_user_role.frm
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/db_servers_oracle.ibd to /mysql1/data/lepus/db_servers_oracle.ibd
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/alarm.frm to /mysql1/data/lepus/alarm.frm
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/admin_user_role.ibd to /mysql1/data/lepus/admin_user_role.ibd
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/mysql_replication.ibd to /mysql1/data/lepus/mysql_replication.ibd
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/oracle_tablespace.ibd to /mysql1/data/lepus/oracle_tablespace.ibd
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/mysql_status.ibd to /mysql1/data/lepus/mysql_status.ibd
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/db_status.ibd to /mysql1/data/lepus/db_status.ibd
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/os_diskio_history.ibd to /mysql1/data/lepus/os_diskio_history.ibd
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/admin_user1.frm to /mysql1/data/lepus/admin_user1.frm
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/os_status_history.frm to /mysql1/data/lepus/os_status_history.frm
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/db_servers_redis.frm to /mysql1/data/lepus/db_servers_redis.frm
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/db_servers_sqlserver.ibd to /mysql1/data/lepus/db_servers_sqlserver.ibd
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/mysql_status_history.ibd to /mysql1/data/lepus/mysql_status_history.ibd
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/mysql_status_history.frm to /mysql1/data/lepus/mysql_status_history.frm
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/db_servers_mongodb.ibd to /mysql1/data/lepus/db_servers_mongodb.ibd
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/redis_status_history.ibd to /mysql1/data/lepus/redis_status_history.ibd
180626 03:15:10 [01]        ...done
180626 03:15:10 [01] Copying ./lepus/os_status.frm to /mysql1/data/lepus/os_status.frm
180626 03:15:10 [01]        ...done
180626 03:15:11 [01] Copying ./lepus/admin_privilege.frm to /mysql1/data/lepus/admin_privilege.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./ib_buffer_pool to /mysql1/data/ib_buffer_pool
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024innodb_buffer_stats_by_schema.frm to /mysql1/data/sys/x@0024innodb_buffer_stats_by_schema.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024statement_analysis.frm to /mysql1/data/sys/x@0024statement_analysis.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024user_summary.frm to /mysql1/data/sys/x@0024user_summary.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024schema_index_statistics.frm to /mysql1/data/sys/x@0024schema_index_statistics.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024statements_with_runtimes_in_95th_percentile.frm to /mysql1/data/sys/x@0024statements_with_runtimes_in_95th_percentile.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024innodb_buffer_stats_by_table.frm to /mysql1/data/sys/x@0024innodb_buffer_stats_by_table.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/sys_config.ibd to /mysql1/data/sys/sys_config.ibd
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/db.opt to /mysql1/data/sys/db.opt
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/host_summary_by_statement_type.frm to /mysql1/data/sys/host_summary_by_statement_type.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/sys_config_insert_set_user.TRN to /mysql1/data/sys/sys_config_insert_set_user.TRN
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/statement_analysis.frm to /mysql1/data/sys/statement_analysis.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/version.frm to /mysql1/data/sys/version.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/host_summary.frm to /mysql1/data/sys/host_summary.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024io_global_by_wait_by_latency.frm to /mysql1/data/sys/x@0024io_global_by_wait_by_latency.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/statements_with_sorting.frm to /mysql1/data/sys/statements_with_sorting.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/sys_config.frm to /mysql1/data/sys/sys_config.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024io_by_thread_by_latency.frm to /mysql1/data/sys/x@0024io_by_thread_by_latency.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/memory_by_host_by_current_bytes.frm to /mysql1/data/sys/memory_by_host_by_current_bytes.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024memory_by_thread_by_current_bytes.frm to /mysql1/data/sys/x@0024memory_by_thread_by_current_bytes.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024schema_flattened_keys.frm to /mysql1/data/sys/x@0024schema_flattened_keys.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024memory_by_host_by_current_bytes.frm to /mysql1/data/sys/x@0024memory_by_host_by_current_bytes.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/sys_config.TRG to /mysql1/data/sys/sys_config.TRG
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/user_summary_by_file_io.frm to /mysql1/data/sys/user_summary_by_file_io.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024latest_file_io.frm to /mysql1/data/sys/x@0024latest_file_io.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/schema_object_overview.frm to /mysql1/data/sys/schema_object_overview.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/schema_unused_indexes.frm to /mysql1/data/sys/schema_unused_indexes.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024processlist.frm to /mysql1/data/sys/x@0024processlist.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/statements_with_full_table_scans.frm to /mysql1/data/sys/statements_with_full_table_scans.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024host_summary_by_file_io.frm to /mysql1/data/sys/x@0024host_summary_by_file_io.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024host_summary.frm to /mysql1/data/sys/x@0024host_summary.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/memory_by_user_by_current_bytes.frm to /mysql1/data/sys/memory_by_user_by_current_bytes.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/wait_classes_global_by_latency.frm to /mysql1/data/sys/wait_classes_global_by_latency.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024io_global_by_wait_by_bytes.frm to /mysql1/data/sys/x@0024io_global_by_wait_by_bytes.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024ps_schema_table_statistics_io.frm to /mysql1/data/sys/x@0024ps_schema_table_statistics_io.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024schema_table_statistics_with_buffer.frm to /mysql1/data/sys/x@0024schema_table_statistics_with_buffer.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/session_ssl_status.frm to /mysql1/data/sys/session_ssl_status.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/innodb_buffer_stats_by_schema.frm to /mysql1/data/sys/innodb_buffer_stats_by_schema.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024waits_by_user_by_latency.frm to /mysql1/data/sys/x@0024waits_by_user_by_latency.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024host_summary_by_stages.frm to /mysql1/data/sys/x@0024host_summary_by_stages.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024waits_global_by_latency.frm to /mysql1/data/sys/x@0024waits_global_by_latency.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/schema_tables_with_full_table_scans.frm to /mysql1/data/sys/schema_tables_with_full_table_scans.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024schema_table_lock_waits.frm to /mysql1/data/sys/x@0024schema_table_lock_waits.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024memory_by_user_by_current_bytes.frm to /mysql1/data/sys/x@0024memory_by_user_by_current_bytes.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/innodb_lock_waits.frm to /mysql1/data/sys/innodb_lock_waits.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024ps_digest_avg_latency_distribution.frm to /mysql1/data/sys/x@0024ps_digest_avg_latency_distribution.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024schema_table_statistics.frm to /mysql1/data/sys/x@0024schema_table_statistics.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024host_summary_by_file_io_type.frm to /mysql1/data/sys/x@0024host_summary_by_file_io_type.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/latest_file_io.frm to /mysql1/data/sys/latest_file_io.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/schema_redundant_indexes.frm to /mysql1/data/sys/schema_redundant_indexes.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/memory_global_by_current_bytes.frm to /mysql1/data/sys/memory_global_by_current_bytes.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024memory_global_by_current_bytes.frm to /mysql1/data/sys/x@0024memory_global_by_current_bytes.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/statements_with_errors_or_warnings.frm to /mysql1/data/sys/statements_with_errors_or_warnings.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024wait_classes_global_by_avg_latency.frm to /mysql1/data/sys/x@0024wait_classes_global_by_avg_latency.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024wait_classes_global_by_latency.frm to /mysql1/data/sys/x@0024wait_classes_global_by_latency.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024session.frm to /mysql1/data/sys/x@0024session.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024schema_tables_with_full_table_scans.frm to /mysql1/data/sys/x@0024schema_tables_with_full_table_scans.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/host_summary_by_file_io_type.frm to /mysql1/data/sys/host_summary_by_file_io_type.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/user_summary.frm to /mysql1/data/sys/user_summary.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024user_summary_by_stages.frm to /mysql1/data/sys/x@0024user_summary_by_stages.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/user_summary_by_file_io_type.frm to /mysql1/data/sys/user_summary_by_file_io_type.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/schema_table_statistics.frm to /mysql1/data/sys/schema_table_statistics.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/waits_by_user_by_latency.frm to /mysql1/data/sys/waits_by_user_by_latency.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/io_global_by_wait_by_bytes.frm to /mysql1/data/sys/io_global_by_wait_by_bytes.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/user_summary_by_statement_type.frm to /mysql1/data/sys/user_summary_by_statement_type.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024statements_with_sorting.frm to /mysql1/data/sys/x@0024statements_with_sorting.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/user_summary_by_statement_latency.frm to /mysql1/data/sys/user_summary_by_statement_latency.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024user_summary_by_statement_latency.frm to /mysql1/data/sys/x@0024user_summary_by_statement_latency.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/session.frm to /mysql1/data/sys/session.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/schema_table_statistics_with_buffer.frm to /mysql1/data/sys/schema_table_statistics_with_buffer.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/host_summary_by_file_io.frm to /mysql1/data/sys/host_summary_by_file_io.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/memory_by_thread_by_current_bytes.frm to /mysql1/data/sys/memory_by_thread_by_current_bytes.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024waits_by_host_by_latency.frm to /mysql1/data/sys/x@0024waits_by_host_by_latency.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/schema_auto_increment_columns.frm to /mysql1/data/sys/schema_auto_increment_columns.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024ps_digest_95th_percentile_by_avg_us.frm to /mysql1/data/sys/x@0024ps_digest_95th_percentile_by_avg_us.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/waits_by_host_by_latency.frm to /mysql1/data/sys/waits_by_host_by_latency.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/host_summary_by_statement_latency.frm to /mysql1/data/sys/host_summary_by_statement_latency.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/metrics.frm to /mysql1/data/sys/metrics.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024user_summary_by_file_io_type.frm to /mysql1/data/sys/x@0024user_summary_by_file_io_type.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/io_global_by_file_by_bytes.frm to /mysql1/data/sys/io_global_by_file_by_bytes.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/io_by_thread_by_latency.frm to /mysql1/data/sys/io_by_thread_by_latency.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/processlist.frm to /mysql1/data/sys/processlist.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/statements_with_runtimes_in_95th_percentile.frm to /mysql1/data/sys/statements_with_runtimes_in_95th_percentile.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/schema_table_lock_waits.frm to /mysql1/data/sys/schema_table_lock_waits.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/wait_classes_global_by_avg_latency.frm to /mysql1/data/sys/wait_classes_global_by_avg_latency.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/user_summary_by_stages.frm to /mysql1/data/sys/user_summary_by_stages.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/io_global_by_file_by_latency.frm to /mysql1/data/sys/io_global_by_file_by_latency.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024host_summary_by_statement_type.frm to /mysql1/data/sys/x@0024host_summary_by_statement_type.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024innodb_lock_waits.frm to /mysql1/data/sys/x@0024innodb_lock_waits.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/ps_check_lost_instrumentation.frm to /mysql1/data/sys/ps_check_lost_instrumentation.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/waits_global_by_latency.frm to /mysql1/data/sys/waits_global_by_latency.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/sys_config_update_set_user.TRN to /mysql1/data/sys/sys_config_update_set_user.TRN
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/innodb_buffer_stats_by_table.frm to /mysql1/data/sys/innodb_buffer_stats_by_table.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024statements_with_temp_tables.frm to /mysql1/data/sys/x@0024statements_with_temp_tables.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/io_global_by_wait_by_latency.frm to /mysql1/data/sys/io_global_by_wait_by_latency.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/host_summary_by_stages.frm to /mysql1/data/sys/host_summary_by_stages.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/statements_with_temp_tables.frm to /mysql1/data/sys/statements_with_temp_tables.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024user_summary_by_file_io.frm to /mysql1/data/sys/x@0024user_summary_by_file_io.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/schema_index_statistics.frm to /mysql1/data/sys/schema_index_statistics.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024host_summary_by_statement_latency.frm to /mysql1/data/sys/x@0024host_summary_by_statement_latency.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024user_summary_by_statement_type.frm to /mysql1/data/sys/x@0024user_summary_by_statement_type.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024statements_with_errors_or_warnings.frm to /mysql1/data/sys/x@0024statements_with_errors_or_warnings.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024io_global_by_file_by_latency.frm to /mysql1/data/sys/x@0024io_global_by_file_by_latency.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024io_global_by_file_by_bytes.frm to /mysql1/data/sys/x@0024io_global_by_file_by_bytes.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024memory_global_total.frm to /mysql1/data/sys/x@0024memory_global_total.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/memory_global_total.frm to /mysql1/data/sys/memory_global_total.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./sys/x@0024statements_with_full_table_scans.frm to /mysql1/data/sys/x@0024statements_with_full_table_scans.frm
180626 03:15:11 [01]        ...done
180626 03:15:11 [01] Copying ./ibtmp1 to /mysql1/data/ibtmp1
180626 03:15:12 [01]        ...done
180626 03:15:12 completed OK!

```

### 查看恢复后的文件



```sql

[mysql@bosenrui mysql1]$ ll /mysql1/data
total 188456
drwxr-x--- 2 mysql mysql     4096 Jun 26 03:15 dao
-rw-r----- 1 mysql mysql      969 Jun 26 03:15 ib_buffer_pool
-rw-r----- 1 mysql mysql 79691776 Jun 26 03:15 ibdata1
-rw-r----- 1 mysql mysql 50331648 Jun 26 03:15 ib_logfile0
-rw-r----- 1 mysql mysql 50331648 Jun 26 03:15 ib_logfile1
-rw-r----- 1 mysql mysql 12582912 Jun 26 03:15 ibtmp1
drwxr-x--- 2 mysql mysql     4096 Jun 26 03:15 lepus
drwxr-x--- 2 mysql mysql     4096 Jun 26 03:15 mysql
drwxr-x--- 2 mysql mysql     4096 Jun 26 03:15 performance_schema
drwxr-x--- 2 mysql mysql    12288 Jun 26 03:15 sys
-rw-r----- 1 mysql mysql       25 Jun 26 03:15 xtrabackup_binlog_pos_innodb
-rw-r----- 1 mysql mysql      536 Jun 26 03:15 xtrabackup_info



```

## 验证

### 启动Mysql


```sql

[mysql@bosenrui mysql1]$ mysqld --defaults-file=/mysql1/mysql1.cnf  &
[1] 10092


```

### 查看数据


```sql


[mysql@bosenrui mysql1]$ mysql -S /tmp/mysql1.sock -uroot -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 35
Server version: 5.7.19-log Source distribution

Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> use dao ;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables ;
+-------------------+
| Tables_in_dao     |
+-------------------+
| committee_members |
| dept              |
| emp               |
| matches           |
| penalties         |
| players           |
| t1                |
| teams             |
+-------------------+
8 rows in set (0.00 sec)


mysql> select * from dept ;
+--------+------------+----------+
| deptNO | DNAME      | LOC      |
+--------+------------+----------+
|     10 | ACCOUNTING | NEW YORK |
|     20 | RESEARCH   | DALLAS   |
|     30 | SALES      | CHICAGO  |
|     40 | OPERATIONS | BOSTON   |
+--------+------------+----------+
4 rows in set (0.01 sec)


```

