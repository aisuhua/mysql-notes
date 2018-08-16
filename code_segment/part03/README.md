# 存储过程和函数

创建表

```sql
CREATE TABLE `u_user_cid` (
  `user_id` int(11) unsigned NOT NULL COMMENT '用户ID',
  `cid` int(10) unsigned NOT NULL DEFAULT '0',
  PRIMARY KEY (`user_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
```

创建函数

```sql
CREATE FUNCTION `get_cid`(user_id int) RETURNS int(11)
begin

    declare result  int unsigned;
    UPDATE u_user_cid set cid=last_insert_id(cid+1) where user_id=user_id;
    set result = last_insert_id();
    return result;
end
```

初始化

```sql
insert into u_user_cid(user_id, cid) values(10001, 0)
```

调用函数

```sql
mysql> select get_cid(10001) as cid;
+------+
| cid  |
+------+
|    1 |
+------+
1 row in set (0.00 sec)

mysql> select get_cid(10001) as cid;
+------+
| cid  |
+------+
|    2 |
+------+
1 row in set (0.01 sec)

mysql> select get_cid(10001) as cid;
+------+
| cid  |
+------+
|    3 |
+------+
1 row in set (0.00 sec)
```



