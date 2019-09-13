[TOC]

# 配置文件

## 1、SpringBoot的外部配置

  Spring Boot允许将配置外部化（externalize） ，这样你就能够在不同的环境下使用相同的代码。你可以使用properties文件，YAML文件，环境变量和命令行参数来外部化配置。使用@Value注解，可以直接将属性值注入到beans中，然后通过Spring的 Environment 抽象或通过 @ConfigurationProperties 绑定到结构化对象来
访问。
Spring Boot设计了一个非常特别的 PropertySource 顺序，以允许对属性值进行合理的覆盖，属性会以如下的顺序进行设值：

1. home目录下的devtools全局设置属性（ ~/.spring-bootdevtools.properties ，如果devtools激活） 。

2. 测试用例上的@TestPropertySource注解。

3. 测试用例上的@SpringBootTest#properties注解。

4. **命令行参数**

5. 来自 SPRING_APPLICATION_JSON 的属性（环境变量或系统属性中内嵌的内联JSON） 。

6. ServletConfig 初始化参数。

7. ServletContext 初始化参数。

8. 来自于 java:comp/env 的JNDI属性。

9. Java系统属性（System.getProperties()） 。

10. 操作系统环境变量。

11. RandomValuePropertySource，只包含 random.* 中的属性。

12. **没有打进jar包的Profile-specific应用属性**（ application-{profile}.properties 和YAML变量） 。

13. 打进jar包中的Profile-specific应用属性（ application-{profile}.properties 和YAML变量） 。

14. 没有打进jar包的应用配置（ application.properties 和YAML变量） 。

15. **打进jar包中的应用配置**（ application.properties 和YAML变量） 。

16. @Configuration 类上的 @PropertySource 注解。

17. 默认属性（使用 SpringApplication.setDefaultProperties 指定） 。
      下面是具体的示例，假设你开发一个使用name属性的 @Component ：

   ```
   import org.springframework.stereotype.*
   import org.springframework.beans.factory.annotation.*
   @Component
   public class MyBean {
   @Value("${name}")
   private String name;
   // ...
   }
   ```

   你可以将一个 application.properties 放到应用的classpath下，为 name 提供一个合适的默认属性值。当在新的环境中运行时，可以在jar包外提供一个 application.properties 覆盖 name 属性。对于一次性的测试，你可以使用特定的命令行开关启动应用（比如， java -jar app.jar --name="Spring" ） 。
   注 SPRING_APPLICATION_JSON 属性可以通过命令行的环境变量设置，例如，在一个UNIX shell中可以这样：
   `$ SPRING_APPLICATION_JSON='{"foo":{"bar":"spam"}}' java -jar myapp.jar`
   本示例中，如果是Spring Environment ，你可以以 foo.bar=spam 结尾；如果在一个系统变量中，可以提供作为 spring.application.json 的JSON字符串：
   `$ java -Dspring.application.json='{"foo":"bar"}' -jar myapp.jar`
   或命令行参数：
   `$ java -jar myapp.jar --spring.application.json='{"foo":"bar"}'`
   或作为一个JNDI变量` java:comp/env/spring.application.json` 。





## 2、配置文件加载顺序

