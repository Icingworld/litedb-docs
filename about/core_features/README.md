# 核心特性

LiteDB 的特点不在于单个功能的新颖，而在于用一套相对紧凑的代码连接数据库的主要层次，并让这些层次能够真实运行和持久化。

## 完整的 SQL 执行流水线

一条 SQL 会依次经过：

```text
Lexer
  -> Parser / AST
  -> Binder
  -> Logical Planner
  -> Rule-based Optimizer
  -> Physical Planner
  -> Executor
```

各阶段保留明确的输入输出模型，便于观察语法结构如何变成带类型的表达式、逻辑算子和最终物理执行。

当前支持基础数据库与集合 DDL、数据增删改查、投影、过滤、排序、分页以及函数表达式。优化器可以按规则选择顺序扫描、标量索引或向量 Top-K 路径。

## 类型化的数据模型

系统提供：

- `INTEGER`、`BIGINT`、`FLOAT`、`DOUBLE`；
- `BOOLEAN`；
- `VARCHAR(n)`；
- 固定维度的 `VECTOR(n)`；
- `NULL`、默认值、列注释与集合注释；
- 类型化的模式、记录和值模型。

模式负责记录布局，Catalog 负责数据库、集合和索引等可变元数据，二者具有不同的所有权边界。

## 可重启的持久化存储

集合记录保存在固定大小页面中，并使用稳定的 `RecordId` 标识。存储层提供：

- 集合文件创建、打开与扫描；
- 记录插入、更新和删除；
- 页面和槽目录管理；
- 变长记录编码；
- 文件头、页面头与校验和；
- 空闲空间复用和启动验证。

这些实现让存储不只是内存容器，而是可以被检查、损坏测试和恢复的真实文件格式。

## 标量与向量两类索引

### B+ 树标量索引

持久化 B+ 树支持：

- 单列标量键；
- 等值查找；
- 开闭区间范围扫描；
- 流式叶子游标；
- 重复键的确定性排序；
- 节点分裂、空页回收和批量构建；
- DML 自动维护和启动恢复。

### HNSW 向量索引

向量子系统支持：

- L2、内积与余弦距离；
- `VECTOR(n)` 合法性和维度检查；
- 精确 Flat 扫描后端；
- 持久化 HNSW 近似最近邻索引；
- SQL Top-K 查询中的 HNSW 路径选择；
- 候选结果的精确距离重算；
- 墓碑、图文件重建和 checkpoint 压缩。

这使 LiteDB 可以在同一查询体系中展示传统有序索引与现代向量检索的差异。

## 跨文件事务与崩溃恢复

每条 DML 或 DDL 语句是一个隐式 `Serializable` 事务。事务层通过文件覆盖层和 redo WAL 协调：

- 集合存储；
- 标量索引；
- 向量索引；
- Catalog 元数据。

提交遵循：

```text
prepare
  -> WAL Begin
  -> WAL FileWrite...
  -> WAL Commit
  -> flush Commit
  -> apply
  -> reload runtime
```

恢复只重放具有完整 Commit 的事务。不完整 WAL 尾部可以安全截断，完整但损坏的记录会被拒绝。Checkpoint 在参与文件持久化后轮换 WAL generation。

## 模块化的系统结构

项目把主要能力拆分为独立模块：

- parser、binder、planner、optimizer、executor；
- meta、schema、storage；
- index、vindex；
- transaction、wal；
- filesystem、io；
- protocol、network、server、client；
- memory 实验模块。

模块之间通过领域接口组合，避免使用一个通用 Store 或协调器抹平 B+ 树、HNSW、Catalog 和 WAL 的语义差异。

## 可验证的实现

测试不仅覆盖正常 SQL 结果，也覆盖：

- 编解码和物理格式；
- 文件截断和校验和损坏；
- 索引创建、维护和恢复；
- WAL 写入预算与重放；
- 提交不同阶段的故障注入；
- DML、DDL 与 checkpoint 崩溃边界；
- 数据库重新打开后的状态一致性；
- 客户端、服务端和协议行为。

这让实现可以作为学习材料，也可以作为架构修改后的回归基线。

## 轻量但不隐藏关键层次

LiteDB 是单节点、单写者的实验系统，没有 MVCC、成本优化器或完整 SQL。它通过控制功能范围保持项目可理解，同时保留存储、索引、事务和恢复等数据库核心层次。

有关尚未实现的能力和适用边界，请继续阅读[应用场景](../use_cases/README.md)。
