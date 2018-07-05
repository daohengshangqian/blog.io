---
layout: post
title:  PL/SQL中的自定义函数
date: 2008-04-17
categories: blog
tags: [PL/SQL]
description:  PL/SQL中的自定义函数。
---

## 函数和过程的区别

- 函数可以出现在SQL中,过程不能出现在SQL中
- 函数必须有一个返回值
- 一般来说函数不包含DML操作,DML操作多集中于存储过程中

```sql
CREATE OR REPLACE FUNCTION GET_NUM(C1 NUMBER, C2 NUMBER)
 RETURN NUMBER
IS
BEGIN
  RETURN C1 + C2;
END;

```
调用函数1
```sql

SELECT EMPNO,ENAME,DEPTNO,GET_NUM(EMPNO,DEPTNO)
  FROM EMP ;
```


执行结果1
```sql
EMPNO ENAME      DEPTNO GET_NUM(EMPNO,DEPTNO)
----- ---------- ------ ---------------------
 7369 SMITH          20                  7389
 7499 ALLEN          30                  7529
 7521 WARD           30                  7551
 7566 JONES          20                  7586
 7654 MARTIN         30                  7684
 7698 BLAKE          30                  7728
 7782 CLARK          10                  7792
 7788 SCOTT          20                  7808
 7839 KING           10                  7849
 7844 TURNER         30                  7874
 7876 ADAMS          20                  7896
 7900 JAMES          30                  7930
 7902 FORD           20                  7922
 7934 MILLER         10                  7944
 
```

调用函数2
```sql

DECLARE
  V1 NUMBER;
BEGIN
  V1 := GET_NUM(1, 2);
  DBMS_OUTPUT.put_line(V1);
END;

```

执行结果2

```sql

3
PL/SQL procedure successfully completed

```
## 函数编写的注意事项

- 函数必须执行RETURN 语句

```sql

CREATE OR REPLACE FUNCTION GET_SAL(I_ENAME EMP.ENAME%TYPE )
  RETURN EMP.SAL%TYPE
  IS
O_SAL EMP.SAL%TYPE;
BEGIN
 SELECT SAL
   INTO O_SAL
   FROM EMP
  WHERE ENAME =I_ENAME ;
 RETURN O_SAL ;
EXCEPTION
  WHEN OTHERS THEN DBMS_OUTPUT.put_line('EMPLOYEE NOT EXISTS!');
END;


```

调用函数1

```sql

declare
  v_sal number;
begin
  v_sal := get_sal('KING');
  DBMS_OUTPUT.put_line(V_SAL);
end;
```

执行结果1

```sql

5000
PL/SQL procedure successfully completed

```

调用函数2

```sql
declare
  v_sal number;
begin
  v_sal := get_sal('DAO');  
  DBMS_OUTPUT.put_line(V_SAL);
end;

```
执行结果2

```sql
ORA-06503: PL/SQL: Function returned without value
ORA-06512: at "DAO.GET_SAL", line 13
ORA-06512: at line 4

```
报错原因为我们传入了一个不存在的ENAME

SELECT INTO 语句会引发一个 NO_DATA_FOUND 异常

在异常处理部分没有RETURN语句.所以报错.

解决方法很简单

```sql

CREATE OR REPLACE FUNCTION GET_SAL(I_ENAME EMP.ENAME%TYPE )
  RETURN EMP.SAL%TYPE
  IS
O_SAL EMP.SAL%TYPE;
BEGIN
 SELECT SAL
   INTO O_SAL
   FROM EMP
  WHERE ENAME =I_ENAME ;
 RETURN O_SAL ;
EXCEPTION
  WHEN OTHERS THEN DBMS_OUTPUT.put_line('EMPLOYEE NOT EXISTS!');
  RETURN NULL ;  --新增此行
END;


```

调用函数3

```sql
declare
  v_sal number;
begin
  v_sal := get_sal('DAO');
  DBMS_OUTPUT.put_line(V_SAL);
end;

```

执行结果3

```sql
EMPLOYEE NOT EXISTS!

PL/SQL procedure successfully completed

```

- 如果函数存在DML语句则不能在SQL中调用 


```sql
CREATE OR REPLACE FUNCTION FUNC_TEST_DML RETURN NUMBER IS
BEGIN
  DELETE FROM EMP;
  RETURN SQL%ROWCOUNT;
END;

```
调用函数1

```sql
SELECT D.*,FUNC_TEST_DML FROM DEPT D WHERE ROWNUM<=1 ;

```

执行结果1 

```sql
ORA-14551: cannot perform a DML operation inside a query 
ORA-06512: at "DAO.FUNC_TEST_DML", line 3

```

大部分的开发规范都要求
- DML语句放在存储过程中
- 查询语句可以放在函数中

以上函数在PL/SQL语句块中不存在问题

```调用函数2 

declare
  v1 number;
begin
  v1 := FUNC_TEST_DML;
  dbms_output.put_line(v1);
end;

```

执行结果2

```sql
14
PL/SQL procedure successfully completed

SQL> select * from emp ;
EMPNO ENAME      JOB         MGR HIREDATE          SAL      COMM DEPTNO
----- ---------- --------- ----- ----------- --------- --------- ------

SQL> rollback ;
Rollback complete

```
