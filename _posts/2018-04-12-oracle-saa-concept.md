---
layout: post
title:  使用SQL Access Advisor优化访问路径
date: 2018-04-12
categories: blog
tags: [Oracle]
description: 使用SQL Access Advisor优化访问路径
---


# SQL Access Advisor 概述

SQL Access Advisor 可以推荐合适的物化视图,物化视图日志,分区,索引等设置,以帮助我们实现性能目标.

SQL Access Advisor 

![image](http://wx2.sinaimg.cn/mw690/006Pv9mtly1ftfym78wdij30ic0bwwh5.jpg)




# SQL Access Advisor 使用流程图如下


![image](http://wx4.sinaimg.cn/mw690/006Pv9mtly1ftfymcbme2j30sk0f840i.jpg)




# 使用范例


## 执行待分析SQL

```sql

declare
V_CNT NUMBER ;
BEGIN
FOR I IN 1..100
LOOP
  SELECT COUNT(*) INTO V_CNT FROM DAO_OBJECTS WHERE OBJECT_ID=1000;
END LOOP ;
END ;


```

## 完整脚本

```sql

DECLARE

V_SQL_SET_NAME  VARCHAR2(30) := 'STS_'||TO_CHAR(SYSDATE,'YYYY-MM-DD_HH24:MI:SS') ;
V_WORKLOAD_NAME VARCHAR2(30) := 'SWL_'||TO_CHAR(SYSDATE,'YYYY-MM-DD_HH24:MI:SS') ;
SPA_cur DBMS_SQLTUNE.SQLSET_CURSOR;

v_saved_rows number ;
v_failed_rows number ;
   
V_TASk_NAME varchar2(30) := 'SPA_TASK'||TO_CHAR(SYSDATE,'YYYY-MM-DD_HH24:MI:SS') ; 
V_Task_id   number :=NULL ;   
begin

--创建SQL tunig set
    dbms_sqltune.create_sqlset(sqlset_name => V_SQL_SET_NAME) ;
    DBMS_OUTPUT.put_line('STS_NAME'||V_SQL_SET_NAME) ;

--注意:这里装载有多种方式
-- 向SQL TUNING SET 中装载SQL 
  OPEN SPA_cur FOR
     SELECT VALUE(P)
       FROM table(DBMS_SQLTUNE.SELECT_CURSOR_CACHE('parsing_schema_name =''DAO'' AND elapsed_time > 0',
                                                   NULL,
                                                   NULL,
                                                   NULL,
                                                   NULL,
                                                   1,
                                                   NULL,
                                                   'ALL')) P;
   dbms_sqltune.load_sqlset(sqlset_name     => V_SQL_SET_NAME,
                            populate_cursor => SPA_cur);
 
  
 DBMS_ADVISOR.CREATE_SQLWKLD(workload_name => V_WORKLOAD_NAME) ;
 DBMS_OUTPUT.put_line(V_WORKLOAD_NAME);
 
-- 将SQL TUNING SET中的SQL语句装载到 SQL WORKLOAD中
 DBMS_ADVISOR.IMPORT_SQLWKLD_STS (workload_name => v_workload_name, 
                                 sts_owner => 'DAO',
                                 sts_name => V_SQL_SET_NAME,
                                 saved_rows =>  v_saved_rows,
                                 failed_rows =>  v_failed_rows) ;
dbms_output.put_line('saved rows :'||v_saved_rows) ;                                

dbms_output.put_line('failed rows :'||v_failed_rows) ; 


--创建SQL ACCESS ADVISOR任务
DBMS_ADVISOR.CREATE_TASK(advisor_name => 'SQL Access Advisor',
                        task_name =>V_TASK_NAME,
                        task_id =>V_Task_id ) ;
DBMS_OUTPUT.put_line(V_TASK_NAME);
--将 SQL WORKLOAD 与 SQL ACCESS ADVISOR任务关联
 DBMS_ADVISOR.ADD_SQLWKLD_REF(task_name => v_task_name,
                              workload_name => V_WORKLOAD_NAME);

--执行任务
 dbms_advisor.execute_task(task_name => v_task_name) ; 
 
end ;


``` 


## 查看执行日志

```sql

SQL> select x.owner,x.task_id,x.task_name
  2   from  dba_advisor_log  x
  3   order by x.execution_start desc
  4  ;

OWNER                             TASK_ID TASK_NAME
------------------------------ ---------- ------------------------------
SYS                                  1275 ADDM:1352521762_1_993
DAO                                  1274 SPA_TASK2017-01-23_20:31:38
SYS                                  1271 ADDM:1352521762_1_992
DAO                                  1261 test_sql_access_taskqq

```


## 编写脚本展现结果

下面提供了Oracle官方的过程可以打印建议

由于本人不是特别喜欢在数据库中建立对象,所以特别改写下.
注意,个人觉得此部分的文档有问题,attr5 虽然已经声明,但是没有进行fetch into ;

改写前

```sql

CREATE OR REPLACE PROCEDURE show_recm (in_task_name IN VARCHAR2) IS 
CURSOR curs IS
  SELECT DISTINCT action_id, command, attr1, attr2, attr3, attr4
  FROM user_advisor_actions
  WHERE task_name = in_task_name
  ORDER BY action_id;
  v_action        number;
  v_command     VARCHAR2(32);
  v_attr1       VARCHAR2(4000);
  v_attr2       VARCHAR2(4000);
  v_attr3       VARCHAR2(4000);
  v_attr4       VARCHAR2(4000);
  v_attr5       VARCHAR2(4000);
BEGIN
  OPEN curs;
  DBMS_OUTPUT.PUT_LINE('=========================================');
  DBMS_OUTPUT.PUT_LINE('Task_name = ' || in_task_name);
  LOOP
     FETCH curs INTO  
       v_action, v_command, v_attr1, v_attr2, v_attr3, v_attr4 ;
   EXIT when curs%NOTFOUND;
   DBMS_OUTPUT.PUT_LINE('Action ID: ' || v_action);
   DBMS_OUTPUT.PUT_LINE('Command : ' || v_command);
   DBMS_OUTPUT.PUT_LINE('Attr1 (name)      : ' || SUBSTR(v_attr1,1,30));
   DBMS_OUTPUT.PUT_LINE('Attr2 (tablespace): ' || SUBSTR(v_attr2,1,30));
   DBMS_OUTPUT.PUT_LINE('Attr3             : ' || SUBSTR(v_attr3,1,30));
   DBMS_OUTPUT.PUT_LINE('Attr4             : ' || v_attr4);
   DBMS_OUTPUT.PUT_LINE('Attr5             : ' || v_attr5);
   DBMS_OUTPUT.PUT_LINE('----------------------------------------');  
   END LOOP;   
   CLOSE curs;      
   DBMS_OUTPUT.PUT_LINE('=========END RECOMMENDATIONS============');
END show_recm;
/

```

改写后

```sql
SQL> set serveroutput on size unlimit 
SQL> declare
  2  v_task_name  varchar2(30) :='SPA_TASK2017-01-23_20:31:38';
  3  begin
  4    DBMS_OUTPUT.PUT_LINE('=========================================');
  5    DBMS_OUTPUT.PUT_LINE('Task_name = ' || v_task_name);
  6    for r in (
  7    SELECT DISTINCT action_id, command, attr1, attr2, attr3, attr4,to_char(attr5) attr5
  8    FROM user_advisor_actions
  9    WHERE task_name = v_task_name
 10    ORDER BY action_id)
 11    LOOP
 12     DBMS_OUTPUT.PUT_LINE('Action ID: ' || r.action_id);
 13     DBMS_OUTPUT.PUT_LINE('Command : ' || r.command);
 14     DBMS_OUTPUT.PUT_LINE('Attr1 (name)      : ' || r.attr1);
 15     DBMS_OUTPUT.PUT_LINE('Attr2 (tablespace): ' || r.attr2);
 16     DBMS_OUTPUT.PUT_LINE('Attr3             : ' || r.attr3);
 17     DBMS_OUTPUT.PUT_LINE('Attr4             : ' || r.attr4);
 18     DBMS_OUTPUT.PUT_LINE('Attr5             : ' || r.attr5);
 19     DBMS_OUTPUT.PUT_LINE('----------------------------------------');  
 20     END LOOP;   
 21     DBMS_OUTPUT.PUT_LINE('=========END RECOMMENDATIONS============');
 22  END ;
 23  /
=========================================
Task_name = SPA_TASK2017-01-23_20:31:38
Action ID: 1
Command : CREATE INDEX
Attr1 (name)      : "DAO"."DAO_OBJECTS_IDX$$_04FA0000"
Attr2 (tablespace):
Attr3             : "DAO"."DAO_OBJECTS"
Attr4             : BTREE
Attr5             : ("OBJECT_ID")
----------------------------------------
=========END RECOMMENDATIONS============

PL/SQL procedure successfully completed.


```


## 生成可执行脚本

```sql

SQL> EXECUTE DBMS_ADVISOR.CREATE_FILE(DBMS_ADVISOR.GET_TASK_SCRIPT('SPA_TASK2017-01-23_20:31:38'), 'ADVISOR_RESULTS', 'advscript.sql');

PL/SQL procedure successfully completed.

SQL> ! cat advscript.sql
cat: advscript.sql: No such file or directory

SQL> ! cat /tmp/advscript.sql
Rem  SQL Access Advisor: Version 11.2.0.3.0 - Production
Rem  
Rem  Username:        DAO
Rem  Task:            SPA_TASK2017-01-23_20:31:38
Rem  Execution date:   
Rem  

--这里就是脚本的内容
CREATE INDEX "DAO"."DAO_OBJECTS_IDX$$_04FA0000"
    ON "DAO"."DAO_OBJECTS"
    ("OBJECT_ID")
    COMPUTE STATISTICS;
```

# 对于单条SQL 使用QUICK_TUNE

## 调用DBMS_ADVISOR.QUICK_TUNE优化语句
```sql

SQL> begin
 DBMS_ADVISOR.QUICK_TUNE(DBMS_ADVISOR.SQLACCESS_ADVISOR,
  2    3                                                    'sql_qucik_tune',
  4                                                    'select object_name from dao_objects where object_id =100');
  5  end ;
  6  /

PL/SQL procedure successfully completed.

```

## 查看任务历史


```sql

SQL> select x.task_id,x.task_name,x.execution_start,x.execution_end
  2    from user_advisor_log   x
  3    where task_name='sql_qucik_tune';

   TASK_ID TASK_NAME                      EXECUTION EXECUTION
---------- ------------------------------ --------- ---------
      1277 sql_qucik_tune                 23-JAN-17 23-JAN-17
```


### 输出建议


```sql

SQL> select x.task_id,x.task_name,x.execution_start,x.execution_end
  2    from user_advisor_log   x
  3    where task_name='sql_qucik_tune';

   TASK_ID TASK_NAME                      EXECUTION EXECUTION
---------- ------------------------------ --------- ---------
      1277 sql_qucik_tune                 23-JAN-17 23-JAN-17

SQL> declare
  2  v_task_name  varchar2(30) :='sql_qucik_tune';
  3  begin
  4    DBMS_OUTPUT.PUT_LINE('=========================================');
  5    DBMS_OUTPUT.PUT_LINE('Task_name = ' || v_task_name);
  6    for r in (
  7    SELECT DISTINCT action_id, command, attr1, attr2, attr3, attr4,to_char(attr5) attr5
  8    FROM user_advisor_actions
  9    WHERE task_name = v_task_name
 10    ORDER BY action_id)
 11    LOOP
 12     DBMS_OUTPUT.PUT_LINE('Action ID: ' || r.action_id);
 13     DBMS_OUTPUT.PUT_LINE('Command : ' || r.command);
 14     DBMS_OUTPUT.PUT_LINE('Attr1 (name)      : ' || r.attr1);
 15     DBMS_OUTPUT.PUT_LINE('Attr2 (tablespace): ' || r.attr2);
 16     DBMS_OUTPUT.PUT_LINE('Attr3             : ' || r.attr3);
 17     DBMS_OUTPUT.PUT_LINE('Attr4             : ' || r.attr4);
 18     DBMS_OUTPUT.PUT_LINE('Attr5             : ' || r.attr5);
 19     DBMS_OUTPUT.PUT_LINE('----------------------------------------');  
 20     END LOOP;   
 21     DBMS_OUTPUT.PUT_LINE('=========END RECOMMENDATIONS============');
 22  END ;
 23  /
 
 --以下是数据结果
 --大家可以看到其建议与我们想象中的理想处理方式还是有一定差距的
=========================================
Task_name = sql_qucik_tune
Action ID: 1
Command : CREATE MATERIALIZED VIEW LOG
Attr1 (name)      : "DAO"."DAO_OBJECTS"
Attr2 (tablespace):
Attr3             : ROWID
Attr4             :
Attr5             :
----------------------------------------
Action ID: 3
Command : CREATE MATERIALIZED VIEW
Attr1 (name)      : "DAO"."MV$$_04FD0000"
Attr2 (tablespace):
Attr3             : REFRESH FAST WITH ROWID
Attr4             : ENABLE QUERY REWRITE
Attr5             : SELECT "DAO"."DAO_OBJECTS"."OBJECT_NAME" M1 FROM DAO.DAO_OBJECTS WHERE (DAO.DAO_OBJECTS.OBJECT_ID = 100)
----------------------------------------
Action ID: 4
Command : GATHER TABLE STATISTICS
Attr1 (name)      : "DAO"."MV$$_04FD0000"
Attr2 (tablespace):
Attr3             : -1
Attr4             :
Attr5             :
----------------------------------------
=========END RECOMMENDATIONS============

```

## 生成并查看建议脚本

```sql
SQL> begin
  2   DBMS_ADVISOR.CREATE_FILE(DBMS_ADVISOR.GET_TASK_SCRIPT('sql_qucik_tune'),
  3                            'ADVISOR_RESULTS', 
  4                            'sql_quick.sql');  
  5  
  6  end ; 
  7  /

SQL> ! cat /tmp/sql_quick.sql
Rem  SQL Access Advisor: Version 11.2.0.3.0 - Production
Rem  
Rem  Username:        DAO
Rem  Task:            sql_qucik_tune
Rem  Execution date:   
Rem  

CREATE MATERIALIZED VIEW LOG ON
    "DAO"."DAO_OBJECTS"
    WITH ROWID ;

CREATE MATERIALIZED VIEW "DAO"."MV$$_04FD0000"
    REFRESH FAST WITH ROWID
    ENABLE QUERY REWRITE
    AS SELECT "DAO"."DAO_OBJECTS"."OBJECT_NAME" M1 FROM DAO.DAO_OBJECTS WHERE (DAO.DAO_OBJECTS.OBJECT_ID
       = 100);

begin
  dbms_stats.gather_table_stats('"DAO"','"MV$$_04FD0000"',NULL,dbms_stats.auto_sample_size);
end;
/
  
```