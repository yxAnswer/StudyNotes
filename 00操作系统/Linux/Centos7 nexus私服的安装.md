[toc]

# Centos7  nexus私服的安装详解

## 1、安装JDK

### 1.1查看当前linux是否安装java

安装之前先查看原linux 是否安装jdk ,

`rpm -qa | grep -i java`  如果没有就安装，如果有，就卸载

`rpm -e --nodeps 要卸载的软件名`

### 1.2 上传jdk到linux文件目录

我们一般把软件安装到/usr/local 下，这里我是创建了java文件夹，与java相关的放到这里了。

我装的是xshell工具，同时装了xftp,可以直接左右拖拽，将文件放到linux服务器文件目录下，非常方便。比如，现在我们把 `jdk-8-linux.gz`安装包，拖到Linux下  `/usr/local/java` 文件夹下。当然一开始是没有这个java文件夹的，需要我们手动创建`mkdir java` 。然后我们需要做的就是解压：

`tar -xvf jdk-8-linux.gz`  解压完成为 jdk1.8.0_201 这就是我们的jdk

>如果使用secure crt   传输出现 rz命令 提示command not found 解决方法
>
>yum -y install lrzsz     先安装传输工具

### 1.3 配置环境变量

按两种方式设置吧：一个是系统级别，所有用户通用。一个是设置到用户级别。

#### （1）修改/etc/profile 系统的配置文件

`cd /etc` 我们打开etc路径下的profile文件

`vim profile` 或者直接 `vim /etc/profile`

直接编辑 /etc/profile 文件，然后在文件末尾 添加以下： 权限不够的话使用sudo vi /etc/profile

```
#set java environment
JAVA_HOME=/usr/local/java/jdk1.8.0_201
CLASSPATH=.:$JAVA_HOME/lib
PATH=$JAVA_HOME/bin:$PATH
export JAVA_HOME CLASSPATH PATH
```

**配置完，需要重新加载配置文件，执行命令：**

`source /etc/profile`

注意：这里的JAVA_HOME 的路径是你自己安装的jdk的路径，根据自己实际情况修改，我这里是安装到了/usr/local/java/jdk1.8.0_201 

此种配置方法是linux系统所有用户通用这个java环境

#### （2）修改 .bash_profile文件

此方法，只有当前配置的账号可用

`cd ~` 进入用户根目录。然后打开编辑 .bash_profile文件。  `vim .bash_profile` 

然后将配置信息复制进去。

```
#set java environment
JAVA_HOME=/usr/local/java/jdk1.8.0_201
CLASSPATH=.:$JAVA_HOME/lib
PATH=$JAVA_HOME/bin:$PATH
export JAVA_HOME CLASSPATH PATH
```

如果出现无法执行二进制的错误，注意需要jdk版本和linux 版本一致，32都是32,64 都是64

## 2、安装maven

同安装jdk一样

- 上传安装包

- 解压到/usr/local/maven

- 配置环境变量

  - ```shell
    # set maven environment
    MAVEN_HOME=/usr/local/maven/apache-maven-3.5.3/bin
    PATH=$MAVEN_HOME:$PATH
    export MAVEN_HOME  PATH
    ```

- 重新加载`source /etc/profile`

## 3、安装nexus

### 3.1 上传安装包到服务器

nexus下载安装包：

