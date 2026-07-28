# 事务与恢复总览

LiteDB 当前实现的是单写者、redo WAL 驱动的隐式事务。事务先在隔离的文件覆盖层中计算最终物理差异，再把这些 after-image 写入 WAL；只有 `Commit` 记录持久化后，才把差异应用到正式数据文件。

```mermaid
flowchart LR
    SQL["一条写语句"] --> Stage["TransactionContext 写集合"]
    Stage --> Prepare["文件覆盖层中准备"]
    Prepare --> Batch["FileWriteBatch"]
    Batch --> WAL["Begin + FileWrite + Commit"]
    WAL --> Flush["持久化 Commit"]
    Flush --> Apply["应用到正式文件"]
    Apply --> Reload["重载运行时状态"]
```

## 核心组件

| 组件 | 职责 |
| --- | --- |
| `TransactionManager` | 单写者协调、准备、提交、回滚、checkpoint |
| `TransactionContext` | 事务状态、行写集合、Catalog 快照和 LSN |
| `TransactionFileOverlay` | 在内存 4 KiB 脏块中模拟文件修改 |
| `FileWriteBatch` | 描述可重放的物理文件 after-image |
| `WalManager` / `WalStore` | WAL 分段、追加、刷盘、扫描和轮换 |
| `RecoveryManager` | 识别已提交事务并重放文件写入 |

## 原子性边界

WAL 的 `FileWrite` 可以指向：

- 集合存储文件；
- 标量索引文件；
- 向量索引文件；
- `meta.lmeta`。

因此一个事务可把多个参与者的物理修改放入同一组 `Begin ... Commit` 记录。恢复只重放具有完整 `Commit` 的事务，从而维持跨文件原子提交。

## 关键分界：Commit 持久化

在 `Commit` 记录刷盘之前，失败可以安全中止，正式数据文件尚未被修改。

在 `Commit` 记录刷盘之后，事务已经持久化，不能再语义回滚。即使应用正式文件或重载运行时失败，系统也会进入 `recovery_required` 状态，由 WAL 恢复完成已承诺的写入。

## DML 与 DDL

DML 写集合保存行级的 before/after 数据，但准备阶段最终导出的是文件级 redo 差异。

DDL 不与行变更混合。它暂存一个完整 Catalog 快照，并在覆盖层中同时计算：

- 新 `meta.lmeta`；
- 新建或删除的集合文件；
- 新建或删除的标量索引文件；
- 新建或删除的向量索引文件。

所以 DDL 的 Catalog 与物理文件也经过同一 WAL 提交边界。

## 当前事务模型

- 只支持隐式事务；
- 只实现 `Serializable` 枚举值；
- 核心层使用互斥锁保证单写者；
- 没有并发多版本读写或用户可见的显式事务块；
- 行写集合在准备阶段会重放到文件覆盖层；
- WAL 是 redo-only，不保存用于 undo 的 before-image；
- checkpoint 在参与文件同步后轮换 WAL。

## 阅读顺序

1. [事务模型与暂存](../model_and_staging/README.md)
2. [提交协议](../commit_protocol/README.md)
3. [WAL 格式与文件写入](../wal/README.md)
4. [崩溃恢复](../recovery/README.md)
5. [Checkpoint](../checkpoint/README.md)
