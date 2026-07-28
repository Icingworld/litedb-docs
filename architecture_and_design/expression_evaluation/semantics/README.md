# 求值语义

表达式语义由 Binder 的静态约束和 Evaluator 的运行时行为共同构成。Binder 决定节点是否合法、结果类型是什么以及是否插入转换；Evaluator 使用具体 `Value` 计算最终结果。

## 数值类型提升

数值类型具有固定等级：

```text
INTEGER < BIGINT < FLOAT < DOUBLE
```

二元算术的结果类型取左右操作数中等级较高的类型。Binder 在较低等级的操作数外插入 `BoundCastExpression`，Evaluator 随后以统一类型执行运算。

例如：

```text
INTEGER + DOUBLE
        ↓ Binder
CAST(INTEGER AS DOUBLE) + DOUBLE
        ↓ Evaluator
DOUBLE
```

当前隐式数值转换只允许向更高等级提升，不允许自动缩窄。`NULL` 可以转换为任意目标类型，但运行时仍保持 `NULL`。

## 算术运算

当前支持：

- 二元 `+`、`-`、`*`、`/`、`%`；
- 一元 `+`、`-`。

任一操作数为 `NULL` 时，算术结果为 `NULL`。

整数结果使用整数运算，整数除法会舍弃小数部分；浮点取模使用浮点余数运算。除法或取模的右操作数为零时返回 `DivisionByZero`。

当前整数算术没有独立的溢出错误语义。文档不能声称整数溢出会被检测或自动提升。

## 比较运算

支持以下比较操作：

```text
=    !=    <    <=    >    >=
```

比较兼容性在 Binder 中检查：

| 类型组合 | 等于/不等于 | 顺序比较 |
| --- | --- | --- |
| 数值与数值 | 支持，先提升到公共类型 | 支持 |
| VARCHAR 与 VARCHAR | 支持 | 支持 |
| BOOLEAN 与 BOOLEAN | 支持 | Binder 当前允许，但 Evaluator 尚不能执行顺序比较 |
| VECTOR 与相同 VECTOR 类型 | 支持等于/不等于 | 不支持 |
| 不兼容类型 | 不支持 | 不支持 |

数值比较允许不同数值类型。字符串顺序比较使用当前字符串值的字节序关系，没有独立 Collation、区域规则或大小写折叠。

任一比较操作数为 `NULL` 时，结果为 `NULL`，不是 `true` 或 `false`。

BOOLEAN 顺序比较目前存在绑定与运行时语义不一致：Binder 会接受它，Evaluator 只实现数值和字符串的 `<`、`<=`、`>`、`>=`，执行时会返回 `InvalidType`。在实现统一前，不应把 BOOLEAN 顺序比较视为已支持功能。

## NULL 与三值逻辑

逻辑表达式可能产生 `TRUE`、`FALSE` 或 `NULL`。

### AND

| AND | TRUE | FALSE | NULL |
| --- | --- | --- | --- |
| TRUE | TRUE | FALSE | NULL |
| FALSE | FALSE | FALSE | FALSE |
| NULL | NULL | FALSE | NULL |

### OR

| OR | TRUE | FALSE | NULL |
| --- | --- | --- | --- |
| TRUE | TRUE | TRUE | TRUE |
| FALSE | TRUE | FALSE | NULL |
| NULL | TRUE | NULL | NULL |

### NOT

| 输入 | 结果 |
| --- | --- |
| TRUE | FALSE |
| FALSE | TRUE |
| NULL | NULL |

Evaluator 当前先求值 `AND`、`OR` 的两侧，再应用上述真值表。因此语义结果符合三值逻辑，但执行过程不提供短路保证。

## 谓词语义

`evaluate_predicate` 将表达式结果转换为执行器使用的行匹配结果：

| 表达式结果 | 是否匹配 |
| --- | --- |
| `TRUE` | 是 |
| `FALSE` | 否 |
| `NULL` | 否 |
| 非布尔值 | 返回 `InvalidType` |

因此，WHERE 条件计算得到 `NULL` 时，该行不会进入后续结果。

## IN

`target IN (value1, value2, ...)` 的规则为：

1. `target` 为 `NULL`，结果为 `NULL`；
2. 任一非空候选值与目标相等，结果为 `TRUE`；
3. 没有匹配值，但候选列表中存在 `NULL`，结果为 `NULL`；
4. 否则结果为 `FALSE`。

