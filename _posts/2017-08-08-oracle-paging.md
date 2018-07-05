---
layout: post
title:  Oracle 分页写法总结
date: 2017-08-08
categories: blog
tags: [SQL]
description:  Oracle 分页写法总结
---

我们在逛一些网站时经常会遇到这种情况,由于返回的数据太多无法在一个页面中完全展示.需要在多个页面中显示.界面类似


注意页面的右下角1/33 告诉我们商品一共有33页.


那么这种需求在SQL语句中如何实现?这就是我们经典的分页问题.

其实在这方面Oracle分页写法的复杂程度要高于MySQL分页写法.

不过在12C中推出的新特性,写完与MySQL已经一模一样,只是语法稍有不同.

在这里我们来总结下,Oracle实现分页查询的几种写法:


建立测试表

```sql

dao@bosenriu.com>create table daoobjects as select from dba_objects ;


Table created.

```

## 经典rownum写法

### 前五行 

```sql
dao@bosenriu.com>select rownum rn, object_id, object_name

2 from (select object_id, object_name

3 from dao_objects d

4 order by object_id desc)

5 where rownum <= 5

RN OBJECT_ID OBJECT_NAME

————— ————— ————————————————————————————————————————————————————————————————

1 92015 DAO_OBJECTS

2 92005 WRM$_DEEP_PURGE_INTERVAL

3 92003 WRI$_OPTSTAT_HISTGRM_HISTORY

4 92002 SYS_IL0000012928C00016$$

5 92001 SYS_LOB0000012928C00016$$


Execution Plan

—————————————————————————————

Plan hash value: 4183165338


———————————————————————————————————————————————-

| Id | Operation | Name | Rows | Bytes |TempSpc| Cost (%CPU)| Time |

———————————————————————————————————————————————-

| 0 | SELECT STATEMENT | | 5 | 395 | | 1168 (1)| 00:00:01 |

| 1 | COUNT STOPKEY | | | | | | |

| 2 | VIEW | | 90914 | 7013K| | 1168 (1)| 00:00:01 |

| 3 | SORT ORDER BY STOPKEY| | 90914 | 2663K| 3576K| 1168 (1)| 00:00:01 |

| 4 | TABLE ACCESS FULL | DAO_OBJECTS | 90914 | 2663K| | 426 (1)| 00:00:01 |

———————————————————————————————————————————————-


Predicate Information (identified by operation id):

—————————————————————————-


1 - filter(ROWNUM<=5)

3 - filter(ROWNUM<=5)



Statistics

—————————————————————————————

2 recursive calls

0 db block gets

1532 consistent gets

1527 physical reads

0 redo size

900 bytes sent via SQLNet to client

551 bytes received via SQLNet from client

2 SQLNet roundtrips to/from client

1 sorts (memory)

0 sorts (disk)

5 rows processed


```

### 6-10行 
```sql

dao@bosenriu.com>select

2 from

3 (select rownum rn,object_id,object_name

4 from (

5 select object_id,object_name

6 from dao_objects d

7 order by object_id desc )

8 )

9 where rn between 6 and 10 ;


RN OBJECT_ID OBJECT_NAME

————— ————— ————————————————————————————————————————————————————————————————

6 92000 WRI$_OPTSTAT_HISTHEAD_HISTORY

7 91999 WRI$_ADV_SEGADV_SEGROW

8 91998 WRI$_ADV_OBJSPACE_CHROW_DATA

9 91997 WRI$_ADV_OBJSPACE_TREND_DATA

10 91991 WRP$_REPORTS_TIME_BANDS



Execution Plan

—————————————————————————————

Plan hash value: 4259792731


——————————————————————————————————————————————-

| Id | Operation | Name | Rows | Bytes |TempSpc| Cost (%CPU)| Time |

——————————————————————————————————————————————-

| 0 | SELECT STATEMENT | | 90914 | 8168K| | 1168 (1)| 00:00:01 |

| 1 | VIEW | | 90914 | 8168K| | 1168 (1)| 00:00:01 |

| 2 | COUNT | | | | | | |

| 3 | VIEW | | 90914 | 7013K| | 1168 (1)| 00:00:01 |

| 4 | SORT ORDER BY | | 90914 | 2663K| 3576K| 1168 (1)| 00:00:01 |

| 5 | TABLE ACCESS FULL| DAO_OBJECTS | 90914 | 2663K| | 426 (1)| 00:00:01 |

——————————————————————————————————————————————-


Predicate Information (identified by operation id):

—————————————————————————-


1 - filter(“RN”<=10 AND “RN”>=6)



Statistics

—————————————————————————————

1 recursive calls

0 db block gets

1532 consistent gets

0 physical reads

0 redo size

917 bytes sent via SQLNet to client

551 bytes received via SQLNet from client

2 SQLNet roundtrips to/from client

1 sorts (memory)

0 sorts (disk)

5 rows processed

```


