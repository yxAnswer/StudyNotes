# 配置文件介绍

[TOC]

## 一、springboot 配置文件分类
- .application.properties
- .application.yml

`YAML Ain't Markup Language`  yaml不是一种标记语言
springboot 有两种配置文件，可以是`properties`,也可以是`yml`，各有各的好处。本人打算使用properties方式，纯属个人爱好
对比下：
```
1)YAML（Yet Another Markup Language）
				写 YAML 要比写 XML 快得多(无需关注标签或引号)
				使用空格 Space 缩进表示分层，不同层次之间的缩进可以使用不同的空格数目
				注意：key后面的冒号，后面一定要跟一个空格,树状结构
			application.properties示例
				server.port=8090  
				server.session-timeout=30  
				server.tomcat.max-threads=0  
				server.tomcat.uri-encoding=UTF-8 

			application.yml示例
				server:  
	  				port: 8090  
	  				session-timeout: 30  
	  				tomcat.max-threads: 0  
	  				tomcat.uri-encoding: UTF-8 

```
## 二、最简单的方式，官网直接复制，需要用啥就复制啥，然后改改

```
https://docs.spring.io/spring-boot/docs/2.1.0.BUILD-SNAPSHOT/reference/htmlsingle/#common-application-properties
```
### 1、 properties这样写
`server.port=8080 `
### 2、YAML这样写
**在yml中缩进一定不能使用TAB，否则会报很奇怪的错误；（缩进特么只能用空格！！！！)**

**每个k的冒号后面一定都要加一个空格；**
```
server:
	port: 8081
	path: /hello
```
 k:(空格)v：表示一对键值对（空格必须有）；
以空格的缩进来控制层级关系；只要是左对齐的一列数据，都是同一个层级的-----属性和值也是大小写敏感


 **值的写法:**
**（1）字面量：普通的值（数字，字符串，布尔）**
k: v：字面直接来写；
字符串默认不用加上单引号或者双引号；
""：双引号；不会转义字符串里面的特殊字符；特殊字符会作为本身想表示的意思
name: "zhangsan \n lisi"：输出；zhangsan 换行 lisi
''：单引号；会转义特殊字符，特殊字符最终只是一个普通的字符串数据
name: ‘zhangsan \n lisi’：输出；zhangsan \n lisi
**（2）对象、Map（属性和值）（键值对）：**

k: v：在下一行来写对象的属性和值的关系；注意缩进
对象还是k: v的方式
```
friends:
	lastName: zhangsan
	age: 20
```
行内写法
```
friends: {lastName: zhangsan,age: 18}
```
**（3）数组（List、Set）**
用- 值表示数组中的一个元素
```
pets:
‐ cat
‐ dog
‐ pig
```
行内写法
`pets: [cat,dog,pig]`

### 3、配置文件值注入
配置文件如下：例如
```
person:
	lastName: hello
	age: 18
	boss: false
	birth: 2017/12/12
	maps: {k1: v1,k2: 12}
	lists:
		‐ zhoumao
		‐ zhangyue
	dog:
	name: 德华
	age: 12
```
javaBean：

```
/**
* 将配置文件中配置的每一个属性的值，映射到这个组件中
* @ConfigurationProperties：告诉SpringBoot将本类中的所有属性和配置文件中相关的配置进行绑定；
* prefix = "person"：配置文件中哪个下面的所有属性进行一一映射
* *
只有这个组件是容器中的组件，才能容器提供的@ConfigurationProperties功能；
* *
/
@Component
@ConfigurationProperties(prefix = "person")
public class Person {
private String lastName;
private Integer age;
private Boolean boss;
private Date birth;
private Map<String,Object> maps;
private List<Object> lists;
private Dog dog;
```
以上配置文件的注入以对象的方式，只需要在javaBean 上添加两个注解，`@Component`标记类  然后`@ConfigurationProperties(prefix = "person")`指定前缀   然后只要javaBean的属性和配置文件一致就可以直接映射。

使用时，需要在使用的类上指定配置文件`@PropertySource({"classpath:application.yml"})`

**小结**：可以使用yaml也可以使用properties方式，基本语法也差不太多，习惯用哪个用哪个，官网有例子，使用的时候直接复制。
所有的配置其实都会被注入成map,都是键值对的方式，取的时候都是根据key 获取value.
所以获取配置文件的方式暂时会一种就可以。

