# 向量索引

LiteDB 的向量索引只能建立在 `VECTOR(n)` 列上，用于加速固定维度向量的 Top-K 相似度检索。当前 SQL DDL 只创建持久化的 HNSW 索引，一个索引只覆盖一个向量列。

创建或删除索引前，需要先 [USE](../database/use/README.md) 切换到目标数据库。向量列和距离函数的基础语义见 [VECTOR(n)](../../data_types/vector/README.md)。

## 支持的语句

| 语句 | 用途 |
| --- | --- |
| [CREATE VINDEX](create_vindex/README.md) | 创建 HNSW 向量索引 |
| [DROP VINDEX](drop_vindex/README.md) | 删除 HNSW 向量索引 |
| [SHOW VINDEXES](show_vindexes/README.md) | 查看集合上的向量索引及其参数 |

LiteDB 使用 `VINDEX` 作为向量索引的 SQL 关键字；当前没有 `CREATE VECTOR INDEX` 这一写法。内部的 Flat 精确扫描后端不对应 SQL DDL 创建的索引类型。

## 基本示例

```sql
CREATE VINDEX vidx_embedding
ON documents (embedding)
USING HNSW
WITH (
    metric = COSINE,
    max_neighbors = 16,
    ef_construction = 200,
    ef_search = 64,
    random_seed = 7
);

SHOW VINDEXES FROM documents;

SELECT id
FROM documents
ORDER BY cosine_distance(embedding, [0.1, 0.2, 0.3]) ASC
LIMIT 10;

DROP VINDEX vidx_embedding ON documents;
```

## 与查询的关系

优化器会在查询形状、距离函数、向量列、索引度量和 Top-K 条件都匹配时自动选择 HNSW。典型形式是：

- `l2_distance(vector_column, constant_vector) ASC`；
- `cosine_distance(vector_column, constant_vector) ASC`；
- `inner_product(vector_column, constant_vector) DESC`；
- 只有一个 `ORDER BY` 项，并且存在正数 `LIMIT`。

可以同时使用 `WHERE` 和 `OFFSET`。HNSW 先产生候选记录，外层流水线仍会执行过滤、距离求值和最终排序；没有可用索引、查询向量不是常量或计划形状不匹配时，会回退到普通扫描与排序。

HNSW 是近似最近邻索引，不提供固定召回率承诺。当前 SQL 没有查询级 hint，也不能在单次 `SELECT` 中覆盖索引的 `ef_search`。

## 重要边界

- 目标列必须是维度大于零的 `VECTOR(n)`；标量列不能创建向量索引。
- SQL 只支持单列 HNSW，不支持复合向量索引。
- 向量索引没有 `UNIQUE` 选项；唯一性属于标量索引或列约束。
- `NULL` 向量不会进入索引；写入向量的维度必须与列声明一致，元素还必须是有限数值。
- 同一集合内索引名称不区分大小写，并且标量索引和向量索引共享名称空间。

向量索引引擎、HNSW 文件和恢复行为见 [向量索引架构](../../../architecture_and_design/vector_index/README.md)。
