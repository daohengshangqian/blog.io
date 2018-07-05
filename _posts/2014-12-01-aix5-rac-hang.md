---
layout: post
title:   AIX5 RAC另一节点重启后 单一节点挂起
date: 2014-12-01
categories: blog
tags: [Oracle]
description:  AIX5 RAC另一节点重启后 单一节点挂起
---

现象:AIX RAC 一个节点重启后,另一个节点会挂起~

```sql

oracle@P570A1:[/oracle/app/oracle/crs/log/p570a1]$sqlplus/ as sysdba  
   
SQL*Plus: Release 10.2.0.2.0 -Production on Sun Nov 30 11:52:30 2014  
   
Copyright (c) 1982, 2005,Oracle.  All Rights Reserved.  
   
NFS server 192.16.10.23 notresponding still trying  

```

# 问题原因:

```sql
参见MOS文档 When NFS Server Is Down, Oracle Server Freezes With No Errors In Alert Log File (文档 ID 1316251.1)  
  
摘录部分以便解释  
  
Oracle code calls a Unix system call, 'getcwd' to get the current working directory. Then, after that, all the control reverts over to the operating system.  From what we can see, the function 'getcwd' calls 'getwd' which in turn calls 'stat'. Once 'stat' is entered it starts processing directory entries in the order shown below by performing a 'statx' call for each entry:  
  
Oracle源代码调用UNIX方法'getcwd'来获取当前工作目录。  
之后Oracle将控制权还到操作系统。根据上面的trace  
'getcwd' 调用 'getwd'，'getwd'将调用'stat'.方法。  
一旦调用'stat'，将使用'statx'方法获取每个文件夹的状态。  
注：此时/下/archlog1目录不正常。  
所以会使sqlplus hang  

``
