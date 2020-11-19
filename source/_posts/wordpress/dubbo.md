---
title: Dubbo
tags:
  - Dubbo
  - Dubbo原理
  - Java 插件
  - Proxy
  - RPC
  - SPI
id: '83'
categories:
  - - RPC
date: 2015-10-31 09:04:44
---

_详解Dubbo设计结构和实现原理。_

### 前言

之前已经有许多介绍Dubbo的文章，但大部分内容比较粗糙或者内容是直接从Dubbo官网复制的，也有少部分内容详实的专题系列，却有不免于流水账式分析Dubbo的源代码。于是计划写一篇关于Dubbo设计结构和实现原理的总结，寄希望帮助读者阅读Dubbo源码时可以清晰明白各个模块如此设计的原理，进而可以轻松阅读Dubbo源码。

### 设计结构

Dubbo是一个分布式的支持服务治理功能的RPC框架，整个系统必然无法逃脱标准的[RPC设计结构](http://www.evoops.com/index.php/2015/10/22/rpc/)，如下图。 [![](https://www.zmannotes.com/wp-content/uploads/2015/10/dubbo_rpc_invoke.jpg)](https://www.zmannotes.com/wp-content/uploads/2015/10/dubbo_rpc_invoke.jpg) 上图即是Dubbo的基础结构，所有模块都是以此为核心扩展出来，完整的Dubbo结构如下图： [![系统结构](https://www.zmannotes.com/wp-content/uploads/2015/10/系统结构.jpg)](https://www.zmannotes.com/wp-content/uploads/2015/10/系统结构.jpg) 上图左侧为Dubbo的组成模块，右侧淡蓝色部分是使用的技术(思想)。 **组成模块** Remote Communication(通信层)：负责数据传输，Client发送Request，Server返回Response。 Protocol(编解码协议)：负责数据编码、解码，协议层接收到数据后，将数据编码后交给通信层，解码后交给Invoker层。 Invoker(逻辑层)：RPC的主要逻辑都在此实现，所有服务治理的接口都抽象为Invoker，组成一个Invoker链，每个完整的请求都会层层经过此链，链中每个invoker完成各自的责任。 Proxy(代理层 )：负责将Client的调用转为Invoker链的调用。 Interface Implement(实现层)：即接口的实现，也叫做业务逻辑层，是Client最终期望引用的远程逻辑。 Spring Config(配置层)：负责将Spring配置实例化为对象，使开发人员只需进行简单的spring配置就完成服务暴露、引用代理实例化，具体实现逻辑透明化。 **底层技术** SPI(Service Provider Interface)：字面意思是服务提供方接口，核心系统规定服务接口，所有实现遵循服务接口规定，当系统需要升级时不需要改动核心代码而是切换服务接口的实现。Dubbo就是基于此思想实现微内核+插件的设计结构，所有模块均可以自行实现和替换。 Proxy：我认为代理模式是RPC的核心模式，通过Client端的代理来调用Server端的实现。 Class Loader：类加载器。Dubbo重新实现SPI，不同模块在程序运行时按需加载到容器中，为了支持此行为，Class Loader自然而然被引入。 NIO：Dubbo数据传输层默认使用NIO框架Netty，NIO降低了服务端的线程数量，进而提高资源利用率。 Spring Schema Ext：Spring Schema扩展实现比较简单，按特定规则完成配置项的定义和解析即可，却实现了通过简单配置替代编码的目的。 **SPI** [![](https://www.zmannotes.com/wp-content/uploads/2015/10/屏幕快照-2015-10-31-上午7.41.58.png)](https://www.zmannotes.com/wp-content/uploads/2015/10/屏幕快照-2015-10-31-上午7.41.58.png) 如上图，基于SPI设计结构的系统一般有两部分：核心程序（主程序）与接口实现程序（插件）。 主程序负责定义SPI(即Interface)和按特定规则加载插件；插件负责实现主程序定义的SPI并按特定规则定义配置文件。 我们几乎每天都在使用标准Java SPI是JDBC定义的java.sql.Driver，用的时候只需要把目标数据库依赖的jar放入Classpath，JDBC主程序会自动找到各DB Driver，准备后续使用。 **[SPI例子](https://github.com/zman2013/java-spi-demo)** [![](https://www.zmannotes.com/wp-content/uploads/2015/10/屏幕快照-2015-10-31-上午8.01.58-300x213.png)](https://www.zmannotes.com/wp-content/uploads/2015/10/屏幕快照-2015-10-31-上午8.01.58.png) 主程序

　定义SPI

　在主程序内部加载并调用SPI

插件

　为SPI提供实现逻辑

　在固定位置声明自身为SPI的提供者

  下面介绍一个加密程序的例子：为了达到信息安全防止被破解的目的，设计的一套加密程序并计划不定时更换加密算法，为了在不改动主程序的情况下灵活切换、新增加密算法，于是使用SPI设计结构，具体实现如下。 主程序

/\*\*
 \* 此接口为标准Java SPI，
   所有加密算法实现实现应当遵循此SPI。
 \*/
public interface Encryption {

    /\*\*
     \* 对data进行加密，然后返回加密数据。
     \* 
     \* @param data
     \*            字节数组
     \* @return 加密后的数据
     \*/
    byte\[\] encrypt(byte\[\] data);
}

/\*\*
 \* 加密器主程序
 \* 负责加载、调用加密算法的Provider。
 \*/
public class EncryptorApplication {

    public static void main(String\[\] args) {

        //准备加密数组
        byte\[\] data = new byte\[\] { 1, 2, 3, 4, 5, 6, 7, 8, 9 };

        //加载所有加密算法
        ServiceLoader<Encryption> encryptions = ServiceLoader.load(Encryption.class);
        Iterator<Encryption> encryptionIter = encryptions.iterator();

        if (!encryptionIter.hasNext()) {
            System.out.println("there is no provider for encryption algorithm.");
        }
        //进行组合加密
        while (encryptionIter.hasNext()) {
            Encryption encryption = encryptionIter.next();
            data = encryption.encrypt(data);
        }
        //输出加密后数据
        for (int i = 0; i < data.length; i++) {
            System.out.print(data\[i\] + ",");
        }
    }
}

插件（加密算法）

/\*\*
 \* 遍历所有字节，对所有字节+1。
 \*/
public class IncrementAlgorithm implements Encryption {

    /\*\*
     \* 对数组中所有字节+1，返回修改后的字节数组。
     \*/
    @Override
    public byte\[\] encrypt(byte\[\] data) {
        if (null == data  data.length == 0) {
            return data;
        }
        for (int i = 0; i < data.length; i++) {
            data\[i\] = (byte) (data\[i\] + 1);
        }

        return data;
    }
}

配置文件META-INF/services/tutorial.jdk.spi.Encryption
    tutorial.jdk.spi.ReverseAlgorithm
    tutorial.jdk.spi.IncrementAlgorithm

为了介绍方便，这个例子将主程序与插件放在同一个project里，线上产品是将各加密算法拆分为单独的project，主程序希望调用哪个算法或者进行组合调用，就将对应的.jar放入Classpath，部署起来要方便的多。 **Proxy** 代理模式是每一个RPC框架的核心设计模式。 Dubbo消费者的配置如下：

<dubbo:reference id="demoService" interface="com.alibaba.dubbo.demo.DemoService" />

此配置信息在消费者端生成接口DemoService的代理DemoServiceProxy，DemoServiceProxy完成了网络数据通信的具体过程，且Proxy的内部逻辑对开发人员完全透明，使用起来就像调用本地的接口一样方便。 代理模式也是每天都会用到，最多的应该是Spring的事务。Proxy的实现技术有多个：最强大的是Aspectj，还有Cglib和基于接口的JDK Proxy。 **[Proxy例子](https://github.com/zman2013/java-proxy-demo)** [![](https://www.zmannotes.com/wp-content/uploads/2015/10/proxy例子.png)](https://www.zmannotes.com/wp-content/uploads/2015/10/proxy例子.png) 调用方

public class Client {

    public static void main(String\[\] args) {
        //通过代理工厂获取proxy
        HelloWorld proxy = HelloWorldProxyFactory.getProxy();
        //调用proxy代理的方法
        proxy.sayHelloWorld();
    }
}

Proxy的回调方法

/\*\*
 \* 该类对应HelloWorld Proxy的具体调用逻辑。
 \*/
public class HelloWorldHandler implements InvocationHandler {

    //被代理的原始对象
    private Object obj;

    public HelloWorldHandler(Object obj) {
        super();
        this.obj = obj;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object\[\] args) throws Throwable {
        Object result = null;
        //调用sayHelloWorld方法前的动作
        System.out.println("Firstly, open mouth!");
        //调用sayHelloWorld方法
        result = method.invoke(obj, args);
        //调用sayHelloWorld方法后续动作
        System.out.println("At last, close mouth!");

        return result;
    }
}

代理工厂

/\*\*
 \* 代理工厂类
 \* 代理工厂可参考:org.springframework.aop.framework.ProxyFactory
 \*/
public class HelloWorldProxyFactory {

    /\*\*
     \* 获得HelloWorld的代理对象。
     \*/
    public static HelloWorld getProxy() {
        //接口的实现类
        HelloWorld helloWorld = new HelloWorldImpl();

        //proxy的调用类
        InvocationHandler handler = new HelloWorldHandler(helloWorld);

        //生成proxy
        HelloWorld proxy = (HelloWorld) Proxy.newProxyInstance(
                helloWorld.getClass().getClassLoader(),
                helloWorld.getClass().getInterfaces(),
                handler);

        return proxy;
    }
}

基于Proxy如何实现RPC框架？请参考[《RPC》](http://www.evoops.com/index.php/2015/10/22/rpc/)。 _顺便说一句，Dubbo的设计思想的确很优秀，只是代码质量不堪入目，实在太差了。_