# VARCHAR(n)

`VARCHAR(n)` 表示长度受限的字符串类型。`n` 是最大长度（按存储字节数计），必须为正整数。

## 声明

```sql
name VARCHAR(64)
```

绑定阶段会校验 `n > 0`；`VARCHAR(0)` 或缺少长度参数均非法。

## 字面量

字符串字面量可用单引号或双引号包裹：

```sql
'Ada'
"Linus"
```

字面量在表达式中的逻辑类型为 `VARCHAR`（无固定长度参数）。写入列时会按目标列的 `n` 做长度校验。

## 语义与约束

- 写入值的字节长度不得超过列声明的 `n`，否则报错（例如 `VARCHAR value exceeds declared length`）。
- `NULL` 可赋给可空的 `VARCHAR` 列。
- 不同长度的 `VARCHAR` 之间允许隐式转换；最终仍以目标列的 `n` 为准做长度检查。
- 不能与数值、布尔或向量类型互相转换。

## 常用运算

- 比较：`=`、`<>`、`<`、`<=`、`>`、`>=`
- 模式匹配：`LIKE`（两侧操作数均须为 `VARCHAR`）

```sql
SELECT id, name
FROM users
WHERE name LIKE 'A%'
ORDER BY name ASC;
```
