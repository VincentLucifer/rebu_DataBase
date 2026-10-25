# Database Systems Lab · Storage, Buffer Pools and B+ Trees

[简体中文](README.md) | [English](README_EN.md)

C++ database-systems experiments built on the **Rucbase teaching framework**, focusing on how pages move between disk and memory, how records are located, and how B+ trees maintain their structure during insertion and deletion.

**Stack:** C++17 · CMake · GoogleTest · Flex/Bison · Bash

## Core implementations

### Pages and buffer management

`BufferPoolManager` combines a page table, free-frame list and replacer:

- Cache hits pin pages and increment their pin count.
- Misses select a replacement frame, write back dirty data when needed and load from disk.
- Operations include `FetchPage`, `UnpinPage`, `NewPage`, `DeletePage`, `FlushPage` and bulk flushing.
- Mutexes protect key buffer-pool operations.

Pin counts describe active use, while dirty flags describe writeback requirements, connecting cache management with persistence.

### LRU / Clock replacement

`LRUReplacer` and `ClockReplacer` implement a common `Replacer` interface with `Victim`, `Pin`, `Unpin` and `Size`. They expose different eviction policies and state-maintenance approaches.

### Record management

`RmFileHandle` maps records to page slots, tracks occupancy using bitmaps and locates records through `Rid(page_no, slot_no)`. The implementation covers record access, insertion, deletion, updates and free-page-list maintenance.

### B+ tree indexing

`IxIndexHandle` and `IxNodeHandle` organise a page-based B+ tree:

| Operation | Key logic |
| --- | --- |
| Lookup | Traverse from root to leaf and locate a record identifier by key |
| Insertion | Leaf insertion, splitting, parent propagation and root creation |
| Deletion | Key removal, redistribution/coalescing and root adjustment |
| Ordered access | Leaf links, `lower_bound` / `upper_bound` and leaf begin/end positions |
| Page integration | Buffer-pool access with pin/unpin lifecycles |

## Source architecture

| Directory | Contents |
| --- | --- |
| `src/storage/` | Disk and buffer-pool management |
| `src/replacer/` | LRU / Clock and the shared replacement interface |
| `src/record/` | Record files, slot bitmaps and scans |
| `src/index/` | B+ tree nodes, operations and scanning |
| `src/parser/` | SQL lexer, grammar and AST |
| `src/system/` | Table, index and database metadata |
| `src/execution/` | Executor framework, SQL inputs and output comparisons |
| `src/transaction/` / `recovery/` | Course extension interfaces for transactions and recovery |

The source is layered from storage through indexing to execution. The implementation focus here is storage and indexing; parser, executor and transaction-related directories also provide a view of the teaching framework's overall design.

## SQL experiment material

The grammar and `src/execution/input_all_task.sql` cover tables, indexes, record operations and multi-table queries. These representative statements illustrate the course input format:

```sql
create table student (id int, name char(32), major char(32));
create index student (id);
insert into student values (1, 'Tom', 'Computer Science');
select * from student where id >= 1;
update student set major = 'Electrical Engineering' where id = 1;
```

The SQL material and executor interfaces provide an entry point for studying query execution.

## Course build environment

This repository is organised as a course `src/` source package. Integrate it with the Rucbase parent project's CMake configuration and dependencies, including a C++17 compiler, GoogleTest, Flex/Bison and the required system libraries.

```bash
git clone https://github.com/VincentLucifer/rebu_DataBase.git
```

Within the configured course parent project, build the SQL experiment target described in `src/execution/README.md`:

```bash
cmake --build build --target exec_sql
```

Course scripts expect `build/bin/exec_sql` and run from `src/execution/`:

```bash
cd src/execution
bash taskall_test.sh
```

The script reads SQL statements and compares actual output with reference output using `diff`. It creates and removes experiment database directories.

## Tests and reading guide

The repository includes GoogleTest sources for buffers, disk management, LRU/Clock, record management and B+ trees, plus SQL input/output comparison scripts.

Suggested reading order:

1. `storage/page.h` and `disk_manager.cpp` for page layout and disk access.
2. `buffer_pool_manager.cpp` and `replacer/` for hits, pinning, eviction and flushing.
3. `record/rm_file_handle.cpp` for records within pages.
4. `index/ix_node_handle.cpp` and `ix_index_handle.cpp` for node operations and tree maintenance.
5. `parser/` and `execution/` for connections to the query framework.

## Background

Database coursework based on the Rucbase teaching framework, exploring kernel data structures and resource lifecycles while retaining course interfaces, tests and extension-module organisation.
