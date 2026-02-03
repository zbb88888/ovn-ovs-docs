# OVN Northbound Database Schema (ovn-nb)

## 简介

`ovn-nb(5)` 文档描述了 OVN 北向数据库 (Northbound DB) 的数据库模式。

- **角色**：作为 OVN 与云管理系统 (CMS, 如 OpenStack/Kubernetes) 之间的接口。
- **内容**：存储逻辑网络配置（交换机、路由器、ACL、负载均衡等），不包含物理基础设施细节。

## 文档目录

本目录下的文档将 `ovn-nb(5)` 的内容按功能模块进行了拆分：

1. **[01-Global-Config.md](./01-Global-Config.md)**
   - `NB_Global`: 全局配置表。
   - `Connection`, `SSL`: 数据库连接与安全配置。

2. **[02-Logical-Switch.md](./02-Logical-Switch.md)**
   - `Logical_Switch`: 逻辑交换机定义。
   - `Logical_Switch_Port`: 逻辑交换机端口。
   - `Forwarding_Group`: 转发组。

3. **[03-Logical-Router.md](./03-Logical-Router.md)**
   - `Logical_Router`: 逻辑路由器定义。
   - `Logical_Router_Port`: 逻辑路由器端口。
   - `Logical_Router_Static_Route`: 静态路由。
   - `NAT`: 网络地址转换。
   - `Logical_Router_Policy`: 策略路由。

4. **[04-Security-Policies.md](./04-Security-Policies.md)**
   - `ACL`: 访问控制列表。
   - `Address_Set`: 地址集合。
   - `Port_Group`: 端口组。

5. **[05-Load-Balancer.md](./05-Load-Balancer.md)**
   - `Load_Balancer`: 负载均衡器核心配置。
   - `Load_Balancer_Health_Check`: 健康检查。
   - `Load_Balancer_VIP`: 虚拟 IP 配置。

6. **[06-Services.md](./06-Services.md)**
   - `DHCP_Options`: DHCP 选项。
   - `DNS`: 内部 DNS 记录。
   - `QoS`: 服务质量。
   - `Mirror`: 端口镜像。

7. **[07-HA-and-Gateway.md](./07-HA-and-Gateway.md)**
   - `HA_Chassis_Group` / `HA_Chassis`: 高可用底盘组。
   - `Gateway_Chassis`: 网关底盘配置。
   - `BFD`: 双向转发检测。
