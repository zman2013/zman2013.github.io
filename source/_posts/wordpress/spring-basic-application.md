---
title: 快速开发Spring Application
tags:
  - Spring Application
  - Spring Demo
  - Spring 例子
  - Spring 框架
  - Spring项目搭建
id: '170'
categories:
  - - Spring
date: 2016-04-16 12:06:03
---

_最近为公司其他语言的同事介绍Spring框架的入门知识，在此整理为Spring系列_ **开发环境** JDK

1.8\_xx

IDE

Eclipse/STS (推荐使用STS，Spring团队基于Eclipse开发的IDE，很适合做Spring项目开发）

IntelliJ

项目管理工具

Maven3.xx

**Spring Basic Application** **1\. 使用maven创建新项目**

命令格式如下：
mvn -B archetype:generate -DarchetypeGroupId=org.apache.maven.archetypes -DgroupId=xxx -DartifactId=xxx
需要指定：
    groupId 一般格式为 com.公司名.系统名
    artifactId 项目名称，单词之间用 - 分割

例子：
$: mvn -B archetype:generate -DarchetypeGroupId=org.apache.maven.archetypes -DgroupId=com.zmannotes.spring -DartifactId=spring-basic-application

在当前目录得到一个名为spring-basic-application的项目

**2\. 将新创建的项目导入STS/Eclipse** 2.1 打开STS，选择菜单 File -> Import 2.2 选择Maven -> Existing Maven Projects，点击Next [![import maven project](https://www.zmannotes.com/wp-content/uploads/2016/04/1.jpg)](https://www.zmannotes.com/wp-content/uploads/2016/04/1.jpg) 2.3 在新的窗口中选择第1步创建的spring-basic-application目录，点击Finish即可。 2.4 最终导入项目如下 [![](https://www.zmannotes.com/wp-content/uploads/2016/04/2-1.jpg)](https://www.zmannotes.com/wp-content/uploads/2016/04/2-1.jpg) **3\. 配置项目依赖，引入Spring框架，编辑pom.xml文件，内容如下**

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4\_0\_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.zmannotes.spring</groupId>
    <artifactId>spring-basic-application</artifactId>
    <packaging>jar</packaging>
    <version>1.0-SNAPSHOT</version>
    <name>spring-basic-application</name>

    <!-- 父配置文件，继承通用的配置 -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.3.3.RELEASE</version>
    </parent>

    <dependencies>
        <!-- spring 依赖 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <!-- 打包用插件 -->
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>

**4\. 实现一个例子** 一个工人每间隔1s工作一段时间 4.1 创建Worker.java

package com.zmannotes.spring.worker;

import org.springframework.stereotype.Component;

@Component    //Component注解 标明该类可以被自动发现
public class Worker {

    public void start() throws InterruptedException {
        while (true) {
            System.out.println("I'm working...");
            // 暂时1s
            Thread.sleep(1000);
        }
    }
}

4.2 修改App.java，使其调用Worker

package com.zmannotes.spring;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

import com.zmannotes.spring.worker.Worker;

/\*\*
 \* Hello world!
 \*
 \*/
@SpringBootApplication  //激活Spring注解：@Component @Autowired等
public class App implements CommandLineRunner {

    @Autowired  //自动注入Worker对象
    private Worker worker;

    /\*\* main函数 \*/
    public static void main( String\[\] args )
    {
        SpringApplication.run(App.class);
    }

    /\*\* 程序入口 \*/
    public void run(String... args) throws Exception {
        worker.start();
    }

}

4.3 最终目录结构 [![spring-basic-application-structure-final](https://www.zmannotes.com/wp-content/uploads/2016/04/4.jpg)](https://www.zmannotes.com/wp-content/uploads/2016/04/4.jpg) **5 运行程序** 5.1 右击App.java，选择Run As -> Java Application [![sts-run-java-application](https://www.zmannotes.com/wp-content/uploads/2016/04/3.jpg)](https://www.zmannotes.com/wp-content/uploads/2016/04/3.jpg) 5.2 输出结果如下

  .   \_\_\_\_          \_            \_\_ \_ \_
 /\\\\ / \_\_\_'\_ \_\_ \_ \_(\_)\_ \_\_  \_\_ \_ \\ \\ \\ \\
( ( )\\\_\_\_  '\_  '\_  '\_ \\/ \_\`  \\ \\ \\ \\
 \\\\/  \_\_\_) \_)      (\_   ) ) ) )
  '  \_\_\_\_ .\_\_\_ \_\_ \_\\\_\_,  / / / /
 =========\_==============\_\_\_/=/\_/\_/\_/
 :: Spring Boot ::        (v1.3.3.RELEASE)

2016-04-16 10:54:08.843  INFO 6072 --- \[           main\] com.zmannotes.spring.App                 : Starting App on evo with PID 6072 (\\workspace\\spring-tutorial\\spring-basic-application\\spring-basic-application\\target\\classes started by zman in \\workspace\\spring-tutorial\\spring-basic-application\\spring-basic-application)
2016-04-16 10:54:08.843  INFO 6072 --- \[           main\] com.zmannotes.spring.App                 : No active profile set, falling back to default profiles: default
2016-04-16 10:54:08.937  INFO 6072 --- \[           main\] s.c.a.AnnotationConfigApplicationContext : Refreshing org.springframework.context.annotation.AnnotationConfigApplicationContext@3891771e: startup date \[Sat Apr 16 10:54:08 CST 2016\]; root of context hierarchy
2016-04-16 10:54:10.614  INFO 6072 --- \[           main\] o.s.j.e.a.AnnotationMBeanExporter        : Registering beans for JMX exposure on startup
I'm working...
I'm working...
I'm working...
I'm working...
I'm working...

**6 打包、运行** 6.1 回到console中spring-basic-application目录下，运行`mvn clean package`，会在target目录下生成spring-basic-application.jar 6.2 运行jar包   `java -jar target/spring-basic-application-1.0-SNAPSHOT.jar` 6.3 输出与5.2相同 :) [源码 GitHub](https://github.com/zman2013/spring-basic-application) **Q&A** Q：pom.xml是什么东西？ A：pom.xml文件时maven项目的标志性文件，在此文件中配置项目的依赖、打包方式等各种配置信息，maven通过解析、执行pom.xml中的配置进行项目的管理、打包、发布。 Q：没感觉到Spring框架的强大 A：的确。这个基础应用只使用了Spring最核心最基本的注入功能，简单来说就是把一个对象注入到另一个对象供其使用。注入关系是通过注解@Autowired的方式声明，由spring统一管理的。（当然还有另一种方式通过配置xml的方式进行声明）可以尝试一下不是Spring，如果完成上面的例子。