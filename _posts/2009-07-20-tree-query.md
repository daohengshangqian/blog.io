---
layout: post
title:  Oracle层次查询详解
date: 2009-07-20
categories: blog
tags: [SQL]
description: Oracle层次查询详解
---

## 树形结构的概念

树形结构指的是数据元素之间存在着“一对多”的树形关系的数据结构，是一类重要的非线性数据结构。
在树形结构中，树根结点没有前驱结点，其余每个结点有且只有一个前驱结点。叶子结点没有后续结点，其余每个结点的后续节点数可以是一个也可以是多个。

## 层次查询的概念

层次查询（Hierarchical Queries）就是通过一定的规则，使用树的遍历（tree walking）来获取树的层次关系数据的一种方法
查询树形结构数据的父子关系，通过以下两个因素操纵树形结构的遍历
1 遍历起始位置
2 遍历扫描的规则 自上而下(Root->Branch->Leaf)自下而上(Leaf->Branch->Root)
层次查询的语法
```sql
 
SELECT [level], column, expr... 
  FROM table | view
[WHERE condition]
 START WITH condition(s)
 CONNECT BY [prior column1= column2 | column1 = prior column2 ...];
```` 

## 相关术语说明

### FROM

数据来源那个表或者视图 

```sql
SQL> select sys_connect_by_path(ename, '/') col1,
  2         ename  ,
  3         sys_connect_by_path(dname, '/') col2
  4    from (select * from emp e, dept d where e.deptno = d.deptno)
  5  connect by prior mgr = empno ;

COL1                           ENAME                          COL2
------------------------------ ------------------------------ ------------------------------
/SMITH                         SMITH                          /RESEARCH
/SMITH/FORD                    FORD                           /RESEARCH/RESEARCH
/SMITH/FORD/JONES              JONES                          /RESEARCH/RESEARCH/RESEARCH
/SMITH/FORD/JONES/KING         KING                           /RESEARCH/RESEARCH/RESEARCH/AC
                                                              COUNTING

/ALLEN                         ALLEN                          /SALES
/ALLEN/BLAKE                   BLAKE                          /SALES/SALES
/ALLEN/BLAKE/KING              KING                           /SALES/SALES/ACCOUNTING
/WARD                          WARD                           /SALES
/WARD/BLAKE                    BLAKE                          /SALES/SALES
/WARD/BLAKE/KING               KING                           /SALES/SALES/ACCOUNTING
/JONES                         JONES                          /RESEARCH
/JONES/KING                    KING                           /RESEARCH/ACCOUNTING
/MARTIN                        MARTIN                         /SALES
/MARTIN/BLAKE                  BLAKE                          /SALES/SALES
/MARTIN/BLAKE/KING             KING                           /SALES/SALES/ACCOUNTING
/BLAKE                         BLAKE                          /SALES
/BLAKE/KING                    KING                           /SALES/ACCOUNTING
/CLARK                         CLARK                          /ACCOUNTING
/CLARK/KING                    KING                           /ACCOUNTING/ACCOUNTING
/SCOTT                         SCOTT                          /RESEARCH
/SCOTT/JONES                   JONES                          /RESEARCH/RESEARCH
/SCOTT/JONES/KING              KING                           /RESEARCH/RESEARCH/ACCOUNTING
/KING                          KING                           /ACCOUNTING
/TURNER                        TURNER                         /SALES
/TURNER/BLAKE                  BLAKE                          /SALES/SALES
/TURNER/BLAKE/KING             KING                           /SALES/SALES/ACCOUNTING
/ADAMS                         ADAMS                          /RESEARCH
/ADAMS/SCOTT                   SCOTT                          /RESEARCH/RESEARCH
/ADAMS/SCOTT/JONES             JONES                          /RESEARCH/RESEARCH/RESEARCH
/ADAMS/SCOTT/JONES/KING        KING                           /RESEARCH/RESEARCH/RESEARCH/AC
                                                              COUNTING

