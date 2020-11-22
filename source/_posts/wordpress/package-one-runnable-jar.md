---
title: Package One Runnable Jar
tags: []
id: '339'
categories:
  - - Java
date: 2020-01-12 22:09:37
---

# 1. Add Maven Plugin
```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-assembly-plugin</artifactId>
            <executions>
                <execution>
                    <phase>package</phase>
                    <goals>
                        <goal>single</goal>
                    </goals>
                    <configuration>
                        <archive>
                            <manifest>
                                <mainClass>
                                    carport.ClientMain
                                </mainClass>
                            </manifest>
                        </archive>
                        <descriptorRefs>
                            <descriptorRef>jar-with-dependencies</descriptorRef>
                        </descriptorRefs>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

#  2. Assembly Command

`mvn clean && mvn package`

#  3. Run the Application

`java -jar xxx.jar`