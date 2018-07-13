---
layout: post
title:    某东笔试题
date: 2018-05-08
categories: blog
tags: [SQL,面试题]
description: 某东笔试题
---

# 新手版

## 题目1 

表A

|ID|NAME|AGE
---|---|---
1  |张三 |27
2  |李四 |28
3  |小明 |29
5  |孙畅 |32
6  |李四 |29

表B

|ID|NAME|INCOME
---|---|---
2  |李四 |5000
3  |小明 |4000
4  |王宏 |5000 
5  |孙畅 |0

### 1.1 请用SQL得到以下数据

|ID|NAME|AGE|INCOME
---|---|--- |---
1  |张三 |27 |0
2  |李四 |28 |5000
3  |小明 |29 |4000
4  |王宏|null|5000
5  |孙畅 |32 |0
6  |李四 |29 |0

```sql
SQL> WITH DAO_A  AS
  2  ( SELECT 1 ID , '张三' NAME,27 AGE  FROM  DUAL
  3   UNION ALL
  4   SELECT 2 ID ,  '李四' NAME,28 AGE  FROM  DUAL
  5   UNION ALL
  6   SELECT 3 ID ,  '小明' NAME,29 AGE  FROM  DUAL
  7    UNION ALL
  8   SELECT 5 ID ,  '孙畅' NAME,32 AGE  FROM  DUAL
  9    UNION ALL
 10   SELECT 6 ID ,  '李四' NAME,29 AGE  FROM  DUAL
 11  ),
 12  DAO_B AS
 13  (
 14   SELECT 2 ID ,  '李四' NAME,5000 INCOME  FROM  DUAL
 15   UNION ALL
 16   SELECT 3 ID ,  '小明' NAME,4000 INCOME  FROM  DUAL
 17   UNION ALL
 18   SELECT 4 ID ,  '王宏' NAME,5000 INCOME  FROM  DUAL
 19    UNION ALL
 20    SELECT 5 ID , '孙畅' NAME,0    INCOME  FROM  DUAL
 21   )
 22  SELECT  NVL(DAO_A.ID,DAO_B.ID) ID,
 23          NVL(DAO_A.NAME,DAO_B.NAME ) NAME,
 24          DAO_A.AGE,
 25          NVL(DAO_B.INCOME,0) INCOME
 26    FROM DAO_A FULL OUTER JOIN  DAO_B  ON (DAO_A.ID=DAO_B.ID) ;

        ID NAME          AGE     INCOME
---------- ------ ---------- ----------
         1 张三           27          0
         2 李四           28       5000
         3 小明           29       4000
         5 孙畅           32          0
         6 李四           29          0
         4 王宏                    5000

6 rows selected

```

### 1.2 用SQL 将收入大于0的人员进行降序排序,同收入的按ID降序

输出结果:

ID|NAME|INCOME|AGE|RANKID
---|---|---|---|---
   |   |   |   |

```sql

SQL> WITH DAO_A  AS
  2  ( SELECT 1 ID , '张三' NAME,27 AGE  FROM  DUAL
  3   UNION ALL
  4   SELECT 2 ID ,  '李四' NAME,28 AGE  FROM  DUAL
  5   UNION ALL
  6   SELECT 3 ID ,  '小明' NAME,29 AGE  FROM  DUAL
  7    UNION ALL
  8   SELECT 5 ID ,  '孙畅' NAME,32 AGE  FROM  DUAL
  9    UNION ALL
 10   SELECT 6 ID ,  '李四' NAME,29 AGE  FROM  DUAL
 11  ),
 12  DAO_B AS
 13  (
 14   SELECT 2 ID ,  '李四' NAME,5000 INCOME  FROM  DUAL
 15   UNION ALL
 16   SELECT 3 ID ,  '小明' NAME,4000 INCOME  FROM  DUAL
 17   UNION ALL
 18   SELECT 4 ID ,  '王宏' NAME,5000 INCOME  FROM  DUAL
 19    UNION ALL
 20    SELECT 5 ID , '孙畅' NAME,0    INCOME  FROM  DUAL
 21   )
 22   SELECT DAO_V.*,ROWNUM RANKID
 23    FROM ( SELECT  DAO_A.ID,
 24                   DAO_A.NAME,
 25                   DAO_B.INCOME,
 26                   DAO_A.AGE
 27              FROM DAO_A  JOIN  DAO_B  ON (DAO_A.ID=DAO_B.ID)
 28  ORDER BY ID )  DAO_V ;

        ID NAME       INCOME        AGE     RANKID
---------- ------ ---------- ---------- ----------
         2 李四         5000         28          1
         3 小明         4000         29          2
         5 孙畅            0         32          3


```
### 1.3 查询相同姓名的人保留年龄较大的人员信息

