

# Centos7安装配置 Jenkins 2.282 

[toc]

# 一、Jenkins安装、配置

## 1、安装jdk、maven、git

### 1.1 jdk的安装

下载地址：http://maven.apache.org/download.cgi

安装之前先查看原linux 是否安装jdk 

```shell
rpm -qa | grep -i java  #如果没有就安装，如果有，就卸载
rpm -e --nodeps [要卸载的软件名] 
```

**上传jdk到linux文件目录**

我习惯把软件安装到/usr/local 下，把java安装到/usr/loca/java目录下

```shell
 cd /usr/local
 mkdir java
 tar -xvf jdk-11.0.10_linux-x64_bin.tar.gz  -C /usr/local/java #解压到你要安装的路径下，这里是/usr/local/java
```

>如果使用secure crt   传输出现 rz命令 提示command not found 解决方法
>
>yum -y install lrzsz     先安装传输工具

 **配置环境变量**

修改/etc/profile 系统的配置文件 `vim /etc/profile`

文件末尾 添加以下： 权限不够的话使用sudo vi /etc/profile

```
#set java environment
JAVA_HOME=/usr/local/java/jdk1.8.0_291
CLASSPATH=.:$JAVA_HOME/lib
PATH=$JAVA_HOME/bin:$PATH
export JAVA_HOME CLASSPATH  PATH
```

**配置完，需要重新加载配置文件，执行命令：**

`source /etc/profile`

### 1.2 安装maven

