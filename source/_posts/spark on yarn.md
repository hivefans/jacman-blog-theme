---
layout: post
title:  "spark on yarn"
keywords: "spark yarn"
description: "为什么要使用YARN? 数据共享、资源利用率、更方便的管理集群等。"
category: spark
tags: [spark,yarn]
---


## 为什么要使用YARN? ##

数据共享、资源利用率、更方便的管理集群等。

详情参见：http://www.cnblogs.com/luogankun/p/3887019.html

## Spark YARN版本编译 ##

编译hadoop对应的支持YARN的Spark版本

    export MAVEN_OPTS="-Xmx2g -XX:MaxPermSize=512M -XX:ReservedCodeCacheSize=512m"
    mvn clean package -DskipTests -Phadoop-2.3 -Dhadoop.version=2.3.0-cdh5.0.0 -Dprotobuf.version=2.5.0 -Pyarn -Phive

详情参见：http://www.cnblogs.com/luogankun/p/3798403.html
Spark On YARN

Spark的Cluster Manager负责管理启动executor进程，集群可以是Standalone、YARN和Mesos

每个SparkContext（换句话说是：Application）对应一个ApplicationMaster（Application启动过程中的第一个容器

ApplicationMaster负责和ResourceManager打交道，并请求资源，当获取资源之后通知NodeManager为其启动container； 每个Container中运行一个ExecutorBackend

ResourceManager决定哪些Application可以运行、什么时候运行以及在哪些NodeManager上运行； NodeManager的Container上运行executor进程

在Standalone模式中有Worker的概念，而在Spark On YARN中没有Worker的概念

由于executor是运行在container中，故container内存要大于executor的内存

Spark On YARN有两种：

1、yarn-client

　　Client和Driver运行在一起，ApplicationMaster只负责获取资源

　　Client会和请求到的资源container通信来调度他们进行工作，也就是说Client不能退出滴；

　　日志信息输出能输出在终端控制台上，适用于交互或者调试，也就是希望快速地看到application的输出，比如SparkStreaming
![yarn-client](/img/161514368446695.png)

2、yarn-cluster

　　Driver和ApplicationMaster运行在一起；负责向YARN申请资源，并检测作业的运行状况；executor运行在container中

　　提交Application之后，即使关掉了Client，作业仍然会继续在YARN上运行

　　日志信息不会输出在终端控制台上
![yarn-cluster](/img/161515016875485.png)

**提交Spark作业到YARN**

提交命令

    ./bin/spark-submit \
      --class <main-class>
      --master <master-url> \
      --deploy-mode <deploy-mode> \
      ... # other options
      <application-jar> \
      [application-arguments]

1、提交本地jar

提交到yarn-cluster/yarn-client

    ./bin/spark-submit \
      --class org.apache.spark.examples.SparkPi \
      --master yarn-cluster \  # can also be `yarn-client` for client mode
      --executor-memory 20G \
      --num-executors 50 \
      /path/to/examples.jar \

如果采用的是yarn-cluster的方式运行的话，想停止执行应用，需要去多个node上干掉；而在yarn-client模式运行时，只需要在client上干掉应用即可。

提交到standalone

    ./bin/spark-submit \
      --class org.apache.spark.examples.SparkPi \
      --master spark://207.184.161.138:7077 \
      --executor-memory 20G \
      --total-executor-cores 100 \
      /path/to/examples.jar \

2、提交hdfs上的jar

    ./bin/spark-submit \
      --class org.apache.spark.examples.SparkPi \
      --master yarn-cluster \  # can also be `yarn-client` for client mode
      --executor-memory 20G \
      --num-executors 50 \
      hdfs://hadoop000:8020/lib/examples.jar \

如果没有在spark-env.sh文件中配置HADOOP_CONF_DIR或者YARN_CONF_DIR，可以在提交作业前指定形如

    export HADOOP_CONF_DIR=XXX
    ./bin/spark-submit \
      --class org.apache.spark.examples.SparkPi \
      --master yarn-cluster \  # can also be `yarn-client` for client mode
      --executor-memory 20G \
      --num-executors 50 \
      /path/to/examples.jar \
详情参见：http://spark.apache.org/docs/latest/submitting-applications.html