## 三、配置文件自动映射属性和实体类
配置文件加载有两种方式，一种直接在Controller类中添加注解，以value的形式直接使用配置文件。另一种，是注入实体类的方式。

### 第一种：Controller类加载配置信息
就拿我们文件的例子来看 application.proterties文件如下

```

web.upload-path=D:/ideWorkspace/StudyProject/HelloWorld/src/main/resources/static/images
spring.resources.static-locations=classpath:/META-INF/resources/,classpath:/resources/,classpath:/static/,classpath:/public/,file:${web.upload-path} 

```
这里我们定义了一个文件路径，当然也可以是其他的。只要是key-value形式定义就可以。
然后，需要在使用配置文件的类中添加注解。
（1）类上添加`@PropertySource({"classpath:application.properties"})`
（2）变量名使用

```
@Value("${web.upload-path}")
private String filePath;
```
这样就可以了。其实就是通过@PropertySource注解将配置文件路径添加进来，这样spring 就会扫描到这个配置文件，并且映射成map到这个类中，接下来通过@Value注解，以key-value的形式，通过定义的key拿到配置文件中的值。这样就避免了硬编码。（以上是在有@Controller 注解的类中使用）

### 第二种  注入配置实体类
接下来用个实例讲解下。
首先在配置文件中配置了两个属性，比如服务器的地址和名称

```
test.domain=www.answerme.xyz
test.name=springboot
```
如果要把他们映射到实体类，需要建一个javaBean 比如：ServerSetting.java
```
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.PropertySource;
import org.springframework.stereotype.Component;

//服务器配置
@Component
@PropertySource({"classpath:application.properties"})
@ConfigurationProperties
public class ServerSettings {
	//名称
	@Value("${test.name}")
	private String name;
	@Value("${test.domain}")
	private String domain;

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public String getDomain() {
		return domain;
	}
	public void setDomain(String domain) {
		this.domain = domain;
	}
	
}
```
然后写一个controller方法测试下：
自动注入
```
    @Autowired
    ServerSetting serverSetting;

    @RequestMapping("/v1/server")
    public Object  serverTest(){

        return serverSetting;
    }
```
结果如下：`{
"name": "springboot",
"domain": "www.answerme.xyz"
}`
说明，这种方式可以获取到配置文件的信息。
当然每次都要将属性和配置文件的key一一对应有点麻烦，所以有更方便的写法。
我们可以再@ConfigurationProperties() 注解中添加一个前缀，代码如下

```
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.PropertySource;
import org.springframework.stereotype.Component;
/**
 * 文件名: ServerSetting.java
 * 描述：
 * Create by Google on 2018/9/10 23:45
 */

//服务器配置
@Component
@PropertySource({"classpath:application.properties"})
@ConfigurationProperties(prefix = "test")
public class ServerSetting {

    private String name;
    private String domain;
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }

    public String getDomain() {
        return domain;
    }

    public void setDomain(String domain) {
        this.domain = domain;
    }

}
```

	常见问题：
				1、配置文件注入失败，Could not resolve placeholder
					解决：根据springboot启动流程，会有自动扫描包没有扫描到相关注解, 
					默认Spring框架实现会从声明@ComponentScan所在的类的package进行扫描，来自动注入，
					因此启动类最好放在根路径下面，或者指定扫描包范围
					spring-boot扫描启动类对应的目录和子目录
				2、注入bean的方式，属性名称和配置文件里面的key一一对应，就不用加@Value 这个注解
					如果不一样，就要加@value("${XXX}")

小结：

			1、添加 @Component或者Configuration 注解，后者标记为配置文件；	
			2、使用 @PropertySource 注解指定配置文件位置；(属性名称规范: 大模块.子模块.属性名)
			3、必须 通过注入IOC对象Resource 进来 ， 才能在类中使用获取的配置文件值。
				@Autowired
	    		private WeChatConfig weChatConfig;
	    		例子：
	    			@Configuration
			@PropertySource(value="classpath:application.properties")
					public class WeChatConfig {
	
						@Value("${wxpay.appid}")
						private String appId;