---
layout: post
title:  MySQL执行计划解析
date: 2016-01-22
categories: blog
tags: [MySQL]
description:   MySQL执行计划解析
---



## 执行计划收集

```sql

DAO_OBJECTS t1 , DAO_OBJECTS t2  WHERE deptno= t1.object_id  AND  t1.object_name=t2.object_name   ;
+----+-------------+-------+------+---------------+------+---------+------+-------+----------------------------------------------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows  | Extra                                              |
+----+-------------+-------+------+---------------+------+---------+------+-------+----------------------------------------------------+
|  1 | SIMPLE      | t1    | ALL  | NULL          | NULL | NULL    | NULL | 49793 | NULL                                               |
|  1 | SIMPLE      | EMP   | ALL  | FK_DEPTNO     | NULL | NULL    | NULL |    14 | Using where; Using join buffer (Block Nested Loop) |
|  1 | SIMPLE      | t2    | ALL  | NULL          | NULL | NULL    | NULL | 49793 | Using where; Using join buffer (Block Nested Loop) |
+----+-------------+-------+------+---------------+------+---------+------+-------+----------------------------------------------------+
```

## 执行计划列详解

### ID

ID列表明了该语句所在的层级
如果ID相同从上到下执行
如果ID不同则ID越大的越先执行
其作用类似于Oracle执行计划中缩进

```sql

mysql> explain SELECT * FROM EMP , DAO_OBJECTS1 t1 , DAO_OBJECTS2 t2  WHERE deptno= t1.object_id  AND  t1.object_name=t2.object_name   ;
+----+-------------+-------+------+---------------+------+---------+------+-------+----------------------------------------------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows  | Extra                                              |
+----+-------------+-------+------+---------------+------+---------+------+-------+----------------------------------------------------+
|  1 | SIMPLE      | t1    | ALL  | NULL          | NULL | NULL    | NULL | 49793 | NULL                                               |
|  1 | SIMPLE      | EMP   | ALL  | FK_DEPTNO     | NULL | NULL    | NULL |    14 | Using where; Using join buffer (Block Nested Loop) |
|  1 | SIMPLE      | t2    | ALL  | NULL          | NULL | NULL    | NULL | 49793 | Using where; Using join buffer (Block Nested Loop) |
+----+-------------+-------+------+---------------+------+---------+------+-------+----------------------------------------------------+

```


- 语义等价查询

```sql

mysql> EXPLAIN
    -> SELECT * 
    -> FROM (SELECT * FROM EMP , DAO_OBJECTS t1 WHERE DEPTNO=t1.object_id) V1 , DAO_OBJECTS t2 
    -> WHERE V1.object_id =t2.object_id  ;
+----+-------------+------------+------+---------------+-------------+---------+------------------+-------+----------------------------------------------------+
| id | select_type | table      | type | possible_keys | key         | key_len | ref              | rows  | Extra                                              |
+----+-------------+------------+------+---------------+-------------+---------+------------------+-------+----------------------------------------------------+
|  1 | PRIMARY     | t2         | ALL  | NULL          | NULL        | NULL    | NULL             | 49793 | Using where                                        |
|  1 | PRIMARY     | <derived2> | ref  | <auto_key0>   | <auto_key0> | 5       | dao.t2.OBJECT_ID |    11 | NULL                                               |
|  2 | DERIVED     | t1         | ALL  | NULL          | NULL        | NULL    | NULL             | 49793 | NULL                                               |
|  2 | DERIVED     | EMP        | ALL  | FK_DEPTNO     | NULL        | NULL    | NULL             |    14 | Using where; Using join buffer (Block Nested Loop) |
+----+-------------+------------+------+---------------+-------------+---------+------------------+-------+----------------------------------------------------+

``` 


以上执行计划ID发生改变 
整个执行计划顺序为 
t1 与EMP关联 得到的结果集与t2关联  (t1 join EMP) join t2  
不过执行计划有所差异 而就结果来说应该是完全相同的

### select_type

#### SIMPLE

简单查询不包含子查询与union


```sql

mysql>  explain SELECT * FROM EMP  ;
+----+-------------+-------+------+---------------+------+---------+------+------+-------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra |
+----+-------------+-------+------+---------------+------+---------+------+------+-------+
|  1 | SIMPLE      | EMP   | ALL  | NULL          | NULL | NULL    | NULL |   14 | NULL  |
+----+-------------+-------+------+---------------+------+---------+------+------+-------+
1 row in set (0.00 sec)

```

### PRIMARY

为复杂查询创建的首要表(也是最外层的表) 这种类型通常与DERIVED 或者union 混合使用见到

```sql


mysql> explain
    -> SELECT * 
    -> FROM (SELECT * FROM EMP , DAO_OBJECTS t1 WHERE DEPTNO=t1.object_id) V1 , DAO_OBJECTS t2  ,DEPT t3
    -> WHERE V1.object_id =t2.object_id   and  t3.DEPTNO= V1.DEPTNO ;
+----+-------------+------------+------+---------------+-------------+---------+------------------+-------+----------------------------------------------------+
| id | select_type | table      | type | possible_keys | key         | key_len | ref              | rows  | Extra                                              |
+----+-------------+------------+------+---------------+-------------+---------+------------------+-------+----------------------------------------------------+
|  1 | PRIMARY     | t2         | ALL  | NULL          | NULL        | NULL    | NULL             | 49793 | Using where                                        |
|  1 | PRIMARY     | <derived2> | ref  | <auto_key1>   | <auto_key1> | 5       | dao.t2.OBJECT_ID |    16 | NULL                                               |
|  1 | PRIMARY     | t3         | ALL  | PRIMARY       | NULL        | NULL    | NULL             |     4 | Using where; Using join buffer (Block Nested Loop) |
|  2 | DERIVED     | t1         | ALL  | NULL          | NULL        | NULL    | NULL             | 49793 | NULL                                               |
|  2 | DERIVED     | EMP        | ALL  | FK_DEPTNO     | NULL        | NULL    | NULL             |    14 | Using where; Using join buffer (Block Nested Loop) |
+----+-------------+------------+------+---------------+-------------+---------+------------------+-------+----------------------------------------------------+
5 rows in set (0.00 sec)

```


可以这么理解 primary 就是SQL在有子查询与union时 最外层的表 第一行(T4 )或者结果集 第二行( <derived2>)

#### UNION

当SELECT 之前的关键字为UNION 或 UNION ALL时 会出现UNION 关键字

