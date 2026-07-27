# 消息协议

LiteDB 网络协议中的所有通信均由消息（Message）组成。每条消息表示一个独立的通信单元，包含消息头（Header）以及消息载荷（Payload）。消息头用于描述消息类型、长度以及请求关联信息，消息载荷用于保存具体业务数据。

当前协议版本为 `1`（`ProtocolVersion = 1`）。整帧使用**大端字节序**编码。

## 消息结构

消息的帧结构如下图所示：

```txt
0                   1                   2                   3
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-------------------------------------------------------------+
|                       Payload Size                          |
+-------------------------------------------------------------+
|            Version          |             Kind              |
+-------------------------------------------------------------+
|                         Request ID                          |
|                          (8 bytes)                          |
+-------------------------------------------------------------+
|                        Payload Data                         |
+-------------------------------------------------------------+
```

| 字段 | 大小（字节） | 说明 |
| --- | --- | --- |
| Payload Size | 4 | **仅**表示 Payload Data 的字节数，**不包含** 16 字节消息头 |
| Version | 2 | 协议版本，当前固定为 `1` |
| Kind | 2 | 消息类型，见下文 |
| Request ID | 8 | 请求关联 ID；客户端分配，服务端原样回显 |
| Payload Data | 变长 | 消息载荷；长度为 Payload Size |

## 通用编码约定

载荷内部多处复用下列编码：

### 字符串

```txt
u32 length  +  length 字节原始内容（无 NUL 结尾）
```

`length` 为大端无符号 32 位整数，表示后续字节数。

### 逻辑类型

```txt
u8  type_id
u8  has_parameter   # 0 或 1
[if has_parameter != 0] u64 parameter
```

| `type_id` | 含义 |
| --- | --- |
| 0 | `NULL` |
| 1 | `BOOLEAN` |
| 2 | `INTEGER` |
| 3 | `BIGINT` |
| 4 | `FLOAT` |
| 5 | `DOUBLE` |
| 6 | `VARCHAR` |
| 7 | `VECTOR` |

`parameter` 用于 `VARCHAR(n)` 的最大长度或 `VECTOR(n)` 的维度。

### 值

先写 1 字节 tag（取值同 `LogicalTypeId`），再按类型写正文：

| tag | 后续字段 |
| --- | --- |
| 0 `NULL` | 无 |
| 1 `BOOLEAN` | `u8`（0 / 非 0） |
| 2 `INTEGER` | `u32`（`int32` 的位模式） |
| 3 `BIGINT` | `u64`（`int64` 的位模式） |
| 4 `FLOAT` | `u32`（IEEE 754 位模式） |
| 5 `DOUBLE` | `u64`（IEEE 754 位模式） |
| 6 `VARCHAR` | 字符串 |
| 7 `VECTOR` | `u32 count` + `count` 个 `f64`（每个为 `u64` 位模式） |

## 消息类型

| Kind 值 | 名称 | 方向 | Payload |
| --- | --- | --- | --- |
| 1 | `ExecuteSqlRequest` | 客户端 → 服务端 | SQL 文本 |
| 2 | `ExecuteSqlResponse` | 服务端 → 客户端 | 执行结果 |
| 3 | `ErrorResponse` | 服务端 → 客户端 | 错误码与消息 |
| 4 | `PingRequest` | 客户端 → 服务端 | 空 |
| 5 | `PongResponse` | 服务端 → 客户端 | 空 |

`Version` 必须为 `1`；`Kind` 必须落在 `1..5`，否则视为非法帧。

### ExecuteSqlRequest

```txt
string sql
```

### ExecuteSqlResponse

```txt
u8   result_kind
u64  affected_rows
u8   has_selected_database_name   # 0 或 1
[if has_selected_database_name != 0] string selected_database_name
u32  column_count
repeat column_count times:
    string column_name
    LogicalType column_type
u32  row_count
repeat row_count times:
    u32 value_count
    repeat value_count times:
        Value
```

`result_kind`：

| 值 | 含义 |
| --- | --- |
| 0 | `Command`（如 DDL / DML 命令结果） |
| 1 | `RowSet`（查询结果集） |
| 2 | `UseDatabase`（切换当前数据库） |

说明：

- 线上只传输 `selected_database_name`，不传输内部的 `selected_database_id`。
- 整份结果装入**一个**消息载荷；大结果集受单帧载荷上限约束。

### ErrorResponse

```txt
u16  code
string message
```

服务端在 SQL 执行失败时返回该消息；`code` 由会话错误码映射而来（实现中为底层错误码数值 `+ 1`）。协议层解码失败或未知请求类型等场景也可能返回 `ErrorResponse`（具体断连策略见会话相关文档）。

### PingRequest / PongResponse

载荷为空（`Payload Size = 0`）。用于连通性检测；服务端收到 `PingRequest` 后回复同 `Request ID` 的 `PongResponse`。

## Request ID

- 由客户端为每个请求分配，通常单调递增。
- 服务端响应（含错误响应与 Pong）原样回显该 ID。
- 客户端应用该字段将响应与请求对应；不匹配时应视为异常响应。

## 编码示例

一次 Ping：

```txt
Payload Size = 0
Version      = 1
Kind         = 4 (PingRequest)
Request ID   = 1
Payload      = （无）
```

整帧长度 = 16 字节。

一次 `ExecuteSqlRequest`（SQL 为 `"SELECT 1"`，假设 UTF-8 共 8 字节）：

```txt
Payload Size = 4 + 8 = 12   # 字符串 length 前缀 + 正文
Version      = 1
Kind         = 1
Request ID   = 2
Payload      = u32(8) + "SELECT 1"
```

整帧长度 = `16 + 12 = 28` 字节。

## 当前限制

- 帧结构缺少 Magic 字段，无法进行协议识别，缺少校验和，无法保证消息的完整性和正确性。
- 消息模型比较简陋，缺少连接管理、心跳检测、重试机制等。
