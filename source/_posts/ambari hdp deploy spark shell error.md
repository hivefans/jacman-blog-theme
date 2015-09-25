---
layout: post
title:  "ambari hdp中部署apache spark运行spark shell遇到的错误解决"
keywords: "ambari hdp spark shell"
description: "ambari hdp中部署apache spark运行spark shell遇到的错误解决方法"
category: [spark, sticky]
tags: [spark,ambari]
---

在运行spark-shell中遇到的ERROR lzo.GPLNativeCodeLoader: Could not load native gpl library  解决方法

bin/spark-shell --driver-library-path :/usr/hdp/2.2.4.2-2/hadoop/lib/native/Linux-amd64-64:/usr/hdp/2.2.4.2-2/hadoop/lib/hadoop-lzo-0.6.0.2.2.4.2-2.jar

在运行spark-shell中遇到的Compression codec com.hadoop.compression.lzo.LzoCodec not found  错误可以配置文件spark-defaults.conf
  spark.executor.extraClassPath    /usr/hdp/2.2.4.2-2/hadoop/lib/hadoop-lzo-0.6.0.2.2.4.2-2.jar
  spark.driver.extraClassPath      /usr/hdp/2.2.4.2-2/hadoop/lib/hadoop-lzo-0.6.0.2.2.4.2-2.jar
保存文件重启spark服务集群即可。

再提供一个Unable to load native-hadoop library 和 Snappy native library not loaded的解决方案。这个问题主要是jre目录下缺少了libgplcompression.so , libhadoop.so和libsnappy.so两个文件。具体是，spark-shell依赖的是scala，scala依赖的是JAVA_HOME下的jdk，libhadoop.so和libsnappy.so两个文件应该放到JAVA_HOME/jre/lib/amd64下面。要注意的是要知道真正依赖到的JAVA_HOME是哪一个，把两个.so放对地方。这两个so：libhadoop.so和libsnappy.so。前一个so可以在HADOOP_HOME下找到，比如hadoop\lib\native\Linux-amd64-64。第二个libsnappy.so需要下载一个snappy-1.1.0.tar.gz，然后./configure，make编译出来。snappy是google的一个压缩算法，在hadoop jira下https://issues.apache.org/jira/browse/HADOOP-7206记录了这次集成。