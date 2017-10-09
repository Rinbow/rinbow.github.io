---
layout: post
title: Hadoop 单机模式安装笔记
tags:
  - program
  - hadoop
  - bigdata
---

实验室第N次服务器配置，简单记录以备不时之需。

### 安装 JDK

- `/etc/profile`
  `sudo vi /etc/profile`，设置 `JAVA` 路径，如下图所示：
  ![etcProfile](\media\files\2017\07\03\etcProfile.png)
- `~/.bashrc`
  `sudo vi ~./bashrc`，设置 `JAVA_HOME`，如下图所示：
  ![bashrc](\media\files\2017\07\03\bashrc.png)
- 执行 `java -version` 检查效果。

### SSH 配置

SSH 无密码登录

```shell
ssh localhost #生成.ssh目录
exit
cd ~/.ssh/
ssh-keygen -t rsa #遇到提示直接按回车即可
cat id_rsa.pub >> authorized_keys #加入授权
```

然后就可以执行 `ssh localhost` 测试是否授权成功了。

### 安装 Hadoop

Hadoop 的配置文件位于 `/usr/local/hadoop/etc/hadoop/` 中，单机模式需要修改2个配置文件 **core-site.xml** 和 **hdfs-site.xml** 。Hadoop 的配置文件是 xml 格式，每个配置以声明 `property` 的 `name` 和 `value` 的方式来实现。

- 文件 `core-site.xml`
  ```xml
  <configuration>
      <property>
          <name>hadoop.tmp.dir</name>
          <value>file:/home/bjut/hadoop-2.6.0/tmp</value>
          <description>Abase for other temporary directories.</description>
      </property>
      <property>
          <name>fs.defaultFS</name>
          <value>hdfs://localhost:9000</value>
      </property>
  </configuration>
  ```
- 文件 `hdfs-site.xml`
  ```xml
  <configuration>
      <property>
          <name>dfs.replication</name>
          <value>1</value>
      </property>
      <property>
          <name>dfs.namenode.name.dir</name>
          <value>file:/home/bjut/hadoop-2.6.0/tmp/dfs/name</value>
      </property>
      <property>
          <name>dfs.datanode.data.dir</name>
          <value>file:/home/bjut/hadoop-2.6.0/tmp/dfs/data</value>
      </property>
  </configuration>
  ```
- 执行 namenode 格式化
  ```shell
  bin/hdfs namenode -format
  ```
  如提示 `Exiting with status 0`，则表示格式化成功。
  **注意**：在这一步以及后面启动 Hadoop 时若提示 **Error: JAVA_HOME is not set and could not be found.** 的错误，则需要在文件 `./etc/hadoop/hadoop-env.sh` 中设置 `JAVA_HOME` 变量，即找到 `export JAVA_HOME=${JAVA_HOME}`这一行，改为 `export JAVA_HOME=/home/bjut/jdk1.7/` (就是之前设置的 `JAVA_HOME` 位置)，再重新尝试即可。
- 启动 Hadoop 进程
  ```shell
  sbin/start-dfs.sh
  ```
  启动完成后，可以通过命令 `jps` 来判断是否成功启动，若成功启动则会列出如下进程: `NameNode`、`DataNode` 和 `SecondaryNameNode`。
- 访问 Web 页面
  成功启动后，可以访问 Web 界面 `http://ip_address:50070` 来查看 Hadoop 的信息。
  ![webHadoop](\media\files\2017\07\03\webHadoop.png)
- 配置 PATH 环境变量
  上面的教程中，我们都是先进入到 `/home/bjut/hadoop-2.6.0` 目录中，再执行  `sbin/hadoop`，实际上等同于运行 `/home/bjut/hadoop-2.6.0/sbin/hadoop`。我们可以将 Hadoop 命令的相关目录加入到 PATH 环境变量中，这样就可以直接通过 `start-dfs.sh` 开启 Hadoop，也可以直接通过 `hdfs` 访问 HDFS 的内容，方便平时的操作。
  执行 `vi ~/.bashrc`，在文件中添加 `export PATH=$PATH:/home/bjut/hadoop-2.6.0/bin:/home/bjut/hadoop-2.6.0/sbin`，添加后执行 `source ~/.bashrc` 使设置生效，生效后，在任意目录中，都可以直接使用 `hdfs dfs -ls input` 等命令，而无需使用绝对目录。

## 参考资料

- [Hadoop安装教程_单机/伪分布式配置](http://www.powerxing.com/install-hadoop/)