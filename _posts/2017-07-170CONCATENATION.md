---
layout: post
title:  执行计划-CONCATENATION释疑
date: 2017-07-17
categories: blog
tags: [优化]
description: 执行计划-CONCATENATION释疑
---

## 创建测试表与索引
```sql
SQL> set linesize 200
SQL> set pagesize 20000
SQL> set autotrace trace
SQL> create table dao_objects as select  * from dba_objects ;
SQL> create index idx_obj_id on dao_objects (object_id) ;  
create index  idx_obj_type on dao_objects (object_type) ;  
  
  
Index created.  
  
  
SQL>   
Index created.  
```

## 查看执行计划

```sql

SQL> create index idx_obj_id on dao_objects (object_id) ;  
create index  idx_obj_type on dao_objects (object_type) ;  
  
  
Index created.  
  
  
SQL>   
Index created.  
```

请注意ID为1的操作CONCATENATION.


CONCATENATION操作类似 union ,只不过与union 不同的是他并不对全部数据去重.
CONCATENATION操作只去除由OR引起的重复值.这点在我们做SQL改写的时候一定要特别注意.
因为empno为7934的记录 部门编号为20

```sql
SQL> select e.empno,e.deptno  
  2    from scott.emp e   
  3   where empno >7900 or deptno=10 ;  
  
  
     EMPNO     DEPTNO  
---------- ----------  
      7782         10  
      7839         10  
      7902         20  
      7934         10  
  
  
SQL> select e.empno,e.deptno  
  2    from scott.emp e   
  3   where empno >7900 ;  
  
  
     EMPNO     DEPTNO  
---------- ----------  
      7902         20  
      7934         10  
  
  
SQL> select e.empno,e.deptno  
  2    from scott.emp e   
  3   where deptno=10;  
  
  
     EMPNO     DEPTNO  
---------- ----------  
      7782         10  
      7839         10  
      7934         10  
```

