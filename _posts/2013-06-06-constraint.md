---
layout: post
title: Oracle约束详解
date: 2013-06-06
categories: blog
tags: [SQL]
description:  Oracle约束详解
---
# 约束的分类

## 非空约束

```sql

SQL> create table bsr_not_Null (c1 number not null,c2 number );

Table created

SQL> insert into bsr_not_Null values (1,1) ;

1 row inserted

SQL> insert into bsr_not_Null values (null,1) ;

insert into bsr_not_Null values (null,1)

ORA-01400: cannot insert NULL into ("DAO"."BSR_NOT_NULL"."C1")

SQL> insert into bsr_not_Null values (1,null) ;

1 row inserted

SQL> commit ;

Commit complete

SQL> select * from bsr_not_Null ;

        C1         C2
---------- ----------
         1          1
         1 

```


## 唯一约束

```sql

SQL> create table bsr_unique (c1 number unique ,c2 number) ;

Table created

SQL> insert into bsr_unique values (1,1) ;

1 row inserted


SQL> insert into bsr_unique values (2,1) ;

1 row inserted

SQL> insert into bsr_unique values (1,2) ;

insert into bsr_unique values (1,2)

ORA-00001: unique constraint (DAO.SYS_C0013390) violated

SQL> insert into bsr_unique values (null,3 ) ;

1 row inserted

SQL>  insert into bsr_unique values (null,3 ) ;

1 row inserted

SQL> commit ;

Commit complete

SQL> select * from bsr_unique ;

        C1         C2
---------- ----------
         1          1
         2          1
                    3
                    3

结论:唯一约束对空值不起作用

```
## 主键约束

```sql
SQL> create table bsr_primary (c1 number primary key, c2 number) ;

Table created

--建立索引后,相关的列就在
SQL> desc bsr_primary;
Name Type   Nullable Default Comments 
---- ------ -------- ------- -------- 
C1   NUMBER                           
C2   NUMBER Y      


```

## 外键约束

```sql

SQL> create table emp as select * from scott.emp ;

Table created

SQL> create table dept as select * from scott.dept ;

Table created


SQL> select * from  dept ;

DEPTNO DNAME          LOC
------ -------------- -------------
    10 ACCOUNTING     NEW YORK
    20 RESEARCH       DALLAS
    30 SALES          CHICAGO
    40 OPERATIONS     BOSTON

SQL> select  ename , deptno from emp ;

ENAME      DEPTNO
---------- ------
SMITH          20
ALLEN          30
WARD           30
JONES          20
MARTIN         30
BLAKE          30
CLARK          10
SCOTT          20
KING           10
TURNER         30
ADAMS          20
JAMES          30
FORD           20
MILLER         10

```

现在我们来添加外键 
注意会报错

```
SQL> alter table emp add constraint fk_deptno foreign key (deptno) references dept(deptno) ;

alter table emp add constraint fk_deptno foreign key (deptno) references dept(deptno)

ORA-02270: no matching unique or primary key for this column-list

```
我们把被引用的表称之为父表
引用其他表的表称之为子表

那么这个报错的原因是父表对应列(dept表的deptno)没有主键或者唯一键

```sql

SQL> alter table dept add constraint pk_dept primary key (deptno) ;

Table altered

SQL> alter table emp add constraint fk_deptno foreign key (deptno) references dept(deptno) ;

Table altered

SQL> insert into emp (ename,deptno) values ('dao',50) ;

insert into emp (ename,deptno) values ('dao',50)

ORA-02291: integrity constraint (DAO.FK_DEPTNO) violated - parent key not found

```

报错的原因为:在向EMP表中添加记录时,会检查父表列(dept.deptno)有没有50的值



```sql
SQL> delete from dept where deptno=40 ;

1 row deleted

SQL> delete from dept where deptno=30 ;

delete from dept where deptno=30

ORA-02292: integrity constraint (DAO.FK_DEPTNO) violated - child record found

SQL> rollback ;

Rollback complete


```
报错的原因为:在删除DEPT表deptno列值为30的记录时,发现子表列(emp.dept)存在数据.




