---
layout: post
title: Oracle 官方Undo 脚本整理
date: 2014-09-23
categories: blog
tags: [oracle]
description: Oracle 官方Undo 脚本整理
---

Oracle 提供了一套脚本来监控undo ，接下来的时间里 我们逐个学习这些脚本

## undoparameters.sql 
```sql
set lines 120  
set pages 999  
clear col  
  
set termout off  
set trimout on  
set trimspool on  
  
connect / as sysdba  
alter session set nls_date_format='dd-Mon-yyyy hh24:mi';  
  
spool undoparameters.out  
  
prompt  
prompt  ############## RUNTIME ##############   
prompt  
  
col rdate head "Run Time"  
  
select sysdate rdate from dual;  
  
col inst_id format 999 head "Instance #"  
col Parameter format a35 wrap  
col "Session Value" format a25 wrapped  
col "Instance Value" format a25 wrapped  
  
prompt  
prompt  ############## PARAMETERS ##############   
prompt  
  
select  a.inst_id, a.ksppinm  "Parameter",  
             b.ksppstvl "Session Value",  
             c.ksppstvl "Instance Value"  
      from x$ksppi a, x$ksppcv b, x$ksppsv c  
     where a.indx = b.indx and a.indx = c.indx  
       and a.inst_id=b.inst_id and b.inst_id=c.inst_id  
       and a.ksppinm in ('_undo_autotune', '_smu_debug_mode',  
                         '_highthreshold_undoretention',  
                'undo_tablespace','undo_retention','undo_management')  
order by 2;  
  
  
/*_undo_autotune  是否开启undo_autotune(undo 自动调整)  
--_smu_debug_mode 具体内容Oracle没有给出解释   
--                metalink只给出如下几个值   
--                alter system set "_smu_debug_mode" = 45;       (240746.1)  
                  只适用于10.1 设置后可手工指定session 回滚段   
                  alter system set "_smu_debug_mode" = 33554432  （420525.1）     
                  更改_undo_autotune计算策略，将使用基于最大SQL执行时间的而不是依据空间设置  
  undo_retention  设定的值，开启_undo_autotune 后不在具有实际意义 而 _undo_autotune 默认是开启的  
  undo_tablespace  当前undo 表空间是哪个  
*/  
                     
spool off  
set termout on  
set trimout off  
set trimspool off  
clear col  
```

## UndoDatafiles.sql

```sql
set lines 120  
set pages 999  
clear col  
  
set termout off  
set trimout on  
set trimspool on  
  
connect / as sysdba  
alter session set nls_date_format='dd-Mon-yyyy hh24:mi';  
  
spool undodatafiles.out  
  
prompt  
prompt  ############## RUNTIME ##############   
prompt  
  
col rdate head "Run Time"  
  
select sysdate rdate from dual;  
  
prompt   
prompt  ############## DATAFILES ##############   
prompt   
  
col retention head "Retention"  
col tablespace_name format a30 head "TBSP Name"  
col file_id format 999 head "File #"  
col a format 999,999,999,999,999 head "Bytes Alloc (MB)"  
col b format 999,999,999,999,999 head "Max Bytes Used (MB)"  
col autoextensible head "Auto|Ext"  
col extent_management head "Ext Mngmnt"  
col allocation_type head "Type"  
col segment_space_management head "SSM"  
  
select tablespace_name,           --undo表空间名称  
       file_id,                   --文件ID  
       sum(bytes)/1024/1024 a,    --当前undo表空间大小   
       sum(maxbytes)/1024/1024 b,  --最大大小  
       autoextensible             --是否开启自动扩展  
from dba_data_files  
where tablespace_name in (select tablespace_name from dba_tablespaces  
   where retention like '%GUARANTEE' )  
group by file_id, tablespace_name, autoextensible  
order by tablespace_name  
/  
  
/*  
  
  
*/  
spool off  
set termout on  
set trimout off  
set trimspool off  
clear col  
```

## UndoUsage.sql 关于UNDO使用率 使用状态的脚本

