---
layout: post
title: PL/SQL中的循环控制语句
date: 2008-01-22
categories: blog
tags: [PL/SQL]
description:  PL/SQL中的循环控制语句。
---



循环是一种编程机制，允许一组指令被反复执行.
在PL/SQL中使用循环的机会并没有其他语言那么多.


## 简单循环

```SQL
[ label ] LOOP
  statements
END LOOP [ label ];
```
[ label ] 是循环中的标签,可以用来标识一个循环.

在嵌套循环中有着广泛的应用.

LOOP 关键字标识循环开始

END LOOP 标识循环结束.

下面的例程是一个死循环,因为我们没有指定循环的退出条件

```sql
begin
  loop
    null ;
  end loop ;
end ;

```

## 循环的退出 

可以在循环中使用EXIT 语句退出循环 

EXIT 语句有两种形式: 

- IF EXIT ;
 
例程:打印1-5
```sql
DECLARE
  V_COUNTER NUMBER := 0;
BEGIN
  LOOP
    V_COUNTER := V_COUNTER + 1;
    DBMS_OUTPUT.put_line(V_COUNTER);
    IF V_COUNTER = 5 THEN
      EXIT;
    END IF;
  END LOOP;
END;

```
执行结果

```sql
1
2
3
4
5

PL/SQL procedure successfully completed

```

- EXIT WHEN ;

例程:打印1-5 
```sql
DECLARE
V_COUNTER NUMBER :=0 ;
BEGIN
 LOOP
   V_COUNTER :=V_COUNTER+1 ;
   DBMS_OUTPUT.put_line(V_COUNTER);
   EXIT WHEN V_COUNTER = 5 ;
 END LOOP ;
END ;  
  
```
执行结果

```sql
1
2
3
4
5

PL/SQL procedure successfully completed

```
## WHILE 循环

WHILE 关键字后是循环可以运行需满足的条件
与简单循环不同,WHILE循环是先判断后执行

例程:打印1-5
```sql
[ label ] WHILE condition LOOP
  statements
END LOOP [ label ];
```

```sql
DECLARE
  V_COUNTER NUMBER := 0;
BEGIN
  WHILE V_COUNTER <5 
  LOOP
    V_COUNTER := V_COUNTER + 1;
    DBMS_OUTPUT.put_line(V_COUNTER);
  END LOOP;
END;

```
执行结构
```sql
1
2
3
4
5

PL/SQL procedure successfully completed
```
## FOR 循环
FOR 循环可以生成一个数列,数列从lower_bound开始到upper_bound

FOR 关键字后是一个数值型变量,这个变量不需要提前声明,也不能被赋值.

REVERSE 关键字可以使数列降序输出

```sql
[ label ] FOR index IN [ REVERSE ] lower_bound..upper_bound LOOP
  statements
END LOOP [ label ];

```
例程:打印1-5
```sql
BEGIN
  FOR I IN 1 .. 5 LOOP
    DBMS_OUTPUT.put_line(I);
  END LOOP;
END;
```

执行结果:

```sql
1
2
3
4
5

PL/SQL procedure successfully completed
```

例程:打印5-1
```SQL
5
4
3
2
1

PL/SQL procedure successfully completed
```

```SQL

BEGIN
  FOR I IN  1 .. 5  LOOP
    I :=10;  --对for循环自带的变量赋值,引发报错
    DBMS_OUTPUT.put_line(I);
  END LOOP;
END;

```
执行结果:

```sql
 ORA-06550: line 3, column 5:
PLS-00363: expression 'I' cannot be used as an assignment target
ORA-06550: line 3, column 5:
PL/SQL: Statement ignored
```
## CONTNUTE 语句
CONTNUTE 语句可以提前结束本轮循环,开启下一轮循环.
CONTNUTE 语句也有两种实现方式
- IF CONTINUE 
例程:打印1-10之间的偶数

```sql

BEGIN
  FOR I IN 1 .. 10 LOOP
    IF MOD(I,2)=1  THEN  --当I的值对2取余等于1的时候,执行CONTINUE语句,提前结束循环,跳过打印语句
      CONTINUE ;   
    END IF ;
    DBMS_OUTPUT.put_line(I);
  END LOOP;
END;

```

执行结果:

```sql
2
4
6
8
10

PL/SQL procedure successfully completed

```
- CONTINUE WHEN 

例程:打印1-10之间的偶数
```sql

BEGIN
  FOR I IN 1 .. 10 LOOP
    CONTINUE WHEN MOD(I,2)=1 ; 
    DBMS_OUTPUT.put_line(I);
  END LOOP;
END;

```

执行结果:

```sql
2
4
6
8
10

PL/SQL procedure successfully completed

```

## 嵌套循环

在循环内加入嵌入另一个循环语句,形成嵌套循环.

简单循环,WHILE循环,FOR循环可以互相嵌套.

### 非相关嵌套循环

如果内部循环循环的次数与外部循环计数器的值没有直接关系,这种循环我们称之为非相关嵌套循环

```sql

BEGIN
  <<outer_loop>>
  FOR i IN 1 .. 2 LOOP
    <<inner_loop>>
    FOR j IN 1 .. 5 LOOP
      DBMS_OUTPUT.PUT_LINE('outer i='||i||' j = ' || j);
    END LOOP inner_loop;
  END LOOP outer_loop;
END;
```
执行结果:

```sql
outer i=1 j = 1
outer i=1 j = 2
outer i=1 j = 3
outer i=1 j = 4
outer i=1 j = 5
outer i=2 j = 1
outer i=2 j = 2
outer i=2 j = 3
outer i=2 j = 4
outer i=2 j = 5

PL/SQL procedure successfully completed

```
### 相关嵌套循环

如果内部循环循环的次数与外部循环计数器的值存在直接关系,这种循环我们称之为相关嵌套循环

例程:九九乘法表 

```sql

BEGIN
  FOR outer_cnt IN  1 .. 9  LOOP
     for inner_cnt in 1..outer_cnt  --内层循环的值受外层循环的值影响
     loop
        dbms_output.put(inner_cnt||'*'||outer_cnt||' = '||rpad(inner_cnt*outer_cnt,3,' '));
     end loop ;
     dbms_output.put_line(' ');
  END LOOP;
END;

```
执行结果:

```sql
1*1 = 1   
1*2 = 2  2*2 = 4   
1*3 = 3  2*3 = 6  3*3 = 9   
1*4 = 4  2*4 = 8  3*4 = 12 4*4 = 16  
1*5 = 5  2*5 = 10 3*5 = 15 4*5 = 20 5*5 = 25  
1*6 = 6  2*6 = 12 3*6 = 18 4*6 = 24 5*6 = 30 6*6 = 36  
1*7 = 7  2*7 = 14 3*7 = 21 4*7 = 28 5*7 = 35 6*7 = 42 7*7 = 49  
1*8 = 8  2*8 = 16 3*8 = 24 4*8 = 32 5*8 = 40 6*8 = 48 7*8 = 56 8*8 = 64  
1*9 = 9  2*9 = 18 3*9 = 27 4*9 = 36 5*9 = 45 6*9 = 54 7*9 = 63 8*9 = 72 9*9 = 81  

PL/SQL procedure successfully completed

```
