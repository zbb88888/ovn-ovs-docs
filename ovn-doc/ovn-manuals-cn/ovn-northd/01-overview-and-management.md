# OVN Northbound Daemon (ovn-northd) 概览与管理

## 简介 (Synopsis)

```bash
ovn-northd [options]
```

## 描述 (Description)

`ovn-northd` 是 OVN (Open Virtual Network) 的集中式控制守护进程。它的主要职责是将 OVN Northbound 数据库中定义的高级网络配置转换为 OVN Southbound 数据库中的逻辑流表 (Logical Flows)。

`ovn-northd` 通常作为集群中的一部分运行，但实际上它是无状态的，所有的状态都存储在数据库中。这意味着多个 `ovn-northd` 实例可以并行运行（通常为主备模式），以提供高可用性。

## 选项 (Options)

### 守护进程选项 (Daemon Options)

*   `--pidfile[=pidfile]`
    创建进程 ID 文件 (默认: `ovn-northd.pid`)。
*   `--detach`
    在后台运行。
*   `--monitor`
    创建监控进程以在守护进程崩溃时自动重启它。
*   `--user`
    以指定用户身份运行。

### 数据库选项 (Database Options)

*   `--ovnnb-db=database`
    指定 OVN Northbound 数据库的位置。可以是 unix socket 路径（如 `unix:/var/run/openvswitch/ovnnb_db.sock`）或 TCP/SSL 地址（如 `tcp:1.2.3.4:6641`）。默认通常为 `unix:/var/run/openvswitch/ovnnb_db.sock`。

*   `--ovnsb-db=database`
    指定 OVN Southbound 数据库的位置。格式同上。默认通常为 `unix:/var/run/openvswitch/ovnsb_db.sock`。

### SSL/TLS 选项

如果连接数据库使用 SSL，需要指定以下选项：

*   `-p privkey.pem`, `--private-key=privkey.pem`
    私钥文件。
*   `-c cert.pem`, `--certificate=cert.pem`
    证书文件。
*   `-C cacert.pem`, `--ca-cert=cacert.pem`
    CA 根证书文件。

### 其他选项

*   `--dry-run`
    如果设置，`ovn-northd` 将连接到数据库并计算逻辑流，但不会将更改提交到 Southbound 数据库。用于测试。
*   `-h`, `--help`
    显示帮助信息。
*   `-V`, `--version`
    显示版本信息。

## 运行时管理命令 (Runtime Management Commands)

`ovn-appctl` 可以向运行中的 `ovn-northd` 发送命令。

*   `exit`
    使 `ovn-northd` 优雅退出。

*   `pause`
    暂停 `ovn-northd` 的处理循环。它将停止向 Southbound 数据库写入更新。

*   `resume`
    恢复 `ovn-northd` 的处理循环。

*   `is-paused`
    返回 `true` 如果当前处于暂停状态，否则返回 `false`。

*   `status`
    显示当前状态信息，包括连接到的数据库状态、上次计算逻辑流的耗时等。

*   `sb-cluster-state-reset`
    重置 Southbound 数据库集群状态的本地缓存。在极少数情况下，如果本地缓存与实际状态不同步，可能需要此命令。

*   `nb-cluster-state-reset`
    重置 Northbound 数据库集群状态的本地缓存。

*   `lflow-cache/flush`
    清除逻辑流缓存。

*   `lflow-cache/stats`
    显示逻辑流缓存的统计信息。

*   `lflow-cache/control [enable|disable]`
    启用或禁用逻辑流缓存。

## Drop Sampling

当在 OVN Northbound 数据库中配置了 `debug_drop_collector_set` 和 `debug_drop_domain_id` 选项时，`ovn-northd` 会将显式的 `drop;` 动作替换为 `sample` 动作，以便对丢弃的数据包进行采样和监控。

*   `debug_drop_collector_set`: 指定采样的 collector set ID。
*   `debug_drop_domain_id`: 指定 Observation Domain ID 的高 8 位（低 24 位为 datapath 的 tunnel key）。
