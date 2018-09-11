---
layout: post
title: MySQL中使用mycat进行实现与读写分离功能
date: 2018-09-12
categories: blog
tags: [MySQL]
description: MySQL中使用mycat实现分库与读写分离功能
---


## 规划


节点名 | IP |角色 
 --- | --- | ---
node1 | 192.168.2.61 | master1  
node2 | 192.168.2.62 | salve1 --> master1  
node3 | 192.168.2.63 | master2
node4 | 192.168.2.64 | mycat  
node5 | 192.168.2.65 | slave2 --> master2 



### 主库建立mycat配置文件中对应的数据库

```sql

mysql> create database dao_db1 ;
Query OK, 1 row affected (0.02 sec)

mysql> create database dao_db2 ;
Query OK, 1 row affected (0.00 sec)


```


## 修改配置文件


### 关键参数说明

- balance

参数值 | 含义
 --- | --- 
 0   |  不开启读写分离机制，所有读操作都发送到当前可用的writeHost上。
 1   |  全部的readHost与stand by writeHost参与select语句的负载均衡，简单的说，当双主双从模式(M1->S1，M2->S2，并且M1与 M2互为主备)，正常情况下，M2,S1,S2都参与select语句的负载均衡。
 2   |  所有读操作都随机的在writeHost、readhost上分发。
 3   |  所有读请求随机的分发到wiriterHost对应的readhost执行，writerHost不负担读压力
 
- writeType

参数值 | 含义
 --- | --- 
 0   | 所有的操作发送到配置的第一个writehost
 1   | 随机发送到配置的所有writehost
 2   | 不执行写操作
 
- switchType

参数值 | 含义
 --- | --- 
 1   |  默认值，表示自动切换
 2   |  基于MySQL主从同步的状态决定是否切换,心跳语句为 show slave status
 3   |  基于MySQL galary cluster的切换机制（适合集群）（1.4.1），心跳语句为 show status like 'wsrep%'。
 
### schema.xml


```sql

[root@node4 bin]# cat  /usr/local/mycat/conf/schema.xml
<?xml version="1.0"?>
<!DOCTYPE mycat:schema SYSTEM "schema.dtd">
<mycat:schema xmlns:mycat="http://io.mycat/">


        <schema name="mycatdb" checkSQLschema="false" sqlMaxLimit="100">
                <!-- auto sharding by id (long) -->
                <table name="dao_table1" dataNode="dn1,dn2" rule="dao-rule1"/>
                <table name="dao_table2" dataNode="dn1,dn2" rule="dao-rule1"/>
        </schema>
        <dataNode name="dn1" dataHost="localhost1" database="dao_db1" />
        <dataNode name="dn2" dataHost="localhost2" database="dao_db2" />


        <dataHost name="localhost1" maxCon="1000" minCon="10" balance="3"
                writeType="0" dbType="mysql" dbDriver="native" switchType="2"  slaveThreshold="100">
                <heartbeat>show slave status</heartbeat>
                <writeHost host="node1" url="192.168.2.61:3306" user="dao" password="dao">
                        <readHost host='node2' url="192.168.2.62:3306"  user="dao" password="dao" />
                </writeHost>
        </dataHost>


        <dataHost name="localhost2" maxCon="1000" minCon="10" balance="3"
                writeType="0" dbType="mysql" dbDriver="native" switchType="2"  slaveThreshold="100">
                <heartbeat>show slave status</heartbeat>
                <writeHost host="node3" url="192.168.2.63:3306" user="dao" password="dao" >
                         <readHost host='node5' url="192.168.2.65:3306"  user="dao" password="dao" />
                </writeHost>
       </dataHost>


</mycat:schema>

```


### server.xml

