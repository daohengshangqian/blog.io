---
layout: post
title: 利用DBMS_STATS包修改统计信息，欺骗优化器，生成糟糕的执行计划
date: 2014-06-20
categories: blog
tags: [PL/SQL]
description: 利用DBMS_STATS包修改统计信息，欺骗优化器，生成糟糕的执行计划
---

在使用基于成本的优化器的优化器时，优化器生产执行计划时要估算每条SQL的执行成本，选择最佳的执行计划来执行sql语句。通过操纵统计信息就可以简介操纵执行计划的生成。

当然 需要强调的一点是，这是非常危险的行为
##  创建测试表

```sql

SQL> create table test_stats  as  
2  select * from dba_objects ;  

```

## 收集统计信息

```sql
SQL> EXEC dbms_stats.gather_table_stats(ownname => 'DAO',tabname => 'TEST_STATS',cascade => TRUE);  
PL/SQL procedure successfully completed.
```

## 创建索引

```sql
SQL> alter table TEST_STATS  
2    add constraint dao_stats_pk primary key (OBJECT_ID);  
Table altered.  
```

## 统计表行数并查看执行计划

```sql
SQL> select  count(*)  
2  from dao.test_stats  t ;  
COUNT(*)  
----------  
75675  
Execution Plan  
----------------------------------------------------------  
Plan hash value: 2395052010  
------------------------------------------------------------------------------  
| Id  | Operation             | Name         | Rows  | Cost (%CPU)| Time     |  
------------------------------------------------------------------------------  
|   0 | SELECT STATEMENT      |              |     1 |    44   (0)| 00:00:01 |  
|   1 |  SORT AGGREGATE       |              |     1 |            |          |  
|   2 |   INDEX FAST FULL SCAN| DAO_STATS_PK | 75675 |    44   (0)| 00:00:01 |  
------------------------------------------------------------------------------  
Statistics  
----------------------------------------------------------  
1  recursive calls  
0  db block gets  
164  consistent gets  
0  physical reads  
0  redo size  
528  bytes sent via SQL*Net to client  
523  bytes received via SQL*Net from client  
2  SQL*Net roundtrips to/from client  
0  sorts (memory)  
0  sorts (disk)  
1  rows processed  
```

## 利用dbms_stats.set_table_stats 修改表统计信息

```sql
SQL> BEGIN  
2  dbms_stats.set_table_stats(ownname =>'DAO',tabname => 'TEST_STATS',numrows => 1,numblks => 1);  
3  END ;  
4  /  
PL/SQL procedure successfully completed.  
```

## 再次查看统计信息

```sql
SQL> select  t.owner,t.table_name,t.num_rows,t.blocks  
2  from dba_tables t  
3  where t.table_name ='TEST_STATS'  
4  AND T.OWNER='DAO';  
OWNER                          TABLE_NAME                       NUM_ROWS     BLOCKS  
------------------------------ ------------------------------ ---------- ----------  
DAO                            TEST_STATS                              1          1  
```

## 查看最新的执行计划

```sql
SQL> select  count(*)  
2  from dao.test_stats  t ;  
COUNT(*)  
----------  
75675  
Execution Plan  
----------------------------------------------------------  
Plan hash value: 1525674154  
-------------------------------------------------------------------------  
| Id  | Operation          | Name       | Rows  | Cost (%CPU)| Time     |  
-------------------------------------------------------------------------  
|   0 | SELECT STATEMENT   |            |     1 |     2   (0)| 00:00:01 |  
|   1 |  SORT AGGREGATE    |            |     1 |            |          |  
|   2 |   TABLE ACCESS FULL| TEST_STATS |     1 |     2   (0)| 00:00:01 |  
-------------------------------------------------------------------------  
Statistics  
----------------------------------------------------------  
0  recursive calls  
0  db block gets  
164  consistent gets  
0  physical reads  
0  redo size  
528  bytes sent via SQL*Net to client  
523  bytes received via SQL*Net from client  
2  SQL*Net roundtrips to/from client  
0  sorts (memory)  
0  sorts (disk)  
1  rows processed  
```
