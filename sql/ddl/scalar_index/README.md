# 标量索引

LiteDB 的标量索引面向 `BOOLEAN`、整数、浮点数和字符串等非向量列。当前 SQL DDL 只创建持久化的 B+Tree 索引，并且一个索引只覆盖一个列。

创建或删除索引前，需要先 [USE](../database/use/README.md) 切换到目标数据库。索引定义属于集合元数据，索引文件由数据库引擎与记录存储一起维护。

## 支持的语句

| 语句 | 用途 |
| --- | --- |
| [CREATE INDEX](create_index/README.md) | 创建普通或唯一的 B+Tree 标量索引 |
| [DROP INDEX](drop_index/README.md) | 删除标量索引 |
| [SHOW INDEXES](show_indexes/README.md) | 查看集合上的标量索引定义 |

## 基本示例

```sql
CREATE INDEX idx_age ON users (age) USING BTREE;
CREATE UNIQUE INDEX idx_email ON users (email);

SHOW INDEXES FROM users;

SELECT id, name
FROM users
WHERE age BETWEEN 18 AND 30;

DROP INDEX idx_age ON users;
```

`USING BTREE` 可以省略，因为它是当前 `CREATE INDEX` 的默认方式。除 `BTREE` 外，SQL 层暂不接受其他标量索引方法。

## 与查询的关系

索引创建成功后，优化器会在符合形状的谓词上自动选择 B+Tree 访问路径。当前可识别的直接单列条件包括：

- `column = constant`；
- `column > constant`、`column >= constant`、`column < constant`、`column <= constant`；
- `column BETWEEN constant AND constant`；
- 常量在左侧的反向比较，例如 `18 <= age`。

常量可以是能够在规划阶段折叠的表达式。索引只负责产生候选记录，原始 `WHERE` 条件仍会保留并进行最终过滤；因此“创建了索引”不等于每一条查询都会使用索引。

没有匹配索引、谓词不是上述直接形状，或查询需要其他关系操作时，计划会回退到顺序扫描等路径。当前优化器不是基于统计信息的成本优化器。

## 重要边界

- SQL 只支持单列索引，不支持 `(a, b)` 形式的复合索引。
- `VECTOR(n)` 列不能创建标量索引；向量列应使用 [CREATE VINDEX](../vector_index/create_vindex/README.md)。
- `NULL` 不写入标量索引，所以等值和范围索引访问不会返回仅因索引键为 `NULL` 的记录；唯一索引也只约束非 `NULL` 值。
- `CREATE COLLECTION` 中的列级 `UNIQUE` 会自动生成隐式唯一 B+Tree 索引。该索引会出现在 `SHOW INDEXES` 中，但不能单独使用 `DROP INDEX` 删除。
- 标量索引名称在集合内按不区分大小写的方式查找，且不能与同集合的向量索引重名。

索引的键类型必须与目标列的运行时类型一致。当前可作为标量键的类型包括 `BOOLEAN`、`INTEGER`、`BIGINT`、`FLOAT`、`DOUBLE` 和 `VARCHAR(n)`；浮点 `NaN` 不能作为索引键。

## 相关架构

索引生命周期、B+Tree 后端和键约束见 [标量索引架构](../../../architecture_and_design/scalar_index/README.md)。