/JAMES                         JAMES                          /SALES
/JAMES/BLAKE                   BLAKE                          /SALES/SALES
/JAMES/BLAKE/KING              KING                           /SALES/SALES/ACCOUNTING
/FORD                          FORD                           /RESEARCH
/FORD/JONES                    JONES                          /RESEARCH/RESEARCH
/FORD/JONES/KING               KING                           /RESEARCH/RESEARCH/ACCOUNTING
/MILLER                        MILLER                         /ACCOUNTING
/MILLER/CLARK                  CLARK                          /ACCOUNTING/ACCOUNTING
/MILLER/CLARK/KING             KING                           /ACCOUNTING/ACCOUNTING/ACCOUNT
                                                              ING


39 rows selected.

```

### Start with子句
Start with 指定查询开始的根节点(一行或多行)，所有满足条件的都将被当成是根节点如果不给定START WITH子句，那么所有的行都会被当成是根节点，然后查找其后代，通常这不是我们想要的结果.

```sql

SQL> SELECT ename, SYS_CONNECT_BY_PATH(ename, '/') col2
  2    FROm scott.emp
  3  CONNECT BY PRIOR empno = mgr
  4   order siblings by ename ;

ENAME  COL2
------ ------------------------------
FORD   /FORD
SMITH  /FORD/SMITH
SCOTT  /SCOTT
ADAMS  /SCOTT/ADAMS
ALLEN  /ALLEN
JAMES  /JAMES
MARTIN /MARTIN
TURNER /TURNER
WARD   /WARD
MILLER /MILLER
ADAMS  /ADAMS
BLAKE  /BLAKE
ALLEN  /BLAKE/ALLEN
JAMES  /BLAKE/JAMES
MARTIN /BLAKE/MARTIN
TURNER /BLAKE/TURNER
WARD   /BLAKE/WARD
CLARK  /CLARK
MILLER /CLARK/MILLER
JONES  /JONES
FORD   /JONES/FORD
SMITH  /JONES/FORD/SMITH
SCOTT  /JONES/SCOTT
ADAMS  /JONES/SCOTT/ADAMS
SMITH  /SMITH
KING   /KING
BLAKE  /KING/BLAKE
ALLEN  /KING/BLAKE/ALLEN
JAMES  /KING/BLAKE/JAMES
MARTIN /KING/BLAKE/MARTIN
TURNER /KING/BLAKE/TURNER
WARD   /KING/BLAKE/WARD
CLARK  /KING/CLARK
MILLER /KING/CLARK/MILLER
JONES  /KING/JONES
FORD   /KING/JONES/FORD
SMITH  /KING/JONES/FORD/SMITH
SCOTT  /KING/JONES/SCOTT
ADAMS  /KING/JONES/SCOTT/ADAMS

39 rows selected.

```

### Connect by子句
Connect by与prior确定一个层次查询的条件和遍历的方向(prior确定)

Connect by prior column_1=column_2

CONNECT BY指定层次数据中父行和子行之间的关系，这里，关系被表示成一个表达式。当前行的字段会
和对应的父行的某个字段进行比较。 CONNECT BY 后的条件必须跟着一个PRIOR操作符
PRIOR是Oracle的一个内建操作符，只用于层次查询。当我们在层次查询的CONNECT BY条件中使用了PRIOR操作符时，位于其后的表达式被当成是当前行的父行进行比较

```sql

SQL> SELECT ename, SYS_CONNECT_BY_PATH(ename, '/') col2
  2    FROm scott.emp
  3  CONNECT BY PRIOR empno = mgr and empno in (SELECT EMPNO FROM  EMP ) ;

