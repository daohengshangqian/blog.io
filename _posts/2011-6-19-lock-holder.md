---
layout: post
title:  编写存储过程批量杀死锁进程
date: 2011-6-19
categories: blog
tags: [Oracle]
description: 编写存储过程批量杀死锁进程。
---

```sql

create or replace procedure proc_kill_lock_holder is
begin
  for r in (select sess.sid,
                   sess.serial# s2,
                   lo.oracle_username,
                   lo.os_user_name,
                   ao.object_name,
                   lo.locked_mode
              from v$locked_object lo, dba_objects ao, v$session sess
             where ao.object_id = lo.object_id
               and lo.session_id = sess.sid) LOOP
    EXECUTE IMMEDIATE ' ALTER  alter system kill session ''' || R.SID ||
                      ' , ' || r.s2 || '''';
  END LOOP;

end;


```

需要执行时直接使用匿名块调用

```sql

Begin
 
 proc_kill_lock_holder ;
end ;

```


