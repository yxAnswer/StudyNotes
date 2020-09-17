# Springboot使用配置文件的方式来整合mybatis
[TOC]

> 这里先不详细讲解mybatis 的使用，只是介绍springboot 如何使用xml方式来进行操作，因为复杂的操作springboot已经替我们做了，如果我们还是写复杂了，那就不用springboot了。

# springboot 整合mytais 步骤 --配置篇

1、添加mybatis 依赖

2、数据库配置

3、mapper的配置文件

4、编写mapper类和映射文件

## 1、添加mytatis依赖

```xml
  <!-- 引入第三方数据源 -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.6</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
        </dependency>
```

## 2、数据库配置

最简单的直接在application.properties配置文件中声明 数据库的地址 用户名密码等，还可以设置其他连接属性，这里就不说了

```
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.url=jdbc:mysql://localhost:3306/movie?useUnicode=true&characterEncoding=utf-8&serverTimezone=Asia/Shanghai
spring.datasource.username=root
spring.datasource.password=112233
```

以上配置好就可以连接数据库进行操作了

## 3、Mapper 的配置文件

xml方式使用mybatis，需要在application.properties中声明两个配置属性,例如：

```
#mybatis
mybatis.mapper-locations= classpath:mybatis/mapper/**/*.xml
mybatis.config-location= classpath:mybatis/mybatis-config.xml
```

`mybatis.mapper-locations` 表示mapper类映射的xml的位置，不然你写的那些sql都找不到，怎么使用

`mybatis.config-location`表示mybatis的配置文件

以上配置如下结构：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019040211305694.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

简单的目录结构是这样的。和之前我们用注解的方式没什么区别，springboot使用配置文件只是service层或者是mapper层不一样。

首先，我们创建了UserMapper,这个接口类，里面声明了一些操作数据库的方法，然后再我们配置文件中`mybatis.mapper-locations`指定的位置创建相应的xml文件，然后里面编写 sql语句操作数据，进行映射。

`mybatis-config.xml` 是配置mybatis的配置文件，比如：

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
		PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
		"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	<settings>
		<setting name="mapUnderscoreToCamelCase" value="true"/>
		<!-- 打印查询语句 -->
		<setting name="logImpl" value="STDOUT_LOGGING" />
	</settings>
</configuration>
```

这个可以去官方文档复制，然后改一改里面的<settings> 就好，上面的配置表示的是，mybatis 忽略大小写，避免驼峰命名法和数据库字段大小写不一致。

## 4、编写mapper类和映射xml文件

比如UserMapper.class ,一个最简单的方法

```
public interface UserMapper {

    /**
     * 查询所有用户
     * @return
     */
    public List<User> getAll();

}
```

然后，编写相应的xml映射文件UserMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.mybatis.mapper.UserMapper">
    
    <select id="getAll" resultType="com.mybatis.domain.User">
   		 select * from user
    </select>
</mapper>
```

这里只是一个简单的实例，使用起来也很简单。

`namespace` 表示当前xml是映射的那个mapper类，后面要跟上**全类名**

然后是增删改查，<insert>,<delete>,<update>,<select>

使用方法都差不多，就是先声明这个操作标签，比如 

```
<select id="getAll" resultType="com.mybatis.domain.User">
   		 //id 表示mapper类中的方法，getAll()方法
   		 // resultType 是这个方法的返回值类型，返回对象或者集合时，表明类的全类名。
</select>
```

最后再 <select> 标签中编写sql语句，

```
 select * from user  //这是一个简单的实例 
 
 当然还有很多复杂的，和注解方式一样，也是用#{}，#{} 的这种方式去一一对应数据库和实体类
 
```

## 5、添加@mapper注解或者使用@mapperScan的方式，让springboot扫描到mapper类

这一步操作什么时候都行，这里放到最后重点提醒，

```
@SpringBootApplication
@MapperScan("com.mybatis.mapper")
public class MainApplication {
    public static void main(String[] args) {
        SpringApplication.run(MainApplication.class,args);
    }
}
```

比如，在启动类里面，加入@mapperScan注解，指明所有的mapper，这样只写一次就可以了。

## 小结：

本篇只是记录了下springboot的xml使用方式，

重点：

1、application.properties 配置文件中声明位置

```
#mybatis
mybatis.mapper-locations= classpath:mybatis/mapper/**/*.xml
mybatis.config-location= classpath:mybatis/mybatis-config.xml
```

2、把原来卸载mapper类中的注解 sql语句，写到 xml的配置文件中

3、`namespace`  `select`等标签的意思要熟悉，    至于映射的全类名和 方法名就可以了

详细的内容由于时间原因，不想写了，因为官方文档都有，忘记了就去官方文档找

[http://www.mybatis.org/mybatis-3/zh/sqlmap-xml.html](http://www.mybatis.org/mybatis-3/zh/sqlmap-xml.html)