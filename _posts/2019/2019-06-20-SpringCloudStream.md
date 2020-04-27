---
layout: post
category: Spring Cloud
title: Spring Cloud Stream 整合 RabbitMQ
tagline: by 城南书客
tags: 
  - Spring Cloud
published: true
---
## 简介
Spring Cloud Stream 是一个构建消息驱动微服务的框架，应用程序通过 input（相当于 consumer ）、output（相当于 producer ）来与 Spring Cloud Stream 中 Binder 交互，而 Binder 负责与消息中间件交互；因此，我们只需关注如何与 Binder 交互即可，而无需关注与具体消息中间件的交互。

![](/assets/images/articles/springcloudstream.jpg)
## 快速上手
1、添加依赖
```
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-stream-binder-rabbit</artifactId>
  <version>2.1.2.RELEASE</version>
</dependency>
```
2、配置

provider配置（采用动态路由键方式）
```
server:
  port: 7071
spring:
  cloud:
    stream:
      binders:
        pro:
          type: rabbit
          environment:
            spring:
              rabbitmq:
                addresses: localhost
                port: 5672
                username: test
                password: test
                virtual-host: test
      bindings:
        myOutPut:
          destination: myOutPut
          content-type: application/json
          default-binder: test
      rabbit:
        bindings:
          myOutPut:
            producer:
              exchangeType: topic
              routing-key-expression: headers.routeId
```
consumer配置
```
server:
  port: 7072
spring:
  cloud:
    stream:
      rabbit:
        bindings:
          input:
            consumer:
              bindingRoutingKey: routeKey1
              acknowledge-mode: manual
      binders:
              protest:
                type: rabbit
                environment:
                  spring:
                    rabbitmq:
                      addresses: localhost
                      port: 5672
                      username: test
                      password: test
                      virtual-host: test
      bindings:
              input:
                destination: myOutPut
                content-type: application/json
                default-binder: protest
                group: group-cus1
```
3、java端

provider
```
public interface MqMessageSource {//自定义通道
    String MY_OUT_PUT = "myOutPut";
    @Output(MY_OUT_PUT)
    MessageChannel testOutPut();
}
```
```
@EnableBinding(MqMessageSource.class)
public class MessageProviderImpl implements IMessageProvider {
    @Autowired
    @Output(MqMessageSource.MY_OUT_PUT)
    private MessageChannel channel;

    @Override
    public void send(Company company) {
        channel.send(MessageBuilder.withPayload(company).setHeader("routeId", company.getTitle()).build());
    }
}
```
```
@Component
@EnableBinding(Sink.class)
public class MessageListener {
    @StreamListener(Sink.INPUT)
    public void input(Message<Company> message) throws IOException {
        Channel channel = (com.rabbitmq.client.Channel)message.getHeaders().get(AmqpHeaders.CHANNEL);
        Long deliveryTag = (Long) message.getHeaders().get(AmqpHeaders.DELIVERY_TAG);
        channel.basicAck(deliveryTag, false);
        System.err.println(JSON.toJSONString(message.getPayload()));
    }
}
```




