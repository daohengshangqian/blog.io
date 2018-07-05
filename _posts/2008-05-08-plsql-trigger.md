---
layout: post
title:  PL/SQL中的触发器
date: 2008-05-08
categories: blog
tags: [PL/SQL]
description:  PL/SQL中的触发器。
---
数据库触发器是存储在数据库中的命名PL/SQL语句块，当触发事件发生时它们会隐含地执行。

触发事件可以是如下任何一种:

- DML语句
- DDL触发器
- 系统事件
- 用户事件

## 简单DML触发器

其中由DML语句触发的触发器我们称之为简单DML触发器.

我们先来看下触发器的语法:

```sql

CREATE [OR REPLACE] TIGGER  TIGGER_name
[before|after] 
[insert|update|delete]  --触发事件
ON table_name
[FOR EACH ROW]  --标识为行级触发器 
WHEN ()         --触发条件
BEGIN
  
END;

```

- BEFORE & AFTER  触发器在触发事件之前执行,还是之后执行
- FOR EACHR ROW    有此关键字为行级触发器

### DML触发器分类

- 语句前触发器

```sql

CREATE OR REPLACE TRIGGER TRI_BEFORE_STATMENT
BEFORE DELETE  ON DEPT 
BEGIN
  DBMS_OUTPUT.put_line('BEFORE_STATMENT');
END ;

```

- 行前触发器

```SQL
CREATE OR REPLACE TRIGGER TRI_BEFORE_ROW
BEFORE DELETE  ON DEPT 
FOR EACH ROW
BEGIN
  DBMS_OUTPUT.put_line('BEFORE_ROW');
END ;
```


- 行后触发器

```SQL

CREATE OR REPLACE TRIGGER TRI_AFTER_ROW
AFTER DELETE  ON DEPT 
FOR EACH ROW
BEGIN
  DBMS_OUTPUT.put_line('AFTER_ROW');
END ;

```

- 语句后触发器

```sql

CREATE OR REPLACE TRIGGER TRI_AFTER_STATMENT
AFTER DELETE  ON DEPT 
BEGIN
  DBMS_OUTPUT.put_line('AFTER_STATMENT');
END ;

```

执行结果

```sql

SQL> DELETE FROM DEPT ;
BEFORE_STATMENT
BEFORE_ROW
AFTER_ROW
BEFORE_ROW
AFTER_ROW
BEFORE_ROW
AFTER_ROW
BEFORE_ROW
AFTER_ROW
AFTER_STATMENT
4 rows deleted

```

根据上面的例子我们可以确定触发器的执行顺序

1. 语句前触发器
2. 行前触发器
3. 行后触发器
4. 语句后触发器
 
### 在触发器引用更改的记录

在行级触发器中,可以使用NEW OLD关键字,引用改变的记录


语句类型| NEW |  OLD
---|--- | ---
INSERT | 有值 | 空值
UPDATE | 有值 | 有值
DELETE | 空值 | 有值


```sql

CREATE OR REPLACE TRIGGER TRI_OLD_NEW
AFTER UPDATE  ON EMP 
FOR EACH ROW 
BEGIN
  DBMS_OUTPUT.put_line('SAL CHANGED '||:OLD.SAL||'---->'||:NEW.SAL);
END ;


```

执行结果

```sql

SQL>  update emp set sal=sal+2 where sal <=1500 ;
SAL CHANGED 800---->802
SAL CHANGED 1250---->1252
SAL CHANGED 1250---->1252
SAL CHANGED 1500---->1502
SAL CHANGED 1100---->1102
SAL CHANGED 950---->952
SAL CHANGED 1300---->1302
7 rows updated

SQL> rollback ;
Rollback complete

```

### DML触发器综合应用

#### 利用触发器实现自增列

```SQL

CREATE TABLE TEST_AUTO_INCR(C1 NUMBER ,C2 NUMBER ) ;

CREATE SEQUENCE SEQ_AUTO_INCR ;

CREATE OR REPLACE TRIGGER TRI_AUTO_INCR
BEFORE INSERT  ON TEST_AUTO_INCR 
FOR EACH ROW 
BEGIN
  :NEW.C1 := SEQ_AUTO_INCR.NEXTVAL ;
END ;

```

