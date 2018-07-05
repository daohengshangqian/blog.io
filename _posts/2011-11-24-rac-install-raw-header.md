---
layout: post
title:  AIX6.1装RAC 遇见裸设备磁盘头问题
date: 2011-11-24
categories: blog
tags: [Oracle]
description: AIX6.1装RAC 遇见裸设备磁盘头问题。
---



问题概述

在AIX 6.1.0.4上安装Oracle 11gr2.0.1

1 安装GRID 建立ASM VOTING_DISK  待选磁盘出现所有的十块磁盘　其中三块作为votingdisk   并且在之后安装GRID的过程中无报错
2 安装GRID后 所需要跑的两个脚本也顺利完成，无报错
3 安装DAtaBase 过程中无报错 
4 准备DBCA之前 建立磁盘组 调用ASM时不显示其他的七块磁盘
5 在图形界面选择扫描磁盘时 久久不能结束  
6 杀掉图形界面后 执行命令 ASMCA 久久不能出现图形界面

节点一alert.log 

```sql

Oracle Database 11g Clusterware Release 11.2.0.1.0 - Production Copyright 1996, 2009 Oracle. All rights reserved.
2011-11-23 18:21:47.213
[client(528606)]CRS-2106:The OLR location /u01/grid/cdata/ora1a.olr is inaccessible. Details in /u01/grid/log/ora1a/client/ocrconfig_528606.log.
2011-11-23 18:21:47.605
[client(528606)]CRS-2101:The OLR was formatted using version 3.
2011-11-23 18:24:34.459
[ohasd(643262)]CRS-2112:The OLR service started on node ora1a.
2011-11-23 18:24:35.701
[ohasd(643262)]CRS-2772:Server 'ora1a' has been assigned to pool 'Free'.
2011-11-23 18:25:27.440
[ctssd(712800)]CRS-2407:The new Cluster Time Synchronization Service reference node is host ora1a.
2011-11-23 18:25:28.139
[ctssd(712800)]CRS-2401:The Cluster Time Synchronization Service started on host ora1a.
2011-11-23 18:26:18.292
[client(528478)]CRS-1006:The OCR location +OCR_VOTE is inaccessible. Details in /u01/grid/log/ora1a/client/ocrconfig_528478.log.
2011-11-23 18:26:18.814
[client(528478)]CRS-1001:The OCR was formatted using version 3.
2011-11-23 18:26:22.063
[crsd(823480)]CRS-1012:The OCR service started on node ora1a.
2011-11-23 18:26:53.382
[ctssd(712800)]CRS-2405:The Cluster Time Synchronization Service on host ora1a is shutdown by user
2011-11-23 18:28:09.445
[ctssd(708716)]CRS-2407:The new Cluster Time Synchronization Service reference node is host ora1a.
2011-11-23 18:28:10.149
[ctssd(708716)]CRS-2401:The Cluster Time Synchronization Service started on host oraa1a.
2011-11-23 18:28:29.728
[crsd(745592)]CRS-1012:The OCR service started on node ora1a.
2011-11-23 18:28:32.019
[crsd(745592)]CRS-1201:CRSD started on node ora1a.
2011-11-23 18:28:32.384
[crsd(745592)]CRS-2772:Server 'ora1a' has been assigned to pool 'Free'.
2011-11-23 18:33:01.100
[crsd(745592)]CRS-2772:Server 'ora1b' has been assigned to pool 'Free'.
2011-11-23 19:58:23.294
[crsd(745592)]CRS-2757:Command 'Stop' timed out waiting for response from the resource 'ora.OCR_VOTE.dg'. Details at (:CRSPE00111:) in /u01/grid/log/ora1a/crsd/crsd.log.
2011-11-23 20:00:10.997
[/u01/grid/bin/orarootagent.bin(860364)]CRS-5822:Agent '/u01/grid/bin/orarootagent_root' disconnected from server. Details at (:CRSAGF00117:) in /u01/grid/log/ora1a/agent/crsd/orarootagent_root/orarootagent_root.log.
2011-11-23 20:00:11.286
[ctssd(708716)]CRS-2402:The Cluster Time Synchronization Service aborted on host ora1a. Details at (:ctsselect_mmg5_1: in /u01/grid/log/ora1a/ctssd/octssd.log.
2011-11-23 20:00:12.634
[ohasd(643262)]CRS-2765:Resource 'ora.asm' has failed on server 'ora1a'.
2011-11-23 20:00:12.643
[ohasd(643262)]CRS-2765:Resource 'ora.evmd' has failed on server 'ora1a'.
2011-11-23 20:00:12.650
[ohasd(643262)]CRS-2765:Resource 'ora.gipcd' has failed on server 'ora1a'.
2011-11-23 20:00:12.671
[ohasd(643262)]CRS-2765:Resource 'ora.mdnsd' has failed on server 'ora1a'.
2011-11-23 20:00:12.685
[ohasd(643262)]CRS-2765:Resource 'ora.gpnpd' has failed on server 'ora1a'.
2011-11-23 20:00:13.682
[ohasd(643262)]CRS-2765:Resource 'ora.diskmon' has failed on server 'ora1a'.
2011-11-23 20:00:14.800
[ohasd(643262)]CRS-2765:Resource 'ora.ctssd' has failed on server 'ora1a'.
2011-11-23 20:04:16.679
[ohasd(450584)]CRS-2112:The OLR service started on node ora1a.
2011-11-23 20:04:17.131
[ohasd(450584)]CRS-8017:location: /etc/oracle/lastgasp has 4 reboot advisory log files, 0 were announced and 0 errors occurred
2011-11-23 20:04:49.079
[ohasd(450584)]CRS-2772:Server 'ora1a' has been assigned to pool 'Free'.
2011-11-23 20:05:13.993
[ctssd(671820)]CRS-2403:The Cluster Time Synchronization Service on host ora1a is in observer mode.
2011-11-23 20:05:14.034
[ctssd(671820)]CRS-2407:The new Cluster Time Synchronization Service reference node is host ora1b.
2011-11-23 20:05:14.667
[ctssd(671820)]CRS-2401:The Cluster Time Synchronization Service started on host ora1a.
2011-11-23 20:05:55.596
[crsd(790666)]CRS-1012:The OCR service started on node ora1a.
2011-11-23 20:06:00.681
[crsd(790666)]CRS-1201:CRSD started on node ora1a.
2011-11-23 20:32:59.576
[ctssd(671820)]CRS-2409:The clock on host ora1a is not synchronous with the mean cluster time. No action has been taken as the Cluster Time Synchronization Service is running in observer mode.
2011-11-23 21:52:46.943
[ctssd(671820)]CRS-2407:The new Cluster Time Synchronization Service reference node is host ora1a.
2011-11-23 21:53:28.675
[ohasd(450584)]CRS-8011:reboot advisory message from host: ora1b, component: mo183213, with time stamp: L-2011-11-23-21:52:55.770
[ohasd(450584)]CRS-8013:reboot advisory message text: clsnomon_status: need to reboot, unexpected failure 8 received from CSS
2011-11-23 21:53:32.803
[crsd(790666)]CRS-5504:Node down event reported for node 'ora1b'.
2011-11-23 21:53:33.135
[crsd(790666)]CRS-2773:Server 'ora1b' has been removed from pool 'Free'.
2011-11-23 22:24:24.399
[crsd(790666)]CRS-2757:Command 'Stop' timed out waiting for response from the resource 'ora.OCR_VOTE.dg'. Details at (:CRSPE00111:) in /u01/grid/log/ora1a/crsd/crsd.log.
2011-11-23 22:27:28.945
[crsd(790666)]CRS-2757:Command 'Clean' timed out waiting for response from the resource 'ora.OCR_VOTE.dg'. Details at (:CRSPE00111:) in /u01/grid/log/ora1a/crsd/crsd.log.
2011-11-23 22:36:53.213
[crsd(790666)]CRS-2757:Command 'Stop' timed out waiting for response from the resource 'ora.OCR_VOTE.dg'. Details at (:CRSPE00111:) in /u01/grid/log/ora1a/crsd/crsd.log.
2011-11-23 22:38:27.053
[ohasd(450584)]CRS-2765:Resource 'ora.asm' has failed on server 'ora1a'.
2011-11-23 22:38:27.333
[crsd(790666)]CRS-2765:Resource 'ora.asm' has failed on server 'ora1a'.
2011-11-23 22:38:34.886
[/u01/grid/bin/orarootagent.bin(811164)]CRS-5822:Agent '/u01/grid/bin/orarootagent_root' disconnected from server. Details at (:CRSAGF00117:) in /u01/grid/log/ora1a/agent/crsd/orarootagent_root/orarootagent_root.log.
2011-11-23 22:38:36.457
[ctssd(671820)]CRS-2405:The Cluster Time Synchronization Service on host ora1a is shutdown by user

```
发现ASM不可用了，还好给出日志信息了


