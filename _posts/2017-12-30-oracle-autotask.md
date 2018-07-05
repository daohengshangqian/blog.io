---
layout: post
title:  Oracle 11g 自带系统任务研究
date: 2017-12-30
categories: blog
tags: [Oracle]
description: Oracle 11g 自带系统任务研究
---

# 概述

Oracle 在创建数据库时有一个选项可选择是否开启自动任务.
默认此选项是勾选的.DBA只是习惯性的勾选这一选项,但对其内容知之甚少.
接下来的文章里,我们将讲解一下自动任务的相关知识.

# 查看自带任务

```sql
SQL> select x.client_name,x.task_name,x.operation_name ,x.STATUS,x.last_try_date
  2  from dba_autotask_task x;

CLIENT_NAME                          TASK_NAME                  OPERATION_NAME             STATUS   LAST_TRY_DATE
------------------------------------ -------------------------- ----------------------------------- -------------------------------------
sql tuning advisor                   AUTO_SQL_TUNING_PROG       automatic sql tuning task  ENABLED  14-JAN-17 03.32.03.473359 PM +08:00
auto optimizer stats collection      gather_stats_prog          auto optimizer stats job   ENABLED  14-JAN-17 08.48.57.427738 PM +08:00
auto space advisor                   auto_space_advisor_prog    auto space advisor job     ENABLED  14-JAN-17 08.48.54.419983 PM +08:00

```

# 各任务的作用

## 自动优化器统计信息收集(auto optimizer stats collection)

非常重要的自动任务,用来收集数据库对象的统计信息.

## SQL优化顾问 (sql tuning advisor)
SQL 语句优化顾问,可给出SQL语句的优化建议

## 自动空间顾问 (auto space advisor )

用来分析空间 

得到接结果类似 
```sql
   	OWNER	TASK_ID	TASK_NAME	EXECUTION_NAME	FINDING_ID	FINDING_NAME	TYPE	TYPE_ID	PARENT	OBJECT_ID	IMPACT_TYPE	IMPACT	MESSAGE	MORE_INFO	FILTERED	FLAGS
59	SYS	1136	SYS_AUTO_SPCADV_37002229122016		1		INFORMATION	4	1	1			Compress object ACAN.SPORT_ORDER_DETAIL, estimated savings is 104857600 bytes.	Allocated Space:293601280: Used Space:187935060: Reclaimable Space :104857600:	N	
```

# 自动任务的执行时间 

```sql
--查看自带任务信息
SQL> select  x.client_name,x.task_name,x.task_target_name from dba_autotask_task x;

CLIENT_NAME                           TASK_NAME                   TASK_TARGET_NAME
------------------------------------- --------------------------  -------------------------
sql tuning advisor                    AUTO_SQL_TUNING_PROG        automatic sql workload
auto optimizer stats collection       gather_stats_prog           system
auto space advisor                    auto_space_advisor_prog     system

--查看执行时间

SQL> select * from DBA_AUTOTASK_WINDOW_CLIENTS;

WINDOW_NAME                    WINDOW_NEXT_TIME                         WINDOW_ACTIVE AUTOTASK_STATUS OPTIMIZER_STATS SEGMENT_ADVISOR SQL_TUNE_ADVISOR HEALTH_MONITOR
------------------------------ ---------------------------------------- ------------- --------------- --------------- --------------- ---------------- --------------
MONDAY_WINDOW                  23-JAN-17 10.00.00.000000 PM PRC         FALSE         ENABLED         ENABLED         ENABLED         ENABLED          DISABLED
TUESDAY_WINDOW                 24-JAN-17 10.00.00.000000 PM PRC         FALSE         ENABLED         ENABLED         ENABLED         ENABLED          DISABLED
WEDNESDAY_WINDOW               25-JAN-17 10.00.00.000000 PM PRC         FALSE         ENABLED         ENABLED         ENABLED         ENABLED          DISABLED
THURSDAY_WINDOW                19-JAN-17 10.00.00.000000 PM PRC         FALSE         ENABLED         ENABLED         ENABLED         ENABLED          DISABLED
FRIDAY_WINDOW                  20-JAN-17 10.00.00.000000 PM PRC         FALSE         ENABLED         ENABLED         ENABLED         ENABLED          DISABLED
SATURDAY_WINDOW                21-JAN-17 06.00.00.000000 AM PRC         FALSE         ENABLED         ENABLED         ENABLED         ENABLED          DISABLED
SUNDAY_WINDOW                  22-JAN-17 06.00.00.000000 AM PRC         FALSE         ENABLED         ENABLED         ENABLED         ENABLED          DISABLED

7 rows selected


```

