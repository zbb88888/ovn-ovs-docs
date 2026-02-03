# 安全策略 (Security Policies)

## ACL (Access Control List) 表

定义细粒度的流量控制规则。

### 核心列

- **priority**: 规则优先级（数字越大优先级越高）。
- **direction**:
  - `from-lport`: 入站（从逻辑端口发出，进入 OVN 网络）。
  - `to-lport`: 出站（从 OVN 网络发出，进入逻辑端口）。
- **match**: 匹配条件（如 `ip4.dst == 10.0.0.1 && tcp.dst == 80`）。
- **action**:
  - `allow`: 允许通过。
  - `allow-related`: 允许通过，并自动允许相关的响应流量（启用连接跟踪）。
  - `drop`: 丢弃。
  - `reject`: 拒绝并发送响应（如 TCP RST 或 ICMP Unreachable）。

## Address_Set 表

命名 IP 地址集合，用于简化 ACL 的编写和管理。

- **name**: 集合名称（如 `web_servers`）。
- **addresses**: IP 地址列表（支持 IPv4/IPv6 CIDR）。
- 在 ACL `match` 中引用：`ip4.dst == $web_servers`。

## Port_Group 表

逻辑端口的集合，用于将同一组 ACL 应用到多个端口，而无需在每个端口上重复配置。

- **name**: 组名称。
- **ports**: 包含的逻辑端口列表。
- **acls**: 应用于该组所有端口的 ACL 列表。
