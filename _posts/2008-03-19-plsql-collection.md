---
layout: post
title:  PL/SQL中的集合
date: 2008-03-19
categories: blog
tags: [PL/SQL]
description: PL/SQL中的集合
---

基本的集合相当于一个一列多行的表.
在PL/SQL中有三种集合类型

- 索引表 
- 嵌套表
- 变长数组


集合类型 | 元素数量限制 |下标类型|密集或稀疏|是否需要初始化|定义级别|是否可以成为抽象数据类型
---|--- |--- |--- |--- |--- |---
索引表   | 无  | 字符或PLS_INTEGER |都可以                   | 否 |PL/SQL块或包级别           | 不可以
嵌套表   | 无  | Integer           |开始密集,删除后变成稀疏  | 是 |PL/SQL块或包级别或用户级别 | 必须是用户级别定义
变长数组 | 有  | Integer           |总是密集                 | 是 |PL/SQL块或包级别或用户级别 | 必须是用户级别定义



## 索引表

```sql
declare

type  t_index_table  is  table of number  index by pls_integer ; 

index_table  t_index_table ;

begin
 index_table(1) :=11 ;
 index_table(3) :=33 ;
 index_table(2) :=22 ;
 --打印下标1的下一个下标
 dbms_output.put_line(index_table.next(1));
 --打印下标2的下一个下标
 dbms_output.put_line(index_table.next(2));
 dbms_output.put_line('~~~~~~~~~~~~~~~~~~~~~~~~~~~');
 dbms_output.put_line(index_table(1));
 dbms_output.put_line(index_table(2));
 dbms_output.put_line(index_table(3));  
end ;

```
执行结果
```sql
2
3
~~~~~~~~~~~~~~~~~~~~~~~~~~~
11
22
33
PL/SQL procedure successfully completed
```

在本例中我们先定义了一个索引表
然后将下标为1的元素赋值为10
将下标为3的元素赋值为30
此时索引表元素是稀疏的,因为存在断层(下标为2)

```
graph LR
A["INDEX_TABLE(1)[10]"]-->B["INDEX_TABLE(3)[30]"]

```
之后我们又将下标为2的元素赋值为20
此时索引表状态变为:
```
graph LR
A["INDEX_TABLE(1)[10]"]-->B["INDEX_TABLE(2)[20]"]
B["INDEX_TABLE(2)[20]"]-->C["INDEX_TABLE(3)[30]"]

```
此时索引表元素是密集的



## 嵌套表
- 嵌套表需要进行初始化
- 嵌套表每次存放元素之前需要扩展

例程:嵌套表
```sql
declare

  type t_nested_table is table of number;

  nested_table t_nested_table := t_nested_table(); --初始化

begin
  nested_table.extend();  --扩展
  nested_table(1) := 11;  

  nested_table.extend();
  nested_table(2) := 22;

  nested_table.extend();
  nested_table(3) := 33;

  dbms_output.put_line(nested_table(1));
  dbms_output.put_line(nested_table(2));
  dbms_output.put_line(nested_table(3));
  
end;


```
执行结果
```sql
11
22
33
PL/SQL procedure successfully completed
```

例程:不进行初始化

```sql
declare

  type t_nested_table is table of number;

  nested_table t_nested_table;

begin
  nested_table.extend();

end;



```
执行结果
```sql
ORA-06531: Reference to uninitialized collection
ORA-06512: at line 8
```
不对嵌套表进行扩展
```sql
declare
  type t_nested_table is table of number;
  nested_table t_nested_table := t_nested_table();
begin
  nested_table(1) := 10;
end;

```
执行结果
```sql
ORA-06533: Subscript beyond count
ORA-06512: at line 5
```
## 变长数组


```sql
declare
  type t_varray is varray(5) of number;
  varr t_varray := t_varray(11,22,33,44); --带参数初始化
begin
  varr.extend;
  varr(5) := 55;
  
  dbms_output.put_line(varr(1));
  dbms_output.put_line(varr(2));
  dbms_output.put_line(varr(3));
  dbms_output.put_line(varr(4));
  dbms_output.put_line(varr(5));      
end;
```
执行结果
```sql
11
22
33
44
55
PL/SQL procedure successfully completed

```
例程:修改变长数组长度
```sql

```

