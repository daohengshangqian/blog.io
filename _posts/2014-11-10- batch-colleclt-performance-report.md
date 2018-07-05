---
layout: post
title: 批量收集Oracle性能报告
date: 2014-11-10
categories: blog
tags: [Oracle]
description: 批量收集Oracle性能报告
---


## AWR

```sql
declare    
begin_time_date date := to_date('20141130 23:00:00', 'yyyymmdd hh24:mi:ss') ;    
end_time_date   date := to_date('20141203 18:00:00', 'yyyymmdd hh24:mi:ss') ;    
    
    
begin    
  dbms_output.put_line('set linesize 2000');    
  dbms_output.put_line('set pagesize 50000');    
  dbms_output.put_line('set head off');    
  for r in (select *    
              from sys.WRM$_SNAPSHOT t    
             where t.begin_interval_time >begin_time_date                      
               and t.begin_interval_time< end_time_date  
               and t.dbid=3628304518  
             order by t.snap_id, t.instance_number)    
      
   loop    
      
    dbms_output.put_line('spool awrrpt_' || r.instance_number || '_' ||    
                         to_char(r.begin_interval_time+1/24,'mmdd_hh24mi') || '_' ||to_char(r.end_interval_time+1/24,'mmdd_hh24mi') || '.html');    
    dbms_output.put_line('select *     
from table    
(dbms_workload_repository.awr_report_html(' ||    
                         r.dbid || ',' || r.instance_number || ',' ||    
                         r.snap_id || ',' || (r.snap_id + 1) || ',0));');    
    dbms_output.put_line('spool off');    
  end loop;    
end; 
```

## ASH

```sql
declare  
  begin_time_date  date := to_date('20141130 23:00:00',  
                                   'yyyymmdd hh24:mi:ss');  
  end_time_date    date := to_date('20141203 13:00:00',  
                                   'yyyymmdd hh24:mi:ss');  
  interval_minutes number := 60;  
begin  
  dbms_output.put_line('set linesize 2000');  
  dbms_output.put_line('set pagesize 50000');  
  dbms_output.put_line('set head off');  
  for r in (select db.DBID,  
                   ins.INSTANCE_NUMBER,  
                   begin_time_date + interval_minutes * (lv - 1) / 1440 begin_time_date,  
                   begin_time_date + interval_minutes * lv / 1440,  
                   'yyyymmdd hh24:mi:ss' end_time_date,  
                   to_char(begin_time_date + 60 * (lv - 1) / 1440,  
                           'yyyymmddhh24mi') begin_time,  
                   to_char(begin_time_date + 60 * lv / 1440,  
                           'yyyymmddhh24miss') end_time  
              from (select /*+ no_merge*/  
                     level lv  
                      from dual  
                    connect by level <= (end_time_date - begin_time_date) * 60 * 24 /  
                               interval_minutes),  
                   gv$instance ins,  
                   v$database db) loop  
    dbms_output.put_line('spool ashrpt_' || to_char(r.instance_number) || '_' ||  
                         r.begin_time || '-' || r.end_time || '.html');  
    
    dbms_output.put_line('select output from table(dbms_workload_repository.ash_report_html( ' ||  
                         r.dbid || ' , ' || r.instance_number || ' , ' ||  
                         'to_date(' || '''' || r.begin_time ||  
                         ''', ''yyyymmddhh24miss''' || ')' || ' , ' ||  
                         'to_date(' || '''' || r.end_time ||  
                         ''', ''yyyymmddhh24miss''' || ')));');  
    
    dbms_output.put_line('spool off ');  
  end loop;  
end;  
```

## ADDM


```sql
begin_time_date date := to_date('20141109 15:00:00', 'yyyymmdd hh24:mi:ss') ;  
end_time_date   date := to_date('20141109 19:00:00', 'yyyymmdd hh24:mi:ss') ;  
  
  
begin  
  dbms_output.put_line('set linesize 100');  
  dbms_output.put_line('set pagesize 50000');  
  dbms_output.put_line('set head off');  
  dbms_output.put_line('serveroutput ON SIZE UNLIMITED ');  
  dbms_output.put_line('spool addm.txt');  
  for r in (select *  
              from sys.WRM$_SNAPSHOT t  
             where t.begin_interval_time >=begin_time_date                    
               and t.begin_interval_time<= end_time_date   
             order by t.snap_id, t.instance_number)  
    
   loop  
    dbms_output.put_line('declare   
    id number ;  
    name varchar2(100) ;  
    v_dec varchar2(200) :='||'''addm_'||to_char(r.snap_id)||'_'||to_char(r.snap_id+1)||''';');  
    dbms_output.put_line( 'begin ') ;  
    dbms_output.put_line( 'dbms_advisor.create_task(''ADDM'',id,name,v_dec,null);');  
    dbms_output.put_line( 'dbms_advisor.set_task_parameter(name, '||'''START_SNAPSHOT'','||r.snap_id||');') ;  
    dbms_output.put_line( 'dbms_advisor.set_task_parameter(name, ' ||'''END_SNAPSHOT'','||to_char(r.snap_id+1)||');') ;  
    dbms_output.put_line( 'dbms_advisor.set_task_parameter(name, '||'''INSTANCE'','||r.instance_number||');') ;  
    dbms_output.put_line( 'dbms_advisor.set_task_parameter(name, '||'''DB_ID'','||r.dbid||');') ;  
    dbms_output.put_line( 'dbms_advisor.execute_task(name); ') ;  
    dbms_output.put_line('dbms_output.put_line('||'''#spool addm_' || r.instance_number || '_' ||  
                         r.snap_id || '_' || (r.snap_id + 1) || '.txt'');');  
    dbms_output.put_line('dbms_output.put_line('||'''#select dbms_advisor.get_task_report('''''''||'||name||'''''','||'''''TEXT'''',''''TYPICAL'''') from dual ;'');');                           
                           
       
                           
    dbms_output.put_line('dbms_output.put_line(''#spool off'');');  
    dbms_output.put_line('end ;   
    /');  
  end loop;  
  dbms_output.put_line('spool off');  
end;  
  
  
  
  
  
  
--clear  
select 'exec dbms_advisor.delete_task('''||x.task_name||''');'  
from dba_advisor_tasks x  
```
