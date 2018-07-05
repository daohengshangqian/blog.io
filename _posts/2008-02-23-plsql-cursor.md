---
layout: post
title:  PL/SQL中的游标
date: 2008-02-23
categories: blog
tags: [PL/SQL]
description: PL/SQL中的游标。
---

## 游标的概念

游标（cursor）可以看作是私有SQL区的句柄或名称，在SQL语句执行过程中保存解析后的语句和其它信息.

## 隐式游标与显式游标

游标分为隐式游标与显式游标

- 隐式游标: 当运行DML与 SELECT INTO 语句时,PLSQL会打开一个隐式游标.隐式游标自动打开,自动关闭无法人工干预
- 显式游标: 用户在代码中定义的游标,并由用户在程序中控制打开,检索,关闭.

## 游标的属性

游标属性 | 隐式游标 | 显式游标
---|---|---
ISOPEN    | 永远为FALSE                  | 游标处于打开状态为TRUE,反之为FALSE
FOUND     | 涉及到数据为TRUE 反之为FALSE | 可以提取到数据为TRUE,反之为FALSE 
NOT FOUND | 与上面含义相反               | 与上面含义相反
ROWCOUNT  | 返回涉及到数据的总行数       | 累计从结果集中提取到多少行

### 隐式游标属性
由于没有显示的声明游标,所以隐式游标也没有名字,引用隐式游标属性只能使用关键字SQL.

如果语句块中存在多个DML 或 SELECT INTO 则隐式游标属性为最近执行的SQL的属性

例程:ROWCOUNT

```SQL
BEGIN
  DELETE FROM SCOTT.EMP;
  DBMS_OUTPUT.put_line(SQL%ROWCOUNT);  --当前游标属性为删除EMP表的属性
  DELETE FROM SCOTT.DEPT;
  DBMS_OUTPUT.put_line(SQL%ROWCOUNT);  --当前游标属性为删除DEPT表的属性
  ROLLBACK ;
END;

```

执行结果

```sql
14
4

PL/SQL procedure successfully completed

```

例程:ISOPEN 

```SQL

 BEGIN
  DELETE FROM SCOTT.EMP;   --隐式游标自动打开关闭,所以ISOPEN 状态永远为FALSE 
  IF SQL%ISOPEN THEN 
    DBMS_OUTPUT.put_line('CURSOR OPEN');
   ELSE 
    DBMS_OUTPUT.put_line('CURSOR CLOSE!');
  END IF ;
  ROLLBACK ;
END;

```
执行结果

```SQL

CURSOR CLOSE!

PL/SQL procedure successfully completed

```

例程:FOUND 
```SQL

BEGIN
  DELETE FROM SCOTT.EMP;
  IF SQL%FOUND  THEN 
    DBMS_OUTPUT.put_line('ROWS FOUND!');
   ELSE 
    DBMS_OUTPUT.put_line('ROWS NOT FOUND!');
  END IF ;
  ROLLBACK ;
END;

```
执行结果
```SQL
ROWS FOUND!

PL/SQL procedure successfully completed
```

例程:NOTFOUND
```SQL
BEGIN
  DELETE FROM SCOTT.EMP WHERE 1=2 ;
  IF SQL%NOTFOUND  THEN 
    DBMS_OUTPUT.put_line('ROWS NOT  FOUND! ');
   ELSE 
    DBMS_OUTPUT.put_line('ROWS  FOUND!');
  END IF ;
  ROLLBACK ;
END;

```
执行结果
```SQL
ROWS NOT  FOUND! 

PL/SQL procedure successfully completed
``` 

### 显示游标属性


例程:ISOPEN


```SQL

 BEGIN
  DELETE FROM SCOTT.EMP;   --隐式游标自动打开关闭,所以ISOPEN 状态永远为FALSE 
  IF SQL%ISOPEN THEN 
    DBMS_OUTPUT.put_line('CURSOR OPEN');
   ELSE 
    DBMS_OUTPUT.put_line('CURSOR CLOSE!');
  END IF ;
  ROLLBACK ;
END;
```
执行结果
```sql
CURSOR CLOSE!

PL/SQL procedure successfully completed

```


```SQL

DECLARE
 CURSOR C1 IS SELECT * FROM EMP ;
BEGIN
  IF C1%ISOPEN 
  THEN  
    DBMS_OUTPUT.put_line('BEFORE OPEN');
  END IF ;
  OPEN C1 ;
  IF C1%ISOPEN 
  THEN  
    DBMS_OUTPUT.put_line('AFTER OPEN');
  END IF ;  
  CLOSE C1 ;
  IF C1%ISOPEN 
  THEN  
    DBMS_OUTPUT.put_line('AFTER CLOSE');
  END IF ;  

END;

```

执行结果
```sql

AFTER OPEN

PL/SQL procedure successfully completed

```

例程:FOUND
```SQL
DECLARE
  CURSOR C1 IS
    SELECT ENAME FROM SCOTT.EMP WHERE ROWNUM <= 2;
  V_ENAME SCOTT.EMP.ENAME%TYPE;
BEGIN
  OPEN C1;
  IF C1%FOUND THEN
    DBMS_OUTPUT.put_line('BEFORE FETCH FOUND !');
  END IF;

  FETCH C1
    INTO V_ENAME;
  IF C1%FOUND THEN
    DBMS_OUTPUT.put_line('1 FETCH  FOUND !');
  END IF;

  FETCH C1
    INTO V_ENAME;
  IF C1%FOUND THEN
    DBMS_OUTPUT.put_line('2 FETCH  FOUND !');
  END IF;

  FETCH C1   --由于SQL只返回两行,此时FETCH 无法获取最新的数据
    INTO V_ENAME;
  IF C1%FOUND THEN
    DBMS_OUTPUT.put_line('3 FETCH  FOUND !');
  END IF;
CLOSE C1 ;
END;
```
执行结果
```sql

1 FETCH  FOUND !
2 FETCH  FOUND !

PL/SQL procedure successfully completed
```