```sql

[root@node4 bin]# cat  /usr/local/mycat/conf/server.xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- - - Licensed under the Apache License, Version 2.0 (the "License");
        - you may not use this file except in compliance with the License. - You
        may obtain a copy of the License at - - http://www.apache.org/licenses/LICENSE-2.0
        - - Unless required by applicable law or agreed to in writing, software -
        distributed under the License is distributed on an "AS IS" BASIS, - WITHOUT
        WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. - See the
        License for the specific language governing permissions and - limitations
        under the License. -->
<!DOCTYPE mycat:server SYSTEM "server.dtd">
<mycat:server xmlns:mycat="http://io.mycat/">
        <system>
        <property name="useSqlStat">0</property>  <!-- 1为开启实时统计、0为关闭 -->
        <property name="useGlobleTableCheck">0</property>  <!-- 1为开启全加班一致性检测、0为关闭 -->

                <property name="sequnceHandlerType">2</property>
      <!--  <property name="useCompression">1</property>--> <!--1为开启mysql压缩协议-->
        <!--  <property name="fakeMySQLVersion">5.6.20</property>--> <!--设置模拟的MySQL版本号-->
        <!-- <property name="processorBufferChunk">40960</property> -->
        <!--
        <property name="processors">1</property>
        <property name="processorExecutor">32</property>
         -->
                <!--默认为type 0: DirectByteBufferPool | type 1 ByteBufferArena-->
                <property name="processorBufferPoolType">0</property>
                <!--默认是65535 64K 用于sql解析时最大文本长度 -->
                <!--<property name="maxStringLiteralLength">65535</property>-->
                <!--<property name="sequnceHandlerType">0</property>-->
                <!--<property name="backSocketNoDelay">1</property>-->
                <!--<property name="frontSocketNoDelay">1</property>-->
                <!--<property name="processorExecutor">16</property>-->
                <!--
                        <property name="serverPort">8066</property> <property name="managerPort">9066</property>
                        <property name="idleTimeout">300000</property> <property name="bindIp">0.0.0.0</property>
                        <property name="frontWriteQueueSize">4096</property> <property name="processors">32</property> -->
                <!--分布式事务开关，0为不过滤分布式事务，1为过滤分布式事务（如果分布式事务内只涉及全局表，则不过滤），2为不过滤分布式事务,但是记录分布式事务日志-->
                <property name="handleDistributedTransactions">0</property>

                        <!--
                        off heap for merge/order/group/limit      1开启   0关闭
                -->
                <property name="useOffHeapForMerge">1</property>

                <!--
                        单位为m
                -->
                <property name="memoryPageSize">1m</property>

                <!--
                        单位为k
                -->
                <property name="spillsFileBufferSize">1k</property>

                <property name="useStreamOutput">0</property>

                <!--
                        单位为m
                -->
                <property name="systemReserveMemorySize">384m</property>


                <!--是否采用zookeeper协调切换  -->
                <property name="useZKSwitch">true</property>


        </system>

        <!-- 全局SQL防火墙设置 -->
        <!--
        <firewall>
           <whitehost>
              <host host="127.0.0.1" user="mycat"/>
              <host host="127.0.0.2" user="mycat"/>
           </whitehost>
       <blacklist check="false">
       </blacklist>
        </firewall>
        -->
        <user name="user">
                <property name="password">user</property>
                <property name="schemas">mycatdb</property>
                <property name="readOnly">false</property>
        </user>

</mycat:server>


```



### rule.xml


