# OVN 守护进程生命周期管理工具 (ovn-ctl)

## 简介 (Synopsis)

```bash
ovn-ctl [options] command [-- extra_args]
```

## 描述 (Description)

此程序旨在由 Open Virtual Network 启动脚本内部调用。系统管理员通常不应直接调用它。

## 命令 (Commands)

### 守护进程管理

*   `start_northd`: 启动 `ovn-northd`。
*   `stop_northd`: 停止 `ovn-northd`。
*   `restart_northd`: 重启 `ovn-northd`。
*   `start_controller`: 启动 `ovn-controller`。
*   `stop_controller`: 停止 `ovn-controller`。
*   `restart_controller`: 重启 `ovn-controller`。
*   `start_ic`: 启动 `ovn-ic`。
*   `stop_ic`: 停止 `ovn-ic`。
*   `restart_ic`: 重启 `ovn-ic`。

### 数据库管理

*   `start_ovsdb`: 启动 OVN 相关的 OVSDB 服务器。
*   `start_nb_ovsdb`: 仅启动 Northbound OVSDB。
*   `start_sb_ovsdb`: 仅启动 Southbound OVSDB。
*   `stop_ovsdb`: 停止所有 OVN OVSDB 服务器。
*   `restart_ovsdb`: 重启所有。
*   `promote_ovnnb` / `demote_ovnnb`: 提升/降级 Northbound 数据库服务器（集群模式）。
*   `status_ovnnb` / `status_ovnsb`: 检查状态。

### 互联数据库管理 (Interconnection DBs)

*   `start_ic_ovsdb`: 启动 IC NB 和 IC SB 数据库。
*   `start_ic_nb_ovsdb` / `start_ic_sb_ovsdb`: 单独启动。
*   `promote_ic_nb` / `demote_ic_nb` ...

### 非分离模式运行 (Running without detaching)

用于容器环境：
*   `run_nb_ovsdb`: 运行 NB OVSDB 服务器（阻塞）。
*   `run_sb_ovsdb`: 运行 SB OVSDB 服务器（阻塞）。

## 选项 (Options)

### 优先级与包装器

*   `--ovn-northd-priority=NICE`: 设置 `ovn-northd` 的 nice 值。
*   `--ovn-northd-wrapper=WRAPPER`: 使用包装器（如 `valgrind`）运行。
*   其他组件 (`ovn-controller`, `ovn-ic` 等) 也有类似的 `priority` 和 `wrapper` 选项。

### 文件位置选项 (File Location Options)

*   `--db-sock=SOCKET`: 数据库套接字位置。
*   `--db-nb-file=FILE`: Northbound 数据库文件路径。
*   `--db-sb-file=FILE`: Southbound 数据库文件路径。
*   `--db-nb-schema=FILE`: NB 模式文件路径。
*   `--db-sb-schema=FILE`: SB 模式文件路径。

### 地址与端口选项 (Address and Port Options)

*   `--db-nb-sync-from-addr=IP`: NB DB 同步源 IP（用于集群）。
*   `--ovn-northd-nb-db=PROTO:IP:PORT`: `ovn-northd` 连接 NB DB 的地址。

### 集群选项 (Clustering Options)

*   `--db-nb-cluster-local-addr=IP`: 本地集群 IP。
*   `--db-nb-cluster-remote-addr=IP`: 远程集群 IP（加入集群时使用）。

### SSL/TLS 选项

*   `--ovn-controller-ssl-key=KEY`
*   `--ovn-controller-ssl-cert=CERT`
*   `--ovn-controller-ssl-ca-cert=CERT`

## 示例用法 (Example Usage)

**在已运行 OVS 的主机上启动 `ovn-controller`**:
```bash
ovn-ctl start_controller
```

**启动中心控制器 `ovn-northd`**:
```bash
ovn-ctl start_northd
```

**创建 3 节点集群**:
在节点 X (x.x.x.x) 上启动:
```bash
ovn-ctl --db-nb-addr=x.x.x.x --db-nb-cluster-local-addr=x.x.x.x ... start_northd
```
在节点 Y (y.y.y.y) 上启动并加入 X:
```bash
ovn-ctl --db-nb-addr=y.y.y.y --db-nb-cluster-local-addr=y.y.y.y --db-nb-cluster-remote-addr=x.x.x.x ... start_northd
```
