---
layout: post
title: hive表操作
date: 2020-08-30
categories: blog
tags: [大数据]
description: hive表相关操作
---
## 建表语句释义


语法帮助

```
CREATE [TEMPORARY] [EXTERNAL] TABLE [IF NOT EXISTS] [db_name.]table_name    -- (Note: TEMPORARY available in Hive 0.14.0 and later)
  [(col_name data_type [column_constraint_specification] [COMMENT col_comment], ... [constraint_specification])]
  [COMMENT table_comment]
  [PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)]
  [CLUSTERED BY (col_name, col_name, ...) [SORTED BY (col_name [ASC|DESC], ...)] INTO num_buckets BUCKETS]
  [SKEWED BY (col_name, col_name, ...)                  -- (Note: Available in Hive 0.10.0 and later)]
     ON ((col_value, col_value, ...), (col_value, col_value, ...), ...)
     [STORED AS DIRECTORIES]
  [
   [ROW FORMAT row_format] 
   [STORED AS file_format]
     | STORED BY 'storage.handler.class.name' [WITH SERDEPROPERTIES (...)]  -- (Note: Available in Hive 0.6.0 and later)
  ]
  [LOCATION hdfs_path]
  [TBLPROPERTIES (property_name=property_value, ...)]   -- (Note: Available in Hive 0.6.0 and later)
  [AS select_statement];   -- (Note: Available in Hive 0.5.0 and later; not supported for external tables)
 
CREATE [TEMPORARY] [EXTERNAL] TABLE [IF NOT EXISTS] [db_name.]table_name
  LIKE existing_table_or_view_name
  [LOCATION hdfs_path];
 
data_type
  : primitive_type
  | array_type
  | map_type
  | struct_type
  | union_type  -- (Note: Available in Hive 0.7.0 and later)
 
primitive_type
  : TINYINT
  | SMALLINT
  | INT
  | BIGINT
  | BOOLEAN
  | FLOAT
  | DOUBLE
  | DOUBLE PRECISION -- (Note: Available in Hive 2.2.0 and later)
  | STRING
  | BINARY      -- (Note: Available in Hive 0.8.0 and later)
  | TIMESTAMP   -- (Note: Available in Hive 0.8.0 and later)
  | DECIMAL     -- (Note: Available in Hive 0.11.0 and later)
  | DECIMAL(precision, scale)  -- (Note: Available in Hive 0.13.0 and later)
  | DATE        -- (Note: Available in Hive 0.12.0 and later)
  | VARCHAR     -- (Note: Available in Hive 0.12.0 and later)
  | CHAR        -- (Note: Available in Hive 0.13.0 and later)
 
array_type
  : ARRAY < data_type >
 
map_type
  : MAP < primitive_type, data_type >
 
struct_type
  : STRUCT < col_name : data_type [COMMENT col_comment], ...>
 
union_type
   : UNIONTYPE < data_type, data_type, ... >  -- (Note: Available in Hive 0.7.0 and later)
 
row_format
  : DELIMITED [FIELDS TERMINATED BY char [ESCAPED BY char]] [COLLECTION ITEMS TERMINATED BY char]
        [MAP KEYS TERMINATED BY char] [LINES TERMINATED BY char]
        [NULL DEFINED AS char]   -- (Note: Available in Hive 0.13 and later)
  | SERDE serde_name [WITH SERDEPROPERTIES (property_name=property_value, property_name=property_value, ...)]
 
file_format:
  : SEQUENCEFILE
  | TEXTFILE    -- (Default, depending on hive.default.fileformat configuration)
  | RCFILE      -- (Note: Available in Hive 0.6.0 and later)
  | ORC         -- (Note: Available in Hive 0.11.0 and later)
  | PARQUET     -- (Note: Available in Hive 0.13.0 and later)
  | AVRO        -- (Note: Available in Hive 0.14.0 and later)
  | JSONFILE    -- (Note: Available in Hive 4.0.0 and later)
  | INPUTFORMAT input_format_classname OUTPUTFORMAT output_format_classname
 
column_constraint_specification:
  : [ PRIMARY KEY|UNIQUE|NOT NULL|DEFAULT [default_value]|CHECK  [check_expression] ENABLE|DISABLE NOVALIDATE RELY/NORELY ]
 
default_value:
  : [ LITERAL|CURRENT_USER()|CURRENT_DATE()|CURRENT_TIMESTAMP()|NULL ] 
 
constraint_specification:
  : [, PRIMARY KEY (col_name, ...) DISABLE NOVALIDATE RELY/NORELY ]
    [, PRIMARY KEY (col_name, ...) DISABLE NOVALIDATE RELY/NORELY ]
    [, CONSTRAINT constraint_name FOREIGN KEY (col_name, ...) REFERENCES table_name(col_name, ...) DISABLE NOVALIDATE 
    [, CONSTRAINT constraint_name UNIQUE (col_name, ...) DISABLE NOVALIDATE RELY/NORELY ]
    [, CONSTRAINT constraint_name CHECK [check_expression] ENABLE|DISABLE NOVALIDATE RELY/NORELY ]

```
[EXTERNAL]：声明为外部表，往往在该表需要被多个工具共享时声明，外部表删表不会删数据，只会删元数据。