ENAME  COL2
------ ------------------------------
SCOTT  /SCOTT
ADAMS  /SCOTT/ADAMS
FORD   /FORD
SMITH  /FORD/SMITH
ALLEN  /ALLEN
JAMES  /JAMES
TURNER /TURNER
MARTIN /MARTIN
WARD   /WARD
MILLER /MILLER
ADAMS  /ADAMS
JONES  /JONES
SCOTT  /JONES/SCOTT
ADAMS  /JONES/SCOTT/ADAMS
FORD   /JONES/FORD
SMITH  /JONES/FORD/SMITH
CLARK  /CLARK
MILLER /CLARK/MILLER
BLAKE  /BLAKE
ALLEN  /BLAKE/ALLEN
JAMES  /BLAKE/JAMES
TURNER /BLAKE/TURNER
MARTIN /BLAKE/MARTIN
WARD   /BLAKE/WARD
SMITH  /SMITH
KING   /KING
JONES  /KING/JONES
SCOTT  /KING/JONES/SCOTT
ADAMS  /KING/JONES/SCOTT/ADAMS
FORD   /KING/JONES/FORD
SMITH  /KING/JONES/FORD/SMITH
CLARK  /KING/CLARK
MILLER /KING/CLARK/MILLER
BLAKE  /KING/BLAKE
ALLEN  /KING/BLAKE/ALLEN
JAMES  /KING/BLAKE/JAMES
TURNER /KING/BLAKE/TURNER
MARTIN /KING/BLAKE/MARTIN
WARD   /KING/BLAKE/WARD

39 rows selected.

```

注意:如果connect by 语句中没有prior关键字,则不会进行深层次的递归查询 

```sql

SQL> select * from emp 
  2  start with mgr is null
  3   connect by  empno = mgr
  4  ;

     EMPNO ENAME                JOB              MGR HIREDATE         SAL       COMM     DEPTNO
---------- -------------------- --------- ---------- --------- ---------- ---------- ----------
      7839 KING                 PRESIDENT            17-NOV-81       5000                    10
      
      
```


## 层次查询举例
自上向下的查询

```sql

 SELECT ename 
   FROm scott.emp
   start with ENAME='JONES'
    CONNECT BY PRIOR empno = mgr ;

ENAME
------
JONES
FORD
SMITH
SCOTT

111
解析:
起始条件为 start with ENAME='JONES'
递归条件为 CONNECT BY PRIOR empno = mgr 
先根据起始条件查找JONES的雇员编号 7566  (步骤1)
根据递归条件父行的雇员编号(7566)=子行的管理者编号(查找JONES的直接下属)(步骤2)
此行为类似于:

```sql
SQL> SELECT * 
  2  FROM EMP 
  3  WHERE MGR= (select EMPNO from scott.emp where ename='JONES' );

     EMPNO ENAME  JOB              MGR HIREDATE         SAL       COMM     DEPTNO
---------- ------ --------- ---------- --------- ---------- ---------- ----------
      7788 SCOTT  ANALYST         7566 19-APR-87       3000                    20
      7902 FORD   ANALYST         7566 03-DEC-81       3000                    20

```

这样就找到了JONES的直接下级

接下来根据JONES的直接下级查找下一级  (步骤3)

其结果类似于

```sql

SQL> SELECT  *
  2  FROM EMP
  3  WHERE EMPNO IN 
  4  (SELECT EMPNO 
  5  FROM EMP 
  6  WHERE MGR= (select EMPNO from scott.emp where ename='JONES' )) ;

     EMPNO ENAME  JOB              MGR HIREDATE         SAL       COMM     DEPTNO
---------- ------ --------- ---------- --------- ---------- ---------- ----------
      7788 SCOTT  ANALYST         7566 19-APR-87       3000                    20
      7902 FORD   ANALYST         7566 03-DEC-81       3000                    20

```

重复操作(2)(3)

直到查找到的行没有子行为止 

自下而上的查询

```sql

SQL> col conn_path for a30
SQL> select ename, sys_connect_by_path(ename, '->') conn_path 
  2    from emp
  3   start with ename = 'MILLER'
  4  connect by prior mgr = empno; 

ENAME                CONN_PATH
-------------------- ------------------------------
MILLER               ->MILLER
CLARK                ->MILLER->CLARK
KING                 ->MILLER->CLARK->KING

```
解析:
(1) 根据ename 查找到到MGR

```sql

SQL> select  * 
  2  from emp 
  3  where ename='MILLER';

     EMPNO ENAME                JOB              MGR HIREDATE         SAL       COMM     DEPTNO
---------- -------------------- --------- ---------- --------- ---------- ---------- ----------
      7934 MILLER               CLERK           7782 23-JAN-82       1300                    10

```

