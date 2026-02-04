# OVN 控制器配置 (Configuration)

`ovn-controller` 从本地 Open vSwitch 的 `ovsdb-server` 实例中获取大部分配置信息。默认连接到本地 OVS "run" 目录下的 `db.sock`。

`ovn-controller` 主要从本地 OVS 实例的 `Open_vSwitch` 表中的 `external_ids` 列读取配置。

## 关键配置项 (Open_vSwitch 表 external_ids)

以下键值对需要在本地 OVS 数据库中配置：

*   `external_ids:system-id`
    **[必须]** 该 Chassis 的唯一名称。用于在 OVN Southbound 数据库的 `Chassis` 表中标识此节点。通常与 hostname 相同，但在集群中必须唯一。

*   `external_ids:ovn-remote`
    **[必须]** OVN Southbound 数据库的连接地址。
    *   格式: `tcp:IP:PORT` 或 `ssl:IP:PORT` (如 `tcp:192.168.1.1:6642`)。
    *   如果是集群模式，可以用逗号分隔多个地址。

*   `external_ids:ovn-encap-type`
    **[必须]** 该 Chassis 用于与其他节点通信的隧道封装类型。
    *   支持的值: `geneve`, `vxlan`。
    *   可以指定多个，用逗号分隔。

*   `external_ids:ovn-encap-ip`
    **[必须]** 该 Chassis 用于建立隧道的本地 IP 地址。其他节点将使用此 IP 向该节点发送隧道流量。

*   `external_ids:hostname`
    该 Chassis 的主机名。如果未设置，可能会尝试自动检测。

*   `external_ids:ovn-bridge`
    集成网桥 (Integration Bridge) 的名称。
    *   默认: `br-int`。
    *   如果该网桥不存在，`ovn-controller` 会自动创建它。

*   `external_ids:ovn-bridge-mappings`
    物理网络名称到本地 OVS 网桥的映射。用于连接物理网络（通过 `localnet` 端口）。
    *   格式: `physnet1:br-eth0,physnet2:br-eth1`。
    *   这意味着逻辑网络中名为 `physnet1` 的网络流量将通过本地的 `br-eth0` 网桥转发。

## 性能与调优选项

*   `external_ids:ovn-monitor-all`
    *   `true`: 无条件监控 OVN Southbound 数据库中的所有记录。
    *   `false` (默认): 仅根据需要有条件地监控记录。
    *   在所有工作负载都需要互通的环境中，设置为 `true` 通常更高效，因为减少了服务端的条件处理开销。

*   `external_ids:ovn-remote-probe-interval`
    OVN 数据库连接的探活间隔 (毫秒)。设置为 0 禁用探活。

*   `external_ids:ovn-enable-lflow-cache`
    是否启用逻辑流内存缓存。默认启用 (`true`)。

*   `external_ids:ovn-limit-lflow-cache`
    限制逻辑流缓存的最大条目数。

*   `external_ids:ovn-memlimit-lflow-cache-kb`
    限制逻辑流缓存的最大内存使用量 (KB)。

## 其他配置

*   `external_ids:ovn-chassis-mac-mappings`
    用于分布式网关端口的 Chassis 特定 MAC 地址映射。

*   `external_ids:ovn-is-interconn`
    指示该 Chassis 是否作为互联 (Interconnection) 网关。

*   `external_ids:ovn-match-northd-version`
    如果设置为 `true`，`ovn-controller` 将检查其版本是否与 `ovn-northd` 版本匹配，不匹配则停止处理。

## 从其他 OVS 表读取的配置

*   **Bridge 表**: `datapath-type`
    从本地集成网桥读取数据路径类型，并上报到 SB 数据库。
*   **SSL 表**: `private_key`, `certificate`, `ca_cert`
    读取 SSL 配置以连接 OVN 数据库。
