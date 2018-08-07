---
layout: post
title: 使用mysqldump在线搭建主从
date: 2018-08-07
categories: blog
tags: [MySQL]
description: 使用mysqldump在线搭建主从
---


## 备份部分

### 建立测试表并插入标识数据

```sql

mysql> create table t1 ( c1 int) ;
Query OK, 0 rows affected (0.07 sec)

mysql> insert into t1 values (1) ;
Query OK, 1 row affected (0.00 sec)

mysql> commit ;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from t1 ;
+------+
| c1   |
+------+
|    1 |
+------+
1 row in set (0.00 sec)

```


### 进行全量备份

```sql


[mysql@bosenrui ~]$  innobackupex  --defaults-file=/mysql1/mysql1.cnf  --no-timestamp   -u root -p root   /home/mysql/db_full_backup/
180729 13:25:46 innobackupex: Starting the backup operation

IMPORTANT: Please check that the backup run completes successfully.
           At the end of a successful backup run innobackupex
           prints "completed OK!".

180729 13:25:46  version_check Connecting to MySQL server with DSN 'dbi:mysql:;mysql_read_default_group=xtrabackup;port=3306;mysql_socket=/tmp/mysql1.sock' as 'root'  (using password: YES).
180729 13:25:46  version_check Connected to MySQL server
180729 13:25:46  version_check Executing a version check against the server...
180729 13:25:46  version_check Done.
180729 13:25:46 Connecting to MySQL server host: localhost, user: root, password: set, port: 3306, socket: /tmp/mysql1.sock
Using server version 5.7.21-log
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
180729 13:25:46 >> log scanned up to (2966340)
xtrabackup: Generating a list of tablespaces
InnoDB: Allocated tablespace ID 8 for mysql/time_zone_name, old maximum was 0
180729 13:25:46 [01] Copying ./ibdata1 to /home/mysql/db_full_backup/ibdata1
180729 13:25:46 [01]        ...done
180729 13:25:46 [01] Copying ./mysql/time_zone_name.ibd to /home/mysql/db_full_backup/mysql/time_zone_name.ibd
180729 13:25:46 [01]        ...done
180729 13:25:46 [01] Copying ./mysql/engine_cost.ibd to /home/mysql/db_full_backup/mysql/engine_cost.ibd
180729 13:25:46 [01]        ...done
180729 13:25:46 [01] Copying ./mysql/help_topic.ibd to /home/mysql/db_full_backup/mysql/help_topic.ibd
180729 13:25:46 [01]        ...done
180729 13:25:46 [01] Copying ./mysql/server_cost.ibd to /home/mysql/db_full_backup/mysql/server_cost.ibd
180729 13:25:46 [01]        ...done
180729 13:25:46 [01] Copying ./mysql/help_category.ibd to /home/mysql/db_full_backup/mysql/help_category.ibd
180729 13:25:46 [01]        ...done
180729 13:25:46 [01] Copying ./mysql/innodb_index_stats.ibd to /home/mysql/db_full_backup/mysql/innodb_index_stats.ibd
180729 13:25:46 [01]        ...done
180729 13:25:46 [01] Copying ./mysql/innodb_table_stats.ibd to /home/mysql/db_full_backup/mysql/innodb_table_stats.ibd
180729 13:25:46 [01]        ...done
180729 13:25:46 [01] Copying ./mysql/time_zone_transition_type.ibd to /home/mysql/db_full_backup/mysql/time_zone_transition_type.ibd
180729 13:25:46 [01]        ...done
180729 13:25:46 [01] Copying ./mysql/slave_worker_info.ibd to /home/mysql/db_full_backup/mysql/slave_worker_info.ibd
180729 13:25:46 [01]        ...done
180729 13:25:46 [01] Copying ./mysql/time_zone_transition.ibd to /home/mysql/db_full_backup/mysql/time_zone_transition.ibd
180729 13:25:46 [01]        ...done
180729 13:25:46 [01] Copying ./mysql/time_zone_leap_second.ibd to /home/mysql/db_full_backup/mysql/time_zone_leap_second.ibd
180729 13:25:46 [01]        ...done
180729 13:25:46 [01] Copying ./mysql/servers.ibd to /home/mysql/db_full_backup/mysql/servers.ibd
180729 13:25:46 [01]        ...done
180729 13:25:46 [01] Copying ./mysql/gtid_executed.ibd to /home/mysql/db_full_backup/mysql/gtid_executed.ibd
180729 13:25:46 [01]        ...done
180729 13:25:46 [01] Copying ./mysql/time_zone.ibd to /home/mysql/db_full_backup/mysql/time_zone.ibd
180729 13:25:46 [01]        ...done
180729 13:25:46 [01] Copying ./mysql/help_relation.ibd to /home/mysql/db_full_backup/mysql/help_relation.ibd
180729 13:25:46 [01]        ...done
180729 13:25:46 [01] Copying ./mysql/plugin.ibd to /home/mysql/db_full_backup/mysql/plugin.ibd
180729 13:25:46 [01]        ...done
180729 13:25:46 [01] Copying ./mysql/slave_master_info.ibd to /home/mysql/db_full_backup/mysql/slave_master_info.ibd
180729 13:25:46 [01]        ...done
180729 13:25:46 [01] Copying ./mysql/help_keyword.ibd to /home/mysql/db_full_backup/mysql/help_keyword.ibd
180729 13:25:46 [01]        ...done
180729 13:25:47 [01] Copying ./mysql/slave_relay_log_info.ibd to /home/mysql/db_full_backup/mysql/slave_relay_log_info.ibd
180729 13:25:47 [01]        ...done
180729 13:25:47 [01] Copying ./dao/emp.ibd to /home/mysql/db_full_backup/dao/emp.ibd
180729 13:25:47 [01]        ...done
180729 13:25:47 [01] Copying ./dao/test_tx_iso.ibd to /home/mysql/db_full_backup/dao/test_tx_iso.ibd
180729 13:25:47 [01]        ...done
180729 13:25:47 [01] Copying ./dao/matches.ibd to /home/mysql/db_full_backup/dao/matches.ibd
180729 13:25:47 [01]        ...done
180729 13:25:47 [01] Copying ./dao/test_binlog.ibd to /home/mysql/db_full_backup/dao/test_binlog.ibd
180729 13:25:47 [01]        ...done
180729 13:25:47 [01] Copying ./dao/players.ibd to /home/mysql/db_full_backup/dao/players.ibd
180729 13:25:47 [01]        ...done
180729 13:25:47 [01] Copying ./dao/sal_grades.ibd to /home/mysql/db_full_backup/dao/sal_grades.ibd
180729 13:25:47 [01]        ...done
180729 13:25:47 [01] Copying ./dao/test_import.ibd to /home/mysql/db_full_backup/dao/test_import.ibd
180729 13:25:47 [01]        ...done
180729 13:25:47 [01] Copying ./dao/penalties.ibd to /home/mysql/db_full_backup/dao/penalties.ibd
180729 13:25:47 [01]        ...done
180729 13:25:47 [01] Copying ./dao/committee_members.ibd to /home/mysql/db_full_backup/dao/committee_members.ibd
180729 13:25:47 [01]        ...done
180729 13:25:47 [01] Copying ./dao/teams.ibd to /home/mysql/db_full_backup/dao/teams.ibd
180729 13:25:47 [01]        ...done
180729 13:25:47 [01] Copying ./dao/t1.ibd to /home/mysql/db_full_backup/dao/t1.ibd
180729 13:25:47 [01]        ...done
180729 13:25:47 [01] Copying ./dao/dept.ibd to /home/mysql/db_full_backup/dao/dept.ibd
180729 13:25:47 [01]        ...done
180729 13:25:47 [01] Copying ./sys/sys_config.ibd to /home/mysql/db_full_backup/sys/sys_config.ibd
180729 13:25:47 [01]        ...done
180729 13:25:47 >> log scanned up to (2966340)
180729 13:25:47 Executing FLUSH NO_WRITE_TO_BINLOG TABLES...
180729 13:25:47 Executing FLUSH TABLES WITH READ LOCK...
180729 13:25:47 Starting to backup non-InnoDB tables and files
180729 13:25:47 [01] Copying ./performance_schema/replication_applier_status.frm to /home/mysql/db_full_backup/performance_schema/replication_applier_status.frm
180729 13:25:47 [01]        ...done
180729 13:25:47 [01] Copying ./performance_schema/file_instances.frm to /home/mysql/db_full_backup/performance_schema/file_instances.frm
180729 13:25:47 [01]        ...done
180729 13:25:47 [01] Copying ./performance_schema/metadata_locks.frm to /home/mysql/db_full_backup/performance_schema/metadata_locks.frm
180729 13:25:47 [01]        ...done
180729 13:25:47 [01] Copying ./performance_schema/memory_summary_by_account_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/memory_summary_by_account_by_event_name.frm
180729 13:25:47 [01]        ...done
180729 13:25:47 [01] Copying ./performance_schema/threads.frm to /home/mysql/db_full_backup/performance_schema/threads.frm
180729 13:25:47 [01]        ...done
180729 13:25:47 [01] Copying ./performance_schema/events_waits_summary_by_user_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/events_waits_summary_by_user_by_event_name.frm
180729 13:25:47 [01]        ...done
180729 13:25:47 [01] Copying ./performance_schema/events_stages_summary_by_account_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/events_stages_summary_by_account_by_event_name.frm
180729 13:25:47 [01]        ...done
180729 13:25:47 [01] Copying ./performance_schema/db.opt to /home/mysql/db_full_backup/performance_schema/db.opt
180729 13:25:47 [01]        ...done
180729 13:25:47 [01] Copying ./performance_schema/table_io_waits_summary_by_table.frm to /home/mysql/db_full_backup/performance_schema/table_io_waits_summary_by_table.frm
180729 13:25:47 [01]        ...done
180729 13:25:47 [01] Copying ./performance_schema/memory_summary_global_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/memory_summary_global_by_event_name.frm
180729 13:25:47 [01]        ...done
180729 13:25:47 [01] Copying ./performance_schema/events_stages_history_long.frm to /home/mysql/db_full_backup/performance_schema/events_stages_history_long.frm
180729 13:25:47 [01]        ...done
180729 13:25:47 [01] Copying ./performance_schema/events_statements_summary_by_program.frm to /home/mysql/db_full_backup/performance_schema/events_statements_summary_by_program.frm
180729 13:25:47 [01]        ...done
180729 13:25:47 [01] Copying ./performance_schema/events_statements_summary_by_user_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/events_statements_summary_by_user_by_event_name.frm
180729 13:25:47 [01]        ...done
180729 13:25:47 [01] Copying ./performance_schema/session_account_connect_attrs.frm to /home/mysql/db_full_backup/performance_schema/session_account_connect_attrs.frm
180729 13:25:47 [01]        ...done
180729 13:25:47 [01] Copying ./performance_schema/session_connect_attrs.frm to /home/mysql/db_full_backup/performance_schema/session_connect_attrs.frm
180729 13:25:47 [01]        ...done
180729 13:25:47 [01] Copying ./performance_schema/events_statements_summary_by_account_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/events_statements_summary_by_account_by_event_name.frm
180729 13:25:47 [01]        ...done
180729 13:25:47 [01] Copying ./performance_schema/events_statements_summary_by_thread_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/events_statements_summary_by_thread_by_event_name.frm
180729 13:25:47 [01]        ...done
180729 13:25:47 [01] Copying ./performance_schema/events_transactions_history.frm to /home/mysql/db_full_backup/performance_schema/events_transactions_history.frm
180729 13:25:47 [01]        ...done
180729 13:25:47 [01] Copying ./performance_schema/memory_summary_by_user_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/memory_summary_by_user_by_event_name.frm
180729 13:25:47 [01]        ...done
180729 13:25:47 [01] Copying ./performance_schema/events_statements_summary_global_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/events_statements_summary_global_by_event_name.frm
180729 13:25:47 [01]        ...done
180729 13:25:47 [01] Copying ./performance_schema/performance_timers.frm to /home/mysql/db_full_backup/performance_schema/performance_timers.frm
180729 13:25:47 [01]        ...done
180729 13:25:47 [01] Copying ./performance_schema/setup_consumers.frm to /home/mysql/db_full_backup/performance_schema/setup_consumers.frm
180729 13:25:47 [01]        ...done
180729 13:25:47 [01] Copying ./performance_schema/events_stages_summary_by_host_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/events_stages_summary_by_host_by_event_name.frm
180729 13:25:47 [01]        ...done
180729 13:25:47 [01] Copying ./performance_schema/events_transactions_history_long.frm to /home/mysql/db_full_backup/performance_schema/events_transactions_history_long.frm
180729 13:25:47 [01]        ...done
180729 13:25:47 [01] Copying ./performance_schema/replication_connection_status.frm to /home/mysql/db_full_backup/performance_schema/replication_connection_status.frm
180729 13:25:47 [01]        ...done
180729 13:25:47 [01] Copying ./performance_schema/replication_group_members.frm to /home/mysql/db_full_backup/performance_schema/replication_group_members.frm
180729 13:25:47 [01]        ...done
180729 13:25:47 [01] Copying ./performance_schema/objects_summary_global_by_type.frm to /home/mysql/db_full_backup/performance_schema/objects_summary_global_by_type.frm
180729 13:25:47 [01]        ...done
180729 13:25:47 [01] Copying ./performance_schema/rwlock_instances.frm to /home/mysql/db_full_backup/performance_schema/rwlock_instances.frm
180729 13:25:47 [01]        ...done
180729 13:25:47 [01] Copying ./performance_schema/user_variables_by_thread.frm to /home/mysql/db_full_backup/performance_schema/user_variables_by_thread.frm
180729 13:25:47 [01]        ...done
180729 13:25:47 [01] Copying ./performance_schema/events_waits_current.frm to /home/mysql/db_full_backup/performance_schema/events_waits_current.frm
180729 13:25:47 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/file_summary_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/file_summary_by_event_name.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/users.frm to /home/mysql/db_full_backup/performance_schema/users.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/replication_applier_status_by_worker.frm to /home/mysql/db_full_backup/performance_schema/replication_applier_status_by_worker.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/socket_instances.frm to /home/mysql/db_full_backup/performance_schema/socket_instances.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/events_stages_current.frm to /home/mysql/db_full_backup/performance_schema/events_stages_current.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/global_status.frm to /home/mysql/db_full_backup/performance_schema/global_status.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/setup_timers.frm to /home/mysql/db_full_backup/performance_schema/setup_timers.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/events_waits_history.frm to /home/mysql/db_full_backup/performance_schema/events_waits_history.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/table_handles.frm to /home/mysql/db_full_backup/performance_schema/table_handles.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/events_statements_current.frm to /home/mysql/db_full_backup/performance_schema/events_statements_current.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/events_transactions_current.frm to /home/mysql/db_full_backup/performance_schema/events_transactions_current.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/events_waits_summary_global_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/events_waits_summary_global_by_event_name.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/replication_applier_status_by_coordinator.frm to /home/mysql/db_full_backup/performance_schema/replication_applier_status_by_coordinator.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/events_statements_summary_by_host_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/events_statements_summary_by_host_by_event_name.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/events_stages_history.frm to /home/mysql/db_full_backup/performance_schema/events_stages_history.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/events_statements_summary_by_digest.frm to /home/mysql/db_full_backup/performance_schema/events_statements_summary_by_digest.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/accounts.frm to /home/mysql/db_full_backup/performance_schema/accounts.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/replication_applier_configuration.frm to /home/mysql/db_full_backup/performance_schema/replication_applier_configuration.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/replication_connection_configuration.frm to /home/mysql/db_full_backup/performance_schema/replication_connection_configuration.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/status_by_user.frm to /home/mysql/db_full_backup/performance_schema/status_by_user.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/events_transactions_summary_global_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/events_transactions_summary_global_by_event_name.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/events_transactions_summary_by_account_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/events_transactions_summary_by_account_by_event_name.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/events_stages_summary_global_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/events_stages_summary_global_by_event_name.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/socket_summary_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/socket_summary_by_event_name.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/memory_summary_by_thread_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/memory_summary_by_thread_by_event_name.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/status_by_thread.frm to /home/mysql/db_full_backup/performance_schema/status_by_thread.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/events_waits_summary_by_instance.frm to /home/mysql/db_full_backup/performance_schema/events_waits_summary_by_instance.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/events_transactions_summary_by_thread_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/events_transactions_summary_by_thread_by_event_name.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/hosts.frm to /home/mysql/db_full_backup/performance_schema/hosts.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/cond_instances.frm to /home/mysql/db_full_backup/performance_schema/cond_instances.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/memory_summary_by_host_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/memory_summary_by_host_by_event_name.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/status_by_account.frm to /home/mysql/db_full_backup/performance_schema/status_by_account.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/socket_summary_by_instance.frm to /home/mysql/db_full_backup/performance_schema/socket_summary_by_instance.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/events_transactions_summary_by_user_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/events_transactions_summary_by_user_by_event_name.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/status_by_host.frm to /home/mysql/db_full_backup/performance_schema/status_by_host.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/events_stages_summary_by_user_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/events_stages_summary_by_user_by_event_name.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/global_variables.frm to /home/mysql/db_full_backup/performance_schema/global_variables.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/setup_instruments.frm to /home/mysql/db_full_backup/performance_schema/setup_instruments.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/events_waits_summary_by_thread_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/events_waits_summary_by_thread_by_event_name.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/replication_group_member_stats.frm to /home/mysql/db_full_backup/performance_schema/replication_group_member_stats.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 >> log scanned up to (2966340)
180729 13:25:48 [01] Copying ./performance_schema/host_cache.frm to /home/mysql/db_full_backup/performance_schema/host_cache.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/setup_objects.frm to /home/mysql/db_full_backup/performance_schema/setup_objects.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/events_waits_history_long.frm to /home/mysql/db_full_backup/performance_schema/events_waits_history_long.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/session_status.frm to /home/mysql/db_full_backup/performance_schema/session_status.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/events_transactions_summary_by_host_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/events_transactions_summary_by_host_by_event_name.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/prepared_statements_instances.frm to /home/mysql/db_full_backup/performance_schema/prepared_statements_instances.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/setup_actors.frm to /home/mysql/db_full_backup/performance_schema/setup_actors.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/mutex_instances.frm to /home/mysql/db_full_backup/performance_schema/mutex_instances.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/table_lock_waits_summary_by_table.frm to /home/mysql/db_full_backup/performance_schema/table_lock_waits_summary_by_table.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/variables_by_thread.frm to /home/mysql/db_full_backup/performance_schema/variables_by_thread.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/events_waits_summary_by_host_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/events_waits_summary_by_host_by_event_name.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/events_statements_history_long.frm to /home/mysql/db_full_backup/performance_schema/events_statements_history_long.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/events_waits_summary_by_account_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/events_waits_summary_by_account_by_event_name.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/session_variables.frm to /home/mysql/db_full_backup/performance_schema/session_variables.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/events_stages_summary_by_thread_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/events_stages_summary_by_thread_by_event_name.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/file_summary_by_instance.frm to /home/mysql/db_full_backup/performance_schema/file_summary_by_instance.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/events_statements_history.frm to /home/mysql/db_full_backup/performance_schema/events_statements_history.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./performance_schema/table_io_waits_summary_by_index_usage.frm to /home/mysql/db_full_backup/performance_schema/table_io_waits_summary_by_index_usage.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./mysql/user.frm to /home/mysql/db_full_backup/mysql/user.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./mysql/func.MYI to /home/mysql/db_full_backup/mysql/func.MYI
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./mysql/help_topic.frm to /home/mysql/db_full_backup/mysql/help_topic.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./mysql/proxies_priv.MYI to /home/mysql/db_full_backup/mysql/proxies_priv.MYI
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./mysql/help_keyword.frm to /home/mysql/db_full_backup/mysql/help_keyword.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./mysql/func.frm to /home/mysql/db_full_backup/mysql/func.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./mysql/ndb_binlog_index.MYD to /home/mysql/db_full_backup/mysql/ndb_binlog_index.MYD
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./mysql/tables_priv.MYI to /home/mysql/db_full_backup/mysql/tables_priv.MYI
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./mysql/db.opt to /home/mysql/db_full_backup/mysql/db.opt
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./mysql/db.MYD to /home/mysql/db_full_backup/mysql/db.MYD
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./mysql/engine_cost.frm to /home/mysql/db_full_backup/mysql/engine_cost.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./mysql/ndb_binlog_index.MYI to /home/mysql/db_full_backup/mysql/ndb_binlog_index.MYI
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./mysql/procs_priv.MYI to /home/mysql/db_full_backup/mysql/procs_priv.MYI
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./mysql/help_category.frm to /home/mysql/db_full_backup/mysql/help_category.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./mysql/ndb_binlog_index.frm to /home/mysql/db_full_backup/mysql/ndb_binlog_index.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./mysql/user.MYI to /home/mysql/db_full_backup/mysql/user.MYI
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./mysql/event.MYI to /home/mysql/db_full_backup/mysql/event.MYI
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./mysql/tables_priv.frm to /home/mysql/db_full_backup/mysql/tables_priv.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./mysql/slave_master_info.frm to /home/mysql/db_full_backup/mysql/slave_master_info.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./mysql/time_zone_transition_type.frm to /home/mysql/db_full_backup/mysql/time_zone_transition_type.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./mysql/columns_priv.MYD to /home/mysql/db_full_backup/mysql/columns_priv.MYD
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./mysql/tables_priv.MYD to /home/mysql/db_full_backup/mysql/tables_priv.MYD
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./mysql/proxies_priv.frm to /home/mysql/db_full_backup/mysql/proxies_priv.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./mysql/event.frm to /home/mysql/db_full_backup/mysql/event.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./mysql/slow_log.CSV to /home/mysql/db_full_backup/mysql/slow_log.CSV
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./mysql/proc.MYI to /home/mysql/db_full_backup/mysql/proc.MYI
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./mysql/func.MYD to /home/mysql/db_full_backup/mysql/func.MYD
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./mysql/proc.frm to /home/mysql/db_full_backup/mysql/proc.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./mysql/slave_relay_log_info.frm to /home/mysql/db_full_backup/mysql/slave_relay_log_info.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./mysql/time_zone.frm to /home/mysql/db_full_backup/mysql/time_zone.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./mysql/db.MYI to /home/mysql/db_full_backup/mysql/db.MYI
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./mysql/gtid_executed.frm to /home/mysql/db_full_backup/mysql/gtid_executed.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./mysql/general_log.CSM to /home/mysql/db_full_backup/mysql/general_log.CSM
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./mysql/db.frm to /home/mysql/db_full_backup/mysql/db.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./mysql/plugin.frm to /home/mysql/db_full_backup/mysql/plugin.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./mysql/time_zone_leap_second.frm to /home/mysql/db_full_backup/mysql/time_zone_leap_second.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./mysql/general_log.frm to /home/mysql/db_full_backup/mysql/general_log.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./mysql/server_cost.frm to /home/mysql/db_full_backup/mysql/server_cost.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./mysql/servers.frm to /home/mysql/db_full_backup/mysql/servers.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./mysql/innodb_table_stats.frm to /home/mysql/db_full_backup/mysql/innodb_table_stats.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./mysql/columns_priv.frm to /home/mysql/db_full_backup/mysql/columns_priv.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./mysql/user.MYD to /home/mysql/db_full_backup/mysql/user.MYD
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./mysql/slave_worker_info.frm to /home/mysql/db_full_backup/mysql/slave_worker_info.frm
180729 13:25:48 [01]        ...done
180729 13:25:48 [01] Copying ./mysql/time_zone_transition.frm to /home/mysql/db_full_backup/mysql/time_zone_transition.frm
180729 13:25:48 [01]        ...done
180729 13:25:49 [01] Copying ./mysql/time_zone_name.frm to /home/mysql/db_full_backup/mysql/time_zone_name.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./mysql/slow_log.frm to /home/mysql/db_full_backup/mysql/slow_log.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./mysql/help_relation.frm to /home/mysql/db_full_backup/mysql/help_relation.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./mysql/proxies_priv.MYD to /home/mysql/db_full_backup/mysql/proxies_priv.MYD
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./mysql/event.MYD to /home/mysql/db_full_backup/mysql/event.MYD
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./mysql/slow_log.CSM to /home/mysql/db_full_backup/mysql/slow_log.CSM
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./mysql/innodb_index_stats.frm to /home/mysql/db_full_backup/mysql/innodb_index_stats.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./mysql/general_log.CSV to /home/mysql/db_full_backup/mysql/general_log.CSV
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./mysql/proc.MYD to /home/mysql/db_full_backup/mysql/proc.MYD
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./mysql/procs_priv.frm to /home/mysql/db_full_backup/mysql/procs_priv.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./mysql/procs_priv.MYD to /home/mysql/db_full_backup/mysql/procs_priv.MYD
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./mysql/columns_priv.MYI to /home/mysql/db_full_backup/mysql/columns_priv.MYI
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./dao/sal_grades.frm to /home/mysql/db_full_backup/dao/sal_grades.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./dao/db.opt to /home/mysql/db_full_backup/dao/db.opt
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./dao/penalties.frm to /home/mysql/db_full_backup/dao/penalties.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./dao/dept.frm to /home/mysql/db_full_backup/dao/dept.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./dao/matches.frm to /home/mysql/db_full_backup/dao/matches.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./dao/test_import.frm to /home/mysql/db_full_backup/dao/test_import.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./dao/test_tx_iso.frm to /home/mysql/db_full_backup/dao/test_tx_iso.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./dao/teams.frm to /home/mysql/db_full_backup/dao/teams.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./dao/committee_members.frm to /home/mysql/db_full_backup/dao/committee_members.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./dao/t1.frm to /home/mysql/db_full_backup/dao/t1.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./dao/test_binlog.frm to /home/mysql/db_full_backup/dao/test_binlog.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./dao/players.frm to /home/mysql/db_full_backup/dao/players.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./dao/emp.frm to /home/mysql/db_full_backup/dao/emp.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/x@0024innodb_buffer_stats_by_schema.frm to /home/mysql/db_full_backup/sys/x@0024innodb_buffer_stats_by_schema.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/x@0024statement_analysis.frm to /home/mysql/db_full_backup/sys/x@0024statement_analysis.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/x@0024user_summary.frm to /home/mysql/db_full_backup/sys/x@0024user_summary.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/x@0024schema_index_statistics.frm to /home/mysql/db_full_backup/sys/x@0024schema_index_statistics.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/x@0024statements_with_runtimes_in_95th_percentile.frm to /home/mysql/db_full_backup/sys/x@0024statements_with_runtimes_in_95th_percentile.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/x@0024innodb_buffer_stats_by_table.frm to /home/mysql/db_full_backup/sys/x@0024innodb_buffer_stats_by_table.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/db.opt to /home/mysql/db_full_backup/sys/db.opt
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/host_summary_by_statement_type.frm to /home/mysql/db_full_backup/sys/host_summary_by_statement_type.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/sys_config_insert_set_user.TRN to /home/mysql/db_full_backup/sys/sys_config_insert_set_user.TRN
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/statement_analysis.frm to /home/mysql/db_full_backup/sys/statement_analysis.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/version.frm to /home/mysql/db_full_backup/sys/version.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/host_summary.frm to /home/mysql/db_full_backup/sys/host_summary.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/x@0024io_global_by_wait_by_latency.frm to /home/mysql/db_full_backup/sys/x@0024io_global_by_wait_by_latency.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/statements_with_sorting.frm to /home/mysql/db_full_backup/sys/statements_with_sorting.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/sys_config.frm to /home/mysql/db_full_backup/sys/sys_config.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 >> log scanned up to (2966340)
180729 13:25:49 [01] Copying ./sys/x@0024io_by_thread_by_latency.frm to /home/mysql/db_full_backup/sys/x@0024io_by_thread_by_latency.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/memory_by_host_by_current_bytes.frm to /home/mysql/db_full_backup/sys/memory_by_host_by_current_bytes.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/x@0024memory_by_thread_by_current_bytes.frm to /home/mysql/db_full_backup/sys/x@0024memory_by_thread_by_current_bytes.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/x@0024schema_flattened_keys.frm to /home/mysql/db_full_backup/sys/x@0024schema_flattened_keys.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/x@0024memory_by_host_by_current_bytes.frm to /home/mysql/db_full_backup/sys/x@0024memory_by_host_by_current_bytes.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/sys_config.TRG to /home/mysql/db_full_backup/sys/sys_config.TRG
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/user_summary_by_file_io.frm to /home/mysql/db_full_backup/sys/user_summary_by_file_io.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/x@0024latest_file_io.frm to /home/mysql/db_full_backup/sys/x@0024latest_file_io.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/schema_object_overview.frm to /home/mysql/db_full_backup/sys/schema_object_overview.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/schema_unused_indexes.frm to /home/mysql/db_full_backup/sys/schema_unused_indexes.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/x@0024processlist.frm to /home/mysql/db_full_backup/sys/x@0024processlist.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/statements_with_full_table_scans.frm to /home/mysql/db_full_backup/sys/statements_with_full_table_scans.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/x@0024host_summary_by_file_io.frm to /home/mysql/db_full_backup/sys/x@0024host_summary_by_file_io.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/x@0024host_summary.frm to /home/mysql/db_full_backup/sys/x@0024host_summary.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/memory_by_user_by_current_bytes.frm to /home/mysql/db_full_backup/sys/memory_by_user_by_current_bytes.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/wait_classes_global_by_latency.frm to /home/mysql/db_full_backup/sys/wait_classes_global_by_latency.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/x@0024io_global_by_wait_by_bytes.frm to /home/mysql/db_full_backup/sys/x@0024io_global_by_wait_by_bytes.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/x@0024ps_schema_table_statistics_io.frm to /home/mysql/db_full_backup/sys/x@0024ps_schema_table_statistics_io.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/x@0024schema_table_statistics_with_buffer.frm to /home/mysql/db_full_backup/sys/x@0024schema_table_statistics_with_buffer.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/session_ssl_status.frm to /home/mysql/db_full_backup/sys/session_ssl_status.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/innodb_buffer_stats_by_schema.frm to /home/mysql/db_full_backup/sys/innodb_buffer_stats_by_schema.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/x@0024waits_by_user_by_latency.frm to /home/mysql/db_full_backup/sys/x@0024waits_by_user_by_latency.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/x@0024host_summary_by_stages.frm to /home/mysql/db_full_backup/sys/x@0024host_summary_by_stages.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/x@0024waits_global_by_latency.frm to /home/mysql/db_full_backup/sys/x@0024waits_global_by_latency.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/schema_tables_with_full_table_scans.frm to /home/mysql/db_full_backup/sys/schema_tables_with_full_table_scans.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/x@0024schema_table_lock_waits.frm to /home/mysql/db_full_backup/sys/x@0024schema_table_lock_waits.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/x@0024memory_by_user_by_current_bytes.frm to /home/mysql/db_full_backup/sys/x@0024memory_by_user_by_current_bytes.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/innodb_lock_waits.frm to /home/mysql/db_full_backup/sys/innodb_lock_waits.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/x@0024ps_digest_avg_latency_distribution.frm to /home/mysql/db_full_backup/sys/x@0024ps_digest_avg_latency_distribution.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/x@0024schema_table_statistics.frm to /home/mysql/db_full_backup/sys/x@0024schema_table_statistics.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/x@0024host_summary_by_file_io_type.frm to /home/mysql/db_full_backup/sys/x@0024host_summary_by_file_io_type.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/latest_file_io.frm to /home/mysql/db_full_backup/sys/latest_file_io.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/schema_redundant_indexes.frm to /home/mysql/db_full_backup/sys/schema_redundant_indexes.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/memory_global_by_current_bytes.frm to /home/mysql/db_full_backup/sys/memory_global_by_current_bytes.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/x@0024memory_global_by_current_bytes.frm to /home/mysql/db_full_backup/sys/x@0024memory_global_by_current_bytes.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/statements_with_errors_or_warnings.frm to /home/mysql/db_full_backup/sys/statements_with_errors_or_warnings.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/x@0024wait_classes_global_by_avg_latency.frm to /home/mysql/db_full_backup/sys/x@0024wait_classes_global_by_avg_latency.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/x@0024wait_classes_global_by_latency.frm to /home/mysql/db_full_backup/sys/x@0024wait_classes_global_by_latency.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/x@0024session.frm to /home/mysql/db_full_backup/sys/x@0024session.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/x@0024schema_tables_with_full_table_scans.frm to /home/mysql/db_full_backup/sys/x@0024schema_tables_with_full_table_scans.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/host_summary_by_file_io_type.frm to /home/mysql/db_full_backup/sys/host_summary_by_file_io_type.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/user_summary.frm to /home/mysql/db_full_backup/sys/user_summary.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/x@0024user_summary_by_stages.frm to /home/mysql/db_full_backup/sys/x@0024user_summary_by_stages.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/user_summary_by_file_io_type.frm to /home/mysql/db_full_backup/sys/user_summary_by_file_io_type.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/schema_table_statistics.frm to /home/mysql/db_full_backup/sys/schema_table_statistics.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/waits_by_user_by_latency.frm to /home/mysql/db_full_backup/sys/waits_by_user_by_latency.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/io_global_by_wait_by_bytes.frm to /home/mysql/db_full_backup/sys/io_global_by_wait_by_bytes.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/user_summary_by_statement_type.frm to /home/mysql/db_full_backup/sys/user_summary_by_statement_type.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/x@0024statements_with_sorting.frm to /home/mysql/db_full_backup/sys/x@0024statements_with_sorting.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/user_summary_by_statement_latency.frm to /home/mysql/db_full_backup/sys/user_summary_by_statement_latency.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/x@0024user_summary_by_statement_latency.frm to /home/mysql/db_full_backup/sys/x@0024user_summary_by_statement_latency.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/session.frm to /home/mysql/db_full_backup/sys/session.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/schema_table_statistics_with_buffer.frm to /home/mysql/db_full_backup/sys/schema_table_statistics_with_buffer.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/host_summary_by_file_io.frm to /home/mysql/db_full_backup/sys/host_summary_by_file_io.frm
180729 13:25:49 [01]        ...done
180729 13:25:49 [01] Copying ./sys/memory_by_thread_by_current_bytes.frm to /home/mysql/db_full_backup/sys/memory_by_thread_by_current_bytes.frm
180729 13:25:49 [01]        ...done
180729 13:25:50 [01] Copying ./sys/x@0024waits_by_host_by_latency.frm to /home/mysql/db_full_backup/sys/x@0024waits_by_host_by_latency.frm
180729 13:25:50 [01]        ...done
180729 13:25:50 [01] Copying ./sys/schema_auto_increment_columns.frm to /home/mysql/db_full_backup/sys/schema_auto_increment_columns.frm
180729 13:25:50 [01]        ...done
180729 13:25:50 [01] Copying ./sys/x@0024ps_digest_95th_percentile_by_avg_us.frm to /home/mysql/db_full_backup/sys/x@0024ps_digest_95th_percentile_by_avg_us.frm
180729 13:25:50 [01]        ...done
180729 13:25:50 [01] Copying ./sys/waits_by_host_by_latency.frm to /home/mysql/db_full_backup/sys/waits_by_host_by_latency.frm
180729 13:25:50 [01]        ...done
180729 13:25:50 [01] Copying ./sys/host_summary_by_statement_latency.frm to /home/mysql/db_full_backup/sys/host_summary_by_statement_latency.frm
180729 13:25:50 [01]        ...done
180729 13:25:50 [01] Copying ./sys/metrics.frm to /home/mysql/db_full_backup/sys/metrics.frm
180729 13:25:50 [01]        ...done
180729 13:25:50 [01] Copying ./sys/x@0024user_summary_by_file_io_type.frm to /home/mysql/db_full_backup/sys/x@0024user_summary_by_file_io_type.frm
180729 13:25:50 [01]        ...done
180729 13:25:50 [01] Copying ./sys/io_global_by_file_by_bytes.frm to /home/mysql/db_full_backup/sys/io_global_by_file_by_bytes.frm
180729 13:25:50 [01]        ...done
180729 13:25:50 [01] Copying ./sys/io_by_thread_by_latency.frm to /home/mysql/db_full_backup/sys/io_by_thread_by_latency.frm
180729 13:25:50 [01]        ...done
180729 13:25:50 [01] Copying ./sys/processlist.frm to /home/mysql/db_full_backup/sys/processlist.frm
180729 13:25:50 [01]        ...done
180729 13:25:50 [01] Copying ./sys/statements_with_runtimes_in_95th_percentile.frm to /home/mysql/db_full_backup/sys/statements_with_runtimes_in_95th_percentile.frm
180729 13:25:50 [01]        ...done
180729 13:25:50 [01] Copying ./sys/schema_table_lock_waits.frm to /home/mysql/db_full_backup/sys/schema_table_lock_waits.frm
180729 13:25:50 [01]        ...done
180729 13:25:50 [01] Copying ./sys/wait_classes_global_by_avg_latency.frm to /home/mysql/db_full_backup/sys/wait_classes_global_by_avg_latency.frm
180729 13:25:50 [01]        ...done
180729 13:25:50 [01] Copying ./sys/user_summary_by_stages.frm to /home/mysql/db_full_backup/sys/user_summary_by_stages.frm
180729 13:25:50 [01]        ...done
180729 13:25:50 [01] Copying ./sys/io_global_by_file_by_latency.frm to /home/mysql/db_full_backup/sys/io_global_by_file_by_latency.frm
180729 13:25:50 [01]        ...done
180729 13:25:50 [01] Copying ./sys/x@0024host_summary_by_statement_type.frm to /home/mysql/db_full_backup/sys/x@0024host_summary_by_statement_type.frm
180729 13:25:50 [01]        ...done
180729 13:25:50 [01] Copying ./sys/x@0024innodb_lock_waits.frm to /home/mysql/db_full_backup/sys/x@0024innodb_lock_waits.frm
180729 13:25:50 [01]        ...done
180729 13:25:50 [01] Copying ./sys/ps_check_lost_instrumentation.frm to /home/mysql/db_full_backup/sys/ps_check_lost_instrumentation.frm
180729 13:25:50 [01]        ...done
180729 13:25:50 [01] Copying ./sys/waits_global_by_latency.frm to /home/mysql/db_full_backup/sys/waits_global_by_latency.frm
180729 13:25:50 [01]        ...done
180729 13:25:50 [01] Copying ./sys/sys_config_update_set_user.TRN to /home/mysql/db_full_backup/sys/sys_config_update_set_user.TRN
180729 13:25:50 [01]        ...done
180729 13:25:50 [01] Copying ./sys/innodb_buffer_stats_by_table.frm to /home/mysql/db_full_backup/sys/innodb_buffer_stats_by_table.frm
180729 13:25:50 [01]        ...done
180729 13:25:50 [01] Copying ./sys/x@0024statements_with_temp_tables.frm to /home/mysql/db_full_backup/sys/x@0024statements_with_temp_tables.frm
180729 13:25:50 [01]        ...done
180729 13:25:50 [01] Copying ./sys/io_global_by_wait_by_latency.frm to /home/mysql/db_full_backup/sys/io_global_by_wait_by_latency.frm
180729 13:25:50 [01]        ...done
180729 13:25:50 [01] Copying ./sys/host_summary_by_stages.frm to /home/mysql/db_full_backup/sys/host_summary_by_stages.frm
180729 13:25:50 [01]        ...done
180729 13:25:50 [01] Copying ./sys/statements_with_temp_tables.frm to /home/mysql/db_full_backup/sys/statements_with_temp_tables.frm
180729 13:25:50 [01]        ...done
180729 13:25:50 [01] Copying ./sys/x@0024user_summary_by_file_io.frm to /home/mysql/db_full_backup/sys/x@0024user_summary_by_file_io.frm
180729 13:25:50 [01]        ...done
180729 13:25:50 [01] Copying ./sys/schema_index_statistics.frm to /home/mysql/db_full_backup/sys/schema_index_statistics.frm
180729 13:25:50 [01]        ...done
180729 13:25:50 [01] Copying ./sys/x@0024host_summary_by_statement_latency.frm to /home/mysql/db_full_backup/sys/x@0024host_summary_by_statement_latency.frm
180729 13:25:50 [01]        ...done
180729 13:25:50 [01] Copying ./sys/x@0024user_summary_by_statement_type.frm to /home/mysql/db_full_backup/sys/x@0024user_summary_by_statement_type.frm
180729 13:25:50 [01]        ...done
180729 13:25:50 [01] Copying ./sys/x@0024statements_with_errors_or_warnings.frm to /home/mysql/db_full_backup/sys/x@0024statements_with_errors_or_warnings.frm
180729 13:25:50 [01]        ...done
180729 13:25:50 [01] Copying ./sys/x@0024io_global_by_file_by_latency.frm to /home/mysql/db_full_backup/sys/x@0024io_global_by_file_by_latency.frm
180729 13:25:50 [01]        ...done
180729 13:25:50 [01] Copying ./sys/x@0024io_global_by_file_by_bytes.frm to /home/mysql/db_full_backup/sys/x@0024io_global_by_file_by_bytes.frm
180729 13:25:50 [01]        ...done
180729 13:25:50 [01] Copying ./sys/x@0024memory_global_total.frm to /home/mysql/db_full_backup/sys/x@0024memory_global_total.frm
180729 13:25:50 [01]        ...done
180729 13:25:50 [01] Copying ./sys/memory_global_total.frm to /home/mysql/db_full_backup/sys/memory_global_total.frm
180729 13:25:50 [01]        ...done
180729 13:25:50 [01] Copying ./sys/x@0024statements_with_full_table_scans.frm to /home/mysql/db_full_backup/sys/x@0024statements_with_full_table_scans.frm
180729 13:25:50 [01]        ...done
180729 13:25:50 Finished backing up non-InnoDB tables and files
180729 13:25:50 [00] Writing /home/mysql/db_full_backup/xtrabackup_binlog_info
180729 13:25:50 [00]        ...done
180729 13:25:50 Executing FLUSH NO_WRITE_TO_BINLOG ENGINE LOGS...
xtrabackup: The latest check point (for incremental): '2966331'
xtrabackup: Stopping log copying thread.
.180729 13:25:50 >> log scanned up to (2966340)

180729 13:25:50 Executing UNLOCK TABLES
180729 13:25:50 All tables unlocked
180729 13:25:50 [00] Copying ib_buffer_pool to /home/mysql/db_full_backup/ib_buffer_pool
180729 13:25:50 [00]        ...done
180729 13:25:50 Backup created in directory '/home/mysql/db_full_backup/'
MySQL binlog position: filename 'mysql-bin.000005', position '2523'
180729 13:25:50 [00] Writing /home/mysql/db_full_backup/backup-my.cnf
180729 13:25:50 [00]        ...done
180729 13:25:50 [00] Writing /home/mysql/db_full_backup/xtrabackup_info
180729 13:25:50 [00]        ...done
xtrabackup: Transaction log of lsn (2966331) to (2966340) was copied.
180729 13:25:50 completed OK!


```

