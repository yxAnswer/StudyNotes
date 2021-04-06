# 全局异常处理及自定义异常

[TOC]

## 1、捕获全局异常

捕获全局异常其实很简单，只需要两个注解就搞定了。首先，创建一个异常捕获类，比如叫`CustomExtHandler`

```java
package com.michael.study.domain;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

import java.util.HashMap;
import java.util.Map;

import javax.servlet.http.HttpServletRequest;

/**
 * 作者    Google
 * 时间    2019/4/15 11:19
 * 文件    CustomExtHandler
 * 描述    异常处理类
 */

@RestControllerAdvice
public class CustomExtHandler {

    private static final Logger logger = LoggerFactory.getLogger(CustomExtHandler.class);

    /**
     * 捕获全局的异常
     * @param e
     * @param request
     * @return
     */
    @ExceptionHandler(value = Exception.class)
    //@ResponseBody
    Object handleException(Exception e, HttpServletRequest request) {
        //输出我们的log日志
        logger.error("url {},msg{}",request.getRequestURL(),e.getMessage());
        Map<String, Object> map = new HashMap<>();
        map.put("msg", e.getMessage());
        map.put("code", 0);
        return map;
    }
}

```

上面的代码就完成自捕获全局异常，并返回指定信息。

- 第一步，在类上添加 `@RestControllerAdvice`注解，表示这个controller建议并且以json格式返回
- 第二步，写一个捕获异常的方法，接收两个参数，`Exception` 和`HttpServletRequest`
- 第三步，在方法上添加注解`@ExceptionHandler(value = Exception.class)` 它表示要接收的异常级别是Exception ，也就是全局的异常
- 第四步，编写需要返回给前端的数据

注意，还有一种方式，就是在类上用注解`@ControllerAdvice` ，在方法上添加两个注解

`@ExceptionHandler(value = Exception.class)`和`@ResponseBody` 结果是一样的。无非就是让自定义的数据以json格式返回。

我们这时候可以写一个controller 手动触发一个异常，比如

```java 
    @GetMapping("/v1/api/test")
    public Object testException(){
        Map<String,String > params=new HashMap<>();
        params.put("b","走了老弟");
        params.put("a","来了老弟");
        int a=10/0;
        return params;
    }
```

然后运行程序，调用接口，就会出现

```json
{
    "msg": "/ by zero",
    "code": 0
}
```

可以看到，我们的异常按照我们自定义的格式返回了。

## 2、自定义异常

自定义异常，首先我们要做的是自定义异常类

### 2.1自定义异常类

```java 
package com.michael.study.exception;

/**
 * 时间    2019/4/15 11:47
 * 文件    MyException
 * 描述      自定义异常类
 */
public class MyException extends RuntimeException {
    private String code;
    private String msg;

    public MyException(String code,String msg){
        this.code=code;
        this.msg=msg;
    }

    public String getCode() {
        return code == null ? "" : code;
    }

    public void setCode(String code) {
        this.code = code;
    }

    public String getMsg() {
        return msg == null ? "" : msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }
}

```

我们自定义的异常类只有两个参数，一个是状态码 另一个就是异常消息。接下来我们要做的是在异常处理类中去捕获我们自定义的异常。

### 2.2处理自定义异常

在我们上面创建的异常处理类`CustomExtHandler` 中，添加我们自定义异常,完整代码如下：

```java
package com.michael.study.domain;

import com.michael.study.exception.MyException;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

import java.util.HashMap;
import java.util.Map;

import javax.servlet.http.HttpServletRequest;

/**
 * 作者    Google
 * 时间    2019/4/15 11:19
 * 文件    CustomExtHandler
 * 描述    异常处理类
 */

@RestControllerAdvice
public class CustomExtHandler {

    private static final Logger logger = LoggerFactory.getLogger(CustomExtHandler.class);

    /**
     * 捕获全局的异常
     *
     * @param e
     * @param request
     * @return
     */
    @ExceptionHandler(value = Exception.class)
    //@ResponseBody
    Object handleException(Exception e, HttpServletRequest request) {
        //输出我们的log日志
        logger.error("url {},msg{}",request.getRequestURL(),e.getMessage());
        Map<String, Object> map = new HashMap<>();
        map.put("msg", e.getMessage());
        map.put("code", 0);
        return map;
    }

    /**
     * 自定义异常处理
     * @param e
     * @param request
     * @return
     */
    @ExceptionHandler(value = MyException.class)
    Object handleMyException(MyException e, HttpServletRequest request){

        logger.error("url {},msg{}",request.getRequestURL(),e.getMessage());
        Map<String, Object> result = new HashMap<>();
        result.put("msg", e.getMsg());
        result.put("code", e.getCode());
        return result;

    }
}

```

我么同样是通过`@RestControllerAdvice` 和` @ExceptionHandler(value = MyException.class)` 两个注解，在这捕获了我们自定义的异常。当我们的代码中有地方抛出`MyException` 异常的时候，就会在这里捕获，然后我们在这打印了下日志，并且返回了我们自定义异常的信息。 

### 2.3 小结

自定义异常，我们上面只是演示，所以取名最好规范一些，并且可以约定自己的异常状态吗，和团队人员一同制定异常规范，返回合适的异常信息。 大概的步骤就是如上面所写

- 创建异常类，用于抛出异常
- 创建异常处理类，用于捕获异常
- 两个关键注解`@RestControllerAdvice` 和` @ExceptionHandler(value = 自定义异常类名.class)`
- 如果抛出的异常没有被指定捕获，就会被定义的全局异常所捕获