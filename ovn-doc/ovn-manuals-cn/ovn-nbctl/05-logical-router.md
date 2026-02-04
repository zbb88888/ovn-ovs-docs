# OVN 逻辑路由器命令 (Logical Router Commands)

## 逻辑路由器管理

*   `lr-add`
    创建一个新的、未命名的逻辑路由器。

*   `[--may-exist | --add-duplicate] lr-add router`
    创建一个名为 `router` 的新逻辑路由器。

*   `[--if-exists] lr-del router`
    删除 `router`。

*   `lr-list`
    列出所有现有的路由器。

## 逻辑路由器端口命令 (Logical Router Port Commands)

*   `[--may-exist] lrp-add router port mac [network]... [peer=peer]`
    在 `router` 上创建一个名为 `port` 的新逻辑路由器端口。
    *   `mac`: 以太网地址。
    *   `network`: IP 地址/子网掩码（如 `192.168.0.1/24`）。
    *   `peer`: 连接到的对端逻辑路由器端口。

*   `[--if-exists] lrp-del port`
    删除 `port`。

*   `lrp-list router`
    列出 `router` 中的所有逻辑路由器端口。

*   `lrp-set-enabled port state`
    设置端口的管理状态（`enabled` 或 `disabled`）。

*   `lrp-get-enabled port`
    打印端口的管理状态。

*   `lrp-set-gateway-chassis port chassis [priority]`
    为 `port` 设置网关机箱。
    *   `chassis`: 机箱名称。
    *   `priority`: 0 到 32767。这在 `Gateway_Chassis` 表中创建条目。

*   `lrp-del-gateway-chassis port chassis`
    从 `port` 中删除网关机箱。

*   `lrp-get-gateway-chassis port`
    列出 `port` 的所有网关机箱及其优先级。

## 逻辑路由器静态路由命令 (Logical Router Static Route Commands)

*   `[--may-exist] [--policy=POLICY] [--route-table=ROUTE_TABLE] [--ecmp] [--ecmp-symmetric-reply] [--bfd[=UUID]] lr-route-add router prefix nexthop [port]`
    向 `router` 添加指定的路由。
    *   `prefix`: IPv4 或 IPv6 前缀（如 `192.168.100.0/24`）。
    *   `nexthop`: 下一跳网关 IP。可以设置为 `discard` 以丢弃数据包。
    *   `port`: 指定输出端口。
    *   `--route-table`: 分配到单独的路由表。
    *   `--policy`: `dst-ip` (默认) 或 `src-ip`。
    *   `--ecmp`: 允许添加具有相同前缀和策略但不同下一跳的路由（等价多路径路由）。
    *   `--ecmp-symmetric-reply`: 意味着 `--ecmp`，并确保流量通过同一条路由回复。
    *   `--bfd`: 将 BFD 会话链接到 OVN 路由。

*   `[--if-exists] [--policy=POLICY] [--route-table=ROUTE_TABLE] lr-route-del router [prefix [nexthop [port]]]`
    从 `router` 中删除路由。

*   `lr-route-list router`
    列出 `router` 上的路由。

## 逻辑路由器策略命令 (Logical Router Policy Commands)

*   `[--may-exist] [--bfd] [--route-table=ROUTE_TABLE] lr-policy-add router priority match action [nexthop[,nexthop,...]] [options key=value]]`
    向 `router` 添加策略路由。
    *   提供类似 ACL 的 permit/deny 策略，以及用于服务插入/链的 reroute 策略。
    *   `action`: `allow`, `drop`, `reroute`。
    *   `nexthop`: 当动作为 `reroute` 时需要。
    *   `options`: 支持 `pkt_mark`。

*   `[--if-exists] lr-policy-del router [{priority | uuid} [match]]`
    从 `router` 中删除策略。

*   `lr-policy-list router`
    列出 `router` 上的策略。

## NAT 命令 (NAT Commands)

*   `[--may-exist] [--stateless] [--gateway-port=GATEWAY_PORT] [-portrange] [--match=MATCH] [--priority=PRIORITY] lr-nat-add router type external_ip logical_ip [logical_port external_mac] [external_port_range]`
    向 `router` 添加 NAT 规则。
    *   `type`: `snat`, `dnat`, `dnat_and_snat`。
    *   `external_ip`: 外部可见的 IP。
    *   `logical_ip`: 内部逻辑 IP 或网络。
    *   `--stateless`: 不使用连接跟踪（仅适用于 `dnat_and_snat`）。
    *   `--gateway-port`: 指定应用 NAT 规则的分布式网关端口。
    *   `--portrange`: 指定外部端口范围。
    *   `logical_port` 和 `external_mac`: 仅当路由器为分布式路由器且类型为 `dnat_and_snat` 时接受。用于在特定机箱上编程 NAT 规则。

*   `[--if-exists] lr-nat-del router [type [ip] [gateway_port]]`
    从 `router` 中删除 NAT 规则。

*   `lr-nat-list router`
    列出 `router` 上的 NAT 规则。