查看\111\ora1a\client\ocrconfig_528478.log

```

Oracle Database 11g Clusterware Release 11.2.0.1.0 - Production Copyright 1996, 2009 Oracle. All rights reserved.
2011-11-23 18:26:13.817: [ OCRCONF][1]ocrconfig starts...
2011-11-23 18:26:13.866: [ OCRCONF][1]Upgrading OCR data
2011-11-23 18:26:16.724: [  OCRASM][1]proprasmo: kgfoCheckMount return [0]. Cannot proceed with dirty open.
2011-11-23 18:26:16.725: [  OCRASM][1]proprasmo: Error in open/create file in dg [OCR_VOTE]
[  OCRASM][1]SLOS : [clsuSlosFormatDiag called with non-error slos.]

2011-11-23 18:26:16.927: [  OCRRAW][1]proprioo: Failed to open [+OCR_VOTE]. Returned proprasmo() with [26]. Marking location as UNAVAILABLE.
2011-11-23 18:26:16.927: [  OCRRAW][1]proprioo: No OCR/OLR devices are usable
2011-11-23 18:26:16.927: [  OCRASM][1]proprasmcl: asmhandle is NULL
2011-11-23 18:26:16.927: [  OCRRAW][1]proprinit: Could not open raw device
2011-11-23 18:26:16.927: [  OCRASM][1]proprasmcl: asmhandle is NULL
2011-11-23 18:26:16.928: [ default][1]a_init:7!: Backend init unsuccessful : [26]
2011-11-23 18:26:16.928: [ OCRCONF][1]Exporting OCR data to [OCRUPGRADEFILE]
2011-11-23 18:26:16.928: [  OCRAPI][1]a_init:7!: Backend init unsuccessful : [33]
2011-11-23 18:26:16.928: [ OCRCONF][1]There was no previous version of OCR. error:[PROC-33: Oracle Cluster Registry is not configured]
2011-11-23 18:26:17.335: [  OCRASM][1]proprasmo: kgfoCheckMount return [0]. Cannot proceed with dirty open.
2011-11-23 18:26:17.335: [  OCRASM][1]proprasmo: Error in open/create file in dg [OCR_VOTE]
[  OCRASM][1]SLOS : [clsuSlosFormatDiag called with non-error slos.]

2011-11-23 18:26:17.521: [  OCRRAW][1]proprioo: Failed to open [+OCR_VOTE]. Returned proprasmo() with [26]. Marking location as UNAVAILABLE.
2011-11-23 18:26:17.521: [  OCRRAW][1]proprioo: No OCR/OLR devices are usable
2011-11-23 18:26:17.521: [  OCRASM][1]proprasmcl: asmhandle is NULL
2011-11-23 18:26:17.521: [  OCRRAW][1]proprinit: Could not open raw device
2011-11-23 18:26:17.521: [  OCRASM][1]proprasmcl: asmhandle is NULL
2011-11-23 18:26:17.521: [ default][1]a_init:7!: Backend init unsuccessful : [26]
2011-11-23 18:26:18.291: [  OCRRAW][1]propriogid:1_2: INVALID FORMAT
2011-11-23 18:26:18.291: [  OCRRAW][1]proprior: Header check from OCR device 0 offset 0 failed (26).
2011-11-23 18:26:18.292: [  OCRRAW][1]ibctx: Failed to read the whole bootblock.  Assumes invalid format.
2011-11-23 18:26:18.292: [  OCRRAW][1]proprinit:problem reading the bootblock or superbloc 22

2011-11-23 18:26:18.491: [  OCRRAW][1]propriogid:1_2: INVALID FORMAT
2011-11-23 18:26:18.814: [  OCRRAW][1]iniconfig:No 92 configuration
2011-11-23 18:26:18.814: [  OCRAPI][1]a_init:6a: Backend init successful
2011-11-23 18:26:18.854: [ OCRCONF][1]Initialized DATABASE keys
2011-11-23 18:26:18.859: [ OCRCONF][1]Successfully set skgfr block 0
2011-11-23 18:26:18.859: [ OCRCONF][1]Exiting [status=success]...

```

