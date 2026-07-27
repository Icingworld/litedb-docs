# CREATE COLLECTION 语句

CREATE COLLECTION 语句用于创建一个新集合。在其他数据库中，该结构也被称为表（Table）。

创建集合前需要先 [USE](../../database/use/README.md) 切换到目标数据库。列类型说明见 [数据类型总览](../../../data_types/overview/README.md)。

## 语法

```ebnf
create_collection_statement ::=
    CREATE COLLECTION [ IF NOT EXISTS ] collection_name
    "(" column_definition { "," column_definition } ")"
    [ COMMENT string_literal ]

collection_name ::= identifier

column_definition ::= column_name data_type { column_constraint }

column_name ::= identifier

column_constraint ::= UNIQUE
                   | DEFAULT literal
                   | NOT NULL
                   | NULL
                   | COMMENT string_literal
```

## 参数

| 参数 | 说明 |
| --- | --- |
| `IF NOT EXISTS` | 集合已存在时不报错、也不覆盖；不存在时正常创建 |
| `collection_name` | 集合名称 |
| `column_definition` | 列定义；至少需要一列，列名在同一集合内不可重复 |
| `column_name` | 列名称 |
| `data_type` | 数据类型，见 [数据类型总览](../../../data_types/overview/README.md) |
| `column_constraint` | 列约束，可写多个 |
| `COMMENT` | 注释；列级写在列定义内，集合级写在右括号之后 |

## 说明

- 每个集合至少包含一列。
- 列默认可空（`NULL`）；`NOT NULL` 表示禁止插入或更新为 `NULL`。`NULL` 与 `NOT NULL` 语义互斥，不应同时声明。
- `DEFAULT` 后跟字面量（含向量字面量，如 `[0.1, 0.2, 0.3]`），类型须与列类型兼容。
- `UNIQUE` 会为该列创建隐式唯一索引。`VECTOR(n)` 列不能声明 `UNIQUE`。
- 列级 `COMMENT` 写在字段定义内；集合级 `COMMENT` 写在列列表右括号之后。

## 示例

```sql
USE demo;

CREATE COLLECTION users (
    id BIGINT NOT NULL,
    name VARCHAR(64) NOT NULL COMMENT 'display name',
    age INTEGER NULL DEFAULT 0,
    active BOOLEAN DEFAULT true,
    embedding VECTOR(3)
) COMMENT 'user collection';

CREATE COLLECTION IF NOT EXISTS users (
    id BIGINT NOT NULL,
    name VARCHAR(64) UNIQUE,
    age INTEGER DEFAULT 18
) COMMENT 'user collection';
```