```sql

[root@node4 bin]# cat  /usr/local/mycat/conf/rule.xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- - - Licensed under the Apache License, Version 2.0 (the "License");
        - you may not use this file except in compliance with the License. - You
        may obtain a copy of the License at - - http://www.apache.org/licenses/LICENSE-2.0
        - - Unless required by applicable law or agreed to in writing, software -
        distributed under the License is distributed on an "AS IS" BASIS, - WITHOUT
        WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. - See the
        License for the specific language governing permissions and - limitations
        under the License. -->
<!DOCTYPE mycat:rule SYSTEM "rule.dtd">
<mycat:rule xmlns:mycat="http://io.mycat/">
        <tableRule name="dao-rule1">
                <rule>
                        <columns>c1</columns>
                        <algorithm>mod-long-2</algorithm>
                </rule>
        </tableRule>

        <tableRule name="rule1">
                <rule>
                        <columns>id</columns>
                        <algorithm>func1</algorithm>
                </rule>
        </tableRule>

        <tableRule name="rule2">
                <rule>
                        <columns>user_id</columns>
                        <algorithm>func1</algorithm>
                </rule>
        </tableRule>

        <tableRule name="sharding-by-intfile">
                <rule>
                        <columns>sharding_id</columns>
                        <algorithm>hash-int</algorithm>
                </rule>
        </tableRule>
        <tableRule name="auto-sharding-long">
                <rule>
                        <columns>id</columns>
                        <algorithm>rang-long</algorithm>
                </rule>
        </tableRule>
        <tableRule name="mod-long">
                <rule>
                        <columns>id</columns>
                        <algorithm>mod-long</algorithm>
                </rule>
        </tableRule>
        <tableRule name="sharding-by-murmur">
                <rule>
                        <columns>id</columns>
                        <algorithm>murmur</algorithm>
                </rule>
        </tableRule>
        <tableRule name="crc32slot">
                <rule>
                        <columns>id</columns>
                        <algorithm>crc32slot</algorithm>
                </rule>
        </tableRule>
        <tableRule name="sharding-by-month">
                <rule>
                        <columns>create_time</columns>
                        <algorithm>partbymonth</algorithm>
                </rule>
        </tableRule>
        <tableRule name="latest-month-calldate">
                <rule>
                        <columns>calldate</columns>
                        <algorithm>latestMonth</algorithm>
                </rule>
        </tableRule>

        <tableRule name="auto-sharding-rang-mod">
                <rule>
                        <columns>id</columns>
                        <algorithm>rang-mod</algorithm>
                </rule>
        </tableRule>

        <tableRule name="jch">
                <rule>
                        <columns>id</columns>
                        <algorithm>jump-consistent-hash</algorithm>
                </rule>
        </tableRule>

        <function name="murmur"
                class="io.mycat.route.function.PartitionByMurmurHash">
                <property name="seed">0</property><!-- 默认是0 -->
                <property name="count">2</property><!-- 要分片的数据库节点数量，必须指定，否则没法分片 -->
                <property name="virtualBucketTimes">160</property><!-- 一个实际的数据库节点被映射为这么多虚拟节点，默认是160倍，也就是虚拟节点数是物理节点数的160倍 -->
                <!-- <property name="weightMapFile">weightMapFile</property> 节点的权重，没有指定权重的节点默认是1。以properties文件的格式填写，以从0开始到count-1的整数值也就是节点索引为key，以节点权重值为值。所有权重值必须是正整数，否则以1代替 -->
                <!-- <property name="bucketMapPath">/etc/mycat/bucketMapPath</property>
                        用于测试时观察各物理节点与虚拟节点的分布情况，如果指定了这个属性，会把虚拟节点的murmur hash值与物理节点的映射按行输出到这个文件，没有默认值，如果不指定，就不会输出任何东西 -->
        </function>

        <function name="crc32slot"
                          class="io.mycat.route.function.PartitionByCRC32PreSlot">
                <property name="count">2</property><!-- 要分片的数据库节点数量，必须指定，否则没法分片 -->
        </function>
        <function name="hash-int"
                class="io.mycat.route.function.PartitionByFileMap">
                <property name="mapFile">partition-hash-int.txt</property>
        </function>
        <function name="rang-long"
                class="io.mycat.route.function.AutoPartitionByLong">
                <property name="mapFile">autopartition-long.txt</property>
        </function>
        <function name="mod-long" class="io.mycat.route.function.PartitionByMod">
                <!-- how many data nodes -->
                <property name="count">3</property>
        </function>


        <function name="mod-long-2" class="io.mycat.route.function.PartitionByMod">
                        <!-- how many data nodes -->
                <property name="count">2</property>
        </function>



        <function name="func1" class="io.mycat.route.function.PartitionByLong">
                <property name="partitionCount">8</property>
                <property name="partitionLength">128</property>
        </function>
        <function name="latestMonth"
                class="io.mycat.route.function.LatestMonthPartion">
                <property name="splitOneDay">24</property>
        </function>
        <function name="partbymonth"
                class="io.mycat.route.function.PartitionByMonth">
                <property name="dateFormat">yyyy-MM-dd</property>
                <property name="sBeginDate">2015-01-01</property>
        </function>

        <function name="rang-mod" class="io.mycat.route.function.PartitionByRangeMod">
                <property name="mapFile">partition-range-mod.txt</property>
        </function>

        <function name="jump-consistent-hash" class="io.mycat.route.function.PartitionByJumpConsistentHash">
                <property name="totalBuckets">3</property>
        </function>
</mycat:rule>


```







## 测试

```sql


[mysql@node4 ~]$ mysql -uuser -puser -P8066 -h127.0.0.1  -Dmycatdb
mysql: [Warning] Using a password on the command line interface can be insecure.
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 4
Server version: 5.6.29-mycat-1.6-RELEASE-20161028204710 MyCat Server (OpenCloundDB)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases ;
+----------+
| DATABASE |
+----------+
| mycatdb  |
+----------+
1 row in set (0.00 sec)

mysql> use mycatdb ;
Database changed
mysql> create table dao_table1 (c1 int auto_increment primary key ,c2 int ) ;
Query OK, 0 rows affected (0.07 sec)

mysql> insert into dao_table1(c1,c2)  values(1,1) ;
Query OK, 1 row affected (0.02 sec)

mysql> insert into dao_table1(c1,c2)  values(2,2) ;
Query OK, 1 row affected (0.02 sec)

mysql> insert into dao_table1(c1,c2)  values(3,3) ;
Query OK, 1 row affected (0.03 sec)

mysql> insert into dao_table1(c1,c2)  values(4,4) ;
Query OK, 1 row affected (0.02 sec)


```

### 查看master1 上的数据 

```sql

[mysql@node1 ~]$ mysql -S /tmp/mysql1.sock -uroot -proot
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 15849
Server version: 5.7.21-log MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> select * from  dao_db1.dao_table1 ;
+----+------+
| c1 | c2   |
+----+------+
|  2 |    2 |
|  4 |    4 |
+----+------+
2 rows in set (0.00 sec)


```
### 查看master2 上的数据

```sql

[mysql@node3 ~]$ mysql  -S /tmp/mysql1.sock -uroot -proot
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 305
Server version: 5.7.21-log MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>  select * from  dao_db2.dao_table1 ;
+----+------+
| c1 | c2   |
+----+------+
|  1 |    1 |
|  3 |    3 |
+----+------+
2 rows in set (0.00 sec)


```