col_name datatype：data_type 一定要严谨定义，避免 bigint、double 等等统统用 string
的偷懒做法，否则不知某天数据就出错了。

[if not exists]：创建时不指定，若存在同名表则返回出错。指定此选项，若存在同名表忽略后续，不存在则创建。

[DEFAULT value]：指定列的默认值，当INSERT操作不指定该列时，该列写入默认值。

[PARTITIONED BY]：指定表的分区字段，当利用分区字段对表进行分区时，新增分区、更新分区内数据和读取分区数据均不需做全表扫描，可以提高处理效率。

[LIFECYCLE]：是表的生命周期，分区表则每个分区的生命周期与表生命周期相同

[AS select_statement]：意味着可直接跟 select 语句插入数据
```
0: jdbc:hive2://localhost:10000/> create table if not exists dept
. . . . . . . . . . . . . . . . > (deptno int comment 'department number ',
. . . . . . . . . . . . . . . . >  dname  string comment 'department name ',
. . . . . . . . . . . . . . . . >  loc    string comment 'department location' )
. . . . . . . . . . . . . . . . >  comment 'department info table '
. . . . . . . . . . . . . . . . >  row format delimited
. . . . . . . . . . . . . . . . >    fields terminated by '\t'
. . . . . . . . . . . . . . . . >    collection items terminated by ','
. . . . . . . . . . . . . . . . >    map keys terminated by ':'
. . . . . . . . . . . . . . . . >    lines terminated by '\n';

```

row format delimited ： 用来声明各类分隔符

fields terminated by '\t' : 声明表中列间分隔符

collection items terminated by ',' ：声明复杂类型元素分隔符

map keys terminated by ':'：声明 MAP类型分隔符

lines terminated by '\n';：声明行间分隔符 


## 数据类型





## 加载数据

### 从本地加载数据

```

0: jdbc:hive2://localhost:10000/> load data local inpath '/root/data.txt' into table dept ;
No rows affected (1.725 seconds)
0: jdbc:hive2://localhost:10000/> select * frrom dept ;
Error: Error while compiling statement: FAILED: ParseException line 1:9 missing EOF at 'frrom' near '*' (state=42000,code=40000)
0: jdbc:hive2://localhost:10000/> select * from dept ;
+--------------+-------------+-----------+--+
| dept.deptno  | dept.dname  | dept.loc  |
+--------------+-------------+-----------+--+
| 1            | IT          | BJ        |
| 2            | MARK        | SH        |
| 3            | SALES       | SZ        |
+--------------+-------------+-----------+--+
3 rows selected (1.877 seconds)


```


### 从HDFS加载数据

```sql 

0: jdbc:hive2://localhost:10000/> load data inpath '/user/hive/warehouse/d2.db/dept/data.txt' into table  dept_2 ;
No rows affected (0.852 seconds)
0: jdbc:hive2://localhost:10000/> select * from dept_2 ;
+----------------+---------------+-------------+--+
| dept_2.deptno  | dept_2.dname  | dept_2.loc  |
+----------------+---------------+-------------+--+
| 1              | IT            | BJ          |
| 2              | MARK          | SH          |
| 3              | SALES         | SZ          |
+----------------+---------------+-------------+--+
3 rows selected (0.336 seconds)

```
### 查看加载后的数据 

