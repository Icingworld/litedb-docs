# BOOLEAN

`BOOLEAN` 表示布尔类型，取值为真或假。

## 声明

```sql
active BOOLEAN
active BOOLEAN DEFAULT true
```

## 字面量

使用关键字 `TRUE` / `FALSE`（大小写不敏感）：

```sql
TRUE
FALSE
true
false
```

也可在 `DEFAULT` 中使用：

```sql
active BOOLEAN DEFAULT true
```

## 语义与约束

- `NULL` 可赋给可空的 `BOOLEAN` 列。
- 不与数值、字符串或向量类型互相转换；仅同类型赋值，或由 `NULL` 赋入。
- `WHERE`、比较表达式等条件结果类型为 `BOOLEAN`。

## 常用运算

支持等值与序比较：

```sql
SELECT id, name
FROM users
WHERE active = true;

SELECT id, name
FROM users
WHERE active <> false;
```

也可直接将布尔列用作过滤条件：

```sql
SELECT id, name
FROM users
WHERE active;
```
