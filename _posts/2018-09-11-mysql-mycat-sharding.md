---
layout: post
title: MySQL中使用mycat进行分库分表
date: 2018-09-11
categories: blog
tags: [MySQL]
description: MySQL中使用mycat进行分库分表
---

## 规划

节点名 | IP |角色 
 --- | --- | ---
node1 | 192.168.2.61 | master  
node2 | 192.168.2.62 | slave1 -->master  
node3 | 192.168.2.63 | slave2 -->master  
node4 | 192.168.2.64 | mycat  


## Mycat安装


### 安装java

```sql

[root@node4 ~]# tar -xf jdk-8u144-linux-x64.tar.gz
[root@node4 ~]# cp -rf   jdk-8u144-linux-x64.tar /usr/local/jdk


```
### 卸载自带Java

```sql

[root@node4 ~]# yum -y remove java
Loaded plugins: fastestmirror, refresh-packagekit, security
Setting up Remove Process
Resolving Dependencies
--> Running transaction check
---> Package java-1.6.0-openjdk.x86_64 1:1.6.0.0-1.66.1.13.0.el6 will be erased
--> Processing Dependency: jre >= 1.5.0 for package: 1:libreoffice-ure-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libjawt.so(SUNWprivate_1.1)(64bit) for package: 1:libreoffice-core-4.0.4.2-9.el6.x86_64
---> Package java-1.7.0-openjdk.x86_64 1:1.7.0.45-2.4.3.3.el6 will be erased
--> Running transaction check
---> Package libreoffice-core.x86_64 1:4.0.4.2-9.el6 will be erased
--> Processing Dependency: libavmedialo.so()(64bit) for package: 1:libreoffice-calc-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libbasegfxlo.so()(64bit) for package: 1:libreoffice-pdfimport-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libbasegfxlo.so()(64bit) for package: 1:libreoffice-writer-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libbasegfxlo.so()(64bit) for package: 1:libreoffice-graphicfilter-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libbasegfxlo.so()(64bit) for package: 1:libreoffice-calc-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libcomphelpgcc3.so()(64bit) for package: 1:libreoffice-math-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libcomphelpgcc3.so()(64bit) for package: 1:libreoffice-writer-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libcomphelpgcc3.so()(64bit) for package: 1:libreoffice-graphicfilter-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libcomphelpgcc3.so()(64bit) for package: 1:libreoffice-calc-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libdbtoolslo.so()(64bit) for package: 1:libreoffice-calc-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libdrawinglayerlo.so()(64bit) for package: 1:libreoffice-calc-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libeditenglo.so()(64bit) for package: 1:libreoffice-math-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libeditenglo.so()(64bit) for package: 1:libreoffice-writer-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libeditenglo.so()(64bit) for package: 1:libreoffice-graphicfilter-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libeditenglo.so()(64bit) for package: 1:libreoffice-calc-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libfilelo.so()(64bit) for package: 1:libreoffice-calc-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libi18nisolang1gcc3.so()(64bit) for package: 1:libreoffice-math-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libi18nisolang1gcc3.so()(64bit) for package: 1:libreoffice-writer-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libi18nisolang1gcc3.so()(64bit) for package: 1:libreoffice-graphicfilter-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libi18nisolang1gcc3.so()(64bit) for package: 1:libreoffice-calc-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libi18nutilgcc3.so()(64bit) for package: 1:libreoffice-math-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libi18nutilgcc3.so()(64bit) for package: 1:libreoffice-writer-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libmsfilterlo.so()(64bit) for package: 1:libreoffice-math-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libmsfilterlo.so()(64bit) for package: 1:libreoffice-writer-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libmsfilterlo.so()(64bit) for package: 1:libreoffice-calc-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libooxlo.so()(64bit) for package: 1:libreoffice-math-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libooxlo.so()(64bit) for package: 1:libreoffice-writer-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libooxlo.so()(64bit) for package: 1:libreoffice-calc-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libsaxlo.so()(64bit) for package: 1:libreoffice-math-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libsaxlo.so()(64bit) for package: 1:libreoffice-calc-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libsaxlo.so()(64bit) for package: 1:libreoffice-graphicfilter-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libsaxlo.so()(64bit) for package: 1:libreoffice-writer-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libsaxlo.so()(64bit) for package: 1:libreoffice-impress-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libsblo.so()(64bit) for package: 1:libreoffice-writer-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libsblo.so()(64bit) for package: 1:libreoffice-calc-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libsfxlo.so()(64bit) for package: 1:libreoffice-math-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libsfxlo.so()(64bit) for package: 1:libreoffice-writer-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libsfxlo.so()(64bit) for package: 1:libreoffice-calc-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libsotlo.so()(64bit) for package: 1:libreoffice-math-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libsotlo.so()(64bit) for package: 1:libreoffice-writer-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libsotlo.so()(64bit) for package: 1:libreoffice-calc-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libsvllo.so()(64bit) for package: 1:libreoffice-math-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libsvllo.so()(64bit) for package: 1:libreoffice-writer-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libsvllo.so()(64bit) for package: 1:libreoffice-graphicfilter-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libsvllo.so()(64bit) for package: 1:libreoffice-calc-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libsvtlo.so()(64bit) for package: 1:libreoffice-math-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libsvtlo.so()(64bit) for package: 1:libreoffice-writer-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libsvtlo.so()(64bit) for package: 1:libreoffice-graphicfilter-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libsvtlo.so()(64bit) for package: 1:libreoffice-calc-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libsvxcorelo.so()(64bit) for package: 1:libreoffice-math-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libsvxcorelo.so()(64bit) for package: 1:libreoffice-writer-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libsvxcorelo.so()(64bit) for package: 1:libreoffice-graphicfilter-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libsvxcorelo.so()(64bit) for package: 1:libreoffice-calc-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libsvxlo.so()(64bit) for package: 1:libreoffice-math-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libsvxlo.so()(64bit) for package: 1:libreoffice-writer-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libsvxlo.so()(64bit) for package: 1:libreoffice-calc-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libswlo.so()(64bit) for package: 1:libreoffice-writer-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libtklo.so()(64bit) for package: 1:libreoffice-math-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libtklo.so()(64bit) for package: 1:libreoffice-writer-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libtklo.so()(64bit) for package: 1:libreoffice-calc-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libtllo.so()(64bit) for package: 1:libreoffice-math-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libtllo.so()(64bit) for package: 1:libreoffice-writer-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libtllo.so()(64bit) for package: 1:libreoffice-graphicfilter-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libtllo.so()(64bit) for package: 1:libreoffice-calc-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libucbhelper4gcc3.so()(64bit) for package: 1:libreoffice-math-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libucbhelper4gcc3.so()(64bit) for package: 1:libreoffice-writer-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libucbhelper4gcc3.so()(64bit) for package: 1:libreoffice-calc-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libutllo.so()(64bit) for package: 1:libreoffice-math-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libutllo.so()(64bit) for package: 1:libreoffice-writer-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libutllo.so()(64bit) for package: 1:libreoffice-graphicfilter-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libutllo.so()(64bit) for package: 1:libreoffice-calc-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libvbahelperlo.so()(64bit) for package: 1:libreoffice-writer-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libvbahelperlo.so()(64bit) for package: 1:libreoffice-calc-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libvcllo.so()(64bit) for package: 1:libreoffice-math-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libvcllo.so()(64bit) for package: 1:libreoffice-calc-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libvcllo.so()(64bit) for package: 1:libreoffice-graphicfilter-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libvcllo.so()(64bit) for package: 1:libreoffice-writer-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libvcllo.so()(64bit) for package: 1:libreoffice-impress-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libxolo.so()(64bit) for package: 1:libreoffice-math-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libxolo.so()(64bit) for package: 1:libreoffice-writer-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libxolo.so()(64bit) for package: 1:libreoffice-graphicfilter-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libxolo.so()(64bit) for package: 1:libreoffice-calc-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libreoffice-core = 1:4.0.4.2-9.el6 for package: 1:libreoffice-langpack-en-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libreoffice-core = 1:4.0.4.2-9.el6 for package: 1:libreoffice-calc-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libreoffice-core = 1:4.0.4.2-9.el6 for package: 1:libreoffice-xsltfilter-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libreoffice-core = 1:4.0.4.2-9.el6 for package: 1:libreoffice-pdfimport-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libreoffice-core = 1:4.0.4.2-9.el6 for package: 1:libreoffice-writer-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libreoffice-core = 1:4.0.4.2-9.el6 for package: 1:libreoffice-graphicfilter-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libreoffice-core = 1:4.0.4.2-9.el6 for package: 1:libreoffice-impress-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libreoffice-core = 1:4.0.4.2-9.el6 for package: 1:libreoffice-math-4.0.4.2-9.el6.x86_64
--> Processing Dependency: libreoffice-core = 1:4.0.4.2-9.el6 for package: 1:libreoffice-draw-4.0.4.2-9.el6.x86_64
---> Package libreoffice-ure.x86_64 1:4.0.4.2-9.el6 will be erased
--> Running transaction check
---> Package libreoffice-calc.x86_64 1:4.0.4.2-9.el6 will be erased
---> Package libreoffice-draw.x86_64 1:4.0.4.2-9.el6 will be erased
---> Package libreoffice-graphicfilter.x86_64 1:4.0.4.2-9.el6 will be erased
---> Package libreoffice-impress.x86_64 1:4.0.4.2-9.el6 will be erased
---> Package libreoffice-langpack-en.x86_64 1:4.0.4.2-9.el6 will be erased
---> Package libreoffice-math.x86_64 1:4.0.4.2-9.el6 will be erased
---> Package libreoffice-pdfimport.x86_64 1:4.0.4.2-9.el6 will be erased
---> Package libreoffice-writer.x86_64 1:4.0.4.2-9.el6 will be erased
---> Package libreoffice-xsltfilter.x86_64 1:4.0.4.2-9.el6 will be erased
--> Finished Dependency Resolution
Repository 'Server' is missing name in configuration, using id
file:///media/repodata/repomd.xml: [Errno 14] Could not open/read file:///media/repodata/repomd.xml
Trying other mirror.

Dependencies Resolved

============================================================================================================================================================
 Package                               Arch               Version                                Repository                                            Size
============================================================================================================================================================
Removing:
 java-1.6.0-openjdk                    x86_64             1:1.6.0.0-1.66.1.13.0.el6              @anaconda-CentOS-201311272149.x86_64/6.5              75 M
 java-1.7.0-openjdk                    x86_64             1:1.7.0.45-2.4.3.3.el6                 @anaconda-CentOS-201311272149.x86_64/6.5              90 M
Removing for dependencies:
 libreoffice-calc                      x86_64             1:4.0.4.2-9.el6                        @anaconda-CentOS-201311272149.x86_64/6.5              24 M
 libreoffice-core                      x86_64             1:4.0.4.2-9.el6                        @anaconda-CentOS-201311272149.x86_64/6.5             229 M
 libreoffice-draw                      x86_64             1:4.0.4.2-9.el6                        @anaconda-CentOS-201311272149.x86_64/6.5             2.2 M
 libreoffice-graphicfilter             x86_64             1:4.0.4.2-9.el6                        @anaconda-CentOS-201311272149.x86_64/6.5             1.2 M
 libreoffice-impress                   x86_64             1:4.0.4.2-9.el6                        @anaconda-CentOS-201311272149.x86_64/6.5             4.3 M
 libreoffice-langpack-en               x86_64             1:4.0.4.2-9.el6                        @anaconda-CentOS-201311272149.x86_64/6.5             0.0
 libreoffice-math                      x86_64             1:4.0.4.2-9.el6                        @anaconda-CentOS-201311272149.x86_64/6.5             3.2 M
 libreoffice-pdfimport                 x86_64             1:4.0.4.2-9.el6                        @anaconda-CentOS-201311272149.x86_64/6.5             641 k
 libreoffice-ure                       x86_64             1:4.0.4.2-9.el6                        @anaconda-CentOS-201311272149.x86_64/6.5             8.1 M
 libreoffice-writer                    x86_64             1:4.0.4.2-9.el6                        @anaconda-CentOS-201311272149.x86_64/6.5              17 M
 libreoffice-xsltfilter                x86_64             1:4.0.4.2-9.el6                        @anaconda-CentOS-201311272149.x86_64/6.5             2.0 M

Transaction Summary
============================================================================================================================================================
Remove       13 Package(s)

Installed size: 456 M
Downloading Packages:
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
  Erasing    : 1:libreoffice-math-4.0.4.2-9.el6.x86_64                                                                                                 1/13
  Erasing    : 1:libreoffice-impress-4.0.4.2-9.el6.x86_64                                                                                              2/13
  Erasing    : 1:libreoffice-calc-4.0.4.2-9.el6.x86_64                                                                                                 3/13
  Erasing    : 1:libreoffice-pdfimport-4.0.4.2-9.el6.x86_64                                                                                            4/13
  Erasing    : 1:libreoffice-draw-4.0.4.2-9.el6.x86_64                                                                                                 5/13
  Erasing    : 1:libreoffice-xsltfilter-4.0.4.2-9.el6.x86_64                                                                                           6/13
  Erasing    : 1:libreoffice-langpack-en-4.0.4.2-9.el6.x86_64                                                                                          7/13
  Erasing    : 1:libreoffice-graphicfilter-4.0.4.2-9.el6.x86_64                                                                                        8/13
  Erasing    : 1:libreoffice-writer-4.0.4.2-9.el6.x86_64                                                                                               9/13
  Erasing    : 1:libreoffice-core-4.0.4.2-9.el6.x86_64                                                                                                10/13
  Erasing    : 1:libreoffice-ure-4.0.4.2-9.el6.x86_64                                                                                                 11/13
  Erasing    : 1:java-1.6.0-openjdk-1.6.0.0-1.66.1.13.0.el6.x86_64                                                                                    12/13
  Erasing    : 1:java-1.7.0-openjdk-1.7.0.45-2.4.3.3.el6.x86_64                                                                                       13/13
  Verifying  : 1:libreoffice-ure-4.0.4.2-9.el6.x86_64                                                                                                  1/13
  Verifying  : 1:libreoffice-draw-4.0.4.2-9.el6.x86_64                                                                                                 2/13
  Verifying  : 1:java-1.7.0-openjdk-1.7.0.45-2.4.3.3.el6.x86_64                                                                                        3/13
  Verifying  : 1:libreoffice-core-4.0.4.2-9.el6.x86_64                                                                                                 4/13
  Verifying  : 1:libreoffice-calc-4.0.4.2-9.el6.x86_64                                                                                                 5/13
  Verifying  : 1:libreoffice-pdfimport-4.0.4.2-9.el6.x86_64                                                                                            6/13
  Verifying  : 1:libreoffice-impress-4.0.4.2-9.el6.x86_64                                                                                              7/13
  Verifying  : 1:libreoffice-graphicfilter-4.0.4.2-9.el6.x86_64                                                                                        8/13
  Verifying  : 1:libreoffice-langpack-en-4.0.4.2-9.el6.x86_64                                                                                          9/13
  Verifying  : 1:java-1.6.0-openjdk-1.6.0.0-1.66.1.13.0.el6.x86_64                                                                                    10/13
  Verifying  : 1:libreoffice-writer-4.0.4.2-9.el6.x86_64                                                                                              11/13
  Verifying  : 1:libreoffice-math-4.0.4.2-9.el6.x86_64                                                                                                12/13
  Verifying  : 1:libreoffice-xsltfilter-4.0.4.2-9.el6.x86_64                                                                                          13/13

Removed:
  java-1.6.0-openjdk.x86_64 1:1.6.0.0-1.66.1.13.0.el6                            java-1.7.0-openjdk.x86_64 1:1.7.0.45-2.4.3.3.el6

Dependency Removed:
  libreoffice-calc.x86_64 1:4.0.4.2-9.el6              libreoffice-core.x86_64 1:4.0.4.2-9.el6           libreoffice-draw.x86_64 1:4.0.4.2-9.el6
  libreoffice-graphicfilter.x86_64 1:4.0.4.2-9.el6     libreoffice-impress.x86_64 1:4.0.4.2-9.el6        libreoffice-langpack-en.x86_64 1:4.0.4.2-9.el6
  libreoffice-math.x86_64 1:4.0.4.2-9.el6              libreoffice-pdfimport.x86_64 1:4.0.4.2-9.el6      libreoffice-ure.x86_64 1:4.0.4.2-9.el6
  libreoffice-writer.x86_64 1:4.0.4.2-9.el6            libreoffice-xsltfilter.x86_64 1:4.0.4.2-9.el6

Complete!


```

