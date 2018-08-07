---
layout: post
title: 使用innobackupex在线搭建主从
date: 2018-08-07
categories: blog
tags: [MySQL]
description: 使用innobackupex在线搭建主从
---


## 对主库进行备份

```sql

[mysql@node1 mysql1]$ mysqldump -uroot -proot -S /tmp/mysql1.sock  --master-data=2 --single-transaction -A >/home/mysql/slavedata.sql
mysqldump: [Warning] Using a password on the command line interface can be insecure.


```
## 查看主库配置文件

```sql

[mysql@node1 mysql1]$ cat /mysql1/mysql1.cnf

[client]
port = 3306
socket = /tmp/mysql1.sock
[mysqld]
port = 3306
socket = /tmp/mysql1.sock
#skip-grant-tables
basedir = /usr/local/mysql
datadir = /mysql1/data
pid-file = /mysql1/mysql1.pid
log-error = /mysql1/mysql1.log
log-bin=/mysql1/data/mysql-bin
server_id = 1


```

## 创建复制用账户


```sql

mysql> GRANT ALL PRIVILEGES ON *.* TO 'dao'@'192.168.2.62' IDENTIFIED BY 'dao' ;
Query OK, 0 rows affected, 1 warning (0.01 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.01 sec)


```

##  全备数据库

