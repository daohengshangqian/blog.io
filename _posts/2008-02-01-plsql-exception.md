---
layout: post
title:  PL/SQL中的异常处理
date: 2008-02-01
categories: blog
tags: [PL/SQL]
description: PL/SQL中的异常处理
---

## 错误的分类
在PL/SQL中可能出现两种错误
- 编译错误   即语法错误 程序编写没有满足语法要求
- 运行时错误 语法没有错误,但在运行时报错.这种错误语法检查器无法监测. 对于运行时错误我们又称之为异常

在PL/SQL的执行部分有一个可选的异常处理部分.

## 异常处理的优点

使程序正常结束而不是中途报错
将错误处理过程独立,增加程序的逻辑性
错误输出更加丰富,友好.

## 预定义异常

Oracle将常见的21个异常定义为预定义异常,并为每个异常指定了一个名字.

PL/SQL块中的异常处理部分通过异常的名字对异常进行捕获处理.

异常名称 | 异常原因 | 错误编号
---|---|---
ACCESS_INTO_NULL     	| 未定义对象                                                          | ORA-06530 -6530
CASE_NOT_FOUND          | CASE 中若未包含相应的WHEN ，并且没有设置ELSE 时		              | ORA-06592 -6592
COLLECTION_IS_NULL      | 集合元素未初始化 	                                                  | ORA-06531 -6531
CURSOR_ALREADY_OPEN	    | 游标已经打开	                                      	              | ORA-06511 -6511
DUP_VAL_ON_INDEX	    | 唯一索引对应的列上有重复的值	 	                                  | ORA-00001 -1
INVALID_NUMBER	        | 无效数字		                                                      | ORA-01722 -1722
INVALID_CURSOR	        | 在不合法的游标上进行操作		                                      | ORA-01001 -1001
NO_DATA_FOUND	        | 使用 select into 未返回行，或应用索引表未初始化的元素时             | ORA-01403 +100
TOO_MANY_ROWS	        | 执行 select into 时，结果集超过一行		                          | ORA-01422 -1422
ZERO_DIVIDE	            | 除数为 0	                                                          | ORA-01476 -1476
SUBSCRIPT_BEYOND_COUNT	| 元素下标超过嵌套表或VARRAY 的最大值		                          | ORA-06533 -6533
SUBSCRIPT_OUTSIDE_LIMIT	| 使用嵌套表或 VARRAY 时，将下标指定为负数		                      | ORA-06532 -6532
VALUE_ERROR	            | 赋值时，变量长度不足以容纳实际数据		                          | ORA-06502 -6502
LOGIN_DENIED            | PL/SQL 应用程序连接到oracle 数据库时，提供了不正确的用户名或密码    | ORA-01017 -1017
NOT_LOGGED_ON	        | PL/SQL 应用程序在没有连接oralce数据库的情况下访问数据	              | ORA-01012 -1012
PROGRAM_ERROR	        | PL/SQL 内部问题，可能需要重装数据字典＆ PL/SQL 系统包	              | ORA-06501 -6501
ROWTYPE_MISMATCH  	    | 宿主游标变量与 PL/SQL 游标变量的返回类型不兼容	                  | ORA-06504 -6504
SELF_IS_NULL	        | 使用对象类型时，在 null 对象上调用对象方法	                      | ORA-30625 -30625
STORAGE_ERROR	        | 运行 PL/SQL 时，超出内存空间	                                      | ORA-06500 -6500
SYS_INVALID_ROWID	        | 无效的 ROWID 字符串	                                              | ORA-01410 -1410
TIMEOUT_ON_RESOURCE	    | Oracle 在等待资源时超时                                             |	ORA-00051 -51


### 预定义异常实例 

#### ZERO_DIVIDE
例程:
```SQL
DECLARE
  V1 NUMBER := 1;
  V2 NUMBER := 0;
  V3 NUMBER;
BEGIN
  V3 := V1 / V2;  --此处0做除数引发异常,直接跳转到异常处理部分
  DBMS_OUTPUT.PUT_LINE('OK1'); --此处不打印OK
EXCEPTION   --异常处理部分
  WHEN ZERO_DIVIDE THEN  --当发现异常是ZERO_DIVIDE时,打印ZERO DIVIDE
    DBMS_OUTPUT.put_line('ZERO DIVIDE');
end;

```

