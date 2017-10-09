---
layout: post
title: Zookeeper 分布式安装笔记
tags: 
  - program
  - bigdata
  - zookeeper
---

实验室第N次服务器配置，简单记录以备不时之需。

## 环境配置

```shell
sudo vi /etc/profile
export ZOOKEEPER_HOME=/home/bjut/zookeeper-3.3.6
export PATH=$PATH:$ZOOKEEPER_HOME/bin
```

执行 `source /etc/profile` 使用设置生效。

## Zookeeper 配置 

- 文件 `zoo.cfg`
  ```shell
  cp conf/zoo_sample.cfg zoo.cfg
  vi zoo.cfg
  # The number of milliseconds of each tick
  tickTime=2000
  # The number of ticks that the initial 
  # synchronization phase can take
  initLimit=10
  # The number of ticks that can pass between 
  # sending a request and getting an acknowledgement
  syncLimit=5
  # the directory where the snapshot is stored.
  dataDir=/tmp/zookeeper
  # the port at which the clients will connect
  clientPort=2181

  server.1=Master:2888:3888
  server.2=Slave1:2888:3888
  server.3=Slave2:2888:3888
  server.4=Slave3:2888:3888
  server.5=Slave4:2888:3888
  server.6=Slave5:2888:3888
  ```
  将配置好的 zookeeper 依次发送到每个 slave 节点。
- 文件 `myid`
  修改各个节点id值。
  在之前配置的 dataDir 指定的目录下创建一个 myid 文件，里面内容为一个数字，用来标识当前主机，Master 为 1，Slave1 为 2，Slave3 为 3，以此类推。
  ```shell
  mkdir -p /tmp//zookeeper
  cd /tmp/zookeeper
  touch myid
  echo "1" > myid
  ```

## 启动 Zookeeper

按照上述配置的顺序**依次**启动，并且在各个节点上都要启动 Zookeeper，在 shell 中输入

```shell
zkServer.sh start  #启动Zookeeper服务，正确启动只有，使用jps命令会看到QuorumPeerMain，如果该进程启动说明Zookeeper服务成功的启动
zkServer.sh status #查看Zookeeper服务的状态，你会看到哪个节点是Leader节点，哪个节点是Follower节点，并且只有一个Zookeeper节点
zkServer.sh stop  #停止Zookeeper服务，每个节点都要停止
```

## 参考资料

- [Zookeeper分布式集群的安装与配置](http://blog.csdn.net/jpzhu16/article/details/51751363)
- [初识zookeeper（一）之zookeeper的安装及配置](http://www.cnblogs.com/bookwed/p/4599829.html)