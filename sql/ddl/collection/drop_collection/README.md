# DROP COLLECTION 语句

DROP COLLECTION 语句用于删除当前数据库中的一个集合。

删除集合前需要先 [USE](../../database/use/README.md) 到目标数据库。删除集合时，其列定义以及关联的标量索引、向量索引会一并清理。

## 语法

```ebnf
drop_collection_statement ::= DROP COLLECTION [ IF EXISTS ] collection_name

collection_name ::= identifier
```

## 参数

| 参数 | 说明 |
| --- | --- |
| `IF EXISTS` | 集合不存在时不报错；存在时正常删除 |
| `collection_name` | 集合名称 |

## 示例

```sql
DROP COLLECTION users;
DROP COLLECTION IF EXISTS users;
```
