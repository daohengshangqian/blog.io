---
layout: post
title:  Oracle SQL优化一例
date: 2012-06-08
categories: blog
tags: [Oracle]
description: Oracle SQL优化一例。
---

## 待优化语句

```sql

UPDATE T_ORDER_INFO oo
   SET oo.ORDER_ID = (SELECT ee.order_id
                        from E_ORDER ee
                       where ee.orderno = oo.order_no),
       oo.ORDER_NO = (SELECT ee.order_id
                        from E_ORDER ee
                       where ee.orderno = oo.order_no)
 where OO.ORDER_NO in (SELECT orderno from E_ORDER) ;
 
```

 
## 执行计划与执行时间

```sql

SQL> set timing on 
SQL> UPDATE T_ORDER_INFO oo
  2     SET oo.ORDER_ID = (SELECT ee.order_id
  3                          from E_ORDER ee
  4                         where ee.orderno = oo.order_no),
  5         oo.ORDER_NO = (SELECT ee.order_id
  6                          from E_ORDER ee
  7                         where ee.orderno = oo.order_no)
  8   where OO.ORDER_NO in (SELECT orderno from E_ORDER) ;

30852 rows updated.

Elapsed: 00:13:57.67

Execution Plan
----------------------------------------------------------
Plan hash value: 3270779860

-------------------------------------------------------------------------------------------------
| Id  | Operation              | Name           | Rows  | Bytes |TempSpc| Cost (%CPU)| Time     |
-------------------------------------------------------------------------------------------------
|   0 | UPDATE STATEMENT       |                |     6 |   390 |       |  9419   (1)| 00:01:54 |
|   1 |  UPDATE                | T_ORDER_INFO   |       |       |       |            |          |
|*  2 |   HASH JOIN SEMI       |                |     6 |   390 |  1816K|   671   (1)| 00:00:09 |
|   3 |    INDEX FULL SCAN     | IDX_ORDER_INFO | 33162 |  1424K|       |   257   (0)| 00:00:04 |
|   4 |    INDEX FAST FULL SCAN| IDX_E_ORDER    | 79766 |  1635K|       |   199   (1)| 00:00:03 |
|*  5 |   INDEX SKIP SCAN      | IDX_E_ORDER    |     1 |    54 |       |   728   (0)| 00:00:09 |
|*  6 |   INDEX SKIP SCAN      | IDX_E_ORDER    |     1 |    54 |       |   728   (0)| 00:00:09 |
-------------------------------------------------------------------------------------------------

Predicate Information (identified by operation id):
---------------------------------------------------

   2 - access("OO"."ORDER_NO"="ORDERNO")
   5 - access("EE"."ORDERNO"=:B1)
       filter("EE"."ORDERNO"=:B1)
   6 - access("EE"."ORDERNO"=:B1)
       filter("EE"."ORDERNO"=:B1)


Statistics
----------------------------------------------------------
        353  recursive calls
     242023  db block gets
   45016382  consistent gets
        124  physical reads
   46463812  redo size
        835  bytes sent via SQL*Net to client
       1111  bytes received via SQL*Net from client
          3  SQL*Net roundtrips to/from client
          2  sorts (memory)
          0  sorts (disk)
      30852  rows processed
      
``` 

# 优化方法 建立索引

```sql

 create index dao_idx1 on E_ORDER (orderno,order_id);
 
```

# 优化后

```sql

SQL> UPDATE T_ORDER_INFO oo
  2     SET oo.ORDER_ID = (SELECT ee.order_id
  3                          from E_ORDER ee
  4                         where ee.orderno = oo.order_no),
  5         oo.ORDER_NO = (SELECT ee.order_id
  6                          from E_ORDER ee
  7                         where ee.orderno = oo.order_no)
  8   where OO.ORDER_NO in (SELECT orderno from E_ORDER) ;

30852 rows updated.

Elapsed: 00:00:01.32

Execution Plan
----------------------------------------------------------
Plan hash value: 2446955784

-------------------------------------------------------------------------------------------------
| Id  | Operation              | Name           | Rows  | Bytes |TempSpc| Cost (%CPU)| Time     |
-------------------------------------------------------------------------------------------------
|   0 | UPDATE STATEMENT       |                |     6 |   390 |       |   719   (3)| 00:00:09 |
|   1 |  UPDATE                | T_ORDER_INFO   |       |       |       |            |          |
|*  2 |   HASH JOIN SEMI       |                |     6 |   390 |  1816K|   671   (1)| 00:00:09 |
|   3 |    INDEX FULL SCAN     | IDX_ORDER_INFO | 33162 |  1424K|       |   257   (0)| 00:00:04 |
|   4 |    INDEX FAST FULL SCAN| DAO_IDX1       | 79766 |  1635K|       |   199   (1)| 00:00:03 |
|*  5 |   INDEX RANGE SCAN     | DAO_IDX1       |     1 |    54 |       |     3   (0)| 00:00:01 |
|*  6 |   INDEX RANGE SCAN     | DAO_IDX1       |     1 |    54 |       |     3   (0)| 00:00:01 |
-------------------------------------------------------------------------------------------------

Predicate Information (identified by operation id):
---------------------------------------------------

   2 - access("OO"."ORDER_NO"="ORDERNO")
   5 - access("EE"."ORDERNO"=:B1)
   6 - access("EE"."ORDERNO"=:B1)


Statistics
----------------------------------------------------------
        154  recursive calls
     222431  db block gets
      79098  consistent gets
        726  physical reads
   32845368  redo size
        834  bytes sent via SQL*Net to client
       1111  bytes received via SQL*Net from client
          3  SQL*Net roundtrips to/from client
          2  sorts (memory)
          0  sorts (disk)
      30852  rows processed
```      

