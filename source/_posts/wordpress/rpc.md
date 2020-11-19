---
title: RPC
tags:
  - Demo
  - Proxy
  - RPC
  - 代理
  - 服务治理
id: '53'
categories:
  - - RPC
date: 2015-10-22 07:22:47
---

详细介绍基于RPC的系统与本地调用程序、常用的网络系统的区别，和基本的RPC系统如何实现。

### RPC是什么

RPC（Remote Procedure Call）从字面理解就是远程过程调用，即像调用本地方法一样调用网络另一端系统中的一段逻辑。我们常说的RPC指的是RPC框架，一个把网络请求、数据编码都封装好的框架或者lib库，内部逻辑对开发人员是透明的，开发人员只关心调用了一个（远程）方法，然后得到了结果。

### RPC的好处

RPC使得引用方在本地直接调用服务接口就可以完成远端逻辑的执行，省去网络请求、数据编码解码等步骤，无疑可以极大提高开发效率。同时由于所有的请求都交给框架处理，我们又可以进一步扩展框架负责的功能，比如：服务发现、容错、负载均衡、路由等，自然而然服务治理就产生了。

### RPC介绍

通过介绍简单的本地调用结构、HTTP调用结构、PRC调用结构，可以直观地理解RPC究竟是什么、做了哪些事情。 **本地调用结构** [![本地调用结构](https://www.zmannotes.com/wp-content/uploads/2015/10/本地调用结构1.jpg)](https://www.zmannotes.com/wp-content/uploads/2015/10/本地调用结构1.jpg) 本地调用结构中所有模块都在一个系统里面，Caller可以直接调用另一段代码逻辑。简单、方便、高效，但耦合度高。 **HTTP API调用结构** [![HTTP-API调用结构](https://www.zmannotes.com/wp-content/uploads/2015/10/HTTP-API调用结构.jpg)](https://www.zmannotes.com/wp-content/uploads/2015/10/HTTP-API调用结构.jpg) 目前中小型的互联网公司均使用HTTP API方式实现系统间的调用，此结构相比本地调用，已经实现了解耦，使得开发工作更加灵活，只要保持API不变，不同功能开发互不影响。但增加了调用端和服务器端的工作量，对于每个服务：服务端要进行数据与json的双向转换（或映射）并封装为HTTP接口，调用端同样要进行数据与json的双向转换并实现http请求的调用逻辑。 **RPC调用结构** [![RPC调用结构](https://www.zmannotes.com/wp-content/uploads/2015/10/RPC调用结构.jpg)](https://www.zmannotes.com/wp-content/uploads/2015/10/RPC调用结构.jpg) 如上图，调用方调用‘API Proxy’即完成远端服务调用过程，RPC框架隐式地完成网络通信逻辑。服务提供方也只需要实现相应的服务逻辑，使工程师将更多精力集中在业务上。 _‘API Proxy’是什么？_ _服务接口的代理，是RPC Framework暴露出的供调用方使用的，输入输出与Implement一样（可参考代理模式）。_ **完整RPC调用过程** [![详细RPC调用结构](https://www.zmannotes.com/wp-content/uploads/2015/10/详细RPC调用结构2.jpg)](https://www.zmannotes.com/wp-content/uploads/2015/10/详细RPC调用结构2.jpg) 上图中所有蓝色块是RPC框架实现的功能模块。 在公司内部，为了实现对系统的监控、增强稳健性、扩展性，经常需要做一些额外的工作。 例如：为了统计接口的调用次数、响应时间，一般是通过日志记录；服务器地址或端口改动时，引起下游消费端进行相应配置改动；负载均衡通过各种第三方工具实现…… 由于所有服务调用均通过RPC框架，因而很容易拓展RPC框架实现服务治理功能，如下图：   [![RPC服务治理1](https://www.zmannotes.com/wp-content/uploads/2015/10/RPC服务治理1.jpg)](https://www.zmannotes.com/wp-content/uploads/2015/10/RPC服务治理1.jpg)

### RPC Demo

先假设RPC框架已经完成，基于RPC框架开发系统需要做那些事情呢？ 以Calculator为例，上游系统调用下游的计算系统获取结果。 **1\. 设计接口/契约**

public interface Calculator {

    /\*\*
     \* @return a \* b
     \*/
    int multiply(int a, int b);

    /\*\*
     \* @return a / b
     \*/
    int devide(int a, int b);
}

**2\. 上游系统(Caller)实现**

public class CalculatorCaller {

    public static void main(String\[\] args) {
        SimpleRPCFramework rpc = new SimpleRPCFramework();   //实例化RPC框架

        Calculator calc = rpc.getProxy(Calculator.class);    //获得Calculator代理

        for (int i = 0; i < 10; i++) {                       //把Calculator代理当做Calculator具体实现来调用,
            int result = calc.multiply(i, (i + 1));          //剩下的事情RPC框架会自行处理
            System.out.println(result);
        }
    }
}

_（有木有很简单？就像本地调用一样简单）_ **3\. 下游系统(Server)实现**

public class CalculatorImpl implements Calculator {      //实现Calculator接口

    @Override
    public int multiply(int a, int b) {
        return a \* b;
    }

    @Override
    public int devide(int a, int b) {
        return a / b;
    }

}

public class CalculatorServer {

    public static void main(String\[\] args) throws IOException {
        //实例化helloWorld
        CalculatorImpl calculator = new CalculatorImpl();
        //创建rpc框架实例
        SimpleRPCFramework rpc = new SimpleRPCFramework();
        //注册接口和对应实现
        rpc.registerImplements(Calculator.class, calculator);     //开放更多服务，只需修改此处
        //暴露服务接口，监听服务
        rpc.expose();
    }
}

至此，接口设计、上下游所有开发工作完成。接下来看下神奇的RPC框架究竟如何实现~ **4\. Simple RPC Framework** 根据上面的例子，可以发现该RPC框架有三个接口：

public interface RPC {
    
    /\*\*
     \* 服务器端注册接口与相应实现。
     \*
     \* @param inteface
     \* @param instance
     \*/
    void registerImplements(Class<?> inteface, Object instance);
    
    /\*\*
     \* 服务器端暴露端口，并监听调用请求
     \*/
    void expose() throws IOException;
    
    /\*\*
     \* 调用端获取接口的代理
     \*/
    <T> T getProxy(final Class<T> inteface);
}

下面看一下具体实现：

/\*\*
 \* @from zmannotes.com
 \*/
public class SimpleRPCFramework {

    /\*\* 存储接口与实现的对应关系 \*/
    private final Map<Class<?>, Object> implCache = new ConcurrentHashMap<>();

    /\*\* 存储接口与代理的对应关系 \*/
    private final Map<Class<?>, Object> proxyCache = new ConcurrentHashMap<>();

    public synchronized void registerImplements(Class<?> inteface, Object instance) {
        //检测接口是否已经存在
        if (implCache.containsKey(inteface)) {
            throw new RuntimeException("This interface\[" + inteface.getName() + "\] have already been exposed.");
        } else {
            implCache.put(inteface, instance);
        }
    }
    
    public void expose() throws IOException {
        //……
    }
    
    public synchronized <T> T getProxy(final Class<T> inteface) {
       //……
    }
}

registerImplements没有复杂逻辑，只是检测缓存中是否已经包含目标接口，完全可以并入expose中，但为使逻辑简单，故为之。 下面是具体的服务暴露逻辑，采用基于Socket的通信方式，编码解码直接使用Java IO自带的序列化流。 原理比较简单，就是上面介绍的三个模块：

1.  服务器负责监听端口和接收/发送数据。
2.  协议模块负责编码/解码数据。
3.  Dispatcher负责将请求分发到对应的实现逻辑。

public void expose() throws IOException {
        //暴露服务端口
        ServerSocket server = new ServerSocket(2008);
        while (true) {
            final Socket socket = server.accept();
            new Thread(new Runnable() {
                public void run() {
                    try (
                            ObjectInputStream in = new ObjectInputStream(socket.getInputStream());
                            ObjectOutputStream out = new ObjectOutputStream(socket.getOutputStream())) {
                        //获取请求数据 - 接收请求,并解码
                        Class<?> inter = (Class<?>) in.readObject();
                        String methodName = in.readUTF();
                        Class<?>\[\] parameterTypes = (Class<?>\[\]) in.readObject();
                        Object\[\] args = (Object\[\]) in.readObject();
                        //根据接口信息获得对应的接口实现 - Dispatcher
                        Object instance = implCache.get(inter);
                        //调用接口实现 - invoke
                        Method method = instance.getClass().getMethod(methodName, parameterTypes);
                        Object result = method.invoke(instance, args);
                        //返回结果 - 编码,并返回结果
                        out.writeObject(result);
                    } catch (Exception e) {
                        throw new RuntimeException(e);
                    }
                }
            }).run();
        }
    }

下面是接口代理类的生成逻辑，采用JDK自带的Proxy生成机制来实现，具体流程参见‘完整RPC调用过程’。

public synchronized <T> T getProxy(final Class<T> inteface) {
        //检测接口是否已经存在
        if (proxyCache.containsKey(inteface)) {
            return (T) proxyCache.get(inteface);
        }

        T proxy = (T) Proxy.newProxyInstance(Thread.currentThread().getContextClassLoader(),
                new Class<?>\[\] { inteface }, new InvocationHandler() {
                    public Object invoke(Object proxy, Method method, Object\[\] args) throws Throwable {
                        Socket socket = new Socket(InetAddress.getLocalHost(), 2008);
                        try(
                                ObjectOutputStream output = new ObjectOutputStream(socket.getOutputStream());
                                ObjectInputStream input = new ObjectInputStream(socket.getInputStream())
                                ){
                            //发送请求数据 - 编码,并发送请求
                            output.writeObject(inteface);
                            output.writeUTF(method.getName());
                            output.writeObject(method.getParameterTypes());
                            output.writeObject(args);
                            //接收结果 - 接受数据,并解码
                            Object result = input.readObject();

                            return result;
                        }
                    }
                });
        proxyCache.put(inteface, proxy);
        return proxy;
    }

### [总结](https://www.zmannotes.com/index.php/2015/10/22/rpc/)

本文对RPC的原理和模块进行较详细介绍，并与其它系统间调用方式进行对比，最后给出简单的RPC框架实现。 例子中各模块都可以进一步扩展、优化，以提高框架的运行效率、稳定性、扩展性。完整Demo代码存于Github: [Go](https://github.com/zman2013/simple-rpc-demo)。