```sql


mysql> EXPLAIN  SELECT OBJECT_ID FROM DAO_OBJECTS1 UNION SELECT DEPTNO FROM EMP  UNION  SELECT OBJECT_ID FROM DAO_OBJECTS2
    -> ;
+----+--------------+--------------+-------+---------------+-----------+---------+------+------+-----------------+
| id | select_type  | table        | type  | possible_keys | key       | key_len | ref  | rows | Extra           |
+----+--------------+--------------+-------+---------------+-----------+---------+------+------+-----------------+
|  1 | PRIMARY      | DAO_OBJECTS1 | ALL   | NULL          | NULL      | NULL    | NULL |    1 | NULL            |
|  2 | UNION        | EMP          | index | NULL          | FK_DEPTNO | 5       | NULL |   14 | Using index     |
|  3 | UNION        | DAO_OBJECTS2 | ALL   | NULL          | NULL      | NULL    | NULL |    1 | NULL            |
| NULL | UNION RESULT | <union1,2,3> | ALL   | NULL          | NULL      | NULL    | NULL | NULL | Using temporary |
+----+--------------+--------------+-------+---------------+-----------+---------+------+------+-----------------+
4 rows in set (0.00 sec)


SELECT * 
FROM DAO_OBJECTS1 
UNION ALL
SELECT *
FROM DAO_OBJECTS2 ) V1
WHERE V1' at line 4
mysql> EXPLAIN
    -> SELECT *
    -> FROM DEPT  D,
    -> (SELECT * 
    -> FROM DAO_OBJECTS1 
    -> UNION ALL
    -> SELECT *
    -> FROM DAO_OBJECTS2 ) V1
    -> WHERE V1.OBJECT_ID=D.DEPTNO  ;
	
+----+--------------+--------------+------+---------------+-------------+---------+--------------+------+-----------------+
| id | select_type  | table        | type | possible_keys | key         | key_len | ref          | rows | Extra           |
+----+--------------+--------------+------+---------------+-------------+---------+--------------+------+-----------------+
|  1 | PRIMARY      | D            | ALL  | PRIMARY       | NULL        | NULL    | NULL         |    4 | NULL            |
|  1 | PRIMARY      | <derived2>   | ref  | <auto_key0>   | <auto_key0> | 5       | dao.D.DEPTNO |    2 | NULL            |
|  2 | DERIVED      | DAO_OBJECTS1 | ALL  | NULL          | NULL        | NULL    | NULL         |    1 | NULL            |
|  3 | UNION        | DAO_OBJECTS2 | ALL  | NULL          | NULL        | NULL    | NULL         |    1 | NULL            |
| NULL | UNION RESULT | <union2,3>   | ALL  | NULL          | NULL        | NULL    | NULL         | NULL | Using temporary |
+----+--------------+--------------+------+---------------+-------------+---------+--------------+------+-----------------+
5 rows in set (0.00 sec)

```

####  DEPENDENT UNION

当子查询中存在UNION 时 UNION 后的SELECT TYPE 会出现 DEPENDENT UNION 而union 语句的第一行为 DEPENDENT SUBQUERY

```sql

mysql> EXPLAIN SELECT * FROM  DEPT D WHERE D.DEPTNO IN (SELECT DEPTNO FROM EMP UNION SELECT OBJECT_ID FROM DAO_OBJECTS1)  ;
+----+--------------------+--------------+------+---------------+-----------+---------+------+------+-----------------+
| id | select_type        | table        | type | possible_keys | key       | key_len | ref  | rows | Extra           |
+----+--------------------+--------------+------+---------------+-----------+---------+------+------+-----------------+
|  1 | PRIMARY            | D            | ALL  | NULL          | NULL      | NULL    | NULL |    4 | Using where     |
|  2 | DEPENDENT SUBQUERY | EMP          | ref  | FK_DEPTNO     | FK_DEPTNO | 5       | func |    2 | Using index     |
|  3 | DEPENDENT UNION    | DAO_OBJECTS1 | ALL  | NULL          | NULL      | NULL    | NULL |    1 | Using where     |
| NULL | UNION RESULT       | <union2,3>   | ALL  | NULL          | NULL      | NULL    | NULL | NULL | Using temporary |
+----+--------------------+--------------+------+---------------+-----------+---------+------+------+-----------------+
4 rows in set (0.00 sec)

mysql> EXPLAIN SELECT * FROM  DEPT D WHERE D.DEPTNO IN ( SELECT OBJECT_ID FROM DAO_OBJECTS1 UNION SELECT DEPTNO FROM EMP )  ;
+----+--------------------+--------------+------+---------------+-----------+---------+------+------+-----------------+
| id | select_type        | table        | type | possible_keys | key       | key_len | ref  | rows | Extra           |
+----+--------------------+--------------+------+---------------+-----------+---------+------+------+-----------------+
|  1 | PRIMARY            | D            | ALL  | NULL          | NULL      | NULL    | NULL |    4 | Using where     |
|  2 | DEPENDENT SUBQUERY | DAO_OBJECTS1 | ALL  | NULL          | NULL      | NULL    | NULL |    1 | Using where     |
|  3 | DEPENDENT UNION    | EMP          | ref  | FK_DEPTNO     | FK_DEPTNO | 5       | func |    2 | Using index     |
| NULL | UNION RESULT       | <union2,3>   | ALL  | NULL          | NULL      | NULL    | NULL | NULL | Using temporary |
+----+--------------------+--------------+------+---------------+-----------+---------+------+------+-----------------+
4 rows in set (0.00 sec)

mysql> EXPLAIN SELECT * FROM  DEPT D WHERE D.DEPTNO IN ( SELECT OBJECT_ID FROM DAO_OBJECTS1 UNION SELECT DEPTNO FROM EMP  UNION  SELECT OBJECT_ID FROM DAO_OBJECTS2)  ;
+----+--------------------+--------------+------+---------------+-----------+---------+------+------+-----------------+
| id | select_type        | table        | type | possible_keys | key       | key_len | ref  | rows | Extra           |
+----+--------------------+--------------+------+---------------+-----------+---------+------+------+-----------------+
|  1 | PRIMARY            | D            | ALL  | NULL          | NULL      | NULL    | NULL |    4 | Using where     |
|  2 | DEPENDENT SUBQUERY | DAO_OBJECTS1 | ALL  | NULL          | NULL      | NULL    | NULL |    1 | Using where     |
|  3 | DEPENDENT UNION    | EMP          | ref  | FK_DEPTNO     | FK_DEPTNO | 5       | func |    2 | Using index     |
|  4 | DEPENDENT UNION    | DAO_OBJECTS2 | ALL  | NULL          | NULL      | NULL    | NULL |    1 | Using where     |
| NULL | UNION RESULT       | <union2,3,4> | ALL  | NULL          | NULL      | NULL    | NULL | NULL | Using temporary |
+----+--------------------+--------------+------+---------------+-----------+---------+------+------+-----------------+
5 rows in set (0.00 sec)




```

####  UNION RESULT

```sql

mysql> EXPLAIN  SELECT OBJECT_ID FROM DAO_OBJECTS1 UNION SELECT DEPTNO FROM EMP  UNION  SELECT OBJECT_ID FROM DAO_OBJECTS2 ;
+----+--------------+--------------+-------+---------------+-----------+---------+------+------+-----------------+
| id | select_type  | table        | type  | possible_keys | key       | key_len | ref  | rows | Extra           |
+----+--------------+--------------+-------+---------------+-----------+---------+------+------+-----------------+
|  1 | PRIMARY      | DAO_OBJECTS1 | ALL   | NULL          | NULL      | NULL    | NULL |    1 | NULL            |
|  2 | UNION        | EMP          | index | NULL          | FK_DEPTNO | 5       | NULL |   14 | Using index     |
|  3 | UNION        | DAO_OBJECTS2 | ALL   | NULL          | NULL      | NULL    | NULL |    1 | NULL            |
| NULL | UNION RESULT | <union1,2,3> | ALL   | NULL          | NULL      | NULL    | NULL | NULL | Using temporary |
+----+--------------+--------------+-------+---------------+-----------+---------+------+------+-----------------+
4 rows in set (0.00 sec)


mysql>  EXPLAIN SELECT * FROM  DEPT D WHERE D.DEPTNO IN ( SELECT OBJECT_ID FROM DAO_OBJECTS1 UNION SELECT DEPTNO FROM EMP  UNION  SELECT OBJECT_ID FROM DAO_OBJECTS2) ;
+----+--------------------+--------------+------+---------------+-----------+---------+------+------+-----------------+
| id | select_type        | table        | type | possible_keys | key       | key_len | ref  | rows | Extra           |
+----+--------------------+--------------+------+---------------+-----------+---------+------+------+-----------------+
|  1 | PRIMARY            | D            | ALL  | NULL          | NULL      | NULL    | NULL |    4 | Using where     |
|  2 | DEPENDENT SUBQUERY | DAO_OBJECTS1 | ALL  | NULL          | NULL      | NULL    | NULL |    1 | Using where     |
|  3 | DEPENDENT UNION    | EMP          | ref  | FK_DEPTNO     | FK_DEPTNO | 5       | func |    2 | Using index     |
|  4 | DEPENDENT UNION    | DAO_OBJECTS2 | ALL  | NULL          | NULL      | NULL    | NULL |    1 | Using where     |
| NULL | UNION RESULT       | <union2,3,4> | ALL  | NULL          | NULL      | NULL    | NULL | NULL | Using temporary |
+----+--------------------+--------------+------+---------------+-----------+---------+------+------+-----------------+
5 rows in set (0.00 sec)



```

