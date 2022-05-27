



### 一、数据库的基本概念



##### 1. 了解数据库和表

```sql
show databases; -- 返回可用数据库的列表
show tables; -- 返回当前选择的数据库内可用的表的列表
use endweek; -- 选择使用 endweek 这个数据库
show columns from PrimarySchool -- 返回表中的所有列表
```

##### 2. select

- 检索列

```sql
select city from PrimarySchool; -- 检索单个列
select SchoolID, city from PrimarySchool; -- 检索多个列，列名之间用逗号隔开
select * from PrimarySchool; -- 检索所有列
```

- 检索不同的行

```sql
select distinct city from PrimarySchool; 
-- 使用distinct关键字，去重，需要放在列名的前面。
-- distinct关键字应用于所有的列，而不只是前置它的列。
```

- 限制结果

```sql
use dbtest
show tables; 
SELECT `Name` FROM Farmer LIMIT 2 ;
-- limit 2 指示 返回不多于2行
-- 使用limit关键字，如果没有那么多行，只返回可以返回的行。
```

```sql
SELECT `Name` FROM Farmer LIMIT 0,2;
-- limit 0,2 显示从0行开始(包括本行)的2条数据
```

- 使用完全限定的名字来引用列或者引用表

```sql
SELECT Farmer.Name 
FROM dbtest.Farmer LIMIT 0,4;
```

##### 3. 排序检索数据

- 单列排序

```sql
SELECT Name 
FROM Farmer 
GROUP BY Name; -- 按首字母排序A-Z
```

- 多列排序

```sql
select * from PrimarySchool 
order by SchoolName, City;
-- 在按多个列排序时，仅在多个行有相同的SchoolName值时，才对City进行排序。
-- 如果SchoolName列中所有的值都是唯一的，则不会按照City进行排序。
```

-  按特定顺序进行排序

```sql
select * from PrimarySchool 
order by SchoolName DESC, City;
-- DESC关键字只应用到直接位于其前面的列名。其他列按默认升序排列。
```

> order by 也可以和limit组合，能够找到一个列中最高或最低的值。

```sql
-- 如何找到最昂贵物品的值：
select prod_price
from products
order by prod_price DESC
limit 1;
```

> 子句的次序：在给出order子句时，应该保证它位于from子句之后。 是SELECT语句中的最后一条子句。如果使用limit，它必须位于order by之后。使用子句的次序不对将产生错误消息。



##### 4. where

> 在select语句中where子句中指定的搜索条件进行过滤。
>
> where子句的位置：
>
> 1. where子句在表名(from子句)之后给出。 
> 2. 在同时使用 order by 和 where时，应该让 order by 位于 where 之后，否则会出现错误。

- 检测单个值

```sql
USE dbtest;
SELECT * 
FROM Farmer 
Where Name = 'Alice';
-- 可以用操作符
```

- 不匹配检查

```sql
SELECT * 
FROM Farmer 
Where Name != 'Alice';
-- or
SELECT * 
FROM Farmer 
Where Name <> 'Alice'; 
-- 注意：此时不返回 NULL行
```

- 范围检查 

```sql
 select * 
 from products
 where prod_price between 5 and 10;
 -- between 匹配范围中所有的值，也包括开始值和结束值。
```

- 空值检查

> Select 语句有一个特殊的where子句，可以用来检查具有NULL值的列： is null。

```sql
 select * 
 from products
 where prod_price is null; -- 返回没有价格的行。
```

- 组合where子句：用AND子句或OR子句的方式

```sql
select * from student where gender = 'M' and age > 10; -- 都要匹配
select * from student where gender = 'M' or age > 10;  -- 匹配一项即可
```

> and 的优先级更高。可以使用括号来调整优先次序，不要过分依赖默认的计算次序

- IN 操作符

> IN 操作符能完成和OR相同的功能

```sql
select * from student 
where age in (10, 12)
order by Name;
```

- NOT 操作符

