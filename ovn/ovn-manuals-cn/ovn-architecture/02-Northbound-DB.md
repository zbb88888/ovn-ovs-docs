# OVN Northbound Database (NB DB)

## 角色与功能

OVN Northbound Database (NB DB) 是 OVN 与云管理系统 (CMS) 之间的接口。它存储了逻辑网络的配置信息，这些信息由 CMS 直接写入。

## 关键资源

### Logical Switch (逻辑交换机)

- 代表一个 L2 广播域。
- 类似于物理交换机或 VLAN。
- 包含多个 Logical Switch Ports。

### Logical Router (逻辑路由器)

- 代表一个 L3 路由域。
- 连接多个 Logical Switches 或其他 Logical Routers。
- 支持静态路由、策略路由等。

### ACL (访问控制列表)

- 定义安全策略。
- 应用于 Logical Switch Ports。
- 支持入站 (Ingress) 和出站 (Egress) 规则。
- 支持有状态 (Stateful) 和无状态 (Stateless) 过滤。

### Load Balancer (负载均衡器)

- 定义 VIP (Virtual IP) 到后端 IP 的映射。
- 支持 TCP/UDP 负载均衡。

### DHCP Options

- 定义 DHCP 选项，由 OVN 原生响应 DHCP 请求，无需专门的 DHCP 代理容器。

## 交互模式

- **写入者**：CMS (如 OpenStack Neutron, OVN-Kubernetes)。
- **读取者**：ovn-northd。
- NB DB 应该反映用户期望的“逻辑”状态，而不包含任何物理基础设施的细节（如 VM 运行在哪个物理节点上）。
