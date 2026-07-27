# 数据类型

## 通用数据类型

LiteDB 支持的数据类型如下表所示：

| 数据类型 | 说明 |
| --- | --- |
| `INTEGER` | 32 位整数 |
| `BIGINT` | 64 位整数 |
| `FLOAT` | 单精度浮点数 |
| `DOUBLE` | 双精度浮点数 |
| `VARCHAR(n)` | 字符串，`n` 为最大长度 |
| `BOOLEAN` | 布尔值 |
| `VECTOR(n)` | 向量，`n` 为维度 |

## 数据类型特殊说明

部分数据类型拥有特殊的表现形式、特性和使用方式，以下分小节详细介绍：

- [VARCHAR(n)](../varchar/README.md)
- [BOOLEAN](../boolean/README.md)
- [VECTOR(n)](../vector/README.md)
