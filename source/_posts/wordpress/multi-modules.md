---
title: 系统分层(单系统多模块)
tags:
  - maven 全局配置
  - Spring 框架
  - 多模块
id: '194'
categories:
  - - Spring
date: 2016-04-28 06:35:07
---

_介绍一般系统的分层思想和实现_ **1\. 主流分层**   [![multiple-models](https://www.zmannotes.com/wp-content/uploads/2016/04/multiple-models.jpg)](https://www.zmannotes.com/wp-content/uploads/2016/04/multiple-models.jpg) (每层对应一个或多个 模块/子系统，根据每层的复杂度进行进一步切分） **2\. 举例** 一个结算系统包含业务层、引擎层、基础组件层，化简后，对应模块为业务模块settle-order-service、引擎模块settle-engine、数据模块settle-data。 2.1 创建项目根目录

\# 进入workspace目录
cd ~/workspace
# 创建根目录
mvn archetype:generate -DarchetypeGroupId=org.codehaus.mojo.archetypes  -DarchetypeArtifactId=pom-root -DgroupId=com.zmannotes.spring -DartifactId=multiple-modules -Dversion=1.0.0-SNAPSHOT

2.2 创建业务模块 settle-order-service

cd multiple-modules

mvn -B archetype:generate -DarchetypeGroupId=org.apache.maven.archetypes -DgroupId=com.zmannotes.spring -DartifactId=settle-order-service

2.3 创建引擎模块settle-engine

mvn -B archetype:generate -DarchetypeGroupId=org.apache.maven.archetypes -DgroupId=com.zmannotes.spring -DartifactId=settle-engine

2.4 创建数据模块settle-data

mvn -B archetype:generate -DarchetypeGroupId=org.apache.maven.archetypes -DgroupId=com.zmannotes.spring -DartifactId=settle-data

**3\. 完整的项目示例** 3.1 所有模块 [![whole-project-modules](https://www.zmannotes.com/wp-content/uploads/2016/04/whole-project-modules-300x250.png)](https://www.zmannotes.com/wp-content/uploads/2016/04/whole-project-modules.png) 3.2 模块内部结构 [![module-structure](https://www.zmannotes.com/wp-content/uploads/2016/04/module-structure-300x150.png)](https://www.zmannotes.com/wp-content/uploads/2016/04/module-structure.png) **4\. 设置全局配置** 修改multiple-modules目录下的pom.xml来设定全局配置。 4.1 统一设置所有子模块的默认依赖

<!-- 添加默认测试依赖 -->
<dependencies>
    <!-- test -->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
    </dependency>                                                                                                                                                    
    <dependency>
        <groupId>org.mockito</groupId>
        <artifactId>mockito-all</artifactId>
        <version>1.10.19</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-test</artifactId>
        <version>4.1.8.RELEASE</version>
        <scope>test</scope>
    </dependency>
</dependencies>

4.2 统一设置所有子模块依赖的版本号

<!-- 设置变量 -->
<properties>
    <!--设置编码-->
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <!-- 添加spring版本号配置 -->
    <spring.version>4.1.8.RELEASE</spring.version>
</properties>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring.version}</version>
        </dependency>
    </dependencies>
</dependencyManagement>

4.3 设置发布仓库地址

<distributionManagement>
    <repository>
        <id>deployRelease</id>
        <url>http://127.0.0.1:5081/nexus/content/repositories/releases/</url>
    </repository>
    <snapshotRepository>
        <id>deploySnapshot</id>
        <url>http://127.0.0.1:5081/nexus/content/repositories/snapshots/</url>
    </snapshotRepository>
</distributionManagement>

4.4 设置JDK版本

<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>2.1</version>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
            </configuration>
        </plugin>
    </plugins>
</build>

 _Good Luck!_