```sql

set lines 120  
set pages 999  
clear col  
  
set termout off  
set trimout on  
set trimspool on  
  
connect / as sysdba  
alter session set nls_date_format='dd-Mon-yyyy hh24:mi';  
  
spool undousage.out  
  
prompt  
prompt  ############## RUNTIME ##############   
prompt  
  
col rdate head "Run Time"  
  
select sysdate rdate from dual;  
  
prompt   
prompt  ############## IN USE Undo Data ##############   
prompt   
--undo 使用率  
select   
((select (nvl(sum(bytes),0))   
from dba_undo_extents   
where tablespace_name in (select tablespace_name from dba_tablespaces  
   where retention like '%GUARANTEE' )  
and status in ('ACTIVE','UNEXPIRED')) *100) /   
(select sum(bytes)   
from dba_data_files   
where tablespace_name in (select tablespace_name from dba_tablespaces  
   where retention like '%GUARANTEE' )) "PCT_INUSE"   
from dual;   
  
--undo 表空间信息  
select tablespace_name,   --undo表空间名称  
       extent_management, --extent 管理模式    
       allocation_type,   --分配方式  SYSTEM UNIFORM USER  
       segment_space_management,  --段管理方式  
       retention          -- GUARANTEE  undo 表空间声明为GUARANTEE   强制保留undo信息到undo_RETENTION  
                          -- GUARANTEE  undo 表空间声明为NOGUARANTEE 尽量保留undo信息到undo_RETENTION  
                          -- NOT APPLY  非undo 表空间   
from dba_tablespaces where retention like '%GUARANTEE'   
/  
  
col c format 999,999,999,999 head "Sum of Free"  
  
select (nvl(sum(bytes),0)) c from dba_free_space --剩余空间  
where tablespace_name in  
(select tablespace_name from dba_tablespaces where retention like '%GUARANTEE')  
/  
  
col d format 999,999,999,999 head "Total Bytes"  
  
select sum(bytes) d from dba_data_files  --表空间大小  
where tablespace_name in  
(select tablespace_name from dba_tablespaces where retention like '%GUARANTEE')  
/  
  
  
PROMPT  
PROMPT  ############## UNDO SEGMENTS ##############   
PROMPT  
  
col status head "Status"  
col z format 999,999 head "Total Extents"  
break on report  
compute sum on report of z  
  
select status, count(*) z  --表各状态空间区数量  
from dba_undo_extents   
group by status  
/  
  
col z format 999,999 head "Undo Segments"    
  
select status, count(*) z   --表空间各状态回滚段数量  
from dba_rollback_segs  
group by status  
/  
  
clear break  
clear compute  
  
prompt   
prompt  ############## CURRENT STATUS OF SEGMENTS  ##############   
prompt  ##############   SNAPSHOT IN TIME INFO     ##############  
prompt  ##############(SHOWS CURRENT UNDO ACTIVITY)##############  
prompt   
  
  
col segment_name format a30 head "Segment Name"  
col "ACT BYTES" format 999,999,999,999 head "Active Bytes"  
col "UNEXP BYTES" format 999,999,999,999 head "Unexpired Bytes"  
col "EXP BYTES" format 999,999,999,999 head "Expired Bytes"  
  
select segment_name,  
       nvl(sum(act), 0) "ACT BYTES",  
       nvl(sum(unexp), 0) "UNEXP BYTES",  
       nvl(sum(exp), 0) "EXP BYTES"  
  from (select segment_name, nvl(sum(bytes), 0) act, 00 unexp, 00 exp  
          from dba_undo_extents  
         where status = 'ACTIVE'  
         group by segment_name  
        union  
        select segment_name, 00 act, nvl(sum(bytes), 0) unexp, 00 exp  
          from dba_undo_extents  
         where status = 'UNEXPIRED'  
         group by segment_name  
        union  
        select segment_name, 00 act, 00 unexp, nvl(sum(bytes), 0) exp  
          from dba_undo_extents  
         where status = 'EXPIRED'  
         group by segment_name)  
 group by segment_name  
 order by 1  
/  
--我自己改写了一下  
select segment_name,  
       sum(case  
             when status = 'ACTIVE' then  
              bytes  
             else  
              0  
           end) "ACT BYTES",  --活动状态的区大小  
       sum(case  
             when status = 'UNEXPIRED' then  
              bytes  
             else  
              0  
           end) "UNEXP BYTES" ,  --未过期区大小  
       sum(case  
             when status = 'EXPIRED' then  
              bytes  
             else  
              0  
           end) "EXP BYTES" --过期区大小  
  from dba_undo_extents  
 group by segment_name  
 order by 1   
  
prompt   
prompt  ############## UNDO SPACE USAGE ##############   
prompt   
  
col usn format 999,999 head "Segment#"  
col shrinks format 999,999,999 head "Shrinks"  
col aveshrink format 999,999,999 head "Avg Shrink Size"  
  
select usn,         --回滚段编号  
       shrinks,     --回滚段收缩次数  
       aveshrink    --平均每次收缩的大小   
  from v$rollstat  
/  
spool off  
set termout on  
set trimout off  
set trimspool of
```

