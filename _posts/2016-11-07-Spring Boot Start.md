[TOC]

1. spring boot兼容maven3.2以及之后的版本

2. pom配置文件中，dependencies必须包涵org.springframework.boot  groupId

3. 需要注意的是pom配置文件需要继承spring-boot-starter-parent项目

pom示例文件：

```

<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"

    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

 

    <groupId>com.example</groupId>

    <artifactId>myproject</artifactId>

    <version>0.0.1-SNAPSHOT</version>

 

    <!-- Inherit defaults from Spring Boot -->

    <parent>

        <groupId>org.springframework.boot</groupId>

        <artifactId>spring-boot-starter-parent</artifactId>

        <version>2.0.0.BUILD-SNAPSHOT</version>

    </parent>

 

    <!-- Add typical dependencies for a web application -->

    <dependencies>

        <dependency>

            <groupId>org.springframework.boot</groupId>

            <artifactId>spring-boot-starter-web</artifactId>

        </dependency>

    </dependencies>

 

    <!-- Package as an executable jar -->

    <build>

        <plugins>

            <plugin>

                <groupId>org.springframework.boot</groupId>

                <artifactId>spring-boot-maven-plugin</artifactId>

            </plugin>

        </plugins>

    </build>

 

    <!-- Add Spring repositories -->

    <!-- (you don't need this if you are using a .RELEASE version) -->

    <repositories>

        <repository>

            <id>spring-snapshots</id>

            <url>http://repo.spring.io/snapshot</url>

            <snapshots><enabled>true</enabled></snapshots>

        </repository>

        <repository>

            <id>spring-milestones</id>

            <url>http://repo.spring.io/milestone</url>

        </repository>

    </repositories>

    <pluginRepositories>

        <pluginRepository>

            <id>spring-snapshots</id>

            <url>http://repo.spring.io/snapshot</url>

        </pluginRepository>

        <pluginRepository>

            <id>spring-milestones</id>

            <url>http://repo.spring.io/milestone</url>

        </pluginRepository>

    </pluginRepositories>

</project>

```

To create an executable jar we need to add the `spring-boot-maven-plugin` to our
`pom.xml`

```

    
    <build>
    
        <plugins>
    
            <plugin>
    
                <groupId>org.springframework.boot</groupId>
    
                <artifactId>spring-boot-maven-plugin</artifactId>
    
            </plugin>
    
        </plugins>
    
    </build>
    

```

##spring热交换

jvm热交换一般是通过替换可以替换的字节码实现的，完整的热交换解决方案可以参考JREBel或者SpringLoaded两个项目，在spring boot中可以使用spring-boot-devtools模块快速重启（quick application restarts）

使用spring-boot-devtools的应用当classpath发生变化时，该模块可以进行快速重启

spring-boot-devtools监视着classpath资源，若发生改变则进行重启，classpath如何改变，不同的ide有不同的判定方式，eclipse在保存一个修改过的文件时会引起classpath更新并触发重启，idea在build工程时会触发一个更新。

spring-boot-devtools依赖应用环境的关闭钩子（shutdown hook），若关闭系统关闭钩子则不能正常使用例如：SpringApplication.setRegisterShutdownHook(false))

###restart技术实现

Spring Boot中restart技术是由两个classloaders实现的，一些静态的不需要改变的文件如第三方的jar包是由一个base classloader加载，我们开发中的代码是由一个restart classloader进行加载当程序重新加载时，restart classload被丢弃，一个新的restart calssloader被创建，这种方式从某种方式加快的应用启动速度，相比较传统的启动应用方式，由于base classloader未改变所以少加载一些文件。如果发现应用速度重启速度不够快可以考虑使用reload技术的框架如JRebel和Spring Loaded。

###spring.devtools.restart.exclude

