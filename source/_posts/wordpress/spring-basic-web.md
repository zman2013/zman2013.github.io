---
title: 快速开发Spring Web应用
tags:
  - Spring MVC
  - Spring Web
  - Spring Web项目搭建
id: '185'
categories:
  - - Spring
date: 2016-04-16 17:28:03
---

_基于spring boot可以秒建一个基本__web应用_ **1\. 创建maven项目**

//创建名为spring-basic-web的maven项目
$ mvn -B archetype:generate -DarchetypeGroupId=org.apache.maven.archetypes -DgroupId=com.zmannotes.spring -DartifactId=spring-basic-web

_（详细说明请参考: [快速开发Spring Application](https://www.zmannotes.com/index.php/2016/04/16/spring-basic-application/))_ **2\. 导入STS/Eclipse/IntelliJ** **3\. 修改pom.xml内容如下**

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4\_0\_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.zmannotes.spring</groupId>
    <artifactId>spring-basic-web</artifactId>
    <packaging>jar</packaging>
    <version>1.0-SNAPSHOT</version>
    <name>spring-basic-web</name>

    <!-- 父配置文件，继承通用的配置 -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.3.3.RELEASE</version>
    </parent>

    <dependencies>
        <!-- spring web 依赖 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <!-- 打包用插件 -->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>

**4\. 例子** 建立一个web服务，访问Url，返回json串。 4.1 创建json串对应的Java对象

package com.zmannotes.spring.domain;

public class HomeResponse {
    /\*\* 响应码 \*/
    private int code;
    /\*\* 响应信息 \*/
    private String message;

    public HomeResponse(int code, String msg) {
        this.code = code;
        this.message = msg;
    }

    public int getCode() {
        return code;
    }

    public String getMessage() {
        return message;
    }

}

4.2 创建Controller对象

package com.zmannotes.spring.controller;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import com.zmannotes.spring.domain.HomeResponse;

@RestController                 //标明此类是一个Controller
@RequestMapping("/home")        //标明Url根路径为/home
public class HomeController {

    @RequestMapping("hi")       //标明Url子路径为hi
    public HomeResponse hi(
            @RequestParam(name = "name", defaultValue = "hacker") String name) {

        String msg = String.format("Welcome %s", name);

        HomeResponse response = new HomeResponse(0, msg);

        return response;
    }

}

4.3 修改App.java

package com.zmannotes.spring;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class App {
    public static void main(String\[\] args) {
        SpringApplication.run(App.class, args);
    }
}

至此开发阶段完成！ **5\. 打包、运行** 5.1 打包 `$ mvn clean package` 5.2 运行

$ java -jar target/spring-basic-web-1.0-SNAPSHOT.jar

**6\. 测试** 6.1 访问Url：http://localhost:8080/home/hi?name=zman 6.2 返回信息如下：

{
    code: 0,
    message: "Welcome zman"
}

[源码 GitHub](https://github.com/zman2013/spring-basic-web) **Q&A** Q：为什么没有用到tomcat这类的容器？ A：Spring boot已经在项目依赖中自动内嵌了tomcat，因此不需要外置的tomcat容器了。