执行结果:

```SQL

ZERO DIVIDE	

PL/SQL procedure successfully completed

```
#### NO_DATA_FOUND

例程:
```sql
DECLARE
  V_1 NUMBER;
BEGIN
  SELECT 1 INTO V_1 FROM DUAL WHERE 1 = 2;
EXCEPTION
  WHEN NO_DATA_FOUND THEN
    DBMS_OUTPUT.PUT_LINE(SQLERRM);
END;



```
执行结果:

```SQL
ORA-01403: no data found
PL/SQL procedure successfully completed
```
#### TOO_MANY_ROWS
例程:
```sql

DECLARE
  V_NUM NUMBER ;
BEGIN
  SELECT 1
    INTO V_NUM 
    FROM DUAL
   CONNECT BY LEVEL<=2 ;
EXCEPTION
  WHEN TOO_MANY_ROWS THEN
    DBMS_OUTPUT.PUT_LINE(SQLERRM);
END;

```
执行结果:

```SQL
ORA-01422: exact fetch returns more than requested number of rows
PL/SQL procedure successfully completed
```

#### CASE_NOT_FOUND
例程:
```sql

DECLARE
  V_1 NUMBER;
BEGIN
  CASE 1
    WHEN 2 THEN
      NULL;
    WHEN 3 THEN
      NULL;
  END CASE;
EXCEPTION
  WHEN CASE_NOT_FOUND THEN
    DBMS_OUTPUT.PUT_LINE(SQLERRM);
END;


```
执行结果:

```SQL

ORA-06592: CASE not found while executing CASE statement
PL/SQL procedure successfully completed

```

#### CURSOR_ALREADY_OPEN
当重复打开游标时会引发此异常

例程:
```sql

DECLARE
 CURSOR C1 IS SELECT * FROM DUAL ;
BEGIN
 OPEN C1 ;
 OPEN C1;
EXCEPTION
  WHEN CURSOR_ALREADY_OPEN THEN
    DBMS_OUTPUT.PUT_LINE(SQLERRM);
END;

```
执行结果:

```SQL
ORA-06511: PL/SQL: cursor already open
PL/SQL procedure successfully completed
```

#### INVALID_CURSOR
例程:
```sql
DECLARE
 CURSOR C1 IS SELECT * FROM DUAL ;
BEGIN
 OPEN C1 ;
 CLOSE C1 ;
 DBMS_OUTPUT.put_line(C1%ROWCOUNT);
EXCEPTION
  WHEN INVALID_CURSOR THEN
    DBMS_OUTPUT.PUT_LINE(SQLERRM);
END;


```
执行结果:

```SQL
ORA-01001: invalid cursor
PL/SQL procedure successfully completed
```

#### DUP_VAL_ON_INDEX
例程:
```sql

DECLARE
  CURSOR C1 IS
    SELECT * FROM DUAL;
BEGIN
 INSERT INTO TEST_PK VALUES(1,1);
 INSERT INTO TEST_PK VALUES(1,1); 
EXCEPTION
  WHEN DUP_VAL_ON_INDEX THEN
    DBMS_OUTPUT.PUT_LINE(SQLERRM);
END;

```
执行结果:

```SQL
ORA-00001: unique constraint (DAO.SYS_C0013692) violated
PL/SQL procedure successfully completed

```
#### INVALID_NUMBER

例程:
```sql
DECLARE
  V_NUM NUMBER ;
BEGIN
  SELECT TO_NUMBER('A')
   INTO V_NUM
    FROM DUAL ; 
EXCEPTION
  WHEN INVALID_NUMBER THEN
    DBMS_OUTPUT.PUT_LINE(SQLERRM);
END;

```
执行结果:

```SQL
ORA-01722: invalid number
PL/SQL procedure successfully completed

```

#### VALUE_ERROR
例程:
```sql

DECLARE
  V_NUM NUMBER ;
BEGIN
  V_NUM :='A';
EXCEPTION
  WHEN VALUE_ERROR THEN
    DBMS_OUTPUT.PUT_LINE(SQLERRM);
END;

```
执行结果:

```SQL

ORA-06502: PL/SQL: numeric or value error: character to number conversion error
PL/SQL procedure successfully completed

```

