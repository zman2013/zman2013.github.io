---
title: Spring HTTP Request & Response Log
tags: []
id: '324'
categories:
  - - log
  - - Spring
date: 2019-10-28 21:13:55
---

# 1. Controller Log

## 1.1 确认是Spring Boot项目

## 1.2 创建Filter
```java
import org.springframework.web.filter.CommonsRequestLoggingFilter;

import javax.servlet.http.HttpServletRequest;

public class ApiAccessFilter extends CommonsRequestLoggingFilter {


    @Override
    protected void beforeRequest(HttpServletRequest request, String message) {
        super.beforeRequest(request, "recevied http request");
    }

    @Override
    protected String createMessage(HttpServletRequest request, String prefix, String suffix) {
        return request.getMethod() + " " + super.createMessage(request, prefix, suffix);
    }

}
```
##  1.3 Java Configuration
```java
@Bean
public CommonsRequestLoggingFilter logFilter() {
    CommonsRequestLoggingFilter filter = new ApiAccessFilter();
    filter.setIncludeQueryString(true);
    filter.setIncludePayload(true);
    filter.setIncludeClientInfo(false);
    filter.setMaxPayloadLength(1000);
    filter.setIncludeHeaders(true);
    filter.setAfterMessagePrefix("");
    return filter;
}
```
##  1.4 设置logback配置
```xml
<logger name="com.chehejia.iot.batch.advice.ApiAccessFilter" additivity="false">
    <level value="DEBUG"/>
</logger>
```

# LINK

1.  [https://www.baeldung.com/spring-http-logging](https://www.baeldung.com/spring-http-logging)