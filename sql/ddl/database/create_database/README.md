# CREATE DATABASE 语句

CREATE DATABASE 语句用于创建一个新数据库。

## 语法

```enbf
create_database_statement ::= CREATE DATABASE [ IF NOT EXISTS ] database_name

database_name ::= identifier
```

## 参数

| 参数 | 说明 |
| --- | --- |
| `IF NOT EXISTS` | 如果数据库不存在，则创建数据库 |
| `database_name` | 数据库名称 |

## 示例

```sql
CREATE DATABASE my_database;
CREATE DATABASE IF NOT EXISTS my_database;
```
