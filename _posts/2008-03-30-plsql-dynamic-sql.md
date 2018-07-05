---
layout: post
title:  PL/SQL中的动态SQL
date: 2008-03-30
categories: blog
tags: [PL/SQL]
description: PL/SQL中的动态SQL。
---
## 动态SQL的概念

本质上来说动态SQL就是将符合SQL语法的字符串当作SQL语句来运行.

动态SQL相对于静态SQL来说更灵活.并且可以突破一些PL/SQL对静态SQL的限制.

由于在编译时SQL的全文还不确定，所以你可使用动态SQL构建灵活的，多用途的
应用程序。

## 动态SQL的分类


对于DML,DDL,SELECT INTO 语句我们使用EXECUTE IMMEDIATE 语句来执行;

对于多行查询我们使用OPEN cursor  FOR 语句来执行


## EXECUTE IMMEDIATE 语句

例程:打印表中记录数
```sql
DECLARE
  TAB_ROW_CNT NUMBER;
BEGIN
  FOR R IN (SELECT TABLE_NAME FROM USER_TABLES where rownum <= 5)
   LOOP
    execute immediate 'SELECT COUNT(*) FROM ' || R.TABLE_NAME
      INTO TAB_ROW_CNT;
    DBMS_OUTPUT.put_line(R.TABLE_NAME || ':' || TAB_ROW_CNT);
  END LOOP;
END;
```
执行结果
```sql
DOMAIN_NAME:225884
TEST_CONNECT_CYCLE:6
TEST_FOJ_B:2
TEST_FOJ_A:2
TEST_INSERT_DEPT:4
PL/SQL procedure successfully completed
```

## OPEN cursor_name FOR 语句

- 只能打开REF CURSOR 类型的游标
- 后面多跟随多行查询语句

```sql

declare
  type ref_cursor is ref cursor;
  c1 ref_cursor;
  r1 emp%rowtype;
begin
  open c1 for 'select * from emp ';
  loop
    fetch c1
      into r1;
     dbms_output.put_line(r1.ename||' '||r1.sal);
    exit when c1%notfound;
  end loop;
end;


```
执行结果

```sql

SMITH 800
ALLEN 1600
WARD 1250
JONES 2975
MARTIN 1250
BLAKE 2850
CLARK 2450
SCOTT 3000
KING 5000
TURNER 1500
ADAMS 1100
JAMES 950
FORD 3000
MILLER 1300
MILLER 1300
PL/SQL procedure successfully completed

```


## 绑定变量

执行一下语句
```sql
declare
  v1 number := 10;
  v2 number := 20;
begin
  execute immediate 'select/* dao */ count(*) from emp where deptno=' || v1;
  execute immediate 'select/* dao */ count(*) from emp where deptno=' || v2;
end;

```

查询V$SQL

```SQL

select sql_id,sql_text
 from v$sql
where sql_text like '%dao%'

```

执行结果:
```sql
SQL_ID        SQL_TEXT
------------- --------------------------------------------------------------------------------
8yz8usu65h27s declare   v1 number := 10;   v2 number := 20; begin   execute immediate 'select/
8h22vrrdn546k select/* dao */ count(*) from emp where deptno=10
47tbn5whrjnpp select/* dao */ count(*) from emp where deptno=20
8ztfnk3pf6pp7 select sql_id,sql_text  from v$sql where sql_text like '%dao%'

```
通过上面的例子我们可以发现,意义类似的语句由于传入的部门编号不同,被Oracle定义为两个不同的SQL语句,拥有不同的SQL_ID.

在SHARED POOL中两个SQL各占一份空间,游标不能被共享,会造成硬解析.

当这种SQL被大量调用,会产生大量的硬解析,产生latch争用,占用大量CPU等一系列问题.

Oracle 建议使用绑定变量来避免上述问题.


```sql
 alter system flush shared_pool ;

 declare
  v1 number := 10;
  v2 number := 20;
begin
  execute immediate 'select/* dao */ count(*) from emp where deptno=' || v1;
  execute immediate 'select/* dao */ count(*) from emp where deptno=' || v2;
end;

```

查询V$SQL

```SQL

select sql_id,sql_text
 from v$sql
where sql_text like '%dao%'

```

执行结果:
```sql

SQL_ID        SQL_TEXT
------------- --------------------------------------------------------------------------------
5s7cf3yw5kfh7 select/* dao */ count(*) from emp where deptno= :b1
8ztfnk3pf6pp7 select sql_id,sql_text  from v$sql where sql_text like '%dao%'
bsgk84htrg15g declare   v1 number := 10;   v2 number := 20; begin   execute immediate 'select/


```
