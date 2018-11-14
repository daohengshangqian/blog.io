---
layout: post
title: 使用SQL统计词频
date: 2018-11-14
categories: blog
tags: [SQL]
description: 使用SQL统计词频
---

##　功能背景


最近在做数据资产梳理,还有还有命名规范.

突然间想统计下表在命名中最常用的简写

通常在数据库中我们使用'_'分割简写

```sql

select TABLE_NAME
 from dba_tableS 
 WHERE ROWNUM <= 10 
  AND INSTR(TABLE_NAME,'_') > 1
  and table_name like 'IN%' ;

SQL> select TABLE_NAME
  2   from dba_tableS
  3   WHERE ROWNUM <= 10
  4    AND INSTR(TABLE_NAME,'_') > 1
  5    and table_name like 'IN%' ;
TABLE_NAME
------------------------------
INVALIDATION_REGISTRY$
INSERT_TSN_LIST$
IND_STATS$
IND_ONLINE$
INDPART_PARAM$


```

此时我们就可以看到短语被下划线分割

上面的数据就被分割为

INVALIDATION REGISTRY$
INSERT TSN LIST$
IND STATS$
IND ONLINE$
INDPART PARAM$

我们希望统计出来的数据为

```SQL



dy              count
--------      ------------
IND               2 
INVALIDATION      1
REGISTRY$         1
INSERT            1
TSN               1
LIST$             1
STATS$            1
ONLINE$           1
INDPART           1
PARAM$            1

```

此时我们使用如下语句即可得到结果

## 实现

```sql


SQL> select word,count(*)
  2   from (select   to_char(substr(char_str,instr(char_str,'_',1,level)+1 ,instr(char_str,'_',1,level+1)-instr(char_str,'_',1,level)-1)) word
  3   from
  4  (select '_'||replace(listagg(table_name,',') within group  (order by table_name),',','_')||'_' char_str
  5   from dba_tables
  6   where INSTR(TABLE_NAME,'_') > 1
  7     and table_name like 'IN%')
  8  connect by level <= length(char_str)-length(replace(char_str,'_',null))-1 )
  9  group by word
 10  order by count(*) desc ;
WORD                                                                               COUNT(*)
-------------------------------------------------------------------------------- ----------
IND                                                                                       2
PARAM$                                                                                    1
STATS$                                                                                    1
INSERT                                                                                    1
LIST$                                                                                     1
INVALIDATION                                                                              1
REGISTRY$                                                                                 1
ONLINE$                                                                                   1
INDPART                                                                                   1
TSN                                                                                       1

```

