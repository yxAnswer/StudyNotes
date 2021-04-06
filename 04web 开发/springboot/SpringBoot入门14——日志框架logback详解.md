[TOC]

# 日志框架logback的整合

## 1、LogBack介绍

java常用处理java的日志组件：slf4j,log4j,logback,common-logging 等

- logback介绍：基于Log4j基础上大量改良，不能单独使用，推荐配合日志框架**SLF4J**来使用
- **logback当前分成三个模块**：`logback-core`,`logback-classic`和`logback-access`;`logback-core`是其它两个模块的基础模块
- Logback的核心对象：

​		Logger：日志记录器
​		Appender：指定日志输出的目的地，目的地可以是控制台，文件
​		Layout：日志布局 格式化日志信息的输出

- 日志级别：DEBUG < INFO < WARN < ERROR
  
- 支持 将log4J 的properties 文件转为logback 配置文件xml

```properties
	===========log4j示例===========		
	 ### 设置###
	log4j.rootLogger = debug,stdout,D,E

	### 输出信息到控制抬 ###
	log4j.appender.stdout = org.apache.log4j.ConsoleAppender
	log4j.appender.stdout.Target = System.out
	log4j.appender.stdout.layout = org.apache.log4j.PatternLayout
	log4j.appender.stdout.layout.ConversionPattern = [%-5p] %d{yyyy-MM-dd HH:mm:ss,SSS} method:%l%n%m%n

	### 输出DEBUG 级别以上的日志到=D://logs/error.log ###
	log4j.appender.D = org.apache.log4j.DailyRollingFileAppender
	log4j.appender.D.File = D://logs/log.log
	log4j.appender.D.Append = true
	log4j.appender.D.Threshold = DEBUG 
	log4j.appender.D.layout = org.apache.log4j.PatternLayout
	log4j.appender.D.layout.ConversionPattern = %-d{yyyy-MM-dd HH:mm:ss}  [ %t:%r ] - [ %p ]  %m%n

	### 输出ERROR 级别以上的日志到=D://logs/error.log ###
	log4j.appender.E = org.apache.log4j.DailyRollingFileAppender
	log4j.appender.E.File =E://logs/error.log 
	log4j.appender.E.Append = true
	log4j.appender.E.Threshold = ERROR 
	log4j.appender.E.layout = org.apache.log4j.PatternLayout
	log4j.appender.E.layout.ConversionPattern = %-d{yyyy-MM-dd HH:mm:ss}  [ %t:%r ] - [ %p ]  %m%n 

	===========logback============
```

**Log4j日志转换为logback在线工具**（支持log4j.properties转换为logback.xml,不支持 log4j.xml转换为logback.xml）

