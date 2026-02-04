# OVN 互联控制器运行时管理

## 运行时管理命令 (Runtime Management Commands)

`ovn-appctl` 可以向运行中的 `ovn-ic` 进程发送命令。

*   `exit`
    使 `ovn-ic` 优雅终止。

*   `pause`
    暂停 `ovn-ic` 的操作，停止处理任何数据库更改。这也将指示 `ovn-ic` 放弃对 Southbound 数据库的任何锁定。

*   `resume`
    恢复 `ovn-ic` 的操作以处理数据库内容。这也将指示 `ovn-ic` 尝试获取 Southbound 数据库的锁定。

*   `is-paused`
    如果 `ovn-ic` 当前处于暂停状态，返回 `true`，否则返回 `false`。

*   `status`
    打印此服务器的状态。
    *   `active`: 已获取 SB DB 的 OVSDB 锁。
    *   `standby`: 未获取锁。
    *   `paused`: 处于暂停状态。

## 高可用性 (Active-Standby for High Availability)

您可以在一个 OVN 部署中运行多个 `ovn-ic` 实例。当连接到独立或集群 DB 设置时，OVN 将自动确保一次只有一个实例处于活动状态 (`active`)。如果多个 `ovn-ic` 实例正在运行且活动的 `ovn-ic` 发生故障，其中一个热备 (`hot standby`) 实例将自动接管。
