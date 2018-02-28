[TOC]

#一、添加依赖

注意依赖版本号version

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

                <version>5.7.1</version>
    </dependency>
    <!--数据源依赖，当然数据源也可以用其他数据库线程池-->
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>1.0.24</version>
    </dependency>

```

#二、创建配置类
配置类告诉Springboot搜索mapper位置和数据源连接信息

```
@Configuration
//@MapperScan是mybatis-spring中提供的一个注解,指定Mapper类所在的包
@MapperScan(basePackages = {"com.demo.springboot.mybatis.mappers"})
public class DatabaseConfig {
        //我们只需要配置一个数据源即可，mybatis-spring-boot-starter会自动帮我们创建SqlSessionFactory和SqlSessionTemplate
    @Bean
    public DataSource druidDataSource(){
        DruidDataSource dataSource=new DruidDataSource();
        dataSource.setUsername("root");
        dataSource.setPassword("password");
        dataSource.setUrl("jdbc:mysql://localhost:3306/zebra");
        dataSource.setDriverClassName("com.mysql.jdbc.Driver");
        // 还可以设置一些其他的信息
        // .......
        return dataSource;
    }
}
```

#三、在mapper包内创建mapper类

这里是用注解方式使用mybatis

```

/**
 * 使用了SpringBoot之后，我们一般不再将sql写在xml映射文件，而是通过相关注解直接写在方法上
 * 所有支持的注解都位于org.apache.ibatis.annotations包中
 */

public interface UserMapper{
    @Insert("INSERT INTO user (id,name) VALUES(#{id},#{name})") 

    @Options(useGeneratedKeys=true, keyProperty="id",keyColumn = "id")
    public int insert(User user);
 
    @Update("UPDATE user SET name=#{name} WHERE id=#{id}")
    public int update(User user);
 
    @Select("SELECT * FROM user")
    @Results({
               @Result(id = true, column = "id", property = "id"),
               @Result(column = "name", property = "name"),
            })
    public List<User> selectAll();
 
    @Delete("DELETE FROM user WHERE id=#{id}")
    public int delete(int id);

}

```