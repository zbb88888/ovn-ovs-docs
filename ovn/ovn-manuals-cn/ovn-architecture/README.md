# OVN 架构文档 (中文版)

本目录包含 [`ovn-architecture(7)`](https://man7.org/linux/man-pages/man7/ovn-architecture.7.html) 手册页的中文梳理文档。文档按照 OVN 的资源模型和核心组件进行了拆分，以便于查阅和理解。

## 文档索引

### 核心架构

- **[01-Architecture-Overview.md](./01-Architecture-Overview.md)**
  - OVN 的总体设计目标
  - 核心组件交互图 (ASCII Diagram)
  - 物理网络与逻辑网络的映射关系
  - 系统组件：CMS, Northbound DB, Southbound DB, ovn-northd, ovn-controller

### 数据库资源

- **[02-Northbound-DB.md](./02-Northbound-DB.md)**
  - 北向数据库的角色
  - 逻辑资源定义：Logical Switch, Logical Router, ACL, Load Balancer
- **[03-Southbound-DB.md](./03-Southbound-DB.md)**
  - 南向数据库的角色
  - 数据分类：Physical Data (Chassis), Logical Data (Logical Flows), Binding Data (Port Binding)

### 物理节点

- **[04-Chassis.md](./04-Chassis.md)**
  - Chassis (传输节点) 的定义与职责
  - 隧道封装协议 (Geneve, STT, VXLAN)
  - 网关 (Gateways) 类型

### 逻辑网络与数据流

- **[05-Logical-Networks.md](./05-Logical-Networks.md)**
  - 逻辑交换机与逻辑路由器的详细行为
  - 逻辑端口与逻辑流 (Logical Flows) 的抽象
- **[06-Packet-Lifecycle.md](./06-Packet-Lifecycle.md)**
  - 数据包生命周期 ASCII 流程图
  - 详细的流水线处理：Ingress Pipeline -> Tunnel -> Egress Pipeline
- **[07-Security-and-LB.md](./07-Security-and-LB.md)**
  - 分布式 ACL 与连接跟踪 (Conntrack)
  - 负载均衡 (Load Balancing) 的实现
  - 服务质量 (QoS)

## 参考来源

- [ovn-architecture(7) - Linux manual page](https://man7.org/linux/man-pages/man7/ovn-architecture.7.html)
- OVN GitHub Repository
