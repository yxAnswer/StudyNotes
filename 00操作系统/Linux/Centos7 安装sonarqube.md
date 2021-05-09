[toc]

# Centos7 安装Sonarqube7.7

官网[https://docs.sonarqube.org/7.7/requirements/requirements/](https://docs.sonarqube.org/7.7/requirements/requirements/)

由于使用虚拟机装并且习惯用myql,所以这里选择Sonarqube7.7看官方文档，7.7以上就不支持mysql了。

环境选择：jdk 11 (8也可以), MYSQL5.7 , 另外还支持（PostgreSQL,SQL Server,Oracle等，具体看官方文档选择版本和环境 ）

## 1、安装jdk11

### 1.1查看当前linux是否安装java

安装之前先查看原linux 是否安装jdk 

```shell
rpm -qa | grep -i java  #如果没有就安装，如果有，就卸载
rpm -e --nodeps [要卸载的软件名] 
```

### 1.2 上传jdk到linux文件目录

下载jdk,官网[https://www.oracle.com/java/technologies/javase-jdk11-downloads.html](https://www.oracle.com/java/technologies/javase-jdk11-downloads.html)

我习惯把软件安装到/usr/local 下，把java安装到/usr/loca/java目录下

```shell
 cd /usr/local
 mkdir java
 tar -xvf jdk-11.0.10_linux-x64_bin.tar.gz  -C /usr/local/java #解压到你要安装的路径下，这里是/usr/local/java
```

>如果使用secure crt   传输出现 rz命令 提示command not found 解决方法
>
>yum -y install lrzsz     先安装传输工具

### 1.3 配置环境变量

修改/etc/profile 系统的配置文件 `vim /etc/profile`

文件末尾 添加以下： 权限不够的话使用sudo vi /etc/profile

```
#set java environment
JAVA_HOME=/usr/local/java/jdk-11.0.10
CLASSPATH=.:$JAVA_HOME/lib
PATH=$JAVA_HOME/bin:$PATH
export JAVA_HOME CLASSPATH  PATH
```

**配置完，需要重新加载配置文件，执行命令：**

`source /etc/profile`

## 2、 安装mysql

下载地址：[https://dev.mysql.com/downloads/repo/yum/](https://dev.mysql.com/downloads/repo/yum/)

可参考我的另一篇博客[https://blog.csdn.net/u011138190/article/details/88821322](https://blog.csdn.net/u011138190/article/details/88821322)

### 2.1 下载mysql 的yum源

```shell
wget http://repo.mysql.com/mysql80-community-release-el7-3.noarch.rpm
rpm -ivh mysql-community-release-el7-5.noarch.rpm
```

### 2.2修改yum源，设置版本(可选)

当我们安装mysql的yum源后，会在/etc/yum.repos.d/  多出两个文件：`mysql-community.repo `和`mysql-community-source.repo `,可以设置我们要安装的mysql版本，这里编辑它，将mysql5.7 enabled设为1，mysql8.0 enabled设为0 即可。因为默认是安装8.0

`vim  /etc/yum.repos.d/mysql-community.repo  `

```shell
# Enable to use MySQL 5.7
[mysql57-community]
name=MySQL 5.7 Community Server
baseurl=http://repo.mysql.com/yum/mysql-5.7-community/el/7/$basearch/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql

[mysql80-community]
name=MySQL 8.0 Community Server
baseurl=http://repo.mysql.com/yum/mysql-8.0-community/el/7/$basearch/
enabled=0
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
```

```shell
yum repolist all |grep mysql  #查看mysql 启用情况
yum repolist enabled | grep mysql  #查看mysql已启用的安装
```

### 2.3 执行安装

**卸载已经安装的mysql**

```shell
yum list installed | grep mysql  #是有已安装mysql,找到它卸载
yum -y remove [软件名]
```

**卸载centos7自带的 mariadb 数据库:**

```shell
rpm -qa | grep mariadb
#如果找到，就卸载
rpm -e --nodeps mariadb-libs-5.5.68-1.el7.x86_64
```

**安装：**

```shell
yum install mysql-community-server
```

**创建用户，专门用于管理mysql:**

```shell
useradd mysql
passwd mysql  #然后输入密码
```

**权限设置：**

```shell
chown mysql:mysql -R /var/lib/mysql   #因为数据库文件在这，需要给权限,这里创建了mysql这个用户以及mysql组
chmod -R 777 /var/lib/mysql   #设置可写权限
```

**初始化并启动：**

```shell
mysqld --initialize
systemctl start mysqld.service
```

**查看mysql 运行状态：**

`systemctl status mysqld`  看到active (running) 表示正常。

```shell
[sonar@localhost yum.repos.d]$ systemctl status mysqld
● mysqld.service - MySQL Server
   Loaded: loaded (/etc/systemd/system/mysqld.service; enabled; vendor preset: disabled)
   Active: active (running) since 六 2021-05-08 18:01:03 CST; 43min ago
     Docs: man:mysqld(8)
           http://dev.mysql.com/doc/refman/en/using-systemd.html
 Main PID: 3332 (mysqld)
   CGroup: /system.slice/mysqld.service
           └─3332 /usr/sbin/mysqld --defaults-file=/etc/my.cnf
```

 **验证MySql的安装**

```shell
[sonar@localhost yum.repos.d]$ mysqladmin --version
mysqladmin  Ver 8.42 Distrib 5.7.34, for Linux on x86_64
```

执行这条命令，如果未输出任何信息，说明未安装成功。

### 2.4 配置mysql

**修改root密码**

mysql5.7安装后会生成一个默认密码，如果直接登录可能报错。

ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)，因为mysql5.7给root用户默认创建了一个密码，存在log日志里面，执行下面命令找出。

```shell
grep "temporary password" /var/log/mysqld.log 

2021-05-07T15:27:11.781922Z 1 [Note] A temporary password is generated for root@localhost: ;(fA9tbfoL*;

#这里的;(fA9tbfoL*; 就是默认密码
```

`mysql -u root -p ;(fA9tbfoL*;  `登录mysql，执行操作会出现如下错误：

`ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.`

需要我们重置root密码：

```shell
alter user 'root'@'localhost' identified by 'Root_11'; #密码有规则，太简单会报错
或set password for 'root'@'localhost'=password('Root_11');
```

如果出现密码强度不够：ERROR 1819 (HY000): Your password does not satisfy the current policy requirements:

```shell
#密码的长度是由validate_password_length决定的,但是可以通过以下命令修改
set global validate_password_length=4;
#validate_password_policy决定密码的验证策略,默认等级为MEDIUM(中等),可通过以下命令修改为LOW(低)
set global validate_password_policy=0;
```


修改完成后密码就可以设置的很简单，比如123456。

**添加账号，设置权限**

```shell
#添加一个sonar账号，用户sonarqube进行数据的连接
grant all on *.* to sonar@'%' identified by "112233"
```

## 3、sonarqube安装配置

### 3.1 下载安装包

官网地址：[https://www.sonarqube.org/downloads/](https://www.sonarqube.org/downloads/)  选择要下载的版本，这里我选择的是sonarqube 7.7  

### 3.2 解压安装

```shell
unzip sonarqube-7.7.zip   #unzip解压
mv sonarqube-7.7  /usr/local  #移动到/usr/local目录下
```

这里sonarqube是zip压缩包，需要用到unzip命令，如果没有，需要安装：

```shell
yum install unzip #安装unzip
```

### 3.3 修改mysql配置

sonarqube我们选择mysql数据库，需要创建对应的database;

```shell
mysql -usonar -p   #输入密码登录mysql
CREATE DATABASE sonar DEFAULT CHARACTER SET utf8; #创建名为sonar的数据库
```

在前面安装mysql时我们已经创建了 sonar用户，后面会用到。

### 3.4 修改sonarqube配置文件

```shell
vim /usr/local/sonarqube-7.7/conf/sonar.properties  #编辑配置文件
#重点修改一下几个,数据库和端口号配置
sonar.jdbc.username=sonar
sonar.jdbc.password=112233
sonar.jdbc.url=jdbc:mysql://192.168.42.133:3306/sonar?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance&useSSL=false

sonar.web.context=/sonar
sonar.web.host=0.0.0.0
sonar.web.port=9000
```

### 3.5 启动sonarqube

由于sonarqube用到了elasticsearch，所以不能用root用户，需要创建新用户用于启动sonarqube。

```shell
useradd sonar  #新增soanr用户 
passwd sonar  #设置密码
```

赋予用户目录权限：

```shell
chown -R sonar:sonar sonarqube-7.7  #将安装目录sonarqube权限赋给sonar
```

启动sonarqube:

```shell
su sonar
cd /usr/local/sonarqube-7.7/bin/linux-x86-64  #进入linux bin目录
./sonar.sh start #启动sonarqube
```

其他命令：

```shell
./sonar.sh start #启动服务
./sonar.sh restart   #重启服务
./sonar.sh stop #停止服务  
```

### 3.6 防火墙配置

上面设置的sonarqube ,端口为9000,需要防火墙开放。

```shell
#开放9000端口
firewall-cmd --zone=public --add-port=9000/tcp --permanent   
#重启防火墙
firewall-cmd --reload
#查看端口号是否开启
firewall-cmd --query-port=9000/tcp
#查看所有打开的端口
firewall-cmd --zone=public --list-ports
```

## 3、界面访问测试

### 3.1 登录

主机浏览器输入：http://192.168.42.133:9000/sonar   服务器ip+端口+/sonr

登录：默认的账号：admin 密码：admin 

### 3.2 汉化

当然登录进来是英文的，这时候可以安装汉化包。因为我在线安装老是失败，可能是因为网络问题，所以我选择手动安装插件。

github地址：[https://github.com/xuhuisheng/sonar-l10n-zh](https://github.com/xuhuisheng/sonar-l10n-zh)

选择支持 sonarqube7.7 插件版本，这里选择 [sonar-l10n-zh-plugin-1.26](https://github.com/xuhuisheng/sonar-l10n-zh/releases/tag/sonar-l10n-zh-plugin-1.26)

将jar包 放到 sonarqube安装目录的 `/usr/local/sonarqube-7.7/extensions/plugins`

然后重启 sonarqube 

### 3.3 创建项目测试

新建一个测试项目：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210508111953314.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210508112205118.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)



![在这里插入图片描述](https://img-blog.csdnimg.cn/20210508112112521.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210508112249118.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)



然后，随便找一个java项目，在根目录执行 这段命令。 

注意：如果是windows系统去执行，不要用powershell,最好用git bash 的命令行去执行。 执行完会自动刷新页面，可以看到效果。

## 4.Sonarqube出现问题汇总：

### 4.1 出现启动不起来：

1、先检查是否用root用户启动的，sonarqube不能使用root用户启用，创建新用户设置权限，重新启动。

2、**sonar.sh console 命令查看日志** ，

出现：` WrapperSimpleApp: Encountered an error running main: java.nio.file.AccessDeniedException: /usr/local/sonarqube-7.7/temp/conf/es/elasticsearch.yml`

解决方案： 手动删除/usr/local/sonarqube-7.7/temp 下的所有文件，重启



### 4.2 MYSQL的max_allowed_packet

出现：Caused by: com.mysql.jdbc.PacketTooBigException: Packet for query is too large (5856124 > 4194304). You can change this value on the server by setting the max_a  owed_packet' variable.

mysql的max_allowed_packet 不够大，需要增大。

解决方案：

修改mysql 参数，max_allowed_packet设为最大。最大为1G，超过1G也按1G算。

**临时解决方案：**

修改该值。但是重启Mysql还是会恢复到默认值,所以需要写在配置文件当中

```shell
show variables like ‘max_allowed_packet’;
set global max_allowed_packet = 10 * 1024 * 1024;
```

**永久性解决方案:**

```shell
vim /etc/my.cnf   #编辑配置文件
#添加如下
[mysqld]
max_allowed_packet = 1024M
```

完整的文件如下：

```shell
# For advice on how to change settings please see
# http://dev.mysql.com/doc/refman/5.7/en/server-configuration-defaults.html

[mysqld]
max_allowed_packet=1024M
#
# Remove leading # and set to the amount of RAM for the most important data
# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
# innodb_buffer_pool_size = 128M
#
# Remove leading # to turn on a very important data integrity option: logging
# changes to the binary log between backups.
# log_bin
#
# Remove leading # to set options mainly useful for reporting servers.
# The server defaults are faster for transactions and fast SELECTs.
# Adjust sizes as needed, experiment to find the optimal values.
# join_buffer_size = 128M
# sort_buffer_size = 2M
# read_rnd_buffer_size = 2M
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
```

重启mysql  `service mysqld restart`

**一定要重启SonarQube，否则虽数据库配置已变更，但对SonarQube的数据库连接不会生效**

```shell
cd  /usr/local/sonarqube-7.7/bin/linux-x86-64
./sonar.sh restart
```



