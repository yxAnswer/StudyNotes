

# 文件上传

>文件上传主要用到了MultipartFile 这个类，它源自 SpringMVC ，可以提高对文件保存的效率

1、创建FileController 开始写文件上传的接口

```java
package com.test.controller;


import com.test.domain.JsonData;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;

import java.io.File;
import java.io.IOException;
import java.util.UUID;

@RestController
public class FileController {



   public static final String  filePath="D:/ideWorkspace/practice/src/main/resources/static/images/";
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

}

```
这个类中，还是用我们最简单的注解方式@RestController  +@RequestMapping  
然后指定访问路径，主要的区别在于，方法参数使用的是MultipartFile   这个类型，然后给参数指定了一个key.
（1）先对文件进行一些基本的校验，是否为空，文件大小限制
（2）然后获取文件现在名称，通过UUID或者时间戳 重命名这个文件
（3） 重点来了，通过 file.transferTo(newFile); 这行代码将文件写入到相应位置，MultipartFile 对象的transferTo方法，用于文件保存（效率和操作比原先用FileOutStream方便和高效）
（4）对结果进行一些处理和返回状态

2、这里用到了一个JsonData的 java类，代码如下

```java
package com.test.domain;

import java.io.Serializable;

public class JsonData implements Serializable {


    private static final long serialVersionUID = 4185756603091671454L;

    private String  result;
    private String  message;
    private Object  data;

    public JsonData(String result, String message, Object data) {
        this.result = result;
        this.message = message;
        this.data = data;
    }

    public String getResult() {
        return result;
    }

    public void setResult(String result) {
        this.result = result;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

    public Object getData() {
        return data;
    }

    public void setData(Object data) {
        this.data = data;
    }
}

```
这里只是一个简单的实例，实际开发中根据规范和实际需求制定。然后以json的方式返回。

3、测试接口
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181109135612894.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181109135621840.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)
当然也可以多文件进行上传，处理都一样都是使用了 MutipartFile  ,通过transferTo进行写入

4、如何对上传大小进行限制
可以再启动类中进行配置。

```java
package com.test;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.servlet.MultipartConfigFactory;
import org.springframework.context.annotation.Bean;

import javax.servlet.MultipartConfigElement;

@SpringBootApplication
public class Application {

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
在启动类中加入上面代码，，自定义相关大小限制。常规应该把大小的描述放到配置文件中，然后进行引用，这里只是一个示范。
但是这只是在本地，上传到项目文件夹。。接下来应该打包程序，指定文件路径。