#### SUBSCRIPT_BEYOND_COUNT
例程:
```sql

DECLARE
  TYPE TYPE_NUM_NT IS TABLE OF NUMBER;
  T1 TYPE_NUM_NT := TYPE_NUM_NT(11);
BEGIN
  T1(2) := 100;
EXCEPTION
  WHEN SUBSCRIPT_BEYOND_COUNT THEN
    DBMS_OUTPUT.PUT_LINE(SQLERRM); 
END;

```
执行结果:

```SQL
ORA-06533: Subscript beyond count
PL/SQL procedure successfully completed
```
#### SUBSCRIPT_OUTSIDE_LIMIT
例程:
```sql
DECLARE
  TYPE TYPE_NUM_NT IS VARRAY(5) OF NUMBER;
  T1 TYPE_NUM_NT := TYPE_NUM_NT(11);
BEGIN
  T1(6) := 100;
EXCEPTION
  WHEN SUBSCRIPT_OUTSIDE_LIMIT THEN
    DBMS_OUTPUT.PUT_LINE(SQLERRM);
END;


```
执行结果:

```SQL
ORA-06532: Subscript outside of limit
PL/SQL procedure successfully completed
```

#### ROWTYPE_MISMATCH
例程:
```sql
DECLARE
  TYPE REF_CURSOR IS REF CURSOR;
  C1      REF_CURSOR;
  V_ENAME EMP.ENAME%TYPE;
BEGIN
  OPEN C1 FOR
    SELECT 1, 2 FROM DUAL;
  FETCH C1
    INTO V_ENAME;
  CLOSE C1;
EXCEPTION
  WHEN ROWTYPE_MISMATCH THEN
    DBMS_OUTPUT.PUT_LINE(SQLERRM);
END;


```
执行结果:

```SQL
ORA-06502: PL/SQL: numeric or value error: character to number conversion error
PL/SQL procedure successfully completed
```
#### ACCESS_INTO_NULL

例程:

```sql
CREATE TYPE address_typ AS OBJECT ( 
   street          VARCHAR2(30),
   city            VARCHAR2(20),
   state           CHAR(2),
   postal_code     VARCHAR2(6) );
/

CREATE TYPE employee_typ AS OBJECT (
  employee_id       NUMBER(6),
  first_name        VARCHAR2(20),
  last_name         VARCHAR2(25),
  email             VARCHAR2(25),
  phone_number      VARCHAR2(20),
  hire_date         DATE,
  job_id            VARCHAR2(10),
  salary            NUMBER(8,2),
  commission_pct    NUMBER(2,2),
  manager_id        NUMBER(6),
  department_id     NUMBER(4),
  address           address_typ,
  MAP MEMBER FUNCTION get_idno RETURN NUMBER,
  MEMBER PROCEDURE display_address ( SELF IN OUT NOCOPY employee_typ ) );
/
CREATE TYPE BODY employee_typ AS
  MAP MEMBER FUNCTION get_idno RETURN NUMBER IS
  BEGIN
    RETURN employee_id;
  END;
  MEMBER PROCEDURE display_address ( SELF IN OUT NOCOPY employee_typ ) IS
  BEGIN
    DBMS_OUTPUT.PUT_LINE(first_name || ' '  || last_name);
    DBMS_OUTPUT.PUT_LINE(address.street);
    DBMS_OUTPUT.PUT_LINE(address.city || ', '  || address.state || ' ' ||
                         address.postal_code);   
  END;
END;
/

DECLARE
  emp employee_typ; -- emp is atomically null
BEGIN
  emp.employee_id := 330;

END;
/


```
执行结果:

```SQL
ORA-06530: Reference to uninitialized composite
ORA-06512: at line 4
```

例程:
```sql

DECLARE
  emp employee_typ; -- emp is atomically null
BEGIN

  emp.employee_id := 330;

EXCEPTION
  WHEN ACCESS_INTO_NULL THEN
    DBMS_OUTPUT.PUT_LINE('Cannot assign value to NULL object');
END;
/


```
执行结果:

```SQL
Cannot assign value to NULL object

```

#### SELF_IS_NULL
例程:
```sql
DECLARE
  EMP    EMPLOYEE_TYP; -- EMP IS ATOMICALLY NULL ;
  V_IDNO NUMBER;
BEGIN
  V_IDNO := EMP.GET_IDNO;
EXCEPTION
  WHEN SELF_IS_NULL THEN
    DBMS_OUTPUT.PUT_LINE('SELF_IS_NULL');
END;
/


```
执行结果:

