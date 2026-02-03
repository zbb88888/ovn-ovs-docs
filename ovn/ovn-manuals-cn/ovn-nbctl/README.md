# ovn-nbctl (Northbound DB CLI)

## 简介

`ovn-nbctl` 是用于配置 OVN 北向数据库 (Northbound DB) 的命令行工具。它是管理员与 OVN 交互的主要方式，用于创建逻辑交换机、路由器、ACL、负载均衡器等。

## 常用命令

### 逻辑交换机 (Logical Switch)

- `ovn-nbctl ls-add [switch]`: 创建逻辑交换机。
- `ovn-nbctl ls-list`: 列出所有逻辑交换机。
- `ovn-nbctl lsp-add <switch> <port>`: 在交换机上添加端口。
- `ovn-nbctl lsp-set-addresses <port> <mac> [ip]...`: 设置端口地址（MAC 和 IP）。
- `ovn-nbctl lsp-set-type <port> <type>`: 设置端口类型（如 `localnet`, `router`）。

### 逻辑路由器 (Logical Router)

- `ovn-nbctl lr-add [router]`: 创建逻辑路由器。
- `ovn-nbctl lrp-add <router> <port> <mac> <network>`: 添加路由器端口。
- `ovn-nbctl lr-route-add <router> <prefix> <nexthop> [port]`: 添加静态路由。
- `ovn-nbctl lr-nat-add <router> <type> <external_ip> <logical_ip>`: 添加 NAT 规则。

### 连接与互联

- `ovn-nbctl lsp-set-options <port> router-port=<peer>`: 连接交换机和路由器。

### 负载均衡 (Load Balancer)

- `ovn-nbctl lb-add <lb> <vip> <backend-ips>`: 创建负载均衡器。
- `ovn-nbctl ls-lb-add <switch> <lb>`: 将负载均衡器绑定到交换机。
- `ovn-nbctl lr-lb-add <router> <lb>`: 将负载均衡器绑定到路由器。

### 访问控制 (ACL)

- `ovn-nbctl acl-add <switch> <direction> <priority> <match> <action>`: 添加 ACL 规则。
  - direction: `to-lport` (出站), `from-lport` (入站)。
  - action: `allow`, `allow-related`, `drop`, `reject`。

## 通用数据库操作

`ovn-nbctl` 支持直接操作数据库表（类似于 SQL）：

- `ovn-nbctl list <table>`: 列出表中的所有记录。
- `ovn-nbctl find <table> <col>=<val>`: 查找记录。
- `ovn-nbctl get <table> <record> <col>`: 获取字段值。
- `ovn-nbctl set <table> <record> <col>=<val>`: 设置字段值。
