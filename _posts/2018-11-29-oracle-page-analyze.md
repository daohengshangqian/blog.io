---
layout: post
title: Oracle分页功能的实现与解析
date: 2018-11-29
categories: blog
tags: [Oracle,SQL]
description: Oracle分页功能的实现与解析
---


##  建立测试表

### 测试表建立

```sql

SQL> create table dao_objects
  2  as select *  from dba_objects ;
  
Table created

```


### 收集统计信息


```sql

SQL> begin
  2     dbms_stats.gather_table_stats(ownname => 'DAO',tabname => 'DAO_OBJECTS',estimate_percent => 100);
  3  end ;
  4  /
PL/SQL procedure successfully completed

```

## 跟踪

###  使用autotrace 跟踪

```sql

SQL> set autotrace on
SQL> set linesize 200
SQL> set pagesize 20000

SQL> col object_id for 999
SQL> col rn for 999
SQL> col object_name for a30



SQL> SELECT RN, OBJECT_ID, OBJECT_NAME
  FROM (SELECT ROWNUM RN, OBJECT_ID, OBJECT_NAME
          FROM (SELECT OBJECT_ID, OBJECT_NAME
                  FROM DAO_OBJECTS
                 ORDER BY OBJECT_ID) E)
 WHERE RN BETWEEN 1 AND 10
 ORDER BY RN ;

  2    3    4    5    6    7
10 rows selected.


Execution Plan
----------------------------------------------------------
Plan hash value: 1522946681

----------------------------------------------------------------------------------------------
| Id  | Operation              | Name        | Rows  | Bytes |TempSpc| Cost (%CPU)| Time     |
----------------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT       |             | 91592 |  8228K|       |  3119   (1)| 00:00:01 |
|   1 |  SORT ORDER BY         |             | 91592 |  8228K|  9408K|  3119   (1)| 00:00:01 |
|*  2 |   VIEW                 |             | 91592 |  8228K|       |  1178   (1)| 00:00:01 |
|   3 |    COUNT               |             |       |       |       |            |          |
|   4 |     VIEW               |             | 91592 |  7066K|       |  1178   (1)| 00:00:01 |
|   5 |      SORT ORDER BY     |             | 91592 |  2683K|  3600K|  1178   (1)| 00:00:01 |
|   6 |       TABLE ACCESS FULL| DAO_OBJECTS | 91592 |  2683K|       |   428   (1)| 00:00:01 |
----------------------------------------------------------------------------------------------

Predicate Information (identified by operation id):
---------------------------------------------------

   2 - filter("RN"<=10 AND "RN">=1)


Statistics
----------------------------------------------------------
          0  recursive calls
          0  db block gets
       1540  consistent gets
       1537  physical reads
          0  redo size
        910  bytes sent via SQL*Net to client
        551  bytes received via SQL*Net from client
          2  SQL*Net roundtrips to/from client
          2  sorts (memory)
          0  sorts (disk)
         10  rows processed

SQL> SQL> SELECT RN, OBJECT_ID, OBJECT_NAME
 FROM (
 SELECT ROW_NUMBER() OVER (ORDER BY OBJECT_ID) RN  ,OBJECT_ID ,OBJECT_NAME
   FROM DAO_OBJECTS)
WHERE RN BETWEEN 11 AND 20
ORDER BY RN ;
  2    3    4    5    6
10 rows selected.


Execution Plan
----------------------------------------------------------
Plan hash value: 1600460794

-------------------------------------------------------------------------------------------------
| Id  | Operation                 | Name        | Rows  | Bytes |TempSpc| Cost (%CPU)| Time     |
-------------------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT          |             |    20 |  1840 |       |  1179   (1)| 00:00:01 |
|   1 |  SORT ORDER BY            |             |    20 |  1840 |       |  1179   (1)| 00:00:01 |
|*  2 |   VIEW                    |             |    20 |  1840 |       |  1178   (1)| 00:00:01 |
|*  3 |    WINDOW SORT PUSHED RANK|             | 91592 |  2683K|  3600K|  1178   (1)| 00:00:01 |
|   4 |     TABLE ACCESS FULL     | DAO_OBJECTS | 91592 |  2683K|       |   428   (1)| 00:00:01 |
-------------------------------------------------------------------------------------------------

Predicate Information (identified by operation id):
---------------------------------------------------

   2 - filter("RN">=11 AND "RN"<=20)
   3 - filter(ROW_NUMBER() OVER ( ORDER BY "OBJECT_ID")<=20)


Statistics
----------------------------------------------------------
          0  recursive calls
          0  db block gets
       1540  consistent gets
       1537  physical reads
          0  redo size
        880  bytes sent via SQL*Net to client
        551  bytes received via SQL*Net from client
          2  SQL*Net roundtrips to/from client
          2  sorts (memory)
          0  sorts (disk)
         10  rows processed

SQL> SELECT OBJECT_ID ,OBJECT_NAME
 FROM  DAO_OBJECTS
 ORDER BY OBJECT_ID
OFFSET 0 ROWS FETCH FIRST 10 ROWS ONLY  ;  2    3    4

10 rows selected.


Execution Plan
----------------------------------------------------------
Plan hash value: 1623199189

------------------------------------------------------------------------------------------------
| Id  | Operation                | Name        | Rows  | Bytes |TempSpc| Cost (%CPU)| Time     |
------------------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT         |             | 91592 |  9391K|       |  1178   (1)| 00:00:01 |
|*  1 |  VIEW                    |             | 91592 |  9391K|       |  1178   (1)| 00:00:01 |
|*  2 |   WINDOW SORT PUSHED RANK|             | 91592 |  2683K|  3600K|  1178   (1)| 00:00:01 |
|   3 |    TABLE ACCESS FULL     | DAO_OBJECTS | 91592 |  2683K|       |   428   (1)| 00:00:01 |
------------------------------------------------------------------------------------------------

Predicate Information (identified by operation id):
---------------------------------------------------

   1 - filter("from$_subquery$_002"."rowlimit_$$_rownumber"<=CASE  WHEN (0>=0) THEN 0
              ELSE 0 END +10 AND "from$_subquery$_002"."rowlimit_$$_rownumber">0)
   2 - filter(ROW_NUMBER() OVER ( ORDER BY "OBJECT_ID")<=CASE  WHEN (0>=0) THEN 0 ELSE
              0 END +10)


Statistics
----------------------------------------------------------
          0  recursive calls
          0  db block gets
       1540  consistent gets
       1537  physical reads
          0  redo size
        816  bytes sent via SQL*Net to client
        551  bytes received via SQL*Net from client
          2  SQL*Net roundtrips to/from client
          1  sorts (memory)
          0  sorts (disk)
         10  rows processed





```