```sql

[mysql@node1 ~]$ innobackupex  --defaults-file=/mysql1/mysql1.cnf  --no-timestamp   -u root -p root   /home/mysql/db_full_backup/
180808 00:04:43 innobackupex: Starting the backup operation

IMPORTANT: Please check that the backup run completes successfully.
           At the end of a successful backup run innobackupex
           prints "completed OK!".

180808 00:04:43  version_check Connecting to MySQL server with DSN 'dbi:mysql:;mysql_read_default_group=xtrabackup;port=3306;mysql_socket=/tmp/mysql1.sock' as 'root'  (using password: YES).
180808 00:04:43  version_check Connected to MySQL server
180808 00:04:43  version_check Executing a version check against the server...
180808 00:04:43  version_check Done.
180808 00:04:43 Connecting to MySQL server host: localhost, user: root, password: set, port: 3306, socket: /tmp/mysql1.sock
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
180808 00:04:43 >> log scanned up to (2551222)
xtrabackup: Generating a list of tablespaces
InnoDB: Allocated tablespace ID 21 for sys/sys_config, old maximum was 0
180808 00:04:44 [01] Copying ./ibdata1 to /home/mysql/db_full_backup/ibdata1
180808 00:04:44 [01]        ...done
180808 00:04:44 [01] Copying ./sys/sys_config.ibd to /home/mysql/db_full_backup/sys/sys_config.ibd
180808 00:04:44 [01]        ...done
180808 00:04:44 [01] Copying ./mysql/time_zone_leap_second.ibd to /home/mysql/db_full_backup/mysql/time_zone_leap_second.ibd
180808 00:04:44 [01]        ...done
180808 00:04:44 [01] Copying ./mysql/innodb_table_stats.ibd to /home/mysql/db_full_backup/mysql/innodb_table_stats.ibd
180808 00:04:44 [01]        ...done
180808 00:04:44 [01] Copying ./mysql/slave_master_info.ibd to /home/mysql/db_full_backup/mysql/slave_master_info.ibd
180808 00:04:44 [01]        ...done
180808 00:04:44 [01] Copying ./mysql/plugin.ibd to /home/mysql/db_full_backup/mysql/plugin.ibd
180808 00:04:44 [01]        ...done
180808 00:04:44 [01] Copying ./mysql/innodb_index_stats.ibd to /home/mysql/db_full_backup/mysql/innodb_index_stats.ibd
180808 00:04:44 [01]        ...done
180808 00:04:44 [01] Copying ./mysql/time_zone_name.ibd to /home/mysql/db_full_backup/mysql/time_zone_name.ibd
180808 00:04:44 [01]        ...done
180808 00:04:44 [01] Copying ./mysql/servers.ibd to /home/mysql/db_full_backup/mysql/servers.ibd
180808 00:04:44 [01]        ...done
180808 00:04:44 [01] Copying ./mysql/help_keyword.ibd to /home/mysql/db_full_backup/mysql/help_keyword.ibd
180808 00:04:44 [01]        ...done
180808 00:04:44 [01] Copying ./mysql/engine_cost.ibd to /home/mysql/db_full_backup/mysql/engine_cost.ibd
180808 00:04:44 [01]        ...done
180808 00:04:44 [01] Copying ./mysql/help_topic.ibd to /home/mysql/db_full_backup/mysql/help_topic.ibd
180808 00:04:44 [01]        ...done
180808 00:04:44 [01] Copying ./mysql/time_zone.ibd to /home/mysql/db_full_backup/mysql/time_zone.ibd
180808 00:04:44 [01]        ...done
180808 00:04:44 [01] Copying ./mysql/time_zone_transition_type.ibd to /home/mysql/db_full_backup/mysql/time_zone_transition_type.ibd
180808 00:04:44 [01]        ...done
180808 00:04:44 [01] Copying ./mysql/time_zone_transition.ibd to /home/mysql/db_full_backup/mysql/time_zone_transition.ibd
180808 00:04:44 [01]        ...done
180808 00:04:44 [01] Copying ./mysql/slave_relay_log_info.ibd to /home/mysql/db_full_backup/mysql/slave_relay_log_info.ibd
180808 00:04:44 [01]        ...done
180808 00:04:44 [01] Copying ./mysql/gtid_executed.ibd to /home/mysql/db_full_backup/mysql/gtid_executed.ibd
180808 00:04:44 [01]        ...done
180808 00:04:44 [01] Copying ./mysql/help_category.ibd to /home/mysql/db_full_backup/mysql/help_category.ibd
180808 00:04:44 [01]        ...done
180808 00:04:44 [01] Copying ./mysql/server_cost.ibd to /home/mysql/db_full_backup/mysql/server_cost.ibd
180808 00:04:44 [01]        ...done
180808 00:04:44 [01] Copying ./mysql/slave_worker_info.ibd to /home/mysql/db_full_backup/mysql/slave_worker_info.ibd
180808 00:04:44 [01]        ...done
180808 00:04:44 [01] Copying ./mysql/help_relation.ibd to /home/mysql/db_full_backup/mysql/help_relation.ibd
180808 00:04:44 [01]        ...done
180808 00:04:44 >> log scanned up to (2551222)
180808 00:04:45 Executing FLUSH NO_WRITE_TO_BINLOG TABLES...
180808 00:04:45 Executing FLUSH TABLES WITH READ LOCK...
180808 00:04:45 Starting to backup non-InnoDB tables and files
180808 00:04:45 [01] Copying ./sys/x@0024schema_flattened_keys.frm to /home/mysql/db_full_backup/sys/x@0024schema_flattened_keys.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/x@0024host_summary.frm to /home/mysql/db_full_backup/sys/x@0024host_summary.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/io_global_by_file_by_latency.frm to /home/mysql/db_full_backup/sys/io_global_by_file_by_latency.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/statement_analysis.frm to /home/mysql/db_full_backup/sys/statement_analysis.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/x@0024user_summary_by_statement_latency.frm to /home/mysql/db_full_backup/sys/x@0024user_summary_by_statement_latency.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/db.opt to /home/mysql/db_full_backup/sys/db.opt
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/x@0024waits_global_by_latency.frm to /home/mysql/db_full_backup/sys/x@0024waits_global_by_latency.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/x@0024statements_with_temp_tables.frm to /home/mysql/db_full_backup/sys/x@0024statements_with_temp_tables.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/x@0024user_summary_by_file_io_type.frm to /home/mysql/db_full_backup/sys/x@0024user_summary_by_file_io_type.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/version.frm to /home/mysql/db_full_backup/sys/version.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/x@0024io_global_by_file_by_bytes.frm to /home/mysql/db_full_backup/sys/x@0024io_global_by_file_by_bytes.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/x@0024host_summary_by_stages.frm to /home/mysql/db_full_backup/sys/x@0024host_summary_by_stages.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/x@0024processlist.frm to /home/mysql/db_full_backup/sys/x@0024processlist.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/host_summary.frm to /home/mysql/db_full_backup/sys/host_summary.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/memory_global_by_current_bytes.frm to /home/mysql/db_full_backup/sys/memory_global_by_current_bytes.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/innodb_buffer_stats_by_schema.frm to /home/mysql/db_full_backup/sys/innodb_buffer_stats_by_schema.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/host_summary_by_file_io.frm to /home/mysql/db_full_backup/sys/host_summary_by_file_io.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/x@0024statement_analysis.frm to /home/mysql/db_full_backup/sys/x@0024statement_analysis.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/sys_config.frm to /home/mysql/db_full_backup/sys/sys_config.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/x@0024host_summary_by_file_io_type.frm to /home/mysql/db_full_backup/sys/x@0024host_summary_by_file_io_type.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/statements_with_errors_or_warnings.frm to /home/mysql/db_full_backup/sys/statements_with_errors_or_warnings.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/schema_redundant_indexes.frm to /home/mysql/db_full_backup/sys/schema_redundant_indexes.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/schema_table_statistics_with_buffer.frm to /home/mysql/db_full_backup/sys/schema_table_statistics_with_buffer.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/x@0024schema_table_statistics.frm to /home/mysql/db_full_backup/sys/x@0024schema_table_statistics.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/x@0024user_summary_by_statement_type.frm to /home/mysql/db_full_backup/sys/x@0024user_summary_by_statement_type.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/x@0024statements_with_errors_or_warnings.frm to /home/mysql/db_full_backup/sys/x@0024statements_with_errors_or_warnings.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/x@0024schema_table_statistics_with_buffer.frm to /home/mysql/db_full_backup/sys/x@0024schema_table_statistics_with_buffer.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/user_summary_by_file_io_type.frm to /home/mysql/db_full_backup/sys/user_summary_by_file_io_type.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/x@0024io_global_by_wait_by_latency.frm to /home/mysql/db_full_backup/sys/x@0024io_global_by_wait_by_latency.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/user_summary.frm to /home/mysql/db_full_backup/sys/user_summary.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/metrics.frm to /home/mysql/db_full_backup/sys/metrics.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/x@0024host_summary_by_statement_latency.frm to /home/mysql/db_full_backup/sys/x@0024host_summary_by_statement_latency.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/x@0024waits_by_host_by_latency.frm to /home/mysql/db_full_backup/sys/x@0024waits_by_host_by_latency.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/x@0024user_summary.frm to /home/mysql/db_full_backup/sys/x@0024user_summary.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/x@0024latest_file_io.frm to /home/mysql/db_full_backup/sys/x@0024latest_file_io.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/processlist.frm to /home/mysql/db_full_backup/sys/processlist.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/x@0024memory_by_user_by_current_bytes.frm to /home/mysql/db_full_backup/sys/x@0024memory_by_user_by_current_bytes.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/session_ssl_status.frm to /home/mysql/db_full_backup/sys/session_ssl_status.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/x@0024user_summary_by_stages.frm to /home/mysql/db_full_backup/sys/x@0024user_summary_by_stages.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/x@0024innodb_buffer_stats_by_schema.frm to /home/mysql/db_full_backup/sys/x@0024innodb_buffer_stats_by_schema.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/schema_table_statistics.frm to /home/mysql/db_full_backup/sys/schema_table_statistics.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/x@0024ps_digest_95th_percentile_by_avg_us.frm to /home/mysql/db_full_backup/sys/x@0024ps_digest_95th_percentile_by_avg_us.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/host_summary_by_file_io_type.frm to /home/mysql/db_full_backup/sys/host_summary_by_file_io_type.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/sys_config_insert_set_user.TRN to /home/mysql/db_full_backup/sys/sys_config_insert_set_user.TRN
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/waits_global_by_latency.frm to /home/mysql/db_full_backup/sys/waits_global_by_latency.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/statements_with_runtimes_in_95th_percentile.frm to /home/mysql/db_full_backup/sys/statements_with_runtimes_in_95th_percentile.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/user_summary_by_stages.frm to /home/mysql/db_full_backup/sys/user_summary_by_stages.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/x@0024statements_with_full_table_scans.frm to /home/mysql/db_full_backup/sys/x@0024statements_with_full_table_scans.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/innodb_lock_waits.frm to /home/mysql/db_full_backup/sys/innodb_lock_waits.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/sys_config_update_set_user.TRN to /home/mysql/db_full_backup/sys/sys_config_update_set_user.TRN
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/x@0024innodb_lock_waits.frm to /home/mysql/db_full_backup/sys/x@0024innodb_lock_waits.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/schema_index_statistics.frm to /home/mysql/db_full_backup/sys/schema_index_statistics.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/sys_config.TRG to /home/mysql/db_full_backup/sys/sys_config.TRG
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/x@0024io_by_thread_by_latency.frm to /home/mysql/db_full_backup/sys/x@0024io_by_thread_by_latency.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/x@0024memory_global_by_current_bytes.frm to /home/mysql/db_full_backup/sys/x@0024memory_global_by_current_bytes.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/x@0024schema_tables_with_full_table_scans.frm to /home/mysql/db_full_backup/sys/x@0024schema_tables_with_full_table_scans.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/host_summary_by_stages.frm to /home/mysql/db_full_backup/sys/host_summary_by_stages.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/host_summary_by_statement_latency.frm to /home/mysql/db_full_backup/sys/host_summary_by_statement_latency.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/io_global_by_wait_by_latency.frm to /home/mysql/db_full_backup/sys/io_global_by_wait_by_latency.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/memory_by_host_by_current_bytes.frm to /home/mysql/db_full_backup/sys/memory_by_host_by_current_bytes.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/memory_by_user_by_current_bytes.frm to /home/mysql/db_full_backup/sys/memory_by_user_by_current_bytes.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/schema_auto_increment_columns.frm to /home/mysql/db_full_backup/sys/schema_auto_increment_columns.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/io_global_by_wait_by_bytes.frm to /home/mysql/db_full_backup/sys/io_global_by_wait_by_bytes.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/x@0024ps_digest_avg_latency_distribution.frm to /home/mysql/db_full_backup/sys/x@0024ps_digest_avg_latency_distribution.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/wait_classes_global_by_latency.frm to /home/mysql/db_full_backup/sys/wait_classes_global_by_latency.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/io_global_by_file_by_bytes.frm to /home/mysql/db_full_backup/sys/io_global_by_file_by_bytes.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/x@0024schema_index_statistics.frm to /home/mysql/db_full_backup/sys/x@0024schema_index_statistics.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/statements_with_full_table_scans.frm to /home/mysql/db_full_backup/sys/statements_with_full_table_scans.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/x@0024statements_with_runtimes_in_95th_percentile.frm to /home/mysql/db_full_backup/sys/x@0024statements_with_runtimes_in_95th_percentile.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/schema_object_overview.frm to /home/mysql/db_full_backup/sys/schema_object_overview.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/x@0024io_global_by_file_by_latency.frm to /home/mysql/db_full_backup/sys/x@0024io_global_by_file_by_latency.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/io_by_thread_by_latency.frm to /home/mysql/db_full_backup/sys/io_by_thread_by_latency.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/user_summary_by_statement_latency.frm to /home/mysql/db_full_backup/sys/user_summary_by_statement_latency.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/x@0024host_summary_by_file_io.frm to /home/mysql/db_full_backup/sys/x@0024host_summary_by_file_io.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/x@0024memory_by_thread_by_current_bytes.frm to /home/mysql/db_full_backup/sys/x@0024memory_by_thread_by_current_bytes.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/x@0024wait_classes_global_by_avg_latency.frm to /home/mysql/db_full_backup/sys/x@0024wait_classes_global_by_avg_latency.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/waits_by_host_by_latency.frm to /home/mysql/db_full_backup/sys/waits_by_host_by_latency.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/x@0024io_global_by_wait_by_bytes.frm to /home/mysql/db_full_backup/sys/x@0024io_global_by_wait_by_bytes.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/x@0024schema_table_lock_waits.frm to /home/mysql/db_full_backup/sys/x@0024schema_table_lock_waits.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/statements_with_temp_tables.frm to /home/mysql/db_full_backup/sys/statements_with_temp_tables.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 >> log scanned up to (2551222)
180808 00:04:45 [01] Copying ./sys/user_summary_by_statement_type.frm to /home/mysql/db_full_backup/sys/user_summary_by_statement_type.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/x@0024user_summary_by_file_io.frm to /home/mysql/db_full_backup/sys/x@0024user_summary_by_file_io.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/x@0024wait_classes_global_by_latency.frm to /home/mysql/db_full_backup/sys/x@0024wait_classes_global_by_latency.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/host_summary_by_statement_type.frm to /home/mysql/db_full_backup/sys/host_summary_by_statement_type.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/memory_by_thread_by_current_bytes.frm to /home/mysql/db_full_backup/sys/memory_by_thread_by_current_bytes.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/schema_table_lock_waits.frm to /home/mysql/db_full_backup/sys/schema_table_lock_waits.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/memory_global_total.frm to /home/mysql/db_full_backup/sys/memory_global_total.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/statements_with_sorting.frm to /home/mysql/db_full_backup/sys/statements_with_sorting.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/ps_check_lost_instrumentation.frm to /home/mysql/db_full_backup/sys/ps_check_lost_instrumentation.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/latest_file_io.frm to /home/mysql/db_full_backup/sys/latest_file_io.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/schema_unused_indexes.frm to /home/mysql/db_full_backup/sys/schema_unused_indexes.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/x@0024innodb_buffer_stats_by_table.frm to /home/mysql/db_full_backup/sys/x@0024innodb_buffer_stats_by_table.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/x@0024host_summary_by_statement_type.frm to /home/mysql/db_full_backup/sys/x@0024host_summary_by_statement_type.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/x@0024memory_by_host_by_current_bytes.frm to /home/mysql/db_full_backup/sys/x@0024memory_by_host_by_current_bytes.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/schema_tables_with_full_table_scans.frm to /home/mysql/db_full_backup/sys/schema_tables_with_full_table_scans.frm
180808 00:04:45 [01]        ...done
180808 00:04:45 [01] Copying ./sys/x@0024memory_global_total.frm to /home/mysql/db_full_backup/sys/x@0024memory_global_total.frm
180808 00:04:45 [01]        ...done
180808 00:04:46 [01] Copying ./sys/x@0024statements_with_sorting.frm to /home/mysql/db_full_backup/sys/x@0024statements_with_sorting.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./sys/user_summary_by_file_io.frm to /home/mysql/db_full_backup/sys/user_summary_by_file_io.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./sys/waits_by_user_by_latency.frm to /home/mysql/db_full_backup/sys/waits_by_user_by_latency.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./sys/x@0024session.frm to /home/mysql/db_full_backup/sys/x@0024session.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./sys/wait_classes_global_by_avg_latency.frm to /home/mysql/db_full_backup/sys/wait_classes_global_by_avg_latency.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./sys/x@0024ps_schema_table_statistics_io.frm to /home/mysql/db_full_backup/sys/x@0024ps_schema_table_statistics_io.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./sys/innodb_buffer_stats_by_table.frm to /home/mysql/db_full_backup/sys/innodb_buffer_stats_by_table.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./sys/x@0024waits_by_user_by_latency.frm to /home/mysql/db_full_backup/sys/x@0024waits_by_user_by_latency.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./sys/session.frm to /home/mysql/db_full_backup/sys/session.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/setup_actors.frm to /home/mysql/db_full_backup/performance_schema/setup_actors.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/file_instances.frm to /home/mysql/db_full_backup/performance_schema/file_instances.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/memory_summary_by_account_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/memory_summary_by_account_by_event_name.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/events_statements_history_long.frm to /home/mysql/db_full_backup/performance_schema/events_statements_history_long.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/events_transactions_current.frm to /home/mysql/db_full_backup/performance_schema/events_transactions_current.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/db.opt to /home/mysql/db_full_backup/performance_schema/db.opt
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/events_transactions_history_long.frm to /home/mysql/db_full_backup/performance_schema/events_transactions_history_long.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/events_statements_history.frm to /home/mysql/db_full_backup/performance_schema/events_statements_history.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/replication_applier_status.frm to /home/mysql/db_full_backup/performance_schema/replication_applier_status.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/memory_summary_by_host_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/memory_summary_by_host_by_event_name.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/events_waits_summary_by_account_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/events_waits_summary_by_account_by_event_name.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/prepared_statements_instances.frm to /home/mysql/db_full_backup/performance_schema/prepared_statements_instances.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/events_stages_current.frm to /home/mysql/db_full_backup/performance_schema/events_stages_current.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/status_by_account.frm to /home/mysql/db_full_backup/performance_schema/status_by_account.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/events_stages_history.frm to /home/mysql/db_full_backup/performance_schema/events_stages_history.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/events_transactions_summary_by_user_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/events_transactions_summary_by_user_by_event_name.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/table_io_waits_summary_by_table.frm to /home/mysql/db_full_backup/performance_schema/table_io_waits_summary_by_table.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/setup_timers.frm to /home/mysql/db_full_backup/performance_schema/setup_timers.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/objects_summary_global_by_type.frm to /home/mysql/db_full_backup/performance_schema/objects_summary_global_by_type.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/events_stages_summary_by_user_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/events_stages_summary_by_user_by_event_name.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/rwlock_instances.frm to /home/mysql/db_full_backup/performance_schema/rwlock_instances.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/setup_objects.frm to /home/mysql/db_full_backup/performance_schema/setup_objects.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/table_lock_waits_summary_by_table.frm to /home/mysql/db_full_backup/performance_schema/table_lock_waits_summary_by_table.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/events_transactions_summary_by_account_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/events_transactions_summary_by_account_by_event_name.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/replication_applier_status_by_worker.frm to /home/mysql/db_full_backup/performance_schema/replication_applier_status_by_worker.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/setup_consumers.frm to /home/mysql/db_full_backup/performance_schema/setup_consumers.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/host_cache.frm to /home/mysql/db_full_backup/performance_schema/host_cache.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/memory_summary_by_user_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/memory_summary_by_user_by_event_name.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/file_summary_by_instance.frm to /home/mysql/db_full_backup/performance_schema/file_summary_by_instance.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/replication_connection_status.frm to /home/mysql/db_full_backup/performance_schema/replication_connection_status.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/file_summary_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/file_summary_by_event_name.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/events_waits_summary_global_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/events_waits_summary_global_by_event_name.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/user_variables_by_thread.frm to /home/mysql/db_full_backup/performance_schema/user_variables_by_thread.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/events_stages_summary_by_host_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/events_stages_summary_by_host_by_event_name.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/events_transactions_summary_by_thread_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/events_transactions_summary_by_thread_by_event_name.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/global_variables.frm to /home/mysql/db_full_backup/performance_schema/global_variables.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/events_waits_current.frm to /home/mysql/db_full_backup/performance_schema/events_waits_current.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/session_variables.frm to /home/mysql/db_full_backup/performance_schema/session_variables.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/table_handles.frm to /home/mysql/db_full_backup/performance_schema/table_handles.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/events_stages_summary_by_account_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/events_stages_summary_by_account_by_event_name.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/cond_instances.frm to /home/mysql/db_full_backup/performance_schema/cond_instances.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/events_transactions_summary_by_host_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/events_transactions_summary_by_host_by_event_name.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/socket_summary_by_instance.frm to /home/mysql/db_full_backup/performance_schema/socket_summary_by_instance.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/session_connect_attrs.frm to /home/mysql/db_full_backup/performance_schema/session_connect_attrs.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/socket_summary_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/socket_summary_by_event_name.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/session_status.frm to /home/mysql/db_full_backup/performance_schema/session_status.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/session_account_connect_attrs.frm to /home/mysql/db_full_backup/performance_schema/session_account_connect_attrs.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/events_statements_summary_by_thread_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/events_statements_summary_by_thread_by_event_name.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/events_transactions_history.frm to /home/mysql/db_full_backup/performance_schema/events_transactions_history.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/replication_group_member_stats.frm to /home/mysql/db_full_backup/performance_schema/replication_group_member_stats.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/events_statements_current.frm to /home/mysql/db_full_backup/performance_schema/events_statements_current.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/events_waits_summary_by_user_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/events_waits_summary_by_user_by_event_name.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/events_waits_history_long.frm to /home/mysql/db_full_backup/performance_schema/events_waits_history_long.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/replication_connection_configuration.frm to /home/mysql/db_full_backup/performance_schema/replication_connection_configuration.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/status_by_host.frm to /home/mysql/db_full_backup/performance_schema/status_by_host.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/socket_instances.frm to /home/mysql/db_full_backup/performance_schema/socket_instances.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/users.frm to /home/mysql/db_full_backup/performance_schema/users.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/events_statements_summary_global_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/events_statements_summary_global_by_event_name.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/events_transactions_summary_global_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/events_transactions_summary_global_by_event_name.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/events_stages_summary_global_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/events_stages_summary_global_by_event_name.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/status_by_thread.frm to /home/mysql/db_full_backup/performance_schema/status_by_thread.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/performance_timers.frm to /home/mysql/db_full_backup/performance_schema/performance_timers.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/threads.frm to /home/mysql/db_full_backup/performance_schema/threads.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/setup_instruments.frm to /home/mysql/db_full_backup/performance_schema/setup_instruments.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/events_waits_summary_by_host_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/events_waits_summary_by_host_by_event_name.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/mutex_instances.frm to /home/mysql/db_full_backup/performance_schema/mutex_instances.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/metadata_locks.frm to /home/mysql/db_full_backup/performance_schema/metadata_locks.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/replication_applier_status_by_coordinator.frm to /home/mysql/db_full_backup/performance_schema/replication_applier_status_by_coordinator.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/memory_summary_global_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/memory_summary_global_by_event_name.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/events_waits_summary_by_instance.frm to /home/mysql/db_full_backup/performance_schema/events_waits_summary_by_instance.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/variables_by_thread.frm to /home/mysql/db_full_backup/performance_schema/variables_by_thread.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/memory_summary_by_thread_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/memory_summary_by_thread_by_event_name.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 >> log scanned up to (2551222)
180808 00:04:46 [01] Copying ./performance_schema/hosts.frm to /home/mysql/db_full_backup/performance_schema/hosts.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/events_waits_summary_by_thread_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/events_waits_summary_by_thread_by_event_name.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/events_waits_history.frm to /home/mysql/db_full_backup/performance_schema/events_waits_history.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/events_statements_summary_by_user_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/events_statements_summary_by_user_by_event_name.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/events_statements_summary_by_account_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/events_statements_summary_by_account_by_event_name.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/status_by_user.frm to /home/mysql/db_full_backup/performance_schema/status_by_user.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/global_status.frm to /home/mysql/db_full_backup/performance_schema/global_status.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/events_stages_summary_by_thread_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/events_stages_summary_by_thread_by_event_name.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/events_statements_summary_by_host_by_event_name.frm to /home/mysql/db_full_backup/performance_schema/events_statements_summary_by_host_by_event_name.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/events_statements_summary_by_program.frm to /home/mysql/db_full_backup/performance_schema/events_statements_summary_by_program.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/replication_group_members.frm to /home/mysql/db_full_backup/performance_schema/replication_group_members.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/events_stages_history_long.frm to /home/mysql/db_full_backup/performance_schema/events_stages_history_long.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/accounts.frm to /home/mysql/db_full_backup/performance_schema/accounts.frm
180808 00:04:46 [01]        ...done
180808 00:04:46 [01] Copying ./performance_schema/table_io_waits_summary_by_index_usage.frm to /home/mysql/db_full_backup/performance_schema/table_io_waits_summary_by_index_usage.frm
180808 00:04:46 [01]        ...done
180808 00:04:47 [01] Copying ./performance_schema/replication_applier_configuration.frm to /home/mysql/db_full_backup/performance_schema/replication_applier_configuration.frm
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./performance_schema/events_statements_summary_by_digest.frm to /home/mysql/db_full_backup/performance_schema/events_statements_summary_by_digest.frm
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/help_topic.frm to /home/mysql/db_full_backup/mysql/help_topic.frm
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/time_zone.frm to /home/mysql/db_full_backup/mysql/time_zone.frm
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/slow_log.CSV to /home/mysql/db_full_backup/mysql/slow_log.CSV
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/procs_priv.frm to /home/mysql/db_full_backup/mysql/procs_priv.frm
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/db.opt to /home/mysql/db_full_backup/mysql/db.opt
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/slave_master_info.frm to /home/mysql/db_full_backup/mysql/slave_master_info.frm
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/ndb_binlog_index.MYI to /home/mysql/db_full_backup/mysql/ndb_binlog_index.MYI
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/procs_priv.MYI to /home/mysql/db_full_backup/mysql/procs_priv.MYI
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/ndb_binlog_index.frm to /home/mysql/db_full_backup/mysql/ndb_binlog_index.frm
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/columns_priv.MYI to /home/mysql/db_full_backup/mysql/columns_priv.MYI
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/help_category.frm to /home/mysql/db_full_backup/mysql/help_category.frm
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/db.MYD to /home/mysql/db_full_backup/mysql/db.MYD
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/event.frm to /home/mysql/db_full_backup/mysql/event.frm
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/servers.frm to /home/mysql/db_full_backup/mysql/servers.frm
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/help_relation.frm to /home/mysql/db_full_backup/mysql/help_relation.frm
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/proxies_priv.frm to /home/mysql/db_full_backup/mysql/proxies_priv.frm
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/time_zone_transition.frm to /home/mysql/db_full_backup/mysql/time_zone_transition.frm
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/time_zone_leap_second.frm to /home/mysql/db_full_backup/mysql/time_zone_leap_second.frm
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/time_zone_name.frm to /home/mysql/db_full_backup/mysql/time_zone_name.frm
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/func.MYI to /home/mysql/db_full_backup/mysql/func.MYI
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/innodb_table_stats.frm to /home/mysql/db_full_backup/mysql/innodb_table_stats.frm
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/user.frm to /home/mysql/db_full_backup/mysql/user.frm
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/user.MYI to /home/mysql/db_full_backup/mysql/user.MYI
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/columns_priv.MYD to /home/mysql/db_full_backup/mysql/columns_priv.MYD
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/slave_relay_log_info.frm to /home/mysql/db_full_backup/mysql/slave_relay_log_info.frm
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/user.MYD to /home/mysql/db_full_backup/mysql/user.MYD
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/server_cost.frm to /home/mysql/db_full_backup/mysql/server_cost.frm
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/proxies_priv.MYD to /home/mysql/db_full_backup/mysql/proxies_priv.MYD
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/procs_priv.MYD to /home/mysql/db_full_backup/mysql/procs_priv.MYD
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/proc.frm to /home/mysql/db_full_backup/mysql/proc.frm
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/tables_priv.MYI to /home/mysql/db_full_backup/mysql/tables_priv.MYI
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/func.frm to /home/mysql/db_full_backup/mysql/func.frm
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/proxies_priv.MYI to /home/mysql/db_full_backup/mysql/proxies_priv.MYI
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/tables_priv.MYD to /home/mysql/db_full_backup/mysql/tables_priv.MYD
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/proc.MYD to /home/mysql/db_full_backup/mysql/proc.MYD
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/tables_priv.frm to /home/mysql/db_full_backup/mysql/tables_priv.frm
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/general_log.CSV to /home/mysql/db_full_backup/mysql/general_log.CSV
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/slow_log.frm to /home/mysql/db_full_backup/mysql/slow_log.frm
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/engine_cost.frm to /home/mysql/db_full_backup/mysql/engine_cost.frm
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/slow_log.CSM to /home/mysql/db_full_backup/mysql/slow_log.CSM
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/innodb_index_stats.frm to /home/mysql/db_full_backup/mysql/innodb_index_stats.frm
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/db.MYI to /home/mysql/db_full_backup/mysql/db.MYI
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/time_zone_transition_type.frm to /home/mysql/db_full_backup/mysql/time_zone_transition_type.frm
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/slave_worker_info.frm to /home/mysql/db_full_backup/mysql/slave_worker_info.frm
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/plugin.frm to /home/mysql/db_full_backup/mysql/plugin.frm
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/general_log.CSM to /home/mysql/db_full_backup/mysql/general_log.CSM
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/db.frm to /home/mysql/db_full_backup/mysql/db.frm
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/columns_priv.frm to /home/mysql/db_full_backup/mysql/columns_priv.frm
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/ndb_binlog_index.MYD to /home/mysql/db_full_backup/mysql/ndb_binlog_index.MYD
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/event.MYI to /home/mysql/db_full_backup/mysql/event.MYI
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/gtid_executed.frm to /home/mysql/db_full_backup/mysql/gtid_executed.frm
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/event.MYD to /home/mysql/db_full_backup/mysql/event.MYD
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/help_keyword.frm to /home/mysql/db_full_backup/mysql/help_keyword.frm
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/func.MYD to /home/mysql/db_full_backup/mysql/func.MYD
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/proc.MYI to /home/mysql/db_full_backup/mysql/proc.MYI
180808 00:04:47 [01]        ...done
180808 00:04:47 [01] Copying ./mysql/general_log.frm to /home/mysql/db_full_backup/mysql/general_log.frm
180808 00:04:47 [01]        ...done
180808 00:04:47 Finished backing up non-InnoDB tables and files
180808 00:04:47 [00] Writing /home/mysql/db_full_backup/xtrabackup_binlog_info
180808 00:04:47 [00]        ...done
180808 00:04:47 Executing FLUSH NO_WRITE_TO_BINLOG ENGINE LOGS...
xtrabackup: The latest check point (for incremental): '2551213'
xtrabackup: Stopping log copying thread.
.180808 00:04:47 >> log scanned up to (2551222)

180808 00:04:47 Executing UNLOCK TABLES
180808 00:04:47 All tables unlocked
180808 00:04:47 [00] Copying ib_buffer_pool to /home/mysql/db_full_backup/ib_buffer_pool
180808 00:04:47 [00]        ...done
180808 00:04:47 Backup created in directory '/home/mysql/db_full_backup/'
MySQL binlog position: filename 'mysql-bin.000003', position '596'
180808 00:04:47 [00] Writing /home/mysql/db_full_backup/backup-my.cnf
180808 00:04:47 [00]        ...done
180808 00:04:47 [00] Writing /home/mysql/db_full_backup/xtrabackup_info
180808 00:04:47 [00]        ...done
xtrabackup: Transaction log of lsn (2551213) to (2551222) was copied.
180808 00:04:48 completed OK!



```



