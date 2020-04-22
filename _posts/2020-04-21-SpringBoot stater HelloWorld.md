---
layout:     post
title:      SpringBoot-Stater 定义
categories:   [SpringBoot]
description:  SpringBoot-Stater 定义
keywords:     SpringBoot, starter
author:     tang
topmost: false    
---

> 在SpringBoot 项目中有很多模块都是由starter模块组成的，为实现一些特定的功能，我们可以自定义Spring-Boot-Stater来实现特定功能。
# 引入配置
开始之前创建maven项目，导入创建SpringStater需要配置：

```java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.6.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.xy</groupId>
    <artifactId>simple-spring-boot-starter</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>simple-spring-boot-starter</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-autoconfigure</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
        </plugins>
        <resources>
            <resource>
                <directory>src/main/resources</directory>
            </resource>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/
                    *.xml</include>
                </includes>
            </resource>
        </resources>
    </build>
</project>

```

>spring-boot-configuration-processor 引入这个包主要为了生成metadata.json,方便在yml文件中引入配置时提供代码提示。


Spring官方对starter的命名有一定的规范，例如spring 官方定义的包命名为：<font color='#ff3333'>spring-boot-starter-{name}</font>,如果是非官方版本的stater定义为：<font color='#ff3333'>{name}-spring-boot-starter</font>。

# 实现步骤
1.在<font color='#ff3333'>simple-spring-boot-starter</font>项目中定义需要实现的功能模块，定义简单的主体方法，<font color='#ff3333'>helloWorld(String word)</font>实现字符串的拼接。

```java
public class ExampleService {

    private String prefix;

    private String suffix;

    public ExampleService(String prefix, String suffix){
        this.prefix = prefix;
        this.suffix = suffix;
    }

    public String helloWorld(String word){
        return prefix + suffix + word;
    }
}

```

2.在SpringBoot中通过<font color=''#ff3333''>@ConfigurationProperties</font>注解通过前缀+后缀批注去除yml中配置文件内容。

```java
@ConfigurationProperties("example.service")
public class ExampleServiceProperties {
    private String prefix;

    private String suffix;

    public String getPrefix() {
        return prefix;
    }

    public String getSuffix() {
        return suffix;
    }

    public void setSuffix(String suffix) {
        this.suffix = suffix;
    }

    public void setPrefix(String prefix) {
        this.prefix = prefix;
    }
}
```

3.最重要的是定义<font color='#ff3333'>AutoConfiguration</font>,在simple-spring-boot-stater项目中定义ExampleAutoConfigure 实现自动注入。

* @Configuration 配置类注解
* @ConditionalOnClass(ExampleService.class) : 当在classpath下 有ExampleService 这个类时才进行加载。
* @EnableConfigurationProperties(ExampleServiceProperties.class) // 开启允许ExampleServiceProperties获取yml配置

```java

@Configuration
@ConditionalOnClass(ExampleService.class)
@EnableConfigurationProperties(ExampleServiceProperties.class)
public class ExampleAutoConfigure {
    private final ExampleServiceProperties properties;

    @Autowired
    public ExampleAutoConfigure(ExampleServiceProperties serviceProperties){
        this.properties = serviceProperties;
    }

    @Bean
    @ConditionalOnMissingBean  
    @ConditionalOnProperty(prefix = "example.service", value = "enabled", havingValue = "true")
    ExampleService exampleService(){
        return new ExampleService(properties.getPrefix(), properties.getSuffix());
    }
}

```

> 在定义bean方法会使用到一些相关注解

* <font color='#ff3333'>@ConditionalOnMissingBean</font> :  表示Spring context 中不存在该bean
* <font color='#ff3333'>@ConditionalOnProperty(prefix = "example.service", value = "enabled", havingValue = "true")</font> : example.service.enabled : true, 这种情况才会加载这个Bean


> springBoot 中相关注解： @Conditional 相关注解

| 条件化注解   | 配置生效条件   |  备注  |
| :----:   | :-----  | :----:  |
| @ConditionalOnBean   | 配置了某个特定的Bean |        |
| @ConditionalOnMissingBean        |   Spring Context中没有配置特定的Bean   |      |
| @ConditionalOnClass     |    classPath下有指定的类    |    |
| @ConditionalOnMissingClass     |    classPath下没有指定的类    |    |
| @ConditionalOnExpression     |    在给定的Spring Expression Language 表达式计算结果为true    |    |
| @ConditionalOnJava     |    java的版本匹配特定指或者一个范围值    |    |
| @ConditionalOnProperty     |    制定配置属性要有一个明确的值    |    |
| @ConditionalOnResource     |    classPath下有指定的资源    |    |
| @ConditionalOnWebApplication     |    指定是一个web应用程序    |    |
| @ConditionalOnNotWebApplication     |    指定不是一个web应用程序    |    |


4.在resouces 环境下创建<font color='#ff3333'>META-INF</font>目录，然后在创建<font color='#ff3333'>spring.factories</font>文件：

```java
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.xy.stater.config.ExampleAutoConfigure
```
> spring Boot 项目启动时会到这个目录下spring.factories文件中查找org.springframework.boot.autoconfigure.EnableAutoConfiguration这个配置下的文件，装载到spring容器中。


# 测试
1.新建项目依赖这个项目进行测试

> 引入maven项目

```java
<dependency>
  <groupId>com.xy</groupId>
  <artifactId>simple-spring-boot-starter</artifactId>
  <version>0.0.1-SNAPSHOT</version>
</dependency>

```

2.在application.properties 中添加配置

```java
example.service.prefix=nihao
example.service.suffix=world
example.service.enabled=true
```

3.在spring测试勒种调用ExampleService中的方法

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SimpleStaterTest {

  @Resource
  private ExampleService exampleService;

  @Test
  public void test(){
    String how = exampleService.helloWorld("how");
    System.out.println(how);
  }
}
```

4.方法执行结果

```
nihaoworldhow
```

# github地址
[simple-spring-boot-stater][1]

 [1]: https://github.com/tangpingl/simple-spring-boot-starter       "simple-spring-boot-stater"
