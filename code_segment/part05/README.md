# 维护和管理

查看表的行数、总大小、平均每行大小等

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
```