插入数据

```sql

INSERT INTO TEST_AUTO_INCR VALUES(2,1) ;
INSERT INTO TEST_AUTO_INCR VALUES(NULL,2) ;
INSERT INTO TEST_AUTO_INCR(C2) VALUES(3) ;
```

查询结构

```sql

SQL> SELECT * FROM TEST_AUTO_INCR ;
        C1         C2
---------- ----------
         1          1
         2          2
         3          3

```

#### 利用触发器监视工资列修改

- 建表
```SQL

CREATE TABLE SAL_CHANGE_INTO
(ENAME VARCHAR2(25),
 CHANGE_TIME DATE ,
 OLD_SAL  NUMBER ,
 NEW_SAL  NUMBER );

END ;

```
- 建立触发器
```
CREATE OR REPLACE TRIGGER TRI_MON_SAL
BEFORE  UPDATE  ON EMP
FOR EACH ROW
WHEN (NEW.SAL !=OLD.SAL) -- 当工资列发生变动时才触发
DECLARE
  PRAGMA  AUTONOMOUS_TRANSACTION;  --自治事务
BEGIN
  INSERT INTO SAL_CHANGE_INTO 
  VALUES (:OLD.ENAME,SYSDATE,:OLD.SAL,:NEW.SAL);
  COMMIT ;
END ;

```
- 修改数据
```sql 
 UPDATE EMP SET SAL=SAL +200 WHERE SAL >2000  ;
 
```
- 查看结果

```sql
SQL> SELECT * FROM  SAL_CHANGE_INTO ;
ENAME                     CHANGE_TIME    OLD_SAL    NEW_SAL
------------------------- ----------- ---------- ----------
JONES                     2017/12/12        2975       3175
BLAKE                     2017/12/12        2850       3050
CLARK                     2017/12/12        2450       2650
SCOTT                     2017/12/12        3000       3200
KING                      2017/12/12        5000       5200
FORD                      2017/12/12        3000       3200
6 rows selected
 
```

## DML替代触发器

在复杂视图中,不可以进行DML操作,此时我们使用替代触发器实现相同的目的.

- 创建复杂视图

```sql

CREATE VIEW V_AVG_SAL AS 
SELECT DEPTNO,AVG(SAL) AVG_SAL
  FROM EMP
 GROUP BY DEPTNO ;
 
```
- 对视图进行DML 

不可以对复杂视图进行DML,此处引发报错
```SQL

SQL> DELETE FROM V_AVG_SAL WHERE DEPTNO=10 ;
DELETE FROM V_AVG_SAL WHERE DEPTNO=10
ORA-01732: data manipulation operation not legal on this view

```

- 创建替代触发器

```sql

CREATE OR REPLACE TRIGGER TRI_AVG_SAL
  INSTEAD OF DELETE ON V_AVG_SAL

BEGIN
  DELETE FROM EMP WHERE  DEPTNO = :OLD.DEPTNO ;
END;

```

- 查看测试数据

```SQL
SQL> SELECT * FROM EMP WHERE DEPTNO=10 ;
EMPNO ENAME      JOB         MGR HIREDATE          SAL      COMM DEPTNO
----- ---------- --------- ----- ----------- --------- --------- ------
 7782 CLARK      MANAGER    7839 1981/6/9      2450.00               10
 7839 KING       PRESIDENT       1981/11/17    5000.00               10
 7934 MILLER     CLERK      7782 1982/1/23     1300.00               10


```

- 在复杂视图上进行DML
```SQL

SQL> DELETE FROM V_AVG_SAL WHERE DEPTNO=10 ; 
1 row deleted

```

- 查看测试数据
```SQL
SQL>  SELECT * FROM EMP WHERE DEPTNO=10 ;
EMPNO ENAME      JOB         MGR HIREDATE          SAL      COMM DEPTNO
----- ---------- --------- ----- ----------- --------- --------- ------
```
