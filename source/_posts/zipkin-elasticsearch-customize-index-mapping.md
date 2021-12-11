---
title: zipkin elasticsearch customize index mapping
date: 2021-12-11 14:39:59
tags:
---

zipkin 默认不对 tags 进行 mapping，这样无法在 kibana 中对 tags.xx 进行检索和建立图标

# 下载 zipkin 源码
`git clone https://github.com/openzipkin/zipkin.git`

# 修改 zipkin 源码
修改 [VersionSpecificTemplates.spanIndexTemplate](https://github.com/openzipkin/zipkin/blob/e89c86a3e930df4a3a77f4b1d060ed11579bdaf4/zipkin-storage/elasticsearch/src/main/java/zipkin2/elasticsearch/VersionSpecificTemplates.java#L114)   
在 mappings -> properties 添加如下代码:
```java
+ "      \"tags.messaging.message_id\": " + KEYWORD + ",\n"
+ "      \"tags.source.uri\": " + KEYWORD + ",\n"
+ "      \"tags.messaging.destination\": " + KEYWORD + ",\n"
```

# 编译 zipkin server
```shell
# Build the server and also make its dependencies
$ ./mvnw -q --batch-mode -DskipTests --also-make -pl zipkin-server clean install
# Run the server
$ java -jar ./zipkin-server/target/zipkin-server-*exec.jar
```

# 停止 zipkin 服务
# 删除 ES 中已保存的 Template
`DELETE _template/{your prefix}-span_template`

# 启动 zipkin 服务