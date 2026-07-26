# 快速使用

当按照 [编译源码](../building/README.md) 完成构建，且示例程序已生成后，可以在本机启动示例服务端，并用 CLI 客户端连接。

## 启动示例服务端

在仓库根目录打开一个终端：

```sh
./build/examples/server/litedb_example_server --host 127.0.0.1 --port 5252
```

Windows（MinGW 等单配置生成器）通常为：

```powershell
.\build\examples\server\litedb_example_server.exe --host 127.0.0.1 --port 5252
```

成功后会看到类似输出：

```text
LiteDB example server listening on 127.0.0.1:5252
```

常用参数：

| 参数 | 默认值 | 说明 |
| --- | --- | --- |
| `--host` | `127.0.0.1` | 监听地址 |
| `--port` | `5252` | 监听端口 |
| `--data-dir` | `litedb-data` | 持久化数据目录 |

指定其他数据目录：

```sh
./build/examples/server/litedb_example_server --host 127.0.0.1 --port 5252 --data-dir ./data
```

`--data-dir` 下会生成 `manifest.ldb`、`meta.lmeta`，以及 `wal/`、`collections/`、`indexes/`、`vindexes/` 等子目录。**当前存储与索引格式仍为实验性，不承诺跨版本二进制兼容；同一数据目录也只允许一个引擎实例以写模式打开**。

用 `Ctrl+C` 可停止服务端。

## 启动客户端 CLI

另开一个终端，连接到同一地址与端口：

```sh
./build/examples/client_cli/litedb_example_client_cli --host 127.0.0.1 --port 5252
```

Windows：

```powershell
.\build\examples\client_cli\litedb_example_client_cli.exe --host 127.0.0.1 --port 5252
```

连接成功后会出现：

```text
Connected to LiteDB at 127.0.0.1:5252
litedb>
```

CLI 会持续读取输入，直到某一行包含分号 `;` 才把整段 SQL 发给服务端。多行语句时，续行提示为 `...>`。输入 `.quit` 或 `.exit` 退出客户端。

常用参数：

| 参数 | 默认值 | 说明 |
| --- | --- | --- |
| `--host` | `127.0.0.1` | 连接地址 |
| `--port` | `5252` | 连接端口 |

## 第一条 SQL

在 `litedb>` 提示符下依次执行：

```sql
CREATE DATABASE demo;
USE demo;

CREATE COLLECTION users (
    id BIGINT NOT NULL,
    name VARCHAR(64) NOT NULL COMMENT 'display name',
    age INTEGER,
    active BOOLEAN DEFAULT true,
    embedding VECTOR(3)
) COMMENT 'user collection';

INSERT INTO users (id, name, age, active, embedding)
VALUES (1, 'Ada', 36, true, [0.1, 0.2, 0.3]);

INSERT INTO users (id, name, age, active, embedding)
VALUES (2, 'Linus', 55, true, [0.2, 0.3, 0.4]);

SELECT id, name, age
FROM users
WHERE active = true
ORDER BY age DESC
LIMIT 10;
```

DDL / DML 成功时，CLI 会打印类似 `OK, affected rows: ...`；`SELECT` 则以表格形式展示结果行。

## 标量索引与向量检索

在已有 `users` 集合上，可继续创建 B+Tree 标量索引与 HNSW 向量索引：

```sql
CREATE INDEX idx_age ON users (age) USING BTREE;
CREATE INDEX idx_name ON users (name) USING BTREE;

CREATE VINDEX idx_embedding ON users (embedding) USING HNSW
WITH (metric = L2, max_neighbors = 16, ef_construction = 200, ef_search = 64);

SELECT id
FROM users
ORDER BY l2_distance(embedding, [0.1, 0.2, 0.3]) ASC
LIMIT 5;

SHOW VINDEXES FROM users;
```

## SQL 语法

LiteDB 实现了一种 SQL 方言，与标准 SQL 语法不完全一致，包含了必要的一些语句，并扩展了有关向量的部分语法，详情参考 [SQL 语句](../../sql/README.md)。

## 常见问题

### 客户端连接失败

确认服务端仍在运行，且 `--host` / `--port` 与客户端一致。若改过端口，两端都要同步修改。

### 打开数据目录失败

同一 `--data-dir` 不能被多个服务端进程同时打开。需要先关掉旧实例，或换一个空目录再进行尝试。

### 旧数据无法打开

存储格式若发生不兼容升级（**数据库目前处于实验性阶段，不承诺跨版本二进制兼容，不建议存储重要业务数据**），旧的存储格式会被明确拒绝且不会自动迁移。请使用新的 `--data-dir`，或按当前版本重新导入数据。
