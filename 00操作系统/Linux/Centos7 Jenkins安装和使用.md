# Centos7安装配置 Jenkins 2.282 

[toc]

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



## 5、系统配置：jdk,maven，gitlab,sonarqube

## 6、手动构建编译发布测试

## 7、配置gitlab webhooks自动构建



# 二、出现问题记录

### 4、注意插件版本与jenkins问题



# 插件下载-国内网络问题解决



1、访问ip地址+端口

2、解锁jenkins,获取密码-- 0562ff5005af4a0f8b76aa14fff899da

3、出现离线，--访问插件管理，切换源-清华源--- submit ,点击check now 下载default.json

4、修改default.json文件, 将https换为http,将updates.jenkins.io/download 换为mirrors.tuna.tsinghua.edu.cn/jenkins   不然还是下载的国外

5、重启jenkins.  http://192.168.42.133:9000/restart 

6、安装插件，结果失败了，就跳过

7、创建管理员用户，密码

8、设置实例名jenkins, 保存完成

9、安装插件-- 

汉化、等



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

安装汉化包：

localization-zh-cn.hpi

trilead-api.hpi

安装git：

-  SSH Credentials  1.18.2
-  git-client 3.7.1
-  git 4.7.1

安装maven：构建工具

- Maven Integration 3.10

安装sonarqube scanner:代码质量管理

- SonarQube Scanner

发布插件：发布到远程服务器

- Publish Over CIFS 
- Publish Over FTP
- Publish Over SSH



系统配置
系统管理--》全局工具配置 

- 1.配置jdk 

- 2.配置maven 

- 3.配置sonar 

  1、配置sonarqube服务器，系统管理-系统设置-sonar server ,配置服务器名称和令牌

  2、配置sonarqube scanner

- 4.邮件配置  zmwg yfyi llzw bahe  qq邮箱授权码

  系统管理——>系统设置——>邮件通知-——>smtp服务器 smtp.qq.com -——>用户默认邮件后缀 @qq.com——>勾选使用SMTP认证——>输入邮箱和授权码 -——>勾选ssl协议——>输入SMTP端口465 ， Reply-To Address发件者邮箱   ——> 之后测试一下配置，无误即可  

- 配置gitlab授权

  Credentials--》system--》Global credentials

这里，生成密钥，会在home目录生成.ssh目录及密钥 id_rsa 和id_rsa.pub。

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

- 在jenkins新建任务，配置源码管理git,配置 gitlab仓库的ssh地址，配置ssh的私钥凭证。 

配置jdk,maven

echo  $JAVA_HOME  查看java安装目录



9faa142aaff7d4cc76e91d3712e810cef525f7dd        jenkins   sonarqube 令牌



--------

在应用服务器上，运行我们的项目。

```shell
nohup java -jar jenkinsdemo-0.0.1-SNAPSHOT.jar > /dev/null 2>&1 &
#后台守护进程运行jar， 不管正确还是错误的输出都重定向/dev/null也就是控设备。 
```

- nohup 守护进程方式运行
- /dev/null 表示空设备文件
- 0 表示stdin标准输入
- 1 表示stdout标准输出
- 2 表示stderr标准错误
- 最后的& 表示后台运行



### window服务器安装 powerhell server 使用ssh

- powershell server 配置，并且配置公钥
- jenkins服务器 系统设置配置ssh服务器，配置私钥
- 添加ssh服务器，windows服务器的地址。 



构件任务，发布到windows服务器，并启动jar包。





# 构建后发送到sonarqube

Excute SonarQube Scanner;



sonarqube整合
sonar.projectKey=xdclass
sonar.projectName=xdclass
sonar.projectVersion=1.0
sonar.sourceEncoding=UTF-8
sonar.modules=java-module
\# Java module
java-module.sonar.projectName=test
java-module.sonar.language=java
\# .表示projectBaseDir指定的目录
java-module.sonar.sources=src
java-module.sonar.projectBaseDir=.
java-module.sonar.java.binaries=target/  







# 设置webhook,配置gitlab,提交代码自动构件

- 安装插件GitLab