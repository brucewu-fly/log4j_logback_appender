# Java日志上云三剑客：Log4J/LogBack/Producer Lib

## 内容

* [日志中心化之路](#日志中心化之路)
   * [中心化四个好处](#中心化四个好处)
* [采集端 (Java系列)](#采集端-java系列)
   * [Appender 介绍与使用](#appender-介绍与使用)
      * [接入 Appender](#接入-appender)
      * [查询与分析](#查询与分析)
         * [开启查询分析](#开启查询分析)
         * [分析实例](#分析实例)
            * [1. 统计过去1小时发生Error最多的3个位置](#1-统计过去1小时发生error最多的3个位置)
            * [2. 统计过去15分钟各种日志级别产生的日志条数](#2-统计过去15分钟各种日志级别产生的日志条数)
            * [3. 日志上下文查询](#3-日志上下文查询)
            * [4. 统计过去1小时，登录次数最多的三个用户](#4-统计过去1小时登录次数最多的三个用户)
            * [5. 统计过去15分钟，每个用户的付款总额](#5-统计过去15分钟每个用户的付款总额)

## 日志中心化之路

近几年来，无状态编程、容器、Serverless 编程方式的诞生极大提升了软件交付与部署的效率。在架构的演化过程中，可以看到两个变化：

![中心化之路1](/pics/中心化之路1.png)

* 应用架构开始从单体系统逐步转变微服务，其中的业务逻辑随之而来就会变成微服务之间调用与请求。
* 资源角度来看，传统服务器这个物理单位也逐渐淡化，变成了看不见摸不到的虚拟资源模式。

从以上两个变化可以看到这种弹性、标准化架构背后，原先运维与诊断的需求也变得越来越复杂。在10年前我们可以快速登陆到服务器上捞取日志，Attach进程的模式已再也不存在，面对我们的更多是一个标准化的“黑盒”。

![中心化之路2](/pics/中心化之路2.png)

​	为了应对这种变化趋势，诞生一系列面向DevOps诊断与分析的工具。例如集中式监控、集中式日志系统、以及SaaS化的各种部署、监控等服务。

​	日志中心化解决的是以上这个问题，既应用产生日志后实时（或准实时）传输到中心化的节点服务器，例如Syslog，Kafka，ELK，Hbase进行集中式存储是一些常见的模式。

### 中心化四个好处

* 使用方便：在无状态应用中最麻烦的要属Grep日志了，集中式存储只要运行一个Search命令就能够替代原先漫长的过程
* 存储与计算分离：定制机器硬件时无需为日志存储考虑空间大小
* 更低成本：集中式日志存储可以削峰填谷，预留更高水位
* 安全：当发生黑客入侵以及灾难时，关键数据留作取证

![中心化四个好处](/pics/中心化四个好处.png)

## 采集端 (Java系列)

日志服务提供[30+数据采集方式](https://help.aliyun.com/document_detail/28981.html)，针对服务器、移动端、嵌入式设备及各种开发语言都提供完整的接入方案。对Java 开发者而言，没有什么比熟悉的日志框架 Log4j、Log4j2、Logback Appender更好使的了。

对Java应用而言，目前有两种主流的日志采集方案：

* Java程序将日志落盘，通过Logtail进行实时采集
* Java程序直接配置日志服务提供的Appender，当程序运行时，实时将日志发完服务端

两者的差别如下：

|        | 日志落盘+Logtai采集      | Appender 直接发送     |
| ------ | ------------------ | ----------------- |
| 实时性    | 日志落文件，通过Logtail采集  | 直接发送              |
| 吞吐量    | 大                  | 大                 |
| 断点续传   | 支持，取决于Logtail 配置大小 | 支持，取决于内存大小        |
| 关心应用位置 | 需要，配置采集机器组时        | 不需要，主动发送          |
| 本地日志   | 支持                 | 支持                |
| 关闭采集   | Logtail移除配置        | 修改Appender配置，重启应用 |

通过Appender可以在不改任何代码情况下，通过Config就能够非常容易完成日志实时采集工作，日志服务提供的Java系列Appender有如下几大优势：

* 无需修改程序，修改配置即生效
* 异步化 + 断点续传：IO不影响主线程，支持一定网络和服务容错
* 高并发设计：满足海量日志写入需求
* 支持上下文查询功能：写入支持支持在服务端还原原始进程中精确上下文（前后N条日志）

### Appender 介绍与使用

目前提供Appender如下，底层均使用 [aliyun-log-producer-java](https://github.com/aliyun/aliyun-log-producer-java) 完成数据的写入。

+ [aliyun-log-log4j-appender](https://github.com/aliyun/aliyun-log-log4j-appender)
+ [aliyun-log-log4j2-appender](https://github.com/aliyun/aliyun-log-log4j2-appender)
+ [aliyun-log-logback-appender](https://github.com/aliyun/aliyun-log-logback-appender)

#### 接入 Appender

可参考 [aliyun-log-log4j-appender](https://github.com/aliyun/aliyun-log-log4j-appender) 的**配置步骤**部分接入 appender。

配置文件`log4j.properties`的内容如下：
```
log4j.rootLogger=WARN,loghub

log4j.appender.loghub=com.aliyun.openservices.log.log4j.LoghubAppender

#日志服务的project名，必选参数
log4j.appender.loghub.projectName=[your project]
#日志服务的logstore名，必选参数
log4j.appender.loghub.logstore=[your logstore]
#日志服务的http地址，必选参数
log4j.appender.loghub.endpoint=[your project endpoint]
#用户身份标识，必选参数
log4j.appender.loghub.accessKeyId=[your accesskey id]
log4j.appender.loghub.accessKey=[your accesskey]
```

#### 查询与分析

通过上述方式配置好 appender 后，Java 应用产生的日志会被自动发往日志服务。可以通过 [LogSearch/Analytics](https://help.aliyun.com/document_detail/43772.html) 对这些日志实时查询和分析。本文提供的样例的日志格式如下：

记录用户登录行为的日志
```
level:  INFO  
location:  com.aliyun.log4jappendertest.Log4jAppenderBizDemo.login(Log4jAppenderBizDemo.java:38)
message:  User login successfully. requestID=id4 userID=user8  
thread:  main  
time:  2018-01-26T15:31+0000  
```
记录用户购买行为的日志
```
level:  INFO  
location:  com.aliyun.log4jappendertest.Log4jAppenderBizDemo.order(Log4jAppenderBizDemo.java:46)
message:  Place an order successfully. requestID=id44 userID=user8 itemID=item3 amount=9  
thread:  main  
time:  2018-01-26T15:31+0000 
```

##### 开启查询分析
若要对数据进行查询和分析，需要首先开启查询分析功能。开启步骤如下：

+ 登录 [日志服务管理控制台](https://sls.console.aliyun.com/#/)。
+ 选择目标项目，单击项目名称或者单击右侧的 **管理**。
+ 选择目标日志库并单击日志索引列下的 **查询**。
+ 单击右上角的 **设置查询分析 > 设置**。
+ 进入设置菜单，为下列字段开启查询。

![3](/pics/3.png)

##### 分析实例

以下视频链接包含了下述5个分析实例。

[视频演示](http://wubo-test-bucket.oss-cn-beijing.aliyuncs.com/video/%E6%97%A5%E5%BF%97%E4%B8%8A%E4%BA%91%E5%AE%A3%E4%BC%A0%E7%89%87.mp4)

###### 1. 统计过去1小时发生Error最多的3个位置

语法示例
```
level: ERROR | select location ,count(*) as count GROUP BY  location  ORDER BY count DESC LIMIT 3
```

###### 2. 统计过去15分钟各种日志级别产生的日志条数

语法示例
```
| select level ,count(*) as count GROUP BY level ORDER BY count DESC
```

###### 3. 日志上下文查询
对于任意一条日志，能够精确还原原始日志文件上下文日志信息。

参阅: [上下文查询](https://help.aliyun.com/document_detail/48148.html)

###### 4. 统计过去1小时，登录次数最多的三个用户

语法示例
```
login | SELECT regexp_extract(message, 'userID=(?<userID>[a-zA-Z\d]+)', 1) AS userID, count(*) as count GROUP BY userID ORDER BY count DESC LIMIT 3
```

###### 5. 统计过去15分钟，每个用户的付款总额

语法示例
```
order | SELECT regexp_extract(message, 'userID=(?<userID>[a-zA-Z\d]+)', 1) AS userID, sum(cast(regexp_extract(message, 'amount=(?<amount>[a-zA-Z\d]+)', 1) AS double)) AS amount GROUP BY userID
```