## 下面这个脚本是显示哪些活动事务占用undo空间

```sql

set lines 200  
set pages 999  
clear col  
  
  
set termout off  
set trimout on  
set trimspool on  
  
  
  
  
REM  
REM  Current transactions  
REM  
REM  Will show only last transaction by a user  
REM  
REM  May need to use 786472.1 for better picture   
REM  of activity  
  
  
connect / as sysdba  
alter session set nls_date_format='dd-Mon-yyyy hh24:mi';  
  
  
  
  
col username format a10 wrapped heading "User"  
col name format a22 wrapped heading "Undo Segment Name"  
col xidusn heading "Undo|Seg #"  
col xidslot heading "Undo|Slot #"  
col xidsqn heading "Undo|Seq #"  
col ubafil heading "File #"  
col ubablk heading "Block #"  
col start_time format a10 word_wrapped heading "Started"  
col status format a8 heading "Status"  
col blk format 999,999,999 heading "KBytes"  
col used_urec heading "Rows"  
  
  
col username format a10  
col OSUSER format a10  
col MACHINE format a10  
col SQL_ID format a10  
col SQL_TEXT format a30  
  
  
spool undoactivity.out  
  
  
prompt  
prompt  ############## RUNTIME ##############   
prompt  
  
  
col rdate head "Run Time"  
  
  
select sysdate rdate from dual;  
  
  
prompt  
prompt  ############## Current Uncommitted Transactions ##############   
prompt  
  
  
select start_time,  --事务起始时间  
       username,    --用户名  
       s.MACHINE,   --机器名称  
       s.OSUSER,    --登录名  
       r.name,      --回滚段名称  
       ubafil,      --Undo block address (UBA) filenum  
       ubablk,      --UBA block number  
       t.status,    --回话状态  
       (used_ublk * p.value) / 1024 blk, --使用的回滚段空间  
       used_urec,   --使用的undo 记录 ,  
       s1.SQL_ID,    --sql_id  
       s1.SQL_TEXT   --sql文本  
  from v$transaction t, v$rollname r, v$session s, v$parameter p,v$sql s1  
 where xidusn = usn  
   and s.saddr = t.ses_addr  
   and p.name = 'db_block_size'   
   and s.SADDR=s1.ADDRESS(+)  
 order by 1;  
  
  
set termout on  
set trimout off  
set trimspool off  
clear col  

```

##  并发与回滚段统计

```sql


set lines 120  
set pages 999  
clear col  
  
set termout off  
set trimout on  
set trimspool on  
  
connect / as sysdba  
alter session set nls_date_format='dd-hh24:mi';  
  
spool undohistoryinfo.out  
  
prompt  
prompt  ############## RUNTIME ##############   
prompt  
  
col rdate head "Run Time"  
  
select sysdate rdate from dual;  
  
prompt   
prompt  ############## HISTORICAL DATA ##############   
prompt   
  
col x format 999,999 head "Max Concurrent|Last 7 Days"  
col y format 999,999 head "Max Concurrent|Since Startup"  
--最大的并发  
select max(maxconcurrency) x from v$undostat  
/  
--历史最大并发  
select max(maxconcurrency) y from sys.wrh$_undostat  
/  
  
col i format 999,999 head "1555 Errors"  
col j format 999,999 head "Undo Space Errors"  
--出现01555的总次数  
select sum(ssolderrcnt) i from v$undostat  
where end_time > sysdate-2  
/  
--出现no spaace错误的次数  
select sum(nospaceerrcnt) j from v$undostat  
where end_time > sysdate-2  
/  
clear break  
clear compute  
  
prompt   
prompt  ############## CURRENT STATUS OF SEGMENTS  ##############   
prompt  ##############   SNAPSHOT IN TIME INFO     ##############  
prompt  ##############(SHOWS CURRENT UNDO ACTIVITY)##############  
prompt   
  
col segment_name format a30 head "Segment Name"  
col "ACT BYTES" format 999,999,999,999 head "Active Bytes"  
col "UNEXP BYTES" format 999,999,999,999 head "Unexpired Bytes"  
col "EXP BYTES" format 999,999,999,999 head "Expired Bytes"  
  
select segment_name,    
       sum(case    
             when status = 'ACTIVE' then    
              bytes    
             else    
              0    
           end) "ACT BYTES",  --活动状态的区大小    
       sum(case    
             when status = 'UNEXPIRED' then    
              bytes    
             else    
              0    
           end) "UNEXP BYTES" ,  --未过期区大小    
       sum(case    
             when status = 'EXPIRED' then    
              bytes    
             else    
              0    
           end) "EXP BYTES" --过期区大小    
  from dba_undo_extents    
 group by segment_name    
 order by 1     
    
  
prompt   
prompt  ############## UNDO SPACE USAGE ##############   
prompt   
  
col usn format 999,999 head "Segment#"  --回滚段编号    
col shrinks format 999,999,999 head "Shrinks"  
col aveshrink format 999,999,999 head "Avg Shrink Size"  
  
    
select usn,         --回滚段编号    
       shrinks,     --回滚段收缩次数    
       aveshrink    --平均每次收缩的大小     
  from v$rollstat    
spool off  
set termout on  
set trimout off  
set trimspool off  
clear col  
```

