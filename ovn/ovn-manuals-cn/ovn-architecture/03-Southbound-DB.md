# OVN Southbound Database (SB DB)

## 角色与功能

OVN Southbound Database (SB DB) 是系统的中心，连接了逻辑控制平面（ovn-northd）和物理数据平面（ovn-controller）。它具有其它组件通常不具备的全局系统视图。

## 数据分类

SB DB 中的数据主要分为三类：

### 1. 物理数据 (Physical Data)

由 `ovn-controller` 写入，描述物理基础设施。

- **Chassis**: 代表一个物理节点（Hypervisor 或 Gateway）。包含主机名、IP 地址、支持的封装类型（Geneve, VXLAN 等）。
- **Encap**: 定义 Chassis 之间通信的隧道配置。

### 2. 逻辑数据 (Logical Data)

由 `ovn-northd` 根据 NB DB 计算得出。

- **Datapath_Binding**: 代表一个逻辑数据路径（逻辑交换机或逻辑路由器）。
- **Logical_Flow**: 核心表。包含平台无关的逻辑流表。这些流表定义了逻辑网络行为（如“如果包发往 MAC A，则输出到端口 B”），但不包含具体的物理端口号或 OpenFlow 表 ID。

### 3. 绑定数据 (Binding Data)

连接逻辑与物理的桥梁。

- **Port_Binding**: 记录逻辑端口（Logical Port）当前绑定在哪个 Chassis 上。
  - 当 VM 在某个节点启动时，该节点的 `ovn-controller` 会更新此表，声明“我拥有这个逻辑端口”。
  - 这使得 `ovn-northd` 和其他 `ovn-controller` 知道该逻辑端口的物理位置。

## 性能与扩展性

由于所有 `ovn-controller` 都需要连接 SB DB，SB DB 往往是大规模集群的瓶颈。

- **ovsdb-server** 支持集群模式（Raft）以提高可用性。
- `ovn-controller` 通过条件监控（Conditional Monitoring）只订阅与本地相关的 SB DB 数据，以减少负载。
