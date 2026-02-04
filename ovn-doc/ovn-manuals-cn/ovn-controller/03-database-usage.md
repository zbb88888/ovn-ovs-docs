# OVN 数据库使用 (Database Usage)

`ovn-controller` 在运行时与本地 Open vSwitch 数据库和远程 OVN Southbound 数据库进行频繁的读写交互。

## Open vSwitch 数据库使用 (Open vSwitch Database Usage)

`ovn-controller` 使用本地 OVS 数据库中的 `external_ids` 键来跟踪端口和接口的状态。**用户不应手动修改或清除这些键。**

### Port 表 (Port Table)

*   `external_ids:ovn-chassis-id`
    标识这是一个由 `ovn-controller` 创建的隧道端口，用于连接到远程 Chassis。其值为远程 Chassis 的 ID。

*   `external_ids:ovn-localnet-port`
    标识这是一个用于实现 `localnet` 逻辑端口的 Patch Port。其值为逻辑端口名称。通常成对出现（一个在 `br-int`，一个在物理网桥）。

*   `external_ids:ovn-l2gateway-port` / `external-ids:ovn-l3gateway-port`
    标识用于实现 L2 或 L3 网关逻辑端口的 Patch Port。

*   `external-ids:ovn-logical-patch-port`
    标识用于实现 OVN 逻辑 Patch Port 的 Patch Port。

### Bridge 表 (Bridge Table)

*   `external_ids:ct-zone-*`
    用于存储连接跟踪 (Connection Tracking) 区域 (Zone) 的分配信息。
    *   格式: `ct-zone-<LOGICAL_PORT_NAME>`。
    *   `ovn-controller` 为每个需要状态化服务的逻辑端口或网关路由器分配唯一的 Zone ID，并在此处持久化，以便重启后保持一致。

*   `external-ids:ovn-nb-cfg` / `external-ids:ovn-nb-cfg-ts`
    记录最近一次成功在 OVS 中安装所有流表对应的 OVN Northbound 配置版本 (`nb_cfg`) 及其时间戳。这用于向北向报告配置已生效。

### Interface 表 (Interface Table)

*   `external_ids:ovn-installed`
    当对应的 OpenFlow 操作已被 `ovs-vswitchd` 处理时设置此键。

*   `external_ids:ovn-evpn-tunnel`
    指示该接口是为 EVPN 流量创建的隧道端口。

## OVN Southbound 数据库使用 (OVN Southbound Database Usage)

`ovn-controller` 主要从 SB 数据库读取逻辑流 (`Logical_Flow`)、端口绑定 (`Port_Binding`) 等表来指导操作，并写入以下表：

### Chassis 表 (Chassis Table)
*   启动时，`ovn-controller` 创建一行来代表自己。
*   优雅退出时，删除该行。
*   报告 `hostname`, `datapath_type`, `iface_types` 等能力信息。

### Encap 表 (Encap Table)
*   启动时创建行，描述其隧道封装配置 (IP, Type)，并关联到 `Chassis` 表。

### Port_Binding 表 (Port_Binding Table)
*   **Chassis 绑定**: 运行时，如果发现某个逻辑端口（VIF）位于本节点（检测到对应的 OVS 接口），`ovn-controller` 会将该 `Port_Binding` 记录的 `chassis` 列更新为自己的 Chassis ID。
*   当端口不再位于本节点时，清除该列。

### MAC_Binding 表 (MAC_Binding Table)
*   根据 `put_arp` 和 `put_nd` 逻辑动作的指示，更新 IP-MAC 绑定信息（即 ARP/ND 表项）。这些更改在控制器重启后依然保留。