执行结果
```sql
```
## 集合方法



方法名  | 方法类型 |描述|支持索引表 |支持嵌套表 |支持变长数组
---|---|---|---|---|---
COUNT  | 函数 | 集合数量                |YES |YES  |YES
FIRST  | 函数 | 当前集合的第一个下标    |YES |YES  |YES
LAST   | 函数 | 当前集合的最后一个下标  |YES |YES  |YES
PRIOR  | 函数 | 当前下标的前一个下标    |YES |YES  |YES
NEXT   | 函数 | 当前下标的下一个下标    |YES |YES  |YES
EXISTS | 函数 | 下标是否存在            |YES |YES  |YES
LIMIT  | 函数 | 集合最大元素数量        |YES |YES  |YES
DELETE | 过程 | 删除集合元素            |YES |YES  |NO
TRIM   | 过程 | 清除集合尾部元素        |NO  |YES  |YES
EXTEND | 过程 | 扩展集合                |NO  |YES  |YES


### COUNT

例程:COUNT
```sql
DECLARE
  TYPE T_VARRAY IS VARRAY(5) OF NUMBER;
  VARR T_VARRAY := T_VARRAY(1111, 2222, 3333, 4444, 5555);
BEGIN
  DBMS_OUTPUT.put_line(VARR.COUNT);
END;

```

执行结果

```sql
5
PL/SQL procedure successfully completed
```

### FIRST && LAST
例程: FIRST && LAST 
```sql

DECLARE
  TYPE T_INDEX_TABLE IS TABLE OF NUMBER INDEX BY PLS_INTEGER;
  INDEX_TABLE T_INDEX_TABLE;
BEGIN

  INDEX_TABLE(3) := 33;
  INDEX_TABLE(5) := 55;
  INDEX_TABLE(7) := 77;

  DBMS_OUTPUT.put_line('FIRST '||INDEX_TABLE.FIRST);
  DBMS_OUTPUT.put_line('LAST  '||INDEX_TABLE.LAST);
END;

```
执行结果
```sql

FIRST 3
LAST  7
PL/SQL procedure successfully completed

```

### PRIOR && NEXT

例程: PRIOR && NEXT
```sql

DECLARE
  TYPE T_INDEX_TABLE IS TABLE OF NUMBER INDEX BY PLS_INTEGER;
  INDEX_TABLE T_INDEX_TABLE;
BEGIN
  INDEX_TABLE(3) := 33;
  INDEX_TABLE(5) := 55;
  INDEX_TABLE(7) := 77;

  DBMS_OUTPUT.put_line('PRIOR(5) '||INDEX_TABLE.PRIOR(5));
  DBMS_OUTPUT.put_line('NEXT(5)  '||INDEX_TABLE.NEXT(5));
END;

```
执行结果
```sql
PRIOR(5) 3
NEXT(5)  7
PL/SQL procedure successfully completed

```

### EXISTS

例程:EXISTS 

```SQL

DECLARE
  TYPE T_VARRAY IS VARRAY(5) OF NUMBER;
  VARR T_VARRAY := T_VARRAY(1111, 2222, 3333, 4444, 5555);
BEGIN
  IF VARR.EXISTS(5) THEN
    DBMS_OUTPUT.put_line('(5) EXISTS');
  ELSE
    DBMS_OUTPUT.put_line('(5)NOT  EXISTS');
  END IF;
  IF VARR.EXISTS(7) THEN
    DBMS_OUTPUT.put_line('(7) EXISTS');
  ELSE
    DBMS_OUTPUT.put_line('(7) NOT  EXISTS');
  END IF;
END;
```

```SQL
PRIOR(5) 3
NEXT(5)  7
PL/SQL procedure successfully completed
```

