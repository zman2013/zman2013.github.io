---
title: Spring项目单元测试
tags:
  - Mock
  - Mockito
  - Sprint Test
  - Unit Test
id: '215'
categories:
  - - Mock
  - - Spring
  - - Unit Test
date: 2016-05-08 20:12:38
---

_JUnit与Spring Test、Mockito配合使用_ **1\. 项目依赖**

<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <scope>test</scope>
    <version>4.2.5.RELEASE</version>
</dependency>
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
</dependency>

**2\. 创建测试类**

@RunWith(SpringJUnit4ClassRunner.class)
//指定配置文件
@ContextConfiguration(locations = {"/spring/stock-session-factory.xml"})
//指定资源文件
@TestPropertySource(locations = {"/application.properties"})
public class StockTest {

    @Autowired
    private StockMapper stockMapper;

    @Transactional
    @Test
    public void test(){
        Stock stock = new Stock();
        stock.setCode("900001");
        stock.setName("tcl");
        stockMapper.insert(stock);
    }

}

**3\. Mock** 3.1 加入依赖[Mockito](http://mockito.org/)

<!-- mock -->
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-all</artifactId>
    <version>1.10.19</version>
    <scope>test</scope>
</dependency>

3.2 Mock数据库访问操作

@RunWith(SpringJUnit4ClassRunner.class)
//指定配置文件
@ContextConfiguration(locations = {"/spring/stock-session-factory.xml"})
//指定资源文件
@TestPropertySource(locations = {"/application.properties"})
public class StockTest {

    @InjectMocks
    @Autowired
    private StockService stockService;
    @Mock
    private StockMapper stockMapperMocker;

    @Before
    public void setup(){
        MockitoAnnotations.initMocks(this);
    }

    @Transactional
    @Test
    public void test(){
        Stock stock = new Stock();
        stock.setName("露露");
        //如果是数据库插入操作，则直接跳过，返回1
        when(stockMapperMocker.insert(any(Stock.class))).thenReturn(1);
        //如果是数据库查询操作，则直接返回stock对象
    when(stockMapperMocker.selectByPrimaryKey(anyString())).thenReturn(stock);

        stockService.insert(null);
        String name = stockService.selectByCode("000848");
        System.out.println( name );
    }

}

项目源码 [Github](https://github.com/zman2013/spring-unit-test)