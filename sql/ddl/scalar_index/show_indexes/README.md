# SHOW INDEXES 语句

`SHOW INDEXES` 用于查看当前数据库中指定集合的标量索引定义。该语句必须明确指定集合，不能省略 `FROM`。

## 语法

```ebnf
show_indexes_statement ::= SHOW INDEXES FROM collection_name

collection_name ::= identifier
```

## 参数

| 参数 | 说明 |
| --- | --- |
| `FROM collection_name` | 要查看索引的集合；集合必须属于当前 `USE` 选中的数据库 |

向量索引不包含在本语句结果中；请使用 [SHOW VINDEXES](../../vector_index/show_vindexes/README.md)。

## 返回结果

每个标量索引返回一行，列顺序和名称如下：

| 列名 | 类型 | 说明 |
| --- | --- | --- |
| `index_name` | `VARCHAR` | 索引名称 |
| `column_name` | `VARCHAR` | 被索引的列名称 |
| `type` | `VARCHAR` | 索引类型，当前为 `BTREE` |
| `unique` | `BOOLEAN` | 是否为唯一索引 |

列级 `UNIQUE` 自动生成的隐式索引也会出现在结果中。没有标量索引时，语句返回相同列结构但没有数据行。

## 示例

```sql
USE demo;

SHOW INDEXES FROM users;
```

可能得到如下结果：

| index_name | column_name | type | unique |
| --- | --- | --- | --- |
| `idx_age` | `age` | `BTREE` | `false` |
| `idx_email` | `email` | `BTREE` | `true` |

实际结果还可能包含列级 `UNIQUE` 产生的隐式索引。
