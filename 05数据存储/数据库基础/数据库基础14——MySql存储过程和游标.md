# 存储过程和游标

[toc]

## 一、存储过程

### 1、存储过程介绍

（1）什么是存储过程？

>  存储过程就是把复杂的一系列操作，封装成一个过程。类似于shell，python脚本等  

（2）存储过程的优缺点

>优点是：
>将复杂操作封装，调用简单，安全，执行速度快
>缺点是：
>封装复杂，需要一定基础。
>没有灵活性，可复用性比较低  

### 2、创建存储过程

procedure    读音 /prəˈsiːdʒər/ 过程的意思，

```sql 
create procedure 名称 (参数....)
begin
   过程体;
   过程体;
end

#创建存储过程很简单
```

例如：

```sql
create procedure  productpriciing() 
begin
  select  AVG(prod_price) as priceaverage from products;
end;
```

**注意：mysql命令行客户机的分隔符**

mysql默认分隔符为`;`，但是如果要解释存储过程内部的`;`，在客户机的命令行下可能会出现语法错误。

解决方法，使用`DELIMITER//`

```sql
delimiter //
create procedure  productpriciing() 
begin
  select  AVG(prod_price) as priceaverage from products;
end//
delimiter;
```

其中，DELIMITER //告诉命令行实用程序使用 //作为新的语句结束分隔符，这样到 end// 就结束了。 然后使用

`delimiter ;` 恢复为原来的分隔符`;` 。 当然了除了`//`任何字符都可以作为语句分隔符。

### 3、调用存储过程

mysql调用存储过程更简单，CALL  存储过程名   即可。

```
call productpriciing();

//结果
priceaverage
6.823333
```

### 4、删除存储过程

```sql
drop  procedure productpriciing;
```

### 5、使用参数

上面创建的是最简单的存储过程，当然工作中存储过程是很复杂的，必须要有参数以及复杂逻辑

参数：

```sql
in|out|inout 参数名称 类型（长度）

in：--表示调用者向过程传入值（传入值可以是字面量或变量）
out：--表示过程向调用者传出值(可以返回多个值)（传出值只能是变量）
inout：--既表示调用者向过程传入值，又表示过程向调用者传出值（值只能是变量）
```

- 变量名： 所有mysql 变量必须以 `@`开头。
- 声明变量：declare 变量名 类型(长度) default 默认值;
- 给变量赋值：set @变量名=值;
- 调用存储命令：call 名称(@变量名);
- 删除存储过程命令：drop procedure 名称  

```sql
-- 正常存储过程需要写一写注释，增加可读性
-- Name ；订单统计
-- 参数 ： onumber= order number
--              taxable =0 if not  taxable ,1 if  taxable
--  			ototal =order total
create procedure ordertotal(
	in  onumber int,
	in  taxable boolean,
	out ototal decimal(8,2)

)
begin
	-- 变量
	declare total decimal(8,2);
	declare taxrate int default 6;
	select  Sum(item_price*quantity) from  orderitems where order_num=onumber into total;
	if taxable then
		select total+(total/100*taxrate) into total;
	end if;
	select  total into  ototal;

end;
call ordertotal(20005,0,@total);
select @total;
```



### 6、检查存储过程

```sql
show crete procedure  <存储过程名>  --可以查看一个存储过程的create语句
```

## 二、游标

游标（ cursor） 是一个存储在MySQL服务器上的数据库查询，它不是一条SELECT语句，而是被该语句检索出来的结果集。  游标主要用于交互式应用，其中用户需要滚动屏幕上的数据，并对数据进行浏览或做出更改  。比如查找下一行，下10行。

**不像多数DBMS， MySQL游标只能用于存储过程（和函数）**。  只能用于存储过程。

- 在能够使用游标前，必须声明（定义）它。这个过程实际上没有检索数据，它只是定义要使用的SELECT语句。
-  一旦声明后，必须打开游标以供使用。这个过程用前面定义的SELECT语句把数据实际检索出来。
-  对于填有数据的游标，根据需要取出（检索）各行。
-  在结束游标使用时，必须关闭游标。  



### 1、使用游标：

```sql
--创建存储过程
create procedure processorders()

begin
	--声明游标
	declare ordernumbers cursor 
	for  select order_num from orders;
	--打开游标
	open ordernumbers;
	-- 使用游标
	
	--关闭游标
	close  ordernumbers;

end;
```



在一个游标被打开后，可以使用`FETCH`语句分别访问它的每一行。`FETCH`指定检索什么数据（所需的列），检索出来的数据存储在什么地方。它还向前移动游标中的内部行指针，使下一条FETCH语句检索下一行（不重复读取同一行）。  

完整示例：

```sql
create procedure processorders()

begin
	-- 定义变量
	declare o int;
	declare done boolean default 0;
	declare t decimal(8,2);

	-- 声明游标
	declare ordernumbers cursor 
	for  select order_num from orders;
	
	declare continue handler for sqlstate '02000' set done=1;
	
	create table if not  exists ordertotals(order_num int,total decimal(8,2));
	-- 打开游标
	open ordernumbers;
	repeat
		
	fetch  ordernumbers into o;
	call ordertotal(o,1,t);
	insert into  ordertotals(order_num,total) values(o,t);
	until done  end  repeat;
	
  -- 	 关闭游标
	close  ordernumbers;

end;
-- 调用存储过程，使用游标
call processorders();
select  * from  ordertotals;
```

