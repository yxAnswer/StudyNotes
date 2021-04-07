### 这里只是介绍一种最简单的逆向生成java实体类的方法
mybatis提供了一种逆向工程，网上有很多，大多通过配置文件的方式和依赖的方式，稍微有一点繁琐。我们的Intellij idea  已经集成了自动生成pojo类的工具，直接使用更加方便。
### idea连接数据库
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181109140546285.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181109140557238.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)
### 修改生成脚本
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181109141103624.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181109141127647.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)
**当然很重要的一点要记得修改我们的包名，图片忘记标记了**

### 快速生成实体类
![在这里插入图片描述](https://img-blog.csdnimg.cn/2018110914114643.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)
然后选择相应的文件路径，打开自动生成的javabean ，可以稍作修改，直接拷贝到相应包中。