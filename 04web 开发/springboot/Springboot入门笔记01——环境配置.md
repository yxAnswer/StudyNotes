# Springboot 简介
[TOC]

>简化Spring应用开发的一个框架；
>整个Spring技术栈的一个大整合；
>J2EE开发的一站式解决方案；

## 1、SpringBoot2.x依赖环境和版本新特性说明
>1、依赖版本jdk8以上, Springboot2.x用JDK8, 因为底层是 Spring framework5, 
>	2、安装maven最新版本，maven3.2以上版本，下载地址 ：https://maven.apache.org/download.cgi
>	3、springbootGitHub地址：https://github.com/spring-projects/spring-boot
>	4、springboot官方文档：https://spring.io/guides/gs/spring-boot/
>我自己使用的是 jdk8、maven 3.5.3、 开发工具为 Intellij Idea 

## 2、基础环境搭建和maven、idea的使用（从头搭建一个简单的web项目）
### 2.1、安装jdk 1.8，配置环境变量
### 2.2、安装maven 3.5.3 ,配置环境变量
### 2.3、idea的相关配置，比如jdk，maven等（ 这些就不讲了）
### 2.4、创建一个简单的maven 管理的web 项目
#### (1)创建一个maven 项目，File——New——Project

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181109131915920.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

#### (2)定义项目的坐标，然后一直next 到finish
groupId：项目名称，定义为组织名+项目名，一般用公司的域名反写，这个按照一定规范些就行
artifactid :这个一般是项目名或者模块的名称
version:当前项目或者moudle 的版本号
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181109132011546.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181109132036493.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)
#### (3)maven 项目结构说明
使用maven创建的工程我们称他们为maven 工程，maven 工程具有一定的目录规范，如下：
src/main/java—— 存放项目的.java 文件
src/main/resources —— 存放项目资源文件，如spring，mybatis配置文件
src/test/java —— 存放所有单元测试的.java 文件，如 JUnit测试类
src/test/resources —— 测试用的资源文件
target —— 项目输出位置，编译后的.class 文件会输出到此目录
pom.xml —— maven 项目的核心配置文件 
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190401113327341.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)
#### (4)为刚才创建的项目添加web 支持，因为刚才我们创建的是一个普通的maven 项目，如果想要被外部访问，需要添加web支持。
快捷键 ctrl+alt+shift+s   或者file ——project structure打开 项目的设置面板
左边选择moudle, 然后点击+号，添加web 支持

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181109132249554.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181109132305872.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)
根据自己项目实际情况，其实就是指定到src \main\webapp下
需要修改资源文件路径为  D:\ideWorkspace\practice\src\main\webapp
修改描述文件路径为 D:\ideWorkspace\practice\src\main\webapp\WEB-INF\web.xml
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181109132428100.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)
可以看到现在下面有个警告，我们还没有添加Artifacts，可以直接点击右边的创建按钮，也可以再左侧点击Artifacts进行添加
如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181109132407722.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)
选择自己刚才创建的，点击ok 就添加完成了。
**这样一个web项目就创建完成了，在这里我们不再演示创建一个servlet 和一个静态页面，去访问资源了。由于我们是springboot 学习，接下来就要直接上springboot了，以上是对maven 项目的一个简单介绍以及 idea 的使用。 （传统web项目以及tomcat、servlet 等可以自行百度）**

## 3、springboot 项目的环境搭建以及包的依赖
可以参考官方的例子，这里只简单说下步骤
[https://spring.io/guides/gs/rest-service/](https://spring.io/guides/gs/rest-service/)
修改pom.xml文件 添加相关依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.gongsi.test</groupId>
    <artifactId>practice</artifactId>
    <version>1.0-SNAPSHOT</version>


    <!--依赖父工程，使用springboot必须加-->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.3.RELEASE</version>
    </parent>

    <!--添加相关依赖-->
    <dependencies>
        <!--使用Spring MVC构建Web（包括RESTful）应用程序的入门者。使用Tomcat作为默认嵌入式容器-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--使用JUnit，Hamcrest和Mockito等库来测试Spring Boot应用程序的-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>com.jayway.jsonpath</groupId>
            <artifactId>json-path</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <properties>
        <java.version>1.8</java.version>
    </properties>


    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

    <repositories>
        <repository>
            <id>spring-releases</id>
            <url>https://repo.spring.io/libs-release</url>
        </repository>
    </repositories>
    <pluginRepositories>
        <pluginRepository>
            <id>spring-releases</id>
            <url>https://repo.spring.io/libs-release</url>
        </pluginRepository>
    </pluginRepositories>
</project>
```
引入依赖后，工程就已经是一个springboot的项目了，可以写相关代码，用内置的tomcat运行

**关于maven 库的依赖**
[http://mvnrepository.com/](http://mvnrepository.com/)
可以从这个仓库进行搜索，然后把相关的xml依赖 添加到 pom.xml下的  dependencies 标签下

**更快的创建springboot项目的方法**
[https://start.spring.io/](https://start.spring.io/)
相当于是个在线的插件，输入相关参数，下载配置好的springboot 工程
**更更方便的创建springboot项目的方法**
使用idea 已经内置的创建插件
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181109132451182.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)
点击创建，然后选择 spring initializr 然后一步步填写相关信息，进行创建。

```xml
       <!-- 仅仅是记录一下，好找 -->
  <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.16</version>
        </dependency>

        <!-- 引入第三方数据源 -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.6</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.mybatis.spring.boot/mybatis-spring-boot-starter -->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>1.3.2</version>
        </dependency>
        <dependency>
            <groupId>com.microsoft.sqlserver</groupId>
            <artifactId>mssql-jdbc</artifactId>
            <version>7.0.0.jre8</version>
            <scope>runtime</scope>
        </dependency>
        <!--pagehelper-->
        <dependency>
            <groupId>com.github.pagehelper</groupId>
            <artifactId>pagehelper-spring-boot-starter</artifactId>
            <version>1.2.3</version>
        </dependency>
```

