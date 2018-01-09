# Log4j，Logback Appender：云上日志解决方案

## 云上开发挑战

越来越多的用户将他们的 Java 应用部署到云上。如何对这些应用产生的海量日志进行采集、处理和分析面临着很多挑战，传统的日志解决方案无法完美满足需求。

## 传统方案（ELK）

+ 搭建一套高可用 ELK 集群。
+ 通过配置 Logstash 采集应用产生的日志。也可以使用第三方提供的 Elastic Search Log Appender 将日志直接发送到Elastic Search上。
+ 通过 Kibana 进行日志分析。


## 传统方案的痛点与挑战

+ 需要自己搭建和维护 ELK 集群。


## 日志服务解决方案

日志服务团队根据多年日志处理经验的沉淀，为 Java 应用程序产生的日志量身定制了一套日志处理解决方案。

### 日志采集

日志服务团队为目前最常用的一些 Java 日志框架 Log4j、Log4j2、Logback 提供了相应的 Appender。通过这些 Appender，用户可以方便地将应用产生的日志通过网络直接写入 Loghub。可参看下面的链接完成相应 Appender 的配置。
+ [aliyun-log-log4j-appender](https://github.com/aliyun/aliyun-log-log4j-appender)
+ [aliyun-log-log4j2-appender](https://github.com/aliyun/aliyun-log-log4j2-appender)
+ [aliyun-log-logback-appender](https://github.com/aliyun/aliyun-log-logback-appender)

这些 Appender 底层使用 [Java Producer Library](https://help.aliyun.com/document_detail/43758.html) 完成数据的写入，数据吞吐量极高。

### 日志存储

Java 日志存储在 Loghub上。 LogHub 覆盖 Kafka 100%功能，并提供完整监控、报警等功能数据，弹性伸缩等（可支持PB/Day规模），使用成本为自建50%以下。

### 日志分析

可以通过 [LogSearch/Analytics](https://help.aliyun.com/document_detail/43772.html) 可以对 Java 应用产生的日志进行实时分析。

写到日志服务中的日志的样式如下：
```
level: ERROR
location: com.aliyun.openservices.log.logback.example.LogbackAppenderExample.main(LogbackAppenderExample.java:18)
message: This is an error message.
thread: main
time: 2018-01-02T03:15+0000
```

#### 1. 统计过去1小时发生Error最多的3个位置

语法示例
```
level is ERROR | select location ,count(*) as count GROUP BY  location  ORDER BY count DESC LIMIT 3
```
使用饼状图进行结果展示
# Log4j，Logback Appender：云上日志解决方案

## 云上开发挑战

越来越多的用户将他们的 Java 应用部署到云上。如何对这些应用产生的海量日志进行采集、处理和分析面临着很多挑战，传统的日志解决方案无法完美满足需求。

## 传统方案（ELK）

+ 搭建一套高可用 ELK 集群。
+ 通过配置 Logstash 采集应用产生的日志。也可以使用第三方提供的 Elastic Search Log Appender 将日志直接发送到Elastic Search上。
+ 通过 Kibana 进行日志分析。


## 传统方案的痛点与挑战

+ 需要自己搭建和维护 ELK 集群。


## 日志服务解决方案

日志服务团队根据多年日志处理经验的沉淀，为 Java 应用程序产生的日志量身定制了一套日志处理解决方案。

### 日志采集

日志服务团队为目前最常用的一些 Java 日志框架 Log4j、Log4j2、Logback 提供了相应的 Appender。通过这些 Appender，用户可以方便地将应用产生的日志通过网络直接写入 Loghub。可参看下面的链接完成相应 Appender 的配置。
+ [aliyun-log-log4j-appender](https://github.com/aliyun/aliyun-log-log4j-appender)
+ [aliyun-log-log4j2-appender](https://github.com/aliyun/aliyun-log-log4j2-appender)
+ [aliyun-log-logback-appender](https://github.com/aliyun/aliyun-log-logback-appender)

这些 Appender 底层使用 [Java Producer Library](https://help.aliyun.com/document_detail/43758.html) 完成数据的写入，数据吞吐量极高。

### 日志存储

Java 日志存储在 Loghub上。 LogHub 覆盖 Kafka 100%功能，并提供完整监控、报警等功能数据，弹性伸缩等（可支持PB/Day规模），使用成本为自建50%以下。

### 日志分析

可以通过 [LogSearch/Analytics](https://help.aliyun.com/document_detail/43772.html) 可以对 Java 应用产生的日志进行实时分析。

写到日志服务中的日志的样式如下：
```
level: ERROR
location: com.aliyun.openservices.log.logback.example.LogbackAppenderExample.main(LogbackAppenderExample.java:18)
message: This is an error message.
thread: main
time: 2018-01-02T03:15+0000
```

#### 1. 统计过去1小时发生Error最多的3个位置

语法示例
```
level is ERROR | select location ,count(*) as count GROUP BY  location  ORDER BY count DESC LIMIT 3
```
使用饼状图进行结果展示
![](/pics/1.png)




