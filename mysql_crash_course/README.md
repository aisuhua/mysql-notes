# MySQL Crash Course

Book link: http://www.forta.com/books/0672327120/

## Import db

```sql
create database crashcourse
use crashcourse
source /path/to/create.sql
source /path/to/populate.sql
```

## SQL

```sql
# 查看服务状态
show status
# 查看所有数据库
show databases
# 使用库
use crashcourse
# 查看所有表
show tables
# 查看库创建语句
show create database crashcourse
# 查看表创建语句
show create table customers
# 显示授权列表
show grants
# 查看服务器错误消息
show errors
# 查看服务器警告消息
show warnings
# 查看 show 命令帮助文档
help show

# 查看所有列
show columns from customers
describe customers (alias)

# 查看 select 命令帮助文档
help select
# 查询非重复行，如查询多列，则要求多列都需要一致时才会去重掉
select distinct vend_id from products
# 限制返回行数
select * from customers limit 0, 2
select * from customers limit 2 offset 0
# 完全限定库名和表名
select customers.cust_name from crashcourse.customers

# 排序
select * from customers order by cust_name asc, cust_id desc

# 范围查询
select * from products where prod_price between 10 and 20
select * from products where prod_price >= 10 and prod_price <=20

# 空值检测
select * from customers where cust_email is null;
select * from customers where cust_email is not null;

# 逻辑操作符：or 和 and
select * from products where (vend_id = 1002 or vend_id = 1003) and prod_price >= 10;

# IN操作符
select * from products where vend_id in(1002, 1003) and prod_price >= 10;

# NOT操作符
select * from products where vend_id not in (1002, 1003);

# LIKE操作符，通配符 % 和 _
select * from products where prod_name like "_ ton anvil"
select * from products where prod_name like "% ton anvil"
select * from products where prod_name like "%anvil%"

# 正则表达式：regexp
select * from products where prod_name regexp '1000'
select * from products where prod_name regexp '.000'
select * from products where prod_name regexp "JetPack [0-9]+"
select * from products where prod_name regexp 'JetPack [[:digit:]]+'
# 匹配字符类
select * from products where prod_name regexp 'JetPack [[:digit:]]{4}'
# 与LIKE相等，regexp 只需包含即可，而like是完全匹配
select * from products where prod_name like '000'
select * from products where prod_name regexp '^000$'
# 特殊字符，使用双斜线转义
select * from vendors where vend_name regexp '\\.'
select * from products where prod_name regexp '\\([1-5] sticks?\\)'
# 匹配范围：| 和 -
select * from products where prod_name regexp '[1-3] ton'
select * from products where prod_name regexp '(1|2|3) ton'
# ^ 的否定含义
select * from products where prod_name regexp '[^1-3] ton'
# 测试正则表达式
select 'suhua' regexp '^[a-z]{5}$';

# 计算字段，函数
select concat(rtrim(vend_name), '(', rtrim(vend_address), ')') as vend_title from vendors
select prod_id, quantity, item_price, sum(quantity * item_price) as expanded_price from orderitems
# 测试计算字段
select 1 + 1
select now()
select trim(' suhua  ')

# 数据处理函数
select abs(-100)
select substr('suhua', 1, 2)
select locate('suhua', 'suhuazizi')
select * from orders where date(order_date) = '2005-09-01'
select * from orders where year(order_date) = 2005 and month(order_date) = 9

# 聚集函数
select avg(prod_price) from products where vend_id = 1003;
select count(*) from customers
select count(cust_email) from customers
select min(prod_price), max(prod_price) from products
select sum(quantity * item_price) from orderitems where order_num = 20005
select count(*), min(prod_price), max(prod_price), avg(prod_price) from products

# distinct 排除重复
select avg(distinct prod_price) from products where vend_id = 1003

# 分组数据
select vend_id, count(*) from products group by vend_id
select vend_id, count(*) as num_prods from products group by vend_id having count(*) >= 2
select vend_id, count(*) as num_prods from products group by vend_id having num_prods >= 2

# 嵌套查询1：利用子查询进行过滤
select order_num from orderitems where prod_id = 'TNT2';
select cust_id  from orders where order_num in (20005, 20007);
select cust_id, cust_name from customers where cust_id in (10001, 10004);
# 相当于（嵌套实现）
select cust_id, cust_name from customers where cust_id in (
    select cust_id from orders where order_num in (
        select order_num from orderitems where prod_id = 'TNT2'
    )
);
# 相当于（笛卡尔积实现）
select distinct customers.cust_id, customers.cust_name
from orderitems, orders, customers
where orderitems.order_num = orders.order_num and
orders.cust_id = customers.cust_id and
orderitems.prod_id = 'TNT2';
# 相当于（内连接实现，跟上面一样，只是写法不同）
select distinct customers.cust_id, customers.cust_name
from orderitems inner join orders on orderitems.order_num = orders.order_num
inner join customers on orders.cust_id = customers.cust_id and orderitems.prod_id = 'TNT2'
# 相当于（自然连接实现）
select customers.cust_id, customers.cust_name
from orderitems join orders using(order_num) join customers using(cust_id)
where orderitems.prod_id = 'TNT2'

# 嵌套查询2：作为计算字段使用子查询
select cust_id, count(*) from orders where cust_id = 10001
# 相当于（嵌套实现）
select cust_id, cust_name, (
    SELECT count(*) from orders where orders.cust_id = customers.cust_id
) as order_count
from customers;
# 相当于（左连接实现）
select customers.cust_id, customers.cust_name, count(orders.cust_id) as order_count
from customers left join orders on customers.cust_id = orders.cust_id
group by customers.cust_id

# 联接表
# 内连接（等值连接）：找出每个供应商及其生产的所有商品信息
select vendors.vend_id, vend_name, products.vend_id, prod_id, prod_name
from vendors, products where vendors.vend_id = products.vend_id
# 方式2：inner join
select vendors.vend_id, vend_name, products.vend_id, prod_id, prod_name
from vendors inner join products on vendors.vend_id = products.vend_id

# 自连接
# 子查询实现：找出生产商品ID为DTNTR的供应商所生产的其他商品
select * from products where vend_id in (select vend_id from products where prod_id = 'DTNTR')
# 自连接实现2
select p2.prod_id, p2.prod_name
from products p1 inner join products p2 on p1.vend_id = p2.vend_id
where p1.prod_id = 'DTNTR'
# 自连接实现2
select p2.prod_id, p2.prod_name
from products p1, products p2
where p1.vend_id = p2.vend_id and p1.prod_id = 'DTNTR'

# 连接和聚合运算结合：找出所有客户及每个客户所下的订单数
select customers.cust_id, customers.cust_name, count(orders.order_num) as num
from customers left outer join orders on customers.cust_id = orders.cust_id
group by customers.cust_id;

# 组合查询
# 去掉重复行
select vend_id, prod_id, prod_price from products where prod_price <= 5
union
select vend_id, prod_id, prod_price from products where vend_id in (1001, 1002)
# 保留重复行
select vend_id, prod_id, prod_price from products where prod_price <= 5
union all
select vend_id, prod_id, prod_price from products where vend_id in (1001, 1002)
# 对组合结果进行排序
select vend_id, prod_id, prod_price from products where prod_price <= 5
union
select vend_id, prod_id, prod_price from products where vend_id in (1001, 1002)
order by prod_price desc

# 全文本搜索：计算相关性
select note_text from productnotes where match(note_text) against('rabbit')
# 查询扩展：with query expansion
select note_text from productnotes where match(note_text) against('rabbit' with query expansion)
# 布尔搜索：包含 rabbit 和 bait 中的至少一个词
select note_text from productnotes where match(note_text) against('rabbit bait' in boolean mode)
# 同时包含词 rabbit 和 bait 的行
select note_text from productnotes where match(note_text) against('+rabbit +bait' in boolean mode)
# 包含词 rabbit，不能包含 bait
select note_text from productnotes where match(note_text) against('+rabbit -bait' in boolean mode)

# 插入完整的行
insert into customers
values
(NULL, 'Pep E. LaPew', '100 Main Street', 'Los Angeles', 'CA', '90064', 'USA', NULL, NULL)
# 插入行的一部分
insert into customers (cust_name, cust_address, cust_city, cust_state, cust_zip, cust_country)
values
('Pep E. LaPew', '100 Main Street', 'Los Angeles', 'CA', '90064', 'USA');
# 插入多行
insert into customers (cust_name, cust_address, cust_city, cust_state, cust_zip, cust_country)
values
('Pep E. LaPew', '100 Main Street', 'Los Angeles', 'CA', '90064', 'USA'),
('suhua', 'Guangdong', 'Guangzhou', 'GD', 50123, 'CHINA')
# 插入检索出的数据：insert select
insert into customers (cust_id, cust_name, cust_address)
select cust_id, cust_name, cust_address from custnew
# 指定插入语句的优先级
insert low_priority into custnew values (null, 'xiaozhang', 'Xicun')
insert high_priority into custnew values (null, 'xiaozhang', 'Xicun')

# 更新数据
update custnew set cust_address = 'Guangzhou' where cust_id = 10012
update custnew set cust_address = 'Guangzhou', cust_name = 'suhua' where cust_id = 10012
# IGNORE 关键字
update ignore custnew set cust_address = 'Guangzhou', cust_name = 'suhua' where cust_id = 10012

# 删除数据
delete from custnew where cust_id = 10013
# 清空整个表
delete from custnew
# 更快的清空方式：先删表，然后再创建一个新表
truncate custnew

# 创建表
create table student(
    id int not null auto_increment,
    name varchar(20) not null,
    age int not null default 0,
    phone char(11) null,
    address varchar(100) null,
    primary key (`id`)
) engine innodb;
# 当表不存在才创建
create table if not exists student(
    id int not null auto_increment,
    name varchar(20) not null,
    age int not null default 0,
    phone char(11) null,
    address varchar(100) null,
    primary key (`id`)
) engine innodb;

# 更新表
# 添加字段
alter table student add sex varchar(10)
alter table student add sex varchar(10) after age
# 删除字段
alter table student drop sex

# 添加索引
alter table student add index name (`name`)
alter table student add index name (`name`), add index age (`age`)
# 删除索引
alter table student drop index name
# 添加外键
alter table orders
add constraint fk_orders_customers
foreign key(cust_id) references customers(cust_id);
# 删除外键
alter table orders drop foreign key fk_orders_customers

# 重命名表
rename table student to student_new

# 删除表
drop table student

# 获取最后一条自增ID
select last_insert_id()

# 视图
select cust_name, cust_contact, prod_id
from orderitems as oi
inner join orders as o on oi.order_num = o.order_num
inner join customers as c on o.cust_id = c.cust_id and
where prod_id = 'TNT2'
# 创建视图
create view productcustomers as
select cust_name, cust_contact, prod_id
from orderitems as oi
inner join orders as o on oi.order_num = o.order_num
inner join customers as c on o.cust_id = c.cust_id
# 使用视图
select * from productcustomers
select * from productcustomers where prod_id = 'TNT2'

# 用视图格式化检索出的数据
create view vendorlocations as
select concat(rtrim(vend_name), '(', vend_country, ')') as vend_title from vendors order by vend_name
# 使用视图
select * from vendorlocations;

# 用视图过滤不想要的数据
create view customeremailist as
select cust_id, cust_name, cust_email from customers where cust_email is not null
# 使用视图
select * from customeremailist

# 使用视图与计算字段
create view orderitemsexpanded as
select order_num, prod_id, quantity, item_price, quantity*item_price as expanded_price from orderitems
# 使用视图
select * from orderitemsexpanded where order_num =20005

# 查看视图定义
show create view orderitemsexpanded
# 删除视图
drop view orderitemsexpanded

# 存储过程

# 修改分隔符
delimiter |
# 创建存储过程
create procedure productpricing()
begin
select avg(prod_price) as priceaverage from products;
end|
# 恢复分隔符
delimiter ;
# 调用存储过程
call productpricing()

# 创建存储过程2
delimiter |
create procedure test(
    IN x int,
    IN y int,
    OUT z int
)
begin
    set z = x + y;
end|
delimiter ;
call test(10, 20, @z)
select @z

# 查看存储过程结构
show create procedure productpricing
# 删除存储过程
drop procedure productpricing

# 查看所有存储过程
show procedure status
show procedure status like 'productpricing'
show procedure status where db = 'crashcourse'
select db, name from mysql.proc

# 使用参数1
create procedure productpricing(
    OUT p1 decimal(8, 2),
    OUT p2 decimal(8, 2),
    OUT p3 decimal(8, 2)
)
begin
select min(prod_price) into p1 from products;
select max(prod_price) into p2 from products;
select avg(prod_price) into p3 from products;
end|
# 调用存储过程
call productpricing(@min, @max, @avg)
# 查看结果
select @min, @max, @avg

# 使用参数2
create procedure ordertotal(
    IN onumber int,
    OUT ototal decimal(8, 2)
)
begin
select sum(item_price * quantity) from orderitems where order_num = onumber into ototal;
end|
# 调用存储过程
call ordertotal(20005, @total)
call ordertotal(20006, @total)
# 查看结果
select @total

# 智能存储过程
create procedure ordertotal(
    IN onumber int,
    IN taxable boolean,
    OUT ototal decimal(8, 2)
) comment 'Obtain order total, optionally adding tax'
begin

-- Declare variable for total
declare total decimal(8, 2);
-- Declare tax percentage
declare taxrate int default 6;

-- Get the order total
select sum(item_price * quantity) from orderitems where order_num = onumber into total;

-- Is this taxable?
if taxable then
    select total + (total * (taxrate / 100)) into total;
end if;

-- And finally save to out variable
select total into ototal;

end|
# 调用存储过程
call ordertotal(20005, 0, @total)
select @total
call ordertotal(20005, 1, @total)
select @total

# 游标的使用
create procedure processorders()
begin

    -- Declare local variable
    declare done boolean default 0;
    declare o int;
    declare t decimal(8, 2);

    -- Declare the cursor
    declare ordernumbers cursor for select order_num from orders;
    -- Declare continue handler
    declare continue handler for sqlstate '02000' set done = 1;

    -- Create a table to store the results
    create table if not exists ordertotals(order_num int, total decimal(8, 2));

    -- Open the cursor
    open ordernumbers;

    -- Loop through all rows
    repeat

        -- Get the order num
        fetch ordernumbers into o;
        -- Get the total for this order
        call ordertotal(o, 1, t);

        -- Insert the order and total into ordertotals
        insert into ordertotals(order_num, total) values (o, t);

    -- End the loop
    until done end repeat;

    -- Close the cursor
    close ordernumbers;

end|

# 调用存储过程
call processorders()
# 查看结果
select * from ordertotals


# 触发器
create trigger newtest after insert on test for each row select 'Insert new row.' into @test
insert into test values ('suhua', 'lala', 20, 'su')
select @test

# INSERT 触发器
create trigger neworder after insert on orders for each row select NEW.order_num into @order_num
insert into orders(order_date, cust_id) values (NOW(), 10001)
select @order_num

# DELETE 触发器
create trigger deleteorder before delete on orders for each row
begin
    insert into archive_orders(order_num, order_date, cust_id)
    values (OLD.order_num, OLD.order_date, OLD.cust_id);
end|
delete from orders where order_num = 20011
select * from archive_orders
delete from orders where order_num = 20010
select * from archive_orders

# UPDATE 触发器
create trigger updatevendor before update on vendors for each row
set NEW.vend_state = upper(NEW.vend_state);
update vendors set vend_state = 'suhua' where vend_id = 1006
select * from vendors

# 事务
start transaction
delete from test where name = 'suhua'
rollback
select * from test

start transaction
delete from test where name = 'suhua'
commit
select * from test

# 使用
start transaction
delete from orderitems where order_num = 20005
delete from orders where order_num = 20005
rollback

# 禁止默认提交
set autocommit = 0
delete from test where name = 'xiaomi'
rollback
select * from test

# 字符集

# 查看所有引擎
show engines
# 查看所有字符集
show character set
# 查看所有校对集：_ci 表示比较或排序时不区分大小写
show collation
# 查看当前使用的字符集
show variables like "character%"
# 查看当前使用的校对集
show variables like "collation%"

# 自定义字符集和校对集
create table demo2 (
    name varchar(10),
    description varchar(255) character set latin1 collate latin1_bin
) engine InnoDB character set utf8mb4 collate utf8mb4_unicode_ci;
```

