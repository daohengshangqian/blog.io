---
layout: post
title:  PL/SQL中的存储过程
date: 2008-04-03
categories: blog
tags: [PL/SQL]
description: PL/SQL中的存储过程。
---






##  存储过程与匿名块的区别

存储过程是命名块,可以存储与数据库中,存储过程创建成功后,以后想要执行存储过程中的语句,直接调用存储过程即可.

而命名块需要重新编写或者粘贴执行.

存储过程与匿名块的写法非常类似,只有两点不同

- 存储过程需要创建,所以需要执行create proceudre 语句
- 存储过程可以有0-N个参数,匿名块没有参数

## 参数的类型

- IN       输入参数      不可赋值
- OUT      输出参数      可以赋值
- IN OUT   输入输出参数  可以赋值

例程:无参数存储过程

```sql
CREATE OR REPLACE PROCEDURE P0 IS
BEGIN
   DBMS_OUTPUT.put_line('HELLO DAO');
END ;
```

调用过程
```sql
exec p0 ; 
```
执行结果
```sql
HELLO DAO
PL/SQL procedure successfully completed

```

例程:IN 类型参数 与 OUT 类型参数

```sql
create or replace procedure p1(v1 number, v2 number, v3 out number) is
begin
  v3 := v1 + v2;
  dbms_output.put_line(v3);
end;

```
调用过程
```sql
DECLARE
  RES NUMBER;
BEGIN
   P1(1, 2,RES);
END ;

```
执行结果
```sql
3
PL/SQL procedure successfully completed
```

例程:IN  OUT 类型参数

```sql
create or replace procedure p2
       (v1 in out number,v2 in number)
       is
begin
  v1:=v1+v2;
  dbms_output.put_line('v1='||v1);
end;
```

调用过程
```sql
DECLARE
  RES NUMBER :=5;
BEGIN
   P2(RES, 2);
END ;

```

执行结果
```sql
v1=7
PL/SQL procedure successfully completed
```
