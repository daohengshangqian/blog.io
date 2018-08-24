---
layout: post
title: MySQL MHA搭建
date: 2018-08-23
categories: blog
tags: [MySQL]
description: MySQL MHA搭建
---

## 主机设置


### 架构规划

IP | 主机名 | 角色 |mha角色
--- | --- | --- | ---
192.168.2.61 | node1 | 主库  | node
192.168.2.62 | node2 | 从库1 | node
192.168.2.63 | node3 | 从库2 | node , mha-master


### 修改/etc/hosts


```sql

[root@node3 ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

192.168.2.61 node1
192.168.2.62 node2
192.168.2.63 node3

```


### 使用Oracle自带工具配置互信

```sql

[root@node1 ~]# ./sshUserSetup.sh  -user root -hosts "node1 node2 node3"  -advanced -noPromptPassphrase
The output of this script is also logged into /tmp/sshUserSetup_2018-08-21-07-10-27.log
Hosts are node1 node2 node3
user is root
Platform:- Linux
Checking if the remote hosts are reachable
PING node1 (192.168.2.61) 56(84) bytes of data.
64 bytes from node1 (192.168.2.61): icmp_seq=1 ttl=64 time=0.029 ms
64 bytes from node1 (192.168.2.61): icmp_seq=2 ttl=64 time=0.031 ms
64 bytes from node1 (192.168.2.61): icmp_seq=3 ttl=64 time=0.036 ms
64 bytes from node1 (192.168.2.61): icmp_seq=4 ttl=64 time=0.022 ms
64 bytes from node1 (192.168.2.61): icmp_seq=5 ttl=64 time=0.026 ms

--- node1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 3999ms
rtt min/avg/max/mdev = 0.022/0.028/0.036/0.008 ms
PING node2 (192.168.2.62) 56(84) bytes of data.
64 bytes from node2 (192.168.2.62): icmp_seq=1 ttl=64 time=0.219 ms
64 bytes from node2 (192.168.2.62): icmp_seq=2 ttl=64 time=0.327 ms
64 bytes from node2 (192.168.2.62): icmp_seq=3 ttl=64 time=0.340 ms
64 bytes from node2 (192.168.2.62): icmp_seq=4 ttl=64 time=0.384 ms
64 bytes from node2 (192.168.2.62): icmp_seq=5 ttl=64 time=0.360 ms

--- node2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4003ms
rtt min/avg/max/mdev = 0.219/0.326/0.384/0.056 ms
PING node3 (192.168.2.63) 56(84) bytes of data.
64 bytes from node3 (192.168.2.63): icmp_seq=1 ttl=64 time=0.389 ms
64 bytes from node3 (192.168.2.63): icmp_seq=2 ttl=64 time=0.467 ms
64 bytes from node3 (192.168.2.63): icmp_seq=3 ttl=64 time=0.379 ms
64 bytes from node3 (192.168.2.63): icmp_seq=4 ttl=64 time=0.490 ms
64 bytes from node3 (192.168.2.63): icmp_seq=5 ttl=64 time=0.286 ms

--- node3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4020ms
rtt min/avg/max/mdev = 0.286/0.402/0.490/0.073 ms
Remote host reachability check succeeded.
The following hosts are reachable: node1 node2 node3.
The following hosts are not reachable: .
All hosts are reachable. Proceeding further...
The script will setup SSH connectivity from the host node1 to all
the remote hosts. After the script is executed, the user can use SSH to run
commands on the remote hosts or copy files between this host node1
and the remote hosts without being prompted for passwords or confirmations.

NOTE 1:
As part of the setup procedure, this script will use ssh and scp to copy
files between the local host and the remote hosts. Since the script does not
store passwords, you may be prompted for the passwords during the execution of
the script whenever ssh or scp is invoked.

NOTE 2:
AS PER SSH REQUIREMENTS, THIS SCRIPT WILL SECURE THE USER HOME DIRECTORY
AND THE .ssh DIRECTORY BY REVOKING GROUP AND WORLD WRITE PRIVILEDGES TO THESE
directories.

Do you want to continue and let the script make the above mentioned changes (yes/no)?
yes

The user chose yes
User chose to skip passphrase related questions.
Creating .ssh directory on local host, if not present already
Creating authorized_keys file on local host
Changing permissions on authorized_keys to 644 on local host
Creating known_hosts file on local host
Changing permissions on known_hosts to 644 on local host
Creating config file on local host
If a config file exists already at /root/.ssh/config, it would be backed up to /root/.ssh/config.backup.
Removing old private/public keys on local host
Running SSH keygen on local host with empty passphrase
Generating public/private rsa key pair.
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
e2:43:6a:f8:c0:e9:80:e9:4b:af:4a:d1:33:a1:68:06 root@node1
The key's randomart image is:
+--[ RSA 1024]----+
|                 |
|                 |
|E  .             |
|o o .            |
|.= +  o S        |
|+o.oo+ .         |
|+o= o o          |
|+o.+   .         |
|o++..            |
+-----------------+
Creating .ssh directory and setting permissions on remote host node1
THE SCRIPT WOULD ALSO BE REVOKING WRITE PERMISSIONS FOR group AND others ON THE HOME DIRECTORY FOR root. THIS IS AN SSH REQUIREMENT.
The script would create ~root/.ssh/config file on remote host node1. If a config file exists already at ~root/.ssh/config, it would be backed up to ~root/.ssh/config.backup.
The user may be prompted for a password here since the script would be running SSH on host node1.
Warning: Permanently added 'node1,192.168.2.61' (RSA) to the list of known hosts.
root@node1's password:
Done with creating .ssh directory and setting permissions on remote host node1.
Creating .ssh directory and setting permissions on remote host node2
THE SCRIPT WOULD ALSO BE REVOKING WRITE PERMISSIONS FOR group AND others ON THE HOME DIRECTORY FOR root. THIS IS AN SSH REQUIREMENT.
The script would create ~root/.ssh/config file on remote host node2. If a config file exists already at ~root/.ssh/config, it would be backed up to ~root/.ssh/config.backup.
The user may be prompted for a password here since the script would be running SSH on host node2.
Warning: Permanently added 'node2,192.168.2.62' (RSA) to the list of known hosts.
root@node2's password:
Done with creating .ssh directory and setting permissions on remote host node2.
Creating .ssh directory and setting permissions on remote host node3
THE SCRIPT WOULD ALSO BE REVOKING WRITE PERMISSIONS FOR group AND others ON THE HOME DIRECTORY FOR root. THIS IS AN SSH REQUIREMENT.
The script would create ~root/.ssh/config file on remote host node3. If a config file exists already at ~root/.ssh/config, it would be backed up to ~root/.ssh/config.backup.
The user may be prompted for a password here since the script would be running SSH on host node3.
Warning: Permanently added 'node3,192.168.2.63' (RSA) to the list of known hosts.
root@node3's password:
Done with creating .ssh directory and setting permissions on remote host node3.
Copying local host public key to the remote host node1
The user may be prompted for a password or passphrase here since the script would be using SCP for host node1.
root@node1's password:
Done copying local host public key to the remote host node1
Copying local host public key to the remote host node2
The user may be prompted for a password or passphrase here since the script would be using SCP for host node2.
root@node2's password:
Done copying local host public key to the remote host node2
Copying local host public key to the remote host node3
The user may be prompted for a password or passphrase here since the script would be using SCP for host node3.
root@node3's password:
Done copying local host public key to the remote host node3
Creating keys on remote host node1 if they do not exist already. This is required to setup SSH on host node1.

Creating keys on remote host node2 if they do not exist already. This is required to setup SSH on host node2.
Generating public/private rsa key pair.
Your identification has been saved in .ssh/id_rsa.
Your public key has been saved in .ssh/id_rsa.pub.
The key fingerprint is:
fd:4a:f7:c5:3b:70:bc:38:5a:07:4e:6c:1c:a9:e7:14 root@node2
The key's randomart image is:
+--[ RSA 1024]----+
|                 |
|              .  |
|             E   |
|         .  + o  |
|        S .. O.  |
|           .B..+ |
|          . o++.+|
|         . o.+.+.|
|          ... o..|
+-----------------+
Creating keys on remote host node3 if they do not exist already. This is required to setup SSH on host node3.
Generating public/private rsa key pair.
Your identification has been saved in .ssh/id_rsa.
Your public key has been saved in .ssh/id_rsa.pub.
The key fingerprint is:
b7:3c:44:2d:24:11:13:23:84:76:82:40:e2:3f:53:64 root@node3
The key's randomart image is:
+--[ RSA 1024]----+
|+o . Eo B+.      |
|o . * .. = .     |
| . . +    o .    |
|  . .    . .     |
|   +    S o      |
|    o    + .     |
|          +      |
|           .     |
|                 |
+-----------------+
Updating authorized_keys file on remote host node1
Updating known_hosts file on remote host node1
Updating authorized_keys file on remote host node2
Updating known_hosts file on remote host node2
Updating authorized_keys file on remote host node3
Updating known_hosts file on remote host node3
cat: /root/.ssh/known_hosts.tmp: No such file or directory
cat: /root/.ssh/authorized_keys.tmp: No such file or directory
SSH setup is complete.

------------------------------------------------------------------------
Verifying SSH setup
===================
The script will now run the date command on the remote nodes using ssh
to verify if ssh is setup correctly. IF THE SETUP IS CORRECTLY SETUP,
THERE SHOULD BE NO OUTPUT OTHER THAN THE DATE AND SSH SHOULD NOT ASK FOR
PASSWORDS. If you see any output other than date or are prompted for the
password, ssh is not setup correctly and you will need to resolve the
issue and set up ssh again.
The possible causes for failure could be:
1. The server settings in /etc/ssh/sshd_config file do not allow ssh
for user root.
2. The server may have disabled public key based authentication.
3. The client public key on the server may be outdated.
4. ~root or ~root/.ssh on the remote host may not be owned by root.
5. User may not have passed -shared option for shared remote users or
may be passing the -shared option for non-shared remote users.
6. If there is output in addition to the date, but no password is asked,
it may be a security alert shown as part of company policy. Append the
additional text to the <OMS HOME>/sysman/prov/resources/ignoreMessages.txt file.
------------------------------------------------------------------------
--node1:--
Running /usr/bin/ssh -x -l root node1 date to verify SSH connectivity has been setup from local host to node1.
IF YOU SEE ANY OTHER OUTPUT BESIDES THE OUTPUT OF THE DATE COMMAND OR IF YOU ARE PROMPTED FOR A PASSWORD HERE, IT MEANS SSH SETUP HAS NOT BEEN SUCCESSFUL. Please note that being prompted for a passphrase may be OK but being prompted for a password is ERROR.
Tue Aug 21 07:11:36 CST 2018
------------------------------------------------------------------------
--node2:--
Running /usr/bin/ssh -x -l root node2 date to verify SSH connectivity has been setup from local host to node2.
IF YOU SEE ANY OTHER OUTPUT BESIDES THE OUTPUT OF THE DATE COMMAND OR IF YOU ARE PROMPTED FOR A PASSWORD HERE, IT MEANS SSH SETUP HAS NOT BEEN SUCCESSFUL. Please note that being prompted for a passphrase may be OK but being prompted for a password is ERROR.
Tue Aug 21 07:11:34 CST 2018
------------------------------------------------------------------------
--node3:--
Running /usr/bin/ssh -x -l root node3 date to verify SSH connectivity has been setup from local host to node3.
IF YOU SEE ANY OTHER OUTPUT BESIDES THE OUTPUT OF THE DATE COMMAND OR IF YOU ARE PROMPTED FOR A PASSWORD HERE, IT MEANS SSH SETUP HAS NOT BEEN SUCCESSFUL. Please note that being prompted for a passphrase may be OK but being prompted for a password is ERROR.
Tue Aug 21 07:11:38 CST 2018
------------------------------------------------------------------------
------------------------------------------------------------------------
Verifying SSH connectivity has been setup from node1 to node1
IF YOU SEE ANY OTHER OUTPUT BESIDES THE OUTPUT OF THE DATE COMMAND OR IF YOU ARE PROMPTED FOR A PASSWORD HERE, IT MEANS SSH SETUP HAS NOT BEEN SUCCESSFUL.
bash: -c: line 0: unexpected EOF while looking for matching `"'
bash: -c: line 1: syntax error: unexpected end of file
------------------------------------------------------------------------
------------------------------------------------------------------------
Verifying SSH connectivity has been setup from node1 to node2
IF YOU SEE ANY OTHER OUTPUT BESIDES THE OUTPUT OF THE DATE COMMAND OR IF YOU ARE PROMPTED FOR A PASSWORD HERE, IT MEANS SSH SETUP HAS NOT BEEN SUCCESSFUL.
bash: -c: line 0: unexpected EOF while looking for matching `"'
bash: -c: line 1: syntax error: unexpected end of file
------------------------------------------------------------------------
------------------------------------------------------------------------
Verifying SSH connectivity has been setup from node1 to node3
IF YOU SEE ANY OTHER OUTPUT BESIDES THE OUTPUT OF THE DATE COMMAND OR IF YOU ARE PROMPTED FOR A PASSWORD HERE, IT MEANS SSH SETUP HAS NOT BEEN SUCCESSFUL.
bash: -c: line 0: unexpected EOF while looking for matching `"'
bash: -c: line 1: syntax error: unexpected end of file
------------------------------------------------------------------------
-Verification from complete-
SSH verification complete.

```

## MHA软件安装

### 安装Perl语言相关包(三节点同时操作)

```sql

