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
- 搜索模式（search pattern）① 由字面值、通配符或两者组合构成的搜索条件。
- 默认不区分大小写，但是可配置。

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

## 灵感

- 如何修改 auto_increment 当前的自动增量和步长？
- 如何设定默认字符集？
