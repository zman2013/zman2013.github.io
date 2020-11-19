---
title: RestTemplate详细配置
tags:
  - HttpClient
  - RestTemplate
id: '221'
categories:
  - - HttpClient
  - - Spring
date: 2016-05-08 21:15:34
---

**1\. 配置文件如下**

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 声明连接池，指定大小 -->
    <bean id="pollingConnectionManager"
        class="org.apache.http.impl.conn.PoolingHttpClientConnectionManager">
        <property name="maxTotal" value="${resttemplate.http.connection.max\_total}" />
        <property name="defaultMaxPerRoute" value="${resttemplate.http.default\_max\_per\_route}" />
    </bean>

    <!-- 声明http client工厂 -->
    <bean id="httpClientBuilder" class="org.apache.http.impl.client.HttpClientBuilder"
        factory-method="create">
        <property name="connectionManager" ref="pollingConnectionManager" />
    </bean>

    <!-- 声明http client -->
    <bean id="httpClient" factory-bean="httpClientBuilder"
        factory-method="build" />

    <!-- 声明http rquest工厂，指定超时时间 -->
    <bean id="clientHttpRequestFactory"
        class="org.springframework.http.client.HttpComponentsClientHttpRequestFactory">
        <constructor-arg ref="httpClient" />
        <property name="connectTimeout" value="${resttemplate.http.connect\_timeout}" />
        <property name="readTimeout" value="${resttemplate.http.read\_timeout}" />
    </bean>
    
    <!-- 声明converter，指定编码 -->
    <bean id="utf8TextConverter" class="org.springframework.http.converter.StringHttpMessageConverter">
            <property name="supportedMediaTypes">
                <list>
                    <value>text/html;charset=UTF-8</value>
                </list>
            </property>
        </bean>
    
    <!-- 最终的RestTemplate -->
    <bean id="restTemplate" class="org.springframework.web.client.RestTemplate">
        <constructor-arg ref="clientHttpRequestFactory" />
        <property name="messageConverters">
            <list>
                <ref bean="utf8TextConverter" />
            </list>
        </property>
    </bean>
</beans>

 **2. 封装RestTemplate**

public class HttpClientWrapper {

    protected Logger logger = LoggerFactory.getLogger(HttpClientWrapper.class);

    @Autowired
    protected RestTemplate restTemplate;

    protected ObjectMapper mapper = Jackson2ObjectMapperBuilder.json().build();

    /\*\*
     \* 请求指定url，并获得返回对象
     \* @param url http://google.com?key1=value1&key2=value2...
     \* @param method GET
     \* @param tClass XXXResponse.class
     \* @return XXXResponse instance
     \*/
    public<T> T request(String url, HttpMethod method, Class<T> tClass) {
        JavaType returnType = TypeFactory.defaultInstance().constructType(
                tClass);
        String content = this.requestForString(url, method, null, null);
        if (content != null) {
            try {
                T object = mapper.readValue(content, returnType);
                return object;
            } catch (JsonParseException e) {
                logger.error("JSON parse exception: \\n");
                logger.error("Content: " + content);
                logger.error(e.getMessage(), e);
            } catch (JsonMappingException e) {
                logger.error("JSON mapping exception: \\n");
                logger.error("Content: " + content);
                logger.error(e.getMessage(), e);
            } catch (IOException e) {
                logger.error(e.getMessage(), e);
            }
        } else {
            logger.error("请求返回空, method:{}, url:{}", method, url);
        }

        return null;

    }

    /\*\*
     \* 记录Request、Response内容和请求耗时
     \*/
    private String requestForString(String url, HttpMethod method,
            Map<String, Object> pathVariables, HttpEntity<String> httpEntity) {
        ResponseEntity<String> response = null;

        if (httpEntity == null) {
            httpEntity = new HttpEntity<String>("");
        }

        this.logRequest(url, method, pathVariables, httpEntity);
        long start = System.currentTimeMillis();
        try {
            if (pathVariables == null) {
                response = restTemplate.exchange(url, method, httpEntity,
                        String.class);
            } else {
                response = restTemplate.exchange(url, method, httpEntity,
                        String.class, pathVariables);
            }
        } catch (Exception e) {
            logger.error(e.getMessage(), e);
            return null;
        }
        long end = System.currentTimeMillis();
        logger.info("Time: " + (end - start));

        if (response.getStatusCode() != HttpStatus.OK) {
            logger.error("Http response is NOT 200, URL: " + method + " " + url
                    + "; HTTP status: " + response.getStatusCode());
            return null;
        }

        String content = response.getBody();

        return content;
    }

    protected void logRequest(String url, HttpMethod method,
            Map<String, Object> pathVariables, HttpEntity<String> httpEntity) {

        if (pathVariables != null) {
            URI expanded = new UriTemplate(url).expand(pathVariables);
            url = expanded.toString();
        }

        logger.info("Http Invoke: " + method + " " + url);
        if (!StringUtils.isEmpty(httpEntity.getBody())) {
            logger.info("Body: " + httpEntity.toString());
        }

    }