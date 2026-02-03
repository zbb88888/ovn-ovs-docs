# ovn-appctl (Runtime Management)

## 简介

`ovn-appctl` 是一个通用工具，用于向正在运行的 OVN 守护进程（如 `ovn-controller`, `ovn-northd`）发送控制命令。

- **原理**：通过 Unix Domain Socket 连接到守护进程的管理接口。
- **用途**：查看内部状态、调整日志级别、触发特定操作（如重新连接 DB）。

## 基本用法

```bash
ovn-appctl -t <target> <command> [args...]
```

- **target**: 目标守护进程的名称（如 `ovn-controller`, `ovn-northd`）或 PID 文件路径。

## 常用命令

### 针对 ovn-controller

- `ovn-appctl -t ovn-controller status`
  - 查看与 Southbound DB 的连接状态。
- `ovn-appctl -t ovn-controller lflow-cache/show`
  - 查看逻辑流缓存统计信息（命中率、缓存条目数）。
- `ovn-appctl -t ovn-controller lflow-cache/flush`
  - 强制刷新逻辑流缓存。
- `ovn-appctl -t ovn-controller inject-pkt <microflow> <packet-data>`
  - 向 `ovn-controller` 注入报文，测试物理流表的处理逻辑（比 `ovn-trace` 更底层的测试）。
- `ovn-appctl -t ovn-controller exit`
  - 优雅停止守护进程。

### 针对 ovn-northd

- `ovn-appctl -t ovn-northd status`
  - 查看主备状态（Active/Standby）和数据库连接状态。
- `ovn-appctl -t ovn-northd sb-cluster-state-reset`
  - 重置 SB DB 集群状态（仅在特殊故障恢复时使用）。

### 通用命令

- `list-commands`: 列出目标守护进程支持的所有命令。
- `vlog/list`: 列出当前的日志模块和级别。
- `vlog/set <module>:<level>`: 动态调整日志级别（如 `vlog/set ovn_northd:dbg`）。
- `memory/show`: 显示内存使用情况统计。
