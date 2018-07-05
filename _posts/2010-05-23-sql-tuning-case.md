---
layout: post
title:  Oracle SQL优化一例
date: 2010-05-23
categories: blog
tags: [Oracle]
description: Oracle SQL优化一例。
---

# 待优化语句

```sql
select mli.src 特服号,
       mli.recvtel 手机号,
       mli.srcservicecodeadd 扩展码,
       to_char(mli.senddate, 'yyyy-mm-dd hh24:mi:ss') 发送时间,
       to_char(c.submitdate, 'yyyy-mm-dd hh24:mi:ss') 提交时间,
       to_char(c.receivedate, 'yyyy-mm-dd hh24:mi:ss') 接收时间,
       c.reportstatus,
       c.errorcode,
       b.sendcontent 内容
  from eucpmanager.mobileloginfo mli, eucpmanager.smscontentinfo b, eucpmanager.eucpstatusreportlog c
where mli.smsid = b.id
   and mli.senddate >=
       TO_DATE('2014-06-22 00:00:00', 'yyyy-mm-dd hh24:mi:ss')
   and mli.senddate <=
       TO_DATE('2014-06-23 23:59:59', 'yyyy-mm-dd hh24:mi:ss')
      --and   mli.src ='0669'
   and c.seqid = mli.seqid
   and c.usercode = mli.recvtel
   and mli.recvtel = :1;
```

# 原执行计划
```sql

SQL> select * from table(dbms_xplan.display);

PLAN_TABLE_OUTPUT                                                                                                                                    
------------------------------------------------------------------------------------------------------------------------------------------------------
Plan hash value: 4117256253                                                                                                                          
                                                                                                                                                     
------------------------------------------------------------------------------------------------------------------------------------                 
| Id  | Operation                            | Name                        | Rows  | Bytes | Cost (%CPU)| Time     | Pstart| Pstop |                 
------------------------------------------------------------------------------------------------------------------------------------                 
|   0 | SELECT STATEMENT                     |                             |     1 |   266 |   188K  (2)| 00:37:48 |       |       |                 
|   1 |  NESTED LOOPS                        |                             |     1 |   266 |   188K  (2)| 00:37:48 |       |       |                 
|*  2 |   HASH JOIN                          |                             |     1 |    90 |   188K  (2)| 00:37:48 |       |       |                 
|*  3 |    TABLE ACCESS BY GLOBAL INDEX ROWID| MOBILELOGINFO               |     1 |    43 |   718   (0)| 00:00:09 | ROWID | ROWID |                 
|*  4 |     INDEX SKIP SCAN                  | UN_IDX_MOBILELOGINFO_SRCTEL |   649 |       |   154   (0)| 00:00:02 |       |       |                 
|   5 |    PARTITION RANGE ALL               |                             |   330 | 15510 |   188K  (2)| 00:37:40 |     1 |   277 |                 
|*  6 |     TABLE ACCESS FULL                | EUCPSTATUSREPORTLOG         |   330 | 15510 |   188K  (2)| 00:37:40 |     1 |   277 |                 
|   7 |   TABLE ACCESS BY GLOBAL INDEX ROWID | SMSCONTENTINFO              |     1 |   176 |     2   (0)| 00:00:01 | ROWID | ROWID |                 
|*  8 |    INDEX UNIQUE SCAN                 | PK_SMSCONTENTINFO_NEW       |     1 |       |     1   (0)| 00:00:01 |       |       |                 
------------------------------------------------------------------------------------------------------------------------------------                 
                                                                                                                                                     
Predicate Information (identified by operation id):                                                                                                  
---------------------------------------------------                                                                                                  
                                                                                                                                                     
   2 - access("C"."SEQID"="MLI"."SEQID" AND "C"."USERCODE"="MLI"."RECVTEL")                                                                          
   3 - filter("MLI"."SENDDATE">=TO_DATE(' 2014-06-22 00:00:00', 'syyyy-mm-dd hh24:mi:ss') AND "MLI"."SENDDATE"<=TO_DATE('                            
              2014-06-23 23:59:59', 'syyyy-mm-dd hh24:mi:ss'))                                                                                       
   4 - access("MLI"."RECVTEL"='18973277876')                                                                                                         
       filter("MLI"."RECVTEL"='18973277876')                                                                                                         
   6 - filter("C"."USERCODE"='18973277876')                                                                                                          
   8 - access("MLI"."SMSID"="B"."ID")                                                                                                                

26 rows selected.

```                                                                                                                        



# 优化方法
在 SENDDATE ,usercode上建立索引

# 优化后执行计划

```sql

SQL> select * from table(dbms_xplan.display);

Plan hash value: 2828921524

------------------------------------------------------------------------------------------------------------------------------------
| Id  | Operation                            | Name                        | Rows  | Bytes | Cost (%CPU)| Time     | Pstart| Pstop |
------------------------------------------------------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT                     |                             |     1 |   266 |   753   (1)| 00:00:10 |       |       |
|   1 |  NESTED LOOPS                        |                             |     1 |   266 |   753   (1)| 00:00:10 |       |       |
|*  2 |   HASH JOIN                          |                             |     1 |    90 |   751   (1)| 00:00:10 |       |       |
|*  3 |    TABLE ACCESS BY GLOBAL INDEX ROWID| MOBILELOGINFO               |     1 |    43 |   750   (0)| 00:00:10 | ROWID | ROWID |
|*  4 |     INDEX SKIP SCAN                  | UN_IDX_MOBILELOGINFO_SRCTEL |   678 |       |   161   (0)| 00:00:02 |       |       |
|   5 |    PARTITION RANGE ALL               |                             |   344 | 16168 |     0   (0)| 00:00:01 |     1 |   277 |
|   6 |     TABLE ACCESS BY LOCAL INDEX ROWID| EUCPSTATUSREPORTLOG         |   344 | 16168 |     0   (0)| 00:00:01 |     1 |   277 |
|*  7 |      INDEX SKIP SCAN                 | L_IND_USERCODE              |     1 |       |     0   (0)| 00:00:01 |     1 |   277 |
|   8 |   TABLE ACCESS BY GLOBAL INDEX ROWID | SMSCONTENTINFO              |     1 |   176 |     2   (0)| 00:00:01 | ROWID | ROWID |
|*  9 |    INDEX UNIQUE SCAN                 | PK_SMSCONTENTINFO_NEW       |     1 |       |     1   (0)| 00:00:01 |       |       |
------------------------------------------------------------------------------------------------------------------------------------

Predicate Information (identified by operation id):
---------------------------------------------------

   2 - access("C"."SEQID"="MLI"."SEQID" AND "C"."USERCODE"="MLI"."RECVTEL")
   3 - filter("MLI"."SENDDATE">=TO_DATE(' 2014-06-22 00:00:00', 'syyyy-mm-dd hh24:mi:ss') AND "MLI"."SENDDATE"<=TO_DATE('
              2014-06-23 23:59:59', 'syyyy-mm-dd hh24:mi:ss'))
   4 - access("MLI"."RECVTEL"=:1)
       filter("MLI"."RECVTEL"=:1)
   7 - access("C"."USERCODE"=:1)
       filter("C"."USERCODE"=:1)
   9 - access("MLI"."SMSID"="B"."ID")

28 rows selected.
```
