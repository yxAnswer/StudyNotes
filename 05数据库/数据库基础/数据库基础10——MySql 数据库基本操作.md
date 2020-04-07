# MySQL数据库的基本操作

[TOC]

# 1、mysql 基础操作

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190708110724815.png)

## 1.1 mysql 服务开启、关闭

**方法一：在服务面板中启动或关闭**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190708111239388.png)



**方法二：通过命令行启动\关闭**

 注意：必须通过管理员身份启动命令行

net start 服务名：		启动MySQL服务

net stop 服务器：		      关闭MySQL服务

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190708111311369.png)

## 1.2 服务器的连接、关闭

**通过命令行面板连接：**

```
host：主机			-h
username：用户名	-u
password：密码		-p
port：端口			-P
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190708111624594.png)



![在这里插入图片描述](https://img-blog.csdnimg.cn/20190708111654118.png)



```
注意：如果MySQL服务器在本地，IP地址可以省略；如果MySQL服务器用的是3306端口，-P也是可以省略
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019070811171858.png)

**关闭连接：**

```mysql
方法一：exit

方法二：quit

方法三：\q
```

```
脚下留心：MySQL中的命令后面要加分号，windows命令行的命令后面不用加分号。
```



# 2、数据库操作命令

## 2.1  显示数据库

```mysql
 语法：show databases
 
 mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| test               |
+--------------------+
4 rows in set (0.11 sec)
```

安装MySQL后，MySQL自带了4个数据库

1. information_schema：存储了MySQL服务器管理数据库的信息。
2. performance_schema：MySQL5.5新增的表，用来保存数据库服务器性能的参数
3. mysql：MySQL系统数据库，保存的登录用户名，密码，以及每个用户的权限等等
4. test：给用户学习和测试的数据库。

查询在哪个数据库里面：

```sql
select database()
```



## 2.2  创建数据库

```mysql
语法：create database [if not exists] `数据名` [字符编码]
```

创建数据库：

```mysql
mysql> create database stu;
Query OK, 1 row affected (0.09 sec)
```

如果创建的数据库已存在，就会报错

```mysql
mysql> create database stu;
ERROR 1007 (HY000): Can't create database 'stu'; database exists
```

解决：创建数据库的时候判断一下数据库是否存在，如果不存在再创建

```mysql
mysql> create database if not exists stu;
Query OK, 1 row affected, 1 warning (0.00 sec)
```

如果数据库名是关键字和特殊字符要报错

 解决：在特殊字符、关键字行加上反引号

```mysql
mysql> create database `create`;
Query OK, 1 row affected (0.05 sec)
```

```php
多学一招：为了创建数据库时万无一失，我们可以在所有的数据库名上加上反引号
```

创建数据库的时候可以指定字符编码

```mysql
mysql> create database teacher charset=gbk;
Query OK, 1 row affected (0.01 sec)
gbk		简体中文
gb2312：	简体中文
utf8：	通用字符编码
```

```php
脚下留心：创建数据库如果不指定字符编码，默认和MySQL服务器的字符编码是一致的。
```

## 2.3 删除数据库

```mysql
语法：drop database [if exists] 数据库名
```

删除数据库

```mysql
mysql> drop database teacher;
Query OK, 0 rows affected (0.00 sec)
```

如果删除的数据库不存在，会报错

```mysql
mysql> drop database teacher;
ERROR 1008 (HY000): Can't drop database 'teacher'; database doesn't exist
mysql>
```

 解决：删除之前判断一下，如果存在就删除

```mysql
mysql> drop database if exists teacher;
Query OK, 0 rows affected, 1 warning (0.00 sec)
```

## 2.4 显示创建数据库的SQL语句

```mysql
语法：show create database 数据库名
```

```mysql
mysql> show create database stu;
+----------+--------------------------------------------------------------+
| Database | Create Database                                              |
+----------+--------------------------------------------------------------+
| stu      | CREATE DATABASE `stu` /*!40100 DEFAULT CHARACTER SET utf8 */ |
+----------+--------------------------------------------------------------+
1 row in set (0.01 sec)

mysql> show create database teacher;
+----------+-----------------------------------------------------------------+
| Database | Create Database                                                 |
+----------+-----------------------------------------------------------------+
| teacher  | CREATE DATABASE `teacher` /*!40100 DEFAULT CHARACTER SET gbk */ |
+----------+-----------------------------------------------------------------+
1 row in set (0.00 sec)
```

## 2.5 修改数据库

修改数据库的字符编码

语法：

```mysql
alter database 数据库名 charset=字符编码
```

例题

```mysql
mysql> alter database teacher charset=utf8;
Query OK, 1 row affected (0.00 sec)

mysql> show create database teacher;
+----------+------------------------------------------------------------------+
| Database | Create Database                                                  |
+----------+------------------------------------------------------------------+
| teacher  | CREATE DATABASE `teacher` /*!40100 DEFAULT CHARACTER SET utf8 */ |
+----------+------------------------------------------------------------------+
1 row in set (0.00 sec)
```

## 2.6 选择数据库

语法：

```mysql
use 数据库名
```

选择数据库

```mysql
mysql> use stu;
Database changed
```

## 2.7 中文乱码问题（字符集问题）

临时解决，直接`set names gbk` 就OK了

字符集：字符在保存和传输时对应的二进制编码集合。

创建测试数据库

```mysql
mysql> create table stu(
    -> id int primary key,
    -> name varchar(20)
    -> );
Query OK, 0 rows affected (0.00 sec)
```

插入中文报错

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190708133303453.png)

------

分析原因：

客户端通过GBK发送的命令

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190708133329462.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

但是，服务用utf8解释命令

查看当前mysql使用的字符集：`show variables like 'character%';`

```sql
mysql> show variables like 'character%';
+--------------------------+------------------------------------------------------------+
| Variable_name            | Value                                                      |
+--------------------------+------------------------------------------------------------+
| character_set_client     | gbk                                                        |
| character_set_connection | gbk                                                        |
| character_set_database   | utf8                                                       |
| character_set_filesystem | binary                                                     |
| character_set_results    | gbk                                                        |
| character_set_server     | latin1                                                     |
| character_set_system     | utf8                                                       |
| character_sets_dir       | D:\work\environment\MySQL\MySQL Server 5.7\share\charsets\ |
+--------------------------+------------------------------------------------------------+
8 rows in set, 1 warning (0.00 sec)
```

- character_set_client：客户端请求数据的字符集
- character_set_connection：客户端与服务器连接的字符集
- character_set_database：数据库服务器中某个库使用的字符集设定，如果建库时没有指明，将默认使用配置
  上的字符集
- character_set_results：返回给客户端的字符集(从数据库读取到的数据是什么编码的)
- character_set_server：为服务器安装时指定的默认字符集设定。
- character_set_system：系统字符集(修改不了的，就是utf8)
- character_sets_dir：mysql字符集文件的保存路径



**临时解决**：`set names gbk;`

character_set_client、character_set_results、character_set_connection 这三个是作用于外部，只是控制显示数据的编码，实际数据库的编码并未改变。所以，这个是临时的修改方法。

```sql
set names gbk 
```



**永久的解决中文乱码问题：修改配置文件my.cnf里边的**
修改库的字符集编码
修改表的字符集编码

```sql
[client]
default-character-set=gbk
作用于外部的显示
[mysqld]
character_set_server=gbk
作用于内部，会作用于创建库表时默认字符集
```


