# 编译源码

在完成 [下载源码](../installation/README.md) 并确认 `third_party/asio` 已初始化后，可使用 CMake 配置并编译 LiteDB。

## 环境要求

| 依赖 | 要求 |
| --- | --- |
| CMake | **4.3.2** 或更高版本 |
| 编译器 | 需要支持 **C++26** |
| 子模块 | `third_party/asio` |

可选工具：

- Ninja 或 Make

用下列命令确认 CMake 版本：

```sh
cmake --version
```

## 配置与编译

在仓库根目录执行：

```sh
cmake -S . -B build -DLITEDB_BUILD_EXAMPLES=ON
cmake --build build
```

- `-S .`：源码目录为当前仓库根目录
- `-B build`：生成文件与产物输出到 `build/`
- `-DLITEDB_BUILD_EXAMPLES=ON`：同时编译示例服务端与客户端（默认即为 `ON`）

若只需库与测试、不需要示例：

```sh
cmake -S . -B build -DLITEDB_BUILD_EXAMPLES=OFF
cmake --build build
```

### 并行编译

多核机器上可加速构建，例如使用 8 个并行任务：

```sh
cmake --build build --parallel 8
```

## 运行测试

测试在配置时通过 CTest 注册，编译完成后执行：

```sh
ctest --test-dir build --output-on-failure
```

当前测试覆盖解析器、meta、schema、持久化 B+Tree / HNSW、存储与恢复、WAL、事务与 checkpoint、绑定器、逻辑 / 物理计划、优化器、求值器、执行器、数据库引擎、函数注册表、协议、内存，以及客户端 / 服务端行为。

仅运行名称匹配某模式的测试时，可使用 `-R`：

```sh
ctest --test-dir build --output-on-failure -R parser
```

## 构建产物

默认开启示例时，单配置生成器下常见产物路径为：

| 产物 | 路径（Linux / macOS / MinGW 等） |
| --- | --- |
| 示例服务端 | `build/examples/server/litedb_example_server` |
| 示例客户端 CLI | `build/examples/client_cli/litedb_example_client_cli` |
| 各子系统测试可执行文件 | `build/tests/...` |

Windows 上可执行文件通常带 `.exe` 后缀，例如：

```text
build\examples\server\litedb_example_server.exe
build\examples\client_cli\litedb_example_client_cli.exe
```

库与内部目标位于 `build/` 下对应的 `internal` / `tests` 子目录；日常使用示例程序即可，无需单独安装。

## 常见问题

### 配置时报找不到 Asio 头文件

可能是子模块未初始化。回到仓库根目录执行：

```sh
git submodule update --init --recursive
```

确认 `third_party/asio/include/asio.hpp` 存在后重新配置。

### CMake 版本过低

顶层 `CMakeLists.txt` 要求 `cmake_minimum_required(VERSION 4.3.2)`。请升级 CMake，或通过官方安装包 / 包管理器获取足够新的版本。

### 编译器不支持 C++26

项目将 `CMAKE_CXX_STANDARD` 设为 `26` 且强制开启。请更换或升级到支持 C++26 的工具链，并在配置时显式指定编译器（示例）：

```sh
cmake -S . -B build -DCMAKE_CXX_COMPILER=g++ -DLITEDB_BUILD_EXAMPLES=ON
```

### 重新配置

更换生成器、编译器或关键选项后，建议清理构建目录再配置：

```sh
cmake -E rm -rf build
cmake -S . -B build -DLITEDB_BUILD_EXAMPLES=ON
cmake --build build
```

## 下一步

编译成功后，阅读 [快速使用](../quick_start/README.md)，启动示例服务端并执行 SQL 测试。