###  使用10053跟踪

####  rownum 写法

```sql

SQL> ALTER SESSION SET EVENTS='10053 trace name context forever, level 1';

Session altered.

SQL> SELECT RN, OBJECT_ID, OBJECT_NAME
  FROM (SELECT ROWNUM RN, OBJECT_ID, OBJECT_NAME
          FROM (SELECT OBJECT_ID, OBJECT_NAME
                  FROM DAO_OBJECTS
                 ORDER BY OBJECT_ID) E)
 WHERE RN BETWEEN 1 AND 10
 ORDER BY RN ;  2    3    4    5    6    7

  RN OBJECT_ID OBJECT_NAME
---- --------- ------------------------------
   1         2 C_OBJ#
   2         3 I_OBJ#
   3         4 TAB$
   4         5 CLU$
   5         6 C_TS#
   6         7 I_TS#
   7         8 C_FILE#_BLOCK#
   8         9 I_FILE#_BLOCK#
   9        10 C_USER#
  10        11 I_USER#

10 rows selected.

SQL> ALTER SESSION SET EVENTS='10053 trace name context off';

Session altered.



```

#### 使用分析函数

```sql


SQL> ALTER SESSION SET EVENTS='10053 trace name context forever, level 1';

Session altered.

SQL> SELECT RN, OBJECT_ID, OBJECT_NAME
 FROM (
 SELECT ROW_NUMBER() OVER (ORDER BY OBJECT_ID) RN  ,OBJECT_ID ,OBJECT_NAME
   FROM DAO_OBJECTS)
WHERE RN BETWEEN 1 AND 10
ORDER BY RN ;  2    3    4    5    6

  RN OBJECT_ID OBJECT_NAME
---- --------- ------------------------------
   1         2 C_OBJ#
   2         3 I_OBJ#
   3         4 TAB$
   4         5 CLU$
   5         6 C_TS#
   6         7 I_TS#
   7         8 C_FILE#_BLOCK#
   8         9 I_FILE#_BLOCK#
   9        10 C_USER#
  10        11 I_USER#

10 rows selected.

SQL> ALTER SESSION SET EVENTS='10053 trace name context off';

```

#### 使用12C新特性