根据MGR查找直接上级 步骤(2)
```sql
SQL> select *
  2    from emp
  3   where empno = (select mgr from emp where ename = 'MILLER');

     EMPNO ENAME                JOB              MGR HIREDATE         SAL       COMM     DEPTNO
---------- -------------------- --------- ---------- --------- ---------- ---------- ----------
      7782 CLARK                MANAGER         7839 09-JUN-81       2450                    10
 ```
 
 根据查找到的上级的MGR(7839) 查找上级的上级 (步骤3)
 
 ```sql
 
SQL> select * 
  2  from emp 
  3  where empno =
  4  (select mgr
  5    from emp
  6   where empno = (select mgr from emp where ename = 'MILLER'));

     EMPNO ENAME                JOB              MGR HIREDATE         SAL       COMM     DEPTNO
---------- -------------------- --------- ---------- --------- ---------- ---------- ----------
      7839 KING                 PRESIDENT            17-NOV-81       5000                    10
      
```      
重复执行(2)(3)直到查询到某一行mgr为空为止
层级伪列LEVEL
伪列level 用以显示某行数据经由几次递归得来.

```sql
SQL>   select level,e.* 
  2      from emp  e
  3     start with mgr is null
  4   connect by  prior empno = mgr ;

     LEVEL      EMPNO ENAME                JOB              MGR HIREDATE         SAL       COMM     DEPTNO
---------- ---------- -------------------- --------- ---------- --------- ---------- ---------- ----------
         1       7839 KING                 PRESIDENT            17-NOV-81       5000                    10
         2       7566 JONES                MANAGER         7839 02-APR-81       2975                    20
         3       7788 SCOTT                ANALYST         7566 19-APR-87       3000                    20
         4       7876 ADAMS                CLERK           7788 23-MAY-87       1100                    20
         3       7902 FORD                 ANALYST         7566 03-DEC-81       3000                    20
         4       7369 SMITH                CLERK           7902 17-DEC-80        800                    20
         2       7698 BLAKE                MANAGER         7839 01-MAY-81       2850                    30
         3       7499 ALLEN                SALESMAN        7698 20-FEB-81       1600        300         30
         3       7521 WARD                 SALESMAN        7698 22-FEB-81       1250        500         30
         3       7654 MARTIN               SALESMAN        7698 28-SEP-81       1250       1400         30
         3       7844 TURNER               SALESMAN        7698 08-SEP-81       1500          0         30
         3       7900 JAMES                CLERK           7698 03-DEC-81        950                    30
         2       7782 CLARK                MANAGER         7839 09-JUN-81       2450                    10
         3       7934 MILLER               CLERK           7782 23-JAN-82       1300                    10

14 rows selected.
```
显示整个数的层级

```sql

SQL>   select  max(level)
  2      from emp  e
  3     start with mgr is null
  4   connect by  prior empno = mgr ;

MAX(LEVEL)
----------
         4
```        

查询各个层级职员的数量

```sql

SQL>   select  level,count(*)
  2      from emp  e
  3     start with mgr is null
  4   connect by  prior empno = mgr
  5   group by level ;

     LEVEL   COUNT(*)
---------- ----------
         1          1
         2          3
         4          2
         3          8
```

使用 WHERE字句过滤数据
查询级别最低的职员信息

```sql

SQL> select *
  2    from (select level lv, e.*
  3            from emp e
  4           start with mgr is null
  5          connect by prior empno = mgr)
  6   where lv = (select max(level)
  7                 from emp e
  8                start with mgr is null
  9               connect by prior empno = mgr);

        LV      EMPNO ENAME                JOB              MGR HIREDATE         SAL       COMM     DEPTNO
---------- ---------- -------------------- --------- ---------- --------- ---------- ---------- ----------
         4       7876 ADAMS                CLERK           7788 23-MAY-87       1100                    20
         4       7369 SMITH                CLERK           7902 17-DEC-80        800                    20
```         
level配合lpad 生成缩进