官网：[https://docs.spring.io/spring-boot/docs/2.1.4.RELEASE/reference/htmlsingle/#boot-features-external-config-application-property-files](https://docs.spring.io/spring-boot/docs/2.1.4.RELEASE/reference/htmlsingle/#boot-features-external-config-application-property-files)

springboot 启动会扫描以下位置的`application.properties`或者`application.yml`文件作为Spring boot的默认配置文件 

```
–file:./config/              #也就是resources目录下的config/
–file:./						# 也就是resources根目录
–classpath:/config/			# 也就是java代码类根目录 即java/config/
–classpath:/				# 类路径，也就是java/ 下
```

原文

```
A /config subdirectory of the current directory   --当前项目目录下的/config目录
The current directory--当前项目目录下的
A classpath /config package  -类路径下 /config 目录
The classpath root  --类路径
```

**优先级由高到底**，高优先级的配置会覆盖低优先级的配置；

## 3、自定义属性与加载

我们在使用Spring Boot的时候，通常也需要定义一些自己使用的属性，我们可以如下方式直接定义：

```
com.text.name.name=姓名
com.text.title=Spring Boot标题
```

然后通过`@Value("${属性名}")`注解来加载对应的配置属性，具体如下：

```
@Component
public class BlogProperties {

    @Value("${com.text.name}")
    private String name;
    @Value("${com.text.title}")
    private String title;

    // 省略getter和setter

}
```



## 4、参数间的引用

在`application.properties`中的各个参数之间也可以直接引用来使用，就像下面的设置：

```
web.upload-path=D:/images
spring.resources.static-locations=classpath:file:${web.upload-path}

```

spring.resources.static-locations参数引用了上文中定义的web.upload-path属性

## 5、使用随机数

在一些情况下，有些参数我们需要希望它不是一个固定的值，比如密钥、服务端口等。Spring Boot的属性配置文件中可以通过`${random}`来产生int值、long值或者string字符串，来支持属性的随机值。

```
//随机字符串
my.secret=${random.value}
//随机int
my.number=${random.int}
//随机long
my.bignumber=${random.long}
//10以内的随机数
my.number.less.than.ten=${random.int(10)}
//1024--65535
my.number.in.range=${random.int[1024,65536]}
```

random.int* 语法是 OPEN value (,max) CLOSE ，此处 OPEN，CLOSE 可以是任何字符，并且 value，max 是整数。如果提供 max ，那么 value 是最小值， max 是最大值（不包含在内） 。 

## 6、通过命令行设置属性值

默认情况下， SpringApplication 会将所有命令行配置参数（以'--'开头，比如 --server.port=9000 ） 转化成一个 property ，并将其添加到SpringEnvironment 中。连续的两个减号`--`就是对`application.properties`中的属性值进行赋值的标识。正如以上章节提过的，命令行属性总是优先于其他属性源。

通过命令行来修改属性值固然提供了不错的便利性，但是通过命令行就能更改应用运行的参数，那岂不是很不安全？是的，所以Spring Boot也贴心的提供了屏蔽命令行访问属性的设置，只需要这句设置就能屏蔽：`SpringApplication.setAddCommandLineProperties(false)`。

# 多环境配置

>   SpringApplication 将从以下位置加载 application.properties 文件，并把
> 它们添加到Spring Environment 中：
>
> 1. 当前项目resources目录下的 /config 子目录。
> 2. 当前项目目录resources。
> 3. java类路径classpath下的 /config 包。 
> 4. java类路径classpath根路径（root） 。
>    该列表是按优先级排序的（列表中位置高的路径下定义的属性将覆盖位置低的） 。

我们在开发Spring Boot应用时，通常同一套程序会被应用和安装到几个不同的环境，比如：开发、测试、生产等。其中每个环境的数据库地址、服务器端口等等配置都会不同，如果在为不同环境打包时都要频繁修改配置文件的话，那必将是个非常繁琐且容易发生错误的事。

对于多环境的配置，各种项目构建工具或是框架的基本思路是一致的，通过配置多份不同环境的配置文件，再通过打包命令指定需要打包的内容之后进行区分打包，Spring Boot也不例外，或者说更加简单。

在Spring Boot中多环境配置文件名需要满足`application-{profile}.properties`的格式，其中`{profile}`对应你的环境标识，比如：

- `application-dev.properties`：开发环境
- `application-test.properties`：测试环境
- `application-prod.properties`：生产环境

至于哪个具体的配置文件会被加载，需要在`application.properties`文件中通过`spring.profiles.active`属性来设置，其值对应`{profile}`值。

如：`spring.profiles.active=test`就会加载`application-test.properties`配置文件内容

下面，以不同环境配置不同的服务端口为例，进行样例实验。

- 针对各环境新建不同的配置文件`application-dev.properties`、`application-test.properties`、`application-prod.properties`
- 在这三个文件均都设置不同的`server.port`属性，如：dev环境设置为1111，test环境设置为2222，prod环境设置为3333
- application.properties中设置`spring.profiles.active=dev`，就是说默认以dev环境设置
- 测试不同配置的加载
  - 执行`java -jar xxx.jar`，可以观察到服务端口被设置为`1111`，也就是默认的开发环境（dev）
  - 执行`java -jar xxx.jar --spring.profiles.active=test`，可以观察到服务端口被设置为`2222`，也就是测试环境的配置（test）
  - 执行`java -jar xxx.jar --spring.profiles.active=prod`，可以观察到服务端口被设置为`3333`，也就是生产环境的配置（prod）

按照上面的实验，可以如下总结多环境的配置思路：

- `application.properties`中配置通用内容，并设置`spring.profiles.active=dev`，以开发环境为默认配置
- `application-{profile}.properties`中配置各个环境不同的内容
- 通过命令行方式去激活不同环境的配置

**原理：配置文件加载顺序和外部属性加载顺序，见最上方，我们需要关注的几个点,优先级：**

1、命令行参数指定的属性

2、没有打进jar包的属性优先于打进jar包的属性

3、`application-{profile}.properties `配置文件优先于 `application.properties`

