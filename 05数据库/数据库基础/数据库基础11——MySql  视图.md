# MySQL视图

[toc]

## 1、什么是视图？有什么用？

> 视图（view）是一种虚拟存在的表，是一个逻辑表，它本身是不包含数据的。作为一个select语句保存在数据字典中的。通过视图，可以展现基表（用来创建视图的表叫做基表base table）的部分数据，说白了视图的数据就是来自于基表  。

视图的优点：

>- 简单：使用视图的用户完全不需要关心后面对应的表的结构、关联条件和筛选条件，对用户来说已经是过滤好的复合条件的结果集
>- 安全：使用视图的用户只能访问他们被允许查询的结果集，对表的权限管理并不能限制到某个行某个列，但是通过视图就可以简单的实现。
>- 数据独立：一旦视图的结构确定了，可以屏蔽表结构变化对用户的影响，源表增加列对视图没有影响;源表修改列名，则可以通过修改视图来解决，不会造成对访问者的影响。
>- 不占用空间：视图是逻辑上的表，不占用内存空间
>  **总而言之，使用视图的大部分情况是为了保障数据安全性，提高查询效率**  

视图的缺点：

>- 性能差：sql server必须把视图查询转化成对基本表的查询，如果这个视图是由一个复杂的多表查询所定义，那么，即使是视图的一个简单查询，sql server也要把它变成一个复杂的结合体，需要花费一定的时间。
>- 修改限制：当用户试图修改试图的某些信息时，数据库必须把它转化为对基本表的某些信息的修改，对于简单的试图来说，这是很方便的，但是，对于比较复杂的试图，可能是不可修改的。  

## 2、视图的规则和限制

- 视图必须唯一命名（不能给视图取与别的视图或表相同的名字）。  
- 对于可以创建的视图数目没有限制  
- 为了创建视图，必须具有足够的访问权限。  
- 视图可以嵌套，即可以利用从其他视图中检索数据的查询来构造一个视图。  
- ORDER BY可以用在视图中，但如果从该视图检索数据SELECT中也含有ORDER BY，那么该视图中的ORDER BY将被覆盖。  
- 视图可以和表一起使用。例如，编写一条联结表和视图的SELECT语句  

## 3、使用视图

### 3.1 常用视图操作语句

#### 3.1.1创建视图

```sql
create view <视图名称> as select 语句;
create view <视图名称> (字段) as select 语句;
```



#### 3.1.2查看创建语句

```sql
show create view <视图名称>
```

#### 3.1.3删除视图

```sql
drop view <视图名称>
```

#### 3.1.4更新视图

```sql
--可以先drop 然后重新create ，当然也可以用以下语句
create or replace view <视图名称>; --不存在就创建，存在替换
--还可以
alter view <视图名称> as select 语句;
```



### 3.2常见视图应用方式

#### 3.2.1用视图简化复杂的联结

```sql
--创建视图 productcustomers；简化联结
CREATE VIEW productcustomers AS SELECT
cust_name,
cust_contact,
prod_id 
FROM
	customers,
	orders,
	orderitems 
WHERE
	customers.cust_id = orders.cust_id 
	AND orderitems.order_num = orders.order_num;
--从视图查询
select  * from  productcustomers;
```

#### 3.2.2用视图重新格式化检出数据

```sql
-- 不用每次都去写格式化sql
CREATE VIEW vendorlocations AS SELECT
Concat( RTRIM( vend_name ), ' (', RTRIM( vend_country ), ')' ) AS vend_title 
FROM
	vendors 
ORDER BY
	vend_name;
--查询数据
select  * from  vendorlocations;
```

#### 3.3.3用视图过滤不想要的数据

```sql
--过滤非空的email
CREATE VIEW customeremaillist AS SELECT
cust_id,
cust_name,
cust_email 
FROM
	customers 
WHERE
	cust_email IS NOT NULL;
--检索数据
select * from customeremaillist;

```

#### 3.3.4视图与计算字段

```sql
--就是类似简单的封装
CREATE VIEW orderitemsexpanded AS SELECT
order_num,
prod_id,
quantity,
item_price,
quantity * item_price AS expanded_price 
FROM
	orderitems;
--查询
SELECT * from orderitemsexpanded;
```

