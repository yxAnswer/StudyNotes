# MySQL存储引擎与索引

[toc]

## 1、mysql存储引擎介绍

### 1.1什么是数据库存储引擎？

> 数据库引擎是数据库底层软件组件，不同的存储引擎提供不同的存储机制，索引技巧，锁定水平等功能，使用不同的数据库引擎，可以获得特定的功能  

### 1.2 查看mysql引擎

```sql
-- 如何查看数据库支持的引擎
show engines;
-- 查看当前数据的引擎：
show create table 表名\G
-- 查看当前库所有表的引擎：
show table status\G
```

### 1.3 指定及修改数据库引擎

```sql
-- 可以创建表的时候指定引擎，不指定mysql默认是InnoDB;
create table User (id int,name varchar(20)) engine='InnoDB';
-- 也可以用alter 修改表引擎
alter table 表名 engine='MyiSAm';
--linux下 修改默认引擎
• vi /etc/my.cnf
• [mysqld]下面
• default-storage-engine=MyIsAM
• 记得保存后重启服务

--windows下 找到mysql安装目录
-- my.ini 文件 修改 default-storage-engine=INNODB
```



### 1.4 MyISAM与InnoDB 引擎的区别

> MyISAM：
>
> - 支持全文索引（full text）;
> - 不支持事务;
> - 表级锁;
> - 保存表的具体行数;
> - 奔溃恢复不好
>
> Innodb：
>
> - 支持事务;
> - 以前的版本是不支持全文索引，但在5.6之后的版本就开始支持这个功能了;
> - 行级锁（并非绝对，当执行sql语句时不能确定范围时，也会进行锁全表例如： update table set id=3 where name like 'a%';）;
> - 不保存表的具体行数;
> - 奔溃恢复好  

## 2、mysql 索引介绍

### 2.1 什么是索引？

>索引是一个单独的，存储在磁盘中上的数据库结构，它们包含着对数据表里的所有记录的引用指针。使用索引可以快速的找出在某列或多列中有特定值的行  

### 2.2 索引的优点：

>通过创建唯一索引，来保证数据库表中的每一行数据的唯一性。
>
>可以加快数据的检索速度。
>
>可以保证表数据的完整性与准确性  

### 2.3 索引的缺点

>索引需要占用物理空间。
>对表中的数据进行改动时，索引也需要跟着动态维护，降低了数据的维护速度。  

### 2.4 索引的常见类型：

-  index：普通索引
-  unique：唯一索引
- primary key：主键索引
-  foreign key：外键索引
- fulltext: 全文索引
- 组合索引  

### 2.5 测试表数据脚本

```sql
--随便创建一个表
create table test (
	id int(7) zerofill auto_increment not null,
	username varchar(20),
	servnumber varchar(30),
	password varchar(20),
	createtime datetime,
	primary key (id)
)DEFAULT CHARSET=utf8;

```

```shell
--生成sql shell脚本
#!/bin/bash
echo "请输入字段servnumber的值："
read serber
echo "请输入创建sql语句的数量："
read number
# char=`head /dev/urandom | tr -dc 0-9 | head -c 11`
for (( i=0;i<$number;i++ ))
do
pass=`head /dev/urandom | tr -dc a-z | head -c 8`
let serber=serber+1
echo "insert into test(id,username,servnumber,password,createtime)
values('$i','user${i}','${serber}','$pass',now());" >>sql.txt
done
```

```shell
#创建 test.sh 文件，复制上面脚本
sh test.sh #执行
#生成sql.txt文件，然后再sql中导入它
source /xxx/xx/sql.txt   #路径是文件的路径
```

## 3、普通索引与唯一索引

### 3.1 普通索引(index)

```
普通索引（index）顾名思义就是各类索引中最为普通的索引，主要任务就是提高查询速度。其特点是允许出现相同的索引内容，允许空（null）值  
```

### 3.2 唯一索引(unique)

```
唯一索引：（unique）顾名思义就是不可以出现相同的索引内容，但是可以为空（null）值
```

### 3.3 创建普通索引和唯一索引

```sql
-- 创建表时候创建
create table test (
	id int(7) zerofill auto_increment not null,
	username varchar(20),
    servnumber varchar(30),
	password varchar(20),
	createtime datetime,
	index (id)  --普通索引
)DEFAULT CHARSET=utf8;
```

```sql
-- 直接为表添加索引
--语法：alter table 表名 add index 索引名称 (字段名称);
eg:
alter table test add unique unique_username (username);  --username添加一个唯一索引
-- 注意：假如没有指定索引名称时，会以默认的字段名为索引名称
```

```sql
--直接创建索引
--语法：create index 索引名 on 表名 (字段名);
eg：create index index_createtime on test (createtime);
```

### 3.4 查看索引sql

```sql
--语法：show index from 表名\G
show index from test\G
```

### 3.5删除索引

```sql
--语法：drop index 索引名称 on 表名;
drop index unique_username on test;

--语法：alter table 表名 drop index 索引名;
alter table test drop index createtime;
```

## 4、主键索引

### 4.1 主键索引介绍

> 把主键添加索引就是主键索引，它是一种特殊的唯一索引，不允许有空值，而唯一索引（unique是允许为空值的）。指定为`PRIMARY KEY`  

- 主键：主键是表的某一列，这一列的值是用来标志表中的每一行数据的。
- 注意：每一张表只能拥有一个主键  

### 4.2 创建主键

```sql
-- 1、创建表的时候创建
-- 2、直接为表添加主键索引
--语法：alter table 表名 add primary key (字段名);
   alter table test add primary key (id);
```

### 4.3 删除主键

```sql
--语法：alter table 表名 drop primary key;

alter table test drop primary key;
--注意：在有自增的情况下，必须先删除自增，才可以删除主键
--删除自增：alter table test change id id int(7) unsigned zerofill not null;
```

## 5、全文索引

### 5.1 全文索引介绍

> 全文索引是将存储在数据库中的文章或者句子等任意内容信息查找出来的索引，单位是词。全文索引也是目前搜索引擎使用的一种关键技术。指定为 **fulltex**  

创建个练习表：

```sql
create table command (
id int(5) unsigned primary key auto_increment,
name varchar(10),
instruction varchar(60)
)engine=MyISAM;
```

插入数据

```sql
insert into command values('1','ls','list directory contents');
insert into command values('2','wc','print newline, word, and byte counts for each
file');
insert into command values('3','cut','remove sections from each line of files');
insert into command values('4','sort','sort lines of text files');
insert into command values('5','find','search for files in a directory hierarchy');
insert into command values('6','cp','复制文件或者文件夹');
insert into command values('7','top','display Linux processes');
insert into command values('8','mv','修改文件名，移动');
insert into command values('9','停止词','is,not,me,yes,no ...');
```

### 5.2 添加全文索引

- 创建表的时候添加全文索引

  ```sql
  --例如
  create table command (
  	id int(5) unsigned primary key auto_increment,
  	name varchar(10),
  	instruction varchar(60),
   	fulltext(instruction)
  )engine=MyISAM;
  
  ```

- 通过alter 修改

  ```sql
  alter table command add fulltext(instruction);
  ```

### 5.3 使用全文索引





## 6、外键约束索引

## 7、联合索引





