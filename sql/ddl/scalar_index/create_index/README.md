# CREATE INDEX 语句

`CREATE INDEX` 用于在当前数据库的集合上创建单列 B+Tree 标量索引。

## 语法

```ebnf
create_index_statement ::=
    CREATE [ UNIQUE ] INDEX [ IF NOT EXISTS ] index_name
    ON collection_name ( column_name )
    [ USING BTREE ]

index_name ::= identifier
collection_name ::= identifier
column_name ::= identifier
```

`UNIQUE` 必须写在 `INDEX` 前面；`IF NOT EXISTS` 写在 `INDEX` 后、索引名之前。当前 SQL 语法只接受一个 `column_name`，不能写逗号分隔的多个列。

## 参数

| 参数 | 说明 |
| --- | --- |
| `UNIQUE` | 可选；创建唯一 B+Tree 索引，禁止相同的非 `NULL` 键对应多条记录 |
| `IF NOT EXISTS` | 可选；同集合中已有同名标量索引时返回空操作，不覆盖原定义 |
| `index_name` | 索引名称；同一集合内不能与已有标量或向量索引重名 |
| `collection_name` | 目标集合名称；必须属于当前 `USE` 选中的数据库 |
| `column_name` | 目标列；必须是目标集合中的非 `VECTOR` 列 |
| `USING BTREE` | 可选；当前唯一支持的标量索引方法，省略时默认也是 `BTREE` |

## 创建行为

创建时，数据库会把索引定义写入 Catalog，并扫描目标集合中的已有记录构建 B+Tree。因此，语句成功返回后，历史数据也已经包含在索引中。

后续 `INSERT`、`UPDATE` 和 `DELETE` 会由索引引擎同步维护索引键。`NULL` 值不会生成索引入口；普通索引允许多个相同的非 `NULL` 值，唯一索引则会在写入准备阶段拒绝重复键。

支持的标量键类型为 `BOOLEAN`、`INTEGER`、`BIGINT`、`FLOAT`、`DOUBLE` 和 `VARCHAR(n)`。索引键不执行跨物理类型的隐式合并，`VECTOR(n)` 列必须改用 [CREATE VINDEX](../../vector_index/create_vindex/README.md)。

如果已经存在同名索引：

- 不带 `IF NOT EXISTS` 时返回“索引已存在”错误；
- 带 `IF NOT EXISTS` 时不修改已有索引，命令结果的 `affected_rows` 为 `0`。

索引方法目前只有 `BTREE`。例如 `USING GIN`、`USING HASH` 或 `USING B_TREE` 会在解析阶段被拒绝。

## 唯一索引与列级 UNIQUE

显式唯一索引：

```sql
CREATE UNIQUE INDEX idx_email ON users (email) USING BTREE;
```

也可以在创建集合时声明列级 `UNIQUE`：

```sql
CREATE COLLECTION users (
    id BIGINT UNIQUE,
    email VARCHAR(128)
);
```

后者由 Catalog 自动生成隐式唯一 B+Tree 索引，并可通过 [SHOW INDEXES](../show_indexes/README.md) 查看。隐式索引不能单独删除；如需移除该约束，应调整集合定义（当前 SQL DDL 尚未提供修改列约束的语句）。

## 示例

```sql
USE demo;

CREATE INDEX idx_age ON users (age);
CREATE INDEX idx_name ON users (name) USING BTREE;
CREATE UNIQUE INDEX idx_email ON users (email);
CREATE INDEX IF NOT EXISTS idx_age ON users (age) USING BTREE;
```

创建后可以用 [SHOW INDEXES](../show_indexes/README.md) 检查定义，也可以在直接的等值、范围或 `BETWEEN` 过滤中让优化器自动选择索引访问路径。
