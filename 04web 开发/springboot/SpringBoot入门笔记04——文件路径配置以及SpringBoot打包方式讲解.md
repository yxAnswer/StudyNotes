# 文件路径配置以及SpringBoot打包方式讲解

[TOC]

## 1、文件路径的配置

由于我们上一节直接将上传的文件写到了静态资源文件夹下面，如果我们打成jar包运行到服务器上，是无法拿到这个路径的。所以我们需要主动去指定这个文件路径，然后去访问资源。（正常我们需要在配置文件中进行配置，然后引用资源文件读取配置文件）

（1）首先在resources资源文件夹下创建配置文件  命名为 application.properties   应用程序会默认读取配置文件的信息，
可以配置端口，数据库地址以及各种。。。
（2）声明文件路径
例如：`web.images-path=D:/test/images/`
映射名称web.images-path  这个名字随便起，自己来定，， 后面指定的是你要将图片存储的路径
（3）将声明的文件路径添加为外部可以访问的资源文件声明 ，添加配置

```
spring.resources.static-locations=classpath:/META-INF/resources/,classpath:/resources/,classpath:/static/,classpath:/public/
,file:${web.images-path} 
```
以上代码添加到配置文件中。这里简单介绍下springboot 项目的资源文件目录
>src/main/java：存放代码
>src/main/resources 存放资源文件
>	 	static: 存放静态文件，比如 css、js、image, （访问方式 http://localhost:8080/js/main.js）
>	 	config:存放配置文件,application.properties
>	 	resources:
>	 	public
>	 	templates:存放静态页面jsp,html,tpl     **此文件夹需要添加接口映射才能跳转访问**spring-boot-starter-thymeleaf

springboot 资源文件的默认查找顺序为：**META/resources > resources > static > public  里面找是否存在相应的资源，如果有则直接返回。**
静态资源配置官网：
[https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-developing-web-applications.html#boot-features-spring-mvc-static-content](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-developing-web-applications.html#boot-features-spring-mvc-static-content)

默认配置

```
spring.resources.static-locations = classpath:/META-INF/resources/,classpath:/resources/,classpath:/static/,classpath:/public/ 
```
如果需要我们自定义一个文件夹，就需要在上面的配置中添加我们的文件夹说明，比如，在后面添加一个`,classpath:/test/`  这样就把test文件夹添加到配置中了。在resources文件夹下创建test 文件夹，就可以访问这个文件夹了。

小结：我们给图片配置路径就是把我们自定义的路径`web.images-path=D:/test/images` 添加到 `spring.resources.static-locations` 的配置说明中，`file:${自己定义的名字} `指定路径。
然后修改FileController 的路径要和配置文件一致；并且指定的文件夹要存在,
接下来，我们不用硬编码，用配置文件注入的方式动态拿到指定的地址
**我们已经在配置文件中指定了文件路径，当然可以重新写一个，只要符合格式**
1、添加`@PropertySource({"classpath:application.properties"}) `spring会自动扫描配置文件
2、修改路径，将硬编码改为动态获取配置文件值
```java
@Value("${web.upload-path}")
private String filePath;
```

注意这里的{}内的名称要和配置文件中指定的key值 完全一致
```java
package com.test.controller;
import com.test.domain.JsonData;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;

import javax.servlet.http.HttpServletRequest;
import java.io.File;
import java.io.IOException;
import java.util.UUID;

@RestController
@PropertySource({"classpath:application.properties"})
public class FileController {
   //public static final String  filePath="D:/test/images/";
    @Value("${web.upload-path}")
    private String filePath;
    @RequestMapping("/v1/upload")
    public Object upLoad(@RequestParam("head_img") MultipartFile  file){
        if (file.isEmpty()){
            //判空
        }
        if (file.getSize()>100000){
            //大小限制
        }
        //获取当前文件名
        String name = file.getOriginalFilename();
        //后缀名
        String substring = name.substring(name.lastIndexOf("."));
        //重新生成唯一文件名
        String fileName = UUID.randomUUID() + substring;
        File newFile=new File(filePath+fileName);
        try {
            file.transferTo(newFile);
            return new JsonData("1","ok",null);
        } catch (IOException e) {
            e.printStackTrace();
        }catch (IllegalStateException e) {
            e.printStackTrace();
        }

        return  new JsonData("0","上传失败",null);
    }

    /**
     * 多个文件
     * @param request
     * @param file
     * @return
     */
    @RequestMapping(value = "uploads")
    public JsonData uploads(HttpServletRequest request, @RequestParam("head_img") MultipartFile... file) {

        int num = 0;
        for (MultipartFile f : file) {

            // 获取文件名
            String fileName = f.getOriginalFilename();
            System.out.println("上传的文件名为：" + fileName);

            // 获取文件的后缀名,比如图片的jpeg,png
            String suffixName = fileName.substring(fileName.lastIndexOf("."));
            System.out.println("上传的后缀名为：" + suffixName);

            // 文件上传后的路径
            fileName = UUID.randomUUID() + suffixName;
            System.out.println("转换后的名称:" + fileName);

            File dest = new File(filePath + fileName);
            try {
                f.transferTo(dest);
                num++;
            } catch (IllegalStateException e) {
                e.printStackTrace();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        if (num == file.length) {
            return new JsonData("1","ok",null);
        } else {
            return new JsonData("0","上传失败",null);
        }

    }
}

```
## 2、测试文件上传下载

运行springboot，然后调用接口，如：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181109135741528.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

然后再浏览器直接获取图片，因为我们已经在配置文件中指明了路径，也配置了。结果如下，直接使用地址+文件名就可以访问
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181109135750219.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)
## 3、springboot 打成jar运行

