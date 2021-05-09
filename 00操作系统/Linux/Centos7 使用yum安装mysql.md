# Centos7 使用yum 安装mysql

[toc]

## 1、下载yum安装源

官网：[https://dev.mysql.com/downloads/repo/yum/](https://dev.mysql.com/downloads/repo/yum/)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190326151953754.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)
如图，目前的版本已经到8了，我们先把yum源下载下来，并且导入到linux服务器目录下。

然后我们执行`rpm -ivh mysql80-community-release-el7-3.noarch.rpm`

**当然还有一种方式**，就是我们直接在线下载：直接安装自己需要的版本

```
wget http://repo.mysql.com/mysql80-community-release-el7-3.noarch.rpm
rpm -ivh mysql-community-release-el7-5.noarch.rpm
```

## 2、修改yum源，设置版本(可选)

当我们安装mysql的yum源后，会在/etc/yum.repos.d/  多出两个文件：`mysql-community.repo `和`mysql-community-source.repo `

> 在MySQL Yum存储库、MySQL社区服务器的不同版本系列托管在不同的子存储库中。最新的GA系列(当前的MySQL 8.0)的子存储库在默认情况下是启用的，所有其他系列(例如，MySQL 5.7系列)的子存储库在默认情况下是禁用的。
>
> 使用此命令查看MySQL Yum存储库中的所有子存储库，并查看哪些子存储库已启用或禁用，

yum repository 安装mysql的方式我们是可以选择自己的软件源的。如果不修改就跳过。/etc/yum.repos.d/  

```
[mysql-connectors-community]
name=MySQL Connectors Community
baseurl=http://repo.mysql.com/yum/mysql-connectors-community/el/7/$basearch/
enabled=1
gpgcheck=1
gpgkey=file:/etc/pki/rpm-gpg/RPM-GPG-KEY-mysql

[mysql-tools-community]
name=MySQL Tools Community
baseurl=http://repo.mysql.com/yum/mysql-tools-community/el/7/$basearch/
enabled=1
gpgcheck=1
gpgkey=file:/etc/pki/rpm-gpg/RPM-GPG-KEY-mysql

# Enable to use MySQL 5.5
[mysql55-community]
name=MySQL 5.5 Community Server
baseurl=http://repo.mysql.com/yum/mysql-5.5-community/el/7/$basearch/
enabled=0
gpgcheck=1
gpgkey=file:/etc/pki/rpm-gpg/RPM-GPG-KEY-mysql

# Enable to use MySQL 5.6
[mysql56-community]
name=MySQL 5.6 Community Server
baseurl=http://repo.mysql.com/yum/mysql-5.6-community/el/7/$basearch/
enabled=1
gpgcheck=1
gpgkey=file:/etc/pki/rpm-gpg/RPM-GPG-KEY-mysql

# Note: MySQL 5.7 is currently in development. For use at your own risk.
# Please read with sub pages: https://dev.mysql.com/doc/relnotes/mysql/5.7/en/
[mysql57-community-dmr]
name=MySQL 5.7 Community Server Development Milestone Release
baseurl=http://repo.mysql.com/yum/mysql-5.7-community/el/7/$basearch/
enabled=0
gpgcheck=1
gpgkey=file:/etc/pki/rpm-gpg/RPM-GPG-KEY-mysql

```

上面我是自己修改了，改为5.7.  主要修改这个enabled就可以。用哪个版本，就把那个版本下改为enabled=1，其他的全部enabled=0; 当然也可以复制粘贴一个新的版本。

**查看不同版本默认启用情况：**

可以看到 enabled 的情况，当前命令其实就是查看/etc/yum.repos.d目录中的  mysql-community.repo和mysql-community-source.repo文件内容

```
[root@iZszxghs0ozok0Z yum.repos.d]# yum repolist all |grep mysql

!mysql-connectors-community/x86_64 MySQL Connectors Community    enabled:     95
mysql-connectors-community-source  MySQL Connectors Community -  disabled
!mysql-tools-community/x86_64      MySQL Tools Community         enabled:     84
mysql-tools-community-source       MySQL Tools Community - Sourc disabled
mysql55-community/x86_64           MySQL 5.5 Community Server    disabled
mysql55-community-source           MySQL 5.5 Community Server -  disabled
!mysql56-community/x86_64          MySQL 5.6 Community Server    enabled:    446
mysql56-community-source           MySQL 5.6 Community Server -  disabled
mysql57-community-dmr/x86_64       MySQL 5.7 Community Server De disabled
mysql57-community-dmr-source       MySQL 5.7 Community Server De disabled

```

修改完查看默认启用的最终版本情况：

```
[root@iZszxghs0ozok0Z yum.repos.d]# yum repolist enabled | grep mysql
!mysql-connectors-community/x86_64 MySQL Connectors Community                 95
!mysql-tools-community/x86_64      MySQL Tools Community                      84
!mysql56-community/x86_64          MySQL 5.6 Community Server                446

```

可以看到，我们默认开启了 5.6 版本，

总结下这几个命令

```
yum repolist all |grep mysql
yum repolist enabled | grep mysql
```

## 3、安装mysql

### 3.1检查是否安装

**卸载之前安装的mysql:**

`yum list installed | grep mysql`或者

`rpm -qa | grep -i mysql`

查看是否已经有安装的mysql，如果有，需要手动卸载掉。

`rpm -e --nodeps  软件名` 或者`yum -y remove 软件名`

**卸载centos7自带的 mariadb 数据库:**

```shell
rpm -qa | grep mariadb
#如果找到，就卸载
rpm -e --nodeps mariadb-libs-5.5.68-1.el7.x86_64
```

### 3.2 安装

安装之前如果没有更新yum 的最好更新下

```shell
yum upgrade   #更新软件源
yum install mysql-community-server #安装mysql
```

添加一个用户，专门负责mysql：

```shell
useradd mysql #新增mysql用户
passwd mysql  #设置密码
```

权限设置：

因为数据库文件在这，需要给权限,这里创建了mysql这个用户以及mysql组，

```shell
chown mysql:mysql -R /var/lib/mysql
```

然后设置可写权限：`chmod -R 777 /var/lib/mysql`  不然innodb报错

初始化并启动mysql:

```shell
mysqld --initialize  #初始化
systemctl start mysqld.service  #启动
```

查看mysql 运行状态：

`systemctl status mysqld`  看到active (running) 表示正常。

```shell
[root@iZszxghs0ozok0Z yum.repos.d]# systemctl status mysqld
● mysqld.service - MySQL Community Server
   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled; vendor preset: disabled)
   Active: active (running) since Sat 2019-03-23 01:40:37 CST; 3 days ago
  Process: 9115 ExecStartPost=/usr/bin/mysql-systemd-start post (code=exited, status=0/SUCCESS)
  Process: 9055 ExecStartPre=/usr/bin/mysql-systemd-start pre (code=exited, status=0/SUCCESS)
 Main PID: 9114 (mysqld_safe)
   CGroup: /system.slice/mysqld.service
           ├─9114 /bin/sh /usr/bin/mysqld_safe --basedir=/usr
           └─9280 /usr/sbin/mysqld --basedir=/usr --datadir=/var/lib/mysql --plugin-dir=/usr/lib64/mysql/plugin --log-error=/var/l...


```

### 3.3 验证MySql的安装

mysqladmin --version

```shell
[root@iZszxghs0ozok0Z yum.repos.d]# mysqladmin --version
mysqladmin  Ver 8.42 Distrib 5.6.43, for Linux on x86_64

```

执行这条命令，如果未输出任何信息，说明未安装成功。

## 4、配置mysql 

### 4.1 设置用户密码

mysql5.7安装后会生成一个默认密码，如果直接登录可能报错。

可能出现： 

`ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)`，因为mysql5.7给root用户默认创建了一个密码，存在log日志里面，执行下面命令找出。

```shell
grep "temporary password" /var/log/mysqld.log 

2021-05-07T15:27:11.781922Z 1 [Note] A temporary password is generated for root@localhost: ;(fA9tbfoL*;

#这里的;(fA9tbfoL*; 就是默认密码
```

`mysql -u root -p ;(fA9tbfoL*;  `登录mysql，进行操作。

执行操作会出现如下错误：

`ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.`

需要我们重置root密码：

```shell
alter user 'root'@'localhost' identified by 'Root_11'; #密码有规则，太简单会报错
或set password for 'root'@'localhost'=password('Root_11');
```

如果出现密码强度不够：`ERROR 1819 (HY000): Your password does not satisfy the current policy requirements`:

```shell
#密码的长度是由validate_password_length决定的,但是可以通过以下命令修改
set global validate_password_length=4;
#validate_password_policy决定密码的验证策略,默认等级为MEDIUM(中等),可通过以下命令修改为LOW(低)
set global validate_password_policy=0;
```


修改完成后密码就可以设置的很简单，比如123456。

### 4.2 添加账号，设置权限

我们默认是用的root权限，如果不想直接用root这个账号远程操作数据库，需要我们手动创建一个账号。

（1）登录mysql ,  mysql -u root -p 

（2）创建账号admin,并将密码设为123456，设置访问权限。

`grant all on *.* to admin@'%' identified by "123456";`

%代表任何客户机都可以连接,localhost代表只可以本机连接

当然可以分步，比如:

`grant all on *.* to admin@'localhost' identified by '123456';`

这里是只添加了账号，外部还是不能访问，我们可以修改权限。

`update user set host = '%' where user = 'admin';`

以上两种方式都是可以的，这里只是说明，避免误解。

创建后在mysql库的user表下查看，有该用户。
但是使用“mysql -u admin -p” 登录，
提示无法登录：ERROR 1045 (28000): Access denied for user 'admin'@'localhost' (using password: YES)

>原因：官网解释
>
>其中两个账户有相同的用户名monty和密码some_pass。两个账户均为超级用户账户，具有完全的权限可以做任何事情。一个账户 ('monty'@'localhost')只用于从本机连接时。另一个账户('monty'@'%')可用于从其它主机连接。
>请注意monty的两个账户必须能从任何主机以monty连接。没有localhost账户，当monty从本机连接时，mysql_install_db创建的localhost的匿名用户账户将占先。结果是，monty将被视为匿名用户。原因是匿名用户账户的Host列值比'monty'@'%'账户更具体，这样在user表排序顺序中排在前面。

所以我们在加一个同样的账号，host设为 localhost即可

`grant all on *.* to admin@'localhost' identified by "123456";`

（3）查看下是否添加或者修改成功

```
use mysql;
select user,host from user where user="admin"
```

这时候应该会输出，

```
mysql> select user,host from user where user='admin';
+-------+------+
| user  | host |
+-------+------+
| admin | %    |
+-------+------+
1 row in set (0.00 sec)

```

(4) 更新数据库

命令：`flush privileges;`

## 5、设置远程登录

由于我们的服务器在远程，比如阿里云，我们设置远程登录，就不用像虚拟机那样了。我们只需要开放阿里云权限策略组的3306 端口。

这个这里就不贴图了，登录自己的阿里云服务器，进入安全组——添加规则——添加mysql协议，默认为3306，授权对象设为 0.0.0.0/0 就可以了。然后用sqlyog或者Navicat试一下。

如果还不行：多半是防火墙的问题，请自行百度+google 很多。

以虚拟机为例，开放3306端口：

```shell
#开放3306端口
firewall-cmd --zone=public --add-port=3306/tcp --permanent   
#重启防火墙
firewall-cmd --reload
#查看端口号是否开启
firewall-cmd --query-port=3306/tcp
#查看所有打开的端口
firewall-cmd --zone=public --list-ports
```

如果没有装防火墙，需要重新设置下（可选，不同情况不一样，保证firewall-cmd可用即可）：

```shell
yum install firewalld systemd -y #安装防火墙
systemctl start  firewalld.service #开启防火墙

yum install -y curl policycoreutils-python openssh-server perl #安装ssh协议
#如果报错，重建数据库
rpm --rebuilddb && yum install -y curl policycoreutils-python openssh-server perl 

systemctl enable sshd #设置ssh服务开机启动
systemctl start sshd #启动ssh服务
firewall-cmd --permanent --add-service=http #添加http服务到firewalld
firewall-cmd --permanent --add-service=https #添加HTTPS服务到firewalld
systemctl reload firewalld #重启防火墙
```

## 6、设置开机启动

- 在`etc/systemd/system `  下面创建mysqld.service 文件

  ```shell
  [root@localhost system]# touch /etc/systemd/system/mysqld.service
  [root@localhost system]# cd /etc/systemd/system
  ```

- 编辑mysqld.service文件,加入下面内容，保存退出

  ```shell
  [root@localhost system]# vim mysqld.service
  
  [Unit]
  Description=MySQL Server
  Documentation=man:mysqld(8)
  Documentation=http://dev.mysql.com/doc/refman/en/using-systemd.html
  After=network.target
  After=syslog.target
  
  [Install]
  WantedBy=multi-user.target
  
  [Service]
  User=mysql
  Group=mysql
  ExecStart=/usr/sbin/mysqld --defaults-file=/etc/my.cnf
  LimitNOFILE = 5000
  ```

- ExecStart=/usr/sbin/mysqld  (这里根据实际情况修改，本机mysql的安装路径)，使用 `which mysqld`命令可查看

  ```shell
  [root@localhost system]# which mysqld
  /usr/sbin/mysqld
  ```

- 设置开启自启动

  ```shell
  systemctl enable mysqld #设置开启自启动
  systemctl start mysqld  #现在启动mysql服务
  ```

- `systemctl enable mysqld`执行后出现（Created symlink from ......）表示加入开机自启成功。如果出错提示已经有了mysql.service ,

## 7、mysql管理常见linux命令

### 7.1.启动命令

```shell
[root@localhost usr]#  service mysqld start
Redirecting to /bin/systemctl start mysqld.service
```

### 7.2.关闭命令

```shell
[root@localhost usr]# service mysqld stop
Redirecting to /bin/systemctl stop mysqld.service
```

### 7.3.重启命令

```shell
[root@localhost usr]# service mysqld restart
Redirecting to /bin/systemctl restart mysqld.service
```

### 7.4.查看服务状态

```shell
[root@localhost usr]# service mysqld status
Redirecting to /bin/systemctl status mysqld.service

● mysqld.service - MySQL Server
   Loaded: loaded (/etc/systemd/system/mysqld.service; enabled; vendor preset: disabled)
   Active: active (running) since 六 2021-05-08 18:01:03 CST; 5h 6min ago
     Docs: man:mysqld(8)
           http://dev.mysql.com/doc/refman/en/using-systemd.html
 Main PID: 3332 (mysqld)
   CGroup: /system.slice/mysqld.service
           └─3332 /usr/sbin/mysqld --defaults-file=/etc/my.cnf

5月 08 18:01:03 localhost.localdomain systemd[1]: Started MySQL Server.
```

### 7.5.查看MySql系统配置

```shell
[root@localhost usr]# cat /etc/my.cnf
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

