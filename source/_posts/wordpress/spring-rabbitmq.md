---
title: Spring RabbitMQ
tags:
  - MQ
  - RabbitMQ
  - Spring
id: '219'
categories:
  - - MQ
  - - Spring
date: 2016-05-08 20:57:06
---

_spring项目使用rabbitmq_ **1\. 项目依赖**

<!-- rabbitmq -->
<dependency>
    <groupId>org.springframework.amqp</groupId>
    <artifactId>spring-amqp</artifactId>
    <version>1.5.5.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.amqp</groupId>
    <artifactId>spring-rabbit</artifactId>
    <version>1.5.5.RELEASE</version>
</dependency>

 **2. 配置文件**

<?xml version="1.0" encoding="UTF-8" ?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:rabbit="http://www.springframework.org/schema/rabbit"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/rabbit
        http://www.springframework.org/schema/rabbit/spring-rabbit.xsd">

    <!-- 声明rabbitmq连接 -->
    <rabbit:connection-factory id="rabbitmqConnectionFactory"
                               host="${rabbitmq.host}"
                               virtual-host="${rabbitmq.vhost}"
                               username="${rabbitmq.username}"
                               password="${rabbitmq.password}"/>
    <!-- 创建rabbitmq admin，用来管理queue、exchanger、binding -->
    <rabbit:admin connection-factory="rabbitmqConnectionFactory"/>

    <!-- 声明队列 -->
    <rabbit:queue id="queue-bean-1" durable="true" name="${rabbitmq.queuename1}"/>
    <rabbit:queue id="queue-bean-2" durable="true" name="${rabbitmq.queuename2}"/>

    <!-- 声明exchange，这里使用的direct-exchange，与rabbitmq服务器的exchange类型对应 -->
    <rabbit:direct-exchange name="${rabbitmq.exchange.name}">
        <rabbit:bindings>
            <rabbit:binding queue="queue-bean-1" key="${rabbitmq.route.key1}"></rabbit:binding>
            <rabbit:binding queue="queue-bean-2" key="${rabbitmq.route.key2}"></rabbit:binding>
        </rabbit:bindings>
    </rabbit:direct-exchange>

    <!-- 声明消息消费者（监听者） -->
    <bean id="rabbitmqListener1" class="com.zmannotes.spring.mq.listener.RabbitMQListener1"/>
    <bean id="rabbitmqListener2" class="com.zmannotes.spring.mq.listener.RabbitMQListener2"/>

    <!-- 声明消费者容器，并将queue和listener关联，此处指定自动返回ack -->
    <rabbit:listener-container
            connection-factory="rabbitmqConnectionFactory" acknowledge="auto">
        <rabbit:listener queues="queue-bean-1" ref="rabbitmqListener1"
                         method="onMessage"/>
        <rabbit:listener queues="queue-bean-2" ref="rabbitmqListener2"
                         method="onMessage"/>
    </rabbit:listener-container>

    <!-- 消息生产者 -->
    <rabbit:template id="rabbitmqTemplate" connection-factory="rabbitmqConnectionFactory"
        reply-timeout="2000" routing-key="remoting.binding"
        exchange="rabbitmqExchange"/>

</beans>

**3\. 实现XXXListener**

import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.core.ChannelAwareMessageListener;

public class RabbitMQListener implements ChannelAwareMessageListener {
    
    public void onMessage(Message message, Channel channel) throws Exception {
        String msg = new String(message.getBody());
        System.out.println( msg );
    }
}

**4\. 发送消息**

void send(Message message) throws AmqpException;

void send(String routingKey, Message message) throws AmqpException;

void send(String exchange, String routingKey, Message message) throws AmqpException;

**LINK** [官方文档 1](http://docs.spring.io/spring-amqp/reference/html/_introduction.html#quick-tour) [官方文档 2](http://docs.spring.io/spring-amqp/reference/html/_reference.html)