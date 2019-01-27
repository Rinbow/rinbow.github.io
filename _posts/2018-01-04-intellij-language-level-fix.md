---
title: 解决 IDEA 中 language level 总是自动变成 5 的问题
tags: [java, intellij]
---

IDEA 默认建议开发者写出兼容 1.5 特性的代码，所以每次打开项目或者新建 module 的时候，都会重置 language level。

解决办法就是在 `pom.xml` 文件中把编译版本固定写死，有两种方式：

- ```xml
  <properties>
      <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
      <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
      <java.version>1.7</java.version>
      <maven.compiler.source>1.7</maven.compiler.source>
      <maven.compiler.target>1.7</maven.compiler.target>
      <encoding>UTF-8</encoding>
  </properties>
  ```

- ```xml
  <build>
      <plugins>
          <plugin>
              <groupId>org.apache.maven.plugins</groupId>
              <artifactId>maven-compiler-plugin</artifactId>
              <version>3.7.0</version>
              <configuration>
                  <source>1.7</source>
                  <target>1.7</target>
              </configuration>
          </plugin>
      </plugins>
  </build>
  ```