```sql

SQL> select level, empno, lpad('  ', 2 * (level - 1)) || ename ename
  2    from emp
  3   start with mgr is null
  4  connect by prior empno = mgr;

     LEVEL      EMPNO ENAME
---------- ---------- --------------------
         1       7839 KING
         2       7566   JONES
         3       7788     SCOTT
         4       7876       ADAMS
         3       7902     FORD
         4       7369       SMITH
         2       7698   BLAKE
         3       7499     ALLEN
         3       7521     WARD
         3       7654     MARTIN
         3       7844     TURNER
         3       7900     JAMES
         2       7782   CLARK
         3       7934     MILLER

14 rows selected.
```

判断节点之间是否具有层次关系

```sql 
SQL> select sum(cnt)
  2    from (select count(*) cnt
  3            from emp
  4           where ename = 'JONES'
  5           start with ename = 'ADAMS'
  6          connect by prior mgr = empno
  7          union all
  8          select count(*)
  9            from emp
 10           where ename = 'JONES'
 11           start with ename = 'ADAMS'
 12          connect by prior empno = mgr);

  SUM(CNT)
----------
         1
```    
     
将层次查询作为子查询使用
```sql

SQL> select * 
  2  from emp ;

     EMPNO ENAME                JOB              MGR HIREDATE         SAL       COMM     DEPTNO
---------- -------------------- --------- ---------- --------- ---------- ---------- ----------
      7369 SMITH                CLERK           7902 17-DEC-80        800                    20
      7499 ALLEN                SALESMAN        7698 20-FEB-81       1600        300         30
      7521 WARD                 SALESMAN        7698 22-FEB-81       1250        500         30
      7566 JONES                MANAGER         7839 02-APR-81       2975                    20
      7654 MARTIN               SALESMAN        7698 28-SEP-81       1250       1400         30
      7698 BLAKE                MANAGER         7839 01-MAY-81       2850                    30
      7782 CLARK                MANAGER         7839 09-JUN-81       2450                    10
      7788 SCOTT                ANALYST         7566 19-APR-87       3000                    20
      7839 KING                 PRESIDENT            17-NOV-81       5000                    10
      7844 TURNER               SALESMAN        7698 08-SEP-81       1500          0         30
      7876 ADAMS                CLERK           7788 23-MAY-87       1100                    20
      7900 JAMES                CLERK           7698 03-DEC-81        950                    30
      7902 FORD                 ANALYST         7566 03-DEC-81       3000                    20
      7934 MILLER               CLERK           7782 23-JAN-82       1300                    10

14 rows selected.

SQL> update emp e
  2     set e.sal = sal + 10000
  3   where empno in (select empno
  4                     from emp
  5                    start with ename = 'JONES'
  6                   connect by prior empno = mgr);

5 rows updated.


SQL>                  
SQL> select * 
  2  from emp ;      

     EMPNO ENAME                JOB              MGR HIREDATE         SAL       COMM     DEPTNO
---------- -------------------- --------- ---------- --------- ---------- ---------- ----------
      7369 SMITH                CLERK           7902 17-DEC-80      10800                    20
      7499 ALLEN                SALESMAN        7698 20-FEB-81       1600        300         30
      7521 WARD                 SALESMAN        7698 22-FEB-81       1250        500         30
      7566 JONES                MANAGER         7839 02-APR-81      12975                    20
      7654 MARTIN               SALESMAN        7698 28-SEP-81       1250       1400         30
      7698 BLAKE                MANAGER         7839 01-MAY-81       2850                    30
      7782 CLARK                MANAGER         7839 09-JUN-81       2450                    10
      7788 SCOTT                ANALYST         7566 19-APR-87      13000                    20
      7839 KING                 PRESIDENT            17-NOV-81       5000                    10
      7844 TURNER               SALESMAN        7698 08-SEP-81       1500          0         30
      7876 ADAMS                CLERK           7788 23-MAY-87      11100                    20
      7900 JAMES                CLERK           7698 03-DEC-81        950                    30
      7902 FORD                 ANALYST         7566 03-DEC-81      13000                    20
      7934 MILLER               CLERK           7782 23-JAN-82       1300                    10

14 rows selected. 
```


