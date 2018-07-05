---
layout: post
title: PL/SQL的概念
date: 2007-12-25
categories: blog
tags: [PL/SQL]
description: PLSQL的概念
---

## PLSQL概述
PL/SQL是对SQL语言的过程式语言扩展.

他是Oracle 数据库的一部分,并不是一个独立的编程语言.

目前PL/SQL在编程语言排行榜中排名20名左右.
## PLSQL结构

PLSQL 块由两部分构成

第一部分为声明部分[可选]
第二部分为执行部分
其中第二部分中可包含异常处理部分[可选]

注意:在sqlplus中需要事先开启输出,否则看不到运行结果

```sql

set serveoutput on 

```

例程1 只有执行部分

``` sql

begin
 dbms_output.put_line('Hello World!');
end ;
/

```

例程2 包含声明部分与异常处理部分

-  declare   与 begin 之间为声明部分
-  begin     与 end   之间为可执行部分
-  exception 与 end   之间为异常处理部分
      
```sql

declare
v1 number :=&v1;
v2 number :=&v2 ;
begin
v1 :=v1/v2 ;
exception 
 when others 
  then dbms_output.put_line(sqlerrm);
end ;
/

```
## 匿名块与命名块

PL/SQL语句块可以分为两种:命名块和匿名块.
- 命名块包含过程 函数 包 ,命名块存储与数据库中,通过名称调用
- 匿名块以declare或begin开头,不存在与数据库中,每次调用都需重新书写

## PLSQL的执行

PL/SQL可被其他语言调用,可以在SQLPLUS , PL/SQL DEVELOPER中执行.
需要注意的是:
1. 如需要使用打印功能(DBMS_OUTPUT)则需要在sqlplus中开启

```sql

set serveroutput on size unlimited 

```

2. PLSQL匿名块的结束符为/ 在调用时必须以/结尾

```sql
SQL> set serveroutput on
SQL> begin
  2   dbms_output.put_line('Hello World!');
  3  end ;
  4  /

Hello World!

PL/SQL procedure successfully completed

```








