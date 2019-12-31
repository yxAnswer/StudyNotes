# SpringBoot整合Mybatis配置多数据源

[TOC]

> 当我们遇到需要访问多个数据库，或者做读写分离的时候，就需要去配置多个数据源。springboot 通过注解配置的方式就可以 通过mybatis 配置多数据库

## 1、添加mybatis依赖

```properties
<dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>1.3.2</version>
</dependency>
```

首先是springboot项目，依赖mybatis 。环境不多说，主要看配置。

## 2、修改配置文件-双数据源

springboot的思想是配置大于依赖，我们要使用多个数据库需要在`application.properties`配置文件中添加数据源信息

```properties
server.port=8081
#mybatis
#mybatis.mapper-locations=classpath:mybatis/mapper/**/*.xml
#这个用于xml方式配置，暂时不用
#mybatis.config-location=classpath:mybatis/mybatis-config.xml
#下划线转驼峰
mybatis.configuration.map-underscore-to-camel-case=true 
#增加打印sql语句，一般用于本地开发测试
mybatis.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl

# 数据库

#mysql
spring.datasource-primary.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource-primary.driver-class-name =com.mysql.cj.jdbc.Driver
spring.datasource-primary.jdbc-url=jdbc:mysql://localhost:3306/videos_server?useUnicode=true&characterEncoding=utf-8&serverTimezone=Asia/Shanghai
spring.datasource-primary.username=sa
spring.datasource-primary.password=112233

#sql server
spring.datasource-second.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource-second.driver-class-name=com.microsoft.sqlserver.jdbc.SQLServerDriver
spring.datasource-second.jdbc-url=jdbc:sqlserver://localhost:1433;database=demo
spring.datasource-second.username=sa
spring.datasource-second.password=112233

```

上面是我测试项目的配置文件，主要看 我标注的mysql 和sql server 这两部分。跟我们一般配置数据库信息不同的是，前缀不同，`spring.datasource-primary` 和`spring.database-second` 。之所以要这样写是要区分多个数据源，未接下来配置mybatis准备。

当我们在配置读写分离的时候，可以设置一个主库写，多个库只读。

## 3、构建新的数据源并配置SqlSessionFactory

由于我们配置多数据源，原来springboot默认替我们做的事情现在需要我们手动指定数据源了。

先看一下我的目录

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190726142449627.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

这里是这个demo的整个目录。为了方便配置，我创建了一个config文件夹，下面创建了三个数据库配置文件。`DataSourceConfig`    `PrimarySourceConfig`   `SecondSourceConfig`  。

DataSourceConfig:

```java
package com.source.test.config;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import javax.sql.DataSource;
/**
 * 作者    ANSWER
 * 时间    2019/7/24 22:53
 * 文件    DataBaseConfig
 * 描述   配置数据源
 */
@Configuration
public class DataSourceConfig {
    /*
     * 前缀为spring.datasource-primary  作为命名为 primary的数据源
     */
    @Bean(name = "primary")
    @ConfigurationProperties(prefix = "spring.datasource-primary")
    public DataSource primaryDataSource() {
        return DataSourceBuilder.create().build();
    }
    /*
    前缀为spring.datasource-second  作为命名为 second的数据源
     */
    @Bean(name = "second")
    @ConfigurationProperties(prefix = "spring.datasource-second")
    public DataSource secondDataSource(){
        return  DataSourceBuilder.create().build();
    }
}

```

我们创建DataSourceConfig 类，添加`@Configuration`注解作为配置类。我们创建了两个数据源，指明了name  和使用配置文件的前缀，会通过前缀读取我们实际的数据库信息。

数据源生成以后，我们需要配置SqlSessionFactory ，分别使用不同的数据源。

PrimarySourceConfig:

```java
package com.source.test.config;

import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.SqlSessionTemplate;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import javax.sql.DataSource;

/**
 * 作者    ANSWER
 * 时间    2019/7/24 23:04
 * 文件    PrimarySourceConfig
 * 描述
 */
@Configuration
@MapperScan(basePackages = {"com.source.test.mapper.primary"}, sqlSessionFactoryRef = "primaryFactory")
public class PrimarySourceConfig {

    public static final String MAPPER_LOCATION="classpath:mybatis/mapper/primary/*.xml";
    @Autowired
    @Qualifier("primary")
    private DataSource primarySource;

    @Bean
    @Primary
    public SqlSessionFactory primaryFactory() throws Exception {
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
        factoryBean.setDataSource(primarySource);
        factoryBean.setMapperLocations(new PathMatchingResourcePatternResolver()
                .getResources(MAPPER_LOCATION));
        return factoryBean.getObject();
    }
}
```

SecondSourceConfig:

```java
package com.source.test.config;

import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;

import javax.sql.DataSource;
/**
 * 作者    ANSWER
 * 时间    2019/7/24 23:04
 * 文件    SecondSourceConfig
 * 描述
 */

@Configuration
@MapperScan(basePackages = {"com.source.test.mapper.second"}, sqlSessionFactoryRef = "secondFactory")
public class SecondSourceConfig {

    public static final String MAPPER_LOCATION = "classpath:mybatis/mapper/second/*.xml";
    @Autowired
    @Qualifier("second")
    private DataSource secondSource;

    @Bean
    public SqlSessionFactory secondFactory() throws Exception {
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
        factoryBean.setDataSource(secondSource);
        factoryBean.setMapperLocations(new PathMatchingResourcePatternResolver()
                .getResources(MAPPER_LOCATION));
        return factoryBean.getObject();
    }

}
```

两个配置类几乎一致，主要是通过`@MapperScan` 注解指定此数据源读取扫描的mapper路径。然后通过`@Autowired`和`@Qualifier` 指定当前使用的数据源，我们创建数据源的时候已经声明了name,就是自己写的那个。

非常关键的一步就是需要配置SqlSessionFactory 使Mybatis读取不同数据库。

注意：如果你使用注解的方式写sql ,那么久不需要指定xml文件的地址了。 如果你是使用xml的方式写sql,那就得配置xml的映射路径，否则会找不到你mapper对应的sql。

这里分别指向了，primary   的mapper对应的xml路径 以及 second对应的路径。 通过`factoryBean.setMapperLocations(new PathMatchingResourcePatternResolver()
                .getResources(MAPPER_LOCATION));` 将mapper_location设置。

上面两个配置类，很关键的一点区别就是，我们需要制定一个是 主库，也就是我们再 primary库中多加了一个注解`@Primary`  不然的话启动就会报错。当然还有另一种方式：

比如PrimarySourceConfig ,添加：

```java
@Bean
public SqlSessionTemplate sqlSessionTemplate1() throws Exception {
  SqlSessionTemplate template = new SqlSessionTemplate(primaryFactory()); 
    // 使用上面配置的primaryFactory
  return template;
}
```

SecondSourceConfig同样如此。  添加了这两个bean ，不用写@Primary 也可以，省事的话就直接用@Primary更快。

## 4、完成

配置多数据源其实就这么简单，就是 先配置好数据库信息，编制好前缀。然后通过前缀创建多数据源DataSource。 然后使用这个DataSource 分别配置不同的SqlSessionFactory。 指定mybatis  的xml的映射路径。那么接下来就直接测试成不成功了。

编写controller  ，不同的mapper，访问不同的数据库，这个就不贴代码了。自己试试就ok 了。