[root@node1 Packages]# yum install perl*
Loaded plugins: fastestmirror, refresh-packagekit, security
Repository 'Server' is missing name in configuration, using id
Loading mirror speeds from cached hostfile
Setting up Install Process
Examining perl-5.10.1-136.el6.x86_64.rpm: 4:perl-5.10.1-136.el6.x86_64
perl-5.10.1-136.el6.x86_64.rpm: does not update installed package.
Examining perl-Archive-Extract-0.38-136.el6.x86_64.rpm: 1:perl-Archive-Extract-0.38-136.el6.x86_64
Marking perl-Archive-Extract-0.38-136.el6.x86_64.rpm to be installed
Examining perl-Archive-Tar-1.58-136.el6.x86_64.rpm: perl-Archive-Tar-1.58-136.el6.x86_64
Marking perl-Archive-Tar-1.58-136.el6.x86_64.rpm to be installed
Examining perl-Authen-SASL-2.13-3.el6.noarch.rpm: perl-Authen-SASL-2.13-3.el6.noarch
Marking perl-Authen-SASL-2.13-3.el6.noarch.rpm to be installed
Examining perl-Bit-Vector-7.1-2.el6.x86_64.rpm: perl-Bit-Vector-7.1-2.el6.x86_64
Marking perl-Bit-Vector-7.1-2.el6.x86_64.rpm to be installed
Examining perl-BSD-Resource-1.29.03-3.el6.x86_64.rpm: perl-BSD-Resource-1.29.03-3.el6.x86_64
Marking perl-BSD-Resource-1.29.03-3.el6.x86_64.rpm to be installed
Examining perl-Cache-Memcached-1.28-2.el6.noarch.rpm: perl-Cache-Memcached-1.28-2.el6.noarch
Marking perl-Cache-Memcached-1.28-2.el6.noarch.rpm to be installed
Examining perl-Carp-Clan-6.03-2.el6.noarch.rpm: perl-Carp-Clan-6.03-2.el6.noarch
Marking perl-Carp-Clan-6.03-2.el6.noarch.rpm to be installed
Examining perl-CGI-3.51-136.el6.x86_64.rpm: perl-CGI-3.51-136.el6.x86_64
perl-CGI-3.51-136.el6.x86_64.rpm: does not update installed package.
Examining perl-CGI-Session-4.35-6.el6.noarch.rpm: perl-CGI-Session-4.35-6.el6.noarch
Marking perl-CGI-Session-4.35-6.el6.noarch.rpm to be installed
Examining perl-Class-MethodMaker-2.16-6.el6.x86_64.rpm: perl-Class-MethodMaker-2.16-6.el6.x86_64
Marking perl-Class-MethodMaker-2.16-6.el6.x86_64.rpm to be installed
Examining perl-Class-Singleton-1.4-6.el6.noarch.rpm: perl-Class-Singleton-1.4-6.el6.noarch
Marking perl-Class-Singleton-1.4-6.el6.noarch.rpm to be installed
Examining perl-Compress-Raw-Bzip2-2.021-136.el6.x86_64.rpm: perl-Compress-Raw-Bzip2-2.021-136.el6.x86_64
Marking perl-Compress-Raw-Bzip2-2.021-136.el6.x86_64.rpm to be installed
Examining perl-Compress-Raw-Zlib-2.021-136.el6.x86_64.rpm: 1:perl-Compress-Raw-Zlib-2.021-136.el6.x86_64
perl-Compress-Raw-Zlib-2.021-136.el6.x86_64.rpm: does not update installed package.
Examining perl-Compress-Zlib-2.021-136.el6.x86_64.rpm: perl-Compress-Zlib-2.021-136.el6.x86_64
perl-Compress-Zlib-2.021-136.el6.x86_64.rpm: does not update installed package.
Examining perl-Config-General-2.52-1.el6.noarch.rpm: perl-Config-General-2.52-1.el6.noarch
Marking perl-Config-General-2.52-1.el6.noarch.rpm to be installed
Examining perl-Convert-ASN1-0.22-1.el6.noarch.rpm: perl-Convert-ASN1-0.22-1.el6.noarch
Marking perl-Convert-ASN1-0.22-1.el6.noarch.rpm to be installed
Examining perl-core-5.10.1-136.el6.x86_64.rpm: perl-core-5.10.1-136.el6.x86_64
Marking perl-core-5.10.1-136.el6.x86_64.rpm to be installed
Examining perl-CPAN-1.9402-136.el6.x86_64.rpm: perl-CPAN-1.9402-136.el6.x86_64
Marking perl-CPAN-1.9402-136.el6.x86_64.rpm to be installed
Examining perl-CPANPLUS-0.88-136.el6.x86_64.rpm: perl-CPANPLUS-0.88-136.el6.x86_64
Marking perl-CPANPLUS-0.88-136.el6.x86_64.rpm to be installed
Examining perl-Crypt-OpenSSL-Bignum-0.04-8.1.el6.x86_64.rpm: perl-Crypt-OpenSSL-Bignum-0.04-8.1.el6.x86_64
Marking perl-Crypt-OpenSSL-Bignum-0.04-8.1.el6.x86_64.rpm to be installed
Examining perl-Crypt-OpenSSL-Random-0.04-9.1.el6.x86_64.rpm: perl-Crypt-OpenSSL-Random-0.04-9.1.el6.x86_64
Marking perl-Crypt-OpenSSL-Random-0.04-9.1.el6.x86_64.rpm to be installed
Examining perl-Crypt-OpenSSL-RSA-0.25-10.1.el6.x86_64.rpm: perl-Crypt-OpenSSL-RSA-0.25-10.1.el6.x86_64
Marking perl-Crypt-OpenSSL-RSA-0.25-10.1.el6.x86_64.rpm to be installed
Examining perl-Crypt-PasswdMD5-1.3-6.el6.noarch.rpm: perl-Crypt-PasswdMD5-1.3-6.el6.noarch
Marking perl-Crypt-PasswdMD5-1.3-6.el6.noarch.rpm to be installed
Examining perl-Crypt-SSLeay-0.57-17.el6.x86_64.rpm: perl-Crypt-SSLeay-0.57-17.el6.x86_64
Marking perl-Crypt-SSLeay-0.57-17.el6.x86_64.rpm to be installed
Examining perl-Date-Calc-6.3-2.el6.noarch.rpm: perl-Date-Calc-6.3-2.el6.noarch
Marking perl-Date-Calc-6.3-2.el6.noarch.rpm to be installed
Examining perl-Date-Manip-6.24-1.el6.noarch.rpm: perl-Date-Manip-6.24-1.el6.noarch
Marking perl-Date-Manip-6.24-1.el6.noarch.rpm to be installed
Examining perl-DateTime-0.5300-2.el6.x86_64.rpm: 1:perl-DateTime-0.5300-2.el6.x86_64
Marking perl-DateTime-0.5300-2.el6.x86_64.rpm to be installed
Examining perl-DBD-MySQL-4.013-3.el6.x86_64.rpm: perl-DBD-MySQL-4.013-3.el6.x86_64
perl-DBD-MySQL-4.013-3.el6.x86_64.rpm: does not update installed package.
Examining perl-DBD-Pg-2.15.1-4.el6_3.x86_64.rpm: perl-DBD-Pg-2.15.1-4.el6_3.x86_64
perl-DBD-Pg-2.15.1-4.el6_3.x86_64.rpm: does not update installed package.
Examining perl-DBD-SQLite-1.27-3.el6.x86_64.rpm: perl-DBD-SQLite-1.27-3.el6.x86_64
perl-DBD-SQLite-1.27-3.el6.x86_64.rpm: does not update installed package.
Examining perl-DBI-1.609-4.el6.x86_64.rpm: perl-DBI-1.609-4.el6.x86_64
perl-DBI-1.609-4.el6.x86_64.rpm: does not update installed package.
Examining perl-DBIx-Simple-1.32-3.el6.noarch.rpm: perl-DBIx-Simple-1.32-3.el6.noarch
Marking perl-DBIx-Simple-1.32-3.el6.noarch.rpm to be installed
Examining perl-devel-5.10.1-136.el6.i686.rpm: 4:perl-devel-5.10.1-136.el6.i686
Marking perl-devel-5.10.1-136.el6.i686.rpm to be installed
Examining perl-devel-5.10.1-136.el6.x86_64.rpm: 4:perl-devel-5.10.1-136.el6.x86_64
perl-devel-5.10.1-136.el6.x86_64.rpm: does not update installed package.
Examining perl-Devel-Symdump-2.08-2.el6.noarch.rpm: 1:perl-Devel-Symdump-2.08-2.el6.noarch
Marking perl-Devel-Symdump-2.08-2.el6.noarch.rpm to be installed
Examining perl-Digest-HMAC-1.01-22.el6.noarch.rpm: perl-Digest-HMAC-1.01-22.el6.noarch
Marking perl-Digest-HMAC-1.01-22.el6.noarch.rpm to be installed
Examining perl-Digest-SHA1-2.12-2.el6.x86_64.rpm: perl-Digest-SHA1-2.12-2.el6.x86_64
Marking perl-Digest-SHA1-2.12-2.el6.x86_64.rpm to be installed
Examining perl-Digest-SHA-5.47-136.el6.x86_64.rpm: 1:perl-Digest-SHA-5.47-136.el6.x86_64
Marking perl-Digest-SHA-5.47-136.el6.x86_64.rpm to be installed
Examining perl-Encode-Detect-1.01-2.el6.x86_64.rpm: perl-Encode-Detect-1.01-2.el6.x86_64
Marking perl-Encode-Detect-1.01-2.el6.x86_64.rpm to be installed
Examining perl-Error-0.17015-4.el6.noarch.rpm: 1:perl-Error-0.17015-4.el6.noarch
perl-Error-0.17015-4.el6.noarch.rpm: does not update installed package.
Examining perl-ExtUtils-CBuilder-0.27-136.el6.x86_64.rpm: 1:perl-ExtUtils-CBuilder-0.27-136.el6.x86_64
Marking perl-ExtUtils-CBuilder-0.27-136.el6.x86_64.rpm to be installed
Examining perl-ExtUtils-Embed-1.28-136.el6.x86_64.rpm: perl-ExtUtils-Embed-1.28-136.el6.x86_64
Marking perl-ExtUtils-Embed-1.28-136.el6.x86_64.rpm to be installed
Examining perl-ExtUtils-MakeMaker-6.55-136.el6.x86_64.rpm: perl-ExtUtils-MakeMaker-6.55-136.el6.x86_64
perl-ExtUtils-MakeMaker-6.55-136.el6.x86_64.rpm: does not update installed package.
Examining perl-ExtUtils-ParseXS-2.2003.0-136.el6.x86_64.rpm: 1:perl-ExtUtils-ParseXS-2.2003.0-136.el6.x86_64
perl-ExtUtils-ParseXS-2.2003.0-136.el6.x86_64.rpm: does not update installed package.
Examining perl-File-Fetch-0.26-136.el6.x86_64.rpm: perl-File-Fetch-0.26-136.el6.x86_64
Marking perl-File-Fetch-0.26-136.el6.x86_64.rpm to be installed
Examining perl-File-Slurp-9999.13-7.el6.noarch.rpm: perl-File-Slurp-9999.13-7.el6.noarch
Marking perl-File-Slurp-9999.13-7.el6.noarch.rpm to be installed
Examining perl-FreezeThaw-0.45-5.el6.noarch.rpm: perl-FreezeThaw-0.45-5.el6.noarch
Marking perl-FreezeThaw-0.45-5.el6.noarch.rpm to be installed
Examining perl-Frontier-RPC-0.07b4p1-9.el6.noarch.rpm: perl-Frontier-RPC-0.07b4p1-9.el6.noarch
Marking perl-Frontier-RPC-0.07b4p1-9.el6.noarch.rpm to be installed
Examining perl-Frontier-RPC-doc-0.07b4p1-9.el6.noarch.rpm: perl-Frontier-RPC-doc-0.07b4p1-9.el6.noarch
Marking perl-Frontier-RPC-doc-0.07b4p1-9.el6.noarch.rpm to be installed
Examining perl-Git-1.7.1-3.el6_4.1.noarch.rpm: perl-Git-1.7.1-3.el6_4.1.noarch
perl-Git-1.7.1-3.el6_4.1.noarch.rpm: does not update installed package.
Examining perl-GSSAPI-0.26-6.el6.x86_64.rpm: perl-GSSAPI-0.26-6.el6.x86_64
Marking perl-GSSAPI-0.26-6.el6.x86_64.rpm to be installed
Examining perl-hivex-1.3.3-4.2.el6.x86_64.rpm: perl-hivex-1.3.3-4.2.el6.x86_64
Marking perl-hivex-1.3.3-4.2.el6.x86_64.rpm to be installed
Examining perl-HTML-Parser-3.64-2.el6.x86_64.rpm: perl-HTML-Parser-3.64-2.el6.x86_64
perl-HTML-Parser-3.64-2.el6.x86_64.rpm: does not update installed package.
Examining perl-HTML-Tagset-3.20-4.el6.noarch.rpm: perl-HTML-Tagset-3.20-4.el6.noarch
perl-HTML-Tagset-3.20-4.el6.noarch.rpm: does not update installed package.
Examining perl-IO-Compress-Base-2.021-136.el6.x86_64.rpm: perl-IO-Compress-Base-2.021-136.el6.x86_64
perl-IO-Compress-Base-2.021-136.el6.x86_64.rpm: does not update installed package.
Examining perl-IO-Compress-Bzip2-2.021-136.el6.x86_64.rpm: perl-IO-Compress-Bzip2-2.021-136.el6.x86_64
Marking perl-IO-Compress-Bzip2-2.021-136.el6.x86_64.rpm to be installed
Examining perl-IO-Compress-Zlib-2.021-136.el6.x86_64.rpm: perl-IO-Compress-Zlib-2.021-136.el6.x86_64
perl-IO-Compress-Zlib-2.021-136.el6.x86_64.rpm: does not update installed package.
Examining perl-IO-Socket-INET6-2.56-4.el6.noarch.rpm: perl-IO-Socket-INET6-2.56-4.el6.noarch
Marking perl-IO-Socket-INET6-2.56-4.el6.noarch.rpm to be installed
Examining perl-IO-Socket-SSL-1.31-2.el6.noarch.rpm: perl-IO-Socket-SSL-1.31-2.el6.noarch
perl-IO-Socket-SSL-1.31-2.el6.noarch.rpm: does not update installed package.
Examining perl-IO-String-1.08-9.el6.noarch.rpm: perl-IO-String-1.08-9.el6.noarch
Marking perl-IO-String-1.08-9.el6.noarch.rpm to be installed
Examining perl-IO-Zlib-1.09-136.el6.x86_64.rpm: 1:perl-IO-Zlib-1.09-136.el6.x86_64
Marking perl-IO-Zlib-1.09-136.el6.x86_64.rpm to be installed
Examining perl-IPC-Cmd-0.56-136.el6.x86_64.rpm: 1:perl-IPC-Cmd-0.56-136.el6.x86_64
Marking perl-IPC-Cmd-0.56-136.el6.x86_64.rpm to be installed
Examining perl-LDAP-0.40-1.el6.noarch.rpm: 1:perl-LDAP-0.40-1.el6.noarch
Marking perl-LDAP-0.40-1.el6.noarch.rpm to be installed
Examining perl-libintl-1.20-1.el6.x86_64.rpm: perl-libintl-1.20-1.el6.x86_64
Marking perl-libintl-1.20-1.el6.x86_64.rpm to be installed
Examining perl-libs-5.10.1-136.el6.i686.rpm: 4:perl-libs-5.10.1-136.el6.i686
Marking perl-libs-5.10.1-136.el6.i686.rpm to be installed
Examining perl-libs-5.10.1-136.el6.x86_64.rpm: 4:perl-libs-5.10.1-136.el6.x86_64
perl-libs-5.10.1-136.el6.x86_64.rpm: does not update installed package.
Examining perl-libwww-perl-5.833-2.el6.noarch.rpm: perl-libwww-perl-5.833-2.el6.noarch
perl-libwww-perl-5.833-2.el6.noarch.rpm: does not update installed package.
Examining perl-libxml-perl-0.08-10.el6.noarch.rpm: perl-libxml-perl-0.08-10.el6.noarch
Marking perl-libxml-perl-0.08-10.el6.noarch.rpm to be installed
Examining perl-List-MoreUtils-0.22-10.el6.x86_64.rpm: perl-List-MoreUtils-0.22-10.el6.x86_64
Marking perl-List-MoreUtils-0.22-10.el6.x86_64.rpm to be installed
Examining perl-Locale-Maketext-Simple-0.18-136.el6.x86_64.rpm: 1:perl-Locale-Maketext-Simple-0.18-136.el6.x86_64
Marking perl-Locale-Maketext-Simple-0.18-136.el6.x86_64.rpm to be installed
Examining perl-Log-Message-0.02-136.el6.x86_64.rpm: 1:perl-Log-Message-0.02-136.el6.x86_64
Marking perl-Log-Message-0.02-136.el6.x86_64.rpm to be installed
Examining perl-Log-Message-Simple-0.04-136.el6.x86_64.rpm: perl-Log-Message-Simple-0.04-136.el6.x86_64
Marking perl-Log-Message-Simple-0.04-136.el6.x86_64.rpm to be installed
Examining perl-Mail-DKIM-0.37-2.el6.noarch.rpm: perl-Mail-DKIM-0.37-2.el6.noarch
Marking perl-Mail-DKIM-0.37-2.el6.noarch.rpm to be installed
Examining perl-MailTools-2.04-4.el6.noarch.rpm: perl-MailTools-2.04-4.el6.noarch
Marking perl-MailTools-2.04-4.el6.noarch.rpm to be installed
Examining perl-Module-Build-0.3500-136.el6.x86_64.rpm: 1:perl-Module-Build-0.3500-136.el6.x86_64
Marking perl-Module-Build-0.3500-136.el6.x86_64.rpm to be installed
Examining perl-Module-CoreList-2.18-136.el6.x86_64.rpm: perl-Module-CoreList-2.18-136.el6.x86_64
Marking perl-Module-CoreList-2.18-136.el6.x86_64.rpm to be installed
Examining perl-Module-Load-0.16-136.el6.x86_64.rpm: 1:perl-Module-Load-0.16-136.el6.x86_64
Marking perl-Module-Load-0.16-136.el6.x86_64.rpm to be installed
Examining perl-Module-Load-Conditional-0.30-136.el6.x86_64.rpm: perl-Module-Load-Conditional-0.30-136.el6.x86_64
Marking perl-Module-Load-Conditional-0.30-136.el6.x86_64.rpm to be installed
Examining perl-Module-Loaded-0.02-136.el6.x86_64.rpm: 1:perl-Module-Loaded-0.02-136.el6.x86_64
Marking perl-Module-Loaded-0.02-136.el6.x86_64.rpm to be installed
Examining perl-Module-Pluggable-3.90-136.el6.x86_64.rpm: 1:perl-Module-Pluggable-3.90-136.el6.x86_64
perl-Module-Pluggable-3.90-136.el6.x86_64.rpm: does not update installed package.
Examining perl-Mozilla-LDAP-1.5.3-4.el6.x86_64.rpm: perl-Mozilla-LDAP-1.5.3-4.el6.x86_64
Marking perl-Mozilla-LDAP-1.5.3-4.el6.x86_64.rpm to be installed
Examining perl-NetAddr-IP-4.027-7.el6.x86_64.rpm: perl-NetAddr-IP-4.027-7.el6.x86_64
Marking perl-NetAddr-IP-4.027-7.el6.x86_64.rpm to be installed
Examining perl-Net-DNS-0.65-5.el6.x86_64.rpm: perl-Net-DNS-0.65-5.el6.x86_64
Marking perl-Net-DNS-0.65-5.el6.x86_64.rpm to be installed
Examining perl-Net-LibIDN-0.12-3.el6.x86_64.rpm: perl-Net-LibIDN-0.12-3.el6.x86_64
perl-Net-LibIDN-0.12-3.el6.x86_64.rpm: does not update installed package.
Examining perl-Net-SSLeay-1.35-9.el6.x86_64.rpm: perl-Net-SSLeay-1.35-9.el6.x86_64
perl-Net-SSLeay-1.35-9.el6.x86_64.rpm: does not update installed package.
Examining perl-Net-Telnet-3.03-11.el6.noarch.rpm: perl-Net-Telnet-3.03-11.el6.noarch
Marking perl-Net-Telnet-3.03-11.el6.noarch.rpm to be installed
Examining perl-Newt-1.08-26.el6.x86_64.rpm: perl-Newt-1.08-26.el6.x86_64
Marking perl-Newt-1.08-26.el6.x86_64.rpm to be installed
Examining perl-Object-Accessor-0.34-136.el6.x86_64.rpm: 1:perl-Object-Accessor-0.34-136.el6.x86_64
Marking perl-Object-Accessor-0.34-136.el6.x86_64.rpm to be installed
Examining perl-Package-Constants-0.02-136.el6.x86_64.rpm: 1:perl-Package-Constants-0.02-136.el6.x86_64
Marking perl-Package-Constants-0.02-136.el6.x86_64.rpm to be installed
Examining perl-Params-Check-0.26-136.el6.x86_64.rpm: 1:perl-Params-Check-0.26-136.el6.x86_64
Marking perl-Params-Check-0.26-136.el6.x86_64.rpm to be installed
Examining perl-Params-Validate-0.92-3.el6.x86_64.rpm: perl-Params-Validate-0.92-3.el6.x86_64
Marking perl-Params-Validate-0.92-3.el6.x86_64.rpm to be installed
Examining perl-parent-0.221-136.el6.x86_64.rpm: 1:perl-parent-0.221-136.el6.x86_64
Marking perl-parent-0.221-136.el6.x86_64.rpm to be installed
Examining perl-Parse-CPAN-Meta-1.40-136.el6.x86_64.rpm: 1:perl-Parse-CPAN-Meta-1.40-136.el6.x86_64
Marking perl-Parse-CPAN-Meta-1.40-136.el6.x86_64.rpm to be installed
Examining perl-Parse-Yapp-1.05-41.el6.noarch.rpm: perl-Parse-Yapp-1.05-41.el6.noarch
Marking perl-Parse-Yapp-1.05-41.el6.noarch.rpm to be installed
Examining perl-PCP-LogImport-3.9.4-5.el6.x86_64.rpm: perl-PCP-LogImport-3.9.4-5.el6.x86_64
Marking perl-PCP-LogImport-3.9.4-5.el6.x86_64.rpm to be installed
Examining perl-PCP-LogSummary-3.9.4-5.el6.x86_64.rpm: perl-PCP-LogSummary-3.9.4-5.el6.x86_64
Marking perl-PCP-LogSummary-3.9.4-5.el6.x86_64.rpm to be installed
Examining perl-PCP-MMV-3.9.4-5.el6.x86_64.rpm: perl-PCP-MMV-3.9.4-5.el6.x86_64
Marking perl-PCP-MMV-3.9.4-5.el6.x86_64.rpm to be installed
Examining perl-PCP-PMDA-3.9.4-5.el6.x86_64.rpm: perl-PCP-PMDA-3.9.4-5.el6.x86_64
Marking perl-PCP-PMDA-3.9.4-5.el6.x86_64.rpm to be installed
Examining perl-Perlilog-0.3-4.el6.noarch.rpm: perl-Perlilog-0.3-4.el6.noarch
Marking perl-Perlilog-0.3-4.el6.noarch.rpm to be installed
Examining perl-Pod-Coverage-0.20-4.el6.noarch.rpm: perl-Pod-Coverage-0.20-4.el6.noarch
Marking perl-Pod-Coverage-0.20-4.el6.noarch.rpm to be installed
Examining perl-Pod-Escapes-1.04-136.el6.x86_64.rpm: 1:perl-Pod-Escapes-1.04-136.el6.x86_64
perl-Pod-Escapes-1.04-136.el6.x86_64.rpm: does not update installed package.
Examining perl-Pod-Simple-3.13-136.el6.x86_64.rpm: 1:perl-Pod-Simple-3.13-136.el6.x86_64
perl-Pod-Simple-3.13-136.el6.x86_64.rpm: does not update installed package.
Examining perl-SGMLSpm-1.03ii-21.el6.noarch.rpm: perl-SGMLSpm-1.03ii-21.el6.noarch
Marking perl-SGMLSpm-1.03ii-21.el6.noarch.rpm to be installed
Examining perl-SNMP_Session-1.12-4.el6.noarch.rpm: perl-SNMP_Session-1.12-4.el6.noarch
Marking perl-SNMP_Session-1.12-4.el6.noarch.rpm to be installed
Examining perl-Socket6-0.23-4.el6.x86_64.rpm: perl-Socket6-0.23-4.el6.x86_64
Marking perl-Socket6-0.23-4.el6.x86_64.rpm to be installed
Examining perl-String-CRC32-1.4-9.el6.x86_64.rpm: perl-String-CRC32-1.4-9.el6.x86_64
Marking perl-String-CRC32-1.4-9.el6.x86_64.rpm to be installed
Examining perl-suidperl-5.10.1-136.el6.x86_64.rpm: 4:perl-suidperl-5.10.1-136.el6.x86_64
Marking perl-suidperl-5.10.1-136.el6.x86_64.rpm to be installed
Examining perl-Sys-Guestfs-1.20.11-11.el6.x86_64.rpm: 1:perl-Sys-Guestfs-1.20.11-11.el6.x86_64
Marking perl-Sys-Guestfs-1.20.11-11.el6.x86_64.rpm to be installed
Examining perl-Sys-Virt-0.10.2-5.el6.x86_64.rpm: perl-Sys-Virt-0.10.2-5.el6.x86_64
Marking perl-Sys-Virt-0.10.2-5.el6.x86_64.rpm to be installed
Examining perl-Term-ProgressBar-2.09-10.el6.noarch.rpm: perl-Term-ProgressBar-2.09-10.el6.noarch
Marking perl-Term-ProgressBar-2.09-10.el6.noarch.rpm to be installed
Examining perl-TermReadKey-2.30-13.el6.x86_64.rpm: perl-TermReadKey-2.30-13.el6.x86_64
perl-TermReadKey-2.30-13.el6.x86_64.rpm: does not update installed package.
Examining perl-Term-UI-0.20-136.el6.x86_64.rpm: perl-Term-UI-0.20-136.el6.x86_64
Marking perl-Term-UI-0.20-136.el6.x86_64.rpm to be installed
Examining perl-Test-Harness-3.17-136.el6.x86_64.rpm: perl-Test-Harness-3.17-136.el6.x86_64
perl-Test-Harness-3.17-136.el6.x86_64.rpm: does not update installed package.
Examining perl-Test-Pod-1.40-1.el6.noarch.rpm: perl-Test-Pod-1.40-1.el6.noarch
Marking perl-Test-Pod-1.40-1.el6.noarch.rpm to be installed
Examining perl-Test-Pod-Coverage-1.08-8.1.el6.noarch.rpm: perl-Test-Pod-Coverage-1.08-8.1.el6.noarch
Marking perl-Test-Pod-Coverage-1.08-8.1.el6.noarch.rpm to be installed
Examining perl-Test-Simple-0.92-136.el6.x86_64.rpm: perl-Test-Simple-0.92-136.el6.x86_64
perl-Test-Simple-0.92-136.el6.x86_64.rpm: does not update installed package.
Examining perl-Text-Iconv-1.7-6.el6.x86_64.rpm: perl-Text-Iconv-1.7-6.el6.x86_64
Marking perl-Text-Iconv-1.7-6.el6.x86_64.rpm to be installed
Examining perltidy-20090616-2.1.el6.noarch.rpm: perltidy-20090616-2.1.el6.noarch
Marking perltidy-20090616-2.1.el6.noarch.rpm to be installed
Examining perl-TimeDate-1.16-13.el6.noarch.rpm: 1:perl-TimeDate-1.16-13.el6.noarch
Marking perl-TimeDate-1.16-13.el6.noarch.rpm to be installed
Examining perl-Time-HiRes-1.9721-136.el6.x86_64.rpm: 4:perl-Time-HiRes-1.9721-136.el6.x86_64
perl-Time-HiRes-1.9721-136.el6.x86_64.rpm: does not update installed package.
Examining perl-Time-Piece-1.15-136.el6.x86_64.rpm: perl-Time-Piece-1.15-136.el6.x86_64
Marking perl-Time-Piece-1.15-136.el6.x86_64.rpm to be installed
Examining perl-URI-1.40-2.el6.noarch.rpm: perl-URI-1.40-2.el6.noarch
perl-URI-1.40-2.el6.noarch.rpm: does not update installed package.
Examining perl-version-0.77-136.el6.x86_64.rpm: 3:perl-version-0.77-136.el6.x86_64
perl-version-0.77-136.el6.x86_64.rpm: does not update installed package.
Examining perl-XML-DOM-1.44-7.el6.noarch.rpm: perl-XML-DOM-1.44-7.el6.noarch
Marking perl-XML-DOM-1.44-7.el6.noarch.rpm to be installed
Examining perl-XML-DOM-XPath-0.14-4.el6.noarch.rpm: perl-XML-DOM-XPath-0.14-4.el6.noarch
Marking perl-XML-DOM-XPath-0.14-4.el6.noarch.rpm to be installed
Examining perl-XML-Dumper-0.81-8.el6.noarch.rpm: perl-XML-Dumper-0.81-8.el6.noarch
Marking perl-XML-Dumper-0.81-8.el6.noarch.rpm to be installed
Examining perl-XML-Filter-BufferText-1.01-8.el6.noarch.rpm: perl-XML-Filter-BufferText-1.01-8.el6.noarch
Marking perl-XML-Filter-BufferText-1.01-8.el6.noarch.rpm to be installed
Examining perl-XML-Grove-0.46alpha-40.el6.noarch.rpm: perl-XML-Grove-0.46alpha-40.el6.noarch
Marking perl-XML-Grove-0.46alpha-40.el6.noarch.rpm to be installed
Examining perl-XML-LibXML-1.70-5.el6.x86_64.rpm: 1:perl-XML-LibXML-1.70-5.el6.x86_64
Marking perl-XML-LibXML-1.70-5.el6.x86_64.rpm to be installed
Examining perl-XML-NamespaceSupport-1.10-3.el6.noarch.rpm: perl-XML-NamespaceSupport-1.10-3.el6.noarch
Marking perl-XML-NamespaceSupport-1.10-3.el6.noarch.rpm to be installed
Examining perl-XML-Parser-2.36-7.el6.x86_64.rpm: perl-XML-Parser-2.36-7.el6.x86_64
perl-XML-Parser-2.36-7.el6.x86_64.rpm: does not update installed package.
Examining perl-XML-RegExp-0.03-7.el6.noarch.rpm: perl-XML-RegExp-0.03-7.el6.noarch
Marking perl-XML-RegExp-0.03-7.el6.noarch.rpm to be installed
Examining perl-XML-SAX-0.96-7.el6.noarch.rpm: perl-XML-SAX-0.96-7.el6.noarch
Marking perl-XML-SAX-0.96-7.el6.noarch.rpm to be installed
Examining perl-XML-SAX-Writer-0.50-8.el6.noarch.rpm: perl-XML-SAX-Writer-0.50-8.el6.noarch
Marking perl-XML-SAX-Writer-0.50-8.el6.noarch.rpm to be installed
Examining perl-XML-Twig-3.34-1.el6.noarch.rpm: perl-XML-Twig-3.34-1.el6.noarch
Marking perl-XML-Twig-3.34-1.el6.noarch.rpm to be installed
Examining perl-XML-Writer-0.606-6.el6.noarch.rpm: perl-XML-Writer-0.606-6.el6.noarch
Marking perl-XML-Writer-0.606-6.el6.noarch.rpm to be installed
Examining perl-XML-XPath-1.13-10.el6.noarch.rpm: perl-XML-XPath-1.13-10.el6.noarch
Marking perl-XML-XPath-1.13-10.el6.noarch.rpm to be installed
Examining perl-XML-XPathEngine-0.12-3.el6.noarch.rpm: perl-XML-XPathEngine-0.12-3.el6.noarch
Marking perl-XML-XPathEngine-0.12-3.el6.noarch.rpm to be installed
Examining perl-YAML-Syck-1.07-4.el6.x86_64.rpm: perl-YAML-Syck-1.07-4.el6.x86_64
Marking perl-YAML-Syck-1.07-4.el6.x86_64.rpm to be installed
Examining perl-YAML-Tiny-1.40-2.el6.noarch.rpm: perl-YAML-Tiny-1.40-2.el6.noarch
Marking perl-YAML-Tiny-1.40-2.el6.noarch.rpm to be installed
Resolving Dependencies
--> Running transaction check
---> Package perl-Archive-Extract.x86_64 1:0.38-136.el6 will be installed
---> Package perl-Archive-Tar.x86_64 0:1.58-136.el6 will be installed
---> Package perl-Authen-SASL.noarch 0:2.13-3.el6 will be installed
---> Package perl-BSD-Resource.x86_64 0:1.29.03-3.el6 will be installed
---> Package perl-Bit-Vector.x86_64 0:7.1-2.el6 will be installed
---> Package perl-CGI-Session.noarch 0:4.35-6.el6 will be installed
---> Package perl-CPAN.x86_64 0:1.9402-136.el6 will be installed
---> Package perl-CPANPLUS.x86_64 0:0.88-136.el6 will be installed
---> Package perl-Cache-Memcached.noarch 0:1.28-2.el6 will be installed
---> Package perl-Carp-Clan.noarch 0:6.03-2.el6 will be installed
---> Package perl-Class-MethodMaker.x86_64 0:2.16-6.el6 will be installed
---> Package perl-Class-Singleton.noarch 0:1.4-6.el6 will be installed
---> Package perl-Compress-Raw-Bzip2.x86_64 0:2.021-136.el6 will be installed
---> Package perl-Config-General.noarch 0:2.52-1.el6 will be installed
---> Package perl-Convert-ASN1.noarch 0:0.22-1.el6 will be installed
---> Package perl-Crypt-OpenSSL-Bignum.x86_64 0:0.04-8.1.el6 will be installed
---> Package perl-Crypt-OpenSSL-RSA.x86_64 0:0.25-10.1.el6 will be installed
---> Package perl-Crypt-OpenSSL-Random.x86_64 0:0.04-9.1.el6 will be installed
---> Package perl-Crypt-PasswdMD5.noarch 0:1.3-6.el6 will be installed
---> Package perl-Crypt-SSLeay.x86_64 0:0.57-17.el6 will be installed
---> Package perl-DBIx-Simple.noarch 0:1.32-3.el6 will be installed
---> Package perl-Date-Calc.noarch 0:6.3-2.el6 will be installed
---> Package perl-Date-Manip.noarch 0:6.24-1.el6 will be installed
---> Package perl-DateTime.x86_64 1:0.5300-2.el6 will be installed
---> Package perl-Devel-Symdump.noarch 1:2.08-2.el6 will be installed
---> Package perl-Digest-HMAC.noarch 0:1.01-22.el6 will be installed
---> Package perl-Digest-SHA.x86_64 1:5.47-136.el6 will be installed
---> Package perl-Digest-SHA1.x86_64 0:2.12-2.el6 will be installed
---> Package perl-Encode-Detect.x86_64 0:1.01-2.el6 will be installed
---> Package perl-ExtUtils-CBuilder.x86_64 1:0.27-136.el6 will be installed
---> Package perl-ExtUtils-Embed.x86_64 0:1.28-136.el6 will be installed
---> Package perl-File-Fetch.x86_64 0:0.26-136.el6 will be installed
---> Package perl-File-Slurp.noarch 0:9999.13-7.el6 will be installed
---> Package perl-FreezeThaw.noarch 0:0.45-5.el6 will be installed
---> Package perl-Frontier-RPC.noarch 0:0.07b4p1-9.el6 will be installed
--> Processing Dependency: perl(Apache2::Const) for package: perl-Frontier-RPC-0.07b4p1-9.el6.noarch
--> Processing Dependency: perl(Apache2::ServerUtil) for package: perl-Frontier-RPC-0.07b4p1-9.el6.noarch
---> Package perl-Frontier-RPC-doc.noarch 0:0.07b4p1-9.el6 will be installed
---> Package perl-GSSAPI.x86_64 0:0.26-6.el6 will be installed
---> Package perl-IO-Compress-Bzip2.x86_64 0:2.021-136.el6 will be installed
---> Package perl-IO-Socket-INET6.noarch 0:2.56-4.el6 will be installed
---> Package perl-IO-String.noarch 0:1.08-9.el6 will be installed
---> Package perl-IO-Zlib.x86_64 1:1.09-136.el6 will be installed
---> Package perl-IPC-Cmd.x86_64 1:0.56-136.el6 will be installed
---> Package perl-LDAP.noarch 1:0.40-1.el6 will be installed
---> Package perl-List-MoreUtils.x86_64 0:0.22-10.el6 will be installed
---> Package perl-Locale-Maketext-Simple.x86_64 1:0.18-136.el6 will be installed
---> Package perl-Log-Message.x86_64 1:0.02-136.el6 will be installed
---> Package perl-Log-Message-Simple.x86_64 0:0.04-136.el6 will be installed
---> Package perl-Mail-DKIM.noarch 0:0.37-2.el6 will be installed
---> Package perl-MailTools.noarch 0:2.04-4.el6 will be installed
---> Package perl-Module-Build.x86_64 1:0.3500-136.el6 will be installed
---> Package perl-Module-CoreList.x86_64 0:2.18-136.el6 will be installed
---> Package perl-Module-Load.x86_64 1:0.16-136.el6 will be installed
---> Package perl-Module-Load-Conditional.x86_64 0:0.30-136.el6 will be installed
---> Package perl-Module-Loaded.x86_64 1:0.02-136.el6 will be installed
---> Package perl-Mozilla-LDAP.x86_64 0:1.5.3-4.el6 will be installed
---> Package perl-Net-DNS.x86_64 0:0.65-5.el6 will be installed
---> Package perl-Net-Telnet.noarch 0:3.03-11.el6 will be installed
---> Package perl-NetAddr-IP.x86_64 0:4.027-7.el6 will be installed
---> Package perl-Newt.x86_64 0:1.08-26.el6 will be installed
---> Package perl-Object-Accessor.x86_64 1:0.34-136.el6 will be installed
---> Package perl-PCP-LogImport.x86_64 0:3.9.4-5.el6 will be installed
--> Processing Dependency: pcp-libs = 3.9.4-5.el6 for package: perl-PCP-LogImport-3.9.4-5.el6.x86_64
--> Processing Dependency: libpcp_import.so.1(PCP_IMPORT_1.0)(64bit) for package: perl-PCP-LogImport-3.9.4-5.el6.x86_64
--> Processing Dependency: libpcp.so.3()(64bit) for package: perl-PCP-LogImport-3.9.4-5.el6.x86_64
--> Processing Dependency: libpcp_import.so.1()(64bit) for package: perl-PCP-LogImport-3.9.4-5.el6.x86_64
---> Package perl-PCP-LogSummary.x86_64 0:3.9.4-5.el6 will be installed
---> Package perl-PCP-MMV.x86_64 0:3.9.4-5.el6 will be installed
---> Package perl-PCP-PMDA.x86_64 0:3.9.4-5.el6 will be installed
---> Package perl-Package-Constants.x86_64 1:0.02-136.el6 will be installed
---> Package perl-Params-Check.x86_64 1:0.26-136.el6 will be installed
---> Package perl-Params-Validate.x86_64 0:0.92-3.el6 will be installed
---> Package perl-Parse-CPAN-Meta.x86_64 1:1.40-136.el6 will be installed
---> Package perl-Parse-Yapp.noarch 0:1.05-41.el6 will be installed
---> Package perl-Perlilog.noarch 0:0.3-4.el6 will be installed
---> Package perl-Pod-Coverage.noarch 0:0.20-4.el6 will be installed
---> Package perl-SGMLSpm.noarch 0:1.03ii-21.el6 will be installed
--> Processing Dependency: openjade for package: perl-SGMLSpm-1.03ii-21.el6.noarch
---> Package perl-SNMP_Session.noarch 0:1.12-4.el6 will be installed
---> Package perl-Socket6.x86_64 0:0.23-4.el6 will be installed
---> Package perl-String-CRC32.x86_64 0:1.4-9.el6 will be installed
---> Package perl-Sys-Guestfs.x86_64 1:1.20.11-11.el6 will be installed
--> Processing Dependency: libguestfs = 1:1.20.11-11.el6 for package: 1:perl-Sys-Guestfs-1.20.11-11.el6.x86_64
--> Processing Dependency: libguestfs.so.0()(64bit) for package: 1:perl-Sys-Guestfs-1.20.11-11.el6.x86_64
---> Package perl-Sys-Virt.x86_64 0:0.10.2-5.el6 will be installed
--> Processing Dependency: libvirt.so.0(LIBVIRT_0.0.3)(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
--> Processing Dependency: libvirt.so.0(LIBVIRT_0.0.5)(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
--> Processing Dependency: libvirt.so.0(LIBVIRT_0.1.0)(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
--> Processing Dependency: libvirt.so.0(LIBVIRT_0.1.1)(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
--> Processing Dependency: libvirt.so.0(LIBVIRT_0.1.4)(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
--> Processing Dependency: libvirt.so.0(LIBVIRT_0.1.5)(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
--> Processing Dependency: libvirt.so.0(LIBVIRT_0.1.9)(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
--> Processing Dependency: libvirt.so.0(LIBVIRT_0.10.0)(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
--> Processing Dependency: libvirt.so.0(LIBVIRT_0.10.2)(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
--> Processing Dependency: libvirt.so.0(LIBVIRT_0.2.0)(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
--> Processing Dependency: libvirt.so.0(LIBVIRT_0.2.1)(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
--> Processing Dependency: libvirt.so.0(LIBVIRT_0.2.3)(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
--> Processing Dependency: libvirt.so.0(LIBVIRT_0.3.0)(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
--> Processing Dependency: libvirt.so.0(LIBVIRT_0.3.2)(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
--> Processing Dependency: libvirt.so.0(LIBVIRT_0.3.3)(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
--> Processing Dependency: libvirt.so.0(LIBVIRT_0.4.0)(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
--> Processing Dependency: libvirt.so.0(LIBVIRT_0.4.1)(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
--> Processing Dependency: libvirt.so.0(LIBVIRT_0.4.2)(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
--> Processing Dependency: libvirt.so.0(LIBVIRT_0.4.5)(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
--> Processing Dependency: libvirt.so.0(LIBVIRT_0.5.0)(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
--> Processing Dependency: libvirt.so.0(LIBVIRT_0.6.0)(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
--> Processing Dependency: libvirt.so.0(LIBVIRT_0.6.1)(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
--> Processing Dependency: libvirt.so.0(LIBVIRT_0.6.3)(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
--> Processing Dependency: libvirt.so.0(LIBVIRT_0.6.4)(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
--> Processing Dependency: libvirt.so.0(LIBVIRT_0.7.0)(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
--> Processing Dependency: libvirt.so.0(LIBVIRT_0.7.1)(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
--> Processing Dependency: libvirt.so.0(LIBVIRT_0.7.2)(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
--> Processing Dependency: libvirt.so.0(LIBVIRT_0.7.3)(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
--> Processing Dependency: libvirt.so.0(LIBVIRT_0.7.5)(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
--> Processing Dependency: libvirt.so.0(LIBVIRT_0.7.7)(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
--> Processing Dependency: libvirt.so.0(LIBVIRT_0.8.0)(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
--> Processing Dependency: libvirt.so.0(LIBVIRT_0.8.1)(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
--> Processing Dependency: libvirt.so.0(LIBVIRT_0.8.2)(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
--> Processing Dependency: libvirt.so.0(LIBVIRT_0.8.5)(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
--> Processing Dependency: libvirt.so.0(LIBVIRT_0.8.6)(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
--> Processing Dependency: libvirt.so.0(LIBVIRT_0.8.8)(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
--> Processing Dependency: libvirt.so.0(LIBVIRT_0.9.0)(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
--> Processing Dependency: libvirt.so.0(LIBVIRT_0.9.10)(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
--> Processing Dependency: libvirt.so.0(LIBVIRT_0.9.11)(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
--> Processing Dependency: libvirt.so.0(LIBVIRT_0.9.13)(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
--> Processing Dependency: libvirt.so.0(LIBVIRT_0.9.2)(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
--> Processing Dependency: libvirt.so.0(LIBVIRT_0.9.3)(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
--> Processing Dependency: libvirt.so.0(LIBVIRT_0.9.4)(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
--> Processing Dependency: libvirt.so.0(LIBVIRT_0.9.5)(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
--> Processing Dependency: libvirt.so.0(LIBVIRT_0.9.7)(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
--> Processing Dependency: libvirt.so.0(LIBVIRT_0.9.8)(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
--> Processing Dependency: libvirt.so.0(LIBVIRT_0.9.9)(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
--> Processing Dependency: libvirt.so.0()(64bit) for package: perl-Sys-Virt-0.10.2-5.el6.x86_64
---> Package perl-Term-ProgressBar.noarch 0:2.09-10.el6 will be installed
---> Package perl-Term-UI.x86_64 0:0.20-136.el6 will be installed
---> Package perl-Test-Pod.noarch 0:1.40-1.el6 will be installed
---> Package perl-Test-Pod-Coverage.noarch 0:1.08-8.1.el6 will be installed
---> Package perl-Text-Iconv.x86_64 0:1.7-6.el6 will be installed
---> Package perl-Time-Piece.x86_64 0:1.15-136.el6 will be installed
---> Package perl-TimeDate.noarch 1:1.16-13.el6 will be installed
---> Package perl-XML-DOM.noarch 0:1.44-7.el6 will be installed
---> Package perl-XML-DOM-XPath.noarch 0:0.14-4.el6 will be installed
---> Package perl-XML-Dumper.noarch 0:0.81-8.el6 will be installed
---> Package perl-XML-Filter-BufferText.noarch 0:1.01-8.el6 will be installed
---> Package perl-XML-Grove.noarch 0:0.46alpha-40.el6 will be installed
---> Package perl-XML-LibXML.x86_64 1:1.70-5.el6 will be installed
---> Package perl-XML-NamespaceSupport.noarch 0:1.10-3.el6 will be installed
---> Package perl-XML-RegExp.noarch 0:0.03-7.el6 will be installed
---> Package perl-XML-SAX.noarch 0:0.96-7.el6 will be installed
---> Package perl-XML-SAX-Writer.noarch 0:0.50-8.el6 will be installed
---> Package perl-XML-Twig.noarch 0:3.34-1.el6 will be installed
---> Package perl-XML-Writer.noarch 0:0.606-6.el6 will be installed
---> Package perl-XML-XPath.noarch 0:1.13-10.el6 will be installed
---> Package perl-XML-XPathEngine.noarch 0:0.12-3.el6 will be installed
---> Package perl-YAML-Syck.x86_64 0:1.07-4.el6 will be installed
---> Package perl-YAML-Tiny.noarch 0:1.40-2.el6 will be installed
---> Package perl-core.x86_64 0:5.10.1-136.el6 will be installed
---> Package perl-devel.i686 4:5.10.1-136.el6 will be installed
---> Package perl-hivex.x86_64 0:1.3.3-4.2.el6 will be installed
--> Processing Dependency: hivex = 1.3.3-4.2.el6 for package: perl-hivex-1.3.3-4.2.el6.x86_64
--> Processing Dependency: libhivex.so.0()(64bit) for package: perl-hivex-1.3.3-4.2.el6.x86_64
---> Package perl-libintl.x86_64 0:1.20-1.el6 will be installed
---> Package perl-libs.i686 4:5.10.1-136.el6 will be installed
--> Processing Dependency: libc.so.6 for package: 4:perl-libs-5.10.1-136.el6.i686
--> Processing Dependency: libc.so.6(GLIBC_2.0) for package: 4:perl-libs-5.10.1-136.el6.i686
--> Processing Dependency: libc.so.6(GLIBC_2.1) for package: 4:perl-libs-5.10.1-136.el6.i686
--> Processing Dependency: libc.so.6(GLIBC_2.1.2) for package: 4:perl-libs-5.10.1-136.el6.i686
--> Processing Dependency: libc.so.6(GLIBC_2.1.3) for package: 4:perl-libs-5.10.1-136.el6.i686
--> Processing Dependency: libc.so.6(GLIBC_2.11) for package: 4:perl-libs-5.10.1-136.el6.i686
--> Processing Dependency: libc.so.6(GLIBC_2.2) for package: 4:perl-libs-5.10.1-136.el6.i686
--> Processing Dependency: libc.so.6(GLIBC_2.3) for package: 4:perl-libs-5.10.1-136.el6.i686
--> Processing Dependency: libc.so.6(GLIBC_2.3.4) for package: 4:perl-libs-5.10.1-136.el6.i686
--> Processing Dependency: libc.so.6(GLIBC_2.4) for package: 4:perl-libs-5.10.1-136.el6.i686
--> Processing Dependency: libcrypt.so.1 for package: 4:perl-libs-5.10.1-136.el6.i686
--> Processing Dependency: libcrypt.so.1(GLIBC_2.0) for package: 4:perl-libs-5.10.1-136.el6.i686
--> Processing Dependency: libdl.so.2 for package: 4:perl-libs-5.10.1-136.el6.i686
--> Processing Dependency: libdl.so.2(GLIBC_2.0) for package: 4:perl-libs-5.10.1-136.el6.i686
--> Processing Dependency: libdl.so.2(GLIBC_2.1) for package: 4:perl-libs-5.10.1-136.el6.i686
--> Processing Dependency: libm.so.6 for package: 4:perl-libs-5.10.1-136.el6.i686
--> Processing Dependency: libm.so.6(GLIBC_2.0) for package: 4:perl-libs-5.10.1-136.el6.i686
--> Processing Dependency: libnsl.so.1 for package: 4:perl-libs-5.10.1-136.el6.i686
--> Processing Dependency: libpthread.so.0 for package: 4:perl-libs-5.10.1-136.el6.i686
--> Processing Dependency: libpthread.so.0(GLIBC_2.0) for package: 4:perl-libs-5.10.1-136.el6.i686
--> Processing Dependency: libpthread.so.0(GLIBC_2.2) for package: 4:perl-libs-5.10.1-136.el6.i686
--> Processing Dependency: libresolv.so.2 for package: 4:perl-libs-5.10.1-136.el6.i686
--> Processing Dependency: libutil.so.1 for package: 4:perl-libs-5.10.1-136.el6.i686
---> Package perl-libxml-perl.noarch 0:0.08-10.el6 will be installed
---> Package perl-parent.x86_64 1:0.221-136.el6 will be installed
---> Package perl-suidperl.x86_64 4:5.10.1-136.el6 will be installed
---> Package perltidy.noarch 0:20090616-2.1.el6 will be installed
--> Running transaction check
---> Package glibc.x86_64 0:2.12-1.132.el6 will be updated
--> Processing Dependency: glibc = 2.12-1.132.el6 for package: glibc-headers-2.12-1.132.el6.x86_64
--> Processing Dependency: glibc = 2.12-1.132.el6 for package: glibc-common-2.12-1.132.el6.x86_64
--> Processing Dependency: glibc = 2.12-1.132.el6 for package: glibc-devel-2.12-1.132.el6.x86_64
---> Package glibc.i686 0:2.12-1.149.el6 will be installed
--> Processing Dependency: libfreebl3.so(NSSRAWHASH_3.12.3) for package: glibc-2.12-1.149.el6.i686
--> Processing Dependency: libfreebl3.so for package: glibc-2.12-1.149.el6.i686
---> Package glibc.x86_64 0:2.12-1.149.el6 will be an update
---> Package hivex.x86_64 0:1.3.3-4.2.el6 will be installed
---> Package libguestfs.x86_64 1:1.20.11-11.el6 will be installed
--> Processing Dependency: qemu-kvm >= 2:0.12.1.0 for package: 1:libguestfs-1.20.11-11.el6.x86_64
--> Processing Dependency: febootstrap-supermin-helper >= 3.21-4 for package: 1:libguestfs-1.20.11-11.el6.x86_64
--> Processing Dependency: augeas-libs >= 1.0.0-5 for package: 1:libguestfs-1.20.11-11.el6.x86_64
--> Processing Dependency: yajl for package: 1:libguestfs-1.20.11-11.el6.x86_64
--> Processing Dependency: scrub for package: 1:libguestfs-1.20.11-11.el6.x86_64
--> Processing Dependency: netpbm-progs for package: 1:libguestfs-1.20.11-11.el6.x86_64
--> Processing Dependency: lzop for package: 1:libguestfs-1.20.11-11.el6.x86_64
--> Processing Dependency: gdisk for package: 1:libguestfs-1.20.11-11.el6.x86_64
--> Processing Dependency: btrfs-progs for package: 1:libguestfs-1.20.11-11.el6.x86_64
--> Processing Dependency: /usr/lib64/libyajl.so.1 for package: 1:libguestfs-1.20.11-11.el6.x86_64
--> Processing Dependency: /usr/lib64/libfa.so.1 for package: 1:libguestfs-1.20.11-11.el6.x86_64
--> Processing Dependency: /usr/lib64/libaugeas.so.0 for package: 1:libguestfs-1.20.11-11.el6.x86_64
---> Package libvirt-client.x86_64 0:0.10.2-46.el6 will be installed
--> Processing Dependency: nc for package: libvirt-client-0.10.2-46.el6.x86_64
--> Processing Dependency: gnutls-utils for package: libvirt-client-0.10.2-46.el6.x86_64
---> Package mod_perl.x86_64 0:2.0.4-11.el6_5 will be installed
---> Package openjade.x86_64 0:1.3.2-36.el6 will be installed
--> Processing Dependency: libosp.so.5()(64bit) for package: openjade-1.3.2-36.el6.x86_64
---> Package pcp-libs.x86_64 0:3.9.4-5.el6 will be installed
--> Processing Dependency: pcp-conf = 3.9.4-5.el6 for package: pcp-libs-3.9.4-5.el6.x86_64
--> Running transaction check
---> Package augeas-libs.x86_64 0:1.0.0-7.el6 will be installed
---> Package btrfs-progs.x86_64 0:0.20-0.2.git91d9eec.el6 will be installed
---> Package febootstrap-supermin-helper.x86_64 0:3.21-4.el6 will be installed
---> Package gdisk.x86_64 0:0.8.10-1.el6 will be installed
---> Package glibc-common.x86_64 0:2.12-1.132.el6 will be updated
---> Package glibc-common.x86_64 0:2.12-1.149.el6 will be an update
---> Package glibc-devel.x86_64 0:2.12-1.132.el6 will be updated
---> Package glibc-devel.x86_64 0:2.12-1.149.el6 will be an update
---> Package glibc-headers.x86_64 0:2.12-1.132.el6 will be updated
---> Package glibc-headers.x86_64 0:2.12-1.149.el6 will be an update
---> Package gnutls-utils.x86_64 0:2.8.5-14.el6_5 will be installed
--> Processing Dependency: gnutls = 2.8.5-14.el6_5 for package: gnutls-utils-2.8.5-14.el6_5.x86_64
---> Package lzop.x86_64 0:1.02-0.9.rc1.el6 will be installed
---> Package nc.x86_64 0:1.84-22.el6 will be installed
---> Package netpbm-progs.x86_64 0:10.47.05-11.el6 will be installed
--> Processing Dependency: netpbm = 10.47.05-11.el6 for package: netpbm-progs-10.47.05-11.el6.x86_64
--> Processing Dependency: libnetpbm.so.10()(64bit) for package: netpbm-progs-10.47.05-11.el6.x86_64
---> Package nss-softokn-freebl.x86_64 0:3.14.3-9.el6 will be updated
---> Package nss-softokn-freebl.i686 0:3.14.3-17.el6 will be installed
---> Package nss-softokn-freebl.x86_64 0:3.14.3-17.el6 will be an update
---> Package opensp.x86_64 0:1.5.2-12.1.el6 will be installed
---> Package pcp-conf.x86_64 0:3.9.4-5.el6 will be installed
---> Package qemu-kvm.x86_64 2:0.12.1.2-2.445.el6 will be installed
--> Processing Dependency: qemu-img = 2:0.12.1.2-2.445.el6 for package: 2:qemu-kvm-0.12.1.2-2.445.el6.x86_64
--> Processing Dependency: seabios >= 0.6.1.2-20.el6 for package: 2:qemu-kvm-0.12.1.2-2.445.el6.x86_64
--> Processing Dependency: vgabios-vmware for package: 2:qemu-kvm-0.12.1.2-2.445.el6.x86_64
--> Processing Dependency: vgabios-stdvga for package: 2:qemu-kvm-0.12.1.2-2.445.el6.x86_64
--> Processing Dependency: vgabios-qxl for package: 2:qemu-kvm-0.12.1.2-2.445.el6.x86_64
--> Processing Dependency: vgabios for package: 2:qemu-kvm-0.12.1.2-2.445.el6.x86_64
--> Processing Dependency: libspice-server.so.1(SPICE_SERVER_0.8.3)(64bit) for package: 2:qemu-kvm-0.12.1.2-2.445.el6.x86_64
--> Processing Dependency: libspice-server.so.1(SPICE_SERVER_0.8.2)(64bit) for package: 2:qemu-kvm-0.12.1.2-2.445.el6.x86_64
--> Processing Dependency: libspice-server.so.1(SPICE_SERVER_0.8.1)(64bit) for package: 2:qemu-kvm-0.12.1.2-2.445.el6.x86_64
--> Processing Dependency: libspice-server.so.1(SPICE_SERVER_0.6.0)(64bit) for package: 2:qemu-kvm-0.12.1.2-2.445.el6.x86_64
--> Processing Dependency: libspice-server.so.1(SPICE_SERVER_0.12.4)(64bit) for package: 2:qemu-kvm-0.12.1.2-2.445.el6.x86_64
--> Processing Dependency: libspice-server.so.1(SPICE_SERVER_0.11.2)(64bit) for package: 2:qemu-kvm-0.12.1.2-2.445.el6.x86_64
--> Processing Dependency: libspice-server.so.1(SPICE_SERVER_0.10.4)(64bit) for package: 2:qemu-kvm-0.12.1.2-2.445.el6.x86_64
--> Processing Dependency: libspice-server.so.1(SPICE_SERVER_0.10.0)(64bit) for package: 2:qemu-kvm-0.12.1.2-2.445.el6.x86_64
--> Processing Dependency: glusterfs-api for package: 2:qemu-kvm-0.12.1.2-2.445.el6.x86_64
--> Processing Dependency: /usr/share/sgabios/sgabios.bin for package: 2:qemu-kvm-0.12.1.2-2.445.el6.x86_64
--> Processing Dependency: /usr/share/gpxe/virtio-net.rom for package: 2:qemu-kvm-0.12.1.2-2.445.el6.x86_64
--> Processing Dependency: /usr/share/gpxe/rtl8139.rom for package: 2:qemu-kvm-0.12.1.2-2.445.el6.x86_64
--> Processing Dependency: /usr/share/gpxe/rtl8029.rom for package: 2:qemu-kvm-0.12.1.2-2.445.el6.x86_64
--> Processing Dependency: /usr/share/gpxe/pcnet32.rom for package: 2:qemu-kvm-0.12.1.2-2.445.el6.x86_64
--> Processing Dependency: /usr/share/gpxe/e1000-0x100e.rom for package: 2:qemu-kvm-0.12.1.2-2.445.el6.x86_64
--> Processing Dependency: libusbredirparser.so.1()(64bit) for package: 2:qemu-kvm-0.12.1.2-2.445.el6.x86_64
--> Processing Dependency: libspice-server.so.1()(64bit) for package: 2:qemu-kvm-0.12.1.2-2.445.el6.x86_64
--> Processing Dependency: libgfxdr.so.0()(64bit) for package: 2:qemu-kvm-0.12.1.2-2.445.el6.x86_64
--> Processing Dependency: libgfrpc.so.0()(64bit) for package: 2:qemu-kvm-0.12.1.2-2.445.el6.x86_64
--> Processing Dependency: libgfapi.so.0()(64bit) for package: 2:qemu-kvm-0.12.1.2-2.445.el6.x86_64
---> Package scrub.x86_64 0:2.2-2.el6 will be installed
---> Package yajl.x86_64 0:1.0.7-3.el6 will be installed
--> Running transaction check
---> Package glusterfs-api.x86_64 0:3.6.0.28-2.el6 will be installed
--> Processing Dependency: glusterfs = 3.6.0.28-2.el6 for package: glusterfs-api-3.6.0.28-2.el6.x86_64
---> Package glusterfs-libs.x86_64 0:3.6.0.28-2.el6 will be installed
---> Package gnutls.x86_64 0:2.8.5-10.el6_4.2 will be updated
---> Package gnutls.x86_64 0:2.8.5-14.el6_5 will be an update
---> Package gpxe-roms-qemu.noarch 0:0.9.7-6.12.el6 will be installed
---> Package netpbm.x86_64 0:10.47.05-11.el6 will be installed
---> Package qemu-img.x86_64 2:0.12.1.2-2.445.el6 will be installed
---> Package seabios.x86_64 0:0.6.1.2-28.el6 will be installed
---> Package sgabios-bin.noarch 0:0-0.3.20110621svn.el6 will be installed
---> Package spice-server.x86_64 0:0.12.4-11.el6 will be installed
--> Processing Dependency: libcelt051.so.0()(64bit) for package: spice-server-0.12.4-11.el6.x86_64
---> Package usbredir.x86_64 0:0.5.1-1.el6 will be installed
---> Package vgabios.noarch 0:0.6b-3.7.el6 will be installed
--> Running transaction check
---> Package celt051.x86_64 0:0.5.1.3-0.el6 will be installed
---> Package glusterfs.x86_64 0:3.6.0.28-2.el6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

============================================================================================================================================================
 Package                               Arch            Version                             Repository                                                  Size
============================================================================================================================================================
Installing:
 perl-Archive-Extract                  x86_64          1:0.38-136.el6                      /perl-Archive-Extract-0.38-136.el6.x86_64                   52 k
 perl-Archive-Tar                      x86_64          1.58-136.el6                        /perl-Archive-Tar-1.58-136.el6.x86_64                      113 k
 perl-Authen-SASL                      noarch          2.13-3.el6                          /perl-Authen-SASL-2.13-3.el6.noarch                        100 k
 perl-BSD-Resource                     x86_64          1.29.03-3.el6                       /perl-BSD-Resource-1.29.03-3.el6.x86_64                     63 k
 perl-Bit-Vector                       x86_64          7.1-2.el6                           /perl-Bit-Vector-7.1-2.el6.x86_64                          503 k
 perl-CGI-Session                      noarch          4.35-6.el6                          /perl-CGI-Session-4.35-6.el6.noarch                        235 k
 perl-CPAN                             x86_64          1.9402-136.el6                      /perl-CPAN-1.9402-136.el6.x86_64                           663 k
 perl-CPANPLUS                         x86_64          0.88-136.el6                        /perl-CPANPLUS-0.88-136.el6.x86_64                         767 k
 perl-Cache-Memcached                  noarch          1.28-2.el6                          /perl-Cache-Memcached-1.28-2.el6.noarch                     54 k
 perl-Carp-Clan                        noarch          6.03-2.el6                          /perl-Carp-Clan-6.03-2.el6.noarch                           46 k
 perl-Class-MethodMaker                x86_64          2.16-6.el6                          /perl-Class-MethodMaker-2.16-6.el6.x86_64                   20 M
 perl-Class-Singleton                  noarch          1.4-6.el6                           /perl-Class-Singleton-1.4-6.el6.noarch                      28 k
 perl-Compress-Raw-Bzip2               x86_64          2.021-136.el6                       /perl-Compress-Raw-Bzip2-2.021-136.el6.x86_64               58 k
 perl-Config-General                   noarch          2.52-1.el6                          /perl-Config-General-2.52-1.el6.noarch                     164 k
 perl-Convert-ASN1                     noarch          0.22-1.el6                          /perl-Convert-ASN1-0.22-1.el6.noarch                       119 k
 perl-Crypt-OpenSSL-Bignum             x86_64          0.04-8.1.el6                        /perl-Crypt-OpenSSL-Bignum-0.04-8.1.el6.x86_64              77 k
 perl-Crypt-OpenSSL-RSA                x86_64          0.25-10.1.el6                       /perl-Crypt-OpenSSL-RSA-0.25-10.1.el6.x86_64                86 k
 perl-Crypt-OpenSSL-Random             x86_64          0.04-9.1.el6                        /perl-Crypt-OpenSSL-Random-0.04-9.1.el6.x86_64              38 k
 perl-Crypt-PasswdMD5                  noarch          1.3-6.el6                           /perl-Crypt-PasswdMD5-1.3-6.el6.noarch                      12 k
 perl-Crypt-SSLeay                     x86_64          0.57-17.el6                         /perl-Crypt-SSLeay-0.57-17.el6.x86_64                      121 k
 perl-DBIx-Simple                      noarch          1.32-3.el6                          /perl-DBIx-Simple-1.32-3.el6.noarch                         62 k
 perl-Date-Calc                        noarch          6.3-2.el6                           /perl-Date-Calc-6.3-2.el6.noarch                           689 k
 perl-Date-Manip                       noarch          6.24-1.el6                          /perl-Date-Manip-6.24-1.el6.noarch                          10 M
 perl-DateTime                         x86_64          1:0.5300-2.el6                      /perl-DateTime-0.5300-2.el6.x86_64                         9.5 M
 perl-Devel-Symdump                    noarch          1:2.08-2.el6                        /perl-Devel-Symdump-2.08-2.el6.noarch                       27 k
 perl-Digest-HMAC                      noarch          1.01-22.el6                         /perl-Digest-HMAC-1.01-22.el6.noarch                        33 k
 perl-Digest-SHA                       x86_64          1:5.47-136.el6                      /perl-Digest-SHA-5.47-136.el6.x86_64                        85 k
 perl-Digest-SHA1                      x86_64          2.12-2.el6                          /perl-Digest-SHA1-2.12-2.el6.x86_64                        103 k
 perl-Encode-Detect                    x86_64          1.01-2.el6                          /perl-Encode-Detect-1.01-2.el6.x86_64                      185 k
 perl-ExtUtils-CBuilder                x86_64          1:0.27-136.el6                      /perl-ExtUtils-CBuilder-0.27-136.el6.x86_64                 59 k
 perl-ExtUtils-Embed                   x86_64          1.28-136.el6                        /perl-ExtUtils-Embed-1.28-136.el6.x86_64                    17 k
 perl-File-Fetch                       x86_64          0.26-136.el6                        /perl-File-Fetch-0.26-136.el6.x86_64                        46 k
 perl-File-Slurp                       noarch          9999.13-7.el6                       /perl-File-Slurp-9999.13-7.el6.noarch                       65 k
 perl-FreezeThaw                       noarch          0.45-5.el6                          /perl-FreezeThaw-0.45-5.el6.noarch                          31 k
 perl-Frontier-RPC                     noarch          0.07b4p1-9.el6                      /perl-Frontier-RPC-0.07b4p1-9.el6.noarch                    62 k
 perl-Frontier-RPC-doc                 noarch          0.07b4p1-9.el6                      /perl-Frontier-RPC-doc-0.07b4p1-9.el6.noarch                37 k
 perl-GSSAPI                           x86_64          0.26-6.el6                          /perl-GSSAPI-0.26-6.el6.x86_64                             157 k
 perl-IO-Compress-Bzip2                x86_64          2.021-136.el6                       /perl-IO-Compress-Bzip2-2.021-136.el6.x86_64                59 k
 perl-IO-Socket-INET6                  noarch          2.56-4.el6                          /perl-IO-Socket-INET6-2.56-4.el6.noarch                     28 k
 perl-IO-String                        noarch          1.08-9.el6                          /perl-IO-String-1.08-9.el6.noarch                           18 k
 perl-IO-Zlib                          x86_64          1:1.09-136.el6                      /perl-IO-Zlib-1.09-136.el6.x86_64                           19 k
 perl-IPC-Cmd                          x86_64          1:0.56-136.el6                      /perl-IPC-Cmd-0.56-136.el6.x86_64                           57 k
 perl-LDAP                             noarch          1:0.40-1.el6                        /perl-LDAP-0.40-1.el6.noarch                               843 k
 perl-List-MoreUtils                   x86_64          0.22-10.el6                         /perl-List-MoreUtils-0.22-10.el6.x86_64                    136 k
 perl-Locale-Maketext-Simple           x86_64          1:0.18-136.el6                      /perl-Locale-Maketext-Simple-0.18-136.el6.x86_64            13 k
 perl-Log-Message                      x86_64          1:0.02-136.el6                      /perl-Log-Message-0.02-136.el6.x86_64                       42 k
 perl-Log-Message-Simple               x86_64          0.04-136.el6                        /perl-Log-Message-Simple-0.04-136.el6.x86_64                11 k
 perl-Mail-DKIM                        noarch          0.37-2.el6                          /perl-Mail-DKIM-0.37-2.el6.noarch                          277 k
 perl-MailTools                        noarch          2.04-4.el6                          /perl-MailTools-2.04-4.el6.noarch                          190 k
 perl-Module-Build                     x86_64          1:0.3500-136.el6                    /perl-Module-Build-0.3500-136.el6.x86_64                   460 k
 perl-Module-CoreList                  x86_64          2.18-136.el6                        /perl-Module-CoreList-2.18-136.el6.x86_64                  521 k
 perl-Module-Load                      x86_64          1:0.16-136.el6                      /perl-Module-Load-0.16-136.el6.x86_64                      7.3 k
 perl-Module-Load-Conditional          x86_64          0.30-136.el6                        /perl-Module-Load-Conditional-0.30-136.el6.x86_64           23 k
 perl-Module-Loaded                    x86_64          1:0.02-136.el6                      /perl-Module-Loaded-0.02-136.el6.x86_64                    5.9 k
 perl-Mozilla-LDAP                     x86_64          1.5.3-4.el6                         /perl-Mozilla-LDAP-1.5.3-4.el6.x86_64                      491 k
 perl-Net-DNS                          x86_64          0.65-5.el6                          /perl-Net-DNS-0.65-5.el6.x86_64                            477 k
 perl-Net-Telnet                       noarch          3.03-11.el6                         /perl-Net-Telnet-3.03-11.el6.noarch                        154 k
 perl-NetAddr-IP                       x86_64          4.027-7.el6                         /perl-NetAddr-IP-4.027-7.el6.x86_64                        250 k
 perl-Newt                             x86_64          1.08-26.el6                         /perl-Newt-1.08-26.el6.x86_64                              194 k
 perl-Object-Accessor                  x86_64          1:0.34-136.el6                      /perl-Object-Accessor-0.34-136.el6.x86_64                   29 k
 perl-PCP-LogImport                    x86_64          3.9.4-5.el6                         /perl-PCP-LogImport-3.9.4-5.el6.x86_64                      41 k
 perl-PCP-LogSummary                   x86_64          3.9.4-5.el6                         /perl-PCP-LogSummary-3.9.4-5.el6.x86_64                    9.1 k
 perl-PCP-MMV                          x86_64          3.9.4-5.el6                         /perl-PCP-MMV-3.9.4-5.el6.x86_64                            44 k
 perl-PCP-PMDA                         x86_64          3.9.4-5.el6                         /perl-PCP-PMDA-3.9.4-5.el6.x86_64                          129 k
 perl-Package-Constants                x86_64          1:0.02-136.el6                      /perl-Package-Constants-0.02-136.el6.x86_64                4.6 k
 perl-Params-Check                     x86_64          1:0.26-136.el6                      /perl-Params-Check-0.26-136.el6.x86_64                      25 k
 perl-Params-Validate                  x86_64          0.92-3.el6                          /perl-Params-Validate-0.92-3.el6.x86_64                    175 k
 perl-Parse-CPAN-Meta                  x86_64          1:1.40-136.el6                      /perl-Parse-CPAN-Meta-1.40-136.el6.x86_64                   13 k
 perl-Parse-Yapp                       noarch          1.05-41.el6                         /perl-Parse-Yapp-1.05-41.el6.noarch                        112 k
 perl-Perlilog                         noarch          0.3-4.el6                           /perl-Perlilog-0.3-4.el6.noarch                            189 k
 perl-Pod-Coverage                     noarch          0.20-4.el6                          /perl-Pod-Coverage-0.20-4.el6.noarch                        33 k
 perl-SGMLSpm                          noarch          1.03ii-21.el6                       /perl-SGMLSpm-1.03ii-21.el6.noarch                          71 k
 perl-SNMP_Session                     noarch          1.12-4.el6                          /perl-SNMP_Session-1.12-4.el6.noarch                       247 k
 perl-Socket6                          x86_64          0.23-4.el6                          /perl-Socket6-0.23-4.el6.x86_64                             54 k
 perl-String-CRC32                     x86_64          1.4-9.el6                           /perl-String-CRC32-1.4-9.el6.x86_64                         14 k
 perl-Sys-Guestfs                      x86_64          1:1.20.11-11.el6                    /perl-Sys-Guestfs-1.20.11-11.el6.x86_64                    1.2 M
 perl-Sys-Virt                         x86_64          0.10.2-5.el6                        /perl-Sys-Virt-0.10.2-5.el6.x86_64                         790 k
 perl-Term-ProgressBar                 noarch          2.09-10.el6                         /perl-Term-ProgressBar-2.09-10.el6.noarch                   38 k
 perl-Term-UI                          x86_64          0.20-136.el6                        /perl-Term-UI-0.20-136.el6.x86_64                           31 k
 perl-Test-Pod                         noarch          1.40-1.el6                          /perl-Test-Pod-1.40-1.el6.noarch                            15 k
 perl-Test-Pod-Coverage                noarch          1.08-8.1.el6                        /perl-Test-Pod-Coverage-1.08-8.1.el6.noarch                 14 k
 perl-Text-Iconv                       x86_64          1.7-6.el6                           /perl-Text-Iconv-1.7-6.el6.x86_64                           33 k
 perl-Time-Piece                       x86_64          1.15-136.el6                        /perl-Time-Piece-1.15-136.el6.x86_64                        50 k
 perl-TimeDate                         noarch          1:1.16-13.el6                       /perl-TimeDate-1.16-13.el6.noarch                           74 k
 perl-XML-DOM                          noarch          1.44-7.el6                          /perl-XML-DOM-1.44-7.el6.noarch                            275 k
 perl-XML-DOM-XPath                    noarch          0.14-4.el6                          /perl-XML-DOM-XPath-0.14-4.el6.noarch                       15 k
 perl-XML-Dumper                       noarch          0.81-8.el6                          /perl-XML-Dumper-0.81-8.el6.noarch                          41 k
 perl-XML-Filter-BufferText            noarch          1.01-8.el6                          /perl-XML-Filter-BufferText-1.01-8.el6.noarch              6.0 k
 perl-XML-Grove                        noarch          0.46alpha-40.el6                    /perl-XML-Grove-0.46alpha-40.el6.noarch                    120 k
 perl-XML-LibXML                       x86_64          1:1.70-5.el6                        /perl-XML-LibXML-1.70-5.el6.x86_64                         951 k
 perl-XML-NamespaceSupport             noarch          1.10-3.el6                          /perl-XML-NamespaceSupport-1.10-3.el6.noarch                28 k
 perl-XML-RegExp                       noarch          0.03-7.el6                          /perl-XML-RegExp-0.03-7.el6.noarch                         9.1 k
 perl-XML-SAX                          noarch          0.96-7.el6                          /perl-XML-SAX-0.96-7.el6.noarch                            256 k
 perl-XML-SAX-Writer                   noarch          0.50-8.el6                          /perl-XML-SAX-Writer-0.50-8.el6.noarch                      51 k
 perl-XML-Twig                         noarch          3.34-1.el6                          /perl-XML-Twig-3.34-1.el6.noarch                           605 k
 perl-XML-Writer                       noarch          0.606-6.el6                         /perl-XML-Writer-0.606-6.el6.noarch                         53 k
 perl-XML-XPath                        noarch          1.13-10.el6                         /perl-XML-XPath-1.13-10.el6.noarch                         188 k
 perl-XML-XPathEngine                  noarch          0.12-3.el6                          /perl-XML-XPathEngine-0.12-3.el6.noarch                    109 k
 perl-YAML-Syck                        x86_64          1.07-4.el6                          /perl-YAML-Syck-1.07-4.el6.x86_64                          175 k
 perl-YAML-Tiny                        noarch          1.40-2.el6                          /perl-YAML-Tiny-1.40-2.el6.noarch                           78 k
 perl-core                             x86_64          5.10.1-136.el6                      /perl-core-5.10.1-136.el6.x86_64                           0.0
 perl-devel                            i686            4:5.10.1-136.el6                    /perl-devel-5.10.1-136.el6.i686                            1.8 M
 perl-hivex                            x86_64          1.3.3-4.2.el6                       /perl-hivex-1.3.3-4.2.el6.x86_64                            83 k
 perl-libintl                          x86_64          1.20-1.el6                          /perl-libintl-1.20-1.el6.x86_64                            4.3 M
 perl-libs                             i686            4:5.10.1-136.el6                    /perl-libs-5.10.1-136.el6.i686                             1.4 M
 perl-libxml-perl                      noarch          0.08-10.el6                         /perl-libxml-perl-0.08-10.el6.noarch                       179 k
 perl-parent                           x86_64          1:0.221-136.el6                     /perl-parent-0.221-136.el6.x86_64                          5.7 k
 perl-suidperl                         x86_64          4:5.10.1-136.el6                    /perl-suidperl-5.10.1-136.el6.x86_64                        75 k
 perltidy                              noarch          20090616-2.1.el6                    /perltidy-20090616-2.1.el6.noarch                          1.3 M
Installing for dependencies:
 augeas-libs                           x86_64          1.0.0-7.el6                         Server                                                     313 k
 btrfs-progs                           x86_64          0.20-0.2.git91d9eec.el6             Server                                                     297 k
 celt051                               x86_64          0.5.1.3-0.el6                       Server                                                      50 k
 febootstrap-supermin-helper           x86_64          3.21-4.el6                          Server                                                     337 k
 gdisk                                 x86_64          0.8.10-1.el6                        Server                                                     167 k
 glibc                                 i686            2.12-1.149.el6                      Server                                                     4.3 M
 glusterfs                             x86_64          3.6.0.28-2.el6                      Server                                                     1.3 M
 glusterfs-api                         x86_64          3.6.0.28-2.el6                      Server                                                      56 k
 glusterfs-libs                        x86_64          3.6.0.28-2.el6                      Server                                                     263 k
 gnutls-utils                          x86_64          2.8.5-14.el6_5                      Server                                                     100 k
 gpxe-roms-qemu                        noarch          0.9.7-6.12.el6                      Server                                                     220 k
 hivex                                 x86_64          1.3.3-4.2.el6                       Server                                                      89 k
 libguestfs                            x86_64          1:1.20.11-11.el6                    Server                                                     2.0 M
 libvirt-client                        x86_64          0.10.2-46.el6                       Server                                                     4.0 M
 lzop                                  x86_64          1.02-0.9.rc1.el6                    Server                                                      50 k
 mod_perl                              x86_64          2.0.4-11.el6_5                      Server                                                     3.2 M
 nc                                    x86_64          1.84-22.el6                         Server                                                      57 k
 netpbm                                x86_64          10.47.05-11.el6                     Server                                                     802 k
 netpbm-progs                          x86_64          10.47.05-11.el6                     Server                                                     1.7 M
 nss-softokn-freebl                    i686            3.14.3-17.el6                       Server                                                     153 k
 openjade                              x86_64          1.3.2-36.el6                        Server                                                     817 k
 opensp                                x86_64          1.5.2-12.1.el6                      Server                                                     872 k
 pcp-conf                              x86_64          3.9.4-5.el6                         Server                                                      22 k
 pcp-libs                              x86_64          3.9.4-5.el6                         Server                                                     252 k
 qemu-img                              x86_64          2:0.12.1.2-2.445.el6                Server                                                     793 k
 qemu-kvm                              x86_64          2:0.12.1.2-2.445.el6                Server                                                     1.6 M
 scrub                                 x86_64          2.2-2.el6                           Server                                                      35 k
 seabios                               x86_64          0.6.1.2-28.el6                      Server                                                      92 k
 sgabios-bin                           noarch          0-0.3.20110621svn.el6               Server                                                     6.6 k
 spice-server                          x86_64          0.12.4-11.el6                       Server                                                     345 k
 usbredir                              x86_64          0.5.1-1.el6                         Server                                                      40 k
 vgabios                               noarch          0.6b-3.7.el6                        Server                                                      42 k
 yajl                                  x86_64          1.0.7-3.el6                         Server                                                      27 k
Updating for dependencies:
 glibc                                 x86_64          2.12-1.149.el6                      Server                                                     3.8 M
 glibc-common                          x86_64          2.12-1.149.el6                      Server                                                      14 M
 glibc-devel                           x86_64          2.12-1.149.el6                      Server                                                     983 k
 glibc-headers                         x86_64          2.12-1.149.el6                      Server                                                     611 k
 gnutls                                x86_64          2.8.5-14.el6_5                      Server                                                     346 k
 nss-softokn-freebl                    x86_64          3.14.3-17.el6                       Server                                                     163 k

Transaction Summary
============================================================================================================================================================
Install     142 Package(s)
Upgrade       6 Package(s)

Total size: 109 M
Total download size: 44 M
Is this ok [y/N]: y
Downloading Packages:
------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                        53 MB/s |  44 MB     00:00
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
  Installing : 1:perl-Locale-Maketext-Simple-0.18-136.el6.x86_64                                                                                      1/154
  Installing : 1:perl-Params-Check-0.26-136.el6.x86_64                                                                                                2/154
  Installing : perl-Perlilog-0.3-4.el6.noarch                                                                                                         3/154
  Installing : 1:perl-Module-Load-0.16-136.el6.x86_64                                                                                                 4/154
  Installing : perl-Module-Load-Conditional-0.30-136.el6.x86_64                                                                                       5/154
  Installing : 1:perl-IPC-Cmd-0.56-136.el6.x86_64                                                                                                     6/154
  Installing : 1:perl-Log-Message-0.02-136.el6.x86_64                                                                                                 7/154
  Installing : perl-XML-NamespaceSupport-1.10-3.el6.noarch                                                                                            8/154
  Installing : 1:perl-Package-Constants-0.02-136.el6.x86_64                                                                                           9/154
  Installing : perl-Log-Message-Simple-0.04-136.el6.x86_64                                                                                           10/154
  Installing : perl-Term-UI-0.20-136.el6.x86_64                                                                                                      11/154
  Installing : perl-File-Fetch-0.26-136.el6.x86_64                                                                                                   12/154
  Installing : 1:perl-Archive-Extract-0.38-136.el6.x86_64                                                                                            13/154
  Installing : 1:perl-Object-Accessor-0.34-136.el6.x86_64                                                                                            14/154
  Installing : 1:perl-Module-Loaded-0.02-136.el6.x86_64                                                                                              15/154
  Installing : perl-Carp-Clan-6.03-2.el6.noarch                                                                                                      16/154
  Installing : 1:perl-Parse-CPAN-Meta-1.40-136.el6.x86_64                                                                                            17/154
  Installing : perl-Module-CoreList-2.18-136.el6.x86_64                                                                                              18/154
  Installing : 1:perl-IO-Zlib-1.09-136.el6.x86_64                                                                                                    19/154
  Installing : perl-Archive-Tar-1.58-136.el6.x86_64                                                                                                  20/154
  Installing : perl-libxml-perl-0.08-10.el6.noarch                                                                                                   21/154
  Updating   : glibc-common-2.12-1.149.el6.x86_64                                                                                                    22/154
  Updating   : glibc-2.12-1.149.el6.x86_64                                                                                                           23/154
  Updating   : nss-softokn-freebl-3.14.3-17.el6.x86_64                                                                                               24/154
  Installing : glusterfs-libs-3.6.0.28-2.el6.x86_64                                                                                                  25/154
  Installing : 1:perl-Digest-SHA-5.47-136.el6.x86_64                                                                                                 26/154
  Updating   : gnutls-2.8.5-14.el6_5.x86_64                                                                                                          27/154
  Installing : perl-hivex-1.3.3-4.2.el6.x86_64                                                                                                       28/154
  Installing : hivex-1.3.3-4.2.el6.x86_64                                                                                                            29/154
  Installing : usbredir-0.5.1-1.el6.x86_64                                                                                                           30/154
  Installing : yajl-1.0.7-3.el6.x86_64                                                                                                               31/154
  Installing : perl-Compress-Raw-Bzip2-2.021-136.el6.x86_64                                                                                          32/154
  Installing : perl-IO-Compress-Bzip2-2.021-136.el6.x86_64                                                                                           33/154
  Installing : gnutls-utils-2.8.5-14.el6_5.x86_64                                                                                                    34/154
  Installing : perl-CPAN-1.9402-136.el6.x86_64                                                                                                       35/154
  Installing : glusterfs-api-3.6.0.28-2.el6.x86_64                                                                                                   36/154
  Installing : glusterfs-3.6.0.28-2.el6.x86_64                                                                                                       37/154
  Installing : 2:qemu-img-0.12.1.2-2.445.el6.x86_64                                                                                                  38/154
  Installing : gdisk-0.8.10-1.el6.x86_64                                                                                                             39/154
  Installing : celt051-0.5.1.3-0.el6.x86_64                                                                                                          40/154
  Installing : spice-server-0.12.4-11.el6.x86_64                                                                                                     41/154
  Installing : perl-BSD-Resource-1.29.03-3.el6.x86_64                                                                                                42/154
  Installing : mod_perl-2.0.4-11.el6_5.x86_64                                                                                                        43/154
  Installing : nc-1.84-22.el6.x86_64                                                                                                                 44/154
  Installing : libvirt-client-0.10.2-46.el6.x86_64                                                                                                   45/154
  Installing : perl-Crypt-OpenSSL-Bignum-0.04-8.1.el6.x86_64                                                                                         46/154
  Installing : perl-Crypt-OpenSSL-Random-0.04-9.1.el6.x86_64                                                                                         47/154
  Installing : perl-Crypt-OpenSSL-RSA-0.25-10.1.el6.x86_64                                                                                           48/154
  Installing : perl-GSSAPI-0.26-6.el6.x86_64                                                                                                         49/154
  Installing : netpbm-10.47.05-11.el6.x86_64                                                                                                         50/154
  Installing : netpbm-progs-10.47.05-11.el6.x86_64                                                                                                   51/154
  Installing : perl-Digest-SHA1-2.12-2.el6.x86_64                                                                                                    52/154
  Installing : perl-Digest-HMAC-1.01-22.el6.noarch                                                                                                   53/154
  Installing : perl-Authen-SASL-2.13-3.el6.noarch                                                                                                    54/154
  Installing : perl-Net-DNS-0.65-5.el6.x86_64                                                                                                        55/154
  Installing : augeas-libs-1.0.0-7.el6.x86_64                                                                                                        56/154
  Installing : btrfs-progs-0.20-0.2.git91d9eec.el6.x86_64                                                                                            57/154
  Installing : perl-Time-Piece-1.15-136.el6.x86_64                                                                                                   58/154
  Installing : perl-Class-MethodMaker-2.16-6.el6.x86_64                                                                                              59/154
  Installing : perl-Text-Iconv-1.7-6.el6.x86_64                                                                                                      60/154
  Installing : perl-Params-Validate-0.92-3.el6.x86_64                                                                                                61/154
  Installing : perl-String-CRC32-1.4-9.el6.x86_64                                                                                                    62/154
  Installing : perl-YAML-Syck-1.07-4.el6.x86_64                                                                                                      63/154
  Installing : febootstrap-supermin-helper-3.21-4.el6.x86_64                                                                                         64/154
  Installing : perl-libintl-1.20-1.el6.x86_64                                                                                                        65/154
  Installing : opensp-1.5.2-12.1.el6.x86_64                                                                                                          66/154
  Installing : openjade-1.3.2-36.el6.x86_64                                                                                                          67/154
  Installing : 1:perl-XML-LibXML-1.70-5.el6.x86_64                                                                                                   68/154
  Installing : perl-XML-SAX-0.96-7.el6.noarch                                                                                                        69/154
  Installing : perl-XML-Filter-BufferText-1.01-8.el6.noarch                                                                                          70/154
  Installing : perl-XML-SAX-Writer-0.50-8.el6.noarch                                                                                                 71/154
  Installing : perl-Socket6-0.23-4.el6.x86_64                                                                                                        72/154
  Updating   : glibc-headers-2.12-1.149.el6.x86_64                                                                                                   73/154
  Updating   : glibc-devel-2.12-1.149.el6.x86_64                                                                                                     74/154
  Installing : 4:perl-devel-5.10.1-136.el6.i686                                                                                                      75/154
  Installing : 1:perl-ExtUtils-CBuilder-0.27-136.el6.x86_64                                                                                          76/154
  Installing : 1:perl-Module-Build-0.3500-136.el6.x86_64                                                                                             77/154
  Installing : perl-ExtUtils-Embed-1.28-136.el6.x86_64                                                                                               78/154
  Installing : perl-Bit-Vector-7.1-2.el6.x86_64                                                                                                      79/154
  Installing : scrub-2.2-2.el6.x86_64                                                                                                                80/154
  Installing : perl-List-MoreUtils-0.22-10.el6.x86_64                                                                                                81/154
  Installing : lzop-1.02-0.9.rc1.el6.x86_64                                                                                                          82/154
  Installing : perl-FreezeThaw-0.45-5.el6.noarch                                                                                                     83/154
  Installing : seabios-0.6.1.2-28.el6.x86_64                                                                                                         84/154
  Installing : gpxe-roms-qemu-0.9.7-6.12.el6.noarch                                                                                                  85/154
  Installing : perl-DBIx-Simple-1.32-3.el6.noarch                                                                                                    86/154
  Installing : perl-CPANPLUS-0.88-136.el6.x86_64                                                                                                     87/154
  Installing : perl-XML-XPath-1.13-10.el6.noarch                                                                                                     88/154
  Installing : perl-XML-RegExp-0.03-7.el6.noarch                                                                                                     89/154
  Installing : perl-XML-DOM-1.44-7.el6.noarch                                                                                                        90/154
  Installing : 1:perl-TimeDate-1.16-13.el6.noarch                                                                                                    91/154
  Installing : perl-MailTools-2.04-4.el6.noarch                                                                                                      92/154
  Installing : pcp-conf-3.9.4-5.el6.x86_64                                                                                                           93/154
  Installing : pcp-libs-3.9.4-5.el6.x86_64                                                                                                           94/154
  Installing : perl-Class-Singleton-1.4-6.el6.noarch                                                                                                 95/154
  Installing : 1:perl-parent-0.221-136.el6.x86_64                                                                                                    96/154
  Installing : 1:perl-Devel-Symdump-2.08-2.el6.noarch                                                                                                97/154
  Installing : perl-Pod-Coverage-0.20-4.el6.noarch                                                                                                   98/154
  Installing : perl-XML-XPathEngine-0.12-3.el6.noarch                                                                                                99/154
  Installing : perl-Convert-ASN1-0.22-1.el6.noarch                                                                                                  100/154
  Installing : sgabios-bin-0-0.3.20110621svn.el6.noarch                                                                                             101/154
  Installing : perl-Frontier-RPC-doc-0.07b4p1-9.el6.noarch                                                                                          102/154
  Installing : vgabios-0.6b-3.7.el6.noarch                                                                                                          103/154
  Installing : 2:qemu-kvm-0.12.1.2-2.445.el6.x86_64                                                                                                 104/154
  Installing : 1:libguestfs-1.20.11-11.el6.x86_64                                                                                                   105/154
  Installing : 1:perl-Sys-Guestfs-1.20.11-11.el6.x86_64                                                                                             106/154
  Installing : perl-Frontier-RPC-0.07b4p1-9.el6.noarch                                                                                              107/154
  Installing : 1:perl-LDAP-0.40-1.el6.noarch                                                                                                        108/154
  Installing : perl-XML-DOM-XPath-0.14-4.el6.noarch                                                                                                 109/154
  Installing : perl-Test-Pod-Coverage-1.08-8.1.el6.noarch                                                                                           110/154
  Installing : 1:perl-DateTime-0.5300-2.el6.x86_64                                                                                                  111/154
  Installing : perl-PCP-MMV-3.9.4-5.el6.x86_64                                                                                                      112/154
  Installing : perl-PCP-PMDA-3.9.4-5.el6.x86_64                                                                                                     113/154
  Installing : perl-PCP-LogSummary-3.9.4-5.el6.x86_64                                                                                               114/154
  Installing : perl-PCP-LogImport-3.9.4-5.el6.x86_64                                                                                                115/154
  Installing : perl-Mail-DKIM-0.37-2.el6.noarch                                                                                                     116/154
  Installing : perl-CGI-Session-4.35-6.el6.noarch                                                                                                   117/154
  Installing : perl-Date-Calc-6.3-2.el6.noarch                                                                                                      118/154
  Installing : perl-IO-Socket-INET6-2.56-4.el6.noarch                                                                                               119/154
  Installing : perl-SGMLSpm-1.03ii-21.el6.noarch                                                                                                    120/154
  Installing : perl-Date-Manip-6.24-1.el6.noarch                                                                                                    121/154
  Installing : perl-Cache-Memcached-1.28-2.el6.noarch                                                                                               122/154
  Installing : perl-Term-ProgressBar-2.09-10.el6.noarch                                                                                             123/154
  Installing : perl-Sys-Virt-0.10.2-5.el6.x86_64                                                                                                    124/154
  Installing : perl-NetAddr-IP-4.027-7.el6.x86_64                                                                                                   125/154
  Installing : perl-Crypt-SSLeay-0.57-17.el6.x86_64                                                                                                 126/154
  Installing : 4:perl-suidperl-5.10.1-136.el6.x86_64                                                                                                127/154
  Installing : perl-Newt-1.08-26.el6.x86_64                                                                                                         128/154
  Installing : perl-Mozilla-LDAP-1.5.3-4.el6.x86_64                                                                                                 129/154
  Installing : perl-Encode-Detect-1.01-2.el6.x86_64                                                                                                 130/154
  Installing : perl-XML-Grove-0.46alpha-40.el6.noarch                                                                                               131/154
  Installing : perl-XML-Twig-3.34-1.el6.noarch                                                                                                      132/154
  Installing : perl-Crypt-PasswdMD5-1.3-6.el6.noarch                                                                                                133/154
  Installing : perl-Parse-Yapp-1.05-41.el6.noarch                                                                                                   134/154
  Installing : perl-File-Slurp-9999.13-7.el6.noarch                                                                                                 135/154
  Installing : perl-YAML-Tiny-1.40-2.el6.noarch                                                                                                     136/154
  Installing : perl-XML-Writer-0.606-6.el6.noarch                                                                                                   137/154
  Installing : perl-IO-String-1.08-9.el6.noarch                                                                                                     138/154
  Installing : perl-Net-Telnet-3.03-11.el6.noarch                                                                                                   139/154
  Installing : perltidy-20090616-2.1.el6.noarch                                                                                                     140/154
  Installing : perl-Config-General-2.52-1.el6.noarch                                                                                                141/154
  Installing : perl-XML-Dumper-0.81-8.el6.noarch                                                                                                    142/154
  Installing : perl-Test-Pod-1.40-1.el6.noarch                                                                                                      143/154
  Installing : perl-SNMP_Session-1.12-4.el6.noarch                                                                                                  144/154
  Installing : nss-softokn-freebl-3.14.3-17.el6.i686                                                                                                145/154
  Installing : glibc-2.12-1.149.el6.i686                                                                                                            146/154
  Installing : 4:perl-libs-5.10.1-136.el6.i686                                                                                                      147/154
  Installing : perl-core-5.10.1-136.el6.x86_64                                                                                                      148/154
  Cleanup    : glibc-devel-2.12-1.132.el6.x86_64                                                                                                    149/154
  Cleanup    : glibc-headers-2.12-1.132.el6.x86_64                                                                                                  150/154
  Cleanup    : gnutls-2.8.5-10.el6_4.2.x86_64                                                                                                       151/154
  Cleanup    : glibc-common-2.12-1.132.el6.x86_64                                                                                                   152/154
  Cleanup    : nss-softokn-freebl-3.14.3-9.el6.x86_64                                                                                               153/154
  Cleanup    : glibc-2.12-1.132.el6.x86_64                                                                                                          154/154
  Verifying  : 1:perl-Log-Message-0.02-136.el6.x86_64                                                                                                 1/154
  Verifying  : glusterfs-3.6.0.28-2.el6.x86_64                                                                                                        2/154
  Verifying  : perl-libxml-perl-0.08-10.el6.noarch                                                                                                    3/154
  Verifying  : vgabios-0.6b-3.7.el6.noarch                                                                                                            4/154
  Verifying  : perl-Frontier-RPC-doc-0.07b4p1-9.el6.noarch                                                                                            5/154
  Verifying  : perl-CGI-Session-4.35-6.el6.noarch                                                                                                     6/154
  Verifying  : gdisk-0.8.10-1.el6.x86_64                                                                                                              7/154
  Verifying  : 4:perl-libs-5.10.1-136.el6.i686                                                                                                        8/154
  Verifying  : sgabios-bin-0-0.3.20110621svn.el6.noarch                                                                                               9/154
  Verifying  : celt051-0.5.1.3-0.el6.x86_64                                                                                                          10/154
  Verifying  : perl-Convert-ASN1-0.22-1.el6.noarch                                                                                                   11/154
  Verifying  : perl-XML-XPathEngine-0.12-3.el6.noarch                                                                                                12/154
  Verifying  : hivex-1.3.3-4.2.el6.x86_64                                                                                                            13/154
  Verifying  : perl-SNMP_Session-1.12-4.el6.noarch                                                                                                   14/154
  Verifying  : mod_perl-2.0.4-11.el6_5.x86_64                                                                                                        15/154
  Verifying  : 2:qemu-img-0.12.1.2-2.445.el6.x86_64                                                                                                  16/154
  Verifying  : perl-XML-SAX-0.96-7.el6.noarch                                                                                                        17/154
  Verifying  : 1:perl-Digest-SHA-5.47-136.el6.x86_64                                                                                                 18/154
  Verifying  : glibc-devel-2.12-1.149.el6.x86_64                                                                                                     19/154
  Verifying  : perl-Pod-Coverage-0.20-4.el6.noarch                                                                                                   20/154
  Verifying  : 1:perl-IO-Zlib-1.09-136.el6.x86_64                                                                                                    21/154
  Verifying  : perl-BSD-Resource-1.29.03-3.el6.x86_64                                                                                                22/154
  Verifying  : 1:perl-IPC-Cmd-0.56-136.el6.x86_64                                                                                                    23/154
  Verifying  : 1:perl-Devel-Symdump-2.08-2.el6.noarch                                                                                                24/154
  Verifying  : perl-Module-CoreList-2.18-136.el6.x86_64                                                                                              25/154
  Verifying  : 1:perl-Module-Load-0.16-136.el6.x86_64                                                                                                26/154
  Verifying  : gnutls-2.8.5-14.el6_5.x86_64                                                                                                          27/154
  Verifying  : perl-PCP-MMV-3.9.4-5.el6.x86_64                                                                                                       28/154
  Verifying  : perl-Test-Pod-1.40-1.el6.noarch                                                                                                       29/154
  Verifying  : nc-1.84-22.el6.x86_64                                                                                                                 30/154
  Verifying  : perl-File-Fetch-0.26-136.el6.x86_64                                                                                                   31/154
  Verifying  : perl-NetAddr-IP-4.027-7.el6.x86_64                                                                                                    32/154
  Verifying  : perl-XML-DOM-XPath-0.14-4.el6.noarch                                                                                                  33/154
  Verifying  : perl-Log-Message-Simple-0.04-136.el6.x86_64                                                                                           34/154
  Verifying  : perl-Sys-Virt-0.10.2-5.el6.x86_64                                                                                                     35/154
  Verifying  : perl-XML-SAX-Writer-0.50-8.el6.noarch                                                                                                 36/154
  Verifying  : perl-Crypt-OpenSSL-Bignum-0.04-8.1.el6.x86_64                                                                                         37/154
  Verifying  : 1:perl-LDAP-0.40-1.el6.noarch                                                                                                         38/154
  Verifying  : perl-Crypt-OpenSSL-Random-0.04-9.1.el6.x86_64                                                                                         39/154
  Verifying  : perl-GSSAPI-0.26-6.el6.x86_64                                                                                                         40/154
  Verifying  : perl-CPAN-1.9402-136.el6.x86_64                                                                                                       41/154
  Verifying  : netpbm-10.47.05-11.el6.x86_64                                                                                                         42/154
  Verifying  : 1:libguestfs-1.20.11-11.el6.x86_64                                                                                                    43/154
  Verifying  : perl-Authen-SASL-2.13-3.el6.noarch                                                                                                    44/154
  Verifying  : 1:perl-Package-Constants-0.02-136.el6.x86_64                                                                                          45/154
  Verifying  : 1:perl-parent-0.221-136.el6.x86_64                                                                                                    46/154
  Verifying  : perl-Digest-SHA1-2.12-2.el6.x86_64                                                                                                    47/154
  Verifying  : augeas-libs-1.0.0-7.el6.x86_64                                                                                                        48/154
  Verifying  : 1:perl-DateTime-0.5300-2.el6.x86_64                                                                                                   49/154
  Verifying  : usbredir-0.5.1-1.el6.x86_64                                                                                                           50/154
  Verifying  : spice-server-0.12.4-11.el6.x86_64                                                                                                     51/154
  Verifying  : 1:perl-Parse-CPAN-Meta-1.40-136.el6.x86_64                                                                                            52/154
  Verifying  : perl-Crypt-SSLeay-0.57-17.el6.x86_64                                                                                                  53/154
  Verifying  : btrfs-progs-0.20-0.2.git91d9eec.el6.x86_64                                                                                            54/154
  Verifying  : perl-Time-Piece-1.15-136.el6.x86_64                                                                                                   55/154
  Verifying  : perl-Class-MethodMaker-2.16-6.el6.x86_64                                                                                              56/154
  Verifying  : 1:perl-Sys-Guestfs-1.20.11-11.el6.x86_64                                                                                              57/154
  Verifying  : perl-XML-Dumper-0.81-8.el6.noarch                                                                                                     58/154
  Verifying  : nss-softokn-freebl-3.14.3-17.el6.x86_64                                                                                               59/154
  Verifying  : perl-Module-Load-Conditional-0.30-136.el6.x86_64                                                                                      60/154
  Verifying  : perl-Config-General-2.52-1.el6.noarch                                                                                                 61/154
  Verifying  : perl-Class-Singleton-1.4-6.el6.noarch                                                                                                 62/154
  Verifying  : 1:perl-Module-Build-0.3500-136.el6.x86_64                                                                                             63/154
  Verifying  : perl-Cache-Memcached-1.28-2.el6.noarch                                                                                                64/154
  Verifying  : perl-Digest-HMAC-1.01-22.el6.noarch                                                                                                   65/154
  Verifying  : pcp-conf-3.9.4-5.el6.x86_64                                                                                                           66/154
  Verifying  : perltidy-20090616-2.1.el6.noarch                                                                                                      67/154
  Verifying  : perl-Text-Iconv-1.7-6.el6.x86_64                                                                                                      68/154
  Verifying  : 1:perl-TimeDate-1.16-13.el6.noarch                                                                                                    69/154
  Verifying  : perl-Params-Validate-0.92-3.el6.x86_64                                                                                                70/154
  Verifying  : 4:perl-suidperl-5.10.1-136.el6.x86_64                                                                                                 71/154
  Verifying  : perl-String-CRC32-1.4-9.el6.x86_64                                                                                                    72/154
  Verifying  : perl-YAML-Syck-1.07-4.el6.x86_64                                                                                                      73/154
  Verifying  : perl-XML-RegExp-0.03-7.el6.noarch                                                                                                     74/154
  Verifying  : perl-XML-XPath-1.13-10.el6.noarch                                                                                                     75/154
  Verifying  : 1:perl-Locale-Maketext-Simple-0.18-136.el6.x86_64                                                                                     76/154
  Verifying  : perl-Net-DNS-0.65-5.el6.x86_64                                                                                                        77/154
  Verifying  : perl-PCP-PMDA-3.9.4-5.el6.x86_64                                                                                                      78/154
  Verifying  : perl-Term-UI-0.20-136.el6.x86_64                                                                                                      79/154
  Verifying  : perl-DBIx-Simple-1.32-3.el6.noarch                                                                                                    80/154
  Verifying  : febootstrap-supermin-helper-3.21-4.el6.x86_64                                                                                         81/154
  Verifying  : glibc-2.12-1.149.el6.x86_64                                                                                                           82/154
  Verifying  : perl-XML-NamespaceSupport-1.10-3.el6.noarch                                                                                           83/154
  Verifying  : perl-ExtUtils-Embed-1.28-136.el6.x86_64                                                                                               84/154
  Verifying  : glusterfs-api-3.6.0.28-2.el6.x86_64                                                                                                   85/154
  Verifying  : perl-libintl-1.20-1.el6.x86_64                                                                                                        86/154
  Verifying  : nss-softokn-freebl-3.14.3-17.el6.i686                                                                                                 87/154
  Verifying  : gpxe-roms-qemu-0.9.7-6.12.el6.noarch                                                                                                  88/154
  Verifying  : opensp-1.5.2-12.1.el6.x86_64                                                                                                          89/154
  Verifying  : perl-Carp-Clan-6.03-2.el6.noarch                                                                                                      90/154
  Verifying  : 1:perl-XML-LibXML-1.70-5.el6.x86_64                                                                                                   91/154
  Verifying  : perl-PCP-LogSummary-3.9.4-5.el6.x86_64                                                                                                92/154
  Verifying  : seabios-0.6.1.2-28.el6.x86_64                                                                                                         93/154
  Verifying  : perl-Newt-1.08-26.el6.x86_64                                                                                                          94/154
  Verifying  : 1:perl-Archive-Extract-0.38-136.el6.x86_64                                                                                            95/154
  Verifying  : perl-Net-Telnet-3.03-11.el6.noarch                                                                                                    96/154
  Verifying  : 1:perl-ExtUtils-CBuilder-0.27-136.el6.x86_64                                                                                          97/154
  Verifying  : pcp-libs-3.9.4-5.el6.x86_64                                                                                                           98/154
  Verifying  : perl-Mozilla-LDAP-1.5.3-4.el6.x86_64                                                                                                  99/154
  Verifying  : perl-IO-String-1.08-9.el6.noarch                                                                                                     100/154
  Verifying  : perl-Socket6-0.23-4.el6.x86_64                                                                                                       101/154
  Verifying  : glibc-headers-2.12-1.149.el6.x86_64                                                                                                  102/154
  Verifying  : 1:perl-Module-Loaded-0.02-136.el6.x86_64                                                                                             103/154
  Verifying  : yajl-1.0.7-3.el6.x86_64                                                                                                              104/154
  Verifying  : glibc-common-2.12-1.149.el6.x86_64                                                                                                   105/154
  Verifying  : perl-Crypt-OpenSSL-RSA-0.25-10.1.el6.x86_64                                                                                          106/154
  Verifying  : perl-XML-Twig-3.34-1.el6.noarch                                                                                                      107/154
  Verifying  : perl-XML-Grove-0.46alpha-40.el6.noarch                                                                                               108/154
  Verifying  : perl-SGMLSpm-1.03ii-21.el6.noarch                                                                                                    109/154
  Verifying  : perl-XML-Writer-0.606-6.el6.noarch                                                                                                   110/154
  Verifying  : netpbm-progs-10.47.05-11.el6.x86_64                                                                                                  111/154
  Verifying  : 4:perl-devel-5.10.1-136.el6.i686                                                                                                     112/154
  Verifying  : libvirt-client-0.10.2-46.el6.x86_64                                                                                                  113/154
  Verifying  : perl-Date-Manip-6.24-1.el6.noarch                                                                                                    114/154
  Verifying  : perl-Bit-Vector-7.1-2.el6.x86_64                                                                                                     115/154
  Verifying  : perl-YAML-Tiny-1.40-2.el6.noarch                                                                                                     116/154
  Verifying  : 2:qemu-kvm-0.12.1.2-2.445.el6.x86_64                                                                                                 117/154
  Verifying  : perl-hivex-1.3.3-4.2.el6.x86_64                                                                                                      118/154
  Verifying  : perl-Perlilog-0.3-4.el6.noarch                                                                                                       119/154
  Verifying  : scrub-2.2-2.el6.x86_64                                                                                                               120/154
  Verifying  : perl-List-MoreUtils-0.22-10.el6.x86_64                                                                                               121/154
  Verifying  : perl-Term-ProgressBar-2.09-10.el6.noarch                                                                                             122/154
  Verifying  : perl-FreezeThaw-0.45-5.el6.noarch                                                                                                    123/154
  Verifying  : perl-Mail-DKIM-0.37-2.el6.noarch                                                                                                     124/154
  Verifying  : perl-XML-Filter-BufferText-1.01-8.el6.noarch                                                                                         125/154
  Verifying  : openjade-1.3.2-36.el6.x86_64                                                                                                         126/154
  Verifying  : lzop-1.02-0.9.rc1.el6.x86_64                                                                                                         127/154
  Verifying  : perl-Test-Pod-Coverage-1.08-8.1.el6.noarch                                                                                           128/154
  Verifying  : gnutls-utils-2.8.5-14.el6_5.x86_64                                                                                                   129/154
  Verifying  : perl-File-Slurp-9999.13-7.el6.noarch                                                                                                 130/154
  Verifying  : perl-Date-Calc-6.3-2.el6.noarch                                                                                                      131/154
  Verifying  : perl-XML-DOM-1.44-7.el6.noarch                                                                                                       132/154
  Verifying  : glusterfs-libs-3.6.0.28-2.el6.x86_64                                                                                                 133/154
  Verifying  : perl-IO-Socket-INET6-2.56-4.el6.noarch                                                                                               134/154
  Verifying  : perl-Frontier-RPC-0.07b4p1-9.el6.noarch                                                                                              135/154
  Verifying  : perl-PCP-LogImport-3.9.4-5.el6.x86_64                                                                                                136/154
  Verifying  : perl-Compress-Raw-Bzip2-2.021-136.el6.x86_64                                                                                         137/154
  Verifying  : 1:perl-Params-Check-0.26-136.el6.x86_64                                                                                              138/154
  Verifying  : perl-IO-Compress-Bzip2-2.021-136.el6.x86_64                                                                                          139/154
  Verifying  : perl-MailTools-2.04-4.el6.noarch                                                                                                     140/154
  Verifying  : glibc-2.12-1.149.el6.i686                                                                                                            141/154
  Verifying  : 1:perl-Object-Accessor-0.34-136.el6.x86_64                                                                                           142/154
  Verifying  : perl-core-5.10.1-136.el6.x86_64                                                                                                      143/154
  Verifying  : perl-Archive-Tar-1.58-136.el6.x86_64                                                                                                 144/154
  Verifying  : perl-Parse-Yapp-1.05-41.el6.noarch                                                                                                   145/154
  Verifying  : perl-Encode-Detect-1.01-2.el6.x86_64                                                                                                 146/154
  Verifying  : perl-CPANPLUS-0.88-136.el6.x86_64                                                                                                    147/154
  Verifying  : perl-Crypt-PasswdMD5-1.3-6.el6.noarch                                                                                                148/154
  Verifying  : nss-softokn-freebl-3.14.3-9.el6.x86_64                                                                                               149/154
  Verifying  : glibc-2.12-1.132.el6.x86_64                                                                                                          150/154
  Verifying  : glibc-devel-2.12-1.132.el6.x86_64                                                                                                    151/154
  Verifying  : glibc-common-2.12-1.132.el6.x86_64                                                                                                   152/154
  Verifying  : gnutls-2.8.5-10.el6_4.2.x86_64                                                                                                       153/154
  Verifying  : glibc-headers-2.12-1.132.el6.x86_64                                                                                                  154/154

Installed:
  perl-Archive-Extract.x86_64 1:0.38-136.el6        perl-Archive-Tar.x86_64 0:1.58-136.el6               perl-Authen-SASL.noarch 0:2.13-3.el6
  perl-BSD-Resource.x86_64 0:1.29.03-3.el6          perl-Bit-Vector.x86_64 0:7.1-2.el6                   perl-CGI-Session.noarch 0:4.35-6.el6
  perl-CPAN.x86_64 0:1.9402-136.el6                 perl-CPANPLUS.x86_64 0:0.88-136.el6                  perl-Cache-Memcached.noarch 0:1.28-2.el6
  perl-Carp-Clan.noarch 0:6.03-2.el6                perl-Class-MethodMaker.x86_64 0:2.16-6.el6           perl-Class-Singleton.noarch 0:1.4-6.el6
  perl-Compress-Raw-Bzip2.x86_64 0:2.021-136.el6    perl-Config-General.noarch 0:2.52-1.el6              perl-Convert-ASN1.noarch 0:0.22-1.el6
  perl-Crypt-OpenSSL-Bignum.x86_64 0:0.04-8.1.el6   perl-Crypt-OpenSSL-RSA.x86_64 0:0.25-10.1.el6        perl-Crypt-OpenSSL-Random.x86_64 0:0.04-9.1.el6
  perl-Crypt-PasswdMD5.noarch 0:1.3-6.el6           perl-Crypt-SSLeay.x86_64 0:0.57-17.el6               perl-DBIx-Simple.noarch 0:1.32-3.el6
  perl-Date-Calc.noarch 0:6.3-2.el6                 perl-Date-Manip.noarch 0:6.24-1.el6                  perl-DateTime.x86_64 1:0.5300-2.el6
  perl-Devel-Symdump.noarch 1:2.08-2.el6            perl-Digest-HMAC.noarch 0:1.01-22.el6                perl-Digest-SHA.x86_64 1:5.47-136.el6
  perl-Digest-SHA1.x86_64 0:2.12-2.el6              perl-Encode-Detect.x86_64 0:1.01-2.el6               perl-ExtUtils-CBuilder.x86_64 1:0.27-136.el6
  perl-ExtUtils-Embed.x86_64 0:1.28-136.el6         perl-File-Fetch.x86_64 0:0.26-136.el6                perl-File-Slurp.noarch 0:9999.13-7.el6
  perl-FreezeThaw.noarch 0:0.45-5.el6               perl-Frontier-RPC.noarch 0:0.07b4p1-9.el6            perl-Frontier-RPC-doc.noarch 0:0.07b4p1-9.el6
  perl-GSSAPI.x86_64 0:0.26-6.el6                   perl-IO-Compress-Bzip2.x86_64 0:2.021-136.el6        perl-IO-Socket-INET6.noarch 0:2.56-4.el6
  perl-IO-String.noarch 0:1.08-9.el6                perl-IO-Zlib.x86_64 1:1.09-136.el6                   perl-IPC-Cmd.x86_64 1:0.56-136.el6
  perl-LDAP.noarch 1:0.40-1.el6                     perl-List-MoreUtils.x86_64 0:0.22-10.el6             perl-Locale-Maketext-Simple.x86_64 1:0.18-136.el6
  perl-Log-Message.x86_64 1:0.02-136.el6            perl-Log-Message-Simple.x86_64 0:0.04-136.el6        perl-Mail-DKIM.noarch 0:0.37-2.el6
  perl-MailTools.noarch 0:2.04-4.el6                perl-Module-Build.x86_64 1:0.3500-136.el6            perl-Module-CoreList.x86_64 0:2.18-136.el6
  perl-Module-Load.x86_64 1:0.16-136.el6            perl-Module-Load-Conditional.x86_64 0:0.30-136.el6   perl-Module-Loaded.x86_64 1:0.02-136.el6
  perl-Mozilla-LDAP.x86_64 0:1.5.3-4.el6            perl-Net-DNS.x86_64 0:0.65-5.el6                     perl-Net-Telnet.noarch 0:3.03-11.el6
  perl-NetAddr-IP.x86_64 0:4.027-7.el6              perl-Newt.x86_64 0:1.08-26.el6                       perl-Object-Accessor.x86_64 1:0.34-136.el6
  perl-PCP-LogImport.x86_64 0:3.9.4-5.el6           perl-PCP-LogSummary.x86_64 0:3.9.4-5.el6             perl-PCP-MMV.x86_64 0:3.9.4-5.el6
  perl-PCP-PMDA.x86_64 0:3.9.4-5.el6                perl-Package-Constants.x86_64 1:0.02-136.el6         perl-Params-Check.x86_64 1:0.26-136.el6
  perl-Params-Validate.x86_64 0:0.92-3.el6          perl-Parse-CPAN-Meta.x86_64 1:1.40-136.el6           perl-Parse-Yapp.noarch 0:1.05-41.el6
  perl-Perlilog.noarch 0:0.3-4.el6                  perl-Pod-Coverage.noarch 0:0.20-4.el6                perl-SGMLSpm.noarch 0:1.03ii-21.el6
  perl-SNMP_Session.noarch 0:1.12-4.el6             perl-Socket6.x86_64 0:0.23-4.el6                     perl-String-CRC32.x86_64 0:1.4-9.el6
  perl-Sys-Guestfs.x86_64 1:1.20.11-11.el6          perl-Sys-Virt.x86_64 0:0.10.2-5.el6                  perl-Term-ProgressBar.noarch 0:2.09-10.el6
  perl-Term-UI.x86_64 0:0.20-136.el6                perl-Test-Pod.noarch 0:1.40-1.el6                    perl-Test-Pod-Coverage.noarch 0:1.08-8.1.el6
  perl-Text-Iconv.x86_64 0:1.7-6.el6                perl-Time-Piece.x86_64 0:1.15-136.el6                perl-TimeDate.noarch 1:1.16-13.el6
  perl-XML-DOM.noarch 0:1.44-7.el6                  perl-XML-DOM-XPath.noarch 0:0.14-4.el6               perl-XML-Dumper.noarch 0:0.81-8.el6
  perl-XML-Filter-BufferText.noarch 0:1.01-8.el6    perl-XML-Grove.noarch 0:0.46alpha-40.el6             perl-XML-LibXML.x86_64 1:1.70-5.el6
  perl-XML-NamespaceSupport.noarch 0:1.10-3.el6     perl-XML-RegExp.noarch 0:0.03-7.el6                  perl-XML-SAX.noarch 0:0.96-7.el6
  perl-XML-SAX-Writer.noarch 0:0.50-8.el6           perl-XML-Twig.noarch 0:3.34-1.el6                    perl-XML-Writer.noarch 0:0.606-6.el6
  perl-XML-XPath.noarch 0:1.13-10.el6               perl-XML-XPathEngine.noarch 0:0.12-3.el6             perl-YAML-Syck.x86_64 0:1.07-4.el6
  perl-YAML-Tiny.noarch 0:1.40-2.el6                perl-core.x86_64 0:5.10.1-136.el6                    perl-devel.i686 4:5.10.1-136.el6
  perl-hivex.x86_64 0:1.3.3-4.2.el6                 perl-libintl.x86_64 0:1.20-1.el6                     perl-libs.i686 4:5.10.1-136.el6
  perl-libxml-perl.noarch 0:0.08-10.el6             perl-parent.x86_64 1:0.221-136.el6                   perl-suidperl.x86_64 4:5.10.1-136.el6
  perltidy.noarch 0:20090616-2.1.el6

Dependency Installed:
  augeas-libs.x86_64 0:1.0.0-7.el6                       btrfs-progs.x86_64 0:0.20-0.2.git91d9eec.el6        celt051.x86_64 0:0.5.1.3-0.el6
  febootstrap-supermin-helper.x86_64 0:3.21-4.el6        gdisk.x86_64 0:0.8.10-1.el6                         glibc.i686 0:2.12-1.149.el6
  glusterfs.x86_64 0:3.6.0.28-2.el6                      glusterfs-api.x86_64 0:3.6.0.28-2.el6               glusterfs-libs.x86_64 0:3.6.0.28-2.el6
  gnutls-utils.x86_64 0:2.8.5-14.el6_5                   gpxe-roms-qemu.noarch 0:0.9.7-6.12.el6              hivex.x86_64 0:1.3.3-4.2.el6
  libguestfs.x86_64 1:1.20.11-11.el6                     libvirt-client.x86_64 0:0.10.2-46.el6               lzop.x86_64 0:1.02-0.9.rc1.el6
  mod_perl.x86_64 0:2.0.4-11.el6_5                       nc.x86_64 0:1.84-22.el6                             netpbm.x86_64 0:10.47.05-11.el6
  netpbm-progs.x86_64 0:10.47.05-11.el6                  nss-softokn-freebl.i686 0:3.14.3-17.el6             openjade.x86_64 0:1.3.2-36.el6
  opensp.x86_64 0:1.5.2-12.1.el6                         pcp-conf.x86_64 0:3.9.4-5.el6                       pcp-libs.x86_64 0:3.9.4-5.el6
  qemu-img.x86_64 2:0.12.1.2-2.445.el6                   qemu-kvm.x86_64 2:0.12.1.2-2.445.el6                scrub.x86_64 0:2.2-2.el6
  seabios.x86_64 0:0.6.1.2-28.el6                        sgabios-bin.noarch 0:0-0.3.20110621svn.el6          spice-server.x86_64 0:0.12.4-11.el6
  usbredir.x86_64 0:0.5.1-1.el6                          vgabios.noarch 0:0.6b-3.7.el6                       yajl.x86_64 0:1.0.7-3.el6

Dependency Updated:
  glibc.x86_64 0:2.12-1.149.el6    glibc-common.x86_64 0:2.12-1.149.el6        glibc-devel.x86_64 0:2.12-1.149.el6   glibc-headers.x86_64 0:2.12-1.149.el6
  gnutls.x86_64 0:2.8.5-14.el6_5   nss-softokn-freebl.x86_64 0:3.14.3-17.el6

Complete!

```


### 安装MHA 依赖包 (三节点同时操作)

```sql


[root@node1 mha_install]# yum install perl-*.rpm  --skip-broken
Loaded plugins: fastestmirror, refresh-packagekit, security
Repository 'Server' is missing name in configuration, using id
Loading mirror speeds from cached hostfile
Setting up Install Process
Examining perl-Config-Tiny-2.12-1.el6.rfx.noarch.rpm: perl-Config-Tiny-2.12-1.el6.rfx.noarch
Marking perl-Config-Tiny-2.12-1.el6.rfx.noarch.rpm to be installed
Examining perl-Config-Tiny-2.14-7.el7.noarch.rpm: perl-Config-Tiny-2.14-7.el7.noarch
Marking perl-Config-Tiny-2.14-7.el7.noarch.rpm to be installed
Examining perl-Log-Dispatch-2.26-1.el6.rf.noarch.rpm: perl-Log-Dispatch-2.26-1.el6.rf.noarch
Marking perl-Log-Dispatch-2.26-1.el6.rf.noarch.rpm to be installed
Examining perl-Log-Dispatch-2.41-1.el7.1.noarch.rpm: perl-Log-Dispatch-2.41-1.el7.1.noarch
Marking perl-Log-Dispatch-2.41-1.el7.1.noarch.rpm to be installed
Examining perl-Parallel-ForkManager-0.7.5-2.2.el6.rf.noarch.rpm: perl-Parallel-ForkManager-0.7.5-2.2.el6.rf.noarch
Marking perl-Parallel-ForkManager-0.7.5-2.2.el6.rf.noarch.rpm to be installed
Examining perl-Parallel-ForkManager-1.18-2.el7.noarch.rpm: perl-Parallel-ForkManager-1.18-2.el7.noarch
Marking perl-Parallel-ForkManager-1.18-2.el7.noarch.rpm to be installed
Examining perl-Params-Validate-0.92-3.el6.x86_64.rpm: perl-Params-Validate-0.92-3.el6.x86_64
perl-Params-Validate-0.92-3.el6.x86_64.rpm: does not update installed package.
Examining perl-Params-Validate-1.08-4.el7.x86_64.rpm: perl-Params-Validate-1.08-4.el7.x86_64
Marking perl-Params-Validate-1.08-4.el7.x86_64.rpm as an update to perl-Params-Validate-0.92-3.el6.x86_64
Resolving Dependencies
--> Running transaction check
---> Package perl-Config-Tiny.noarch 0:2.12-1.el6.rfx will be installed
---> Package perl-Config-Tiny.noarch 0:2.14-7.el7 will be installed
--> Processing Dependency: perl(:MODULE_COMPAT_5.16.3) for package: perl-Config-Tiny-2.14-7.el7.noarch
---> Package perl-Log-Dispatch.noarch 0:2.26-1.el6.rf will be installed
---> Package perl-Log-Dispatch.noarch 0:2.41-1.el7.1 will be installed
--> Processing Dependency: perl(:MODULE_COMPAT_5.16.3) for package: perl-Log-Dispatch-2.41-1.el7.1.noarch
--> Processing Dependency: perl(Class::Load) for package: perl-Log-Dispatch-2.41-1.el7.1.noarch
--> Processing Dependency: perl(MIME::Lite) for package: perl-Log-Dispatch-2.41-1.el7.1.noarch
--> Processing Dependency: perl(Mail::Sender) for package: perl-Log-Dispatch-2.41-1.el7.1.noarch
--> Processing Dependency: perl(Mail::Sendmail) for package: perl-Log-Dispatch-2.41-1.el7.1.noarch
---> Package perl-Parallel-ForkManager.noarch 0:0.7.5-2.2.el6.rf will be installed
---> Package perl-Parallel-ForkManager.noarch 0:1.18-2.el7 will be installed
--> Processing Dependency: perl(:MODULE_COMPAT_5.16.3) for package: perl-Parallel-ForkManager-1.18-2.el7.noarch
---> Package perl-Params-Validate.x86_64 0:0.92-3.el6 will be updated
---> Package perl-Params-Validate.x86_64 0:1.08-4.el7 will be an update
--> Processing Dependency: perl(:MODULE_COMPAT_5.16.3) for package: perl-Params-Validate-1.08-4.el7.x86_64
--> Processing Dependency: perl(Module::Implementation) for package: perl-Params-Validate-1.08-4.el7.x86_64
--> Running transaction check
---> Package perl-Config-Tiny.noarch 0:2.14-7.el7 will be installed
--> Processing Dependency: perl(:MODULE_COMPAT_5.16.3) for package: perl-Config-Tiny-2.14-7.el7.noarch
---> Package perl-Log-Dispatch.noarch 0:2.41-1.el7.1 will be installed
--> Processing Dependency: perl(:MODULE_COMPAT_5.16.3) for package: perl-Log-Dispatch-2.41-1.el7.1.noarch
--> Processing Dependency: perl(Class::Load) for package: perl-Log-Dispatch-2.41-1.el7.1.noarch
--> Processing Dependency: perl(Mail::Sender) for package: perl-Log-Dispatch-2.41-1.el7.1.noarch
--> Processing Dependency: perl(Mail::Sendmail) for package: perl-Log-Dispatch-2.41-1.el7.1.noarch
---> Package perl-MIME-Lite.noarch 0:3.027-2.el6 will be installed
--> Processing Dependency: perl(MIME::Types) >= 1.28 for package: perl-MIME-Lite-3.027-2.el6.noarch
--> Processing Dependency: perl(Email::Date::Format) for package: perl-MIME-Lite-3.027-2.el6.noarch
---> Package perl-Parallel-ForkManager.noarch 0:1.18-2.el7 will be installed
--> Processing Dependency: perl(:MODULE_COMPAT_5.16.3) for package: perl-Parallel-ForkManager-1.18-2.el7.noarch
---> Package perl-Params-Validate.x86_64 0:1.08-4.el7 will be an update
--> Processing Dependency: perl(:MODULE_COMPAT_5.16.3) for package: perl-Params-Validate-1.08-4.el7.x86_64
--> Processing Dependency: perl(Module::Implementation) for package: perl-Params-Validate-1.08-4.el7.x86_64
--> Running transaction check
---> Package perl-Config-Tiny.noarch 0:2.14-7.el7 will be installed
--> Processing Dependency: perl(:MODULE_COMPAT_5.16.3) for package: perl-Config-Tiny-2.14-7.el7.noarch
---> Package perl-Email-Date-Format.noarch 0:1.002-5.el6 will be installed
---> Package perl-Log-Dispatch.noarch 0:2.41-1.el7.1 will be installed
--> Processing Dependency: perl(:MODULE_COMPAT_5.16.3) for package: perl-Log-Dispatch-2.41-1.el7.1.noarch
--> Processing Dependency: perl(Class::Load) for package: perl-Log-Dispatch-2.41-1.el7.1.noarch
--> Processing Dependency: perl(Mail::Sender) for package: perl-Log-Dispatch-2.41-1.el7.1.noarch
--> Processing Dependency: perl(Mail::Sendmail) for package: perl-Log-Dispatch-2.41-1.el7.1.noarch
---> Package perl-MIME-Types.noarch 0:1.28-2.el6 will be installed
---> Package perl-Parallel-ForkManager.noarch 0:1.18-2.el7 will be installed
--> Processing Dependency: perl(:MODULE_COMPAT_5.16.3) for package: perl-Parallel-ForkManager-1.18-2.el7.noarch
---> Package perl-Params-Validate.x86_64 0:1.08-4.el7 will be an update
--> Processing Dependency: perl(:MODULE_COMPAT_5.16.3) for package: perl-Params-Validate-1.08-4.el7.x86_64
--> Processing Dependency: perl(Module::Implementation) for package: perl-Params-Validate-1.08-4.el7.x86_64
--> Running transaction check
---> Package perl-Config-Tiny.noarch 0:2.12-1.el6.rfx will be installed
---> Package perl-Parallel-ForkManager.noarch 0:0.7.5-2.2.el6.rf will be installed
--> Running transaction check
---> Package perl-Config-Tiny.noarch 0:2.12-1.el6.rfx will be installed
---> Package perl-Parallel-ForkManager.noarch 0:0.7.5-2.2.el6.rf will be installed
--> Finished Dependency Resolution

Packages skipped because of dependency problems:
    perl-Config-Tiny-2.14-7.el7.noarch from /perl-Config-Tiny-2.14-7.el7.noarch
    perl-Email-Date-Format-1.002-5.el6.noarch from Server
    perl-Log-Dispatch-2.26-1.el6.rf.noarch from /perl-Log-Dispatch-2.26-1.el6.rf.noarch
    perl-Log-Dispatch-2.41-1.el7.1.noarch from /perl-Log-Dispatch-2.41-1.el7.1.noarch
    perl-MIME-Lite-3.027-2.el6.noarch from Server
    perl-MIME-Types-1.28-2.el6.noarch from Server
    perl-Parallel-ForkManager-1.18-2.el7.noarch from /perl-Parallel-ForkManager-1.18-2.el7.noarch
    perl-Params-Validate-1.08-4.el7.x86_64 from /perl-Params-Validate-1.08-4.el7.x86_64

Dependencies Resolved

============================================================================================================================================================
 Package                               Arch               Version                      Repository                                                      Size
============================================================================================================================================================
Installing:
 perl-Config-Tiny                      noarch             2.12-1.el6.rfx               /perl-Config-Tiny-2.12-1.el6.rfx.noarch                         39 k
 perl-Parallel-ForkManager             noarch             0.7.5-2.2.el6.rf             /perl-Parallel-ForkManager-0.7.5-2.2.el6.rf.noarch              19 k
Skipped (dependency problems):
 perl-Config-Tiny                      noarch             2.14-7.el7                   /perl-Config-Tiny-2.14-7.el7.noarch                             39 k
 perl-Email-Date-Format                noarch             1.002-5.el6                  Server                                                          16 k
 perl-Log-Dispatch                     noarch             2.26-1.el6.rf                /perl-Log-Dispatch-2.26-1.el6.rf.noarch                        126 k
 perl-Log-Dispatch                     noarch             2.41-1.el7.1                 /perl-Log-Dispatch-2.41-1.el7.1.noarch                         129 k
 perl-MIME-Lite                        noarch             3.027-2.el6                  Server                                                          82 k
 perl-MIME-Types                       noarch             1.28-2.el6                   Server                                                          32 k
 perl-Parallel-ForkManager             noarch             1.18-2.el7                   /perl-Parallel-ForkManager-1.18-2.el7.noarch                    41 k
 perl-Params-Validate                  x86_64             1.08-4.el7                   /perl-Params-Validate-1.08-4.el7.x86_64                        140 k

Transaction Summary
============================================================================================================================================================
Install       2 Package(s)

Total size: 58 k
Installed size: 58 k
Is this ok [y/N]: y
Downloading Packages:
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
  Installing : perl-Parallel-ForkManager-0.7.5-2.2.el6.rf.noarch                                                                                        1/2
  Installing : perl-Config-Tiny-2.12-1.el6.rfx.noarch                                                                                                   2/2
  Verifying  : perl-Config-Tiny-2.12-1.el6.rfx.noarch                                                                                                   1/2
  Verifying  : perl-Parallel-ForkManager-0.7.5-2.2.el6.rf.noarch                                                                                        2/2

Installed:
  perl-Config-Tiny.noarch 0:2.12-1.el6.rfx                                perl-Parallel-ForkManager.noarch 0:0.7.5-2.2.el6.rf

Skipped (dependency problems):
  perl-Config-Tiny.noarch 0:2.14-7.el7                  perl-Email-Date-Format.noarch 0:1.002-5.el6         perl-Log-Dispatch.noarch 0:2.26-1.el6.rf
  perl-Log-Dispatch.noarch 0:2.41-1.el7.1               perl-MIME-Lite.noarch 0:3.027-2.el6                 perl-MIME-Types.noarch 0:1.28-2.el6
  perl-Parallel-ForkManager.noarch 0:1.18-2.el7         perl-Params-Validate.x86_64 0:1.08-4.el7

Complete!


```


### 在node3上安装node(三节点同时进行)

```sql

[root@node1 ~]# mkdir -p /etc/mha
[root@node1 ~]# cd mha_install/
[root@node1 mha_install]# tar -zxf mha4mysql-node-0.57.tar.gz
[root@node1 mha_install]# mv mha4mysql-node-0.57 /etc/mha/mhanode
[root@node1 mha_install]# cd /etc/mha/mhanode/
[root@node1 mhanode]# perl M
Makefile.PL  MANIFEST     META.yml
[root@node1 mhanode]# perl Makefile.PL
*** Module::AutoInstall version 1.06
*** Checking for Perl dependencies...
[Core Features]
- DBI        ...loaded. (1.609)
- DBD::mysql ...loaded. (4.013)
*** Module::AutoInstall configuration finished.
Checking if your kit is complete...
Looks good
Writing Makefile for mha4mysql::node
[root@node1 mhanode]# make
cp lib/MHA/NodeUtil.pm blib/lib/MHA/NodeUtil.pm
cp lib/MHA/BinlogManager.pm blib/lib/MHA/BinlogManager.pm
cp lib/MHA/BinlogPosFinderElp.pm blib/lib/MHA/BinlogPosFinderElp.pm
cp lib/MHA/SlaveUtil.pm blib/lib/MHA/SlaveUtil.pm
cp lib/MHA/NodeConst.pm blib/lib/MHA/NodeConst.pm
cp lib/MHA/BinlogPosFindManager.pm blib/lib/MHA/BinlogPosFindManager.pm
cp lib/MHA/BinlogPosFinderXid.pm blib/lib/MHA/BinlogPosFinderXid.pm
cp lib/MHA/BinlogHeaderParser.pm blib/lib/MHA/BinlogHeaderParser.pm
cp lib/MHA/BinlogPosFinder.pm blib/lib/MHA/BinlogPosFinder.pm
cp bin/filter_mysqlbinlog blib/script/filter_mysqlbinlog
/usr/bin/perl "-Iinc" -MExtUtils::MY -e 'MY->fixin(shift)' -- blib/script/filter_mysqlbinlog
cp bin/apply_diff_relay_logs blib/script/apply_diff_relay_logs
/usr/bin/perl "-Iinc" -MExtUtils::MY -e 'MY->fixin(shift)' -- blib/script/apply_diff_relay_logs
cp bin/purge_relay_logs blib/script/purge_relay_logs
/usr/bin/perl "-Iinc" -MExtUtils::MY -e 'MY->fixin(shift)' -- blib/script/purge_relay_logs
cp bin/save_binary_logs blib/script/save_binary_logs
/usr/bin/perl "-Iinc" -MExtUtils::MY -e 'MY->fixin(shift)' -- blib/script/save_binary_logs
Manifying blib/man1/filter_mysqlbinlog.1
Manifying blib/man1/apply_diff_relay_logs.1
Manifying blib/man1/purge_relay_logs.1
Manifying blib/man1/save_binary_logs.1
[root@node1 mhanode]# make install
Installing /usr/local/share/perl5/MHA/NodeUtil.pm
Installing /usr/local/share/perl5/MHA/BinlogPosFinderXid.pm
Installing /usr/local/share/perl5/MHA/BinlogManager.pm
Installing /usr/local/share/perl5/MHA/BinlogPosFinder.pm
Installing /usr/local/share/perl5/MHA/BinlogPosFinderElp.pm
Installing /usr/local/share/perl5/MHA/NodeConst.pm
Installing /usr/local/share/perl5/MHA/SlaveUtil.pm
Installing /usr/local/share/perl5/MHA/BinlogPosFindManager.pm
Installing /usr/local/share/perl5/MHA/BinlogHeaderParser.pm
Installing /usr/local/share/man/man1/apply_diff_relay_logs.1
Installing /usr/local/share/man/man1/filter_mysqlbinlog.1
Installing /usr/local/share/man/man1/purge_relay_logs.1
Installing /usr/local/share/man/man1/save_binary_logs.1
Installing /usr/local/bin/purge_relay_logs
Installing /usr/local/bin/filter_mysqlbinlog
Installing /usr/local/bin/save_binary_logs
Installing /usr/local/bin/apply_diff_relay_logs
Appending installation info to /usr/lib64/perl5/perllocal.pod

```

### 在node3上装manager

```sql

[root@node3 mhamanager]# perl Makefile.PL
*** Module::AutoInstall version 1.06
*** Checking for Perl dependencies...
[Core Features]
- DBI                   ...loaded. (1.609)
- DBD::mysql            ...loaded. (4.013)
- Time::HiRes           ...loaded. (1.9721)
- Config::Tiny          ...loaded. (2.12)
- Log::Dispatch         ...loaded. (2.26)
- Parallel::ForkManager ...loaded. (0.7.5)
- MHA::NodeConst        ...loaded. (0.57)
*** Module::AutoInstall configuration finished.
Checking if your kit is complete...
Looks good
Writing Makefile for mha4mysql::manager
[root@node3 mhamanager]# make
cp lib/MHA/ManagerUtil.pm blib/lib/MHA/ManagerUtil.pm
cp lib/MHA/HealthCheck.pm blib/lib/MHA/HealthCheck.pm
cp lib/MHA/Config.pm blib/lib/MHA/Config.pm
cp lib/MHA/ManagerConst.pm blib/lib/MHA/ManagerConst.pm
cp lib/MHA/ServerManager.pm blib/lib/MHA/ServerManager.pm
cp lib/MHA/FileStatus.pm blib/lib/MHA/FileStatus.pm
cp lib/MHA/ManagerAdmin.pm blib/lib/MHA/ManagerAdmin.pm
cp lib/MHA/ManagerAdminWrapper.pm blib/lib/MHA/ManagerAdminWrapper.pm
cp lib/MHA/MasterFailover.pm blib/lib/MHA/MasterFailover.pm
cp lib/MHA/MasterRotate.pm blib/lib/MHA/MasterRotate.pm
cp lib/MHA/MasterMonitor.pm blib/lib/MHA/MasterMonitor.pm
cp lib/MHA/Server.pm blib/lib/MHA/Server.pm
cp lib/MHA/SSHCheck.pm blib/lib/MHA/SSHCheck.pm
cp lib/MHA/DBHelper.pm blib/lib/MHA/DBHelper.pm
cp bin/masterha_stop blib/script/masterha_stop
/usr/bin/perl "-Iinc" -MExtUtils::MY -e 'MY->fixin(shift)' -- blib/script/masterha_stop
cp bin/masterha_conf_host blib/script/masterha_conf_host
/usr/bin/perl "-Iinc" -MExtUtils::MY -e 'MY->fixin(shift)' -- blib/script/masterha_conf_host
cp bin/masterha_check_repl blib/script/masterha_check_repl
/usr/bin/perl "-Iinc" -MExtUtils::MY -e 'MY->fixin(shift)' -- blib/script/masterha_check_repl
cp bin/masterha_check_status blib/script/masterha_check_status
/usr/bin/perl "-Iinc" -MExtUtils::MY -e 'MY->fixin(shift)' -- blib/script/masterha_check_status
cp bin/masterha_master_monitor blib/script/masterha_master_monitor
/usr/bin/perl "-Iinc" -MExtUtils::MY -e 'MY->fixin(shift)' -- blib/script/masterha_master_monitor
cp bin/masterha_check_ssh blib/script/masterha_check_ssh
/usr/bin/perl "-Iinc" -MExtUtils::MY -e 'MY->fixin(shift)' -- blib/script/masterha_check_ssh
cp bin/masterha_master_switch blib/script/masterha_master_switch
/usr/bin/perl "-Iinc" -MExtUtils::MY -e 'MY->fixin(shift)' -- blib/script/masterha_master_switch
cp bin/masterha_secondary_check blib/script/masterha_secondary_check
/usr/bin/perl "-Iinc" -MExtUtils::MY -e 'MY->fixin(shift)' -- blib/script/masterha_secondary_check
cp bin/masterha_manager blib/script/masterha_manager
/usr/bin/perl "-Iinc" -MExtUtils::MY -e 'MY->fixin(shift)' -- blib/script/masterha_manager
Manifying blib/man1/masterha_stop.1
Manifying blib/man1/masterha_conf_host.1
Manifying blib/man1/masterha_check_repl.1
Manifying blib/man1/masterha_check_status.1
Manifying blib/man1/masterha_master_monitor.1
Manifying blib/man1/masterha_check_ssh.1
Manifying blib/man1/masterha_master_switch.1
Manifying blib/man1/masterha_secondary_check.1
Manifying blib/man1/masterha_manager.1
[root@node3 mhamanager]# make install
Installing /usr/local/share/man/man1/masterha_conf_host.1
Installing /usr/local/share/man/man1/masterha_manager.1
Installing /usr/local/share/man/man1/masterha_secondary_check.1
Installing /usr/local/share/man/man1/masterha_check_ssh.1
Installing /usr/local/share/man/man1/masterha_stop.1
Installing /usr/local/share/man/man1/masterha_check_status.1
Installing /usr/local/share/man/man1/masterha_master_monitor.1
Installing /usr/local/share/man/man1/masterha_master_switch.1
Installing /usr/local/share/man/man1/masterha_check_repl.1
Appending installation info to /usr/lib64/perl5/perllocal.pod

```

##  对MHA进行配置

### 创建配置文件

```sql

[root@node3 mhamanager]# mkdir -p /etc/mha/app1
[root@node3 ~]# cat /etc/mha/app1/app1.cnf

[server default]
# working directory on the manager
manager_workdir=/etc/mha/app1
manager_log=/etc/mha/app1/manager.log
remote_workdir=/etc/mha/app1
# In case of automatic file-over, the following script will be used
master_ip_failover_script=/etc/mha/master_ip_failover
master_ip_online_change_script=/etc/mha/master_ip_online_change

#mha连接到mysql的帐号
user=root
password=root

#mysql主从复制架构的复制用户
repl_user=dao
repl_password=dao


ssh_user=root

#心跳
ping_interval=1


secondary_check_script= masterha_secondary_check -s 192.168.2.62 -s 192.168.2.63

[server1]
hostname=192.168.2.61
port=3306
master_binlog_dir="/mysql1/data"
candidate_master=1
ssh_port=22
[server2]
hostname=192.168.2.62
port=3306
master_binlog_dir="/mysql1/data"
candidate_master=1
ssh_port=22
[server3]
hostname=192.168.2.63
port=3306
master_binlog_dir="/mysql1/data"
no_master=1
ssh_port=22



```




### 测试SSH

```sql

[root@node3 app1]# /etc/mha/mhamanager/bin/masterha_check_ssh --conf=/etc/mha/
app1/                    master_ip_failover       master_ip_online_change  mhamanager/              mhanode/
[root@node3 app1]# /etc/mha/mhamanager/bin/masterha_check_ssh --conf=/etc/mha/app1/app1.cnf
Thu Aug 23 15:14:27 2018 - [warning] Global configuration file /etc/masterha_default.cnf not found. Skipping.
Thu Aug 23 15:14:27 2018 - [info] Reading application default configuration from /etc/mha/app1/app1.cnf..
Thu Aug 23 15:14:27 2018 - [info] Reading server configuration from /etc/mha/app1/app1.cnf..
Thu Aug 23 15:14:27 2018 - [info] Starting SSH connection tests..
Thu Aug 23 15:14:29 2018 - [debug]
Thu Aug 23 15:14:27 2018 - [debug]  Connecting via SSH from root@192.168.2.61(192.168.2.61:22) to root@192.168.2.62(192.168.2.                                                                      62:22)..
Thu Aug 23 15:14:29 2018 - [debug]   ok.
Thu Aug 23 15:14:29 2018 - [debug]  Connecting via SSH from root@192.168.2.61(192.168.2.61:22) to root@192.168.2.63(192.168.2.                                                                      63:22)..
Thu Aug 23 15:14:29 2018 - [debug]   ok.
Thu Aug 23 15:14:29 2018 - [debug]
Thu Aug 23 15:14:28 2018 - [debug]  Connecting via SSH from root@192.168.2.63(192.168.2.63:22) to root@192.168.2.61(192.168.2.                                                                      61:22)..
Thu Aug 23 15:14:29 2018 - [debug]   ok.
Thu Aug 23 15:14:29 2018 - [debug]  Connecting via SSH from root@192.168.2.63(192.168.2.63:22) to root@192.168.2.62(192.168.2.                                                                      62:22)..
Thu Aug 23 15:14:29 2018 - [debug]   ok.
Thu Aug 23 15:14:29 2018 - [debug]
Thu Aug 23 15:14:28 2018 - [debug]  Connecting via SSH from root@192.168.2.62(192.168.2.62:22) to root@192.168.2.61(192.168.2.                                                                      61:22)..
Thu Aug 23 15:14:29 2018 - [debug]   ok.
Thu Aug 23 15:14:29 2018 - [debug]  Connecting via SSH from root@192.168.2.62(192.168.2.62:22) to root@192.168.2.63(192.168.2.                                                                      63:22)..
Thu Aug 23 15:14:29 2018 - [debug]   ok.
Thu Aug 23 15:14:29 2018 - [info] All SSH connection tests passed successfully.

```


### 检查复制状态

```sql

[root@node3 ~]# /etc/mha/mhamanager/bin/masterha_check_repl --conf=/etc/mha/app1/app1.cnf
Fri Aug 24 00:48:24 2018 - [warning] Global configuration file /etc/masterha_default.cnf not found. Skipping.
Fri Aug 24 00:48:24 2018 - [info] Reading application default configuration from /etc/mha/app1/app1.cnf..
Fri Aug 24 00:48:24 2018 - [info] Reading server configuration from /etc/mha/app1/app1.cnf..
Fri Aug 24 00:48:24 2018 - [info] MHA::MasterMonitor version 0.57.
Fri Aug 24 00:48:24 2018 - [info] GTID failover mode = 0
Fri Aug 24 00:48:24 2018 - [info] Dead Servers:
Fri Aug 24 00:48:24 2018 - [info] Alive Servers:
Fri Aug 24 00:48:24 2018 - [info]   192.168.2.61(192.168.2.61:3306)
Fri Aug 24 00:48:24 2018 - [info]   192.168.2.62(192.168.2.62:3306)
Fri Aug 24 00:48:24 2018 - [info]   192.168.2.63(192.168.2.63:3306)
Fri Aug 24 00:48:24 2018 - [info] Alive Slaves:
Fri Aug 24 00:48:24 2018 - [info]   192.168.2.62(192.168.2.62:3306)  Version=5.7.21-log (oldest major version between slaves) log-bin:enabled
Fri Aug 24 00:48:24 2018 - [info]     Replicating from 192.168.2.61(192.168.2.61:3306)
Fri Aug 24 00:48:24 2018 - [info]     Primary candidate for the new Master (candidate_master is set)
Fri Aug 24 00:48:24 2018 - [info]   192.168.2.63(192.168.2.63:3306)  Version=5.7.21-log (oldest major version between slaves) log-bin:enabled
Fri Aug 24 00:48:24 2018 - [info]     Replicating from 192.168.2.61(192.168.2.61:3306)
Fri Aug 24 00:48:24 2018 - [info]     Not candidate for the new Master (no_master is set)
Fri Aug 24 00:48:24 2018 - [info] Current Alive Master: 192.168.2.61(192.168.2.61:3306)
Fri Aug 24 00:48:24 2018 - [info] Checking slave configurations..
Fri Aug 24 00:48:24 2018 - [info]  read_only=1 is not set on slave 192.168.2.62(192.168.2.62:3306).
Fri Aug 24 00:48:24 2018 - [warning]  relay_log_purge=0 is not set on slave 192.168.2.62(192.168.2.62:3306).
Fri Aug 24 00:48:24 2018 - [info]  read_only=1 is not set on slave 192.168.2.63(192.168.2.63:3306).
Fri Aug 24 00:48:24 2018 - [warning]  relay_log_purge=0 is not set on slave 192.168.2.63(192.168.2.63:3306).
Fri Aug 24 00:48:24 2018 - [info] Checking replication filtering settings..
Fri Aug 24 00:48:24 2018 - [info]  binlog_do_db= , binlog_ignore_db=
Fri Aug 24 00:48:24 2018 - [info]  Replication filtering check ok.
Fri Aug 24 00:48:24 2018 - [info] GTID (with auto-pos) is not supported
Fri Aug 24 00:48:24 2018 - [info] Starting SSH connection tests..
Fri Aug 24 00:48:26 2018 - [info] All SSH connection tests passed successfully.
Fri Aug 24 00:48:26 2018 - [info] Checking MHA Node version..
Fri Aug 24 00:48:27 2018 - [info]  Version check ok.
Fri Aug 24 00:48:27 2018 - [info] Checking SSH publickey authentication settings on the current master..
Fri Aug 24 00:48:27 2018 - [info] HealthCheck: SSH to 192.168.2.61 is reachable.
Fri Aug 24 00:48:27 2018 - [info] Master MHA Node version is 0.57.
Fri Aug 24 00:48:27 2018 - [info] Checking recovery script configurations on 192.168.2.61(192.168.2.61:3306)..
Fri Aug 24 00:48:27 2018 - [info]   Executing command: save_binary_logs --command=test --start_pos=4 --binlog_dir=/mysql1/data --output_file=/etc/mha/app1/save_binary_logs_test --manager_version=0.57 --start_file=mysql-bin.000003
Fri Aug 24 00:48:27 2018 - [info]   Connecting to root@192.168.2.61(192.168.2.61:22)..
  Creating /etc/mha/app1 if not exists..    ok.
  Checking output directory is accessible or not..
   ok.
  Binlog found at /mysql1/data, up to mysql-bin.000003
Fri Aug 24 00:48:27 2018 - [info] Binlog setting check done.
Fri Aug 24 00:48:27 2018 - [info] Checking SSH publickey authentication and checking recovery script configurations on all alive slave servers..
Fri Aug 24 00:48:27 2018 - [info]   Executing command : apply_diff_relay_logs --command=test --slave_user='root' --slave_host=192.168.2.62 --slave_ip=192.168.2.62 --slave_port=3306 --workdir=/etc/mha/app1 --target_version=5.7.21-log --manager_version=0.57 --relay_log_info=/mysql1/data/relay-log.info  --relay_dir=/mysql1/data/  --slave_pass=xxx
Fri Aug 24 00:48:27 2018 - [info]   Connecting to root@192.168.2.62(192.168.2.62:22)..
  Checking slave recovery environment settings..
    Opening /mysql1/data/relay-log.info ... ok.
    Relay log found at /mysql1/data, up to node2-relay-bin.000008
    Temporary relay log file is /mysql1/data/node2-relay-bin.000008
    Testing mysql connection and privileges..mysql: [Warning] Using a password on the command line interface can be insecure.
 done.
    Testing mysqlbinlog output.. done.
    Cleaning up test file(s).. done.
Fri Aug 24 00:48:28 2018 - [info]   Executing command : apply_diff_relay_logs --command=test --slave_user='root' --slave_host=192.168.2.63 --slave_ip=192.168.2.63 --slave_port=3306 --workdir=/etc/mha/app1 --target_version=5.7.21-log --manager_version=0.57 --relay_log_info=/mysql1/data/relay-log.info  --relay_dir=/mysql1/data/  --slave_pass=xxx
Fri Aug 24 00:48:28 2018 - [info]   Connecting to root@192.168.2.63(192.168.2.63:22)..
  Checking slave recovery environment settings..
    Opening /mysql1/data/relay-log.info ... ok.
    Relay log found at /mysql1/data, up to node3-relay-bin.000008
    Temporary relay log file is /mysql1/data/node3-relay-bin.000008
    Testing mysql connection and privileges..mysql: [Warning] Using a password on the command line interface can be insecure.
 done.
    Testing mysqlbinlog output.. done.
    Cleaning up test file(s).. done.
Fri Aug 24 00:48:28 2018 - [info] Slaves settings check done.
Fri Aug 24 00:48:28 2018 - [info]
192.168.2.61(192.168.2.61:3306) (current master)
 +--192.168.2.62(192.168.2.62:3306)
 +--192.168.2.63(192.168.2.63:3306)

Fri Aug 24 00:48:28 2018 - [info] Checking replication health on 192.168.2.62..
Fri Aug 24 00:48:28 2018 - [info]  ok.
Fri Aug 24 00:48:28 2018 - [info] Checking replication health on 192.168.2.63..
Fri Aug 24 00:48:28 2018 - [info]  ok.
Fri Aug 24 00:48:28 2018 - [info] Checking master_ip_failover_script status:
Fri Aug 24 00:48:28 2018 - [info]   /etc/mha/master_ip_failover --command=status --ssh_user=root --orig_master_host=192.168.2.61 --orig_master_ip=192.168.2.61 --orig_master_port=3306


IN SCRIPT TEST====/sbin/ifconfig eth0:0 down==/sbin/ifconfig eth0:0 192.168.6.66/24===

Checking the Status of the script.. OK
Fri Aug 24 00:48:28 2018 - [info]  OK.
Fri Aug 24 00:48:28 2018 - [warning] shutdown_script is not defined.
Fri Aug 24 00:48:28 2018 - [info] Got exit code 0 (Not master dead).

MySQL Replication Health is OK.


```

### 主库添加虚拟IP

```sql



[root@node1 ~]# ifconfig eth0:0 192.168.6.66 up
[root@node1 ~]# ifconfig
eth0      Link encap:Ethernet  HWaddr 00:0C:29:1F:01:3A
          inet addr:192.168.2.61  Bcast:192.168.2.255  Mask:255.255.255.0
          inet6 addr: fe80::20c:29ff:fe1f:13a/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:7907 errors:0 dropped:0 overruns:0 frame:0
          TX packets:4497 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:823718 (804.4 KiB)  TX bytes:1051966 (1.0 MiB)

eth0:0    Link encap:Ethernet  HWaddr 00:0C:29:1F:01:3A
          inet addr:192.168.6.66  Bcast:192.168.6.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:16436  Metric:1
          RX packets:440 errors:0 dropped:0 overruns:0 frame:0
          TX packets:440 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:33376 (32.5 KiB)  TX bytes:33376 (32.5 KiB)


```

### 启动MHAmanager 

```sql


[root@node3 ~]# nohup /etc/mha/mhamanager/bin/masterha_manager --conf=/etc/mha/app1/app1.cnf --ignore_last_failover >/tmp/mha_manager.log < /dev/null 2>&1 &
[1] 30778
[root@node3 ~]# cat /tmp/mha_manager.log
Fri Aug 24 00:55:27 2018 - [warning] Global configuration file /etc/masterha_default.cnf not found. Skipping.
Fri Aug 24 00:55:27 2018 - [info] Reading application default configuration from /etc/mha/app1/app1.cnf..
Fri Aug 24 00:55:27 2018 - [info] Reading server configuration from /etc/mha/app1/app1.cnf..
[root@node3 ~]# /etc/mha/mhamanager/bin/masterha_check_status --conf=/etc/mha/app1/app1.cnf
app1 (pid:30778) is running(0:PING_OK), master:192.168.2.61

```

## 对MHA进行测试

### 停掉node1上的节点
```sql

[root@node1 ~]# pkill mysql

```

### 查看日志信息

```sql

[root@node3 ~]# echo "" > /etc/mha/app1/manager.log
[root@node3 ~]# tail -f /etc/mha/app1/manager.log

Fri Aug 24 02:29:46 2018 - [info] MHA::MasterMonitor version 0.57.
Fri Aug 24 02:29:46 2018 - [info] GTID failover mode = 0
Fri Aug 24 02:29:46 2018 - [info] Dead Servers:
Fri Aug 24 02:29:46 2018 - [info]   192.168.2.61(192.168.2.61:3306)
Fri Aug 24 02:29:46 2018 - [info] Alive Servers:
Fri Aug 24 02:29:46 2018 - [info]   192.168.2.62(192.168.2.62:3306)
Fri Aug 24 02:29:46 2018 - [info]   192.168.2.63(192.168.2.63:3306)
Fri Aug 24 02:29:46 2018 - [info] Alive Slaves:
Fri Aug 24 02:29:46 2018 - [info]   192.168.2.62(192.168.2.62:3306)  Version=5.7.21-log (oldest major version between slaves) log-bin:enabled
Fri Aug 24 02:29:46 2018 - [info]     Replicating from 192.168.2.61(192.168.2.61:3306)
Fri Aug 24 02:29:46 2018 - [info]     Primary candidate for the new Master (candidate_master is set)
Fri Aug 24 02:29:46 2018 - [info]   192.168.2.63(192.168.2.63:3306)  Version=5.7.21-log (oldest major version between slaves) log-bin:enabled
Fri Aug 24 02:29:46 2018 - [info]     Replicating from 192.168.2.61(192.168.2.61:3306)
Fri Aug 24 02:29:46 2018 - [info]     Not candidate for the new Master (no_master is set)
Fri Aug 24 02:29:46 2018 - [warning] MySQL master is not currently alive!
Fri Aug 24 02:29:46 2018 - [info] Checking slave configurations..
Fri Aug 24 02:29:46 2018 - [info]  read_only=1 is not set on slave 192.168.2.62(192.168.2.62:3306).
Fri Aug 24 02:29:46 2018 - [warning]  relay_log_purge=0 is not set on slave 192.168.2.62(192.168.2.62:3306).
Fri Aug 24 02:29:46 2018 - [info]  read_only=1 is not set on slave 192.168.2.63(192.168.2.63:3306).
Fri Aug 24 02:29:46 2018 - [warning]  relay_log_purge=0 is not set on slave 192.168.2.63(192.168.2.63:3306).
Fri Aug 24 02:29:46 2018 - [info] Checking replication filtering settings..
Fri Aug 24 02:29:46 2018 - [info]  Replication filtering check ok.
Fri Aug 24 02:29:46 2018 - [info] GTID (with auto-pos) is not supported
Fri Aug 24 02:29:46 2018 - [info] Starting SSH connection tests..
Fri Aug 24 02:29:47 2018 - [info] All SSH connection tests passed successfully.
Fri Aug 24 02:29:47 2018 - [info] Checking MHA Node version..
Fri Aug 24 02:29:48 2018 - [info]  Version check ok.
Fri Aug 24 02:29:48 2018 - [info] Getting current master (maybe dead) info ..
Fri Aug 24 02:29:48 2018 - [info] Identified master is 192.168.2.61(192.168.2.61:3306).
Fri Aug 24 02:29:48 2018 - [info] Checking SSH publickey authentication settings on the current master..
Fri Aug 24 02:29:48 2018 - [info] HealthCheck: SSH to 192.168.2.61 is reachable.
Fri Aug 24 02:29:48 2018 - [info] Master MHA Node version is 0.57.
Fri Aug 24 02:29:48 2018 - [info] Checking recovery script configurations on 192.168.2.61(192.168.2.61:3306)..
Fri Aug 24 02:29:48 2018 - [info]   Executing command: save_binary_logs --command=test --start_pos=4 --binlog_dir=/mysql1/data --output_file=/etc/mha/app1/save_binary_logs_test --manager_version=0.57 --start_file=mysql-bin.000004
Fri Aug 24 02:29:48 2018 - [info]   Connecting to root@192.168.2.61(192.168.2.61:22)..
  Creating /etc/mha/app1 if not exists..    ok.
  Checking output directory is accessible or not..
   ok.
  Binlog found at /mysql1/data, up to mysql-bin.000004
Fri Aug 24 02:29:48 2018 - [info] Binlog setting check done.
Fri Aug 24 02:29:48 2018 - [info] Checking SSH publickey authentication and checking recovery script configurations on all alive slave servers..
Fri Aug 24 02:29:48 2018 - [info]   Executing command : apply_diff_relay_logs --command=test --slave_user='root' --slave_host=192.168.2.62 --slave_ip=192.168.2.62 --slave_port=3306 --workdir=/etc/mha/app1 --target_version=5.7.21-log --manager_version=0.57 --relay_log_info=/mysql1/data/relay-log.info  --relay_dir=/mysql1/data/  --slave_pass=xxx
Fri Aug 24 02:29:48 2018 - [info]   Connecting to root@192.168.2.62(192.168.2.62:22)..
  Checking slave recovery environment settings..
    Opening /mysql1/data/relay-log.info ... ok.
    Relay log found at /mysql1/data, up to node2-relay-bin.000002
    Temporary relay log file is /mysql1/data/node2-relay-bin.000002
    Testing mysql connection and privileges..mysql: [Warning] Using a password on the command line interface can be insecure.
 done.
    Testing mysqlbinlog output.. done.
    Cleaning up test file(s).. done.
Fri Aug 24 02:29:49 2018 - [info]   Executing command : apply_diff_relay_logs --command=test --slave_user='root' --slave_host=192.168.2.63 --slave_ip=192.168.2.63 --slave_port=3306 --workdir=/etc/mha/app1 --target_version=5.7.21-log --manager_version=0.57 --relay_log_info=/mysql1/data/relay-log.info  --relay_dir=/mysql1/data/  --slave_pass=xxx
Fri Aug 24 02:29:49 2018 - [info]   Connecting to root@192.168.2.63(192.168.2.63:22)..
  Checking slave recovery environment settings..
    Opening /mysql1/data/relay-log.info ... ok.
    Relay log found at /mysql1/data, up to node3-relay-bin.000002
    Temporary relay log file is /mysql1/data/node3-relay-bin.000002
    Testing mysql connection and privileges..mysql: [Warning] Using a password on the command line interface can be insecure.
 done.
    Testing mysqlbinlog output.. done.
    Cleaning up test file(s).. done.
Fri Aug 24 02:29:49 2018 - [info] Slaves settings check done.
Fri Aug 24 02:29:49 2018 - [info]
192.168.2.61(192.168.2.61:3306) (current master)
 +--192.168.2.62(192.168.2.62:3306)
 +--192.168.2.63(192.168.2.63:3306)

Fri Aug 24 02:29:49 2018 - [info] Checking master_ip_failover_script status:
Fri Aug 24 02:29:49 2018 - [info]   /etc/mha/master_ip_failover --command=status --ssh_user=root --orig_master_host=192.168.2.61 --orig_master_ip=192.168.2.61 --orig_master_port=3306


IN SCRIPT TEST====/sbin/ifconfig eth0:0 down==/sbin/ifconfig eth0:0 192.168.6.66/24===

Checking the Status of the script.. OK
Fri Aug 24 02:29:49 2018 - [info]  OK.
Fri Aug 24 02:29:49 2018 - [warning] shutdown_script is not defined.
Fri Aug 24 02:29:49 2018 - [error][/usr/local/share/perl5/MHA/Server.pm, ln462] Checking slave status failed on 192.168.2.62(192.168.2.62:3306). err=Got error when executing SHOW SLAVE STATUS. MySQL server has gone away
Fri Aug 24 02:29:49 2018 - [info] Set master ping interval 1 seconds.
Fri Aug 24 02:29:49 2018 - [info] Set secondary check script: masterha_secondary_check -s 192.168.2.62 -s 192.168.2.63
Fri Aug 24 02:29:49 2018 - [info] Starting ping health check on 192.168.2.61(192.168.2.61:3306)..
Fri Aug 24 02:29:49 2018 - [warning] Got error on MySQL connect: 2013 (Lost connection to MySQL server at 'reading initial communication packet', system error: 111)
Fri Aug 24 02:29:49 2018 - [warning] Connection failed 1 time(s)..
Fri Aug 24 02:29:49 2018 - [info] Executing SSH check script: save_binary_logs --command=test --start_pos=4 --binlog_dir=/mysql1/data --output_file=/etc/mha/app1/save_binary_logs_test --manager_version=0.57 --binlog_prefix=mysql-bin
Fri Aug 24 02:29:49 2018 - [info] Executing secondary network check script: masterha_secondary_check -s 192.168.2.62 -s 192.168.2.63  --user=root  --master_host=192.168.2.61  --master_ip=192.168.2.61  --master_port=3306 --master_user=root --master_password=root --ping_type=SELECT
Fri Aug 24 02:29:49 2018 - [info] HealthCheck: SSH to 192.168.2.61 is reachable.
Monitoring server 192.168.2.62 is reachable, Master is not reachable from 192.168.2.62. OK.
Monitoring server 192.168.2.63 is reachable, Master is not reachable from 192.168.2.63. OK.
Fri Aug 24 02:29:50 2018 - [info] Master is not reachable from all other monitoring servers. Failover should start.
Fri Aug 24 02:29:50 2018 - [warning] Got error on MySQL connect: 2013 (Lost connection to MySQL server at 'reading initial communication packet', system error: 111)
Fri Aug 24 02:29:50 2018 - [warning] Connection failed 2 time(s)..
Fri Aug 24 02:29:51 2018 - [warning] Got error on MySQL connect: 2013 (Lost connection to MySQL server at 'reading initial communication packet', system error: 111)
Fri Aug 24 02:29:51 2018 - [warning] Connection failed 3 time(s)..
Fri Aug 24 02:29:52 2018 - [warning] Got error on MySQL connect: 2013 (Lost connection to MySQL server at 'reading initial communication packet', system error: 111)
Fri Aug 24 02:29:52 2018 - [warning] Connection failed 4 time(s)..
Fri Aug 24 02:29:52 2018 - [warning] Master is not reachable from health checker!
Fri Aug 24 02:29:52 2018 - [warning] Master 192.168.2.61(192.168.2.61:3306) is not reachable!
Fri Aug 24 02:29:52 2018 - [warning] SSH is reachable.
Fri Aug 24 02:29:52 2018 - [info] Connecting to a master server failed. Reading configuration file /etc/masterha_default.cnf and /etc/mha/app1/app1.cnf again, and trying to connect to all servers to check server status..
Fri Aug 24 02:29:52 2018 - [warning] Global configuration file /etc/masterha_default.cnf not found. Skipping.
Fri Aug 24 02:29:52 2018 - [info] Reading application default configuration from /etc/mha/app1/app1.cnf..
Fri Aug 24 02:29:52 2018 - [info] Reading server configuration from /etc/mha/app1/app1.cnf..
Fri Aug 24 02:29:52 2018 - [info] GTID failover mode = 0
Fri Aug 24 02:29:52 2018 - [info] Dead Servers:
Fri Aug 24 02:29:52 2018 - [info]   192.168.2.61(192.168.2.61:3306)
Fri Aug 24 02:29:52 2018 - [info] Alive Servers:
Fri Aug 24 02:29:52 2018 - [info]   192.168.2.62(192.168.2.62:3306)
Fri Aug 24 02:29:52 2018 - [info]   192.168.2.63(192.168.2.63:3306)
Fri Aug 24 02:29:52 2018 - [info] Alive Slaves:
Fri Aug 24 02:29:52 2018 - [info]   192.168.2.62(192.168.2.62:3306)  Version=5.7.21-log (oldest major version between slaves) log-bin:enabled
Fri Aug 24 02:29:52 2018 - [info]     Replicating from 192.168.2.61(192.168.2.61:3306)
Fri Aug 24 02:29:52 2018 - [info]     Primary candidate for the new Master (candidate_master is set)
Fri Aug 24 02:29:52 2018 - [info]   192.168.2.63(192.168.2.63:3306)  Version=5.7.21-log (oldest major version between slaves) log-bin:enabled
Fri Aug 24 02:29:52 2018 - [info]     Replicating from 192.168.2.61(192.168.2.61:3306)
Fri Aug 24 02:29:52 2018 - [info]     Not candidate for the new Master (no_master is set)
Fri Aug 24 02:29:52 2018 - [info] Checking slave configurations..
Fri Aug 24 02:29:52 2018 - [info]  read_only=1 is not set on slave 192.168.2.62(192.168.2.62:3306).
Fri Aug 24 02:29:52 2018 - [warning]  relay_log_purge=0 is not set on slave 192.168.2.62(192.168.2.62:3306).
Fri Aug 24 02:29:52 2018 - [info]  read_only=1 is not set on slave 192.168.2.63(192.168.2.63:3306).
Fri Aug 24 02:29:52 2018 - [warning]  relay_log_purge=0 is not set on slave 192.168.2.63(192.168.2.63:3306).
Fri Aug 24 02:29:52 2018 - [info] Checking replication filtering settings..
Fri Aug 24 02:29:52 2018 - [info]  Replication filtering check ok.
Fri Aug 24 02:29:52 2018 - [info] Master is down!
Fri Aug 24 02:29:52 2018 - [info] Terminating monitoring script.
Fri Aug 24 02:29:52 2018 - [info] Got exit code 20 (Master dead).
Fri Aug 24 02:29:52 2018 - [info] MHA::MasterFailover version 0.57.
Fri Aug 24 02:29:52 2018 - [info] Starting master failover.
Fri Aug 24 02:29:52 2018 - [info]
Fri Aug 24 02:29:52 2018 - [info] * Phase 1: Configuration Check Phase..
Fri Aug 24 02:29:52 2018 - [info]
Fri Aug 24 02:29:52 2018 - [info] GTID failover mode = 0
Fri Aug 24 02:29:52 2018 - [info] Dead Servers:
Fri Aug 24 02:29:52 2018 - [info]   192.168.2.61(192.168.2.61:3306)
Fri Aug 24 02:29:52 2018 - [info] Checking master reachability via MySQL(double check)...
Fri Aug 24 02:29:52 2018 - [info]  ok.
Fri Aug 24 02:29:52 2018 - [info] Alive Servers:
Fri Aug 24 02:29:52 2018 - [info]   192.168.2.62(192.168.2.62:3306)
Fri Aug 24 02:29:52 2018 - [info]   192.168.2.63(192.168.2.63:3306)
Fri Aug 24 02:29:52 2018 - [info] Alive Slaves:
Fri Aug 24 02:29:52 2018 - [info]   192.168.2.62(192.168.2.62:3306)  Version=5.7.21-log (oldest major version between slaves) log-bin:enabled
Fri Aug 24 02:29:52 2018 - [info]     Replicating from 192.168.2.61(192.168.2.61:3306)
Fri Aug 24 02:29:52 2018 - [info]     Primary candidate for the new Master (candidate_master is set)
Fri Aug 24 02:29:52 2018 - [info]   192.168.2.63(192.168.2.63:3306)  Version=5.7.21-log (oldest major version between slaves) log-bin:enabled
Fri Aug 24 02:29:52 2018 - [info]     Replicating from 192.168.2.61(192.168.2.61:3306)
Fri Aug 24 02:29:52 2018 - [info]     Not candidate for the new Master (no_master is set)
Fri Aug 24 02:29:52 2018 - [info] Starting Non-GTID based failover.
Fri Aug 24 02:29:52 2018 - [info]
Fri Aug 24 02:29:52 2018 - [info] ** Phase 1: Configuration Check Phase completed.
Fri Aug 24 02:29:52 2018 - [info]
Fri Aug 24 02:29:52 2018 - [info] * Phase 2: Dead Master Shutdown Phase..
Fri Aug 24 02:29:52 2018 - [info]
Fri Aug 24 02:29:52 2018 - [info] Forcing shutdown so that applications never connect to the current master..
Fri Aug 24 02:29:52 2018 - [info] Executing master IP deactivation script:
Fri Aug 24 02:29:52 2018 - [info]   /etc/mha/master_ip_failover --orig_master_host=192.168.2.61 --orig_master_ip=192.168.2.61 --orig_master_port=3306 --command=stopssh --ssh_user=root


IN SCRIPT TEST====/sbin/ifconfig eth0:0 down==/sbin/ifconfig eth0:0 192.168.6.66/24===

Disabling the VIP on old master: 192.168.2.61
Fri Aug 24 02:29:52 2018 - [info]  done.
Fri Aug 24 02:29:52 2018 - [warning] shutdown_script is not set. Skipping explicit shutting down of the dead master.
Fri Aug 24 02:29:52 2018 - [info] * Phase 2: Dead Master Shutdown Phase completed.
Fri Aug 24 02:29:52 2018 - [info]
Fri Aug 24 02:29:52 2018 - [info] * Phase 3: Master Recovery Phase..
Fri Aug 24 02:29:52 2018 - [info]
Fri Aug 24 02:29:52 2018 - [info] * Phase 3.1: Getting Latest Slaves Phase..
Fri Aug 24 02:29:52 2018 - [info]
Fri Aug 24 02:29:52 2018 - [info] The latest binary log file/position on all slaves is mysql-bin.000004:154
Fri Aug 24 02:29:52 2018 - [info] Latest slaves (Slaves that received relay log files to the latest):
Fri Aug 24 02:29:52 2018 - [info]   192.168.2.62(192.168.2.62:3306)  Version=5.7.21-log (oldest major version between slaves) log-bin:enabled
Fri Aug 24 02:29:52 2018 - [info]     Replicating from 192.168.2.61(192.168.2.61:3306)
Fri Aug 24 02:29:52 2018 - [info]     Primary candidate for the new Master (candidate_master is set)
Fri Aug 24 02:29:52 2018 - [info]   192.168.2.63(192.168.2.63:3306)  Version=5.7.21-log (oldest major version between slaves) log-bin:enabled
Fri Aug 24 02:29:52 2018 - [info]     Replicating from 192.168.2.61(192.168.2.61:3306)
Fri Aug 24 02:29:52 2018 - [info]     Not candidate for the new Master (no_master is set)
Fri Aug 24 02:29:52 2018 - [info] The oldest binary log file/position on all slaves is mysql-bin.000004:154
Fri Aug 24 02:29:52 2018 - [info] Oldest slaves:
Fri Aug 24 02:29:52 2018 - [info]   192.168.2.62(192.168.2.62:3306)  Version=5.7.21-log (oldest major version between slaves) log-bin:enabled
Fri Aug 24 02:29:52 2018 - [info]     Replicating from 192.168.2.61(192.168.2.61:3306)
Fri Aug 24 02:29:52 2018 - [info]     Primary candidate for the new Master (candidate_master is set)
Fri Aug 24 02:29:52 2018 - [info]   192.168.2.63(192.168.2.63:3306)  Version=5.7.21-log (oldest major version between slaves) log-bin:enabled
Fri Aug 24 02:29:52 2018 - [info]     Replicating from 192.168.2.61(192.168.2.61:3306)
Fri Aug 24 02:29:52 2018 - [info]     Not candidate for the new Master (no_master is set)
Fri Aug 24 02:29:52 2018 - [info]
Fri Aug 24 02:29:52 2018 - [info] * Phase 3.2: Saving Dead Master's Binlog Phase..
Fri Aug 24 02:29:52 2018 - [info]
Fri Aug 24 02:29:53 2018 - [info] Fetching dead master's binary logs..
Fri Aug 24 02:29:53 2018 - [info] Executing command on the dead master 192.168.2.61(192.168.2.61:3306): save_binary_logs --command=save --start_file=mysql-bin.000004  --start_pos=154 --binlog_dir=/mysql1/data --output_file=/etc/mha/app1/saved_master_binlog_from_192.168.2.61_3306_20180824022952.binlog --handle_raw_binlog=1 --disable_log_bin=0 --manager_version=0.57
  Creating /etc/mha/app1 if not exists..    ok.
 Concat binary/relay logs from mysql-bin.000004 pos 154 to mysql-bin.000004 EOF into /etc/mha/app1/saved_master_binlog_from_192.168.2.61_3306_20180824022952.binlog ..
 Binlog Checksum enabled
  Dumping binlog format description event, from position 0 to 154.. ok.
  Dumping effective binlog data from /mysql1/data/mysql-bin.000004 position 154 to tail(177).. ok.
 Binlog Checksum enabled
 Concat succeeded.
Fri Aug 24 02:29:53 2018 - [info] scp from root@192.168.2.61:/etc/mha/app1/saved_master_binlog_from_192.168.2.61_3306_20180824022952.binlog to local:/etc/mha/app1/saved_master_binlog_from_192.168.2.61_3306_20180824022952.binlog succeeded.
Fri Aug 24 02:29:53 2018 - [info] HealthCheck: SSH to 192.168.2.62 is reachable.
Fri Aug 24 02:29:54 2018 - [info] HealthCheck: SSH to 192.168.2.63 is reachable.
Fri Aug 24 02:29:54 2018 - [info]
Fri Aug 24 02:29:54 2018 - [info] * Phase 3.3: Determining New Master Phase..
Fri Aug 24 02:29:54 2018 - [info]
Fri Aug 24 02:29:54 2018 - [info] Finding the latest slave that has all relay logs for recovering other slaves..
Fri Aug 24 02:29:54 2018 - [info] All slaves received relay logs to the same position. No need to resync each other.
Fri Aug 24 02:29:54 2018 - [info] Searching new master from slaves..
Fri Aug 24 02:29:54 2018 - [info]  Candidate masters from the configuration file:
Fri Aug 24 02:29:54 2018 - [info]   192.168.2.62(192.168.2.62:3306)  Version=5.7.21-log (oldest major version between slaves) log-bin:enabled
Fri Aug 24 02:29:54 2018 - [info]     Replicating from 192.168.2.61(192.168.2.61:3306)
Fri Aug 24 02:29:54 2018 - [info]     Primary candidate for the new Master (candidate_master is set)
Fri Aug 24 02:29:54 2018 - [info]  Non-candidate masters:
Fri Aug 24 02:29:54 2018 - [info]   192.168.2.63(192.168.2.63:3306)  Version=5.7.21-log (oldest major version between slaves) log-bin:enabled
Fri Aug 24 02:29:54 2018 - [info]     Replicating from 192.168.2.61(192.168.2.61:3306)
Fri Aug 24 02:29:54 2018 - [info]     Not candidate for the new Master (no_master is set)
Fri Aug 24 02:29:54 2018 - [info]  Searching from candidate_master slaves which have received the latest relay log events..
Fri Aug 24 02:29:54 2018 - [info] New master is 192.168.2.62(192.168.2.62:3306)
Fri Aug 24 02:29:54 2018 - [info] Starting master failover..
Fri Aug 24 02:29:54 2018 - [info]
From:
192.168.2.61(192.168.2.61:3306) (current master)
 +--192.168.2.62(192.168.2.62:3306)
 +--192.168.2.63(192.168.2.63:3306)

To:
192.168.2.62(192.168.2.62:3306) (new master)
 +--192.168.2.63(192.168.2.63:3306)
Fri Aug 24 02:29:54 2018 - [info]
Fri Aug 24 02:29:54 2018 - [info] * Phase 3.3: New Master Diff Log Generation Phase..
Fri Aug 24 02:29:54 2018 - [info]
Fri Aug 24 02:29:54 2018 - [info]  This server has all relay logs. No need to generate diff files from the latest slave.
Fri Aug 24 02:29:54 2018 - [info] Sending binlog..
Fri Aug 24 02:29:55 2018 - [info] scp from local:/etc/mha/app1/saved_master_binlog_from_192.168.2.61_3306_20180824022952.binlog to root@192.168.2.62:/etc/mha/app1/saved_master_binlog_from_192.168.2.61_3306_20180824022952.binlog succeeded.
Fri Aug 24 02:29:55 2018 - [info]
Fri Aug 24 02:29:55 2018 - [info] * Phase 3.4: Master Log Apply Phase..
Fri Aug 24 02:29:55 2018 - [info]
Fri Aug 24 02:29:55 2018 - [info] *NOTICE: If any error happens from this phase, manual recovery is needed.
Fri Aug 24 02:29:55 2018 - [info] Starting recovery on 192.168.2.62(192.168.2.62:3306)..
Fri Aug 24 02:29:55 2018 - [info]  Generating diffs succeeded.
Fri Aug 24 02:29:55 2018 - [info] Waiting until all relay logs are applied.
Fri Aug 24 02:29:55 2018 - [info]  done.
Fri Aug 24 02:29:55 2018 - [info] Getting slave status..
Fri Aug 24 02:29:55 2018 - [info] This slave(192.168.2.62)'s Exec_Master_Log_Pos equals to Read_Master_Log_Pos(mysql-bin.000004:154). No need to recover from Exec_Master_Log_Pos.
Fri Aug 24 02:29:55 2018 - [info] Connecting to the target slave host 192.168.2.62, running recover script..
Fri Aug 24 02:29:55 2018 - [info] Executing command: apply_diff_relay_logs --command=apply --slave_user='root' --slave_host=192.168.2.62 --slave_ip=192.168.2.62  --slave_port=3306 --apply_files=/etc/mha/app1/saved_master_binlog_from_192.168.2.61_3306_20180824022952.binlog --workdir=/etc/mha/app1 --target_version=5.7.21-log --timestamp=20180824022952 --handle_raw_binlog=1 --disable_log_bin=0 --manager_version=0.57 --slave_pass=xxx
Fri Aug 24 02:29:55 2018 - [info]
MySQL client version is 5.7.21. Using --binary-mode.
Applying differential binary/relay log files /etc/mha/app1/saved_master_binlog_from_192.168.2.61_3306_20180824022952.binlog on 192.168.2.62:3306. This may take long time...
Applying log files succeeded.
Fri Aug 24 02:29:55 2018 - [info]  All relay logs were successfully applied.
Fri Aug 24 02:29:55 2018 - [info] Getting new master's binlog name and position..
Fri Aug 24 02:29:55 2018 - [info]  mysql-bin.000012:234
Fri Aug 24 02:29:55 2018 - [info]  All other slaves should start replication from here. Statement should be: CHANGE MASTER TO MASTER_HOST='192.168.2.62', MASTER_PORT=3306, MASTER_LOG_FILE='mysql-bin.000012', MASTER_LOG_POS=234, MASTER_USER='dao', MASTER_PASSWORD='xxx';
Fri Aug 24 02:29:55 2018 - [info] Executing master IP activate script:
Fri Aug 24 02:29:55 2018 - [info]   /etc/mha/master_ip_failover --command=start --ssh_user=root --orig_master_host=192.168.2.61 --orig_master_ip=192.168.2.61 --orig_master_port=3306 --new_master_host=192.168.2.62 --new_master_ip=192.168.2.62 --new_master_port=3306 --new_master_user='root'   --new_master_password=xxx
Unknown option: new_master_user
Unknown option: new_master_password


IN SCRIPT TEST====/sbin/ifconfig eth0:0 down==/sbin/ifconfig eth0:0 192.168.6.66/24===

Enabling the VIP - 192.168.6.66/24 on the new master - 192.168.2.62
Fri Aug 24 02:29:56 2018 - [info]  OK.
Fri Aug 24 02:29:56 2018 - [info] ** Finished master recovery successfully.
Fri Aug 24 02:29:56 2018 - [info] * Phase 3: Master Recovery Phase completed.
Fri Aug 24 02:29:56 2018 - [info]
Fri Aug 24 02:29:56 2018 - [info] * Phase 4: Slaves Recovery Phase..
Fri Aug 24 02:29:56 2018 - [info]
Fri Aug 24 02:29:56 2018 - [info] * Phase 4.1: Starting Parallel Slave Diff Log Generation Phase..
Fri Aug 24 02:29:56 2018 - [info]
Fri Aug 24 02:29:56 2018 - [info] -- Slave diff file generation on host 192.168.2.63(192.168.2.63:3306) started, pid: 3019. Check tmp log /etc/mha/app1/192.168.2.63_3306_20180824022952.log if it takes time..
Fri Aug 24 02:29:56 2018 - [info]
Fri Aug 24 02:29:56 2018 - [info] Log messages from 192.168.2.63 ...
Fri Aug 24 02:29:56 2018 - [info]
Fri Aug 24 02:29:56 2018 - [info]  This server has all relay logs. No need to generate diff files from the latest slave.
Fri Aug 24 02:29:56 2018 - [info] End of log messages from 192.168.2.63.
Fri Aug 24 02:29:56 2018 - [info] -- 192.168.2.63(192.168.2.63:3306) has the latest relay log events.
Fri Aug 24 02:29:56 2018 - [info] Generating relay diff files from the latest slave succeeded.
Fri Aug 24 02:29:56 2018 - [info]
Fri Aug 24 02:29:56 2018 - [info] * Phase 4.2: Starting Parallel Slave Log Apply Phase..
Fri Aug 24 02:29:56 2018 - [info]
Fri Aug 24 02:29:56 2018 - [info] -- Slave recovery on host 192.168.2.63(192.168.2.63:3306) started, pid: 3021. Check tmp log /etc/mha/app1/192.168.2.63_3306_20180824022952.log if it takes time..
Fri Aug 24 02:29:56 2018 - [info]
Fri Aug 24 02:29:56 2018 - [info] Log messages from 192.168.2.63 ...
Fri Aug 24 02:29:56 2018 - [info]
Fri Aug 24 02:29:56 2018 - [info] Sending binlog..
Fri Aug 24 02:29:56 2018 - [info] scp from local:/etc/mha/app1/saved_master_binlog_from_192.168.2.61_3306_20180824022952.binlog to root@192.168.2.63:/etc/mha/app1/saved_master_binlog_from_192.168.2.61_3306_20180824022952.binlog succeeded.
Fri Aug 24 02:29:56 2018 - [info] Starting recovery on 192.168.2.63(192.168.2.63:3306)..
Fri Aug 24 02:29:56 2018 - [info]  Generating diffs succeeded.
Fri Aug 24 02:29:56 2018 - [info] Waiting until all relay logs are applied.
Fri Aug 24 02:29:56 2018 - [info]  done.
Fri Aug 24 02:29:56 2018 - [info] Getting slave status..
Fri Aug 24 02:29:56 2018 - [info] This slave(192.168.2.63)'s Exec_Master_Log_Pos equals to Read_Master_Log_Pos(mysql-bin.000004:154). No need to recover from Exec_Master_Log_Pos.
Fri Aug 24 02:29:56 2018 - [info] Connecting to the target slave host 192.168.2.63, running recover script..
Fri Aug 24 02:29:56 2018 - [info] Executing command: apply_diff_relay_logs --command=apply --slave_user='root' --slave_host=192.168.2.63 --slave_ip=192.168.2.63  --slave_port=3306 --apply_files=/etc/mha/app1/saved_master_binlog_from_192.168.2.61_3306_20180824022952.binlog --workdir=/etc/mha/app1 --target_version=5.7.21-log --timestamp=20180824022952 --handle_raw_binlog=1 --disable_log_bin=0 --manager_version=0.57 --slave_pass=xxx
Fri Aug 24 02:29:56 2018 - [info]
MySQL client version is 5.7.21. Using --binary-mode.
Applying differential binary/relay log files /etc/mha/app1/saved_master_binlog_from_192.168.2.61_3306_20180824022952.binlog on 192.168.2.63:3306. This may take long time...
Applying log files succeeded.
Fri Aug 24 02:29:56 2018 - [info]  All relay logs were successfully applied.
Fri Aug 24 02:29:56 2018 - [info]  Resetting slave 192.168.2.63(192.168.2.63:3306) and starting replication from the new master 192.168.2.62(192.168.2.62:3306)..
Fri Aug 24 02:29:56 2018 - [info]  Executed CHANGE MASTER.
Fri Aug 24 02:29:56 2018 - [info]  Slave started.
Fri Aug 24 02:29:56 2018 - [info] End of log messages from 192.168.2.63.
Fri Aug 24 02:29:56 2018 - [info] -- Slave recovery on host 192.168.2.63(192.168.2.63:3306) succeeded.
Fri Aug 24 02:29:56 2018 - [info] All new slave servers recovered successfully.
Fri Aug 24 02:29:56 2018 - [info]
Fri Aug 24 02:29:56 2018 - [info] * Phase 5: New master cleanup phase..
Fri Aug 24 02:29:56 2018 - [info]
Fri Aug 24 02:29:56 2018 - [info] Resetting slave info on the new master..
Fri Aug 24 02:29:56 2018 - [info]  192.168.2.62: Resetting slave info succeeded.
Fri Aug 24 02:29:56 2018 - [info] Master failover to 192.168.2.62(192.168.2.62:3306) completed successfully.
Fri Aug 24 02:29:56 2018 - [info]

----- Failover Report -----

app1: MySQL Master failover 192.168.2.61(192.168.2.61:3306) to 192.168.2.62(192.168.2.62:3306) succeeded

Master 192.168.2.61(192.168.2.61:3306) is down!

Check MHA Manager logs at node3:/etc/mha/app1/manager.log for details.

Started automated(non-interactive) failover.
Invalidated master IP address on 192.168.2.61(192.168.2.61:3306)
The latest slave 192.168.2.62(192.168.2.62:3306) has all relay logs for recovery.
Selected 192.168.2.62(192.168.2.62:3306) as a new master.
192.168.2.62(192.168.2.62:3306): OK: Applying all logs succeeded.
192.168.2.62(192.168.2.62:3306): OK: Activated master IP address.
192.168.2.63(192.168.2.63:3306): This host has the latest relay log events.
Generating relay diff files from the latest slave succeeded.
192.168.2.63(192.168.2.63:3306): OK: Applying all logs succeeded. Slave started, replicating from 192.168.2.62(192.168.2.62:3306)
192.168.2.62(192.168.2.62:3306): Resetting slave info succeeded.
Master failover to 192.168.2.62(192.168.2.62:3306) completed successfully.




```

### 查看从库同步信息

- 发现已经切换到62

```sql

mysql> show slave status \G ;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.2.62
                  Master_User: dao
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000006
          Read_Master_Log_Pos: 1098
               Relay_Log_File: node3-relay-bin.000002
                Relay_Log_Pos: 320
        Relay_Master_Log_File: mysql-bin.000006
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 1098
              Relay_Log_Space: 527
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 0
               Last_SQL_Error:
  Replicate_Ignore_Server_Ids:
             Master_Server_Id: 2
                  Master_UUID: f63b765b-9c1d-11e8-aa1e-005056296858
             Master_Info_File: /mysql1/data/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind:
      Last_IO_Error_Timestamp:
     Last_SQL_Error_Timestamp:
               Master_SSL_Crl:
           Master_SSL_Crlpath:
           Retrieved_Gtid_Set:
            Executed_Gtid_Set:
                Auto_Position: 0
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
1 row in set (0.00 sec)


```

### 查看IP地址信息

```sql


[root@node2 ~]# ifconfig
eth0      Link encap:Ethernet  HWaddr 00:50:56:29:68:58
          inet addr:192.168.2.62  Bcast:192.168.2.255  Mask:255.255.255.0
          inet6 addr: fe80::250:56ff:fe29:6858/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:5161 errors:0 dropped:0 overruns:0 frame:0
          TX packets:4634 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:588932 (575.1 KiB)  TX bytes:643527 (628.4 KiB)

eth0:0    Link encap:Ethernet  HWaddr 00:50:56:29:68:58
          inet addr:192.168.6.66  Bcast:192.168.6.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:16436  Metric:1
          RX packets:1522 errors:0 dropped:0 overruns:0 frame:0
          TX packets:1522 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:136401 (133.2 KiB)  TX bytes:136401 (133.2 KiB)


```