```sql

SQL> ALTER SESSION SET EVENTS='10053 trace name context forever, level 1';

Session altered.

SQL> SELECT OBJECT_ID ,OBJECT_NAME
 FROM  DAO_OBJECTS
 ORDER BY OBJECT_ID
OFFSET 0 ROWS FETCH FIRST 10 ROWS ONLY  ;  2    3    4

OBJECT_ID OBJECT_NAME
--------- ------------------------------
        2 C_OBJ#
        3 I_OBJ#
        4 TAB$
        5 CLU$
        6 C_TS#
        7 I_TS#
        8 C_FILE#_BLOCK#
        9 I_FILE#_BLOCK#
       10 C_USER#
       11 I_USER#

10 rows selected.

SQL> ALTER SESSION SET EVENTS='10053 trace name context off';

Session altered.



```

### 对跟踪文件进行解析

###  rownum 写法

执行计划如下

```sql

sql=SELECT RN, OBJECT_ID, OBJECT_NAME
  FROM (SELECT ROWNUM RN, OBJECT_ID, OBJECT_NAME
          FROM (SELECT OBJECT_ID, OBJECT_NAME
                  FROM DAO_OBJECTS
                 ORDER BY OBJECT_ID) E)
 WHERE RN BETWEEN 1 AND 10
 ORDER BY RN 
 
----- Explain Plan Dump -----
----- Plan Table -----
 
============
Plan Table
============
---------------------------------------------+-----------------------------------+
| Id  | Operation               | Name       | Rows  | Bytes | Cost  | Time      |
---------------------------------------------+-----------------------------------+
| 0   | SELECT STATEMENT        |            |       |       |  3119 |           |
| 1   |  SORT ORDER BY          |            |   89K | 8229K |  3119 |  00:00:38 |
| 2   |   VIEW                  |            |   89K | 8229K |  1178 |  00:00:15 |
| 3   |    COUNT                |            |       |       |       |           |
| 4   |     VIEW                |            |   89K | 7066K |  1178 |  00:00:15 |
| 5   |      SORT ORDER BY      |            |   89K | 2683K |  1178 |  00:00:15 |
| 6   |       TABLE ACCESS FULL | DAO_OBJECTS|   89K | 2683K |   428 |  00:00:06 |
---------------------------------------------+-----------------------------------+
Predicate Information:
----------------------
2 - filter(("RN"<=10 AND "RN">=1))


```

在10053 trace中 语句被改写如下:

```sql

SELECT "from$_subquery$_001"."RN"          "RN",
       "from$_subquery$_001"."OBJECT_ID"   "OBJECT_ID",
       "from$_subquery$_001"."OBJECT_NAME" "OBJECT_NAME"
  FROM (SELECT ROWNUM            "RN",
               "E"."OBJECT_ID"   "OBJECT_ID",
               "E"."OBJECT_NAME" "OBJECT_NAME"
          FROM (SELECT "DAO_OBJECTS"."OBJECT_ID"   "OBJECT_ID",
                       "DAO_OBJECTS"."OBJECT_NAME" "OBJECT_NAME"
                  FROM "DAO"."DAO_OBJECTS" "DAO_OBJECTS"
                 ORDER BY "DAO_OBJECTS"."OBJECT_ID") "E") "from$_subquery$_001"
 WHERE 10 >= 1
   AND "from$_subquery$_001"."RN" <= 10
   AND "from$_subquery$_001"."RN" >= 1
 ORDER BY "from$_subquery$_001"."RN"

```
我们发现:对于本条语句的改写,仅限于增加了用户名,表别名,内联视图名

### 使用分析函数

执行计划如下

```sql

----- Explain Plan Dump -----
----- Plan Table -----
 
============
Plan Table
============
------------------------------------------------+-----------------------------------+
| Id  | Operation                  | Name       | Rows  | Bytes | Cost  | Time      |
------------------------------------------------+-----------------------------------+
| 0   | SELECT STATEMENT           |            |       |       |  1179 |           |
| 1   |  SORT ORDER BY             |            |    10 |   920 |  1179 |  00:00:15 |
| 2   |   VIEW                     |            |    10 |   920 |  1178 |  00:00:15 |
| 3   |    WINDOW SORT PUSHED RANK |            |   89K | 2683K |  1178 |  00:00:15 |
| 4   |     TABLE ACCESS FULL      | DAO_OBJECTS|   89K | 2683K |   428 |  00:00:06 |
------------------------------------------------+-----------------------------------+
Predicate Information:
----------------------
2 - filter(("RN">=1 AND "RN"<=10))
3 - filter(ROW_NUMBER() OVER ( ORDER BY "OBJECT_ID")<=10)

```
在trace文件总可以发现 语句被改写如下