## 分析函数写法

### 前五行　
```sql

dao@bosenriu.com>select rn,object_id,object_name

2 from (

3 select object_id ,object_name ,row_number() over(order by object_id desc) rn

4 from dao_objects )

5 where rn <=5 ;


RN OBJECT_ID OBJECT_NAME

————— ————— ————————————————————————————————————————————————————————————————

1 92015 DAO_OBJECTS

2 92005 WRM$_DEEP_PURGE_INTERVAL

3 92003 WRI$_OPTSTAT_HISTGRM_HISTORY

4 92002 SYS_IL0000012928C00016$$

5 92001 SYS_LOB0000012928C00016$$



Execution Plan

—————————————————————————————

Plan hash value: 1623199189


————————————————————————————————————————————————

| Id | Operation | Name | Rows | Bytes |TempSpc| Cost (%CPU)| Time |

————————————————————————————————————————————————

| 0 | SELECT STATEMENT | | 5 | 460 | | 1168 (1)| 00:00:01 |

| 1 | VIEW | | 5 | 460 | | 1168 (1)| 00:00:01 |

| 2 | WINDOW SORT PUSHED RANK| | 90914 | 2663K| 3576K| 1168 (1)| 00:00:01 |

| 3 | TABLE ACCESS FULL | DAO_OBJECTS | 90914 | 2663K| | 426 (1)| 00:00:01 |

————————————————————————————————————————————————


Predicate Information (identified by operation id):

—————————————————————————-


1 - filter(“RN”<=5)

2 - filter(ROW_NUMBER() OVER ( ORDER BY INTERNAL_FUNCTION(“OBJECT_ID”) DESC )<=5)



Statistics

—————————————————————————————

1 recursive calls

0 db block gets

1532 consistent gets

0 physical reads

0 redo size

900 bytes sent via SQLNet to client

551 bytes received via SQLNet from client

2 SQLNet roundtrips to/from client

1 sorts (memory)

0 sorts (disk)

5 rows processed

```
### 6-10行
```sql

dao@bosenriu.com>select rn,object_id,object_name

2 from (

3 select object_id ,object_name ,row_number() over(order by object_id desc) rn

4 from dao_objects )

5 where rn between 6 and 10 ;



RN OBJECT_ID OBJECT_NAME

————— ————— ————————————————————————————————————————————————————————————————

6 92000 WRI$_OPTSTAT_HISTHEAD_HISTORY

7 91999 WRI$_ADV_SEGADV_SEGROW

8 91998 WRI$_ADV_OBJSPACE_CHROW_DATA

9 91997 WRI$_ADV_OBJSPACE_TREND_DATA

10 91991 WRP$_REPORTS_TIME_BANDS



Execution Plan

—————————————————————————————

Plan hash value: 1623199189


————————————————————————————————————————————————

| Id | Operation | Name | Rows | Bytes |TempSpc| Cost (%CPU)| Time |

————————————————————————————————————————————————

| 0 | SELECT STATEMENT | | 10 | 920 | | 1168 (1)| 00:00:01 |

| 1 | VIEW | | 10 | 920 | | 1168 (1)| 00:00:01 |

| 2 | WINDOW SORT PUSHED RANK| | 90914 | 2663K| 3576K| 1168 (1)| 00:00:01 |

| 3 | TABLE ACCESS FULL | DAO_OBJECTS | 90914 | 2663K| | 426 (1)| 00:00:01 |

————————————————————————————————————————————————


Predicate Information (identified by operation id):

—————————————————————————-


1 - filter(“RN”>=6 AND “RN”<=10)

2 - filter(ROW_NUMBER() OVER ( ORDER BY INTERNAL_FUNCTION(“OBJECT_ID”) DESC )<=10)



Statistics

—————————————————————————————

1 recursive calls

0 db block gets

1532 consistent gets

0 physical reads

0 redo size

917 bytes sent via SQLNet to client

551 bytes received via SQLNet from client

2 SQLNet roundtrips to/from client

1 sorts (memory)

0 sorts (disk)

5 rows processed
```


