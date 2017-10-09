---
layout: post
title: Hadoop 分布式安装笔记
tags: 
  - program
  - hadoop
  - bigdata
---

实验室第N次服务器配置，简单记录以备不时之需。

## Master 配置

### 安装 JDK

- `/etc/profile`

  `sudo vi /etc/profile`，设置 `JAVA` 路径，如下图所示：
  ![etcProfile](\media\files\2017\07\04\etcProfile.png)
- `~/.bashrc`

  `sudo vi ~./bashrc`，设置 `JAVA_HOME`，如下图所示：
  ![bashrc](\media\files\2017\07\04\bashrc.png)
- 执行 `java -version` 检查效果。

### 修改主机名

- `sudo vi /etc/hostname`

  更改 master 节点名称，这里我将其更改为`Master`。

- `sudo vi /etc/hosts`

  修改主机名与 IP 的映射关系，内容如下图所示，master 其余五台 slave 机器的 IP 即主机名按如下方式填写。

  ![modifyNameIP](\media\files\2017\07\04\modifyNameIP.png)

### SSH 配置

为了让 Master 节点可以无密码登录到各个 Slave 节点上，需要配置 SSH。

- 首先执行 `ssh localhost` 生成 `.shh` 目录，然后 `exit` 退出当前 SSH 登录

- ```shell
  cd ~/.ssh/
  ssh-keygen -t rsa #遇到提示直接按回车即可
  cat id_rsa.pub >> authorized_keys #加入授权
  ```

- 执行 `ssh localhost` 测试是否成功授权

### 安装 Hadoop

- 文件 `slave`

  ```shell
  cd /home/bjut/hadoop-2.6.0/etc/hadoop
  vi slaves
  ```

  由于我有 5 台 Slave 节点，所以文件内容为：

   ![slaves](\media\files\2017\07\04\slaves.png)

- 文件 `core-site.xml`

  ```xml
  <property>
      <name>fs.defaultFS</name>
      <value>hdfs://Master:9000</value>
  </property>
  <property>
      <name>hadoop.tmp.dir</name>
      <value>file:/home/bjut/hadoop-2.6.0/tmp</value>
      <description>Abase for other temporary directories.</description>
  </property>
  ```

- 文件 `hdfs-site.xml`

  ```xml
  <property>
      <name>dfs.namenode.secondary.http-address</name>
      <value>Master:50090</value>
  </property>
  <property>
      <name>dfs.namenode.name.dir</name>
      <value>file:/home/bjut/hadoop-2.6.0/tmp/dfs/name</value>
  </property>
  <property>
      <name>dfs.datanode.data.dir</name>
      <value>file:/home/bjut/hadoop-2.6.0/tmp/dfs/data</value>
  </property>
  <property>
      <name>dfs.replication</name>
      <value>5</value>
  </property>
  ```

- 文件 `mapred-site.xml`

  这个文件不存在，首先需要从模板中复制一份：

  `cp mapred-site.xml.template mapred-site.xml`

  然后配置如下：

  ```xml
  <property>
      <name>mapreduce.framework.name</name>
      <value>yarn</value>
  </property>
  ```

- 文件 `yarn-site.xml`

  ```xml
  <property>
      <name>yarn.resourcemanager.hostname</name>
      <value>Master</value>
  </property>
  <property>
      <name>yarn.nodemanager.aux-services</name>
      <value>mapreduce_shuffle</value>
  </property>
  ```

## Slave 配置

### 安装 JDK

同 Master 节点操作一样。

### 修改主机名

- `sudo vi /etc/hostname`

  slave节点分别命名为Slave1、Slave2、...

- `sudo vi /etc/hosts`

  内容同Master节点一样。

### SSH 配置

- `ssh localhost` (为了产生 `.ssh` 文件夹)

- 执行 `scp ~/.ssh/id_rsa.pub bjut@SlaveN:/home/bjut/` 将 Master 节点下生成的 `id_rsa.pub` 文件发送到各个 Slave 节点，然后在各个 Slave 节点上将 ssh 公钥保存到相应位置，执行 `cat ~/id_rsa.pub >> ~/.ssh/authorized_keys`

- 在 Master 节点上执行 `ssh SlaveN` 测试是否授权成功


- 安装 Hadoop

  在 Master 节点上发送配置好的 hadoop 文件夹到各个 slave 节点上。

  ```shell
  scp -r hadoop-2.6.0/ bjut@SlaveN:/home/bjut/
  ```


## 参考资料

- [Hadoop集群安装配置教程](http://www.powerxing.com/install-hadoop-cluster/)