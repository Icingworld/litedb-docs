# SHOW COLLECTIONS 语句

SHOW COLLECTIONS 语句用于列出数据库中的集合。省略 `FROM` 时，列出当前会话数据库中的集合；指定 `FROM` 时可查看其他数据库。

未指定 `FROM` 时，需要先 [USE](../../database/use/README.md) 到某个数据库。

## 语法

```ebnf
show_collections_statement ::= SHOW COLLECTIONS [ FROM database_name ]

database_name ::= identifier
```

## 参数

| 参数 | 说明 |
| --- | --- |
| `FROM database_name` | 可选；指定要列出集合的数据库名称 |
| `database_name` | 数据库名称 |

## 返回结果

| 列名 | 类型 | 说明 |
| --- | --- | --- |
| `collection_name` | `VARCHAR` | 集合名称 |

## 示例

```sql
SHOW COLLECTIONS;
SHOW COLLECTIONS FROM demo;
```
