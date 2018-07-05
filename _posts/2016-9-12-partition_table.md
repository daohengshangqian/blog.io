---
layout: post
title: Oracle分区表详解
date: 2016-09-12
categories: blog
tags: [Oracle,SQL]
description: Oracle分区表详解
---
# 分区表概念
## 1 what 什么是分区
Partitioning enhances the performance, manageability, and availability of a wide variety of applications and helps reduce the total cost of ownership for storing large amounts of data.
Partitioning allows tables, indexes, and index-organized tables to be subdivided into smaller pieces, enabling these database objects to be managed and accessed at a finer level of granularity.
Oracle provides a rich variety of partitioning strategies and extensions to address every business requirement. Because it is entirely transparent, partitioning can be applied to almost any application without the need for potentially expensive and time consuming application changes. 

![image](http://wx4.sinaimg.cn/mw690/006Pv9mtly1fn3rfxl7ldg30d609q3ym.gif)

## 2 why 为什么要分区

数据库随着数据量的增加，一个表越来越庞大。带来了很多问题。
主要表现为
1. 执行sql 语句没有索引，扫描过多无用数据
1. 需要一部分数据，由于索引单块读特性，优化器选择全表扫描，读入大量不需要的数据
1. 维护数据时，话费大量时间

使用分区表可以避免以上问题
使用分区表还可以
1.  均衡IO 分区表每个分区都是一个独立的段，可以放到不同的表空间去
1.  增加可用性，一个分区数据出问题不影响其他分区的使用

## 3 when 什么时候采用分区

Oracle建议

- 分区表
1. 超过2G可以考虑进行分区
2. 存在历史数据，新数据放在最新的区里面去。
3. 数据放在不同类型的存储上。


- 分区索引

1. 在删除数据后，避免重建整个索引
2. 维护部分数据后，不让整个索引失效
3. 减少由于索引列单调增长引起的索引失衡

## 4 how many Oracle 支持多少分区

Oracle 9   Tables can be partitioned into up to 64,000 separate partitions.
Oracle 10  Tables can be partitioned into up to 1024K-1 separate partitions.
Oracle 11  a million separate partitions 

## 5 how  to 如何分区？分区有那些类型

1 范围分区

范围分区就是根据分区字段的取值范围进行分区，将数据存储在不同的分区字段中。

2 散列分区

散列分区是指按照Oracle提供的散列（HASH）函数计算列值数据，并最终按照函数结果进行分区。
在进行范围分区时，有时用户无法对各个分区中可能具有的记录数目进行预测，这会导致某个分区中记录数目过多而其它分区中的记录很少的不平衡分区情况。
在这种情况下，用户可以考虑创建散列分区表。

3 列表分区

如果分区字段的值不能划分范围（非数字或日期数据类型），并且分区字段的取值范围只有一个包含少数值的集合，则可以对表进行列表分区。在进行列表分区时，需要为每个分区指定一个取值列表，分区字段的取值处于同一个取值列

# 分区表如何创建

## 1范围分区
```sql
CREATE TABLE time_range_sales
   ( prod_id        NUMBER(6)
   , cust_id        NUMBER
   , time_id        DATE
   , channel_id     CHAR(1)
   , promo_id       NUMBER(6)
   , quantity_sold  NUMBER(3)
   , amount_sold    NUMBER(10,2)
   )
PARTITION BY RANGE (time_id)
 (PARTITION SALES_1998 VALUES LESS THAN (TO_DATE('01-JAN-1999','DD-MON-YYYY')),
  PARTITION SALES_1999 VALUES LESS THAN (TO_DATE('01-JAN-2000','DD-MON-YYYY')),
  PARTITION SALES_2000 VALUES LESS THAN (TO_DATE('01-JAN-2001','DD-MON-YYYY')),
  PARTITION SALES_2001 VALUES LESS THAN (MAXVALUE)
 ); 

```
## 2 列表分区
```sql
CREATE TABLE list_sales
   ( prod_id        NUMBER(6)
   , cust_id        NUMBER
   , time_id        DATE
   , channel_id     CHAR(1)
   , promo_id       NUMBER(6)
   , quantity_sold  NUMBER(3)
   , amount_sold    NUMBER(10,2)
   )
PARTITION BY LIST (channel_id)
 (PARTITION even_channels VALUES (2,4),
  PARTITION odd_channels VALUES (3,9)
 ); 

```
## 3 哈希分区
```sql 
CREATE TABLE hash_sales
   ( prod_id        NUMBER(6)
   , cust_id        NUMBER
   , time_id        DATE
   , channel_id     CHAR(1)
   , promo_id       NUMBER(6)
   , quantity_sold  NUMBER(3)
   , amount_sold    NUMBER(10,2)
   )
PARTITION BY HASH (prod_id)
PARTITIONS 2; 

```
## 4 range –hash 组合分区
```sql
CREATE TABLE composite_sales
    ( prod_id        NUMBER(6)
    , cust_id        NUMBER
    , time_id        DATE
    , channel_id     CHAR(1)
    , promo_id       NUMBER(6)
    , quantity_sold  NUMBER(3)
    , amount_sold         NUMBER(10,2)
    ) 
PARTITION BY RANGE (time_id)
SUBPARTITION BY HASH (channel_id)
  (PARTITION SALES_Q1_1998 VALUES LESS THAN (TO_DATE('01-APR-1998','DD-MON-YYYY')),
   PARTITION SALES_Q2_1998 VALUES LESS THAN (TO_DATE('01-JUL-1998','DD-MON-YYYY')),
   PARTITION SALES_Q3_1998 VALUES LESS THAN (TO_DATE('01-OCT-1998','DD-MON-YYYY')),
   PARTITION SALES_Q4_1998 VALUES LESS THAN (TO_DATE('01-JAN-1999','DD-MON-YYYY')),
   PARTITION SALES_Q1_1999 VALUES LESS THAN (TO_DATE('01-APR-1999','DD-MON-YYYY')),
   PARTITION SALES_Q2_1999 VALUES LESS THAN (TO_DATE('01-JUL-1999','DD-MON-YYYY')),
   PARTITION SALES_Q3_1999 VALUES LESS THAN (TO_DATE('01-OCT-1999','DD-MON-YYYY')),
   PARTITION SALES_Q4_1999 VALUES LESS THAN (TO_DATE('01-JAN-2000','DD-MON-YYYY')),
   PARTITION SALES_Q1_2000 VALUES LESS THAN (TO_DATE('01-APR-2000','DD-MON-YYYY')),
   PARTITION SALES_Q2_2000 VALUES LESS THAN (TO_DATE('01-JUL-2000','DD-MON-YYYY'))
      SUBPARTITIONS 8,
   PARTITION SALES_Q3_2000 VALUES LESS THAN (TO_DATE('01-OCT-2000','DD-MON-YYYY'))
     (SUBPARTITION ch_c,
      SUBPARTITION ch_i,
      SUBPARTITION ch_p,
      SUBPARTITION ch_s,
      SUBPARTITION ch_t),
   PARTITION SALES_Q4_2000 VALUES LESS THAN (MAXVALUE)
      SUBPARTITIONS 4)

```
## 5 range-list组合分区
```sql
CREATE TABLE customers_part (
   customer_id        NUMBER(6),
   cust_first_name    VARCHAR2(20),
   cust_last_name     VARCHAR2(20),
   nls_territory      VARCHAR2(30),
   credit_limit       NUMBER(9,2)) 
   PARTITION BY RANGE (credit_limit)
   SUBPARTITION BY LIST (nls_territory)
      SUBPARTITION TEMPLATE 
         (SUBPARTITION east  VALUES 
            ('CHINA', 'JAPAN', 'INDIA', 'THAILAND'),
          SUBPARTITION west VALUES 
             ('AMERICA', 'GERMANY', 'ITALY', 'SWITZERLAND'),
          SUBPARTITION other VALUES (DEFAULT))
      (PARTITION p1 VALUES LESS THAN (1000),
       PARTITION p2 VALUES LESS THAN (2500),
       PARTITION p3 VALUES LESS THAN (MAXVALUE));

```
## 6  range-range组合分区
```sql
CREATE TABLE shipments
( order_id      NUMBER NOT NULL
, order_date    DATE NOT NULL
, delivery_date DATE NOT NULL
, customer_id   NUMBER NOT NULL
, sales_amount  NUMBER NOT NULL
)
PARTITION BY RANGE (order_date)
SUBPARTITION BY RANGE (delivery_date)
( PARTITION p_2006_jul VALUES LESS THAN (TO_DATE('01-AUG-2006','dd-MON-yyyy'))
  ( SUBPARTITION p06_jul_e VALUES LESS THAN (TO_DATE('15-AUG-2006','dd-MON-yyyy'))
  , SUBPARTITION p06_jul_a VALUES LESS THAN (TO_DATE('01-SEP-2006','dd-MON-yyyy'))
  , SUBPARTITION p06_jul_l VALUES LESS THAN (MAXVALUE)
  )
, PARTITION p_2006_aug VALUES LESS THAN (TO_DATE('01-SEP-2006','dd-MON-yyyy'))
  ( SUBPARTITION p06_aug_e VALUES LESS THAN (TO_DATE('15-SEP-2006','dd-MON-yyyy'))
  , SUBPARTITION p06_aug_a VALUES LESS THAN (TO_DATE('01-OCT-2006','dd-MON-yyyy'))
  , SUBPARTITION p06_aug_l VALUES LESS THAN (MAXVALUE)
  )
, PARTITION p_2006_sep VALUES LESS THAN (TO_DATE('01-OCT-2006','dd-MON-yyyy'))
  ( SUBPARTITION p06_sep_e VALUES LESS THAN (TO_DATE('15-OCT-2006','dd-MON-yyyy'))
  , SUBPARTITION p06_sep_a VALUES LESS THAN (TO_DATE('01-NOV-2006','dd-MON-yyyy'))
  , SUBPARTITION p06_sep_l VALUES LESS THAN (MAXVALUE)
  )
, PARTITION p_2006_oct VALUES LESS THAN (TO_DATE('01-NOV-2006','dd-MON-yyyy'))
  ( SUBPARTITION p06_oct_e VALUES LESS THAN (TO_DATE('15-NOV-2006','dd-MON-yyyy'))
  , SUBPARTITION p06_oct_a VALUES LESS THAN (TO_DATE('01-DEC-2006','dd-MON-yyyy'))
  , SUBPARTITION p06_oct_l VALUES LESS THAN (MAXVALUE)
  )
, PARTITION p_2006_nov VALUES LESS THAN (TO_DATE('01-DEC-2006','dd-MON-yyyy'))
  ( SUBPARTITION p06_nov_e VALUES LESS THAN (TO_DATE('15-DEC-2006','dd-MON-yyyy'))
  , SUBPARTITION p06_nov_a VALUES LESS THAN (TO_DATE('01-JAN-2007','dd-MON-yyyy'))
  , SUBPARTITION p06_nov_l VALUES LESS THAN (MAXVALUE)
  )
, PARTITION p_2006_dec VALUES LESS THAN (TO_DATE('01-JAN-2007','dd-MON-yyyy'))
  ( SUBPARTITION p06_dec_e VALUES LESS THAN (TO_DATE('15-JAN-2007','dd-MON-yyyy'))
  , SUBPARTITION p06_dec_a VALUES LESS THAN (TO_DATE('01-FEB-2007','dd-MON-yyyy'))
  , SUBPARTITION p06_dec_l VALUES LESS THAN (MAXVALUE)
  )
);

```
## 7 List-hash组合分区
```sql
CREATE TABLE accounts
( id             NUMBER
, account_number NUMBER
, customer_id    NUMBER
, balance        NUMBER
, branch_id      NUMBER
, region         VARCHAR(2)
, status         VARCHAR2(1)
)
PARTITION BY LIST (region)
SUBPARTITION BY HASH (customer_id) SUBPARTITIONS 8
( PARTITION p_northwest VALUES ('OR', 'WA')
, PARTITION p_southwest VALUES ('AZ', 'UT', 'NM')
, PARTITION p_northeast VALUES ('NY', 'VM', 'NJ')
, PARTITION p_southeast VALUES ('FL', 'GA')
, PARTITION p_northcentral VALUES ('SD', 'WI')
, PARTITION p_southcentral VALUES ('OK', 'TX')
);

```
## 8  list-list组合分区
```sql
CREATE TABLE accounts
( id             NUMBER
, account_number NUMBER
, customer_id    NUMBER
, balance        NUMBER
, branch_id      NUMBER
, region         VARCHAR(2)
, status         VARCHAR2(1)
)
PARTITION BY LIST (region)
SUBPARTITION BY LIST (status)
( PARTITION p_northwest VALUES ('OR', 'WA')
  ( SUBPARTITION p_nw_bad VALUES ('B')
  , SUBPARTITION p_nw_average VALUES ('A')
  , SUBPARTITION p_nw_good VALUES ('G')
  )
, PARTITION p_southwest VALUES ('AZ', 'UT', 'NM')
  ( SUBPARTITION p_sw_bad VALUES ('B')
  , SUBPARTITION p_sw_average VALUES ('A')
  , SUBPARTITION p_sw_good VALUES ('G')
  )
, PARTITION p_northeast VALUES ('NY', 'VM', 'NJ')
  ( SUBPARTITION p_ne_bad VALUES ('B')
  , SUBPARTITION p_ne_average VALUES ('A')
  , SUBPARTITION p_ne_good VALUES ('G')
  )
, PARTITION p_southeast VALUES ('FL', 'GA')
  ( SUBPARTITION p_se_bad VALUES ('B')
  , SUBPARTITION p_se_average VALUES ('A')
  , SUBPARTITION p_se_good VALUES ('G')
  )
, PARTITION p_northcentral VALUES ('SD', 'WI')
  ( SUBPARTITION p_nc_bad VALUES ('B')
  , SUBPARTITION p_nc_average VALUES ('A')
  , SUBPARTITION p_nc_good VALUES ('G')
  )
, PARTITION p_southcentral VALUES ('OK', 'TX')
  ( SUBPARTITION p_sc_bad VALUES ('B')
  , SUBPARTITION p_sc_average VALUES ('A')
  , SUBPARTITION p_sc_good VALUES ('G')
  )
);

```
## 9 list-range组合分区
```sql
CREATE TABLE accounts
( id             NUMBER
, account_number NUMBER
, customer_id    NUMBER
, balance        NUMBER
, branch_id      NUMBER
, region         VARCHAR(2)
, status         VARCHAR2(1)
)
PARTITION BY LIST (region)
SUBPARTITION BY RANGE (balance)
( PARTITION p_northwest VALUES ('OR', 'WA')
  ( SUBPARTITION p_nw_low VALUES LESS THAN (1000)
  , SUBPARTITION p_nw_average VALUES LESS THAN (10000)
  , SUBPARTITION p_nw_high VALUES LESS THAN (100000)
  , SUBPARTITION p_nw_extraordinary VALUES LESS THAN (MAXVALUE)
  )
, PARTITION p_southwest VALUES ('AZ', 'UT', 'NM')
  ( SUBPARTITION p_sw_low VALUES LESS THAN (1000)
  , SUBPARTITION p_sw_average VALUES LESS THAN (10000)
  , SUBPARTITION p_sw_high VALUES LESS THAN (100000)
  , SUBPARTITION p_sw_extraordinary VALUES LESS THAN (MAXVALUE)
  )
, PARTITION p_northeast VALUES ('NY', 'VM', 'NJ')
  ( SUBPARTITION p_ne_low VALUES LESS THAN (1000)
  , SUBPARTITION p_ne_average VALUES LESS THAN (10000)
  , SUBPARTITION p_ne_high VALUES LESS THAN (100000)
  , SUBPARTITION p_ne_extraordinary VALUES LESS THAN (MAXVALUE)
  )
, PARTITION p_southeast VALUES ('FL', 'GA')
  ( SUBPARTITION p_se_low VALUES LESS THAN (1000)
  , SUBPARTITION p_se_average VALUES LESS THAN (10000)
  , SUBPARTITION p_se_high VALUES LESS THAN (100000)
  , SUBPARTITION p_se_extraordinary VALUES LESS THAN (MAXVALUE)
  )
, PARTITION p_northcentral VALUES ('SD', 'WI')
  ( SUBPARTITION p_nc_low VALUES LESS THAN (1000)
  , SUBPARTITION p_nc_average VALUES LESS THAN (10000)
  , SUBPARTITION p_nc_high VALUES LESS THAN (100000)
  , SUBPARTITION p_nc_extraordinary VALUES LESS THAN (MAXVALUE)
  )
, PARTITION p_southcentral VALUES ('OK', 'TX')
  ( SUBPARTITION p_sc_low VALUES LESS THAN (1000)
  , SUBPARTITION p_sc_average VALUES LESS THAN (10000)
  , SUBPARTITION p_sc_high VALUES LESS THAN (100000)
  , SUBPARTITION p_sc_extraordinary VALUES LESS THAN (MAXVALUE)
  )
) ENABLE ROW MOVEMENT;

```
## 10  interval-hash分区
```sql
CREATE TABLE sales
  ( prod_id       NUMBER(6)
  , cust_id       NUMBER
  , time_id       DATE
  , channel_id    CHAR(1)
  , promo_id      NUMBER(6)
  , quantity_sold NUMBER(3)
  , amount_sold   NUMBER(10,2)
  )
 PARTITION BY RANGE (time_id) INTERVAL (NUMTOYMINTERVAL(1,'MONTH'))
 SUBPARTITION BY HASH (cust_id) SUBPARTITIONS 4
 ( PARTITION before_2000 VALUES LESS THAN (TO_DATE('01-JAN-2000','dd-MON-yyyy')))
PARALLEL;

```
## 11  interval-list  分区
```sql
CREATE TABLE sales
  ( prod_id       NUMBER(6)
  , cust_id       NUMBER
  , time_id       DATE
  , channel_id    CHAR(1)
  , promo_id      NUMBER(6)
  , quantity_sold NUMBER(3)
  , amount_sold   NUMBER(10,2)
  )
 PARTITION BY RANGE (time_id) INTERVAL (NUMTODSINTERVAL(1,'DAY'))
 SUBPARTITION BY LIST (channel_id)
   SUBPARTITION TEMPLATE
   ( SUBPARTITION p_catalog VALUES ('C')
   , SUBPARTITION p_internet VALUES ('I')
   , SUBPARTITION p_partners VALUES ('P')
   , SUBPARTITION p_direct_sales VALUES ('S')
   , SUBPARTITION p_tele_sales VALUES ('T')
   )
 ( PARTITION before_2000 VALUES LESS THAN (TO_DATE('01-JAN-2000','dd-MON-yyyy')))
PARALLEL;

```
## 12 interval-range
```sql
CREATE TABLE sales
  ( prod_id       NUMBER(6)
  , cust_id       NUMBER
  , time_id       DATE
  , channel_id    CHAR(1)
  , promo_id      NUMBER(6)
  , quantity_sold NUMBER(3)
  , amount_sold   NUMBER(10,2)
  )
 PARTITION BY RANGE (time_id) INTERVAL (NUMTODSINTERVAL(1,'DAY'))
SUBPARTITION BY RANGE(amount_sold)
   SUBPARTITION TEMPLATE
   ( SUBPARTITION p_low VALUES LESS THAN (1000)
   , SUBPARTITION p_medium VALUES LESS THAN (4000)
   , SUBPARTITION p_high VALUES LESS THAN (8000)
   , SUBPARTITION p_ultimate VALUES LESS THAN (maxvalue)
   )
 ( PARTITION before_2000 VALUES LESS THAN (TO_DATE('01-JAN-2000','dd-MON-yyyy')))
PARALLEL;

```
## 13 使用分区模版 （ range-hash ）
```sql
CREATE TABLE emp_sub_template (deptno NUMBER, empname VARCHAR(32), grade NUMBER)
     PARTITION BY RANGE(deptno) SUBPARTITION BY HASH(empname)
     SUBPARTITION TEMPLATE
         (SUBPARTITION a TABLESPACE ts1,
          SUBPARTITION b TABLESPACE ts2,
          SUBPARTITION c TABLESPACE ts3,
          SUBPARTITION d TABLESPACE ts4
         )
    (PARTITION p1 VALUES LESS THAN (1000),
     PARTITION p2 VALUES LESS THAN (2000),
     PARTITION p3 VALUES LESS THAN (MAXVALUE)
    );

```
## 14 使用分区模版 （ range-list  ）
```sql
CREATE TABLE stripe_regional_sales
            ( deptno number, item_no varchar2(20),
              txn_date date, txn_amount number, state varchar2(2))
   PARTITION BY RANGE (txn_date)
   SUBPARTITION BY LIST (state)
   SUBPARTITION TEMPLATE 
      (SUBPARTITION northwest VALUES ('OR', 'WA') TABLESPACE tbs_1,
       SUBPARTITION southwest VALUES ('AZ', 'UT', 'NM') TABLESPACE tbs_2,
       SUBPARTITION northeast VALUES ('NY', 'VM', 'NJ') TABLESPACE tbs_3,
       SUBPARTITION southeast VALUES ('FL', 'GA') TABLESPACE tbs_4,
       SUBPARTITION midwest VALUES ('SD', 'WI') TABLESPACE tbs_5,
       SUBPARTITION south VALUES ('AL', 'AK') TABLESPACE tbs_6,
       SUBPARTITION others VALUES (DEFAULT ) TABLESPACE tbs_7
      )
  (PARTITION q1_1999 VALUES LESS THAN ( TO_DATE('01-APR-1999','DD-MON-YYYY')),
   PARTITION q2_1999 VALUES LESS THAN ( TO_DATE('01-JUL-1999','DD-MON-YYYY')),
   PARTITION q3_1999 VALUES LESS THAN ( TO_DATE('01-OCT-1999','DD-MON-YYYY')),
   PARTITION q4_1999 VALUES LESS THAN ( TO_DATE('1-JAN-2000','DD-MON-YYYY'))
  );


```
## 15  组合列范围分区
```sql
CREATE TABLE sales_demo (
   year          NUMBER, 
   month         NUMBER,
   day           NUMBER,
   amount_sold   NUMBER) 
PARTITION BY RANGE (year,month) 
  (PARTITION before2001 VALUES LESS THAN (2001,1),
   PARTITION q1_2001    VALUES LESS THAN (2001,4),
   PARTITION q2_2001    VALUES LESS THAN (2001,7),
   PARTITION q3_2001    VALUES LESS THAN (2001,10),
   PARTITION q4_2001    VALUES LESS THAN (2002,1),
   PARTITION future     VALUES LESS THAN (MAXVALUE,0));

REM  12-DEC-2000
INSERT INTO sales_demo VALUES(2000,12,12, 1000);
REM  17-MAR-2001
INSERT INTO sales_demo VALUES(2001,3,17, 2000);
REM  1-NOV-2001
INSERT INTO sales_demo VALUES(2001,11,1, 5000);
REM  1-JAN-2002
INSERT INTO sales_demo VALUES(2002,1,1, 4000);

```
## 16   虚拟列分区表
```sql
CREATE TABLE sales
  ( prod_id       NUMBER(6) NOT NULL
  , cust_id       NUMBER NOT NULL
  , time_id       DATE NOT NULL
  , channel_id    CHAR(1) NOT NULL
  , promo_id      NUMBER(6) NOT NULL
  , quantity_sold NUMBER(3) NOT NULL
  , amount_sold   NUMBER(10,2) NOT NULL
  , total_amount AS (quantity_sold * amount_sold)
  )
 PARTITION BY RANGE (time_id) INTERVAL (NUMTOYMINTERVAL(1,'MONTH'))
 SUBPARTITION BY RANGE(total_amount)
 SUBPARTITION TEMPLATE
   ( SUBPARTITION p_small VALUES LESS THAN (1000)
   , SUBPARTITION p_medium VALUES LESS THAN (5000)
   , SUBPARTITION p_large VALUES LESS THAN (10000)
   , SUBPARTITION p_extreme VALUES LESS THAN (MAXVALUE)
   )
 (PARTITION sales_before_2007 VALUES LESS THAN
        (TO_DATE('01-JAN-2007','dd-MON-yyyy'))
)
ENABLE ROW MOVEMENT
PARALLEL NOLOGGING;

```
## 17  索引组织范围分区表
```sql
CREATE TABLE sales(acct_no NUMBER(5), 
                   acct_name CHAR(30), 
                   amount_of_sale NUMBER(6), 
                   week_no INTEGER,
                   sale_details VARCHAR2(1000),
             PRIMARY KEY (acct_no, acct_name, week_no)) 
     ORGANIZATION INDEX 
             INCLUDING week_no
             OVERFLOW TABLESPACE overflow_here
     PARTITION BY RANGE (week_no)
            (PARTITION VALUES LESS THAN (5) 
                   TABLESPACE ts1,
             PARTITION VALUES LESS THAN (9) 
                   TABLESPACE ts2 OVERFLOW TABLESPACE overflow_ts2,
             ...
             PARTITION VALUES LESS THAN (MAXVALUE) 
                   TABLESPACE ts13);

```
## 18 索引组织hash分区表
```sql
CREATE TABLE sales(acct_no NUMBER(5), 
                   acct_name CHAR(30), 
                   amount_of_sale NUMBER(6), 
                   week_no INTEGER,
                   sale_details VARCHAR2(1000),
             PRIMARY KEY (acct_no, acct_name, week_no)) 
     ORGANIZATION INDEX 
             INCLUDING week_no
     OVERFLOW
          PARTITION BY HASH (week_no)
             PARTITIONS 16
             STORE IN (ts1, ts2, ts3, ts4)
             OVERFLOW STORE IN (ts3, ts6, ts9);

```
## 19 索引组织列表分区表
```sql
CREATE TABLE sales(acct_no NUMBER(5), 
                   acct_name CHAR(30), 
                   amount_of_sale NUMBER(6), 
                   week_no INTEGER,
                   sale_details VARCHAR2(1000),
             PRIMARY KEY (acct_no, acct_name, week_no)) 
     ORGANIZATION INDEX 
             INCLUDING week_no
             OVERFLOW TABLESPACE example
     PARTITION BY LIST (week_no)
            (PARTITION VALUES (1, 2, 3, 4) 
                   TABLESPACE example,
             PARTITION VALUES (5, 6, 7, 8) 
                   TABLESPACE example OVERFLOW TABLESPACE example,
             PARTITION VALUES (DEFAULT) 
                   TABLESPACE example);

```
## 20 11G 新特性 虚拟列分区

```sql

CREATE TABLE car_rentals
( id                  NUMBER NOT NULL
 , customer_id         NUMBER NOT NULL
 , confirmation_number VARCHAR2(12) NOT NULL
 , car_id              NUMBER
 , car_type            VARCHAR2(10)
 , requested_car_type  VARCHAR2(10) NOT NULL
 , reservation_date    DATE NOT NULL
 , start_date          DATE NOT NULL
 , end_date            DATE
 , country as (substr(confirmation_number,9,2))
) PARTITION BY LIST (country)
SUBPARTITION BY HASH (customer_id)
SUBPARTITIONS 16
( PARTITION north_america VALUES ('US','CA','MX')
 , PARTITION south_america VALUES ('BR','AR','PE')
 , PARTITION europe VALUES ('GB','DE','NL','BE','FR','ES','IT','CH')
 , PARTITION apac VALUES ('NZ','AU','IN','CN')
) ENABLE ROW MOVEMENT;

```

# 分区表维护

## RANGE分区
### 增加分区
#### 无maxvalue分区下增加分区
```sql
SQL> alter table  dao_range_partition add PARTITION SALES_2001  VALUES LESS THAN (TO_DATE('01-JAN-2002','DD-MON-YYYY'));

Table altered.

SQL> select  DTP.TABLE_OWNER,DTP.TABLE_OWNER ,DTP.PARTITION_NAME
  2  from dba_tab_partitions DTP
  3  where table_owner='DAO'
  4  AND  TABLE_NAME='DAO_RANGE_PARTITION';

TABLE_OWNER  TABLE_OWNER     PARTITION_NAME
------------ --------------- --------------
DAO          DAO             SALES_1998
DAO          DAO             SALES_1999
DAO          DAO             SALES_2000
DAO          DAO             SALES_2001

```
#### 增加max分区

```sql
SQL> ALTER TABLE  DAO_RANGE_PARTITION ADD PARTITION SALES_MAX  VALUES LESS THAN ( MAXVALUE );

Table altered.

SQL> select  DTP.TABLE_OWNER,DTP.TABLE_OWNER ,DTP.PARTITION_NAME
  2  from dba_tab_partitions DTP
  3  where table_owner='DAO'
  4  AND  TABLE_NAME='DAO_RANGE_PARTITION';

TABLE_OWNER  TABLE_OWNER PARTITION_NAME
------------------------------------------
DAO          DAO         SALES_1998
DAO          DAO         SALES_1999
DAO          DAO         SALES_2000
DAO          DAO         SALES_2001
DAO          DAO         SALES_MAX   

```

#### 通过拆分maxvalue分区增加分区

```sql
SQL> alter table DAO_RANGE_PARTITION  split   partition SALES_MAX  at
    (TO_DATE(' 2003-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN'))
    into (partition SALES_2002 tablespace users ,partition SALES_MAX tablespace users );

Table altered.

```
#### 通过drop maxvalue分区 加入分区
```sql
SQL> alter table DAO_RANGE_PARTITION drop partition SALES_MAX ;

Table altered.

SQL> select  DTP.TABLE_OWNER,DTP.TABLE_OWNER ,DTP.PARTITION_NAME
  2  from dba_tab_partitions DTP
  3  where table_owner='DAO'
  4  AND  TABLE_NAME='DAO_RANGE_PARTITION'
  5  ;

TABLE_OWNER  TABLE_OWNER  PARTITION_NAME
------------ ------------ ---------------
DAO          DAO          SALES_1998
DAO          DAO          SALES_1999
DAO          DAO          SALES_2000
DAO          DAO          SALES_2001
DAO          DAO          SALES_2002


SQL> alter table  dao_range_partition add PARTITION SALES_2003  VALUES LESS THAN (TO_DATE('01-JAN-2004','DD-MON-YYYY'));

Table altered

SQL> select  DTP.TABLE_OWNER,DTP.TABLE_OWNER ,DTP.PARTITION_NAME
  2  from dba_tab_partitions DTP
  3  where table_owner='DAO'
  4  AND  TABLE_NAME='DAO_RANGE_PARTITION';

TABLE_OWNER TABLE_OWNER  PARTITION_NAME
----------- ------------ ----------------
DAO         DAO          SALES_1998
DAO         DAO          SALES_1999
DAO         DAO          SALES_2000
DAO         DAO          SALES_2001
DAO         DAO          SALES_2002
DAO         DAO          SALES_2003


SQL>  ALTER TABLE  DAO_RANGE_PARTITION ADD PARTITION SALES_MAX  VALUES LESS THAN ( MAXVALUE );

Table altered.

```
### 分区删除

```sql

SQL> alter table DAO_RANGE_PARTITION drop partition SALES_MAX ;

Table altered.

SQL> select  DTP.TABLE_OWNER,DTP.TABLE_OWNER ,DTP.PARTITION_NAME
  2  from dba_tab_partitions DTP
  3  where table_owner='DAO'
  4  AND  TABLE_NAME='DAO_RANGE_PARTITION'
  5  ;

TABLE_OWNER  TABLE_OWNER  PARTITION_NAME
------------ ------------ ---------------
DAO          DAO          SALES_1998
DAO          DAO          SALES_1999
DAO          DAO          SALES_2000
DAO          DAO          SALES_2001
DAO          DAO          SALES_2002

```
### 分区TRUNCATE

```sql
SQL> alter table DAO_RANGE_PARTITION truncate partition  SALES_2002 ;

Table truncated.

```

### 分区拆分

```sql
SQL> alter table DAO_RANGE_PARTITION  split   partition SALES_MAX  at
    (TO_DATE(' 2003-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN'))
    into (partition SALES_2002 tablespace users ,partition SALES_MAX tablespace users );

Table altered.

```

### 分区合并
```sql
SQL>  select DTP.TABLE_OWNER, DTP.TABLE_NAME, DTP.PARTITION_NAME
  2     from dba_tab_partitions DTP
  3    where table_owner = 'DAO'
  4      AND TABLE_NAME = 'DAO_RANGE_PARTITION';

TABLE_OWNER        TABLE_NAME             PARTITION_NAME
----------------------------------------- ---------------
DAO                DAO_RANGE_PARTITION    SALES_1998
DAO                DAO_RANGE_PARTITION    SALES_1999
DAO                DAO_RANGE_PARTITION    SALES_2000
DAO                DAO_RANGE_PARTITION    SALES_2001
DAO                DAO_RANGE_PARTITION    SALES_2002
DAO                DAO_RANGE_PARTITION    SALES_2003
DAO                DAO_RANGE_PARTITION    SALES_MAX

7 rows selected.
 
 
SQL> ALTER TABLE DAO_RANGE_PARTITION MERGE PARTITIONS  SALES_1998,SALES_1999  INTO  PARTITION SALES_1999 ;

Table altered.


SQL>  select  DTP.TABLE_OWNER,DTP.TABLE_NAME ,DTP.PARTITION_NAME from dba_tab_partitions DTP where table_owner='DAO'   AND  TABLE_NAME='DAO_RANGE_PARTITION';

TABLE_OWNER     TABLE_NAME                     PARTITION_NAME
--------------- ------------------------------ --------------
DAO             DAO_RANGE_PARTITION            SALES_1999
DAO             DAO_RANGE_PARTITION            SALES_2000
DAO             DAO_RANGE_PARTITION            SALES_2001
DAO             DAO_RANGE_PARTITION            SALES_2002
DAO             DAO_RANGE_PARTITION            SALES_2003
DAO             DAO_RANGE_PARTITION            SALES_MAX

6 rows selected.

```
### 分区交换
```sql
SQL>  select  DTP.TABLE_OWNER,DTP.TABLE_NAME ,DTP.PARTITION_NAME from dba_tab_partitions DTP where table_owner='DAO'   AND  TABLE_NAME='DAO_RANGE_PARTITION';

TABLE_OWNER    TABLE_NAME            PARTITION_NAME
-------------- --------------------- ---------------
DAO            DAO_RANGE_PARTITION   SALES_1999
DAO            DAO_RANGE_PARTITION   SALES_2000
DAO            DAO_RANGE_PARTITION   SALES_2001
DAO            DAO_RANGE_PARTITION   SALES_2002
DAO            DAO_RANGE_PARTITION   SALES_2003
DAO            DAO_RANGE_PARTITION   SALES_MAX

```

## HASH分区
### 增加分区
```sql
SQL> insert into  dao_test_hash 
  2  select level,to_char(level)||to_char(level)
  3  from dual
  4  connect by level<=100;

100 rows created.

SQL> commit ;

Commit complete.


SQL> select count(*), 'all'
  2    from dao_test_hash
  3  union all
  4  select count(*), 'p1'
  5    from dao_test_hash partition (p1)
  6  union all
  7  select count(*), 'p2' 
  8     from dao_test_hash partition (p2);

  COUNT(*) 'AL
---------- ---
       100 all
        45 p1
        55 p2
SQL> alter table dao_test_hash add partition p3 ;

Table altered.

SQL> select count(*), 'all'
  2    from dao_test_hash
  3  union all
  4  select count(*), 'p1'
  5    from dao_test_hash partition(p1)
  6  union all
  7  select count(*), 'p2' 
  8     from dao_test_hash partition(p2)
  9  union all   
 10  select count(*), 'p3' 
 11     from dao_test_hash partition(p3);   

  COUNT(*) 'AL
---------- ---
       100 all
        20 p1
        55 p2
        25 p3


SQL> alter table dao_test_hash add partition p4 ;


Table altered.

SQL> SQL> select count(*), 'all'
  2    from dao_test_hash
  3  union all
  4  select count(*), 'p1'
  5    from dao_test_hash partition(p1)
  6  union all
  7  select count(*), 'p2' 
  8     from dao_test_hash partition(p2)
  9  union all   
 10  select count(*), 'p3' 
 11     from dao_test_hash partition(p3)
 12  union all
 13  select count(*), 'p4' 
 14     from dao_test_hash partition(p4);

  COUNT(*) 'AL
---------- ---
       100 all
        20 p1
        28 p2
        25 p3
        27 p4

SQL> select dao_xx+1,count(*)
  2  from
  3  (select ora_hash(level,3,0) dao_xx
  4  from dual 
  5  connect by level <=100 )
  6  group by dao_xx ;

  DAO_XX+1   COUNT(*)
---------- ----------
         2         28
         3         25
         4         27
         1         20
         
注意:Hash 分区数据分布函数使用的是ORA_HASH函数

```
## 列表分区

- 同RANGE分区
- 

# 分区索引

![image](http://wx3.sinaimg.cn/mw690/006Pv9mtly1fn3sgik15ag30d805f3yc.gif)


## 全局索引

- 指向任何分区中的行
- 索引可以分区或不分区
- 表可以是分区表或非分区表
- 全局分区索引只能是范围分区
- 分区键必须是索引的前缀


一个b-tree结构索引跨所有分区

- 优点: 高效访问个另单条记录
- 缺点: 相对于分区索引缺乏可管理性
- 在快速访问单条记录时建议使用

索引被分区但不取决于数据 每个索引分区可以引用任何一个或所有表分区

- 优点: 平衡性能和可管理性
- 缺点: 在分区DDL操作时较本地分区索引效率低

本地索引

- 索引分区和表分区对应，仅指向应用分区中的行
- 索引分区键总是和表分区键相同


- 已分区索引 (b-tree or bitmap) 使用相同的数据连界优点: 易于管理; 并行索引扫描
- 缺点: 检索少量数据时效率略降低 

几乎所有数据仓库环境的索引都为本地索引

## 对比

全局索引与本地分区索引不同的是，全局分区索引的分区机制与表的分区机制不一样。全局分区索引只能是B树索引，到目前为止(10gR2)，oracle只支持有前缀的全局索引。另外oracle不会自动的维护全局分区索引，当我们在对表的分区做修改之后，如果执行修改的语句不加上update global indexes的话，那么索引将不可用。由于本地索引的分区机制和表的分区机制一样，所以分区数量和表的分区数一致，维护也可以自动实现，对表分区的操作只会影响相关的索引分区。就性能而言，如果不考虑取数是否局限于某些分区，那么理论上对于单值检索，无分区全局索引性能优于分区索引（含本地索引和全局分区索引），但是实际情况通常数据查询都局限于少数几个分区，所以对于数据量很大的分区表，可以考虑创建本地索引。本地索引的好处是对分区的操作只会影响相应的分区，不影响未操作的分区。全局分区索引的好处在于由于分区机制和表分区机制不同，一个分区索引可能包含来自好几个数据分区的数据，所以可以分散读写，较好的缓解索引在大并发下的争用
。 
总而言之，选择索引的类型，要求对于性能和可用性能够较好的平衡，一般建议通过对比测试来选择最适合业务的索引策略

![image](http://wx1.sinaimg.cn/mw690/006Pv9mtly1fn3sn52cguj30uv0kbmy0.jpg)

# 分区表与执行计划

## 单分区单子分区全扫描
![image](http://wx3.sinaimg.cn/mw690/006Pv9mtly1fn3szh8e8ij30po0brdg5.jpg)
## 单分区多子分区全扫描
![image] (http://wx2.sinaimg.cn/mw690/006Pv9mtly1fn3szobgubj30q30bm3yv.jpg)
## 多分区单子分区
![image](http://wx4.sinaimg.cn/mw690/006Pv9mtly1fn3szsyin9j30qg0k0750.jpg)
## 多分区多子分区
![image](http://wx1.sinaimg.cn/mw690/006Pv9mtly1fn3szxn4xzj30q30bb3yw.jpg)
## 多分区多全分区
![image](http://wx1.sinaimg.cn/mw690/006Pv9mtly1fn3t02pb3vj30q20bfq3a.jpg)

## 全分区全子分区
![image](http://wx2.sinaimg.cn/mw690/006Pv9mtly1fn3t4yotnuj30nq0anaac.jpg)

## 绑定变量与分区表

![image](http://wx3.sinaimg.cn/mw690/006Pv9mtly1fn3t5z74r4j30ma0buglz.jpg)

## 子查询过滤分区
![image](http://wx3.sinaimg.cn/mw690/006Pv9mtly1fn3t64jxaoj30ns0fn0tf.jpg)

## 关联列过滤分区
![image](http://wx4.sinaimg.cn/mw690/006Pv9mtly1fn3t6cr0luj30oi0eot9c.jpg)

## 星型转换
![image](http://wx3.sinaimg.cn/mw690/006Pv9mtly1fn3t8t43qtj30kj0md0tz.jpg)
