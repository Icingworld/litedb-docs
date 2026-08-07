# SHOW VINDEXES 语句

`SHOW VINDEXES` 用于查看当前数据库中指定集合的向量索引定义和 HNSW 参数。该语句必须明确指定集合，不能省略 `FROM`。

## 语法

```ebnf
show_vindexes_statement ::= SHOW VINDEXES FROM collection_name

collection_name ::= identifier
```

## 参数

| 参数 | 说明 |
| --- | --- |
| `FROM collection_name` | 要查看向量索引的集合；集合必须属于当前 `USE` 选中的数据库 |

标量索引不包含在本语句结果中；请使用 [SHOW INDEXES](../../scalar_index/show_indexes/README.md)。

## 返回结果

每个向量索引返回一行，列顺序和名称如下：

| 列名 | 类型 | 说明 |
| --- | --- | --- |
| `index_name` | `VARCHAR` | 向量索引名称 |
| `column_name` | `VARCHAR` | 被索引的 `VECTOR(n)` 列名称 |
| `type` | `VARCHAR` | 索引类型，当前为 `HNSW` |
| `metric` | `VARCHAR` | 距离度量：`L2`、`COSINE` 或 `INNER_PRODUCT` |
| `dimension` | `BIGINT` | 向量维度 `n` |
| `max_neighbors` | `BIGINT` | HNSW 最大邻居数 |
| `ef_construction` | `BIGINT` | 构建候选宽度 |
| `ef_search` | `BIGINT` | 默认搜索候选宽度 |
| `random_seed` | `BIGINT` | 随机种子 |

没有向量索引时，语句返回相同列结构但没有数据行。

## 示例

```sql
USE demo;

SHOW VINDEXES FROM documents;
```

可能得到如下结果：

| index_name | column_name | type | metric | dimension | max_neighbors | ef_construction | ef_search | random_seed |
| --- | --- | --- | --- | ---: | ---: | ---: | ---: | ---: |
| `vidx_embedding` | `embedding` | `HNSW` | `COSINE` | 3 | 24 | 240 | 80 | 7 |
