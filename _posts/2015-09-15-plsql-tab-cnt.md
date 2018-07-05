---
layout: post
title: PL/SQL脚本-精确统计表行数
date: 2015-09-15
categories: blog
tags: [SQL,面试题]
description: PL/SQL脚本-精确统计表行数
---

# 建立表


```sql

SQL> create table table_row_count
  2  (stat_date date ,
  3    table_owner varchar2(40),
  4   table_name  varchar2(40),
  5  table_count  number
  6  ) ;
Table created


```


# 写脚本将结果插入的表中

```sql

SQL> begin
  2  
  3    for r in (select *
  4                from dba_tables
  5               where owner in
  6                     (select username
  7                        from dba_users
  8                       where created > (select created
  9                                          from dba_users
 10                                         where username = 'SCOTT'))) loop
 11      --dbms_output.put_line('insert into table_row_count  select sysdate, '''||r.owner||''', '''||r.table_name||''', count(*)  from  '||r.owner||'.'||r.table_name  );
 12    BEGIN
 13      execute immediate 'insert into table_row_count  select sysdate, ''' ||
 14                        r.owner || ''', ''' || r.table_name ||
 15                        ''', count(*)  from  ' || r.owner || '.' ||
 16                        r.table_name;
 17      EXCEPTION
 18      WHEN OTHERS  THEN
 19       execute immediate  'insert into table_row_count  select sysdate, '''||r.owner||''', '''||r.table_name||''', NULL  from  DUAL ' ;
 20    END ;
 21    end loop;
 22    COMMIT ;
 23  end;
 24  /
PL/SQL procedure successfully completed

```


# 查询表中的结果

```sql

SQL> select * from  table_row_count where rownum<=10 ;
STAT_DATE   TABLE_OWNER                              TABLE_NAME                               TABLE_COUNT
----------- ---------------------------------------- ---------------------------------------- -----------
2018/1/31 1 DAO                                      TABLE_ROW_COUNT                                    0
2018/1/31 1 OE                                       CUSTOMERS                                        319
2018/1/31 1 OE                                       ORDER_ITEMS                                      665
2018/1/31 1 IX                                       AQ$_ORDERS_QUEUETABLE_T                            0
2018/1/31 1 PM                                       PRINT_MEDIA                                        4
2018/1/31 1 OE                                       PRODUCT_INFORMATION                              288
2018/1/31 1 OE                                       INVENTORIES                                     1112
2018/1/31 1 DAO                                      TEST_SKIP                                    1000000
2018/1/31 1 SH                                       CHANNELS                                           5
2018/1/31 1 IX                                       AQ$_ORDERS_QUEUETABLE_G                            0
10 rows selected

SQL> 

```
