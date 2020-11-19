---
title: Mybatis Generator
tags:
  - mybatis代码生成器
id: '202'
categories:
  - - code generator
  - - mybatis
date: 2016-05-05 21:31:10
---

_代码生成器 自动生成mybatis代码、配置文件_ **1\. 定义规则文件**

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
  PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
  "http://mybatis.org/dtd/mybatis-generator-config\_1\_0.dtd">

<generatorConfiguration>
    <!-- 声明jdbc位置-->
    <classPathEntry
        location="/Users/{用户名}/.m2/repository/mysql/mysql-connector-java/5.1.37/mysql-connector-java-5.1.37.jar" />

    <context id="DB2Tables" targetRuntime="MyBatis3">
        <!-- 数据库地址、用户名密码 -->
        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
            connectionURL="jdbc:mysql://xxx:2301/stock" userId="msandbox"
            password="msandbox">
        </jdbcConnection>

        <javaTypeResolver>
            <property name="forceBigDecimals" value="false" />
        </javaTypeResolver>

        <!-- 定义domain类路径 -->
        <javaModelGenerator targetPackage="com.zmannotes.spring.mybatis.domain"
            targetProject="src/main/java">
            <property name="enableSubPackages" value="true" />
            <property name="trimStrings" value="true" />
        </javaModelGenerator>

        <!-- 定义XXXMapper文件路径 -->
        <sqlMapGenerator targetPackage="mybatis/stock"
            targetProject="src/main/resources">
            <property name="enableSubPackages" value="true" />
        </sqlMapGenerator>

        <!-- 定义dao类的路径 -->
        <javaClientGenerator type="XMLMAPPER"
            targetPackage="com.zmannotes.spring.mybatis.dao" targetProject="src/main/java">
            <property name="enableSubPackages" value="true" />
        </javaClientGenerator>

        <!-- 指明表名stock -->
        <table schema="" tableName="stock" domainObjectName="Stock"
            enableCountByExample="false" enableUpdateByExample="false"
            enableDeleteByExample="false" enableSelectByExample="false"
            selectByExampleQueryId="false" />
        <!-- 指明表名stock\_detail （同一个文件中可以指定多个表） -->
        <table schema="" tableName="stock\_detail" domainObjectName="StockDetail"
            enableCountByExample="false" enableUpdateByExample="false"
            enableDeleteByExample="false" enableSelectByExample="false"
            selectByExampleQueryId="false" />


    </context>
</generatorConfiguration>

**2\. 修改pom文件**

<build>

    <plugins>
        <plugin>
            <groupId>org.mybatis.generator</groupId>
            <artifactId>mybatis-generator-maven-plugin</artifactId>
            <version>1.3.2</version>
            <configuration>
                <verbose>true</verbose>
                <overwrite>true</overwrite>
            </configuration>
        </plugin>
    </plugins>

</build>

 **3. 生成代码**

$ mvn mybatis-generator:generate

结果如下图（Stock、StockMapper、StockMapper.xml均自动生成，且包含基本访问数据的接口） [![mybatis-generator](https://www.zmannotes.com/wp-content/uploads/2016/05/mybatis-generator.jpg)](https://www.zmannotes.com/wp-content/uploads/2016/05/mybatis-generator.jpg) 增删改查基本操作，自动生成的代码已经满足。如果我们需要更多自定义的方法，可以直接基于初始代码进行修改。 [Mybatis Generator 官网](http://www.mybatis.org/generator/index.html)