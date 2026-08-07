# DROP INDEX 语句

`DROP INDEX` 用于删除当前数据库中指定集合上的标量索引。

## 语法

```ebnf
drop_index_statement ::= DROP INDEX [ IF EXISTS ] index_name ON collection_name

index_name ::= identifier
collection_name ::= identifier
```

索引名称写在 `IF EXISTS` 后，集合名称写在 `ON` 后。删除向量索引应使用 [DROP VINDEX](../../vector_index/drop_vindex/README.md)，不能混用两种语句。

## 参数

| 参数 | 说明 |
| --- | --- |
| `IF EXISTS` | 可选；索引不存在时返回空操作，不报错 |
| `index_name` | 要删除的标量索引名称 |
| `collection_name` | 索引所属集合；必须属于当前 `USE` 选中的数据库 |

## 行为

删除成功后，索引定义从 Catalog 移除，运行时索引也不再参与后续查询和写入维护。正常删除命令的 `affected_rows` 为 `1`；使用 `IF EXISTS` 删除不存在的索引时为 `0`。

列级 `UNIQUE` 创建的隐式唯一索引不能单独删除。尝试删除它会返回错误；删除整个集合时，该索引会随集合一起清理。

不带 `IF EXISTS` 时，目标集合不存在或索引不存在都会报错。`DROP INDEX` 不会删除集合中的记录。

## 示例

```sql
USE demo;

DROP INDEX idx_age ON users;
DROP INDEX IF EXISTS idx_name ON users;
```