## 12C新特性 offset 写法

### 前五行
```sql

如果仔细看执行计划和谓词信息发现其原理应该也是基于分析函数的

dao@bosenriu.com>dao@bosenriu.com>select object_id ,object_name

2 from dao_objects

3 order BY object_id desc

4 fetch first 5 rows only ;


OBJECT_ID OBJECT_NAME

————— ————————————————————————————————————————————————————————————————

92015 DAO_OBJECTS

92005 WRM$_DEEP_PURGE_INTERVAL

92003 WRI$_OPTSTAT_HISTGRM_HISTORY

92002 SYS_IL0000012928C00016$$

92001 SYS_LOB0000012928C00016$$



Execution Plan

—————————————————————————————

Plan hash value: 1623199189


————————————————————————————————————————————————

| Id | Operation | Name | Rows | Bytes |TempSpc| Cost (%CPU)| Time |

————————————————————————————————————————————————

| 0 | SELECT STATEMENT | | 5 | 525 | | 1168 (1)| 00:00:01 |

| 1 | VIEW | | 5 | 525 | | 1168 (1)| 00:00:01 |

|* 2 | WINDOW SORT PUSHED RANK| | 90914 | 2663K| 3576K| 1168 (1)| 00:00:01 |

| 3 | TABLE ACCESS FULL | DAO_OBJECTS | 90914 | 2663K| | 426 (1)| 00:00:01 |

————————————————————————————————————————————————


Predicate Information (identified by operation id):

—————————————————————————-


1 - filter(“from$_subquery$_002”.”rowlimit$$rownumber”<=5)

2 - filter(ROW_NUMBER() OVER ( ORDER BY INTERNAL_FUNCTION(“OBJECT_ID”) DESC )<=5)



Statistics

—————————————————————————————

1 recursive calls

0 db block gets

1532 consistent gets

0 physical reads

0 redo size

821 bytes sent via SQLNet to client

551 bytes received via SQLNet from client

2 SQLNet roundtrips to/from client

1 sorts (memory)

0 sorts (disk)

5 rows processed
```
### 6-10行
```sql
dao@bosenriu.com>select object_id ,object_name

2 from dao_objects

3 order BY object_id desc

4 offset 5 rows fetch first 5 rows only ;


OBJECT_ID OBJECT_NAME

————— ————————————————————————————————————————————————————————————————

92000 WRI$_OPTSTAT_HISTHEAD_HISTORY

91999 WRI$_ADV_SEGADV_SEGROW

91998 WRI$_ADV_OBJSPACE_CHROW_DATA

91997 WRI$_ADV_OBJSPACE_TREND_DATA

91991 WRP$_REPORTS_TIME_BANDS



Execution Plan

—————————————————————————————

Plan hash value: 1623199189


————————————————————————————————————————————————

| Id | Operation | Name | Rows | Bytes |TempSpc| Cost (%CPU)| Time |

————————————————————————————————————————————————

| 0 | SELECT STATEMENT | | 90914 | 9322K| | 1168 (1)| 00:00:01 |

| 1 | VIEW | | 90914 | 9322K| | 1168 (1)| 00:00:01 |

|* 2 | WINDOW SORT PUSHED RANK| | 90914 | 2663K| 3576K| 1168 (1)| 00:00:01 |

| 3 | TABLE ACCESS FULL | DAO_OBJECTS | 90914 | 2663K| | 426 (1)| 00:00:01 |

————————————————————————————————————————————————


Predicate Information (identified by operation id):

—————————————————————————-


1 - filter(“from$_subquery$_002”.”rowlimit$$rownumber”<=CASE WHEN (5>=0) THEN 5

ELSE 0 END +5 AND “from$_subquery$_002”.”rowlimit$$_rownumber”>5)

2 - filter(ROW_NUMBER() OVER ( ORDER BY INTERNAL_FUNCTION(“OBJECT_ID”) DESC )<=CASE

WHEN (5>=0) THEN 5 ELSE 0 END +5)



Statistics

—————————————————————————————

1 recursive calls

0 db block gets

1532 consistent gets

0 physical reads

0 redo size

838 bytes sent via SQLNet to client

551 bytes received via SQLNet from client

2 SQL*Net roundtrips to/from client

1 sorts (memory)

0 sorts (disk)

5 rows processed

```
