[TOC]

# 开始写代码

# 一、创建一个简单的接口，返回json
## 1、创建相应的包和类
	 一般我们会分包进行创建，我这里简单创建了一个controller 的包，里面写相关的接口controller
	 然后再test包下我创建了一个Application 类，这个名词可以自己起。这个类是运行springboot的入口，需要进行配置下。
	 Application 代码如下：

## 2、Application  类代码讲解
```java
package com.test;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {

    public static void main(String[] args){
        SpringApplication.run(Application.class,args);
    }
}

```
**@SpringBootApplication 注解：**
`@SpringBootApplication`注解一般放在项目的一个启动类上，用来把启动类注入到容器中，用来定义容器扫描的范围，用来加载classpath环境中一些bean
`@SpringBootApplication` =` @Configuration`+`@EnableAutoConfiguration`+`@ComponentScan`
springboot 把这几个注解整合了。只需要写一个就可以

然后在main方法中添加`SpringApplication.run(Application.class,args);` 来启动应用程序

## 3、TestController类讲解

```java
package com.test.controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import java.util.HashMap;
import java.util.Map;

@RestController
public class TestController {
    //测试用的集合
    Map<String,Object> params=new HashMap<>();

    /**
     * 第一个接口测试
     * @param name
     * @return
     */
    @RequestMapping("/test")
    public Object getTest(String name){
        params.clear();
        params.put("name",name);
        return  params;
    }
}

```
这里用到了两个注解，@RestController、 @RequestMapping
**@RestController and @RequestMapping是springMVC的注解，不是springboot特有的**	
**(1) @RestController**
`@RestController` 是spring4 新加入的注解，相当于`@Controller +@ResponseBody`  两个注解的功能和
`@Controller`的作用表示该类是一个控制器，可以接受用户的输入并调用模型和视图去完成用户的需求。控制器本身不输出也不处理任何东西。 我们写的接口也就是控制器里面的方法。
`@ResponseBody` 表示 请求以json映射的方式返回。
所以`@RestController `注解的类就可以 接受请求——返回json
**(2) @RequestMapping**
 这个注解可以加载类和方法上，我理解是表示资源的映射吧，为web请求指明路径。可以定义不同的映射规则
 注解在方法上表示当前方法时一个web请求的处理方法，注解在类上，应该是常用的请求地址或路由啥的

上面方法中在`@RequestMapping("/test")`  方法中的"/test" 就是外部访问的接口地址，暂时我们没有指定请求方法
这样外部就能通过 http://localhost:8080/test?name=张三  来进行调用这个接口。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181109135309612.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)
# 二、 get、post请求实战
## get请求实战

```java
package com.test.controller;
import com.test.domain.User;
import org.springframework.web.bind.annotation.*;
import javax.servlet.http.HttpServletRequest;
import java.util.HashMap;
import java.util.Map;
@RestController
public class GetController {

    Map<String, Object> params = new HashMap<>();

    /**
     * 从路径中取值
     *
     * @param cityID
     * @param userID
     * @return
     */
    @RequestMapping(path = "/{city_id}/{user_id}")
    public Object getUserID(@PathVariable("city_id") String cityID,
                            @PathVariable("user_id") String userID) {

        params.clear();
        params.put("cityID", cityID);
        params.put("userID", userID);
        return params;
    }

    /**
     * 测试@GetMapping注解，以get方式请求接口
     * @param id
     * @param name
     * @return
     */
    @GetMapping(value = "/v1/getUser")
    public Object getMappingTest(String id, String name) {

        params.clear();
        params.put("name", name);
        params.put("id", id);
        return params;
    }

    /**
     * 测试，设置参数默认值，别名、以及是否必传参数
     * @param id
     * @param username
     * @return
     */
    @RequestMapping("/v1/getUser1")
    public Object getMyappingTest1(@RequestParam(defaultValue = "888",name = "uid") String id, @RequestParam(required = true) String username){
        params.clear();
        params.put("username", username);
        params.put("id", id);
        return params;
    }
    /**
     * 功能描述：bean对象传参
     * 注意：1、注意需要指定http头为 content-type为application/json
     * 		2、使用body传输数据
     * @param user
     * @return
     */
    @RequestMapping("/v1/save_user")
    public Object saveUser(@RequestBody User user){
        params.clear();
        params.put("user", user);
        return params;
    }

    /**
     * 功能描述：测试获取http头信息
     * @param accessToken
     * @param id
     * @return
     */
    @GetMapping("/v1/get_header")
    public Object getHeader(@RequestHeader("access_token") String accessToken, String id){
        params.clear();
        params.put("access_token", accessToken);
        params.put("id", id);
        return params;
    }

    /**
     * 以HttpServletRequest获取所有请求数据
     * @param request
     * @return
     */
    @GetMapping("/v1/test_request")
    public Object testRequest(HttpServletRequest request){
        
        params.clear();
        String id = request.getParameter("id");
        params.put("id",id);
        return params;
    }
    @PostMapping("/v1/login")
    public Object login(@RequestParam(required = true) String  userName, @RequestParam(required = true)String passWrod){
        params.clear();
        params.put("name",userName);
        params.put("pwd",passWrod);
        return params;
    }
    @PostMapping("/v1/class")
    public Object loginTest(User user){
        params.clear();
        params.put("name",user.getName);
        params.put("pwd",user.getPassword);
        return params;
    }

    @PutMapping("/v1/put")
    public Object put(String id){
        params.clear();
        params.put("id", id);
        return params;
    }


    @DeleteMapping("/v1/del")
    public Object del(String id){
        params.clear();
        params.put("id", id);
        return params;
    }
}

```

```
注解简介：		1、单一参数@RequestMapping(path = "/{id}", method = RequestMethod.GET)
				1) public String getUser(@PathVariable String id ) {}
				
				2）@RequestMapping(path = "/{depid}/{userid}", method = RequestMethod.GET) 可以同时指定多个提交方法
				getUser(@PathVariable("depid") String departmentID,@PathVariable("userid") String userid)

				3）get、post、put、delete四种注解的封装
				@GetMapping = @RequestMapping(method = RequestMethod.GET)
				@PostMapping = @RequestMapping(method = RequestMethod.POST)
				@PutMapping = @RequestMapping(method = RequestMethod.PUT)
				@DeleteMapping = @RequestMapping(method = RequestMethod.DELETE)

				4）@RequestParam(value = "name", required = true)
					可以设置默认值，可以设置别名name=""，可以设置是否必传 requeid=true或false

				4)@RequestBody 请求体映射实体类,以json映射javaBean
					需要指定http头为 content-type为application/json charset=utf-8

				5）@RequestHeader 请求头，比如鉴权,可以获取请求头
					@RequestHeader("access_token") String accessToken

				6）HttpServletRequest request自动注入获取参数，可以拿到任何请求数据
				

```