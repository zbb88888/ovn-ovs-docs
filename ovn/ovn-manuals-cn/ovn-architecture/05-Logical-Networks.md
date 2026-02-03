# 逻辑网络 (Logical Networks)

OVN 的核心能力是创建与物理拓扑解耦的逻辑网络。

## 逻辑交换机 (Logical Switch)

- **功能**: 模拟传统的 L2 以太网交换机。
- **行为**: 支持单播转发、广播、组播。
- **隔离**: 每个逻辑交换机是一个独立的广播域。
- **端口 (Logical Switch Ports)**:
  - **VIF (Virtual Interface)**: 连接虚拟机或容器。
  - **Router Port**: 连接逻辑路由器。
  - **Localnet Port**: 连接物理网络（用于桥接）。

## 逻辑路由器 (Logical Router)

- **功能**: 模拟 L3 IP 路由器。
- **分布式特性**: OVN 的逻辑路由器是完全分布式的（Distributed Logical Router）。路由决策在源节点的 Hypervisor 上本地进行，避免了流量集中到单一节点产生的发夹（Hairpin）效应。
- **端口 (Logical Router Ports)**:
  - 连接逻辑交换机。
  - 连接其他逻辑路由器。
  - **Gateway Port**: 连接物理网络，提供 NAT 和 SNAT/DNAT 功能。

## 逻辑端口 (Logical Ports)

逻辑端口是逻辑网络的接入点。

- 它们存在于 NB DB 中。
- 它们通过 **Port Binding** 映射到特定的 Chassis 上。
- OVN 支持 **Port Security**（端口安全），防止 IP/MAC 欺骗。

## 逻辑流 (Logical Flows)

逻辑网络不是通过物理线缆连接的，而是通过 **逻辑流** 定义的。

- 逻辑流类似于 OpenFlow，但匹配的是逻辑字段（如 `logical_inport`, `ip.dst`），而不是物理端口。
- 所有的逻辑网络行为（L2 交换、L3 路由、ACL、LB）都被编译成 SB DB 中的 `Logical_Flow` 表项。
- 这种抽象使得 OVN 能够支持异构的物理环境，因为逻辑定义与物理实现是分离的。
