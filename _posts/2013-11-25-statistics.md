---
layout: post
title:  Oracle统计信息详解
date: 2013-11-25
categories: blog
tags: [PL/SQL]
description: Oracle统计信息详解
---

# 统计信息概述

个人认为,统计信息是对数据分布与对象结构的一种综合性描述.Oracle优化器在生成执行计划的时,就需要依赖统计信息计算成本.并选取成本最低的执行计划.

![image](http://wx3.sinaimg.cn/mw690/006Pv9mtly1fnilgvqzehj30dk0ezjtj.jpg)

## 统计信息分类

### 表统计信息


选项 | 含义 | 对应列
---|---|---
Number of rows | 表中的行数| DBA_TABLES.NUM_ROWS
Number of blocks | 表包含的块数 |DBA_TABLES.BLOCKS,DBA_TABLES.EMPTY_BLOCKS
Average row length | 行平均长度 |DBA_TABLES.AVG_ROW_LEN


### 列统计信息
选项 | 含义 | 对应列
---|---|---
Number of distinct values (NDV) in a column| 列中唯一值数量 |DBA_TAB_COLS.NUM_DISTINCT
Number of nulls in a column| 列中空值数量 |DBA_TAB_COLS.NUM_NULLS
Data distribution (histogram)|数据分布(直方图) |DBA_TAB_COLS.NUM_BUCKETS
Extended statistics|扩展统计信息 |
### 索引统计信息

选项 | 含义 | 对应列
---|---|---
Number of leaf blocks |叶子块数量 | DBA_INDEXES.LEAF_BLOCKS
Number of levels      |索引层数 | DBA_INDEXES.BLEVEL
Index clustering factor| 聚簇因子|DBA_INDEXES.CLUSTERING_FACTOR

### 系统统计信息

选项 | 值 
---|---
CPUSPEED||
CPUSPEEDNW|2070|
IOSEEKTIM|1|
IOTFRSPEED|4096|
MAXTHR||
MBRC||
MREADTIM||
SLAVETHR||
SREADTIM||



# 统计信息的收集

##  表级别的统计信息收集 

我们可以使用DBMS_STATS.GATHER_TABLE_STATS来收集表的统计信息.

```sql

DBMS_STATS.GATHER_TABLE_STATS (
   ownname          VARCHAR2, 
   tabname          VARCHAR2, 
   partname         VARCHAR2 DEFAULT NULL,
   estimate_percent NUMBER   DEFAULT to_estimate_percent_type 
                                                  (get_param('ESTIMATE_PERCENT')),
   block_sample     BOOLEAN  DEFAULT FALSE,
   method_opt       VARCHAR2 DEFAULT get_param('METHOD_OPT'),
   degree           NUMBER   DEFAULT to_degree_type(get_param('DEGREE')),
   granularity      VARCHAR2 DEFAULT GET_PARAM('GRANULARITY'), 
   cascade          BOOLEAN  DEFAULT to_cascade_type(get_param('CASCADE')),
   stattab          VARCHAR2 DEFAULT NULL, 
   statid           VARCHAR2 DEFAULT NULL,
   statown          VARCHAR2 DEFAULT NULL,
   no_invalidate    BOOLEAN  DEFAULT  to_no_invalidate_type (
                                                      get_param('NO_INVALIDATE')),
   stattype         VARCHAR2 DEFAULT 'DATA',
   force            BOOLEAN  DEFAULT FALSE,
   context          DBMS_STATS.CCONTEXT DEFAULT NULL, -- non operative
   options          VARCHAR2 DEFAULT get_param('OPTIONS'));

```

下面看一下这里面的参数


参数名 | 含义
---|---
ownname | 表所有者
tabname | 表明
partname| 分区名(仅适用于分区表)
estimate_percent|采样比率
block_sample|适口开启随机块采样
method_opt|方法选项 
degree|并发度
granularity|粒度
cascade|串联
stattab|状态表名
statid|状态ID
statown|状态表所有者
no_invalidate|游标失效选项
stattype|状态类型 只有一个选项为 DATA
force| 是否强制收集
context|NOT used
options|选项


### 重点参数

#### estimate_percent 

抽样百分比.介于0.000001 和 100之间.
可以使用DBMS_STATS.AUTO_SAMPLE_SIZE 让Oracle自己决定抽样百分比.

#### block_sample 

决定数据库采用哪种随即采样方式.快随机采样,还是行随机采样.

#### method_opt 
方法选项,控制列数据的收集.
- FOR ALL [INDEXED | HIDDEN] COLUMNS [size_clause] 指定收集所有索引(隐藏)列的统计信息

- FOR COLUMNS [column_clause] [size_clause]
指定收集某些列的统计信息

column_clause 的选项

1. column_name 列名
2. extension   可以是一组列或表达式

size_clause 选项 SIZE {integer | REPEAT | AUTO | SKEWONLY}

1. integer  数字范围从1到2048
2. REPEAT   如果该列存在直方图,则收集直方图
3. AUTO      Oracle 根据列的数据分布与负载来决定是否收集直方图
4. SKEWONLY  Oracle 根据列的数据分布来决定是否收集直方图

#### degree
并行度.
默认是空,意思为根据表的degree来决定并行度.




#### granularity 

粒度  仅限于分区表
 选项|含义
 ---|---
 ALL |收集所有的统计信息(子分区,分区,还有全局)统计信息
 APPROX_GLOBAL AND PARTITION |获取分区信息与全局统计信息,只不过全局信息由分区信息合计得到
 AUTO    |取决于分区类型.这是默认值 
 DEFAULT |获取全局与分区级别的统计信息.该选项已经过时
 GLOBAL  |只获取全局信息
 GLOBAL AND PARTITION | 获取全局级别的分区统计信息.
PARTITION  |只获取分区级别的统计信息
SUBPARTITION |收集子分区级别的统计信息

#### cascade
在收集表统计信息时是否收集相关索引的统计信息

#### stattab
指定讲收集的统计信息存在哪张统计信息表中.

#### no_invalidate
收集统计表信息后，相关游标是否失效.
- TRUE  不立即失效
- FALSE 立即失效
- DBMS_STATS.AUTO_INVALIDATE 在一定时间段内失效. 可以避免同一时刻产生大量的硬解析.

#### force
在表统计信息被锁时是否收集统计信息

#### options
- GATHER 收集表中所有对象的统计信息
- 收集必要的统计信息.Oracle推荐表在经历过大量装载后使用此选项收集统计信息.此选项与INCREMENTAL(增量统计信息) 选项冲突.

## 用户级别的统计信息收集

我们可以使用DBMS_STATS.GATHER_SCHEMA_STATS 来收集某个用户下所有对象的统计信息.

```sql
DBMS_STATS.GATHER_SCHEMA_STATS ( 
   ownname          VARCHAR2, 
   estimate_percent NUMBER   DEFAULT to_estimate_percent_type 
                                                (get_param('ESTIMATE_PERCENT')), 
   block_sample     BOOLEAN  DEFAULT FALSE, 
   method_opt       VARCHAR2 DEFAULT get_param('METHOD_OPT'), 
   degree           NUMBER   DEFAULT to_degree_type(get_param('DEGREE')), 
   granularity      VARCHAR2 DEFAULT GET_PARAM('GRANULARITY'), 
   cascade          BOOLEAN  DEFAULT to_cascade_type(get_param('CASCADE')), 
   stattab          VARCHAR2 DEFAULT NULL, 
   statid           VARCHAR2 DEFAULT NULL, 
   options          VARCHAR2 DEFAULT 'GATHER', 
   statown          VARCHAR2 DEFAULT NULL, 
   no_invalidate    BOOLEAN  DEFAULT to_no_invalidate_type (
                                     get_param('NO_INVALIDATE'), 
   force            BOOLEAN DEFAULT FALSE,
   obj_filter_list  ObjectTab DEFAULT NULL);
   
```

### 重点参数

#### obj_filter_list

对象过滤器,可以过滤某一部分对象.如下面的例子只收集SH用户下表SALES和表COSTS的统计信息.

```sql

DECLARE
   filter_lst  DBMS_STATS.OBJECTTAB := DBMS_STATS.OBJECTTAB();
BEGIN
   filter_lst.extend(2);
   filter_lst(1).ownname := 'SH';
   filter_lst(1).objname := 'SALES';
   filter_lst(2).ownname := 'SH';
   filter_lst(2).objname := 'COSTS';
   DBMS_STATS.GATHER_SCHEMA_STATS(ownname=>'SH',obj_filter_list=>filter_lst);
 END;
```





## 数据库级别的统计信息收集
我们可以使用DBMS_STATS.GATHER_DATABASE_STATS 来收集数据库下所有对象的统计信息

```sql
DBMS_STATS.GATHER_DATABASE_STATS (
   estimate_percent NUMBER   DEFAULT to_estimate_percent_type 
                                                (get_param('ESTIMATE_PERCENT')),
   block_sample     BOOLEAN  DEFAULT FALSE,
   method_opt       VARCHAR2 DEFAULT get_param('METHOD_OPT'),
   degree           NUMBER   DEFAULT to_degree_type(get_param('DEGREE')),
   granularity      VARCHAR2 DEFAULT GET_PARAM('GRANULARITY'), 
   cascade          BOOLEAN  DEFAULT to_cascade_type(get_param('CASCADE')),
   stattab          VARCHAR2 DEFAULT NULL, 
   statid           VARCHAR2 DEFAULT NULL,
   options          VARCHAR2 DEFAULT 'GATHER',
   objlist          OUT      ObjectTab,
   statown          VARCHAR2 DEFAULT NULL,
   gather_sys       BOOLEAN  DEFAULT TRUE,
   no_invalidate    BOOLEAN  DEFAULT to_no_invalidate_type (
                                     get_param('NO_INVALIDATE')),
   obj_filter_list ObjectTab DEFAULT NULL);

```


# 统计信息的编辑

## 修改表的统计信息

### 查看当前表的统计信息
``` sql
SQL> select t.owner,
  2         t.table_name,
  3         t.num_rows,
  4         t.avg_row_len,
  5         t.blocks,
  6         t.empty_blocks
  7    from dba_tables t
  8   where t.table_name = 'DAO_OBJECTS' ;

OWNER  TABLE_NAME      NUM_ROWS AVG_ROW_LEN     BLOCKS EMPTY_BLOCKS
------ ------------- ---------- ----------- ---------- ------------
DAO    DAO_OBJECTS        75583          97       1102            0 　

```

### 查看某条SQL的执行计划

```sql


SQL> select count(*) from dao_objects ;


Execution Plan
----------------------------------------------------------
Plan hash value: 2395052010

------------------------------------------------------------------------------
| Id  | Operation             | Name         | Rows  | Cost (%CPU)| Time     |
------------------------------------------------------------------------------
|   0 | SELECT STATEMENT      |              |     1 |   115   (2)| 00:00:01 |
|   1 |  SORT AGGREGATE       |              |     1 |            |          |
|   2 |   INDEX FAST FULL SCAN| DAO_STATS_PK | 75583 |   115   (2)| 00:00:01 |
------------------------------------------------------------------------------


Statistics
----------------------------------------------------------
          1  recursive calls
          0  db block gets
        164  consistent gets
          0  physical reads
          0  redo size
        528  bytes sent via SQL*Net to client
        524  bytes received via SQL*Net from client
          2  SQL*Net roundtrips to/from client
          0  sorts (memory)
          0  sorts (disk)
          1  rows processed
```

### 修改表统计信息

```sql

SQL> begin
  2   dbms_stats.set_table_stats(ownname => 'DAO',
  3                              tabname => 'DAO_OBJECTS',
  4                              numrows => 1,
  5                              avgrlen => 200,
  6                              numblks => 1,
  7                              no_invalidate => false
  8                              );
  9  end ;
 10  /

PL/SQL procedure successfully completed

```


### 查看表统计信息

```sql

SQL> select t.owner,
  2         t.table_name,
  3         t.num_rows,
  4         t.avg_row_len,
  5         t.blocks,
  6         t.empty_blocks
  7    from dba_tables t
  8   where t.table_name = 'DAO_OBJECTS' ;

OWNER  TABLE_NAME     NUM_ROWS AVG_ROW_LEN     BLOCKS EMPTY_BLOCKS
------ ------------ ---------- ----------- ---------- ------------
DAO    DAO_OBJECTS           1         200          1            0

```



### 再次查看执行计划


```sql

SQL> select count(*) from dao_objects ;


Execution Plan
----------------------------------------------------------
Plan hash value: 2941237911

--------------------------------------------------------------------------
| Id  | Operation          | Name        | Rows  | Cost (%CPU)| Time     |
--------------------------------------------------------------------------
|   0 | SELECT STATEMENT   |             |     1 |     3   (0)| 00:00:01 |
|   1 |  SORT AGGREGATE    |             |     1 |            |          |
|   2 |   TABLE ACCESS FULL| DAO_OBJECTS |     1 |     3   (0)| 00:00:01 |
--------------------------------------------------------------------------


Statistics
----------------------------------------------------------
          1  recursive calls
          0  db block gets
       1080  consistent gets
       1076  physical reads
          0  redo size
        528  bytes sent via SQL*Net to client
        524  bytes received via SQL*Net from client
          2  SQL*Net roundtrips to/from client
          0  sorts (memory)
          0  sorts (disk)
          1  rows processed
```

# 统计信息的锁定

我们可以使用dbms_stats.lock_***_stats过程来锁定对象的统计信息.如果锁定了统计信息,在收集统计信息时声明FORCE=TRUE才可以使用新收集的统计信息覆盖现有统计信息.

## 锁定表统计信息 

### 查看统计信息状态
``` sql
SQL> select t.owner,
  2         t.table_name,
  3         t.object_type,
  4         t.stattype_locked
  5    from dba_tab_statistics t
  6   where t.table_name = 'DAO_OBJECTS' ;

OWNER                          TABLE_NAME                     OBJECT_TYPE  STATTYPE_LOCKED
------------------------------ ------------------------------ ------------ ---------------
DAO                            DAO_OBJECTS                    TABLE        


```

### 锁定表统计信息

```sql

SQL> begin
  2   dbms_stats.lock_table_stats(ownname => 'DAO',
  3                               tabname => 'DAO_OBJECTS');
  4  
  5  
  6  end  ;
  7  /

PL/SQL procedure successfully completed
```
### 查看统计信息状态

```sql
SQL> select t.owner,
  2         t.table_name,
  3         t.object_type,
  4         t.stattype_locked
  5    from dba_tab_statistics t
  6   where t.table_name = 'DAO_OBJECTS' ;

OWNER                          TABLE_NAME                     OBJECT_TYPE  STATTYPE_LOCKED
------------------------------ ------------------------------ ------------ ---------------
DAO                            DAO_OBJECTS                    TABLE        ALL
```


### 解锁表统计信息

```sql
SQL> begin
  2   dbms_stats.unlock_table_stats(ownname => 'DAO',
  3                               tabname => 'DAO_OBJECTS');
  4  end  ;
  5  /

PL/SQL procedure successfully completed


```
###  查看表统计信息状态

```sql

SQL> select t.owner,
  2         t.table_name,
  3         t.object_type,
  4         t.stattype_locked
  5    from dba_tab_statistics t
  6   where t.table_name = 'DAO_OBJECTS' ;

OWNER                          TABLE_NAME                     OBJECT_TYPE  STATTYPE_LOCKED
------------------------------ ------------------------------ ------------ ---------------
DAO                            DAO_OBJECTS                    TABLE        

```


## 锁定分区统计信息

### 建立测试分区表

```sql
SQL> CREATE TABLE DAO_PAR
  2  (
  3      EMPNO NUMBER,
  4      ENAME VARCHAR2(20) ,
  5      DEPTNO NUMBER
  6  )
  7  PARTITION BY LIST (DEPTNO)
  8  (
  9      PARTITION DAO_PART1 VALUES  (10) ,
 10      PARTITION DAO_PART2 VALUES  (20)
 11  );

Table created
``` 

### 查看分区统计信息状态

```sql
SQL> select t.owner,
  2         t.table_name,
  3         t.partition_name,
  4         t.object_type,
  5         t.stattype_locked
  6    from dba_tab_statistics t
  7   where t.table_name = 'DAO_PAR'
  8    order by t.partition_position ;

OWNER  TABLE_NAME   PARTITION_NAME   OBJECT_TYPE  STATTYPE_LOCKED
------ ------------ ---------------- ------------ ---------------
DAO    DAO_PAR      DAO_PART1        PARTITION    
DAO    DAO_PAR      DAO_PART2        PARTITION    
DAO    DAO_PAR                       TABLE        
``` 

### 锁定某一分区

```sql
SQL> begin
  2   dbms_stats.lock_partition_stats(ownname => 'DAO',
  3                                   tabname => 'DAO_PAR',
  4                                   partname => 'DAO_PART2');
  5  end  ;
  6  /

PL/SQL procedure successfully completed

``` 

### 查看分区统计信息状态

```sql
SQL> select t.owner,
  2         t.table_name,
  3         t.partition_name,
  4         t.object_type,
  5         t.stattype_locked
  6    from dba_tab_statistics t
  7   where t.table_name = 'DAO_PAR'
  8    order by t.partition_position ;

OWNER   TABLE_NAME   PARTITION_NAME   OBJECT_TYPE  STATTYPE_LOCKED
------- ------------ ---------------- ------------ ---------------
DAO     DAO_PAR      DAO_PART1        PARTITION    
DAO     DAO_PAR      DAO_PART2        PARTITION    ALL
DAO     DAO_PAR                       TABLE   
``` 

### 解锁某一分区

```sql

SQL> begin
  2   dbms_stats.unlock_partition_stats(ownname => 'DAO',
  3                                   tabname => 'DAO_PAR',
  4                                   partname => 'DAO_PART2');
  5  end  ;
  6  /

PL/SQL procedure successfully completed


``` 
### 查看分区统计信息状态

```sql

SQL> select t.owner,
  2         t.table_name,
  3         t.partition_name,
  4         t.object_type,
  5         t.stattype_locked
  6    from dba_tab_statistics t
  7   where t.table_name = 'DAO_PAR'
  8    order by t.partition_position ;

OWNER  TABLE_NAME   PARTITION_NAME   OBJECT_TYPE  STATTYPE_LOCKED
------ ------------ ---------------- ------------ ---------------
DAO    DAO_PAR      DAO_PART1        PARTITION    
DAO    DAO_PAR      DAO_PART2        PARTITION    
DAO    DAO_PAR                       TABLE   

``` 

## 锁定用户下对象的统计信息

### 查看用户统计信息
```sql
SQL> select t.owner,
  2         t.table_name,
  3         t.partition_name,
  4         t.object_type,
  5         t.stattype_locked
  6    from dba_tab_statistics t
  7   where t.owner = 'DAO'  ;

OWNER    TABLE_NAME                     PARTITION_NAME OBJECT_TYPE  STATTYPE_LOCKED
-------- ------------------------------ -------------- ------------ ---------------
DAO      TEST_CONNECT_CYCLE                            TABLE        
DAO      DEPT                                          TABLE        
DAO      DAO_OBJECTS                                   TABLE        
DAO      TEST_FOJ_B                                    TABLE        
DAO      TEST_FOJ_A                                    TABLE        
DAO      TEST_INSERT_DEPT                              TABLE        
DAO      TEST_INSERT_EMP                               TABLE        
DAO      EMP1                                          TABLE        
DAO      GRADE                                         TABLE        
DAO      INSTRUCTOR                                    TABLE        
DAO      GRADE_TYPE                                    TABLE        
DAO      GRADE_CONVERSION                              TABLE        
DAO      GRADE_TYPE_WEIGHT                             TABLE        
DAO      SECTION                                       TABLE        
DAO      COURSE                                        TABLE        
DAO      ENROLLMENT                                    TABLE        
DAO      STUDENT                                       TABLE        
DAO      ZIPCODE                                       TABLE        
DAO      EMPLOYEES_COMB                                TABLE        
DAO      DFC                                           TABLE        
DAO      TEST_FOR_TRC                                  TABLE        
DAO      TEST_NULL                                     TABLE        
DAO      SYS_EXPORT_SCHEMA_01                          TABLE        
DAO      TEST_LISTAA                                   TABLE        
DAO      TEST_OBJECTS                                  TABLE        
DAO      EMP                                           TABLE        
DAO      EMP_DOUBLE                                    TABLE        
DAO      ZHUAN_QIAN                                    TABLE        
DAO      TEST_NULL1                                    TABLE        
DAO      TEST_UK                                       TABLE        
DAO      TEST_UK1                                      TABLE        
DAO      TEST_UK2                                      TABLE        
DAO      TEST_UK3                                      TABLE        
DAO      TEST_PK                                       TABLE        
DAO      TEST_PK1                                      TABLE        
DAO      TEST_PK2                                      TABLE        
DAO      TEST_FK_DEPT                                  TABLE        
DAO      TEST_PK3                                      TABLE        
DAO      TEST_FK_DEPT1                                 TABLE        
DAO      TEST_FK_EMP                                   TABLE        
DAO      TEST_LHR                                      TABLE        
DAO      TEST_LHR1                                     TABLE        
DAO      T_COL_ROW                                     TABLE        
DAO      TEST_FK_EMP1                                  TABLE        
DAO      TEST_FK_DEPT2                                 TABLE        
DAO      TEST_FK_EMP2                                  TABLE        
DAO      TEST_FK_EMP3                                  TABLE        
DAO      TEST_FK_DEPT3                                 TABLE        
DAO      TEST_CHECK                                    TABLE        
DAO      TEST_CHECK1                                   TABLE        
DAO      TEST_CHECK3                                   TABLE        
DAO      TEST_CHECK2                                   TABLE        
DAO      DAO_PAR                                       TABLE        
DAO      SIGN_INFO                                     TABLE        
DAO      SQL_FAULT                                     TABLE        
DAO      BIG_TABLE                                     TABLE        
DAO      TEST                                          TABLE        
DAO      E_ORDER                                       TABLE        
DAO      T_ORDER_INFO                                  TABLE        
DAO      TEST_SEGMENT_ADVISOR                          TABLE        
DAO      EMPLOYEES_DATE                                TABLE        
DAO      EMPLOYEES_CHAR                                TABLE        
DAO      EMPLOYEES_NUMBER                              TABLE        
DAO      AAA                                           TABLE        
DAO      BBB                                           TABLE        
DAO      EMPLOYEES_COMB                 P201609        PARTITION    
DAO      EMPLOYEES_COMB                 P201610        PARTITION    
DAO      EMPLOYEES_COMB                 P201611        PARTITION    
DAO      EMPLOYEES_COMB                 P201612        PARTITION    
DAO      EMPLOYEES_DATE                 P201601        PARTITION    
DAO      EMPLOYEES_DATE                 P201602        PARTITION    
DAO      EMPLOYEES_DATE                 P201603        PARTITION    
DAO      EMPLOYEES_DATE                 P201604        PARTITION    
DAO      EMPLOYEES_DATE                 P201605        PARTITION    
DAO      EMPLOYEES_DATE                 P201606        PARTITION    
DAO      EMPLOYEES_DATE                 P201607        PARTITION    
DAO      EMPLOYEES_DATE                 P201608        PARTITION    
DAO      EMPLOYEES_DATE                 P201609        PARTITION    
DAO      EMPLOYEES_DATE                 P201610        PARTITION    
DAO      EMPLOYEES_DATE                 P201611        PARTITION    
DAO      EMPLOYEES_DATE                 P201612        PARTITION    
DAO      EMPLOYEES_CHAR                 P201601        PARTITION    
DAO      EMPLOYEES_CHAR                 P201602        PARTITION    
DAO      EMPLOYEES_CHAR                 P201603        PARTITION    
DAO      EMPLOYEES_CHAR                 P201604        PARTITION    
DAO      EMPLOYEES_CHAR                 P201605        PARTITION    
DAO      EMPLOYEES_CHAR                 P201606        PARTITION    
DAO      EMPLOYEES_CHAR                 P201607        PARTITION    
DAO      EMPLOYEES_CHAR                 P201608        PARTITION    
DAO      EMPLOYEES_CHAR                 P201609        PARTITION    
DAO      EMPLOYEES_CHAR                 P201610        PARTITION    
DAO      EMPLOYEES_CHAR                 P201611        PARTITION    
DAO      EMPLOYEES_CHAR                 P201612        PARTITION    
DAO      EMPLOYEES_NUMBER               P201601        PARTITION    
DAO      EMPLOYEES_NUMBER               P201602        PARTITION    
DAO      EMPLOYEES_NUMBER               P201603        PARTITION    
DAO      EMPLOYEES_NUMBER               P201604        PARTITION    
DAO      EMPLOYEES_NUMBER               P201605        PARTITION    
DAO      EMPLOYEES_NUMBER               P201606        PARTITION    
DAO      EMPLOYEES_NUMBER               P201607        PARTITION    
DAO      EMPLOYEES_NUMBER               P201608        PARTITION    
DAO      EMPLOYEES_NUMBER               P201609        PARTITION    
DAO      EMPLOYEES_NUMBER               P201610        PARTITION    
DAO      EMPLOYEES_NUMBER               P201611        PARTITION    
DAO      EMPLOYEES_NUMBER               P201612        PARTITION    
DAO      EMPLOYEES_COMB                 P201601        PARTITION    
DAO      EMPLOYEES_COMB                 P201602        PARTITION    
DAO      EMPLOYEES_COMB                 P201603        PARTITION    
DAO      EMPLOYEES_COMB                 P201604        PARTITION    
DAO      EMPLOYEES_COMB                 P201605        PARTITION    
DAO      EMPLOYEES_COMB                 P201606        PARTITION    
DAO      EMPLOYEES_COMB                 P201607        PARTITION    
DAO      EMPLOYEES_COMB                 P201608        PARTITION    
DAO      BIN$R8oRAUHZX8ngUwEAAH/a0A==$0 CUS_PART1      PARTITION    
DAO      BIN$R8oRAUHZX8ngUwEAAH/a0A==$0 CUS_PART2      PARTITION    
DAO      DAO_PAR                        DAO_PART1      PARTITION    
DAO      DAO_PAR                        DAO_PART2      PARTITION    
```

### 锁定用户统计信息

```sql

SQL> begin
  2  dbms_stats.lock_schema_stats(ownname => 'DAO');
  3  end ;
  4  /

PL/SQL procedure successfully completed
```



### 查看用户统计信息

```sql

SQL> select t.owner,
  2         t.table_name,
  3         t.partition_name,
  4         t.object_type,
  5         t.stattype_locked
  6    from dba_tab_statistics t
  7   where t.owner = 'DAO'  ;

OWNER  TABLE_NAME                     PARTITION OBJECT_TYPE  STATTYPE_LOCKED
------ ------------------------------ --------- ------------ ---------------
DAO    TEST_CONNECT_CYCLE                       TABLE        ALL
DAO    DEPT                                     TABLE        ALL
DAO    DAO_OBJECTS                              TABLE        ALL
DAO    TEST_FOJ_B                               TABLE        ALL
DAO    TEST_FOJ_A                               TABLE        ALL
DAO    TEST_INSERT_DEPT                         TABLE        ALL
DAO    TEST_INSERT_EMP                          TABLE        ALL
DAO    EMP1                                     TABLE        ALL
DAO    GRADE                                    TABLE        ALL
DAO    INSTRUCTOR                               TABLE        ALL
DAO    GRADE_TYPE                               TABLE        ALL
DAO    GRADE_CONVERSION                         TABLE        ALL
DAO    GRADE_TYPE_WEIGHT                        TABLE        ALL
DAO    SECTION                                  TABLE        ALL
DAO    COURSE                                   TABLE        ALL
DAO    ENROLLMENT                               TABLE        ALL
DAO    STUDENT                                  TABLE        ALL
DAO    ZIPCODE                                  TABLE        ALL
DAO    EMPLOYEES_COMB                           TABLE        ALL
DAO    DFC                                      TABLE        ALL
DAO    TEST_FOR_TRC                             TABLE        ALL
DAO    TEST_NULL                                TABLE        ALL
DAO    SYS_EXPORT_SCHEMA_01                     TABLE        ALL
DAO    TEST_LISTAA                              TABLE        ALL
DAO    TEST_OBJECTS                             TABLE        ALL
DAO    EMP                                      TABLE        ALL
DAO    EMP_DOUBLE                               TABLE        ALL
DAO    ZHUAN_QIAN                               TABLE        ALL
DAO    TEST_NULL1                               TABLE        ALL
DAO    TEST_UK                                  TABLE        ALL
DAO    TEST_UK1                                 TABLE        ALL
DAO    TEST_UK2                                 TABLE        ALL
DAO    TEST_UK3                                 TABLE        ALL
DAO    TEST_PK                                  TABLE        ALL
DAO    TEST_PK1                                 TABLE        ALL
DAO    TEST_PK2                                 TABLE        ALL
DAO    TEST_FK_DEPT                             TABLE        ALL
DAO    TEST_PK3                                 TABLE        ALL
DAO    TEST_FK_DEPT1                            TABLE        ALL
DAO    TEST_FK_EMP                              TABLE        ALL
DAO    TEST_LHR                                 TABLE        ALL
DAO    TEST_LHR1                                TABLE        ALL
DAO    T_COL_ROW                                TABLE        ALL
DAO    TEST_FK_EMP1                             TABLE        ALL
DAO    TEST_FK_DEPT2                            TABLE        ALL
DAO    TEST_FK_EMP2                             TABLE        ALL
DAO    TEST_FK_EMP3                             TABLE        ALL
DAO    TEST_FK_DEPT3                            TABLE        ALL
DAO    TEST_CHECK                               TABLE        ALL
DAO    TEST_CHECK1                              TABLE        ALL
DAO    TEST_CHECK3                              TABLE        ALL
DAO    TEST_CHECK2                              TABLE        ALL
DAO    DAO_PAR                                  TABLE        ALL
DAO    SIGN_INFO                                TABLE        ALL
DAO    SQL_FAULT                                TABLE        ALL
DAO    BIG_TABLE                                TABLE        ALL
DAO    TEST                                     TABLE        ALL
DAO    E_ORDER                                  TABLE        ALL
DAO    T_ORDER_INFO                             TABLE        ALL
DAO    TEST_SEGMENT_ADVISOR                     TABLE        ALL
DAO    EMPLOYEES_DATE                           TABLE        ALL
DAO    EMPLOYEES_CHAR                           TABLE        ALL
DAO    EMPLOYEES_NUMBER                         TABLE        ALL
DAO    AAA                                      TABLE        ALL
DAO    BBB                                      TABLE        ALL
DAO    EMPLOYEES_COMB                 P201609   PARTITION    ALL
DAO    EMPLOYEES_COMB                 P201610   PARTITION    ALL
DAO    EMPLOYEES_COMB                 P201611   PARTITION    ALL
DAO    EMPLOYEES_COMB                 P201612   PARTITION    ALL
DAO    EMPLOYEES_DATE                 P201601   PARTITION    ALL
DAO    EMPLOYEES_DATE                 P201602   PARTITION    ALL
DAO    EMPLOYEES_DATE                 P201603   PARTITION    ALL
DAO    EMPLOYEES_DATE                 P201604   PARTITION    ALL
DAO    EMPLOYEES_DATE                 P201605   PARTITION    ALL
DAO    EMPLOYEES_DATE                 P201606   PARTITION    ALL
DAO    EMPLOYEES_DATE                 P201607   PARTITION    ALL
DAO    EMPLOYEES_DATE                 P201608   PARTITION    ALL
DAO    EMPLOYEES_DATE                 P201609   PARTITION    ALL
DAO    EMPLOYEES_DATE                 P201610   PARTITION    ALL
DAO    EMPLOYEES_DATE                 P201611   PARTITION    ALL
DAO    EMPLOYEES_DATE                 P201612   PARTITION    ALL
DAO    EMPLOYEES_CHAR                 P201601   PARTITION    ALL
DAO    EMPLOYEES_CHAR                 P201602   PARTITION    ALL
DAO    EMPLOYEES_CHAR                 P201603   PARTITION    ALL
DAO    EMPLOYEES_CHAR                 P201604   PARTITION    ALL
DAO    EMPLOYEES_CHAR                 P201605   PARTITION    ALL
DAO    EMPLOYEES_CHAR                 P201606   PARTITION    ALL
DAO    EMPLOYEES_CHAR                 P201607   PARTITION    ALL
DAO    EMPLOYEES_CHAR                 P201608   PARTITION    ALL
DAO    EMPLOYEES_CHAR                 P201609   PARTITION    ALL
DAO    EMPLOYEES_CHAR                 P201610   PARTITION    ALL
DAO    EMPLOYEES_CHAR                 P201611   PARTITION    ALL
DAO    EMPLOYEES_CHAR                 P201612   PARTITION    ALL
DAO    EMPLOYEES_NUMBER               P201601   PARTITION    ALL
DAO    EMPLOYEES_NUMBER               P201602   PARTITION    ALL
DAO    EMPLOYEES_NUMBER               P201603   PARTITION    ALL
DAO    EMPLOYEES_NUMBER               P201604   PARTITION    ALL
DAO    EMPLOYEES_NUMBER               P201605   PARTITION    ALL
DAO    EMPLOYEES_NUMBER               P201606   PARTITION    ALL
DAO    EMPLOYEES_NUMBER               P201607   PARTITION    ALL
DAO    EMPLOYEES_NUMBER               P201608   PARTITION    ALL
DAO    EMPLOYEES_NUMBER               P201609   PARTITION    ALL
DAO    EMPLOYEES_NUMBER               P201610   PARTITION    ALL
DAO    EMPLOYEES_NUMBER               P201611   PARTITION    ALL
DAO    EMPLOYEES_NUMBER               P201612   PARTITION    ALL
DAO    EMPLOYEES_COMB                 P201601   PARTITION    ALL
DAO    EMPLOYEES_COMB                 P201602   PARTITION    ALL
DAO    EMPLOYEES_COMB                 P201603   PARTITION    ALL
DAO    EMPLOYEES_COMB                 P201604   PARTITION    ALL
DAO    EMPLOYEES_COMB                 P201605   PARTITION    ALL
DAO    EMPLOYEES_COMB                 P201606   PARTITION    ALL
DAO    EMPLOYEES_COMB                 P201607   PARTITION    ALL
DAO    EMPLOYEES_COMB                 P201608   PARTITION    ALL
DAO    BIN$R8oRAUHZX8ngUwEAAH/a0A==$0 CUS_PART1 PARTITION    
DAO    BIN$R8oRAUHZX8ngUwEAAH/a0A==$0 CUS_PART2 PARTITION    
DAO    DAO_PAR                        DAO_PART1 PARTITION    ALL
DAO    DAO_PAR                        DAO_PART2 PARTITION    ALL
```

### 解锁用户统计信息
```sql

SQL>  begin
  2  dbms_stats.unlock_schema_stats(ownname => 'DAO');
  3  end ;
  4  /

PL/SQL procedure successfully completed

```


### 查看用户统计信息

```sql
SQL> select t.owner,
  2         t.table_name,
  3         t.partition_name,
  4         t.object_type,
  5         t.stattype_locked
  6    from dba_tab_statistics t
  7   where t.owner = 'DAO'  ;

OWNER    TABLE_NAME                     PARTITION_NAME OBJECT_TYPE  STATTYPE_LOCKED
-------- ------------------------------ -------------- ------------ ---------------
DAO      TEST_CONNECT_CYCLE                            TABLE        
DAO      DEPT                                          TABLE        
DAO      DAO_OBJECTS                                   TABLE        
DAO      TEST_FOJ_B                                    TABLE        
DAO      TEST_FOJ_A                                    TABLE        
DAO      TEST_INSERT_DEPT                              TABLE        
DAO      TEST_INSERT_EMP                               TABLE        
DAO      EMP1                                          TABLE        
DAO      GRADE                                         TABLE        
DAO      INSTRUCTOR                                    TABLE        
DAO      GRADE_TYPE                                    TABLE        
DAO      GRADE_CONVERSION                              TABLE        
DAO      GRADE_TYPE_WEIGHT                             TABLE        
DAO      SECTION                                       TABLE        
DAO      COURSE                                        TABLE        
DAO      ENROLLMENT                                    TABLE        
DAO      STUDENT                                       TABLE        
DAO      ZIPCODE                                       TABLE        
DAO      EMPLOYEES_COMB                                TABLE        
DAO      DFC                                           TABLE        
DAO      TEST_FOR_TRC                                  TABLE        
DAO      TEST_NULL                                     TABLE        
DAO      SYS_EXPORT_SCHEMA_01                          TABLE        
DAO      TEST_LISTAA                                   TABLE        
DAO      TEST_OBJECTS                                  TABLE        
DAO      EMP                                           TABLE        
DAO      EMP_DOUBLE                                    TABLE        
DAO      ZHUAN_QIAN                                    TABLE        
DAO      TEST_NULL1                                    TABLE        
DAO      TEST_UK                                       TABLE        
DAO      TEST_UK1                                      TABLE        
DAO      TEST_UK2                                      TABLE        
DAO      TEST_UK3                                      TABLE        
DAO      TEST_PK                                       TABLE        
DAO      TEST_PK1                                      TABLE        
DAO      TEST_PK2                                      TABLE        
DAO      TEST_FK_DEPT                                  TABLE        
DAO      TEST_PK3                                      TABLE        
DAO      TEST_FK_DEPT1                                 TABLE        
DAO      TEST_FK_EMP                                   TABLE        
DAO      TEST_LHR                                      TABLE        
DAO      TEST_LHR1                                     TABLE        
DAO      T_COL_ROW                                     TABLE        
DAO      TEST_FK_EMP1                                  TABLE        
DAO      TEST_FK_DEPT2                                 TABLE        
DAO      TEST_FK_EMP2                                  TABLE        
DAO      TEST_FK_EMP3                                  TABLE        
DAO      TEST_FK_DEPT3                                 TABLE        
DAO      TEST_CHECK                                    TABLE        
DAO      TEST_CHECK1                                   TABLE        
DAO      TEST_CHECK3                                   TABLE        
DAO      TEST_CHECK2                                   TABLE        
DAO      DAO_PAR                                       TABLE        
DAO      SIGN_INFO                                     TABLE        
DAO      SQL_FAULT                                     TABLE        
DAO      BIG_TABLE                                     TABLE        
DAO      TEST                                          TABLE        
DAO      E_ORDER                                       TABLE        
DAO      T_ORDER_INFO                                  TABLE        
DAO      TEST_SEGMENT_ADVISOR                          TABLE        
DAO      EMPLOYEES_DATE                                TABLE        
DAO      EMPLOYEES_CHAR                                TABLE        
DAO      EMPLOYEES_NUMBER                              TABLE        
DAO      AAA                                           TABLE        
DAO      BBB                                           TABLE        
DAO      EMPLOYEES_COMB                 P201609        PARTITION    
DAO      EMPLOYEES_COMB                 P201610        PARTITION    
DAO      EMPLOYEES_COMB                 P201611        PARTITION    
DAO      EMPLOYEES_COMB                 P201612        PARTITION    
DAO      EMPLOYEES_DATE                 P201601        PARTITION    
DAO      EMPLOYEES_DATE                 P201602        PARTITION    
DAO      EMPLOYEES_DATE                 P201603        PARTITION    
DAO      EMPLOYEES_DATE                 P201604        PARTITION    
DAO      EMPLOYEES_DATE                 P201605        PARTITION    
DAO      EMPLOYEES_DATE                 P201606        PARTITION    
DAO      EMPLOYEES_DATE                 P201607        PARTITION    
DAO      EMPLOYEES_DATE                 P201608        PARTITION    
DAO      EMPLOYEES_DATE                 P201609        PARTITION    
DAO      EMPLOYEES_DATE                 P201610        PARTITION    
DAO      EMPLOYEES_DATE                 P201611        PARTITION    
DAO      EMPLOYEES_DATE                 P201612        PARTITION    
DAO      EMPLOYEES_CHAR                 P201601        PARTITION    
DAO      EMPLOYEES_CHAR                 P201602        PARTITION    
DAO      EMPLOYEES_CHAR                 P201603        PARTITION    
DAO      EMPLOYEES_CHAR                 P201604        PARTITION    
DAO      EMPLOYEES_CHAR                 P201605        PARTITION    
DAO      EMPLOYEES_CHAR                 P201606        PARTITION    
DAO      EMPLOYEES_CHAR                 P201607        PARTITION    
DAO      EMPLOYEES_CHAR                 P201608        PARTITION    
DAO      EMPLOYEES_CHAR                 P201609        PARTITION    
DAO      EMPLOYEES_CHAR                 P201610        PARTITION    
DAO      EMPLOYEES_CHAR                 P201611        PARTITION    
DAO      EMPLOYEES_CHAR                 P201612        PARTITION    
DAO      EMPLOYEES_NUMBER               P201601        PARTITION    
DAO      EMPLOYEES_NUMBER               P201602        PARTITION    
DAO      EMPLOYEES_NUMBER               P201603        PARTITION    
DAO      EMPLOYEES_NUMBER               P201604        PARTITION    
DAO      EMPLOYEES_NUMBER               P201605        PARTITION    
DAO      EMPLOYEES_NUMBER               P201606        PARTITION    
DAO      EMPLOYEES_NUMBER               P201607        PARTITION    
DAO      EMPLOYEES_NUMBER               P201608        PARTITION    
DAO      EMPLOYEES_NUMBER               P201609        PARTITION    
DAO      EMPLOYEES_NUMBER               P201610        PARTITION    
DAO      EMPLOYEES_NUMBER               P201611        PARTITION    
DAO      EMPLOYEES_NUMBER               P201612        PARTITION    
DAO      EMPLOYEES_COMB                 P201601        PARTITION    
DAO      EMPLOYEES_COMB                 P201602        PARTITION    
DAO      EMPLOYEES_COMB                 P201603        PARTITION    
DAO      EMPLOYEES_COMB                 P201604        PARTITION    
DAO      EMPLOYEES_COMB                 P201605        PARTITION    
DAO      EMPLOYEES_COMB                 P201606        PARTITION    
DAO      EMPLOYEES_COMB                 P201607        PARTITION    
DAO      EMPLOYEES_COMB                 P201608        PARTITION    
DAO      BIN$R8oRAUHZX8ngUwEAAH/a0A==$0 CUS_PART1      PARTITION    
DAO      BIN$R8oRAUHZX8ngUwEAAH/a0A==$0 CUS_PART2      PARTITION    
DAO      DAO_PAR                        DAO_PART1      PARTITION    
DAO      DAO_PAR                        DAO_PART2      PARTITION    
```


# 统计信息的导出导入

## 建立统计信息表

```sql
SQL> begin
  2   dbms_stats.create_stat_table(ownname => 'DAO',
  3                                stattab => 'DAO_STATS') ;
  4  end ;
  5  /

PL/SQL procedure successfully completed

```


## 查看统计信息表

```sql

SQL> desc DAO.DAO_STATS ;
Name    Type           Nullable Default Comments 
------- -------------- -------- ------- -------- 
STATID  VARCHAR2(30)   Y                         
TYPE    CHAR(1)        Y                         
VERSION NUMBER         Y                         
FLAGS   NUMBER         Y                         
C1      VARCHAR2(30)   Y                         
C2      VARCHAR2(30)   Y                         
C3      VARCHAR2(30)   Y                         
C4      VARCHAR2(30)   Y                         
C5      VARCHAR2(30)   Y                         
N1      NUMBER         Y                         
N2      NUMBER         Y                         
N3      NUMBER         Y                         
N4      NUMBER         Y                         
N5      NUMBER         Y                         
N6      NUMBER         Y                         
N7      NUMBER         Y                         
N8      NUMBER         Y                         
N9      NUMBER         Y                         
N10     NUMBER         Y                         
N11     NUMBER         Y                         
N12     NUMBER         Y                         
D1      DATE           Y                         
R1      RAW(32)        Y                         
R2      RAW(32)        Y                         
CH1     VARCHAR2(1000) Y                         
CL1     CLOB           Y        

```

### 查看统计信息
```sql

SQL> select t.owner,
  2         t.table_name,
  3         t.object_type,
  4         t.avg_row_len,
  5         t.num_rows,
  6         t.blocks
  7    from dba_tab_statistics t
  8   where t.table_name = 'DAO_OBJECTS' ;

OWNER                          TABLE_NAME                     OBJECT_TYPE  AVG_ROW_LEN   NUM_ROWS     BLOCKS
------------------------------ ------------------------------ ------------ ----------- ---------- ----------
DAO                            DAO_OBJECTS                    TABLE                200          1          1
 
```

### 将统计信息保存到统计信息表中
```sql
SQL> begin
  2  dbms_stats.export_table_stats(ownname => 'DAO',tabname => 'DAO_OBJECTS',stattab => 'DAO_STATS') ;
  3  end ;
  4  /

PL/SQL procedure successfully completed


```


### 查看统计信息表数据

```sql

SQL> select *  from dao.DAO_STATS ;

STATID TYPE    VERSION      FLAGS C1           C2  C3  C4                             C5                                     N1         N2         N3         N4         N5         N6         N7         N8         N9        N10        N11        N12 D1          R1                                                               R2                                                               CH1                                                                              CL1
------ ---- ---------- ---------- ------------ --- --- ------------------------------ ---- ---------- ---------- ---------- ---------- ---------- ---------- ---------- ---------- ---------- ---------- ---------- ---------- ----------- ---------------------------------------------------------------- ---------------------------------------------------------------- -------------------------------------------------------------------------------- --------------------------------------------------------------------------------
       T             6          3 DAO_OBJECTS                                         DAO           1          1        200      75583                                                      0                                  2017/2/5 21                                                                                                                                                                                                                    
       I             6          2 DAO_STATS_PK                                        DAO       75583        157      75583          1          1       1208          1      75583                                             2017/2/5 17                                                                                                                                                                                                                    
       C             6          2 DAO_OBJECTS          OWNER                          DAO          32    0.03125         32      75583          0 3.38863391 4.58306556          6                                             2017/2/5 17 4143414E                                                         584442                                                                                                                                            
       C             6          2 DAO_OBJECTS          OBJECT_NAME                    DAO       46460 2.15238915      46460      75583          0 2.45035608 6.29634626         25                                             2017/2/5 17 2F31303030333233645F44656C6567617465496E766F636174696F6E4861     794362437253756253616D706C696E67547970653232395F54                                                                                                
       C             6          2 DAO_OBJECTS          SUBOBJECT_NAME                 DAO         191 0.00523560        191        624      74959 1.88673574 4.53398699          2                                             2017/2/5 17 2456534E5F31                                                     575248245F5741495453545F313335323532313736325F31303733                                                                                            
       C             6          2 DAO_OBJECTS          OBJECT_ID                      DAO       75583 1.32304883      75583      75583          0          2      81900          5                                             2017/2/5 17 C103                                                             C30914                                                                                                                                            
       C             6          2 DAO_OBJECTS          DATA_OBJECT_ID                 DAO        9849 0.00010153       9849       9899      65684          0      81900          2                                             2017/2/5 17 80                                                               C30914                                                                                                                                            
       C             6          2 DAO_OBJECTS          OBJECT_TYPE                    DAO          45 0.02222222         45      75583          0 3.49432112 4.58489900          9                                             2017/2/5 17 434C5553544552                                                   584D4C20534348454D41                                                                                                                              
       C             6          2 DAO_OBJECTS          CREATED                        DAO        1315 0.00076045       1315      75583          0 2455822.40 2457790.71          8                                             2017/2/5 17 786F09110A2F0E                                                   78750205120E0C                                                                                                                                    
       C             6          2 DAO_OBJECTS          LAST_DDL_TIME                  DAO        1378 0.00072568       1378      75583          0 2452549.52 2457790.71          8                                             2017/2/5 17 78660A010D2A32                                                   78750205120E0C                                                                                                                                    
       C             6          2 DAO_OBJECTS          TIMESTAMP                      DAO        1451 0.00068917       1451      75583          0 2.55583174 2.60592297         20                                             2017/2/5 17 313939302D30382D32363A31313A32353A3030                           323031372D30322D30353A31373A31333A3131                                                                                                            
       C             6          2 DAO_OBJECTS          STATUS                         DAO           2        0.5          2      75583          0 3.80626532 4.47861930          7                                             2017/2/5 17 494E56414C4944                                                   56414C4944                                                                                                                                        
       C             6          2 DAO_OBJECTS          TEMPORARY                      DAO           2        0.5          2      75583          0 4.04999154 4.62114420          2                                             2017/2/5 17 4E                                                               59                                                                                                                                                
       C             6          2 DAO_OBJECTS          GENERATED                      DAO           2        0.5          2      75583          0 4.04999154 4.62114420          2                                             2017/2/5 17 4E                                                               59                                                                                                                                                
       C             6          2 DAO_OBJECTS          SECONDARY                      DAO           2        0.5          2      75583          0 4.04999154 4.62114420          2                                             2017/2/5 17 4E                                                               59                                                                                                                                                
       C             6          2 DAO_OBJECTS          NAMESPACE                      DAO          21 0.04761904         21      75583          0          1         64          3                                             2017/2/5 17 C102                                                             C141                                                                                                                                              
       C             6          2 DAO_OBJECTS          EDITION_NAME                   DAO           0          0          0                 75583          0          0          0                                             2017/2/5 17                                                                                                                                                                                                                    

17 rows selected

```



### 重新收集表统计信息

```sql
SQL> begin
  2   dbms_stats.gather_table_stats(ownname => 'DAO',tabname => 'DAO_OBJECTS');
  3  END;
  4  /

PL/SQL procedure successfully completed

```
###  查看统计信息

```sql
                                                                   
SQL> select t.owner,                                               
  2         t.table_name,                                          
  3         t.object_type,                                         
  4         t.avg_row_len,                                         
  5         t.num_rows,                                            
  6         t.blocks                                               
  7    from dba_tab_statistics t                                   
  8   where t.table_name = 'DAO_OBJECTS' ;                         
                                                                   
OWNER  TABLE_NAME    OBJECT_TYPE  AVG_ROW_LEN   NUM_ROWS     BLOCKS
------ ------------- ------------ ----------- ---------- ----------
DAO    DAO_OBJECTS   TABLE                 97      75583       1102

```

### 导入之前备份的统计信息
```sql

SQL> begin
  2  dbms_stats.import_table_stats(ownname => 'DAO',tabname => 'DAO_OBJECTS',stattab => 'DAO_STATS') ;
  3  end ;
  4  /

PL/SQL procedure successfully completed

```

###  查看统计信息
```sql

                                                                   
SQL> select t.owner,                                               
  2         t.table_name,                                          
  3         t.object_type,                                         
  4         t.avg_row_len,                                         
  5         t.num_rows,                                            
  6         t.blocks                                               
  7    from dba_tab_statistics t                                   
  8   where t.table_name = 'DAO_OBJECTS' ;                         
                                                                   
OWNER   TABLE_NAME   OBJECT_TYPE  AVG_ROW_LEN   NUM_ROWS     BLOCKS
------- ------------ ------------ ----------- ---------- ----------
DAO     DAO_OBJECTS  TABLE                200          1          1

```


# 特殊的统计信息

## 系统统计信息

```sql

SQL>  begin
  2   dbms_stats.gather_system_stats;
  3   end ;
  4  /

PL/SQL procedure successfully completed
  
```

## 动态性能视图统计信息

```sql
SQL>  begin
  2   dbms_stats.gather_fixed_objects_stats;
  3   end ;
  4  /

PL/SQL procedure successfully completed

```

## 数据字典统计信息

```sql
SQL> begin
  2   dbms_stats.gather_dictionary_stats;
  3   end ;
  4  /

PL/SQL procedure successfully completed

```