### 第二次插入数据

```sql



mysql> insert into t1 values (2) ;
Query OK, 1 row affected (0.02 sec)

mysql> select  *  from t1 ;
+------+
| c1   |
+------+
|    1 |
|    2 |
+------+
2 rows in set (0.00 sec)



```

### 进行第一次增量备份

```sql
[mysql@bosenrui ~]$ innobackupex  --defaults-file=/mysql1/mysql1.cnf -u root -p root --no-timestamp --incremental --incremental-basedir=/home/mysql/db_full_backup  /home/mysql/db_incr_backup1
180729 13:33:00 innobackupex: Starting the backup operation

IMPORTANT: Please check that the backup run completes successfully.
           At the end of a successful backup run innobackupex
           prints "completed OK!".

180729 13:33:00  version_check Connecting to MySQL server with DSN 'dbi:mysql:;mysql_read_default_group=xtrabackup;port=3306;mysql_socket=/tmp/mysql1.sock' as 'root'  (using password: YES).
180729 13:33:00  version_check Connected to MySQL server
180729 13:33:00  version_check Executing a version check against the server...
180729 13:33:00  version_check Done.
180729 13:33:00 Connecting to MySQL server host: localhost, user: root, password: set, port: 3306, socket: /tmp/mysql1.sock
Using server version 5.7.21-log
innobackupex version 2.4.8 based on MySQL server 5.7.13 Linux (x86_64) (revision id: 97330f7)
incremental backup from 2966331 is enabled.
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
180729 13:33:00 >> log scanned up to (2966684)
xtrabackup: Generating a list of tablespaces
InnoDB: Allocated tablespace ID 8 for mysql/time_zone_name, old maximum was 0
xtrabackup: using the full scan for incremental backup
180729 13:33:01 [01] Copying ./ibdata1 to /home/mysql/db_incr_backup1/ibdata1.delta
180729 13:33:01 [01]        ...done
180729 13:33:01 [01] Copying ./mysql/time_zone_name.ibd to /home/mysql/db_incr_backup1/mysql/time_zone_name.ibd.delta
180729 13:33:01 [01]        ...done
180729 13:33:01 [01] Copying ./mysql/engine_cost.ibd to /home/mysql/db_incr_backup1/mysql/engine_cost.ibd.delta
180729 13:33:01 [01]        ...done
180729 13:33:01 [01] Copying ./mysql/help_topic.ibd to /home/mysql/db_incr_backup1/mysql/help_topic.ibd.delta
180729 13:33:01 [01]        ...done
180729 13:33:01 >> log scanned up to (2966684)
180729 13:33:01 [01] Copying ./mysql/server_cost.ibd to /home/mysql/db_incr_backup1/mysql/server_cost.ibd.delta
180729 13:33:01 [01]        ...done
180729 13:33:01 [01] Copying ./mysql/help_category.ibd to /home/mysql/db_incr_backup1/mysql/help_category.ibd.delta
180729 13:33:01 [01]        ...done
180729 13:33:01 [01] Copying ./mysql/innodb_index_stats.ibd to /home/mysql/db_incr_backup1/mysql/innodb_index_stats.ibd.delta
180729 13:33:01 [01]        ...done
180729 13:33:01 [01] Copying ./mysql/innodb_table_stats.ibd to /home/mysql/db_incr_backup1/mysql/innodb_table_stats.ibd.delta
180729 13:33:01 [01]        ...done
180729 13:33:01 [01] Copying ./mysql/time_zone_transition_type.ibd to /home/mysql/db_incr_backup1/mysql/time_zone_transition_type.ibd.delta
180729 13:33:01 [01]        ...done
180729 13:33:01 [01] Copying ./mysql/slave_worker_info.ibd to /home/mysql/db_incr_backup1/mysql/slave_worker_info.ibd.delta
180729 13:33:01 [01]        ...done
180729 13:33:01 [01] Copying ./mysql/time_zone_transition.ibd to /home/mysql/db_incr_backup1/mysql/time_zone_transition.ibd.delta
180729 13:33:01 [01]        ...done
180729 13:33:01 [01] Copying ./mysql/time_zone_leap_second.ibd to /home/mysql/db_incr_backup1/mysql/time_zone_leap_second.ibd.delta
180729 13:33:01 [01]        ...done
180729 13:33:02 [01] Copying ./mysql/servers.ibd to /home/mysql/db_incr_backup1/mysql/servers.ibd.delta
180729 13:33:02 [01]        ...done
180729 13:33:02 [01] Copying ./mysql/gtid_executed.ibd to /home/mysql/db_incr_backup1/mysql/gtid_executed.ibd.delta
180729 13:33:02 [01]        ...done
180729 13:33:02 [01] Copying ./mysql/time_zone.ibd to /home/mysql/db_incr_backup1/mysql/time_zone.ibd.delta
180729 13:33:02 [01]        ...done
180729 13:33:02 [01] Copying ./mysql/help_relation.ibd to /home/mysql/db_incr_backup1/mysql/help_relation.ibd.delta
180729 13:33:02 [01]        ...done
180729 13:33:02 [01] Copying ./mysql/plugin.ibd to /home/mysql/db_incr_backup1/mysql/plugin.ibd.delta
180729 13:33:02 [01]        ...done
180729 13:33:02 [01] Copying ./mysql/slave_master_info.ibd to /home/mysql/db_incr_backup1/mysql/slave_master_info.ibd.delta
180729 13:33:02 [01]        ...done
180729 13:33:02 [01] Copying ./mysql/help_keyword.ibd to /home/mysql/db_incr_backup1/mysql/help_keyword.ibd.delta
180729 13:33:02 [01]        ...done
180729 13:33:02 [01] Copying ./mysql/slave_relay_log_info.ibd to /home/mysql/db_incr_backup1/mysql/slave_relay_log_info.ibd.delta
180729 13:33:02 [01]        ...done
180729 13:33:02 [01] Copying ./dao/emp.ibd to /home/mysql/db_incr_backup1/dao/emp.ibd.delta
180729 13:33:02 [01]        ...done
180729 13:33:02 [01] Copying ./dao/test_tx_iso.ibd to /home/mysql/db_incr_backup1/dao/test_tx_iso.ibd.delta
180729 13:33:02 [01]        ...done
180729 13:33:02 [01] Copying ./dao/matches.ibd to /home/mysql/db_incr_backup1/dao/matches.ibd.delta
180729 13:33:02 [01]        ...done
180729 13:33:02 [01] Copying ./dao/test_binlog.ibd to /home/mysql/db_incr_backup1/dao/test_binlog.ibd.delta
180729 13:33:02 [01]        ...done
180729 13:33:02 [01] Copying ./dao/players.ibd to /home/mysql/db_incr_backup1/dao/players.ibd.delta
180729 13:33:02 [01]        ...done
180729 13:33:02 [01] Copying ./dao/sal_grades.ibd to /home/mysql/db_incr_backup1/dao/sal_grades.ibd.delta
180729 13:33:02 [01]        ...done
180729 13:33:02 [01] Copying ./dao/test_import.ibd to /home/mysql/db_incr_backup1/dao/test_import.ibd.delta
180729 13:33:02 [01]        ...done
180729 13:33:02 [01] Copying ./dao/penalties.ibd to /home/mysql/db_incr_backup1/dao/penalties.ibd.delta
180729 13:33:02 [01]        ...done
180729 13:33:02 [01] Copying ./dao/committee_members.ibd to /home/mysql/db_incr_backup1/dao/committee_members.ibd.delta
180729 13:33:02 [01]        ...done
180729 13:33:02 [01] Copying ./dao/teams.ibd to /home/mysql/db_incr_backup1/dao/teams.ibd.delta
180729 13:33:02 [01]        ...done
180729 13:33:02 [01] Copying ./dao/t1.ibd to /home/mysql/db_incr_backup1/dao/t1.ibd.delta
180729 13:33:02 [01]        ...done
180729 13:33:02 [01] Copying ./dao/dept.ibd to /home/mysql/db_incr_backup1/dao/dept.ibd.delta
180729 13:33:02 [01]        ...done
180729 13:33:02 [01] Copying ./sys/sys_config.ibd to /home/mysql/db_incr_backup1/sys/sys_config.ibd.delta
180729 13:33:02 [01]        ...done
180729 13:33:02 >> log scanned up to (2966684)
180729 13:33:03 Executing FLUSH NO_WRITE_TO_BINLOG TABLES...
180729 13:33:03 Executing FLUSH TABLES WITH READ LOCK...
180729 13:33:03 Starting to backup non-InnoDB tables and files
180729 13:33:03 [01] Copying ./performance_schema/replication_applier_status.frm to /home/mysql/db_incr_backup1/performance_schema/replication_applier_status.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/file_instances.frm to /home/mysql/db_incr_backup1/performance_schema/file_instances.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/metadata_locks.frm to /home/mysql/db_incr_backup1/performance_schema/metadata_locks.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/memory_summary_by_account_by_event_name.frm to /home/mysql/db_incr_backup1/performance_schema/memory_summary_by_account_by_event_name.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/threads.frm to /home/mysql/db_incr_backup1/performance_schema/threads.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/events_waits_summary_by_user_by_event_name.frm to /home/mysql/db_incr_backup1/performance_schema/events_waits_summary_by_user_by_event_name.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/events_stages_summary_by_account_by_event_name.frm to /home/mysql/db_incr_backup1/performance_schema/events_stages_summary_by_account_by_event_name.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/db.opt to /home/mysql/db_incr_backup1/performance_schema/db.opt
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/table_io_waits_summary_by_table.frm to /home/mysql/db_incr_backup1/performance_schema/table_io_waits_summary_by_table.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/memory_summary_global_by_event_name.frm to /home/mysql/db_incr_backup1/performance_schema/memory_summary_global_by_event_name.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/events_stages_history_long.frm to /home/mysql/db_incr_backup1/performance_schema/events_stages_history_long.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/events_statements_summary_by_program.frm to /home/mysql/db_incr_backup1/performance_schema/events_statements_summary_by_program.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/events_statements_summary_by_user_by_event_name.frm to /home/mysql/db_incr_backup1/performance_schema/events_statements_summary_by_user_by_event_name.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/session_account_connect_attrs.frm to /home/mysql/db_incr_backup1/performance_schema/session_account_connect_attrs.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/session_connect_attrs.frm to /home/mysql/db_incr_backup1/performance_schema/session_connect_attrs.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/events_statements_summary_by_account_by_event_name.frm to /home/mysql/db_incr_backup1/performance_schema/events_statements_summary_by_account_by_event_name.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/events_statements_summary_by_thread_by_event_name.frm to /home/mysql/db_incr_backup1/performance_schema/events_statements_summary_by_thread_by_event_name.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/events_transactions_history.frm to /home/mysql/db_incr_backup1/performance_schema/events_transactions_history.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/memory_summary_by_user_by_event_name.frm to /home/mysql/db_incr_backup1/performance_schema/memory_summary_by_user_by_event_name.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/events_statements_summary_global_by_event_name.frm to /home/mysql/db_incr_backup1/performance_schema/events_statements_summary_global_by_event_name.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/performance_timers.frm to /home/mysql/db_incr_backup1/performance_schema/performance_timers.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/setup_consumers.frm to /home/mysql/db_incr_backup1/performance_schema/setup_consumers.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/events_stages_summary_by_host_by_event_name.frm to /home/mysql/db_incr_backup1/performance_schema/events_stages_summary_by_host_by_event_name.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/events_transactions_history_long.frm to /home/mysql/db_incr_backup1/performance_schema/events_transactions_history_long.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/replication_connection_status.frm to /home/mysql/db_incr_backup1/performance_schema/replication_connection_status.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/replication_group_members.frm to /home/mysql/db_incr_backup1/performance_schema/replication_group_members.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/objects_summary_global_by_type.frm to /home/mysql/db_incr_backup1/performance_schema/objects_summary_global_by_type.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/rwlock_instances.frm to /home/mysql/db_incr_backup1/performance_schema/rwlock_instances.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/user_variables_by_thread.frm to /home/mysql/db_incr_backup1/performance_schema/user_variables_by_thread.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/events_waits_current.frm to /home/mysql/db_incr_backup1/performance_schema/events_waits_current.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/file_summary_by_event_name.frm to /home/mysql/db_incr_backup1/performance_schema/file_summary_by_event_name.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/users.frm to /home/mysql/db_incr_backup1/performance_schema/users.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/replication_applier_status_by_worker.frm to /home/mysql/db_incr_backup1/performance_schema/replication_applier_status_by_worker.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/socket_instances.frm to /home/mysql/db_incr_backup1/performance_schema/socket_instances.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/events_stages_current.frm to /home/mysql/db_incr_backup1/performance_schema/events_stages_current.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/global_status.frm to /home/mysql/db_incr_backup1/performance_schema/global_status.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/setup_timers.frm to /home/mysql/db_incr_backup1/performance_schema/setup_timers.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/events_waits_history.frm to /home/mysql/db_incr_backup1/performance_schema/events_waits_history.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/table_handles.frm to /home/mysql/db_incr_backup1/performance_schema/table_handles.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/events_statements_current.frm to /home/mysql/db_incr_backup1/performance_schema/events_statements_current.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/events_transactions_current.frm to /home/mysql/db_incr_backup1/performance_schema/events_transactions_current.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/events_waits_summary_global_by_event_name.frm to /home/mysql/db_incr_backup1/performance_schema/events_waits_summary_global_by_event_name.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/replication_applier_status_by_coordinator.frm to /home/mysql/db_incr_backup1/performance_schema/replication_applier_status_by_coordinator.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/events_statements_summary_by_host_by_event_name.frm to /home/mysql/db_incr_backup1/performance_schema/events_statements_summary_by_host_by_event_name.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/events_stages_history.frm to /home/mysql/db_incr_backup1/performance_schema/events_stages_history.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/events_statements_summary_by_digest.frm to /home/mysql/db_incr_backup1/performance_schema/events_statements_summary_by_digest.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/accounts.frm to /home/mysql/db_incr_backup1/performance_schema/accounts.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/replication_applier_configuration.frm to /home/mysql/db_incr_backup1/performance_schema/replication_applier_configuration.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/replication_connection_configuration.frm to /home/mysql/db_incr_backup1/performance_schema/replication_connection_configuration.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/status_by_user.frm to /home/mysql/db_incr_backup1/performance_schema/status_by_user.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/events_transactions_summary_global_by_event_name.frm to /home/mysql/db_incr_backup1/performance_schema/events_transactions_summary_global_by_event_name.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/events_transactions_summary_by_account_by_event_name.frm to /home/mysql/db_incr_backup1/performance_schema/events_transactions_summary_by_account_by_event_name.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/events_stages_summary_global_by_event_name.frm to /home/mysql/db_incr_backup1/performance_schema/events_stages_summary_global_by_event_name.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/socket_summary_by_event_name.frm to /home/mysql/db_incr_backup1/performance_schema/socket_summary_by_event_name.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/memory_summary_by_thread_by_event_name.frm to /home/mysql/db_incr_backup1/performance_schema/memory_summary_by_thread_by_event_name.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/status_by_thread.frm to /home/mysql/db_incr_backup1/performance_schema/status_by_thread.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/events_waits_summary_by_instance.frm to /home/mysql/db_incr_backup1/performance_schema/events_waits_summary_by_instance.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/events_transactions_summary_by_thread_by_event_name.frm to /home/mysql/db_incr_backup1/performance_schema/events_transactions_summary_by_thread_by_event_name.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/hosts.frm to /home/mysql/db_incr_backup1/performance_schema/hosts.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/cond_instances.frm to /home/mysql/db_incr_backup1/performance_schema/cond_instances.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/memory_summary_by_host_by_event_name.frm to /home/mysql/db_incr_backup1/performance_schema/memory_summary_by_host_by_event_name.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/status_by_account.frm to /home/mysql/db_incr_backup1/performance_schema/status_by_account.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/socket_summary_by_instance.frm to /home/mysql/db_incr_backup1/performance_schema/socket_summary_by_instance.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/events_transactions_summary_by_user_by_event_name.frm to /home/mysql/db_incr_backup1/performance_schema/events_transactions_summary_by_user_by_event_name.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/status_by_host.frm to /home/mysql/db_incr_backup1/performance_schema/status_by_host.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/events_stages_summary_by_user_by_event_name.frm to /home/mysql/db_incr_backup1/performance_schema/events_stages_summary_by_user_by_event_name.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/global_variables.frm to /home/mysql/db_incr_backup1/performance_schema/global_variables.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/setup_instruments.frm to /home/mysql/db_incr_backup1/performance_schema/setup_instruments.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/events_waits_summary_by_thread_by_event_name.frm to /home/mysql/db_incr_backup1/performance_schema/events_waits_summary_by_thread_by_event_name.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/replication_group_member_stats.frm to /home/mysql/db_incr_backup1/performance_schema/replication_group_member_stats.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/host_cache.frm to /home/mysql/db_incr_backup1/performance_schema/host_cache.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/setup_objects.frm to /home/mysql/db_incr_backup1/performance_schema/setup_objects.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/events_waits_history_long.frm to /home/mysql/db_incr_backup1/performance_schema/events_waits_history_long.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/session_status.frm to /home/mysql/db_incr_backup1/performance_schema/session_status.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/events_transactions_summary_by_host_by_event_name.frm to /home/mysql/db_incr_backup1/performance_schema/events_transactions_summary_by_host_by_event_name.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/prepared_statements_instances.frm to /home/mysql/db_incr_backup1/performance_schema/prepared_statements_instances.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/setup_actors.frm to /home/mysql/db_incr_backup1/performance_schema/setup_actors.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/mutex_instances.frm to /home/mysql/db_incr_backup1/performance_schema/mutex_instances.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/table_lock_waits_summary_by_table.frm to /home/mysql/db_incr_backup1/performance_schema/table_lock_waits_summary_by_table.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/variables_by_thread.frm to /home/mysql/db_incr_backup1/performance_schema/variables_by_thread.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 >> log scanned up to (2966684)
180729 13:33:03 [01] Copying ./performance_schema/events_waits_summary_by_host_by_event_name.frm to /home/mysql/db_incr_backup1/performance_schema/events_waits_summary_by_host_by_event_name.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/events_statements_history_long.frm to /home/mysql/db_incr_backup1/performance_schema/events_statements_history_long.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/events_waits_summary_by_account_by_event_name.frm to /home/mysql/db_incr_backup1/performance_schema/events_waits_summary_by_account_by_event_name.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/session_variables.frm to /home/mysql/db_incr_backup1/performance_schema/session_variables.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/events_stages_summary_by_thread_by_event_name.frm to /home/mysql/db_incr_backup1/performance_schema/events_stages_summary_by_thread_by_event_name.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/file_summary_by_instance.frm to /home/mysql/db_incr_backup1/performance_schema/file_summary_by_instance.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/events_statements_history.frm to /home/mysql/db_incr_backup1/performance_schema/events_statements_history.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./performance_schema/table_io_waits_summary_by_index_usage.frm to /home/mysql/db_incr_backup1/performance_schema/table_io_waits_summary_by_index_usage.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./mysql/user.frm to /home/mysql/db_incr_backup1/mysql/user.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./mysql/func.MYI to /home/mysql/db_incr_backup1/mysql/func.MYI
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./mysql/help_topic.frm to /home/mysql/db_incr_backup1/mysql/help_topic.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./mysql/proxies_priv.MYI to /home/mysql/db_incr_backup1/mysql/proxies_priv.MYI
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./mysql/help_keyword.frm to /home/mysql/db_incr_backup1/mysql/help_keyword.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./mysql/func.frm to /home/mysql/db_incr_backup1/mysql/func.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./mysql/ndb_binlog_index.MYD to /home/mysql/db_incr_backup1/mysql/ndb_binlog_index.MYD
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./mysql/tables_priv.MYI to /home/mysql/db_incr_backup1/mysql/tables_priv.MYI
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./mysql/db.opt to /home/mysql/db_incr_backup1/mysql/db.opt
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./mysql/db.MYD to /home/mysql/db_incr_backup1/mysql/db.MYD
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./mysql/engine_cost.frm to /home/mysql/db_incr_backup1/mysql/engine_cost.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./mysql/ndb_binlog_index.MYI to /home/mysql/db_incr_backup1/mysql/ndb_binlog_index.MYI
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./mysql/procs_priv.MYI to /home/mysql/db_incr_backup1/mysql/procs_priv.MYI
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./mysql/help_category.frm to /home/mysql/db_incr_backup1/mysql/help_category.frm
180729 13:33:03 [01]        ...done
180729 13:33:03 [01] Copying ./mysql/ndb_binlog_index.frm to /home/mysql/db_incr_backup1/mysql/ndb_binlog_index.frm
180729 13:33:03 [01]        ...done
180729 13:33:04 [01] Copying ./mysql/user.MYI to /home/mysql/db_incr_backup1/mysql/user.MYI
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./mysql/event.MYI to /home/mysql/db_incr_backup1/mysql/event.MYI
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./mysql/tables_priv.frm to /home/mysql/db_incr_backup1/mysql/tables_priv.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./mysql/slave_master_info.frm to /home/mysql/db_incr_backup1/mysql/slave_master_info.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./mysql/time_zone_transition_type.frm to /home/mysql/db_incr_backup1/mysql/time_zone_transition_type.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./mysql/columns_priv.MYD to /home/mysql/db_incr_backup1/mysql/columns_priv.MYD
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./mysql/tables_priv.MYD to /home/mysql/db_incr_backup1/mysql/tables_priv.MYD
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./mysql/proxies_priv.frm to /home/mysql/db_incr_backup1/mysql/proxies_priv.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./mysql/event.frm to /home/mysql/db_incr_backup1/mysql/event.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./mysql/slow_log.CSV to /home/mysql/db_incr_backup1/mysql/slow_log.CSV
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./mysql/proc.MYI to /home/mysql/db_incr_backup1/mysql/proc.MYI
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./mysql/func.MYD to /home/mysql/db_incr_backup1/mysql/func.MYD
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./mysql/proc.frm to /home/mysql/db_incr_backup1/mysql/proc.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./mysql/slave_relay_log_info.frm to /home/mysql/db_incr_backup1/mysql/slave_relay_log_info.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./mysql/time_zone.frm to /home/mysql/db_incr_backup1/mysql/time_zone.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./mysql/db.MYI to /home/mysql/db_incr_backup1/mysql/db.MYI
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./mysql/gtid_executed.frm to /home/mysql/db_incr_backup1/mysql/gtid_executed.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./mysql/general_log.CSM to /home/mysql/db_incr_backup1/mysql/general_log.CSM
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./mysql/db.frm to /home/mysql/db_incr_backup1/mysql/db.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./mysql/plugin.frm to /home/mysql/db_incr_backup1/mysql/plugin.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./mysql/time_zone_leap_second.frm to /home/mysql/db_incr_backup1/mysql/time_zone_leap_second.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./mysql/general_log.frm to /home/mysql/db_incr_backup1/mysql/general_log.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./mysql/server_cost.frm to /home/mysql/db_incr_backup1/mysql/server_cost.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./mysql/servers.frm to /home/mysql/db_incr_backup1/mysql/servers.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./mysql/innodb_table_stats.frm to /home/mysql/db_incr_backup1/mysql/innodb_table_stats.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./mysql/columns_priv.frm to /home/mysql/db_incr_backup1/mysql/columns_priv.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./mysql/user.MYD to /home/mysql/db_incr_backup1/mysql/user.MYD
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./mysql/slave_worker_info.frm to /home/mysql/db_incr_backup1/mysql/slave_worker_info.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./mysql/time_zone_transition.frm to /home/mysql/db_incr_backup1/mysql/time_zone_transition.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./mysql/time_zone_name.frm to /home/mysql/db_incr_backup1/mysql/time_zone_name.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./mysql/slow_log.frm to /home/mysql/db_incr_backup1/mysql/slow_log.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./mysql/help_relation.frm to /home/mysql/db_incr_backup1/mysql/help_relation.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./mysql/proxies_priv.MYD to /home/mysql/db_incr_backup1/mysql/proxies_priv.MYD
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./mysql/event.MYD to /home/mysql/db_incr_backup1/mysql/event.MYD
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./mysql/slow_log.CSM to /home/mysql/db_incr_backup1/mysql/slow_log.CSM
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./mysql/innodb_index_stats.frm to /home/mysql/db_incr_backup1/mysql/innodb_index_stats.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./mysql/general_log.CSV to /home/mysql/db_incr_backup1/mysql/general_log.CSV
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./mysql/proc.MYD to /home/mysql/db_incr_backup1/mysql/proc.MYD
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./mysql/procs_priv.frm to /home/mysql/db_incr_backup1/mysql/procs_priv.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./mysql/procs_priv.MYD to /home/mysql/db_incr_backup1/mysql/procs_priv.MYD
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./mysql/columns_priv.MYI to /home/mysql/db_incr_backup1/mysql/columns_priv.MYI
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./dao/sal_grades.frm to /home/mysql/db_incr_backup1/dao/sal_grades.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./dao/db.opt to /home/mysql/db_incr_backup1/dao/db.opt
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./dao/penalties.frm to /home/mysql/db_incr_backup1/dao/penalties.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./dao/dept.frm to /home/mysql/db_incr_backup1/dao/dept.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./dao/matches.frm to /home/mysql/db_incr_backup1/dao/matches.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./dao/test_import.frm to /home/mysql/db_incr_backup1/dao/test_import.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./dao/test_tx_iso.frm to /home/mysql/db_incr_backup1/dao/test_tx_iso.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./dao/teams.frm to /home/mysql/db_incr_backup1/dao/teams.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./dao/committee_members.frm to /home/mysql/db_incr_backup1/dao/committee_members.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./dao/t1.frm to /home/mysql/db_incr_backup1/dao/t1.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./dao/test_binlog.frm to /home/mysql/db_incr_backup1/dao/test_binlog.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./dao/players.frm to /home/mysql/db_incr_backup1/dao/players.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./dao/emp.frm to /home/mysql/db_incr_backup1/dao/emp.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/x@0024innodb_buffer_stats_by_schema.frm to /home/mysql/db_incr_backup1/sys/x@0024innodb_buffer_stats_by_schema.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/x@0024statement_analysis.frm to /home/mysql/db_incr_backup1/sys/x@0024statement_analysis.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/x@0024user_summary.frm to /home/mysql/db_incr_backup1/sys/x@0024user_summary.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/x@0024schema_index_statistics.frm to /home/mysql/db_incr_backup1/sys/x@0024schema_index_statistics.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/x@0024statements_with_runtimes_in_95th_percentile.frm to /home/mysql/db_incr_backup1/sys/x@0024statements_with_runtimes_in_95th_percentile.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/x@0024innodb_buffer_stats_by_table.frm to /home/mysql/db_incr_backup1/sys/x@0024innodb_buffer_stats_by_table.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/db.opt to /home/mysql/db_incr_backup1/sys/db.opt
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/host_summary_by_statement_type.frm to /home/mysql/db_incr_backup1/sys/host_summary_by_statement_type.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/sys_config_insert_set_user.TRN to /home/mysql/db_incr_backup1/sys/sys_config_insert_set_user.TRN
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/statement_analysis.frm to /home/mysql/db_incr_backup1/sys/statement_analysis.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/version.frm to /home/mysql/db_incr_backup1/sys/version.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/host_summary.frm to /home/mysql/db_incr_backup1/sys/host_summary.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/x@0024io_global_by_wait_by_latency.frm to /home/mysql/db_incr_backup1/sys/x@0024io_global_by_wait_by_latency.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/statements_with_sorting.frm to /home/mysql/db_incr_backup1/sys/statements_with_sorting.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/sys_config.frm to /home/mysql/db_incr_backup1/sys/sys_config.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/x@0024io_by_thread_by_latency.frm to /home/mysql/db_incr_backup1/sys/x@0024io_by_thread_by_latency.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/memory_by_host_by_current_bytes.frm to /home/mysql/db_incr_backup1/sys/memory_by_host_by_current_bytes.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/x@0024memory_by_thread_by_current_bytes.frm to /home/mysql/db_incr_backup1/sys/x@0024memory_by_thread_by_current_bytes.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/x@0024schema_flattened_keys.frm to /home/mysql/db_incr_backup1/sys/x@0024schema_flattened_keys.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/x@0024memory_by_host_by_current_bytes.frm to /home/mysql/db_incr_backup1/sys/x@0024memory_by_host_by_current_bytes.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/sys_config.TRG to /home/mysql/db_incr_backup1/sys/sys_config.TRG
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/user_summary_by_file_io.frm to /home/mysql/db_incr_backup1/sys/user_summary_by_file_io.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/x@0024latest_file_io.frm to /home/mysql/db_incr_backup1/sys/x@0024latest_file_io.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/schema_object_overview.frm to /home/mysql/db_incr_backup1/sys/schema_object_overview.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/schema_unused_indexes.frm to /home/mysql/db_incr_backup1/sys/schema_unused_indexes.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/x@0024processlist.frm to /home/mysql/db_incr_backup1/sys/x@0024processlist.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/statements_with_full_table_scans.frm to /home/mysql/db_incr_backup1/sys/statements_with_full_table_scans.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/x@0024host_summary_by_file_io.frm to /home/mysql/db_incr_backup1/sys/x@0024host_summary_by_file_io.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/x@0024host_summary.frm to /home/mysql/db_incr_backup1/sys/x@0024host_summary.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/memory_by_user_by_current_bytes.frm to /home/mysql/db_incr_backup1/sys/memory_by_user_by_current_bytes.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/wait_classes_global_by_latency.frm to /home/mysql/db_incr_backup1/sys/wait_classes_global_by_latency.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 >> log scanned up to (2966684)
180729 13:33:04 [01] Copying ./sys/x@0024io_global_by_wait_by_bytes.frm to /home/mysql/db_incr_backup1/sys/x@0024io_global_by_wait_by_bytes.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/x@0024ps_schema_table_statistics_io.frm to /home/mysql/db_incr_backup1/sys/x@0024ps_schema_table_statistics_io.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/x@0024schema_table_statistics_with_buffer.frm to /home/mysql/db_incr_backup1/sys/x@0024schema_table_statistics_with_buffer.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/session_ssl_status.frm to /home/mysql/db_incr_backup1/sys/session_ssl_status.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/innodb_buffer_stats_by_schema.frm to /home/mysql/db_incr_backup1/sys/innodb_buffer_stats_by_schema.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/x@0024waits_by_user_by_latency.frm to /home/mysql/db_incr_backup1/sys/x@0024waits_by_user_by_latency.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/x@0024host_summary_by_stages.frm to /home/mysql/db_incr_backup1/sys/x@0024host_summary_by_stages.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/x@0024waits_global_by_latency.frm to /home/mysql/db_incr_backup1/sys/x@0024waits_global_by_latency.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/schema_tables_with_full_table_scans.frm to /home/mysql/db_incr_backup1/sys/schema_tables_with_full_table_scans.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/x@0024schema_table_lock_waits.frm to /home/mysql/db_incr_backup1/sys/x@0024schema_table_lock_waits.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/x@0024memory_by_user_by_current_bytes.frm to /home/mysql/db_incr_backup1/sys/x@0024memory_by_user_by_current_bytes.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/innodb_lock_waits.frm to /home/mysql/db_incr_backup1/sys/innodb_lock_waits.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/x@0024ps_digest_avg_latency_distribution.frm to /home/mysql/db_incr_backup1/sys/x@0024ps_digest_avg_latency_distribution.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/x@0024schema_table_statistics.frm to /home/mysql/db_incr_backup1/sys/x@0024schema_table_statistics.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/x@0024host_summary_by_file_io_type.frm to /home/mysql/db_incr_backup1/sys/x@0024host_summary_by_file_io_type.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/latest_file_io.frm to /home/mysql/db_incr_backup1/sys/latest_file_io.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/schema_redundant_indexes.frm to /home/mysql/db_incr_backup1/sys/schema_redundant_indexes.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/memory_global_by_current_bytes.frm to /home/mysql/db_incr_backup1/sys/memory_global_by_current_bytes.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/x@0024memory_global_by_current_bytes.frm to /home/mysql/db_incr_backup1/sys/x@0024memory_global_by_current_bytes.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/statements_with_errors_or_warnings.frm to /home/mysql/db_incr_backup1/sys/statements_with_errors_or_warnings.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/x@0024wait_classes_global_by_avg_latency.frm to /home/mysql/db_incr_backup1/sys/x@0024wait_classes_global_by_avg_latency.frm
180729 13:33:04 [01]        ...done
180729 13:33:04 [01] Copying ./sys/x@0024wait_classes_global_by_latency.frm to /home/mysql/db_incr_backup1/sys/x@0024wait_classes_global_by_latency.frm
180729 13:33:04 [01]        ...done
180729 13:33:05 [01] Copying ./sys/x@0024session.frm to /home/mysql/db_incr_backup1/sys/x@0024session.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/x@0024schema_tables_with_full_table_scans.frm to /home/mysql/db_incr_backup1/sys/x@0024schema_tables_with_full_table_scans.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/host_summary_by_file_io_type.frm to /home/mysql/db_incr_backup1/sys/host_summary_by_file_io_type.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/user_summary.frm to /home/mysql/db_incr_backup1/sys/user_summary.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/x@0024user_summary_by_stages.frm to /home/mysql/db_incr_backup1/sys/x@0024user_summary_by_stages.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/user_summary_by_file_io_type.frm to /home/mysql/db_incr_backup1/sys/user_summary_by_file_io_type.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/schema_table_statistics.frm to /home/mysql/db_incr_backup1/sys/schema_table_statistics.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/waits_by_user_by_latency.frm to /home/mysql/db_incr_backup1/sys/waits_by_user_by_latency.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/io_global_by_wait_by_bytes.frm to /home/mysql/db_incr_backup1/sys/io_global_by_wait_by_bytes.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/user_summary_by_statement_type.frm to /home/mysql/db_incr_backup1/sys/user_summary_by_statement_type.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/x@0024statements_with_sorting.frm to /home/mysql/db_incr_backup1/sys/x@0024statements_with_sorting.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/user_summary_by_statement_latency.frm to /home/mysql/db_incr_backup1/sys/user_summary_by_statement_latency.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/x@0024user_summary_by_statement_latency.frm to /home/mysql/db_incr_backup1/sys/x@0024user_summary_by_statement_latency.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/session.frm to /home/mysql/db_incr_backup1/sys/session.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/schema_table_statistics_with_buffer.frm to /home/mysql/db_incr_backup1/sys/schema_table_statistics_with_buffer.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/host_summary_by_file_io.frm to /home/mysql/db_incr_backup1/sys/host_summary_by_file_io.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/memory_by_thread_by_current_bytes.frm to /home/mysql/db_incr_backup1/sys/memory_by_thread_by_current_bytes.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/x@0024waits_by_host_by_latency.frm to /home/mysql/db_incr_backup1/sys/x@0024waits_by_host_by_latency.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/schema_auto_increment_columns.frm to /home/mysql/db_incr_backup1/sys/schema_auto_increment_columns.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/x@0024ps_digest_95th_percentile_by_avg_us.frm to /home/mysql/db_incr_backup1/sys/x@0024ps_digest_95th_percentile_by_avg_us.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/waits_by_host_by_latency.frm to /home/mysql/db_incr_backup1/sys/waits_by_host_by_latency.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/host_summary_by_statement_latency.frm to /home/mysql/db_incr_backup1/sys/host_summary_by_statement_latency.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/metrics.frm to /home/mysql/db_incr_backup1/sys/metrics.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/x@0024user_summary_by_file_io_type.frm to /home/mysql/db_incr_backup1/sys/x@0024user_summary_by_file_io_type.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/io_global_by_file_by_bytes.frm to /home/mysql/db_incr_backup1/sys/io_global_by_file_by_bytes.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/io_by_thread_by_latency.frm to /home/mysql/db_incr_backup1/sys/io_by_thread_by_latency.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/processlist.frm to /home/mysql/db_incr_backup1/sys/processlist.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/statements_with_runtimes_in_95th_percentile.frm to /home/mysql/db_incr_backup1/sys/statements_with_runtimes_in_95th_percentile.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/schema_table_lock_waits.frm to /home/mysql/db_incr_backup1/sys/schema_table_lock_waits.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/wait_classes_global_by_avg_latency.frm to /home/mysql/db_incr_backup1/sys/wait_classes_global_by_avg_latency.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/user_summary_by_stages.frm to /home/mysql/db_incr_backup1/sys/user_summary_by_stages.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/io_global_by_file_by_latency.frm to /home/mysql/db_incr_backup1/sys/io_global_by_file_by_latency.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/x@0024host_summary_by_statement_type.frm to /home/mysql/db_incr_backup1/sys/x@0024host_summary_by_statement_type.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/x@0024innodb_lock_waits.frm to /home/mysql/db_incr_backup1/sys/x@0024innodb_lock_waits.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/ps_check_lost_instrumentation.frm to /home/mysql/db_incr_backup1/sys/ps_check_lost_instrumentation.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/waits_global_by_latency.frm to /home/mysql/db_incr_backup1/sys/waits_global_by_latency.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/sys_config_update_set_user.TRN to /home/mysql/db_incr_backup1/sys/sys_config_update_set_user.TRN
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/innodb_buffer_stats_by_table.frm to /home/mysql/db_incr_backup1/sys/innodb_buffer_stats_by_table.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/x@0024statements_with_temp_tables.frm to /home/mysql/db_incr_backup1/sys/x@0024statements_with_temp_tables.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/io_global_by_wait_by_latency.frm to /home/mysql/db_incr_backup1/sys/io_global_by_wait_by_latency.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/host_summary_by_stages.frm to /home/mysql/db_incr_backup1/sys/host_summary_by_stages.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/statements_with_temp_tables.frm to /home/mysql/db_incr_backup1/sys/statements_with_temp_tables.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/x@0024user_summary_by_file_io.frm to /home/mysql/db_incr_backup1/sys/x@0024user_summary_by_file_io.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/schema_index_statistics.frm to /home/mysql/db_incr_backup1/sys/schema_index_statistics.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/x@0024host_summary_by_statement_latency.frm to /home/mysql/db_incr_backup1/sys/x@0024host_summary_by_statement_latency.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/x@0024user_summary_by_statement_type.frm to /home/mysql/db_incr_backup1/sys/x@0024user_summary_by_statement_type.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/x@0024statements_with_errors_or_warnings.frm to /home/mysql/db_incr_backup1/sys/x@0024statements_with_errors_or_warnings.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/x@0024io_global_by_file_by_latency.frm to /home/mysql/db_incr_backup1/sys/x@0024io_global_by_file_by_latency.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/x@0024io_global_by_file_by_bytes.frm to /home/mysql/db_incr_backup1/sys/x@0024io_global_by_file_by_bytes.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/x@0024memory_global_total.frm to /home/mysql/db_incr_backup1/sys/x@0024memory_global_total.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/memory_global_total.frm to /home/mysql/db_incr_backup1/sys/memory_global_total.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 [01] Copying ./sys/x@0024statements_with_full_table_scans.frm to /home/mysql/db_incr_backup1/sys/x@0024statements_with_full_table_scans.frm
180729 13:33:05 [01]        ...done
180729 13:33:05 Finished backing up non-InnoDB tables and files
180729 13:33:05 [00] Writing /home/mysql/db_incr_backup1/xtrabackup_binlog_info
180729 13:33:05 [00]        ...done
180729 13:33:05 Executing FLUSH NO_WRITE_TO_BINLOG ENGINE LOGS...
xtrabackup: The latest check point (for incremental): '2966675'
xtrabackup: Stopping log copying thread.
.180729 13:33:05 >> log scanned up to (2966684)

180729 13:33:05 Executing UNLOCK TABLES
180729 13:33:05 All tables unlocked
180729 13:33:05 [00] Copying ib_buffer_pool to /home/mysql/db_incr_backup1/ib_buffer_pool
180729 13:33:05 [00]        ...done
180729 13:33:05 Backup created in directory '/home/mysql/db_incr_backup1/'
MySQL binlog position: filename 'mysql-bin.000005', position '2774'
180729 13:33:05 [00] Writing /home/mysql/db_incr_backup1/backup-my.cnf
180729 13:33:05 [00]        ...done
180729 13:33:05 [00] Writing /home/mysql/db_incr_backup1/xtrabackup_info
180729 13:33:05 [00]        ...done
xtrabackup: Transaction log of lsn (2966675) to (2966684) was copied.
180729 13:33:05 completed OK!


```