> 用于否定跟在它之后的条件

```sql
select * from student 
where age not in (10, 12)
order by Name;
```

##### 5. 创建计算字段

- 拼接值
- 使用别名
- 执行算数计算

##### 6.  聚合函数（aggregate function）



### 二、分组数据

> 介绍如何分组数据，以便汇总表内容的子集。涉及 group by 子句和 having 子句。分组允许把数据分为多个逻辑组，以便对每个组进行聚集计算。

##### 1. 创建分组： group by

> 返回每个组的结果。

- group by

```sql
use endweek;
SELECT City, count(*) as num_city  
FROM PrimarySchool 
GROUP BY City; 
```

| City      | num_city |
| --------- | -------- |
| Sheffield | 1        |
| Leeds     | 2        |

> 1. 如果分组列中具有NULL值，则NULL将作为一个分组返回。如果列中有多行NULL值，它们将分为一组。
> 2. group by 子句必须出现在where子句之后，order by 子句之前。

##### 2. 过滤分组： having

> where 子句的作用是：过滤行而不是分组。事实上，where没有分组的概念。用having来过滤分组。目前为止所学过的where子句都可以用having来替代。唯一的差别是where过滤行，having过滤分组。

```sql
use endweek;
SELECT City, count(*) as num_city  
FROM PrimarySchool 
GROUP BY City
HAVING count(*) >= 2; 
```

| City | num_city |
| ---- | -------- |
| 1    | 2        |

> where 在数据分组前进行过滤，having在数据分组后进行过滤

```sql
select vend_id, count(*) as num_prods
from products
where prod_price >= 10
group by vend_id
having count(*) >= 2
-- 列出具有2个（含）以上、价格为10（含）以上的产品的供应商。
```

##### 3. 分组和排序：order by 和 group by

```sql
select order_num, sum(quantity*item_price) as ordertotal
from orderitems
group by order_num
having sum(quantity*item_price) >= 50
order by ordertotal; -- 最后排序输出
```

##### 4. 回顾select子句顺序

| 子句     | 说明               | 是否必须要使用         |
| -------- | ------------------ | ---------------------- |
| select   | 要返回的列或表达式 | 是                     |
| from     | 从中检索数据的表   | 仅在从表选择数据时使用 |
| where    | 行级过滤           | 否                     |
| group by | 分组说明           | 仅在按组计算聚集时使用 |
| having   | 组级过滤           | 否                     |
| order by | 输出排序的顺序     | 否                     |
| limit    | 要检索的行数       | 否                     |

### 三、子查询

> 可以把一条SELECT语句返回的结果用于另一条SELECT语句的WHERE子句。
>
> 在WHERE子句中使用子查询能够编写出功能很强并且很灵活的SQL语句。对于能嵌套的子查询的数目没有限制，不过在实际使用时由于性能的限制，不能嵌套太多的子查询。

##### 1. 用子查询进行过滤

```sql
select cust_id
from orders
where order_num in (select order_num
                    from orderitems
                    where prod_id = 'TNT2');
-- 一般用in操作符结合使用，但也可用 =，<> 等。                   
```

##### 2. 作为计算字段使用子查询

```sql
select cust_name, 
			 cust_state, 
			 (select count(*)
       	from orders
        where orders.cust_id = customers.cust_id ) as orders
from customers
order by cust_names;
```

> 子查询中的WHERE子句与前面使用的WHERE子句稍有不同，因为它使用了完全限定列名。



### 四、连结表

##### 1. 创建联结

```sql
select vend_name, prod_name, prod_price
from vendors, products
where vendors.vend_id = products.vend_id
order by vend_name, prod_name;
-- 这里需要这种完全限定列名，因为如果只给出vend_id，则MySQL不知道指的是哪一个。
```

> 如果没有where子句，则第一个表中的每个行将与第二个表中的每个行配对，而不管它们逻辑上是否可以配在一起。

- 用 inner join ... on ...改写