####  SUBQUERY

出现在复杂非相关子查询中 ,简单相关子查询MySQL会进行改写

```sql


mysql> EXPLAIN
    -> SELECT *
    ->   FROM EMP
    ->  WHERE EMPNO IN ( SELECT COUNT(*) FROM DAO_OBJECTS1 ) ;
+----+-------------+--------------+-------+---------------+---------+---------+------+------+-------------+
| id | select_type | table        | type  | possible_keys | key     | key_len | ref  | rows | Extra       |
+----+-------------+--------------+-------+---------------+---------+---------+------+------+-------------+
|  1 | PRIMARY     | EMP          | ALL   | NULL          | NULL    | NULL    | NULL |   14 | Using where |
|  2 | SUBQUERY    | DAO_OBJECTS1 | index | NULL          | PRIMARY | 32      | NULL |    1 | Using index |
+----+-------------+--------------+-------+---------------+---------+---------+------+------+-------------+
2 rows in set (0.03 sec)


mysql> EXPLAIN SELECT * FROM DEPT D WHERE D.DEPTNO IN (SELECT DEPTNO FROM EMP E ) ;
+----+-------------+-------+------+---------------+-----------+---------+--------------+------+----------------------------+
| id | select_type | table | type | possible_keys | key       | key_len | ref          | rows | Extra                      |
+----+-------------+-------+------+---------------+-----------+---------+--------------+------+----------------------------+
|  1 | SIMPLE      | D     | ALL  | PRIMARY       | NULL      | NULL    | NULL         |    4 | NULL                       |
|  1 | SIMPLE      | E     | ref  | FK_DEPTNO     | FK_DEPTNO | 5       | dao.D.DEPTNO |    2 | Using index; FirstMatch(D) |
+----+-------------+-------+------+---------------+-----------+---------+--------------+------+----------------------------+
2 rows in set (0.00 sec)

```

注意:这里发生了改写 变成了关联 不过是特殊关联 Extra 部分 FirstMatch(D) 只取一条


####  DEPENDENT SUBQUERY


出现在相关子查询中 而非相关子查询MySQL可以进行改写


```sql

mysql> EXPLAIN SELECT * FROM DEPT D WHERE EXISTS  (SELECT DEPTNO FROM EMP E WHERE D.DEPTNO=E.DEPTNO) ;
+----+--------------------+-------+------+---------------+-----------+---------+--------------+------+-------------+
| id | select_type        | table | type | possible_keys | key       | key_len | ref          | rows | Extra       |
+----+--------------------+-------+------+---------------+-----------+---------+--------------+------+-------------+
|  1 | PRIMARY            | D     | ALL  | NULL          | NULL      | NULL    | NULL         |    4 | Using where |
|  2 | DEPENDENT SUBQUERY | E     | ref  | FK_DEPTNO     | FK_DEPTNO | 5       | dao.D.DEPTNO |    2 | Using index |
+----+--------------------+-------+------+---------------+-----------+---------+--------------+------+-------------+
2 rows in set (0.00 sec)


```

####  DERIVED 

衍生表 当查询使用内联视图时 会出现此关键字


```sql

mysql> EXPLAIN
    -> SELECT *
    ->   FROM EMP T1, (SELECT T2.DEPTNO, AVG(SAL) AVG_SAL FROM EMP T2 GROUP BY T2.DEPTNO) V1
    ->   WHERE T1.DEPTNO=V1.DEPTNO
    ->     AND T1.SAL < AVG_SAL ;
+----+-------------+------------+-------+---------------+-------------+---------+---------------+------+--------------------------+
| id | select_type | table      | type  | possible_keys | key         | key_len | ref           | rows | Extra                    |
+----+-------------+------------+-------+---------------+-------------+---------+---------------+------+--------------------------+
|  1 | PRIMARY     | T1         | ALL   | FK_DEPTNO     | NULL        | NULL    | NULL          |   14 | Using where              |
|  1 | PRIMARY     | <derived2> | ref   | <auto_key0>   | <auto_key0> | 5       | dao.T1.DEPTNO |    2 | Using where; Using index |
|  2 | DERIVED     | T2         | index | FK_DEPTNO     | FK_DEPTNO   | 5       | NULL          |   14 | NULL                     |
+----+-------------+------------+-------+---------------+-------------+---------+---------------+------+--------------------------+
3 rows in set (0.00 sec)



```

#### MATERIALIZED

子查询物化
当表出现在非相关子查询中 并且需要进行物化时会出现MATERIALIZED关键词


```sql


mysql> EXPLAIN
    -> SELECT *
    ->   FROM EMP
    ->  WHERE EMPNO IN (SELECT OBJECT_ID
    ->                    FROM DAO_OBJECTS1 T1 
    ->                   WHERE EXISTS (SELECT  1 
    ->                                   FROM DEPT  D 
    ->                                  WHERE  T1.OBJECT_ID = D.DEPTNO) );
+----+--------------------+-------------+--------+---------------+---------+---------+-----------------------+------+-------------+
| id | select_type        | table       | type   | possible_keys | key     | key_len | ref                   | rows | Extra       |
+----+--------------------+-------------+--------+---------------+---------+---------+-----------------------+------+-------------+
|  1 | PRIMARY            | <subquery2> | ALL    | NULL          | NULL    | NULL    | NULL                  | NULL | Using where |
|  1 | PRIMARY            | EMP         | eq_ref | PRIMARY       | PRIMARY | 4       | <subquery2>.OBJECT_ID |    1 | NULL        |
|  2 | MATERIALIZED       | T1          | ALL    | NULL          | NULL    | NULL    | NULL                  |    1 | Using where |
|  3 | DEPENDENT SUBQUERY | D           | eq_ref | PRIMARY       | PRIMARY | 4       | dao.T1.OBJECT_ID      |    1 | Using index |
+----+--------------------+-------------+--------+---------------+---------+---------+-----------------------+------+-------------+
4 rows in set (0.01 sec)


mysql> EXPLAIN EXTENDED
    -> SELECT *
    ->   FROM EMP
    ->  WHERE EMPNO IN (SELECT T1.OBJECT_ID
    ->                    FROM DAO_OBJECTS1 T1 ,DAO_OBJECTS1 T2 
    ->                   WHERE T1.OBJECT_ID =T2.OBJECT_ID
    ->                     AND EXISTS (SELECT  1
    ->                                  FROM DEPT D       
    ->                                 WHERE D.DEPTNO= T1.OBJECT_ID )
    ->                      );
+----+--------------------+-------------+--------+---------------+---------+---------+-----------------------+------+----------+----------------------------------------------------+
| id | select_type        | table       | type   | possible_keys | key     | key_len | ref                   | rows | filtered | Extra                                              |
+----+--------------------+-------------+--------+---------------+---------+---------+-----------------------+------+----------+----------------------------------------------------+
|  1 | PRIMARY            | <subquery2> | ALL    | NULL          | NULL    | NULL    | NULL                  | NULL |     0.00 | Using where                                        |
|  1 | PRIMARY            | EMP         | eq_ref | PRIMARY       | PRIMARY | 4       | <subquery2>.OBJECT_ID |    1 |   100.00 | NULL                                               |
|  2 | MATERIALIZED       | T1          | ALL    | NULL          | NULL    | NULL    | NULL                  |    1 |   100.00 | Using where                                        |
|  2 | MATERIALIZED       | T2          | ALL    | NULL          | NULL    | NULL    | NULL                  |    1 |   100.00 | Using where; Using join buffer (Block Nested Loop) |
|  3 | DEPENDENT SUBQUERY | D           | eq_ref | PRIMARY       | PRIMARY | 4       | dao.T1.OBJECT_ID      |    1 |   100.00 | Using index                                        |
+----+--------------------+-------------+--------+---------------+---------+---------+-----------------------+------+----------+----------------------------------------------------+
5 rows in set, 2 warnings (0.00 sec)



```