### LIMIT 
```sql

DECLARE

  TYPE T_INDEX_TABLE IS TABLE OF NUMBER INDEX BY PLS_INTEGER;
  INDEX_TABLE T_INDEX_TABLE;

  TYPE T_NESTED_TABLE IS TABLE OF NUMBER;
  NESTED_TABLE T_NESTED_TABLE := T_NESTED_TABLE(111, 222, 333, 444, 555);

  TYPE T_VARRAY IS VARRAY(5) OF NUMBER;
  VARR T_VARRAY := T_VARRAY(1111, 2222, 3333, 4444, 5555);

BEGIN

  INDEX_TABLE(1) := 11;
  INDEX_TABLE(3) := 33;
  INDEX_TABLE(5) := 55;
  INDEX_TABLE(7) := 77;
  DBMS_OUTPUT.PUT_LINE('INDEX_TABLE.LIMIT : '||INDEX_TABLE.LIMIT);
  DBMS_OUTPUT.PUT_LINE('NESTED_TABLE.LIMIT : '||NESTED_TABLE.LIMIT);
  DBMS_OUTPUT.PUT_LINE('VARR.LIMIT: '||VARR.LIMIT);
END;

```
执行结果
```sql

INDEX_TABLE.LIMIT : 
NESTED_TABLE.LIMIT : 
VARR.LIMIT: 5
PL/SQL procedure successfully completed

```

### DELETE 

- DELETE()     删除所有元素

例程: DELETE() 
```sql
DECLARE

  TYPE T_NESTED_TABLE IS TABLE OF NUMBER;
  NESTED_TABLE T_NESTED_TABLE := T_NESTED_TABLE(111, 222, 333, 444, 555);
BEGIN
  DBMS_OUTPUT.put_line(NESTED_TABLE.COUNT);
  NESTED_TABLE.DELETE;
  DBMS_OUTPUT.put_line(NESTED_TABLE.COUNT);
END;
```
执行结果
```sql
5
0
PL/SQL procedure successfully completed
```
- DELETE(n)    删除下标为n的元素
例程: DELETE(N) 
```sql
DECLARE
  TYPE T_INDEX_TABLE IS TABLE OF NUMBER INDEX BY PLS_INTEGER;
  INDEX_TABLE T_INDEX_TABLE;
BEGIN
  INDEX_TABLE(1) := 11;
  INDEX_TABLE(3) := 33;
  INDEX_TABLE(5) := 55;
  INDEX_TABLE(7) := 77;

  DBMS_OUTPUT.put_line(INDEX_TABLE.COUNT);
  INDEX_TABLE.DELETE(2); --并没有2号下标的元素,所以此步没有删除任何元素
  DBMS_OUTPUT.put_line(INDEX_TABLE.COUNT);
  INDEX_TABLE.DELETE(3);
  DBMS_OUTPUT.put_line(INDEX_TABLE.COUNT);
END;

```
执行结果
```sql
4
4
3
PL/SQL procedure successfully completed
```

- DELETE(N,M)  删除下标在n,m之间的元素

例程: DELETE(N,M) 
```sql
DECLARE
  TYPE T_INDEX_TABLE IS TABLE OF NUMBER INDEX BY PLS_INTEGER;
  INDEX_TABLE T_INDEX_TABLE;
BEGIN
  INDEX_TABLE(1) := 11;
  INDEX_TABLE(3) := 33;
  INDEX_TABLE(5) := 55;
  INDEX_TABLE(7) := 77;

  DBMS_OUTPUT.put_line(INDEX_TABLE.COUNT);
  INDEX_TABLE.DELETE(2,4);  --删除下标为2,3,4的元素 本例中只存在下标为3的元素
  DBMS_OUTPUT.put_line(INDEX_TABLE.COUNT);
  INDEX_TABLE.DELETE(5,7);--删除下标为5,6,7的元素 本例中存在下标为5,7的元素
  DBMS_OUTPUT.put_line(INDEX_TABLE.COUNT);
END;

```
执行结果
```sql
4
3
1
PL/SQL procedure successfully completed

```
- DELETE 不可用于变长数组

