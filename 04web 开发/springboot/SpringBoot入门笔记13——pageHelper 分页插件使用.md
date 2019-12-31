[TOC]

# pageHelper分页插件的使用

实现方式有好几种，无非就是 spring的还是springboot的，是注解的还是配置的。

比如：

## 1、引入依赖

```xml
<!-- 分页插件依赖 -->
<dependency>
  <groupId>com.github.pagehelper</groupId>
  <artifactId>pagehelper</artifactId>
  <version>4.1.0</version>
</dependency>
```



## 2、增加配置文件

   ```java
@Configuration
public class MyBatisConfig {
    @Bean
    public PageHelper pageHelper() {
        PageHelper pageHelper = new PageHelper();
        Properties p = new Properties();

        // 设置为true时，会将RowBounds第一个参数offset当成pageNum页码使用
        p.setProperty("offsetAsPageNum", "true");

        //设置为true时，使用RowBounds分页会进行count查询
        p.setProperty("rowBoundsWithCount", "true");
        p.setProperty("reasonable", "true");
        pageHelper.setProperties(p);
        return pageHelper;
    }
}
   ```

## 3、包装类

```java
PageHelper.startPage(page, size);
PageInfo<VideoOrder> pageInfo = new PageInfo<>(list);
```

controller层代码展示：

```java
    @GetMapping("/get_all_system_user")
    public Object getAllUserInfo(@RequestParam(value = "page", defaultValue = "1") int page, @RequestParam(value = "size", defaultValue = "10") int size) {

        //开启分页
        PageHelper.startPage(page, size);
        //根据userID排序 还可以字段名+desc 指定正反
        PageHelper.orderBy("UserID"); 
        //实际的调用service查询数据
        List<PrjPersonInfo> allSystemUser = systemUserService.getAllSystemUser();
        //使用PageInfo类包装
        PageInfo<PrjPersonInfo> pageInfo = new PageInfo<>(allSystemUser);
        //根据自定义类指定返回json样式
        PageJson pageJson = new PageJson(page, pageInfo.getSize(), pageInfo.getPages(), pageInfo.getTotal(), pageInfo.getList());
        return JsonData.success(pageJson);
    }
```

我们肯定不能把所有的pagehelper数据返回，按照需要自定义。如PageJson类

```java

/**
 * 文件名: PageJson.java
 * 描述：用于返回分页数据
 * Create by Google on 2018/10/17 15:18
 */
public class PageJson {

    private int pageIndex;
    private int size;
    private int totalPage;
    private long totalSize;
    private Object data;

    public PageJson(int pageIndex, int size, int totalPage, long totalSize, Object data) {
        this.pageIndex = pageIndex;
        this.size = size;
        this.totalPage = totalPage;
        this.totalSize = totalSize;
        this.data = data;
    }

    public int getPageIndex() {
        return pageIndex;
    }

    public void setPageIndex(int pageIndex) {
        this.pageIndex = pageIndex;
    }

    public int getSize() {
        return size;
    }

    public void setSize(int size) {
        this.size = size;
    }

    public int getTotalPage() {
        return totalPage;
    }

    public void setTotalPage(int totalPage) {
        this.totalPage = totalPage;
    }

    public long getTotalSize() {
        return totalSize;
    }

    public void setTotalSize(long totalSize) {
        this.totalSize = totalSize;
    }

    public Object getData() {
        return data;
    }

    public void setData(Object data) {
        this.data = data;
    }
}

```



## 4、基本原理	

```java
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