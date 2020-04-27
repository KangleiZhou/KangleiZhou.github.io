---
layout: post
category: Spring Cloud
title: Spring Cloud Gateway 使用
tagline: by 城南书客
tags: 
  - Spring Cloud
published: true
---
## 简介
Spring Cloud Gateway是Spring Cloud官方推出的网关框架，网关作为流量入口，在微服务系统中有着十分重要的作用，常用功能包括：鉴权、路由转发、熔断、限流等。

Spring Cloud Gateway是通过Spring WebFlux的HandlerMapping做为底层支持来匹配到转发路由，使用时不要引入SpringMVC，否则初始化时会出错；Spring Cloud Gateway内置了很多Predicates工厂，这些 Predicates 工厂通过不同的 HTTP 请求参数来匹配，多个 Predicates 工厂可以组合使用。

## 快速上手
1、添加依赖
```
<dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```
2、配置
```
server:
  port: 8066
spring:
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true
          lower-case-service-id: true  #设置serviceId小写，默认大写
      routes:
      - id: user-server
        uri: lb://user-server   #lb表示从注册中心获取服务
        predicates:
        - Path=/userapi/**         # 如果请求地址满足/userapi/**,则转发到user-server服务
        filters:
        - StripPrefix=1           # 去除原请求地址中的userapi
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8088/eureka/
```
3、集成Hystrix

依赖
```
<dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```
配置
```
filters:
- name: Hystrix
  args:
    name : default
    fallbackUri: 'forward:/dfallback'
hystrix:
  command:
    default:
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 6000
```
Java端
```
@RestController
public class DHystrixController {
    @RequestMapping("/dfallback")
    public Map<String,String> dfallback(){
        System.out.println("降级了。。。");
        Map<String,String> map = new HashMap<String,String>();
        map.put("rCode","-1");
        map.put("rMsg","出错了");
        return map;
    }
}
```