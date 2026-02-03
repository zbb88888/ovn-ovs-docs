# OVN Interconnection (ovn-ic)

## 简介

OVN Interconnection (OVN IC) 功能允许将多个独立的 OVN 部署（称为 Availability Zones, AZ）连接起来，形成一个更大的逻辑网络。

- **场景**：跨数据中心互联、大规模集群的分片扩展。
- **架构**：引入了两个新的全局数据库（IC NB 和 IC SB）以及一个新的守护进程 (`ovn-ic`)。

## 架构拓扑 (Architecture Topology)

### 管理平面拓扑 (Management Plane)

此图展示了全局 Interconnection 数据库如何通过 `ovn-ic` 守护进程与本地 Availability Zones (AZs) 交互。

```text
                                     +-------------------+
                                     |  IC Northbound DB | (Global Config)
                                     +---------+---------+
                                               |
                                               |
                                     +---------v---------+
                                     |  IC Southbound DB | (Global State)
                                     +---------+---------+
                                               |
                +------------------------------+------------------------------+
                |                                                             |
        +-------v-------+                                             +-------v-------+
        |  AZ 1 (Local) |                                             |  AZ 2 (Local) |
        |               |                                             |               |
        |   +-------+   |                                             |   +-------+   |
        |   | ovn-ic| <---------------------------------------------> | ovn-ic|   |
        |   +---+---+   |          (Inter-AZ Communication)           |   +---+---+   |
        |       |       |                                             |       |       |
        |   +---v---+   |                                             |   +---v---+   |
        |   | NB DB |   |                                             |   | NB DB |   |
        |   +---+---+   |                                             |   +---+---+   |
        |       |       |                                             |       |       |
        |   +---v---+   |                                             |   +---v---+   |
        |   | SB DB |   |                                             |   | SB DB |   |
        |   +-------+   |                                             |       +-------+
        +---------------+                                             +---------------+
```

### 数据平面拓扑 (Data Plane)

此图展示了逻辑互联拓扑。不同的 AZ 通过 **Transit Switch** 连接，Transit Switch 充当本地逻辑路由器之间的 L2 桥梁。

```text
             AZ 1                                     AZ 2
    +------------------+                    +------------------+
    |  Logical Router  |                    |  Logical Router  |
    |      (R1)        |                    |      (R2)        |
    |        |         |                    |        |         |
    |    +---+---+     |                    |    +---+---+     |
    |    |Transit|     |                    |    |Transit|     |
    |    |Switch |     |                    |    |Switch |     |
    |    |Port(P1)     |                    |    |Port(P2)     |
    |    +-------+     |                    |    +-------+     |
    |        |         |                    |        |         |
    +--------|---------+                    +--------|---------+
             |                                       |
             +-------------------+-------------------+
                                 |
                        +-----------------+
                        |  Transit Switch | (L2 Interconnect)
                        +-----------------+
```

## 核心组件

### ovn-ic (Interconnection Daemon)

运行在每个 AZ 的边缘节点或控制节点上。

- **职责**：作为本地 OVN 集群与全局 IC 数据库之间的同步代理。
- **数据流**：
    1. 监听全局 `IC_NB` 中的 Transit Switch 配置，在本地创建对应的逻辑交换机。
    2. 将本地的网关信息和路由发布到全局 `IC_SB`。
    3. 从全局 `IC_SB` 学习其他 AZ 的网关和路由，并导入本地 OVN。

### ovn-ic-nb (IC Northbound DB)

存储互联网络的逻辑配置。

- **Transit_Switch**: 定义跨 AZ 的逻辑交换机。这是连接不同 AZ 的 L2 桥梁。
- **IC_Global**: 全局配置。

### ovn-ic-sb (IC Southbound DB)

存储互联网络的运行时状态。

- **Availability_Zone**: 注册的可用区列表。
- **Gateway**: 各个 AZ 的出口网关信息（Chassis、Encap IP）。
- **Route**: 跨 AZ 的路由表。
- **Port_Binding**: Transit Switch 上的端口绑定信息。

## 工作原理

1. **Transit Switch**: 管理员在 IC NB 中创建一个 Transit Switch。
2. **同步**: 所有 AZ 的 `ovn-ic` 看到这个 Transit Switch，分别在各自的本地 NB DB 中创建一个同名的逻辑交换机，并将其标记为互联端口。
3. **隧道**: 不同 AZ 的网关通过隧道（通常是 Geneve）直接通信，封装流量经过 Transit Switch。
4. **路由**: `ovn-ic` 自动同步路由，使得 AZ A 中的 VM 可以通过路由指向 Transit Switch，进而到达 AZ B。
