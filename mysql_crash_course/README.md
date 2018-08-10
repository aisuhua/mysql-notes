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

## 灵感

- 如何修改 auto_increment 当前的自动增量和步长？
- 如何设定默认字符集？
- count(*) 和 count(field) 哪个更快？
- 什么是自然连接？
- 笛卡尔积与自然连接的区别？
- 内连接、自连接、自然连接、外连接？
- 如何杀死查询时间太长的进程？[1](https://blog.csdn.net/baidu_33615716/article/details/78986799)、[2](https://gist.github.com/paulrosania/2883869)
- 如何实现字段内容压缩存储？
- MyISAM 和 InnoDB 的区别？
- MySQL 的全文搜索（Full Text Search）好用吗？
