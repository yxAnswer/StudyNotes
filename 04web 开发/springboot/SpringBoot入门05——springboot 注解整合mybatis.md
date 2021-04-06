# springboot 注解整合mybatis
[TOC]

## springboot 持久化数据方式介绍

> 1、原始java访问数据库
> 		开发流程麻烦
> 		（1）注册驱动/加载驱动
> 			Class.forName("com.mysql.jdbc.Driver")
> 		（2）建立连接
> 			Connection con = DriverManager.getConnection("jdbc:mysql://localhost:3306/dbname","root","root");
> 		（3）创建Statement
> 		（4）执行SQL语句
> 		（5）处理结果集
> 		  (6)关闭连接，释放资源
> 	2、apache dbutils框架
> 		比上一步简单点
> 		官网:https://commons.apache.org/proper/commons-dbutils/
> 	3、jpa框架
> 		spring-data-jpa
> 		jpa在复杂查询的时候性能不是很好
> 	4、Hiberante   解释：ORM：对象关系映射Object Relational Mapping
> 		企业大都喜欢使用hibernate
> 	5、Mybatis框架   
> 		互联网行业通常使用mybatis
> 		不提供对象和关系模型的直接映射,半ORM

## springboot 集成mybatis 的步骤
1、`pom.xml`添加依赖
2、配置文件配置 jdbc  数据库连接地址、以及数据源等等
3、分包进行分层，编码结构清晰
4、创建表和相应的pojo类
5、实现mapper或者叫dao 层的功能编码，sql注入
6、实现service层的编码，调用dao层实现操作数据库
7、实现controller 层的代码，编写外部调用接口，内部调用service层方法处理mapper层的数据
8、添加`@MapperScan`注解，运行程序，调试bug,可能会修改配置比如mysql serverTimezone的影响，测试

### 1、首先来添加mybatis的相关依赖

```xml
			<!-- 引入starter-->
			<!-- https://mvnrepository.com/artifact/org.mybatis.spring.boot/mybatis-spring-boot-starter -->
            <dependency>
                <groupId>org.mybatis.spring.boot</groupId>
                <artifactId>mybatis-spring-boot-starter</artifactId>
                <version>1.3.2</version>
            </dependency>

		 			
		 	<!-- MySQL的JDBC驱动包	-->	
		 			<dependency>
						<groupId>mysql</groupId>
						<artifactId>mysql-connector-java</artifactId>
						<scope>runtime</scope>
					</dependency> 
			<!-- 引入第三方数据源 -->		
					<dependency>
						<groupId>com.alibaba</groupId>
						<artifactId>druid</artifactId>
						<version>1.1.6</version>
					</dependency>
```
### 2、修改配置文件application.properties
```
#可以自动识别,所以下面这行可写可不写
#spring.datasource.driver-class-name =com.mysql.jdbc.Driver
spring.datasource.driver-class-name =com.mysql.cj.jdbc.Driver
#数据库的连接地址，用户名，密码
spring.datasource.url=jdbc:mysql://localhost:3306/movie?useUnicode=true&characterEncoding=utf-8&serverTimezone=Asia/Shanghai
spring.datasource.username =root
spring.datasource.password =112233
#数据源
#如果不使用默认的数据源 （com.zaxxer.hikari.HikariDataSource）
spring.datasource.type =com.alibaba.druid.pool.DruidDataSource

```
这里简单讲解下配置文件：
1、首先，声明jdbc的驱动这个可以写也可以不写，springboot 会自动识别当前的jdbc,这里仅供参考
2、配置好数据库的连接地址：修改为自己的本地或者远程数据库，这里注意，如果你的表里有time字段，最好设置下serverTimezone ，不设置的话 可能会有错误。。然后是数据库用户名和密码
3、数据源：这个可以改也可以不改，默认的是`com.zaxxer.hikari.HikariDataSource`；但是如果想改的话也可以改为淘宝的这个。
### 3、分包
一般写代码都要进行分层，这里最起码我们简单分一下包
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190401161001101.png)
一个简单的项目，打开分为这么几个包，
controller：主要方我们的api接口  
domain:放一些实体类，同javaBean 、pojo 都可以，是我们的module层。 
mapper:是存放所有访问数据库的接口，同dao.  
service :业务逻辑层，我们的业务代码都放在这，service 又可以分为 接口 和实现类 两个包进行分类
utils:是我们的工具类
大家可以根据自己的习惯进行分一下包，然后就开始创建相关类了。
### 4、创建表和相应的pojo类
创建表，就打开数据库创建就行了。不管用什么方法，图形界面，命令行都可以。我们这里创建了一个movie数据库，一个user表。 有 id 、name 、create_time、age、phone   这几个字段
可以这样建 ` mysql -u root -p `  密码 连接数据

