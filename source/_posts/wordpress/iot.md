---
title: AWS IoT
tags:
  - iot
id: '285'
categories:
  - - IoT
date: 2018-02-13 14:54:15
---

`去年调研了所有的IoT平台，最后总结只有两家，分别是：AWS IoT和其他IoT。本文从AWS IoT Document中摘取核心功能和特点进行说明。`

## 1 IoT是什么？

IOT能够提供在设备与云端进行安全的双向通信的能力，可以进行数据的远程采集、分析、存储，并可以进行设备的远程控制。

## 2 功能组件

[![aws_iot](./aws_iot.png)](./aws_iot.png)
[![aws_iot_data_services](./aws_iot_data_services.png)](./aws_iot_data_services.png) 
下图是阿里的，基本没差别。 
[![ali_iot_architecture](./ali_iot_architecture.png)](https://www.zmannotes.com/wp-content/uploads/2018/02/ali_iot_architecture.png)

## 3 组件介绍

### 3.1 安全与认证

#### 3.1.1 原理

任何设备都必要拥有证书才能访问Broker，所有经过IOT的数据都使用TLS进行加密。IOT基于标准的TLS协议进行设备和云端双向认证来验证双方的身份。 设备需要将包含私钥的证书埋在安全可靠的地方，并且每个设备拥有唯一的证书，云端使用对应的公钥进行验证。 设备也需要将AWS的根证书加入本地可信任证书列表中，用来验证云端的身份。

#### 3.1.2 证书生成方式

证书可以在AWS控制台直接生成，也可以通过命令行，以及RESTful API进行生成。 
[https://docs.aws.amazon.com/cli/latest/reference/iot/create-keys-and-certificate.html](https://docs.aws.amazon.com/cli/latest/reference/iot/create-keys-and-certificate.html) 
[https://docs.aws.amazon.com/iot/latest/apireference/API_CreateKeysAndCertificate.html](https://docs.aws.amazon.com/iot/latest/apireference/API_CreateKeysAndCertificate.html)

#### 3.1.3 授权策略

IOT授权策略使用与IAM相同的规范进行管理，通过将设备唯一标识或组与策略进行关联实现权限的控制。举例如下：
```json
{
    "Version": "2012-10-17",
    "Statement": [{
        "Effect": "Allow",
        "Action":["iot:Publish"],
        "Resource": ["arn:aws:iot:us-east-1:123456789012:/topic/foo/bar"]
    },
    {
        "Effect": "Allow",
        "Action": ["iot:Connect"],
        "Resource": ["*"]
        }]
}
```
解释： 版本2012-10-17 允许进行发布到 us-east-1 区 topic/foo/bar 队列，同时允许连接到任何Broker。 
策略列表： [https://docs.aws.amazon.com/iot/latest/developerguide/action-resources.html](https://docs.aws.amazon.com/iot/latest/developerguide/action-resources.html)

**Action**

**Resource**

iot:DeleteThingShadow

A thing ARN - arn:aws:iot:us-east-1:123456789012:thing/thingOne

iot:Connect

A client ID ARN - arn:aws:iot:us-east1:123456789012:client/myClientId

iot:Publish

A topic ARN - arn:aws:iot:us-east-1:123456789012:topic/myTopicName

iot:Subscribe

A topic filter ARN - arn:aws:iot:us-east-1:123456789012:topicfilter/myTopicFilter

iot:Receive

A topic ARN - arn:aws:iot:us-east-1:123456789012:topic/myTopicName

iot:UpdateThingShadow

A thing ARN - arn:aws:iot:us-east-1:123456789012:thing/thingOne

### 3.2 Message Broker

#### 3.2.1 功能

Message Broker允许Client向某个Topic发送消息，订阅一个或多个Topic。Broker保存所有连接信息及订阅信息（这里介绍不清楚，直接影响到实现方式：所有连接信息是指当前broker上的所有连接，还是整个Broker集群的所有连接信息）。

#### 3.2.2 协议

##### 3.2.2.1 端口映射

**Protocol**|**Authentication**|**Port**
--- | --- | ---
MQTT | Client Certificate | 8883
HTTP | Client Certificate | 8443
HTTPS | SigV4 | 443
MQTT + WebSocket | SigV4 | 443

##### 3.2.2.2 MQTT

AWS IOT没有完全实现MQTT协议 [aws protocols](https://docs.aws.amazon.com/iot/latest/developerguide/protocols.html) 不支持QoS2； 不支持Retained消息； 不支持session持久化存储； …

#### 3.2.3 Topics

Topic用来在设备和云端之间路由消息。 `/`用来对topic进行分层。 `$`表示保留topic。 
[https://docs.aws.amazon.com/iot/latest/developerguide/topics.html](https://docs.aws.amazon.com/iot/latest/developerguide/topics.html) 
通配符功能如下：

**Wildcard**

**Description**
`
#
`
Must be the last character in the topic to which you are subscribing. Works as a wildcard by matching the current tree and all subtrees. For example, a subscription to Sensor/# will receive messages published to Sensor/,Sensor/temp, Sensor/temp/room1, but not the messages published to Sensor.
`
+
`
Matches exactly one item in the topic hierarchy. For example, a subscription to Sensor/+/room1 will receive messages published to Sensor/temp/room1, Sensor/moisture/room1, and so on.


#### 3.2.4 事件

[https://docs.aws.amazon.com/iot/latest/developerguide/life-cycle-events.html](https://docs.aws.amazon.com/iot/latest/developerguide/life-cycle-events.html) 
客户端建立连接时，IOT向如下topic发送事件，业务可以监听此topic获取通知： $aws/events/presence/connected/clientId 对应内容：
```json
{
    "clientId": "a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6",
    "timestamp": 1460065214626,
    "eventType": "connected",
    "sessionIdentifier": "00000000-0000-0000-0000-000000000000",
    "principalIdentifier": "000000000000/ABCDEFGHIJKLMNOPQRSTU:some-user/ABCDEFGHIJKLMNOPQRSTU:some-user"
}
```
连接断开时，对应的topic如下： 
`$aws/events/presence/disconnected/_clientId_ `
订阅topic时： 
`$aws/events/subscriptions/subscribed/_clientId_`
取消订阅时： 
`$aws/events/subscriptions/unsubscribed/_clientId_`

### 3.3 影子服务

#### 3.3.1 定义

设备影子是一个json串用来存储和检索设备的当前状态。

#### 3.3.2 功能

AWS IOT提供的三个方法来操作影子服务 
UPDATE：更新影子服务的状态，如果目标不存在就直接创建。 
GET：获取设备在影子服务中最新的快照。 
DELETE：删除设备对应的影子。 
[https://docs.aws.amazon.com/iot/latest/developerguide/thing-shadow-rest-api.html](https://docs.aws.amazon.com/iot/latest/developerguide/thing-shadow-rest-api.html)

#### 3.3.3 影子JSON内容

HTTP GET https://_endpoint_/things/_thingName_/shadow
```json
{
    "state": {
        "desired": {
            "lights": {
                "color": "RED"
            },
            "engine": "ON"
        },
        "reported": {
            "lights": {
                "color": "GREEN"
            },
            "engine": "ON"
        },
        "delta": {
            "lights": {
                "color": "RED"
            }
        }
    },
    "metadata": {
        "desired": {
            "lights": {
                "color": {
                    "timestamp": 123456
                },
             }
             "engine": {
                "timestamp": 123456
            }
        },
        "reported": {
            "lights": {
                "color": {
                    "timestamp": 789012
                }
            },
            "engine": {
                "timestamp": 789012
            }
        },
        "delta": {
            "lights": {
                "color": {
                    "timestamp": 123456
                }
            }
        }
    },
    "version": 10,
    "timestamp": 123456789
}
```
### 3.4 规则引擎

#### 3.4.1 定义

规则引擎提供一种将设备和云服务关联起来的能力。规则引擎基于mqtt topic stream来工作的。

#### 3.4.2 功能

过滤来自某一个设备的数据。 将数据路由到DynamoDB。 将数据保存到S3。 将数据路由到SQS。 将数据录入Elastic Search服务。 将数据路由到Amazon ML服务基于模型进行分析。 …

#### 3.4.3 例子

将目标为Topic ‘iot/test’ 的所有消息路由到名称为my-dynamodb-table的表中。
```json
{
  "sql": "SELECT * FROM 'iot/test'",
  "ruleDisabled": false,
  "awsIotSqlVersion": "2016-03-23",
  "actions": [{
      "dynamoDB": {
          "tableName": "my-dynamodb-table",
          "roleArn": "arn:aws:iam::123456789012:role/my-iot-role",
          "hashKeyField": "topic",
          "hashKeyValue": "${topic(2)}",
          "rangeKeyField": "timestamp",
          "rangeKeyValue": "${timestamp()}"
      }
  }]
}
``
将目标为Topic ‘some/topic’的所有消息路由到my_sqs_queue的队列中。
```json
{
    "rule": {
        "sql": "SELECT * FROM 'some/topic'", 
        "ruleDisabled": false, 
        "actions": [{
            "sqs": {
                "queueUrl": "https://sqs.us-east-2.amazonaws.com/123456789012/my_sqs_queue", 
                "roleArn": "arn:aws:iam::123456789012:role/aws_iot_sqs", 
                "useBase64": false
            }
        }]
    }
}
```

### 3.5 监控

#### 3.5.1 说明

监控是IOT服务的重要组成部分，用来监测服务的稳定性、可用性、性能。

#### 3.5.2 提出问题

AWS IoT没有指明监控如何实现，但是提出了设计监控时应当考虑的问题，挺专业的）_ 监控的目标是什么？ 监控那些资源？ 监控的频率是多长时间？ 使用什么监控工具？ 谁来执行监控任务？ 当监控到错误时，由谁来处理？ 
[https://docs.aws.amazon.com/iot/latest/developerguide/monitoring_overview.html](https://docs.aws.amazon.com/iot/latest/developerguide/monitoring_overview.html)

## 4 费用

### 4.1 计费

单连接每百万分钟$0.08; Broker每百万条消息$1.00; 影子服务每百万次操作$1.25; 规则引擎每百万次操作$0.15; SQS每百万条消息$0.40;

### 4.2 举例

以美西区为例，一百万辆车，每分钟上报两条数据，忽略（车控指令产生的费用，读取影子服务产生的费用）经过规则引擎路由到SQS计算一年中费用： 一百万辆车保持一年长连接：0.08 * 60 * 24 * 365 一百万辆车每辆车每分钟上报两条车辆状态(↑128k)：1 * 2 * 60 * 24 * 365 一百万辆车每辆车每分钟更新两次影子服务：1.25 * 2 * 60 * 24 * 365 一百万辆车每辆车每分钟触发两次规则引擎，并执行规则（触发和执行分别计费）：0.15 * 2 * 2 * 60 * 24 * 365 一百万辆车每辆车每分钟写两条消息到SQS（消费端读单独计费）：0.4 * 2 * 2 * 60 * 24 * 365 乘以6.4汇率，一百万辆车一年的保守费用：(0.08 + 1*2 + 1.25*2 + 0.15*2*2 + 0.4*2*2) * 60 * 24 * 365 * 6.4 = 22,806,836人民币。

## 5 缺点

5.1 只能通过日志查看消息的投递状态 5.2 对接依然需要较高开发成本