### 第三次插入数据

```sql
mysql> insert into t1 values (3) ;
Query OK, 1 row affected (0.01 sec)

mysql> select  *  from t1 ;
+------+
| c1   |
+------+
|    1 |
|    2 |
|    3 |
+------+
3 rows in set (0.00 sec)


```

### 进行第二次增量备份

```sql

[mysql@bosenrui ~]$ innobackupex  --defaults-file=/mysql1/mysql1.cnf -u root -p root --no-timestamp --incremental --incremental-basedir=/home/mysql/db_incr_backup1  /home/mysql/db_incr_backup2
180729 13:36:35 innobackupex: Starting the backup operation

IMPORTANT: Please check that the backup run completes successfully.
           At the end of a successful backup run innobackupex
           prints "completed OK!".

180729 13:36:35  version_check Connecting to MySQL server with DSN 'dbi:mysql:;mysql_read_default_group=xtrabackup;port=3306;mysql_socket=/tmp/mysql1.sock' as 'root'  (using password: YES).
180729 13:36:35  version_check Connected to MySQL server
180729 13:36:35  version_check Executing a version check against the server...
180729 13:36:35  version_check Done.
180729 13:36:35 Connecting to MySQL server host: localhost, user: root, password: set, port: 3306, socket: /tmp/mysql1.sock
Using server version 5.7.21-log
innobackupex version 2.4.8 based on MySQL server 5.7.13 Linux (x86_64) (revision id: 97330f7)
incremental backup from 2966675 is enabled.
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
180729 13:36:35 >> log scanned up to (2967012)
xtrabackup: Generating a list of tablespaces
InnoDB: Allocated tablespace ID 8 for mysql/time_zone_name, old maximum was 0
xtrabackup: using the full scan for incremental backup
180729 13:36:36 [01] Copying ./ibdata1 to /home/mysql/db_incr_backup2/ibdata1.delta
180729 13:36:36 [01]        ...done
180729 13:36:36 [01] Copying ./mysql/time_zone_name.ibd to /home/mysql/db_incr_backup2/mysql/time_zone_name.ibd.delta
180729 13:36:36 [01]        ...done
180729 13:36:36 [01] Copying ./mysql/engine_cost.ibd to /home/mysql/db_incr_backup2/mysql/engine_cost.ibd.delta
180729 13:36:36 [01]        ...done
180729 13:36:36 [01] Copying ./mysql/help_topic.ibd to /home/mysql/db_incr_backup2/mysql/help_topic.ibd.delta
180729 13:36:36 [01]        ...done
180729 13:36:36 [01] Copying ./mysql/server_cost.ibd to /home/mysql/db_incr_backup2/mysql/server_cost.ibd.delta
180729 13:36:36 [01]        ...done
180729 13:36:36 [01] Copying ./mysql/help_category.ibd to /home/mysql/db_incr_backup2/mysql/help_category.ibd.delta
180729 13:36:36 [01]        ...done
180729 13:36:36 [01] Copying ./mysql/innodb_index_stats.ibd to /home/mysql/db_incr_backup2/mysql/innodb_index_stats.ibd.delta
180729 13:36:36 [01]        ...done
180729 13:36:36 [01] Copying ./mysql/innodb_table_stats.ibd to /home/mysql/db_incr_backup2/mysql/innodb_table_stats.ibd.delta
180729 13:36:36 [01]        ...done
180729 13:36:36 [01] Copying ./mysql/time_zone_transition_type.ibd to /home/mysql/db_incr_backup2/mysql/time_zone_transition_type.ibd.delta
180729 13:36:36 [01]        ...done
180729 13:36:36 [01] Copying ./mysql/slave_worker_info.ibd to /home/mysql/db_incr_backup2/mysql/slave_worker_info.ibd.delta
180729 13:36:36 [01]        ...done
180729 13:36:36 [01] Copying ./mysql/time_zone_transition.ibd to /home/mysql/db_incr_backup2/mysql/time_zone_transition.ibd.delta
180729 13:36:36 [01]        ...done
180729 13:36:36 [01] Copying ./mysql/time_zone_leap_second.ibd to /home/mysql/db_incr_backup2/mysql/time_zone_leap_second.ibd.delta
180729 13:36:36 [01]        ...done
180729 13:36:36 [01] Copying ./mysql/servers.ibd to /home/mysql/db_incr_backup2/mysql/servers.ibd.delta
180729 13:36:36 [01]        ...done
180729 13:36:36 [01] Copying ./mysql/gtid_executed.ibd to /home/mysql/db_incr_backup2/mysql/gtid_executed.ibd.delta
180729 13:36:36 [01]        ...done
180729 13:36:36 [01] Copying ./mysql/time_zone.ibd to /home/mysql/db_incr_backup2/mysql/time_zone.ibd.delta
180729 13:36:36 [01]        ...done
180729 13:36:36 [01] Copying ./mysql/help_relation.ibd to /home/mysql/db_incr_backup2/mysql/help_relation.ibd.delta
180729 13:36:36 [01]        ...done
180729 13:36:36 [01] Copying ./mysql/plugin.ibd to /home/mysql/db_incr_backup2/mysql/plugin.ibd.delta
180729 13:36:36 [01]        ...done
180729 13:36:36 [01] Copying ./mysql/slave_master_info.ibd to /home/mysql/db_incr_backup2/mysql/slave_master_info.ibd.delta
180729 13:36:36 [01]        ...done
180729 13:36:36 [01] Copying ./mysql/help_keyword.ibd to /home/mysql/db_incr_backup2/mysql/help_keyword.ibd.delta
180729 13:36:36 [01]        ...done
180729 13:36:36 [01] Copying ./mysql/slave_relay_log_info.ibd to /home/mysql/db_incr_backup2/mysql/slave_relay_log_info.ibd.delta
180729 13:36:36 [01]        ...done
180729 13:36:36 [01] Copying ./dao/emp.ibd to /home/mysql/db_incr_backup2/dao/emp.ibd.delta
180729 13:36:36 [01]        ...done
180729 13:36:36 [01] Copying ./dao/test_tx_iso.ibd to /home/mysql/db_incr_backup2/dao/test_tx_iso.ibd.delta
180729 13:36:36 [01]        ...done
180729 13:36:36 [01] Copying ./dao/matches.ibd to /home/mysql/db_incr_backup2/dao/matches.ibd.delta
180729 13:36:36 [01]        ...done
180729 13:36:36 [01] Copying ./dao/test_binlog.ibd to /home/mysql/db_incr_backup2/dao/test_binlog.ibd.delta
180729 13:36:36 [01]        ...done
180729 13:36:36 >> log scanned up to (2967012)
180729 13:36:36 [01] Copying ./dao/players.ibd to /home/mysql/db_incr_backup2/dao/players.ibd.delta
180729 13:36:36 [01]        ...done
180729 13:36:36 [01] Copying ./dao/sal_grades.ibd to /home/mysql/db_incr_backup2/dao/sal_grades.ibd.delta
180729 13:36:36 [01]        ...done
180729 13:36:36 [01] Copying ./dao/test_import.ibd to /home/mysql/db_incr_backup2/dao/test_import.ibd.delta
180729 13:36:36 [01]        ...done
180729 13:36:36 [01] Copying ./dao/penalties.ibd to /home/mysql/db_incr_backup2/dao/penalties.ibd.delta
180729 13:36:36 [01]        ...done
180729 13:36:36 [01] Copying ./dao/committee_members.ibd to /home/mysql/db_incr_backup2/dao/committee_members.ibd.delta
180729 13:36:36 [01]        ...done
180729 13:36:36 [01] Copying ./dao/teams.ibd to /home/mysql/db_incr_backup2/dao/teams.ibd.delta
180729 13:36:36 [01]        ...done
180729 13:36:36 [01] Copying ./dao/t1.ibd to /home/mysql/db_incr_backup2/dao/t1.ibd.delta
180729 13:36:36 [01]        ...done
180729 13:36:37 [01] Copying ./dao/dept.ibd to /home/mysql/db_incr_backup2/dao/dept.ibd.delta
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./sys/sys_config.ibd to /home/mysql/db_incr_backup2/sys/sys_config.ibd.delta
180729 13:36:37 [01]        ...done
180729 13:36:37 Executing FLUSH NO_WRITE_TO_BINLOG TABLES...
180729 13:36:37 Executing FLUSH TABLES WITH READ LOCK...
180729 13:36:37 Starting to backup non-InnoDB tables and files
180729 13:36:37 [01] Copying ./performance_schema/replication_applier_status.frm to /home/mysql/db_incr_backup2/performance_schema/replication_applier_status.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/file_instances.frm to /home/mysql/db_incr_backup2/performance_schema/file_instances.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/metadata_locks.frm to /home/mysql/db_incr_backup2/performance_schema/metadata_locks.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/memory_summary_by_account_by_event_name.frm to /home/mysql/db_incr_backup2/performance_schema/memory_summary_by_account_by_event_name.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/threads.frm to /home/mysql/db_incr_backup2/performance_schema/threads.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/events_waits_summary_by_user_by_event_name.frm to /home/mysql/db_incr_backup2/performance_schema/events_waits_summary_by_user_by_event_name.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/events_stages_summary_by_account_by_event_name.frm to /home/mysql/db_incr_backup2/performance_schema/events_stages_summary_by_account_by_event_name.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/db.opt to /home/mysql/db_incr_backup2/performance_schema/db.opt
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/table_io_waits_summary_by_table.frm to /home/mysql/db_incr_backup2/performance_schema/table_io_waits_summary_by_table.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/memory_summary_global_by_event_name.frm to /home/mysql/db_incr_backup2/performance_schema/memory_summary_global_by_event_name.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/events_stages_history_long.frm to /home/mysql/db_incr_backup2/performance_schema/events_stages_history_long.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/events_statements_summary_by_program.frm to /home/mysql/db_incr_backup2/performance_schema/events_statements_summary_by_program.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/events_statements_summary_by_user_by_event_name.frm to /home/mysql/db_incr_backup2/performance_schema/events_statements_summary_by_user_by_event_name.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/session_account_connect_attrs.frm to /home/mysql/db_incr_backup2/performance_schema/session_account_connect_attrs.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/session_connect_attrs.frm to /home/mysql/db_incr_backup2/performance_schema/session_connect_attrs.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/events_statements_summary_by_account_by_event_name.frm to /home/mysql/db_incr_backup2/performance_schema/events_statements_summary_by_account_by_event_name.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/events_statements_summary_by_thread_by_event_name.frm to /home/mysql/db_incr_backup2/performance_schema/events_statements_summary_by_thread_by_event_name.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/events_transactions_history.frm to /home/mysql/db_incr_backup2/performance_schema/events_transactions_history.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/memory_summary_by_user_by_event_name.frm to /home/mysql/db_incr_backup2/performance_schema/memory_summary_by_user_by_event_name.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/events_statements_summary_global_by_event_name.frm to /home/mysql/db_incr_backup2/performance_schema/events_statements_summary_global_by_event_name.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/performance_timers.frm to /home/mysql/db_incr_backup2/performance_schema/performance_timers.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/setup_consumers.frm to /home/mysql/db_incr_backup2/performance_schema/setup_consumers.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/events_stages_summary_by_host_by_event_name.frm to /home/mysql/db_incr_backup2/performance_schema/events_stages_summary_by_host_by_event_name.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/events_transactions_history_long.frm to /home/mysql/db_incr_backup2/performance_schema/events_transactions_history_long.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/replication_connection_status.frm to /home/mysql/db_incr_backup2/performance_schema/replication_connection_status.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/replication_group_members.frm to /home/mysql/db_incr_backup2/performance_schema/replication_group_members.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/objects_summary_global_by_type.frm to /home/mysql/db_incr_backup2/performance_schema/objects_summary_global_by_type.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/rwlock_instances.frm to /home/mysql/db_incr_backup2/performance_schema/rwlock_instances.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/user_variables_by_thread.frm to /home/mysql/db_incr_backup2/performance_schema/user_variables_by_thread.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/events_waits_current.frm to /home/mysql/db_incr_backup2/performance_schema/events_waits_current.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/file_summary_by_event_name.frm to /home/mysql/db_incr_backup2/performance_schema/file_summary_by_event_name.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/users.frm to /home/mysql/db_incr_backup2/performance_schema/users.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/replication_applier_status_by_worker.frm to /home/mysql/db_incr_backup2/performance_schema/replication_applier_status_by_worker.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/socket_instances.frm to /home/mysql/db_incr_backup2/performance_schema/socket_instances.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/events_stages_current.frm to /home/mysql/db_incr_backup2/performance_schema/events_stages_current.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/global_status.frm to /home/mysql/db_incr_backup2/performance_schema/global_status.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/setup_timers.frm to /home/mysql/db_incr_backup2/performance_schema/setup_timers.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/events_waits_history.frm to /home/mysql/db_incr_backup2/performance_schema/events_waits_history.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/table_handles.frm to /home/mysql/db_incr_backup2/performance_schema/table_handles.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/events_statements_current.frm to /home/mysql/db_incr_backup2/performance_schema/events_statements_current.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/events_transactions_current.frm to /home/mysql/db_incr_backup2/performance_schema/events_transactions_current.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/events_waits_summary_global_by_event_name.frm to /home/mysql/db_incr_backup2/performance_schema/events_waits_summary_global_by_event_name.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/replication_applier_status_by_coordinator.frm to /home/mysql/db_incr_backup2/performance_schema/replication_applier_status_by_coordinator.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/events_statements_summary_by_host_by_event_name.frm to /home/mysql/db_incr_backup2/performance_schema/events_statements_summary_by_host_by_event_name.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/events_stages_history.frm to /home/mysql/db_incr_backup2/performance_schema/events_stages_history.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/events_statements_summary_by_digest.frm to /home/mysql/db_incr_backup2/performance_schema/events_statements_summary_by_digest.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/accounts.frm to /home/mysql/db_incr_backup2/performance_schema/accounts.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/replication_applier_configuration.frm to /home/mysql/db_incr_backup2/performance_schema/replication_applier_configuration.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/replication_connection_configuration.frm to /home/mysql/db_incr_backup2/performance_schema/replication_connection_configuration.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/status_by_user.frm to /home/mysql/db_incr_backup2/performance_schema/status_by_user.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/events_transactions_summary_global_by_event_name.frm to /home/mysql/db_incr_backup2/performance_schema/events_transactions_summary_global_by_event_name.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/events_transactions_summary_by_account_by_event_name.frm to /home/mysql/db_incr_backup2/performance_schema/events_transactions_summary_by_account_by_event_name.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/events_stages_summary_global_by_event_name.frm to /home/mysql/db_incr_backup2/performance_schema/events_stages_summary_global_by_event_name.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/socket_summary_by_event_name.frm to /home/mysql/db_incr_backup2/performance_schema/socket_summary_by_event_name.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/memory_summary_by_thread_by_event_name.frm to /home/mysql/db_incr_backup2/performance_schema/memory_summary_by_thread_by_event_name.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/status_by_thread.frm to /home/mysql/db_incr_backup2/performance_schema/status_by_thread.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/events_waits_summary_by_instance.frm to /home/mysql/db_incr_backup2/performance_schema/events_waits_summary_by_instance.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/events_transactions_summary_by_thread_by_event_name.frm to /home/mysql/db_incr_backup2/performance_schema/events_transactions_summary_by_thread_by_event_name.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/hosts.frm to /home/mysql/db_incr_backup2/performance_schema/hosts.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/cond_instances.frm to /home/mysql/db_incr_backup2/performance_schema/cond_instances.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/memory_summary_by_host_by_event_name.frm to /home/mysql/db_incr_backup2/performance_schema/memory_summary_by_host_by_event_name.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/status_by_account.frm to /home/mysql/db_incr_backup2/performance_schema/status_by_account.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/socket_summary_by_instance.frm to /home/mysql/db_incr_backup2/performance_schema/socket_summary_by_instance.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/events_transactions_summary_by_user_by_event_name.frm to /home/mysql/db_incr_backup2/performance_schema/events_transactions_summary_by_user_by_event_name.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/status_by_host.frm to /home/mysql/db_incr_backup2/performance_schema/status_by_host.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/events_stages_summary_by_user_by_event_name.frm to /home/mysql/db_incr_backup2/performance_schema/events_stages_summary_by_user_by_event_name.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/global_variables.frm to /home/mysql/db_incr_backup2/performance_schema/global_variables.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/setup_instruments.frm to /home/mysql/db_incr_backup2/performance_schema/setup_instruments.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/events_waits_summary_by_thread_by_event_name.frm to /home/mysql/db_incr_backup2/performance_schema/events_waits_summary_by_thread_by_event_name.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/replication_group_member_stats.frm to /home/mysql/db_incr_backup2/performance_schema/replication_group_member_stats.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/host_cache.frm to /home/mysql/db_incr_backup2/performance_schema/host_cache.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/setup_objects.frm to /home/mysql/db_incr_backup2/performance_schema/setup_objects.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/events_waits_history_long.frm to /home/mysql/db_incr_backup2/performance_schema/events_waits_history_long.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 >> log scanned up to (2967012)
180729 13:36:37 [01] Copying ./performance_schema/session_status.frm to /home/mysql/db_incr_backup2/performance_schema/session_status.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/events_transactions_summary_by_host_by_event_name.frm to /home/mysql/db_incr_backup2/performance_schema/events_transactions_summary_by_host_by_event_name.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/prepared_statements_instances.frm to /home/mysql/db_incr_backup2/performance_schema/prepared_statements_instances.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/setup_actors.frm to /home/mysql/db_incr_backup2/performance_schema/setup_actors.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/mutex_instances.frm to /home/mysql/db_incr_backup2/performance_schema/mutex_instances.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/table_lock_waits_summary_by_table.frm to /home/mysql/db_incr_backup2/performance_schema/table_lock_waits_summary_by_table.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/variables_by_thread.frm to /home/mysql/db_incr_backup2/performance_schema/variables_by_thread.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/events_waits_summary_by_host_by_event_name.frm to /home/mysql/db_incr_backup2/performance_schema/events_waits_summary_by_host_by_event_name.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/events_statements_history_long.frm to /home/mysql/db_incr_backup2/performance_schema/events_statements_history_long.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/events_waits_summary_by_account_by_event_name.frm to /home/mysql/db_incr_backup2/performance_schema/events_waits_summary_by_account_by_event_name.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/session_variables.frm to /home/mysql/db_incr_backup2/performance_schema/session_variables.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/events_stages_summary_by_thread_by_event_name.frm to /home/mysql/db_incr_backup2/performance_schema/events_stages_summary_by_thread_by_event_name.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/file_summary_by_instance.frm to /home/mysql/db_incr_backup2/performance_schema/file_summary_by_instance.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/events_statements_history.frm to /home/mysql/db_incr_backup2/performance_schema/events_statements_history.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./performance_schema/table_io_waits_summary_by_index_usage.frm to /home/mysql/db_incr_backup2/performance_schema/table_io_waits_summary_by_index_usage.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./mysql/user.frm to /home/mysql/db_incr_backup2/mysql/user.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./mysql/func.MYI to /home/mysql/db_incr_backup2/mysql/func.MYI
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./mysql/help_topic.frm to /home/mysql/db_incr_backup2/mysql/help_topic.frm
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./mysql/proxies_priv.MYI to /home/mysql/db_incr_backup2/mysql/proxies_priv.MYI
180729 13:36:37 [01]        ...done
180729 13:36:37 [01] Copying ./mysql/help_keyword.frm to /home/mysql/db_incr_backup2/mysql/help_keyword.frm
180729 13:36:37 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/func.frm to /home/mysql/db_incr_backup2/mysql/func.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/ndb_binlog_index.MYD to /home/mysql/db_incr_backup2/mysql/ndb_binlog_index.MYD
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/tables_priv.MYI to /home/mysql/db_incr_backup2/mysql/tables_priv.MYI
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/db.opt to /home/mysql/db_incr_backup2/mysql/db.opt
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/db.MYD to /home/mysql/db_incr_backup2/mysql/db.MYD
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/engine_cost.frm to /home/mysql/db_incr_backup2/mysql/engine_cost.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/ndb_binlog_index.MYI to /home/mysql/db_incr_backup2/mysql/ndb_binlog_index.MYI
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/procs_priv.MYI to /home/mysql/db_incr_backup2/mysql/procs_priv.MYI
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/help_category.frm to /home/mysql/db_incr_backup2/mysql/help_category.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/ndb_binlog_index.frm to /home/mysql/db_incr_backup2/mysql/ndb_binlog_index.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/user.MYI to /home/mysql/db_incr_backup2/mysql/user.MYI
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/event.MYI to /home/mysql/db_incr_backup2/mysql/event.MYI
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/tables_priv.frm to /home/mysql/db_incr_backup2/mysql/tables_priv.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/slave_master_info.frm to /home/mysql/db_incr_backup2/mysql/slave_master_info.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/time_zone_transition_type.frm to /home/mysql/db_incr_backup2/mysql/time_zone_transition_type.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/columns_priv.MYD to /home/mysql/db_incr_backup2/mysql/columns_priv.MYD
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/tables_priv.MYD to /home/mysql/db_incr_backup2/mysql/tables_priv.MYD
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/proxies_priv.frm to /home/mysql/db_incr_backup2/mysql/proxies_priv.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/event.frm to /home/mysql/db_incr_backup2/mysql/event.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/slow_log.CSV to /home/mysql/db_incr_backup2/mysql/slow_log.CSV
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/proc.MYI to /home/mysql/db_incr_backup2/mysql/proc.MYI
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/func.MYD to /home/mysql/db_incr_backup2/mysql/func.MYD
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/proc.frm to /home/mysql/db_incr_backup2/mysql/proc.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/slave_relay_log_info.frm to /home/mysql/db_incr_backup2/mysql/slave_relay_log_info.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/time_zone.frm to /home/mysql/db_incr_backup2/mysql/time_zone.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/db.MYI to /home/mysql/db_incr_backup2/mysql/db.MYI
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/gtid_executed.frm to /home/mysql/db_incr_backup2/mysql/gtid_executed.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/general_log.CSM to /home/mysql/db_incr_backup2/mysql/general_log.CSM
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/db.frm to /home/mysql/db_incr_backup2/mysql/db.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/plugin.frm to /home/mysql/db_incr_backup2/mysql/plugin.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/time_zone_leap_second.frm to /home/mysql/db_incr_backup2/mysql/time_zone_leap_second.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/general_log.frm to /home/mysql/db_incr_backup2/mysql/general_log.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/server_cost.frm to /home/mysql/db_incr_backup2/mysql/server_cost.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/servers.frm to /home/mysql/db_incr_backup2/mysql/servers.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/innodb_table_stats.frm to /home/mysql/db_incr_backup2/mysql/innodb_table_stats.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/columns_priv.frm to /home/mysql/db_incr_backup2/mysql/columns_priv.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/user.MYD to /home/mysql/db_incr_backup2/mysql/user.MYD
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/slave_worker_info.frm to /home/mysql/db_incr_backup2/mysql/slave_worker_info.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/time_zone_transition.frm to /home/mysql/db_incr_backup2/mysql/time_zone_transition.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/time_zone_name.frm to /home/mysql/db_incr_backup2/mysql/time_zone_name.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/slow_log.frm to /home/mysql/db_incr_backup2/mysql/slow_log.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/help_relation.frm to /home/mysql/db_incr_backup2/mysql/help_relation.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/proxies_priv.MYD to /home/mysql/db_incr_backup2/mysql/proxies_priv.MYD
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/event.MYD to /home/mysql/db_incr_backup2/mysql/event.MYD
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/slow_log.CSM to /home/mysql/db_incr_backup2/mysql/slow_log.CSM
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/innodb_index_stats.frm to /home/mysql/db_incr_backup2/mysql/innodb_index_stats.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/general_log.CSV to /home/mysql/db_incr_backup2/mysql/general_log.CSV
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/proc.MYD to /home/mysql/db_incr_backup2/mysql/proc.MYD
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/procs_priv.frm to /home/mysql/db_incr_backup2/mysql/procs_priv.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/procs_priv.MYD to /home/mysql/db_incr_backup2/mysql/procs_priv.MYD
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./mysql/columns_priv.MYI to /home/mysql/db_incr_backup2/mysql/columns_priv.MYI
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./dao/sal_grades.frm to /home/mysql/db_incr_backup2/dao/sal_grades.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./dao/db.opt to /home/mysql/db_incr_backup2/dao/db.opt
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./dao/penalties.frm to /home/mysql/db_incr_backup2/dao/penalties.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./dao/dept.frm to /home/mysql/db_incr_backup2/dao/dept.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./dao/matches.frm to /home/mysql/db_incr_backup2/dao/matches.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./dao/test_import.frm to /home/mysql/db_incr_backup2/dao/test_import.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./dao/test_tx_iso.frm to /home/mysql/db_incr_backup2/dao/test_tx_iso.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./dao/teams.frm to /home/mysql/db_incr_backup2/dao/teams.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./dao/committee_members.frm to /home/mysql/db_incr_backup2/dao/committee_members.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./dao/t1.frm to /home/mysql/db_incr_backup2/dao/t1.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./dao/test_binlog.frm to /home/mysql/db_incr_backup2/dao/test_binlog.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./dao/players.frm to /home/mysql/db_incr_backup2/dao/players.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./dao/emp.frm to /home/mysql/db_incr_backup2/dao/emp.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/x@0024innodb_buffer_stats_by_schema.frm to /home/mysql/db_incr_backup2/sys/x@0024innodb_buffer_stats_by_schema.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/x@0024statement_analysis.frm to /home/mysql/db_incr_backup2/sys/x@0024statement_analysis.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/x@0024user_summary.frm to /home/mysql/db_incr_backup2/sys/x@0024user_summary.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/x@0024schema_index_statistics.frm to /home/mysql/db_incr_backup2/sys/x@0024schema_index_statistics.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/x@0024statements_with_runtimes_in_95th_percentile.frm to /home/mysql/db_incr_backup2/sys/x@0024statements_with_runtimes_in_95th_percentile.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/x@0024innodb_buffer_stats_by_table.frm to /home/mysql/db_incr_backup2/sys/x@0024innodb_buffer_stats_by_table.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/db.opt to /home/mysql/db_incr_backup2/sys/db.opt
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/host_summary_by_statement_type.frm to /home/mysql/db_incr_backup2/sys/host_summary_by_statement_type.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/sys_config_insert_set_user.TRN to /home/mysql/db_incr_backup2/sys/sys_config_insert_set_user.TRN
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/statement_analysis.frm to /home/mysql/db_incr_backup2/sys/statement_analysis.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/version.frm to /home/mysql/db_incr_backup2/sys/version.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/host_summary.frm to /home/mysql/db_incr_backup2/sys/host_summary.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/x@0024io_global_by_wait_by_latency.frm to /home/mysql/db_incr_backup2/sys/x@0024io_global_by_wait_by_latency.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/statements_with_sorting.frm to /home/mysql/db_incr_backup2/sys/statements_with_sorting.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/sys_config.frm to /home/mysql/db_incr_backup2/sys/sys_config.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/x@0024io_by_thread_by_latency.frm to /home/mysql/db_incr_backup2/sys/x@0024io_by_thread_by_latency.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/memory_by_host_by_current_bytes.frm to /home/mysql/db_incr_backup2/sys/memory_by_host_by_current_bytes.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/x@0024memory_by_thread_by_current_bytes.frm to /home/mysql/db_incr_backup2/sys/x@0024memory_by_thread_by_current_bytes.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/x@0024schema_flattened_keys.frm to /home/mysql/db_incr_backup2/sys/x@0024schema_flattened_keys.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/x@0024memory_by_host_by_current_bytes.frm to /home/mysql/db_incr_backup2/sys/x@0024memory_by_host_by_current_bytes.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/sys_config.TRG to /home/mysql/db_incr_backup2/sys/sys_config.TRG
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/user_summary_by_file_io.frm to /home/mysql/db_incr_backup2/sys/user_summary_by_file_io.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/x@0024latest_file_io.frm to /home/mysql/db_incr_backup2/sys/x@0024latest_file_io.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/schema_object_overview.frm to /home/mysql/db_incr_backup2/sys/schema_object_overview.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/schema_unused_indexes.frm to /home/mysql/db_incr_backup2/sys/schema_unused_indexes.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/x@0024processlist.frm to /home/mysql/db_incr_backup2/sys/x@0024processlist.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/statements_with_full_table_scans.frm to /home/mysql/db_incr_backup2/sys/statements_with_full_table_scans.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/x@0024host_summary_by_file_io.frm to /home/mysql/db_incr_backup2/sys/x@0024host_summary_by_file_io.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/x@0024host_summary.frm to /home/mysql/db_incr_backup2/sys/x@0024host_summary.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 >> log scanned up to (2967012)
180729 13:36:38 [01] Copying ./sys/memory_by_user_by_current_bytes.frm to /home/mysql/db_incr_backup2/sys/memory_by_user_by_current_bytes.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/wait_classes_global_by_latency.frm to /home/mysql/db_incr_backup2/sys/wait_classes_global_by_latency.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/x@0024io_global_by_wait_by_bytes.frm to /home/mysql/db_incr_backup2/sys/x@0024io_global_by_wait_by_bytes.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/x@0024ps_schema_table_statistics_io.frm to /home/mysql/db_incr_backup2/sys/x@0024ps_schema_table_statistics_io.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/x@0024schema_table_statistics_with_buffer.frm to /home/mysql/db_incr_backup2/sys/x@0024schema_table_statistics_with_buffer.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/session_ssl_status.frm to /home/mysql/db_incr_backup2/sys/session_ssl_status.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/innodb_buffer_stats_by_schema.frm to /home/mysql/db_incr_backup2/sys/innodb_buffer_stats_by_schema.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/x@0024waits_by_user_by_latency.frm to /home/mysql/db_incr_backup2/sys/x@0024waits_by_user_by_latency.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/x@0024host_summary_by_stages.frm to /home/mysql/db_incr_backup2/sys/x@0024host_summary_by_stages.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/x@0024waits_global_by_latency.frm to /home/mysql/db_incr_backup2/sys/x@0024waits_global_by_latency.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/schema_tables_with_full_table_scans.frm to /home/mysql/db_incr_backup2/sys/schema_tables_with_full_table_scans.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/x@0024schema_table_lock_waits.frm to /home/mysql/db_incr_backup2/sys/x@0024schema_table_lock_waits.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/x@0024memory_by_user_by_current_bytes.frm to /home/mysql/db_incr_backup2/sys/x@0024memory_by_user_by_current_bytes.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/innodb_lock_waits.frm to /home/mysql/db_incr_backup2/sys/innodb_lock_waits.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/x@0024ps_digest_avg_latency_distribution.frm to /home/mysql/db_incr_backup2/sys/x@0024ps_digest_avg_latency_distribution.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/x@0024schema_table_statistics.frm to /home/mysql/db_incr_backup2/sys/x@0024schema_table_statistics.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/x@0024host_summary_by_file_io_type.frm to /home/mysql/db_incr_backup2/sys/x@0024host_summary_by_file_io_type.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/latest_file_io.frm to /home/mysql/db_incr_backup2/sys/latest_file_io.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/schema_redundant_indexes.frm to /home/mysql/db_incr_backup2/sys/schema_redundant_indexes.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/memory_global_by_current_bytes.frm to /home/mysql/db_incr_backup2/sys/memory_global_by_current_bytes.frm
180729 13:36:38 [01]        ...done
180729 13:36:38 [01] Copying ./sys/x@0024memory_global_by_current_bytes.frm to /home/mysql/db_incr_backup2/sys/x@0024memory_global_by_current_bytes.frm
180729 13:36:38 [01]        ...done
180729 13:36:39 [01] Copying ./sys/statements_with_errors_or_warnings.frm to /home/mysql/db_incr_backup2/sys/statements_with_errors_or_warnings.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/x@0024wait_classes_global_by_avg_latency.frm to /home/mysql/db_incr_backup2/sys/x@0024wait_classes_global_by_avg_latency.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/x@0024wait_classes_global_by_latency.frm to /home/mysql/db_incr_backup2/sys/x@0024wait_classes_global_by_latency.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/x@0024session.frm to /home/mysql/db_incr_backup2/sys/x@0024session.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/x@0024schema_tables_with_full_table_scans.frm to /home/mysql/db_incr_backup2/sys/x@0024schema_tables_with_full_table_scans.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/host_summary_by_file_io_type.frm to /home/mysql/db_incr_backup2/sys/host_summary_by_file_io_type.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/user_summary.frm to /home/mysql/db_incr_backup2/sys/user_summary.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/x@0024user_summary_by_stages.frm to /home/mysql/db_incr_backup2/sys/x@0024user_summary_by_stages.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/user_summary_by_file_io_type.frm to /home/mysql/db_incr_backup2/sys/user_summary_by_file_io_type.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/schema_table_statistics.frm to /home/mysql/db_incr_backup2/sys/schema_table_statistics.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/waits_by_user_by_latency.frm to /home/mysql/db_incr_backup2/sys/waits_by_user_by_latency.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/io_global_by_wait_by_bytes.frm to /home/mysql/db_incr_backup2/sys/io_global_by_wait_by_bytes.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/user_summary_by_statement_type.frm to /home/mysql/db_incr_backup2/sys/user_summary_by_statement_type.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/x@0024statements_with_sorting.frm to /home/mysql/db_incr_backup2/sys/x@0024statements_with_sorting.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/user_summary_by_statement_latency.frm to /home/mysql/db_incr_backup2/sys/user_summary_by_statement_latency.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/x@0024user_summary_by_statement_latency.frm to /home/mysql/db_incr_backup2/sys/x@0024user_summary_by_statement_latency.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/session.frm to /home/mysql/db_incr_backup2/sys/session.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/schema_table_statistics_with_buffer.frm to /home/mysql/db_incr_backup2/sys/schema_table_statistics_with_buffer.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/host_summary_by_file_io.frm to /home/mysql/db_incr_backup2/sys/host_summary_by_file_io.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/memory_by_thread_by_current_bytes.frm to /home/mysql/db_incr_backup2/sys/memory_by_thread_by_current_bytes.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/x@0024waits_by_host_by_latency.frm to /home/mysql/db_incr_backup2/sys/x@0024waits_by_host_by_latency.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/schema_auto_increment_columns.frm to /home/mysql/db_incr_backup2/sys/schema_auto_increment_columns.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/x@0024ps_digest_95th_percentile_by_avg_us.frm to /home/mysql/db_incr_backup2/sys/x@0024ps_digest_95th_percentile_by_avg_us.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/waits_by_host_by_latency.frm to /home/mysql/db_incr_backup2/sys/waits_by_host_by_latency.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/host_summary_by_statement_latency.frm to /home/mysql/db_incr_backup2/sys/host_summary_by_statement_latency.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/metrics.frm to /home/mysql/db_incr_backup2/sys/metrics.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/x@0024user_summary_by_file_io_type.frm to /home/mysql/db_incr_backup2/sys/x@0024user_summary_by_file_io_type.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/io_global_by_file_by_bytes.frm to /home/mysql/db_incr_backup2/sys/io_global_by_file_by_bytes.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/io_by_thread_by_latency.frm to /home/mysql/db_incr_backup2/sys/io_by_thread_by_latency.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/processlist.frm to /home/mysql/db_incr_backup2/sys/processlist.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/statements_with_runtimes_in_95th_percentile.frm to /home/mysql/db_incr_backup2/sys/statements_with_runtimes_in_95th_percentile.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/schema_table_lock_waits.frm to /home/mysql/db_incr_backup2/sys/schema_table_lock_waits.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/wait_classes_global_by_avg_latency.frm to /home/mysql/db_incr_backup2/sys/wait_classes_global_by_avg_latency.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/user_summary_by_stages.frm to /home/mysql/db_incr_backup2/sys/user_summary_by_stages.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/io_global_by_file_by_latency.frm to /home/mysql/db_incr_backup2/sys/io_global_by_file_by_latency.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/x@0024host_summary_by_statement_type.frm to /home/mysql/db_incr_backup2/sys/x@0024host_summary_by_statement_type.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/x@0024innodb_lock_waits.frm to /home/mysql/db_incr_backup2/sys/x@0024innodb_lock_waits.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/ps_check_lost_instrumentation.frm to /home/mysql/db_incr_backup2/sys/ps_check_lost_instrumentation.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/waits_global_by_latency.frm to /home/mysql/db_incr_backup2/sys/waits_global_by_latency.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/sys_config_update_set_user.TRN to /home/mysql/db_incr_backup2/sys/sys_config_update_set_user.TRN
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/innodb_buffer_stats_by_table.frm to /home/mysql/db_incr_backup2/sys/innodb_buffer_stats_by_table.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/x@0024statements_with_temp_tables.frm to /home/mysql/db_incr_backup2/sys/x@0024statements_with_temp_tables.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/io_global_by_wait_by_latency.frm to /home/mysql/db_incr_backup2/sys/io_global_by_wait_by_latency.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/host_summary_by_stages.frm to /home/mysql/db_incr_backup2/sys/host_summary_by_stages.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/statements_with_temp_tables.frm to /home/mysql/db_incr_backup2/sys/statements_with_temp_tables.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/x@0024user_summary_by_file_io.frm to /home/mysql/db_incr_backup2/sys/x@0024user_summary_by_file_io.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/schema_index_statistics.frm to /home/mysql/db_incr_backup2/sys/schema_index_statistics.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/x@0024host_summary_by_statement_latency.frm to /home/mysql/db_incr_backup2/sys/x@0024host_summary_by_statement_latency.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/x@0024user_summary_by_statement_type.frm to /home/mysql/db_incr_backup2/sys/x@0024user_summary_by_statement_type.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/x@0024statements_with_errors_or_warnings.frm to /home/mysql/db_incr_backup2/sys/x@0024statements_with_errors_or_warnings.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/x@0024io_global_by_file_by_latency.frm to /home/mysql/db_incr_backup2/sys/x@0024io_global_by_file_by_latency.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/x@0024io_global_by_file_by_bytes.frm to /home/mysql/db_incr_backup2/sys/x@0024io_global_by_file_by_bytes.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/x@0024memory_global_total.frm to /home/mysql/db_incr_backup2/sys/x@0024memory_global_total.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/memory_global_total.frm to /home/mysql/db_incr_backup2/sys/memory_global_total.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 [01] Copying ./sys/x@0024statements_with_full_table_scans.frm to /home/mysql/db_incr_backup2/sys/x@0024statements_with_full_table_scans.frm
180729 13:36:39 [01]        ...done
180729 13:36:39 Finished backing up non-InnoDB tables and files
180729 13:36:39 [00] Writing /home/mysql/db_incr_backup2/xtrabackup_binlog_info
180729 13:36:39 [00]        ...done
180729 13:36:39 Executing FLUSH NO_WRITE_TO_BINLOG ENGINE LOGS...
xtrabackup: The latest check point (for incremental): '2967003'
xtrabackup: Stopping log copying thread.
.180729 13:36:39 >> log scanned up to (2967012)

180729 13:36:39 Executing UNLOCK TABLES
180729 13:36:39 All tables unlocked
180729 13:36:39 [00] Copying ib_buffer_pool to /home/mysql/db_incr_backup2/ib_buffer_pool
180729 13:36:39 [00]        ...done
180729 13:36:39 Backup created in directory '/home/mysql/db_incr_backup2/'
MySQL binlog position: filename 'mysql-bin.000005', position '3025'
180729 13:36:39 [00] Writing /home/mysql/db_incr_backup2/backup-my.cnf
180729 13:36:39 [00]        ...done
180729 13:36:39 [00] Writing /home/mysql/db_incr_backup2/xtrabackup_info
180729 13:36:39 [00]        ...done
xtrabackup: Transaction log of lsn (2967003) to (2967012) was copied.
180729 13:36:39 completed OK!


```


