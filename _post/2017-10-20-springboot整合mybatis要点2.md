[TOC]

mybatis结合druid数据源整合到spring中

#添加依赖

```

    <!--mybatis-spring-starter依赖-->
    <dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.1.1</version>
    </dependency>
    <!--驱动依赖-->
    <dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    </dependency>
    <!--数据源依赖-->
    <dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.0.29</version>
    </dependency>

```
#修改配置文件

这里以修改application.yml文件为例，换成properties文件一样

```

    spring:
      datasource:
        name: springdemo
    url: jdbc:mysql://127.0.0.1:3306/springdemo
    username: ekpapi
    password: ekpapi
    # 使用druid数据源
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: com.mysql.jdbc.Driver
    filters: stat
    maxActive: 20
    initialSize: 1
    maxWait: 60000
    minIdle: 1
    timeBetweenEvictionRunsMillis: 60000
    minEvictableIdleTimeMillis: 300000
    validationQuery: select 'x'
    testWhileIdle: true
    testOnBorrow: false
    testOnReturn: false
    poolPreparedStatements: true
    maxOpenPreparedStatements: 50

```

#设置mapper路径

有两种方式直接配置yml文件中或创建个配置类告诉mybatis搜索包

##创建该类用来告诉mybatis映射mapper在哪里

```

@Configuration
@MapperScan(basePackages = "com.example.demo.mappers")
public class MybatisConfig {

}

```

##使用MapperScannerConfiger查找mapper

```

// MyBatis基础配置

@Configuration

@EnableTransactionManagement

public class MyBatisConfig implements TransactionManagementConfigurer {

    // 由上面配置druid数据源

    @Autowired

    DataSource dataSource;

    // 创建SqlSessionFactory

    @Bean(name = "sqlSessionFactory")

    public SqlSessionFactory sqlSessionFactoryBean() {

        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();

        bean.setDataSource(dataSource);

        bean.setTypeAliasesPackage("tk.mybatis.springboot.model");

        //分页插件

        PageHelper pageHelper = new PageHelper();

        Properties properties = new Properties();

        properties.setProperty("reasonable", "true");

        properties.setProperty("supportMethodsArguments", "true");

        properties.setProperty("returnPageInfo", "check");

        properties.setProperty("params", "count=countSql");

        pageHelper.setProperties(properties);

        //添加插件

        bean.setPlugins(new Interceptor[]{pageHelper});

        //添加XML目录

        ResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();

        try {

            bean.setMapperLocations(resolver.getResources("classpath:mapper/*.xml"));

            return bean.getObject();

        } catch (Exception e) {

            e.printStackTrace();

            throw new RuntimeException(e);

        }

    }

    @Bean

    public SqlSessionTemplate sqlSessionTemplate(SqlSessionFactory sqlSessionFactory) {

        return new SqlSessionTemplate(sqlSessionFactory);

    }

    @Bean

    @Override

    public PlatformTransactionManager annotationDrivenTransactionManager() {

        return new DataSourceTransactionManager(dataSource);

    }

}

@Configuration

// 注意，由于MapperScannerConfigurer执行的比较早，所以必须有下面的注解

@AutoConfigureAfter(MyBatisConfig.class)

public class MyBatisMapperScannerConfig {

    @Bean

    public MapperScannerConfigurer mapperScannerConfigurer() {

        MapperScannerConfigurer mapperScannerConfigurer = new MapperScannerConfigurer();

        mapperScannerConfigurer.setSqlSessionFactoryBeanName("sqlSessionFactory");

        mapperScannerConfigurer.setBasePackage("tk.mybatis.springboot.mapper");

        return mapperScannerConfigurer;

    }

}

```

##yml配置文件设置

使用xml配置文件

在`application.yml`中增加配置：

    mybatis: 

              // 使用配置文件添加下面这个配置，使用注解方式可以不用使用的

        mapperLocations: classpath:mapper/*.xml

        typeAliasesPackage: com.example.demo.mappers

除了上面常见的两项配置，还有：

- 
mybatis.config：mybatis-config.xml配置文件的路径

- 
mybatis.typeHandlersPackage：扫描typeHandlers的包

- 
mybatis.checkConfigLocation：检查配置文件是否存在

- 
mybatis.executorType：设置执行模式（`SIMPLE, REUSE, BATCH`），默认为`SIMPLE`