### 外键约束的删除属性

#### NO ACTION
删除父表数据时,发现子表存在对应记录,则报错停止删除

```sql
SQL> delete from dept where deptno=30 ;

delete from dept where deptno=30

ORA-02292: integrity constraint (DAO.FK_DEPTNO) violated - child record found

SQL> rollback ;

Rollback complete


```

#### ON DELETE CASCADE 
删除父表数据时,发现子表存在对应记录,则将子表记录的引用列设置为NULL

```sql

SQL> alter table emp drop  constraint fk_deptno ;

Table altered

SQL> alter table emp add constraint fk_deptno foreign key (deptno) references dept(deptno) on delete  set null ;

Table altered

SQL> select ename,deptno from emp  order by deptno ;

ENAME      DEPTNO
---------- ------
CLARK          10
KING           10
MILLER         10
JONES          20
FORD           20
ADAMS          20
SMITH          20
SCOTT          20
WARD           30
TURNER         30
ALLEN          30
JAMES          30
BLAKE          30
MARTIN         30

14 rows selected

SQL> delete from  dept where deptno=30 ;

1 row deleted

SQL>  select ename,deptno from emp  order by deptno ;

ENAME      DEPTNO
---------- ------
CLARK          10
KING           10
MILLER         10
JONES          20
FORD           20
ADAMS          20
SMITH          20
SCOTT          20
WARD       
TURNER     
ALLEN      
JAMES      
BLAKE      
MARTIN     

14 rows selected

SQL> rollback ;

Rollback complete

```


#### ON DELETE SET NULL

删除附表记录同时删除子表对应记录

```sql

SQL> alter table emp drop  constraint fk_deptno ;

Table altered

SQL> alter table emp add constraint fk_deptno foreign key (deptno) references dept(deptno) on delete cascade ;

Table altered

SQL> select ename,deptno from emp  order by deptno ;

ENAME      DEPTNO
---------- ------
CLARK          10
KING           10
MILLER         10
JONES          20
FORD           20
ADAMS          20
SMITH          20
SCOTT          20
WARD           30
TURNER         30
ALLEN          30
JAMES          30
BLAKE          30
MARTIN         30

14 rows selected

SQL> delete from  dept where deptno=30 ;

1 row deleted

SQL>  select ename,deptno from emp  order by deptno ;

ENAME      DEPTNO
---------- ------
MILLER         10
CLARK          10
KING           10
FORD           20
ADAMS          20
JONES          20
SMITH          20
SCOTT          20

8 rows selected

SQL> rollback ;

Rollback complete

```


## 检查约束

```sql

SQL> alter table emp add constraint check_sal_0 check (sal>0) ;

Table altered

SQL> update emp set sal=-1 where rownum =1 ;

update emp set sal=-1 where rownum =1

ORA-02290: check constraint (DAO.CHECK_SAL_0) violated


```

# 约束的创建方式



## 建表未命名
```sql
SQL> create table bsr_1 (c1  number not null, c2 number ) ;

Table created

```

## 建表命名
```sql

SQL> create table bsr_2 (c1  number not null, c2 number ,constraint pk_bsr_2 primary key (c2)) ;

Table created

 
```
## 后添加

```sql

SQL> create table bsr_3 (c1 number ,c2 number ) ;

Table created

SQL> alter table bsr_3 add constraint   pk_bsr_3 primary key (c2) ;

Table altered


```

# 约束的查看

## 查看表上所拥有的约束