光配置完文件路径还不可以，需要进行springboot 的打包。
当然可以用maven 命令直接打包，但是如果不添加一个maven 插件的话，会报错no main manifest attribute, in XXX.jar。
因为maven没有在清单文件中写入，进行启动文件的指定
（1）在pom.xml文件中添加maven插件

```xml
  <build>
        <plugins>
            <!--使用maven打包需要添加这个插件，不然运行没有配置文件 MAINFEST.MF-->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
```

（2）使用maven 进行打包
   可以使用命令  mvn spring-boot:run 直接启动， 或者mvn  package 打包，
   默认是打成jar包，由于springboot 内置了tomcat容器，直接运行jar 包就可以开发服务。
   可以指定最后打包的包名：
   在build 标签下 添加 finalname
   比如：

```xml
   <build>
        <finalName>practice</finalName>
        <plugins>
            <!--使用maven打包需要添加这个插件，不然运行没有配置文件 MAINFEST.MF-->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181109135825694.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)
在idea工具中，可以点击maven 生命周期下的 相关命令，双击直接执行。

（3）直接运行jar   执行命令 `java  -jar  parctice.jar`   即：java -jar  包名.jar
如果是访问我们上传的图片，就  localhost:8080/图片名  就能访问当，如果是服务器就把localhost换位服务器ip
但是如果是打成war 包 访问以及上传都需要加上包名。

以上是打成jar 包。 如果我们需要用一个外置的Tomcat容器来部署服务的话，那我们就可以直接打成war包，，直接扔到服务器上就tomcat  下的webapps 目录下，只要启动tomcat服务器，就会自动部署
## 4、springboot 打成war部署服务器

（1）需要添加spring-boot-starter-tomcat依赖
 因为是要使用外部的tomcat，所以就需要修改pom.xml 的依赖如下
 修改tomcat 的作用域。 当然也有其他方法
```xml
   <!--因配置外部TOMCAT 而配置-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
            <scope>provided</scope>
        </dependency>
```
当然也有其他方法  比如移除内置tomcat 依赖

```xml

   <!-- 移除嵌入式tomcat插件 -->
   <exclusions>
      <exclusion>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-tomcat</artifactId>
      </exclusion>

```
我使用第一种，直接修改tomcat的作用域 为 provided ,表示仅仅在编译时使用，最终不会打到包里面

（2）修改pom.xml文件指明 打包方式为 war
`<packaging>war</packaging>`
不加默认打包方式为jar包，，修改为war,比如我的项目，添加一行代码
```xml
    <groupId>com.gongsi.test</groupId>
    <artifactId>practice</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>war</packaging>

```
（3）最关键的，修改启动类创建的Application。继承SpringBootServletInitializer并且重写configure方法
 代码如下：
```java
package com.test;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.boot.web.servlet.MultipartConfigFactory;
import org.springframework.boot.web.servlet.support.SpringBootServletInitializer;
import org.springframework.context.annotation.Bean;

import javax.servlet.MultipartConfigElement;

@SpringBootApplication
public class Application extends SpringBootServletInitializer {
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return builder.sources(Application.class);
    }

    public static void main(String[] args){
        SpringApplication.run(Application.class,args);
    }

    /**
     * 配置文件大小限制
     * @return
     */
    @Bean
    public MultipartConfigElement multipartConfigElement() {
        MultipartConfigFactory factory = new MultipartConfigFactory();
        //单个文件最大  -----这里的硬编码应该放到配置文件中进行配置，这里只演示下
        factory.setMaxFileSize("10240KB"); //KB,MB
        /// 设置总上传数据总大小
        factory.setMaxRequestSize("1024000KB");
        return factory.createMultipartConfig();
    }
}

```
总共需要做两步操作，先继承 SpringBootServletInitializer ，然后重写 configure(SpringApplicationBuilder builder) 方法，
return  builder.sources(Application.class);  注意： 这个Appication 为你自己定义的类。

（4） mvn  package  进行打包，，并将打成的war 包丢到 tomcat 容器的 webapps路径下，启动tomcat  等待部署完成。

请求接口的时候，比如，我这里的war包 名为practice ,调用接口就需要 http://ip地址:8080/practice/接口名

**如果并发量多或者多个应用，可以用fastdfs，阿里云oss，或者用nginx搭建一个简单的文件服务器，将我们的资源和应用程序分离开来**