```SQL

 SELF_IS_NULL
 
```

####  COLLECTION_IS_NULL
例程:

```sql
declare

type ct is table of number ;

collection1 ct ;
begin
   
 collection1(0):=0 ;
 
exception
  when COLLECTION_IS_NULL
    then dbms_output.put_line('COLLECTION_IS_NULL');     
end ;

```
执行结果:

```SQL
COLLECTION_IS_NULL
```

#### ROWTYPE_MISMATCH
例程:

```sql
DECLARE
  TYPE REF_CURSOR IS REF CURSOR ;
  C1 REF_CURSOR ;
  EMP_REC  EMP%ROWTYPE ;
BEGIN
   OPEN C1 FOR SELECT * FROM DEPT ;
   FETCH C1 INTO EMP_REC ;
   CLOSE C1 ;
   EXCEPTION
      WHEN ROWTYPE_MISMATCH THEN 
        DBMS_OUTPUT.PUT_LINE('ROWTYPE_MISMATCH');
END ;

```
执行结果:

```SQL
  ROWTYPE_MISMATCH
```
#### SYS_INVALID_ROWID
例程:
```sql

DECLARE
 C1 ROWID :='AAAAAAAAAAAAAAAAAA' ;
  BEGIN
   DELETE FROM EMP WHERE ROWID=C1 ;
EXCEPTION
  WHEN    SYS_INVALID_ROWID THEN
       DBMS_OUTPUT.PUT_LINE('SYS_INVALID_ROWID');
 END ;

```
执行结果:

```SQL
 SYS_INVALID_ROWID
```

#### STORAGE_ERROR

例程:
```sql

DECLARE
 TYPE TC1 IS TABLE OF CHAR(2000) ;
 DAO_COL TC1 :=TC1();
BEGIN
   WHILE TRUE
     LOOP
       DAO_COL.EXTEND();
     END LOOP ;
     EXCEPTION
       WHEN STORAGE_ERROR
         THEN DBMS_OUTPUT.PUT_LINE('STORAGE_ERROR');
END ;
```
执行结果:

```SQL

STORAGE_ERROR

```

## 异常的抛出与捕获 

- 对于内置异常Oracle采取隐式抛出
- 对于用户自定义异常使用RAISE语句抛出
- 异常在异常处理部分依靠名字来捕获
- OTHERS 关键字写在异常处理的部分的最后,可以捕获用户自定异常或未在前面声明的异常

例程:隐式抛出
```SQL

BEGIN
  DELETE FROM SCOTT.DEPT;
EXCEPTION
  WHEN OTHERS THEN
    DBMS_OUTPUT.PUT_LINE('ERROR');
END;

```

执行结果

```SQL
ERROR

PL/SQL procedure successfully completed
```

例程:显式抛出
```SQL
BEGIN
  RAISE NO_DATA_FOUND ;
EXCEPTION
  WHEN OTHERS THEN
    DBMS_OUTPUT.PUT_LINE('ERROR');
END;
```

```SQL
ERROR

PL/SQL procedure successfully completed
```
例程:OTHERS 

```sql

BEGIN
  DELETE FROM SCOTT.DEPT;
EXCEPTION
  WHEN NO_DATA_FOUND THEN
    DBMS_OUTPUT.put_line('RETURN NO VALUE');
  WHEN OTHERS THEN
    DBMS_OUTPUT.put_line(SQLERRM);
END;

```

执行结果

```SQL
ORA-02292: integrity constraint (SCOTT.FK_DEPTNO) violated - child record found

PL/SQL procedure successfully completed

```

## 自定义异常

- 当我们违反了Oracle的内部规则时Oracle会抛出异常,这些Oracle自带的错误信息我们称之为内置异常
- 而有时程序数据会违反一些业务逻辑,而非Oracle内部规则.此时我们需用自定义异常进行处理

例程:Oracle自定义异常

```SQL

DECLARE
  V_SAL NUMBER;
  EXCP_SAL_ZERO EXCEPTION;  -- 定义自定义异常
BEGIN
  V_SAL := &1;
  IF V_SAL < 0 THEN         -- 发现工资小于0抛出异常
    RAISE EXCP_SAL_ZERO;
  END IF;
  UPDATE EMP SET SAL = SAL WHERE ROWNUM = 1;
EXCEPTION
  WHEN EXCP_SAL_ZERO THEN   -- 对异常进行捕获
    DBMS_OUTPUT.PUT_LINE('ERROR  SAL LESS THAN ZERO ');
END;

```