例程:NOTFOUND
```SQL
DECLARE
  CURSOR C1 IS
    SELECT ENAME FROM SCOTT.EMP WHERE ROWNUM <= 2;
  V_ENAME SCOTT.EMP.ENAME%TYPE;
BEGIN
  OPEN C1;
  IF C1%NOTFOUND THEN
    DBMS_OUTPUT.put_line('BEFORE FETCH NOT FOUND !');
  END IF;

  FETCH C1
    INTO V_ENAME;
  IF C1%FOUND THEN
    DBMS_OUTPUT.put_line('1 FETCH NOT  FOUND !');
  END IF;

  FETCH C1
    INTO V_ENAME;
  IF C1%FOUND THEN
    DBMS_OUTPUT.put_line('2 FETCH NOT  FOUND !');
  END IF;

  FETCH C1
    INTO V_ENAME;
  IF C1%FOUND THEN
    DBMS_OUTPUT.put_line('3 FETCH  NOT FOUND !');
  END IF;
 
 CLOSE C1 ;
END;
```
执行结果
```sql

1 FETCH NOT  FOUND !
2 FETCH NOT  FOUND !

PL/SQL procedure successfully completed

```

例程:ROWCOUNT

```SQL
DECLARE
  CURSOR C1 IS
    SELECT ENAME FROM SCOTT.EMP WHERE ROWNUM <= 2;
  V_ENAME SCOTT.EMP.ENAME%TYPE;
BEGIN
  OPEN C1;
  DBMS_OUTPUT.put_line(C1%ROWCOUNT);  --此处返回0 因为还没提取到数据
  FETCH C1
    INTO V_ENAME;
  DBMS_OUTPUT.put_line(C1%ROWCOUNT);  --此处返回1 已经提取了1行数据
  FETCH C1
    INTO V_ENAME;
  DBMS_OUTPUT.put_line(C1%ROWCOUNT);  --此处返回2 已经提取了2行数据
  FETCH C1
    INTO V_ENAME;
  DBMS_OUTPUT.put_line(C1%ROWCOUNT);  --此处返回2 因为并没有新行被提取
  CLOSE C1;
END;

```
执行结果

```sql
0
1
2
2

PL/SQL procedure successfully completed
```


## 游标循环

游标对应的SQL语句可能会返回很多的行,而游标的只能一次提取一行,此时我们需要使用循环遍历这些行.

### 简单游标循环
例程:简单游标循环
```SQL
DECLARE
  CURSOR C1 IS
    SELECT ENAME FROM SCOTT.EMP ORDER BY ENAME ASC;
  V_ENAME SCOTT.EMP.ENAME%TYPE;
BEGIN
  OPEN C1;
  LOOP
    FETCH C1
      INTO V_ENAME;
    EXIT WHEN C1%NOTFOUND;
    DBMS_OUTPUT.put_line(V_ENAME);
  END LOOP;
  CLOSE C1;
END;

```
执行结果
```SQL


ADAMS
ALLEN
BLAKE
CLARK
FORD
JAMES
JONES
KING
MARTIN
MILLER
SCOTT
SMITH
TURNER
WARD

PL/SQL procedure successfully completed

```

### WHILE 游标循环
例程: WHILE 游标循环
```SQL
DECLARE
  CURSOR C1 IS
    SELECT ENAME FROM SCOTT.EMP ORDER BY ENAME ASC;
  V_ENAME SCOTT.EMP.ENAME%TYPE;
BEGIN
  OPEN C1;
  FETCH C1 INTO V_ENAME ; --由于WHILE循环是先判断后循环,所以需要先FETCH 一次
  WHILE C1%FOUND 
  LOOP
   DBMS_OUTPUT.put_line(V_ENAME);
    FETCH C1
      INTO V_ENAME;
  END LOOP;
  CLOSE C1;
END;
```
执行结果
```SQL
ADAMS
ALLEN
BLAKE
CLARK
FORD
JAMES
JONES
KING
MARTIN
MILLER
SCOTT
SMITH
TURNER
WARD

PL/SQL procedure successfully completed
```
### 游标FOR循环

游标FOR循环比较简单,并且自动打开游标,自动关闭游标,

游标FOR循环有两种实现形式

例程:游标FOR循环1
```SQL
DECLARE
  CURSOR C1 IS
    SELECT ENAME FROM SCOTT.EMP ORDER BY ENAME ASC;
BEGIN
  FOR R IN C1   --R为记录类型
  LOOP
   DBMS_OUTPUT.put_line(R.ENAME);
  END LOOP ;
END;

```
执行结果
```SQL

ADAMS
ALLEN
BLAKE
CLARK
FORD
JAMES
JONES
KING
MARTIN
MILLER
SCOTT
SMITH
TURNER
WARD

PL/SQL procedure successfully completed

```
例程:游标FOR循环2
```sql
BEGIN
  FOR R IN ( SELECT ENAME FROM SCOTT.EMP ORDER BY ENAME ASC)  --R为记录类型
  LOOP
   DBMS_OUTPUT.put_line(R.ENAME);
  END LOOP ;
END;
```

执行结果
```SQL

ADAMS
ALLEN
BLAKE
CLARK
FORD
JAMES
JONES
KING
MARTIN
MILLER
SCOTT
SMITH
TURNER
WARD

PL/SQL procedure successfully completed

```