## 摘录

不指定排序方式的说明

> 其实，检索出的数据并不是以纯粹的随机顺序显示的。
> 如果不排序，数据一般将以它在底层表中出现的顺序显示。这可以是数据最初添加到表中的顺序。
> 但是，如果数据后来进行过更新或删除，则此顺序将会受到MySQL重用回收存储空间的影响。
> 因此，如果不明确控制的话，不能（也不应该）依赖该排序顺序。
> 关系数据库设计理论认为，如果不明确规定排序顺序，则不应该假定检索出的数据的顺序有意义。

大小写排序问题

> 在对文本性的数据进行排序时，A与a相同吗？a位于B之前还是位于Z之后？这些问题不是理论问题，其答案取决于数据库如何设置。
> 在字典（dictionary）排序顺序中，A被视为与a相同，这是MySQL（和大多数数据库管理系统）的默认行为。
> 但是，许多数据库管理员能够在需要时改变这种行为（如果你的数据库包含大量外语字符，可能必须这样做）。
> 这里，关键的问题是，如果确实需要改变这种排序顺序，用简单的ORDER BY子句做不到。你必须请求数据库管理员的帮助。

不区分大小写

> MySQL在执行匹配时默认不区分大小写，所以fuses与Fuses匹配。

NULL和不等于