输出结果:

ID|NAME|INCOME
---|---|---

```sql
SQL> WITH DAO_A  AS
  2  ( SELECT 1 ID , '张三' NAME,27 AGE  FROM  DUAL
  3   UNION ALL
  4   SELECT 2 ID ,  '李四' NAME,28 AGE  FROM  DUAL
  5   UNION ALL
  6   SELECT 3 ID ,  '小明' NAME,29 AGE  FROM  DUAL
  7    UNION ALL
  8   SELECT 5 ID ,  '孙畅' NAME,32 AGE  FROM  DUAL
  9    UNION ALL
 10   SELECT 6 ID ,  '李四' NAME,29 AGE  FROM  DUAL
 11  ),
 12  DAO_B AS
 13  (
 14   SELECT 2 ID ,  '李四' NAME,5000 INCOME  FROM  DUAL
 15   UNION ALL
 16   SELECT 3 ID ,  '小明' NAME,4000 INCOME  FROM  DUAL
 17   UNION ALL
 18   SELECT 4 ID ,  '王宏' NAME,5000 INCOME  FROM  DUAL
 19    UNION ALL
 20    SELECT 5 ID , '孙畅' NAME,0    INCOME  FROM  DUAL
 21   )
 22   SELECT DAO_A.ID,DAO_A.NAME ,DAO_B.INCOME
 23   FROM DAO_A ,
 24       (SELECT NAME, MAX(AGE) MAX_AGE
 25        FROM  DAO_A
 26        GROUP BY NAME  )DAO_V ,
 27        DAO_B
 28   WHERE DAO_A.AGE=DAO_V.MAX_AGE
 29     AND DAO_A.NAME=DAO_V.NAME
 30     AND DAO_A.ID=DAO_B.ID(+)
 31     ORDER BY DAO_A.ID
 32  ;

        ID NAME       INCOME
---------- ------ ----------
         1 张三   
         3 小明         4000
         5 孙畅            0
         6 李四   


```


# 老司机版

## 题目1 用一条SQL语句查询所有科目成绩都大于80分的同学

NAME | CLASS | SCORE
---  | ---   | ---
张三 | 语文   | 81
张三 | 数学   | 90
李四 | 语文   | 95
李四 | 数学   | 79
王五 | 语文   | 60
王五 | 数学   | 60


```sql

SQL> WITH DAO_1 AS
  2  ( SELECT  '张三' NAME ,'语文' COURSE , 81 SCORE
  3     FROM  DUAL
  4   UNION ALL
  5    SELECT  '张三' NAME ,'数学' COURSE , 90 SCORE
  6     FROM  DUAL
  7   UNION ALL
  8    SELECT  '李四' NAME ,'语文' COURSE , 95 SCORE
  9     FROM  DUAL
 10   UNION ALL
 11    SELECT  '李四' NAME ,'数学' COURSE , 79 SCORE
 12     FROM  DUAL
 13   UNION ALL
 14    SELECT  '王五' NAME ,'语文' COURSE , 60 SCORE
 15     FROM  DUAL
 16   UNION ALL
 17    SELECT  '王五' NAME ,'数学' COURSE , 60 SCORE
 18     FROM  DUAL
 19  )
 20   SELECT NAME
 21     FROM DAO_1
 22   GROUP BY NAME
 23   HAVING MIN(SCORE)>80
 24  ;

NAME
------
张三

```

## 题目2 怎么样把一个表t1 转变成表T2

表T1

YEAR | M1 |  M2 | M3 | M4
--- | --- | --- |---|---
1991 |1.1|1.2|1.3|1.4
1992| 21 |2.2|2.3|2.4

