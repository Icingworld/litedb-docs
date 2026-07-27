# DROP DATABASE 语句

DROP DATABASE 语句用于删除一个数据库。

## 语法

```enbf
drop_database_statement ::= DROP DATABASE [ IF EXISTS ] database_name

database_name ::= identifier
```

## 参数

| 参数 | 说明 |
| --- | --- |
| `IF EXISTS` | 如果数据库存在，则删除数据库 |
| `database_name` | 数据库名称 |

## 示例

```sql
DROP DATABASE my_database;
DROP DATABASE IF EXISTS my_database;
```
