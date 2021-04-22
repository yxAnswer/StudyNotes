[TOC]

# SpringBoot 的Filter和Listener

# 1、Filter 过滤器

## 1.1 Springboot默认启动时加载的Filter

`characterEncodingFilter`
`hiddenHttpMethodFilter`
`httpPutFormContentFilter`
`equestContextFilter`

我们可以打开我们的springboot源码看下，在`org.springframework.boot:spring-boot:2.0.4.RELEASE`下的

springframework->boot->web->servlet->filter 可以看到有五个类

```
ApplicationContextHeaderFilter
OrderedCharacterEncodingFilter
OrderedHiddenHttpMethodFilter
OrderedHttpPutFormContentFilter
OrderedRequestContextFilter
```



另外 在servelt 目录下还有一个类 `FilterRegistrationBean`

```java
public class FilterRegistrationBean<T extends Filter>
		extends AbstractFilterRegistrationBean<T> {
	/**
	 * Filters that wrap the servlet request should be ordered less than or equal to this.
	 */
	public static final int REQUEST_WRAPPER_FILTER_MAX_ORDER = AbstractFilterRegistrationBean.REQUEST_WRAPPER_FILTER_MAX_ORDER;

	private T filter;

	/**
	 * Create a new {@link FilterRegistrationBean} instance.
	 */
	public FilterRegistrationBean() {
	}

	/**
	 * Create a new {@link FilterRegistrationBean} instance to be registered with the
	 * specified {@link ServletRegistrationBean}s.
	 * @param filter the filter to register
	 * @param servletRegistrationBeans associate {@link ServletRegistrationBean}s
	 */
	public FilterRegistrationBean(T filter,
			ServletRegistrationBean<?>... servletRegistrationBeans) {
		super(servletRegistrationBeans);
		Assert.notNull(filter, "Filter must not be null");
		this.filter = filter;
	}

	@Override
	public T getFilter() {
		return this.filter;
	}

	/**
	 * Set the filter to be registered.
	 * @param filter the filter
	 */
	public void setFilter(T filter) {
		Assert.notNull(filter, "Filter must not be null");
		this.filter = filter;
	}

}
```

这个类是 注册Filter的bean   FilterRegistrationBean

## 1.2 Springboot Filter优先级

随便点开一个，比如OrderedCharacterEncodingFilter

```java
/**
 * {@link CharacterEncodingFilter} that also implements {@link Ordered}.
 *
 * @author Phillip Webb
 * @since 2.0.0
 */
public class OrderedCharacterEncodingFilter extends CharacterEncodingFilter
		implements Ordered {

	private int order = Ordered.HIGHEST_PRECEDENCE;

	@Override
	public int getOrder() {
		return this.order;
	}

	/**
	 * Set the order for this filter.
	 * @param order the order to set
	 */
	public void setOrder(int order) {
		this.order = order;
	}

}
```

可以看到它继承了`CharacterEncodingFilter`, 他有一个优先级`Ordered.HIGHEST_PRECEDENCE`, 继续点击`Ordered` 看源码：

```java
public interface Ordered {
	/**
	 * Useful constant for the highest precedence value.
	 * @see java.lang.Integer#MIN_VALUE
	 */
	int HIGHEST_PRECEDENCE = Integer.MIN_VALUE;

	/**
	 * Useful constant for the lowest precedence value.
	 * @see java.lang.Integer#MAX_VALUE
	 */
	int LOWEST_PRECEDENCE = Integer.MAX_VALUE;


	/**
	 * Get the order value of this object.
	 * <p>Higher values are interpreted as lower priority. As a consequence,
	 * the object with the lowest value has the highest priority (somewhat
	 * analogous to Servlet {@code load-on-startup} values).
	 * <p>Same order values will result in arbitrary sort positions for the
	 * affected objects.
	 * @return the order value
	 * @see #HIGHEST_PRECEDENCE
	 * @see #LOWEST_PRECEDENCE
	 */
	int getOrder();

}
```

`Ordered.HIGHEST_PRECEDENCE`  ：Integer的最小值
`Ordered.LOWEST_PRECEDENCE` ：Integer 的最大值

**低位值意味着更高的优先级   Higher values are interpreted as lower priority**

**也就是，值越小，优先级越高：**

注意：

**自定义Filter，避免和默认的Filter优先级一样，不然会冲突**