###  解压安装MyCAT
```sql

[root@node4 ~]# tar -zxf Mycat-server-1.6.tar.gz
[root@node4 ~]# mv mycat /usr/local/mycat

```

### 创建用户


```sql

[root@node4 ~]# useradd mycat
[root@node4 ~]# id mycat
uid=501(mycat) gid=501(mycat) groups=501(mycat)
[root@node4 ~]# chown -R mycat:mycat /usr/local/mycat

```

## 修改配置文件

```sql

[root@node4 ~]# cd /usr/local/mycat/conf

```

### 修改server.xml

```sql

[root@node4 ~]# cat  /usr/local/mycat/conf/server.xml
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

### 修改schema.xml

```sql

[root@node4 ~]# cat /usr/local/mycat/conf/schema.xml
<?xml version="1.0"?>
<!DOCTYPE mycat:schema SYSTEM "schema.dtd">
<mycat:schema xmlns:mycat="http://io.mycat/">


        <schema name="mycatdb" checkSQLschema="false" sqlMaxLimit="100">
                <!-- auto sharding by id (long) -->
                <table name="dao_table1" dataNode="dn1,dn2,dn3" rule="dao-rule1"/>  此处添加分片规则为dao-rule1
                <table name="dao_table2" dataNode="dn1,dn2,dn3" rule="mod-long"/>
                <table name="dao_table3" dataNode="dn1,dn2,dn3" rule="mod-long"/>
        </schema>
        <dataNode name="dn1" dataHost="localhost1" database="dao_db1" />
        <dataNode name="dn2" dataHost="localhost1" database="dao_db2" />
        <dataNode name="dn3" dataHost="localhost1" database="dao_db3"/>


        <dataHost name="localhost1" maxCon="1000" minCon="10" balance="0"
                writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
                <heartbeat>show slave status</heartbeat>
                <writeHost host="node1" url="192.168.2.61:3306" user="dao" password="dao">
                </writeHost>
        </dataHost>


