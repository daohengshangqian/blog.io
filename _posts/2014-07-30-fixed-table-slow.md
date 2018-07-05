---
layout: post
title: SQL优化一例，由于动态性能视图统计信息不准确引起的查询缓慢
date: 2014-07-30
categories: blog
tags: [SQL优化]
description: SQL优化一例，由于动态性能视图统计信息不准确引起的查询缓慢
---

## 某库查询锁相关信息语句如下:
```sql
Execution Plan                                                                                
----------------------------------------------------------                                    
Plan hash value: 914685293                                                                    
                                                                                              
--------------------------------------------------------------------------------------        
| Id  | Operation               | Name       | Rows  | Bytes | Cost (%CPU)| Time     |        
--------------------------------------------------------------------------------------        
|   0 | SELECT STATEMENT        |            |     1 |    89 |     1 (100)| 00:00:01 |        
|*  1 |  FILTER                 |            |       |       |            |          |        
|*  2 |   HASH JOIN             |            |     1 |    89 |     1 (100)| 00:00:01 |        
|   3 |    MERGE JOIN CARTESIAN |            |   100 |  6400 |     0   (0)| 00:00:01 |        
|*  4 |     FIXED TABLE FULL    | X$KSUSE    |     1 |    32 |     0   (0)| 00:00:01 |        
|   5 |     BUFFER SORT         |            |   100 |  3200 |     0   (0)| 00:00:01 |        
|   6 |      FIXED TABLE FULL   | X$KSQRS    |   100 |  3200 |     0   (0)| 00:00:01 |        
|   7 |    VIEW                 | GV$_LOCK   |    10 |   250 |     0   (0)| 00:00:01 |        
|   8 |     UNION-ALL           |            |       |       |            |          |        
|*  9 |      FILTER             |            |       |       |            |          |        
|  10 |       VIEW              | GV$_LOCK1  |     2 |   152 |     0   (0)| 00:00:01 |        
|  11 |        UNION-ALL        |            |       |       |            |          |        
|* 12 |         FIXED TABLE FULL| X$KDNSSF   |     1 |   102 |     0   (0)| 00:00:01 |        
|* 13 |         FIXED TABLE FULL| X$KSQEQ    |     1 |   102 |     0   (0)| 00:00:01 |        
|* 14 |      FIXED TABLE FULL   | X$KTADM    |     1 |   102 |     0   (0)| 00:00:01 |        
|* 15 |      FIXED TABLE FULL   | X$KTATRFIL |     1 |   102 |     0   (0)| 00:00:01 |        
|* 16 |      FIXED TABLE FULL   | X$KTATRFSL |     1 |   102 |     0   (0)| 00:00:01 |        
|* 17 |      FIXED TABLE FULL   | X$KTATL    |     1 |   102 |     0   (0)| 00:00:01 |        
|* 18 |      FIXED TABLE FULL   | X$KTSTUSC  |     1 |   102 |     0   (0)| 00:00:01 |        
|* 19 |      FIXED TABLE FULL   | X$KTSTUSS  |     1 |   102 |     0   (0)| 00:00:01 |        
|* 20 |      FIXED TABLE FULL   | X$KTSTUSG  |     1 |   102 |     0   (0)| 00:00:01 |        
|* 21 |      FIXED TABLE FULL   | X$KTCXB    |     1 |   102 |     0   (0)| 00:00:01 |        
|* 22 |   HASH JOIN             |            |     1 |    63 |     1 (100)| 00:00:01 |        
|  23 |    MERGE JOIN CARTESIAN |            |     1 |    51 |     0   (0)| 00:00:01 |        
|* 24 |     FIXED TABLE FULL    | X$KSUSE    |     1 |    19 |     0   (0)| 00:00:01 |        
|  25 |     BUFFER SORT         |            |     1 |    32 |     0   (0)| 00:00:01 |        
|* 26 |      FIXED TABLE FULL   | X$KSQRS    |     1 |    32 |     0   (0)| 00:00:01 |        
|  27 |    VIEW                 | GV$_LOCK   |    10 |   120 |     0   (0)| 00:00:01 |        
|  28 |     UNION-ALL           |            |       |       |            |          |        
|* 29 |      FILTER             |            |       |       |            |          |        
|  30 |       VIEW              | GV$_LOCK1  |     2 |    24 |     0   (0)| 00:00:01 |        
|  31 |        UNION-ALL        |            |       |       |            |          |        
|* 32 |         FIXED TABLE FULL| X$KDNSSF   |     1 |    77 |     0   (0)| 00:00:01 |        
|* 33 |         FIXED TABLE FULL| X$KSQEQ    |     1 |    77 |     0   (0)| 00:00:01 |        
|* 34 |      FIXED TABLE FULL   | X$KTADM    |     1 |    77 |     0   (0)| 00:00:01 |        
|* 35 |      FIXED TABLE FULL   | X$KTATRFIL |     1 |    77 |     0   (0)| 00:00:01 |        
|* 36 |      FIXED TABLE FULL   | X$KTATRFSL |     1 |    77 |     0   (0)| 00:00:01 |        
|* 37 |      FIXED TABLE FULL   | X$KTATL    |     1 |    77 |     0   (0)| 00:00:01 |        
|* 38 |      FIXED TABLE FULL   | X$KTSTUSC  |     1 |    77 |     0   (0)| 00:00:01 |        
|* 39 |      FIXED TABLE FULL   | X$KTSTUSS  |     1 |    77 |     0   (0)| 00:00:01 |        
|* 40 |      FIXED TABLE FULL   | X$KTSTUSG  |     1 |    77 |     0   (0)| 00:00:01 |        
|* 41 |      FIXED TABLE FULL   | X$KTCXB    |     1 |    77 |     0   (0)| 00:00:01 |        
--------------------------------------------------------------------------------------        
                                                                                              
Predicate Information (identified by operation id):                                           
---------------------------------------------------                                           
                                                                                              
   1 - filter( EXISTS (SELECT 0 FROM SYS."X$KSQRS" "R",SYS."X$KSUSE" "S", (                   
              (SELECT "SADDR" "SADDR","RADDR" "RADDR" FROM  ( (SELECT "KSQLKSES"              
              "SADDR","KSQLKRES" "RADDR" FROM SYS."X$KDNSSF" "X$KDNSSF" WHERE                 
              ("KSQLKMOD"<>0 OR "KSQLKREQ"<>0) AND "INST_ID"=USERENV('INSTANCE') AND          
              DECODE("KSQLKMOD",0,0,"KSQLKLBLK")=1 AND BITAND("KSSOBFLG",1)<>0) UNION ALL     
              (SELECT "KSQLKSES" "SADDR","KSQLKRES" "RADDR" FROM SYS."X$KSQEQ" "X$KSQEQ"      
              WHERE ("KSQLKMOD"<>0 OR "KSQLKREQ"<>0) AND "INST_ID"=USERENV('INSTANCE') AND    
              DECODE("KSQLKMOD",0,0,"KSQLKLBLK")=1 AND BITAND("KSSOBFLG",1)<>0))              
              "GV$_LOCK1" WHERE USERENV('INSTANCE') IS NOT NULL) UNION ALL  (SELECT           
              "KSQLKSES" "SADDR","KSQLKRES" "RADDR" FROM SYS."X$KTADM" "X$KTADM" WHERE        
              ("KSQLKMOD"<>0 OR "KSQLKREQ"<>0) AND DECODE("KSQLKMOD",0,0,"KSQLKLBLK")=1 AND   
              "INST_ID"=USERENV('INSTANCE') AND BITAND("KSSOBFLG",1)<>0) UNION ALL  (SELECT   
              "KSQLKSES" "SADDR","KSQLKRES" "RADDR" FROM SYS."X$KTATRFIL" "X$KTATRFIL"        
              WHERE ("KSQLKMOD"<>0 OR "KSQLKREQ"<>0) AND                                      
              DECODE("KSQLKMOD",0,0,"KSQLKLBLK")=1 AND "INST_ID"=USERENV('INSTANCE') AND      
              BITAND("KSSOBFLG",1)<>0) UNION ALL  (SELECT "KSQLKSES" "SADDR","KSQLKRES"       
              "RADDR" FROM SYS."X$KTATRFSL" "X$KTATRFSL" WHERE ("KSQLKMOD"<>0 OR              
              "KSQLKREQ"<>0) AND DECODE("KSQLKMOD",0,0,"KSQLKLBLK")=1 AND                     
              "INST_ID"=USERENV('INSTANCE') AND BITAND("KSSOBFLG",1)<>0) UNION ALL  (SELECT   
              "KSQLKSES" "SADDR","KSQLKRES" "RADDR" FROM SYS."X$KTATL" "X$KTATL" WHERE        
              ("KSQLKMOD"<>0 OR "KSQLKREQ"<>0) AND DECODE("KSQLKMOD",0,0,"KSQLKLBLK")=1 AND   
              "INST_ID"=USERENV('INSTANCE') AND BITAND("KSSOBFLG",1)<>0) UNION ALL  (SELECT   
              "KSQLKSES" "SADDR","KSQLKRES" "RADDR" FROM SYS."X$KTSTUSC" "X$KTSTUSC" WHERE    
              ("KSQLKMOD"<>0 OR "KSQLKREQ"<>0) AND DECODE("KSQLKMOD",0,0,"KSQLKLBLK")=1 AND   
              "INST_ID"=USERENV('INSTANCE') AND BITAND("KSSOBFLG",1)<>0) UNION ALL  (SELECT   
              "KSQLKSES" "SADDR","KSQLKRES" "RADDR" FROM SYS."X$KTSTUSS" "X$KTSTUSS" WHERE    
              ("KSQLKMOD"<>0 OR "KSQLKREQ"<>0) AND DECODE("KSQLKMOD",0,0,"KSQLKLBLK")=1 AND   
              "INST_ID"=USERENV('INSTANCE') AND BITAND("KSSOBFLG",1)<>0) UNION ALL  (SELECT   
              "KSQLKSES" "SADDR","KSQLKRES" "RADDR" FROM SYS."X$KTSTUSG" "X$KTSTUSG" WHERE    
              ("KSQLKMOD"<>0 OR "KSQLKREQ"<>0) AND DECODE("KSQLKMOD",0,0,"KSQLKLBLK")=1 AND   
              "INST_ID"=USERENV('INSTANCE') AND BITAND("KSSOBFLG",1)<>0) UNION ALL  (SELECT   
              "KSQLKSES" "SADDR","KSQLKRES" "RADDR" FROM SYS."X$KTCXB" "X$KTCXB" WHERE        
              ("KSQLKMOD"<>0 OR "KSQLKREQ"<>0) AND DECODE("KSQLKMOD",0,0,"KSQLKLBLK")=1 AND   
              "INST_ID"=USERENV('INSTANCE') AND BITAND("KSSPAFLG",1)<>0)) "GV$_LOCK" WHERE    
              TO_CHAR(USERENV('INSTANCE'))||RAWTOHEX("RADDR")=TO_CHAR("R"."INST_ID")||RAWTOH  
              EX("R"."ADDR") AND "SADDR"="S"."ADDR" AND "S"."INST_ID"=USERENV('INSTANCE')     
              AND "R"."KSQRSID2"=:B1))                                                        
   2 - access("SADDR"="S"."ADDR" AND TO_CHAR(USERENV('INSTANCE'))||RAWTOHEX("R                
              ADDR")=TO_CHAR("R"."INST_ID")||RAWTOHEX("R"."ADDR"))                            
   4 - filter("S"."INST_ID"=USERENV('INSTANCE'))                                              
   9 - filter(USERENV('INSTANCE') IS NOT NULL)                                                
  12 - filter(("KSQLKMOD"<>0 OR "KSQLKREQ"<>0) AND                                            
              "INST_ID"=USERENV('INSTANCE') AND DECODE("KSQLKMOD",0,0,"KSQLKLBLK")=0 AND      
              BITAND("KSSOBFLG",1)<>0)                                                        
  13 - filter(("KSQLKMOD"<>0 OR "KSQLKREQ"<>0) AND                                            
              "INST_ID"=USERENV('INSTANCE') AND DECODE("KSQLKMOD",0,0,"KSQLKLBLK")=0 AND      
              BITAND("KSSOBFLG",1)<>0)                                                        
  14 - filter(("KSQLKMOD"<>0 OR "KSQLKREQ"<>0) AND                                            
              DECODE("KSQLKMOD",0,0,"KSQLKLBLK")=0 AND "INST_ID"=USERENV('INSTANCE') AND      
              BITAND("KSSOBFLG",1)<>0)                                                        
  15 - filter(("KSQLKMOD"<>0 OR "KSQLKREQ"<>0) AND                                            
              DECODE("KSQLKMOD",0,0,"KSQLKLBLK")=0 AND "INST_ID"=USERENV('INSTANCE') AND      
              BITAND("KSSOBFLG",1)<>0)                                                        
  16 - filter(("KSQLKMOD"<>0 OR "KSQLKREQ"<>0) AND                                            
              DECODE("KSQLKMOD",0,0,"KSQLKLBLK")=0 AND "INST_ID"=USERENV('INSTANCE') AND      
              BITAND("KSSOBFLG",1)<>0)                                                        
  17 - filter(("KSQLKMOD"<>0 OR "KSQLKREQ"<>0) AND                                            
              DECODE("KSQLKMOD",0,0,"KSQLKLBLK")=0 AND "INST_ID"=USERENV('INSTANCE') AND      
              BITAND("KSSOBFLG",1)<>0)                                                        
  18 - filter(("KSQLKMOD"<>0 OR "KSQLKREQ"<>0) AND                                            
              DECODE("KSQLKMOD",0,0,"KSQLKLBLK")=0 AND "INST_ID"=USERENV('INSTANCE') AND      
              BITAND("KSSOBFLG",1)<>0)                                                        
  19 - filter(("KSQLKMOD"<>0 OR "KSQLKREQ"<>0) AND                                            
              DECODE("KSQLKMOD",0,0,"KSQLKLBLK")=0 AND "INST_ID"=USERENV('INSTANCE') AND      
              BITAND("KSSOBFLG",1)<>0)                                                        
  20 - filter(("KSQLKMOD"<>0 OR "KSQLKREQ"<>0) AND                                            
              DECODE("KSQLKMOD",0,0,"KSQLKLBLK")=0 AND "INST_ID"=USERENV('INSTANCE') AND      
              BITAND("KSSOBFLG",1)<>0)                                                        
  21 - filter(("KSQLKMOD"<>0 OR "KSQLKREQ"<>0) AND                                            
              DECODE("KSQLKMOD",0,0,"KSQLKLBLK")=0 AND "INST_ID"=USERENV('INSTANCE') AND      
              BITAND("KSSPAFLG",1)<>0)                                                        
  22 - access("SADDR"="S"."ADDR" AND TO_CHAR(USERENV('INSTANCE'))||RAWTOHEX("R                
              ADDR")=TO_CHAR("R"."INST_ID")||RAWTOHEX("R"."ADDR"))                            
  24 - filter("S"."INST_ID"=USERENV('INSTANCE'))                                              
  26 - filter("R"."KSQRSID2"=:B1)                                                             
  29 - filter(USERENV('INSTANCE') IS NOT NULL)                                                
  32 - filter(("KSQLKMOD"<>0 OR "KSQLKREQ"<>0) AND                                            
              "INST_ID"=USERENV('INSTANCE') AND DECODE("KSQLKMOD",0,0,"KSQLKLBLK")=1 AND      
              BITAND("KSSOBFLG",1)<>0)                                                        
  33 - filter(("KSQLKMOD"<>0 OR "KSQLKREQ"<>0) AND                                            
              "INST_ID"=USERENV('INSTANCE') AND DECODE("KSQLKMOD",0,0,"KSQLKLBLK")=1 AND      
              BITAND("KSSOBFLG",1)<>0)                                                        
  34 - filter(("KSQLKMOD"<>0 OR "KSQLKREQ"<>0) AND                                            
              DECODE("KSQLKMOD",0,0,"KSQLKLBLK")=1 AND "INST_ID"=USERENV('INSTANCE') AND      
              BITAND("KSSOBFLG",1)<>0)                                                        
  35 - filter(("KSQLKMOD"<>0 OR "KSQLKREQ"<>0) AND                                            
              DECODE("KSQLKMOD",0,0,"KSQLKLBLK")=1 AND "INST_ID"=USERENV('INSTANCE') AND      
              BITAND("KSSOBFLG",1)<>0)                                                        
  36 - filter(("KSQLKMOD"<>0 OR "KSQLKREQ"<>0) AND                                            
              DECODE("KSQLKMOD",0,0,"KSQLKLBLK")=1 AND "INST_ID"=USERENV('INSTANCE') AND      
              BITAND("KSSOBFLG",1)<>0)                                                        
  37 - filter(("KSQLKMOD"<>0 OR "KSQLKREQ"<>0) AND                                            
              DECODE("KSQLKMOD",0,0,"KSQLKLBLK")=1 AND "INST_ID"=USERENV('INSTANCE') AND      
              BITAND("KSSOBFLG",1)<>0)                                                        
  38 - filter(("KSQLKMOD"<>0 OR "KSQLKREQ"<>0) AND                                            
              DECODE("KSQLKMOD",0,0,"KSQLKLBLK")=1 AND "INST_ID"=USERENV('INSTANCE') AND      
              BITAND("KSSOBFLG",1)<>0)                                                        
  39 - filter(("KSQLKMOD"<>0 OR "KSQLKREQ"<>0) AND                                            
              DECODE("KSQLKMOD",0,0,"KSQLKLBLK")=1 AND "INST_ID"=USERENV('INSTANCE') AND      
              BITAND("KSSOBFLG",1)<>0)                                                        
  40 - filter(("KSQLKMOD"<>0 OR "KSQLKREQ"<>0) AND                                            
              DECODE("KSQLKMOD",0,0,"KSQLKLBLK")=1 AND "INST_ID"=USERENV('INSTANCE') AND      
              BITAND("KSSOBFLG",1)<>0)                                                        
  41 - filter(("KSQLKMOD"<>0 OR "KSQLKREQ"<>0) AND                                            
              DECODE("KSQLKMOD",0,0,"KSQLKLBLK")=1 AND "INST_ID"=USERENV('INSTANCE') AND      
              BITAND("KSSPAFLG",1)<>0)                                                        
                                                                                              
                                                                                              
Statistics                                                                                    
----------------------------------------------------------                                    
       2384  recursive calls                                                                  
          0  db block gets                                                                    
          0  consistent gets                                                                  
      89838  physical reads                                                                   
          0  redo size                                                                        
        398  bytes sent via SQL*Net to client                                                 
        513  bytes received via SQL*Net from client                                           
          1  SQL*Net roundtrips to/from client                                                
          1  sorts (memory)                                                                   
          0  sorts (disk)                                                                     
          0  rows processed   
```

