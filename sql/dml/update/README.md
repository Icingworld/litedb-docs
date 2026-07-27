# UPDATE 语句

UPDATE 语句用于更新集合中已有记录的列值。

执行前需要先 [USE](../../ddl/database/use/README.md) 切换到目标数据库。

## 语法

```ebnf
update_statement ::=
    UPDATE collection_name
    SET assignment { "," assignment }
    [ WHERE expression ]

collection_name ::= identifier

assignment ::= column_name "=" expression

column_name ::= identifier
```

## 参数

| 参数 | 说明 |
| --- | --- |
| `collection_name` | 目标集合名称 |
| `assignment` | 赋值项；同一语句中目标列不可重复 |
| `column_name` | 要更新的列名称 |
| `expression`（`SET`） | 新值表达式，可引用同行其他列，例如 `age = age + 1` |
| `WHERE expression` | 可选过滤条件，结果类型须为 `BOOLEAN` |

## 说明

- 省略 `WHERE` 时，更新集合中的**全部**记录。
- 赋值表达式的类型须与目标列兼容；将 `NULL` 赋给 `NOT NULL` 列会失败。
- `VARCHAR` 仍受最大长度约束，`VECTOR(n)` 维度须与列定义一致。
- 当前不支持 `LIMIT`、`ORDER BY` 或 `RETURNING`。
- 更新成功后会维护相关标量索引与向量索引。

## 示例

```sql
UPDATE users
SET age = age + 1
WHERE name = 'Ada';

UPDATE users
SET active = false, embedding = [0.2, 0.3, 0.4]
WHERE id = 1;

-- 无 WHERE：更新全部行
UPDATE users
SET active = true;
```