```sql

DECLARE

  TYPE T_VARRAY IS VARRAY(5) OF NUMBER;
  VARR T_VARRAY := T_VARRAY(1111, 2222, 3333, 4444, 5555);

BEGIN
  VARR.DELETE(3);
END ;
```
执行结果
```sql

ORA-06550: line 7, column 3:
PLS-00306: wrong number or types of arguments in call to 'DELETE'
ORA-06550: line 7, column 3:
PL/SQL: Statement ignored

```
### TRIM 
例程:TRIM 
```sql
DECLARE
  TYPE T_NESTED_TABLE IS TABLE OF NUMBER;
  NESTED_TABLE T_NESTED_TABLE := T_NESTED_TABLE(111, 222, 333, 444, 555);

  TYPE T_VARRAY IS VARRAY(5) OF NUMBER;
  VARR T_VARRAY := T_VARRAY(1111, 2222, 3333, 4444, 5555);
BEGIN
  DBMS_OUTPUT.PUT_LINE('NESTED_TABLE.LAST : ' || NESTED_TABLE.LAST);
  NESTED_TABLE.TRIM();
  DBMS_OUTPUT.PUT_LINE('NESTED_TABLE.LAST : ' || NESTED_TABLE.LAST);
  
  DBMS_OUTPUT.PUT_LINE('~~~~~~~~~~~```');
    
  DBMS_OUTPUT.PUT_LINE('VARR.LAST : ' || VARR.LAST);
  VARR.TRIM();
  DBMS_OUTPUT.PUT_LINE('VARR.LAST: ' || VARR.LAST);
END;

```
执行结果
```sql
NESTED_TABLE.LAST : 5
NESTED_TABLE.LAST : 4
~~~~~~~~~~~```
VARR.LAST : 5
VARR.LAST: 4
PL/SQL procedure successfully completed

```
### TRIM(n)
例程:TRIM(n)    
```sql
DECLARE
  TYPE T_NESTED_TABLE IS TABLE OF NUMBER;
  NESTED_TABLE T_NESTED_TABLE := T_NESTED_TABLE(111, 222, 333, 444, 555);

  TYPE T_VARRAY IS VARRAY(5) OF NUMBER;
  VARR T_VARRAY := T_VARRAY(1111, 2222, 3333, 4444, 5555);
BEGIN
  DBMS_OUTPUT.PUT_LINE('NESTED_TABLE.LAST : ' || NESTED_TABLE.LAST);
  NESTED_TABLE.TRIM(2);
  DBMS_OUTPUT.PUT_LINE('NESTED_TABLE.LAST : ' || NESTED_TABLE.LAST);
  
  DBMS_OUTPUT.PUT_LINE('~~~~~~~~~~~```');
    
  DBMS_OUTPUT.PUT_LINE('VARR.LAST : ' || VARR.LAST);
  VARR.TRIM(2);
  DBMS_OUTPUT.PUT_LINE('VARR.LAST: ' || VARR.LAST);
END;

```
执行结果
```sql

NESTED_TABLE.LAST : 5
NESTED_TABLE.LAST : 3
~~~~~~~~~~~```
VARR.LAST : 5
VARR.LAST: 3
PL/SQL procedure successfully completed

```

### TRIM 与 索引表

例程:TRIM 与 索引表

```sql
DECLARE
  TYPE T_INDEX_TABLE IS TABLE OF NUMBER INDEX BY PLS_INTEGER;
  INDEX_TABLE T_INDEX_TABLE;
BEGIN
  INDEX_TABLE(1) := 11;
  INDEX_TABLE(3) := 33;
  INDEX_TABLE(5) := 55;
  INDEX_TABLE(7) := 77;
  
  INDEX_TABLE.TRIM(); --TRIM函数不支持索引表,此处引发报错
  DBMS_OUTPUT.PUT_LINE('INDEX_TABLE.LAST : '||INDEX_TABLE.LAST);
END;

```
执行结果

```sql

ORA-06550: line 10, column 3:
PLS-00306: wrong number or types of arguments in call to 'TRIM'
ORA-06550: line 10, column 3:
PL/SQL: Statement ignored

```

