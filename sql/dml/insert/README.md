# INSERT 语句

INSERT 语句用于向集合插入一条记录。

执行前需要先 [USE](../../ddl/database/use/README.md) 切换到目标数据库。类型与字面量说明见 [数据类型总览](../../data_types/overview/README.md)。

## 语法

```ebnf
insert_statement ::=
    INSERT INTO collection_name
    [ "(" column_name { "," column_name } ")" ]
    VALUES "(" expression { "," expression } ")"

collection_name ::= identifier

column_name ::= identifier
```

当前每条 `INSERT` 只支持插入一行（一组 `VALUES`）。

## 参数

| 参数 | 说明 |
| --- | --- |
| `collection_name` | 目标集合名称 |
| `column_name` | 可选列列表；指定时，列数须与 `VALUES` 中表达式个数一致，且列名不可重复 |
| `expression` | 插入值表达式（字面量或可求值的表达式） |

## 说明

- 省略列列表时，`VALUES` 按集合字段定义顺序提供全部列的值，个数须与列数一致。
- 指定列列表时，未出现的列按以下顺序补齐：有 `DEFAULT` 则用默认值；否则若列可空则填 `NULL`；若列是 `NOT NULL` 且无默认值则报错。
- 值类型须与目标列兼容（支持数值拓宽等隐式转换）。`VARCHAR` 受长度限制，`VECTOR(n)` 维度须一致。
- 显式写入 `NULL` 到 `NOT NULL` 列会失败。
- 声明了 `UNIQUE` 的列若插入重复值会失败。
- 插入成功后会维护相关标量索引与向量索引。

## 示例

```sql
INSERT INTO users (id, name, age, active, embedding)
VALUES (1, 'Ada', 36, true, [0.1, 0.2, 0.3]);

-- 省略列列表：按集合列顺序提供全部值
INSERT INTO users
VALUES (2, 'Linus', 55, true, [0.2, 0.3, 0.4]);

-- 省略部分列：使用 DEFAULT 或 NULL
INSERT INTO users (id, name, embedding)
VALUES (3, 'Grace', [0.3, 0.4, 0.5]);
```