层次查询其他特性
SYS_CONNECT_BY_PATH
sys_connect_by_path(column,char)
其中，column是字符型或能自动转换成字符型的列名。char是用来连接路径的字符串。它的主要目的就是将父节点到当前节点的路径按照指定的模式展现出来 该函数只能使用在层次查询中

```sql

SQL> select ENAME,SYS_CONNECT_BY_PATH(ENAME,'->') CONN_PATH
  2    from emp
  3   start with ename = 'KING' 
  4  connect by prior empno = mgr ;

ENAME                CONN_PATH
-------------------- ------------------------------
KING                 ->KING
JONES                ->KING->JONES
SCOTT                ->KING->JONES->SCOTT
ADAMS                ->KING->JONES->SCOTT->ADAMS
FORD                 ->KING->JONES->FORD
SMITH                ->KING->JONES->FORD->SMITH
BLAKE                ->KING->BLAKE
ALLEN                ->KING->BLAKE->ALLEN
WARD                 ->KING->BLAKE->WARD
MARTIN               ->KING->BLAKE->MARTIN
TURNER               ->KING->BLAKE->TURNER
JAMES                ->KING->BLAKE->JAMES
CLARK                ->KING->CLARK
MILLER               ->KING->CLARK->MILLER

14 rows selected.
 
```

### CONNECT_BY_ISLEAF

CONNECT_BY_ISLEAF伪列
用于判断层次查询结果集中的行是不是叶子节点。返回值0表示不是叶子节点，1表示是叶子节点

```sql

SQL> select ENAME,SYS_CONNECT_BY_PATH(ENAME,'->') CONN_PATH ,CONNECT_BY_ISLEAF 
  2    from emp
  3   start with ename = 'KING' 
  4  connect by prior empno = mgr ;

ENAME                CONN_PATH                      CONNECT_BY_ISLEAF
-------------------- ------------------------------ -----------------
KING                 ->KING                                         0
JONES                ->KING->JONES                                  0
SCOTT                ->KING->JONES->SCOTT                           0
ADAMS                ->KING->JONES->SCOTT->ADAMS                    1
FORD                 ->KING->JONES->FORD                            0
SMITH                ->KING->JONES->FORD->SMITH                     1
BLAKE                ->KING->BLAKE                                  0
ALLEN                ->KING->BLAKE->ALLEN                           1
WARD                 ->KING->BLAKE->WARD                            1
MARTIN               ->KING->BLAKE->MARTIN                          1
TURNER               ->KING->BLAKE->TURNER                          1
JAMES                ->KING->BLAKE->JAMES                           1
CLARK                ->KING->CLARK                                  0
MILLER               ->KING->CLARK->MILLER                          1

14 rows selected.
```

### CONNECT_BY_ROOT

如果查询为多颗树,显示每棵树的跟节点
```sql
SQL> select ename, CONNECT_BY_ROOT ename
  2    from emp e
  3   start with mgr = 7839
  4  connect by prior empno = mgr ;

ENAME                CONNECT_BY
-------------------- ----------
JONES                JONES
SCOTT                JONES
ADAMS                JONES
FORD                 JONES
SMITH                JONES
BLAKE                BLAKE
ALLEN                BLAKE
WARD                 BLAKE
MARTIN               BLAKE
TURNER               BLAKE
JAMES                BLAKE
CLARK                CLARK
MILLER               CLARK

13 rows selected.

```
SIBLINGS
使用SIBLINGS关键字排序
对于层次查询的结果集，如果使用order by排序则会严格按照列的值进行排序,这可能会破坏层级表示
为了解决这个问题Oracle 使用SIBLINGS关键字对层次查询的结果集进行排序
```sql

SQL> select level, lpad(' ',2*level-1,' ')||ename conn_path ,ename
  2    from emp e
  3   start with mgr is null 
  4  connect by prior empno = mgr 
  5  order by ename ;

     LEVEL CONN_PATH                      ENAME
---------- ------------------------------ --------------------
         4        ADAMS                   ADAMS
         3      ALLEN                     ALLEN
         2    BLAKE                       BLAKE
         2    CLARK                       CLARK
         3      FORD                      FORD
         3      JAMES                     JAMES
         2    JONES                       JONES
         1  KING                          KING
         3      MARTIN                    MARTIN
         3      MILLER                    MILLER
         3      SCOTT                     SCOTT
         4        SMITH                   SMITH
         3      TURNER                    TURNER
         3      WARD                      WARD

14 rows selected.

SQL> select level, lpad(' ',2*level-1,' ')||ename conn_path ,ename
  2    from emp e
  3   start with mgr is null 
  4  connect by prior empno = mgr 
  5  order SIBLINGS by ename ;

     LEVEL CONN_PATH                      ENAME
---------- ------------------------------ --------------------
         1  KING                          KING
         2    BLAKE                       BLAKE
         3      ALLEN                     ALLEN
         3      JAMES                     JAMES
         3      MARTIN                    MARTIN
         3      TURNER                    TURNER
         3      WARD                      WARD
         2    CLARK                       CLARK
         3      MILLER                    MILLER
         2    JONES                       JONES
         3      FORD                      FORD
         4        SMITH                   SMITH
         3      SCOTT                     SCOTT
         4        ADAMS                   ADAMS

14 rows selected. 

```
CONNECT_BY的死循环
有数据如下

