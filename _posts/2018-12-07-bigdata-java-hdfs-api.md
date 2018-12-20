---
layout: post
title: java调用Hadoop API操作HDFS
date: 2018-12-07
categories: blog
tags: [bigdata]
description: java调用Hadoop API操作HDFS
---
##  代码

```sql


package hadoop_study;

import java.io.InputStream;
import java.net.URL;
import java.net.URLConnection;

import org.apache.commons.configuration.ConfigurationRuntimeException;
import org.apache.hadoop.fs.FsUrlStreamHandlerFactory;
import org.apache.tomcat.dbcp.pool.impl.GenericKeyedObjectPool.Config;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FSDataInputStream;
import org.apache.hadoop.fs.FSDataOutputStream;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IOUtils;
import org.junit.Test;

public class TestHDFSApi {

	private boolean delete;


	@Test
	public void readFile() throws Exception {
		URL.setURLStreamHandlerFactory(new FsUrlStreamHandlerFactory());

		URL url = new URL("hdfs://192.168.2.61:8020/hello.txt");
		URLConnection conn = url.openConnection();
		InputStream is = conn.getInputStream();
		byte[] buf = new byte[is.available()];
		is.read(buf);
		is.close();
		String str = new String(buf);
		System.out.println(str);

	}
	
	@Test
	public void mkdir() throws Exception {
		Configuration conf = new Configuration();
		conf.set("fs.defaultFS","hdfs://192.168.2.61:8020/") ;
		FileSystem fs = FileSystem.get(conf);
		fs.mkdirs(new Path("/daodir/aaa"));
	}

	@Test
	public void putFile() throws Exception{
		Configuration conf = new Configuration();
		conf.set("fs.defaultFS","hdfs://192.168.2.61:8020/") ;
		FileSystem fs = FileSystem.get(conf) ;
		FSDataOutputStream out = fs.create(new Path("/dao_put.txt"));
		out.write("helloworld".getBytes());
		out.close();
	}
	
   @Test
   public void removeFile() throws Exception{
		Configuration conf = new Configuration();
		conf.set("fs.defaultFS","hdfs://192.168.2.61:8020/") ;
		FileSystem fs = FileSystem.get(conf);
		Path p = new Path("/dao_put.txt");
		fs.delete(p, true);
   }
   
```

## 验证

###  初始 
···sql

[root@node1 ~]# hdfs dfs -ls /
Found 2 items
-rw-r--r--   3 root supergroup         15 2018-11-22 22:56 /hello.txt
-rw-r--r--   3 root supergroup        153 2018-11-19 17:17 /xcall.sh


···

### 调用readfile

```sql

hello hadoop !


```

###  调用mkdir

```sql

[root@node1 ~]# hdfs dfs -ls /
Found 3 items

drwxr-xr-x   - dao  supergroup          0 2018-11-24 20:26 /daodir
-rw-r--r--   3 root supergroup         15 2018-11-22 22:56 /hello.txt
-rw-r--r--   3 root supergroup        153 2018-11-19 17:17 /xcall.sh



```

### 调用put

```sql

[root@node1 ~]# hdfs dfs -ls /
Found 4 items
-rw-r--r--   3 dao  supergroup          0 2018-11-24 20:32 /dao_put.txt
drwxr-xr-x   - dao  supergroup          0 2018-11-24 20:26 /daodir
-rw-r--r--   3 root supergroup         15 2018-11-22 22:56 /hello.txt
-rw-r--r--   3 root supergroup        153 2018-11-19 17:17 /xcall.sh


```

###  调用removeFile

```


[root@node1 ~]# hdfs dfs -ls /
Found 3 items
drwxr-xr-x   - dao  supergroup          0 2018-11-24 20:26 /daodir
-rw-r--r--   3 root supergroup         15 2018-11-22 22:56 /hello.txt
-rw-r--r--   3 root supergroup        153 2018-11-19 17:17 /xcall.sh


```