## 传送备份到从库

```sql
[mysql@node1 ~]$ scp -r db_full_backup 192.168.2.62:/home/mysql/
The authenticity of host '192.168.2.62 (192.168.2.62)' can't be established.
RSA key fingerprint is f0:60:1f:47:af:50:21:c4:f5:98:87:2a:1e:9c:3f:7e.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.2.62' (RSA) to the list of known hosts.
mysql@192.168.2.62's password:
performance_timers.frm                                                                                                               100% 8776     8.6KB/s   00:00
table_handles.frm                                                                                                                    100% 8928     8.7KB/s   00:00
session_status.frm                                                                                                                   100% 8628     8.4KB/s   00:00
memory_summary_by_account_by_event_name.frm                                                                                          100% 9240     9.0KB/s   00:00
events_waits_history.frm                                                                                                             100% 9401     9.2KB/s   00:00
objects_summary_global_by_type.frm                                                                                                   100% 8908     8.7KB/s   00:00
events_waits_summary_by_account_by_event_name.frm                                                                                    100% 8874     8.7KB/s   00:00
events_transactions_summary_global_by_event_name.frm                                                                                 100% 9408     9.2KB/s   00:00
events_transactions_current.frm                                                                                                      100% 9800     9.6KB/s   00:00
metadata_locks.frm                                                                                                                   100% 8998     8.8KB/s   00:00
table_io_waits_summary_by_table.frm                                                                                                  100%   10KB  10.2KB/s   00:00
replication_applier_status_by_coordinator.frm                                                                                        100% 8849     8.6KB/s   00:00
hosts.frm                                                                                                                            100% 8676     8.5KB/s   00:00
status_by_thread.frm                                                                                                                 100% 8668     8.5KB/s   00:00
memory_summary_global_by_event_name.frm                                                                                              100% 9180     9.0KB/s   00:00
events_stages_summary_global_by_event_name.frm                                                                                       100% 8814     8.6KB/s   00:00
replication_group_member_stats.frm                                                                                                   100% 9134     8.9KB/s   00:00
events_waits_summary_by_thread_by_event_name.frm                                                                                     100% 8854     8.7KB/s   00:00
events_stages_summary_by_thread_by_event_name.frm                                                                                    100% 8854     8.7KB/s   00:00
events_stages_summary_by_account_by_event_name.frm                                                                                   100% 8874     8.7KB/s   00:00
socket_summary_by_instance.frm                                                                                                       100% 9804     9.6KB/s   00:00
events_statements_summary_by_user_by_event_name.frm                                                                                  100% 9970     9.7KB/s   00:00
rwlock_instances.frm                                                                                                                 100% 8758     8.6KB/s   00:00
global_status.frm                                                                                                                    100% 8628     8.4KB/s   00:00
events_waits_summary_global_by_event_name.frm                                                                                        100% 8814     8.6KB/s   00:00
session_account_connect_attrs.frm                                                                                                    100% 8716     8.5KB/s   00:00
events_statements_history_long.frm                                                                                                   100%   10KB  10.4KB/s   00:00
events_stages_history.frm                                                                                                            100% 9103     8.9KB/s   00:00
cond_instances.frm                                                                                                                   100% 8624     8.4KB/s   00:00
events_transactions_history.frm                                                                                                      100% 9800     9.6KB/s   00:00
events_transactions_summary_by_host_by_event_name.frm                                                                                100% 9438     9.2KB/s   00:00
users.frm                                                                                                                            100% 8676     8.5KB/s   00:00
events_stages_summary_by_host_by_event_name.frm                                                                                      100% 8844     8.6KB/s   00:00
replication_connection_configuration.frm                                                                                             100%   17KB  17.3KB/s   00:00
events_transactions_summary_by_account_by_event_name.frm                                                                             100% 9468     9.3KB/s   00:00
session_variables.frm                                                                                                                100% 8628     8.4KB/s   00:00
events_stages_history_long.frm                                                                                                       100% 9103     8.9KB/s   00:00
setup_timers.frm                                                                                                                     100% 8650     8.5KB/s   00:00
host_cache.frm                                                                                                                       100%   10KB  10.2KB/s   00:00
mutex_instances.frm                                                                                                                  100% 8684     8.5KB/s   00:00
file_instances.frm                                                                                                                   100% 8654     8.5KB/s   00:00
table_io_waits_summary_by_index_usage.frm                                                                                            100%   10KB  10.3KB/s   00:00
replication_connection_status.frm                                                                                                    100% 9215     9.0KB/s   00:00
events_waits_history_long.frm                                                                                                        100% 9401     9.2KB/s   00:00
socket_summary_by_event_name.frm                                                                                                     100% 9740     9.5KB/s   00:00
events_waits_summary_by_user_by_event_name.frm                                                                                       100% 8844     8.6KB/s   00:00
events_waits_current.frm                                                                                                             100% 9401     9.2KB/s   00:00
session_connect_attrs.frm                                                                                                            100% 8716     8.5KB/s   00:00
setup_instruments.frm                                                                                                                100% 8637     8.4KB/s   00:00
events_stages_current.frm                                                                                                            100% 9103     8.9KB/s   00:00
status_by_user.frm                                                                                                                   100% 8658     8.5KB/s   00:00
table_lock_waits_summary_by_table.frm                                                                                                100%   13KB  12.9KB/s   00:00
replication_group_members.frm                                                                                                        100% 8750     8.5KB/s   00:00
memory_summary_by_thread_by_event_name.frm                                                                                           100% 9220     9.0KB/s   00:00
setup_consumers.frm                                                                                                                  100% 8605     8.4KB/s   00:00
events_transactions_history_long.frm                                                                                                 100% 9800     9.6KB/s   00:00
file_summary_by_event_name.frm                                                                                                       100% 9740     9.5KB/s   00:00
prepared_statements_instances.frm                                                                                                    100%   10KB  10.3KB/s   00:00
events_statements_history.frm                                                                                                        100%   10KB  10.4KB/s   00:00
events_statements_summary_by_thread_by_event_name.frm                                                                                100% 9980     9.8KB/s   00:00
threads.frm                                                                                                                          100% 9335     9.1KB/s   00:00
replication_applier_configuration.frm                                                                                                100% 8624     8.4KB/s   00:00
events_statements_summary_by_digest.frm                                                                                              100%   10KB   9.9KB/s   00:00
memory_summary_by_user_by_event_name.frm                                                                                             100% 9210     9.0KB/s   00:00
events_transactions_summary_by_user_by_event_name.frm                                                                                100% 9438     9.2KB/s   00:00
events_statements_current.frm                                                                                                        100%   10KB  10.4KB/s   00:00
replication_applier_status.frm                                                                                                       100% 8759     8.6KB/s   00:00
variables_by_thread.frm                                                                                                              100% 8668     8.5KB/s   00:00
socket_instances.frm                                                                                                                 100% 8818     8.6KB/s   00:00
events_statements_summary_by_program.frm                                                                                             100%   10KB  10.1KB/s   00:00
status_by_host.frm                                                                                                                   100% 8658     8.5KB/s   00:00
events_statements_summary_global_by_event_name.frm                                                                                   100% 9940     9.7KB/s   00:00
events_waits_summary_by_host_by_event_name.frm                                                                                       100% 8844     8.6KB/s   00:00
events_statements_summary_by_host_by_event_name.frm                                                                                  100% 9970     9.7KB/s   00:00
status_by_account.frm                                                                                                                100% 8688     8.5KB/s   00:00
memory_summary_by_host_by_event_name.frm                                                                                             100% 9210     9.0KB/s   00:00
events_transactions_summary_by_thread_by_event_name.frm                                                                              100% 9448     9.2KB/s   00:00
events_statements_summary_by_account_by_event_name.frm                                                                               100%   10KB   9.8KB/s   00:00
replication_applier_status_by_worker.frm                                                                                             100% 8953     8.7KB/s   00:00
setup_objects.frm                                                                                                                    100% 8784     8.6KB/s   00:00
events_stages_summary_by_user_by_event_name.frm                                                                                      100% 8844     8.6KB/s   00:00
user_variables_by_thread.frm                                                                                                         100% 8668     8.5KB/s   00:00
global_variables.frm                                                                                                                 100% 8628     8.4KB/s   00:00
db.opt                                                                                                                               100%   61     0.1KB/s   00:00
accounts.frm                                                                                                                         100% 8706     8.5KB/s   00:00
file_summary_by_instance.frm                                                                                                         100% 9844     9.6KB/s   00:00
events_waits_summary_by_instance.frm                                                                                                 100% 8878     8.7KB/s   00:00
setup_actors.frm                                                                                                                     100% 8701     8.5KB/s   00:00
ib_buffer_pool                                                                                                                       100%  357     0.4KB/s   00:00
backup-my.cnf                                                                                                                        100%  424     0.4KB/s   00:00
db.MYI                                                                                                                               100% 5120     5.0KB/s   00:00
plugin.ibd                                                                                                                           100%   96KB  96.0KB/s   00:00
db.MYD                                                                                                                               100%  976     1.0KB/s   00:00
event.MYD                                                                                                                            100%    0     0.0KB/s   00:00
tables_priv.MYI                                                                                                                      100% 9216     9.0KB/s   00:00
time_zone_name.frm                                                                                                                   100% 8606     8.4KB/s   00:00
innodb_table_stats.frm                                                                                                               100% 8830     8.6KB/s   00:00
general_log.CSV                                                                                                                      100%    0     0.0KB/s   00:00
engine_cost.frm                                                                                                                      100% 8780     8.6KB/s   00:00
help_keyword.ibd                                                                                                                     100%  240KB 240.0KB/s   00:00
slow_log.CSV                                                                                                                         100%    0     0.0KB/s   00:00
proc.MYD                                                                                                                             100%  293KB 293.5KB/s   00:00
slave_relay_log_info.frm                                                                                                             100% 9468     9.3KB/s   00:00
innodb_index_stats.frm                                                                                                               100%   13KB  12.7KB/s   00:00
innodb_table_stats.ibd                                                                                                               100%   96KB  96.0KB/s   00:00
slave_worker_info.ibd                                                                                                                100%   96KB  96.0KB/s   00:00
help_topic.ibd                                                                                                                       100% 9216KB   9.0MB/s   00:00
plugin.frm                                                                                                                           100% 8586     8.4KB/s   00:00
columns_priv.frm                                                                                                                     100% 8820     8.6KB/s   00:00
help_relation.ibd                                                                                                                    100%  128KB 128.0KB/s   00:00
db.frm                                                                                                                               100% 9582     9.4KB/s   00:00
proc.frm                                                                                                                             100% 9996     9.8KB/s   00:00
event.MYI                                                                                                                            100% 2048     2.0KB/s   00:00
user.MYI                                                                                                                             100% 4096     4.0KB/s   00:00
proxies_priv.MYI                                                                                                                     100% 9216     9.0KB/s   00:00
slow_log.CSM                                                                                                                         100%   35     0.0KB/s   00:00
func.frm                                                                                                                             100% 8665     8.5KB/s   00:00
help_category.ibd                                                                                                                    100%  112KB 112.0KB/s   00:00
procs_priv.frm                                                                                                                       100% 8875     8.7KB/s   00:00
time_zone_leap_second.frm                                                                                                            100% 8624     8.4KB/s   00:00
time_zone_transition_type.ibd                                                                                                        100%   96KB  96.0KB/s   00:00
engine_cost.ibd                                                                                                                      100%   96KB  96.0KB/s   00:00
server_cost.ibd                                                                                                                      100%   96KB  96.0KB/s   00:00
proxies_priv.frm                                                                                                                     100% 8800     8.6KB/s   00:00
ndb_binlog_index.MYD                                                                                                                 100%    0     0.0KB/s   00:00
server_cost.frm                                                                                                                      100% 8692     8.5KB/s   00:00
slave_master_info.frm                                                                                                                100%   11KB  10.7KB/s   00:00
help_keyword.frm                                                                                                                     100% 8612     8.4KB/s   00:00
time_zone_leap_second.ibd                                                                                                            100%   96KB  96.0KB/s   00:00
columns_priv.MYD                                                                                                                     100%    0     0.0KB/s   00:00
help_relation.frm                                                                                                                    100% 8630     8.4KB/s   00:00
gtid_executed.frm                                                                                                                    100% 8784     8.6KB/s   00:00
slave_worker_info.frm                                                                                                                100% 9364     9.1KB/s   00:00
time_zone.frm                                                                                                                        100% 8636     8.4KB/s   00:00
ndb_binlog_index.MYI                                                                                                                 100% 1024     1.0KB/s   00:00
slave_master_info.ibd                                                                                                                100%   96KB  96.0KB/s   00:00
time_zone_transition.ibd                                                                                                             100%   96KB  96.0KB/s   00:00
time_zone_transition.frm                                                                                                             100% 8686     8.5KB/s   00:00
ndb_binlog_index.frm                                                                                                                 100% 8986     8.8KB/s   00:00
procs_priv.MYD                                                                                                                       100%    0     0.0KB/s   00:00
user.MYD                                                                                                                             100%  520     0.5KB/s   00:00
event.frm                                                                                                                            100%   10KB  10.0KB/s   00:00
procs_priv.MYI                                                                                                                       100% 4096     4.0KB/s   00:00
innodb_index_stats.ibd                                                                                                               100%   96KB  96.0KB/s   00:00
slow_log.frm                                                                                                                         100% 9016     8.8KB/s   00:00
servers.ibd                                                                                                                          100%   96KB  96.0KB/s   00:00
general_log.CSM                                                                                                                      100%   35     0.0KB/s   00:00
help_category.frm                                                                                                                    100% 8700     8.5KB/s   00:00
user.frm                                                                                                                             100%   11KB  10.6KB/s   00:00
general_log.frm                                                                                                                      100% 8776     8.6KB/s   00:00
tables_priv.MYD                                                                                                                      100% 1894     1.9KB/s   00:00
time_zone_transition_type.frm                                                                                                        100% 8748     8.5KB/s   00:00
servers.frm                                                                                                                          100% 8838     8.6KB/s   00:00
func.MYD                                                                                                                             100%    0     0.0KB/s   00:00
gtid_executed.ibd                                                                                                                    100%   96KB  96.0KB/s   00:00
help_topic.frm                                                                                                                       100% 8770     8.6KB/s   00:00
func.MYI                                                                                                                             100% 1024     1.0KB/s   00:00
db.opt                                                                                                                               100%   65     0.1KB/s   00:00
time_zone_name.ibd                                                                                                                   100%   96KB  96.0KB/s   00:00
tables_priv.frm                                                                                                                      100% 8955     8.8KB/s   00:00
proxies_priv.MYD                                                                                                                     100%  837     0.8KB/s   00:00
slave_relay_log_info.ibd                                                                                                             100%   96KB  96.0KB/s   00:00
time_zone.ibd                                                                                                                        100%   96KB  96.0KB/s   00:00
columns_priv.MYI                                                                                                                     100% 4096     4.0KB/s   00:00
proc.MYI                                                                                                                             100% 4096     4.0KB/s   00:00
ibdata1                                                                                                                              100%   12MB  12.0MB/s   00:01
xtrabackup_checkpoints                                                                                                               100%  113     0.1KB/s   00:00
xtrabackup_binlog_info                                                                                                               100%   21     0.0KB/s   00:00
xtrabackup_info                                                                                                                      100%  522     0.5KB/s   00:00
xtrabackup_logfile                                                                                                                   100% 2560     2.5KB/s   00:00
x@0024io_global_by_file_by_latency.frm                                                                                               100% 2266     2.2KB/s   00:00
x@0024user_summary_by_statement_latency.frm                                                                                          100% 3923     3.8KB/s   00:00
x@0024io_by_thread_by_latency.frm                                                                                                    100% 4154     4.1KB/s   00:00
memory_by_thread_by_current_bytes.frm                                                                                                100% 3214     3.1KB/s   00:00
ps_check_lost_instrumentation.frm                                                                                                    100% 1093     1.1KB/s   00:00
user_summary_by_statement_type.frm                                                                                                   100% 4425     4.3KB/s   00:00
x@0024io_global_by_wait_by_bytes.frm                                                                                                 100% 4623     4.5KB/s   00:00
wait_classes_global_by_latency.frm                                                                                                   100% 3420     3.3KB/s   00:00
x@0024memory_by_user_by_current_bytes.frm                                                                                            100% 3227     3.2KB/s   00:00
schema_table_statistics_with_buffer.frm                                                                                              100% 5400     5.3KB/s   00:00
memory_global_by_current_bytes.frm                                                                                                   100% 3405     3.3KB/s   00:00
x@0024memory_by_thread_by_current_bytes.frm                                                                                          100% 2967     2.9KB/s   00:00
metrics.frm                                                                                                                          100%   10KB  10.0KB/s   00:00
x@0024innodb_buffer_stats_by_schema.frm                                                                                              100% 2430     2.4KB/s   00:00
x@0024schema_table_statistics.frm                                                                                                    100% 3391     3.3KB/s   00:00
x@0024host_summary.frm                                                                                                               100% 3651     3.6KB/s   00:00
x@0024host_summary_by_statement_type.frm                                                                                             100% 4282     4.2KB/s   00:00
innodb_buffer_stats_by_schema.frm                                                                                                    100% 2492     2.4KB/s   00:00
memory_global_total.frm                                                                                                              100%  827     0.8KB/s   00:00
processlist.frm                                                                                                                      100% 8183     8.0KB/s   00:00
x@0024schema_table_lock_waits.frm                                                                                                    100% 5029     4.9KB/s   00:00
io_global_by_wait_by_bytes.frm                                                                                                       100% 5166     5.0KB/s   00:00
session.frm                                                                                                                          100% 3044     3.0KB/s   00:00
x@0024host_summary_by_file_io_type.frm                                                                                               100% 2858     2.8KB/s   00:00
user_summary_by_statement_latency.frm                                                                                                100% 4100     4.0KB/s   00:00
x@0024host_summary_by_statement_latency.frm                                                                                          100% 3923     3.8KB/s   00:00
x@0024memory_global_total.frm                                                                                                        100%  765     0.8KB/s   00:00
x@0024io_global_by_wait_by_latency.frm                                                                                               100% 4381     4.3KB/s   00:00
sys_config_insert_set_user.TRN                                                                                                       100%   42     0.0KB/s   00:00
x@0024statements_with_temp_tables.frm                                                                                                100% 4192     4.1KB/s   00:00
schema_unused_indexes.frm                                                                                                            100% 2294     2.2KB/s   00:00
x@0024user_summary_by_file_io.frm                                                                                                    100% 2284     2.2KB/s   00:00
io_by_thread_by_latency.frm                                                                                                          100% 4390     4.3KB/s   00:00
statements_with_sorting.frm                                                                                                          100% 4305     4.2KB/s   00:00
x@0024ps_schema_table_statistics_io.frm                                                                                              100% 2926     2.9KB/s   00:00
schema_tables_with_full_table_scans.frm                                                                                              100% 1982     1.9KB/s   00:00
x@0024waits_by_user_by_latency.frm                                                                                                   100% 3236     3.2KB/s   00:00
x@0024user_summary_by_stages.frm                                                                                                     100% 2607     2.6KB/s   00:00
x@0024user_summary_by_statement_type.frm                                                                                             100% 4248     4.2KB/s   00:00
session_ssl_status.frm                                                                                                               100% 2034     2.0KB/s   00:00
io_global_by_file_by_bytes.frm                                                                                                       100% 4169     4.1KB/s   00:00
x@0024statements_with_full_table_scans.frm                                                                                           100% 5481     5.4KB/s   00:00
x@0024statement_analysis.frm                                                                                                         100% 6738     6.6KB/s   00:00
version.frm                                                                                                                          100%  461     0.5KB/s   00:00
schema_object_overview.frm                                                                                                           100% 3725     3.6KB/s   00:00
x@0024memory_global_by_current_bytes.frm                                                                                             100% 3157     3.1KB/s   00:00
schema_index_statistics.frm                                                                                                          100% 3606     3.5KB/s   00:00
x@0024memory_by_host_by_current_bytes.frm                                                                                            100% 3227     3.2KB/s   00:00
statement_analysis.frm                                                                                                               100% 7048     6.9KB/s   00:00
user_summary.frm                                                                                                                     100% 4697     4.6KB/s   00:00
x@0024user_summary_by_file_io_type.frm                                                                                               100% 2806     2.7KB/s   00:00
schema_table_statistics.frm                                                                                                          100% 3987     3.9KB/s   00:00
x@0024wait_classes_global_by_latency.frm                                                                                             100% 3187     3.1KB/s   00:00
waits_by_user_by_latency.frm                                                                                                         100% 3413     3.3KB/s   00:00
io_global_by_file_by_latency.frm                                                                                                     100% 2561     2.5KB/s   00:00
statements_with_errors_or_warnings.frm                                                                                               100% 3758     3.7KB/s   00:00
host_summary_by_stages.frm                                                                                                           100% 2758     2.7KB/s   00:00
x@0024schema_index_statistics.frm                                                                                                    100% 3370     3.3KB/s   00:00
x@0024session.frm                                                                                                                    100% 3170     3.1KB/s   00:00
innodb_buffer_stats_by_table.frm                                                                                                     100% 2782     2.7KB/s   00:00
schema_auto_increment_columns.frm                                                                                                    100% 6163     6.0KB/s   00:00
sys_config_update_set_user.TRN                                                                                                       100%   42     0.0KB/s   00:00
io_global_by_wait_by_latency.frm                                                                                                     100% 4983     4.9KB/s   00:00
sys_config.ibd                                                                                                                       100%   96KB  96.0KB/s   00:00
x@0024wait_classes_global_by_avg_latency.frm                                                                                         100% 3236     3.2KB/s   00:00
x@0024user_summary.frm                                                                                                               100% 4396     4.3KB/s   00:00
user_summary_by_stages.frm                                                                                                           100% 2725     2.7KB/s   00:00
x@0024latest_file_io.frm                                                                                                             100% 3261     3.2KB/s   00:00
host_summary_by_statement_type.frm                                                                                                   100% 4459     4.4KB/s   00:00
statements_with_full_table_scans.frm                                                                                                 100% 5656     5.5KB/s   00:00
x@0024statements_with_errors_or_warnings.frm                                                                                         100% 3684     3.6KB/s   00:00
user_summary_by_file_io_type.frm                                                                                                     100% 2924     2.9KB/s   00:00
x@0024ps_digest_95th_percentile_by_avg_us.frm                                                                                        100% 1747     1.7KB/s   00:00
x@0024innodb_buffer_stats_by_table.frm                                                                                               100% 2720     2.7KB/s   00:00
waits_global_by_latency.frm                                                                                                          100% 2427     2.4KB/s   00:00
x@0024processlist.frm                                                                                                                100% 7679     7.5KB/s   00:00
x@0024statements_with_sorting.frm                                                                                                    100% 4172     4.1KB/s   00:00
host_summary_by_file_io.frm                                                                                                          100% 2343     2.3KB/s   00:00
innodb_lock_waits.frm                                                                                                                100% 5264     5.1KB/s   00:00
user_summary_by_file_io.frm                                                                                                          100% 2343     2.3KB/s   00:00
host_summary.frm                                                                                                                     100% 4005     3.9KB/s   00:00
waits_by_host_by_latency.frm                                                                                                         100% 3210     3.1KB/s   00:00
x@0024statements_with_runtimes_in_95th_percentile.frm                                                                                100% 3228     3.2KB/s   00:00
memory_by_host_by_current_bytes.frm                                                                                                  100% 3475     3.4KB/s   00:00
x@0024waits_global_by_latency.frm                                                                                                    100% 2250     2.2KB/s   00:00
memory_by_user_by_current_bytes.frm                                                                                                  100% 3475     3.4KB/s   00:00
sys_config.frm                                                                                                                       100% 8672     8.5KB/s   00:00
x@0024ps_digest_avg_latency_distribution.frm                                                                                         100%  844     0.8KB/s   00:00
x@0024io_global_by_file_by_bytes.frm                                                                                                 100% 3800     3.7KB/s   00:00
x@0024innodb_lock_waits.frm                                                                                                          100% 5116     5.0KB/s   00:00
x@0024host_summary_by_stages.frm                                                                                                     100% 2640     2.6KB/s   00:00
x@0024schema_flattened_keys.frm                                                                                                      100% 2469     2.4KB/s   00:00
statements_with_runtimes_in_95th_percentile.frm                                                                                      100% 3479     3.4KB/s   00:00
db.opt                                                                                                                               100%   61     0.1KB/s   00:00
wait_classes_global_by_avg_latency.frm                                                                                               100% 3580     3.5KB/s   00:00
statements_with_temp_tables.frm                                                                                                      100% 4325     4.2KB/s   00:00
schema_redundant_indexes.frm                                                                                                         100% 5226     5.1KB/s   00:00
latest_file_io.frm                                                                                                                   100% 3441     3.4KB/s   00:00
sys_config.TRG                                                                                                                       100%  720     0.7KB/s   00:00
x@0024host_summary_by_file_io.frm                                                                                                    100% 2284     2.2KB/s   00:00
x@0024schema_tables_with_full_table_scans.frm                                                                                        100% 1923     1.9KB/s   00:00
schema_table_lock_waits.frm                                                                                                          100% 5103     5.0KB/s   00:00
x@0024schema_table_statistics_with_buffer.frm                                                                                        100% 4679     4.6KB/s   00:00
host_summary_by_file_io_type.frm                                                                                                     100% 2976     2.9KB/s   00:00
host_summary_by_statement_latency.frm                                                                                                100% 4100     4.0KB/s   00:00
x@0024waits_by_host_by_latency.frm  

```

