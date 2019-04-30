---
title: SpringBoot入门笔记08——Mybatis逆向工程之IDE自动生成实体类
date: 2018-10-6 
tags:
- java
categories:
- java

---

###  这里只是介绍一种最简单的逆向生成java实体类的方法

mybatis提供了一种逆向工程，网上有很多，大多通过配置文件的方式和依赖的方式，稍微有一点繁琐。我们的Intellij idea  已经集成了自动生成pojo类的工具，直接使用更加方便。
### idea连接数据库
![mark](http://oima95jt3.bkt.clouddn.com/blog/180910/GidBI2FB89.png?imageslim)
![mark](http://oima95jt3.bkt.clouddn.com/blog/180910/3E1c6480jg.png?imageslim)

### 修改生成脚本

![mark](http://oima95jt3.bkt.clouddn.com/blog/180910/lfdD1D59hc.png?imageslim)

![mark](http://oima95jt3.bkt.clouddn.com/blog/180910/KFDfciab9E.png?imageslim)
**当然很重要的一点要记得修改我们的包名，图片忘记标记了**


### 快速生成实体类
![mark](http://oima95jt3.bkt.clouddn.com/blog/180910/LflB6GAi6i.png?imageslim)
然后选择相应的文件路径，打开自动生成的javabean ，可以稍作修改，直接拷贝到相应包中。