# Java日志上云三剑客：Log4J/LogBack/Producer Lib

## 内容

* [日志中心化之路](#日志中心化之路)
   * [中心化四个好处](#中心化四个好处)
* [采集端 (Java系列)](#采集端-java系列)
   * [Appender 介绍与使用](#appender-介绍与使用)
      * [接入 Appender](#接入-appender)
         * [1. maven 工程中引入依赖](#1-maven-工程中引入依赖)
         * [2. 修改配置文件](#2-修改配置文件)
      * [查询与分析](#查询与分析)
         * [开启查询分析](#开启查询分析)
         * [分析实例](#分析实例)
            * [1. 统计过去1小时发生Error最多的3个位置](#1-统计过去1小时发生error最多的3个位置)
            * [2. 统计过去15分钟各种日志级别产生的日志条数](#2-统计过去15分钟各种日志级别产生的日志条数)
            * [3. 日志上下文查询](#3-日志上下文查询)

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

这里以 aliyun-log-log4j-appender 为例。

##### 1. maven 工程中引入依赖
```
<dependency>
    <groupId>com.google.protobuf</groupId>
    <artifactId>protobuf-java</artifactId>
    <version>2.5.0</version>
</dependency>
<dependency>
    <groupId>com.aliyun.openservices</groupId>
    <artifactId>aliyun-log-log4j-appender</artifactId>
    <version>0.1.5</version>
</dependency>
```

##### 2. 修改配置文件
以配置文件 log4j.properties 为例（不存在则在项目根目录创建），配置 Loghub 相关的 appender 与 Logger，例如：
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

#被缓存起来的日志的发送超时时间，如果缓存超时，则会被立即发送，单位是毫秒，默认值为3000，最小值为10，可选参数
log4j.appender.loghub.packageTimeoutInMS=3000
#每个缓存的日志包中包含日志数量的最大值，不能超过 4096，可选参数
log4j.appender.loghub.logsCountPerPackage=4096
#每个缓存的日志包的大小的上限，不能超过 5MB，单位是字节，可选参数
log4j.appender.loghub.logsBytesPerPackage=5242880
#Appender 实例可以使用的内存的上限，单位是字节，默认是 100MB，可选参数
log4j.appender.loghub.memPoolSizeInByte=1048576000
#指定I/O线程池最大线程数量，主要用于发送数据到日志服务，默认是8，可选参数
log4j.appender.loghub.maxIOThreadSizeInPool=8
#指定发送失败时重试的次数，如果超过该值，会把失败信息通过Log4j的LogLog进行记录，默认是3，可选参数
log4j.appender.loghub.retryTimes=3

#指定日志主题
log4j.appender.loghub.topic = [your topic]

#输出到日志服务的时间格式，使用 Java 中 SimpleDateFormat 格式化时间，默认是 ISO8601，可选参数
log4j.appender.loghub.timeFormat=yyyy-MM-dd'T'HH:mmZ
log4j.appender.loghub.timeZone=UTC
```

#### 查询与分析

通过上述方式配置好 appender 后，Java 应用产生的日志会被自动发往日志服务。可以通过 [LogSearch/Analytics](https://help.aliyun.com/document_detail/43772.html) 对这些日志实时查询和分析。

写到日志服务中的日志的样式如下：
```
level: ERROR
location: com.aliyun.openservices.log.logback.example.Log4jAppenderExample.main(Log4jAppenderExample.java:18)
message: This is an error message.
thread: main
time: 2018-01-02T03:15+0000
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

###### 1. 统计过去1小时发生Error最多的3个位置

语法示例
```
level: ERROR | select location ,count(*) as count GROUP BY  location  ORDER BY count DESC LIMIT 3
```
使用饼状图进行结果展示

![1](/pics/1.png)

###### 2. 统计过去15分钟各种日志级别产生的日志条数

语法示例
```
| select level ,count(*) as count GROUP BY level ORDER BY count DESC
```
使用柱状图进行结果展示

![2](/pics/2.png)

###### 3. 日志上下文查询
对于任意一条日志，能够精确还原原始日志文件上下文日志信息。

**点击上下文浏览链接**

![4](/pics/4.png)

**查看选中日志周边的日志信息**

![5](/pics/5.png)

参阅: [上下文查询](https://help.aliyun.com/document_detail/48148.html)