# 查看自动任务历史执行信息
```sql
SQL> select *
  2       from dba_autotask_job_history x
  3   order by 1,2      ;

CLIENT_NAME                                                      WINDOW_NAME                                                       WINDOW_START_TIME                                                                WINDOW_DURATION                         JOB_NAME                                                                         JOB_STATUS                     JOB_START_TIME                                                                   JOB_DURATION                             JOB_ERROR JOB_INFO
---------------------------------------------------------------- ----------------------------------------------------------------- -------------------------------------------------------------------------------- --------------------------------------- -------------------------------------------------------------------------------- ------------------------------ -------------------------------------------------------------------------------- --------------------------------------- ---------- --------------------------------------------------------------------------------
auto optimizer stats collection                                  MONDAY_WINDOW                                                     19-DEC-16 10.00.00.084787 PM +08:00                                              +000000000 23:41:31.184238              ORA$AT_OS_OPT_SY_741                                                             SUCCEEDED                      19-DEC-16 10.00.01.812127 PM PRC                                                 +000 00:00:39                                    0 
auto optimizer stats collection                                  SATURDAY_WINDOW                                                   31-DEC-16 11.51.55.295681 AM +08:00                                              +000000000 23:48:01.323740              ORA$AT_OS_OPT_SY_759                                                             SUCCEEDED                      31-DEC-16 11.51.57.548335 AM PRC                                                 +000 00:00:23                                    0 
auto optimizer stats collection                                  SATURDAY_WINDOW                                                   14-JAN-17 03.31.58.244746 PM +08:00                                              +000000002 14:54:41.820780              ORA$AT_OS_OPT_SY_804                                                             SUCCEEDED                      14-JAN-17 03.32.00.500306 PM PRC                                                 +000 00:00:27                                    0 
auto optimizer stats collection                                  SATURDAY_WINDOW                                                   14-JAN-17 03.31.58.244746 PM +08:00                                              +000000002 14:54:41.820780              ORA$AT_OS_OPT_SY_807                                                             SUCCEEDED                      14-JAN-17 08.48.52.889934 PM PRC                                                 +000 00:00:05                                    0 
auto optimizer stats collection                                  SUNDAY_WINDOW                                                     01-JAN-17 11.39.58.902678 AM +08:00                                              +000000010 09:45:49.547041              ORA$AT_OS_OPT_SY_762                                                             SUCCEEDED                      01-JAN-17 11.40.01.039279 AM PRC                                                 +000 00:00:19                                    0 
auto optimizer stats collection                                  SUNDAY_WINDOW                                                     01-JAN-17 11.39.58.902678 AM +08:00                                              +000000010 09:45:49.547041              ORA$AT_OS_OPT_SY_765                                                             SUCCEEDED                      01-JAN-17 03.42.47.867292 PM PRC                                                 +000 00:00:25                                    0 
auto optimizer stats collection                                  THURSDAY_WINDOW                                                   22-DEC-16 11.00.26.059413 PM +08:00                                              +000000000 20:12:13.824873              ORA$AT_OS_OPT_SY_750                                                             SUCCEEDED                      22-DEC-16 11.00.28.254568 PM PRC                                                 +000 00:00:15                                    0 
auto optimizer stats collection                                  THURSDAY_WINDOW                                                   29-DEC-16 10.00.00.053374 PM +08:00                                              +000000000 10:54:18.392389              ORA$AT_OS_OPT_SY_756                                                             SUCCEEDED                      29-DEC-16 10.00.02.223489 PM PRC                                                 +000 00:00:34                                    0 
auto optimizer stats collection                                  THURSDAY_WINDOW                                                   12-JAN-17 10.00.00.190096 PM +08:00                                              +000000000 03:59:59.836632              ORA$AT_OS_OPT_SY_801                                                             SUCCEEDED                      12-JAN-17 10.00.02.580410 PM PRC                                                 +000 00:00:25                                    0 
auto optimizer stats collection                                  TUESDAY_WINDOW                                                    17-JAN-17 10.00.00.086343 PM +08:00                                              +000000000 09:00:54.670505              ORA$AT_OS_OPT_SY_809                                                             SUCCEEDED                      17-JAN-17 10.00.02.308196 PM PRC                                                 +000 00:00:26                                    0 
auto optimizer stats collection                                  TUESDAY_WINDOW                                                    20-DEC-16 10.00.00.039334 PM +08:00                                              +000000000 22:23:47.254343              ORA$AT_OS_OPT_SY_744                                                             SUCCEEDED                      20-DEC-16 10.00.02.259633 PM PRC                                                 +000 00:00:10                                    0 
auto optimizer stats collection                                  WEDNESDAY_WINDOW                                                  18-JAN-17 10.00.00.068667 PM +08:00                                              +000000000 13:45:54.282865              ORA$AT_OS_OPT_SY_812                                                             SUCCEEDED                      18-JAN-17 10.00.00.754082 PM PRC                                                 +000 00:00:23                                    0 
auto optimizer stats collection                                  WEDNESDAY_WINDOW                                                  21-DEC-16 10.00.00.106868 PM +08:00                                              +000000001 01:00:23.610445              ORA$AT_OS_OPT_SY_747                                                             SUCCEEDED                      21-DEC-16 10.00.02.564457 PM PRC                                                 +000 00:00:27                                    0 
auto optimizer stats collection                                  WEDNESDAY_WINDOW                                                  28-DEC-16 10.25.28.570426 PM +08:00                                              +000000000 21:28:56.274297              ORA$AT_OS_OPT_SY_753                                                             SUCCEEDED                      28-DEC-16 10.25.30.821786 PM PRC                                                 +000 00:00:18                                    0 
auto optimizer stats collection                                  WEDNESDAY_WINDOW                                                  11-JAN-17 10.00.01.101799 PM +08:00                                              +000000000 07:49:01.929914              ORA$AT_OS_OPT_SY_781                                                             SUCCEEDED                      11-JAN-17 10.00.01.978848 PM PRC                                                 +000 00:00:31                                    0 
auto space advisor                                               MONDAY_WINDOW                                                     19-DEC-16 10.00.00.084787 PM +08:00                                              +000000000 23:41:31.184238              ORA$AT_SA_SPC_SY_742                                                             SUCCEEDED                      19-DEC-16 10.00.01.916163 PM PRC                                                 +000 00:00:42                                    0 
auto space advisor                                               SATURDAY_WINDOW                                                   14-JAN-17 03.31.58.244746 PM +08:00                                              +000000002 14:54:41.820780              ORA$AT_SA_SPC_SY_805                                                             SUCCEEDED                      14-JAN-17 03.32.00.390418 PM PRC                                                 +000 00:00:04                                    0 
auto space advisor                                               SATURDAY_WINDOW                                                   14-JAN-17 03.31.58.244746 PM +08:00                                              +000000002 14:54:41.820780              ORA$AT_SA_SPC_SY_808                                                             SUCCEEDED                      14-JAN-17 08.48.52.973503 PM PRC                                                 +000 00:00:01                                    0 
auto space advisor                                               SATURDAY_WINDOW                                                   31-DEC-16 11.51.55.295681 AM +08:00                                              +000000000 23:48:01.323740              ORA$AT_SA_SPC_SY_760                                                             SUCCEEDED                      31-DEC-16 11.51.57.486080 AM PRC                                                 +000 00:00:04                                    0 
auto space advisor                                               SUNDAY_WINDOW                                                     01-JAN-17 11.39.58.902678 AM +08:00                                              +000000010 09:45:49.547041              ORA$AT_SA_SPC_SY_766                                                             SUCCEEDED                      01-JAN-17 03.42.47.874855 PM PRC                                                 +000 00:00:02                                    0 
auto space advisor                                               SUNDAY_WINDOW                                                     01-JAN-17 11.39.58.902678 AM +08:00                                              +000000010 09:45:49.547041              ORA$AT_SA_SPC_SY_763                                                             SUCCEEDED                      01-JAN-17 11.40.00.878839 AM PRC                                                 +000 00:00:03                                    0 
auto space advisor                                               THURSDAY_WINDOW                                                   29-DEC-16 10.00.00.053374 PM +08:00                                              +000000000 10:54:18.392389              ORA$AT_SA_SPC_SY_757                                                             SUCCEEDED                      29-DEC-16 10.00.02.081544 PM PRC                                                 +000 00:00:42                                    0 
auto space advisor                                               THURSDAY_WINDOW                                                   12-JAN-17 10.00.00.190096 PM +08:00                                              +000000000 03:59:59.836632              ORA$AT_SA_SPC_SY_802                                                             SUCCEEDED                      12-JAN-17 10.00.02.257269 PM PRC                                                 +000 00:00:10                                    0 
auto space advisor                                               THURSDAY_WINDOW                                                   22-DEC-16 11.00.26.059413 PM +08:00                                              +000000000 20:12:13.824873              ORA$AT_SA_SPC_SY_751                                                             SUCCEEDED                      22-DEC-16 11.00.28.144717 PM PRC                                                 +000 00:00:02                                    0 
auto space advisor                                               TUESDAY_WINDOW                                                    17-JAN-17 10.00.00.086343 PM +08:00                                              +000000000 09:00:54.670505              ORA$AT_SA_SPC_SY_810                                                             SUCCEEDED                      17-JAN-17 10.00.01.849690 PM PRC                                                 +000 00:00:04                                    0 
auto space advisor                                               TUESDAY_WINDOW                                                    20-DEC-16 10.00.00.039334 PM +08:00                                              +000000000 22:23:47.254343              ORA$AT_SA_SPC_SY_745                                                             SUCCEEDED                      20-DEC-16 10.00.02.061609 PM PRC                                                 +000 00:00:02                                    0 
auto space advisor                                               WEDNESDAY_WINDOW                                                  18-JAN-17 10.00.00.068667 PM +08:00                                              +000000000 13:45:54.282865              ORA$AT_SA_SPC_SY_813                                                             SUCCEEDED                      18-JAN-17 10.00.00.547567 PM PRC                                                 +000 00:00:03                                    0 
auto space advisor                                               WEDNESDAY_WINDOW                                                  11-JAN-17 10.00.01.101799 PM +08:00                                              +000000000 07:49:01.929914              ORA$AT_SA_SPC_SY_782                                                             SUCCEEDED                      11-JAN-17 10.00.01.865932 PM PRC                                                 +000 00:00:32                                    0 
auto space advisor                                               WEDNESDAY_WINDOW                                                  21-DEC-16 10.00.00.106868 PM +08:00                                              +000000001 01:00:23.610445              ORA$AT_SA_SPC_SY_748                                                             SUCCEEDED                      21-DEC-16 10.00.02.488099 PM PRC                                                 +000 00:00:02                                    0 
auto space advisor                                               WEDNESDAY_WINDOW                                                  28-DEC-16 10.25.28.570426 PM +08:00                                              +000000000 21:28:56.274297              ORA$AT_SA_SPC_SY_754                                                             SUCCEEDED                      28-DEC-16 10.25.30.758271 PM PRC                                                 +000 00:00:02                                    0 
sql tuning advisor                                               MONDAY_WINDOW                                                     19-DEC-16 10.00.00.084787 PM +08:00                                              +000000000 23:41:31.184238              ORA$AT_SQ_SQL_SW_743                                                             SUCCEEDED                      19-DEC-16 10.00.01.706307 PM PRC                                                 +000 00:00:01                                    0 
sql tuning advisor                                               SATURDAY_WINDOW                                                   14-JAN-17 03.31.58.244746 PM +08:00                                              +000000002 14:54:41.820780              ORA$AT_SQ_SQL_SW_806                                                             SUCCEEDED                      14-JAN-17 03.32.00.390463 PM PRC                                                 +000 00:00:03                                    0 
sql tuning advisor                                               SATURDAY_WINDOW                                                   31-DEC-16 11.51.55.295681 AM +08:00                                              +000000000 23:48:01.323740              ORA$AT_SQ_SQL_SW_761                                                             SUCCEEDED                      31-DEC-16 11.51.57.491570 AM PRC                                                 +000 00:00:07                                    0 
sql tuning advisor                                               SUNDAY_WINDOW                                                     01-JAN-17 11.39.58.902678 AM +08:00                                              +000000010 09:45:49.547041              ORA$AT_SQ_SQL_SW_764                                                             SUCCEEDED                      01-JAN-17 11.40.00.872086 AM PRC                                                 +000 00:00:03                                    0 
sql tuning advisor                                               THURSDAY_WINDOW                                                   12-JAN-17 10.00.00.190096 PM +08:00                                              +000000000 03:59:59.836632              ORA$AT_SQ_SQL_SW_803                                                             SUCCEEDED                      12-JAN-17 10.00.02.248683 PM PRC                                                 +000 00:00:08                                    0 
sql tuning advisor                                               THURSDAY_WINDOW                                                   29-DEC-16 10.00.00.053374 PM +08:00                                              +000000000 10:54:18.392389              ORA$AT_SQ_SQL_SW_758                                                             SUCCEEDED                      29-DEC-16 10.00.02.086582 PM PRC                                                 +000 00:00:01                                    0 
sql tuning advisor                                               THURSDAY_WINDOW                                                   22-DEC-16 11.00.26.059413 PM +08:00                                              +000000000 20:12:13.824873              ORA$AT_SQ_SQL_SW_752                                                             SUCCEEDED                      22-DEC-16 11.00.28.144722 PM PRC                                                 +000 00:00:02                                    0 
sql tuning advisor                                               TUESDAY_WINDOW                                                    20-DEC-16 10.00.00.039334 PM +08:00                                              +000000000 22:23:47.254343              ORA$AT_SQ_SQL_SW_746                                                             SUCCEEDED                      20-DEC-16 10.00.02.098570 PM PRC                                                 +000 00:00:01                                    0 
sql tuning advisor                                               TUESDAY_WINDOW                                                    17-JAN-17 10.00.00.086343 PM +08:00                                              +000000000 09:00:54.670505              ORA$AT_SQ_SQL_SW_811                                                             SUCCEEDED                      17-JAN-17 10.00.01.996124 PM PRC                                                 +000 00:00:03                                    0 
sql tuning advisor                                               WEDNESDAY_WINDOW                                                  28-DEC-16 10.25.28.570426 PM +08:00                                              +000000000 21:28:56.274297              ORA$AT_SQ_SQL_SW_755                                                             SUCCEEDED                      28-DEC-16 10.25.30.758883 PM PRC                                                 +000 00:00:00                                    0 
sql tuning advisor                                               WEDNESDAY_WINDOW                                                  18-JAN-17 10.00.00.068667 PM +08:00                                              +000000000 13:45:54.282865              ORA$AT_SQ_SQL_SW_814                                                             SUCCEEDED                      18-JAN-17 10.00.00.562467 PM PRC                                                 +000 00:00:29                                    0 
sql tuning advisor                                               WEDNESDAY_WINDOW                                                  21-DEC-16 10.00.00.106868 PM +08:00                                              +000000001 01:00:23.610445              ORA$AT_SQ_SQL_SW_749                                                             SUCCEEDED                      21-DEC-16 10.00.02.202398 PM PRC                                                 +000 00:00:02                                    0 
sql tuning advisor                                               WEDNESDAY_WINDOW                                                  11-JAN-17 10.00.01.101799 PM +08:00                                              +000000000 07:49:01.929914              ORA$AT_SQ_SQL_SW_783                                                             SUCCEEDED                      11-JAN-17 10.00.01.869626 PM PRC                                                 +000 00:00:01                                    0 

43 rows selected

```

