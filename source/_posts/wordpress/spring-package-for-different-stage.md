---
title: Maven项目不同环境打包不同配置
tags:
  - 不同环境打包
  - 多配置
id: '207'
categories:
  - - Spring
date: 2016-05-05 22:01:08
---

_同一项目在开发、测试、线上环境下分别使用不同配置，如何打包_ 基于项目[Spring Web](https://www.zmannotes.com/index.php/2016/04/16/spring-basic-web/)进行开发，支持为不同环境打包功能 **1\. 创建文件夹src/main/config** **2\. 创建测试环境配置文件test.properties，内容如下**

env=test

**3\. 创建线上环境配置文件prod.properties，内容如下**

env=prod

**4\. 修改pom.xml文件，添加如下内容**

<profiles>
    <profile>
        <id>test</id>
        <build>
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-antrun-plugin</artifactId>
                    <version>1.7</version>
                    <executions>
                        <execution>
                            <phase>compile</phase>
                            <goals>
                                <goal>run</goal>
                            </goals>
                            <configuration>
                                <target>
                                    <echo message="copy test.properties to ${project.build.directory}/classes/"/>
                                    <copy file="src/main/config/test.properties"
                                          tofile="${project.build.directory}/classes/application.properties"
                                          overwrite="true"/>
                                </target>
                            </configuration>
                        </execution>
                    </executions>
                </plugin>
            </plugins>
        </build>
    </profile>
    <profile>
        <id>prod</id>
        <build>
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-antrun-plugin</artifactId>
                    <version>1.7</version>
                    <executions>
                        <execution>
                            <phase>compile</phase>
                            <goals>
                                <goal>run</goal>
                            </goals>
                            <configuration>
                                <target name="prod-copying">
                                    <echo message="copy  prod.properties to /classes/"/>
                                    <copy file="src/main/config/prod.properties"
                                          tofile="${project.build.directory}/classes/application.properties"
                                          overwrite="true"/>
                                </target>
                            </configuration>
                        </execution>
                    </executions>
                </plugin>
            </plugins>
        </build>
    </profile>
</profiles>

**5\. 分别为开发、测试、线上环境打包** 5.1 为开发环境打包 `mvn clean package` 5.2 为测试环境打包 `mvn clean package -Ptest` 5.3 为线上环境打包 `mvn clean package -Pprod` [源码Github](https://github.com/zman2013/spring-multiple-config)