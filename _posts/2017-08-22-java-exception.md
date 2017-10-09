---
layout: post
title: Java 中的异常处理笔记
tags:
  - program
  - java
---

Java 异常机制可以使程序中异常处理代码和正常业务代码分离，保证程序代码更加优雅，并提高程序健壮性。在有效使用异常的情况下，异常能清晰的回答 what, where, why 这 3 个问题：异常类型回答了“什么”被抛出，异常堆栈跟踪回答了“在哪“抛出，异常信息回答了“为什么“会抛出。

### 概念

Java 异常是 Java 提供的用于处理程序中错误的一种机制。

所谓错误是指在程序运行的过程中发生的一些异常事件（如：除 0 溢出，数组下标越界，所要读取的文件不存在等）。

设计良好的程序应该在异常发生时提供处理这些错误的方法，使得程序不会应为异常的发生而阻断或产生不可预见的结果。

Java 程序的执行过程中如出现异常事件，可以生成一个异常类对象，该异常对象封装了异常事件的信息并提交给 Java 运行时系统，这个过程称为抛出（throw）异常。

当 Java 运行时系统接受到异常对象时，会寻找能处理这一异常的代码并把当前异常对象交给其处理，这一过程称为捕获（catch）异常。

### 分类

 Java 异常类层次结构图如下：

![java_exception_hierarchy](\media\files\2017\08\22\java_exception_hierarchy.jpg)

- **Throwable**: 有两个重要的子类：Exception 和 Error，二者都是 Java 异常处理的重要子类，各自都包含大量子类。
- **Error**: 是程序无法处理的错误，通常发生于虚拟机自身，表示运行应用程序中较严重问题。例如，Java虚拟机运行错误（Virtual MachineError），当 JVM 不再有继续执行操作所需的内存资源时，将出现 OutOfMemoryError。这些异常发生时，Java 虚拟机（JVM）一般会选择线程终止。**不需要我们进行捕获处理。**
- **Exception**: 是程序本身可以处理的异常。
  - 必须处理的异常：这种异常的特点是 Java 编译器会检查它，也就是说，当程序中可能出现这类异常，要么用 try-catch 语句捕获它，要么用 throws 子句声明抛出它，否则编译不会通过。
  - 可处理可不处理的异常（RuntimeException）：运行时异常的特点是 Java 编译器不会检查它，也就是说，当程序中可能出现这类异常，即使没有用 try-catch 语句捕获它，也没有用 throws 子句声明抛出它，也会编译通过。

### finally 语句

- **try 块:** 用于捕获异常。其后可接零个或多个 catch 块，如果没有 catch 块，则必须跟一个 finally 块。
- **catch 块:** 用于处理 try 捕获到的异常。
- **finally 块:** 无论是否捕获或处理异常，finally 块里的语句都会被执行。当在 try 块或 catch 块中遇到 return 语句时，finally 语句块将在方法返回之前被执行。在以下 4 种特殊情况下，finally 块不会被执行：
  1. 在 finally 语句块中发生了异常。
  2. 在前面的代码中用了 System.exit() 退出程序。
  3. 程序所在的线程死亡。
  4. 关闭 CPU。

### throws 语句

注意：重写方法需要抛出与原方法所抛出异常类型一致的异常或者不抛出异常。

## 参考资料

- [深入理解java异常处理机制](http://blog.csdn.net/hguisu/article/details/6155636)
- [Java异常(一) Java异常简介及其架构](http://www.cnblogs.com/skywang12345/p/3544168.html)