```sql

SQL> select owner,
  2         CONSTRAINT_NAME,
  3         CONSTRAINT_TYPE,
  4         TABLE_NAME,
  5         SEARCH_CONDITION,
  6         R_OWNER,
  7         R_CONSTRAINT_NAME
  8    from dba_constraints
  9   where owner = 'DAO'
 10    and (table_name like 'BSR%' or TABLE_NAME IN ('EMP','DEPT')) ;

OWNER    CONSTRAINT_NAME   CONSTRAINT_TYPE TABLE_NAME     SEARCH_CONDITION   R_OWNER   R_CONSTRAINT_NAME
-------- ----------------- --------------- -------------- ------------------ --------- -----------------
DAO      PK_BSR_3          P               BSR_3                                                        
DAO      PK_BSR_2          P               BSR_2                                                        
DAO      SYS_C0013399      C               BSR_2          "C1" IS NOT NULL                              
DAO      CHECK_SAL_0       C               EMP            sal>0                                         
DAO      PK_DEPT           P               DEPT                                                         
DAO      SYS_C0013390      U               BSR_UNIQUE                                                   
DAO      SYS_C0013391      P               BSR_PRIMARY                                                  
DAO      SYS_C0013389      C               BSR_NOT_NULL   "C1" IS NOT NULL                              
DAO      FK_DEPTNO         R               EMP                               DAO       PK_DEPT          
DAO      SYS_C0013398      C               BSR_1          "C1" IS NOT NULL                              
                                                                                                        
10 rows selected                                                                                        


```







## 查看哪些表外键列没有索引

```sql
SQL> select ucc.owner,ucc.column_name,ucc.table_name,ucc.column_name                                                                                                                                                           
  2   from user_cons_columns  ucc ,                                                                                                                                                                                            
  3        user_constraints  uc                                                                                                                                                                                                
  4   where  ucc.owner=uc.owner                                                                                                                                                                                                
  5    and ucc.table_name=uc.table_name                                                                                                                                                                                        
  6    and ucc.constraint_name=uc.constraint_name                                                                                                                                                                              
  7    and uc.constraint_type ='R'                                                                                                                                                                                             
  8    AND NOT EXISTS (select 1 from user_ind_columns uic where uic.column_position=1                                                                                                                                          
  9                      and uic.table_name=uc.table_name                                                                                                                                                                      
 10                      and uic.column_name=ucc.column_name );                                                                                                                                                                
                                                                                                                                                                     
OWNER                          COLUMN_NAME             TABLE_NAME                     COLUMN_NAME            
------------------------------ ----------------------- ------------------------------ -----------------------
DAO                            GRADE_TYPE_CODE         GRADE                          GRADE_TYPE_CODE        
DAO                            DEPTNO                  TEST_FK_EMP                    DEPTNO                 
DAO                            DEPTNO                  TEST_FK_EMP3                   DEPTNO                 
DAO                            DEPTNO                  EMP                            DEPTNO                 
DAO                            interest_id             interest_match_info            interest_id        
        
```


# 约束验证的启停

约束的启停属性一共有两组 enable/disable ,validata/novalidate 
这两组属性两两组合 一共可以得到四种属性 


```sql

SQL> create table bsr_con_en_va(c1 number ,c2 number ) ;

Table created

SQL> create index idx_bsr_con on bsr_con_en_va (c1 ) ;

Index created

SQL> alter table bsr_con_en_va add constraint  bsr_con_1  primary key (c1) ;

Table altered

```
## disable [novalidate]  停用约束

停用后可插入重复值

```sql

SQL> alter table bsr_con_en_va disable novalidate  primary key ; 

Table altered


SQL> alter table bsr_con_en_va disable novalidate  primary key ; 

Table altered

SQL> insert into bsr_con_en_va values (1,1) ;

1 row inserted

SQL>  insert into bsr_con_en_va values (1,1) ;

1 row inserted


```

## enable novalidate   约束新增数据不约束已有数据

启用后可以看到数据中存在重复值,但是不允许再次插入重复值

```sql

SQL>  alter table bsr_con_en_va enable novalidate primary key ; 

Table altered

SQL> select * from bsr_con_en_va ;

        C1         C2
---------- ----------
         1          1
         1          1

SQL>  insert into bsr_con_en_va values (1,1) ; 

insert into bsr_con_en_va values (1,1)

ORA-00001: unique constraint (DAO.BSR_CON_1) violated


``` 





