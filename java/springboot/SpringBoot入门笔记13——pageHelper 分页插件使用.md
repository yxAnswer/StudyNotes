[TOC]

# pageHelper分页插件的使用

实现方式有好几种，无非就是 spring的还是springboot的，是注解的还是配置的。

比如：

## 1、引入依赖

```
	<!-- 分页插件依赖 -->
			<dependency>
    	        <groupId>com.github.pagehelper</groupId>
    	        <artifactId>pagehelper</artifactId>
    	        <version>4.1.0</version>
    	    </dependency>
```



## 2、增加配置文件

   ```
	@Configuration
			public class MyBatisConfig {
			    @Bean
			    public PageHelper pageHelper(){
			        PageHelper pageHelper = new PageHelper();
			        Properties p = new Properties();
			        p.setProperty("offsetAsPageNum","true");
			        p.setProperty("rowBoundsWithCount","true");
			        p.setProperty("reasonable","true");
			        pageHelper.setProperties(p);
			        return pageHelper;
			    }
			}
   ```

## 3、包装类

```

			PageHelper.startPage(page, size);

			PageInfo<VideoOrder> pageInfo = new PageInfo<>(list);
```

## 4、基本原理	

```

		sqlsessionFactory -> sqlSession-> executor -> mybatis sql statement
		通过mybatis plugin 增加拦截器，然后拼装分页
		org.apache.ibatis.plugin.Interceptor
```



上面是一种，还有比如：

```
 <!--pagehelper-->
<dependency>
   <groupId>com.github.pagehelper</groupId>
     <artifactId>pagehelper-spring-boot-starter</artifactId>
     <version>1.2.3</version>
 </dependency>
```

```
#pagehelper
pagehelper.helperDialect=mysql
pagehelper.reasonable=true
pagehelper.supportMethodsArguments=true
pagehelper.params=count=countSql

然后使用：
PageHelper.startPage(page, size);
PageInfo<VideoOrder> pageInfo = new PageInfo<>(list);
```

## 5、总结

总的来说使用过程就是，1、引入依赖 2、配置（其实也就是配置拦截器）3、PageHelper.startPage(page, size);
PageInfo<VideoOrder> pageInfo = new PageInfo<>(list); 两行代码，开启分页拦截，用pageInfo包装数据，然后用PageInfo再取出来就有了分页信息和数据。

之所以没有详细的讲解每一步怎么使用，因为这个插件在github上面已经有很详细的中文文档了，不想再抄一遍了。

[https://github.com/pagehelper/Mybatis-PageHelper/blob/master/README_zh.md](https://github.com/pagehelper/Mybatis-PageHelper/blob/master/README_zh.md)

详细配置：

[https://github.com/pagehelper/Mybatis-PageHelper/blob/master/wikis/zh/HowToUse.md](https://github.com/pagehelper/Mybatis-PageHelper/blob/master/wikis/zh/HowToUse.md)

集成springboot

[https://github.com/abel533/MyBatis-Spring-Boot](https://github.com/abel533/MyBatis-Spring-Boot)

并且上面还有例子，不会用的时候查两遍就ok了