## 关于undo的统计 包括最长执行的sql语句等

```sql


set lines 120  
set pages 999  
clear col  
  
  
set termout off  
set trimout on  
set trimspool on  
  
  
connect / as sysdba  
alter session set nls_date_format='dd-hh24:mi';  
  
  
spool undostatistics.out  
  
  
prompt  
prompt  ############## RUNTIME ##############   
prompt  
  
  
col rdate head "Run Time"  
  
  
select sysdate rdate from dual;  
  
  
col current_scn head "SCN Now"  
col start_date head "Trans Started"  
col start_scn head "SCN for Trans"  
col ses_addr head "ADDR"  
  
  
prompt   
prompt  ############## Historical V$UNDOSTAT (Last 2 Days) ##############   
prompt   
  
  
  
  
col end_time format a18 Head "Date/Time"  
col maxq format 999,999 head "Query|Maximum|Minutes"  
col maxquerysqlid head "SqlID"  
col undotsn format 999,999 head "TBS"  
col undoblks format 999,999,999 head "Undo|Blocks"  
col txncount format 999,999,999 head "# of|Trans"  
col unexpiredblks format 999,999,999 head "# of Unexpired"  
col expiredblks format 999,999,999 head "# of Expired"  
col tuned format 999,999 head "Tuned Retention|(Minutes)"  
  
  
select BEGIN_TIME,  
       end_time,    
       round(maxquerylen / 60, 0) maxq,  --最长sql执行时间  
       maxquerysqlid,                    --最长sql执行时间的sqlid  
       undotsn,                          --最后活动的undo tablespace 编号  
       undoblks,                         --消耗的undo block size  
       txncount,                         --时段内的食物数量  
       unexpiredblks,                    --未过期的  
       expiredblks,                      --时段内未过期的undp block总数   
       round(tuned_undoretention / 60, 0) Tuned  --auto undoundoretention tuned之后的undo_retention  
  from dba_hist_undostat   
 where end_time > sysdate - 2  
 order by 1  
/  
  
  
prompt   
prompt  ############## RECENT MISSES FOR UNDO (Last 2 Days) ##############   
prompt   
  
  
clear col  
set lines 500  
--四天内语句执行时长>自动调整的undo_retention  
select * from v$undostat where maxquerylen > tuned_undoretention  
and end_time > sysdate-2  
order by 2  
/  
--awr中语句执行时长>自动调整的undo_retention  
select * from sys.wrh$_undostat where maxquerylen > tuned_undoretention  
and end_time > sysdate-2  
order by 2  
/  
  
  
prompt   
prompt  ############## AUTO-TUNING TUNE-DOWN DATA    ##############   
prompt  ############## ROLLBACK DATA (Since Startup) ##############   
prompt   
  
  
col name format a60 head "Name"  
col value format 999,999,999 head "Counters"  
  
  
select name, value from v$sysstat  
where name like '%down retention%' or name like 'une down%'  
or name like '%undo segment%' or name like '%rollback%'  
or name like '%undo record%'  
/  
  
  
prompt   
prompt  ############## Long Running Query History ##############   
prompt   
  
  
col end_time head "Date"  
col maxquerysqlid head "SQL ID"  
col runawayquerysqlid format a15 head "Runaway SQL ID"  
col results format a35 word_wrapped head "Space Issues"  
col status head "Status"  
col newret head "Tuned Down|Retention"  
  
  
select end_time, maxquerysqlid, runawayquerysqlid, status,  
        decode(status,1,'Slot Active',4,'Reached Best Retention',5,'Reached Best Retention',  
                    8, 'Runaway Query',9,'Runaway Query-Active',10,'Space Pressure',  
                   11,'Space Pressure Currently',  
                   16, 'Tuned Down (to undo_retention) due to Space Pressure',   
                   17,'Tuned Down (to undo_retention) due to Space Pressure-Active',  
                   18, 'Tuning Down due to Runaway', 19, 'Tuning Down due to Runaway-Active',  
                   28, 'Runaway tuned down to last tune down value',  
                   29, 'Runaway tuned down to last tune down value',  
                   32, 'Max Tuned Down - Not Auto-Tuning',  
                   33, 'Max Tuned Down - Not Auto-Tuning (Active)',  
                   37, 'Max Tuned Down - Not Auto-Tuning (Active)',   
                   38, 'Max Tuned Down - Not Auto-Tuning',   
                   39, 'Max Tuned Down - Not Auto-Tuning (Active)',   
                   40, 'Max Tuned Down - Not Auto-Tuning',   
                   41, 'Max Tuned Down - Not Auto-Tuning (Active)',   
                   42, 'Max Tuned Down - Not Auto-Tuning',   
                   44, 'Max Tuned Down - Not Auto-Tuning',   
                   45, 'Max Tuned Down - Not Auto-Tuning (Active)',   
                   'Other ('||status||')') Results, spcprs_retention NewRet  
from sys.wrh$_undostat  
where status > 1  
/  
  
  
  
  
  
  
prompt   
prompt  ############## Details on Long Run Queries ##############   
prompt   
  
  
col sql_fulltext head "SQL Text"  
Col sql_id heading "SQL ID"  
  
  
select sql_id,  
       sql_fulltext,   
       last_load_time "Last Load",              --最后一次执行时间  
round(elapsed_time/60/60/24,0) "Elapsed Days"   --执行了多久  
from v$sql where sql_id in   
(select maxquerysqlid from sys.wrh$_undostat   
where status > 1)  
/  
  
  
spool off  
set termout on  
set trimout off  
set trimspool off  
clear col  
```

