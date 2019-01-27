---
title: Spring 中使用 Quartz 入门实例
tags: [java, spring]
---

### 什么是 Quartz

可以理解为它是一个任务调度框架，当我们碰到时间任务调度的需求的时候，比如每天凌晨生成报表，每分钟同步一下远程文件等等，使用 Quartz 可以很方便地完成这些任务。

### Quartz 中的主要概念

主要有三个核心概念：任务、触发器和调度器。

三者关系简单来说就是，调度器负责调度各个任务，到了某个时刻或者过了一定时间，触发器触动了，特定任务便启动执行。概念相对应的类和接口有：

1. JobDetail：描述具体任务。
2. Trigger：描述任务执行的时间触发规则。有 SimpleTrigger 和 CronTrigger 两个子类代表两种方式，一种是每隔多少分钟小时执行，则用 SimpleTrigger；另一种是日历相关的重复时间间隔，如每天凌晨，每周星期一运行的话，通过 Cron 表达式便可定义出复杂的调度方案（类似 Linux 下的 Crontab）。
3. Scheduler：代表一个 Quartz 的独立运行容器，Trigger 和 JobDetail 要注册到 Scheduler 中才会生效，也就是让调度器知道有哪些触发器和任务，才能进行按规则进行调度任务。 

### 简单使用

- 添加 Maven 依赖

  ```xml
  <dependency>
      <groupId>org.quartz-scheduler</groupId>
      <artifactId>quartz</artifactId>
      <version>2.3.0</version>
  </dependency>
  ```

  我用的 IDEA 生成的 Spring 项目，自带 Spring 相关库，所以只导入了 quartz 的依赖库。

- 添加定时业务逻辑类

  ```java
  package com.test.task;
  
  import java.util.Date;
  
  /**
   * 业务逻辑处理类
   */
  public class Task {
  
      public void doTask(){
          System.out.println("execute task..." + new Date());
      }
  }
  ```

- Spring 配置文件

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
  
      <!--配置job类-->
      <bean id="taskT" class="com.test.task.Task"/>
  
      <!--调度业务-->
      <bean id="jobDetail" class="org.springframework.scheduling.quartz.MethodInvokingJobDetailFactoryBean">
          <!--要执行的job-->
          <property name="targetObject" ref="taskT"/>
          <!--要执行的方法-->
          <property name="targetMethod" value="doTask"/>
      </bean>
  
      <!--调度触发器-->
      <bean id="trigger" class="org.springframework.scheduling.quartz.CronTriggerFactoryBean">
          <!--jobDetail-->
          <property name="jobDetail" ref="jobDetail"/>
          <!-- Cron表达式，每隔2秒执行1次 -->
          <property name="cronExpression" value="0/2 * * * * ?"/>
      </bean>
  
      <!--调度器-->
      <bean id="scheduler" class="org.springframework.scheduling.quartz.SchedulerFactoryBean">
          <property name="triggers">
              <list>
                  <ref bean="trigger"/>
              </list>
          </property>
      </bean>
  </beans>
  ```

- main 函数里加载该配置文件就可以执行定时任务了。

### Cron 表达式

Cron 表达式定义时间规则，由 6 或 7 个空格分隔的时间字段组成：秒 分钟 小时 日期 月份 星期 年（可选）。

| 字段 |     允许值      | 允许的特殊字符  |
| :--: | :-------------: | :-------------: |
|  秒  |      0-59       |     , - * /     |
|  分  |      0-59       |     , - * /     |
|  时  |      0-23       |     , - * /     |
|  日  |      1-31       | , - * ? / L W C |
|  月  | 1-12 或 JAN-DEC |     , - * /     |
|  周  | 1-7 或 SUN-SAT  | , - * ? / L C # |
|  年  |    1970-2099    |     , - * /     |

0/5 * * * * ?：每 5 秒执行一次。

"\*" 字符被用来指定所有的值。如："\*" 在分钟的字段域里表示“每分钟”。 

“?” 字符只在日期域和星期域中使用。它被用来指定“非明确的值”。当你需要通过在这两个域中的一个来指定一些东西的时候，它是有用的。看下面的例子你就会明白。 
月份中的日期和星期中的日期这两个元素时互斥的一起应该通过设置一个问号来表明不想设置那个字段。

“-” 字符被用来指定一个范围。如：“10-12” 在小时域意味着 “10点、11点、12点”。

“,” 字符被用来指定另外的值。如：“MON, WED, FRI” 在星期域里表示”星期一、星期三、星期五”。

“/” 字符用于指定增量。如：“0/15” 在秒域意思是每分钟的 0，15，30 和 45 秒。“5/15” 在分钟域表示每小时的 5，20，35 和 50。 符号 “\*” 在 “/” 前面（如：\*/10）等价于0在 “/” 前面（如：0/10）。记住一条本质：表达式的每个数值域都是一个有最大值和最小值的集合，如： 秒域和分钟域的集合是 0-59，日期域是 1-31，月份域是 1-12。字符 “/” 可以帮助你在每个字符域中取相应的数值。如：“7/6” 在月份域的时候只 有当7月的时候才会触发，并不是表示每个6月。

L是 ”last“ 的省略写法可以表示 day-of-month 和 day-of-week 域，但在两个字段中的意思不同，例如day-of- month 域中表示一个月的最后一天。如果在day-of-week域表示 “7” 或者 “SAT”，如果在 day-of-week 域中前面加上数字，它表示 一个月的最后几天，例如 ”6L“ 就表示一个月的最后一个星期五。

字符 “W” 只允许日期域出现。这个字符用于指定日期的最近工作日。例如：如果你在日期域中写 “15W”，表示：这个月 15 号最近的工作日。所以，如果 15 号是周六，则任务会在 14 号触发。如果 15 号是周日，则任务会在周一也就是 16 号触发。如果 是在日期域填写 “1W” 即使 1 号是周六，那么任务也只会在下周一，也就是 3 号触发，“W” 字符指定的最近工作日是不能够跨月份的。字符 “W” 只能配合一个 单独的数值使用，不能够是一个数字段，如：“1-15W” 是错误的。

“L” 和 “W” 可以在日期域中联合使用，LW 表示这个月最后一周的工作日。

字符 “#” 只允许在星期域中出现。这个字符用于指定本月的某某天。例如：“6#3” 表示本月第三周的星期五（6 表示星期五，3 表示第三周）。“2#1” 表示本月第一周的星期一。“4#5” 表示第五周的星期三。

字符 “C” 允许在日期域和星期域出现。这个字符依靠一个指定的“日历”。也就是说这个表达式的值依赖于相关的“日历”的计算结果，如果没有“日历”关联，则等价于所有包含的“日历”。如：日期域是 “5C” 表示关联“日历”中第一天，或者这个月开始的第一天的后 5 天。星期域是 “1C” 表示关联“日历”中第一天，或者星期的第一天的后 1 天，也就是周日的后一天（周一）。

---

### 参考资料

- [Quartz-Spring集成Quartz通过XML配置的方式](https://blog.csdn.net/yangshangwei/article/details/78505730?locationNum=6&fps=1)

- [Spring Quartz定时器 配置文件详解](https://www.cnblogs.com/henuyuxiang/p/4152805.html)

- [Spring+Quartz配置定时任务](https://www.cnblogs.com/jianzhi/p/3384436.html)

