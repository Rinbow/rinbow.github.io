---
title: Zookeeper 客户端 API 入门实例
tags: [java, zookeeper, distributed]
---

### 服务器安装 Zookeeper

参考[Zookeeper 分布式安装笔记](https://silocean.github.io/2017/07/05/zookeeper-distribution-installation.html) 

### 客户端 API 实例

- 创建 Maven 项目，在 `pom.xml` 文件中添加相关依赖：

  ```xml
  <!-- https://mvnrepository.com/artifact/org.apache.zookeeper/zookeeper -->
  <dependency>
      <groupId>org.apache.zookeeper</groupId>
      <artifactId>zookeeper</artifactId>
      <version>3.3.1</version>
  </dependency>
  ```

- 创建 `ZookeeperClient` 文件，部分 API 实验代码如下：

  ```java
  import org.apache.zookeeper.*;
  
  import java.util.List;
  import java.util.concurrent.CountDownLatch;
  
  public class ZookeeperClient implements Watcher {
  
      private CountDownLatch latch = new CountDownLatch(1);
  
      private ZooKeeper zookeeper;
  
      public static void main(String[] args) {
          try {
              ZookeeperClient client = new ZookeeperClient();
              client.connect();
              client.testAPI();
              client.close();
          } catch (Exception e) {
              e.printStackTrace();
          }
      }
  
      private void testAPI() throws Exception {
          if (zookeeper.exists("/app", false) == null) {
              zookeeper.create("/app", "appdata".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
              System.out.println(new String(zookeeper.getData("/app", false, null)));
          } else {
              zookeeper.setData("/app", "hello appdata".getBytes(), -1);
              System.out.println(new String(zookeeper.getData("/app", false, null)));
              zookeeper.create("/app/child1", "child1Data".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
              zookeeper.create("/app/child2", "child2Data".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
              List<String> children = zookeeper.getChildren("/app", null);
              if (children.size() != 0) {
                  for (String child : children) {
                      System.out.println(child);
                  }
              }
              zookeeper.delete("/app/child1", -1);
              zookeeper.delete("/app/child2", -1);
          }
      }
  
      private void connect() throws Exception {
          zookeeper = new ZooKeeper("192.168.88.128:2181", 3000, this);
          latch.await();
      }
  
      private void close() throws Exception {
          zookeeper.close();
      }
  
      @Override
      public void process(WatchedEvent event) {
          if (event.getState() == Event.KeeperState.SyncConnected) {
              latch.countDown();
              System.out.println("zookeeper connected!");
          }
      }
  }
  ```

---

### 参考资料

- [zookeeper的安装与JavaAPI的使用](https://blog.csdn.net/u010853261/article/details/63684864)