# 自动任务启停 


## 停止单一任务
```sql

SQL> BEGIN
  2    dbms_auto_task_admin.disable(
  3      client_name => 'sql tuning advisor',
  4      operation   => NULL,
  5      window_name => NULL);
  6  END;
  7  /

PL/SQL procedure successfully completed


SQL> select * from DBA_AUTOTASK_OPERATION ;                                                                                                                                                                                                                                       
                                                                                                                                                                                                                                                                                  
CLIENT_NAME                       OPERATION_NAME                OPERATION_TAG PRIORITY_OVERRIDE ATTRIBUTES                                                USE_RESOURCE_ESTIMATES STATUS  
--------------------------------- ----------------------------- ------------- ----------------- --------------------------------------------------------- ---------------------- --------
auto optimizer stats collection   auto optimizer stats job      OPT           INVALID           ON BY DEFAULT, VOLATILE, SAFE TO KILL                     FALSE                  ENABLED 
auto space advisor                auto space advisor job        SPC           INVALID           ON BY DEFAULT, VOLATILE, SAFE TO KILL                     FALSE                  ENABLED 
sql tuning advisor                automatic sql tuning task     SQL           INVALID           ONCE PER WINDOW, ON BY DEFAULT, VOLATILE, SAFE TO KILL    FALSE                  DISABLED
                                                                                                                                                                                                                
SQL>                                                                                                                                                                                                                                               

```



## 启动单一任务

```sql
SQL> BEGIN
  2   dbms_auto_task_admin.enable(
  3     client_name => 'sql tuning advisor',
  4     operation   => NULL,
  5     window_name => NULL);
  6  END;
  7  /

PL/SQL procedure successfully completed


SQL> select * from DBA_AUTOTASK_OPERATION ;
                                                                                                                                                            
CLIENT_NAME                          OPERATION_NAME                  OPERATION_TAG PRIORITY_OVERRIDE ATTRIBUTES                                              USE_RESOURCE_ESTIMATES STATUS
------------------------------------ ------------------------------- ------------- ----------------- ------------------------------------------------------  ---------------------- --------
auto optimizer stats collection      auto optimizer stats job        OPT           INVALID           ON BY DEFAULT, VOLATILE, SAFE TO KILL                   FALSE                  ENABLED
auto space advisor                   auto space advisor job          SPC           INVALID           ON BY DEFAULT, VOLATILE, SAFE TO KILL                   FALSE                  ENABLED
sql tuning advisor                   automatic sql tuning task       SQL           INVALID           ONCE PER WINDOW, ON BY DEFAULT, VOLATILE, SAFE TO KILL  FALSE                  ENABLED

```

## 按时停止单一任务

