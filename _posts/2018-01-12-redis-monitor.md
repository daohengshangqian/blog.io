---
layout: post
title:  Redis监控搭建
date: 2018-01-12
categories: blog
tags: [Redis]
description: redis监控搭建
---

## 下载python

```sql
[root@etlbi ~]# wget https://www.python.org/ftp/python/2.7.8/Python-2.7.8.tgz
--2018-01-01 04:15:12--  https://www.python.org/ftp/python/2.7.8/Python-2.7.8.tgz
Resolving www.python.org... 151.101.24.223, 2a04:4e42:2e::223
Connecting to www.python.org|151.101.24.223|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 14846119 (14M) [application/octet-stream]
Saving to: â€œPython-2.7.8.tgzâ€

100%[=================================================================================================================>] 14,846,119   444K/s   in 39s     

2018-01-01 04:15:56 (373 KB/s) - â€œPython-2.7.8.tgzâ€ saved [14846119/14846119]

```

## 解压安装

```sql
[root@etlbi ~]# tar -zxvf Python-2.7.8.tgz 

[root@etlbi ~]# mv ./Python-2.7.8 /

[root@etlbi Python-2.7.8]# ./configure 

[root@etlbi Python-2.7.8]# make && make install

```


## 修改环境变量

```sql
[root@etlbi ~]# vi .bash_profile 
# .bash_profile

# Get the aliases and functions
if [ -f ~/.bashrc ]; then
        . ~/.bashrc
fi

# User specific environment and startup programs

PATH=./Python-2.7.8:$PATH:$HOME/bin

export PATH
~

[root@etlbi ~]# source .bash_profile 


```

## 下载pip


```sql

[root@etlbi Python-2.7.8]# wget https://bootstrap.pypa.io/get-pip.py
--2018-01-01 04:37:22--  https://bootstrap.pypa.io/get-pip.py
Resolving bootstrap.pypa.io... 151.101.24.175
Connecting to bootstrap.pypa.io|151.101.24.175|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1595408 (1.5M) [text/x-python]
Saving to: â€œget-pip.py.1â€

100%[=================================================================================================================>] 1,595,408    445K/s   in 3.5s    

2018-01-01 04:37:26 (445 KB/s) - â€œget-pip.py.1â€ saved [1595408/1595408]

```

## 安装pip

```sql
[root@etlbi Python-2.7.8]# python get-pip.py 
Collecting pip
/tmp/tmppgAvK7/pip.zip/pip/_vendor/requests/packages/urllib3/util/ssl_.py:318: SNIMissingWarning: An HTTPS request has been made, but the SNI (Subject Name Indication) extension to TLS is not available on this platform. This may cause the server to present an incorrect TLS certificate, which can cause validation failures. You can upgrade to a newer version of Python to solve this. For more information, see https://urllib3.readthedocs.io/en/latest/security.html#snimissingwarning.
/tmp/tmppgAvK7/pip.zip/pip/_vendor/requests/packages/urllib3/util/ssl_.py:122: InsecurePlatformWarning: A true SSLContext object is not available. This prevents urllib3 from configuring SSL appropriately and may cause certain SSL connections to fail. You can upgrade to a newer version of Python to solve this. For more information, see https://urllib3.readthedocs.io/en/latest/security.html#insecureplatformwarning.
  Downloading pip-9.0.1-py2.py3-none-any.whl (1.3MB)
    100% |â–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆ| 1.3MB 81kB/s 
Collecting setuptools
  Downloading setuptools-38.2.5-py2.py3-none-any.whl (489kB)
    100% |â–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆ| 491kB 48kB/s 
Collecting wheel
  Downloading wheel-0.30.0-py2.py3-none-any.whl (49kB)
    100% |â–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆ| 51kB 42kB/s 
Installing collected packages: pip, setuptools, wheel
Successfully installed pip-9.0.1 setuptools-38.2.5 wheel-0.30.0
/tmp/tmppgAvK7/pip.zip/pip/_vendor/requests/packages/urllib3/util/ssl_.py:122: InsecurePlatformWarning: A true SSLContext object is not available. This prevents urllib3 from configuring SSL appropriately and may cause certain SSL connections to fail. You can upgrade to a newer version of Python to solve this. For more information, see https://urllib3.readthedocs.io/en/latest/security.html#insecureplatformwarning.

```


## 上传解压redis monitor

```sql

[root@etlbi ~]# mv redis-monitor-master.zip /redis-monitor/

[root@etlbi ~]# cd /redis-monitor/

[root@etlbi redis-monitor]# unzip redis-monitor-master.zip 

[root@etlbi redis-monitor]# cd redis-monitor-master

```

## 安装依赖库

```sql

[root@etlbi redis-monitor-master]# pip install argparse
[root@etlbi redis-monitor-master]# pip install python-dateutil
[root@etlbi redis-monitor-master]# pip install redis
[root@etlbi redis-monitor-master]# pip install tornado

```

## 修改配置文件


```sql
{"master_slave_sms": "1,1", "RedisStatsServer": {"port": 6379, "server": "192.168.3.19"}^M
, "sms_alert": "127.0.0.1:9999", "DataStoreType": "redis", "RedisServers":[
 {"instance": "master", "group": "etlbi", "port": 6379, "server": "192.168.3.19"}^M
]}^M
```

## 启动监控

```sql

[root@etlbi src]# ./redis_live.py 
[root@etlbi src]# python ./redis_monitor.py 

```

## 查看监控

![image](http://wx4.sinaimg.cn/mw690/006Pv9mtly1fn0ay8sau9j31fl0pqwho.jpg)
