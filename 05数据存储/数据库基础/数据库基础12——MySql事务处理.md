# MySQL事务处理

[toc]

## 1、事务简介

### 1.1什么是事务？

事务处理是一种机制，用来管理必须成批执行的MySQL操作，以保证数据库不包含不完整的操作结果。利用事务处理，可以保证一组操作不会中途停止，它们或者作为整体执行，或者完全不执行（除非明确指示）。如果没有错误发生，整组语句提交给（写到）数据库表。如果发生错误，则进行回退（撤销）以恢复数据库到某个已知且安全的状态。  (简单讲就是一批sql要不都成功，要么都不成功，we are a team!!!)

有两个目的:

- 第一个是为数据库操作提供了一个从失败中恢复到正常状态的方法，同时提供了数据库即使在异常状态下仍能保持一致性的方法；
- 第二个是当多个应用程序在并发访问数据库时，可以在这些应用程序之间提供一个隔离方法，以防止彼此的操作互相干扰。  

### 1.2事务的特性

**并非所有的引擎都支持事务。`MyISAM`和`InnoDB`是两种最常使用的引擎。  只有`InnoDB `支持事务，所以要使用事务的话，表的引擎要为innodb引擎  。**

- **原子性(Atomicity)**：事务必须是原子工作单元，保证任务中的所有操作都执行完毕；否则，事务会在出现错误时终止，并回滚之前所有操作到原始状态。一个事务中的所有语句，应该做到：要么全做，要么一个都不做；
- **一致性(Consistency)**:让数据保持逻辑上的“合理性”，比如：小明给小红打10000块钱，既要让小明的账户减少10000，又要让小红的账户上增加10000块钱；
- **隔离性(Isolation)**：如果多个事务同时并发执行，保证不同的事务相互独立、透明地执行。
- **持久性(Durability)**：一个事务执行成功，则对数据来说应该是一个明确的硬盘数据更改（而不仅仅是内存中的变化）。即使出现系统故障，之前成功执行的事务的结果也会持久存在。


## 2、控制事务

了解几个术语：

- **事务 transaction**：指一组SQL语句；
-  **回退 rollback**：指撤销指定SQL语句的过程；
-  **提交 commit**：指将未存储的SQL语句结果写入数据库表；
- **保留点 savepoint**：指事务处理中设置的临时占位符（ **placeholder**），你可以对它发布回退（与回退整个事务处理不同）。  

管理事务处理的关键在于将SQL语句组分解为逻辑块，并明确规定数据何时应该回退，何时不应该回退。  

### 2.1 事务的开启及回滚 begin;rollback;

事务处理用来管理INSERT、 UPDATE和DELETE语句。  

事务开启： `begin;`      5.7之前使用 `start transaction`

事务回滚： `rollback`

```sql
--创建一个测试表
create table account (
    id tinyint(5) zerofill auto_increment not null comment 'id编号',
    name varchar(20) default null comment '客户姓名',
    money decimal(10,2) not null comment '账户金额',
    primary key (id)
)engine=innodb charset=utf8;
--插入测试数据
insert into account values(null,'张三1',11.5);
insert into account values(null,'张三2',12.5);
insert into account values(null,'张三3',13.5);
insert into account values(null,'张三4',14.5);
select  * from account;--查询应该有的

begin; --开启事务或者用start transaction;
DELETE from account;    --开启事务后，执行删表操作
select  * from account; --查询，应该没有了
ROLLBACK;  --使用rollback回滚；
select  * from account; --删掉的数据神奇的回来了

--注意 truncate table  是删了表重建，事务不起作用
```

### 2.2 事务的提交 commit

事务的提交，使用`commit`语句

```sql
--实例
BEGIN;
DELETE from  table1 where <条件>;
DELETE from  table2 where <条件>;
COMMIT;
--只有当两个都删除成功，事务才算完成
```

**隐含事务关闭 当`COMMIT`或`ROLLBACK`语句执行后，事务会自动关闭（原来的更改会隐含提交）。**  

### 2.3 使用保留点 SAVEPOINT

简单的ROLLBACK和COMMIT语句就可以写入或撤销整个事务处理。但是，只是对简单的事务处理才能这样做，更复杂的事务处理可能需要部分提交或回退。  实现部分提交和回退，需要使用保留点。

```sql
savepoint <保留点名称>;
--比如：
savepoint delete1;
...
savepoint delete2;
rollback to delete1;-- 回到对应占位符位置
```

- 保留点越多越好： 可以更加详细的管理事务及回滚到需要的地方
- 保留点的释放：保留点在事务处理完成（执行一条ROLLBACK或COMMIT）后自动释放。自MySQL 5以来，也可以用`RELEASESAVEPOINT`明确地释放保留点。
  

### 2.4 auto commit 更改默认的提交行为

首先明确：**标志为连接专用**——commit操作是针对每个连接而不是服务器，也就是a客户端和b客户端是不同的。

默认的MySQL行为是自动提交所有更改。换句话说，任何时候你执行一条MySQL语句，该语句实际上都是针对表执行的，而且所做的更改立即生效。  

**autocommit  默认打开on:**

 我理解为： 当`autocommit `为开启状态时（默认就是开启），即使我们没有手动开启一个事务`begin` 或`start transaction` ，mysql 会默认将用户的操作当做一个事务即时提交，一条SQL执行完就提交一次。

如果我们手动开启一个事务，`begin`:  那么在`begin`————`commit` 之间的SQL整体算一个事务，需要我们手动commit提交。

**autocommit 设置为off 时：**

首先，系统会默认开启了事务，并不会默认提交。也就是每条sql 我们不执行commit，在最后其实事务是没有关闭的。这个时候另一个客户端看到的数据 还是sql执行之前的数据。



```sql
mysql> set autocommit=0;
Query OK, 0 rows affected (0.00 sec)
mysql> show variables like 'autocommit';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| autocommit | OFF |
+---------------+-------+
mysql> set autocommit=1;
Query OK, 0 rows affected (0.00 sec)
mysql>
mysql> show variables like 'autocommit';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| autocommit | ON |
```

举个例子：

**A客户端**：`set autocommit=0`   关闭状态；做了一些列增删改操作，，但是没有执行commit之前。**B客户端**看到的数据是没有任何变化的。