## 1.3 自定义Filter

官方文档：[https://docs.spring.io/spring-boot/docs/2.1.4.RELEASE/reference/htmlsingle/#boot-features-embedded-container-servlets-filters-listeners](https://docs.spring.io/spring-boot/docs/2.1.4.RELEASE/reference/htmlsingle/#boot-features-embedded-container-servlets-filters-listeners)

自定义Filter，我们使用最简单的方式，不使用FilterRegistrationBean 了。 我们使用Servlet3.0 的注解进行配置。

- 使用Servlet3.0的注解进行配置
- 启动类里面增加 @ServletComponentScan，进行扫描
- 新建一个Filter类，implements Filter，并实现对应的接口
-  @WebFilter 标记一个类为filter，被spring进行扫描 urlPatterns：拦截规则，/开头，支持正则
- 控制chain.doFilter的方法的调用，来实现是否通过放行，不放行，web应用resp.sendRedirect("/index.html");
- 场景：权限控制、用户登录(**非前端后端分离场景**)等



```java
package com.michael.study.filter;

import java.io.IOException;
import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.annotation.WebFilter;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;


/**
 * urlPatterns   拦截请求的url,进行正则匹配，/*是拦截所有的，必须以/开头
 * filterName    过滤器名称
 */
@WebFilter(urlPatterns = "/api/*", filterName = "loginFilter")
public class LoginFilter  implements Filter{



    /**
     * 容器加载的时候调用
     */
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("init loginFilter");
    }


    /**
     * 请求被拦截的时候进行调用
     */
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("doFilter loginFilter");

        HttpServletRequest req = (HttpServletRequest) servletRequest;
        HttpServletResponse resp = (HttpServletResponse) servletResponse;
        String username = req.getParameter("username");

        if ("admin".equals(username)) {
            filterChain.doFilter(servletRequest,servletResponse);
        } else {
            resp.sendRedirect("/index.html");
            //或者返回我们约定的json
//        resp.setCharacterEncoding("utf-8");
//        try {
//            PrintWriter writer = resp.getWriter();
//            writer.print(json);
//        } catch (IOException e) {
//            e.printStackTrace();
//        }
        }



    }

    /**
     * 容器被销毁的时候被调用
     */
    @Override
    public void destroy() {
        System.out.println("destroy loginFilter");
    }

}
```

自定义Filter就是这么简单，， 启动类中添加`@ServletComponentScan` 注解，然后自定一个类实现Filter 接口，注意这里要导包的话是 Servlet的包。  然后，重写方法， 最重要的就是编写`doFilter`方法的过滤逻辑。

放行：调用chain.doFilter

不放行：重定向 resp.sendRedirect("/index.html");



注意：这里的登录过滤，只适用于非前后端分离的情况，对于前后端分离的情况，我们一般使用拦截器进行处理。

# 2、自定义Servlet容器

有时候我们需要用到自定义的Servlet，但是java 原生的Servlet 写起来比较麻烦，springboot封装了更简单的自定义Servlet方法，我们一起看下。

- 启动类里面增加 `@ServletComponentScan`，进行扫描
- 新建一个类继承 `HttpServlet`类，实现doGet() 或doPost()方法
- 在类上添加注解 `@WebServlet` 表示这是一个Servlet容器并且能被扫描到
- 在注解`@WebServlet`参数中指定 路由 ，以及doGet 和doPost中写逻辑

```java
import java.io.IOException;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
    自定义原生Servlet
     urlPatterns   路由
    name
 */
@WebServlet(urlPatterns = "/v1/api/test/customs",name = "userServlet")
public class UserServlet extends HttpServlet{

    @Override
    public void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

        resp.getWriter().print("custom sevlet");//写内容
        resp.getWriter().flush();//刷新
        resp.getWriter().close();//关流
    }


    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        this.doGet(req, resp);
    }
    
}

```

自定义完成以后，我们可以直接访问我们的路由地址，我们自定义的Servlet就会接收请求，执行逻辑代码，响应请求并返回。， 其实就 一个`@WebServlet`注解就搞定了，很方便。

# 3、Listener

listener 是监听我们的服务，当满足一定条件，就会触发我们定义的listener 代码。

**常用的监听器：**

`servletContextListener` :监听程序启动到销毁

`httpSessionListener` ：监听session被创建直到失效的过程

`servletRequestListener`： 监听网络请求初始化到销毁

## 3.1自定义servletRequestListener

`servletRequestListener` 我们一般用于对请求监听，比如统计信息

```java
package com.michael.study.listener;

import javax.servlet.ServletRequestEvent;
import javax.servlet.ServletRequestListener;
import javax.servlet.annotation.WebListener;

/**
 * web请求的监听器，一般用于做一些统计
 */
@WebListener
public class RequestListener implements ServletRequestListener {
	@Override
	public void requestInitialized(ServletRequestEvent sre) {
		System.out.println("======初始化请求========");

	}


	@Override
	public void requestDestroyed(ServletRequestEvent sre) {
		System.out.println("======请求销毁========");
	}

   
}
```

- 启动类里面增加 `@ServletComponentScan`，进行扫描
- 创建监听类，实现 `EventListener`接口 ,比如ServletRequestListener也实现了EventListener
- 添加`@WebListener` 注解，表示这个是个监听器，并且能被扫描
- 在初始化方法和销毁方法 写我们自己的逻辑

上面代码演示的是`ServletRequestListener` 监听器，监听我们Servlet请求，我们随便写个接口，比如

```java 
    @GetMapping("/api/test")
    public Object getAll() {
        Map<String,String> reslut=new HashMap<>();
        reslut.put("message","请求成功");
        reslut.put("result","1");
        System.out.println("接口进行业务处理");
        return reslut;
    }

```

当我们开启监听器，(请注意下是否还开启了其他过滤器或者拦截器)， 我们调用这个接口，看下 log输出。

```
======初始化请求========
接口进行业务处理
======请求销毁========

```

这表示，我们的请求先走的 `requestInitialized`方法，然后是接口的逻辑，最后是`requestDestroyed`，我们要做的就是在合适的地方写合适的逻辑就可以了。

## 3.2 自定义上下文监听器

`servletContextListener`  一般用于在程序启动时监听，做一些初始化操作，比如，连接数据库，redis，ElasticSearch等等

```java
package com.michael.study.listener;

import javax.servlet.ServletContextEvent;
import javax.servlet.ServletContextListener;
import javax.servlet.annotation.WebListener;
@WebListener
public class CustomContextListener implements ServletContextListener{

	@Override
	public void contextInitialized(ServletContextEvent sce) {
		//可以在启动的时候做一些资源的加载；比如连接数据库，redis，等等组件
		//需要在这开启子线程进行处理，不能阻塞主线程
		System.out.println("======上下文初始化========");
	}
	@Override
	public void contextDestroyed(ServletContextEvent sce) {
		System.out.println("======上下文销毁========");
		
	}

}

```

同样的步骤，

1、启动类添加`@ServletComponentScan`注解 

2、继承ServletContextListener 

3、添加`@WebListener` 注解 

4、 写逻辑代码



# 4、自定义拦截器

## 4.1 自定义拦截器实战

springboot 2.x以上，自定义拦截器的api发生了变化，我们使用更方便。核心接口 `WebMvcConfigurer`和`HandlerInterceptor`

`WebMvcConfigurer` : 继承它我们可以声明一些我们拦截器的路径，配置管理拦截器

`HandlerInterceptor` :这个是我们真正处理拦截器的逻辑，也就是拦截器的业务类。

- 创建拦截器配置类，实现`WebMvcConfigurer`接口，并重写`addInterceptors(InterceptorRegistry registry)`方法
- 在配置类上添加 `@Configuration` 注解
- 创建拦截器业务类， 实现`HandlerInterceptor`接口，并重写，`preHandle()`,`postHandle`，`afterCompletion`三个方法
- 编写完逻辑后，将我们的拦截器业务类 在配置类进行注册，通过调用上面配置类的`registry.addInterceptor(业务类对象).addPathPatterns("要拦截路由");`
- 注意，按照注册顺序进行拦截，先注册，先被拦截，可以注册多个



`preHandle()`：**调用Controller某个方法之前**
`postHandle()`：**Controller之后调用，视图渲染之前，如果控制器Controller出现了异常，则不会执行此方法**
`afterCompletion()`：**不管有没有异常，这个afterCompletion都会被调用，用于资源清理**

上代码：

```java 
package com.michael.study.intecpter;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class CustomWebMvcConfigurer implements WebMvcConfigurer  {
	@Override
	public void addInterceptors(InterceptorRegistry registry) {

		registry.addInterceptor(new LoginIntercepter()).addPathPatterns("/api/test/*/**");
        //当然可以设置部分不拦截，比如不拦截test下面的a开头的所有接口
		//.excludePathPatterns("/api/test/a/**");  
        
        //拦截全部可以这样 /*/*/**
        
       // 这里可以注册多个，先注册，先拦截
		WebMvcConfigurer.super.addInterceptors(registry);
	}

}

```

**注意：**

```
//    拦截器最后路径一定要 “/**”， 如果是目录的话则是 /*/
```

```java
package com.michael.study.intecpter;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

/**
 * 拦截器的业务类
 */
public class LoginIntercepter implements HandlerInterceptor {

    /**
     * 进入controller方法之前
     * 可以做一些校验
     */
    @Override
    public boolean preHandle(HttpServletRequest request,
                             HttpServletResponse response, Object handler) throws Exception {
        System.out.println("LoginIntercepter------->preHandle");

        //在这里进行token 校验等等
        return HandlerInterceptor.super.preHandle(request, response, handler);
    }

    /**
     * 调用完controller之后，视图渲染之前
     */
    @Override
    public void postHandle(HttpServletRequest request,
                           HttpServletResponse response, Object handler,
                           ModelAndView modelAndView) throws Exception {

        System.out.println("LoginIntercepter------->postHandle");

        HandlerInterceptor.super.postHandle(request, response, handler, modelAndView);
    }

    /**
     * 整个完成之后，通常用于资源清理
     */
    @Override
    public void afterCompletion(HttpServletRequest request,
                                HttpServletResponse response, Object handler, Exception ex)
            throws Exception {
        System.out.println("LoginIntercepter------->afterCompletion");

        HandlerInterceptor.super.afterCompletion(request, response, handler, ex);
    }

}

```

运行程序，我们访问之前的测试接口，`http://localhost:8080/api/test`

这里我们拦截了 api/test/ 下的所有接口，打印如下

```
------------ThreeIntercepter--------------------preHandle()----------
接口进行业务处理
------------ThreeIntercepter--------------------postHandle()----------
------------ThreeIntercepter--------------------afterCompletion()----------
```

可以看到，打印顺序之前说的一致。

## 4.2  拦截器(Interceptor)和 过滤器(Filter)的区别

- `Filter`是基于函数回调` doFilter()`，而`Interceptor`则是基于AOP思想
- `Filter`在只在`Servlet`前后起作用，而`Interceptor`够深入到方法前后、异常抛出前后等
- `Filter`依赖于`Servlet`容器即web应用中，而`Interceptor`不依赖于`Servlet容器`所以可以运行在多种环境。
- 在接口调用的生命周期里，`Interceptor`可以被多次调用，而`Filter`只能在容器初始化时调用一次。
- 顺序：过滤前->拦截前->action执⾏行行->拦截后->过滤后  

## 4.3 Listener（监听）、 Interceptor（拦截器）、Filter（过滤器） 的拦截顺序

以一个Login接口请求到相应结束为例，看一下各个拦截器和监听器的顺序打印情况：

```shell
================Listener 的requestInitialized()调用了====================
================Filter 的doFilter调用了====================
================Interceptor 的preHandle()调用了====================
================controller 接口调用了====================
================Interceptor 的postHandle()调用了====================
================Interceptor 的afterCompletion()调用了====================
================Listener 的requestDestroyed()调用了====================
```



可以看到，

- **程序启动**，filter的init()先调用
- **开始请求**，首先Listener  的requestInitialized()先调用初始化
- 然后走 Fileter 的doFilter()方法，执行过滤逻辑
- **接口方法执行前**：Interceptor拦截器的preHandle()调用
- **接口执行**：controller层的接口执行具体的代码
- 接口执行后：Interceptor拦截器的的postHandle()调用
- 然后是：Interceptor拦截器的的的afterCompletion()调用
- **请求结束**：Listener的的requestDestroyed

总的来说，Listener监听的是 servlet 初始化和销毁；然后Filter 依赖于servlet容器，初始化了以后执行，然后销毁之前进行执行，Interceptor拦截器 是作用于 接口方法执行前、执行后，不依赖于servlet。

Listener-——>Filter——>Interceptor——>接口方法——>Interceptor——>Listener









