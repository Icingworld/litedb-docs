# VECTOR(n)

`VECTOR(n)` 是 LiteDB 的一等向量类型，表示固定维度的数值向量。`n` 为维度，必须为正整数。

运行时每个元素以双精度浮点数（`double`）存储。

## 声明

```sql
embedding VECTOR(3)
embedding VECTOR(128)
```

绑定阶段会校验 `n > 0`；`VECTOR(0)` 或缺少维度参数均非法。

## 字面量

使用方括号书写向量字面量，元素为数值表达式：

```sql
[0.1, 0.2, 0.3]
[1, 2, 3]
```

绑定规则：

- 元素必须可判定为数值类型，否则报错（`Vector elements must be numeric`）。
- 元素会隐式转换为 `DOUBLE`。
- 字面量的逻辑类型为 `VECTOR(k)`，其中 `k` 为元素个数。

## 语义与约束

- 写入值的维度必须与列声明的 `n` 完全一致，否则报错（`VECTOR dimension mismatch`）。
- `NULL` 可赋给可空的 `VECTOR` 列；`NULL` 向量不会进入向量索引。
- 仅同维度的 `VECTOR` 之间可兼容；不能与标量类型互相转换。
- 向量索引键拒绝空向量，以及含有 `NaN` / `Infinity` 的非有限值。

## 距离函数

内置向量距离函数（两侧参数须为同维度 `VECTOR`）：

| 函数 | 含义 |
| --- | --- |
| `l2_distance(a, b)` | 欧氏距离（L2） |
| `cosine_distance(a, b)` | `1 -` 余弦相似度 |
| `inner_product(a, b)` | 内积（点积） |

典型 TopK 检索写法：

```sql
SELECT id
FROM users
ORDER BY l2_distance(embedding, [0.1, 0.2, 0.3]) ASC
LIMIT 5;
```

对常量查询向量的有限 TopK，优化器可自动选择 HNSW：`l2_distance` / `cosine_distance` 升序，或 `inner_product` 降序。

## 索引

- **不能**在 `VECTOR` 列上创建标量索引。
- 应使用 `CREATE VINDEX ... USING HNSW` 创建向量索引，以加速相似度检索。
- 索引参数见 [向量索引](../../../architecture_and_design/vector_index/README.md)。

```sql
CREATE VINDEX idx_embedding ON users (embedding) USING HNSW
WITH (metric = L2, max_neighbors = 16, ef_construction = 200, ef_search = 64);
```