## disable validate    禁止修改数据

```sql


SQL> delete from bsr_con_en_va ;

2 rows deleted

SQL> insert into bsr_con_en_va values (1,1) ;

1 row inserted


SQL> alter table bsr_con_en_va disable  validate primary key ;

Table altered

SQL>  insert into bsr_con_en_va values (2,1) ;

insert into bsr_con_en_va values (2,1)

ORA-25128: No insert/update/delete on table with constraint (DAO.BSR_CON_1) disabled and validated



```

## enable [validate] 启用约束,约束已有数据和新进数据


```sql

SQL>  alter table bsr_con_en_va disable novalidate primary key ;

Table altered

SQL> insert into bsr_con_en_va values (1,1) ;

1 row inserted

SQL> select * from bsr_con_en_va ;

        C1         C2
---------- ----------
         1          1
         1          1

启用约束时检查到重复数据,所以报错

SQL>  alter table bsr_con_en_va enable  validate primary key ;

alter table bsr_con_en_va enable  validate primary key

ORA-02437: cannot validate (DAO.BSR_CON_1) - primary key violated

SQL> delete from bsr_con_en_va where rownum =1 ;

1 row deleted

启用约束

SQL>  alter table bsr_con_en_va enable  validate primary key ;

Table altered

插入重复数据再次报错,说明对新人数据也是检查的

SQL> insert into bsr_con_en_va values (1,1) ;

insert into bsr_con_en_va values (1,1)

ORA-00001: unique constraint (DAO.BSR_CON_1) violated

```




# 约束的延迟

延迟约束只能适用于唯一约束,主键约束,与外键约束


```sql

SQL> create table bsr_defer_1 (c1 number ,
  2                            c2 number ,
  3                            constraint pk_bsr_defer  primary key (c1)  not DEFERRABLE INITIALLY IMMEDIATE );

Table created

```

下面建表语句会报错,是由于约束类型为not deferable,不可设置初始化属性为 defered

```sql

SQL> create table bsr_defer_2 (c1 number ,
  2                            c2 number ,
  3                            constraint pk_bsr_defer2  primary key (c1)  not DEFERRABLE INITIALLY DEFERRED  );

create table bsr_defer_2 (c1 number ,
                          c2 number ,
                          constraint pk_bsr_defer2  primary key (c1)  not DEFERRABLE INITIALLY DEFERRED  )

ORA-02447: cannot defer a constraint that is not deferrable


```


```sql

SQL> create table bsr_defer_3 (c1 number ,
  2                            c2 number ,
  3                            constraint pk_bsr_defer3  primary key (c1)   DEFERRABLE INITIALLY immediate  );

Table created

```


```sql

create table bsr_defer_4 (c1 number ,
                          c2 number ,
                          constraint pk_bsr_defer4  primary key (c1)   DEFERRABLE INITIALLY deferred  );

SQL> create table bsr_defer_4 (c1 number ,
  2                            c2 number ,
  3                            constraint pk_bsr_defer4  primary key (c1)   DEFERRABLE INITIALLY deferred  );

Table created


SQL> insert into bsr_defer_4 values (1,1) ;

1 row inserted

--注意,下面插入重复数据,但是并未立即报错,而是在提交时报错
SQL> insert into bsr_defer_4 values (1,1) ;

1 row inserted

SQL> commit ;

commit

ORA-02091: transaction rolled back
ORA-00001: unique constraint (DAO.PK_BSR_DEFER4) violated

```

可以在会话中改变约束的响应级别

```
SQL>  alter session set constraints = immediate;

Session altered

SQL> insert into bsr_defer_4 values (1,1) ;

1 row inserted

SQL> insert into bsr_defer_4 values (1,1) ;

insert into bsr_defer_4 values (1,1)

ORA-00001: unique constraint (DAO.PK_BSR_DEFER4) violated

SQL> rollback ;

Rollback complete

```