[https://logback.qos.ch/translator/](https://logback.qos.ch/translator/)

## 2、对日志进行配置

**官网**：[https://docs.spring.io/spring-boot/docs/2.1.0.BUILD-SNAPSHOT/reference/htmlsingle/#boot-features-logging](https://docs.spring.io/spring-boot/docs/2.1.0.BUILD-SNAPSHOT/reference/htmlsingle/#boot-features-logging)

直接看官网，根据不同版本都有详细的配置说明

## 3、自定义Logback配置

**官网**：[https://docs.spring.io/spring-boot/docs/2.1.0.BUILD-SNAPSHOT/reference/htmlsingle/#boot-features-logging](https://docs.spring.io/spring-boot/docs/2.1.0.BUILD-SNAPSHOT/reference/htmlsingle/#boot-features-logging)

**各个组件案例**：[https://logback.qos.ch/manual/index.html](https://logback.qos.ch/manual/index.html)

springboot 默认使用logback 日志框架，启动springboot在控制台格式化输出的配置就是出自logback。

在默认情况下：springboot将日志输出到控制台

**如何自定义logback配置？**

### 3.1 创建日志配置文件logback-spring.xml

官方强烈建议，日志文件以`-spring.xml`结尾。默认加载加载配置顺序 `logback-spring.xml`,` logback-spring.groovy`, `logback.xml`, `or logback.groovy`

我们再我们springboot项目的 resource 文件夹下，创建一个文件名为logback-spring.xml的日志配置文件。

文件内容如下：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<configuration>

	 <appender name="consoleApp" class="ch.qos.logback.core.ConsoleAppender">
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>
                %date{yyyy-MM-dd HH:mm:ss.SSS}  %highlight(%-5level)  %boldMagenta([%thread])  %boldYellow(%logger{56}.%method:%L)  -%msg%n
            </pattern>
        </layout>
    </appender>

    <appender name="fileInfoApp" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
             <level>ERROR</level>
            <onMatch>DENY</onMatch>
            <onMismatch>ACCEPT</onMismatch>
        </filter>
        <encoder>
            <pattern>
                %date{yyyy-MM-dd HH:mm:ss.SSS}       %-5level   [%thread]    %logger{56}.%method:%L   -[ %msg ]%n
            </pattern>
        </encoder>
        <!-- 滚动策略 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 路径 -->
            <fileNamePattern>D:/tomcat-tmp/log/app.info.%d.log</fileNamePattern>
            <maxHistory>30</maxHistory>
            <totalSizeCap>2GB</totalSizeCap>
        </rollingPolicy>
    </appender>

    <appender name="fileErrorApp" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>ERROR</level>
        </filter>
        <encoder>
            <pattern>
                %date{yyyy-MM-dd HH:mm:ss.SSS}       %-5level   [  %thread]----  %logger{56}.%method:%L -[ %msg ]%n
            </pattern>
        </encoder>
        
        <!-- 设置滚动策略 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 路径 -->
            <fileNamePattern>D:/tomcat-tmp/log/app.err.%d.log</fileNamePattern>
            
            <!-- 控制保留的归档文件的最大数量，超出数量就删除旧文件，假设设置每个月滚动，
            且<maxHistory> 是1，则只保存最近1个月的文件，删除之前的旧文件 -->
            <maxHistory>30</maxHistory>
            <totalSizeCap>2GB</totalSizeCap>
            
        </rollingPolicy>
    </appender>
   <root level="INFO">  
        <appender-ref ref="consoleApp"/>
        <appender-ref ref="fileInfoApp"/>
        <appender-ref ref="fileErrorApp"/>
    </root>
</configuration>
```



上面是我们配置好的一个配置文件。总体来说配置文件有这么几个节点

```xml
<configuration> 子节点
<appender></appender>   					
<logger></logger>
<root></root> (要加在最后) 用来定义我们日志级别输出到哪一个appender
```

上面我们配置了三个 appender ,分别输出到控制台， 文件(info级别)，文件(error级别)

具体还可以看官网：[https://logback.qos.ch/manual/index.html](https://logback.qos.ch/manual/index.html)

还可以配置日志颜色：https://logback.qos.ch/manual/layouts.html#coloring

总之官网

用到哪个就去查，复制粘贴。

比如：filter 过滤

```xml

<filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>ERROR</level>  过滤error级别
            <onMatch>DENY</onMatch> 匹配的时候 不显示
            <onMismatch>ACCEPT</onMismatch> 不匹配的时候加进去
           <!--最终结果是，不显示error 级别的日志-->
</filter>


<filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>ERROR</level>
         <!--最终结果，只显示error级别-->
</filter>
```

比如 格式化log format   

```xml
<pattern>
   %date{yyyy-MM-dd HH:mm:ss.SSS} %-5level[%thread]%logger{56}.%method:%L -%msg%n
</pattern>
```

这些了解就好，我们只需要知道想配置，去哪里去找，达到目的就好。

### 3.2 打印测试配置是否成功

```java
private Logger logger =LoggerFactory.getLogger(this.getClass());
logger.debug("打印debug日志");
logger.info("打印info日志");
logger.warn("打印warn日志");
logger.error("打印error日志");
```





