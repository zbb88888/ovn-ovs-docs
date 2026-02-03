# 逻辑路由器 (Logical Router)

## Logical_Router 表

每个记录代表一个逻辑路由器（L3 转发域）。

### 核心列

- **name**: 路由器名称。
- **ports**: 指向 `Logical_Router_Port` 表的引用列表。
- **static_routes**: 指向 `Logical_Router_Static_Route` 表的引用列表。
- **policies**: 指向 `Logical_Router_Policy` 表的引用列表。
- **nat**: 指向 `NAT` 表的引用列表。
- **load_balancer**: 应用于该路由器的负载均衡器列表。

## Logical_Router_Port 表

定义逻辑路由器的接口。

### 核心列

- **name**: 接口名称。
- **networks**: 该接口的 IP 地址和子网掩码（如 `192.168.1.1/24`）。
- **mac**: 接口的 MAC 地址。
- **gateway_chassis**: (可选) 指定该端口绑定到的特定网关底盘（用于集中式路由或连接物理网）。

## NAT 表

定义网络地址转换规则。

### type (NAT 类型)

- **dnat**: 目的地址转换（通常用于 Floating IP）。
- **snat**: 源地址转换（通常用于访问外网）。
- **dnat_and_snat**: 同时进行源和目的转换（1:1 映射）。

### 核心列

- **external_ip**: 外部（公网）IP。
- **logical_ip**: 内部（逻辑）IP 或网段。
- **logical_port**: (可选) 仅对特定端口生效。

## Logical_Router_Static_Route 表

静态路由配置。

- **ip_prefix**: 目标网段 (CIDR)。
- **nexthop**: 下一跳 IP 地址。
- **output_port**: (可选) 指定出接口。

## Logical_Router_Policy 表

策略路由 (PBR)，优先级高于标准静态路由。

- **priority**: 优先级。
- **match**: 匹配条件（使用 OVN 匹配语法，如 `ip4.src == 10.0.0.5`）。
- **action**:
  - `allow`: 允许（继续常规路由）。
  - `drop`: 丢弃。
  - `reroute`: 重路由到指定的 `nexthop`。
