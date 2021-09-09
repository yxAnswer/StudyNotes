# linux rpm方式安装jdk+mysql

[TOC]



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

## 2、rpm方式安装mysql

当然以下只是一种安装方式，其他的大同小异。可以用rpm 也可以用yum. 注意二进制安装包的下载。

注意：centos6  一般没啥问题，需要调整下安装顺序，centos 7可能需要先卸载MariaDB这个自带的数据库库。。 总之都需要注意安装顺序和 依赖包缺失的问题。如果不顺利，百度，google答案，这里记录下centos6 安装用 rpm 安装mysql方式。。不过还是推荐yum,应该比较省事。

### 1、远程连接Linux

远程连接linux有很多种方式，工具有Puttty、secureCRT、SSH Secure 等等，我在windows上装的是Xshell,通过IP或者ssh的方式登陆Linux，这一步基本都会，不做说明了。大家可以去网上找工具。

### 2、拷贝mysql到服务器目录

我装的是xshell工具，同时装了xftp,可以直接左右拖拽，将文件放到linux服务器文件目录下，非常方便。比如，现在我们把 `mysql5.6.tar`安装包，拖到Linux下  `/usr/local/mysql` 文件夹下。当然一开始是没有这个mysql文件夹的，需要我们手动创建`mkdir mysql` 。然后我们需要做的就是解压：

`tar -xvf mysql5.6.tar`

### 3、查看本机是否安装，并卸载自带软件

解压完安装包，接下来做的就应该是安装了，但是有的时候服务器镜像默认给我们安装了一个mysql版本，需要我们先卸载掉，再安装

查看本机是否安装

`rpm -qa | grep -i mysql`   稍微说下这个命令，rpm -qa查处所有应用 |管道命令 将结果给grep命令进行 通过mysql名称进行过滤。 如果命令行有输出，说明已经安装了，需要卸载。如果没有直接安装。

卸载已安装的mysql:

`rpm -e --nodeps mysql-ibs-5.1.i686`  使用命令rpm -e ---nodeps  +要卸载的软件名 ，讲自带mysql卸载掉

### 4、安装mysql ，启动并设置开机启动

#### 4.1安装：如果失败，查看常见错误

安装命令：`rpm -ivh 要安装的文件名`

比如，我解压的mysql有两个要安装一个是server端，另一个是client端

```
rpm -ivh MySQL-server-5.6.22-1.el6.i686.rpm //安装服务端
rpm -ivh MySQL-client-5.6.22-1.el6.i686.rpm //安装客户端
```

当然你可以选择 yum命令安装。都一样的

如果此时报错，说缺少依赖包。根据错误提示去安装相应的依赖包，然后再安装mysql

```
yum -y install libaio.so.1 libgcc_s.so.1 libstdc++.so.6
yum  update libstdc++-4.4.7-4.el6.x86_64
```



#### 4.2 启动mysql 服务

`service mysql start` 执行完这条命令，我们可以看下 ps -ef 我们的mysql启动了。但是这个是单次启动，我们关机重启，mysql服务又关闭了。所以我们需要为mysql 设置为开机自动启动，把它加入系统服务

加入系统服务：`chkconfig --add mysql`

设置自动登陆：`chkconfig mysql on`

### 5、登录MySQL，修改初始密码

#### 5.1 登录

mysql安装完会自动生成一个随机密码，从安装日志可以看到，存储在/root/.mysql_secret  ,这是个隐藏文件，可以使用ls -a 查看。我们用vim 打开它，就可以看到初始密码，然后复制，用此密码登陆mysql

`mysql -u root -p 初始密码`

#### 5.2 修改初始密码

修改密码必须再登录了mysql以后我们再修改。执行修改命令

`set password=password('新密码')`  

我们自己设置自己的密码，比如我设置为123456  `set password = password('123456')`

### 6、开启远程登录权限，开放端口

现在我们已经成功安装，登录，修改密码。 如果我们远程连接到服务器，现在是已经可以进行数据库的操作了。但是我们是无法通过代码远程连接数据库，也不能通过比如 sql yog 或者navicat等数据库工具连接我们的mysql.因为：

1、我们的mysql为了安全，默认是没有开启远程登录的权限的。

2、我们mysql的端口没有开发，3306是mysql 默认端口，但是防火墙并没有开放它，任何软件都访问不到。

#### 6.1 开启远程登录权限

登录mysql，执行以下命令：（不用记，用到的时候复制下）

```
//首先给账号开放远程登录权限
grant all privileges on *.* to 'root' @'%'  identified by 'admin';
//第二步，需要刷新权限才起作用
flush privileges;

```

第一条命令，我们给root 账号授权远程登录，并且给他指定的密码是admin，当然你也可以是其他任何的密码，这就避免了将linux 服务器用户的真实密码暴露出去。

第二条命令是刷新我们的权限

#### 6.2 开放3306 端口