###  UNCACHEABLE SUBQUERY

UNCACHEABLE SUBQUERY表示子查询不可被物化 需要逐次运行

```sql


mysql> EXPLAIN SELECT     ( SELECT @rownum:= @rownum + sal+D.deptno  from EMP limit 1  )                           
    ->   FROM DEPT D  ;                                                                  
+----+--------------------+-------+-------+---------------+---------+---------+------+------+-------------+
| id | select_type        | table | type  | possible_keys | key     | key_len | ref  | rows | Extra       |
+----+--------------------+-------+-------+---------------+---------+---------+------+------+-------------+
|  1 | PRIMARY            | D     | index | NULL          | PRIMARY | 4       | NULL |    4 | Using index |
|  2 | DEPENDENT SUBQUERY | EMP   | ALL   | NULL          | NULL    | NULL    | NULL |   14 | NULL        |
+----+--------------------+-------+-------+---------------+---------+---------+------+------+-------------+
2 rows in set (0.00 sec)



```

####   UNCACHEABLE UNION

```sql

mysql> explain  SELECT      (select xx.c1 from ( SELECT @rownum c1 from EMP  union  SELECT @rownum from EMP  )  xx limit 1 )
    ->     FROM DEPT D  ;
+----+----------------------+------------+-------+---------------+-----------+---------+------+------+-----------------+
| id | select_type          | table      | type  | possible_keys | key       | key_len | ref  | rows | Extra           |
+----+----------------------+------------+-------+---------------+-----------+---------+------+------+-----------------+
|  1 | PRIMARY              | D          | index | NULL          | PRIMARY   | 4       | NULL |    4 | Using index     |
|  2 | UNCACHEABLE SUBQUERY | <derived3> | ALL   | NULL          | NULL      | NULL    | NULL |   28 | NULL            |
|  3 | DERIVED              | EMP        | index | NULL          | FK_DEPTNO | 5       | NULL |   14 | Using index     |
|  4 | UNCACHEABLE UNION    | EMP        | index | NULL          | FK_DEPTNO | 5       | NULL |   14 | Using index     |
| NULL | UNION RESULT         | <union3,4> | ALL   | NULL          | NULL      | NULL    | NULL | NULL | Using temporary |
+----+----------------------+------------+-------+---------------+-----------+---------+------+------+-----------------+
5 rows in set (0.00 sec)


```


### table

#### table_name

访问的表,注意即使访问的是索引 这里也显示访问的表名
关于索引信息在 type key 中进行描述


```sql

mysql> EXPLAIN   SELECT OBJECT_ID FROM DAO_OBJECTS1 ;
+----+-------------+--------------+-------+---------------+---------+---------+------+-------+-------------+
| id | select_type | table        | type  | possible_keys | key     | key_len | ref  | rows  | Extra       |
+----+-------------+--------------+-------+---------------+---------+---------+------+-------+-------------+
|  1 | SIMPLE      | DAO_OBJECTS1 | index | NULL          | PRIMARY | 4       | NULL | 52025 | Using index |
+----+-------------+--------------+-------+---------------+---------+---------+------+-------+-------------+
1 row in set (0.00 sec)

mysql> explain  SELECT      (select xx.c1 from ( SELECT @rownum c1 from EMP  union  SELECT @rownum from EMP  )  xx limit 1 )
    ->     FROM DEPT D  ;
+----+----------------------+------------+-------+---------------+-----------+---------+------+------+-----------------+
| id | select_type          | table      | type  | possible_keys | key       | key_len | ref  | rows | Extra           |
+----+----------------------+------------+-------+---------------+-----------+---------+------+------+-----------------+
|  1 | PRIMARY              | D          | index | NULL          | PRIMARY   | 4       | NULL |    4 | Using index     |
|  2 | UNCACHEABLE SUBQUERY | <derived3> | ALL   | NULL          | NULL      | NULL    | NULL |   28 | NULL            |
|  3 | DERIVED              | **EMP**        | index | NULL          | FK_DEPTNO | 5       | NULL |   14 | Using index     |
|  4 | UNCACHEABLE UNION    | EMP        | index | NULL          | FK_DEPTNO | 5       | NULL |   14 | Using index     |
| NULL | UNION RESULT         | <union3,4> | ALL   | NULL          | NULL      | NULL    | NULL | NULL | Using temporary |
+----+----------------------+------------+-------+---------------+-----------+---------+------+------+-----------------+
5 rows in set (0.00 sec)


```

####  (union m,n)


表示几个行源进行union

```sql

+----+----------------------+------------+-------+---------------+-----------+---------+------+------+-----------------+
| id | select_type          | table      | type  | possible_keys | key       | key_len | ref  | rows | Extra           |
+----+----------------------+------------+-------+---------------+-----------+---------+------+------+-----------------+
|  1 | PRIMARY              | D          | index | NULL          | PRIMARY   | 4       | NULL |    4 | Using index     |
|  2 | UNCACHEABLE SUBQUERY | <derived3> | ALL   | NULL          | NULL      | NULL    | NULL |   28 | NULL            |
|  3 | DERIVED              | EMP        | index | NULL          | FK_DEPTNO | 5       | NULL |   14 | Using index     |
|  4 | UNCACHEABLE UNION    | EMP        | index | NULL          | FK_DEPTNO | 5       | NULL |   14 | Using index     |
| NULL | UNION RESULT       | **<union3,4>** | ALL   | NULL          | NULL      | NULL    | NULL | NULL | Using temporary |
+----+----------------------+------------+-------+---------------+-----------+---------+------+------+-----------------+
5 rows in set (0.00 sec)
```
#### <derivedn>

表示使用内联视图

```sql

mysql> explain  SELECT      **(select xx.c1 from ( SELECT @rownum c1 from EMP  union  SELECT @rownum from EMP  )**  xx limit 1 )
    ->     FROM DEPT D  ;
+----+----------------------+------------+-------+---------------+-----------+---------+------+------+-----------------+
| id | select_type          | table      | type  | possible_keys | key       | key_len | ref  | rows | Extra           |
+----+----------------------+------------+-------+---------------+-----------+---------+------+------+-----------------+
|  1 | PRIMARY              | D          | index | NULL          | PRIMARY   | 4       | NULL |    4 | Using index     |
|  2 | UNCACHEABLE SUBQUERY | **<derived3>** | ALL   | NULL          | NULL      | NULL    | NULL |   28 | NULL            |
|  3 | DERIVED              | EMP        | index | NULL          | FK_DEPTNO | 5       | NULL |   14 | Using index     |
|  4 | UNCACHEABLE UNION    | EMP        | index | NULL          | FK_DEPTNO | 5       | NULL |   14 | Using index     |
| NULL | UNION RESULT       | <union3,4> | ALL   | NULL          | NULL      | NULL    | NULL | NULL | Using temporary |
+----+----------------------+------------+-------+---------------+-----------+---------+------+------+-----------------+
5 rows in set (0.00 sec)


```

### type

type 表示的访问路径,及数据获取途径

####  type 优先级

