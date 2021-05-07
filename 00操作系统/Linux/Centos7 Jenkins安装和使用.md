# Jenkins

- 官网下载最新的war包
- 上传到服务器
- nohup java -jar jenkins.war --httpProt=9000 >msg.log &   守护进程启动
- 开放9000 防火墙端口
- 访问网址 管理员登录
- 创建账户
- 修改下载的源，设置为国内
- 修改default.json的 路径，替换为源。  或者设置nginx代理
- 下载插件

[https://www.jenkins.io/download/](https://www.jenkins.io/download/)



# 1、安装jdk、maven

# 2、以war包方式安装jenkins

# 3、下载插件-解决网络问题

# 4、注意插件版本与jenkins问题





# 插件下载-国内网络问题解决



**2.277.3**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210425104436234.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)





![在这里插入图片描述](https://img-blog.csdnimg.cn/20210425130248825.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)



访问插件管理地址""

http://192.168.42.133:9000/pluginManager/advanced

原来的url:  https://updates.jenkins.io/update-center.json

替换国内的源，不然有可能下载插件特别慢和超时。

http://mirror.esuni.jp/jenkins/updates/update-center.json



http://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json

修改default.json文件

```shell
#进入jenkins的工作目录下的updates
cd /root/.jenkins/updates  

#里面有个default.json ,替换它的内容
sed -i 's/https:\/\/updates.jenkins.io\/download/http:\/\/mirror.esuni.jp\/jenkins/g' default.json && sed -i 's/http:\/\/www.google.com/http:\/\/www.baidu.com/g' default.json

#清华源  里面有个default.json ,替换它的内容
sed -i 's/https:\/\/updates.jenkins.io\/download/http:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g' default.json && sed -i 's/http:\/\/www.google.com/http:\/\/www.baidu.com/g' default.json
```

好像还不行：换个源

jenkins这些源有点坑，下载的default.json里面还是重定向到官方地址，，所以，需要我们手动去替换成我们设置的源的下载地址。

还有一种方式就是为jenkins设置 nginx代理，将官方下载地址替换为国内镜像地址。真麻烦。。



【详细的Jenkins的镜像地址查询：http://mirrors.jenkins-ci.org/status.html】


http://192.168.42.133:9000/restart 后面拼个restart即可重启jenkins



![在这里插入图片描述](https://img-blog.csdnimg.cn/20210425105128913.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)





## 安装插件记录

直接安装git可能会因为依赖版本问题，所以就一个个的装，去官网看支持的最低jenkins版本。

安装git：

-  SSH Credentials  1.18.2
-  git-client 3.7.1
-  git 4.7.1

安装maven：

- Maven Integration 3.10
- SonarQube Scanner
- Publish Over CIFS 
- Publish Over FTP
- Publish Over SSH



配置jdk,maven

echo  $JAVA_HOME  查看java安装目录









