---
title: Spring MVC项目框架
tags:
  - 'jetty:run'
  - Spring MVC
  - Spring Web
  - Spring Web项目搭建
id: '190'
categories:
  - - Spring
date: 2016-04-16 20:43:47
---

_搭建基本的Spring MVC项目框架_ 最终目录结构 [![spring mvc web structure](https://www.zmannotes.com/wp-content/uploads/2016/04/1-1.jpg)](https://www.zmannotes.com/wp-content/uploads/2016/04/1-1.jpg) **1\. 创建Maven Web项目**

// 建立名为spring-mvc-web的maven web项目
$ mvn archetype:generate -DarchetypeArtifactId=maven-archetype-webapp -DinteractiveMode=false -DgroupId=com.zmannotes.spring -DartifactId=spring-mvc-web

2\. 修改pom.xml如下

<?xml version="1.0"?>
<project
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd"
    xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.zmannotes.spring</groupId>
    <artifactId>spring-mvc-web</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>war</packaging>
    <name>spring mvc web</name>

    <properties>
        <spring.version>4.2.5.RELEASE</spring.version>
    </properties>

    <dependencies>

        <!-- spring mvc -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>${spring.version}</version>
        </dependency>

    </dependencies>

</project>

3\. 修改web.xml如下

<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns="http://java.sun.com/xml/ns/javaee" xmlns:jsp="http://java.sun.com/xml/ns/javaee/jsp"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app\_3\_0.xsd"
    version="3.0">

    <display-name>Spring MVC Web</display-name>

    <!-- 默认加载配置文件applicationContext.xml，配置web运行的上下文 -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <!-- 指定请求分发器，所有.htm结尾的请求都会统一交给DispatcherServlet进行分发 -->
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>\*.htm</url-pattern>
    </servlet-mapping>

    <!-- 指定request和response编码，统一为utf-8 -->
    <filter>
        <filter-name>CharacterFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>utf-8</param-value>
        </init-param>
        <init-param>
            <param-name>force</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>CharacterFilter</filter-name>
        <url-pattern>/\*</url-pattern>
    </filter-mapping>

</web-app>

4\. 添加配置文件applicationContext.xml

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="  http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd ">

    <!-- 此文件定义共享的配置信息 -->
    
    <!-- 加载配置文件 -->
    <context:property-placeholder
            location="classpath:application.properties"/>
    
</beans>

5\. 添加配置文件dispatcher-servlet.xml

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/mvc 
        http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <!-- 扫描指定package下的Controller类 -->
    <context:component-scan base-package="com.zmannotes.spring.web.controller" />
    
    <!-- 激活spring mvc功能 -->
    <mvc:annotation-driven/>

    <!-- 指定web页面根路径和拓展名 -->
    <bean id="viewResolver"
        class="org.springframework.web.servlet.view.UrlBasedViewResolver">
        <property name="viewClass"
            value="org.springframework.web.servlet.view.JstlView" />
        <property name="prefix" value="/view/" />
        <property name="suffix" value=".jsp" />
    </bean>
</beans>

6 创建key-value配置文件application.properties（非必须）

key1=value1

7 创建HomeController.java

package com.zmannotes.spring.web.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;

@Controller                     // 标明此类是一个Controller
@RequestMapping("/home")        // 标明Url根路径为/home
public class HomeController {

    @RequestMapping("hi")       // 标明Url子路径为hi
    public String hi(
            @RequestParam(name = "name", defaultValue = "hacker") String name,
            Model model) {

        String msg = String.format("Welcome %s", name);

        model.addAttribute("msg", msg);

        // 返回名为home的页面
        return "home";
    }

}

8 启动Web 8.1 启动

$ mvn jetty:run

8.2 看到如下信息，表明启动成功

...
\[INFO\] Started ServerConnector@2d64160c{HTTP/1.1,\[http/1.1\]}{0.0.0.0:8080}
\[INFO\] Started @12089ms
\[INFO\] Started Jetty Server

8.3 访问Url: localhost:8080/home/hi.htm?name=zman [源码 GitHub](https://github.com/zman2013/spring-mvc-web) Q&A Q：applicationContext.xml与xxx-servlet.xml的关系？ A：servlet可以有多个，每个xxx-servlet.xml配置应当只配置此servlet专有的信息。而applicationContext.xml用来配置多个servlet共享的bean等信息，且是非必须的。