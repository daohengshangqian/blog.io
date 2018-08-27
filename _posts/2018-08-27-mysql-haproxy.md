---
layout: post
title: MySQL 使用Haproxy进行负载均衡
date: 2018-08-27
categories: blog
tags: [MySQL]
description: MySQL 使用Haproxy进行负载均衡
---


## 架构图

```
graph TD
    A[APP] --> B{haproxy}
    B --> | IP1 |C[node1]
    B --> | IP2 |D[node2]

```
## 安装haproxy

### 解压haproxy
```sql

[root@node4 ~]# tar -zxf haproxy-1.4.20.tar.gz

```

### 安装haproxy

```sql

[root@node4 ~]# cd haproxy-1.4.20
[root@node4 haproxy-1.4.20]# make TARGET=linux26 PREFIX=/usr/local/haproxy ARCH=X86_64
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" \
              -DBUILD_TARGET='"linux26"' \
              -DBUILD_ARCH='"X86_64"' \
              -DBUILD_CPU='"generic"' \
              -DBUILD_CC='"gcc"' \
              -DBUILD_CFLAGS='"-O2 -g -fno-strict-aliasing"' \
              -DBUILD_OPTIONS='""' \
               -c -o src/haproxy.o src/haproxy.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o src/sessionhash.o src/sessionhash.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o src/base64.o src/base64.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o src/protocols.o src/protocols.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o src/uri_auth.o src/uri_auth.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o src/standard.o src/standard.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o src/buffers.o src/buffers.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o src/log.o src/log.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o src/task.o src/task.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o src/time.o src/time.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o src/fd.o src/fd.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o src/pipe.o src/pipe.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o src/regex.o src/regex.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o src/cfgparse.o src/cfgparse.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o src/server.o src/server.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o src/checks.o src/checks.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o src/queue.o src/queue.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o src/client.o src/client.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o src/proxy.o src/proxy.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o src/stick_table.o src/stick_table.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o src/proto_uxst.o src/proto_uxst.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o src/proto_http.o src/proto_http.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o src/stream_sock.o src/stream_sock.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o src/appsession.o src/appsession.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o src/backend.o src/backend.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o src/lb_chash.o src/lb_chash.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o src/lb_fwlc.o src/lb_fwlc.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o src/lb_fwrr.o src/lb_fwrr.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o src/lb_map.o src/lb_map.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o src/stream_interface.o src/stream_interface.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o src/dumpstats.o src/dumpstats.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o src/proto_tcp.o src/proto_tcp.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o src/session.o src/session.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o src/hdr_idx.o src/hdr_idx.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o src/ev_select.o src/ev_select.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o src/signal.o src/signal.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o src/acl.o src/acl.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o src/pattern.o src/pattern.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o src/memory.o src/memory.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o src/freq_ctr.o src/freq_ctr.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o src/auth.o src/auth.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o src/ev_poll.o src/ev_poll.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o src/ev_epoll.o src/ev_epoll.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o src/ev_sepoll.o src/ev_sepoll.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o ebtree/ebtree.o ebtree/ebtree.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o ebtree/eb32tree.o ebtree/eb32tree.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o ebtree/eb64tree.o ebtree/eb64tree.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o ebtree/ebmbtree.o ebtree/ebmbtree.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o ebtree/ebsttree.o ebtree/ebsttree.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o ebtree/ebimtree.o ebtree/ebimtree.c
gcc -Iinclude -Iebtree -Wall  -O2 -g -fno-strict-aliasing       -DTPROXY -DCONFIG_HAP_CRYPT -DENABLE_POLL -DENABLE_EPOLL -DENABLE_SEPOLL -DNETFILTER -DUSE_GETSOCKNAME  -DCONFIG_HAPROXY_VERSION=\"1.4.20\" -DCONFIG_HAPROXY_DATE=\"2012/03/10\" -c -o ebtree/ebistree.o ebtree/ebistree.c
gcc  -g -o haproxy src/haproxy.o src/sessionhash.o src/base64.o src/protocols.o src/uri_auth.o src/standard.o src/buffers.o src/log.o src/task.o src/time.o src/fd.o src/pipe.o src/regex.o src/cfgparse.o src/server.o src/checks.o src/queue.o src/client.o src/proxy.o src/stick_table.o src/proto_uxst.o src/proto_http.o src/stream_sock.o src/appsession.o src/backend.o src/lb_chash.o src/lb_fwlc.o src/lb_fwrr.o src/lb_map.o src/stream_interface.o src/dumpstats.o src/proto_tcp.o src/session.o src/hdr_idx.o src/ev_select.o src/signal.o src/acl.o src/pattern.o src/memory.o src/freq_ctr.o src/auth.o src/ev_poll.o src/ev_epoll.o src/ev_sepoll.o ebtree/ebtree.o ebtree/eb32tree.o ebtree/eb64tree.o ebtree/ebmbtree.o ebtree/ebsttree.o ebtree/ebimtree.o ebtree/ebistree.o   -lcrypt
[root@node4 haproxy-1.4.20]# make install PREFIX=/usr/local/haproxy
install -d /usr/local/haproxy/sbin
install haproxy /usr/local/haproxy/sbin
install -d /usr/local/haproxy/share/man/man1
install -m 644 doc/haproxy.1 /usr/local/haproxy/share/man/man1
install -d /usr/local/haproxy/doc/haproxy
for x in configuration architecture haproxy-en haproxy-fr; do \
                install -m 644 doc/$x.txt /usr/local/haproxy/doc/haproxy ; \
        done

```

