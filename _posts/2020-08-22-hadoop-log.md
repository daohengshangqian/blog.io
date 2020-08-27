---
layout: post
title: hadoop日志服务相关配置
date: 2020-08-22
categories: blog
tags: [大数据]
description: hadoop日志服务相关配置
---


## 概述
在执行mapredecue任务是，可以通过Web UI界面查看任务运行状态，界面如下
![image](https://wx1.sinaimg.cn/mw690/006Pv9mtgy1gi52vc1wpnj31gy0c1ac8.jpg)

在任务报错时，可查看相关的日志
![image](https://wx2.sinaimg.cn/mw690/006Pv9mtgy1gi52zcp540j30ty0q90vf.jpg)

如没有进行配置，报错如下：

![image](https://wx4.sinaimg.cn/mw690/006Pv9mtgy1gi53c66w9dj31b8078tas.jpg)


## 配置

- yarn-site.xml

```xml

<property>
        <name>yarn.log-aggregation-enable</name>
        <value>true</value>
        <description>启用日志聚合，默认值为False，即禁用</description>
    </property>
    <property>
        <name>yarn.log-aggregation.retain-seconds</name>
        <value>86400</value>
        <description>日志保留时间设置为一天</description>
    </property>
    <property>
        <name>yarn.nodemanager.log-dirs</name>
        <value>${yarn.log.dir}/userlogs</value>
        <description>应用程序的本地化的日志目录</description>
    </property>
    <property>
      <name>yarn.log.server.url</name>
      <value>http://node1:19888/jobhistory/logs</value>
    </property>
    
```

- mapred-site.xml

```xml

 <!-- 指定jobhistory server的rpc地址 -->
  <property>
    <name>mapreduce.jobhistory.address</name>
    <value>node1:10020</value>
  </property>

  <!-- 指定jobhistory server的http地址 -->
  <property>
    <name>mapreduce.jobhistory.webapp.address</name>
    <value>node1:19888</value>
  </property>
    
```

## 验证

点击失败任务的日志

![image](https://wx1.sinaimg.cn/mw690/006Pv9mtgy1gi53fckt9aj30sb0gxwhe.jpg)