只开启权限，当然还是无法访问。需要开放linux 对外访问的端口3306.注意：这里的操作是在linux命令行下，而不是mysql 下面。所以第一步，退出mysql，执行下面命令。

`/sbin/iptables -I INPUT -p tcp --dport 3306 -j ACCEPT`

上边命令是开放了3306，但是并不是永久开放，重启又关闭了

`/etc/rc.d/init.d/iptables save ---将修改永久保存到防火墙中`

所以，我们需要将它永久的保存到防火墙中。

### 7、远程连接测试

我使用的是Navicat， 输入服务器的地址，输入端口3306 ，输入用户名root，输入刚才授权的密码 admin，连接成功。接下来的就是数据库操作了。

### 8、常见错误--

1、有可能发生32位包和系统位数不统一失败，如果安装失败检查系统`getconf LONG_BIT` ，确保软件和系统位数一致

2、原有数据库卸载不干净

3、centos 7 默认数据库是mariadb  ,需要手动强制卸载，再装mysql

4、常见可能会有mysql 依赖包缺少，这个需要缺什么安装什么。比如

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190326093856431.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

5、安装是会提示安装顺序出错，比如 server 需要先装common，需要先装client等，具体查看提示，依次安装依赖包

这里总结的挺好的 [http://blog.itpub.net/31015730/viewspace-2152272/](http://blog.itpub.net/31015730/viewspace-2152272/)

## 3、RPM命令

**rpm： 执行安装包**

> 二进制包（Binary）以及源代码包（Source）两种。二进制包可以直接安装在计算机中，而源代码包将会由 RPM自动编译、安装。源代码包经常以src.rpm作为后缀名。

**常用命令组合：**

**－ivh**：安装显示安装进度--install--verbose--hash
**－Uvh**：升级软件包--Update；
**－qpl**： 列出RPM软件包内的文件信息[Query Package list]；
**－qpi**：列出RPM软件包的描述信息[Query Package install package(s)]；
**－qf**：查找指定文件属于哪个RPM软件包[Query File]；
**－Va**：校验所有的 RPM软件包，查找丢失的文件[View Lost]；
**－e**： 删除包

**rpm -q samba** //查询程序是否安装

**rpm -ivh /media/cdrom/RedHat/RPMS/samba-3.0.10-1.4E.i386.rpm** //按路径安装并显示进度

**rpm -ivh --relocate /=/opt/gaim gaim-1.3.0-1.fc4.i386.rpm**    //指定安装目录

**rpm -ivh --test gaim-1.3.0-1.fc4.i386.rpm**　　　 //用来检查依赖关系；并不是真正的安装；

**rpm -Uvh --oldpackage gaim-1.3.0-1.fc4.i386.rpm** //新版本降级为旧版本

**rpm -qa | grep httpd**　　　　　 ＃[搜索指定rpm包是否安装]--all搜索*httpd*

**rpm -ql httpd**　　　　　　　　　＃[搜索rpm包]--list所有文件安装目录

**rpm -qpi Linux-1.4-6.i368.rpm**　＃[查看rpm包]--query--package--install package信息

**rpm -qpf Linux-1.4-6.i368.rpm**　＃[查看rpm包]--file

**rpm -qpR file.rpm**　　　　　　　＃[查看包]依赖关系

**rpm2cpio file.rpm |cpio -div**    ＃[抽出文件]

**rpm -ivh file.rpm** 　＃[安装新的rpm]--install--verbose--hash

**rpm -Uvh file.rpm**    ＃[升级一个rpm]--upgrade

**rpm -e file.rpm**      ＃[删除一个rpm包]--erase

**常用参数：**

**Install/Upgrade/Erase options:**
 -i, --install                     install package(s)
 -v, --verbose                     provide more detailed output
 -h, --hash                        print hash marks as package installs (good with -v)
 -e, --erase                       erase (uninstall) package
 -U, --upgrade=<packagefile>+      upgrade package(s)
 －-replacepkge                    无论软件包是否已被安装，都强行安装软件包
 --test                            安装测试，并不实际安装
 --nodeps                          忽略软件包的依赖关系强行安装
 --force                           忽略软件包及文件的冲突

**Query options (with -q or --query):**
 -a, --all                         query/verify all packages
 -p, --package                     query/verify a package file
 -l, --list                        list files in package
 -d, --docfiles                    list all documentation files
 -f, --file                        query/verify package(s) owning file

**RPM源代码包装安装**

.src.rpm结尾的文件，这些文件是由软件的源代码包装而成的，用户要安装这类RPM软件包，必须使用命令：

rpm　--recompile　vim-4.6-4.src.rpm   ＃这个命令会把源代码解包并编译、安装它，如果用户使用命令：

rpm　--rebuild　vim-4.6-4.src.rpm　　＃在安装完成后，还会把编译生成的可执行文件重新包装成i386.rpm 的RPM软件包。