```sql

--停止周一的STA


SQL> BEGIN
  2    dbms_auto_task_admin.disable(
  3      client_name => 'sql tuning advisor',
  4      operation   => NULL,
  5      window_name => 'MONDAY_WINDOW');
  6  END;
  7  /

PL/SQL procedure successfully completed



SQL> select *
  2    from dba_scheduler_wingroup_members    ;

WINDOW_GROUP_NAME              WINDOW_NAME
------------------------------ ----------------
MAINTENANCE_WINDOW_GROUP       MONDAY_WINDOW
MAINTENANCE_WINDOW_GROUP       TUESDAY_WINDOW
MAINTENANCE_WINDOW_GROUP       WEDNESDAY_WINDOW
MAINTENANCE_WINDOW_GROUP       THURSDAY_WINDOW
MAINTENANCE_WINDOW_GROUP       FRIDAY_WINDOW
MAINTENANCE_WINDOW_GROUP       SATURDAY_WINDOW
MAINTENANCE_WINDOW_GROUP       SUNDAY_WINDOW
ORA$AT_WGRP_OS                 MONDAY_WINDOW
ORA$AT_WGRP_OS                 TUESDAY_WINDOW
ORA$AT_WGRP_OS                 WEDNESDAY_WINDOW
ORA$AT_WGRP_OS                 THURSDAY_WINDOW
ORA$AT_WGRP_OS                 FRIDAY_WINDOW
ORA$AT_WGRP_OS                 SATURDAY_WINDOW
ORA$AT_WGRP_OS                 SUNDAY_WINDOW
ORA$AT_WGRP_SA                 MONDAY_WINDOW
ORA$AT_WGRP_SA                 TUESDAY_WINDOW
ORA$AT_WGRP_SA                 WEDNESDAY_WINDOW
ORA$AT_WGRP_SA                 THURSDAY_WINDOW
ORA$AT_WGRP_SA                 FRIDAY_WINDOW
ORA$AT_WGRP_SA                 SATURDAY_WINDOW
ORA$AT_WGRP_SA                 SUNDAY_WINDOW
ORA$AT_WGRP_SQ                 TUESDAY_WINDOW
ORA$AT_WGRP_SQ                 WEDNESDAY_WINDOW
ORA$AT_WGRP_SQ                 THURSDAY_WINDOW
ORA$AT_WGRP_SQ                 FRIDAY_WINDOW
ORA$AT_WGRP_SQ                 SATURDAY_WINDOW
ORA$AT_WGRP_SQ                 SUNDAY_WINDOW

27 rows selected
```
## 按时开启单一任务

```sql

SQL> BEGIN
  2    dbms_auto_task_admin.enable(
  3      client_name => 'sql tuning advisor',
  4      operation   => NULL,
  5      window_name => 'MONDAY_WINDOW');
  6  END;
  7  /

PL/SQL procedure successfully completed

SQL> 
SQL> select *
  2     from dba_scheduler_wingroup_members    ;

WINDOW_GROUP_NAME              WINDOW_NAME
------------------------------ ------------------------------
MAINTENANCE_WINDOW_GROUP       MONDAY_WINDOW
MAINTENANCE_WINDOW_GROUP       TUESDAY_WINDOW
MAINTENANCE_WINDOW_GROUP       WEDNESDAY_WINDOW
MAINTENANCE_WINDOW_GROUP       THURSDAY_WINDOW
MAINTENANCE_WINDOW_GROUP       FRIDAY_WINDOW
MAINTENANCE_WINDOW_GROUP       SATURDAY_WINDOW
MAINTENANCE_WINDOW_GROUP       SUNDAY_WINDOW
ORA$AT_WGRP_OS                 MONDAY_WINDOW
ORA$AT_WGRP_OS                 TUESDAY_WINDOW
ORA$AT_WGRP_OS                 WEDNESDAY_WINDOW
ORA$AT_WGRP_OS                 THURSDAY_WINDOW
ORA$AT_WGRP_OS                 FRIDAY_WINDOW
ORA$AT_WGRP_OS                 SATURDAY_WINDOW
ORA$AT_WGRP_OS                 SUNDAY_WINDOW
ORA$AT_WGRP_SA                 MONDAY_WINDOW
ORA$AT_WGRP_SA                 TUESDAY_WINDOW
ORA$AT_WGRP_SA                 WEDNESDAY_WINDOW
ORA$AT_WGRP_SA                 THURSDAY_WINDOW
ORA$AT_WGRP_SA                 FRIDAY_WINDOW
ORA$AT_WGRP_SA                 SATURDAY_WINDOW
ORA$AT_WGRP_SA                 SUNDAY_WINDOW
ORA$AT_WGRP_SQ                 MONDAY_WINDOW
ORA$AT_WGRP_SQ                 TUESDAY_WINDOW
ORA$AT_WGRP_SQ                 WEDNESDAY_WINDOW
ORA$AT_WGRP_SQ                 THURSDAY_WINDOW
ORA$AT_WGRP_SQ                 FRIDAY_WINDOW
ORA$AT_WGRP_SQ                 SATURDAY_WINDOW
ORA$AT_WGRP_SQ                 SUNDAY_WINDOW

28 rows selected

```

## 停止所有任务
```sql

SQL> EXECUTE DBMS_AUTO_TASK_ADMIN.DISABLE;

PL/SQL procedure successfully completed

SQL> select * from DBA_AUTOTASK_WINDOW_CLIENTS;'

WINDOW_NAME       WINDOW_NEXT_TIME                      WINDOW_ACTIVE AUTOTASK_STATUS OPTIMIZER_STATS SEGMENT_ADVISOR SQL_TUNE_ADVISOR HEALTH_MONITOR
----------------- --------------------------------------------------- --------------- --------------- --------------- ---------------- --------------
MONDAY_WINDOW     23-JAN-17 10.00.00.000000 PM PRC      FALSE         DISABLED        ENABLED         ENABLED         ENABLED          DISABLED
TUESDAY_WINDOW    24-JAN-17 10.00.00.000000 PM PRC      FALSE         DISABLED        ENABLED         ENABLED         ENABLED          DISABLED
WEDNESDAY_WINDOW  25-JAN-17 10.00.00.000000 PM PRC      FALSE         DISABLED        ENABLED         ENABLED         ENABLED          DISABLED
THURSDAY_WINDOW   19-JAN-17 10.00.00.000000 PM PRC      FALSE         DISABLED        ENABLED         ENABLED         ENABLED          DISABLED
FRIDAY_WINDOW     20-JAN-17 10.00.00.000000 PM PRC      FALSE         DISABLED        ENABLED         ENABLED         ENABLED          DISABLED
SATURDAY_WINDOW   21-JAN-17 06.00.00.000000 AM PRC      FALSE         DISABLED        ENABLED         ENABLED         ENABLED          DISABLED
SUNDAY_WINDOW     22-JAN-17 06.00.00.000000 AM PRC      FALSE         DISABLED        ENABLED         ENABLED         ENABLED          DISABLED

7 rows selected

```

## 启动所有任务
```sql

SQL> EXECUTE DBMS_AUTO_TASK_ADMIN.ENABLE ;

PL/SQL procedure successfully completed

SQL>  select * from DBA_AUTOTASK_WINDOW_CLIENTS;

WINDOW_NAME        WINDOW_NEXT_TIME                    WINDOW_ACTIVE AUTOTASK_STATUS OPTIMIZER_STATS SEGMENT_ADVISOR SQL_TUNE_ADVISOR HEALTH_MONITOR
------------------ ------------------------------------------------- --------------- --------------- --------------- ---------------- --------------
MONDAY_WINDOW      23-JAN-17 10.00.00.000000 PM PRC    FALSE         ENABLED         ENABLED         ENABLED         ENABLED          DISABLED
TUESDAY_WINDOW     24-JAN-17 10.00.00.000000 PM PRC    FALSE         ENABLED         ENABLED         ENABLED         ENABLED          DISABLED
WEDNESDAY_WINDOW   25-JAN-17 10.00.00.000000 PM PRC    FALSE         ENABLED         ENABLED         ENABLED         ENABLED          DISABLED
THURSDAY_WINDOW    19-JAN-17 10.00.00.000000 PM PRC    FALSE         ENABLED         ENABLED         ENABLED         ENABLED          DISABLED
FRIDAY_WINDOW      20-JAN-17 10.00.00.000000 PM PRC    FALSE         ENABLED         ENABLED         ENABLED         ENABLED          DISABLED
SATURDAY_WINDOW    21-JAN-17 06.00.00.000000 AM PRC    FALSE         ENABLED         ENABLED         ENABLED         ENABLED          DISABLED
SUNDAY_WINDOW      22-JAN-17 06.00.00.000000 AM PRC    FALSE         ENABLED         ENABLED         ENABLED         ENABLED          DISABLED

7 rows selected

```

