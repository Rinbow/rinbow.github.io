---
layout: post
title: Spark 单机和分布式安装笔记
categories: [BigData, Spark]
tags:
  - program
  - bigdata
  - spark
---

实验室第N次服务器配置，简单记录以备不时之需。

## 单机模式安装

### 环境配置

- `/etc/profile`
  ```shell
  sudo vi /etc/profile
  export SCALA_HOME=/home/bjut/scala-2.11.7
  export PATH=$SCALA_HOME/bin:$PATH
  ```
- `~./bashrc`
  ```shell
  vi ~/.bashrc
  export HADOOP_HOME=/home/bjut/hadoop-2.6.0
  export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
  export SCALA_HOME=/home/bjut/scala-2.11.7
  export SPARK_HOME=/home/bjut/software/spark-1.5.2-bin-hadoop2.6
  export SPARK_LIBARY_PATH=.:$JAVA_HOME/lib:$JAVA_HOME/jre/lib:$HADOOP_HOME/lib/native
  export YARN_CONF_DIR=$HADOOP_HOME/etc/hadoop
  export PATH=$PATH:$SCALA_HOME/bin:$SPARK_HOME/bin:/home/bjut/hadoop-2.6.0/bin:/home/bjut/hadoop-2.6.0/sbin
  ```
- 让配置生效
  ```shell
  source /etc/proifle
  source ~/.bashrc
  ```

### 启动 Spark

```shell
sbin/start-all.sh
```

  执行 `jps`，有一个 Master 和 Worker 进程，说明启动成功。

## 分布式安装

### 环境配置

同单机模式操作一样。

### Spark 配置

- 文件 `spark-env.sh`
  ```shell
  cd spark-1.5.2-bin-hadoop2.6/conf
  cp spark-env.sh.template spark-env.sh
  vi spark-env.sh
  export SCALA_HOME=/home/bjut/scala-2.11.7
  export JAVA_HOME=/home/bjut/jdk1.7
  export SPARK_MASTER_IP=172.21.20.97
  export SPARK_WORKER_INSTANCES=3
  export SPARK_MASTER_PORT=8070
  export SPARK_MASTER_WEBUI_PORT=8090
  export SPARK_WORKER_PORT=8092
  export SPARK_WORKER_MEMORY=5000m
  ```
- 文件 `slaves`
  ```shell
  cp slaves.template slaves
  172.21.20.97
  172.21.20.119
  172.21.20.94
  172.21.20.167
  172.21.20.221
  172.21.20.127
  ```
- 启动 Spark 集群
  ```shell
  sbin/start-all.sh
  ```

## 参考资料

- [Spark 伪分布式 & 全分布式 安装指南](https://my.oschina.net/leejun2005/blog/394928)