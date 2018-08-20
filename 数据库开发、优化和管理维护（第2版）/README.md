# 数据库开发、优化与管理维护（第2版）

## 日期时间类型

### timestamp 的使用

> 实际上 datetime 也可以声明为 default CURRENT_TIMESTAMP on update CURRENT_TIMESTAMP。
> 区别在于如果插入的值为 null 时，datetime 为 null，而 timestamp 为时间戳。

演示

```sql
mysql> create table demo (a int, b timestamp);
Query OK, 0 rows affected (0.03 sec)

mysql> desc demo;
+-------+-----------+------+-----+-------------------+-----------------------------+
| Field | Type      | Null | Key | Default           | Extra                       |
+-------+-----------+------+-----+-------------------+-----------------------------+
| a     | int(11)   | YES  |     | NULL              |                             |
| b     | timestamp | NO   |     | CURRENT_TIMESTAMP | on update CURRENT_TIMESTAMP |
+-------+-----------+------+-----+-------------------+-----------------------------+
2 rows in set (0.01 sec)

mysql> show create table demo\G
*************************** 1. row ***************************
       Table: demo
Create Table: CREATE TABLE `demo` (
  `a` int(11) DEFAULT NULL,
  `b` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=latin1
1 row in set (0.00 sec)

mysql> insert into demo values (1, null);
Query OK, 1 row affected (0.00 sec)

mysql> select * from demo;
+------+---------------------+
| a    | b                   |
+------+---------------------+
|    1 | 2018-08-19 18:16:43 |
+------+---------------------+
1 row in set (0.00 sec)

mysql> update demo set a = 2 where a = 1;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> select * from demo;
+------+---------------------+
| a    | b                   |
+------+---------------------+
|    2 | 2018-08-19 18:17:05 |
+------+---------------------+
```

