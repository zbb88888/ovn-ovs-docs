# 安全与负载均衡 (Security and Load Balancing)

## 访问控制列表 (ACLs)

OVN 提供了原生的分布式防火墙功能，通过 ACL 实现。

### 特性

- **分布式**: ACL 规则被推送到所有相关的计算节点，在数据包产生的源头或目的地直接处理，无需经过中心防火墙。
- **连接跟踪 (Connection Tracking)**:
  - OVN 利用 OVS 的 `conntrack` 模块实现有状态防火墙。
  - 允许配置如 `allow-related` 的规则，自动允许已建立连接的返回流量。
- **粒度**: 可以应用到逻辑交换机端口或端口组（Port Groups）。
- **方向**:
  - `to-lport`: 出站流量（发往 VM）。
  - `from-lport`: 入站流量（来自 VM）。

### 性能优化

- **Tiered ACLs**: 支持优先级分层。
- **Address Sets**: 允许使用 IP 地址集合代替单个 IP，减少流表数量，提高匹配效率。

## 负载均衡 (Load Balancing)

OVN 支持 L3/L4 负载均衡。

### 实现方式

- **分布式 VIP**: 负载均衡的 Virtual IP (VIP) 在所有相关的 Chassis 上都是可达的。
- **NAT 转换**: 当 VM 访问 VIP 时，本地 OVS 使用 DNAT 将 VIP 转换为后端实例的真实 IP。
- **健康检查**: OVN 可以配置简单的健康检查，剔除不健康的后端。

### 使用场景

- **Kubernetes Service**: OVN-Kubernetes 广泛使用 OVN LB 来实现 K8s Service (ClusterIP, NodePort)。
- **OpenStack Octavia**: 可以通过 OVN 驱动实现。

## 服务质量 (QoS)

OVN 支持在逻辑交换机端口上配置 QoS 规则。

- **DSCP 标记**: 修改 IP 报文的 DSCP 字段。
- **带宽限制**: 限制端口的入站和出站带宽（Ingress/Egress Policing）。
