# LiteDB

LiteDB 是一个使用现代 C++ 从零构建的轻量级实验型数据库。

项目希望把数据库教材、论文和工业系统中的核心知识，整理为一套可以阅读、运行、调试和继续改造的实现：从 SQL 文本进入解析、绑定、规划与执行流水线，再到记录存储、B+ 树、HNSW、事务、WAL 与崩溃恢复。

LiteDB 的首要目标是学习和架构实验，而不是替代成熟的生产数据库。代码更重视模块边界、执行过程和持久化协议的可观察性，便于沿着一条完整链路理解数据库如何工作。

项目地址：[github/Icingworld/litedb](https://github.com/Icingworld/litedb)

## 从这里开始

你可以根据目的选择阅读入口：

| 你的目标 | 推荐入口 |
| --- | --- |
| 第一次接触 LiteDB | [开始使用](getting_started/README.md) |
| 编译并运行项目 | [下载源码](getting_started/installation/README.md) → [编译源码](getting_started/building/README.md) → [快速使用](getting_started/quick_start/README.md) |
| 查询 SQL 语法 | [SQL 语句](sql/README.md) |
| 理解一次 SQL 如何执行 | [SQL 处理流水线](architecture_and_design/sql_processing_pipeline/overview/README.md) |
| 学习记录如何持久化 | [存储引擎](architecture_and_design/storage_engine/overview/README.md) |
| 学习 B+ 树与 HNSW | [标量索引](architecture_and_design/scalar_index/overview/README.md) / [向量索引](architecture_and_design/vector_index/overview/README.md) |
| 理解提交与崩溃恢复 | [事务与恢复](architecture_and_design/transaction_and_recovery/overview/README.md) |
| 阅读完整系统设计 | [架构与设计](architecture_and_design/README.md) |
| 了解项目为什么建立 | [设计目标](about/design_goals/README.md) |

## 当前能力

当前源码版本为 **0.7.1**，已经形成一条可持久化、可重启的单节点数据库主链路：

- SQL 词法分析、语法分析、绑定、逻辑计划、规则优化、物理计划与执行；
- 数据库、集合和模式元数据管理；
- `INSERT`、`SELECT`、`UPDATE`、`DELETE`；
- 投影、过滤、排序、`LIMIT` 与 `OFFSET`；
- 持久化集合存储和启动恢复；
- 持久化单列 B+ 树索引及等值、范围查询；
- `VECTOR(n)`、向量距离函数和持久化 HNSW 索引；
- 隐式语句级事务、redo WAL、崩溃恢复和 checkpoint；
- 简单的 TCP 服务端、客户端与消息协议；
- 覆盖主要模块、持久化格式和崩溃边界的测试。

更完整的能力说明请阅读[核心特性](about/core_features/README.md)。

## 开发状态

> LiteDB 仍处于早期实验阶段，当前版本适合学习、测试和架构研究，不适合承载生产数据。

| 领域 | 当前状态 |
| --- | --- |
| SQL 主流水线 | 已实现，可执行基础 DDL、DML 与查询 |
| 持久化存储 | 已实现，文件格式仍处于实验期 |
| 标量索引 | 已实现单列 B+ 树 |
| 向量能力 | 已实现距离函数和 HNSW Top-K 路径 |
| 事务与恢复 | 已实现单写者、隐式语句事务和 redo WAL |
| 查询优化 | 已实现规则优化，尚无统计信息和成本模型 |
| 并发控制 | 仅单写者，尚无 MVCC 和多写者调度 |
| SQL 覆盖 | 尚无连接、子查询、聚合与 `GROUP BY` |
| 兼容性承诺 | 暂不保证跨实验版本的磁盘格式兼容 |

近期演进方向包括统计信息与成本优化、`EXPLAIN`、后台 checkpoint、显式多语句事务、MVCC，以及更完整的 SQL 能力。

## 关于项目

- [设计目标](about/design_goals/README.md)：为什么从零构建 LiteDB；
- [核心特性](about/core_features/README.md)：LiteDB 当前实现了什么；
- [应用场景](about/use_cases/README.md)：项目适合和不适合用于什么场景。

## 许可证

LiteDB 使用 MIT License 发布。