此查询运行30S以上。通过查看执行计划，发现执行计划中出现两次MERGE JOIN CARTESIAN
遂怀疑执行计划有问题，

而动态性能视图执行计划不准多见于

1 优化器BUG

2 基表统计信息不准

尝试过session级别的优化器版本降级，问题没有得以解决。

所以采取另一种方法收集基表统计信息

## 解决方法 

```sql
sql> exec dbms_stats.gather_fixed_objects_stats;  

```

收集统计信息后运行只需0.8s

```sql

SQL> SELECT sid, ctime  
  2    FROM v$lock  
  3   WHERE BLOCK = 0  
   AND id2 IN (SELECT id2 FROM v$lock WHERE BLOCK = 1)  4  ;  
  
no rows selected  
  
Elapsed: 00:00:00.06  
  
Execution Plan  
----------------------------------------------------------  
Plan hash value: 564248026  
  
---------------------------------------------------------------------------------------  
| Id  | Operation                | Name       | Rows  | Bytes | Cost (%CPU)| Time     |  
---------------------------------------------------------------------------------------  
|   0 | SELECT STATEMENT         |            |   292 | 24236 |    28 (100)| 00:00:01 |  
|   1 |  HASH JOIN RIGHT SEMI    |            |   292 | 24236 |    28 (100)| 00:00:01 |  
|   2 |   VIEW                   | VW_NSO_1   |  1354 | 17602 |    14 (100)| 00:00:01 |  
|   3 |    HASH JOIN             |            |  1354 |   104K|    14 (100)| 00:00:01 |  
|   4 |     HASH JOIN            |            |    10 |   630 |    11 (100)| 00:00:01 |  
|   5 |      VIEW                | GV$_LOCK   |    10 |   510 |    10 (100)| 00:00:01 |  
|   6 |       UNION-ALL          |            |       |       |            |          |  
|   7 |        VIEW              | GV$_LOCK1  |     2 |   178 |     7 (100)| 00:00:01 |  
|   8 |         UNION-ALL        |            |       |       |            |          |  
|   9 |          FIXED TABLE FULL| X$KDNSSF   |     1 |    40 |     1 (100)| 00:00:01 |  
|  10 |          FIXED TABLE FULL| X$KSQEQ    |     1 |    41 |     6 (100)| 00:00:01 |  
|  11 |        FIXED TABLE FULL  | X$KTADM    |     1 |    41 |     2 (100)| 00:00:01 |  
|  12 |        FIXED TABLE FULL  | X$KTATRFIL |     1 |    36 |     0   (0)| 00:00:01 |  
|  13 |        FIXED TABLE FULL  | X$KTATRFSL |     1 |    36 |     0   (0)| 00:00:01 |  
|  14 |        FIXED TABLE FULL  | X$KTATL    |     1 |    54 |     0   (0)| 00:00:01 |  
|  15 |        FIXED TABLE FULL  | X$KTSTUSC  |     1 |    39 |     0   (0)| 00:00:01 |  
|  16 |        FIXED TABLE FULL  | X$KTSTUSS  |     1 |    39 |     0   (0)| 00:00:01 |  
|  17 |        FIXED TABLE FULL  | X$KTSTUSG  |     1 |    42 |     0   (0)| 00:00:01 |  
|  18 |        FIXED TABLE FULL  | X$KTCXB    |     1 |    38 |     1 (100)| 00:00:01 |  
|  19 |      FIXED TABLE FULL    | X$KSUSE    |  3024 | 36288 |     1 (100)| 00:00:01 |  
|  20 |     FIXED TABLE FULL     | X$KSQRS    | 13536 |   211K|     2 (100)| 00:00:01 |  
|  21 |   HASH JOIN              |            |  1354 | 94780 |    14 (100)| 00:00:01 |  
|  22 |    HASH JOIN             |            |    10 |   540 |    11 (100)| 00:00:01 |  
|  23 |     VIEW                 | GV$_LOCK   |    10 |   380 |    10 (100)| 00:00:01 |  
|  24 |      UNION-ALL           |            |       |       |            |          |  
|  25 |       VIEW               | GV$_LOCK1  |     2 |   178 |     7 (100)| 00:00:01 |  
|  26 |        UNION-ALL         |            |       |       |            |          |  
|  27 |         FIXED TABLE FULL | X$KDNSSF   |     1 |    40 |     1 (100)| 00:00:01 |  
|  28 |         FIXED TABLE FULL | X$KSQEQ    |     1 |    41 |     6 (100)| 00:00:01 |  
|  29 |       FIXED TABLE FULL   | X$KTADM    |     1 |    41 |     2 (100)| 00:00:01 |  
|  30 |       FIXED TABLE FULL   | X$KTATRFIL |     1 |    36 |     0   (0)| 00:00:01 |  
|  31 |       FIXED TABLE FULL   | X$KTATRFSL |     1 |    36 |     0   (0)| 00:00:01 |  
|  32 |       FIXED TABLE FULL   | X$KTATL    |     1 |    54 |     0   (0)| 00:00:01 |  
|  33 |       FIXED TABLE FULL   | X$KTSTUSC  |     1 |    39 |     0   (0)| 00:00:01 |  
|  34 |       FIXED TABLE FULL   | X$KTSTUSS  |     1 |    39 |     0   (0)| 00:00:01 |  
|  35 |       FIXED TABLE FULL   | X$KTSTUSG  |     1 |    42 |     0   (0)| 00:00:01 |  
|  36 |       FIXED TABLE FULL   | X$KTCXB    |     1 |    38 |     1 (100)| 00:00:01 |  
|  37 |     FIXED TABLE FULL     | X$KSUSE    |  3024 | 48384 |     1 (100)| 00:00:01 |  
|  38 |    FIXED TABLE FULL      | X$KSQRS    | 13536 |   211K|     2 (100)| 00:00:01 |  
---------------------------------------------------------------------------------------  
  
  
Statistics  
----------------------------------------------------------  
          0  recursive calls  
          0  db block gets  
          0  consistent gets  
          0  physical reads  
          0  redo size  
        398  bytes sent via SQL*Net to client  
        513  bytes received via SQL*Net from client  
          1  SQL*Net roundtrips to/from client  
          0  sorts (memory)  
          0  sorts (disk)  
          0  rows processed  

```