当一些资源文件变更时我们希望应用不进行自动重启，可以使用spring.devtools.restart.exclude=static/**,public/**来将不需要触发重启的资源告诉spring-boot-devyools，当这些资源文件发生变化时就不会触发应用restart了

###spring.devtools.restart.additional-paths

有些时候一些不在classpath中的文件发生变化时，需要将应用重启，可以通过指定spring.devtools.restart.additional-paths参数实现此功能。就像上面所说的spring.devtools.restart.exclude配置项一样使用。

###spring.devtools.restart.enabled

禁止使用自动重启功能。当不希望restart应用时可以设置系统变量来实现。需要注意的时设置系统变量需要在SpringApplication.run之前，如下所示：

```

public static void main(String[] args) {

    System.setProperty("spring.devtools.restart.enabled", "false");

    SpringApplication.run(MyApp.class, args);

}

```

###spring.devtools.restart.trigger-file

可以使用spring.devtools.restart.trigger-file来指定某个文件发生变化时应用程序restart。当其他文件发生变化应用不会进行restart。

###自定义restart classloader

创建一个META-INF/spring-devtools.properties用于指定base classloader和restart classloader的加载范围，配置文件中包含两种前缀的参数：

1. restart.exclude.

2. restart.include.

其中exclude代表需要通过base classloader加载，include代表需要通过restart classloader加载。

例如：

```

restart.exclude.companycommonlibs=/mycorp-common-[\\w-]+\.jar

restart.include.projectcommon=/mycorp-myproj-[\\w-]+\.jar

```

###restart局限性

restart功能对于标准的ObjectInputStream反序列化支持不是很好，如果需要对数据进行反序列化操作请使用Spring中的ConfigurableObjectInputStream配合Thread.currentThread().getContextClassLoader()来处理。

###restart 全局设置

通过配置spring-boot-devtools.properties的文件来配置全部的devtools。这个店戴文件需要放置到电脑的$HOME路径下。

#Part IV. Spring Boot features

## 23. SpringApplication

`SpringApplication` 提供了一种方便会计的方式启动spring应用。

```

    publicstaticvoid main(String[] args) {
        SpringApplication.run(MySpringConfiguration.class, args);
    }

```

###23.1 Startup failure

如果应用启动失败，建议使用FailureAnalyzers来显示详细的错误信息和解决方式

###23.2 Customizing the Banner

banner 是应用程序启动前打印输出，可以通过在classpath中添加一个名叫banner.txt文件来修改它，或者设置一个banner.location参数来指定像这样的一个文件在哪里，如果文件为特殊的编码格式可以设置banner.charset来制定文件编码格式(参数默认是utf-8)。也可以在calsspath中添加banner.gif或者banner.jpg或者banner.png图片文件来展示到启动界面，这些图片我呢见会被转换成ascii码打印到启动界面上，当然也可以指定一个banner.image.location参数来确定图片位置。

在banner.txt文件中有以下集中占位符：

|Variable                           |               Description|

| -------------------------------|---------------------------|

|${application.version}     |               The version number of your application as declared in MANIFEST.MF. For example Implementation-Version: 1.0 is printed as 1.0.|

|${application.formatted-version}     |      The version number of your application as declared in MANIFEST.MF formatted for display (surrounded with brackets and prefixed with v). For example (v1.0).|

|${spring-boot.version}     |     The Spring Boot version that you are using. For example 2.0.0.BUILD-SNAPSHOT.|

|${spring-boot.formatted-version}     |     The Spring Boot version that you are using formatted for display (surrounded with brackets and prefixed with v). For example (v2.0.0.BUILD-SNAPSHOT).|

|${Ansi.NAME} (or ${AnsiColor.NAME}, ${AnsiBackground.NAME}, ${AnsiStyle.NAME})     |     Where NAME is the name of an ANSI escape code. See AnsiPropertySource for details.|

|${application.title}      |      The title of your application as declared in MANIFEST.MF. For example Implementation-Title: MyApp is printed as MyApp.|

###23.3 Customizing SpringApplication

自定义SpringApplication

```

public static void main(String[] args) {

    SpringApplication app = new SpringApplication(MySpringConfiguration.class);

    app.setBannerMode(Banner.Mode.OFF);

    app.run(args);

}

```

代码中关闭启动banner。

自定义SpringApplication支持xml文件中配置，也支持通过application.properties文件配置（详细请看24章节）

####23.4 Fluent builder API

If you need to build an ApplicationContext hierarchy (multiple contexts with a parent/child relationship), or if you just prefer using a ‘fluent’ builder API, you can use the SpringApplicationBuilder.

 

The SpringApplicationBuilder allows you to chain together multiple method calls, and includes parent and child methods that allow you to create a hierarchy.

 

For example:

```

new SpringApplicationBuilder()

        .sources(Parent.class)

        .child(Application.class)

        .bannerMode(Banner.Mode.OFF)

        .run(args);

 ```

[Note]

There are some restrictions when creating an ApplicationContext hierarchy, e.g. Web components must be contained within the child context, and the same Environment will be used for both parent and child contexts. See the SpringApplicationBuilder Javadoc for full details.

###23.5 Application events and listeners

应用程序的时间和监听机制

###23.6 Web environment

###23.7 Accessing application arguments

访问应用参数args

通过使用ApplicationArguments类来获取应用参数args

```

import org.springframework.boot.*

import org.springframework.beans.factory.annotation.*

import org.springframework.stereotype.*

 

@Component

public class MyBean {

 

    @Autowired

    public MyBean(ApplicationArguments args) {

        boolean debug = args.containsOption("debug");

        List<String> files = args.getNonOptionArgs();

        // if run with "--debug logfile.txt" debug=true, files=["logfile.txt"]

    }

 

}

```

###23.8 Using the ApplicationRunner or CommandLineRunner

使用ApplicationRunner或者CommandLineRunner接口来实现应用程序启动时执行一次的功能。