> 在通过过滤选择出不具有特定值的行时，你可能希望返回具有NULL值的行。但是，不行。
> 因为未知具有特殊的含义，数据库不知道它们是否匹配，所以在匹配过滤或不匹配过滤时不返回它们。

IN操作符

- 在使用长的合法选项清单时，IN操作符的语法更清楚且更直观。
- 在使用IN时，计算的次序更容易管理（因为使用的操作符更少）。
- IN操作符一般比OR操作符清单执行更快。
- IN的最大优点是可以包含其他SELECT语句，使得能够更动态地建立WHERE子句。第14章将对此进行详细介绍。

NOT操作符

> WHERE子句中用来否定后跟条件的关键字。
> MySQL支持使用NOT对IN、BETWEEN和EXISTS子句取反，这与多数其他DBMS允许使用NOT对各种条件取反有很大的差别。

LIKE操作符

- 通配符（wildcard） 用来匹配值的一部分的特殊字符。
- 搜索模式（search pattern） 由字面值、通配符或两者组合构成的搜索条件。
- 搜索关键字默认不区分大小写，但是可配置。

> 注意尾空格，尾空格可能会干扰通配符匹配。
> 例如，在保存词anvil 时，如果它后面有一个或多个空格，则子句WHERE prod_name LIKE '%anvil'将不会匹配它们，
> 因为在最后的l后有多余的字符。解决这个问题的一个简单的办法是在搜索模式最后附加一个%。
> 一个更好的办法是使用函数（第11章将会介绍）去掉首尾空格。
>
> 虽然似乎%通配符可以匹配任何东西，但有一个例外，即NULL。
> 即使是WHERE prod_name LIKE '%'也不能匹配用值NULL作为产品名的行。