## undo 压力部分与锁 还有部分的undo统计

```sql

set lines 120  
set pages 999  
clear col  
  
  
set termout off  
set trimout on  
set trimspool on  
  
  
connect / as sysdba  
alter session set nls_date_format='dd-hh24:mi';  
  
  
spool undopressure.out  
  
  
prompt  
prompt  ############## RUNTIME ##############   
prompt  
  
  
col rdate head "Run Time"  
  
  
select sysdate rdate from dual;  
  
  
prompt   
prompt  ############## WAITS FOR UNDO (Since Startup) ##############   
prompt   
  
  
col inst_id head "Instance#"  
col eq_type format a3 head "Enq"  
col total_req# format 999,999,999,999,999,999 head "Total Requests"  
col total_wait# format 999,999 head "Total Waits"  
col succ_req# format 999,999,999,999,999,999 head "Successes"  
col failed_req# format 999,999,999999 head "Failures"  
col cum_wait_time format 999,999,999 head "Cummalitve|Time"  
  
  
select * from v$enqueue_stat where eq_type='US' --Undo Segment  
union  
select * from v$enqueue_stat where eq_type='HW'  --high water   
/  
  
  
prompt   
prompt  ############## LOCKS FOR UNDO ##############   
prompt   
  
  
col addr head "ADDR"  
col KADDR head "KADDR"  
col sid head "Session"  
col osuser format a10 head "OS User"  
col machine format a15 head "Machine"  
col program format a17 head "Program"  
col process format a7 head "Process"  
col lmode head "Lmode"  
col request head "Request"  
col ctime format 9,999 head "Time|(Mins)"  
col block head "Blocking?"  
--当前锁状态  
select /*+ RULE */   
 a.SID,      --session id  
 b.process,  --process id  
 b.OSUSER,   --登录者os username  
 b.MACHINE,  --登录机器  
 b.PROGRAM,  --登录所用终端  
 addr,       -- address of lock state object  
 kaddr,      --lock address   
 lmode,      --LOCK MODE  
 request,    --Lock mode in which the process requests the lock:  
 round(ctime/60/60,0) ctime, block  --发生的时间  
from   
v$lock a,   
v$session b   
where   
a.sid=b.sid  
and a.type='US'  
/  
  
  
prompt   
prompt  ############## TUNED RETENTION HISTORY (Last 2 Days) ##############   
prompt  ##############        LOWEST AND HIGHEST DATA        ##############   
prompt   
  
  
col low format 999,999,999,999 head "Undo Retention|Lowest Tuned Value"  
col high format 999,999,999,999 head "Undo Retention|Highest Tuned Value"  
  
  
--找出两条内tuned_undoretention最短的时段  
select end_time, tuned_undoretention from v$undostat where tuned_undoretention = (  
select min(tuned_undoretention) low  
from v$undostat  
where end_time > sysdate-2)  
/  
--找出两条内tuned_undoretention最长的时段  
select end_time, tuned_undoretention from v$undostat where tuned_undoretention = (  
select max(tuned_undoretention) high  
from v$undostat  
where end_time > sysdate-2)  
/  
  
  
prompt   
prompt  ############## CURRENT TRANSACTIONS ##############   
prompt   
  
  
col sql_text format a40 word_wrapped head "SQL Code"  
  
  
select a.start_date,  --开始时间  
       a.start_scn,   --开始scn  
       a.status,      --事务状态  
       c.sql_text     --sql语句  
  from v$transaction a, v$session b, v$sqlarea c  
 where b.saddr = a.ses_addr  
   and c.address = b.sql_address  
   and b.sql_hash_value = c.hash_value  
/  
  
  
select current_scn from v$database  
/  
  
  
col a format 999,999 head "UnexStolen"  
col b format 999,999 head "ExStolen"  
col c format 999,999 head "UnexReuse"  
col d format 999,999 head "ExReuse"  
  
  
prompt   
prompt  ############## WHO'S STEALING WHAT? (Last 2 Days) ##############   
prompt   
  
  
select unxpstealcnt a,  --试图偷取未过期undo block的次数  
       expstealcnt b,   --试图偷取过期undo block的次数  
       unxpblkreucnt c, --同一回滚段事务重用未过期undo block的数量  
       expblkreucnt d   --同一回滚段事务重用过期  undo block的数量  
from v$undostat  
where (unxpstealcnt > 0 or expstealcnt > 0)  
and end_time > sysdate-2  
/  
  
  
spool off  
set termout on  
set trimout off  
set trimspool off  
clear col  

```