```

[root@node1 ~]#  hdfs dfs -ls -R /user/hive/warehouse/
drwxr-xr-x   - root supergroup          0 2020-08-25 09:18 /user/hive/warehouse/d2.db
drwxr-xr-x   - root supergroup          0 2020-08-25 09:21 /user/hive/warehouse/d2.db/dept
drwxr-xr-x   - root supergroup          0 2020-08-25 09:21 /user/hive/warehouse/d2.db/dept_2
-rwxr-xr-x   3 root supergroup         29 2020-08-25 05:57 /user/hive/warehouse/d2.db/dept_2/data.txt
drwxr-xr-x   - root supergroup          0 2020-08-24 16:32 /user/hive/warehouse/d2.db/t1
-rwxr-xr-x   3 root supergroup          2 2020-08-24 16:32 /user/hive/warehouse/d2.db/t1/000000_0
drwxr-xr-x   - root supergroup          0 2020-08-25 05:46 /user/hive/warehouse/d2.db/t3
drwxr-xr-x   - root supergroup          0 2020-08-25 05:47 /user/hive/warehouse/d2.db/t4
drwxr-xr-x   - root supergroup          0 2020-08-22 02:58 /user/hive/warehouse/dao_t1
-rwxr-xr-x   3 root supergroup         11 2020-08-22 02:57 /user/hive/warehouse/dao_t1/000000_0


```

诡异的事情出现了，data.txt从dept目录下，迁移至dept2目录下。
我们查看一下dept表，发现数据也没有了。

```

0: jdbc:hive2://localhost:10000/> select * from dept ;
+--------------+-------------+-----------+--+
| dept.deptno  | dept.dname  | dept.loc  |
+--------------+-------------+-----------+--+
+--------------+-------------+-----------+--+
No rows selected (0.953 seconds)

```

## 分区表的创建

```
0: jdbc:hive2://localhost:10000/> create table  dao_part (
. . . . . . . . . . . . . . . . > userid string  ,
. . . . . . . . . . . . . . . . > login_channel int ,
. . . . . . . . . . . . . . . . > device_type string
. . . . . . . . . . . . . . . . > ) PARTITIONED  by (month int , day int ) ;
No rows affected (0.563 seconds)

```

###  向分区表中插入数据

```sql

0: jdbc:hive2://localhost:10000/> insert into dao_part   PARTITION (month=202008, day=03)  values
. . . . . . . . . . . . . . . . > ('user1',2,'ios' ) ;
WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
No rows affected (31.873 seconds)
0: jdbc:hive2://localhost:10000/> insert into dao_part   PARTITION (month=201907, day=07)  values
. . . . . . . . . . . . . . . . > ('user1',2,'anroid' ) ;
WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
No rows affected (22.594 seconds)
0: jdbc:hive2://localhost:10000/> select * from dao_part ;
+------------------+-------------------------+-----------------------+-----------------+---------------+--+
| dao_part.userid  | dao_part.login_channel  | dao_part.device_type  | dao_part.month  | dao_part.day  |
+------------------+-------------------------+-----------------------+-----------------+---------------+--+
| user1            | 2                       | anroid                | 201907          | 7             |
| user1            | 2                       | ios                   | 202008          | 3             |
+------------------+-------------------------+-----------------------+-----------------+---------------+--+

```

### 查看分区

```sql

0: jdbc:hive2://localhost:10000/> SHOW PARTITIONS dao_part ;
+---------------------+--+
|      partition      |
+---------------------+--+
| month=201907/day=7  |
| month=202008/day=3  |
+---------------------+--+

```

### 加载数据值分区表

制作测试数据文本

```

[root@node1 ~]# cat -A data_user.txt
user3^A3^Aweb$
user4^A4^Amac$


```

加载数据

```sql

0: jdbc:hive2://localhost:10000/> load data local inpath '/root/data_user.txt' into table dao_part  partition(month=202008,day=03);load data local inpath '/root/data_user.txt' into table dao_part  partition(month=202008,day=03);
No rows affected (1.26 seconds)
No rows affected (1.229 seconds)

```

查看数据

```sql

0: jdbc:hive2://localhost:10000/> select * from dao_part ;
+------------------+-------------------------+-----------------------+-----------------+---------------+--+
| dao_part.userid  | dao_part.login_channel  | dao_part.device_type  | dao_part.month  | dao_part.day  |
+------------------+-------------------------+-----------------------+-----------------+---------------+--+
| user1            | 2                       | anroid                | 201907          | 7             |
| user1            | 2                       | ios                   | 202008          | 3             |
| user3            | 3                       | web                   | 202008          | 3             |
| user4            | 4                       | mac                   | 202008          | 3             |
| user3            | 3                       | web                   | 202008          | 3             |
| user4            | 4                       | mac                   | 202008          | 3             |
+------------------+-------------------------+-----------------------+-----------------+---------------+--+
6 rows selected (0.358 seconds)
```

查看相关路径


