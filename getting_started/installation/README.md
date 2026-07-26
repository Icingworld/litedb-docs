# 下载源码

LiteDB 源码托管在 GitHub：[Icingworld/litedb](https://github.com/Icingworld/litedb)。本节说明如何获取最新代码或指定历史版本。编译步骤见 [编译源码](../building/README.md)。

## 环境准备

下载源码前，请确保本机已安装：

- [Git](https://git-scm.com/)

项目依赖独立 Asio，通过 Git 子模块 `third_party/asio` 引入。克隆时需要一并拉取子模块。

## 克隆最新源码

使用 `--recurse-submodules` 一次性拉取主仓库与子模块：

```sh
git clone --recurse-submodules https://github.com/Icingworld/litedb.git
cd litedb
```

若克隆时未带上子模块，可在仓库根目录补拉：

```sh
git submodule update --init --recursive
```

默认检出的是 `main` 分支，对应当前开发主干。

## 下载指定版本

LiteDB 通过 Git 标签发布版本（例如 `v0.7.1`、`v0.7.0`）。获取某一历史版本有两种常用方式。

### 方式一：克隆后切换到标签

```sh
git clone --recurse-submodules https://github.com/Icingworld/litedb.git
cd litedb
git checkout v0.7.1
git submodule update --init --recursive
```

切换标签后务必再次执行 `git submodule update`，以保证子模块与该版本一致。

### 方式二：浅克隆指定标签

若只需要某一发布版本、不需要完整提交历史，可浅克隆该标签：

```sh
git clone --depth 1 --branch v0.7.1 --recurse-submodules https://github.com/Icingworld/litedb.git
cd litedb
```

可用标签列表见仓库的 [Tags](https://github.com/Icingworld/litedb/tags) 页面，或在已克隆的仓库中执行：

```sh
git tag --sort=-v:refname
```
