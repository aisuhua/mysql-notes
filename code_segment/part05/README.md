# 维护和管理

## 查看表的行数、总大小、平均每行大小等

```sql
mysql> SELECT * FROM tables WHERE information_schema.table_schema = 'study' AND table_name = 'demo'\G
*************************** 1. row ***************************
  TABLE_CATALOG: def
   TABLE_SCHEMA: study
     TABLE_NAME: demo
     TABLE_TYPE: BASE TABLE
         ENGINE: InnoDB
        VERSION: 10
     ROW_FORMAT: Compact
     TABLE_ROWS: 9209756
 AVG_ROW_LENGTH: 336
    DATA_LENGTH: 3095396352
MAX_DATA_LENGTH: 0
   INDEX_LENGTH: 684720128
      DATA_FREE: 4194304
 AUTO_INCREMENT: 9964860
    CREATE_TIME: 2018-07-20 17:58:28
    UPDATE_TIME: NULL
     CHECK_TIME: NULL
TABLE_COLLATION: utf8mb4_general_ci
       CHECKSUM: NULL
 CREATE_OPTIONS:
  TABLE_COMMENT: some comment here.
   BLOCK_FORMAT: Original
1 row in set (0.00 sec)

# 或者通过下面方式查看
show table status like "city"\G
```

## 修改表当前的 auto_increment 值

```sql
ALTER TABLE demo AUTO_INCREMENT = 13;
```

## 如何修改 auto_increment 默认的自动增量初始值和步长？

```sql
# 修改初始值
mysql> set @@auto_increment_offset = 5;
Query OK, 0 rows affected (0.00 sec)

# 修改自动增长步长
mysql> set @@auto_increment_increment = 10;
Query OK, 0 rows affected (0.00 sec)

mysql> show variables like "auto_incr%";
+--------------------------+-------+
| Variable_name            | Value |
+--------------------------+-------+
| auto_increment_increment | 10    |
| auto_increment_offset    | 5     |
+--------------------------+-------+

mysql> create table demo (id int auto_increment primary key);
Query OK, 0 rows affected (0.08 sec)

mysql> insert into demo values(null);
Query OK, 1 row affected (0.05 sec)

mysql> select * from demo;
+----+
| id |
+----+
|  5 |
+----+
1 row in set (0.00 sec)

mysql> insert into demo values(null);
Query OK, 1 row affected (0.05 sec)

mysql> select * from demo;
+----+
| id |
+----+
|  5 |
| 15 |
+----+
2 rows in set (0.00 sec)
```

## 查看系统状态

``` sql
# 服务器状态
mysql> show status;

# 当前字符集
mysql> show variables like "character%";

# 当前校对集
mysql> show variables like "collation%";

# 当前默认引擎
mysql> show engines;

# 自动提交是否开启
mysql> show variables like "autocommit";

# 运行模式
mysql> show variables like "sql_mode";

# 事件调度器服务是否开启
mysql> show variables like "event_scheduler";
```

- [16.1.6.2 Replication Master Options and Variables](https://dev.mysql.com/doc/refman/5.7/en/replication-options-master.html#sysvar_auto_increment_increment)
