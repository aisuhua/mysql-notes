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

mysql> inser into demo values (1, null, null);
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'inser into demo values (1, null, null)' at line 1
mysql> insert into demo values (1, null, null);
ERROR 1048 (23000): Column 'b' cannot be null
```
