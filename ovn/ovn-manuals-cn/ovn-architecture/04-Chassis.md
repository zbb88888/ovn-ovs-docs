# Chassis (传输节点)

## 定义

在 OVN 架构中，**Chassis** 指的是任何运行 `ovn-controller` 并参与数据包转发的物理或虚拟节点。这通常包括：

- **Hypervisors**: 运行虚拟机的物理服务器。
- **Gateways**: 提供逻辑网络与物理网络连接的网关节点。
- **Container Hosts**: 运行容器的节点。

## 组件与职责

每个 Chassis 必须运行：

1. **Open vSwitch (OVS)**: 负责实际的数据包交换。
2. **ovn-controller**: OVN 的本地代理。

### ovn-controller 的职责

- **连接 SB DB**: 读取逻辑流 (Logical Flows) 和端口绑定信息。
- **配置 OVS**: 通过 OpenFlow 和 OVSDB 配置本地 OVS 实例。
- **状态上报**: 将 Chassis 的状态（IP、系统信息）和端口绑定状态（Port Claims）写入 SB DB。
- **物理-逻辑转换**: 实现 Geneve/VXLAN 隧道封装与解封装，将逻辑网络映射到物理网络。

## 封装 (Encapsulation)

Chassis 之间通过隧道技术通信，以承载逻辑网络流量。OVN 支持多种封装协议，记录在 SB DB 的 `Encap` 表中：

### Geneve (Generic Network Virtualization Encapsulation)

- **推荐协议**。
- 支持 TLV (Type-Length-Value) 字段，允许 OVN 在隧道头中携带丰富的逻辑元数据（如 Logical Datapath ID, Logical Inport/Outport ID）。
- 这使得接收端 Chassis 无需重新查找数据库即可直接处理数据包。

### STT (Stateless Transport Tunneling)

- 类似于 Geneve，也支持携带元数据。
- 利用 TCP 卸载功能提高性能，但不是标准协议。

### VXLAN

- 标准协议，广泛支持。
- **限制**: 头部空间有限（仅 24-bit VNI），无法携带 OVN 所需的所有逻辑元数据。
- OVN 使用 VXLAN 通常用于与非 OVN 设备（如硬件交换机 TOR）互操作。

## 网关 (Gateways)

Chassis 可以被配置为网关，用于处理进出逻辑网络的流量（North-South Traffic）。

- **L2 Gateway**: 将逻辑交换机桥接到物理 VLAN。
- **L3 Gateway**: 提供 NAT 和路由功能，连接逻辑路由器和物理网络。
- **Distributed Gateway Port**: 分布式网关端口，允许流量在本地处理或重定向到特定的网关 Chassis。
