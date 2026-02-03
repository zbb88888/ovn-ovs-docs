# ovn-ctl (Service Control Script)

## 简介

`ovn-ctl` 是一个 Bash 脚本，用于简化 OVN 服务的生命周期管理。它通常被 systemd 单元文件或初始化脚本调用，用于启动、停止和重启 OVN 数据库 (`ovsdb-server`) 和守护进程 (`ovn-northd`, `ovn-controller`)。

## 基本用法

```bash
ovn-ctl [options] <command>
```

## 常用命令

### 数据库管理

- `start_nb_ovsdb`: 启动北向数据库服务。
- `start_sb_ovsdb`: 启动南向数据库服务。
- `stop_nb_ovsdb`: 停止北向数据库。
- `stop_sb_ovsdb`: 停止南向数据库。
- `promote_ovnnb`: 在 Raft 集群模式下，将当前节点提升为主 (Leader)。
- `demote_ovnnb`: 将当前节点降级为备。

### 守护进程管理

- `start_northd`: 启动 `ovn-northd`。
- `stop_northd`: 停止 `ovn-northd`。
- `start_controller`: 启动 `ovn-controller`。
- `stop_controller`: 停止 `ovn-controller`。
- `restart_northd`: 重启 `ovn-northd`。
- `restart_controller`: 重启 `ovn-controller`。

## 关键选项

- `--ovn-nb-db=proto:ip:port`: 指定 NB DB 连接地址。
- `--ovn-sb-db=proto:ip:port`: 指定 SB DB 连接地址。
- `--db-nb-create-insecure-remote=yes|no`: 是否自动创建不安全的 NB DB 监听端口 (ptcp)。
- `--db-sb-create-insecure-remote=yes|no`: 是否自动创建不安全的 SB DB 监听端口 (ptcp)。