## 查看备份

### 查看备份文件夹

```sql

[mysql@bosenrui ~]$ ll -d  /home/mysql/db_*
drwxr-x--- 6 mysql mysql 4096 Jul 29 13:25 /home/mysql/db_full_backup
drwxr-x--- 6 mysql mysql 4096 Jul 29 13:33 /home/mysql/db_incr_backup1
drwxr-x--- 6 mysql mysql 4096 Jul 29 13:36 /home/mysql/db_incr_backup2

```


### 查看备份文件夹大小

```sql

[mysql@bosenrui ~]$ du -sh  db*
27M     db_full_backup
3.3M    db_incr_backup1
3.3M    db_incr_backup2


```


### 查看全备信息

#### 查看备份文件夹

```sql
[mysql@bosenrui ~]$ cd /home/mysql/db_full_backup/
[mysql@bosenrui db_full_backup]$ ll
total 12336
-rw-r----- 1 mysql mysql      424 Jul 29 13:25 backup-my.cnf
drwxr-x--- 2 mysql mysql     4096 Jul 29 13:25 dao
-rw-r----- 1 mysql mysql      494 Jul 29 13:25 ib_buffer_pool
-rw-r----- 1 mysql mysql 12582912 Jul 29 13:25 ibdata1
drwxr-x--- 2 mysql mysql     4096 Jul 29 13:25 mysql
drwxr-x--- 2 mysql mysql     4096 Jul 29 13:25 performance_schema
drwxr-x--- 2 mysql mysql    12288 Jul 29 13:25 sys
-rw-r----- 1 mysql mysql       22 Jul 29 13:25 xtrabackup_binlog_info
-rw-r----- 1 mysql mysql      113 Jul 29 13:25 xtrabackup_checkpoints
-rw-r----- 1 mysql mysql      523 Jul 29 13:25 xtrabackup_info
-rw-r----- 1 mysql mysql     2560 Jul 29 13:25 xtrabackup_logfile

```
####  查看备份信息

