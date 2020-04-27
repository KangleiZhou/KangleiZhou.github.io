---
layout: post
category: Spring Cloud
title: Spring Cloud 全链路追踪实现
tagline: by 城南书客
tags: 
  - Spring Cloud
published: true
---
## 简介
在微服务架构下存在多个服务之间的相互调用，当某个请求变慢或不可用时，我们如何快速定位服务故障点呢？链路追踪的实现就是为了解决这一问题，本文采用Sleuth+Zipkin+RabbitMQ+ES+Kibana实现。

**Spring Cloud Sleuth**

Trace：从客户端请求到系统边界，再到系统边界返回客户端响应。

Span：每一次调用埋入一个调用记录，即为 “Span”，一系列有序的Span构成一个Trace。

![](/assets/images/articles/trace.png)

**Zipkin**

Zipkin 是由Twitter公司开源的一个分布式追踪系统，用于收集服务的定时数据，实现数据的收集、存储、查找和展现。提供了可插拔的数据存储方式：In-Memory、MySql、Cassandra以及Elasticsearch。

**RabbitMQ**

RabbitMQ是一个开源的AMQP实现，服务器端用Erlang语言编写，支持多种客户端，用于在分布式系统中存储转发消息，在易用性、扩展性、高可用性等方面表现不俗。 

**Elasticsearch**

Elasticsearch（ES）是一个基于Lucene构建的开源、分布式、RESTful接口的全文搜索引擎。Elasticsearch还是一个分布式文档数据库，其中每个字段均可被索引，而且每个字段的数据均可被搜索，ES能够横向扩展至数以百计的服务器存储以及处理PB级的数据。可以在极短的时间内存储、搜索和分析大量的数据。

**Kibana**

Kibana可以为 Logstash 和 ElasticSearch 提供友好的日志分析 Web 界面，可以实现汇总、分析和搜索重要数据日志。

## 快速上手
**1、Zipkin服务端**

创建zipkin-server项目（也可到官方网站：https://zipkin.io/下载jar包直接使用）

依赖
```
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<dependency>
  <groupId>io.zipkin.java</groupId>
  <artifactId>zipkin-server</artifactId>
  <version>2.11.8</version>
</dependency>
<dependency>
  <groupId>io.zipkin.java</groupId>
  <artifactId>zipkin-autoconfigure-ui</artifactId>
  <version>2.11.8</version>
</dependency>
<dependency>
  <groupId>io.zipkin.java</groupId>
  <artifactId>zipkin-autoconfigure-collector-rabbitmq</artifactId>
  <version>2.11.8</version>
</dependency>
<dependency>
  <groupId>io.zipkin.java</groupId>
  <artifactId>zipkin-autoconfigure-storage-elasticsearch-http</artifactId>
  <version>2.8.4</version>
</dependency>
```
配置
```
spring:
  application:
    name: zipkin-server
server:
  port: 8033
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8088/eureka/
  instance:
      prefer-ip-address: true
management:
  metrics:
    web:
      server:
        auto-time-requests: false
zipkin:
  collector:
    rabbitmq:
      addresses: 192.168.233.128
      port: 5672
      username: zipkin
      password: zipkin
      virtual-host: vh1
      queue: zipkin
  storage:
    StorageComponent: elasticsearch
    type: elasticsearch
    elasticsearch:
      hosts: 192.168.233.171:9200
      cluster: elasticsearch
      index: zipkin
      index-shards: 5
      index-replicas: 1
```
启动类
```
@SpringBootApplication
@EnableEurekaClient
@EnableZipkinServer
public class ZipkinServerApplication {
  public static void main(String[] args) {
    SpringApplication.run(ZipkinServerApplication.class, args);
  }
}
```
访问 http://localhost:8033/zipkin/

![](/assets/images/articles/zipkin.JPG)

**2、Zipkin客户端**

依赖
```
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
```
配置
```
spring:
  sleuth:
    sampler:
      probability: 1.0
  zipkin:
    sender:
      type: RABBIT
  rabbitmq:
    addresses: 192.168.233.128
    port: 5672
    username: zipkin
    password: zipkin
    virtual-host: vh1
```
**3、测试**

访问zipkin客户端服务，如我本地user-server http://localhost:8061/user/findAll

![](/assets/images/articles/u.JPG)

点“Find Traces”，看一下zipkin服务端 

![](/assets/images/articles/zk.JPG)

访问 http://192.168.233.171:5601 ，看一下Kibana，配置一个index pattern

![](/assets/images/articles/lgs.JPG)

修改默认时间格式

![](/assets/images/articles/lgs2.JPG)

看一下效果

![](/assets/images/articles/lgs3.JPG)

注：之前的图片不清楚，后来重新更新了图片，因此，图片上的时间会晚于文章时间。