示例：

| 表达式 | 结果 |
| --- | --- |
| `2 IN (1, 2, NULL)` | TRUE |
| `3 IN (1, 2)` | FALSE |
| `3 IN (1, 2, NULL)` | NULL |
| `NULL IN (1, 2)` | NULL |

Binder 要求每个候选值都能与目标执行等值比较。

## BETWEEN

当前：

```sql
value BETWEEN lower AND upper
```

按以下语义计算：

```text
value >= lower AND value <= upper
```

上下界都包含在范围内。任一比较产生 `NULL` 时，最终结果按照三值 `AND` 组合。

## LIKE

`LIKE` 只接受两个 `VARCHAR` 表达式。

当前模式规则：

- `%` 匹配零个或多个字符；
- `_` 匹配一个字符；
- 其他字符按原值匹配。

任一操作数为 `NULL` 时，结果为 `NULL`。当前实现没有独立的 `ESCAPE` 字符、Collation 或大小写不敏感模式。

匹配通过动态规划实现，其内存和时间开销与目标字符串长度和模式长度的乘积相关。

## 类型转换

Binder 的 `can_cast` 控制当前允许进入表达式树的转换：

| 源类型 | 目标类型 | 是否允许 |
| --- | --- | --- |
| `NULL` | 任意类型 | 是 |
| 相同完整类型 | 相同类型 | 是 |
| 较低等级数值 | 较高等级数值 | 是 |
| 较高等级数值 | 较低等级数值 | 否，不能隐式缩窄 |
| `VARCHAR(n)` | `VARCHAR(m)` | 是 |
| `VECTOR(n)` | `VECTOR(n)` | 是 |
| 已知不同维度的 VECTOR | 另一 VECTOR | 否 |
| 其他类型组合 | 另一类型 | 否 |

Evaluator 的运行时转换能力还包括数值之间的具体表示转换、基础标量到字符串的格式化，以及相同运行时 VECTOR 的传递。不过正常 SQL 绑定路径只会生成 Binder 允许的转换节点，不能据此宣称 SQL 支持任意显式 CAST。

`NULL` 转换后仍是 `NULL`。运行时值与目标类型不兼容时返回 `CastFailed`。

## 向量表达式

向量表达式由一组数值子表达式组成：

```sql
[1, 2.5, 3]
```

Binder 执行：

1. 检查每个元素是数值类型；
2. 将元素转换为 `DOUBLE`；
3. 将元素数量记录为 `VECTOR(n)` 的维度。

Evaluator 逐个计算元素并生成 `std::vector<double>`。任一元素为 `NULL` 时，整个向量结果为 `NULL`。

向量距离等操作不是内建二元操作符，而是通过标量函数完成。函数绑定和维度检查参见[函数系统](../../function_system/README.md)。

## 函数的 NULL 语义

Evaluator 不统一替函数处理 `NULL`。它会正常计算全部参数，并把 `Value` 列表交给已经绑定的 `ScalarFunction`。

因此：

- 函数是否接受 `NULL`；
- `NULL` 是否传播；
- 参数值是否还有额外约束；

都由具体函数实现及其签名决定。表达式层只负责传播函数返回值或函数错误。

## 错误与 NULL 的区别

`NULL` 是合法的 SQL 值，错误则会终止当前表达式和执行路径。

| 情况 | 结果 |
| --- | --- |
| `NULL + 1` | `NULL` |
| `NULL = 1` | `NULL` |
| `1 / 0` | `DivisionByZero` |
| 无效列 ID | `InvalidColumnReference` |
| 非布尔值作为谓词 | `InvalidType` |
| 不支持的运行时转换 | `CastFailed` |

执行器不会把求值错误静默替换为 `NULL`，也不会仅跳过发生错误的行。

## 当前语义边界

- 没有 `IS NULL`、`IS DISTINCT FROM` 等独立求值节点。
- 没有 CASE、子查询表达式或聚合表达式求值。
- 没有短路 `AND`/`OR`。
- 没有字符串 Collation 和 LIKE ESCAPE。
- 没有整数溢出检测语义。
- 没有显式的用户级 CAST 语法节点来源；当前 Cast 主要由 Binder 插入。
- 没有批量表达式或向量化执行语义。