官网下载地址：[https://www.sonatype.com/download-oss-sonatype  ](https://www.sonatype.com/download-oss-sonatype  )

解压到/usr/local/

`tar -xvf nexus-3.12.1-01-unix.tar.gz -C /usr/local/  `

### 3.2 nexus文件目录介绍：

```shell
[root@localhost local]# ll
drwxr-xr-x.  3 root root    69 4月  25 01:19 bin     
drwxr-xr-x.  2 root root    25 4月  25 01:19 deploy
drwxr-xr-x.  7 root root    98 4月  25 01:22 etc       
drwxr-xr-x.  4 root root  4096 4月  25 01:19 lib
-rw-r--r--.  1 root root 39222 6月   8 2018 LICENSE.txt
-rw-r--r--.  1 root root   395 6月   8 2018 NOTICE.txt
drwxr-xr-x.  3 root root  4096 4月  25 01:19 public
drwxr-xr-x. 21 root root  4096 4月  25 01:19 system
```

- NOTICE.txt/OSS-LICENSE.txt/PRO-LICENSE.txt 有关许可证和版权申明的文件。
- bin 此目录包含nexus的启动脚本和与启动相关的配置文件，其中的nexus文件是nexus的启动文件
- etc 配置文件目录
- lib依赖库目录 
- public 公共资源目录
- system 此目录包含构成nexus的所有组件和插件

```shell
[root@localhost nexus-3.12.1-01]# cd etc
[root@localhost etc]# ll
总用量 16
drwxr-xr-x. 2 yangxu yangxu 4096 4月  25 01:19 fabric
drwxr-xr-x. 2 yangxu yangxu 4096 4月  25 01:19 jetty
drwxr-xr-x. 2 yangxu yangxu 4096 4月  25 01:19 karaf
drwxr-xr-x. 2 yangxu yangxu   49 4月  25 01:19 logback
-rw-r--r--. 1 yangxu yangxu  341 6月   8 2018 nexus-default.properties
drwxr-xr-x. 2 yangxu yangxu   25 4月  25 01:19 ssl

# nexus-default.properties 这就是nexus的配置文件
```

### 3.3 修改nexus配置文件

```shell
vim /usr/local/nexus-3.12.1-01/etc/nexus-default.properties #修改对应的端口

## DO NOT EDIT - CUSTOMIZATIONS BELONG IN $data-dir/etc/nexus.properties
##
# Jetty section
application-port=8081
application-host=0.0.0.0
nexus-args=${jetty.etc}/jetty.xml,${jetty.etc}/jetty-http.xml,${jetty.etc}/jetty-requestlog.xml
nexus-context-path=/

# Nexus section
nexus-edition=nexus-pro-edition
nexus-features=\
 nexus-pro-feature
```

如上所示，可以修改端口,默认端口 8081

### 3.4 修改防火墙，开放端口

```shell
#开放端口
firewall-cmd --zone=public --add-port=8081/tcp --permanent   
#重启防火墙
firewall-cmd --reload
#查看端口号是否开启
firewall-cmd --query-port=8081/tcp
```

## 3.5 启动 nexus 服务

```shell
cd /usr/local/nexus-3.12.1-01/bin  #进入nexus bin目录
./nexus start   #启动nexus
#查看下有没有启动起来
ps aux|grep  nexus

#注意：如果是root用户，nexus会出现 警告：
WARNING:Detected execution as "root" user. This is NOT  recommended！
#尽量不要使用root用户去开启服务，那么我们需要新建用户，或者用别的用户开启nexus服务，比如：

useradd yangxu #新建一个用户 yangxu
chown -R yangxu:yangxu /usr/local/nexus-3.12.1-01 #将nexus-3.12.1-01目录所有权所属组给新用户
chown -R yangxu:yangxu /usr/local/sonatype-work #这个也是nexus的目录，需要给权限

#然后就 开启服务吧
cd /usr/local/nexus-3.12.1-01/bin
./nexus start 
```



## 3.6 登录nexus 网站服务

http://xxxxx:8081   访问nexus服务所在的服务器ip+设置的端口号，访问

默认账号：admin 密码 ： admin123



## 3.7 修改 ulimit 

当我们登录以后，会出现这么一串提示：

**System Requirement:maxfile descriptors [4096] likely too low,increase to at least [65536] **

所以需要修改下：

```shell
vim /etc/security/limits.conf
#新增两行，然后保存
* soft nofile 65536
* hard nofile 65536 
```

重新启动：`./nexus start`

































