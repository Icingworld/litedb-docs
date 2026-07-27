# DESCRIBE 语句

DESCRIBE 语句用于查看集合的结构。`DESC` 是 `DESCRIBE` 的简写。

查看集合前需要先 [USE](../../database/use/README.md) 到目标数据库。

## 语法

```ebnf
describe_statement ::= (DESCRIBE | DESC) [ COLLECTION ] collection_name

collection_name ::= identifier
```

## 参数

| 参数 | 说明 |
| --- | --- |
| `COLLECTION` | 可选关键字，不影响语义 |
| `collection_name` | 集合名称 |

## 返回结果

每一列对应结果集中的一行：

| 列名 | 类型 | 说明 |
| --- | --- | --- |
| `column_name` | `VARCHAR` | 列名称 |
| `type` | `VARCHAR` | 列类型，如 `BIGINT`、`VARCHAR(64)`、`VECTOR(3)` |
| `nullable` | `BOOLEAN` | 是否可空 |
| `unique` | `BOOLEAN` | 是否声明了 `UNIQUE` |
| `comment` | `VARCHAR` | 列注释；无则为 `NULL` |
| `collection_comment` | `VARCHAR` | 集合注释；无则为 `NULL`（各行重复同一值） |

## 示例

```sql
DESCRIBE users;
DESC users;
DESCRIBE COLLECTION users;
```
