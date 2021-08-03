# MySQL触发器

[toc]

## 1、触发器简介

什么是触发器？

触发器就是监听（insert、update、delete）操作，然后自动执行其他sql语句（位于begin和end之间）。

## 2、使用触发器

### 2.1创建触发器

注意事项：

- 唯一的触发器名；
- 触发器关联的表；
- 触发器应该响应的活动（ DELETE、 INSERT或UPDATE）；
- 触发器何时执行（处理之前或之后）。  

**仅支持表：只有表支持触发器，视图不支持，临时表也不支持。**

触发器按每个表每个事件每次地定义，每个表每个事件每次只允许输入一个触发器。因此，每个表最多支持6个触发器（**每条INSERT、 UPDATE和DELETE的之前和之后**）。单一触发器不能与多个事件或多个表关联，所
以，如果你需要一个对INSERT和UPDATE操作执行的触发器，则应该定义两个触发器  

```sql
create trigger 触发器名称 after/before insert/update/delete on 表名
for each row
begin
sql语句;
end
```

```sql
after/before:可以设置为事件发生前或后
insert/update/delete:它们可以在执行insert、update或delete的过程中触发
for each row:每隔一行执行一次动作
```

delimiter：

mysql默认是以;结束，如果在定义触发器中使用其他字符作为结束标志，保证sql语句后面的；不影响触发器。可以使用delimiter来定义，比如：

```sql
delimiter $$  -- 修改分隔符为$$
create trigger 触发器名称 after/before insert/update/delete on 表名
for each row
begin
sql语句;
end
$$ 
delimiter ;   -- 恢复分隔符为 ；
```

或者：等等自定义字符

```sql
delimiter //   
create trigger 触发器名称 after/before insert/update/delete on 表名
for each row
begin
sql语句;
end
// 
delimiter ; 
```



### 2.2 删除触发器

触发器不能更新或覆盖。为了修改一个触发器，必须先删除它，然后再重新创建。  

```sql
drop trigger 触发器名称;  --删除触发器
```

### 2.3 insert触发器

- 在INSERT触发器代码内，可引用一个名为NEW的虚拟表，访问被插入的行；
-  在BEFORE INSERT触发器中， NEW中的值也可以被更新（允许更改被插入的值）；
- 对于AUTO_INCREMENT列， NEW在INSERT执行之前包含0，在INSERT执行之后包含新的自动生成值。 

```sql
create trigger neworder after insert on orders
for each row 
select new.order_num;
/*
当插入一个新数据时，mysql会生成一个新的订单编号，以为是auto_increment的。 这时候NEW这张虚拟表存的就是新生成的数据，淡然可以从New.order_num 拿出当前插入的订单ID。
after 事件之后肯定会将新数据存到new虚拟表。
*/
```

New :就是事件发生 before 或after 保存的**新数据**。

### 2.4 delete触发器

delete 语句before 和after  执行触发器。

- 在DELETE触发器代码内，你可以引用一个名为**OLD**的虚拟表，访问被删除的行；  
- **OLD**的值全部是只读的，不能更新。

```sql
create trigger deleteorder before delete on orders
for each row
begin
	sql语句
end;
```

在delete执行之前，旧数据会被存到old虚拟表，可以查出来。进行存档。

begin 和and 之间可以有多条语句。

### 2.5 update触发器

update触发器在 update 语句 **before** 或**after** 执行。

- 在`UPDATE`触发器代码中，你可以引用一个名为**OLD**的虚拟表访问以前（ `UPDATE`语句前）的值，引用一个名为**NEW**的虚拟表访问新更新的值；
- 在`BEFORE UPDATE`触发器中， **NEW**中的值可能也被更新（允许更改将要用于`UPDATE`语句中的值）；
- **OLD**中的值全都是只读的，不能更新。 

例如：

```sql
create trigger updatevendor before update on vendors
for each row 
set new.vend_state=Upper(new.vend_state);

-- update 之前执行触发器，并且要更改的新数据存在new这个虚拟表，在执行update之前可以修改，这里过滤数据变为大写，然后再执行update。
```

