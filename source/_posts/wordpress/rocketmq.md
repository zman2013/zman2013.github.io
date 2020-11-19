---
title: RocketMQ，队列选型
tags:
  - ActiveMQ
  - RabbitMQ
  - RocketMQ
  - 队列选型
id: '143'
categories:
  - - MQ
date: 2016-01-17 21:05:44
---

_介绍RocketMQ和队列选型方法_ **[RocketMQ](https://github.com/alibaba/RocketMQ)** RocketMQ是国内比较成功的开源消息队列，由阿里的团队贡献，不少互联网公司使用。 性能好，Broker收发速率都是万级别水平。 水平扩展很方便，Broker不够用，直接加机器。 已消费的消息可以再次消费。 不支持任何消息协议，如：AMQP,STOMP,MQTT。 开源客户端只有Java版（C版有开源，但未完成，无法使用）。 没有监控机制。 **多语言支持** RocketMQ只有Java客户端，C是一个未成品。 RocketMQ客户端的逻辑很重，既要从nameserver获取broker信息，又包含负载均衡逻辑，还会向broker同步offset信息。 经评估，用JNI封装一个C客户端成本更低。开发完成一定要进行长时间大数据性能测试，观察cpu、memory使用情况。 Php使用php-java-bridge做为agent来间接实现，太不舒服了。 最初选型是看上了RocketMQ已消费消息可查的特性，但目前综合来看，RocketMQ并不是一个好的选择。已消费消息可查是否有必要作为mq的特性呢？ **主从切换** 主从切换主要是对于有序消费有用。 具体实现步骤：(涉及nameserver与broker逻辑改动）

1.  Master与Slave断开连接后，Slave会在时间窗口(30s)内尝试与Master重连，超时后，Slave向NameServer申请升级为Master。
2.  如果此时NameServer也与Master没有连接，则授权Slave升级为Master。
3.  Slave收到确认信息后，停止申请请求。停止ReputMessageService，启动ScheduleMessageService。
4.  Slave修改自身brokerId为0，修改自身标志位ASYNC\_SLAVE。
5.  Slave向NameServer注册为Master。
6.  NameServer新增逻辑支持每个BrokerName有且仅有一个Master(保证old Master不能重新注册)。

**队列选型** 队列选型可以从以下几个指标出发：

1.  支持什么语言？
2.  是否有监控？
3.  HA如何？
4.  是否支持持久化？
5.  是否支持有序消费？
6.  是否支持延迟消息？
7.  开发团队是否可以理解并修改源码？
8.  社区是否活跃，响应是否迅速？
9.  性能

**Q：**支持什么语言的客户端？ **A：**我觉得这是最优先考虑的，如果没有你期望的语言，可以直接放弃。因为开发一个新语言版本，首先要求对client与broker的通信协议有清楚的了解，另外从编码完成到成熟稳定一般是一个长期过程。（其实也可以通过jni或者agent来间接封装client，事实我们就是这么做的，但终究失去了美感）

RocketMQ只有Java版本。

ActiveMQ和RabbitMQ支持所有主流变成语言，由于都支持AMQP、STOMP，于是基本覆盖了全部语言。

**Q：**是否有监控？ **A：**没监控的直接pass，否则mq响应慢，甚至挂了都不知道。直观了解各queue的状态信息也能为业务优化带来很大便利。

RabbitMQ监控平台最友好，ActiveMQ次之，RocketMQ的第三方监控平台可以用。

**Q：**高可用性？是否支持failover？failover过程中是否会丢消息？ **A：**高可用性差，MQ就会变成系统的单点，进而引起上下游系统的容错性、稳定性问题。

RabbitMQ集群化最方便，而且能动态配置，集群中所有节点拥有相同的元数据。所有client都与master进行通信，master将update同步给所有slave。slave过多会占用较多网络带宽。([LINK](http://www.rabbitmq.com/ha.html))

ActiveMQ有三种集群化方式，共享文件系统、数据库方式、备份方式。备份方式不支持延迟消息。主从切换过程中非持久化消息会丢失。([LINK](http://activemq.apache.org/masterslave.html))

RocketMQ可以配置多Master，且每个Master可以配置一个Slave。如果一个Master挂掉，此Master对应Slave不会自动升级为Master但会继续提供读服务，写只能向其他Master写。

以上三种在failover过程中，都可能出现重复消息。

**Q：**持久化、有序性、延迟消息？ **A：**Rabbit、Active、RocketMQ都支持消息持久化和有序消费。RabbitMQ不支持延迟消息，Active和RocketMQ支持延迟消息。 **Q：**自己的开发团队能够掌握源码并进行功能开发？社区活跃度？ **A：**这个比较关键，随时都可能遇到一个罕见的bug或者业务需要定制的功能，如果开发团队不能hold住，那就真没办法了。

ActiveMQ和RabbitMQ社区很活跃，问题也能及时得到响应，RabbitMQ用Erlang开发的。RocketMQ没有社区。。

**Q：**单机broker性能？ **A：**RocketMQ > RabbitMQ > ActiveMQ。对于一般业务，性能不会是瓶颈，也不是选型的核心指标，没必要花费较多时间在上面。单机4000、5000的qps，足够了，不够建集群。 **Q：**有没有MQ可以避免消息重复？ **A：**N/A，消费端做好幂等吧~ **踩过的坑** **1.** 低版本的ActiveMQ存在hang住情况，broker拒绝接受producer的消息，consumer接受不到任何消息。

由非持久化消息的临时存储文件泄露导致无法自动删除引起。

5.9.0以后的版本已经修复了这bug。

**2.** RabbitMQ消息堆积，超出内存限制引起数据写入硬盘会导致producer生产、consumer消费极慢。

从硬盘读取消息，然后在五个内部队列间切换消息状态，导致消息消费极慢，进而流控消息写入的速度。

只要消息堆积过多，引起写入硬盘就有此问题。

  _RocketMQ的代码跟他妈狗屎一样。_