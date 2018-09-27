---
layout: post
title:  使用Oracle STA自动优化SQL
date: 2018-03-06
categories: blog
tags: [Oracle]
description: 使用Oracle STA自动优化SQL
---

# SQL TUNING ADVISOR 简介与架构

STA(SQL Tuning Advisor )是Oracle提供的sql诊断工具.用户可以提交一条或多条SQL 语句到STA,STA会给出建议.

![image](http://wx3.sinaimg.cn/mw690/006Pv9mtly1ftcwv8vx1wj30dr0huq53.jpg)



如图所示可以从ADDM AWR SHARED POOL与SQL优化集找到需要优化的SQL语句,然后将这些语句传入到STA中,STA 生成执行SQL优化建议.

SQL TUNING ADVISOR 的输出通常包含以下几个方面

1. 执行计划的建议
2. 优化的依据
3. 估算的性能提升比=(优化前执行时间-优化后执行时间)/优化前执行时间
4. SQL语句应用建议


STA生成优化建议最核心的部分由Automatic Tuning Optimizer来完成.



## 自动调整优化器 (Automatic Tuning Optimizer )

自动调整优化器主要的工作内容包含以下几个方面

## 统计信息分析 (Statistical Analysis)

优化器在生成执行计划时依赖于统计信息.如果这些统计信息过期或者缺失,优化器就可能非最优的执行计划.
而 ATO会检查SQL语句引用对象的统计信息是否缺失或过期.

![image](http://wx2.sinaimg.cn/mw690/006Pv9mtly1ftcwx6r165j309e0bq766.jpg)



## SQL Profiling

SQL profile 是ATO对其自己估算结果的验证集.
其实就是一系列的提示用以固定执行计划.
在下面的图片中我们能清楚的看到 
当用户向STA中提交SQL语句,自动调整优化器 (Automatic Tuning Optimizer )生成SQL profile.
如果SQL profile被使用,则可以获取优化后的执行计划. 在整个过程中,应用代码都不会被改变.

![image](http://wx1.sinaimg.cn/mw690/006Pv9mtly1ftcxl1iwgrj30gw0d6di7.jpg)


SQL profile只支持 SELECT ,INSERT SELECT ,UPDATE ,DELETE,MERGE , CREATE TABLE AS SELECT  . 在评估后STA会作出决定是否推荐使用SQL PROFILE .

![image](http://wx1.sinaimg.cn/mw690/006Pv9mtly1ftcxmurxuej30c60blac2.jpg)


如果优化器在统计信息分析或计算SQL profile 是生成了辅助信息,则优化器会推荐使用SQL Pfile.
如果STA是手工调用,则用户需要选择是否使用SQL profile.

如果配置STA自动任务,则是否自动应用SQLprofile,取决于优化任务的参数ACCEPT_SQL_PROFILE试着的值.

如果为 true,则自动应用SQL profile.
如果为flase,则需要用户干预.
如果为 auto,则如果sql 语句存在一个sql profile该值取为true.
反之则为flase ,直到满足前面的条件.

![image](http://wx4.sinaimg.cn/mw690/006Pv9mtly1ftcxoms5g0j30hq05w3yn.jpg)




## 访问路径分析 (Access Path Analysis)

访问路径即数据库如何获取数据.

在OLTP中,我们通常希望使用索引来减少全表扫描.这也通常被认为比较高效.

Automatic Tuning Optimizer 将会探查是否新建一个索引将会提高查询的性能,
并会推荐以下两个操作

### 创建索引

### 运行SQL Access Advisor 

![image](http://wx3.sinaimg.cn/mw690/006Pv9mtly1ftcx0eu2irj30ar0bqtav.jpg)

## SQL 结构分析(SQL Structural Analysis)

![image](http://wx3.sinaimg.cn/mw690/006Pv9mtly1ftcx2uo4cvj30an09xjtc.jpg)


在SQL 结构分析中, Automatic Tuning Optimize 会识别语法,语义,或设计问题等可以引起不理想执行计划的因素.目的是找出写法低效的SQL,并建议重构他们.

 Automatic Tuning Optimize丛以下方面查找结构问题
 
### SQL 结构低效

 使用union 而不是union all
 使用not in 而不是弄通exists
 
### 数据类型不匹配

数据类型不匹配导致不能走索引.

### 设计失误

典型的为关联谓词缺失
 




## 替代计划分析 (Alternative Plan Analysis)

当调整SQL语句时,STA会为替代执行计划查找当前与历史的性能数据.
如果发现其他执行计划,则会报告找到替代执行计划.




![image](http://wx4.sinaimg.cn/mw690/006Pv9mtly1ftcx4q1dexj30dq0c3dhw.jpg)


# SQL TUNING ADVISOR 调用方式

## 自动调用

关于自动任务我单独的写了一片文档.
这里简单说下下面的图片

```sql
使用 DBMS_AUTO_TASK_ADMIN.ENABLE  启用自动SQL调整.

使用DBMS_SQLTUNE.SET_TUNING_TASK_PARAMETER 修改参数

使用 DBMS_SQLTUNE.REPORT_AUTO_TUNING_TASK来报告任务.

使用 DBMS_AUTO_TASK_ADMIN.DISABLE  禁用自动SQL调整.

```


![image](http://wx3.sinaimg.cn/mw690/006Pv9mtly1ftcx79mpxwj30dw0atglr.jpg)

## 手动调用

```sql
 
  创建SQL tuning set (STS)
  
  创建优化任务
  
  使用set_tuning_task_parameter修改参数[可选操作]
  
  执行优化任务
  
  监视优化任务[可选]
  
  查看报告
  
  应用建议
  
```

![image](http://wx1.sinaimg.cn/mw690/006Pv9mtly1ftcx9nc0c5j30dl0du74j.jpg)

# SQL TUNING ADVISOR 使用(手工)

关于STA的自动调用,在<<Oracle 11g 自带系统任务研究>>中已经解释的很清楚.这里我们来介绍下STA的自动调用

## 创建任务





### 单条SQL 

```sql

--sql文本模式
 FUNCTION create_tuning_task(              
    sql_text    IN CLOB,  	                              --sql文本                
    bind_list   IN sql_binds := NULL,                     --绑定变量列表
    user_name   IN VARCHAR2  := NULL,                     --调用用户名称
    scope       IN VARCHAR2  := SCOPE_COMPREHENSIVE,      --调整范围    
    time_limit  IN NUMBER    := TIME_LIMIT_DEFAULT,       --运行时长限制
    task_name   IN VARCHAR2  := NULL,                     --任务名称    
    description IN VARCHAR2  := NULL)                     --描述        
  RETURN VARCHAR2;
  
--sqlid模式  
 FUNCTION create_tuning_task(
    sql_id          IN VARCHAR2,                         --sqlid         
    plan_hash_value IN NUMBER   := NULL,                 --执行计划哈希值
    scope           IN VARCHAR2 := SCOPE_COMPREHENSIVE,  --调整范围      
    time_limit      IN NUMBER   := TIME_LIMIT_DEFAULT,   --运行时长限制  
    task_name       IN VARCHAR2 := NULL,                 --任务名称      
    description     IN VARCHAR2 := NULL)                 --描述          
  RETURN VARCHAR2;
  
 
-- AWR模式
 FUNCTION create_tuning_task(                        
    begin_snap      IN NUMBER,                           --awr起始快照编号   
    end_snap        IN NUMBER,                           --awr结束快照编号 
    sql_id          IN VARCHAR2,                         --sql_id          
    plan_hash_value IN NUMBER   := NULL,                 --执行计划哈希值  
    scope           IN VARCHAR2 := SCOPE_COMPREHENSIVE,  --调整范围        
    time_limit      IN NUMBER   := TIME_LIMIT_DEFAULT,   --运行时长限制    
    task_name       IN VARCHAR2 := NULL,                 --任务名称        
    description     IN VARCHAR2 := NULL)                 --描述            
  RETURN VARCHAR2;
  
 ```
### sql调整集
 
```sql

--sql 调优集合模式  
FUNCTION create_tuning_task(
    sqlset_name       IN VARCHAR2,                         --sql集合的名字             
    basic_filter      IN VARCHAR2 :=  NULL,                --基本过滤器可以过滤语句类型
    object_filter     IN VARCHAR2 :=  NULL,                --对象过滤器假的暂不支持    
    rank1             IN VARCHAR2 :=  NULL,                --排序规则1                 
    rank2             IN VARCHAR2 :=  NULL,                --排序规则2                 
    rank3             IN VARCHAR2 :=  NULL,                --排序规则3                 
    result_percentage IN NUMBER   :=  NULL,                --排名和的百分比            
    result_limit      IN NUMBER   :=  NULL,                --SQL结果限制               
    scope             IN VARCHAR2 :=  SCOPE_COMPREHENSIVE, --调整范围                  
    time_limit        IN NUMBER   :=  TIME_LIMIT_DEFAULT,  --运行时长限制              
    task_name         IN VARCHAR2 :=  NULL,                --任务名称                  
    description       IN VARCHAR2 :=  NULL,                --描述                      
    plan_filter       IN VARCHAR2 :=  'MAX_ELAPSED_TIME',  --执行计划过滤器            
    sqlset_owner      IN VARCHAR2 :=  NULL)                --sql调整集名称             
  RETURN VARCHAR2;

```
### SPA
```sql
--  SPA模式

   FUNCTION create_tuning_task(
    spa_task_name     IN VARCHAR2,                         --spa任务名称     
    spa_task_owner    IN VARCHAR2 :=  NULL,                --spa集合所有者   
    spa_compare_exec  IN VARCHAR2 :=  NULL,                --SPASQL性能比较名
    basic_filter      IN VARCHAR2 :=  NULL,                --基本过滤器      
    time_limit        IN NUMBER   :=  TIME_LIMIT_DEFAULT,  --运行时长限制    
    task_name         IN VARCHAR2 :=  NULL,                --任务名称        
    description       IN VARCHAR2 :=  NULL)                --描述            
  RETURN VARCHAR2;      
  
```


## 执行任务

```sql

FUNCTION execute_tuning_task(
     task_name        IN VARCHAR2,                          --任务名称
     execution_name   IN VARCHAR2               := NULL,    --执行名称
     execution_params IN dbms_advisor.argList   := NULL,    --执行参数
     execution_desc   IN VARCHAR2               := NULL)    --执行描述
  RETURN VARCHAR2;                                          --返回值为执行名称

PROCEDURE execute_tuning_task(
     task_name        IN VARCHAR2,                         --任务名称            
     execution_name   IN VARCHAR2               := NULL,   --执行名称             
     execution_params IN dbms_advisor.argList   := NULL,   --执行参数            
     execution_desc   IN VARCHAR2               := NULL);  --执行描述            

```


## 监视任务

```sql

SELECT STATUS 
FROM   USER_ADVISOR_TASKS
WHERE  TASK_NAME = ;:




SELECT TASK_ID, ADVISOR_NAME, SOFAR, TOTALWORK, 
       ROUND(SOFAR/TOTALWORK*100,2) "%_COMPLETE"
FROM   V$ADVISOR_PROGRESS
WHERE  TASK_ID = :my_tid;


```

## 查看任务结果


```sql


SET LONG 1000
SET LONGCHUNKSIZE 1000
SET LINESIZE 100
SELECT DBMS_SQLTUNE.REPORT_TUNING_TASK( 'STA_SPECIFIC_EMP_TASK' )
FROM   DUAL;

PRINT :my_rept

```


# SQL TUNING ADVISOR 使用范例 

## 使用SQL文本创建任务

### 创建实验表
 ```sql
 
 SQL> CREATE TABLE DAO_OBJECTS AS SELECT * FROM DBA_OBJECTS ;

Table created.
 
 ```

### 设置sqlplus 参数

```sql

SQL> set serveroutput on size unlimited 
SQL> set linesize 200
SQL> set long 10000
SET LONGCHUNKSIZE 1000

```

### 创建并执行任务

```sql

SQL> declare
  2  v_task_name  varchar2(30) ;
  3  
  4  begin
  5  
  6    v_task_name := dbms_sqltune.create_tuning_task(sql_text => 'select count(*) from dao_objects where object_name=''Dba_Tables'' and object_id>20');
  7    
  8    dbms_sqltune.execute_tuning_task(v_task_name);
  9    
 10    dbms_output.put_line(v_task_name) ;
 11  end;
 12  /
TASK_1208  --打印出来的任务名称


```

### 查看建议


```sql
SQL> SELECT DBMS_SQLTUNE.REPORT_TUNING_TASK( 'TASK_1208' )
  2  FROM   DUAL;

DBMS_SQLTUNE.REPORT_TUNING_TASK('TASK_1208')
----------------------------------------------------------------------------------------------------
GENERAL INFORMATION SECTION
-------------------------------------------------------------------------------
Tuning Task Name   : TASK_1208
Tuning Task Owner  : DAO
Workload Type      : Single SQL Statement
Scope              : COMPREHENSIVE
Time Limit(seconds): 1800
Completion Status  : COMPLETED
Started at         : 01/21/2017 14:25:04
Completed at       : 01/21/2017 14:25:04

-------------------------------------------------------------------------------
Schema Name: DAO
SQL ID     : 74mna65jdab5c
SQL Text   : select count(*) from dao_objects where object_name='Dba_Tables'
             and object_id>20

-------------------------------------------------------------------------------
FINDINGS SECTION (2 findings)
-------------------------------------------------------------------------------

1- Statistics Finding
---------------------
  Table "DAO"."DAO_OBJECTS" was not analyzed.

  Recommendation
  --------------
  - Consider collecting optimizer statistics for this table.
    execute dbms_stats.gather_table_stats(ownname => 'DAO', tabname =>
            'DAO_OBJECTS', estimate_percent => DBMS_STATS.AUTO_SAMPLE_SIZE,
            method_opt => 'FOR ALL COLUMNS SIZE AUTO');

  Rationale
  ---------
    The optimizer requires up-to-date statistics for the table in order to
    select a good execution plan.

2- Index Finding (see explain plans section below)
--------------------------------------------------
  The execution plan of this statement can be improved by creating one or more
  indices.

  Recommendation (estimated benefit: 99.33%)
  ------------------------------------------
  - Consider running the Access Advisor to improve the physical schema design
    or creating the recommended index.
    create index DAO.IDX$$_04B80001 on DAO.DAO_OBJECTS("OBJECT_NAME","OBJECT_ID
    ");

  Rationale
  ---------
    Creating the recommended indices significantly improves the execution plan
    of this statement. However, it might be preferable to run "Access Advisor"
    using a representative SQL workload as opposed to a single statement. This
    will allow to get comprehensive index recommendations which takes into
    account index maintenance overhead and additional space consumption.

-------------------------------------------------------------------------------
EXPLAIN PLANS SECTION
-------------------------------------------------------------------------------

1- Original
-----------
Plan hash value: 2941237911

----------------------------------------------------------------------------------
| Id  | Operation          | Name        | Rows  | Bytes | Cost (%CPU)| Time     |
----------------------------------------------------------------------------------
|   0 | SELECT STATEMENT   |             |     1 |    79 |   301   (1)| 00:00:04 |
|   1 |  SORT AGGREGATE    |             |     1 |    79 |            |          |
|*  2 |   TABLE ACCESS FULL| DAO_OBJECTS |    43 |  3397 |   301   (1)| 00:00:04 |
----------------------------------------------------------------------------------

Predicate Information (identified by operation id):
---------------------------------------------------

   2 - filter("OBJECT_NAME"='Dba_Tables' AND "OBJECT_ID">20)

2- Using New Indices
--------------------
Plan hash value: 1317834891

------------------------------------------------------------------------------------
| Id  | Operation         | Name           | Rows  | Bytes | Cost (%CPU)| Time     |
------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT  |                |     1 |    79 |     2   (0)| 00:00:01 |
|   1 |  SORT AGGREGATE   |                |     1 |    79 |            |          |
|*  2 |   INDEX RANGE SCAN| IDX$$_04B80001 |     1 |    79 |     2   (0)| 00:00:01 |
------------------------------------------------------------------------------------

Predicate Information (identified by operation id):
---------------------------------------------------

   2 - access("OBJECT_NAME"='Dba_Tables' AND "OBJECT_ID">20 AND "OBJECT_ID"
              IS NOT NULL)
```



## 使用SQ_ID创建任务


### 查找SQL_ID

```sql

SQL> select sql_id,substr(sql_text,1,80)  sub_text from v$sql where sql_text like '%Dba%' ; 

SQL_ID          SUB_TEXT
--------------- --------------------------------------------------------------------------------
80pzvqn0w04py   select sql_id from v$sql where sql_text like '%Dba%'
1ggc4ypqdjdys   select sql_id,substr(sql_text,1,50) from v$sql where sql_text like '%Dba%'
73d4fks18trgr   declare v_task_name  varchar2(30) ;  begin    v_task_name := dbms_sqltune.create
7u0nnap2b9tf3   select * from v$sql where sql_text like '%Dba%'
33h8q2sabq022   select sql_id,substr(sql_text,1,80)  sub_text from v$sql where sql_text like '%D
4hj3m9269y6v1   SELECT /* DS_SVC */ /*+ cursor_sharing_exact dynamic_sampling(0) no_sql_tune no_
1su8zwbpx6fc9   select sql_id,substr(sql_text,1,50)  sub_text from v$sql where sql_text like '%D
0jvnab20bkm59   select sql_id,substr(sql_text,1,100)  sub_text from v$sql where sql_text like '%
8r0c9cz52ankt   SELECT /* DS_SVC */ /*+ cursor_sharing_exact dynamic_sampling(0) no_sql_tune no_
9p3g01zxmaqjd   select * from v$sql where sql_text like 'Dba%'
68cvzz5113yvc   select count(*) from dao_objects where object_name='Dba_Tables'              and

11 rows selected.


```


###  创建并执行任务


```sql

SQL> declare
  2  v_task_name  varchar2(30) ;
  3  
  4  begin
  5  
  6    v_task_name := dbms_sqltune.create_tuning_task(sql_id=>'68cvzz5113yvc');
  7    
  8    dbms_sqltune.execute_tuning_task(v_task_name);
  9    
 10    dbms_output.put_line(v_task_name) ;
 11  end;
 12  /
TASK_1210

PL/SQL procedure successfully completed.


```

### 查看建议

```sql


SQL>  SELECT DBMS_SQLTUNE.REPORT_TUNING_TASK( 'TASK_1210' )
  2      FROM   DUAL;

DBMS_SQLTUNE.REPORT_TUNING_TASK('TASK_1210')
----------------------------------------------------------------------------------------------------
GENERAL INFORMATION SECTION
-------------------------------------------------------------------------------
Tuning Task Name   : TASK_1210
Tuning Task Owner  : DAO
Workload Type      : Single SQL Statement
Scope              : COMPREHENSIVE
Time Limit(seconds): 1800
Completion Status  : COMPLETED
Started at         : 01/21/2017 14:55:41
Completed at       : 01/21/2017 14:55:41

-------------------------------------------------------------------------------
Schema Name: DAO
SQL ID     : 68cvzz5113yvc
SQL Text   : select count(*) from dao_objects where object_name='Dba_Tables'
                          and object_id>20

-------------------------------------------------------------------------------
FINDINGS SECTION (2 findings)
-------------------------------------------------------------------------------

1- Statistics Finding
---------------------
  Table "DAO"."DAO_OBJECTS" was not analyzed.

  Recommendation
  --------------
  - Consider collecting optimizer statistics for this table.
    execute dbms_stats.gather_table_stats(ownname => 'DAO', tabname =>
            'DAO_OBJECTS', estimate_percent => DBMS_STATS.AUTO_SAMPLE_SIZE,
            method_opt => 'FOR ALL COLUMNS SIZE AUTO');

  Rationale
  ---------
    The optimizer requires up-to-date statistics for the table in order to
    select a good execution plan.

2- Index Finding (see explain plans section below)
--------------------------------------------------
  The execution plan of this statement can be improved by creating one or more
  indices.

  Recommendation (estimated benefit: 99.33%)
  ------------------------------------------
  - Consider running the Access Advisor to improve the physical schema design
    or creating the recommended index.
    create index DAO.IDX$$_04BA0001 on DAO.DAO_OBJECTS("OBJECT_NAME","OBJECT_ID
    ");

  Rationale
  ---------
    Creating the recommended indices significantly improves the execution plan
    of this statement. However, it might be preferable to run "Access Advisor"
    using a representative SQL workload as opposed to a single statement. This
    will allow to get comprehensive index recommendations which takes into
    account index maintenance overhead and additional space consumption.

-------------------------------------------------------------------------------
EXPLAIN PLANS SECTION
-------------------------------------------------------------------------------

1- Original
-----------
Plan hash value: 2941237911

----------------------------------------------------------------------------------
| Id  | Operation          | Name        | Rows  | Bytes | Cost (%CPU)| Time     |
----------------------------------------------------------------------------------
|   0 | SELECT STATEMENT   |             |     1 |    79 |   301   (1)| 00:00:04 |
|   1 |  SORT AGGREGATE    |             |     1 |    79 |            |          |
|*  2 |   TABLE ACCESS FULL| DAO_OBJECTS |    43 |  3397 |   301   (1)| 00:00:04 |
----------------------------------------------------------------------------------

Predicate Information (identified by operation id):
---------------------------------------------------

   2 - filter("OBJECT_NAME"='Dba_Tables' AND "OBJECT_ID">20)

2- Using New Indices
--------------------
Plan hash value: 2275035492

------------------------------------------------------------------------------------
| Id  | Operation         | Name           | Rows  | Bytes | Cost (%CPU)| Time     |
------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT  |                |     1 |    79 |     2   (0)| 00:00:01 |
|   1 |  SORT AGGREGATE   |                |     1 |    79 |            |          |
|*  2 |   INDEX RANGE SCAN| IDX$$_04BA0001 |     1 |    79 |     2   (0)| 00:00:01 |
------------------------------------------------------------------------------------

Predicate Information (identified by operation id):
---------------------------------------------------

   2 - access("OBJECT_NAME"='Dba_Tables' AND "OBJECT_ID">20 AND "OBJECT_ID"
              IS NOT NULL)

-------------------------------------------------------------------------------

```

## 使用SQL TUNING SET



### 创建SQL TUNING SET


```sql

SQL> BEGIN
  2    DBMS_SQLTUNE.CREATE_SQLSET (
  3      sqlset_name  => 'DAO_SQL_TUNING_SETS' 
  4  ,   description  => 'CREATED BY DAO' 
  5  );
  6  END;
  7  /

PL/SQL procedure successfully completed.


```

### 查看刚创建的SQL TUNING SET 

```sql

SQL> COLUMN NAME FORMAT a20
SQL> COLUMN COUNT FORMAT 99999
SQL> COLUMN DESCRIPTION FORMAT a30
SQL> 
SQL> SELECT NAME, STATEMENT_COUNT AS "SQLCNT", DESCRIPTION
  2  FROM   USER_SQLSET;

NAME                     SQLCNT DESCRIPTION
-------------------- ---------- ------------------------------
DAO_SQL_TUNING_SETS           0 CREATED BY DAO


```


### 选择SQL填充 SQL TUNING SET


```sql

SQL> SELECT NAME, STATEMENT_COUNT AS "SQLCNT", DESCRIPTION
  2  FROM   USER_SQLSET;

NAME                     SQLCNT DESCRIPTION
-------------------- ---------- ------------------------------
DAO_SQL_TUNING_SETS           0 CREATED BY DAO

SQL> DECLARE
  c_sqlarea_cursor DBMS_SQLTUNE.SQLSET_CURSOR;
  2    3  BEGIN
  4   OPEN c_sqlarea_cursor FOR
  5     SELECT VALUE(p)
  6     FROM   TABLE( 
  7              DBMS_SQLTUNE.SELECT_CURSOR_CACHE(
  8              '  parsing_schema_name = ''DAO'' ')
  9            ) p;
 10  
 11    DBMS_SQLTUNE.LOAD_SQLSET (  
 12      sqlset_name     => 'DAO_SQL_TUNING_SETS'
 13  ,   populate_cursor =>  c_sqlarea_cursor 
 14  );
 15  END;
 16  /

PL/SQL procedure successfully completed.

```

##  查看SQL TUNING SET 内容

```sql

SQL> SELECT SQL_ID, PARSING_SCHEMA_NAME AS "SCH", SQL_TEXT, 
  2         ELAPSED_TIME AS "ELAPSED", BUFFER_GETS
  3  FROM   TABLE( DBMS_SQLTUNE.SELECT_SQLSET( 'DAO_SQL_TUNING_SETS' ) )
  4  where rownum <5;

SQL_ID          SCH SQL_TEXT                          ELAPSED BUFFER_GETS
--------------- --- ------------------------------ ---------- -----------
00429y0zbwuks   DAO select * from DBA_AUTOTASK_JOB     126995       26423
                    _HISTORY

00429y0zbwuks   DAO select * from DBA_AUTOTASK_JOB      61572        7635
                    _HISTORY

01gdk03crk0nj   DAO SELECT /* DS_SVC */ /*+ cursor      87262        5586
                    _sharing_exact dynamic_samplin
                    g(0) no_sql_tune no_monitoring
                     optimizer_features_enable(def
                    ault) */ SUM(C1) FROM (SELECT
                    /*+ qb_name("innerQuery") NO_I
                    NDEX_FFS( "C")  */ 1 AS C1 FRO
                    M "SYS"."COL$" SAMPLE BLOCK(55
                    .4785, 8) SEED(1)  "C" WHERE (
                    DECODE("C"."PROPERTY",0,'NO',D
                    ECODE(BITAND("C"."PROPERTY",32
                    ),32,'YES','NO'))='NO')) inner
                    Query

05a5yc9a8zmr4   DAO select * from dba_scheduler_sc       7115         144
                    hedules



```


### 创建并执行优化任务


```sql

SQL> declare
  v_task_name  varchar2(30) ;
  2    3  begin
  4  
  5    v_task_name :=dbms_sqltune.create_tuning_task(
  6        sqlset_name =>'DAO_SQL_TUNING_SETS',
  7        RANK1=>'BUFFER_GETS' );
  8    DBMS_SQLTUNE.execute_tuning_task(V_TASK_NAME);
  9    
 10    DBMS_OUTPUT.put_line(V_TASK_name);
 11  end ;
 12  /
TASK_1217

PL/SQL procedure successfully completed.

```

###　查看建议


```sql

SQL> select dbms_sqltune.report_tuning_task('TASK_1217') from dual ;

DBMS_SQLTUNE.REPORT_TUNING_TASK('TASK_1217')
----------------------------------------------------------------------------------------------------
GENERAL INFORMATION SECTION
-------------------------------------------------------------------------------
Tuning Task Name                  : TASK_1217
Tuning Task Owner                 : DAO
Workload Type                     : SQL Tuning Set
Scope                             : COMPREHENSIVE
Time Limit(seconds)               : 1800
Completion Status                 : COMPLETED
Started at                        : 01/21/2017 16:10:49
Completed at                      : 01/21/2017 16:14:03
SQL Tuning Set (STS) Name         : DAO_SQL_TUNING_SETS
SQL Tuning Set Owner              : DAO
Number of Statements in the STS   : 252

-------------------------------------------------------------------------------
SUMMARY SECTION
-------------------------------------------------------------------------------
                      Global SQL Tuning Result Statistics
-------------------------------------------------------------------------------
Number of SQLs Analyzed                      : 252
Number of SQLs in the Report                 : 210
Number of SQLs with Findings                 : 208
Number of SQLs with Statistic Findings       : 1
Number of SQLs with Alternative Plan Findings: 6
Number of SQLs with SQL profiles recommended : 28
Number of SQLs with SQL Restructure Findings : 17
Number of SQLs with Errors                   : 2

-------------------------------------------------------------------------------
    SQLs with Findings Ordered by Maximum (Profile/Index) Benefit, Object ID
-------------------------------------------------------------------------------
object ID  SQL ID        statistics profile(benefit) index(benefit) restructure
---------- ------------- ---------- ---------------- -------------- -----------
        33 6ymv3v3b43tgz                      89.73%
         2 gd5xqw4zn0szz                      89.17%
         5 5xpm5pzth53pv                      89.17%
        27 8v0m2s6q2htx8                      89.17%
        31 1g0cvsx700vx8                      84.71%
        32 dqt2z3bcjzx7c                      84.71%
        30 bq6gbf5th5jru                      84.48%
       122 gfg8kummfbttj                      83.95%
        35 drr9cjsdjn4rj                      65.89%
        13 87vm0umarvvr1                      56.08%
         6 d4yfajjkzh632                      56.07%
        21 a8un5qc2zhpy4                      46.91%                          2
        25 cgs1f2h8q7hja                      44.61%
        78 77v6ms19zm72w                      44.01%
       137 9z7amrsj0g4su                      44.01%
         4 13u83qfzhh56a                      43.66%                          1
       229 7pwsy9k04skn4                      40.95%
        81 3wn2yrr214dnq                      13.22%                          1
       124 anwm5xw251vwt                      13.20%                          1
         9 cvryukxb6tnpw                      12.13%
         7 7npm3tnvyf098                      12.12%
         8 d5cfyhna9p0xx                      12.12%
        18 3zdrgftnp0s0h                      12.12%
        19 00429y0zbwuks                      12.12%
        29 8zwa8hydu4665                      12.12%
        41 ck56t3bzfsaqq                      12.12%
        34 av6u1g3pbjfs0                      11.71%                          2
       131 gm5g4r5vdj0s7                    <=10.00%                          1
        73 d3vdgnnm2x4f7                                                      2
        74 cnu8yzf3sy5ku                                                      2
        77 21sh40mz4cdkx                                                      3
       123 4xyqmvt7rrbxv
       143 dstp0jb623k1t                                                      2
       144 3dxx30utp188t                                                      1
       150 4uhk86dbvhuty
       153 dvq5ru9qkbpa0                                                      1
       192 bcckn5s3djyxj                                                      1
       196 fa0hg388g2qsn                                                      2
       198 5bwpadtzgvu00          1
       212 cf8fyu7n9yc27                                                      2
       230 f216nw9uc60mq                                                      1
       231 gg26ykxrfrbpf                                                      4

-------------------------------------------------------------------------------
    Objects with Missing/Stale Statistics (ordered by schema, object, type)
-------------------------------------------------------------------------------
Schema Name                  Object Name                  Type  State   Cascade
---------------------------- ---------------------------- ----- ------- -------
                         SYS WRI$_SQLSET_DEFINITIONS      TABLE STALE   NO

-------------------------------------------------------------------------------
                             Statements with Errors
-------------------------------------------------------------------------------
object ID  SQL ID        Error
---------- ------------- ------------------------------------------------------
        46 2rhpjdn1299sb ORA-06553: PLS-306: wrong number or types of argume...
        93 b6q2gkaw6g45v ORA-00907: missing right parenthesis

-------------------------------------------------------------------------------
DETAILS SECTION
-------------------------------------------------------------------------------
 Statements with Results Ordered by Maximum (Profile/Index) Benefit, Object ID
-------------------------------------------------------------------------------
Object ID  : 33
Schema Name: DAO
SQL ID     : 6ymv3v3b43tgz
SQL Text   : /* SQL Analyze(25,1) */ select object_name, object_type
             from sys.user_objects o
             where o.object_type in ('TABLE', 'VIEW', 'PACKAGE','TYPE',
             'PROCEDURE', 'FUNCTION', 'SEQUENCE')

-------------------------------------------------------------------------------
FINDINGS SECTION (2 findings)
-------------------------------------------------------------------------------

1- SQL Profile Finding (see explain plans section below)
--------------------------------------------------------
  A potentially better execution plan was found for this statement.

  Recommendation (estimated benefit: 89.73%)
  ------------------------------------------
  - Consider accepting the recommended SQL profile.
    execute dbms_sqltune.accept_sql_profile(task_name => 'TASK_1217',
            object_id => 33, task_owner => 'DAO', replace => TRUE);

  Validation results
  ------------------
  The SQL profile was tested by executing both its plan and the original plan
  and measuring their respective execution statistics. A plan may have been
  only partially executed if the other could be run to completion in less time.

                           Original Plan  With SQL Profile  % Improved
                           -------------  ----------------  ----------
  Completion Status:            COMPLETE          COMPLETE
  Elapsed Time (s):             .014438           .000805      94.42 %
  CPU Time (s):                 .014497           .000899      93.79 %
  User I/O Time (s):                  0                 0
  Buffer Gets:                      964                99      89.73 %
  Physical Read Requests:             0                 0
  Physical Write Requests:            0                 0
  Physical Read Bytes:                0                 0
  Physical Write Bytes:               0                 0
  Rows Processed:                    82                82
  Fetches:                           82                82
  Executions:                         1                 1

  Notes
  -----
  1. Statistics for the original plan were averaged over 10 executions.
  2. Statistics for the SQL profile plan were averaged over 10 executions.

2- Alternative Plan Finding
---------------------------
  Some alternative execution plans for this statement were found by searching
  the system's real-time and historical performance data.

  The following table lists these plans ranked by their average elapsed time.
  See section "ALTERNATIVE PLANS SECTION" for detailed information on each
  plan.

  id plan hash  last seen            elapsed (s)  origin          note
  -- ---------- -------------------- ------------ --------------- ----------------
   1 3296348383  2017-01-21/10:56:23        0.001 Cursor Cache
   2 3165931518  2017-01-21/10:56:23        0.001 Cursor Cache
   3 3921040117  2017-01-21/10:56:23        0.001 Cursor Cache
   4 1739778074  2017-01-21/10:56:23        0.001 Cursor Cache
   5 3943719814  2017-01-21/10:56:23        0.015 Cursor Cache    original plan

  Recommendation
  --------------
  - Consider creating a SQL plan baseline for the plan with the best average
    elapsed time.
    execute dbms_sqltune.create_sql_plan_baseline(task_name => 'TASK_1217',
            object_id => 33, owner_name => 'DAO', plan_hash_value =>
            3296348383);

------------------------------------------------------------


```