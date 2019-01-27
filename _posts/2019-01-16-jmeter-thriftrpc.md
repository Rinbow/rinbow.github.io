---
title: Jmeter 压测 thrift rpc 服务
tags: [java, jmeter]
---

#### 一、Jmeter 测试类编写

1. 新建 Java 工程并添加 Jmeter 依赖

   ```xml
   <dependency>
       <groupId>org.apache.jmeter</groupId>
       <artifactId>ApacheJMeter_java</artifactId>
       <version>2.13</version>
       <exclusions>
           <exclusion>
               <artifactId>commons-math3</artifactId>
               <groupId>commons-math3</groupId>
           </exclusion>
           <exclusion>
               <artifactId>commons-pool2</artifactId>
               <groupId>commons-pool2</groupId>
           </exclusion>
       </exclusions>
   </dependency>
   ```

   注意：如果不加 `exclusions`，build 的时候会报如下错误：

   ```
   [ERROR] Failed to execute goal on project rr-rpc-client: Could not resolve dependencies for project com.test:rr-rpc-client:jar:1.0-SNAPSHOT: The following artifacts could not be re
   solved: commons-math3:commons-math3:jar:3.4.1, commons-pool2:commons-pool2:jar:2.3: Could not find artifact commons-math3:commons-math3:jar:3.4.1 -> [Help 1]
   ```

   这个是 Jmeter Maven pom 依赖的问题，参考 [https://stackoverflow.com/questions/35363948/jars-could-not-be-resolved-for-apache-jmter-2-13-with-maven](https://stackoverflow.com/questions/35363948/jars-could-not-be-resolved-for-apache-jmter-2-13-with-maven)。

2. 新建一个测试类并继承 `AbstractJavaSamplerClient`，实现该类的四个方法:

   ```java
   public class ThriftClientTest extends AbstractJavaSamplerClient {
       private IRelatedRecoService.Iface recoService;
       
       /**
        * 设置入参，已设置的参数会显示在jmeter GUI的参数列表中
        *
        * @return
        */
       @Override
       public Arguments getDefaultParameters() {
           Arguments arguments = new Arguments();
           arguments.addArgument("deviceId", "0123");
           arguments.addArgument("reqId", "r_4342314";
           return arguments;
       }
   
       /**
        * 初始化方法，用于初始化性能测试的每个线程，每个线程前都会执行一次
        *
        * @param context
        */
       @Override
       public void setupTest(JavaSamplerContext context) {
           ApplicationContext ctx = new ClassPathXmlApplicationContext("application-thrift.xml");
           recoService = (IRelatedRecoService.Iface) ctx.getBean("relatedRecoService");
       }
   
       /**
        * 性能测试的线程运行体，测试执行主体，从context中获取参数，并调用被测方法，完成与服务器的交互。
        * 该方法是java Sampler实现的重点，执行次数取决于线程数和循环次数
        *
        * @param javaSamplerContext
        * @return
        */
       @Override
       public SampleResult runTest(JavaSamplerContext javaSamplerContext) {
           SampleResult sampleResult = new SampleResult();
   
           RelatedRecoRequest request = new RelatedRecoRequest();
           Map<String, String> map = new HashMap<>();
           map.put("recId", javaSamplerContext.getParameter("recId"));
           map.put("resultNumber", "10");
           map.put("deviceId", javaSamplerContext.getParameter("deviceId"));
           request.setParams(map);
           try {
               sampleResult.sampleStart();
               long start = System.currentTimeMillis();
               recoService.relatedRecommend(request);
               long end = System.currentTimeMillis();
               System.out.println("==============cost:" + (end - start) + "ms");
               sampleResult.setSuccessful(true);
           } catch (TException e) {
               e.printStackTrace();
               sampleResult.setSuccessful(false);
           } finally {
               sampleResult.sampleEnd();
           }
   
           return sampleResult;
       }
   
       /**
        * 测试结束时调用，每个线程执行一次，常用于关闭资源
        *
        * @param context
        */
       @Override
       public void teardownTest(JavaSamplerContext context) {
   
       }
   }
   
   ```

#### 二、使用 Jmeter 进行压测

1. 首先将上面项目用 maven 打包，然后将 jar 文件以及 lib 文件夹拷贝到 apache-jmeter-5.0\lib\ext 目录下。

2. 启动 Jmeter

   - 添加 thread group

   - 在 thread group 下添加 java request，会看到如下界面：

     ![QQ截图20190116194026](\media\files\2019\01\16\QQ截图20190116194026.png)

   - 添加 summary report

   - thread group 中可以设置线程数、循环次数、持续时间等等选项，如下图所示：

     ![QQ截图20190116194351](\media\files\2019\01\16\QQ截图20190116194351.png)

   - 查看压测报告

     ![QQ截图20190116194617](\media\files\2019\01\16\QQ截图20190116194617.png)

---

#### 参考资料

- [Jmeter压测Thrift服务接口](https://sq.163yun.com/blog/article/212992437005803520)

- [Apache Jmeter进阶-RPC服务压测](https://blog.csdn.net/f59130/article/details/74171190)

