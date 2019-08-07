# 1.如何查看数据库占用磁盘的大小

```sql
select 
    sum((data_length+index_length)/1024/1024) M
from 
    information_schema.tables 
where 
    table_schema="eim" ;
```

# 2.如何查看某张表的数据条数

```
SELECT COUNT(*) FROM ucPointFeeDetail;
```

