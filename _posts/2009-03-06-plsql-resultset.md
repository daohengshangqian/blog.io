---
layout: post
title:  PL/SQL返回结果集的三种方式
date: 2009-03-06
categories: blog
tags: [PL/SQL]
description:  PL/SQL返回结果集的三种方式
---

PL/SQL在使用中经常需要传递和返回结果集，也就是多行记录
通常有以下三种方式

- 使用REF CURSOR
- 使用行集
- 将数据插入的临时表中，返回针对临时表的查询语句

下面请看实例

- 包的定义部分
```SQL

create or replace package p111 is
  - 定义一个ref cursor 
  type ref_cursor is ref cursor;
  
  - 定义一个行集 行集中每个行的结构和emp表相同 在实际应用中可能会使用用户自定的记录格式
  type row_table is table of emp%rowtype;
  procedure proc_1(o_res out ref_cursor);

  procedure proc_2(o_res out row_table);

  procedure proc3(o_sql out varchar2);

end;


```

- 包的实现部分 

```sql

create or replace package body p111
is

-- 返回游标i
procedure proc_1 (o_res out ref_cursor) 
is
begin
 open o_res for select * from  emp ;
end ;

-- 返回行集
procedure proc_2 (o_res out row_table) 
is

begin
 select  *
  bulk collect into o_res
   from emp ;
 end ;
 
 -- 返回对存放结果集中临时表的查询语句
 procedure proc3  (o_sql out varchar2) 
   is
   begin
     insert into temp_res
     select * from emp ;
     
     o_sql :='select * from temp_res' ;
   end ;
 
end ;



```

- 附临时表的创建语句

```sql
SQL> create global temporary table  temp_res
  2  on commit delete rows as select * from scott.emp  ;
Table created

```