```sql

SQL> WITH LOOP_DATA
  2  AS 
  3  (
  4  SELECT 1 node,4 parent_node FROM DUAL
  5  UNION ALL 
  6  SELECT 2 node,1 parent_node FROM DUAL
  7  UNION ALL 
  8  SELECT 3 node,2 parent_node FROM DUAL
  9  UNION ALL 
 10  SELECT 4 node,3 parent_node FROM DUAL
 11  )
 12  select * from LOOP_DATA;

      NODE PARENT_NODE
---------- -----------
         1           4
         2           1
         3           2
         4           3
```

现在使用节点1作为根节点查找其下属节点将会出发一个死循环
因为节点4既是1的下属节点又是1的上级节点 从而引发ora-1436错误

```sql

SQL> WITH LOOP_DATA
  2  AS 
  3  (
  4  SELECT 1 node,4 parent_node FROM DUAL
  5  UNION ALL 
  6  SELECT 2 node,1 parent_node FROM DUAL
  7  UNION ALL 
  8  SELECT 3 node,2 parent_node FROM DUAL
  9  UNION ALL 
 10  SELECT 4 node,3 parent_node FROM DUAL
 11  )
 12  select * 
 13   from LOOP_DATA
 14  start with node =1 
 15  connect by prior node  = parent_node ;
ERROR:
ORA-01436: CONNECT BY loop in user data



no rows selected

```

为解决此问题可以使用 nocycle 关键字 来跳过循环
并使用CONNECT_BY_ISCYCLE标识出出现循环的行 
```sql

SQL> WITH LOOP_DATA
  2  AS 
  3  (
  4  SELECT 1 node,4 parent_node FROM DUAL
  5  UNION ALL 
  6  SELECT 2 node,1 parent_node FROM DUAL
  7  UNION ALL 
  8  SELECT 3 node,2 parent_node FROM DUAL
  9  UNION ALL 
 10  SELECT 4 node,3 parent_node FROM DUAL
 11  )
 12  select x.*,CONNECT_BY_ISCYCLE
 13   from LOOP_DATA x
 14  start with node =1 
 15  connect   by nocycle prior node  = parent_node ;

      NODE PARENT_NODE CONNECT_BY_ISCYCLE
---------- ----------- ------------------
         1           4                  0
         2           1                  0
         3           2                  0
         4           3                  1
```
         
## CONNECT BY的特殊用法
使用CONNECY BY 来生成数列

```sql

SQL> select level
  2    from dual 
  3  connect by level <= 32;

     LEVEL
----------
         1
         2
         3
         4
         5
         6
         7
         8
         9
        10
        11
        12
        13
        14
        15
        16
        17
        18
        19
        20
        21
        22
        23
        24
        25
        26
        27
        28
        29
        30
        31
        32

32 rows selected.

```