## 对haproxy进行配置

### 创建配置文件

```sql

[root@node4 haproxy-1.4.20]# cd /usr/local/haproxy
[root@node4 haproxy]# mkdir conf
[root@node4 haproxy]# cd conf/
[root@node4 conf]# touch haproxy.cfg
[root@node4 conf]# vi haproxy.cfg


```

### 编辑配置文件

```sql

[root@node4 ~]# cat  /usr/local/haproxy/conf/haproxy.cfg
# this config needs haproxy-1.1.28 or haproxy-1.2.1
global
log 127.0.0.1 local0
maxconn 4096
log 127.0.0.1 local1 notice
#log loghost local0 info
#maxconn 4096
chroot /usr/local/haproxy
uid 501
gid 501
daemon
nbproc 1
pidfile /usr/local/haproxy/haproxy.pid
#debug
#quiet
defaults
log global
#option dontlognull
retries 3
option redispatch
maxconn 4096
timeout http-keep-alive 10s
timeout check 10s
contimeout 5000
clitimeout 50000
srvtimeout 50000
listen admin_stats 192.168.2.64:48800
stats enable
stats hide-version
stats realm <realm>
stats refresh 5s
stats uri /haproxy-status
stats auth admin:admin
stats admin if TRUE
mode http
option httplog
timeout connect 50000
timeout check 50000
timeout client 300000
timeout server 300000
listen mysql_proxy 192.168.2.64:3308
mode tcp
balance roundrobin
option tcpka
option tcplog
server pxc01 192.168.2.61:3306 weight 1 check inter 2000 rise 2 fall 5
server pxc02 192.168.2.62:3306 weight 1 check inter 2000 rise 2 fall 5


```

### 启动haproxy

```sql

[root@node4 conf]# /usr/local/haproxy/sbin/haproxy -f /usr/local/haproxy/conf/haproxy.cfg


```

## 验证


```sql


[root@node4 ~]#  /usr/local/mysql/bin/mysql -h 192.168.2.64 -P 3308  -udao -pdao  -e "select @@hostname"
mysql: [Warning] Using a password on the command line interface can be insecure.
+------------+
| @@hostname |
+------------+
| node1      |
+------------+
[root@node4 ~]#
[root@node4 ~]#  /usr/local/mysql/bin/mysql -h 192.168.2.64 -P 3308  -udao -pdao  -e "select @@hostname"
mysql: [Warning] Using a password on the command line interface can be insecure.
+------------+
| @@hostname |
+------------+
| node2      |
+------------+


```