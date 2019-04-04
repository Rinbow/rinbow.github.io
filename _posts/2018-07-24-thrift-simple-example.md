---
title: Thrift 入门实例
tags: [java, thrift, distributed]
---

### 下载安装

- Windows系统的话直接到 Thrift 官网下载 [Thrift compiler for Windows](http://www.apache.org/dyn/closer.cgi?path=/thrift/0.11.0/thrift-0.11.0.exe)，然后将该 exe 文件所在文件夹加入到 `path` 环境变量中；
- Ubuntu系统直接命令行执行  `sudo apt-get install thrift-compiler`;

命令行执行 `thrift`，输出如下表示安装成功：

```
Usage: thrift [options] file

Use thrift -help for a list of options
```

### 实例

假设现在有这样一个需求：客户端需要调用后台服务提供API来计算两个数的和。

1. 创建一个服务 AddService，创建 `AddService.thrift` 文件，其代码如下：

   ```
   namespace java com.demo.service
   
   service AddService {
   	i32 addNumbers(1:i32 para1, 2:i32 para2)
   }
   ```

   这里定义了一个名为 `addNumbers` 的方法，传入两个 `int` 类型整数，返回一个 `int` 类型整数。

2. 终端进入 `AddService.thrift` 所在目录，执行命令 `thrift -r -gen java AddServer.thrift`。此时当前目录下产生了一个 gen-java 目录，这个文件中包含了 AddService 服务的接口定义 AddService.Iface，以及服务调用的底层通信细节，包括客户端的调用逻辑 AddService.Client 以及服务端的处理逻辑 AddService.Processor。 

3. 创建一个 Maven 项目，在 `pom.xml` 文件中添加相关依赖：

   ```xml
   <!-- https://mvnrepository.com/artifact/org.apache.thrift/libthrift -->
   <dependency>
       <groupId>org.apache.thrift</groupId>
       <artifactId>libthrift</artifactId>
       <version>0.11.0</version>
   </dependency>
   ```

4. 然后将 `AddService.java` 文件复制到项目中

5. 创建 `AddServiceImpl` 实现 `AddService.Iface` 接口，代码如下：

   ```java
   package com.demo.service;
   
   import org.apache.thrift.TException;
   
   public class AddServiceImpl implements AddService.Iface {
   
       @Override
       public int addNumbers(int para1, int para2) throws TException {
           return para1 + para2;
       }
   }
   ```

6. 创建服务端代码 `AddServiceServer`，并把 `AddServerImpl` 作为具体的处理逻辑传给 Thrift 服务器

   ```java
   package com.demo.service;
   
   import org.apache.thrift.TProcessor;
   import org.apache.thrift.protocol.TBinaryProtocol;
   import org.apache.thrift.server.TServer;
   import org.apache.thrift.server.TSimpleServer;
   import org.apache.thrift.transport.TServerSocket;
   
   public class AddServiceServer {
       public static void main(String[] args) {
           try {
               System.out.println("server is started...");
               TProcessor tProcessor = new AddService.Processor<AddService.Iface>(new AddServiceImpl());
               TServerSocket serverSocket = new TServerSocket(9898);
               TServer.Args tArts = new TServer.Args(serverSocket);
               tArts.processor(tProcessor);
               tArts.protocolFactory(new TBinaryProtocol.Factory());
               TServer server = new TSimpleServer(tArts);
               server.serve();
           } catch (Exception e) {
               e.printStackTrace();
           }
       }
   }
   ```

7. 启动 `AddServiceServer`，正常输出如下：

   ```
   server is started...
   ```

8. 创建客户端代码 `AddServiceClient`，用 `AddServiceClient.Client` 调用 `addNumbers` 方法

   ```java
   package com.demo.service;
   
   import org.apache.thrift.protocol.TBinaryProtocol;
   import org.apache.thrift.protocol.TProtocol;
   import org.apache.thrift.transport.TSocket;
   import org.apache.thrift.transport.TTransport;
   
   public class AddServiceClient {
       public static void main(String[] args) {
           try {
               System.out.println("client is started...");
               TTransport transport = new TSocket("localhost", 9898, 30000);
               TProtocol protocol = new TBinaryProtocol(transport);
               AddService.Client client = new AddService.Client(protocol);
               transport.open();
               int result = client.addNumbers(1, 2);
               System.out.println("服务器端计算结果为: "+result);
           } catch (Exception e) {
               e.printStackTrace();
           }
       }
   }
   ```

9. 启动 `AddServiceClient`，正常输出如下：

   ```
   client is started...
   服务器端计算结果为: 3
   ```

备注：简单起见，这个例子中把所有代码都放在一起了。实际应用中，客户端和服务端代码可以放在不同机器中。

### 版本/兼容性问题

我们使用的协议可能会随着时间而变更，如果一个已经存在的消息类型不再符合我们的需求，比如打算为消息格式添加一个额外字段，但又想继续使用之前旧的 thrift 消息格式生成的代码，这对 thrift 来说就很简单，而且不需要更改当前使用的任何代码，而仅仅需要满足以下规则：

- 绝不要修改 thrift idl 中已经存在字段的整数编号；
- 任何新添加的字段需要设置成 optional。这就意味着任何使用你的“旧”消息格式的代码序列化的消息可以被你的新代码所解析，因为它们不会丢掉任何 required 的元素。你应该为这些元素设置合理的默认值，这样新的代码就能够正确地与老代码生成的消息交互了。类似地，你的新代码创建的消息也能被你的老代码解析（老的二进制程序在解析的时候只是简单地将新字段忽略）。然而，未知的字段是不会被抛弃的，如果之后消息被序列化，未知的字段会随之一起被序列化——所以，如果消息传到了新代码那里，则新的字段仍然可用；
- 非 required 字段可以删除，只要它的整数编号不会被其他字段重复使用（更好的做法是重命名该字段，比如名字前面可添加 "OBSOLETE_" 以防止其他字段使用它的整数编号；
- 改变默认值通常是没问题的，但需要记着默认值是不会发送到网络对端的。如果你的程序接收到的消息中某一字段没有设置值，你的程序会读取定义在你程序使用的 thrift 协议版本下的默认值，而不会读取发送端协议版本下的默认值；

---

### 参考资料

- [Thrift入门初探--thrift安装及java入门实例](https://www.cnblogs.com/fingerboy/p/6424248.html)
- [Thrift使用指南及语法介绍](http://www.cpper.cn/2016/03/18/develop/Thrift-The-Missing-Guide/)

