# 存储引擎

存储引擎是集合存储的运行时管理层。它以集合 ID 为键持有多个 `CollectionState`，对外提供集合生命周期和记录操作，并在写入前执行 Schema 校验。

## 运行时状态

每个已加载集合对应：

```cpp
CollectionState {
    CollectionSchema schema;
    unique_ptr<StorageStore> store;
}
```

`CollectionSchema` 按值保存。Catalog 中的 Entry 指针可能在 Catalog 变更或恢复后失效，而按值 Schema 可以为在线读取和暂存写入提供稳定的列顺序与类型信息。

`StorageStore` 独占对应集合文件句柄和文件内运行状态。

## Schema 加载

`load_collection_schema` 从 `CatalogView` 读取：

- 数据库 ID；
- 集合 ID、名称和注释；
- 按 Catalog 顺序排列的列；
- 列 ID、序号、名称和逻辑类型；
- 可空、唯一、默认表达式和注释。

加载过程要求集合和所属数据库都存在。得到的 `CollectionSchema` 是记录布局快照，不包含索引定义。

Catalog 与 Schema 的详细关系参见[元数据总览](../../metadata/overview/README.md)和[数据模型](../../data_model/README.md)。

## 存储路径

StorageEngine 根据集合 ID 构造：

```text
data_directory / "collections" / "<collection-id>.store"
```

`data_directory` 可以是正式数据库目录，也可以是事务暂存目录。相同 StorageEngine 代码因此可以打开在线文件或构建暂存后像。

## 打开模式

### LiveReadOnly

默认模式用于正式在线状态。允许：

- `open_collection`
- `reload_collection`
- `contains_collection`
- `get`
- `scan`
- `metrics`
- `clear`

拒绝：

- `create_collection`
- `drop_collection`
- `insert`
- `update`
- `erase`

被拒绝的写操作返回 `InvalidState`，消息表明在线 StorageEngine 是只读的。

### TransactionalStaging

事务暂存模式允许完整 CRUD 和集合文件创建、删除。TransactionManager 使用指向事务目录的暂存 StorageEngine 应用逻辑变更，再把形成的文件后像纳入提交批次。

打开模式是 StorageEngine 公共边界，不是操作系统文件权限。底层 StorageStore 当前仍以读写方式打开文件；禁止在线写入依赖 StorageEngine 不暴露可变 Store。

## 集合生命周期

### 创建

`create_collection`：

1. 要求 `TransactionalStaging`；
2. 检查集合尚未加载；
3. 检查文件系统已经配置；
4. 拒绝覆盖已有 Store 文件；
5. 创建 `StorageStore`；
6. 保存 Schema 和 Store。

StorageStore 创建失败时会关闭句柄并尝试删除未完成文件。

### 打开

`open_collection`：

1. 拒绝重复加载同一集合；
2. 检查 Store 文件存在；
3. 打开 StorageStore；
4. 验证文件头和所有数据页；
5. 重建位置目录和空间索引；
6. 注册 CollectionState。

Catalog 中存在集合但 Store 文件缺失时返回 `CollectionStoreNotFound`，不能把它当成空集合。

### 重载

`reload_collection` 先构造一个新的 StorageStore：

```text
open new store
    ├── failed -> 保留当前 CollectionState
    └── success -> insert_or_assign 新状态
```

这一顺序用于提交后发布和恢复后的运行时刷新。

### 删除

`drop_collection` 只允许暂存模式。它先移除内存状态，再删除对应 Store 文件。跨 Catalog、Storage 和 Index 的最终删除原子性由事务层提供。

## 记录校验

插入和更新在进入 StorageStore 前调用 `validate`。

### 值数量

`RecordData::values` 必须与 Schema 列数量完全相同，并按列序号排列。缺少或多出值都会返回 `ValueCountMismatch`。

### NULL

`NULL` 只允许写入 `nullable` 列。非空列收到 NULL 时返回 `NullConstraintViolation`。

### 逻辑类型

非空值必须由 `Value::matches_type` 匹配列逻辑类型，否则返回 `TypeMismatch`。

### VARCHAR 长度

`VARCHAR(n)` 使用 `std::string::size()` 检查，因此当前限制的是编码后的字节数，而不是 Unicode 字符或字素数量。

### VECTOR 维度

`VECTOR(n)` 要求运行时 `VectorValue` 的元素数量正好为 `n`。

### 单页记录限制

校验器使用与实际记录编码相同的字段写入有界内存缓冲区。可用上限为：

```text
StoragePageSize - StoragePageHeaderSize - StorageSlotSize
```

这保证完整编码记录至少能作为一个槽放入空数据页。超过上限返回 `RecordTooLarge`。

## Schema 约束边界

Schema 中虽然包含列级 `unique` 和默认表达式，但 StorageEngine：

- 不计算默认值；
- 不自动创建索引；
- 不执行唯一性检查；
- 不检查外键或跨记录约束。

默认值在更上层绑定或执行路径处理；唯一性需要实际索引语义支持。StorageEngine 只执行单条记录的布局和大小验证。

## 记录操作

### 读取

`get(collection_id, record_id)`：

1. 查找 CollectionState；
2. 委托 StorageStore 按 RecordId 查找 PhysicalRid；
3. 读取并验证页面；
4. 解码记录。

### 插入

`insert`：

1. 要求暂存模式；
2. 查找集合；
3. 校验 RecordData；
4. 委托 StorageStore 分配 RecordId 和槽位。

### 更新

`update` 保留原 RecordId。新记录可能在原页面重排，也可能移动到另一个页面。

### 删除

`erase` 将物理槽标记为 Deleted，并从内存位置目录移除 RecordId。空间以后可由插入或更新复用。

### 扫描

`scan` 返回拥有记录快照的 `StorageCursor`。创建游标时 Store 已经读取、验证并解码所有活动记录；之后的 `next()` 只在内存向量上移动。

## 指标

StorageEngine 汇总所有 Store 的：

| 指标 | 含义 |
| --- | --- |
| `page_reads` | 数据页读取次数 |
| `page_writes` | 数据页写入次数 |
| `bytes_read` | 读取字节数 |
| `bytes_written` | 写入字节数 |
| `compactions` | 页内整理次数 |
| `reused_pages` | 成功复用已有页面次数 |
| `new_pages` | 新建数据页次数 |
| `checksum_failures` | 打开或读取时发现的校验失败 |

这些指标是进程内累计值，不持久化到 Store 文件。

## 错误上下文

`StorageErrorContext` 可以记录：

- 操作种类；
- 文件路径；
- CollectionId；
- RecordId；
- PageId；
- SlotId；
- 底层编码错误码。

StorageEngine 将文件系统和 IO 失败映射到稳定 Storage 错误，同时保留底层编码码用于诊断。

## 当前边界

- CollectionState 不包含索引状态。
- Schema 按值保存，但 Catalog 仍是权威来源。
- 在线只读约束位于 StorageEngine，不是底层只读文件句柄。
- 校验只覆盖单条记录，不覆盖跨记录约束。
- `clear` 只清空内存状态，不删除文件。
- `reload_collection` 以单集合为粒度。
