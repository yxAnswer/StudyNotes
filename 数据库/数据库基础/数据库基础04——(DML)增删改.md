## 数据库基础04——(DML)增删改

### 1、INSERT

语法：

```sql
insert into 表名(列名1，列名2，列名3..) values(值1，值2，值3..);--向表中插入某些列
insert into 表名	values(值1，值2，值3..); /*向表中插入所有列，一般少用，尽量用上一种，因为即使表结构改了上一种也能用*/
```

如果要省略部分列就要保证，这些列可以为null或者建表的时候设置了默认值。

注意：

- 列名数于values后面的值得个数相等
- 列的顺序与插入的值的顺序一致
- 列名的类型与插入的值要一致
- 插入值的时候不能超过最大长度
- 值如果是字符串或者日期需要加单引号

### 2、INSERT  SELECT（插入检索出的结果）

注意：**INSERT SELECT，它可以用一条INSERT插入多行，不管SELECT语句返回多少行，都将被 INSERT 插入。** 

```sql
insert into  表名1[(列名1，列名2...)]--  可选插入部分列 或者所有列
select [*|(列名1，列名2...)]--这里查询的列名要和插入的一一对应
from 表名2
[where 条件]  
```

例如：

```sql
INSERT INTO Customers(cust_id,
					  cust_contact,
					  cust_email,
					  cust_name,
					  cust_address)
SELECT  cust_id,
		cust_contact,
		cust_email,
		cust_name,
		cust_address
FROM CustNew;
```

### 3、UPDATE 

```sql
update 表名 set 字段名=值，字段名=值...;			  --更新所有行
update 表名 set 字段名=值，字段名=值...  where  条件；--更新特定行
```

**注意：**

**使用Update 时一定要加 where 条件，如果不加条件就会将整个表的字段更新。所以一定一定一定加条件**

- 列名的类型与修改的值一致
- 修改值的时候不能超过最大长度
- 值如果是字符串或者日期要加引号

### 4、DELETE

```sql
delete from  表名 [where 条件]
/*如果不带where 条件就是删除所有行，如果带where 就是删除特定行*/
```

删除表中所有记录用delete from 表名；还是用 truncate table  表名？

注意：delete 是一条一条的删除，不亲空 auto_increment记录数。

​	   truncate是直接将整张表删除，重新建表，auto_increment 将重置为0。 所以删除效率更高。

事务方面：delete 删除的数据，如果在一个事务内是可以找回，但是truncate 删除的数据是不可以找回的。

**UPDATE 或 DELETE 时所遵循的重要原则： **

- 除非确实打算更新和删除每一行，否则绝对不要使用不带 WHERE 子句的 UPDATE 或 DELETE 语句。
- 保证每个表都有主键，尽可能像 WHERE 子句那样使用它（可以指定各主键、多个值或值的范围）。
-  在 UPDATE 或 DELETE 语句使用 WHERE 子句前，应该先用 SELECT 进行测试，保证它过滤的是正确的记录，以防编写的 WHERE 子句不正确。
- 使用强制实施引用完整性的数据库（关联的），这样 DBMS 将不允许删除其数据与其他表相关联的行。
- 有的 DBMS 允许数据库管理员施加约束，防止执行不带 WHERE 子句的 UPDATE 或 DELETE 语句。如果所采用的 DBMS 支持这个特性，应该使用它 