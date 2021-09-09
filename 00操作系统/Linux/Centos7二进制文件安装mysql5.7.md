# Centos7 二进制安装mysql5.7

## 1. 下载mysql5.7 二进制文件tar.gz

官网：[https://downloads.mysql.com/archives/community/](https://downloads.mysql.com/archives/community/)

![在这里插入图片描述](https://img-blog.csdnimg.cn/bdb445b6d3004e9e8c7c52beea9ac44a.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA6K-75LiN5oeC55qE562U5qGI,size_20,color_FFFFFF,t_70,g_se,x_16)

![在这里插入图片描述](https://img-blog.csdnimg.cn/6865e2884f224a3faabc532a9d5a9c03.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA6K-75LiN5oeC55qE562U5qGI,size_20,color_FFFFFF,t_70,g_se,x_16)

## 2.解压并移动到指定路径

```shell
# 我的压缩包上传到 /usr/local/software
tar -xvf mysql-5.7.34-el7-x86_64.tar.gz
#移动并重命名
mv mysql-5.7.34-el7-x86_64 /usr/local/mysql
```

## 3. 新增用户并修改权限

添加一个用户，专门负责mysql：

```shell
useradd mysql #新增mysql用户
passwd mysql  #设置密码
```

权限设置：

因为数据库文件在这，需要给权限,这里创建了mysql这个用户以及mysql组，

```shell
chown mysql:mysql -R /usr/local/mysql
```

然后设置可写权限：`chmod -R 777  /usr/local/mysqll` 

## 4.初始化

### 4.1创建数据目录

```shell
mkdir -p /usr/local/mysql/data  #创建数据目录
```

### 4.2 设置配置文件my.cnf

创建配置文件到`/etc/my.cnf`,默认全局配置文件路径

```shell
[mysqld]
#sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
 
basedir=/usr/local/mysql
datadir=/usr/local/mysql/data
port=3306
pid-file = /usr/local/mysql/data/mysql.pid
user = mysql
socket= /usr/local/mysql/data/mysql.sock
bind-address = 0.0.0.0
server-id = 1
 
character-set-server = utf8
 
max_connections = 1000
 
max_connect_errors = 6000
 
open_files_limit = 65535
 
table_open_cache = 128
 
max_allowed_packet = 4M
 
binlog_cache_size = 1M
 
max_heap_table_size = 8M
 
tmp_table_size = 16M
 
read_buffer_size = 2M
 
read_rnd_buffer_size = 8M
 
sort_buffer_size = 8M
 
join_buffer_size = 8M
 
key_buffer_size = 4M
 
thread_cache_size = 8
 
query_cache_type = 1
 
query_cache_size = 8M
 
query_cache_limit = 2M
 
ft_min_word_len = 4
 
log_bin = mysql-bin
 
binlog_format = mixed
 
expire_logs_days = 30
 
log_error = /usr/local/mysql/data/mysql-error.log
 
slow_query_log = 1
 
long_query_time = 1
 
slow_query_log_file = /usr/local/mysql/data/mysql-slow.log
 
performance_schema = 0
 
explicit_defaults_for_timestamp
#lower_case_table_names = 1
 
skip-external-locking
 
default_storage_engine = InnoDB
#default-storage-engine = MyISAM
 
innodb_file_per_table = 1
 
innodb_open_files = 500
 
innodb_buffer_pool_size = 64M
 
innodb_write_io_threads = 4
 
innodb_read_io_threads = 4
 
innodb_thread_concurrency = 0
 
innodb_purge_threads = 1
 
innodb_flush_log_at_trx_commit = 2
 
innodb_log_buffer_size = 2M
 
innodb_log_file_size = 32M
 
innodb_log_files_in_group = 3
 
innodb_max_dirty_pages_pct = 90
 
innodb_lock_wait_timeout = 120
 
bulk_insert_buffer_size = 8M
 
myisam_sort_buffer_size = 8M
 
myisam_max_sort_file_size = 10G
 
myisam_repair_threads = 1
 
interactive_timeout = 28800
 
wait_timeout = 28800
#lower_case_table_names = 1
 
skip-external-locking
 
default_storage_engine = InnoDB
#default-storage-engine = MyISAM
 
innodb_file_per_table = 1
 
innodb_open_files = 500
 
innodb_buffer_pool_size = 64M
 
innodb_write_io_threads = 4
 
innodb_read_io_threads = 4
 
innodb_thread_concurrency = 0
 
innodb_purge_threads = 1
 
innodb_flush_log_at_trx_commit = 2
 
innodb_log_buffer_size = 2M
 
innodb_log_file_size = 32M
 
innodb_log_files_in_group = 3
 
innodb_max_dirty_pages_pct = 90
 
innodb_lock_wait_timeout = 120
 
bulk_insert_buffer_size = 8M
 
myisam_sort_buffer_size = 8M
 
myisam_max_sort_file_size = 10G
 
myisam_repair_threads = 1
 
interactive_timeout = 28800
 
wait_timeout = 28800
 
 
[client]
port=3306
socket = /usr/local/mysql/data/mysql.sock
 
```



### 4.3初始化数据库

```shell
cd /usr/local/mysql/bin
#初始化数据库
./mysqld  --basedir=/usr/local/mysql/ --datadir=/usr/local/mysql/data --user=mysql --explicit_defaults_for_timestamp --initialize
```

### 4.4设置环境变量

```shell
vim /etc/profile
# mysql
export PATH=$PATH:/usr/local/mysql/bin
```




### 4.5 启动、登录mysql



注意： mysql 服务的启动时在，/usr/local/mysql/support-files 目录下的 mysql.server ，比如：

```shell
cd /usr/local/mysql/support-files
./mysql-server start
./mysql-server stop
./mysql-server restart
```

**查找root的默认密码：**

```shell
[mysql@iZszxghs0ozok0Z data]$ cat mysql-error.log 
2021-09-09T15:44:43.354956Z 0 [Warning] InnoDB: New log files created, LSN=45790
2021-09-09T15:44:43.529982Z 0 [Warning] InnoDB: Creating foreign key constraint system tables.
2021-09-09T15:44:43.596351Z 0 [Warning] No existing UUID has been found, so we assume that this is the first time that this server has been started. Generating a new UUID: da35e5f8-1184-11ec-b43c-00163e0a0bb8.
2021-09-09T15:44:43.601031Z 0 [Warning] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
2021-09-09T15:44:44.715460Z 0 [Warning] CA certificate ca.pem is self signed.
2021-09-09T15:44:44.942712Z 1 [Note] A temporary password is generated for root@localhost: T>V?Co0h-h*J
```



**mysql的登录**

```shell
mysql -uroot -p  #输入上方的密码
```

默认密码登录是不能操作的，需要修改默认密码


### 4.6 root登录并修改密码

修改默认密码：修改密码必须再登录了mysql以后我们再修改。执行修改命令

`set password=password('新密码')`  

我们自己设置自己的密码，比如我设置为123456  `set password = password('123456')`



### 4.7 添加账号，设置权限

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




