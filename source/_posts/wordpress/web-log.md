---
title: Java Web项目添加log模块
tags:
  - log
  - Log4j2
id: '210'
categories:
  - - log
  - - Spring
date: 2016-05-08 16:54:53
---

_Java EE Web项目添加log模块_ 目前最好的Java日志组件是**[Log4j 2](http://logging.apache.org/log4j/2.x/manual/webapp.html)**，就以Log4j 2为例，且Tomcat版本>=7.0.43。 **1\. 项目依赖**

<dependencies>
  <dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-api</artifactId>
    <version>2.5</version>
  </dependency>
  <dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.5</version>
  </dependency>
  <dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-web</artifactId>
    <version>2.5</version>
    <scope>runtime</scope>
  </dependency>
</dependencies>

 **2. 创建log配置文件log4j2.xml**

在目录WEB-INF中创建配置文件log4j2.xml。
内容如下：
<?xml version="1.0" encoding="UTF-8"?>
<Configuration monitorInterval="30">
    <Appenders>
        <Console name="STDOUT" target="SYSTEM\_OUT">
            <PatternLayout pattern="%d{ISO8601} %-5p \[%t\] %-40.40c (%F:%L) - %m%n"/>
        </Console>
    </Appenders>
    <Loggers>
        <Root level="info">
            <AppenderRef ref="STDOUT"/>
        </Root>
    </Loggers>
</Configuration>

**3\. 修改web.xml** 3.1 Servlet 3.0 及以上 对于Servlet 3.0及以上的Web项目，不需要修改web.xml，LogContext会自动绑定生效。 （如果在web.xml手动配置了isLog4jAutoInitializationDisabled为true，即时是Servlet3.0，也要按Servlet2.5进行配置。原因是：这个参数导致ServletContainerInitializer没生效） 3.2 Servlet 2.5及以下

<!-- 此listener必须放在web.xml中其他listener和filter之前，保证在web初始化之前生效 -->
<listener>
    <listener-class>org.apache.logging.log4j.web.Log4jServletContextListener</listener-class>
</listener>
<!-- 此filter绑定LogContext到所有类型请求 -->
<filter>
    <filter-name>log4jServletFilter</filter-name>
    <filter-class>org.apache.logging.log4j.web.Log4jServletFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>log4jServletFilter</filter-name>
    <url-pattern>/\*</url-pattern>
    <dispatcher>REQUEST</dispatcher>
    <dispatcher>FORWARD</dispatcher>
    <dispatcher>INCLUDE</dispatcher>
    <dispatcher>ERROR</dispatcher>
    <dispatcher>ASYNC</dispatcher><!-- Servlet 3.0 w/ disabled auto-initialization only; not supported in 2.5 -->
</filter-mapping>

**4\. 高级应用** 4.1 log4j2与slf4j配合使用 [LINK](https://github.com/zman2013/Spring-web-log/blob/master/pom.xml) 4.2 使用Spring MVC拦截器配置全局traceId，方便快速定位  [LINK](https://github.com/zman2013/Spring-web-log/blob/master/src/main/webapp/WEB-INF/dispatcher-servlet.xml) 项目源码 [Github](https://github.com/zman2013/Spring-web-log) **Q&A** Q：如何判断Servlet版本？ A：web.xml中web-app标签的version属性表示Servlet版本。 Q：接口ServletContainerInitializer是做什么的？ A：ServletContainerInitializer初始化Log4jServletContextListener和Log4jServletFilter。 Q：为什么tomcat版本>=7.0.43？ A：Tomcat7.0.43(包含7.0.43)以上版本会自动加载Log4jServletContainerInitializer(实现ServletContainerInitializer接口)，以下版本会过滤 log4j\*.jar，需要手动修改catalina.properties，在属性jarsToSkip中删除log4j\*.jar。