## 从库进行恢复 

### 准备阶段


```sql

[mysql@node2 ~]$ innobackupex --defaults-file=/mysql1/mysql1.cnf --user root --port 3306 --apply-log /home/mysql/mysql_backup
180808 00:09:46 innobackupex: Starting the apply-log operation

IMPORTANT: Please check that the apply-log run completes successfully.
           At the end of a successful apply-log run innobackupex
           prints "completed OK!".

innobackupex version 2.4.8 based on MySQL server 5.7.13 Linux (x86_64) (revision id: 97330f7)
innobackupex: Can't change dir to '/home/mysql/mysql_backup/' (Errcode: 2 - No such file or directory)
xtrabackup: cannot my_setwd /home/mysql/mysql_backup/
[mysql@node2 ~]$ innobackupex --defaults-file=/mysql1/mysql1.cnf   --apply-log /home/mysql/
.bash_history   .bash_logout    .bash_profile   .bashrc         db_full_backup/ .gnome2/        .mozilla/       .mysql_history  .viminfo
[mysql@node2 ~]$ innobackupex --defaults-file=/mysql1/mysql1.cnf   --apply-log /home/mysql/db_full_backup
180808 00:10:09 innobackupex: Starting the apply-log operation

IMPORTANT: Please check that the apply-log run completes successfully.
           At the end of a successful apply-log run innobackupex
           prints "completed OK!".

innobackupex version 2.4.8 based on MySQL server 5.7.13 Linux (x86_64) (revision id: 97330f7)
xtrabackup: cd to /home/mysql/db_full_backup/
xtrabackup: This target seems to be not prepared yet.
InnoDB: Number of pools: 1
xtrabackup: xtrabackup_logfile detected: size=8388608, start_lsn=(2551213)
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
InnoDB: Log scan progressed past the checkpoint lsn 2551213
InnoDB: Doing recovery: scanned up to log sequence number 2551222 (0%)
InnoDB: Doing recovery: scanned up to log sequence number 2551222 (0%)
InnoDB: Database was not shutdown normally!
InnoDB: Starting crash recovery.
InnoDB: Creating shared tablespace for temporary tables
InnoDB: Setting file './ibtmp1' size to 12 MB. Physically writing the file full; Please wait ...
InnoDB: File './ibtmp1' size is now 12 MB.
InnoDB: 96 redo rollback segment(s) found. 1 redo rollback segment(s) are active.
InnoDB: 32 non-redo rollback segment(s) are active.
InnoDB: 5.7.13 started; log sequence number 2551222

xtrabackup: starting shutdown with innodb_fast_shutdown = 1
InnoDB: FTS optimize thread exiting.
InnoDB: Starting shutdown...
InnoDB: Shutdown completed; log sequence number 2551241
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
InnoDB: New log files created, LSN=2551241
InnoDB: Highest supported file format is Barracuda.
InnoDB: Log scan progressed past the checkpoint lsn 2551308
InnoDB: Doing recovery: scanned up to log sequence number 2551317 (0%)
InnoDB: Doing recovery: scanned up to log sequence number 2551317 (0%)
InnoDB: Database was not shutdown normally!
InnoDB: Starting crash recovery.
InnoDB: Removed temporary tablespace data file: "ibtmp1"
InnoDB: Creating shared tablespace for temporary tables
InnoDB: Setting file './ibtmp1' size to 12 MB. Physically writing the file full; Please wait ...
InnoDB: File './ibtmp1' size is now 12 MB.
InnoDB: 96 redo rollback segment(s) found. 1 redo rollback segment(s) are active.
InnoDB: 32 non-redo rollback segment(s) are active.
InnoDB: Waiting for purge to start
InnoDB: 5.7.13 started; log sequence number 2551317
xtrabackup: starting shutdown with innodb_fast_shutdown = 1
InnoDB: FTS optimize thread exiting.
InnoDB: Starting shutdown...
InnoDB: Shutdown completed; log sequence number 2551336
180808 00:10:15 completed OK!

```