- [https://dev.mysql.com/doc/refman/5.7/en/timestamp-initialization.html](https://dev.mysql.com/doc/refman/5.7/en/timestamp-initialization.html)

## datetime 和 timestamp

演示

```sql
mysql> create table demo (a int, b datetime, c timestamp);
Query OK, 0 rows affected (0.05 sec)

mysql> desc demo;
+-------+-----------+------+-----+-------------------+-----------------------------+
| Field | Type      | Null | Key | Default           | Extra                       |
+-------+-----------+------+-----+-------------------+-----------------------------+
| a     | int(11)   | YES  |     | NULL              |                             |
| b     | datetime  | YES  |     | NULL              |                             |
| c     | timestamp | NO   |     | CURRENT_TIMESTAMP | on update CURRENT_TIMESTAMP |
+-------+-----------+------+-----+-------------------+-----------------------------+

mysql> insert into demo values (1, null, null);
Query OK, 1 row affected (0.01 sec)

mysql> select * from demo;
+------+------+---------------------+
| a    | b    | c                   |
+------+------+---------------------+
|    1 | NULL | 2018-08-19 18:28:29 |
+------+------+---------------------+
1 row in set (0.00 sec)

mysql> alter table demo modify b datetime default CURRENT_TIMESTAMP on update CURRENT_TIMESTAMP;
Query OK, 0 rows affected (0.02 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> insert into demo values (1, null, null);
Query OK, 1 row affected (0.01 sec)

mysql> select * from demo;
+------+------+---------------------+
| a    | b    | c                   |
+------+------+---------------------+
|    1 | NULL | 2018-08-19 18:28:29 |
|    1 | NULL | 2018-08-19 18:29:39 |
+------+------+---------------------+
2 rows in set (0.00 sec)

mysql> update demo set a = 2 where a = 1;
Query OK, 2 rows affected (0.00 sec)
Rows matched: 2  Changed: 2  Warnings: 0

mysql> select * from demo;
+------+---------------------+---------------------+
| a    | b                   | c                   |
+------+---------------------+---------------------+
|    2 | 2018-08-19 18:29:59 | 2018-08-19 18:29:59 |
|    2 | 2018-08-19 18:29:59 | 2018-08-19 18:29:59 |
+------+---------------------+---------------------+
2 rows in set (0.00 sec)

mysql> alter table demo modify b datetime not null DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
    -> ;
Query OK, 0 rows affected (0.13 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> desc demo;
+-------+-----------+------+-----+-------------------+-----------------------------+
| Field | Type      | Null | Key | Default           | Extra                       |
+-------+-----------+------+-----+-------------------+-----------------------------+
| a     | int(11)   | YES  |     | NULL              |                             |
| b     | datetime  | NO   |     | CURRENT_TIMESTAMP | on update CURRENT_TIMESTAMP |
| c     | timestamp | NO   |     | CURRENT_TIMESTAMP | on update CURRENT_TIMESTAMP |
+-------+-----------+------+-----+-------------------+-----------------------------+
3 rows in set (0.00 sec)

mysql> select * from demo;
+------+---------------------+---------------------+
| a    | b                   | c                   |
+------+---------------------+---------------------+
|    2 | 2018-08-19 18:29:59 | 2018-08-19 18:29:59 |
|    2 | 2018-08-19 18:29:59 | 2018-08-19 18:29:59 |
+------+---------------------+---------------------+
2 rows in set (0.00 sec)

mysql> insert into demo values (1, null, null);
ERROR 1048 (23000): Column 'b' cannot be null
```

## enum 枚举类型

演示

```sql
mysql> create table demo (gender enum('M', 'F'));
Query OK, 0 rows affected (0.06 sec)

mysql> desc demo;
+--------+---------------+------+-----+---------+-------+
| Field  | Type          | Null | Key | Default | Extra |
+--------+---------------+------+-----+---------+-------+
| gender | enum('M','F') | YES  |     | NULL    |       |
+--------+---------------+------+-----+---------+-------+
1 row in set (0.00 sec)

mysql> insert into demo values ('M'), ('f'), (1), (null);
Query OK, 4 rows affected (0.06 sec)
Records: 4  Duplicates: 0  Warnings: 0

mysql> select * from demo;
+--------+
| gender |
+--------+
| M      |
| F      |
| M      |
| NULL   |
+--------+
4 rows in set (0.00 sec)
```

从上面例子中可以看出，ENUM 类型是忽略大小写的，在存储 “M” 和 “f” 时将它们都转成了大写，
还可以看出对于插入不在 ENUM 指定范围内的值时，并没有警告，而是插入了 enum('M', 'F') 的第一个值 “M”，这点用户使用时要特别注意。

另外，ENUM 类型只允许从值集合中选取单个值，而不能一次取多个值。

## SET 集合类型

演示

```sql
mysql> create table demo (col set('a', 'b', 'c', 'd'));
Query OK, 0 rows affected (0.01 sec)

mysql> desc demo;
+-------+----------------------+------+-----+---------+-------+
| Field | Type                 | Null | Key | Default | Extra |
+-------+----------------------+------+-----+---------+-------+
| col   | set('a','b','c','d') | YES  |     | NULL    |       |
+-------+----------------------+------+-----+---------+-------+
1 row in set (0.00 sec)

mysql> insert into demo values('a,b'), ('a,d,a'), ('a,c'), ('c');
Query OK, 4 rows affected (0.05 sec)
Records: 4  Duplicates: 0  Warnings: 0

mysql> select * from demo;
+------+
| col  |
+------+
| a,b  |
| a,d  |
| a,c  |
| c    |
+------+
4 rows in set (0.00 sec)

mysql> insert into demo values ('a,');
ERROR 1265 (01000): Data truncated for column 'col' at row 1
mysql> insert into demo values ('f');
ERROR 1265 (01000): Data truncated for column 'col' at row 1
```

SET 和 ENUM 除了存储以外，最主要的区别在于 SET 类型一次可以选取多个成员，而 ENUM 只能选一个。

SET 类型可以从允许的值集合中选取任意 1 个或多个元素进行组合，所以对于输入的值只要是在允许的组合范围内，
都可以正确地注入到 SET 类型的列中。对于超出允许范围的值例如（'a,b,f'）将不允许注入到上面的例子中设置的 SET 类型列中，
而对于（'a,b,a'）这样包含重复的成员的集合将只取一次，写入后的结果为 “a,b”，这一点请注意。

补充：多个元素时后面不能带逗号，否则插入失败。例如（'a,b,'）会插入失败。

查询的方式

```sql
mysql> select * from demo where FIND_IN_SET('a', col);
+------+
| col  |
+------+
| a,b  |
| a,d  |
| a,c  |
+------+
3 rows in set (0.00 sec)

mysql> select * from demo where col like "a%";
+------+
| col  |
+------+
| a,b  |
| a,d  |
| a,c  |
+------+
3 rows in set (0.00 sec)
```

- [11.4.5 The SET Type](https://dev.mysql.com/doc/refman/5.7/en/set.html)

## 时间函数

演示

```sql
mysql> select unix_timestamp('2018-8-20 14:54:54');
+--------------------------------------+
| unix_timestamp('2018-8-20 14:54:54') |
+--------------------------------------+
|                           1534748094 |
+--------------------------------------+
1 row in set (0.01 sec)

mysql> select from_unixtime(1534748094);
+---------------------------+
| from_unixtime(1534748094) |
+---------------------------+
| 2018-08-20 14:54:54       |
+---------------------------+
1 row in set (0.00 sec)

mysql> select now() current,
    > date_add(now(), INTERVAL 31 day) after31days,
    > date_add(now(), INTERVAL '1_2' year_month) after_oneyear_twomonth;
+---------------------+---------------------+------------------------+
| current             | after31days         | after_oneyear_twomonth |
+---------------------+---------------------+------------------------+
| 2018-08-20 15:03:00 | 2018-09-20 15:03:00 | 2019-10-20 15:03:00    |
+---------------------+---------------------+------------------------+

mysql> select now() current,
    > date_add(now(), INTERVAL -31 day) before31days,
    > date_add(now(), INTERVAL '-1_-2' year_month) before_oneyear_twomonth;
+---------------------+---------------------+-------------------------+
| current             | before31days        | before_oneyear_twomonth |
+---------------------+---------------------+-------------------------+
| 2018-08-20 15:04:41 | 2018-07-20 15:04:41 | 2017-06-20 15:04:41     |
+---------------------+---------------------+-------------------------+
1 row in set (0.00 sec)

mysql> select datediff('2019-09-09', NOW());
+-------------------------------+
| datediff('2019-09-09', NOW()) |
+-------------------------------+
|                           385 |
+-------------------------------+
1 row in set (0.00 sec)
```

- UNIX_TIMESTAMP(date) 返回日期的 Unix 时间戳
- FROM_UNIXTIME(timestamp) 返回 Unix 时间戳的日期值
- DATE_ADD(date, INTERVAL expr type) 返回一个日期或时间值加上一个时间间隔的时间值
- DATEDIFF(expr, expr) 返回起始时间和结束时间
- [12.7 Date and Time Functions](https://dev.mysql.com/doc/refman/5.7/en/date-and-time-functions.html)

















