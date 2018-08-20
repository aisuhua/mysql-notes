# 维护和管理

查看表的行数、总大小、平均每行大小等

```sql
SELECT * FROM tables WHERE information_schema.table_schema = 'study' AND table_name = 'demo'\G
```