# 建议关闭的任务 
虽然Oracle 自动任务含有sql tuning advisor  与 auto space advisor .
但是大部分DBA都很少查看.所以建议关闭这两个自动任务.


## 查看自动任务结果

```sql
SQL> select * from  DBA_ADVISOR_FINDINGS  where   task_name like '%AUTO%'order by 3;

OWNER    TASK_ID TASK_NAME                      EXECUTION_NAME   FINDING_ID FINDING_NAME                     TYPE           TYPE_ID     PARENT  OBJECT_ID IMPACT_TYPE                             IMPACT MESSAGE                                                      MORE_INFO                                                                   FILTERED      FLAGS
---------------- ------------------------------ ---------------- ---------- ------------------------------- ----------- ---------- ---------- ---------- -------------------------- -------------------------------------------------------------------------------- -------------------------------------------------------------------------------- -------- ----------
SYS         1151 SYS_AUTO_SPCADV_03002211012017                          25                                 INFORMATION          4         25        137                            Perform shrink, estimated savings is 11104796 bytes.                             Allocated Space:41943040: Used Space:30838244: Reclaimable Space :11104796:      N        
SYS         1151 SYS_AUTO_SPCADV_03002211012017                          27                                 INFORMATION          4         27        144                            Perform shrink, estimated savings is 10953961 bytes.                             Allocated Space:25165824: Used Space:14211863: Reclaimable Space :10953961:      N        
SYS         1151 SYS_AUTO_SPCADV_03002211012017                          23                                 INFORMATION          4         23        134                            Perform shrink, estimated savings is 45055423 bytes.                             Allocated Space:92274688: Used Space:47219265: Reclaimable Space :45055423:      N        
SYS         1151 SYS_AUTO_SPCADV_03002211012017                          22                                 INFORMATION          4         22        108                            Perform shrink, estimated savings is 15408953 bytes.                             Allocated Space:44040192: Used Space:28631239: Reclaimable Space :15408953:      N        
SYS         1151 SYS_AUTO_SPCADV_03002211012017                          21                                 INFORMATION          4         21         92                            Perform shrink, estimated savings is 100104473 bytes.                            Allocated Space:243269632: Used Space:143165159: Reclaimable Space :100104473:   N        
SYS         1151 SYS_AUTO_SPCADV_03002211012017                          20                                 INFORMATION          4         20         91                            Perform shrink, estimated savings is 37623316 bytes.                             Allocated Space:352321536: Used Space:314698220: Reclaimable Space :37623316:    N        
SYS         1151 SYS_AUTO_SPCADV_03002211012017                          19                                 INFORMATION          4         19         90                            Perform shrink, estimated savings is 216761376 bytes.                            Allocated Space:352321536: Used Space:135560160: Reclaimable Space :216761376:   N        
SYS         1151 SYS_AUTO_SPCADV_03002211012017                          18                                 INFORMATION          4         18         87                            Perform shrink, estimated savings is 38238662 bytes.                             Allocated Space:159383552: Used Space:121144890: Reclaimable Space :38238662:    N        
SYS         1151 SYS_AUTO_SPCADV_03002211012017                          17                                 INFORMATION          4         17         86                            Perform shrink, estimated savings is 43049945 bytes.                             Allocated Space:150994944: Used Space:107944999: Reclaimable Space :43049945:    N        
SYS         1151 SYS_AUTO_SPCADV_03002211012017                          16                                 INFORMATION          4         16         85                            Perform shrink, estimated savings is 58420262 bytes.                             Allocated Space:150994944: Used Space:92574682: Reclaimable Space :58420262:     N        
SYS         1151 SYS_AUTO_SPCADV_03002211012017                          15                                 INFORMATION          4         15         84                            Perform shrink, estimated savings is 34471318 bytes.                             Allocated Space:134217728: Used Space:99746410: Reclaimable Space :34471318:     N        
SYS         1151 SYS_AUTO_SPCADV_03002211012017                          14                                 INFORMATION          4         14         83                            Perform shrink, estimated savings is 108959867 bytes.                            Allocated Space:276824064: Used Space:167864197: Reclaimable Space :108959867:   N        
SYS         1151 SYS_AUTO_SPCADV_03002211012017                          13                                 INFORMATION          4         13         82                            Perform shrink, estimated savings is 64976135 bytes.                             Allocated Space:243269632: Used Space:178293497: Reclaimable Space :64976135:    N        
SYS         1151 SYS_AUTO_SPCADV_03002211012017                          12                                 INFORMATION          4         12         81                            Perform shrink, estimated savings is 67751547 bytes.                             Allocated Space:192937984: Used Space:125186437: Reclaimable Space :67751547:    N        
SYS         1151 SYS_AUTO_SPCADV_03002211012017                          11                                 INFORMATION          4         11         80                            Perform shrink, estimated savings is 56670461 bytes.                             Allocated Space:163577856: Used Space:106907395: Reclaimable Space :56670461:    N        
SYS         1151 SYS_AUTO_SPCADV_03002211012017                          10                                 INFORMATION          4         10         78                            Perform shrink, estimated savings is 129617460 bytes.                            Allocated Space:293601280: Used Space:163983820: Reclaimable Space :129617460:   N        
SYS         1151 SYS_AUTO_SPCADV_03002211012017                           9                                 INFORMATION          4          9         77                            Perform shrink, estimated savings is 50031654 bytes.                             Allocated Space:142606336: Used Space:92574682: Reclaimable Space :50031654:     N        
SYS         1151 SYS_AUTO_SPCADV_03002211012017                           8                                 INFORMATION          4          8         76                            Perform shrink, estimated savings is 42843237 bytes.                             Allocated Space:142606336: Used Space:99763099: Reclaimable Space :42843237:     N        
SYS         1151 SYS_AUTO_SPCADV_03002211012017                           7                                 INFORMATION          4          7         75                            Perform shrink, estimated savings is 60409477 bytes.                             Allocated Space:159383552: Used Space:98974075: Reclaimable Space :60409477:     N        
SYS         1151 SYS_AUTO_SPCADV_03002211012017                           6                                 INFORMATION          4          6         74                            Perform shrink, estimated savings is 33233494 bytes.                             Allocated Space:226492416: Used Space:193258922: Reclaimable Space :33233494:    N        
SYS         1151 SYS_AUTO_SPCADV_03002211012017                           5                                 INFORMATION          4          5         73                            Perform shrink, estimated savings is 324938624 bytes.                            Allocated Space:713031680: Used Space:388093056: Reclaimable Space :324938624:   N        
SYS         1151 SYS_AUTO_SPCADV_03002211012017                           2                                 INFORMATION          4          2         70                            Perform shrink, estimated savings is 837591531 bytes.                            Allocated Space:1146093568: Used Space:308502037: Reclaimable Space :837591531:  N        
SYS         1151 SYS_AUTO_SPCADV_03002211012017                           1                                 INFORMATION          4          1          8                            Enable row movement of the table DAO.TEST and perform shrink, estimated savings  Allocated Space:8848932864: Used Space:8797402036: Reclaimable Space :51530828:  N        
SYS         1151 SYS_AUTO_SPCADV_03002211012017                           4                                 INFORMATION          4          4         72                            Perform shrink, estimated savings is 161843926 bytes.                            Allocated Space:318767104: Used Space:156923178: Reclaimable Space :161843926:   N        
SYS         1151 SYS_AUTO_SPCADV_03002211012017                           3                                 INFORMATION          4          3         71                            Perform shrink, estimated savings is 269888074 bytes.                            Allocated Space:578813952: Used Space:308925878: Reclaimable Space :269888074:   N        
SYS         1151 SYS_AUTO_SPCADV_03002211012017                          26                                 INFORMATION          4         26        140                            Perform shrink, estimated savings is 12292764 bytes.                             Allocated Space:41943040: Used Space:29650276: Reclaimable Space :12292764:      N        
SYS         1151 SYS_AUTO_SPCADV_03002211012017                          24                                 INFORMATION          4         24        135                            Perform shrink, estimated savings is 16815868 bytes.                             Allocated Space:65011712: Used Space:48195844: Reclaimable Space :16815868:      N        
SYS         1135 SYS_AUTO_SPCADV_04002229122016                          27                                 INFORMATION          4         27        144                            Perform shrink, estimated savings is 10953961 bytes.                             Allocated Space:25165824: Used Space:14211863: Reclaimable Space :10953961:      N        
SYS         1135 SYS_AUTO_SPCADV_04002229122016                          26                                 INFORMATION          4         26        140                            Perform shrink, estimated savings is 12292764 bytes.                             Allocated Space:41943040: Used Space:29650276: Reclaimable Space :12292764:      N        
SYS         1135 SYS_AUTO_SPCADV_04002229122016                          25                                 INFORMATION          4         25        137                            Perform shrink, estimated savings is 11104796 bytes.                             Allocated Space:41943040: Used Space:30838244: Reclaimable Space :11104796:      N        
SYS         1135 SYS_AUTO_SPCADV_04002229122016                          24                                 INFORMATION          4         24        135                            Perform shrink, estimated savings is 16815868 bytes.                             Allocated Space:65011712: Used Space:48195844: Reclaimable Space :16815868:      N        
SYS         1135 SYS_AUTO_SPCADV_04002229122016                          23                                 INFORMATION          4         23        134                            Perform shrink, estimated savings is 45055423 bytes.                             Allocated Space:92274688: Used Space:47219265: Reclaimable Space :45055423:      N        
SYS         1135 SYS_AUTO_SPCADV_04002229122016                          22                                 INFORMATION          4         22        108                            Perform shrink, estimated savings is 15408953 bytes.                             Allocated Space:44040192: Used Space:28631239: Reclaimable Space :15408953:      N        
SYS         1135 SYS_AUTO_SPCADV_04002229122016                          21                                 INFORMATION          4         21         92                            Perform shrink, estimated savings is 100104473 bytes.                            Allocated Space:243269632: Used Space:143165159: Reclaimable Space :100104473:   N        
SYS         1135 SYS_AUTO_SPCADV_04002229122016                          20                                 INFORMATION          4         20         91                            Perform shrink, estimated savings is 37623316 bytes.                             Allocated Space:352321536: Used Space:314698220: Reclaimable Space :37623316:    N        
SYS         1135 SYS_AUTO_SPCADV_04002229122016                          19                                 INFORMATION          4         19         90                            Perform shrink, estimated savings is 216761376 bytes.                            Allocated Space:352321536: Used Space:135560160: Reclaimable Space :216761376:   N        
SYS         1135 SYS_AUTO_SPCADV_04002229122016                          18                                 INFORMATION          4         18         87                            Perform shrink, estimated savings is 38238662 bytes.                             Allocated Space:159383552: Used Space:121144890: Reclaimable Space :38238662:    N        
SYS         1135 SYS_AUTO_SPCADV_04002229122016                          17                                 INFORMATION          4         17         86                            Perform shrink, estimated savings is 43049945 bytes.                             Allocated Space:150994944: Used Space:107944999: Reclaimable Space :43049945:    N        
SYS         1135 SYS_AUTO_SPCADV_04002229122016                          16                                 INFORMATION          4         16         85                            Perform shrink, estimated savings is 58420262 bytes.                             Allocated Space:150994944: Used Space:92574682: Reclaimable Space :58420262:     N        
SYS         1135 SYS_AUTO_SPCADV_04002229122016                          15                                 INFORMATION          4         15         84                            Perform shrink, estimated savings is 34471318 bytes.                             Allocated Space:134217728: Used Space:99746410: Reclaimable Space :34471318:     N        
SYS         1135 SYS_AUTO_SPCADV_04002229122016                          14                                 INFORMATION          4         14         83                            Perform shrink, estimated savings is 108959867 bytes.                            Allocated Space:276824064: Used Space:167864197: Reclaimable Space :108959867:   N        
SYS         1135 SYS_AUTO_SPCADV_04002229122016                          13                                 INFORMATION          4         13         82                            Perform shrink, estimated savings is 64976135 bytes.                             Allocated Space:243269632: Used Space:178293497: Reclaimable Space :64976135:    N        
SYS         1135 SYS_AUTO_SPCADV_04002229122016                          12                                 INFORMATION          4         12         81                            Perform shrink, estimated savings is 67751547 bytes.                             Allocated Space:192937984: Used Space:125186437: Reclaimable Space :67751547:    N        
SYS         1135 SYS_AUTO_SPCADV_04002229122016                          11                                 INFORMATION          4         11         80                            Perform shrink, estimated savings is 56670461 bytes.                             Allocated Space:163577856: Used Space:106907395: Reclaimable Space :56670461:    N        
SYS         1135 SYS_AUTO_SPCADV_04002229122016                          10                                 INFORMATION          4         10         78                            Perform shrink, estimated savings is 129617460 bytes.                            Allocated Space:293601280: Used Space:163983820: Reclaimable Space :129617460:   N        
SYS         1135 SYS_AUTO_SPCADV_04002229122016                           9                                 INFORMATION          4          9         77                            Perform shrink, estimated savings is 50031654 bytes.                             Allocated Space:142606336: Used Space:92574682: Reclaimable Space :50031654:     N        
SYS         1135 SYS_AUTO_SPCADV_04002229122016                           8                                 INFORMATION          4          8         76                            Perform shrink, estimated savings is 42843237 bytes.                             Allocated Space:142606336: Used Space:99763099: Reclaimable Space :42843237:     N        
SYS         1135 SYS_AUTO_SPCADV_04002229122016                           7                                 INFORMATION          4          7         75                            Perform shrink, estimated savings is 60409477 bytes.                             Allocated Space:159383552: Used Space:98974075: Reclaimable Space :60409477:     N        
SYS         1135 SYS_AUTO_SPCADV_04002229122016                           6                                 INFORMATION          4          6         74                            Perform shrink, estimated savings is 33233494 bytes.                             Allocated Space:226492416: Used Space:193258922: Reclaimable Space :33233494:    N        
SYS         1135 SYS_AUTO_SPCADV_04002229122016                           5                                 INFORMATION          4          5         73                            Perform shrink, estimated savings is 324938624 bytes.                            Allocated Space:713031680: Used Space:388093056: Reclaimable Space :324938624:   N        
SYS         1135 SYS_AUTO_SPCADV_04002229122016                           4                                 INFORMATION          4          4         72                            Perform shrink, estimated savings is 161843926 bytes.                            Allocated Space:318767104: Used Space:156923178: Reclaimable Space :161843926:   N        
SYS         1135 SYS_AUTO_SPCADV_04002229122016                           3                                 INFORMATION          4          3         71                            Perform shrink, estimated savings is 269888074 bytes.                            Allocated Space:578813952: Used Space:308925878: Reclaimable Space :269888074:   N        
SYS         1135 SYS_AUTO_SPCADV_04002229122016                           1                                 INFORMATION          4          1          8                            Enable row movement of the table DAO.TEST and perform shrink, estimated savings  Allocated Space:8848932864: Used Space:8797402036: Reclaimable Space :51530828:  N        
SYS         1135 SYS_AUTO_SPCADV_04002229122016                           2                                 INFORMATION          4          2         70                            Perform shrink, estimated savings is 837591531 bytes.                            Allocated Space:1146093568: Used Space:308502037: Reclaimable Space :837591531:  N        
SYS         1165 SYS_AUTO_SPCADV_05002212012017                           1                                 INFORMATION          4          1          3                            Enable row movement of the table DAO.TEST and perform shrink, estimated savings  Allocated Space:8848932864: Used Space:8797402036: Reclaimable Space :51530828:  N        
SYS         1152 SYS_AUTO_SPCADV_27002211012017                           5                                 INFORMATION          4          5          5                            Compress object TIAO.E_ORDER, estimated savings is 10485760 bytes.               Allocated Space:27262976: Used Space:16010990: Reclaimable Space :10485760:      N        
SYS         1152 SYS_AUTO_SPCADV_27002211012017                           1                                 INFORMATION          4          1          1                            Compress object ACAN.SPORT_ORDER_DETAIL, estimated savings is 104857600 bytes.   Allocated Space:293601280: Used Space:188358665: Reclaimable Space :104857600:   N        
SYS         1152 SYS_AUTO_SPCADV_27002211012017                           2                                 INFORMATION          4          2          2                            Compress object ACAN.SPORT_ORDER, estimated savings is 51380224 bytes.           Allocated Space:209715200: Used Space:158132388: Reclaimable Space :51380224:    N        
SYS         1152 SYS_AUTO_SPCADV_27002211012017                           3                                 INFORMATION          4          3          3                            Compress object ACAN.SPORT_ACCOUNT, estimated savings is 13631488 bytes.         Allocated Space:33554432: Used Space:19229920: Reclaimable Space :13631488:      N        
SYS         1152 SYS_AUTO_SPCADV_27002211012017                           4                                 INFORMATION          4          4          4                            Compress object DAO.E_ORDER, estimated savings is 10485760 bytes.                Allocated Space:27262976: Used Space:16089091: Reclaimable Space :10485760:      N        
SYS         1136 SYS_AUTO_SPCADV_37002229122016                           1                                 INFORMATION          4          1          1                            Compress object ACAN.SPORT_ORDER_DETAIL, estimated savings is 104857600 bytes.   Allocated Space:293601280: Used Space:187935060: Reclaimable Space :104857600:   N        
SYS         1136 SYS_AUTO_SPCADV_37002229122016                           2                                 INFORMATION          4          2          2                            Compress object ACAN.SPORT_ORDER, estimated savings is 54525952 bytes.           Allocated Space:209715200: Used Space:154347149: Reclaimable Space :54525952:    N        
SYS         1136 SYS_AUTO_SPCADV_37002229122016                           5                                 INFORMATION          4          5          5                            Compress object TIAO.E_ORDER, estimated savings is 10485760 bytes.               Allocated Space:27262976: Used Space:15888477: Reclaimable Space :10485760:      N        
SYS         1136 SYS_AUTO_SPCADV_37002229122016                           4                                 INFORMATION          4          4          4                            Compress object DAO.E_ORDER, estimated savings is 10485760 bytes.                Allocated Space:27262976: Used Space:15950270: Reclaimable Space :10485760:      N        
SYS         1136 SYS_AUTO_SPCADV_37002229122016                           3                                 INFORMATION          4          3          3                            Compress object ACAN.SPORT_ACCOUNT, estimated savings is 13631488 bytes.         Allocated Space:33554432: Used Space:19483882: Reclaimable Space :13631488:      N        
SYS            1 SYS_AUTO_SQL_TUNING_TASK       EXEC_1217               793 normal, successful completion   INFORMATION          4          0        869                          0 This statement was skipped because it has already been tuned recently.  See task                                                                                  N                 1
SYS            1 SYS_AUTO_SQL_TUNING_TASK       EXEC_1217               794 normal, successful completion   INFORMATION          4          0        870                          0 This statement was skipped because it has already been tuned recently.  See task                                                                                  N                 1
SYS            1 SYS_AUTO_SQL_TUNING_TASK       EXEC_1220               795 normal, successful completion   INFORMATION          4          0        872                          0 This statement was skipped because it has already been tuned recently.  See task                                                                                  N                 1
SYS            1 SYS_AUTO_SQL_TUNING_TASK       EXEC_1220               796 normal, successful completion   INFORMATION          4          0        873                          0 This statement was skipped because it has already been tuned recently.  See task                                                                                  N                 1
SYS            1 SYS_AUTO_SQL_TUNING_TASK       EXEC_1282               815 normal, successful completion   INFORMATION          4          0        901                          0 This statement was skipped because it has already been tuned recently.  See task                                                                                  N                 1
SYS            1 SYS_AUTO_SQL_TUNING_TASK       EXEC_1282               816 normal, successful completion   INFORMATION          4          0        902                          0 This statement was skipped because it has already been tuned recently.  See task                                                                                  N                 1
SYS            1 SYS_AUTO_SQL_TUNING_TASK       EXEC_1282               817 normal, successful completion   INFORMATION          4          0        903                          0 This statement was skipped because it has already been tuned recently.  See task                                                                                  N                 1
SYS            1 SYS_AUTO_SQL_TUNING_TASK       EXEC_1282               818 normal, successful completion   INFORMATION          4          0        904                          0 This statement was skipped because it has already been tuned recently.  See task                                                                                  N                 1
SYS            1 SYS_AUTO_SQL_TUNING_TASK       EXEC_1282               819 normal, successful completion   INFORMATION          4          0        905                          0 This statement was skipped because it has already been tuned recently.  See task                                                                                  N                 1
SYS            1 SYS_AUTO_SQL_TUNING_TASK       EXEC_1291               820 normal, successful completion   INFORMATION          4          0        907                          0 This statement was skipped because it has already been tuned recently.  See task                                                                                  N                 1
SYS            1 SYS_AUTO_SQL_TUNING_TASK       EXEC_1291               821 normal, successful completion   INFORMATION          4          0        908                          0 This statement was skipped because it has already been tuned recently.  See task                                                                                  N                 1
SYS            1 SYS_AUTO_SQL_TUNING_TASK       EXEC_1291               822 normal, successful completion   INFORMATION          4          0        909                          0 This statement was skipped because it has already been tuned recently.  See task                                                                                  N                 1
SYS            1 SYS_AUTO_SQL_TUNING_TASK       EXEC_1291               823 normal, successful completion   INFORMATION          4          0        910                          0 This statement was skipped because it has already been tuned recently.  See task                                                                                  N                 1
SYS            1 SYS_AUTO_SQL_TUNING_TASK       EXEC_1291               824 normal, successful completion   INFORMATION          4          0        911                          0 This statement was skipped because it has already been tuned recently.  See task                                                                                  N                 1
SYS            1 SYS_AUTO_SQL_TUNING_TASK       EXEC_1297               825 normal, successful completion   INFORMATION          4          0        913                          0 This statement was skipped because it has already been tuned recently.  See task                                                                                  N                 1
SYS            1 SYS_AUTO_SQL_TUNING_TASK       EXEC_1297               826 normal, successful completion   INFORMATION          4          0        914                          0 This statement was skipped because it has already been tuned recently.  See task                                                                                  N                 1
SYS            1 SYS_AUTO_SQL_TUNING_TASK       EXEC_1297               827 normal, successful completion   PROBLEM              1          0        915                          0 A potentially better execution plan was found for this statement.                                                                                                 N                 0
SYS            1 SYS_AUTO_SQL_TUNING_TASK       EXEC_1297               828 normal, successful completion   PROBLEM              1          0        916                          0 A potentially better execution plan was found for this statement.                                                                                                 N                 0
SYS            1 SYS_AUTO_SQL_TUNING_TASK       EXEC_1297               829 normal, successful completion   INFORMATION          4          0        917                          0 This statement was skipped because it has already been tuned recently.  See task                                                                                  N                 1
SYS            1 SYS_AUTO_SQL_TUNING_TASK       EXEC_1297               830 normal, successful completion   PROBLEM              1          0        918                          0 A potentially better execution plan was found for this statement.                                                                                                 N                 0
SYS            1 SYS_AUTO_SQL_TUNING_TASK       EXEC_1297               831 normal, successful completion   INFORMATION          4          0        918                          0 The optimizer could not merge the view at line ID 46 of the execution plan.      The optimizer cannot merge a view that contains a "START WITH" clause.           N                 0
SYS            1 SYS_AUTO_SQL_TUNING_TASK       EXEC_1297               832 normal, successful completion   PROBLEM              1          0        918                          0 An expensive "UNION" operation was found at line ID 3 of the execution plan.                                                                                      N                 0
SYS            1 SYS_AUTO_SQL_TUNING_TASK       EXEC_1297               833 normal, successful completion   INFORMATION          4          0        918                          0 The optimizer could not merge the view at line ID 2 of the execution plan.       The optimizer cannot merge a view that contains a set operator.                  N                 0
SYS            1 SYS_AUTO_SQL_TUNING_TASK       EXEC_1297               834 normal, successful completion   PROBLEM              1          0        918                          0 Some alternative execution plans for this statement were found by searching the                                                                                   N                 0
SYS            1 SYS_AUTO_SQL_TUNING_TASK       EXEC_1297               835 normal, successful completion   PROBLEM              1          0        919                          0 A potentially better execution plan was found for this statement.                                                                                                 N                 0
SYS            1 SYS_AUTO_SQL_TUNING_TASK       EXEC_1297               836 normal, successful completion   PROBLEM              1          0        919                          0 Some alternative execution plans for this statement were found by searching the                                                                                   N                 0
SYS            1 SYS_AUTO_SQL_TUNING_TASK       EXEC_1297               837 normal, successful completion   INFORMATION          4          0        920                          0 This statement was skipped because it has already been tuned recently.  See task                                                                                  N                 1
SYS            1 SYS_AUTO_SQL_TUNING_TASK       EXEC_1212               791 normal, successful completion   INFORMATION          4          0        866                          0 This statement was skipped because it has already been tuned recently.  See task                                                                                  N                 1
SYS            1 SYS_AUTO_SQL_TUNING_TASK       EXEC_1212               792 normal, successful completion   INFORMATION          4          0        867                          0 This statement was skipped because it has already been tuned recently.  See task                                                                                  N                 1
SYS            1 SYS_AUTO_SQL_TUNING_TASK       EXEC_1230               799 normal, successful completion   INFORMATION          4          0        878                          0 This statement was skipped because it has already been tuned recently.  See task                                                                                  N                 1
SYS            1 SYS_AUTO_SQL_TUNING_TASK       EXEC_1230               800 normal, successful completion   INFORMATION          4          0        879                          0 This statement was skipped because it has already been tuned recently.  See task                                                                                  N                 1
SYS            1 SYS_AUTO_SQL_TUNING_TASK       EXEC_1230               801 normal, successful completion   INFORMATION          4          0        880                          0 This statement was skipped because it has already been tuned recently.  See task                                                                                  N                 1
SYS            1 SYS_AUTO_SQL_TUNING_TASK       EXEC_1225               797 normal, successful completion   INFORMATION          4          0        875                          0 This statement was skipped because it has already been tuned recently.  See task                                                                                  N                 1
SYS            1 SYS_AUTO_SQL_TUNING_TASK       EXEC_1225               798 normal, successful completion   INFORMATION          4          0        876                          0 This statement was skipped because it has already been tuned recently.  See task                                                                                  N                 1
SYS            1 SYS_AUTO_SQL_TUNING_TASK       EXEC_1237               802 normal, successful completion   INFORMATION          4          0        882                          0 This statement was skipped because it has already been tuned recently.  See task                                                                                  N                 1
SYS            1 SYS_AUTO_SQL_TUNING_TASK       EXEC_1237               803 normal, successful completion   INFORMATION          4          0        883                          0 This statement was skipped because it has already been tuned recently.  See task                                                                                  N                 1
SYS            1 SYS_AUTO_SQL_TUNING_TASK       EXEC_1237               804 normal, successful completion   INFORMATION          4          0        884                          0 This statement was skipped because it has already been tuned recently.  See task                                                                                  N                 1
SYS            1 SYS_AUTO_SQL_TUNING_TASK       EXEC_1237               805 normal, successful completion   PROBLEM              1          0        885                          0 A potentially better execution plan was found for this statement.                                                                                                 N                 0
SYS            1 SYS_AUTO_SQL_TUNING_TASK       EXEC_1240               806 normal, successful completion   INFORMATION          4          0        888                          0 This statement was skipped because it has already been tuned recently.  See task                                                                                  N                 1
SYS            1 SYS_AUTO_SQL_TUNING_TASK       EXEC_1240               807 normal, successful completion   INFORMATION          4          0        890                          0 This statement was skipped because it has already been tuned recently.  See task                                                                                  N                 1
SYS            1 SYS_AUTO_SQL_TUNING_TASK       EXEC_1240               808 normal, successful completion   INFORMATION          4          0        891                          0 This statement was skipped because it has already been tuned recently.  See task                                                                                  N                 1
SYS            1 SYS_AUTO_SQL_TUNING_TASK       EXEC_1240               809 normal, successful completion   INFORMATION          4          0        892                          0 This statement was skipped because it has already been tuned recently.  See task                                                                                  N                 1
SYS            1 SYS_AUTO_SQL_TUNING_TASK       EXEC_1240               810 normal, successful completion   INFORMATION          4          0        893                          0 This statement was skipped because it has already been tuned recently.  See task                                                                                  N                 1
SYS            1 SYS_AUTO_SQL_TUNING_TASK       EXEC_1271               811 normal, successful completion   INFORMATION          4          0        896                          0 The optimizer could not merge the view at line ID 50 of the execution plan.      The optimizer cannot merge a view that contains a set operator.                  N                 0
SYS            1 SYS_AUTO_SQL_TUNING_TASK       EXEC_1271               812 normal, successful completion   INFORMATION          4          0        896                          0 The optimizer could not merge the view at line ID 21 of the execution plan.      The optimizer cannot merge a view that contains a set operator.                  N                 0
SYS            1 SYS_AUTO_SQL_TUNING_TASK       EXEC_1271               813 normal, successful completion   PROBLEM              1          0        896                          0 Some alternative execution plans for this statement were found by searching the                                                                                   N                 0
SYS            1 SYS_AUTO_SQL_TUNING_TASK       EXEC_1271               814 normal, successful completion   PROBLEM              1          0        897                          0 A potentially better execution plan was found for this statement.                                                                                                 N                 0

112 rows selected


```

## 关闭建议的任务
```sql


BEGIN
 dbms_auto_task_admin.disable(
   client_name => 'sql tuning advisor',
   operation   => NULL,
   window_name => NULL);
   
 dbms_auto_task_admin.disable(
   client_name => 'auto space advisor job',
   operation   => NULL,
   window_name => NULL);
      
END;

```
