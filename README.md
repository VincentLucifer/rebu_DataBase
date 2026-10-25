# Database Systems Lab · 存储、缓冲池与 B+ 树

[简体中文](README.md) | [English](README_EN.md)

基于 **Rucbase 教学数据库框架**的 C++ 数据库系统实验，重点探索页面如何在磁盘与内存间流动、记录如何定位，以及 B+ 树如何在插入和删除时维护结构。

**技术栈：** C++17 · CMake · GoogleTest · Flex/Bison · Bash

## 核心实现

### 页面与缓冲池

`BufferPoolManager` 使用页表、空闲帧列表和置换器管理内存页：

- 命中缓存时固定页面并增加 pin count；
- 缺页时选择可替换帧，按需写回脏页，再从磁盘加载；
- 支持 `FetchPage`、`UnpinPage`、`NewPage`、`DeletePage`、`FlushPage` 和批量刷盘；
- 使用互斥量保护关键缓冲池操作。

页面“是否被使用”和“是否需要写回”分别由 pin count 与 dirty 标记表达，连接缓存管理和持久化流程。

### LRU / Clock 置换

仓库分别实现 `LRUReplacer` 与 `ClockReplacer`。两者通过统一 `Replacer` 接口提供 `Victim`、`Pin`、`Unpin` 和 `Size`，用于比较不同页面淘汰策略及其状态维护方式。

### 记录管理

`RmFileHandle` 将记录映射到页内 slot，使用位图表示占用情况，以 `Rid(page_no, slot_no)` 定位记录。实现包含记录读写、插入、删除、更新，以及空闲页链的维护。

### B+ 树索引

`IxIndexHandle` 与 `IxNodeHandle` 组织页式 B+ 树：

| 操作 | 关键逻辑 |
| --- | --- |
| 查询 | 从根向下定位叶子，按键查找记录位置 |
| 插入 | 叶子插入、节点分裂、向父节点传播、新根创建 |
| 删除 | 删除键值、重分配/合并、根节点调整 |
| 顺序访问 | 叶节点链接、`lower_bound` / `upper_bound`、叶子起止位置 |
| 页面协作 | 通过缓冲池访问节点，配合 pin/unpin 生命周期 |

## 源码架构

| 目录 | 内容 |
| --- | --- |
| `src/storage/` | 磁盘与缓冲池管理 |
| `src/replacer/` | LRU / Clock 与统一置换接口 |
| `src/record/` | 记录文件、slot 位图、记录扫描 |
| `src/index/` | B+ 树节点、索引操作与扫描 |
| `src/parser/` | SQL 词法、语法与 AST |
| `src/system/` | 表、索引与数据库元数据管理 |
| `src/execution/` | 执行器框架、SQL 实验输入和输出对照 |
| `src/transaction/` / `recovery/` | 课程事务与恢复拓展接口 |

源码由存储、索引和执行层逐级组织。本文的实现重点是存储与索引；解析器、执行器和事务相关目录也可用于阅读课程数据库框架的整体设计。

## SQL 实验材料

语法文件与 `src/execution/input_all_task.sql` 涵盖建表、索引、记录操作与多表查询。下面摘取同类语句，展示课程实验的输入形式：

```sql
create table student (id int, name char(32), major char(32));
create index student (id);
insert into student values (1, 'Tom', 'Computer Science');
select * from student where id >= 1;
update student set major = 'Electrical Engineering' where id = 1;
```

SQL 材料与执行器接口共同构成查询执行实验的阅读入口。

## 课程构建环境

本仓库按课程 `src/` 源码包组织。使用课程 Rucbase 父工程的 CMake 配置与依赖环境集成该目录，准备 C++17 编译器、GoogleTest、Flex/Bison，以及相应系统库。

```bash
git clone https://github.com/VincentLucifer/Rucbase-Lab.git
```

在课程父工程已配置的构建目录中，可按 `src/execution/README.md` 构建 SQL 实验目标：

```bash
cmake --build build --target exec_sql
```

课程执行脚本约定二进制位于 `build/bin/exec_sql`，脚本从 `src/execution/` 运行：

```bash
cd src/execution
bash taskall_test.sh
```

脚本读取 SQL 输入并使用 `diff` 比较实际输出和参考输出；执行时会创建和清理实验数据库目录。

## 测试与阅读入口

仓库包括缓冲池、磁盘、LRU/Clock、记录管理与 B+ 树相关 GoogleTest 源码，以及 SQL 输入/输出对照脚本。

建议按以下顺序阅读：

1. `storage/page.h` 与 `disk_manager.cpp`：页面布局和磁盘访问。
2. `buffer_pool_manager.cpp` 与 `replacer/`：缓存命中、固定、置换和刷盘。
3. `record/rm_file_handle.cpp`：页面上的记录组织。
4. `index/ix_node_handle.cpp` 与 `ix_index_handle.cpp`：节点操作和树结构维护。
5. `parser/` 与 `execution/`：理解存储接口如何连接上层查询框架。

## 项目背景

本项目是基于 Rucbase 教学框架的数据库课程实践，围绕数据库内核的数据结构与资源生命周期展开，保留课程接口、测试和拓展模块的组织方式。
