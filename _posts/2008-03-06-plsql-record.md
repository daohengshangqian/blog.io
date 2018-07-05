---
layout: post
title:  PL/SQL中的记录
date: 2008-03-06
categories: blog
tags: [PL/SQL]
description: PL/SQL中的记录
---


# 记录

# 记录的概念
记录是一种复合数据类型.记录类似与一个一行多列的表.

当然我们也可以声明记录类型为一行一列,不过并没有实际意义,这种情况下我们应该使用变量.

我们可以通过三种方式定义记录

- 基于表的记录
- 基于游标的记录
- 用户自定义记录

## 基于表的记录

我们使用表名%rowtype来定义基于表的记录



例程:使用变量接收数据
```SQL
declare
  v_deptNo dept.deptno%type;
  v_dname  dept.dname%type;
  v_loc    dept.loc%type;
begin
  SELECT * INTO V_DEPTNO, V_DNAME, V_LOC FROM DEPT WHERE ROWNUM = 1;
  DBMS_OUTPUT.put_line(V_DEPTNO);
  DBMS_OUTPUT.put_line(V_DNAME);
  DBMS_OUTPUT.put_line(V_LOC);
end;
```

执行结果:

```sql

10
ACCOUNTING
NEW YORK

PL/SQL procedure successfully completed

```
例程:
使用基于表的记录类型接收数据

```sql
declare
 R_DEPT DEPT%ROWTYPE ;
begin
  SELECT * 
    INTO R_DEPT
    FROM DEPT 
   WHERE ROWNUM = 1;
 DBMS_OUTPUT.put_line(r_dept.deptno);   
 DBMS_OUTPUT.put_line(r_dept.dname);
 DBMS_OUTPUT.put_line(r_dept.loc);  
end;

```

执行结果:
```sql

10
ACCOUNTING
NEW YORK

PL/SQL procedure successfully completed

```

我们来讲解一下上面的例程


```
graph LR
发现表名rowtype关键字-->查找表结构
查找表结构-->根据表结构构造记录结构

```

查看表结构
```sql

SQL> desc dept ;
 Name            Null?    Type
 --------------- -------- ----------------------------
 DEPTNO          NOT NULL NUMBER(2)
 DNAME                    VARCHAR2(14)
 LOC                      VARCHAR2(13)

```


- 记录结构


deptno |  dname |loc
---|--- |---
NUMBER(2) |  VARCHAR2(14) |VARCHAR2(13)
NULL | NULL | NULL 





## 基于游标的记录


```
graph LR
发现游标rowtype关键字-->查找游标引用的列
查找游标引用的列--> 根据列名查找数据类型
根据列名查找数据类型-->构造记录类型

```

例程:基于游标的记录

```sql
DECLARE
 CURSOR C1 IS  
  SELECT DEPTNO,DNAME FROM DEPT ;
 R_C1 C1%ROWTYPE ;
BEGIN
 OPEN  C1 ;
 FETCH C1 INTO R_C1 ;
 CLOSE C1 ;
  DBMS_OUTPUT.put_line(R_C1.DEPTNO) ;
  DBMS_OUTPUT.put_line(R_C1.DNAME) ;  
END ;
   

```

执行结果
```sql

10
ACCOUNTING

PL/SQL procedure successfully completed

```

## 用户自定义记录

这种记录类型多与动态SQL游标匹配使用

```
graph LR

定义数据类型RECORD_TYPE -->根据类型定义变量R_USER_DEFINE
```
例程:用户自定记录

```SQL
DECLARE
  CURSOR C1 IS
    SELECT DEPTNO, DNAME FROM DEPT;
  TYPE RECORD_TYPE IS RECORD(
    DEPTNO NUMBER(2),
    DNAME  VARCHAR2(14));
  R_USER_DEFINE RECORD_TYPE;
BEGIN
  OPEN C1;
  FETCH C1
    INTO R_USER_DEFINE;
  CLOSE C1;
  DBMS_OUTPUT.put_line(R_USER_DEFINE.DEPTNO);
  DBMS_OUTPUT.put_line(R_USER_DEFINE.DNAME);
END;

```

## 记录的兼容性