执行结果

```sql

 ERROR  SAL LESS THAN ZERO 

PL/SQL procedure successfully completed
 
```


## 异常信息的获取

- SQLCODE 返回Oracle的报错编号
- SQLERRM 返回Oracle的报错信息



例程:SQLCODE && SQLERRM
```sql
BEGIN
 DELETE FROM SCOTT.DEPT ;
EXCEPTION 
 WHEN OTHERS 
  THEN DBMS_OUTPUT.put_line(SQLCODE);
       DBMS_OUTPUT.put_line(SQLERRM);
END ;

```
执行结果

```sql

-2292
ORA-02292: integrity constraint (SCOTT.FK_DEPTNO) violated - child record found

PL/SQL procedure successfully completed
 
```

## 异常的作用域

异常作用域主要是指自定义异常的作用域.
自定义异常的作用域与变量相同

- 父块声明的异常可以被子块引用
- 子块声明的异常不可以被父块引用
- 兄弟块之间的异常不可引用
- 如果子块与父块之间存在同名异常,子块在引用变量时需要借助标签,但是这并没有什么意义


```sql
DECLARE
  E1 EXCEPTION;
BEGIN
  BEGIN
    RAISE E1;
  END;
END;
```
执行结果
```sql
ORA-06510: PL/SQL: unhandled user-defined exception
ORA-06512: at line 5
```

```sql
BEGIN
  DECLARE
   E1 EXCEPTION ;
  BEGIN
    NULL ;
  END ;
  RAISE E1 ;  --父块抛出子块异常引发报错
END ;
```

执行结果

```sql
ORA-06550: line 7, column 9:
PLS-00201: identifier 'E1' must be declared
ORA-06550: line 7, column 3:
PL/SQL: Statement ignored
```


```sql

BEGIN
  DECLARE
   E1 EXCEPTION ;
  BEGIN
    NULL ;
  END ;
  BEGIN
   RAISE E1 ;  --子块2调用子块1定义的异常引发报错
  END ;
END ;

```

执行结果

```sql

ORA-06550: line 8, column 10:
PLS-00201: identifier 'E1' must be declared
ORA-06550: line 8, column 4:
PL/SQL: Statement ignored

```

## 异常的抛出与捕获 
- 
BEGIN

END ;



## 异常的传播规则



### 执行部分引发的异常
- 子块中执行部分引发的异常先由异常处理部分进行捕获
- 如果子块中异常处理部分无法处理,抛由父块进行处理,以此类推,一直到程序的调用者



```SQL

BEGIN
  BEGIN
    RAISE NO_DATA_FOUND;
  EXCEPTION
    WHEN NO_DATA_FOUND THEN   --执行部分引发的异常先由子块处理
      DBMS_OUTPUT.PUT_LINE('INNER BLOCK NO_DATA_FOUND!');
  END;
EXCEPTION
  WHEN NO_DATA_FOUND THEN
    DBMS_OUTPUT.PUT_LINE('OUTER BLOCK NO_DATA_FOUND!');
END;

```

执行结果
```SQL

INNER BLOCK NO_DATA_FOUND!

PL/SQL procedure successfully completed

```


```sql

BEGIN
  BEGIN
    RAISE TOO_MANY_ROWS;
  EXCEPTION
    WHEN NO_DATA_FOUND THEN  --子块中的异常处理部分无法处理TOO_MANY_ROWS
      DBMS_OUTPUT.PUT_LINE('INNER BLOCK NO_DATA_FOUND!');
  END;
EXCEPTION
  WHEN TOO_MANY_ROWS THEN  --父块异常处理部分可以对TOO_MANY_ROWS进行处理
    DBMS_OUTPUT.PUT_LINE('OUTER BLOCK TOO_MANY_ROWS!');
END;

```

执行结果
```SQL

OUTER BLOCK TOO_MANY_ROWS!

PL/SQL procedure successfully completed

```




### 定义部分与异常处理部分引发的异常
- 由父块中的异常处理部分进行捕获
- 如果父块中异常处理部分无法处理,交由父块的父块进行处理


