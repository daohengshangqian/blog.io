---
layout: post
title:    MySQL删除重复记录并保留一条
date: 2018-05-22
categories: blog
tags: [SQL,面试题]
description: MySQL删除重复记录并保留一条
---

今天有同学笔试遇到这样的问题,我就顺便总结下.

## 原数据是经典的emp表 


```sql

mysql> select * from emp1 ;
+-------+--------+-----------+------+---------------------+------+------+--------+
| empNO | ENAME  | JOB       | MGR  | HIREDATE            | SAL  | COMM | deptNO |
+-------+--------+-----------+------+---------------------+------+------+--------+
|  7369 | SMITH  | CLERK     | 7902 | 1980-12-17 00:00:00 |  800 | NULL |     20 |
|  7499 | ALLEN  | SALESMAN  | 7698 | 1981-02-20 00:00:00 | 1600 |  300 |     30 |
|  7521 | WARD   | SALESMAN  | 7698 | 1981-02-22 00:00:00 | 1250 |  500 |     30 |
|  7566 | JONES  | MANAGER   | 7839 | 1981-04-02 00:00:00 | 2975 | NULL |     20 |
|  7654 | MARTIN | SALESMAN  | 7698 | 1981-09-28 00:00:00 | 1250 | 1400 |     30 |
|  7698 | BLAKE  | MANAGER   | 7839 | 1981-05-01 00:00:00 | 2850 | NULL |     30 |
|  7782 | CLARK  | MANAGER   | 7839 | 1981-06-09 00:00:00 | 2450 | NULL |     10 |
|  7788 | SCOTT  | ANALYST   | 7566 | 1987-04-19 00:00:00 | 3000 | NULL |     20 |
|  7839 | KING   | PRESIDENT | NULL | 1981-11-17 00:00:00 | 5000 | NULL |     10 |
|  7844 | TURNER | SALESMAN  | 7698 | 1981-09-08 00:00:00 | 1500 |    0 |     30 |
|  7876 | ADAMS  | CLERK     | 7788 | 1987-05-23 00:00:00 | 1100 | NULL |     20 |
|  7900 | JAMES  | CLERK     | 7698 | 1981-12-03 00:00:00 |  950 | NULL |     30 |
|  7902 | FORD   | ANALYST   | 7566 | 1981-12-03 00:00:00 | 3000 | NULL |     20 |
|  7934 | MILLER | CLERK     | 7782 | 1982-01-23 00:00:00 | 1300 | NULL |     10 |
+-------+--------+-----------+------+---------------------+------+------+--------+
14 rows in set (0.00 sec)



```



## 要求:删除部门编号相同的记录,只保留员工编号最大的记录

### 查询下每个部门的最高雇员编号

```sql


mysql> select deptno,max(empno) from emp group by deptno ;
+--------+------------+
| deptno | max(empno) |
+--------+------------+
|     10 |       7934 |
|     20 |       7902 |
|     30 |       7900 |
+--------+------------+
3 rows in set (0.09 sec)


```

### 关联删除法


```sql


mysql> delete a from emp1  a ,emp1 b  where a.empno < b.empno   and a.deptno=b.deptno;
Query OK, 11 rows affected (0.01 sec)

mysql> select * from emp1 ;
+-------+--------+---------+------+---------------------+------+------+--------+
| empNO | ENAME  | JOB     | MGR  | HIREDATE            | SAL  | COMM | deptNO |
+-------+--------+---------+------+---------------------+------+------+--------+
|  7900 | JAMES  | CLERK   | 7698 | 1981-12-03 00:00:00 |  950 | NULL |     30 |
|  7902 | FORD   | ANALYST | 7566 | 1981-12-03 00:00:00 | 3000 | NULL |     20 |
|  7934 | MILLER | CLERK   | 7782 | 1982-01-23 00:00:00 | 1300 | NULL |     10 |
+-------+--------+---------+------+---------------------+------+------+--------+
3 rows in set (0.00 sec)



```

### 临时表删除法 


```sql

mysql> delete from emp1
    ->  where empno in(
    ->  select empno
    ->  from (select empno
    ->           from emp1
    ->          where deptno in (select deptno
    ->                             from emp1
    ->                            group by deptno
    ->                           having count(deptno) > 1)
    ->            and empno not in (select max(empno)
    ->                                 from emp1
    ->                                group by deptno
    ->                               having count(deptno) > 1))  tmptable  );
Query OK, 11 rows affected (0.02 sec)

mysql> select * from emp1 ;
+-------+--------+---------+------+---------------------+------+------+--------+
| empNO | ENAME  | JOB     | MGR  | HIREDATE            | SAL  | COMM | deptNO |
+-------+--------+---------+------+---------------------+------+------+--------+
|  7900 | JAMES  | CLERK   | 7698 | 1981-12-03 00:00:00 |  950 | NULL |     30 |
|  7902 | FORD   | ANALYST | 7566 | 1981-12-03 00:00:00 | 3000 | NULL |     20 |
|  7934 | MILLER | CLERK   | 7782 | 1982-01-23 00:00:00 | 1300 | NULL |     10 |
+-------+--------+---------+------+---------------------+------+------+--------+
3 rows in set (0.00 sec)




```