```sql

[root@node1 ~]# hdfs dfs -ls -R /user/hive/warehouse/d2.db/dao_part/month=202008/day=3/data_user_copy_1.txt
-rwxr-xr-x   3 root supergroup         24 2020-08-25 11:31 /user/hive/warehouse/d2.db/dao_part/month=202008/day=3/data_user_copy_1.txt
[root@node1 ~]# hdfs dfs -cat  /user/hive/warehouse/d2.db/dao_part/month=202008/day=3/data_user_copy_1.txt
user33web
user44mac

```

## 桶表

如果说分区表对应Oracle数据库的列表分区表，那么桶表就对应Oracle中的哈希分区表。

### 建立桶表 

```sql

0: jdbc:hive2://localhost:10000/>  create table  dao_part_bucket (
. . . . . . . . . . . . . . . . >  userid string  ,
. . . . . . . . . . . . . . . . >  login_channel int ,
. . . . . . . . . . . . . . . . >  device_type string
. . . . . . . . . . . . . . . . >  ) clustered  by (login_channel )
. . . . . . . . . . . . . . . . >  into 3 buckets  ;
No rows affected (0.212 seconds)

```

### 查看桶表 

```sql

0: jdbc:hive2://localhost:10000/> show  create table dao_part_bucket ;
+-----------------------------------------------------------------+--+
|                         createtab_stmt                          |
+-----------------------------------------------------------------+--+
| CREATE TABLE `dao_part_bucket`(                                 |
|   `userid` string,                                              |
|   `login_channel` int,                                          |
|   `device_type` string)                                         |
| CLUSTERED BY (                                                  |
|   login_channel)                                                |
| INTO 3 BUCKETS                                                  |
| ROW FORMAT SERDE                                                |
|   'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe'          |
| STORED AS INPUTFORMAT                                           |
|   'org.apache.hadoop.mapred.TextInputFormat'                    |
| OUTPUTFORMAT                                                    |
|   'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'  |
| LOCATION                                                        |
|   'hdfs://mycluster/user/hive/warehouse/d2.db/dao_part_bucket'  |
| TBLPROPERTIES (                                                 |
|   'COLUMN_STATS_ACCURATE'='{\"BASIC_STATS\":\"true\"}',         |
|   'numFiles'='0',                                               |
|   'numRows'='0',                                                |
|   'rawDataSize'='0',                                            |
|   'totalSize'='0',                                              |
|   'transient_lastDdlTime'='1598372677')                         |
+-----------------------------------------------------------------+--+
22 rows selected (0.634 seconds)

```


### 插入数据


```sql

0: jdbc:hive2://localhost:10000/> insert into dao_part_bucket (userid,login_channel,device_type) select userid,login_channel,device_type from dao_part ;
WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
No rows affected (64.268 seconds)

0: jdbc:hive2://localhost:10000/> select * from dao_part_bucket ;
+-------------------------+--------------------------------+------------------------------+--+
| dao_part_bucket.userid  | dao_part_bucket.login_channel  | dao_part_bucket.device_type  |
+-------------------------+--------------------------------+------------------------------+--+
| user3                   | 3                              | web                          |
| user3                   | 3                              | web                          |
| user4                   | 4                              | mac                          |
| user4                   | 4                              | mac                          |
| user1                   | 2                              | ios                          |
| user1                   | 2                              | anroid                       |
+-------------------------+--------------------------------+------------------------------+--+

```


### 查看表目录

```sql

[root@node1 ~]# hdfs dfs -ls -R /user/hive/warehouse/d2.db/dao_part_bucket
-rwxr-xr-x   3 root supergroup         24 2020-08-25 12:40 /user/hive/warehouse/d2.db/dao_part_bucket/000000_0
-rwxr-xr-x   3 root supergroup         24 2020-08-25 12:39 /user/hive/warehouse/d2.db/dao_part_bucket/000001_0
-rwxr-xr-x   3 root supergroup         27 2020-08-25 12:40 /user/hive/warehouse/d2.db/dao_part_bucket/000002_0

```
#### 查看各个文件内的数据

```sql
[root@node1 ~]# hdfs dfs -cat /user/hive/warehouse/d2.db/dao_part_bucket/000000_0
user33web
user33web
[root@node1 ~]# hdfs dfs -cat /user/hive/warehouse/d2.db/dao_part_bucket/000001_0
user44mac
user44mac
[root@node1 ~]# hdfs dfs -cat /user/hive/warehouse/d2.db/dao_part_bucket/000002_0
user12ios
user12anroid


```