# 聊一聊跨域问题

[TOC]

## 1、什么是跨域问题

**跨域**：浏览器同源策略
1995年，同源政策由 Netscape 公司引入浏览器。目前，所有浏览器都实行这个政策。最初，它的含义是指，A网页设置的 Cookie，B网页不能打开，除非这两个网页"同源"。所谓"同源"指的是"三个相同"
- 协议相同  http https
- 域名相同	 www.xdcass.ent
- 端口相同  80  81

**一句话**：浏览器从一个域名的网页去请求另一个域名的资源时，域名、端口、协议任一不同都是跨域
浏览器控制台跨域提示：
```
No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'null' is therefore not allowed access.
```
## 2、有些解决方法

常见的解决办法：

- JSONP 
- HTTP响应头配置允许跨域
  - (1) nginx层进行配置
  - (2) 后端代码层进行配置

解决跨域常用有JSONp方式和 在http响应头配置两种，另外又分为使用Nginx配置和我们代码控制。

关于JSONP  大家自行百度就可以了。反正我不用。

nginx配置是可以的，但是如果项目大，节点多，nginx 配置稍显复杂，最重要的是这个万一出错还是挺危险的。所以我选择使用后端代码进行配置。

spring mvc是支持跨域设置的。

## 3、SpringBoot自带处理方式

上官方文档：大家可以根据自己用的版本看，有可能你的版本用的已经过期的方法。

[https://docs.spring.io/spring-boot/docs/2.1.4.RELEASE/reference/htmlsingle/#boot-features-cors](https://docs.spring.io/spring-boot/docs/2.1.4.RELEASE/reference/htmlsingle/#boot-features-cors)

然后是；

[https://docs.spring.io/spring/docs/5.1.6.RELEASE/spring-framework-reference/web.html#mvc-cors](https://docs.spring.io/spring/docs/5.1.6.RELEASE/spring-framework-reference/web.html#mvc-cors)

大概说一下如何配置。

###  第一种：`@CrossOrigin`注解

**直接在接口上添加：**

官网说，直接加上这个注解，表示此接口就支持跨域，比如

```java
    @CrossOrigin
    @GetMapping("/api/test")
    public Object getAll() {
        Map<String,String> reslut=new HashMap<>();
        reslut.put("message","请求成功");
        reslut.put("result","1");
        System.out.println("接口进行业务处理");
        return reslut;
    }
```

简单粗暴，直接用。 当然`@CrossOrigin`也支持类级别

**在类上使用`@CrossOrigin`**:

```java
@RestController
@CrossOrigin(origins = "https://domain2.com", maxAge = 3600)
public class TestController {
   
    @GetMapping("/api/test")
    public Object getAll() {
        Map<String,String> reslut=new HashMap<>();
        reslut.put("message","请求成功");
        reslut.put("result","1");
        System.out.println("接口进行业务处理");
        return reslut;
    }
    
}
```

这样我们整个类的方法都设置了跨域，当然还可以这样：

```java
@RestController
@CrossOrigin( maxAge = 3600)
public class TestController {
   
    @CrossOrigin(origins = "https://domain2.com")
    @GetMapping("/api/test")
    public Object getAll() {
        Map<String,String> reslut=new HashMap<>();
        reslut.put("message","请求成功");
        reslut.put("result","1");
        System.out.println("接口进行业务处理");
        return reslut;
    }
    
}
```

这样是，我们在类上设置的是过期时间，具体支持哪些源我们再具体方法进行设置。

这种情况下`@CrossOrigin` ：

- 支持所有源
- 支持所有headers
- 支持controller里面所有方法

### 第二种：全局配置跨域

全局配置，默认情况下：

- 所有的源
- 所有headers
- GET,HEAD,POST方法

默认情况下不会开启 `allowedCredentials` ,maxAge设置为30分钟

上代码：

java 代码注解配置：

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {

        registry.addMapping("/api/**")
            .allowedOrigins("https://domain2.com")
            .allowedMethods("PUT", "DELETE")
            .allowedHeaders("header1", "header2", "header3")
            .exposedHeaders("header1", "header2")
            .allowCredentials(true).maxAge(3600);

        // Add more mappings...
    }
}
```

上面是spring mvc 的，当然我们可以这么设置，具体参数大家一看就明白

addMapping()  表示是那些接口 支持跨域，allowedOrigins（）允许的源，allowedHeaders() 允许的header ,allowCredentials() 是否信任， maxAge()有效期。



上面的我们都能在官网找到：

[https://docs.spring.io/spring/docs/5.1.6.RELEASE/spring-framework-reference/web.html#mvc-cors](https://docs.spring.io/spring/docs/5.1.6.RELEASE/spring-framework-reference/web.html#mvc-cors)



**在springboot中 官方配置：**

```java
@Configuration
public class MyConfiguration {

	@Bean
	public WebMvcConfigurer corsConfigurer() {
		return new WebMvcConfigurer() {
			@Override
			public void addCorsMappings(CorsRegistry registry) {
				registry.addMapping("/api/**");
			}
		};
	}
    
}
```

加上我们需要的配置。

当然还有其他的方式比如，

```java
@Bean
CorsWebFilter corsFilter() {

    CorsConfiguration config = new CorsConfiguration();

    // Possibly...
    // config.applyPermitDefaultValues()

    config.setAllowCredentials(true);
    config.addAllowedOrigin("https://domain1.com");
    config.addAllowedHeader("*");
    config.addAllowedMethod("*");

    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
    source.registerCorsConfiguration("/**", config);

    return new CorsWebFilter(source);
}
```



一般情况下，我们直接写一个配置类，用springboot官网的配置，一个`@Configuration`加上一个`@Bean`就够用了。