## EXTEND
例程:EXTEND
```sql
DECLARE
  TYPE T_NESTED_TABLE IS TABLE OF NUMBER;
  NESTED_TABLE T_NESTED_TABLE := T_NESTED_TABLE(111, 222, 333, 444);

  TYPE T_VARRAY IS VARRAY(5) OF NUMBER;
  VARR T_VARRAY := T_VARRAY(1111, 2222, 3333, 4444);
BEGIN
  DBMS_OUTPUT.PUT_LINE('NESTED_TABLE.LAST : ' || NESTED_TABLE.LAST);
  NESTED_TABLE.EXTEND;
  DBMS_OUTPUT.PUT_LINE('NESTED_TABLE.LAST : ' || NESTED_TABLE.LAST);

  DBMS_OUTPUT.PUT_LINE('VARR.LAST: ' || VARR.LAST);
  VARR.EXTEND;
  DBMS_OUTPUT.PUT_LINE('VARR.LAST: ' || VARR.LAST);
END;

```
执行结果
```sql

NESTED_TABLE.LAST : 4
NESTED_TABLE.LAST : 5
VARR.LAST: 4
VARR.LAST: 5
PL/SQL procedure successfully completed

```
例程:EXTEND与索引表
```sql
DECLARE

  TYPE T_INDEX_TABLE IS TABLE OF NUMBER INDEX BY PLS_INTEGER;
  INDEX_TABLE T_INDEX_TABLE;
BEGIN
  INDEX_TABLE.EXTEND; --EXTEND过程不支持索引表,此处引发报错
END;

```
执行结果
```sql
ORA-06550: line 6, column 3:
PLS-00306: wrong number or types of arguments in call to 'EXTEND'
ORA-06550: line 6, column 3:
PL/SQL: Statement ignored
```

## 多层集合
我们将一个集合类型作为另一个集合的元素,可以构造出多层集合

```sql
DECLARE

  TYPE T_INDEX_TABLE  IS TABLE OF NUMBER INDEX BY PLS_INTEGER;
  TYPE T_INDEX_TABLE_S IS TABLE OF T_INDEX_TABLE INDEX BY PLS_INTEGER;

  TYPE T_NESTED_TABLE   IS TABLE OF NUMBER;
  TYPE T_NESTED_TABLE_S IS TABLE OF T_NESTED_TABLE;

  TYPE T_VARRAY IS  VARRAY(5) OF NUMBER;
  TYPE T_VARRAY_S IS VARRAY(2) OF T_VARRAY;
BEGIN
  NULL ;
END ;  

```

## 记录集合

将记录类型放入到一个集合当中,则可以构造个一个基于记录的集合.

通过集合解决多行问题,通过记录解决多列的问题.

此时记录集合就可以缓存整张表的内容


```sql
declare
  CURSOR C1 IS
    SELECT DNAME, LOC FROM DEPT ORDER BY DEPTNO;

  TYPE REC_COLl IS TABLE OF C1%ROWTYPE;

  DEPT_PART_TAB REC_COLl := REC_COLl();

  V_COUNTER NUMBER := 0;
  
  V_SCRIPTNO NUMBER ;
begin
  --将游标记录放入到记录集合中
  OPEN C1;
  LOOP
    V_COUNTER := V_COUNTER + 1;
    DEPT_PART_TAB.EXTEND();
    FETCH C1
      INTO DEPT_PART_TAB(V_COUNTER);
    IF C1%NOTFOUND THEN
      DEPT_PART_TAB.TRIM();
      EXIT;
    END IF;
  END LOOP;
  
 --下面这种写法适用于所有集合类型,无论其下标是否连续
  V_SCRIPTNO :=DEPT_PART_TAB.FIRST;
 LOOP
   DBMS_OUTPUT.put_line(DEPT_PART_TAB(V_SCRIPTNO).DNAME);
    V_SCRIPTNO:=DEPT_PART_TAB.NEXT(V_SCRIPTNO);
    EXIT WHEN V_SCRIPTNO IS NULL ;
 END LOOP ;
end;

```
执行结果
```sql
ACCOUNTING
RESEARCH
SALES
OPERATIONS
PL/SQL procedure successfully completed

```