- 不要过度使用通配符。如果其他操作符能达到相同的目的，应该使用其他操作符。
- 在确实需要使用通配符时，除非绝对有必要，否则不要把它们用在搜索模式的开始处。把通配符置于搜索模式的开始处，搜索起来是最慢的。
- 仔细注意通配符的位置。如果放错地方，可能不会返回想要的数据。

REGEXP正在表达式

- 为了匹配反斜杠（\）字符本身，需要使用\\\。
- 多数正则表达式实现使用单个反斜杠转义特殊字符，以便能使用这些字符本身。但MySQL要求两个反斜杠（MySQL自己解释一个，正则表达式库解释另一个）。
- ^有两种用法。在集合中（用[和]定义），用它来否定该集合，否则，用来指串的开始处。

数据处理函数

> 函数没有SQL的可移植性强

- [12.1 Function and Operator Reference](https://dev.mysql.com/doc/refman/5.7/en/func-op-summary-ref.html)

聚集函数

> 除了COUNT()函数外，其他聚集函数都会忽略NULL行。
>
> 如果指定列名，则指定列的值为NULL的行被COUNT()函数忽略，但如果COUNT()函数中用的是星号（*），则不忽略。
>
> 如果指定列名，则DISTINCT只能用于COUNT()。
> DISTINCT不能用于COUNT(*)，因此不允许使用COUNT（DISTINCT），否则会产生错误。
> 类似地，DISTINCT必须使用列名，不能用于计算或表达式。

- [12.19.1 Aggregate (GROUP BY) Function Descriptions](https://dev.mysql.com/doc/refman/5.7/en/group-by-functions.html)

分组统计

- 除聚集计算语句外，SELECT语句中的每个列都必须在GROUP BY子句中给出。
- 如果分组列中具有NULL值，则NULL将作为一个分组返回。如果列中有多行NULL值，它们将分为一组。

```
ERROR 1055 (42000): Expression #2 of SELECT list is not in GROUP
BY clause and contains nonaggregated column 'mydb.t.address' which
is not functionally dependent on columns in GROUP BY clause; this
is incompatible with sql_mode=only_full_group_by
```

> 在分组统计中，select 所选择的字段除了聚集函数外，其余字段必须出现在 group by 分组中。
> 但是也有例外，如果 group by 的字段是主键或唯一建则 select 所选择字段不做限制。

- [12.19.3 MySQL Handling of GROUP BY](https://dev.mysql.com/doc/refman/5.7/en/group-by-handling.html)

> HAVING和WHERE的差别
>
> 这里有另一种理解方法，WHERE在数据分组前进行过滤，HAVING在数据分组后进行过滤。
> 这是一个重要的区别，WHERE排除的行不包括在分组中。这可能会改变计算值，从而影响HAVING子句中基于这些值过滤掉的分组。

嵌套查询（子查询）

> 列必须匹配

> 在WHERE子句中使用子查询（如这里所示），应该保证SELECT语句具有与WHERE子句中相同数目的列。
> 通常，子查询将返回单个列并且与单个列匹配，但如果需要也可以使用多个列。

联接表

- 等值连接又称为内连接。

组合查询

- UNION必须由两条或两条以上的SELECT语句组成，语句之间用关键字UNION分隔（因此，如果组合4条SELECT语句，将要使用3个UNION关键字）。
- UNION中的每个查询必须包含相同的列、表达式或聚集函数（不过各个列不需要以相同的次序列出）。
- 列数据类型必须兼容：类型不必完全相同，但必须是DBMS可以隐含地转换的类型（例如，不同的数值类型或不同的日期类型）。

全文搜索

- 在索引全文本数据时，短词被忽略且从索引中排除。短词定义为那些具有3个或3个以下字符的词（如果需要，这个数目可以更改）。
- MySQL带有一个内建的非用词（stopword）列表，这些词在索引全文本数据时总是被忽略。如果需要，可以覆盖这个列表（请参阅MySQL文档以了解如何完成此工作）。
- 许多词出现的频率很高，搜索它们没有用处（返回太多的结果）。因此，MySQL规定了一条50%规则，如果一个词出现在50%以上的行中，则将它作为一个非用词忽略。50%规则不用于IN BOOLEANMODE。
- 如果表中的行数少于3行，则全文本搜索不返回结果（因为每个词或者不出现，或者至少出现在50%的行中）。
- 忽略词中的单引号。例如，don't索引为dont。
- 不具有词分隔符（包括日语和汉语）的语言不能恰当地返回全文本搜索结果。

插入数据

- 插入完整的行；
- 插入行的一部分；
- 插入多行；
- 插入某些查询的结果。

> 提高整体性能
>
> 数据库经常被多个客户访问，对处理什么请求以及用什么次序处理进行管理是MySQL的任务。
> INSERT操 作可能很耗时（特别是有很多索引需要更新时），而且它可能降低等待处理的SELECT语句的性能。
> 如果数据检索是最重要的（通常是这样），则你可以通过在INSERT和INTO之间添加关键字LOW_PRIORITY，指示MySQL降低INSERT语句的优先级，如下所示：
> INSERT LOW_PRIORITY INTO
> 顺便说一下，这也适用于下一章介绍的UPDATE和DELETE语句。

更新数据

> IGNORE关键字：如果用UPDATE语句更新多行，并且在更新这些行中的一行或多行时出一个现错误，
> 则整个UPDATE操作被取消（错误发生前更新的所有行被恢复到它们原来的值）。
> 为即使是发生错误，也继续进行更新，可使用IGNORE关键字，如下所示：
> UPDATE IGNORE customers…

删除数据

> 更快的删除：如果想从表中删除所有行，不要使用DELETE。
> 可使用TRUNCATE TABLE语句，它完成相同的工作，
> 但速度更快（TRUNCATE实际是删除原来的表并重新创建一个表，而不是逐行删除表中的数据）。

- 除非确实打算更新和删除每一行，否则绝对不要使用不带WHERE子句的UPDATE或DELETE语句。
- 保证每个表都有主键（如果忘记这个内容，请参阅第15章），尽可能像WHERE子句那样使用它（可以指定各主键、多个值或值的范围）。
- 在对UPDATE或DELETE语句使用WHERE子句前，应该先用SELECT进行测试，保证它过滤的是正确的记录，以防编写的WHERE子句不正确。
- 使用强制实施引用完整性的数据库（关于这个内容，请参阅第15章），这样MySQL将不允许删除具有与其他表相关联的数据的行。

创建和操纵表

- 用新的列布局创建一个新表；
- 使用INSERT SELECT语句（关于这条语句的详细介绍，请参阅第19章）从旧表复制数据到新表。如果有必要，可使用转换函数和计算字段；
- 检验包含所需数据的新表；
- 重命名旧表（如果确定，可以删除它）；
- 用旧表原来的名字重命名新表；
- 根据需要，重新创建触发器、存储过程、索引和外键。

> 覆盖AUTO_INCREMENT： 如果一个列被指定为AUTO_INCREMENT，则它需要使用特殊的值吗？
> 你可以简单地在INSERT语句中指定一个值，只要它是唯一的（至今尚未使用过）即可，
> 该值将被用来替代自动生成的值。后续的增量将开始使用该手工插入的值。（相关的例子请参阅本书中使用的表填充脚本。）
>
> 确定AUTO_INCREMENT值：让MySQL生成（通过自动增量）主键的一个缺点是你不知道这些值都是谁。
> 那么，如何在使用AUTO_INCREMENT列时获得这个值呢？可使用last_insert_id()函数获得这个值，如下所示：
> select last_insert_id() 此语句返回最后一个AUTO_INCREMENT值，然后可以将它用于后续的MySQL语句。

- [13.1.7 ALTER TABLE Syntax](https://dev.mysql.com/doc/refman/5.6/en/alter-table.html)
- [13.1.32 RENAME TABLE Syntax](https://dev.mysql.com/doc/refman/5.6/en/rename-table.html)

视图

为什么使用视图？

- 重用SQL语句。
- 简化复杂的SQL操作。在编写查询后，可以方便地重用它而不必知道它的基本查询细节。
- 使用表的组成部分而不是整个表。
- 保护数据。可以给用户授予表的特定部分的访问权限而不是整个表的访问权限。
- 更改数据格式和表示。视图可返回与底层表的表示和格式不同的数据。

视图的规则和限制

- 与表一样，视图必须唯一命名（不能给视图取与别的视图或表相同的名字）。
- 对于可以创建的视图数目没有限制。
- 为了创建视图，必须具有足够的访问权限。这些限制通常由数据库管理人员授予。
- 视图可以嵌套，即可以利用从其他视图中检索数据的查询来构造一个视图。
- ORDER BY可以用在视图中，但如果从该视图检索数据SELECT中也含有ORDER BY，那么该视图中的ORDER BY将被覆盖。
- 视图不能索引，也不能有关联的触发器或默认值。
- 视图可以和表一起使用。例如，编写一条联结表和视图的SELECT语句。

更新视图

> 如果视图定义中有以下操作，则不能进行视图的更新：

- 分组（使用GROUP BY和HAVING）；
- 联结；
- 子查询；
- 并；
- 聚集函数（Min()、Count()、Sum()等）；
- DISTINCT；
- 导出（计算）列。

存储过程

- 所有MySQL变量都必须以@开始。

为什么要使用存储过程？

- 通过把处理封装在容易使用的单元中，简化复杂的操作（正如前面例子所述）。
- 由于不要求反复建立一系列处理步骤，这保证了数据的完整性。如果所有开发人员和应用程序都使用同一（试验和测试）存储过程，则所使用的代码都是相同的。这一点的延伸就是防止错误。需要执行的步骤越多，出错的可能性就越大。防止错误保证了数据的一致性。
- 简化对变动的管理。如果表名、列名或业务逻辑（或别的内容）有变化，只需要更改存储过程的代码。使用它的人员甚至不需要知道这些变化。
- 提高性能。因为使用存储过程比使用单独的SQL语句要快。
- 存在一些只能用在单个请求中的MySQL元素和特性，存储过程可以使用它们来编写功能更强更灵活的代码。
- [13.1.16 CREATE PROCEDURE and CREATE FUNCTION Syntax](https://dev.mysql.com/doc/refman/5.7/en/create-procedure.html)

游标

> 只能用于存储过程。不像多数DBMS，MySQL游标只能用于存储过程（和函数）。

- 在能够使用游标前，必须声明（定义）它。这个过程实际上没有检索数据，它只是定义要使用的SELECT语句。
- 一旦声明后，必须打开游标以供使用。这个过程用前面定义的SELECT语句把数据实际检索出来。
- 对于填有数据的游标，根据需要取出（检索）各行。
- 在结束游标使用时，必须关闭游标。
- [13.6.6 Cursors](https://dev.mysql.com/doc/refman/5.7/en/cursors.html)

触发器

在创建触发器时，需要给出4条信息：

- 唯一的触发器名；
- 触发器关联的表；
- 触发器应该响应的活动（DELETE、INSERT或UPDATE）；
- 触发器何时执行（处理之前或之后）。
- [13.1.20 CREATE TRIGGER Syntax](https://dev.mysql.com/doc/refman/5.7/en/create-trigger.html)

知识点：

- 创建触发器可能需要特殊的安全访问权限，但是，触发器的执行是自动的。如果INSERT、UPDATE或DELETE语句能够执行，则相关的触发器也能执行。
- 应该用触发器来保证数据的一致性（大小写、格式等）。在触发器中执行这种类型的处理的优点是它总是进行这种处理，而且是透明地进行，与客户机应用无关。
- 触发器的一种非常有意义的使用是创建审计跟踪。使用触发器，把更改（如果需要，甚至还有之前和之后的状态）记录到另一个表非常容易。
- 遗憾的是，MySQL触发器中不支持CALL语句。这表示不能从触发器内调用存储过程。所需的存储过程代码需要复制到触发器内。

事务

- 事务（transaction）指一组SQL语句；
- 回退（rollback）指撤销指定SQL语句的过程；
- 提交（commit）指将未存储的SQL语句结果写入数据库表；
- 保留点（savepoint）指事务处理中设置的临时占位符（placeholder），你可以对它发布回退（与回退整个事务处理不同）。
- [13.3.1 START TRANSACTION, COMMIT, and ROLLBACK Syntax](https://dev.mysql.com/doc/refman/5.7/en/commit.html)

ACID特性

- 原子性（Atomicity）
- 一致性（Consistency）
- 隔离性（Isolation）
- 持久性（Durability）

> 哪些语句可以回退？
> 事务处理用来管理INSERT、UPDATE和DELETE语句。你不能回退SELECT语句。（这样做也没有什么意义。）
> 你不能回退CREATE或DROP操作。事务处理块中可以使用这两条语句，但如果你执行回退，它们不会被撤销。
>
> 隐含事务关闭：当COMMIT或ROLLBACK语句执行后，事务会自动关闭（将来的更改会隐含提交）。

字符集

- 字符集为字母和符号的集合；
- 编码为某个字符集成员的内部表示；
- 校对为规定字符如何比较的指令。

> 校对为什么重要：排序英文正文很容易，对吗？或许不。考虑词APE、apex和Apple。它们处于正确的排序顺序吗？这有赖于你是否想区分大小写。
> 使用区分大小写的校对顺序，这些词有一种排序方式，使用不区分大小写的校对顺序有另外一种排序方式。
> 这不仅影响排序（如用ORDER BY排序数据），还影响搜索（例如，寻找apple的WHERE子句是否能找到APPLE）。
> 在使用诸如法文à或德文ö这样的字符时，情况更复杂，在使用不基于拉丁文的字符集（日文、希伯来文、俄文等）时，情况更为复杂。

## 灵感

- 如何修改 auto_increment 当前的自动增量和步长？
- 如何设定默认字符集？
- count(*) 和 count(field) 哪个更快？
- 什么是自然连接？
- 笛卡尔积与自然连接的区别？
- 内连接、自连接、自然连接、外连接？
- 如何杀死查询时间太长的进程？[1](https://blog.csdn.net/baidu_33615716/article/details/78986799)、[2](https://gist.github.com/paulrosania/2883869)
- 如何实现字段内容压缩存储？
- MyISAM/InnoDB/Memory 引擎的区别？
- MySQL 的全文搜索（Full Text Search）好用吗？
- 中文字（不带分隔符）的全文搜索可以用吗？
- 认识 select/insert/update/delete 的完整语法；
- 如何在同一条SQL语句里，根据不同的条件更新不同的记录？ when than
- 如何生成分布式ID？
- 新增字段时，使用after或before指定添加的位置会慢一些吗？
- char 和 varchar 有什么区别？
- 熟悉存储过程和自定义函数的语法。
- 如何优化分页数较大的速度？offset limit
- 如何实现数据库的SQL审计功能？trigger
- 如何加快查询速度？索引有哪些使用方法？
- 如何保证在读和写之间数据的一致性？
- 分布式事务？
- 如何保证并发事务的一致性？
- 并发事务的效果需要进行演示。
- 事务的ACID特征？[1](https://blog.csdn.net/u012440687/article/details/52116108)
- [Difference between SET autocommit=1 and START TRANSACTION?](https://stackoverflow.com/questions/2950676/difference-between-set-autocommit-1-and-start-transaction-in-mysql-have-i-misse)
- 字符集 utf8 和 utf8mb4 的区别？
- 较对集 utf8mb4_unicode_ci 和 utf8mb4_general_ci 的区别？
- 字符集 character_set_client/connection/database/server/results 的含义？
- Emoji表情应该如何存储？
- ROW_FORMAT=COMPRESSED 是什么意思？
