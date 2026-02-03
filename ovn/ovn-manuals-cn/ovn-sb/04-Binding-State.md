# 绑定状态 (Binding State)

## Port_Binding 表

逻辑端口与物理位置的映射表。这是连接逻辑网络与物理网络的桥梁。

### 核心列

- **logical_port**: 逻辑端口名称（与 NB DB 中的名称一致）。
- **datapath**: 所属的逻辑数据路径。
- **tunnel_key**: 逻辑端口的隧道 ID。
- **type**: 端口类型。
  - `patch`: 连接逻辑交换机和逻辑路由器。
  - `chassisredirect`: 用于分布式网关端口的集中式重定向。
  - `l3gateway`: L3 网关端口。
  - `localport`: 本地端口。
- **chassis**: (关键) 当前绑定该端口的物理 Chassis。当 VM 迁移时，此字段会更新。
- **gateway_chassis**: 用于高可用网关端口的 Chassis 列表。

## MAC_Binding 表

类似于物理网络中的 ARP 表或 IPv6 Neighbor Cache。记录了逻辑端口已知的 IP 到 MAC 的映射。

- **logical_port**: 相关的逻辑端口。
- **ip**: IP 地址。
- **mac**: 对应的 MAC 地址。
- **datapath**: 所属数据路径。

此表通常由 `ovn-controller` 动态学习并填充（通过 ARP 响应），用于在逻辑流中构造以太网帧头。
