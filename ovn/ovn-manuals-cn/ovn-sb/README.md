# OVN Southbound Database Schema (ovn-sb)

## 简介

`ovn-sb(5)` 文档描述了 OVN 南向数据库 (Southbound DB) 的数据库模式。

- **角色**：系统的中心数据交换点，连接逻辑控制面 (`ovn-northd`) 和物理数据面 (`ovn-controller`)。
- **内容**：包含物理节点信息 (Chassis)、逻辑流表 (Logical Flows) 以及逻辑到物理的绑定关系 (Bindings)。

## 文档目录

本目录下的文档将 `ovn-sb(5)` 的内容按功能模块进行了拆分：

1. **[01-Global-Config.md](./01-Global-Config.md)**
   - `SB_Global`: 全局状态与配置。
   - `Connection`, `SSL`: 数据库连接配置。
   - `RBAC_Role`, `RBAC_Permission`: 基于角色的访问控制。

2. **[02-Physical-Infrastructure.md](./02-Physical-Infrastructure.md)**
   - `Chassis`: 物理底盘/节点信息。
   - `Encap`: 隧道封装配置。
   - `Chassis_Private`: 底盘私有状态。

3. **[03-Logical-Pipeline.md](./03-Logical-Pipeline.md)**
   - `Logical_Flow`: 逻辑流表（核心）。
   - `Multicast_Group`: 组播组。
   - `Datapath_Binding`: 逻辑数据路径绑定。

4. **[04-Binding-State.md](./04-Binding-State.md)**
   - `Port_Binding`: 端口绑定状态（逻辑端口 -> 物理位置）。
   - `MAC_Binding`: ARP/邻居缓存。
   - `Static_MAC_Binding`: 静态 MAC 绑定。

5. **[05-Services-and-Monitoring.md](./05-Services-and-Monitoring.md)**
   - `DHCP_Options`, `DNS`: 网络服务。
   - `Load_Balancer`, `Service_Monitor`: 负载均衡状态。
   - `BFD`: 双向转发检测状态。
   - `Controller_Event`: 控制器事件。