## LOB相关

```sql

REM   List table and column information for LOBs for a specific user  
REM     
REM   UNDO handling with LOBs is not designed for frequent updates  
REM   Frequent updates are best handled with PCTVERSION at 100  
REM   This means you must have a lot of space available in the LOB  
REM     tablespace as all UNDO will be maintained over time.  
REM    
REM   Using RETENTION does not work as expected  
REM   It is set to UNDO_RETENTION at the time of the creation of the  
REM   object.   It does not change over time as UNDO_RETENTION  
REM   or auto-tuned undo retention changes.  
  
set pages 999  
set lines 110  
  
spool lobdata.out  
  
col column_name format a25 head "Column"  
col table_name format a25 head "Table"  
col tablespace_name format a25 head "Tablespace"  
col pctversion format 999 head "PCTVersion %"  
col segment_space_management format a30 head "Space|Mngmnt"  
col retention format 999,999,999 head "Retention"  
--此脚本输出某个用户下的lob，及相应与回滚相关的设置  
select l.table_name,   --表名  
       l.column_name,  --列名   
       l.tablespace_name,  --表空间名称  
       l.pctversion,       --声明旧数据可用占比 ，lob undo机制依靠自身所在表空间  
       l.retention,        --旧数据最大保留时间  
       t.segment_space_management  
  from dba_lobs l, dba_tablespaces t  
 where owner = upper('&USER')  
   and l.tablespace_name = t.tablespace_name  
/  
  
spool off  
```
