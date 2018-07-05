---
layout: post
title:  PL/SQL中的条件控制语句
date: 2008-01-10
categories: blog
tags: [PL/SQL]
description: PL/SQL中的条件控制语句
---


使用条件控制，可以基于某种条件来控制程序的执行.

## IF语句
IF-THEN语句是最基本的条件控制形式,其有三种条件控制形式.


### if  end if ;

关键字IF开始后面是条件表达式,多为比较表达式 

关键字THEN 之后是满足条件后需要执行的语句

语句可以为多行,也可以是一个子块
```sql

IF condition THEN
  statements
END IF;

```

```sql

DECLARE
V_IN_NUM NUMBER :=&V_IN_NUM;
BEGIN
 IF MOD(V_IN_NUM,2)=0 THEN  --如果输入的值对2取余等于0 那么
   DBMS_OUTPUT.put_line(V_IN_NUM ||' IS '||'even number');
 END IF ;
END ;    
    
```
执行结果:当我们输入10时

```sql
10 IS even number

PL/SQL procedure successfully completed

```

### if  else end if;
关键字IF开始后面是条件表达式,多为比较表达式 

关键字THEN 之后是满足条件后需要执行的语句

关键字ELSE 之后是如果不满足上面条件所需要执行的语句

```sql

IF condition THEN
  statements
ELSE
  else_statements
END IF;

```

```sql

DECLARE
  V_IN_NUM NUMBER := &V_IN_NUM;
BEGIN
  IF MOD(V_IN_NUM, 2) = 0 THEN   --如果输入的值对2取余等于0 那么
    DBMS_OUTPUT.put_line(V_IN_NUM || ' IS ' || 'even number');
  ELSE                           --其他情况
    DBMS_OUTPUT.put_line(V_IN_NUM || ' IS ' || 'odd number');
  END IF;
END;


```

执行结果

输入10
```sql
10 IS even number

PL/SQL procedure successfully completed

```

输入11

```sql

11 IS odd number

PL/SQL procedure successfully completed

```
### if elsif else end if ;
关键字IF开始后面是条件表达式,多为比较表达式 

关键字THEN 之后是满足条件后需要执行的语句

关键字ELSIF 进行下一次条件判断

关键字ELSE 之后是如果不满足上面条件所需要执行的语句

```sql

IF condition_1 THEN
  statements_1
ELSIF condition_2 THEN
  statements_2
[ ELSIF condition_3 THEN
    statements_3
]...
[ ELSE
    else_statements
]
END IF;

```
下面语句块实现了成绩换算
逻辑如下


成绩区间 | 等级
---|---
91-100 | A
81-90  | B
71-80  | C
60-70  | D
<=59   | E
其他   | X


```SQL

DECLARE
V_SCORE NUMBER :=&IN_SCORE ;
V_RES CHAR(1) ;
BEGIN
  IF V_SCORE     BETWEEN 91 AND 100  THEN V_RES :='A';
    ELSIF  V_SCORE BETWEEN 81 AND 100  THEN V_RES :='B';
    ELSIF  V_SCORE BETWEEN 71 AND 100  THEN V_RES :='C';
    ELSIF  V_SCORE BETWEEN 60 AND 100  THEN V_RES :='D';
    ELSIF  V_SCORE <=59                THEN V_RES :='E';
    ELSE
      V_RES :='X';
  END IF ;
  DBMS_OUTPUT.PUT_LINE(V_SCORE||':'||V_RES) ;
END ;

```
执行结果

输入85

```SQL
85:B

PL/SQL procedure successfully completed
```
### 嵌套IF

在关键字THEN 或 ELSE 之后嵌入IF语句,形成IF语句的嵌套.

IF语句的嵌套经常出现在复杂逻辑中.

一般来说两层IF语句嵌套有4种可能性,3层嵌套有8中可能性,4层有16种.

条件控制语句的多层嵌套是对代码编写者逻辑思维严峻的考验.

我们在写程序的时候,应该尽量减少条件控制语句的嵌套

```sql

BEGIN

  IF CONDITION THEN
    IF CONDITION THEN
      STATEMENTS;
    ELSE
      ELSE_STATEMENTS;
    END IF;
  ELSE
    IF CONDITION THEN
      STATEMENTS;
    ELSE
      ELSE_STATEMENTS;
    END IF;
  END IF;

END;

```


## CASE语句

CASE语句是PL/SQL中条件控制的另一种形式.
其语法上分为简单CASE 与 搜索式CASE
CASE语句与IF ELSIF 比较类似.


CASE 语句 | 支持多个条件 | 比较运算符必须是等号
---|---      |---
简单CASE     | NO |   YES
搜索式CASE   | YES |  NO


### 简单CASE 

CASE 关键字之后是需要判断的变量值