</mycat:schema>




```

### 修改规则表

- 添加规则dao-rule1 使其可以根据c1列分片

``` sql


[root@node4 ~]# cat /usr/local/mycat/conf/rule.xml
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
        <tableRule name="dao-rule1">  添加规则dao-rule1 指定分片列为c1
                <rule>
                        <columns>c1</columns>
                        <algorithm>mod-long</algorithm>
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
        <tableRule name="mod-long">  --此处可以看到mod-long模式使用id列进行分片
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


### 启动Mmycat

``` sql

[mysql@node4 ~]$ /usr/local/mycat/bin/mycat start
Starting Mycat-server...


```



## 进行测试

### 在mycat中建表

```sql

[mysql@node4 ~]$   mysql -uuser -puser -P8066 -h127.0.0.1  -Dmycatdb
mysql: [Warning] Using a password on the command line interface can be insecure.
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1
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

mysql> use mycatdb
Database changed
mysql>  create table dao_table1(c1 int auto_increment primary key , c2 int ) ;
Query OK, 0 rows affected (0.10 sec)

```

### 查看数据库中的表

```sql


mysql> use dao_db1 ;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables ;
+-------------------+
| Tables_in_dao_db1 |
+-------------------+
| dao_table1        |
+-------------------+
1 row in set (0.00 sec)

mysql> use dao_db2 ;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables ;
+-------------------+
| Tables_in_dao_db2 |
+-------------------+
| dao_table1        |
+-------------------+
1 row in set (0.00 sec)

mysql> use dao_db3 ;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables ;
+-------------------+
| Tables_in_dao_db3 |
+-------------------+
| dao_table1        |
+-------------------+
1 row in set (0.00 sec)


```

### 在mycat中插入数据 

```sql

mysql> use mycatdb ;
Database changed

mysql> insert into dao_table1(c1,c2) values(1,1) ;
Query OK, 1 row affected (0.02 sec)

mysql> insert into dao_table1(c1,c2) values(2,1) ;
Query OK, 1 row affected (0.02 sec)

mysql> insert into dao_table1(c1,c2) values(3,3) ;
Query OK, 1 row affected (0.02 sec)



```

### 在数据库中查询

```sql

mysql> select * from dao_db1.dao_table1 ;
+----+------+
| c1 | c2   |
+----+------+
|  3 |    3 |
+----+------+
1 row in set (0.00 sec)

mysql> select * from dao_db2.dao_table1 ;
+----+------+
| c1 | c2   |
+----+------+
|  1 |    1 |
+----+------+
1 row in set (0.00 sec)

mysql> select * from dao_db3.dao_table1 ;
+----+------+
| c1 | c2   |
+----+------+
|  2 |    1 |
+----+------+
1 row in set (0.01 sec)


```