```sql


[mysql@bosenrui db_full_backup]$ cat xtrabackup_info
uuid = db20ef4a-92ef-11e8-a7d8-000c2916156f
name =
tool_name = innobackupex
tool_command = --defaults-file=/mysql1/mysql1.cnf --no-timestamp -u root -p root /home/mysql/db_full_backup/
tool_version = 2.4.8
ibbackup_version = 2.4.8
server_version = 5.7.21-log
start_time = 2018-07-29 13:25:46
end_time = 2018-07-29 13:25:50
lock_time = 0
binlog_pos = filename 'mysql-bin.000005', position '2523'
innodb_from_lsn = 0
innodb_to_lsn = 2966331
partial = N
incremental = N
format = file
compact = N
compressed = N
encrypted = N


```

#### 查看备份binlog信息

```sql

[mysql@bosenrui db_full_backup]$ cat xtrabackup_binlog_info
mysql-bin.000005        2523

```

#### 查看检查点信息 

````sql

[mysql@bosenrui db_full_backup]$ cat xtrabackup_checkpoints
backup_type = full-backuped
from_lsn = 0
to_lsn = 2966331
last_lsn = 2966340
compact = 0
recover_binlog_info = 0
[mysql@bosenrui db_full_backu

```

### 查看着第一次增量备份信息



#### 查看备份文件夹

```sql
[mysql@bosenrui db_full_backup]$  cd /home/mysql/db_incr_backup1/
[mysql@bosenrui db_incr_backup1]$ ll
total 148
-rw-r----- 1 mysql mysql   424 Jul 29 13:33 backup-my.cnf
drwxr-x--- 2 mysql mysql  4096 Jul 29 13:33 dao
-rw-r----- 1 mysql mysql   494 Jul 29 13:33 ib_buffer_pool
-rw-r----- 1 mysql mysql 98304 Jul 29 13:33 ibdata1.delta
-rw-r----- 1 mysql mysql    44 Jul 29 13:33 ibdata1.meta
drwxr-x--- 2 mysql mysql  4096 Jul 29 13:33 mysql
drwxr-x--- 2 mysql mysql  4096 Jul 29 13:33 performance_schema
drwxr-x--- 2 mysql mysql 12288 Jul 29 13:33 sys
-rw-r----- 1 mysql mysql    22 Jul 29 13:33 xtrabackup_binlog_info
-rw-r----- 1 mysql mysql   117 Jul 29 13:33 xtrabackup_checkpoints
-rw-r----- 1 mysql mysql   592 Jul 29 13:33 xtrabackup_info
-rw-r----- 1 mysql mysql  2560 Jul 29 13:33 xtrabackup_logfile

```

####  查看备份信息

```sql

[mysql@bosenrui db_incr_backup1]$ cat xtrabackup_info
uuid = de7aafbb-92f0-11e8-a7d8-000c2916156f
name =
tool_name = innobackupex
tool_command = --defaults-file=/mysql1/mysql1.cnf -u root -p root --no-timestamp --incremental --incremental-basedir=/home/mysql/db_full_backup /home/mysql/db_incr_backup1
tool_version = 2.4.8
ibbackup_version = 2.4.8
server_version = 5.7.21-log
start_time = 2018-07-29 13:33:00
end_time = 2018-07-29 13:33:05
lock_time = 0
binlog_pos = filename 'mysql-bin.000005', position '2774'
innodb_from_lsn = 2966331
innodb_to_lsn = 2966675
partial = N
incremental = Y
format = file
compact = N
compressed = N
encrypted = N

```

#### 查看备份binlog信息

```sql

[mysql@bosenrui db_incr_backup1]$ cat xtrabackup_binlog_info
mysql-bin.000005        2774

```

#### 查看检查点信息 

````sql

[mysql@bosenrui db_incr_backup1]$ cat xtrabackup_checkpoints
backup_type = incremental
from_lsn = 2966331
to_lsn = 2966675
last_lsn = 2966684
compact = 0
recover_binlog_info = 0


```

### 查看着第二次增量备份信息

#### 查看备份文件夹

```sql

[mysql@bosenrui db_incr_backup1]$ cd /home/mysql/db_incr_backup2/
[mysql@bosenrui db_incr_backup2]$ ll
total 148
-rw-r----- 1 mysql mysql   424 Jul 29 13:36 backup-my.cnf
drwxr-x--- 2 mysql mysql  4096 Jul 29 13:36 dao
-rw-r----- 1 mysql mysql   494 Jul 29 13:36 ib_buffer_pool
-rw-r----- 1 mysql mysql 98304 Jul 29 13:36 ibdata1.delta
-rw-r----- 1 mysql mysql    44 Jul 29 13:36 ibdata1.meta
drwxr-x--- 2 mysql mysql  4096 Jul 29 13:36 mysql
drwxr-x--- 2 mysql mysql  4096 Jul 29 13:36 performance_schema
drwxr-x--- 2 mysql mysql 12288 Jul 29 13:36 sys
-rw-r----- 1 mysql mysql    22 Jul 29 13:36 xtrabackup_binlog_info
-rw-r----- 1 mysql mysql   117 Jul 29 13:36 xtrabackup_checkpoints
-rw-r----- 1 mysql mysql   593 Jul 29 13:36 xtrabackup_info
-rw-r----- 1 mysql mysql  2560 Jul 29 13:36 xtrabackup_logfile



```
####  查看备份信息

```sql

[mysql@bosenrui db_incr_backup2]$ cat xtrabackup_info
uuid = 5e0af281-92f1-11e8-a7d8-000c2916156f
name =
tool_name = innobackupex
tool_command = --defaults-file=/mysql1/mysql1.cnf -u root -p root --no-timestamp --incremental --incremental-basedir=/home/mysql/db_incr_backup1 /home/mysql/db_incr_backup2
tool_version = 2.4.8
ibbackup_version = 2.4.8
server_version = 5.7.21-log
start_time = 2018-07-29 13:36:35
end_time = 2018-07-29 13:36:39
lock_time = 0
binlog_pos = filename 'mysql-bin.000005', position '3025'
innodb_from_lsn = 2966675
innodb_to_lsn = 2967003
partial = N
incremental = Y
format = file
compact = N
compressed = N
encrypted = N


```


#### 查看备份binlog信息

```sql

[mysql@bosenrui db_incr_backup2]$ cat xtrabackup_binlog_info
mysql-bin.000005        3025



```

#### 查看检查点信息 

````sql

[mysql@bosenrui db_incr_backup2]$ cat xtrabackup_checkpoints
backup_type = incremental
from_lsn = 2966675
to_lsn = 2967003
last_lsn = 2967012
compact = 0
recover_binlog_info = 0

```

## 删除数据库

```sql

[mysql@bosenrui db_incr_backup2]$ mysqladmin  -S /tmp/mysql1.sock -uroot -proot shutdown
mysqladmin: [Warning] Using a password on the command line interface can be insecure.
[mysql@bosenrui db_incr_backup2]$ rm -rf /mysql1/data
[mysql@bosenrui db_incr_backup2]$ rm -rf /mysql1/mysql-bin.index

```

## 恢复 

### 恢复全备-apply-log

```sql

[mysql@bosenrui ~]$ innobackupex --defaults-file=/mysql1/mysql1.cnf --no-timestamp -u root -p root --apply-log /home/mysql/db_full_backup/
180729 15:02:23 innobackupex: Starting the apply-log operation

IMPORTANT: Please check that the apply-log run completes successfully.
           At the end of a successful apply-log run innobackupex
           prints "completed OK!".

innobackupex version 2.4.8 based on MySQL server 5.7.13 Linux (x86_64) (revision id: 97330f7)
xtrabackup: cd to /home/mysql/db_full_backup/
xtrabackup: This target seems to be not prepared yet.
InnoDB: Number of pools: 1
xtrabackup: xtrabackup_logfile detected: size=8388608, start_lsn=(2966331)
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
InnoDB: Log scan progressed past the checkpoint lsn 2966331
InnoDB: Doing recovery: scanned up to log sequence number 2966340 (0%)
InnoDB: Doing recovery: scanned up to log sequence number 2966340 (0%)
InnoDB: Database was not shutdown normally!
InnoDB: Starting crash recovery.
InnoDB: xtrabackup: Last MySQL binlog file position 2523, file name mysql-bin.000005
InnoDB: Creating shared tablespace for temporary tables
InnoDB: Setting file './ibtmp1' size to 12 MB. Physically writing the file full; Please wait ...
InnoDB: File './ibtmp1' size is now 12 MB.
InnoDB: 96 redo rollback segment(s) found. 1 redo rollback segment(s) are active.
InnoDB: 32 non-redo rollback segment(s) are active.
InnoDB: 5.7.13 started; log sequence number 2966340
InnoDB: xtrabackup: Last MySQL binlog file position 2523, file name mysql-bin.000005

xtrabackup: starting shutdown with innodb_fast_shutdown = 1
InnoDB: FTS optimize thread exiting.
InnoDB: Starting shutdown...
InnoDB: Shutdown completed; log sequence number 2966359
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
InnoDB: New log files created, LSN=2966359
InnoDB: Highest supported file format is Barracuda.
InnoDB: Log scan progressed past the checkpoint lsn 2966540
InnoDB: Doing recovery: scanned up to log sequence number 2966549 (0%)
InnoDB: Doing recovery: scanned up to log sequence number 2966549 (0%)
InnoDB: Database was not shutdown normally!
InnoDB: Starting crash recovery.
InnoDB: xtrabackup: Last MySQL binlog file position 2523, file name mysql-bin.000005
InnoDB: Removed temporary tablespace data file: "ibtmp1"
InnoDB: Creating shared tablespace for temporary tables
InnoDB: Setting file './ibtmp1' size to 12 MB. Physically writing the file full; Please wait ...
InnoDB: File './ibtmp1' size is now 12 MB.
InnoDB: 96 redo rollback segment(s) found. 1 redo rollback segment(s) are active.
InnoDB: 32 non-redo rollback segment(s) are active.
InnoDB: Waiting for purge to start
InnoDB: 5.7.13 started; log sequence number 2966549
xtrabackup: starting shutdown with innodb_fast_shutdown = 1
InnoDB: FTS optimize thread exiting.
InnoDB: Starting shutdown...
InnoDB: Shutdown completed; log sequence number 2966568
180729 15:02:28 completed OK!

```

### 启动数据库并查看表中数据

```sql

[mysql@bosenrui ~]$ mysqld --defaults-file=/mysql1/mysql1.cnf &
[1] 8044

[mysql@bosenrui ~]$ mysql -S /tmp/mysql1.sock -uroot -proot
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.21-log MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> use dao ;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> select * from t1 ;
+------+
| c1   |
+------+
|    1 |
+------+
1 row in set (0.00 sec)


```

#### 停止并删除数据

```sql


[mysql@bosenrui ~]$ mysqladmin  -S /tmp/mysql1.sock  -uroot -proot shutdown
mysqladmin: [Warning] Using a password on the command line interface can be insecure.
[1]+  Done                    mysqld --defaults-file=/mysql1/mysql1.cnf


[mysql@bosenrui ~]$ rm -rf /mysql1/data
[mysql@bosenrui ~]$ rm -rf /mysql1/mysql-bin.index


```
### 恢复增量数据库1

- 注意:此步骤在虚拟机进行了快照恢复 恢复到刚刚备份完的状态

#### 全备应用日志

```sql
[mysql@bosenrui ~]$ xtrabackup --prepare --apply-log-only --target-dir=/home/mysql/db_full_backup/
xtrabackup version 2.4.8 based on MySQL server 5.7.13 Linux (x86_64) (revision id: 97330f7)
xtrabackup: cd to /home/mysql/db_full_backup/
xtrabackup: This target seems to be not prepared yet.
InnoDB: Number of pools: 1
xtrabackup: xtrabackup_logfile detected: size=8388608, start_lsn=(2966331)
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
InnoDB: Log scan progressed past the checkpoint lsn 2966331
InnoDB: Doing recovery: scanned up to log sequence number 2966340 (0%)
InnoDB: Doing recovery: scanned up to log sequence number 2966340 (0%)
InnoDB: Database was not shutdown normally!
InnoDB: Starting crash recovery.
InnoDB: xtrabackup: Last MySQL binlog file position 2523, file name mysql-bin.000005
InnoDB: xtrabackup: Last MySQL binlog file position 2523, file name mysql-bin.000005

xtrabackup: starting shutdown with innodb_fast_shutdown = 1
InnoDB: Starting shutdown...
InnoDB: Shutdown completed; log sequence number 2966349
InnoDB: Number of pools: 1
180729 14:56:18 completed OK!


```

#### 应用增量1备份

```sql
[mysql@bosenrui ~]$  xtrabackup --prepare  --target-dir=/home/mysql/db_full_backup/ --incremental-dir=/home/mysql/db_
db_full_backup/  db_incr_backup1/ db_incr_backup2/
[mysql@bosenrui ~]$  xtrabackup --prepare  --target-dir=/home/mysql/db_full_backup/ --incremental-dir=/home/mysql/db_incr_backup1
xtrabackup version 2.4.8 based on MySQL server 5.7.13 Linux (x86_64) (revision id: 97330f7)
incremental backup from 2966331 is enabled.
xtrabackup: cd to /home/mysql/db_full_backup/
xtrabackup: This target seems to be already prepared with --apply-log-only.
InnoDB: Number of pools: 1
xtrabackup: xtrabackup_logfile detected: size=8388608, start_lsn=(2966675)
xtrabackup: using the following InnoDB configuration for recovery:
xtrabackup:   innodb_data_home_dir = .
xtrabackup:   innodb_data_file_path = ibdata1:12M:autoextend
xtrabackup:   innodb_log_group_home_dir = /home/mysql/db_incr_backup1/
xtrabackup:   innodb_log_files_in_group = 1
xtrabackup:   innodb_log_file_size = 8388608
xtrabackup: Generating a list of tablespaces
InnoDB: Allocated tablespace ID 8 for mysql/time_zone_name, old maximum was 0
xtrabackup: page size for /home/mysql/db_incr_backup1//ibdata1.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//ibdata1.delta to ./ibdata1...
xtrabackup: page size for /home/mysql/db_incr_backup1//mysql/time_zone_transition_type.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//mysql/time_zone_transition_type.ibd.delta to ./mysql/time_zone_transition_type.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//mysql/slave_worker_info.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//mysql/slave_worker_info.ibd.delta to ./mysql/slave_worker_info.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//mysql/engine_cost.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//mysql/engine_cost.ibd.delta to ./mysql/engine_cost.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//mysql/help_relation.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//mysql/help_relation.ibd.delta to ./mysql/help_relation.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//mysql/innodb_index_stats.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//mysql/innodb_index_stats.ibd.delta to ./mysql/innodb_index_stats.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//mysql/time_zone_transition.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//mysql/time_zone_transition.ibd.delta to ./mysql/time_zone_transition.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//mysql/help_keyword.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//mysql/help_keyword.ibd.delta to ./mysql/help_keyword.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//mysql/innodb_table_stats.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//mysql/innodb_table_stats.ibd.delta to ./mysql/innodb_table_stats.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//mysql/slave_relay_log_info.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//mysql/slave_relay_log_info.ibd.delta to ./mysql/slave_relay_log_info.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//mysql/help_topic.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//mysql/help_topic.ibd.delta to ./mysql/help_topic.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//mysql/time_zone_name.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//mysql/time_zone_name.ibd.delta to ./mysql/time_zone_name.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//mysql/slave_master_info.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//mysql/slave_master_info.ibd.delta to ./mysql/slave_master_info.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//mysql/plugin.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//mysql/plugin.ibd.delta to ./mysql/plugin.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//mysql/time_zone_leap_second.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//mysql/time_zone_leap_second.ibd.delta to ./mysql/time_zone_leap_second.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//mysql/gtid_executed.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//mysql/gtid_executed.ibd.delta to ./mysql/gtid_executed.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//mysql/help_category.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//mysql/help_category.ibd.delta to ./mysql/help_category.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//mysql/servers.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//mysql/servers.ibd.delta to ./mysql/servers.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//mysql/time_zone.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//mysql/time_zone.ibd.delta to ./mysql/time_zone.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//mysql/server_cost.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//mysql/server_cost.ibd.delta to ./mysql/server_cost.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//dao/test_import.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//dao/test_import.ibd.delta to ./dao/test_import.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//dao/test_binlog.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//dao/test_binlog.ibd.delta to ./dao/test_binlog.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//dao/matches.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//dao/matches.ibd.delta to ./dao/matches.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//dao/sal_grades.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//dao/sal_grades.ibd.delta to ./dao/sal_grades.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//dao/players.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//dao/players.ibd.delta to ./dao/players.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//dao/teams.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//dao/teams.ibd.delta to ./dao/teams.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//dao/penalties.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//dao/penalties.ibd.delta to ./dao/penalties.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//dao/committee_members.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//dao/committee_members.ibd.delta to ./dao/committee_members.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//dao/t1.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//dao/t1.ibd.delta to ./dao/t1.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//dao/test_tx_iso.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//dao/test_tx_iso.ibd.delta to ./dao/test_tx_iso.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//dao/dept.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//dao/dept.ibd.delta to ./dao/dept.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//dao/emp.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//dao/emp.ibd.delta to ./dao/emp.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//sys/sys_config.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//sys/sys_config.ibd.delta to ./sys/sys_config.ibd...
xtrabackup: using the following InnoDB configuration for recovery:
xtrabackup:   innodb_data_home_dir = .
xtrabackup:   innodb_data_file_path = ibdata1:12M:autoextend
xtrabackup:   innodb_log_group_home_dir = /home/mysql/db_incr_backup1/
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
InnoDB: Log scan progressed past the checkpoint lsn 2966675
InnoDB: Doing recovery: scanned up to log sequence number 2966684 (0%)
InnoDB: Doing recovery: scanned up to log sequence number 2966684 (0%)
InnoDB: Database was not shutdown normally!
InnoDB: Starting crash recovery.
InnoDB: xtrabackup: Last MySQL binlog file position 2774, file name mysql-bin.000005
InnoDB: Creating shared tablespace for temporary tables
InnoDB: Setting file './ibtmp1' size to 12 MB. Physically writing the file full; Please wait ...
InnoDB: File './ibtmp1' size is now 12 MB.
InnoDB: 96 redo rollback segment(s) found. 1 redo rollback segment(s) are active.
InnoDB: 32 non-redo rollback segment(s) are active.
InnoDB: Waiting for purge to start
InnoDB: 5.7.13 started; log sequence number 2966684
InnoDB: xtrabackup: Last MySQL binlog file position 2774, file name mysql-bin.000005

xtrabackup: starting shutdown with innodb_fast_shutdown = 1
InnoDB: FTS optimize thread exiting.
InnoDB: Starting shutdown...
InnoDB: Shutdown completed; log sequence number 2966703
InnoDB: Number of pools: 1
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/replication_applier_status.frm to ./performance_schema/replication_applier_status.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/file_instances.frm to ./performance_schema/file_instances.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/metadata_locks.frm to ./performance_schema/metadata_locks.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/memory_summary_by_account_by_event_name.frm to ./performance_schema/memory_summary_by_account_by_event_name.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/threads.frm to ./performance_schema/threads.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_waits_summary_by_user_by_event_name.frm to ./performance_schema/events_waits_summary_by_user_by_event_name.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_stages_summary_by_account_by_event_name.frm to ./performance_schema/events_stages_summary_by_account_by_event_name.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/db.opt to ./performance_schema/db.opt
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/table_io_waits_summary_by_table.frm to ./performance_schema/table_io_waits_summary_by_table.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/memory_summary_global_by_event_name.frm to ./performance_schema/memory_summary_global_by_event_name.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_stages_history_long.frm to ./performance_schema/events_stages_history_long.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_statements_summary_by_program.frm to ./performance_schema/events_statements_summary_by_program.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_statements_summary_by_user_by_event_name.frm to ./performance_schema/events_statements_summary_by_user_by_event_name.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/session_account_connect_attrs.frm to ./performance_schema/session_account_connect_attrs.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/session_connect_attrs.frm to ./performance_schema/session_connect_attrs.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_statements_summary_by_account_by_event_name.frm to ./performance_schema/events_statements_summary_by_account_by_event_name.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_statements_summary_by_thread_by_event_name.frm to ./performance_schema/events_statements_summary_by_thread_by_event_name.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_transactions_history.frm to ./performance_schema/events_transactions_history.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/memory_summary_by_user_by_event_name.frm to ./performance_schema/memory_summary_by_user_by_event_name.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_statements_summary_global_by_event_name.frm to ./performance_schema/events_statements_summary_global_by_event_name.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/performance_timers.frm to ./performance_schema/performance_timers.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/setup_consumers.frm to ./performance_schema/setup_consumers.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_stages_summary_by_host_by_event_name.frm to ./performance_schema/events_stages_summary_by_host_by_event_name.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_transactions_history_long.frm to ./performance_schema/events_transactions_history_long.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/replication_connection_status.frm to ./performance_schema/replication_connection_status.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/replication_group_members.frm to ./performance_schema/replication_group_members.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/objects_summary_global_by_type.frm to ./performance_schema/objects_summary_global_by_type.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/rwlock_instances.frm to ./performance_schema/rwlock_instances.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/user_variables_by_thread.frm to ./performance_schema/user_variables_by_thread.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_waits_current.frm to ./performance_schema/events_waits_current.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/file_summary_by_event_name.frm to ./performance_schema/file_summary_by_event_name.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/users.frm to ./performance_schema/users.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/replication_applier_status_by_worker.frm to ./performance_schema/replication_applier_status_by_worker.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/socket_instances.frm to ./performance_schema/socket_instances.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_stages_current.frm to ./performance_schema/events_stages_current.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/global_status.frm to ./performance_schema/global_status.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/setup_timers.frm to ./performance_schema/setup_timers.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_waits_history.frm to ./performance_schema/events_waits_history.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/table_handles.frm to ./performance_schema/table_handles.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_statements_current.frm to ./performance_schema/events_statements_current.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_transactions_current.frm to ./performance_schema/events_transactions_current.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_waits_summary_global_by_event_name.frm to ./performance_schema/events_waits_summary_global_by_event_name.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/replication_applier_status_by_coordinator.frm to ./performance_schema/replication_applier_status_by_coordinator.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_statements_summary_by_host_by_event_name.frm to ./performance_schema/events_statements_summary_by_host_by_event_name.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_stages_history.frm to ./performance_schema/events_stages_history.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_statements_summary_by_digest.frm to ./performance_schema/events_statements_summary_by_digest.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/accounts.frm to ./performance_schema/accounts.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/replication_applier_configuration.frm to ./performance_schema/replication_applier_configuration.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/replication_connection_configuration.frm to ./performance_schema/replication_connection_configuration.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/status_by_user.frm to ./performance_schema/status_by_user.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_transactions_summary_global_by_event_name.frm to ./performance_schema/events_transactions_summary_global_by_event_name.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_transactions_summary_by_account_by_event_name.frm to ./performance_schema/events_transactions_summary_by_account_by_event_name.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_stages_summary_global_by_event_name.frm to ./performance_schema/events_stages_summary_global_by_event_name.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/socket_summary_by_event_name.frm to ./performance_schema/socket_summary_by_event_name.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/memory_summary_by_thread_by_event_name.frm to ./performance_schema/memory_summary_by_thread_by_event_name.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/status_by_thread.frm to ./performance_schema/status_by_thread.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_waits_summary_by_instance.frm to ./performance_schema/events_waits_summary_by_instance.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_transactions_summary_by_thread_by_event_name.frm to ./performance_schema/events_transactions_summary_by_thread_by_event_name.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/hosts.frm to ./performance_schema/hosts.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/cond_instances.frm to ./performance_schema/cond_instances.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/memory_summary_by_host_by_event_name.frm to ./performance_schema/memory_summary_by_host_by_event_name.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/status_by_account.frm to ./performance_schema/status_by_account.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/socket_summary_by_instance.frm to ./performance_schema/socket_summary_by_instance.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_transactions_summary_by_user_by_event_name.frm to ./performance_schema/events_transactions_summary_by_user_by_event_name.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/status_by_host.frm to ./performance_schema/status_by_host.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_stages_summary_by_user_by_event_name.frm to ./performance_schema/events_stages_summary_by_user_by_event_name.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/global_variables.frm to ./performance_schema/global_variables.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/setup_instruments.frm to ./performance_schema/setup_instruments.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_waits_summary_by_thread_by_event_name.frm to ./performance_schema/events_waits_summary_by_thread_by_event_name.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/replication_group_member_stats.frm to ./performance_schema/replication_group_member_stats.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/host_cache.frm to ./performance_schema/host_cache.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/setup_objects.frm to ./performance_schema/setup_objects.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_waits_history_long.frm to ./performance_schema/events_waits_history_long.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/session_status.frm to ./performance_schema/session_status.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_transactions_summary_by_host_by_event_name.frm to ./performance_schema/events_transactions_summary_by_host_by_event_name.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/prepared_statements_instances.frm to ./performance_schema/prepared_statements_instances.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/setup_actors.frm to ./performance_schema/setup_actors.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/mutex_instances.frm to ./performance_schema/mutex_instances.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/table_lock_waits_summary_by_table.frm to ./performance_schema/table_lock_waits_summary_by_table.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/variables_by_thread.frm to ./performance_schema/variables_by_thread.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_waits_summary_by_host_by_event_name.frm to ./performance_schema/events_waits_summary_by_host_by_event_name.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_statements_history_long.frm to ./performance_schema/events_statements_history_long.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_waits_summary_by_account_by_event_name.frm to ./performance_schema/events_waits_summary_by_account_by_event_name.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/session_variables.frm to ./performance_schema/session_variables.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_stages_summary_by_thread_by_event_name.frm to ./performance_schema/events_stages_summary_by_thread_by_event_name.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/file_summary_by_instance.frm to ./performance_schema/file_summary_by_instance.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_statements_history.frm to ./performance_schema/events_statements_history.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/performance_schema/table_io_waits_summary_by_index_usage.frm to ./performance_schema/table_io_waits_summary_by_index_usage.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/mysql/user.frm to ./mysql/user.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/mysql/func.MYI to ./mysql/func.MYI
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/mysql/help_topic.frm to ./mysql/help_topic.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/mysql/proxies_priv.MYI to ./mysql/proxies_priv.MYI
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/mysql/help_keyword.frm to ./mysql/help_keyword.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/mysql/func.frm to ./mysql/func.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/mysql/ndb_binlog_index.MYD to ./mysql/ndb_binlog_index.MYD
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/mysql/tables_priv.MYI to ./mysql/tables_priv.MYI
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/mysql/db.opt to ./mysql/db.opt
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/mysql/db.MYD to ./mysql/db.MYD
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/mysql/engine_cost.frm to ./mysql/engine_cost.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/mysql/ndb_binlog_index.MYI to ./mysql/ndb_binlog_index.MYI
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/mysql/procs_priv.MYI to ./mysql/procs_priv.MYI
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/mysql/help_category.frm to ./mysql/help_category.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/mysql/ndb_binlog_index.frm to ./mysql/ndb_binlog_index.frm
180729 14:58:53 [01]        ...done
180729 14:58:53 [01] Copying /home/mysql/db_incr_backup1/mysql/user.MYI to ./mysql/user.MYI
180729 14:58:53 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/mysql/event.MYI to ./mysql/event.MYI
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/mysql/tables_priv.frm to ./mysql/tables_priv.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/mysql/slave_master_info.frm to ./mysql/slave_master_info.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/mysql/time_zone_transition_type.frm to ./mysql/time_zone_transition_type.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/mysql/columns_priv.MYD to ./mysql/columns_priv.MYD
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/mysql/tables_priv.MYD to ./mysql/tables_priv.MYD
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/mysql/proxies_priv.frm to ./mysql/proxies_priv.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/mysql/event.frm to ./mysql/event.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/mysql/slow_log.CSV to ./mysql/slow_log.CSV
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/mysql/proc.MYI to ./mysql/proc.MYI
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/mysql/func.MYD to ./mysql/func.MYD
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/mysql/proc.frm to ./mysql/proc.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/mysql/slave_relay_log_info.frm to ./mysql/slave_relay_log_info.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/mysql/time_zone.frm to ./mysql/time_zone.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/mysql/db.MYI to ./mysql/db.MYI
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/mysql/gtid_executed.frm to ./mysql/gtid_executed.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/mysql/general_log.CSM to ./mysql/general_log.CSM
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/mysql/db.frm to ./mysql/db.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/mysql/plugin.frm to ./mysql/plugin.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/mysql/time_zone_leap_second.frm to ./mysql/time_zone_leap_second.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/mysql/general_log.frm to ./mysql/general_log.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/mysql/server_cost.frm to ./mysql/server_cost.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/mysql/servers.frm to ./mysql/servers.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/mysql/innodb_table_stats.frm to ./mysql/innodb_table_stats.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/mysql/columns_priv.frm to ./mysql/columns_priv.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/mysql/user.MYD to ./mysql/user.MYD
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/mysql/slave_worker_info.frm to ./mysql/slave_worker_info.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/mysql/time_zone_transition.frm to ./mysql/time_zone_transition.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/mysql/time_zone_name.frm to ./mysql/time_zone_name.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/mysql/slow_log.frm to ./mysql/slow_log.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/mysql/help_relation.frm to ./mysql/help_relation.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/mysql/proxies_priv.MYD to ./mysql/proxies_priv.MYD
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/mysql/event.MYD to ./mysql/event.MYD
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/mysql/slow_log.CSM to ./mysql/slow_log.CSM
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/mysql/innodb_index_stats.frm to ./mysql/innodb_index_stats.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/mysql/general_log.CSV to ./mysql/general_log.CSV
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/mysql/proc.MYD to ./mysql/proc.MYD
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/mysql/procs_priv.frm to ./mysql/procs_priv.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/mysql/procs_priv.MYD to ./mysql/procs_priv.MYD
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/mysql/columns_priv.MYI to ./mysql/columns_priv.MYI
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/dao/sal_grades.frm to ./dao/sal_grades.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/dao/db.opt to ./dao/db.opt
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/dao/penalties.frm to ./dao/penalties.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/dao/dept.frm to ./dao/dept.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/dao/matches.frm to ./dao/matches.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/dao/test_import.frm to ./dao/test_import.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/dao/test_tx_iso.frm to ./dao/test_tx_iso.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/dao/teams.frm to ./dao/teams.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/dao/committee_members.frm to ./dao/committee_members.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/dao/t1.frm to ./dao/t1.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/dao/test_binlog.frm to ./dao/test_binlog.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/dao/players.frm to ./dao/players.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/dao/emp.frm to ./dao/emp.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024innodb_buffer_stats_by_schema.frm to ./sys/x@0024innodb_buffer_stats_by_schema.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024statement_analysis.frm to ./sys/x@0024statement_analysis.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024user_summary.frm to ./sys/x@0024user_summary.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024schema_index_statistics.frm to ./sys/x@0024schema_index_statistics.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024statements_with_runtimes_in_95th_percentile.frm to ./sys/x@0024statements_with_runtimes_in_95th_percentile.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024innodb_buffer_stats_by_table.frm to ./sys/x@0024innodb_buffer_stats_by_table.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/db.opt to ./sys/db.opt
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/host_summary_by_statement_type.frm to ./sys/host_summary_by_statement_type.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/sys_config_insert_set_user.TRN to ./sys/sys_config_insert_set_user.TRN
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/statement_analysis.frm to ./sys/statement_analysis.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/version.frm to ./sys/version.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/host_summary.frm to ./sys/host_summary.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024io_global_by_wait_by_latency.frm to ./sys/x@0024io_global_by_wait_by_latency.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/statements_with_sorting.frm to ./sys/statements_with_sorting.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/sys_config.frm to ./sys/sys_config.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024io_by_thread_by_latency.frm to ./sys/x@0024io_by_thread_by_latency.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/memory_by_host_by_current_bytes.frm to ./sys/memory_by_host_by_current_bytes.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024memory_by_thread_by_current_bytes.frm to ./sys/x@0024memory_by_thread_by_current_bytes.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024schema_flattened_keys.frm to ./sys/x@0024schema_flattened_keys.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024memory_by_host_by_current_bytes.frm to ./sys/x@0024memory_by_host_by_current_bytes.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/sys_config.TRG to ./sys/sys_config.TRG
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/user_summary_by_file_io.frm to ./sys/user_summary_by_file_io.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024latest_file_io.frm to ./sys/x@0024latest_file_io.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/schema_object_overview.frm to ./sys/schema_object_overview.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/schema_unused_indexes.frm to ./sys/schema_unused_indexes.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024processlist.frm to ./sys/x@0024processlist.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/statements_with_full_table_scans.frm to ./sys/statements_with_full_table_scans.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024host_summary_by_file_io.frm to ./sys/x@0024host_summary_by_file_io.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024host_summary.frm to ./sys/x@0024host_summary.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/memory_by_user_by_current_bytes.frm to ./sys/memory_by_user_by_current_bytes.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/wait_classes_global_by_latency.frm to ./sys/wait_classes_global_by_latency.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024io_global_by_wait_by_bytes.frm to ./sys/x@0024io_global_by_wait_by_bytes.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024ps_schema_table_statistics_io.frm to ./sys/x@0024ps_schema_table_statistics_io.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024schema_table_statistics_with_buffer.frm to ./sys/x@0024schema_table_statistics_with_buffer.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/session_ssl_status.frm to ./sys/session_ssl_status.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/innodb_buffer_stats_by_schema.frm to ./sys/innodb_buffer_stats_by_schema.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024waits_by_user_by_latency.frm to ./sys/x@0024waits_by_user_by_latency.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024host_summary_by_stages.frm to ./sys/x@0024host_summary_by_stages.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024waits_global_by_latency.frm to ./sys/x@0024waits_global_by_latency.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/schema_tables_with_full_table_scans.frm to ./sys/schema_tables_with_full_table_scans.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024schema_table_lock_waits.frm to ./sys/x@0024schema_table_lock_waits.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024memory_by_user_by_current_bytes.frm to ./sys/x@0024memory_by_user_by_current_bytes.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/innodb_lock_waits.frm to ./sys/innodb_lock_waits.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024ps_digest_avg_latency_distribution.frm to ./sys/x@0024ps_digest_avg_latency_distribution.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024schema_table_statistics.frm to ./sys/x@0024schema_table_statistics.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024host_summary_by_file_io_type.frm to ./sys/x@0024host_summary_by_file_io_type.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/latest_file_io.frm to ./sys/latest_file_io.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/schema_redundant_indexes.frm to ./sys/schema_redundant_indexes.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/memory_global_by_current_bytes.frm to ./sys/memory_global_by_current_bytes.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024memory_global_by_current_bytes.frm to ./sys/x@0024memory_global_by_current_bytes.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/statements_with_errors_or_warnings.frm to ./sys/statements_with_errors_or_warnings.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024wait_classes_global_by_avg_latency.frm to ./sys/x@0024wait_classes_global_by_avg_latency.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024wait_classes_global_by_latency.frm to ./sys/x@0024wait_classes_global_by_latency.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024session.frm to ./sys/x@0024session.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024schema_tables_with_full_table_scans.frm to ./sys/x@0024schema_tables_with_full_table_scans.frm
180729 14:58:54 [01]        ...done
180729 14:58:54 [01] Copying /home/mysql/db_incr_backup1/sys/host_summary_by_file_io_type.frm to ./sys/host_summary_by_file_io_type.frm
180729 14:58:54 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/user_summary.frm to ./sys/user_summary.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024user_summary_by_stages.frm to ./sys/x@0024user_summary_by_stages.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/user_summary_by_file_io_type.frm to ./sys/user_summary_by_file_io_type.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/schema_table_statistics.frm to ./sys/schema_table_statistics.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/waits_by_user_by_latency.frm to ./sys/waits_by_user_by_latency.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/io_global_by_wait_by_bytes.frm to ./sys/io_global_by_wait_by_bytes.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/user_summary_by_statement_type.frm to ./sys/user_summary_by_statement_type.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024statements_with_sorting.frm to ./sys/x@0024statements_with_sorting.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/user_summary_by_statement_latency.frm to ./sys/user_summary_by_statement_latency.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024user_summary_by_statement_latency.frm to ./sys/x@0024user_summary_by_statement_latency.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/session.frm to ./sys/session.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/schema_table_statistics_with_buffer.frm to ./sys/schema_table_statistics_with_buffer.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/host_summary_by_file_io.frm to ./sys/host_summary_by_file_io.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/memory_by_thread_by_current_bytes.frm to ./sys/memory_by_thread_by_current_bytes.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024waits_by_host_by_latency.frm to ./sys/x@0024waits_by_host_by_latency.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/schema_auto_increment_columns.frm to ./sys/schema_auto_increment_columns.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024ps_digest_95th_percentile_by_avg_us.frm to ./sys/x@0024ps_digest_95th_percentile_by_avg_us.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/waits_by_host_by_latency.frm to ./sys/waits_by_host_by_latency.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/host_summary_by_statement_latency.frm to ./sys/host_summary_by_statement_latency.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/metrics.frm to ./sys/metrics.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024user_summary_by_file_io_type.frm to ./sys/x@0024user_summary_by_file_io_type.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/io_global_by_file_by_bytes.frm to ./sys/io_global_by_file_by_bytes.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/io_by_thread_by_latency.frm to ./sys/io_by_thread_by_latency.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/processlist.frm to ./sys/processlist.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/statements_with_runtimes_in_95th_percentile.frm to ./sys/statements_with_runtimes_in_95th_percentile.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/schema_table_lock_waits.frm to ./sys/schema_table_lock_waits.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/wait_classes_global_by_avg_latency.frm to ./sys/wait_classes_global_by_avg_latency.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/user_summary_by_stages.frm to ./sys/user_summary_by_stages.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/io_global_by_file_by_latency.frm to ./sys/io_global_by_file_by_latency.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024host_summary_by_statement_type.frm to ./sys/x@0024host_summary_by_statement_type.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024innodb_lock_waits.frm to ./sys/x@0024innodb_lock_waits.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/ps_check_lost_instrumentation.frm to ./sys/ps_check_lost_instrumentation.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/waits_global_by_latency.frm to ./sys/waits_global_by_latency.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/sys_config_update_set_user.TRN to ./sys/sys_config_update_set_user.TRN
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/innodb_buffer_stats_by_table.frm to ./sys/innodb_buffer_stats_by_table.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024statements_with_temp_tables.frm to ./sys/x@0024statements_with_temp_tables.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/io_global_by_wait_by_latency.frm to ./sys/io_global_by_wait_by_latency.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/host_summary_by_stages.frm to ./sys/host_summary_by_stages.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/statements_with_temp_tables.frm to ./sys/statements_with_temp_tables.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024user_summary_by_file_io.frm to ./sys/x@0024user_summary_by_file_io.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/schema_index_statistics.frm to ./sys/schema_index_statistics.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024host_summary_by_statement_latency.frm to ./sys/x@0024host_summary_by_statement_latency.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024user_summary_by_statement_type.frm to ./sys/x@0024user_summary_by_statement_type.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024statements_with_errors_or_warnings.frm to ./sys/x@0024statements_with_errors_or_warnings.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024io_global_by_file_by_latency.frm to ./sys/x@0024io_global_by_file_by_latency.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024io_global_by_file_by_bytes.frm to ./sys/x@0024io_global_by_file_by_bytes.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024memory_global_total.frm to ./sys/x@0024memory_global_total.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/memory_global_total.frm to ./sys/memory_global_total.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024statements_with_full_table_scans.frm to ./sys/x@0024statements_with_full_table_scans.frm
180729 14:58:55 [01]        ...done
180729 14:58:55 [00] Copying /home/mysql/db_incr_backup1//xtrabackup_binlog_info to ./xtrabackup_binlog_info
180729 14:58:55 [00]        ...done
180729 14:58:55 [00] Copying /home/mysql/db_incr_backup1//xtrabackup_info to ./xtrabackup_info
180729 14:58:55 [00]        ...done
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
InnoDB: New log files created, LSN=2966703
InnoDB: Highest supported file format is Barracuda.
InnoDB: Log scan progressed past the checkpoint lsn 2967052
InnoDB: Doing recovery: scanned up to log sequence number 2967061 (0%)
InnoDB: Doing recovery: scanned up to log sequence number 2967061 (0%)
InnoDB: Database was not shutdown normally!
InnoDB: Starting crash recovery.
InnoDB: xtrabackup: Last MySQL binlog file position 2774, file name mysql-bin.000005
InnoDB: Removed temporary tablespace data file: "ibtmp1"
InnoDB: Creating shared tablespace for temporary tables
InnoDB: Setting file './ibtmp1' size to 12 MB. Physically writing the file full; Please wait ...
InnoDB: File './ibtmp1' size is now 12 MB.
InnoDB: 96 redo rollback segment(s) found. 1 redo rollback segment(s) are active.
InnoDB: 32 non-redo rollback segment(s) are active.
InnoDB: Waiting for purge to start
InnoDB: 5.7.13 started; log sequence number 2967061
xtrabackup: starting shutdown with innodb_fast_shutdown = 1
InnoDB: FTS optimize thread exiting.
InnoDB: Starting shutdown...
InnoDB: Shutdown completed; log sequence number 2967080
180729 14:58:57 completed OK!

```

#### 还原备份,启动数据库,查看表中数据

```sql
[mysql@bosenrui ~]$ cp -r /home/mysql/db_full_backup /mysql1/data
[mysql@bosenrui ~]$ mysqld --defaults-file=/mysql1/mysql1.cnf &
[1] 8097


[mysql@bosenrui ~]$ mysql -S /tmp/mysql1.sock -uroot -proot
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.7.21-log MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> use dao ;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> select  * from t1 ;
+------+
| c1   |
+------+
|    1 |
|    2 |
+------+
2 rows in set (0.00 sec)

```

### 恢复增量数据库2

- 注意:此步骤在虚拟机进行了快照恢复 恢复到刚刚备份完的状态
 

#### 全备应用日志

```sql

[root@bosenrui ~]# su - mysql
[mysql@bosenrui ~]$ xtrabackup --prepare --apply-log-only --target-dir=/home/mysql/db_full_backup/
xtrabackup version 2.4.8 based on MySQL server 5.7.13 Linux (x86_64) (revision id: 97330f7)
xtrabackup: cd to /home/mysql/db_full_backup/
xtrabackup: This target seems to be not prepared yet.
InnoDB: Number of pools: 1
xtrabackup: xtrabackup_logfile detected: size=8388608, start_lsn=(2966331)
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
InnoDB: Log scan progressed past the checkpoint lsn 2966331
InnoDB: Doing recovery: scanned up to log sequence number 2966340 (0%)
InnoDB: Doing recovery: scanned up to log sequence number 2966340 (0%)
InnoDB: Database was not shutdown normally!
InnoDB: Starting crash recovery.
InnoDB: xtrabackup: Last MySQL binlog file position 2523, file name mysql-bin.000005
InnoDB: xtrabackup: Last MySQL binlog file position 2523, file name mysql-bin.000005

xtrabackup: starting shutdown with innodb_fast_shutdown = 1
InnoDB: Starting shutdown...
InnoDB: Shutdown completed; log sequence number 2966349
InnoDB: Number of pools: 1
180729 15:49:33 completed OK!

```

#### 应用增量备份1

```sql

[mysql@bosenrui ~]$ xtrabackup --prepare --apply-log-only --target-dir=/home/mysql/db_full_backup --incremental-dir=/home/mysql/db_incr_backup1
xtrabackup version 2.4.8 based on MySQL server 5.7.13 Linux (x86_64) (revision id: 97330f7)
incremental backup from 2966331 is enabled.
xtrabackup: cd to /home/mysql/db_full_backup/
xtrabackup: This target seems to be already prepared with --apply-log-only.
InnoDB: Number of pools: 1
xtrabackup: xtrabackup_logfile detected: size=8388608, start_lsn=(2966675)
xtrabackup: using the following InnoDB configuration for recovery:
xtrabackup:   innodb_data_home_dir = .
xtrabackup:   innodb_data_file_path = ibdata1:12M:autoextend
xtrabackup:   innodb_log_group_home_dir = /home/mysql/db_incr_backup1/
xtrabackup:   innodb_log_files_in_group = 1
xtrabackup:   innodb_log_file_size = 8388608
xtrabackup: Generating a list of tablespaces
InnoDB: Allocated tablespace ID 8 for mysql/time_zone_name, old maximum was 0
xtrabackup: page size for /home/mysql/db_incr_backup1//ibdata1.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//ibdata1.delta to ./ibdata1...
xtrabackup: page size for /home/mysql/db_incr_backup1//mysql/time_zone_transition_type.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//mysql/time_zone_transition_type.ibd.delta to ./mysql/time_zone_transition_type.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//mysql/slave_worker_info.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//mysql/slave_worker_info.ibd.delta to ./mysql/slave_worker_info.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//mysql/engine_cost.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//mysql/engine_cost.ibd.delta to ./mysql/engine_cost.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//mysql/help_relation.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//mysql/help_relation.ibd.delta to ./mysql/help_relation.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//mysql/innodb_index_stats.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//mysql/innodb_index_stats.ibd.delta to ./mysql/innodb_index_stats.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//mysql/time_zone_transition.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//mysql/time_zone_transition.ibd.delta to ./mysql/time_zone_transition.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//mysql/help_keyword.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//mysql/help_keyword.ibd.delta to ./mysql/help_keyword.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//mysql/innodb_table_stats.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//mysql/innodb_table_stats.ibd.delta to ./mysql/innodb_table_stats.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//mysql/slave_relay_log_info.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//mysql/slave_relay_log_info.ibd.delta to ./mysql/slave_relay_log_info.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//mysql/help_topic.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//mysql/help_topic.ibd.delta to ./mysql/help_topic.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//mysql/time_zone_name.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//mysql/time_zone_name.ibd.delta to ./mysql/time_zone_name.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//mysql/slave_master_info.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//mysql/slave_master_info.ibd.delta to ./mysql/slave_master_info.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//mysql/plugin.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//mysql/plugin.ibd.delta to ./mysql/plugin.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//mysql/time_zone_leap_second.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//mysql/time_zone_leap_second.ibd.delta to ./mysql/time_zone_leap_second.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//mysql/gtid_executed.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//mysql/gtid_executed.ibd.delta to ./mysql/gtid_executed.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//mysql/help_category.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//mysql/help_category.ibd.delta to ./mysql/help_category.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//mysql/servers.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//mysql/servers.ibd.delta to ./mysql/servers.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//mysql/time_zone.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//mysql/time_zone.ibd.delta to ./mysql/time_zone.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//mysql/server_cost.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//mysql/server_cost.ibd.delta to ./mysql/server_cost.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//dao/test_import.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//dao/test_import.ibd.delta to ./dao/test_import.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//dao/test_binlog.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//dao/test_binlog.ibd.delta to ./dao/test_binlog.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//dao/matches.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//dao/matches.ibd.delta to ./dao/matches.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//dao/sal_grades.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//dao/sal_grades.ibd.delta to ./dao/sal_grades.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//dao/players.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//dao/players.ibd.delta to ./dao/players.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//dao/teams.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//dao/teams.ibd.delta to ./dao/teams.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//dao/penalties.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//dao/penalties.ibd.delta to ./dao/penalties.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//dao/committee_members.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//dao/committee_members.ibd.delta to ./dao/committee_members.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//dao/t1.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//dao/t1.ibd.delta to ./dao/t1.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//dao/test_tx_iso.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//dao/test_tx_iso.ibd.delta to ./dao/test_tx_iso.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//dao/dept.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//dao/dept.ibd.delta to ./dao/dept.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//dao/emp.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//dao/emp.ibd.delta to ./dao/emp.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup1//sys/sys_config.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup1//sys/sys_config.ibd.delta to ./sys/sys_config.ibd...
xtrabackup: using the following InnoDB configuration for recovery:
xtrabackup:   innodb_data_home_dir = .
xtrabackup:   innodb_data_file_path = ibdata1:12M:autoextend
xtrabackup:   innodb_log_group_home_dir = /home/mysql/db_incr_backup1/
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
InnoDB: Log scan progressed past the checkpoint lsn 2966675
InnoDB: Doing recovery: scanned up to log sequence number 2966684 (0%)
InnoDB: Doing recovery: scanned up to log sequence number 2966684 (0%)
InnoDB: Database was not shutdown normally!
InnoDB: Starting crash recovery.
InnoDB: xtrabackup: Last MySQL binlog file position 2774, file name mysql-bin.000005
InnoDB: xtrabackup: Last MySQL binlog file position 2774, file name mysql-bin.000005

xtrabackup: starting shutdown with innodb_fast_shutdown = 1
InnoDB: Starting shutdown...
InnoDB: Shutdown completed; log sequence number 2966693
InnoDB: Number of pools: 1
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/replication_applier_status.frm to ./performance_schema/replication_applier_status.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/file_instances.frm to ./performance_schema/file_instances.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/metadata_locks.frm to ./performance_schema/metadata_locks.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/memory_summary_by_account_by_event_name.frm to ./performance_schema/memory_summary_by_account_by_event_name.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/threads.frm to ./performance_schema/threads.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_waits_summary_by_user_by_event_name.frm to ./performance_schema/events_waits_summary_by_user_by_event_name.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_stages_summary_by_account_by_event_name.frm to ./performance_schema/events_stages_summary_by_account_by_event_name.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/db.opt to ./performance_schema/db.opt
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/table_io_waits_summary_by_table.frm to ./performance_schema/table_io_waits_summary_by_table.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/memory_summary_global_by_event_name.frm to ./performance_schema/memory_summary_global_by_event_name.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_stages_history_long.frm to ./performance_schema/events_stages_history_long.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_statements_summary_by_program.frm to ./performance_schema/events_statements_summary_by_program.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_statements_summary_by_user_by_event_name.frm to ./performance_schema/events_statements_summary_by_user_by_event_name.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/session_account_connect_attrs.frm to ./performance_schema/session_account_connect_attrs.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/session_connect_attrs.frm to ./performance_schema/session_connect_attrs.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_statements_summary_by_account_by_event_name.frm to ./performance_schema/events_statements_summary_by_account_by_event_name.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_statements_summary_by_thread_by_event_name.frm to ./performance_schema/events_statements_summary_by_thread_by_event_name.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_transactions_history.frm to ./performance_schema/events_transactions_history.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/memory_summary_by_user_by_event_name.frm to ./performance_schema/memory_summary_by_user_by_event_name.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_statements_summary_global_by_event_name.frm to ./performance_schema/events_statements_summary_global_by_event_name.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/performance_timers.frm to ./performance_schema/performance_timers.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/setup_consumers.frm to ./performance_schema/setup_consumers.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_stages_summary_by_host_by_event_name.frm to ./performance_schema/events_stages_summary_by_host_by_event_name.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_transactions_history_long.frm to ./performance_schema/events_transactions_history_long.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/replication_connection_status.frm to ./performance_schema/replication_connection_status.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/replication_group_members.frm to ./performance_schema/replication_group_members.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/objects_summary_global_by_type.frm to ./performance_schema/objects_summary_global_by_type.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/rwlock_instances.frm to ./performance_schema/rwlock_instances.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/user_variables_by_thread.frm to ./performance_schema/user_variables_by_thread.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_waits_current.frm to ./performance_schema/events_waits_current.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/file_summary_by_event_name.frm to ./performance_schema/file_summary_by_event_name.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/users.frm to ./performance_schema/users.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/replication_applier_status_by_worker.frm to ./performance_schema/replication_applier_status_by_worker.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/socket_instances.frm to ./performance_schema/socket_instances.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_stages_current.frm to ./performance_schema/events_stages_current.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/global_status.frm to ./performance_schema/global_status.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/setup_timers.frm to ./performance_schema/setup_timers.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_waits_history.frm to ./performance_schema/events_waits_history.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/table_handles.frm to ./performance_schema/table_handles.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_statements_current.frm to ./performance_schema/events_statements_current.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_transactions_current.frm to ./performance_schema/events_transactions_current.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_waits_summary_global_by_event_name.frm to ./performance_schema/events_waits_summary_global_by_event_name.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/replication_applier_status_by_coordinator.frm to ./performance_schema/replication_applier_status_by_coordinator.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_statements_summary_by_host_by_event_name.frm to ./performance_schema/events_statements_summary_by_host_by_event_name.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_stages_history.frm to ./performance_schema/events_stages_history.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_statements_summary_by_digest.frm to ./performance_schema/events_statements_summary_by_digest.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/accounts.frm to ./performance_schema/accounts.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/replication_applier_configuration.frm to ./performance_schema/replication_applier_configuration.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/replication_connection_configuration.frm to ./performance_schema/replication_connection_configuration.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/status_by_user.frm to ./performance_schema/status_by_user.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_transactions_summary_global_by_event_name.frm to ./performance_schema/events_transactions_summary_global_by_event_name.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_transactions_summary_by_account_by_event_name.frm to ./performance_schema/events_transactions_summary_by_account_by_event_name.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_stages_summary_global_by_event_name.frm to ./performance_schema/events_stages_summary_global_by_event_name.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/socket_summary_by_event_name.frm to ./performance_schema/socket_summary_by_event_name.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/memory_summary_by_thread_by_event_name.frm to ./performance_schema/memory_summary_by_thread_by_event_name.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/status_by_thread.frm to ./performance_schema/status_by_thread.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_waits_summary_by_instance.frm to ./performance_schema/events_waits_summary_by_instance.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_transactions_summary_by_thread_by_event_name.frm to ./performance_schema/events_transactions_summary_by_thread_by_event_name.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/hosts.frm to ./performance_schema/hosts.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/cond_instances.frm to ./performance_schema/cond_instances.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/memory_summary_by_host_by_event_name.frm to ./performance_schema/memory_summary_by_host_by_event_name.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/status_by_account.frm to ./performance_schema/status_by_account.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/socket_summary_by_instance.frm to ./performance_schema/socket_summary_by_instance.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_transactions_summary_by_user_by_event_name.frm to ./performance_schema/events_transactions_summary_by_user_by_event_name.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/status_by_host.frm to ./performance_schema/status_by_host.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_stages_summary_by_user_by_event_name.frm to ./performance_schema/events_stages_summary_by_user_by_event_name.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/global_variables.frm to ./performance_schema/global_variables.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/setup_instruments.frm to ./performance_schema/setup_instruments.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_waits_summary_by_thread_by_event_name.frm to ./performance_schema/events_waits_summary_by_thread_by_event_name.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/replication_group_member_stats.frm to ./performance_schema/replication_group_member_stats.frm
180729 15:51:05 [01]        ...done
180729 15:51:05 [01] Copying /home/mysql/db_incr_backup1/performance_schema/host_cache.frm to ./performance_schema/host_cache.frm
180729 15:51:05 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/performance_schema/setup_objects.frm to ./performance_schema/setup_objects.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_waits_history_long.frm to ./performance_schema/events_waits_history_long.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/performance_schema/session_status.frm to ./performance_schema/session_status.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_transactions_summary_by_host_by_event_name.frm to ./performance_schema/events_transactions_summary_by_host_by_event_name.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/performance_schema/prepared_statements_instances.frm to ./performance_schema/prepared_statements_instances.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/performance_schema/setup_actors.frm to ./performance_schema/setup_actors.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/performance_schema/mutex_instances.frm to ./performance_schema/mutex_instances.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/performance_schema/table_lock_waits_summary_by_table.frm to ./performance_schema/table_lock_waits_summary_by_table.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/performance_schema/variables_by_thread.frm to ./performance_schema/variables_by_thread.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_waits_summary_by_host_by_event_name.frm to ./performance_schema/events_waits_summary_by_host_by_event_name.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_statements_history_long.frm to ./performance_schema/events_statements_history_long.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_waits_summary_by_account_by_event_name.frm to ./performance_schema/events_waits_summary_by_account_by_event_name.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/performance_schema/session_variables.frm to ./performance_schema/session_variables.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_stages_summary_by_thread_by_event_name.frm to ./performance_schema/events_stages_summary_by_thread_by_event_name.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/performance_schema/file_summary_by_instance.frm to ./performance_schema/file_summary_by_instance.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/performance_schema/events_statements_history.frm to ./performance_schema/events_statements_history.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/performance_schema/table_io_waits_summary_by_index_usage.frm to ./performance_schema/table_io_waits_summary_by_index_usage.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/user.frm to ./mysql/user.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/func.MYI to ./mysql/func.MYI
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/help_topic.frm to ./mysql/help_topic.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/proxies_priv.MYI to ./mysql/proxies_priv.MYI
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/help_keyword.frm to ./mysql/help_keyword.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/func.frm to ./mysql/func.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/ndb_binlog_index.MYD to ./mysql/ndb_binlog_index.MYD
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/tables_priv.MYI to ./mysql/tables_priv.MYI
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/db.opt to ./mysql/db.opt
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/db.MYD to ./mysql/db.MYD
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/engine_cost.frm to ./mysql/engine_cost.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/ndb_binlog_index.MYI to ./mysql/ndb_binlog_index.MYI
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/procs_priv.MYI to ./mysql/procs_priv.MYI
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/help_category.frm to ./mysql/help_category.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/ndb_binlog_index.frm to ./mysql/ndb_binlog_index.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/user.MYI to ./mysql/user.MYI
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/event.MYI to ./mysql/event.MYI
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/tables_priv.frm to ./mysql/tables_priv.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/slave_master_info.frm to ./mysql/slave_master_info.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/time_zone_transition_type.frm to ./mysql/time_zone_transition_type.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/columns_priv.MYD to ./mysql/columns_priv.MYD
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/tables_priv.MYD to ./mysql/tables_priv.MYD
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/proxies_priv.frm to ./mysql/proxies_priv.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/event.frm to ./mysql/event.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/slow_log.CSV to ./mysql/slow_log.CSV
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/proc.MYI to ./mysql/proc.MYI
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/func.MYD to ./mysql/func.MYD
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/proc.frm to ./mysql/proc.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/slave_relay_log_info.frm to ./mysql/slave_relay_log_info.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/time_zone.frm to ./mysql/time_zone.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/db.MYI to ./mysql/db.MYI
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/gtid_executed.frm to ./mysql/gtid_executed.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/general_log.CSM to ./mysql/general_log.CSM
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/db.frm to ./mysql/db.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/plugin.frm to ./mysql/plugin.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/time_zone_leap_second.frm to ./mysql/time_zone_leap_second.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/general_log.frm to ./mysql/general_log.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/server_cost.frm to ./mysql/server_cost.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/servers.frm to ./mysql/servers.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/innodb_table_stats.frm to ./mysql/innodb_table_stats.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/columns_priv.frm to ./mysql/columns_priv.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/user.MYD to ./mysql/user.MYD
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/slave_worker_info.frm to ./mysql/slave_worker_info.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/time_zone_transition.frm to ./mysql/time_zone_transition.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/time_zone_name.frm to ./mysql/time_zone_name.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/slow_log.frm to ./mysql/slow_log.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/help_relation.frm to ./mysql/help_relation.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/proxies_priv.MYD to ./mysql/proxies_priv.MYD
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/event.MYD to ./mysql/event.MYD
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/slow_log.CSM to ./mysql/slow_log.CSM
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/innodb_index_stats.frm to ./mysql/innodb_index_stats.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/general_log.CSV to ./mysql/general_log.CSV
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/proc.MYD to ./mysql/proc.MYD
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/procs_priv.frm to ./mysql/procs_priv.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/procs_priv.MYD to ./mysql/procs_priv.MYD
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/mysql/columns_priv.MYI to ./mysql/columns_priv.MYI
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/dao/sal_grades.frm to ./dao/sal_grades.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/dao/db.opt to ./dao/db.opt
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/dao/penalties.frm to ./dao/penalties.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/dao/dept.frm to ./dao/dept.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/dao/matches.frm to ./dao/matches.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/dao/test_import.frm to ./dao/test_import.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/dao/test_tx_iso.frm to ./dao/test_tx_iso.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/dao/teams.frm to ./dao/teams.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/dao/committee_members.frm to ./dao/committee_members.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/dao/t1.frm to ./dao/t1.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/dao/test_binlog.frm to ./dao/test_binlog.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/dao/players.frm to ./dao/players.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/dao/emp.frm to ./dao/emp.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024innodb_buffer_stats_by_schema.frm to ./sys/x@0024innodb_buffer_stats_by_schema.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024statement_analysis.frm to ./sys/x@0024statement_analysis.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024user_summary.frm to ./sys/x@0024user_summary.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024schema_index_statistics.frm to ./sys/x@0024schema_index_statistics.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024statements_with_runtimes_in_95th_percentile.frm to ./sys/x@0024statements_with_runtimes_in_95th_percentile.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024innodb_buffer_stats_by_table.frm to ./sys/x@0024innodb_buffer_stats_by_table.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/sys/db.opt to ./sys/db.opt
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/sys/host_summary_by_statement_type.frm to ./sys/host_summary_by_statement_type.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/sys/sys_config_insert_set_user.TRN to ./sys/sys_config_insert_set_user.TRN
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/sys/statement_analysis.frm to ./sys/statement_analysis.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/sys/version.frm to ./sys/version.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/sys/host_summary.frm to ./sys/host_summary.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024io_global_by_wait_by_latency.frm to ./sys/x@0024io_global_by_wait_by_latency.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/sys/statements_with_sorting.frm to ./sys/statements_with_sorting.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/sys/sys_config.frm to ./sys/sys_config.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024io_by_thread_by_latency.frm to ./sys/x@0024io_by_thread_by_latency.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/sys/memory_by_host_by_current_bytes.frm to ./sys/memory_by_host_by_current_bytes.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024memory_by_thread_by_current_bytes.frm to ./sys/x@0024memory_by_thread_by_current_bytes.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024schema_flattened_keys.frm to ./sys/x@0024schema_flattened_keys.frm
180729 15:51:06 [01]        ...done
180729 15:51:06 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024memory_by_host_by_current_bytes.frm to ./sys/x@0024memory_by_host_by_current_bytes.frm
180729 15:51:06 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/sys_config.TRG to ./sys/sys_config.TRG
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/user_summary_by_file_io.frm to ./sys/user_summary_by_file_io.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024latest_file_io.frm to ./sys/x@0024latest_file_io.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/schema_object_overview.frm to ./sys/schema_object_overview.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/schema_unused_indexes.frm to ./sys/schema_unused_indexes.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024processlist.frm to ./sys/x@0024processlist.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/statements_with_full_table_scans.frm to ./sys/statements_with_full_table_scans.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024host_summary_by_file_io.frm to ./sys/x@0024host_summary_by_file_io.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024host_summary.frm to ./sys/x@0024host_summary.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/memory_by_user_by_current_bytes.frm to ./sys/memory_by_user_by_current_bytes.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/wait_classes_global_by_latency.frm to ./sys/wait_classes_global_by_latency.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024io_global_by_wait_by_bytes.frm to ./sys/x@0024io_global_by_wait_by_bytes.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024ps_schema_table_statistics_io.frm to ./sys/x@0024ps_schema_table_statistics_io.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024schema_table_statistics_with_buffer.frm to ./sys/x@0024schema_table_statistics_with_buffer.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/session_ssl_status.frm to ./sys/session_ssl_status.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/innodb_buffer_stats_by_schema.frm to ./sys/innodb_buffer_stats_by_schema.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024waits_by_user_by_latency.frm to ./sys/x@0024waits_by_user_by_latency.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024host_summary_by_stages.frm to ./sys/x@0024host_summary_by_stages.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024waits_global_by_latency.frm to ./sys/x@0024waits_global_by_latency.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/schema_tables_with_full_table_scans.frm to ./sys/schema_tables_with_full_table_scans.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024schema_table_lock_waits.frm to ./sys/x@0024schema_table_lock_waits.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024memory_by_user_by_current_bytes.frm to ./sys/x@0024memory_by_user_by_current_bytes.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/innodb_lock_waits.frm to ./sys/innodb_lock_waits.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024ps_digest_avg_latency_distribution.frm to ./sys/x@0024ps_digest_avg_latency_distribution.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024schema_table_statistics.frm to ./sys/x@0024schema_table_statistics.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024host_summary_by_file_io_type.frm to ./sys/x@0024host_summary_by_file_io_type.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/latest_file_io.frm to ./sys/latest_file_io.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/schema_redundant_indexes.frm to ./sys/schema_redundant_indexes.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/memory_global_by_current_bytes.frm to ./sys/memory_global_by_current_bytes.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024memory_global_by_current_bytes.frm to ./sys/x@0024memory_global_by_current_bytes.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/statements_with_errors_or_warnings.frm to ./sys/statements_with_errors_or_warnings.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024wait_classes_global_by_avg_latency.frm to ./sys/x@0024wait_classes_global_by_avg_latency.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024wait_classes_global_by_latency.frm to ./sys/x@0024wait_classes_global_by_latency.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024session.frm to ./sys/x@0024session.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024schema_tables_with_full_table_scans.frm to ./sys/x@0024schema_tables_with_full_table_scans.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/host_summary_by_file_io_type.frm to ./sys/host_summary_by_file_io_type.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/user_summary.frm to ./sys/user_summary.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024user_summary_by_stages.frm to ./sys/x@0024user_summary_by_stages.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/user_summary_by_file_io_type.frm to ./sys/user_summary_by_file_io_type.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/schema_table_statistics.frm to ./sys/schema_table_statistics.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/waits_by_user_by_latency.frm to ./sys/waits_by_user_by_latency.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/io_global_by_wait_by_bytes.frm to ./sys/io_global_by_wait_by_bytes.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/user_summary_by_statement_type.frm to ./sys/user_summary_by_statement_type.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024statements_with_sorting.frm to ./sys/x@0024statements_with_sorting.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/user_summary_by_statement_latency.frm to ./sys/user_summary_by_statement_latency.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024user_summary_by_statement_latency.frm to ./sys/x@0024user_summary_by_statement_latency.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/session.frm to ./sys/session.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/schema_table_statistics_with_buffer.frm to ./sys/schema_table_statistics_with_buffer.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/host_summary_by_file_io.frm to ./sys/host_summary_by_file_io.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/memory_by_thread_by_current_bytes.frm to ./sys/memory_by_thread_by_current_bytes.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024waits_by_host_by_latency.frm to ./sys/x@0024waits_by_host_by_latency.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/schema_auto_increment_columns.frm to ./sys/schema_auto_increment_columns.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024ps_digest_95th_percentile_by_avg_us.frm to ./sys/x@0024ps_digest_95th_percentile_by_avg_us.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/waits_by_host_by_latency.frm to ./sys/waits_by_host_by_latency.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/host_summary_by_statement_latency.frm to ./sys/host_summary_by_statement_latency.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/metrics.frm to ./sys/metrics.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024user_summary_by_file_io_type.frm to ./sys/x@0024user_summary_by_file_io_type.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/io_global_by_file_by_bytes.frm to ./sys/io_global_by_file_by_bytes.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/io_by_thread_by_latency.frm to ./sys/io_by_thread_by_latency.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/processlist.frm to ./sys/processlist.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/statements_with_runtimes_in_95th_percentile.frm to ./sys/statements_with_runtimes_in_95th_percentile.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/schema_table_lock_waits.frm to ./sys/schema_table_lock_waits.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/wait_classes_global_by_avg_latency.frm to ./sys/wait_classes_global_by_avg_latency.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/user_summary_by_stages.frm to ./sys/user_summary_by_stages.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/io_global_by_file_by_latency.frm to ./sys/io_global_by_file_by_latency.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024host_summary_by_statement_type.frm to ./sys/x@0024host_summary_by_statement_type.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024innodb_lock_waits.frm to ./sys/x@0024innodb_lock_waits.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/ps_check_lost_instrumentation.frm to ./sys/ps_check_lost_instrumentation.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/waits_global_by_latency.frm to ./sys/waits_global_by_latency.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/sys_config_update_set_user.TRN to ./sys/sys_config_update_set_user.TRN
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/innodb_buffer_stats_by_table.frm to ./sys/innodb_buffer_stats_by_table.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024statements_with_temp_tables.frm to ./sys/x@0024statements_with_temp_tables.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/io_global_by_wait_by_latency.frm to ./sys/io_global_by_wait_by_latency.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/host_summary_by_stages.frm to ./sys/host_summary_by_stages.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/statements_with_temp_tables.frm to ./sys/statements_with_temp_tables.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024user_summary_by_file_io.frm to ./sys/x@0024user_summary_by_file_io.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/schema_index_statistics.frm to ./sys/schema_index_statistics.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024host_summary_by_statement_latency.frm to ./sys/x@0024host_summary_by_statement_latency.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024user_summary_by_statement_type.frm to ./sys/x@0024user_summary_by_statement_type.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024statements_with_errors_or_warnings.frm to ./sys/x@0024statements_with_errors_or_warnings.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024io_global_by_file_by_latency.frm to ./sys/x@0024io_global_by_file_by_latency.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024io_global_by_file_by_bytes.frm to ./sys/x@0024io_global_by_file_by_bytes.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024memory_global_total.frm to ./sys/x@0024memory_global_total.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/memory_global_total.frm to ./sys/memory_global_total.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [01] Copying /home/mysql/db_incr_backup1/sys/x@0024statements_with_full_table_scans.frm to ./sys/x@0024statements_with_full_table_scans.frm
180729 15:51:07 [01]        ...done
180729 15:51:07 [00] Copying /home/mysql/db_incr_backup1//xtrabackup_binlog_info to ./xtrabackup_binlog_info
180729 15:51:07 [00]        ...done
180729 15:51:07 [00] Copying /home/mysql/db_incr_backup1//xtrabackup_info to ./xtrabackup_info
180729 15:51:07 [00]        ...done
180729 15:51:07 completed OK!


```


#### 应用增量备份2

```sql

[mysql@bosenrui ~]$ xtrabackup --prepare  --target-dir=/home/mysql/db_full_backup --incremental-dir=/home/mysql/db_incr_backup2          xtrabackup version 2.4.8 based on MySQL server 5.7.13 Linux (x86_64) (revision id: 97330f7)
incremental backup from 2966675 is enabled.
xtrabackup: cd to /home/mysql/db_full_backup/
xtrabackup: This target seems to be already prepared with --apply-log-only.
InnoDB: Number of pools: 1
xtrabackup: xtrabackup_logfile detected: size=8388608, start_lsn=(2967003)
xtrabackup: using the following InnoDB configuration for recovery:
xtrabackup:   innodb_data_home_dir = .
xtrabackup:   innodb_data_file_path = ibdata1:12M:autoextend
xtrabackup:   innodb_log_group_home_dir = /home/mysql/db_incr_backup2/
xtrabackup:   innodb_log_files_in_group = 1
xtrabackup:   innodb_log_file_size = 8388608
xtrabackup: Generating a list of tablespaces
InnoDB: Allocated tablespace ID 8 for mysql/time_zone_name, old maximum was 0
xtrabackup: page size for /home/mysql/db_incr_backup2//ibdata1.delta is 16384 bytes
Applying /home/mysql/db_incr_backup2//ibdata1.delta to ./ibdata1...
xtrabackup: page size for /home/mysql/db_incr_backup2//mysql/time_zone_transition_type.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup2//mysql/time_zone_transition_type.ibd.delta to ./mysql/time_zone_transition_type.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup2//mysql/slave_worker_info.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup2//mysql/slave_worker_info.ibd.delta to ./mysql/slave_worker_info.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup2//mysql/engine_cost.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup2//mysql/engine_cost.ibd.delta to ./mysql/engine_cost.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup2//mysql/help_relation.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup2//mysql/help_relation.ibd.delta to ./mysql/help_relation.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup2//mysql/innodb_index_stats.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup2//mysql/innodb_index_stats.ibd.delta to ./mysql/innodb_index_stats.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup2//mysql/time_zone_transition.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup2//mysql/time_zone_transition.ibd.delta to ./mysql/time_zone_transition.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup2//mysql/help_keyword.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup2//mysql/help_keyword.ibd.delta to ./mysql/help_keyword.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup2//mysql/innodb_table_stats.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup2//mysql/innodb_table_stats.ibd.delta to ./mysql/innodb_table_stats.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup2//mysql/slave_relay_log_info.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup2//mysql/slave_relay_log_info.ibd.delta to ./mysql/slave_relay_log_info.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup2//mysql/help_topic.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup2//mysql/help_topic.ibd.delta to ./mysql/help_topic.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup2//mysql/time_zone_name.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup2//mysql/time_zone_name.ibd.delta to ./mysql/time_zone_name.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup2//mysql/slave_master_info.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup2//mysql/slave_master_info.ibd.delta to ./mysql/slave_master_info.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup2//mysql/plugin.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup2//mysql/plugin.ibd.delta to ./mysql/plugin.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup2//mysql/time_zone_leap_second.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup2//mysql/time_zone_leap_second.ibd.delta to ./mysql/time_zone_leap_second.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup2//mysql/gtid_executed.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup2//mysql/gtid_executed.ibd.delta to ./mysql/gtid_executed.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup2//mysql/help_category.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup2//mysql/help_category.ibd.delta to ./mysql/help_category.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup2//mysql/servers.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup2//mysql/servers.ibd.delta to ./mysql/servers.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup2//mysql/time_zone.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup2//mysql/time_zone.ibd.delta to ./mysql/time_zone.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup2//mysql/server_cost.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup2//mysql/server_cost.ibd.delta to ./mysql/server_cost.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup2//dao/test_import.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup2//dao/test_import.ibd.delta to ./dao/test_import.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup2//dao/test_binlog.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup2//dao/test_binlog.ibd.delta to ./dao/test_binlog.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup2//dao/matches.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup2//dao/matches.ibd.delta to ./dao/matches.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup2//dao/sal_grades.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup2//dao/sal_grades.ibd.delta to ./dao/sal_grades.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup2//dao/players.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup2//dao/players.ibd.delta to ./dao/players.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup2//dao/teams.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup2//dao/teams.ibd.delta to ./dao/teams.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup2//dao/penalties.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup2//dao/penalties.ibd.delta to ./dao/penalties.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup2//dao/committee_members.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup2//dao/committee_members.ibd.delta to ./dao/committee_members.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup2//dao/t1.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup2//dao/t1.ibd.delta to ./dao/t1.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup2//dao/test_tx_iso.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup2//dao/test_tx_iso.ibd.delta to ./dao/test_tx_iso.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup2//dao/dept.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup2//dao/dept.ibd.delta to ./dao/dept.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup2//dao/emp.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup2//dao/emp.ibd.delta to ./dao/emp.ibd...
xtrabackup: page size for /home/mysql/db_incr_backup2//sys/sys_config.ibd.delta is 16384 bytes
Applying /home/mysql/db_incr_backup2//sys/sys_config.ibd.delta to ./sys/sys_config.ibd...
xtrabackup: using the following InnoDB configuration for recovery:
xtrabackup:   innodb_data_home_dir = .
xtrabackup:   innodb_data_file_path = ibdata1:12M:autoextend
xtrabackup:   innodb_log_group_home_dir = /home/mysql/db_incr_backup2/
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
InnoDB: Log scan progressed past the checkpoint lsn 2967003
InnoDB: Doing recovery: scanned up to log sequence number 2967012 (0%)
InnoDB: Doing recovery: scanned up to log sequence number 2967012 (0%)
InnoDB: Database was not shutdown normally!
InnoDB: Starting crash recovery.
InnoDB: xtrabackup: Last MySQL binlog file position 3025, file name mysql-bin.000005
InnoDB: Creating shared tablespace for temporary tables
InnoDB: Setting file './ibtmp1' size to 12 MB. Physically writing the file full; Please wait ...
InnoDB: File './ibtmp1' size is now 12 MB.
InnoDB: 96 redo rollback segment(s) found. 1 redo rollback segment(s) are active.
InnoDB: 32 non-redo rollback segment(s) are active.
InnoDB: Waiting for purge to start
InnoDB: 5.7.13 started; log sequence number 2967012
InnoDB: xtrabackup: Last MySQL binlog file position 3025, file name mysql-bin.000005

xtrabackup: starting shutdown with innodb_fast_shutdown = 1
InnoDB: FTS optimize thread exiting.
InnoDB: Starting shutdown...
InnoDB: Shutdown completed; log sequence number 2967031
InnoDB: Number of pools: 1
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/replication_applier_status.frm to ./performance_schema/replication_applier_status.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/file_instances.frm to ./performance_schema/file_instances.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/metadata_locks.frm to ./performance_schema/metadata_locks.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/memory_summary_by_account_by_event_name.frm to ./performance_schema/memory_summary_by_account_by_event_name.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/threads.frm to ./performance_schema/threads.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/events_waits_summary_by_user_by_event_name.frm to ./performance_schema/events_waits_summary_by_user_by_event_name.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/events_stages_summary_by_account_by_event_name.frm to ./performance_schema/events_stages_summary_by_account_by_event_name.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/db.opt to ./performance_schema/db.opt
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/table_io_waits_summary_by_table.frm to ./performance_schema/table_io_waits_summary_by_table.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/memory_summary_global_by_event_name.frm to ./performance_schema/memory_summary_global_by_event_name.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/events_stages_history_long.frm to ./performance_schema/events_stages_history_long.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/events_statements_summary_by_program.frm to ./performance_schema/events_statements_summary_by_program.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/events_statements_summary_by_user_by_event_name.frm to ./performance_schema/events_statements_summary_by_user_by_event_name.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/session_account_connect_attrs.frm to ./performance_schema/session_account_connect_attrs.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/session_connect_attrs.frm to ./performance_schema/session_connect_attrs.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/events_statements_summary_by_account_by_event_name.frm to ./performance_schema/events_statements_summary_by_account_by_event_name.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/events_statements_summary_by_thread_by_event_name.frm to ./performance_schema/events_statements_summary_by_thread_by_event_name.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/events_transactions_history.frm to ./performance_schema/events_transactions_history.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/memory_summary_by_user_by_event_name.frm to ./performance_schema/memory_summary_by_user_by_event_name.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/events_statements_summary_global_by_event_name.frm to ./performance_schema/events_statements_summary_global_by_event_name.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/performance_timers.frm to ./performance_schema/performance_timers.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/setup_consumers.frm to ./performance_schema/setup_consumers.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/events_stages_summary_by_host_by_event_name.frm to ./performance_schema/events_stages_summary_by_host_by_event_name.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/events_transactions_history_long.frm to ./performance_schema/events_transactions_history_long.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/replication_connection_status.frm to ./performance_schema/replication_connection_status.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/replication_group_members.frm to ./performance_schema/replication_group_members.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/objects_summary_global_by_type.frm to ./performance_schema/objects_summary_global_by_type.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/rwlock_instances.frm to ./performance_schema/rwlock_instances.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/user_variables_by_thread.frm to ./performance_schema/user_variables_by_thread.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/events_waits_current.frm to ./performance_schema/events_waits_current.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/file_summary_by_event_name.frm to ./performance_schema/file_summary_by_event_name.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/users.frm to ./performance_schema/users.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/replication_applier_status_by_worker.frm to ./performance_schema/replication_applier_status_by_worker.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/socket_instances.frm to ./performance_schema/socket_instances.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/events_stages_current.frm to ./performance_schema/events_stages_current.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/global_status.frm to ./performance_schema/global_status.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/setup_timers.frm to ./performance_schema/setup_timers.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/events_waits_history.frm to ./performance_schema/events_waits_history.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/table_handles.frm to ./performance_schema/table_handles.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/events_statements_current.frm to ./performance_schema/events_statements_current.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/events_transactions_current.frm to ./performance_schema/events_transactions_current.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/events_waits_summary_global_by_event_name.frm to ./performance_schema/events_waits_summary_global_by_event_name.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/replication_applier_status_by_coordinator.frm to ./performance_schema/replication_applier_status_by_coordinator.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/events_statements_summary_by_host_by_event_name.frm to ./performance_schema/events_statements_summary_by_host_by_event_name.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/events_stages_history.frm to ./performance_schema/events_stages_history.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/events_statements_summary_by_digest.frm to ./performance_schema/events_statements_summary_by_digest.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/accounts.frm to ./performance_schema/accounts.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/replication_applier_configuration.frm to ./performance_schema/replication_applier_configuration.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/replication_connection_configuration.frm to ./performance_schema/replication_connection_configuration.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/status_by_user.frm to ./performance_schema/status_by_user.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/events_transactions_summary_global_by_event_name.frm to ./performance_schema/events_transactions_summary_global_by_event_name.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/events_transactions_summary_by_account_by_event_name.frm to ./performance_schema/events_transactions_summary_by_account_by_event_name.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/events_stages_summary_global_by_event_name.frm to ./performance_schema/events_stages_summary_global_by_event_name.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/socket_summary_by_event_name.frm to ./performance_schema/socket_summary_by_event_name.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/memory_summary_by_thread_by_event_name.frm to ./performance_schema/memory_summary_by_thread_by_event_name.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/status_by_thread.frm to ./performance_schema/status_by_thread.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/events_waits_summary_by_instance.frm to ./performance_schema/events_waits_summary_by_instance.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/events_transactions_summary_by_thread_by_event_name.frm to ./performance_schema/events_transactions_summary_by_thread_by_event_name.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/hosts.frm to ./performance_schema/hosts.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/cond_instances.frm to ./performance_schema/cond_instances.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/memory_summary_by_host_by_event_name.frm to ./performance_schema/memory_summary_by_host_by_event_name.frm
180729 15:52:16 [01]        ...done
180729 15:52:16 [01] Copying /home/mysql/db_incr_backup2/performance_schema/status_by_account.frm to ./performance_schema/status_by_account.frm
180729 15:52:16 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/performance_schema/socket_summary_by_instance.frm to ./performance_schema/socket_summary_by_instance.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/performance_schema/events_transactions_summary_by_user_by_event_name.frm to ./performance_schema/events_transactions_summary_by_user_by_event_name.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/performance_schema/status_by_host.frm to ./performance_schema/status_by_host.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/performance_schema/events_stages_summary_by_user_by_event_name.frm to ./performance_schema/events_stages_summary_by_user_by_event_name.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/performance_schema/global_variables.frm to ./performance_schema/global_variables.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/performance_schema/setup_instruments.frm to ./performance_schema/setup_instruments.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/performance_schema/events_waits_summary_by_thread_by_event_name.frm to ./performance_schema/events_waits_summary_by_thread_by_event_name.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/performance_schema/replication_group_member_stats.frm to ./performance_schema/replication_group_member_stats.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/performance_schema/host_cache.frm to ./performance_schema/host_cache.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/performance_schema/setup_objects.frm to ./performance_schema/setup_objects.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/performance_schema/events_waits_history_long.frm to ./performance_schema/events_waits_history_long.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/performance_schema/session_status.frm to ./performance_schema/session_status.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/performance_schema/events_transactions_summary_by_host_by_event_name.frm to ./performance_schema/events_transactions_summary_by_host_by_event_name.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/performance_schema/prepared_statements_instances.frm to ./performance_schema/prepared_statements_instances.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/performance_schema/setup_actors.frm to ./performance_schema/setup_actors.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/performance_schema/mutex_instances.frm to ./performance_schema/mutex_instances.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/performance_schema/table_lock_waits_summary_by_table.frm to ./performance_schema/table_lock_waits_summary_by_table.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/performance_schema/variables_by_thread.frm to ./performance_schema/variables_by_thread.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/performance_schema/events_waits_summary_by_host_by_event_name.frm to ./performance_schema/events_waits_summary_by_host_by_event_name.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/performance_schema/events_statements_history_long.frm to ./performance_schema/events_statements_history_long.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/performance_schema/events_waits_summary_by_account_by_event_name.frm to ./performance_schema/events_waits_summary_by_account_by_event_name.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/performance_schema/session_variables.frm to ./performance_schema/session_variables.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/performance_schema/events_stages_summary_by_thread_by_event_name.frm to ./performance_schema/events_stages_summary_by_thread_by_event_name.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/performance_schema/file_summary_by_instance.frm to ./performance_schema/file_summary_by_instance.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/performance_schema/events_statements_history.frm to ./performance_schema/events_statements_history.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/performance_schema/table_io_waits_summary_by_index_usage.frm to ./performance_schema/table_io_waits_summary_by_index_usage.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/user.frm to ./mysql/user.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/func.MYI to ./mysql/func.MYI
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/help_topic.frm to ./mysql/help_topic.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/proxies_priv.MYI to ./mysql/proxies_priv.MYI
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/help_keyword.frm to ./mysql/help_keyword.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/func.frm to ./mysql/func.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/ndb_binlog_index.MYD to ./mysql/ndb_binlog_index.MYD
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/tables_priv.MYI to ./mysql/tables_priv.MYI
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/db.opt to ./mysql/db.opt
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/db.MYD to ./mysql/db.MYD
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/engine_cost.frm to ./mysql/engine_cost.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/ndb_binlog_index.MYI to ./mysql/ndb_binlog_index.MYI
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/procs_priv.MYI to ./mysql/procs_priv.MYI
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/help_category.frm to ./mysql/help_category.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/ndb_binlog_index.frm to ./mysql/ndb_binlog_index.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/user.MYI to ./mysql/user.MYI
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/event.MYI to ./mysql/event.MYI
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/tables_priv.frm to ./mysql/tables_priv.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/slave_master_info.frm to ./mysql/slave_master_info.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/time_zone_transition_type.frm to ./mysql/time_zone_transition_type.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/columns_priv.MYD to ./mysql/columns_priv.MYD
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/tables_priv.MYD to ./mysql/tables_priv.MYD
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/proxies_priv.frm to ./mysql/proxies_priv.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/event.frm to ./mysql/event.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/slow_log.CSV to ./mysql/slow_log.CSV
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/proc.MYI to ./mysql/proc.MYI
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/func.MYD to ./mysql/func.MYD
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/proc.frm to ./mysql/proc.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/slave_relay_log_info.frm to ./mysql/slave_relay_log_info.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/time_zone.frm to ./mysql/time_zone.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/db.MYI to ./mysql/db.MYI
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/gtid_executed.frm to ./mysql/gtid_executed.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/general_log.CSM to ./mysql/general_log.CSM
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/db.frm to ./mysql/db.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/plugin.frm to ./mysql/plugin.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/time_zone_leap_second.frm to ./mysql/time_zone_leap_second.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/general_log.frm to ./mysql/general_log.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/server_cost.frm to ./mysql/server_cost.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/servers.frm to ./mysql/servers.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/innodb_table_stats.frm to ./mysql/innodb_table_stats.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/columns_priv.frm to ./mysql/columns_priv.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/user.MYD to ./mysql/user.MYD
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/slave_worker_info.frm to ./mysql/slave_worker_info.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/time_zone_transition.frm to ./mysql/time_zone_transition.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/time_zone_name.frm to ./mysql/time_zone_name.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/slow_log.frm to ./mysql/slow_log.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/help_relation.frm to ./mysql/help_relation.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/proxies_priv.MYD to ./mysql/proxies_priv.MYD
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/event.MYD to ./mysql/event.MYD
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/slow_log.CSM to ./mysql/slow_log.CSM
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/innodb_index_stats.frm to ./mysql/innodb_index_stats.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/general_log.CSV to ./mysql/general_log.CSV
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/proc.MYD to ./mysql/proc.MYD
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/procs_priv.frm to ./mysql/procs_priv.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/procs_priv.MYD to ./mysql/procs_priv.MYD
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/mysql/columns_priv.MYI to ./mysql/columns_priv.MYI
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/dao/sal_grades.frm to ./dao/sal_grades.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/dao/db.opt to ./dao/db.opt
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/dao/penalties.frm to ./dao/penalties.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/dao/dept.frm to ./dao/dept.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/dao/matches.frm to ./dao/matches.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/dao/test_import.frm to ./dao/test_import.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/dao/test_tx_iso.frm to ./dao/test_tx_iso.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/dao/teams.frm to ./dao/teams.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/dao/committee_members.frm to ./dao/committee_members.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/dao/t1.frm to ./dao/t1.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/dao/test_binlog.frm to ./dao/test_binlog.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/dao/players.frm to ./dao/players.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/dao/emp.frm to ./dao/emp.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024innodb_buffer_stats_by_schema.frm to ./sys/x@0024innodb_buffer_stats_by_schema.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024statement_analysis.frm to ./sys/x@0024statement_analysis.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024user_summary.frm to ./sys/x@0024user_summary.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024schema_index_statistics.frm to ./sys/x@0024schema_index_statistics.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024statements_with_runtimes_in_95th_percentile.frm to ./sys/x@0024statements_with_runtimes_in_95th_percentile.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024innodb_buffer_stats_by_table.frm to ./sys/x@0024innodb_buffer_stats_by_table.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/sys/db.opt to ./sys/db.opt
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/sys/host_summary_by_statement_type.frm to ./sys/host_summary_by_statement_type.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/sys/sys_config_insert_set_user.TRN to ./sys/sys_config_insert_set_user.TRN
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/sys/statement_analysis.frm to ./sys/statement_analysis.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/sys/version.frm to ./sys/version.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/sys/host_summary.frm to ./sys/host_summary.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024io_global_by_wait_by_latency.frm to ./sys/x@0024io_global_by_wait_by_latency.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/sys/statements_with_sorting.frm to ./sys/statements_with_sorting.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/sys/sys_config.frm to ./sys/sys_config.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024io_by_thread_by_latency.frm to ./sys/x@0024io_by_thread_by_latency.frm
180729 15:52:17 [01]        ...done
180729 15:52:17 [01] Copying /home/mysql/db_incr_backup2/sys/memory_by_host_by_current_bytes.frm to ./sys/memory_by_host_by_current_bytes.frm
180729 15:52:17 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024memory_by_thread_by_current_bytes.frm to ./sys/x@0024memory_by_thread_by_current_bytes.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024schema_flattened_keys.frm to ./sys/x@0024schema_flattened_keys.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024memory_by_host_by_current_bytes.frm to ./sys/x@0024memory_by_host_by_current_bytes.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/sys_config.TRG to ./sys/sys_config.TRG
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/user_summary_by_file_io.frm to ./sys/user_summary_by_file_io.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024latest_file_io.frm to ./sys/x@0024latest_file_io.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/schema_object_overview.frm to ./sys/schema_object_overview.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/schema_unused_indexes.frm to ./sys/schema_unused_indexes.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024processlist.frm to ./sys/x@0024processlist.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/statements_with_full_table_scans.frm to ./sys/statements_with_full_table_scans.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024host_summary_by_file_io.frm to ./sys/x@0024host_summary_by_file_io.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024host_summary.frm to ./sys/x@0024host_summary.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/memory_by_user_by_current_bytes.frm to ./sys/memory_by_user_by_current_bytes.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/wait_classes_global_by_latency.frm to ./sys/wait_classes_global_by_latency.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024io_global_by_wait_by_bytes.frm to ./sys/x@0024io_global_by_wait_by_bytes.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024ps_schema_table_statistics_io.frm to ./sys/x@0024ps_schema_table_statistics_io.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024schema_table_statistics_with_buffer.frm to ./sys/x@0024schema_table_statistics_with_buffer.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/session_ssl_status.frm to ./sys/session_ssl_status.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/innodb_buffer_stats_by_schema.frm to ./sys/innodb_buffer_stats_by_schema.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024waits_by_user_by_latency.frm to ./sys/x@0024waits_by_user_by_latency.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024host_summary_by_stages.frm to ./sys/x@0024host_summary_by_stages.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024waits_global_by_latency.frm to ./sys/x@0024waits_global_by_latency.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/schema_tables_with_full_table_scans.frm to ./sys/schema_tables_with_full_table_scans.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024schema_table_lock_waits.frm to ./sys/x@0024schema_table_lock_waits.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024memory_by_user_by_current_bytes.frm to ./sys/x@0024memory_by_user_by_current_bytes.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/innodb_lock_waits.frm to ./sys/innodb_lock_waits.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024ps_digest_avg_latency_distribution.frm to ./sys/x@0024ps_digest_avg_latency_distribution.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024schema_table_statistics.frm to ./sys/x@0024schema_table_statistics.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024host_summary_by_file_io_type.frm to ./sys/x@0024host_summary_by_file_io_type.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/latest_file_io.frm to ./sys/latest_file_io.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/schema_redundant_indexes.frm to ./sys/schema_redundant_indexes.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/memory_global_by_current_bytes.frm to ./sys/memory_global_by_current_bytes.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024memory_global_by_current_bytes.frm to ./sys/x@0024memory_global_by_current_bytes.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/statements_with_errors_or_warnings.frm to ./sys/statements_with_errors_or_warnings.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024wait_classes_global_by_avg_latency.frm to ./sys/x@0024wait_classes_global_by_avg_latency.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024wait_classes_global_by_latency.frm to ./sys/x@0024wait_classes_global_by_latency.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024session.frm to ./sys/x@0024session.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024schema_tables_with_full_table_scans.frm to ./sys/x@0024schema_tables_with_full_table_scans.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/host_summary_by_file_io_type.frm to ./sys/host_summary_by_file_io_type.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/user_summary.frm to ./sys/user_summary.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024user_summary_by_stages.frm to ./sys/x@0024user_summary_by_stages.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/user_summary_by_file_io_type.frm to ./sys/user_summary_by_file_io_type.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/schema_table_statistics.frm to ./sys/schema_table_statistics.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/waits_by_user_by_latency.frm to ./sys/waits_by_user_by_latency.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/io_global_by_wait_by_bytes.frm to ./sys/io_global_by_wait_by_bytes.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/user_summary_by_statement_type.frm to ./sys/user_summary_by_statement_type.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024statements_with_sorting.frm to ./sys/x@0024statements_with_sorting.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/user_summary_by_statement_latency.frm to ./sys/user_summary_by_statement_latency.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024user_summary_by_statement_latency.frm to ./sys/x@0024user_summary_by_statement_latency.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/session.frm to ./sys/session.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/schema_table_statistics_with_buffer.frm to ./sys/schema_table_statistics_with_buffer.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/host_summary_by_file_io.frm to ./sys/host_summary_by_file_io.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/memory_by_thread_by_current_bytes.frm to ./sys/memory_by_thread_by_current_bytes.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024waits_by_host_by_latency.frm to ./sys/x@0024waits_by_host_by_latency.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/schema_auto_increment_columns.frm to ./sys/schema_auto_increment_columns.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024ps_digest_95th_percentile_by_avg_us.frm to ./sys/x@0024ps_digest_95th_percentile_by_avg_us.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/waits_by_host_by_latency.frm to ./sys/waits_by_host_by_latency.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/host_summary_by_statement_latency.frm to ./sys/host_summary_by_statement_latency.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/metrics.frm to ./sys/metrics.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024user_summary_by_file_io_type.frm to ./sys/x@0024user_summary_by_file_io_type.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/io_global_by_file_by_bytes.frm to ./sys/io_global_by_file_by_bytes.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/io_by_thread_by_latency.frm to ./sys/io_by_thread_by_latency.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/processlist.frm to ./sys/processlist.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/statements_with_runtimes_in_95th_percentile.frm to ./sys/statements_with_runtimes_in_95th_percentile.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/schema_table_lock_waits.frm to ./sys/schema_table_lock_waits.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/wait_classes_global_by_avg_latency.frm to ./sys/wait_classes_global_by_avg_latency.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/user_summary_by_stages.frm to ./sys/user_summary_by_stages.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/io_global_by_file_by_latency.frm to ./sys/io_global_by_file_by_latency.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024host_summary_by_statement_type.frm to ./sys/x@0024host_summary_by_statement_type.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024innodb_lock_waits.frm to ./sys/x@0024innodb_lock_waits.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/ps_check_lost_instrumentation.frm to ./sys/ps_check_lost_instrumentation.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/waits_global_by_latency.frm to ./sys/waits_global_by_latency.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/sys_config_update_set_user.TRN to ./sys/sys_config_update_set_user.TRN
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/innodb_buffer_stats_by_table.frm to ./sys/innodb_buffer_stats_by_table.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024statements_with_temp_tables.frm to ./sys/x@0024statements_with_temp_tables.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/io_global_by_wait_by_latency.frm to ./sys/io_global_by_wait_by_latency.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/host_summary_by_stages.frm to ./sys/host_summary_by_stages.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/statements_with_temp_tables.frm to ./sys/statements_with_temp_tables.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024user_summary_by_file_io.frm to ./sys/x@0024user_summary_by_file_io.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/schema_index_statistics.frm to ./sys/schema_index_statistics.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024host_summary_by_statement_latency.frm to ./sys/x@0024host_summary_by_statement_latency.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024user_summary_by_statement_type.frm to ./sys/x@0024user_summary_by_statement_type.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024statements_with_errors_or_warnings.frm to ./sys/x@0024statements_with_errors_or_warnings.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024io_global_by_file_by_latency.frm to ./sys/x@0024io_global_by_file_by_latency.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024io_global_by_file_by_bytes.frm to ./sys/x@0024io_global_by_file_by_bytes.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024memory_global_total.frm to ./sys/x@0024memory_global_total.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/memory_global_total.frm to ./sys/memory_global_total.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [01] Copying /home/mysql/db_incr_backup2/sys/x@0024statements_with_full_table_scans.frm to ./sys/x@0024statements_with_full_table_scans.frm
180729 15:52:18 [01]        ...done
180729 15:52:18 [00] Copying /home/mysql/db_incr_backup2//xtrabackup_binlog_info to ./xtrabackup_binlog_info
180729 15:52:18 [00]        ...done
180729 15:52:18 [00] Copying /home/mysql/db_incr_backup2//xtrabackup_info to ./xtrabackup_info
180729 15:52:18 [00]        ...done
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
InnoDB: New log files created, LSN=2967031
InnoDB: Highest supported file format is Barracuda.
InnoDB: Log scan progressed past the checkpoint lsn 2967052
InnoDB: Doing recovery: scanned up to log sequence number 2967061 (0%)
InnoDB: Doing recovery: scanned up to log sequence number 2967061 (0%)
InnoDB: Database was not shutdown normally!
InnoDB: Starting crash recovery.
InnoDB: xtrabackup: Last MySQL binlog file position 3025, file name mysql-bin.000005
InnoDB: Removed temporary tablespace data file: "ibtmp1"
InnoDB: Creating shared tablespace for temporary tables
InnoDB: Setting file './ibtmp1' size to 12 MB. Physically writing the file full; Please wait ...
InnoDB: File './ibtmp1' size is now 12 MB.
InnoDB: 96 redo rollback segment(s) found. 1 redo rollback segment(s) are active.
InnoDB: 32 non-redo rollback segment(s) are active.
InnoDB: Waiting for purge to start
InnoDB: 5.7.13 started; log sequence number 2967061
xtrabackup: starting shutdown with innodb_fast_shutdown = 1
InnoDB: FTS optimize thread exiting.
InnoDB: Starting shutdown...
InnoDB: Shutdown completed; log sequence number 2967080
180729 15:52:21 completed OK!


```



#### 删除数据库

```sql

[mysql@bosenrui ~]$ mysqladmin  -S /tmp/mysql1.sock  -uroot -proot shutdown
mysqladmin: [Warning] Using a password on the command line interface can be insecure.

[mysql@bosenrui ~]$ rm -rf /mysql1/data/
[mysql@bosenrui ~]$ rm -rf /mysql1/mysql-bin.index
[mysql@bosenrui ~]$


```

#### 恢复数据库并查看表中数据

```sql
[mysql@bosenrui ~]$ cp -r /home/mysql/db_full_backup /mysql1/data
[mysql@bosenrui ~]$ mysqld --defaults-file=/mysql1/mysql1.cnf &
[1] 8881



[mysql@bosenrui ~]$ mysql -S /tmp/mysql1.sock -uroot -proot
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.21-log MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> use dao ;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> select * from t1 ;
+------+
| c1   |
+------+
|    1 |
|    2 |
|    3 |
+------+
3 rows in set (0.00 sec)


```