此段信息在OTN上存在相同的案例  问题原因是 两边的权限不同  

但是经过我们验证 发现没有此类问题

下面很奇怪的就是为什么在 ASMCA 在进行磁盘扫描的时候等待，是什么触发了它的等待 ，他在等待什么

在运行ASMCA同时开一个窗口 tail日志 我们发现：

发生等待是对应的日志是 oracle.asm.**.**.fetchdiskgroup。。。

根据方法名称推断，正在获得ASM磁盘组信息 。 

实用SQLPLUS 进入ASM实例中 查询磁盘组信息

select * from  V$ASM_DISKGROUP   发现hang住 


不死心啊 

select * from V$ASM_DISKGROUP_STAT  这个没hang住 

继续 

select * from  V$ASM_DISK  又hang住了 

到底为什么呢？

View Name X$ Table name Description
V$ASM_DISKGROUP X$KFGRP performs disk discovery and lists diskgroups
V$ASM_DISKGROUP_STAT X$KFGRP_STAT diskgroup stats without disk discovery
V$ASM_DISK X$KFDSK, X$KFKID performs disk discovery, lists disks and their usage metrics
V$ASM_DISK_STAT X$KFDSK_STAT, X$KFKID lists disks and their usage metrics
V$ASM_FILE X$KFFIL lists ASM files, including metadata/asmdisk files
V$ASM_ALIAS X$KFALS lists ASM aliases, files and directories
V$ASM_TEMPLATE X$KFTMTA lists the available templates and their properties
V$ASM_CLIENT X$KFNCL lists DB instances connected to ASM
V$ASM_OPERATION X$KFGMG lists rebalancing operations
N.A. X$KFKLIB available libraries, includes asmlib path
N.A. X$KFDPARTNER lists disk-to-partner relationships
N.A. X$KFFXP extent map table for all ASM files
N.A. X$KFDAT extent list for all ASM disks
N.A. X$KFBH describes the ASM cache (buffer cache of ASM in blocks of 4K (_asm_blksize)
N.A. X$KFCCE a linked list of ASM blocks. to be further investiga



现在问题已经很很明显了，就是扫描磁盘的问题。

 正所谓点背不能怨社会，命苦不能怨政府 我们打算重来 

删除了grid的安装，再次进行grid安装

在建立ASM磁盘组作为仲裁盘的时候，依然可以识别到10块硬盘 但是问题在于 
无法识别的硬盘每个都显示为400+T 但是SA给我们的信息是每个磁盘为1T 再次运行bootinfo命令  确认为1T

问题很明显 ，出在存储上。

联系SA 确认存储之前用过，SA只是在LV层面进行了回收，但是在存储层面没有做清空操作，导致磁盘头部存在脏信息。导致ASMCA进程认为该盘有400+T
以至于进行超长时间扫描

解决方法：SA全面清理存储 并重新划分LV  再次进行安装  一切顺利，RAC环境搭建成功。
