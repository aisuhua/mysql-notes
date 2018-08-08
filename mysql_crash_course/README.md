# MySQL Crash Course

Book link: http://www.forta.com/books/0672327120/

Import db

```sql
create database crashcourse
use crashcourse
source /path/to/create.sql
source /path/to/populate.sql
```

SQL

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
help show
# 
```

灵感

- 如何修改 auto_increment 当前的自动增量和步长？
