---
layout: post
title: Oracle 三表全外联结实践
date: 2018-12-29
categories: blog
tags: [SQL]
description: Oracle 三表全外联结实践
---

## 问题描述

- 有三张表内容如下 

- t1

c1 | c2 
--- | ---
1   | t1
2   | t1

- t2 

c1  | c2 
--- | ---
2   |  t2
3   |  t2

- t3 

c3  | c2 
--- | ---
2   |  t3
3   |  t3

## 希望得到的目标


c1 | c2 | c1 |c2 |c1 |c2
--- | --- | --- | --- | ---|---
1   | t1  |  |  |  1 |  3
2   | t1  | 2    | t2   |   |
' |  | 3 | t2 | 3 |t3 


### 编写SQL语句

```sql

with t1 as 
(select 1 c1, 't1' c2 from dual union all  select 2  c1, 't1' c2 from dual  ),
t2 as 
(select 2 c1, 't2' c2 from dual union all  select 3  c2, 't2' c2 from dual ),
t3 as 
(select 1 c1, 't3' c2 from dual union all  select 3  c2, 't3' c2 from dual )

select * 
 from t1 full  join t2 on t1.c1 = t2.c1 
         full  join  t3 on t1.c1 = t3.c1  
order by 1 

```

### 得到如下结果


c1 | c2 | c1 |c2 |c1 |c2
--- | --- | --- | --- | ---|---
1 | t1  | | |  1 | t3
2 | t1  |  2 | t2  |  |
' | ' |  ' |  ' | 3 | t3 
' | ' | 3  | t2 | 

这里比较奇怪的事情是最后的两行并没有如我们所愿合并为一行。

## 正确的语句

上面问题原因在于t1 yu t2 表全外关联后，t1.c1 会表现为空值，空值无法与t3的列进行匹配，只能另起一行。
这时使用coalesce函数，对连接列进行处理，保证总是能返回非空值

```sql

with t1 as 
(select 1 c1, 't1' c2 from dual union all  select 2  c1, 't1' c2 from dual  ),
t2 as 
(select 2 c1, 't2' c2 from dual union all  select 3  c2, 't2' c2 from dual ),
t3 as 
(select 1 c1, 't3' c2 from dual union all  select 3  c2, 't3' c2 from dual )

select * 
 from t1 full  join t2 on t1.c1 = t2.c1 
         full  join t3 on coalesce(t1.c1,t2.c1)= t3.c1
order by 1 

```


###  正确结果


c1 | c2 | c1 |c2 |c1 |c2
--- | --- | --- | --- | ---|---
1  | t1 | ' | ' | 1  | t3
2  | t1 | 2 | t2  |  | 
' | ' | 3 | t2 | 3 | t3