### 恢复备份到datadir
```sql
 cp -r /home/mysql/db_full_backup/ /mysql1/data
```

### 查看日志信息

```sql

[mysql@node2 ~]$ cat /mysql1/data/xtrabackup_info
uuid = 9ba2413e-9a5b-11e8-8a25-000c291f013a
name =
tool_name = innobackupex
tool_command = --defaults-file=/mysql1/mysql1.cnf --no-timestamp -u root -p root /home/mysql/db_full_backup/
tool_version = 2.4.8
ibbackup_version = 2.4.8
server_version = 5.7.21-log
start_time = 2018-08-08 00:04:43
end_time = 2018-08-08 00:04:47
lock_time = 0
binlog_pos = filename 'mysql-bin.000003', position '596'
innodb_from_lsn = 0
innodb_to_lsn = 2551213
partial = N
incremental = N
format = file
compact = N
compressed = N
encrypted = N


```
### 启动从库

```sql

[mysql@node2 data]$ mysqld --defaults-file=/mysql1/mysql1.cnf  &
[1] 2033

```
### 开始同步


```sql


[mysql@node2 data]$ mysql -S /tmp/mysql1.sock -uroot -proot
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.21-log MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> change master to master_host='192.168.2.61' ,master_port=3306, master_user='dao',master_password='dao',master_log_file='mysql-bin.000003' ,master_log_pos=596 ;
Query OK, 0 rows affected, 2 warnings (0.02 sec)

mysql> start slave ;
Query OK, 0 rows affected (0.01 sec)



```


### 查看同步状态

```sql


mysql> show slave status \G ;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.2.61
                  Master_User: dao
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000003
          Read_Master_Log_Pos: 951
               Relay_Log_File: node2-relay-bin.000003
                Relay_Log_Pos: 675
        Relay_Master_Log_File: mysql-bin.000003
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
          Exec_Master_Log_Pos: 951
              Relay_Log_Space: 882
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


```