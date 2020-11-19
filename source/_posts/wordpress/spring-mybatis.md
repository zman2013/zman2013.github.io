---
title: Spring+Mybatis
tags:
  - Spring Mybatis
id: '200'
categories:
  - - mybatis
  - - Spring
date: 2016-05-04 22:34:05
---

_基于之前的spring项目，引入mybatis作为数据层基本框架_ **1\. 引入mybatis依赖**

<properties>
    <mybatis.version>3.3.0</mybatis.version>
    <mybatis-spring.version>1.2.3</mybatis-spring.version>
</properties>

<!-- mybatis -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>${mybatis-spring.version}</version>
</dependency>
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>${mybatis.version}</version>
</dependency>

 **2. 创建Domain类** 2.1 表结构

CREATE TABLE \`stock\` (
  \`code\` varchar(10) NOT NULL,
  \`name\` varchar(10) NOT NULL,
  \`count\` int(11) DEFAULT NULL,
  \`main\_business\` varchar(256) DEFAULT NULL,
  \`create\_time\` timestamp NOT NULL DEFAULT CURRENT\_TIMESTAMP,
  \`price\` decimal(10,2) DEFAULT NULL,
  PRIMARY KEY (\`code\`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4

2.2 Stock.java

package com.zmannotes.spring.mybatis.domain;

public class Stock {

    private String code;

    private String name;

    private Integer count;

    private String mainBusiness;

    private Date createTime;

    private BigDecimal price;

//setter and getter
//...
}

 **3. 创建Dao(Mapper)**

package com.zmannotes.spring.mybatis.dao;

import com.zmannotes.spring.mybatis.domain.Stock;

public interface StockMapper {
    //根据PK删除
    int deleteByPrimaryKey(String code);
    //插入新纪录
    int insert(Stock record);
    //根据PK查询
    Stock selectByPrimaryKey(String code);
}

 **4. 配置映射文件**

//通过配置文件将 数据库表 与 对象、Dao关联起来
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.zmannotes.spring.mybatis.dao.StockMapper" > <!--指定Dao-->
  <resultMap id="BaseResultMap" type="com.zmannotes.spring.mybatis.domain.Stock" > <!--指定Domain类-->
    <id column="code" property="code" jdbcType="VARCHAR" />
    <result column="name" property="name" jdbcType="VARCHAR" />
    <result column="count" property="count" jdbcType="INTEGER" />
    <result column="main\_business" property="mainBusiness" jdbcType="VARCHAR" />
    <result column="create\_time" property="createTime" jdbcType="TIMESTAMP" />
    <result column="price" property="price" jdbcType="DECIMAL" />
  </resultMap>
  <sql id="Base\_Column\_List" >
    code, name, count, main\_business, create\_time, price
  </sql>
  <select id="selectByPrimaryKey" resultMap="BaseResultMap" parameterType="java.lang.String" >
    select 
    <include refid="Base\_Column\_List" />
    from stock
    where code = #{code,jdbcType=VARCHAR}
  </select>
  <delete id="deleteByPrimaryKey" parameterType="java.lang.String" >
    delete from stock
    where code = #{code,jdbcType=VARCHAR}
  </delete>
  <insert id="insert" parameterType="com.zmannotes.spring.mybatis.domain.Stock" >
    insert into stock (code, name, count, 
      main\_business, create\_time, price
      )
    values (#{code,jdbcType=VARCHAR}, #{name,jdbcType=VARCHAR}, #{count,jdbcType=INTEGER}, 
      #{mainBusiness,jdbcType=VARCHAR}, #{createTime,jdbcType=TIMESTAMP}, #{price,jdbcType=DECIMAL}
      )
  </insert>
</xml>

**5\. 配置数据源及SessionFactory**

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/tx
        http://www.springframework.org/schema/tx/spring-tx.xsd">
    <!-- dbcp2 数据源 -->
    <bean id="stock\_dataSource" class="org.apache.commons.dbcp2.BasicDataSource"
          destroy-method="close">
        <property name="driverClassName" value="com.mysql.jdbc.Driver" />
        <property name="url" value="jdbc:mysql://123.57.144.1:3306/stock" />
        <property name="username" value="stock" />
        <property name="password" value="stock" />
        <property name="connectionInitSqls">
            <list>
                <value>set names utf8mb4;</value>
            </list>
        </property>
    </bean>

    <!-- 事务管理器 -->
    <bean id="stock\_txManager"
        class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="stock\_dataSource" />
    </bean>

    <tx:annotation-driven transaction-manager="stock\_txManager"
        mode="proxy" proxy-target-class="true" />

    <!-- 在使用mybatis时 spring使用sqlsessionFactoryBean 来管理mybatis的sqlsessionFactory -->
    <!-- 创建sqlsessionFactory 并指定数据源 -->
    <bean id="stock\_sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="stock\_dataSource" />
        <property name="typeAliasesPackage"
            value="com.zmannotes.spring.mybatis.domain" />
        <!-- 配置扫描Mapper XML的位置 -->
        <property name="mapperLocations">
            <list>
                <value>classpath:mybatis/stock/\*.xml</value>
            </list>
        </property>
    </bean>

    <!-- 使用MapperScannerConfiguer 扫描来实现 -->
    <!-- 这里指定了要扫描的映射接口的路径 -->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage">
            <value>
                com.zmannotes.spring.mybatis.dao
            </value>
        </property>
        <property name="sqlSessionFactoryBeanName" value="stock\_sqlSessionFactory" />
    </bean>

</beans>

**6\. 完成单测**

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "/spring/stock-session-factory.xml")
public class StockTest {
    @Autowired
    private StockMapper stockMapper;

    /\*\*插入一条数据并自动回滚\*/
    @Transactional
    @Test
    public void test(){
        Stock stock = new Stock();
        stock.setCode("900001");
        stock.setName("tcl");
        stockMapper.insert(stock);
    }

}

[源码Github](https://github.com/zman2013/spring-mybatis) **Q&A** Q：无聊的2~4步有没有快捷完成办法？ A：当然有！那就是Mybatis Generator，自动生成2~4所有代码。参考 [LINK](https://www.zmannotes.com/index.php/2016/05/05/mybatis-generator/)