# CREATE VINDEX 语句

`CREATE VINDEX` 用于在当前数据库的集合上创建单列 HNSW 向量索引。目标列必须是 `VECTOR(n)`。

## 语法

```ebnf
create_vindex_statement ::=
    CREATE VINDEX [ IF NOT EXISTS ] index_name
    ON collection_name ( column_name )
    USING HNSW
    [ WITH ( vector_index_option { "," vector_index_option } ) ]

vector_index_option ::= metric_option
                      | max_neighbors_option
                      | ef_construction_option
                      | ef_search_option
                      | random_seed_option

metric_option ::= metric = ( L2 | COSINE | INNER_PRODUCT )
max_neighbors_option ::= max_neighbors = integer_literal
ef_construction_option ::= ef_construction = integer_literal
ef_search_option ::= ef_search = integer_literal
random_seed_option ::= random_seed = integer_literal

index_name ::= identifier
collection_name ::= identifier
column_name ::= identifier
```

`WITH` 内的选项可以按任意顺序出现，但每个选项最多出现一次；空的 `WITH ()`、未知选项和重复选项都会被拒绝。`USING HNSW` 是必需的，当前不接受其他方法。

## 参数与默认值

| 参数 | 默认值 | 说明 |
| --- | ---: | --- |
| `IF NOT EXISTS` | — | 同集合中已有同名向量索引时返回空操作，不覆盖原定义 |
| `metric` | `L2` | 向量距离度量，可选 `L2`、`COSINE` 或 `INNER_PRODUCT` |
| `max_neighbors` | `16` | HNSW 每层的最大邻居数，当前运行时范围为 `1..2^20` |
| `ef_construction` | `200` | 构建图时的候选宽度，必须满足 `max_neighbors <= ef_construction <= 2^24` |
| `ef_search` | `64` | 查询时默认的候选宽度，当前运行时范围为 `1..2^24` |
| `random_seed` | `0` | 节点层级生成所用的随机种子 |

选项名和度量名按不区分大小写的方式解析，例如 `metric = cosine` 与 `metric = COSINE` 等价。所有参数值必须是非负整数字面量。目标列的维度 `n` 由 `VECTOR(n)` 声明给出，HNSW 当前要求 `1 <= n <= 2^20`；超出这些范围的定义会在绑定或运行时被拒绝。

## 创建行为

创建时，向量索引引擎会根据目标列的 `VECTOR(n)` 声明记录维度，并扫描集合中的已有记录构建 HNSW。非 `NULL` 向量会成为索引节点；`NULL` 不进入索引。创建成功后，后续 `INSERT`、`UPDATE` 和 `DELETE` 会同步维护 HNSW。

向量索引不是唯一索引，同一向量或相同距离都不会因为索引本身而被拒绝。向量索引名称不能与同集合的标量索引重名。

如果已经存在同名向量索引：

- 不带 `IF NOT EXISTS` 时返回“向量索引已存在”错误；
- 带 `IF NOT EXISTS` 时不修改已有索引，命令结果的 `affected_rows` 为 `0`。

创建成功的命令结果 `affected_rows` 为 `1`。索引文件和 Catalog 定义由数据库引擎在 DDL 事务中协调持久化。

## 距离度量与 SELECT

`metric` 必须与查询中使用的距离函数和排序方向匹配：

| 索引度量 | Top-K 查询形式 |
| --- | --- |
| `L2` | `ORDER BY l2_distance(column, query) ASC` |
| `COSINE` | `ORDER BY cosine_distance(column, query) ASC` |
| `INNER_PRODUCT` | `ORDER BY inner_product(column, query) DESC` |

查询向量必须是规划阶段可以求值的常量，并且与目标列维度一致。更多查询语法见 [SELECT 的向量相似度检索](../../../dql/select/README.md)。

## 示例

```sql
USE demo;

CREATE VINDEX vidx_l2
ON documents (embedding)
USING HNSW;

CREATE VINDEX vidx_cosine
ON documents (embedding)
USING HNSW
WITH (metric = COSINE, max_neighbors = 24, ef_construction = 240, ef_search = 80);

CREATE VINDEX IF NOT EXISTS vidx_inner
ON documents (embedding)
USING HNSW
WITH (metric = INNER_PRODUCT, random_seed = 7);
```
