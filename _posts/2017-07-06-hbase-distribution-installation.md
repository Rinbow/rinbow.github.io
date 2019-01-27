---
title: HBase 分布式安装笔记
tags: [bigdata, hbase]
---

实验室第N次服务器配置，简单记录以备不时之需。

## 环境配置

```shell
sudo vi /etc/profile
export HBASE_HOME=/home/bjut/hbase-1.1.11
export PATH=$PATH:$HBASE_HOME/bin
```

## 安装 HBase

- 文件 `hbase-env.sh`

  ```shell
  export JAVA_HOME=/home/bjut/jdk1.7
  ```

- 文件 `hbase-site.xml`

  ```xml
  <configuration>
      <property>
          <name>hbase.master.maxclockskew</name>
          <value>180000</value>
          <description>Time difference of regionserver from master</description>
      </property>
      <property>
          <name>hbase.rootdir</name>
          <value>hdfs://master:9000/hbase</value>
      </property>
      <property>
          <name>hbase.cluster.distributed</name>
          <value>true</value>
      </property>
      <property>
          <name>hbase.zookeeper.quorum</name>
          <value>Slave1,Slave2,Slave3,Slave4,Slave5</value>
      </property>
      <property>
          <name>hbase.zookeeper.property.dataDir</name>
          <value>/home/bjut/zookeeper-3.3.6</value>
      </property>
  </configuration>
  ```

- 文件 `regionservers`

  ```
  Slave1
  Slave2
  Slave3
  Slave4
  Slave5
  ```

- 发送配置好的 hbase 到各个 slave 节点

- 启动 HBase

  启动 HBase 时要确保 hdfs 已经启动，HBase 的启动顺序为：HDFS -> Zookeeper -> HBase。

  在 Master 节点执行 `jps`，结果如下：

   ![masterHbaseJps](\media\files\2017\07\06\masterHbaseJps.png)

  在 Slave 节点执行 `jps`，结果如下：

  ![slaveHbaseJps](\media\files\2017\07\06\slaveHbaseJps.png)

## 访问 Web 页面

访问地址 `http://127.21.20.127:16030`，效果如下：

![webHbase](\media\files\2017\07\06\webHbase.png)

---

### 参考资料

- [hadoop2.6完全分布式安装HBase1.1](https://yq.aliyun.com/articles/32314)
- [Hbase的regionServer无法启动报ClockOutOfSyncException解决方法](http://myhadoop.iteye.com/blog/2044253)
- [Hbase完全分布式集群安装配置(Hbase1.0.0,Hadoop2.6.0)](http://blog.csdn.net/wuwenxiang91322/article/details/44684655)
- [手把手教你安装Hbase,一次成功！](http://blog.csdn.net/achuo/article/details/51170946)