WHEN 关键字之后是待匹配的值 

THEN 关键字之后是匹配后需要执行的语句
```sql
CASE selector
WHEN selector_value_1 THEN statements_1
WHEN selector_value_2 THEN statements_2
...
WHEN selector_value_n THEN statements_n
[ ELSE
  else_statements ]
END CASE;]

```
```sql
DECLARE
V_DAY_NUM CHAR(1) :=TO_CHAR(SYSDATE,'D') ;
V_DAY_NAME VARCHAR2(10) ;
BEGIN
 CASE V_DAY_NUM 
      WHEN '1' THEN V_DAY_NAME:='SUNDAY'    ;
      WHEN '2' THEN V_DAY_NAME:='MONDAY'    ;
      WHEN '3' THEN V_DAY_NAME:='TUESDAY'   ;
      WHEN '4' THEN V_DAY_NAME:='WEDNESDAY' ;
      WHEN '5' THEN V_DAY_NAME:='THURSDAY'  ;
      WHEN '6' THEN V_DAY_NAME:='FRIDAY'    ;
      WHEN '7' THEN V_DAY_NAME:='SATURDAY'  ;
 END CASE ;
 DBMS_OUTPUT.put_line(V_DAY_NAME);                              
END ;

```

执行结果
```sql

SUNDAY

PL/SQL procedure successfully completed

```

### 搜索式CASE 

CASE 关键字之后是WHEN 关键字

WHEN 关键字之后是待判断条件

THEN 关键字之后是匹配后需要执行的语句

```sql

CASE
WHEN condition_1 THEN statements_1
WHEN condition_2 THEN statements_2
...
WHEN condition_n THEN statements_n
[ ELSE
  else_statements ]
END CASE;]

```

```sql

DECLARE
V_DAY_NUM CHAR(1) :=TO_CHAR(SYSDATE,'D') ;
V_DAY_NAME VARCHAR2(10) ;
BEGIN
 CASE  
      WHEN V_DAY_NUM= '1' THEN V_DAY_NAME:='SUNDAY'    ;
      WHEN V_DAY_NUM= '2' THEN V_DAY_NAME:='MONDAY'    ;
      WHEN V_DAY_NUM= '3' THEN V_DAY_NAME:='TUESDAY'   ;
      WHEN V_DAY_NUM= '4' THEN V_DAY_NAME:='WEDNESDAY' ;
      WHEN V_DAY_NUM= '5' THEN V_DAY_NAME:='THURSDAY'  ;
      WHEN V_DAY_NUM= '6' THEN V_DAY_NAME:='FRIDAY'    ;
      WHEN V_DAY_NUM= '7' THEN V_DAY_NAME:='SATURDAY'  ;
 END CASE ;
 DBMS_OUTPUT.put_line(V_DAY_NAME);                              
END ;

```

执行结果
```sql

SUNDAY

PL/SQL procedure successfully completed

```

### CASE语句的不足

CASE 语句相比与IF语句,存在一个不足.

当没有任何一个条件为TRUE 同时没有ELSE语句时会触发CASE NOT FOUND报错 


```sql

DECLARE
V_SCORE NUMBER :=150 ;
V_RES CHAR(1) ;
BEGIN
  CASE
    WHEN  V_SCORE BETWEEN 91 AND 100  THEN V_RES :='A';
    WHEN  V_SCORE BETWEEN 81 AND 100  THEN V_RES :='B';
    WHEN  V_SCORE BETWEEN 71 AND 100  THEN V_RES :='C';
    WHEN  V_SCORE BETWEEN 60 AND 100  THEN V_RES :='D';
    WHEN  V_SCORE <=59                THEN V_RES :='E';
  END CASE  ;
  DBMS_OUTPUT.PUT_LINE(V_SCORE||':'||V_RES) ;
END ;

```
执行结果

输入值为150,不满足任何一个条件

```sql

ORA-06592: CASE not found while executing CASE statement
ORA-06512: at line 5


```

将CASE语句修改为IF语句不存在这个问题.

```sql

DECLARE
V_SCORE NUMBER :=150 ;
V_RES CHAR(1) ;
BEGIN
  IF V_SCORE BETWEEN 91 AND 100  THEN V_RES :='A';
    ELSIF  V_SCORE BETWEEN 81 AND 100  THEN V_RES :='B';
    ELSIF  V_SCORE BETWEEN 71 AND 100  THEN V_RES :='C';
    ELSIF  V_SCORE BETWEEN 60 AND 100  THEN V_RES :='D';
    ELSIF  V_SCORE <=59                THEN V_RES :='E';
  END IF  ;
  DBMS_OUTPUT.PUT_LINE(V_SCORE||':'||V_RES) ;
END ;

```

```sql

150:

PL/SQL procedure successfully completed

```
