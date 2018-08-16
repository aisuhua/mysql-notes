
## 创建用户及授权

创建后的效果

```sql
mysql> SHOW GRANTS FOR 'myname'@'172.16.%';
+-------------------------------------------------------------------------------------------------------------------+
| Grants for myname@172.16.%                                                                                        |
+-------------------------------------------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO 'myname'@'172.16.%'                                                                         |
| GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, CREATE TEMPORARY TABLES, EXECUTE ON `mydb`.* TO 'myname'@'172.16.%' |
+-------------------------------------------------------------------------------------------------------------------+
```

## 创建用户

创建用户 `'myname'@'172.16.%'`

```sql
CREATE USER 'myname'@'172.16.%' IDENTIFIED BY '123456'
```

查看用户信息

```sql
SELECT * FROM mysql.user WHERE user = 'myname'\G
```

## 用户授权

查看用户权限（授权前）

```sql
mysql> SHOW GRANTS FOR 'myname'@'172.16.%';
+-------------------------------------------+
| Grants for myname@172.16.%                |
+-------------------------------------------+
| GRANT USAGE ON *.* TO 'myname'@'172.16.%' |
+-------------------------------------------+
```

用户授权

```sql
GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, CREATE TEMPORARY TABLES, EXECUTE ON `mydb`.* TO 'myname'@'172.16.%'
```

查看用户权限（授权后）

```sql
mysql> SHOW GRANTS FOR 'myname'@'172.16.%';
+-------------------------------------------------------------------------------------------------------------------+
| Grants for myname@172.16.%                                                                                        |
+-------------------------------------------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO 'myname'@'172.16.%'                                                                         |
| GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, CREATE TEMPORARY TABLES, EXECUTE ON `mydb`.* TO 'myname'@'172.16.%' |
+-------------------------------------------------------------------------------------------------------------------+
```

## 其他

新增授权（`DROP`）

```sql
GRANT DROP ON mydb.* TO 'myname'@'172.16.%';
```

授予全部权限

```sql
GRANT ALL ON mydb.* TO 'myname'@'172.16.%';
```

删除某个权限

```sql
REVOKE DROP ON mydb.* FROM 'myname'@'172.16.%'
```

创建只读帐号

```sql
# 创建用户
CREATE USER 'readonly'@'172.16.%' IDENTIFIED BY '123456'
# 授权
GRANT SELECT ON mydb.* TO 'readonly'@'172.16.%'
```

删除用户帐号

```sql
DROP USER 'readonly'@'172.16.%'
```

修改用户密码

```sql
SET PASSWORD FOR 'myname'@'172.16.%' = '654321'
```

修改用户名（可通过修改用户名允许更多服务器访问数据库）

```sql
RENAME 'myname'@'172.16.%' TO 'myname'@'%'
```




