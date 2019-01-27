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

---

### 参考资料

- [Thrift入门初探--thrift安装及java入门实例](https://www.cnblogs.com/fingerboy/p/6424248.html)