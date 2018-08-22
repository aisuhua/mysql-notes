# 创建数据库和表

## 创建数据库

创建数据库

```sql
CREATE DATABASE mydb CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

## 创建表

```sql
CREATE TABLE files (
    `file_id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键ID',
    `file_name` varchar(512) NOT NULL COMMENT '文件名称',
    `parent_id` bigint(20) NOT NULL DEFAULT '0' COMMENT '父目录ID',
    `area_id` tinyint(1) NOT NULL DEFAULT '1' COMMENT 'aid',
    `user_id` int(11) NOT NULL COMMENT '用户id',
    `sha1` char(40) CHARACTER SET latin1 NOT NULL COMMENT 'sha1值',
    PRIMARY KEY (`file_id`),
    KEY `name` (`file_name`),
    KEY `sha1` (`sha1`),
    KEY `key1` (`user_id`, `area_id`, `parent_id`)
) ENGINE InnoDB CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

## 克隆表

- LIKE

```sql
CREATE TABLE new_tbl LIKE orig_tbl;
```

- [AS] query_expression

```sql
CREATE TABLE new_tbl AS SELECT * FROM orig_tbl;
```

- [Cloning or Copying a Table](https://dev.mysql.com/doc/refman/5.7/en/create-table.html#create-table-clone-copy)

## 修改表

- 重命名表
- 添加字段
- 添加索引
- 修改自增ID