表T2 

YEAR | MONTH | AMOUNT 
---  | ---   |  ---
1991 |  1    | 1.1
1991 |  2    | 1.2
1991 |  3    | 1.3
1991 |  4    | 1.4
1992 |  1    | 2.1
1992 |  2    | 2.2
1992 |  3    | 2.3
1992 |  4    | 2.4


```sql
SQL> with dao as (
  2  select  1993 year ,1.1 m1 ,1.2 m2 ,1.3 m3 ,1.4 m4 from dual
  3  union all
  4  select  1994 year ,2.1,2.2,2.3,2.4 from dual
  5  )
  6  select year,substr(month,2,1)month,amount
  7    from dao
  8   unpivot(
  9      amount for month in (m1, m2, m3, m4)
 10  )
 11  order by 1 ,2 ;

      YEAR MONTH     AMOUNT
---------- ----- ----------
      1993 1            1.1
      1993 2            1.2
      1993 3            1.3
      1993 4            1.4
      1994 1            2.1
      1994 2            2.2
      1994 3            2.3
      1994 4            2.4

8 rows selected
```

## 用SQL从表A 到表B ,从表A 到表C


 表A
 
ID  | NAME 
--- | ---
1   | AA
2   | AA
3   | AA
4   |  D 
5   |  C
6   | AA
7   | AA
8   | E
9   | F 
10  | G

表B

ID | NAME
--- | ---
7   |AA,AA,AA,AA,AA
4   |D
5   |C
8   |E
9   |F
10  |G


表C

ID  | NAME
--- | ---
3   | AA,AA,AA
4   | D
5   | C
7   | AA,AA
8   | E
9   | F
10  | G


### A -> B 

```sql
WITH DAO_1 AS 
(SELECT 1 ID ,'AA' NAME FROM DUAL UNION ALL
SELECT 2 ID ,'AA' FROM DUAL UNION ALL 
SELECT 3 ID ,'AA' FROM DUAL UNION ALL
SELECT 4 ID ,'D' FROM DUAL UNION ALL
SELECT 5 ID ,'C' FROM DUAL UNION ALL
SELECT 6 ID ,'AA' FROM DUAL UNION ALL
SELECT 7 ID ,'AA' FROM DUAL UNION ALL
SELECT 8 ID ,'E' FROM DUAL UNION ALL
SELECT 9 ID ,'F' FROM DUAL UNION ALL
SELECT 10 ID ,'G' FROM DUAL )
SELECT MAX(ID),RTRIM(LISTAGG(NAME||',') WITHIN GROUP(ORDER BY NAME),',') NAME
 FROM DAO_1
 GROUP BY NAME ;

```

### A -> C

```sql

SQL> WITH DAO_1 AS
  2  (SELECT 1 ID ,'AA' NAME FROM DUAL UNION ALL
  3  SELECT 2 ID ,'AA' FROM DUAL UNION ALL
  4  SELECT 3 ID ,'AA' FROM DUAL UNION ALL
  5  SELECT 4 ID ,'D' FROM DUAL UNION ALL
  6  SELECT 5 ID ,'C' FROM DUAL UNION ALL
  7  SELECT 6 ID ,'AA' FROM DUAL UNION ALL
  8  SELECT 7 ID ,'AA' FROM DUAL UNION ALL
  9  SELECT 8 ID ,'E' FROM DUAL UNION ALL
 10  SELECT 9 ID ,'F' FROM DUAL UNION ALL
 11  SELECT 10 ID ,'G' FROM DUAL )
 12  SELECT MAX(ID),RTRIM(LISTAGG(NAME||',') WITHIN GROUP (ORDER BY NAME),',')
 13   FROM (
 14  SELECT ID ,NAME ,ID-ROW_NUMBER() OVER(PARTITION BY NAME ORDER BY ID) RN
 15   FROM DAO_1)
 16   GROUP BY RN
 17   ORDER BY 1 ;

   MAX(ID) RTRIM(LISTAGG(NAME||',')WITHIN
---------- --------------------------------------------------------------------------------
         3 AA,AA,AA
         4 D
         5 C
         7 AA,AA
         8 E
         9 F
        10 G

7 rows selected



```