官网：[https://maven.apache.org/download.cgi](https://maven.apache.org/download.cgi)

同安装jdk一样

- 上传安装包

- 解压到/usr/local/maven

- 配置环境变量

  - ```shell
    # set maven environment
    MAVEN_HOME=/usr/local/maven/apache-maven-3.8.1
    PATH=$MAVEN_HOME/bin:$PATH
    export MAVEN_HOME  PATH
    ```

- 重新加载`source /etc/profile`

### 1.3编译安装git

编译安装git:    https://mirrors.edge.kernel.org/pub/software/scm/git/  

安装git的依赖项

```shell
yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel
yum install gcc perl-ExtUtils-MakeMaker
```


移除已经安装的git

```shell
yum remove git
```

预编译git

```shell
cd git 解压目录
./configure --prefix=/usr/local/git_2.9.5
```

编译并安装git

```shell
make && make install
```

如果报错：

```
undefined reference to `libiconv’
```

则先安装[libiconv库](http://blog.ganyongmeng.com/?p=589)，如果还报错，则执行以下命令：

```
./configure --prefix=/usr/local/git --with-iconv=/usr/local/libiconv make && make install
```

安装完成后，将git的脚本软连接到/usr/bin/ 目录下

```shell
ln -s /usr/local/git_2.9.5/bin/* /usr/bin/
```

或

```
ln -s /usr/local/git_2.9.5/bin/git /usr/bin/git
```

如果报错，则

```
无法创建符号链接"/usr/bin/git": 文件已存在 等等
```

则删掉或者重命名原来文件，再执行

```
ln -s /usr/local/git_2.9.5/bin/git /usr/bin/git
```



## 2、以war包方式安装jenkins

官网：[https://www.jenkins.io/download/](https://www.jenkins.io/download/)

有多种形式的安装包，这里我选择 war包的方式，因为有jvm 比较省事，在所有平台启动war包即可。

- 下载war包 

- 上传至服务器，我选择到/usr/local/jenkins目录下 

- 可以新建个用户，用于管理jenkins，因为不能每次都是root吧

  ```shell
  useradd   jenkins
  passwd jenkins  
  #并且把 /usr/local/jenkins文件夹的所属赋给用户jenkins
  chown -R jenkins:jenkins /usr/local/jenkins
  ```

- 启动war包，这里可以不用外部tomcat

  ```shell
  nohup java -jar jenkins.war --httpPort=9010 >msg.log &
  ```

  这里nohup 以守护进程方式启动，--httpPort制定端口，这里我没有去修改配置文件，直接在命令行设置 。  `>msg.log &`  表示将输出重定向到msg.log文件下，也就是我们把日志都输出到msg.log。此时文件还没有，执行`touch msg.log`  新建即可。

- 防火墙开放9010端口，让外网访问（如果是阿里云，就从安全组开放即可）

  ```shell
  #开放9010端口
  firewall-cmd --zone=public --add-port=9010/tcp --permanent   
  #重启防火墙
  firewall-cmd --reload
  #查看端口号是否开启
  firewall-cmd --query-port=9010/tcp
  #查看所有打开的端口
  firewall-cmd --zone=public --list-ports
  ```

## 3、 访问并配置jenkins

### 3.1 输入管理员密码解锁jenkins

界面访问http://jenkins所在ip:9010 访问，出现一个解锁jenkins页面，第一次登录，查看initialAdminPassword找到管理员密码并输入。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210508172633744.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210508172856426.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)



### 3.2 配置插件管理地址

国内网比较慢，有时候会失败，第一，将https改为http或者直接切换为国内源。

 如果出现离线页面，这时候可以去配置一下插件管理：http://xxxxxx:9010/pluginManager/advanced 就可以进入管理页面，最下面有个升级站点：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021050817332332.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

注意Update Site(升级站点) 默认值https://updates.jenkins.io/update-center.json  ,如果网络不好，可以改为清华的源：http://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json  ，点击check now 立即获取，其实就是去下载这个 default.json文件，里面是一些插件下载地址。

但是，即使改为清华的源，有时候也是不行的，这个后面总结。

重启jenkins: 输入 http://xxxxx:9010/restart  在url后面输入restart 即可重启jenkins。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210508173540528.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)





![在这里插入图片描述](https://img-blog.csdnimg.cn/20210508173558379.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

### 3.3 跳过插件安装

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210425105128913.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

在这里有两个选择，可以安装推荐插件，也可选择来装，我建议是跳过去，因为在有些网络问题，安装插件现在会失败，比如：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210508173741751.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)





### 3.4 创建第一个管理员用户

输入用户名、密码创建第一个管理员用户，以后登录就使用这个账号了。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210508173755603.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

### 3.5 jenkins url配置实例地址

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210508174648722.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)



![在这里插入图片描述](https://img-blog.csdnimg.cn/20210508174706703.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)



## 4、下载插件-解决网络问题

安装完成后，进入主面板，我们需要安装所需的插件，这里列一下我们所需的插件

> 汉化：
>
> - Localization Chinese(Simplefied)
>
> git：
>
> -  Git plugin
>
> maven
>
> - Maven Integration 3.10
>
> 安装sonarqube scanner:代码质量管理
>
> - SonarQube Scanner
>
> 发布插件：发布到远程服务器按需装
>
> - Publish Over CIFS 
> - Publish Over FTP
> - Publish Over SSH
>
> gitlab:
>
> - Gitlab Plugin



![在这里插入图片描述](https://img-blog.csdnimg.cn/20210517212403754.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210517212656289.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)



在系统管理——插件管理中搜索我们需要的插件，进行下载。如何下载失败，可能有以下原因

- 网络太差，由于jenkins插件地址都是从国外网站下载

  解决办法：1、代理2、手动下载完导入3、配国内源

- 插件依赖的其他插件部分下载失败

  解决办法：将插件的依赖插件一个个装好再去装此插件，可以使用手动导入方式

- 插件和jenkins版本不匹配，不支持当前版本

  解决办法：升级jenkins版本或者找其他插件替代

- 插件和已经下载的插件版本不匹配，需要升级对应插件

  解决办法：升级已安装插件，去官网看支持版本

对于由于网络问题无法下载，我们可以这样解决：

### 4.1、手动下载，然后进行安装上传

下载地址：[https://plugins.jenkins.io/](https://plugins.jenkins.io/)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210508175320906.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

搜索需要安装的插件以及子插件，导入jenkins。比如：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210508175449837.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210508175350577.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

然后就是，缺什么下载什么，按照依赖顺序安装。

### 4.2 配置国内源，提高下载速度

【详细的Jenkins的镜像地址查询：http://mirrors.jenkins-ci.org/status.html】

打开插件管理——>高级或者直接访问http://xxxxxx:9010/pluginManager/advanced  

将 https://updates.jenkins.io/update-center.json 替换为：

http://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json ，点击立即获取。

此操作将会从url网站下载并存储到default.json文件下，位于/root/.jenkins/updates 目录下。

但是不幸的是，这时候点击下载插件还是失败：为什么呢？

jenkins这些源有点坑，下载的default.json里面还是重定向到官方地址，根本没有做到完全走国内的网络，大家可以打开default.json查看下，还都是以updates.jenkins.io开头的网址。怎么解决呢？

- nginx代理方式，讲所有请求转换为国内源地址（不过这个我没有成功，可能是我虚拟机的问题）
- 手动替换的方式（我是选择的这种方式，并将命令写到脚本，需要时执行一下）

```shell
#进入jenkins的工作目录下的updates
cd /root/.jenkins/updates  
#清华源  里面有个default.json ,替换它的内容
sed -i 's/https:\/\/updates.jenkins.io\/download/http:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g' default.json && sed -i 's/http:\/\/www.google.com/http:\/\/www.baidu.com/g' default.json
```

直接简单粗暴将请求地址替换掉，重启，安装插件试一下。如果不行，换个其他源。（注意这里替换完就不要点击check now(立即获取) 按钮了，因为会覆盖掉，如果点了就需要重新执行下脚本）



## 5、系统配置：jdk,maven，sonarqube,邮件

### 5.1 配置jdk

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210509223450518.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210509223541333.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210509223809419.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

### 5.2 配置maven

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210509224323752.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

### 5.3 配置sonarqube(可选)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210509224752179.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210509230101805.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

我们需要新建一个凭证用来链接sonarqube服务器。

进入sonarqube平台，点击配置、权限、用户，生成令牌。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210509230406638.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210509230552339.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

回到jenkins界面，讲令牌写入，创建凭证，然后选择保存即可。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210517223847526.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

接下来配置sonarqube sanner ,进入全局工具配置：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210509223450518.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

找到SonarQube Scanner安装它。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210509231006859.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)



### 5.4邮件配置

在配置邮件之前，我们选择的是qq邮箱，因为163邮箱发几次就不能发了。 如果要配置我们需要获取授权码，如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021050923185675.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

**jenkins配置邮件通知流程：**

系统管理——>系统设置——>邮件通知-——>smtp服务器 smtp.qq.com -——>用户默认邮件后缀 @qq.com——>勾选使用SMTP认证——>输入邮箱和授权码 -——>勾选ssl协议——>输入SMTP端口465 ， Reply-To Address发件者邮箱   ——> 之后测试一下配置，无误即可  



![在这里插入图片描述](https://img-blog.csdnimg.cn/20210509232612621.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210509232837626.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210509233019459.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

然后再点击测试发件，成功了 Email was successfully sent

## 6、配置gitlab,手动构建编译发布

### 6.1 生成一个密钥对用于鉴权

这里，生成密钥，会在home目录生成.ssh目录及密钥 id_rsa （私钥）和id_rsa.pub（公钥）。

```shell
[root@localhost ~]# ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:wO2zYd6YXMTci2e7XhI5uRw66k//E3i8hcnxEncPjik root@localhost.localdomain
The key's randomart image is:
+---[RSA 2048]----+
|                 |
|     . . o .     |
|      o . + .    |
|       o . . ++ o|
|        S o XB Oo|
|       + OE=+*X +|
|        * =.=..= |
|         o o +o  |
|       .o...+... |
+----[SHA256]-----+
```

如果没有命令，安装 `yum -y install openssh-clients`

配置免密登陆

```
yum -y install openssh-clients ssh-keygen -t rsa -- 产生私钥 配置git登陆 将Jenkins所在机子的公钥 more
~/.ssh/id_rsa.pub 的内容拷贝到gitlab项目上  
```

查看公钥，并配置到gitlab:

```shell
[root@localhost .ssh]# more id_rsa.pub 
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDPUbcaWTyF8rsGG7X0sLfS+iyDcCZ5kjQwEAxpMHJrj7iyiXNJUv/HMYwL1l6+SXz1K8HSvlbD9B+6shRj/N0FZeyzin
eEjOmg4t/fwOmBXLfCX2ZGBlMsxKwQ0s1rts1xvr1/ESO0Iu/xONwMH/fx1Ze/ZWVii3Jtm6L7P4L/FhkxrcDYfxhzR8888EUOpW1q15/U4JF7KVmwiW3gbgxIaPTIzs+B
O/5Nj8PmhdCtI4oh2E5++gCjbNz4xTJut2Qj69JeKRY4EqBLVlZklvF8imYn3DaKdqWKVABHOuCinaSPdx1eg09c0gmHsnBmLSfnWwB/uhYs6qs1O/YeQAgh root@loca
lhost.localdomain
```

### 6.2 创建用于连接gitlab的凭证

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210509233901935.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210509233921895.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210510102726938.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021051010280068.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

启动jenkins的用户，自动获取名，这里是root

### 6.3 新建任务配置gitlab

在jenkins新建任务，配置源码管理git,配置 gitlab仓库的ssh地址，配置ssh的私钥凭证。 
新建任务：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210510102116884.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

配置gitlab地址和凭证：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210510102310429.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)



前往gitlab仓库地址，配置ssh keys,将我们对jenkins生成秘钥的公钥配置给制定仓库。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210510095419587.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

可以指定我们要构建的分支：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210517225845287.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

### 6.4 配置自动构建部署

首先配置我们要部署的服务器地址：点击系统管理——>系统设置——>找到Publish over ssh ，当然如果使用其他方式另说。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210517231841688.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

配置完服务器地址，进入我们创建的任务页面，配置构建环境：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210517232713603.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

添加构建完成后操作，点击 add post build step: 通过ssh选择发送文件或执行脚本

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210517230725434.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

这里选择我们刚才创建的服务器名称：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210517230507237.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)



![在这里插入图片描述](https://img-blog.csdnimg.cn/20210517230643409.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)



保存后，点击立即构建。不出意外，就成功了。当然，意外总是有的

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210517230105215.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)



构建后，会先从gitlab仓库拉取代码，然后用我们配置的maven ，jdk进行构建打包，然后讲jar包通过ssh服务发送到目标服务器，然后执行构建脚本，启动服务。 我们可以通过页面点击去查看命令行输出，

- 如果拉取代码环节失败了，检查gitlab配置
- 如果打包环节失败了。查看maven配置及命令
- 如果发送成功，但是执行脚本失败了。那就得好好看看脚本内容了，这一步有很多坑。

### 6.5 window服务器如何使用ssh

如果我们的目标服务器是windows,那么我们还要用 ssh的方式将jar包发送到服务器目录，并且远程执行脚本，就需要我们的windows服务器开启ssh服务，这个有很多种方式，我选择在服务器上安装 powerShellServer这个软件，手动起一个ssh服务。 当然直接命令行安装openssh并开启ssh服务也可以，但是我试过有的服务器执行脚本不能执行call ,只能执行start，这让我很郁闷，所以就选择了powerShellServer。

参考文章：

[windows服务器里实现通过ssh工具SecureCRT](https://blog.csdn.net/achenyuan/article/details/81166526)

[Jenkins连接Window服务器，上传jar并启动](https://blog.csdn.net/achenyuan/article/details/81181347)

基本思路：

- 安装powershell server 
- 配置powershell server 
- 远程通过ssh连接

## 7、配置gitlab webhooks自动构建

现在我们要实现的是：开发人员提交代码到gitlab，由gitlab通过webhook触发jenkins的构建任务，编译打包发布。

### 7.1 jenkins配置

首先：安装gitlab插件，这个上面讲插件部分已经说过了。

![image-20210518223721328](C:\Users\Answer\AppData\Roaming\Typora\typora-user-images\image-20210518223721328.png)

要特别注意，这里有个url,`http://xxxxxx:9010/project/jenkinsdemo`  正常情况下这个路径应该是当前创建的这个任务的地址，也就是jenkins配置的这个项目地址，但是我拿这个去访问根本访问不到怎么回事？为了这个问题查了很多资料，搞了半天没搞懂，为什么自动生成的路径和项目实际路径不符合呢？

项目实际路径：`http://xxxxxx:9010/job/jenkinsdemo` ，我就日了狗了，为啥一个是job,一个是project呢，

这让我百思不得其解。不管了，继续。

### 7.2配置gitlab

首先使用管理员账号，打开允许Webhook和服务队本地网络的请求设置，如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210518222628994.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)



进入对应代码库，设置webhook ：

![image-20210518224338449](C:\Users\Answer\AppData\Roaming\Typora\typora-user-images\image-20210518224338449.png)



![image-20210518224553552](C:\Users\Answer\AppData\Roaming\Typora\typora-user-images\image-20210518224553552.png)

这样最终：gitlab网址配的是：`http://xxxxxx:9010/job/jenkinsdemo` 这个真实地址，不是那个自动生成的地址，然后配上生成的token，点击测试，如果返回200 ok,说明这个webhook已经通了。



## 8、pipeline和blue ocean



# 二、出现问题记录

- 插件无法下载的问题
- 版本冲突的问题
- gitlab 无法正常使用webhook的问题
- CRFS跨站请求伪造的问题，导致无法使用webhook

下面简单说下，如何解决高版本无法关闭CRFS设置的问题，Jenkins版本自2.204.6以来的重大变更有：删除禁用 CSRF 保护的功能。 

老版本：直接在系统管理——>全局安全配置中关闭即可。

新版本：配置启动参数：

- war包方式直接启动，添加如下参数

  ```shell
  java -Dhudson.security.csrf.GlobalCrumbIssuerConfiguration.DISABLE_CSRF_PROTECTION=true -jar   xxxx.jar
  ```

- 其他的比如tomcat或k8s 等在启动参数里面加上它即可。

重启，就会看到CRFS关闭了。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210518225744447.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)



