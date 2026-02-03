# ovn-sbctl (Southbound DB CLI)

## 简介

`ovn-sbctl` 是用于查询和操作 OVN 南向数据库 (Southbound DB) 的命令行工具。虽然它支持写操作，但在生产环境中，它主要用于**监控**和**调试**，因为 SB DB 的大部分内容是由 `ovn-northd` 和 `ovn-controller` 自动维护的。

## 常用命令

### 系统概览

- `ovn-sbctl show`: 显示 OVN 系统的物理概览。
  - 列出所有的 Chassis（主机名、封装类型）。
  - 列出每个 Chassis 上绑定的 Port。

### Chassis 管理

- `ovn-sbctl chassis-list`: 列出所有注册的 Chassis。
- `ovn-sbctl chassis-del <chassis>`: 删除 Chassis 记录（通常用于清理已下线的节点）。

### 逻辑流表 (Logical Flows)

这是 `ovn-sbctl` 最强大的功能，用于调试网络连通性问题。

- `ovn-sbctl lflow-list [logical-datapath]`: 列出逻辑流表。
  - 可以看到 NB DB 配置是如何被翻译成具体的逻辑流的。
  - 输出包含 `table=X (pipeline)`, `priority=Y`, `match`, `actions`。
- `ovn-sbctl dump-flows`: 同 `lflow-list`。

### 绑定调试

- `ovn-sbctl lsp-bind <port> <chassis>`: 手动将逻辑端口绑定到 Chassis（仅用于测试）。

## 通用数据库操作

与 `ovn-nbctl` 一样，支持底层的数据库记录操作：

- `ovn-sbctl list <table>`
- `ovn-sbctl find <table> <condition>`
- `ovn-sbctl get <table> <record> <column>`
