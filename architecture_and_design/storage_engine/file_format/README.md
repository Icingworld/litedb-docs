# 存储文件格式

每个集合的 `.store` 文件采用固定 4096 字节块：

```text
+-------------------------------+
| File Header Page (4096 bytes) |
+-------------------------------+
| Data Page 0 (4096 bytes)      |
+-------------------------------+
| Data Page 1 (4096 bytes)      |
+-------------------------------+
| ...                           |
+-------------------------------+
```

数据页 `page_id` 从 0 开始，其文件偏移为：

```text
(page_id + 1) * 4096
```

当前格式版本为 2。

## 文件头页

第一个 4096 字节块是文件头：

| 偏移 | 大小 | 字段 |
| ---: | ---: | --- |
| 0 | 4 | Magic `LDS2` |
| 4 | 2 | Format Version |
| 6 | 2 | Page Size |
| 8 | 4 | Header Size |
| 12 | 4 | Flags，当前必须为 0 |
| 16 | 8 | CollectionId |
| 24 | 8 | Next RecordId |
| 32 | 4 | Data Page Count |
| 36 | 4 | Header CRC32 |
| 40 | 4056 | Reserved，必须全部为 0 |

数值均以小端序编码。计算 CRC32 时，Checksum 字段先置零，再覆盖整个 4096 字节头页。

打开文件时验证：

- Magic；
- 格式版本；
- Page Size 和 Header Size；
- Flags；
- CRC32；
- Reserved 区域；
- CollectionId 与文件预期集合一致；
- Next RecordId 非零；
- Page Count 与实际文件长度一致。

旧版本会返回 `UnsupportedVersion`，而不是按当前布局尝试读取。

## 数据页头

每个数据页前 24 字节为：

| 偏移 | 大小 | 字段 |
| ---: | ---: | --- |
| 0 | 4 | Magic `LPG2` |
| 4 | 4 | PageId |
| 8 | 2 | Slot Count |
| 10 | 2 | Free Start |
| 12 | 2 | Free End |
| 14 | 2 | Flags，当前必须为 0 |
| 16 | 4 | Page CRC32 |
| 20 | 4 | Generation |

CRC32 同样覆盖完整 4096 字节页面，并在计算前把校验字段清零。

`generation` 在页面内容变化时递增。目前它用于记录页面版本变化，但没有 Buffer Pool 或并发校验逻辑消费它。

## 数据页结构校验

解码数据页时检查：

- Page Magic 和 PageId；
- Flags 为零；
- CRC32；
- `free_start` 等于页头加槽目录长度；
- `free_start <= free_end <= PageSize`；
- 每个槽的状态有效；
- 槽 Reserved 字节为零；
- Active 槽具有非空 Payload；
- Payload 位于合法数据区域；
- 不同 Payload 范围不重叠。

这些检查在访问记录内容前完成。通过页面结构校验后，活动槽中的记录 Payload 还要继续执行记录和值解码检查。

## 记录编码

每条记录 Payload 为：

```text
+--------------------+
| RecordId : u64     |
+--------------------+
| Value Count : u32  |
+--------------------+
| Value 0            |
+--------------------+
| Value 1            |
+--------------------+
| ...                |
+--------------------+
```

解码要求：

- RecordId 不为 0；
- Value Count 不超过剩余数据的基本边界；
- 每个 Value 都能完整解码；
- Payload 末尾没有多余字节；
- 总长度不超过单页记录上限。

记录 Payload 自身没有独立 CRC；它由所在数据页的整页 CRC32 保护。

## Value 编码

每个 Value 先写入 1 字节类型标签：

| 标签 | 类型 | 后续内容 |
| ---: | --- | --- |
| 0 | NULL | 无 |
| 1 | BOOLEAN | u8，只允许 0 或 1 |
| 2 | INTEGER | i32 |
| 3 | BIGINT | i64 |
| 4 | FLOAT | IEEE 754 f32 |
| 5 | DOUBLE | IEEE 754 f64 |
| 6 | VARCHAR | u32 字节长度 + 原始字节 |
| 7 | VECTOR | u32 元素数量 + 连续 f64 |

字符串和向量长度均在分配内存前检查。向量元素统一以 double 持久化。

IO 层的小端编码和资源预算参见[IO](../../file_system_and_io/io/README.md)。

## 文件打开与目录重建

StorageStore 打开已有文件时：

1. 检查文件长度至少为一个头页；
2. 检查头页之后是完整数据页倍数；
3. 读取并验证文件头；
4. 检查声明 Page Count；
5. 逐页读取并验证数据页；
6. 解码每个 Active 槽；
7. 重建 `RecordId -> PhysicalRid`；
8. 拒绝重复 RecordId；
9. 重建页面空间信息；
10. 验证 `next_record_id` 大于文件中最大 RecordId。

因此打开成本与数据页数量和活动记录数量线性相关。位置目录和空间索引没有单独持久化。

## 损坏分类

典型 Storage 错误包括：

| 错误 | 含义 |
| --- | --- |
| `UnexpectedEof` | 头页或数据页被截断 |
| `InvalidFormat` | Magic、计数器、文件长度或记录编码非法 |
| `UnsupportedVersion` | 文件格式版本不受支持 |
| `CorruptedPage` | 页头、槽目录或 Payload 范围非法 |
| `ChecksumMismatch` | 文件头页或数据页 CRC 不匹配 |
| `ResourceLimitExceeded` | PageId、RecordId 或页面空间耗尽 |

打开时发现损坏会拒绝整个集合 Store，而不是跳过坏页或坏记录继续运行。

## CRC32 边界

CRC32 用于检测意外损坏，不提供恶意篡改防护。它不能代替：

- 格式版本检查；
- 长度和范围检查；
- RecordId 唯一性检查；
- Schema 校验；
- WAL 的提交和恢复协议。

## 格式演进

修改 Store 格式时需要同步更新：

- `StorageFormatVersion`；
- 文件头 Codec；
- 数据页 Codec；
- Record/Value Codec；
- 打开与兼容策略；
- 精确字节布局测试；
- 旧版本和损坏文件测试；
- 恢复与事务集成测试。

不能只修改内存结构并继续使用相同版本号。

## 测试重点

当前存储测试覆盖：

- 全部 Value 类型精确编码和往返；
- 文件头和页面布局；
- 关闭后重新打开；
- 更新后 RecordId 稳定；
- 页内整理和删除空间复用；
- 扫描每页一次和快照游标；
- 随机 CRUD 与周期性重开模型；
- 旧版本、头 CRC、页 CRC；
- 槽状态、保留字段、Payload 越界和重叠；
- 截断文件；
- 超大记录和资源边界。

## 当前边界

- 当前只支持格式版本 2。
- 没有旧版本在线迁移器。
- 记录不支持跨页存储。
- 文件没有独立空闲页链或尾页回收。
- 位置目录和空间索引每次打开都重建。
- 页面 CRC32 不提供密码学完整性。
