# 逻辑交换机 (Logical Switch)

## Logical_Switch 表

每个记录代表一个逻辑交换机（即一个 L2 广播域）。

### 核心列

- **name**: 交换机名称。
- **ports**: 指向 `Logical_Switch_Port` 表的引用列表。
- **acls**: 指向 `ACL` 表的引用列表。
- **qos_rules**: 指向 `QoS` 表的引用列表。
- **load_balancer**: 应用于该交换机的负载均衡器列表。
- **other_config**:
  - `subnet`: 定义该交换机的子网（如 `192.168.1.0/24`），用于优化 ARP/ND 处理。
  - `mcast_snoop`: 是否启用 IGMP 监听。

## Logical_Switch_Port 表

定义连接到逻辑交换机的端口。

### type (端口类型)

- **(empty)**: 默认类型，连接 VM 或容器 (VIF)。
- **router**: 连接到逻辑路由器。
- **localnet**: 连接到物理网络（通过本地网桥映射）。
- **localport**: 本地端口，流量不经过隧道，仅在本地处理。
- **vtep**: 连接到硬件 VTEP 网关。

### 核心列

- **name**: 端口名称（必须唯一）。
- **addresses**: 端口拥有的 MAC 和 IP 地址列表。
  - 格式如 `"00:00:00:00:00:01 10.0.0.1"`。
  - 支持特殊值 `"dynamic"`，让 OVN 自动分配。
- **port_security**: 端口安全设置，限制该端口允许发送/接收的源/目的地址。
- **dhcpv4_options** / **dhcpv6_options**: 引用 `DHCP_Options` 表，为此端口提供 DHCP 服务。