例程:定义部分的异常
```sql

BEGIN
  DECLARE
    V1 NUMBER(1) := 10;
  BEGIN
    NULL;
  EXCEPTION
    WHEN VALUE_ERROR THEN
      DBMS_OUTPUT.PUT_LINE('INNER BLOCK VALUE_ERROR!');
  END;
EXCEPTION
  WHEN VALUE_ERROR THEN
    DBMS_OUTPUT.PUT_LINE('OUTER BLOCK VALUE_ERROR!');
END;

```

执行结果
```SQL
OUTER BLOCK VALUE_ERROR!

PL/SQL procedure successfully completed
```

例程:

```sql

BEGIN
  DECLARE
    V1 NUMBER(1);
  BEGIN
    RAISE NO_DATA_FOUND;    --1 抛出异常 
  EXCEPTION
    WHEN NO_DATA_FOUND THEN --2 捕获后引发VALUES_ERROR异常
      V1 := 10;
    WHEN VALUE_ERROR THEN   -- 子块异常处理部分无法处理
      DBMS_OUTPUT.PUT_LINE('INNER BLOCK VALUE_ERROR!');
  END;
EXCEPTION
  WHEN VALUE_ERROR THEN    --3 交由父块异常处理部分处理
    DBMS_OUTPUT.PUT_LINE('OUTER BLOCK VALUE_ERROR!');
END;

```

执行结果
```SQL
OUTER BLOCK VALUE_ERROR!

PL/SQL procedure successfully completed
```





## RAISE_ APPLICATION_ERROR
RAISE_ APPLICATION_ERROR是Oracle提供的一个过程,允许创建有意义的错误消息.

本质上来说是Oracle部分开放了内部报错机制供开发人员使用

RAISE_ APPLICATION_ERROR有三个参数

1. error_number  报错编号从[-20999 , -20000] 
2. error_message 报错信息
3. keep_errors   是否保留错误栈

```sql
BEGIN
  RAISE NO_DATA_FOUND ;
EXCEPTION
  WHEN OTHERS THEN                                  --保留错误栈
    RAISE_APPLICATION_ERROR(-20000,'ERROR MESSAGES',TRUE);
END;

ORA-20000: ERROR MESSAGES
ORA-06512: at line 5
ORA-01403: no data found

```
执行结果
```SQL
ORA-20000: ERROR MESSAGES
ORA-06512: at line 5
ORA-01403: no data found
```

```SQL

BEGIN
  RAISE NO_DATA_FOUND ;
EXCEPTION
  WHEN OTHERS THEN                                  --不保留错误栈,之前的报错信息会被覆盖
    RAISE_APPLICATION_ERROR(-20000,'ERROR MESSAGES',FALSE);
END;
```

执行结果
```SQL
ORA-20000: ERROR MESSAGES  
ORA-06512: at line 5  --没有打印 ORA-01403: no data found
```
## EXCEPTION_INIT

让我们先来总结下各种类型的异常

异常类别 | 是否有名字 |是否存在报错编号|是否存在报错信息
---|--- |--- |---
预定义异常   | YES | YES  | YES
其他内置异常 | NO  | YES  | YES
自定义异常   | YES | NO   | NO

而我们又知道在异常处理部分中,异常是通过名字来被捕获的,所以其他内置异常无法通过名字捕获.

此时我们可以使用编译指令EXCEPTION_INIT将自定义异常与其他内置异常进行组合

组合后的结果非常类似于预定义异常,即存在名字,报错编号,报错名称


```SQL

DECLARE
  EXCP_CHILD_RECORD_FOUND EXCEPTION;
  PRAGMA EXCEPTION_INIT(EXCP_CHILD_RECORD_FOUND, -2292);  --将自定义异常EXCP_CHILD_RECORD_FOUND与ora-02292绑定
BEGIN
  DELETE FROM SCOTT.DEPT;
EXCEPTION
  WHEN EXCP_CHILD_RECORD_FOUND THEN  --通过捕获绑定的自定义异常捕获报错编码为ora-02292的内置异常
    DBMS_OUTPUT.put_line(SQLERRM);
END;

```

执行结果

```sql
ORA-02292: integrity constraint (SCOTT.FK_DEPTNO) violated - child record found

PL/SQL procedure successfully completed

```
