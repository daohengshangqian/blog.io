---
layout: post
title: 使用java读取hdfs上的文件
date: 2018-11-28
categories: blog
tags: [bigdata]
description: 使用java读取hdfs上的文件
---
##  代码

```sql

package hadoop_study;

import java.io.InputStream;
import java.net.URL;
import java.net.URLConnection;

import org.apache.hadoop.fs.FsUrlStreamHandlerFactory;
import org.junit.Test;

public class TestHDFS {
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

	public static void main(String[] args) {
		TestHDFS th = new TestHDFS();
		try {
			th.readFile();
		} catch (Exception e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
}


```

## 运行结果

```sql

hello hadoop !

```

##  hdfs shell 验证


```sql

[root@node1 ~]# hdfs dfs -ls /hello.txt
-rw-r--r--   3 root supergroup         15 2018-11-22 22:56 /hello.txt
[root@node1 ~]# hdfs dfs -cat  /hello.txt
hello hadoop !



```