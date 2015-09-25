---
layout: post
title:  "Spark Standalone模式HA环境搭建"
keywords: "spark HA"
description: "Spark Standalone模式常见的HA部署方式有两种：基于文件系统的HA和基于ZK的HA"
category: spark
tags: [spark,HA]
---

Spark Standalone模式常见的HA部署方式有两种：基于文件系统的HA和基于ZK的HA 

本篇只介绍基于ZK的HA环境搭建：

$SPARK_HOME/conf/spark-env.sh

添加SPARK_DAEMON_JAVA_OPTS的配置信息：

    export SPARK_DAEMON_JAVA_OPTS="-Dspark.deploy.recoveryMode=ZOOKEEPER -Dspark.deploy.zookeeper.url=hadoop000:2181,hadoop001:2181,hadoop002:2181 -Dspark.deploy.zookeeper.dir=/spark"
配置参数说明：

spark.deploy.recoveryMode: 设置恢复模式为zk，默认为NONE

spark.deploy.zookeeper.url: 设置ZK集群的url，形如：192.168.1.100:2181,192.168.1.101:2181

spark.deploy.zookeeper.dir: 设置zk保存恢复状态的路径，默认为spark

 

实现HA的原理：利用ZK的Leader Election机制，选择一个Active状态的Master，其余的Master均为Standby状态；当Active状态的Master死掉后，通过ZK选举一个Standby状态的Master为Active状态。

 

测试步骤：

启动standalone集群后，在各个Standby节点上启动start-master.sh，jps观察是否已经正确启动Master进程；

将Active状态的Master kill掉，观察8080端口对应的页面，发现已经从Standby状态中选举出一个当作Active状态。

采用ZK后由于会有多个Master，在提交任务时不知道哪个为Active状态的Master，可以采用如下的方式提交：

spark-shell --master spark://hadoop000:7077,hadoop001:7077,hadoop002:7077 --executor-memory 2g --total-executor-cores 1

详细信息参见官方文档：http://spark.apache.org/docs/latest/spark-standalone.html#standby-masters-with-zookeeper