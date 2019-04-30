[TOC]



# SpringBoot 热部署实战



spring为开发者提供了一个名为spring-boot-devtools的模块来使Spring Boot应用支持热部署，提高开发者的开发效率，无需手动重启Spring Boot应用。

## 1.热部署的原理

深层原理是使用了两个ClassLoader，一个Classloader加载那些不会改变的类（第三方Jar包），另一个ClassLoader加载会更改的类，称为restart ClassLoader,这样在有代码更改的时候，原来的restart ClassLoader 被丢弃，重新创建一个restart ClassLoader，由于需要加载的类相比较少，所以实现了较快的重启时间。

## 2.如何使用devtool

springboot 使用devtool特别简单，首先看官网的地址

[https://docs.spring.io/spring-boot/docs/2.1.0.BUILD-SNAPSHOT/reference/htmlsingle/#using-boot-devtools](https://docs.spring.io/spring-boot/docs/2.1.0.BUILD-SNAPSHOT/reference/htmlsingle/#using-boot-devtools) 

在项目的pom.xml文件下添加依赖，

```
//核心依赖包
<dependency>  
		  <groupId>org.springframework.boot</groupId>  
		  <artifactId>spring-boot-devtools</artifactId>  
		  <optional>true</optional>  
</dependency>
```

由于DevTools监视类路径资源，因此触发重新启动的唯一方法是更新类路径。导致更新类路径的方式取决于您使用的IDE。在eclipse中保存文件就会重启。idea中如果设置自动保存，每自动保存一次就会触发启动一次。

## 3.记录条件评估中的更改

默认情况下，每次应用程序重新启动时，都会记录一个显示条件评估增量的报告。该报告显示了在进行更改（例如添加或删除Bean以及设置配置属性）时对应用程序的自动配置所做的更改。

要禁用报告的日志记录，请设置以下属性：

`spring.devtools.restart.log-condition-evaluation-delta=false`

## 4.不包括资源（Excluding Resources）

某些资源在更改时不一定需要触发重启。例如，可以就地编辑Thymeleaf模板。默认情况下，更改/ META-INF / maven，/ META-INF / resources，/ resources，/ static，/ public或/ templates中的资源不会触发重新启动，但会触发实时重新加载。如果要自定义这些排除项，可以使用spring.devtools.restart.exclude属性。

例如，要仅排除/ static和/ public，您需要设置以下属性：

```
spring.devtools.restart.exclude=static/**,public/**
```

如果要保留这些默认值并添加其他排除项，请改用`spring.devtools.restart.additional-exclude`属性。

## 5.禁用重启

如果您不想使用重启功能，可以使用`spring.devtools.restart.enabled`属性将其禁用。在大多数情况下，您可以在`application.properties中`设置此属性（这样做仍会初始化重新启动的类加载器，但它不会监视文件更改）。

如果需要完全禁用重新启动支持（例如，因为它不能与特定库一起使用），则需要在调用SpringApplication.run（...）之前将spring.devtools.restart.enabled System属性设置为false，如如下例所示：

```java
public static void main(String[] args) {
	System.setProperty("spring.devtools.restart.enabled", "false");
	SpringApplication.run(MyApp.class, args);
}
```

## 6.使用触发器文件（Using a Trigger File）

这个应该很有用，创建一个触发器文件来进行热部署

如果使用不断编译已更改文件的IDE，则可能更喜欢仅在特定时间触发重新启动。为此，您可以使用“触发器文件”，这是一个特殊文件，当您想要实际触发重新启动检查时必须对其进行修改。更改文件只会触发检查，只有在Devtools检测到必须执行某些操作时才会重新启动。触发器文件可以手动更新，也可以使用IDE插件更新。

要使用触发器文件，请将`spring.devtools.restart.trigger-file`属性设置为触发器文件的路径。

您可能希望将`spring.devtools.restart.trigger`文件设置为全局设置，以便所有项目的行为方式相同

## 7.小结

还有很多功能，还是看官方文档，用到的时候再去查。对于devtool的使用我觉得，使用eclipse 很简单，ctrl+s就可以了。对于使用Idea 兄弟我觉得使用触发器文件是个非常好的方式。

首先在application.properties配置文件下配置 触发器文件的路径，比如

`spring.devtools.restart.trigger-file=trigger.txt`

我们在resources文件夹的根目录创建了一个txt文件 trigger.txt

里面内容比如 version=1 其实没什么，，只要这个配置文件更改了，就会触发devtool去重新检查启动。

比如我们更改完我们的代码，想使用热部署，就把1改一下，比如改为2 这个时候就会自动重启。当然还可以用插件，自行百度吧