```sql
select vend_name, prod_name, prod_price
from vendors inner join products
on vendors.vend_id = products.vend_id
```

- 联结多个表

```sql
select vend_name, prod_name, prod_price, quaantity
from orderitems, vendors, products
where vendors.vend_id = products.vend_id
and orderitems.prod_id = products.prod_id
and order_num = 20005;
-- 这里的FROM子句列出了3个表，而WHERE子句定义了这两个联结条件，而第三个联结条件用来过滤出订单20005中的物品。
```

##### 2. 使用不同类型的联结

- 自联结

> 在单条select语句中不止一次引用相同的表。

> 假如你发现某物品（其ID为DTNTR）存在问题，因此想知道生产该物品的供应商生产的其他物品是否也存在这些问题。此查询要求首先找到生产ID为DTNTR的物品的供应商，然后找出这个供应商生产的其他物品。

```sql
select prod_id, prod_name
from products
where vend_id = (select vend_id from products where prod_id = 'DTNTR')
```

> 使用联结的相同查询

```sql
select p1.prod_id, p1.prod_name
from products as p1, products as p2
where p1.vend_id = p2.vend_id and p2.prod_id = 'DTNTR'
```

> 有时候，自联结比子查询性能好很多。

- 内部联结与外部联结

> 内部联结

```sql
select customers.cust_id, orders.order_num
from customers inner join orders
on customers.cust_id = orders.cust_id;
```

> 外部联结语法类似。为了检索所有客户，包括那些没有订单的客户，可如下进行：

```sql
select customers.cust_id, orders.order_num
from customers left outer join orders
on customers.cust_id = orders.cust_id;
-- 使用LEFT OUTER JOIN从FROM子句的左边表（customers表）中选择所有行。
```

> 存在两种基本的外部联结形式：左外部联结和右外部联结。
>
> 它们之间的唯一差别是所关联的表的顺序不同。换句话说，左外部联结可通过颠倒FROM或WHERE子句中表的顺序转换为右外部联结。因此，两种类型的外部联结可互换使用，而究竟使用哪一种纯粹是根据方便而定。

- 使用带聚合函数的联结

> 检索所有客户及每个客户所下的订单数

```sql
select customers.cust_id, customers.cust_name, count(orders.order_num) as num_ord
from customers inner join orders
on customers.cust_id = orders.cust_id
group by customers.cust_id;
```

> 用left outer join

```sql
select customers.cust_id, customers.cust_name, count(orders.order_num) as num_ord
from customers left outer join orders
on customers.cust_id = orders.cust_id
group by customers.cust_id;
-- 结果显示也包含了客户Mouse House，它有0个订单。
```

##### 3. 使用联结和联结条件

> 1. 注意所使用的联结类型。一般我们使用内部联结，但使用外部联结也是有效的。
>
> 2. 应该总是提供联结条件，否则会得出笛卡儿积。

##### 4. 组合查询：union

> 可用union 操作来组合数条sql查询。利用union, 可给出多条select语句，将它们的结果组合成单个结果集。

```sql
SELECT vend_id, prod_id, prod_price FROM products
WHERE prod_price <= 5
OR vend_id IN (1001,1002);
```

> 或者：使用union

```sql
SELECT vend_id, prod_id, prod_price 
FROM products
WHERE prod_price <= 5
union
SELECT vend_id, prod_id, prod_price 
FROM products
WHERE vend_id IN (1001,1002);
```

> Union all 可以包含重复的行。

- 对组合查询结果排序

```sql
SELECT vend_id, prod_id, prod_price 
FROM products
WHERE prod_price <= 5
union
SELECT vend_id, prod_id, prod_price 
FROM products
WHERE vend_id IN (1001,1002)
group by vend_id, prod_price;
-- 虽然ORDER BY子句似乎只是最后一条SELECT语句的组成部分，但实际上MySQL将用它来排序所有SELECT语句返回的所有结果。
```