```sql

DECLARE
  TYPE RECORD_TYPE1 IS RECORD(
    DEPTNO NUMBER(2),
    DNAME  VARCHAR2(14));
  TYPE RECORD_TYPE2 IS RECORD(
    DEPTNO NUMBER(2),
    DNAME  VARCHAR2(14));  
      
  R_USER_DEFINE_1 RECORD_TYPE1;
  R_USER_DEFINE_2 RECORD_TYPE2;
BEGIN
  R_USER_DEFINE_1.deptno :=10;
  R_USER_DEFINE_1.DNAME  :='AAA';
  -- 此处引发报错
  R_USER_DEFINE_2 :=R_USER_DEFINE_1 ;
  
END;

```

由于
R_USER_DEFINE_1 数据类型为RECORD_TYPE1 
R_USER_DEFINE_2 数据类型为RECORD_TYPE2 

虽然数据类型RECORD_TYPE1与RECORD_TYPE2 结构一致,但是Oracle并不会判定由两个类型构造出来的记录变量类型一致.

当然这种情况我们很少遇到,更多的时候我们只会定义一个类型,然后根据这个数据类型构造两个记录变量. 参见例程:记录间赋值

执行结果
```sql
ORA-06550: line 15, column 21:
PLS-00382: expression is of wrong type
ORA-06550: line 15, column 3:
PL/SQL: Statement ignored
```

例程:记录间赋值1 

```sql
DECLARE
  TYPE RECORD_TYPE1 IS RECORD(
    DEPTNO NUMBER(2),
    DNAME  VARCHAR2(14));

  R_USER_DEFINE_1 RECORD_TYPE1;
  R_USER_DEFINE_2 RECORD_TYPE1;
BEGIN
  R_USER_DEFINE_1.deptno :=10;
  R_USER_DEFINE_1.DNAME  :='AAA';
  
  --此处直接使用记录赋值
  R_USER_DEFINE_2 :=R_USER_DEFINE_1;
  
  DBMS_OUTPUT.put_line(R_USER_DEFINE_2.DEPTNO);
  DBMS_OUTPUT.put_line(R_USER_DEFINE_2.DNAME);  
END;

```
执行结果
```sql
10
AAA

PL/SQL procedure successfully completed
```

例程:记录间赋值2

下面这种方法适用于不同类型的记录类型.

```sql
DECLARE
  TYPE RECORD_TYPE1 IS RECORD(
    DEPTNO NUMBER(2),
    DNAME  VARCHAR2(14));
  TYPE RECORD_TYPE2 IS RECORD(
    DEPTNO NUMBER(2),
    DNAME  VARCHAR2(14));  
      
  R_USER_DEFINE_1 RECORD_TYPE1;
  R_USER_DEFINE_2 RECORD_TYPE2;
BEGIN
  R_USER_DEFINE_1.deptno :=10;
  R_USER_DEFINE_1.DNAME  :='AAA';
  
  R_USER_DEFINE_2.DEPTNO :=R_USER_DEFINE_1.DEPTNO ;
  R_USER_DEFINE_2.DNAME :=R_USER_DEFINE_1.DNAME ;
  
  DBMS_OUTPUT.put_line(R_USER_DEFINE_2.DEPTNO);
  DBMS_OUTPUT.put_line(R_USER_DEFINE_2.DNAME);  
END;

```
执行结果

```sql
10
AAA

PL/SQL procedure successfully completed

```

## 嵌套记录

将一种记录类型嵌套另一个记录类型的定义部分,可以构造出嵌套记录

```sql

DECLARE
  TYPE RECORD_TYPE1 IS RECORD(
    DEPTNO NUMBER(2),
    DNAME  VARCHAR2(14));
  TYPE RECORD_TYPE2 IS RECORD(
    COL1 RECORD_TYPE1 ,  --将类型1的记录放入到类型2的定义部分
    LOC  VARCHAR2(13));  
  R_USER_DEFINE_1 RECORD_TYPE2;
BEGIN
  R_USER_DEFINE_1.COL1.deptno :=10;
  R_USER_DEFINE_1.COL1.DNAME  :='AAA';
  R_USER_DEFINE_1.LOC  :='CCCC';
  
  DBMS_OUTPUT.put_line(R_USER_DEFINE_1.COL1.deptno );
  DBMS_OUTPUT.put_line(R_USER_DEFINE_1.COL1.DNAME );
  DBMS_OUTPUT.put_line(R_USER_DEFINE_1.LOC );
END;

```

执行结果
```sql

10
AAA
CCCC

PL/SQL procedure successfully completed
`
