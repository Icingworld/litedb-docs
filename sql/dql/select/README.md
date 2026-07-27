# SELECT 语句

SELECT 语句用于查询集合中的记录，支持投影、过滤、排序与分页。

执行前需要先 [USE](../../ddl/database/use/README.md) 切换到目标数据库。表达式细节见后续「表达式」章节；向量类型与距离函数见 [VECTOR(n)](../../data_types/vector/README.md)。

## 语法

```ebnf
select_statement ::=
    SELECT select_item { "," select_item }
    FROM collection_name
    [ WHERE expression ]
    [ ORDER BY order_item { "," order_item } ]
    [ LIMIT integer_literal ]
    [ OFFSET integer_literal ]

collection_name ::= identifier

select_item ::= "*"
              | qualifier "." "*"
              | expression [ AS alias ]

qualifier ::= identifier

alias ::= identifier

order_item ::= expression [ ASC | DESC ]
```

当前只支持单集合查询：`FROM` 后只能跟一个集合名。

## 参数

| 参数 | 说明 |
| --- | --- |
| `select_item` | 投影项：通配符、列引用、表达式；表达式可用 `AS` 起别名 |
| `collection_name` | 源集合名称 |
| `WHERE expression` | 可选过滤条件，结果类型须为 `BOOLEAN` |
| `ORDER BY` | 可选排序；可写多个排序项，默认 `ASC` |
| `LIMIT` | 可选；返回行数上限，须为非负整数字面量 |
| `OFFSET` | 可选；跳过前若干行，须为非负整数字面量 |

## SELECT 列表

- `*` 展开为集合全部列；`collection.*` 为带限定符的通配符。通配符**不能**加 `AS` 别名。
- 可投影列引用（`id`、`users.name`）、字面量、算术 / 比较结果、函数调用、向量字面量等。
- 表达式别名必须写显式 `AS`（不支持 `SELECT age + 1 next_age` 这种隐式别名）。
- 未命名的表达式列，结果集中会显示为自动生成的列名，例如 `expr1`、`expr3`。需要稳定列名时请使用 `AS`。

```sql
SELECT * FROM users;
SELECT id, users.name, users.* FROM users;
SELECT age + 1 AS next_age FROM users;
SELECT l2_distance(embedding, [0.1, 0.2, 0.3]) AS dist FROM users;
```

## WHERE

- 条件表达式必须返回 `BOOLEAN`。
- 支持比较、`AND` / `OR` / `NOT`、`LIKE`、`IN`、`BETWEEN` 等（详见表达式章节）。
- 也可直接使用布尔列：`WHERE active`。

```sql
SELECT id, name
FROM users
WHERE age >= 18 AND active = true;

SELECT * FROM users WHERE name LIKE 'A%';
SELECT * FROM users WHERE age BETWEEN 18 AND 30;
SELECT * FROM users WHERE category IN ('book', 'tool');
```

若列上存在 B+Tree 标量索引，优化器可对部分等值 / 范围谓词（如 `=`、`>`、`>=`、`<`、`<=`、`BETWEEN`）选择索引访问路径。

## ORDER BY

- 可按表达式排序，也可引用 `SELECT` 列表中的显式投影别名。
- 同一别名在投影中出现多次时，`ORDER BY` 引用该别名会因歧义失败。
- 默认升序 `ASC`；可写 `DESC`。
- 可写多个排序键，按从左到右优先级排序。

```sql
SELECT age + 1 AS next_age
FROM users
ORDER BY next_age DESC;

SELECT id, name
FROM users
ORDER BY age DESC, name ASC;
```

## LIMIT 与 OFFSET

- `LIMIT n`：最多返回 `n` 行。
- `OFFSET m`：跳过前 `m` 行后再取结果。
- 两者都只接受非负整数字面量，不能是表达式。
- 可单独使用，也可组合：先 `OFFSET`，再截取 `LIMIT`。

```sql
SELECT id, name
FROM users
ORDER BY age DESC
LIMIT 10 OFFSET 20;
```

## 向量相似度检索（TopK）

对带向量索引的集合，下列模式可自动走 HNSW 近似检索：

1. `ORDER BY` **恰好一项**，且为距离函数；
2. 查询向量为**常量**向量字面量（元素可折叠为常量）；
3. 存在正的 `LIMIT`（有限 TopK）；
4. 排序方向与函数匹配：
   - `l2_distance(...) ASC`
   - `cosine_distance(...) ASC`
   - `inner_product(...) DESC`

可同时带 `WHERE`、`OFFSET`。ANN 负责产生候选；最终距离由外层流水线重新计算并精确排序。候选不足时会扩大搜索，必要时回退顺序扫描。

```sql
SELECT id, name
FROM users
ORDER BY l2_distance(embedding, [0.1, 0.2, 0.3]) ASC
LIMIT 5;

SELECT id, cosine_distance(embedding, [0.1, 0.2, 0.3]) AS dist
FROM users
ORDER BY dist ASC
LIMIT 5;

SELECT id
FROM users
WHERE active = true
ORDER BY inner_product(embedding, [0.1, 0.2, 0.3]) DESC
LIMIT 10 OFFSET 5;
```

距离函数的两个参数必须是相同维度的 `VECTOR(n)`。更多说明见 [VECTOR(n)](../../data_types/vector/README.md)。

## 当前限制

- 不支持 `JOIN`、子查询、聚合、`GROUP BY`、`HAVING`、`DISTINCT`、`UNION` 等。
- 不支持 `SELECT` 无 `FROM`、多集合 `FROM`。
- 尚无 `EXPLAIN`。
- HNSW 结果为近似检索；暂无查询级 hint 或单次查询覆盖 `ef_search`。
- 优化器基于规则，尚无基于统计信息的代价比较。

## 示例

```sql
-- 基础查询
SELECT id, name, age
FROM users
WHERE age >= 18
ORDER BY age DESC
LIMIT 10;

-- 表达式投影与别名排序
SELECT id, age + 1 AS next_age
FROM users
ORDER BY next_age DESC;

-- 向量 TopK
SELECT id, name, l2_distance(embedding, [0.1, 0.2, 0.3]) AS dist
FROM users
ORDER BY dist ASC
LIMIT 5;
```