[查看type优先级]( http://dev.mysql.com/doc/refman/5.7/en/explain-output.html#jointype_ref)

#### null

不访问任何表

```sql


mysql> explain select current_timestamp ;
+----+-------------+-------+------+---------------+------+---------+------+------+----------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra          |
+----+-------------+-------+------+---------------+------+---------+------+------+----------------+
|  1 | SIMPLE      | NULL  | NULL | NULL          | NULL | NULL    | NULL | NULL | No tables used |
+----+-------------+-------+------+---------------+------+---------+------+------+----------------+
1 row in set (0.00 sec)


mysql>  EXPLAIN 
    ->   SELECT * 
    ->     FROM DAO_OBJECTS1 
    ->    WHERE OBJECT_NAME ='DBA_TABLES' 
    ->     and  OBJECT_ID='9527';
+----+-------------+-------+------+---------------+------+---------+------+------+-----------------------------------------------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra                                               |
+----+-------------+-------+------+---------------+------+---------+------+------+-----------------------------------------------------+
|  1 | SIMPLE      | NULL  | NULL | NULL          | NULL | NULL    | NULL | NULL | Impossible WHERE noticed after reading const tables |
+----+-------------+-------+------+---------------+------+---------+------+------+-----------------------------------------------------+
1 row in set (0.01 sec)

```


####  system

根据官档所说 这种方式属于一种特殊的const 要求表中只有一条记录

```sql

mysql> analyze table TYPE_SYSTEM ;
+-----------------+---------+----------+----------+
| Table           | Op      | Msg_type | Msg_text |
+-----------------+---------+----------+----------+
| dao.TYPE_SYSTEM | analyze | status   | OK       |
+-----------------+---------+----------+----------+
1 row in set (0.02 sec)

mysql> use dao ;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> create table TYPE_SYSTEM (c1 int primary key ) ;
Query OK, 0 rows affected (0.05 sec)

mysql> insert into TYPE_SYSTEM values(1) ;
Query OK, 1 row affected (0.02 sec)

mysql> explain select 1 from TYPE_SYSTEM where c1=1 ;
+----+-------------+-------------+-------+---------------+---------+---------+-------+------+-------------+
| id | select_type | table       | type  | possible_keys | key     | key_len | ref   | rows | Extra       |
+----+-------------+-------------+-------+---------------+---------+---------+-------+------+-------------+
|  1 | SIMPLE      | TYPE_SYSTEM | const | PRIMARY       | PRIMARY | 4       | const |    1 | Using index |
+----+-------------+-------------+-------+---------------+---------+---------+-------+------+-------------+
1 row in set (0.00 sec)

```

#### const

此种类型查询最多返回一行记录,由于只有一行记录,后续优化器会把它当作常量.
基本可以认为表进行了索引唯一性扫描,并且唯一性索引只返回一行记录


```sql

mysql> explain select object_id  from dao.DAO_OBJECTS1 WHERE OBJECT_ID =2  ;
+----+-------------+--------------+-------+---------------+---------+---------+-------+------+-------------+
| id | select_type | table        | type  | possible_keys | key     | key_len | ref   | rows | Extra       |
+----+-------------+--------------+-------+---------------+---------+---------+-------+------+-------------+
|  1 | SIMPLE      | DAO_OBJECTS1 | const | PRIMARY       | PRIMARY | 4       | const |    1 | Using index |
+----+-------------+--------------+-------+---------------+---------+---------+-------+------+-------------+
1 row in set (0.00 sec)


mysql> explain
    -> select *
    ->   from DAO_OBJECTS2
    ->  where object_name in
    ->        (select object_name
    ->           from dao.DAO_OBJECTS1 
    ->          WHERE OBJECT_ID = 9527);
+----+-------------+--------------+-------+---------------+---------+---------+-------+-------+-------------+
| id | select_type | table        | type  | possible_keys | key     | key_len | ref   | rows  | Extra       |
+----+-------------+--------------+-------+---------------+---------+---------+-------+-------+-------------+
|  1 | SIMPLE      | DAO_OBJECTS1 | const | PRIMARY       | PRIMARY | 4       | const |     1 | NULL        |
|  1 | SIMPLE      | DAO_OBJECTS2 | ALL   | NULL          | NULL    | NULL    | NULL  | 51976 | Using where |
+----+-------------+--------------+-------+---------------+---------+---------+-------+-------+-------------+
2 rows in set (0.00 sec)




mysql> explain
    -> select *
    ->   from DAO_OBJECTS2
    ->  where object_id in
    ->        (select object_id 
    ->           from dao.DAO_OBJECTS1 
    ->          WHERE OBJECT_ID = 9527);
+----+-------------+--------------+-------+---------------+---------+---------+-------+------+-------------+
| id | select_type | table        | type  | possible_keys | key     | key_len | ref   | rows | Extra       |
+----+-------------+--------------+-------+---------------+---------+---------+-------+------+-------------+
|  1 | SIMPLE      | DAO_OBJECTS2 | const | PRIMARY       | PRIMARY | 4       | const |    1 | NULL        |
|  1 | SIMPLE      | DAO_OBJECTS1 | const | PRIMARY       | PRIMARY | 4       | const |    1 | Using index |
+----+-------------+--------------+-------+---------------+---------+---------+-------+------+-------------+
2 rows in set (0.00 sec)



mysql> explain select * from DAO_OBJECTS2 where object_id =9527 ;
+----+-------------+--------------+-------+---------------+---------+---------+-------+------+-------+
| id | select_type | table        | type  | possible_keys | key     | key_len | ref   | rows | Extra |
+----+-------------+--------------+-------+---------------+---------+---------+-------+------+-------+
|  1 | SIMPLE      | DAO_OBJECTS2 | const | PRIMARY       | PRIMARY | 4       | const |    1 | NULL  |
+----+-------------+--------------+-------+---------------+---------+---------+-------+------+-------+
1 row in set (0.00 sec)


```

####  eq_ref

关联时进行多次索引唯一性扫描

```sql

mysql> EXPLAIN 
    -> select * 
    ->   from DAO_OBJECTS1 T1 ,EMP E
    ->  WHERE E.DEPTNO= T1.OBJECT_ID ;
+----+-------------+-------+--------+---------------+---------+---------+--------------+------+-------------+
| id | select_type | table | type   | possible_keys | key     | key_len | ref          | rows | Extra       |
+----+-------------+-------+--------+---------------+---------+---------+--------------+------+-------------+
|  1 | SIMPLE      | E     | ALL    | FK_DEPTNO     | NULL    | NULL    | NULL         |   14 | Using where |
|  1 | SIMPLE      | T1    | eq_ref | PRIMARY       | PRIMARY | 4       | dao.E.DEPTNO |    1 | NULL        |
+----+-------------+-------+--------+---------------+---------+---------+--------------+------+-------------+
2 rows in set (0.00 sec)


```

####  ref

索引非唯一性扫描

```sql

mysql> EXPLAIN
    -> SELECT * 
    ->   FROM DAO_OBJECTS1 
    ->  WHERE OBJECT_NAME = 'DBA_TABLES' ;
+----+-------------+--------------+------+---------------+---------------+---------+-------+------+-----------------------+
| id | select_type | table        | type | possible_keys | key           | key_len | ref   | rows | Extra                 |
+----+-------------+--------------+------+---------------+---------------+---------+-------+------+-----------------------+
|  1 | SIMPLE      | DAO_OBJECTS1 | ref  | IDX2_OBJECTS2 | IDX2_OBJECTS2 | 131     | const |    2 | Using index condition |
+----+-------------+--------------+------+---------------+---------------+---------+-------+------+-----------------------+
1 row in set (0.02 sec)

```

#### fulltext

使用全文索引时出现

```sql

mysql> ALTER TABLE DAO_OBJECTS2 ADD FULLTEXT( OBJECT_NAME );
Query OK, 0 rows affected, 1 warning (2.15 sec)
Records: 0  Duplicates: 0  Warnings: 1

mysql> EXPLAIN 
    -> SELECT * 
    ->   FROM DAO_OBJECTS2 
    ->  WHERE match(OBJECT_NAME) AGAINST('TABLES') ;
+----+-------------+--------------+----------+---------------+-------------+---------+------+------+-------------+
| id | select_type | table        | type     | possible_keys | key         | key_len | ref  | rows | Extra       |
+----+-------------+--------------+----------+---------------+-------------+---------+------+------+-------------+
|  1 | SIMPLE      | DAO_OBJECTS2 | fulltext | OBJECT_NAME   | OBJECT_NAME | 0       | NULL |    1 | Using where |
+----+-------------+--------------+----------+---------------+-------------+---------+------+------+-------------+
1 row in set (0.00 sec)


```

#### ref_or_null

当谓词出现索引列等于某值或为空时出现

```sql

mysql>  EXPLAIN 
    ->   SELECT * 
    ->     FROM DAO_OBJECTS1 
    ->    WHERE OBJECT_NAME ='DBA_TABLES' OR OBJECT_NAME is NULL ;
+----+-------------+--------------+-------------+---------------+---------------+---------+-------+------+-----------------------+
| id | select_type | table        | type        | possible_keys | key           | key_len | ref   | rows | Extra                 |
+----+-------------+--------------+-------------+---------------+---------------+---------+-------+------+-----------------------+
|  1 | SIMPLE      | DAO_OBJECTS1 | ref_or_null | IDX2_OBJECTS2 | IDX2_OBJECTS2 | 131     | const |    3 | Using index condition |
+----+-------------+--------------+-------------+---------------+---------------+---------+-------+------+-----------------------+
1 row in set (0.00 sec)


```

#### index_merge


```sql

mysql>  EXPLAIN 
    ->   SELECT * 
    ->     FROM DAO_OBJECTS1 
    ->    WHERE OBJECT_NAME ='DBA_TABLES' OR OBJECT_ID='9527';
+----+-------------+--------------+-------------+-----------------------+-----------------------+---------+------+------+-------------------------------------------------+
| id | select_type | table        | type        | possible_keys         | key                   | key_len | ref  | rows | Extra                                           |
+----+-------------+--------------+-------------+-----------------------+-----------------------+---------+------+------+-------------------------------------------------+
|  1 | SIMPLE      | DAO_OBJECTS1 | index_merge | PRIMARY,IDX2_OBJECTS2 | IDX2_OBJECTS2,PRIMARY | 131,4   | NULL |    3 | Using union(IDX2_OBJECTS2,PRIMARY); Using where |
+----+-------------+--------------+-------------+-----------------------+-----------------------+---------+------+------+-------------------------------------------------+
1 row in set (0.01 sec)



```

####  range

通过范围方式访问索引

```sql

mysql> explain 
    -> SELECT * FROM DAO_OBJECTS1 WHERE OBJECT_ID BETWEEN 2000 AND 4000 ;
+----+-------------+--------------+-------+---------------+---------+---------+------+------+-------------+
| id | select_type | table        | type  | possible_keys | key     | key_len | ref  | rows | Extra       |
+----+-------------+--------------+-------+---------------+---------+---------+------+------+-------------+
|  1 | SIMPLE      | DAO_OBJECTS1 | range | PRIMARY       | PRIMARY | 4       | NULL | 4012 | Using where |
+----+-------------+--------------+-------+---------------+---------+---------+------+------+-------------+
1 row in set (0.05 sec)


```
####  index


索引全扫描  正常情况下应该在在没有过滤谓词时出现
```sql

mysql> EXPLAIN SELECT OBJECT_NAME FROM DAO_OBJECTS1  ORDER BY OBJECT_NAME DESC ;
+----+-------------+--------------+-------+---------------+---------------+---------+------+-------+-------------+
| id | select_type | table        | type  | possible_keys | key           | key_len | ref  | rows  | Extra       |
+----+-------------+--------------+-------+---------------+---------------+---------+------+-------+-------------+
|  1 | SIMPLE      | DAO_OBJECTS1 | index | NULL          | IDX2_OBJECTS2 | 131     | NULL | 52025 | Using index |
+----+-------------+--------------+-------+---------------+---------------+---------+------+-------+-------------+
1 row in set (0.00 sec)


```

#### ALL


全表扫描

```sql


mysql> EXPLAIN SELECT OBJECT_TYPE FROM DAO_OBJECTS1 
    -> ;
+----+-------------+--------------+------+---------------+------+---------+------+-------+-------+
| id | select_type | table        | type | possible_keys | key  | key_len | ref  | rows  | Extra |
+----+-------------+--------------+------+---------------+------+---------+------+-------+-------+
|  1 | SIMPLE      | DAO_OBJECTS1 | ALL  | NULL          | NULL | NULL    | NULL | 52025 | NULL  |
+----+-------------+--------------+------+---------------+------+---------+------+-------+-------+
1 row in set (0.00 sec)

```

## possible_keys

显示可能被使用的索引 ,并不是所有的索引

```sql


mysql> SHOW INDEXES FROM DAO_OBJECTS1 ;
+--------------+------------+---------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| Table        | Non_unique | Key_name      | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment |
+--------------+------------+---------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| DAO_OBJECTS1 |          0 | PRIMARY       |            1 | OBJECT_ID   | A         |       52025 |     NULL | NULL   |      | BTREE      |         |               |
| DAO_OBJECTS1 |          1 | IDX2_OBJECTS2 |            1 | OBJECT_NAME | A         |       52025 |     NULL | NULL   | YES  | BTREE      |         |               |
| DAO_OBJECTS1 |          1 | IDX3_OBJECTS1 |            1 | OWNER       | A         |          54 |     NULL | NULL   |      | BTREE      |         |               |
+--------------+------------+---------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
3 rows in set (0.00 sec)

mysql> EXPLAIN SELECT * FROM DAO_OBJECTS1 WHERE OBJECT_NAME = 'DBA_TABLES' ;
+----+-------------+--------------+------+---------------+---------------+---------+-------+------+-----------------------+
| id | select_type | table        | type | possible_keys | key           | key_len | ref   | rows | Extra                 |
+----+-------------+--------------+------+---------------+---------------+---------+-------+------+-----------------------+
|  1 | SIMPLE      | DAO_OBJECTS1 | ref  | IDX2_OBJECTS2 | IDX2_OBJECTS2 | 131     | const |    2 | Using index condition |
+----+-------------+--------------+------+---------------+---------------+---------+-------+------+-----------------------+
1 row in set (0.00 sec)


mysql> EXPLAIN SELECT * FROM DAO_OBJECTS1 WHERE OBJECT_NAME = 'DBA_TABLES' AND OWNER='DAO' ;
+----+-------------+--------------+-------------+-----------------------------+-----------------------------+---------+------+------+-----------------------------------------------------------+
| id | select_type | table        | type        | possible_keys               | key                         | key_len | ref  | rows | Extra                                                     |
+----+-------------+--------------+-------------+-----------------------------+-----------------------------+---------+------+------+-----------------------------------------------------------+
|  1 | SIMPLE      | DAO_OBJECTS1 | index_merge | IDX2_OBJECTS2,IDX3_OBJECTS1 | IDX2_OBJECTS2,IDX3_OBJECTS1 | 131,32  | NULL |    1 | Using intersect(IDX2_OBJECTS2,IDX3_OBJECTS1); Using where |
+----+-------------+--------------+-------------+-----------------------------+-----------------------------+---------+------+------+-----------------------------------------------------------+
1 row in set (0.01 sec)


```

注意:这里possible_keys中并没有出现PRIMARY
     但是我仍然感叹与MySQL中index merge 出现的频率.这可能innodb 存储结构有关系

### key

被使用的索引

```sql


mysql> EXPLAIN SELECT * FROM DAO_OBJECTS1 WHERE OBJECT_NAME = 'DBA_TABLES' ;
+----+-------------+--------------+------+---------------+---------------+---------+-------+------+-----------------------+
| id | select_type | table        | type | possible_keys | key           | key_len | ref   | rows | Extra                 |
+----+-------------+--------------+------+---------------+---------------+---------+-------+------+-----------------------+
|  1 | SIMPLE      | DAO_OBJECTS1 | ref  | IDX2_OBJECTS2 | IDX2_OBJECTS2 | 131     | const |    2 | Using index condition |
+----+-------------+--------------+------+---------------+---------------+---------+-------+------+-----------------------+
1 row in set (0.00 sec)

```

###  key_len

key_len 计算公式为
可变长度*字符集位数+(如果可为null+1 否则+0)+2  声明变长字段
固定字段 int 4 +(not null +1  else 0 )

```sql




mysql> EXPLAIN SELECT * FROM DAO_OBJECTS1 WHERE OBJECT_NAME = 'DBA_TABLES' ;
+----+-------------+--------------+------+---------------+---------------+---------+-------+------+-----------------------+
| id | select_type | table        | type | possible_keys | key           | key_len | ref   | rows | Extra                 |
+----+-------------+--------------+------+---------------+---------------+---------+-------+------+-----------------------+
|  1 | SIMPLE      | DAO_OBJECTS1 | ref  | IDX2_OBJECTS2 | IDX2_OBJECTS2 | 131     | const |    2 | Using index condition |
+----+-------------+--------------+------+---------------+---------------+---------+-------+------+-----------------------+
1 row in set (0.00 sec)

 CREATE TABLE `DAO_OBJECTS1` (
  `OWNER` varchar(30) NOT NULL,
  `OBJECT_NAME` varchar(128) DEFAULT NULL,
  `SUBOBJECT_NAME` varchar(30) DEFAULT NULL,
  `OBJECT_ID` int(11) NOT NULL DEFAULT '0',
  `DATA_OBJECT_ID` int(11) DEFAULT NULL,
  `OBJECT_TYPE` varchar(19) DEFAULT NULL,
  `CREATED` datetime DEFAULT NULL,
  `LAST_DDL_TIME` datetime DEFAULT NULL,
  `TIMESTAMP` varchar(19) DEFAULT NULL,
  `STATUS` varchar(7) DEFAULT NULL,
  `TEMPORARY` varchar(1) DEFAULT NULL,
  `GENERATED` varchar(1) DEFAULT NULL,
  `SECONDARY` varchar(1) DEFAULT NULL,
  PRIMARY KEY (`OBJECT_ID`),
  KEY `IDX2_OBJECTS2` (`OBJECT_NAME`),
  KEY `IDX3_OBJECTS1` (`OWNER`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1 |

key_len=128+1(null)+2(var变长)


mysql> EXPLAIN SELECT * FROM DAO_OBJECTS1 WHERE OBJECT_ID=9527 ;
+----+-------------+--------------+-------+---------------+---------+---------+-------+------+-------+
| id | select_type | table        | type  | possible_keys | key     | key_len | ref   | rows | Extra |
+----+-------------+--------------+-------+---------------+---------+---------+-------+------+-------+
|  1 | SIMPLE      | DAO_OBJECTS1 | const | PRIMARY       | PRIMARY | 4       | const |    1 | NULL  |
+----+-------------+--------------+-------+---------------+---------+---------+-------+------+-------+
1 row in set (0.00 sec)

key_len=4(int)+0(primary key not null 0 ) +0 (固定长度)


```

### ref

```sql
mysql> EXPLAIN 
    -> select * 
    ->   from DAO_OBJECTS1 T1 ,EMP E
    ->  WHERE E.DEPTNO= T1.OBJECT_ID ;
+----+-------------+-------+--------+---------------+---------+---------+--------------+------+-------------+
| id | select_type | table | type   | possible_keys | key     | key_len | ref          | rows | Extra       |
+----+-------------+-------+--------+---------------+---------+---------+--------------+------+-------------+
|  1 | SIMPLE      | E     | ALL    | FK_DEPTNO     | NULL    | NULL    | NULL         |   14 | Using where |
|  1 | SIMPLE      | T1    | eq_ref | PRIMARY       | PRIMARY | 4       | dao.E.DEPTNO |    1 | NULL        |
+----+-------------+-------+--------+---------------+---------+---------+--------------+------+-------------+
2 rows in set (0.00 sec)


mysql> EXPLAIN SELECT COUNT(*) FROM DAO_OBJECTS1 T1 ,DAO_OBJECTS2 T2 WHERE T1.OBJECT_ID=T2.OBJECT_ID
    -> ;
+----+-------------+-------+--------+---------------+---------+---------+------------------+-------+-------------+
| id | select_type | table | type   | possible_keys | key     | key_len | ref              | rows  | Extra       |
+----+-------------+-------+--------+---------------+---------+---------+------------------+-------+-------------+
|  1 | SIMPLE      | T2    | index  | PRIMARY       | PRIMARY | 4       | NULL             | 50887 | Using index |
|  1 | SIMPLE      | T1    | eq_ref | PRIMARY       | PRIMARY | 4       | dao.T2.OBJECT_ID |     1 | Using index |
+----+-------------+-------+--------+---------------+---------+---------+------------------+-------+-------------+
2 rows in set (0.00 sec)


mysql> EXPLAIN SELECT * FROM DAO_OBJECTS1 WHERE OBJECT_NAME = 'DBA_TABLES' ;
+----+-------------+--------------+------+---------------+---------------+---------+-------+------+-----------------------+
| id | select_type | table        | type | possible_keys | key           | key_len | ref   | rows | Extra                 |
+----+-------------+--------------+------+---------------+---------------+---------+-------+------+-----------------------+
|  1 | SIMPLE      | DAO_OBJECTS1 | ref  | IDX2_OBJECTS2 | IDX2_OBJECTS2 | 131     | const |    2 | Using index condition |
+----+-------------+--------------+------+---------------+---------------+---------+-------+------+-----------------------+
1 row in set (0.00 sec)


```

### rows

```sql

mysql> EXPLAIN SELECT COUNT(*) FROM DAO_OBJECTS1 T1 ,DAO_OBJECTS2 T2 WHERE T1.OBJECT_ID=T2.OBJECT_ID
    -> ;
+----+-------------+-------+--------+---------------+---------+---------+------------------+-------+-------------+
| id | select_type | table | type   | possible_keys | key     | key_len | ref              | rows  | Extra       |
+----+-------------+-------+--------+---------------+---------+---------+------------------+-------+-------------+
|  1 | SIMPLE      | T2    | index  | PRIMARY       | PRIMARY | 4       | NULL             | 50887 | Using index |
|  1 | SIMPLE      | T1    | eq_ref | PRIMARY       | PRIMARY | 4       | dao.T2.OBJECT_ID |     1 | Using index |
+----+-------------+-------+--------+---------------+---------+---------+------------------+-------+-------------+
2 rows in set (0.00 sec)


```

第一行 row = 50887 为T2表总大小 
第二行 row = 1     代表从T2表取一行 到T1表中遍历可获取的行数的大小 



### Extra

Extra 显示了一些不适合在其他列显示 但是很重要的信息

下面只列举了一些我遇见过的 后面有的再补充


####  Using where


相当于Oracle 谓词信息中的filter 先取出数据之后再比对

```sql

mysql> EXPLAIN 
    -> select * 
    ->   from DAO_OBJECTS1 T1 ,EMP E
    ->  WHERE E.DEPTNO= T1.OBJECT_ID ;
+----+-------------+-------+--------+---------------+---------+---------+--------------+------+-------------+
| id | select_type | table | type   | possible_keys | key     | key_len | ref          | rows | Extra       |
+----+-------------+-------+--------+---------------+---------+---------+--------------+------+-------------+
|  1 | SIMPLE      | E     | ALL    | FK_DEPTNO     | NULL    | NULL    | NULL         |   14 | Using where |
|  1 | SIMPLE      | T1    | eq_ref | PRIMARY       | PRIMARY | 4       | dao.E.DEPTNO |    1 | NULL        |
+----+-------------+-------+--------+---------------+---------+---------+--------------+------+-------------+
2 rows in set (0.00 sec)


```

####  Using index

使用索引来访问数据而避免回表

```sql


mysql> EXPLAIN SELECT COUNT(*) FROM DAO_OBJECTS1 T1 ,DAO_OBJECTS2 T2 WHERE T1.OBJECT_ID=T2.OBJECT_ID
    -> ;
+----+-------------+-------+--------+---------------+---------+---------+------------------+-------+-------------+
| id | select_type | table | type   | possible_keys | key     | key_len | ref              | rows  | Extra       |
+----+-------------+-------+--------+---------------+---------+---------+------------------+-------+-------------+
|  1 | SIMPLE      | T2    | index  | PRIMARY       | PRIMARY | 4       | NULL             | 50887 | Using index |
|  1 | SIMPLE      | T1    | eq_ref | PRIMARY       | PRIMARY | 4       | dao.T2.OBJECT_ID |     1 | Using index |
+----+-------------+-------+--------+---------------+---------+---------+------------------+-------+-------------+
2 rows in set (0.00 sec)


```

#### Using index conditio

使用索引过滤数据

```sql


mysql> EXPLAIN SELECT * FROM DAO_OBJECTS1 WHERE OBJECT_NAME = 'DBA_TABLES' ;
+----+-------------+--------------+------+---------------+---------------+---------+-------+------+-----------------------+
| id | select_type | table        | type | possible_keys | key           | key_len | ref   | rows | Extra                 |
+----+-------------+--------------+------+---------------+---------------+---------+-------+------+-----------------------+
|  1 | SIMPLE      | DAO_OBJECTS1 | ref  | IDX2_OBJECTS2 | IDX2_OBJECTS2 | 131     | const |    2 | Using index condition |
+----+-------------+--------------+------+---------------+---------------+---------+-------+------+-----------------------+
1 row in set (0.00 sec)

```


####  Using temporary

使用了临时表

```sql


mysql> explain select distinct object_type from DAO_OBJECTS ;
+----+-------------+-------------+------+---------------+------+---------+------+-------+-----------------+
| id | select_type | table       | type | possible_keys | key  | key_len | ref  | rows  | Extra           |
+----+-------------+-------------+------+---------------+------+---------+------+-------+-----------------+
|  1 | SIMPLE      | DAO_OBJECTS | ALL  | NULL          | NULL | NULL    | NULL | 50235 | Using temporary |
+----+-------------+-------------+------+---------------+------+---------+------+-------+-----------------+
1 row in set (0.00 sec)


```

####  Using filesort

无法利用索引来完成的排序

```sql


mysql> EXPLAIN select  * FROM DAO_OBJECTS1 ORDER BY OBJECT_TYPE ;
+----+-------------+--------------+------+---------------+------+---------+------+-------+----------------+
| id | select_type | table        | type | possible_keys | key  | key_len | ref  | rows  | Extra          |
+----+-------------+--------------+------+---------------+------+---------+------+-------+----------------+
|  1 | SIMPLE      | DAO_OBJECTS1 | ALL  | NULL          | NULL | NULL    | NULL | 52025 | Using filesort |
+----+-------------+--------------+------+---------------+------+---------+------+-------+----------------+
1 row in set (0.00 sec)


```

####  Using join buffer

将之前访问的数据存放于join buffer中

```sql


mysql> EXPLAIN EXTENDED
    -> SELECT *
    ->   FROM EMP
    ->  WHERE EMPNO IN (SELECT T1.OBJECT_ID
    ->                    FROM DAO_OBJECTS1 T1 ,DAO_OBJECTS1 T2 
    ->                   WHERE T1.OBJECT_ID =T2.OBJECT_ID
    ->                     AND EXISTS (SELECT  1
    ->                                  FROM DEPT D       
    ->                                 WHERE D.DEPTNO= T1.OBJECT_ID )
    ->                      );
+----+--------------------+-------------+--------+---------------+---------+---------+-----------------------+------+----------+----------------------------------------------------+
| id | select_type        | table       | type   | possible_keys | key     | key_len | ref                   | rows | filtered | Extra                                              |
+----+--------------------+-------------+--------+---------------+---------+---------+-----------------------+------+----------+----------------------------------------------------+
|  1 | PRIMARY            | <subquery2> | ALL    | NULL          | NULL    | NULL    | NULL                  | NULL |     0.00 | Using where                                        |
|  1 | PRIMARY            | EMP         | eq_ref | PRIMARY       | PRIMARY | 4       | <subquery2>.OBJECT_ID |    1 |   100.00 | NULL                                               |
|  2 | MATERIALIZED       | T1          | ALL    | NULL          | NULL    | NULL    | NULL                  |    1 |   100.00 | Using where                                        |
|  2 | MATERIALIZED       | T2          | ALL    | NULL          | NULL    | NULL    | NULL                  |    1 |   100.00 | Using where; Using join buffer (Block Nested Loop) |
|  3 | DEPENDENT SUBQUERY | D           | eq_ref | PRIMARY       | PRIMARY | 4       | dao.T1.OBJECT_ID      |    1 |   100.00 | Using index                                        |
+----+--------------------+-------------+--------+---------------+---------+---------+-----------------------+------+----------+----------------------------------------------------+
5 rows in set, 2 warnings (0.00 sec)


```

####  Impossible WHERE


 谓词不成立

```sql


mysql> EXPLAIN SELECT *  FROM DAO_OBJECTS1 T1  LIMIT 0 ;
+----+-------------+-------+------+---------------+------+---------+------+------+------------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra            |
+----+-------------+-------+------+---------------+------+---------+------+------+------------------+
|  1 | SIMPLE      | NULL  | NULL | NULL          | NULL | NULL    | NULL | NULL | Impossible WHERE |
+----+-------------+-------+------+---------------+------+---------+------+------+------------------+
1 row in set (0.00 sec)

mysql> EXPLAIN SELECT *  FROM DAO_OBJECTS1 T1 WHERE 1=2 ;
+----+-------------+-------+------+---------------+------+---------+------+------+------------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra            |
+----+-------------+-------+------+---------------+------+---------+------+------+------------------+
|  1 | SIMPLE      | NULL  | NULL | NULL          | NULL | NULL    | NULL | NULL | Impossible WHERE |
+----+-------------+-------+------+---------------+------+---------+------+------+------------------+
1 row in set (0.00 sec)


mysql>  EXPLAIN 
    ->   SELECT * 
    ->     FROM DAO_OBJECTS1 
    ->    WHERE OBJECT_NAME ='DBA_TABLES' 
    ->     and  OBJECT_ID='9527';
+----+-------------+-------+------+---------------+------+---------+------+------+-----------------------------------------------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra                                               |
+----+-------------+-------+------+---------------+------+---------+------+------+-----------------------------------------------------+
|  1 | SIMPLE      | NULL  | NULL | NULL          | NULL | NULL    | NULL | NULL | Impossible WHERE noticed after reading const tables |
+----+-------------+-------+------+---------------+------+---------+------+------+-----------------------------------------------------+
1 row in set (0.01 sec)

```