```sql

SELECT "from$_subquery$_001"."RN"          "RN",
       "from$_subquery$_001"."OBJECT_ID"   "OBJECT_ID",
       "from$_subquery$_001"."OBJECT_NAME" "OBJECT_NAME"
  FROM (SELECT ROW_NUMBER() OVER(ORDER BY "DAO_OBJECTS"."OBJECT_ID") "RN",
               "DAO_OBJECTS"."OBJECT_ID" "OBJECT_ID",
               "DAO_OBJECTS"."OBJECT_NAME" "OBJECT_NAME"
          FROM "DAO"."DAO_OBJECTS" "DAO_OBJECTS"
         WHERE 10 >= 1) "from$_subquery$_001"
 WHERE "from$_subquery$_001"."RN" >= 1
   AND "from$_subquery$_001"."RN" <= 10
 ORDER BY "from$_subquery$_001"."RN"

```
我们发现:对于本条语句的改写,仅限于增加了用户名,表别名,内联视图名


### 12C新特性


```sql


sql=SELECT OBJECT_ID ,OBJECT_NAME
 FROM  DAO_OBJECTS
 ORDER BY OBJECT_ID
OFFSET 0 ROWS FETCH FIRST 10 ROWS ONLY  
----- Explain Plan Dump -----
----- Plan Table -----
 
============
Plan Table
============
-----------------------------------------------+-----------------------------------+
| Id  | Operation                 | Name       | Rows  | Bytes | Cost  | Time      |
-----------------------------------------------+-----------------------------------+
| 0   | SELECT STATEMENT          |            |       |       |  1178 |           |
| 1   |  VIEW                     |            |   89K | 9392K |  1178 |  00:00:15 |
| 2   |   WINDOW SORT PUSHED RANK |            |   89K | 2683K |  1178 |  00:00:15 |
| 3   |    TABLE ACCESS FULL      | DAO_OBJECTS|   89K | 2683K |   428 |  00:00:06 |
-----------------------------------------------+-----------------------------------+
Predicate Information:
----------------------
1 - filter(("from$_subquery$_002"."rowlimit_$$_rownumber"<=CASE  WHEN (0>=0) THEN 0 ELSE 0 END +10 AND "from$_subquery$_002"."rowlimit_$$_rownumber">0))
2 - filter(ROW_NUMBER() OVER ( ORDER BY "OBJECT_ID")<=CASE  WHEN (0>=0) THEN 0 ELSE 0 END +10)
 

```

语句被改写如下

```sql


SELECT "from$_subquery$_002"."OBJECT_ID"   "OBJECT_ID",
       "from$_subquery$_002"."OBJECT_NAME" "OBJECT_NAME"
  FROM (SELECT "DAO_OBJECTS"."OBJECT_ID" "OBJECT_ID",
               "DAO_OBJECTS"."OBJECT_NAME" "OBJECT_NAME",
               "DAO_OBJECTS"."OBJECT_ID" "rowlimit_$_0",
               ROW_NUMBER() OVER(ORDER BY "DAO_OBJECTS"."OBJECT_ID") "rowlimit_$$_rownumber"
          FROM "DAO"."DAO_OBJECTS" "DAO_OBJECTS"
         WHERE 0 < CASE
                 WHEN (0 >= 0) THEN
                  0
                 ELSE
                  0
               END + 10) "from$_subquery$_002"
 WHERE "from$_subquery$_002"."rowlimit_$$_rownumber" <= CASE
         WHEN (0 >= 0) THEN
          0
         ELSE
          0
       END + 10
   AND "from$_subquery$_002"."rowlimit_$$_rownumber" > 0
 ORDER BY "from$_subquery$_002"."rowlimit_$_0"


```

可以看到,12C的新特性,也是使用了分析函数.这一点在执行计划上尤为明显,两条语句执行计划仅限于最后的一个sort order by 操作.



## 总结: 

         从最后的结果来看,三种写法都对数据进行了排序,只是表现不同,且使用12C的新特性本质上就是使用了分析函数.
         
         三种排序方式的成本由大到小依次为 rownum > 分析函数 > 12C新特性 
         
         就现阶段来讲,使用分析函数完成分页,也许是最好的选择.
         

# 写在最后

排序一直是高消耗CPU的操作,并且排序操作基本无法优化,在数据量日益增大的今天,我们是不是有必要要将排序操作放在数据库中来执行,是一个值得我们思考的问题.