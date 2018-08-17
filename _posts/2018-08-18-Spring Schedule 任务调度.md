---
title: Spring Schedule 任务调度
key: 20180818
tags: Spring
---

# 一、简介

Spring官方提供的用于定时任务的框架，通常使用Cron表达式(CronTrigger)来触发定时任务

Spring Scheduler里有两个概念：任务（Task）和运行任务的框架（TaskExecutor/TaskScheduler）

1.TaskExecutor任务执行器，异步执行多个任务

2.TaskScheduler任务调度器，运行定时任务

Spring内置了多种类型的TaskExecutor和TaskScheduler，方便用户根据不同业务场景选择。

<!--more-->

# 二、Cron表达式

1.结构

> 秒 分 时 日 月 周 年

名称|是否必须|允许值|允许特殊字符
:---:|:---:|:---:|:---:
秒（Seconds）|√|0-59（整数）|, - * /
分（Minutes）|√|0-59（整数）|, - * /
时（Hours）|√|0-23（整数）|, - * /
日（DayofMonth）|√|1-31（整数,考虑月的天数）|, - * / ? L C W
月（Month）|√|1-12（整数或JAN-DEC）|, - * /
周（DayofWeek）|√|1-7（整数或SUN-SAT,1=SUN）|, - * / ? L C #
年（Year）|×|1970-2099|, - * /

月份和星期的名称不区分大小写，FRI 和 fri 是一样的

2.特殊字符含义

值|含义|示例
:---:|:---:|:---:
,|列出枚举值|在Minutes域使用5,20表述在5分和20分每分钟触发一次事件
-|列出范围|在Minutes域使用5-20表示从5分到20分每分钟触发一次事件
*|列出任意值|在Minutes域使用*表示每分钟都触发一次事件
/|列出起始时间|x/y表达一个等步长序列，x为起始值，y为增量步长值。在Minutes域使用5/20表示5，25，45分钟分别触发一次事件，*/y同于0/y
?|点位符|想每月的20日，不管20日是星期几，都触发调度，写法： 13 13 15 20 * ?，其中最后一位只能用 ? ，而不能使用 * ，如果使用 * 表示不管星期几都会触发，但实际上并不是这样，日和周会相互影响
L|最后|在DayofWeek域使用5L表示在最后一个星期四触发
W|有效工作日（周一到周五）|在 DayofMonth使用5W，如果5日是星期六，则将在最近的工作日：星期五，即4日触发。如果5日是星期天，则在6日(周一)触发；如果5日在星期一到星期五中的一天，则就在5日触发。另外一点，W的最近寻找不会跨过月份
\#|确定每月第几个星期几|4#2表示某月的第二个星期三

LW连用表示在某个月最后一个工作日，即最后一个星期五

3.常用示例：

表达式|含义
:---:|:---:
0 15 10 * * ? *|每天10点15分触发  
0 15 10 * * ? 2017|2017年每天10点15分触发
0 * 14 * * ?|每天下午的 2点到2点59分每分触发
0 0/5 14 * * ?|每天下午的 2点到2点59分(整点开始，每隔5分触发)
0 0/5 14,18 * * ?|每天下午的 2点到2点59分、18点到18点59分(整点开始，每隔5分触发)
0 0-5 14 * * ?|每天下午的 2点到2点05分每分触发
0 15 10 ? * 6L|每月最后一周的星期五的10点15分触发
0 15 10 ? * 6#3|每月的第三周的星期五开始触发

可以使用[Cron Maker](http://www.cronmaker.com/)在线生成cron表达式

# 三、Spring Schedule使用

## 单个任务

1.添加引用，Spring Schedule在context包里面，引入context会自动引入其他依赖

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.0.8.RELEASE</version>
</dependency>
```

2.添加Spring Schedule配置

```xml
<!--开启定时任务注解-->
<task:annotation-driven/>

<!-- 配置taskScheduler，默认ThreadPoolTaskScheduler -->
<bean class="org.springframework.scheduling.concurrent.ThreadPoolTaskScheduler">
    <!-- 开启10个线程执行 -->
    <property name="poolSize" value="10"/>
</bean>
```

3.添加定时任务

```java
//一秒执行一次
@Scheduled(cron="*/1 * * * * ?")
public static void kk(){
    Thread thread = Thread.currentThread();
    System.out.println("Hello World!"+thread.getName()+";"+thread.getId());
}
```

![img](/myres/20180818/20180817011938.png)

## 多个任务

1.再添加个带延迟的任务

```java
//一秒执行一次
@Scheduled(cron="*/1 * * * * ?")
public static void kk2() throws InterruptedException {
    Thread thread = Thread.currentThread();
    System.out.println("Hello World22222!"+thread.getName()+";"+thread.getId());
    Thread.sleep(1000000);
}
```

![img](/myres/20180818/20180817012332.png)

可以看到执行到第二个延迟任务时第一个任务一直处于等待状态，这是因为默认只有一个线程在执行。

2.添加TaskScheduler配置

```xml
<bean class="org.springframework.scheduling.concurrent.ThreadPoolTaskScheduler">
    <property name="poolSize" value="10"/>
</bean>
```

![img](/myres/20180818/20180817012901.png)

# [Demo地址](https://github.com/A175A174/Demo/tree/master/springschedule)

---