```
create database movie;
use movie;
create table user(
	id int(32) primary key auto_increment,
	name varchar(32) ,
	age  int(32),
	phone varchar(32),
	create_time  datetime
);
```
然后创建相应的javaBean User,字段类型一定要和表相对应
### 5、实现mapper或者叫dao 层的功能编码，sql注入
在mapper包下创建 UserMapper 类，主要来操作User这个表

```java
package com.mybatis.mapper;
import com.mybatis.domain.User;
import org.apache.ibatis.annotations.Insert;
import org.apache.ibatis.annotations.Options;
/**
 * 访问数据库的接口
 */
public interface UserMapper   {
    /**
     * @param user
     * @return 注意： 这里的#{}内容必须和User 类的属性一一对应；推荐使用#{}取值，不要用${},因为存在注入的风险
     */
    @Insert("INSERT INTO USER(name,phone,create_time,age) VALUES(#{name},#{phone},#{createTime},#{age})")
    @Options(useGeneratedKeys = true, keyProperty = "id", keyColumn = "id")
    int insert(User user);


}
```
`@Insert()`注解可以将sql 映射user类，然后将数据写入user表   sql语句 values() 值以 #{属性名} 进行一一匹配
`@Options `配置，主键id是自增长的，我们如果要想把id映射到user类中，需要表明使用主键true，属性名和列名
`useGeneratedKeys=true `表示使用主键，true就可以返回。  keyProperty java对象的属性；keyColumn表示数据库的字段
mapper方法写完后，就可以写service 层方法进行调用
### 6、实现service层的编码，调用dao层实现操作数据库
一般先创建一个UserService 的接口，声明操作数据库的方法，比如需要有增删改查的方法，就在接口中声明。
然后创建一个impl 包，用来存放具体实现service的实现类 。比如现在创建一个` UserServiceImpl.java` 类。
这两个类的代码如下：

```java
package com.mybatis.service;
import com.mybatis.domain.User;
public interface UserService {
    public int add(User user);
    //其他方法。
    //删
    //改
    //查
}

```


UserServiceImpl ：
```java
package com.mybatis.service.impl;


import com.mybatis.domain.User;
import com.mybatis.mapper.UserMapper;
import com.mybatis.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class UserServiceImpl implements UserService {


    @Autowired
    private UserMapper userMapper;
    @Override
    public int add(User user) {
        userMapper.insert(user);
        int id = user.getId();
        return id;
    }
}

```
`UserServiceImpl `首先实现了 UserService 接口，重写了操作方法
有两个注解 @Service 和 @Autowired   通过注解声明，springmvc就能扫描到这个类，`@Autowired`封装了自动装配`UserMapper ` 。调用 mapper层的`insert()` 方法进行注入，还可以拿到id 

### 7、实现controller 层的代码，编写外部调用接口，内部调用service层方法处理mapper层的数据

```java
package com.mybatis.controller;

import com.mybatis.domain.JsonData;
import com.mybatis.domain.User;
import com.mybatis.service.impl.UserServiceImpl;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.Date;

@RestController
@RequestMapping("/api/v1/user")
public class UserController {
    
    @Autowired
    private UserServiceImpl userService;
    
    /**
     * 功能描述: user 保存接口
     *
     * @return
     */
    @GetMapping("add")
    public Object add() {

        User user = new User();
        user.setAge(22);
        user.setCreateTime(new Date());
        user.setName("我最帅");
        user.setPhone("17615881722");
        int id = userService.add(user);
        return JsonData.buildSuccess(id);
    }

}
```
和前面一样，`@RestController    `
`@RequestMapping`指定路径
方法get post根据情况自己定，这里就简单的将一个user对象写入user 表， 至于结果返回 自己定吧。

### 8、添加@MapperScan注解，运行主程序，进行测试，调用接口，查看数据库表。
**必须在主程序中添加注解@MapperScan("com.mybatis.mapper")**
告诉程序要扫描mapper包，不然写的dao层的代码无法找到

```java
package com.mybatis;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
@MapperScan("com.mybatis.mapper")
public class ActionApplication  {

    public  static void main(String[] args){
        SpringApplication.run(ActionApplication.class,args);
    }
}

```