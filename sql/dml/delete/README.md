# DELETE 语句

DELETE 语句用于删除集合中的记录。

执行前需要先 [USE](../../ddl/database/use/README.md) 切换到目标数据库。

## 语法

```ebnf
delete_statement ::=
    DELETE FROM collection_name
    [ WHERE expression ]

collection_name ::= identifier
```

## 参数

| 参数 | 说明 |
| --- | --- |
| `collection_name` | 目标集合名称 |
| `WHERE expression` | 可选过滤条件，结果类型须为 `BOOLEAN` |

## 说明

- 省略 `WHERE` 时，删除集合中的**全部**记录。
- 当前不支持 `LIMIT`、`ORDER BY` 或 `RETURNING`。
- 删除成功后会同步维护相关标量索引与向量索引。

## 示例

```sql
DELETE FROM users WHERE age < 18;

DELETE FROM users WHERE id = 1;

-- 无 WHERE：删除全部行
DELETE FROM users;
```
