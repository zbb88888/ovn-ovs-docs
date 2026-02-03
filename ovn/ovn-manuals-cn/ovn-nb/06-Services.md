# 网络服务 (Network Services)

## DHCP_Options 表

定义 DHCPv4 和 DHCPv6 选项，由 OVN 原生响应，无需额外的 DHCP 代理。

### 核心列

- **cidr**: 该选项适用的子网 CIDR。
- **options**: DHCP 选项键值对。
  - `server_id`: DHCP 服务器 IP。
  - `server_mac`: DHCP 服务器 MAC。
  - `router`: 默认网关。
  - `dns_server`: DNS 服务器列表。
  - `domain_name`: 域名。
  - `lease_time`: 租约时间。

## DNS 表

定义内部 DNS 记录，允许 VM 通过主机名相互访问。

- **records**: 主机名到 IP 的映射（如 `"vm1.ovn": "10.0.0.2"`）。

## QoS 表

服务质量控制，应用于逻辑交换机端口。

- **priority**: 优先级。
- **match**: 匹配流量。
- **action**: 包含带宽限制参数。
  - `dscp`: 设置 DSCP 值。
- **bandwidth**: 带宽限制字典（rate, burst）。

## Mirror 表

端口镜像配置（SPAN/RSPAN）。

- **type**:
  - `local`: 镜像到本地端口。
  - `gre` / `erspan`: 镜像到远程隧道端点。
- **filter**: 过滤条件（入站/出站）。
