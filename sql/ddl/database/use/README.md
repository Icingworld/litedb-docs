# USE 语句

USE 语句用于切换当前会话所使用的数据库。

## 语法

```enbf
use_statement ::= USE database_name

database_name ::= identifier
```

## 参数

| 参数 | 说明 |
| --- | --- |
| `database_name` | 数据库名称 |

## 示例

```sql
USE my_database;
```
