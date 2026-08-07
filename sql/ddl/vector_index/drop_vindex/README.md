# DROP VINDEX 语句

`DROP VINDEX` 用于删除当前数据库中指定集合上的向量索引。

## 语法

```ebnf
drop_vindex_statement ::= DROP VINDEX [ IF EXISTS ] index_name ON collection_name

index_name ::= identifier
collection_name ::= identifier
```

向量索引使用 `DROP VINDEX`，不能用标量索引的 `DROP INDEX` 代替。

## 参数

| 参数 | 说明 |
| --- | --- |
| `IF EXISTS` | 可选；向量索引不存在时返回空操作，不报错 |
| `index_name` | 要删除的向量索引名称 |
| `collection_name` | 索引所属集合；必须属于当前 `USE` 选中的数据库 |

## 行为

删除成功后，向量索引定义从 Catalog 移除，运行时 HNSW 也不再参与 Top-K 查询或后续写入维护。正常删除命令的 `affected_rows` 为 `1`；使用 `IF EXISTS` 删除不存在的索引时为 `0`。

删除向量索引不会删除集合记录。删除后，满足查询语义但没有匹配 HNSW 的 `SELECT` 会回退到普通扫描和排序。

不带 `IF EXISTS` 时，目标集合不存在或向量索引不存在都会报错。

## 示例

```sql
USE demo;

DROP VINDEX vidx_embedding ON documents;
DROP VINDEX IF EXISTS vidx_embedding ON documents;
```
