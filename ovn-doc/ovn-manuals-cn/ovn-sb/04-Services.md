# DHCP_Options 表

此表中的每一行存储原生 OVN DHCP 支持的 DHCP 选项。`ovn-northd` 使用支持的 DHCP 选项填充此表。`ovn-controller` 查找此表以获取在 "put_dhcp_opts" 操作中定义的 DHCP 选项的 DHCP 代码。有关可以在此处定义的 DHCP 选项的可能列表，请参阅 RFC 2132 "https://tools.ietf.org/html/rfc2132"。

## 汇总

*   `name`
    *   字符串
*   `code`
    *   整数，范围 0 到 254
*   `type`
    *   字符串，`bool`, `domains`, `host_id`, `ipv4`, `static_routes`, `str`, `uint16`, `uint32`, 或 `uint8` 之一

## 详细信息

*   `name`: 字符串
    *   DHCP 选项的名称。
    *   例如：`name="router"`

*   `code`: 整数，范围 0 到 254
    *   RFC 2132 中定义的 DHCP 选项的 DHCP 选项代码。
    *   例如：`code=3`

*   `type`: 字符串，`bool`, `domains`, `host_id`, `ipv4`, `static_routes`, `str`, `uint16`, `uint32`, 或 `uint8` 之一
    *   DHCP 选项代码的数据类型。

    *   `value: bool`
        *   这表示 DHCP 选项的值是一个布尔值。
        *   例如：`"name=ip_forward_enable", "code=19", "type=bool"`。
        *   `put_dhcp_opts(..., ip_forward_enable = 1,...)`

    *   `value: uint8`
        *   这表示 DHCP 选项的值是一个无符号 int8（8 位）。
        *   例如：`"name=default_ttl", "code=23", "type=uint8"`。
        *   `put_dhcp_opts(..., default_ttl = 50,...)`

    *   `value: uint16`
        *   这表示 DHCP 选项的值是一个无符号 int16（16 位）。
        *   例如：`"name=mtu", "code=26", "type=uint16"`。
        *   `put_dhcp_opts(..., mtu = 1450,...)`

    *   `value: uint32`
        *   这表示 DHCP 选项的值是一个无符号 int32（32 位）。
        *   例如：`"name=lease_time", "code=51", "type=uint32"`。
        *   `put_dhcp_opts(..., lease_time = 86400,...)`

    *   `value: ipv4`
        *   这表示 DHCP 选项的值是一个 IPv4 地址或地址。
        *   例如：`"name=router", "code=3", "type=ipv4"`。
        *   `put_dhcp_opts(..., router = 10.0.0.1,...)`
        *   例如：`"name=dns_server", "code=6", "type=ipv4"`。
        *   `put_dhcp_opts(..., dns_server = {8.8.8.8 7.7.7.7},...)`

    *   `value: static_routes`
        *   这表示 DHCP 选项的值包含一对 IPv4 路由和下一跳地址。
        *   例如：`"name=classless_static_route", "code=121", "type=static_routes"`。
        *   `put_dhcp_opts(..., classless_static_route = {30.0.0.0/24,10.0.0.4,0.0.0.0/0,10.0.0.1}...)`

    *   `value: str`
        *   这表示 DHCP 选项的值是一个字符串。
        *   例如：`"name=host_name", "code=12", "type=str"`。

    *   `value: host_id`
        *   这表示 DHCP 选项的值是一个 host_id。它可以是 host_name 或 IP 地址。
        *   例如：`"name=tftp_server", "code=66", "type=host_id"`。

    *   `value: domains`
        *   这表示 DHCP 选项的值是一个域名或域名的逗号分隔列表。
        *   例如：`"name=domain_search_list", "code=119", "type=domains"`。

# DHCPv6_Options 表

此表中的每一行存储原生 OVN DHCPv6 支持的 DHCPv6 选项。`ovn-northd` 使用支持的 DHCPv6 选项填充此表。`ovn-controller` 查找此表以获取在 `put_dhcpv6_opts` 操作中定义的 DHCPv6 选项的 DHCPv6 代码。有关可以在此处定义的 DHCPv6 选项的列表，请参阅 RFC 3315 和 RFC 3646。

## 汇总

*   `name`
    *   字符串
*   `code`
    *   整数，范围 0 到 254
*   `type`
    *   字符串，`domain`, `ipv6`, `mac`, 或 `str` 之一

## 详细信息

*   `name`: 字符串
    *   DHCPv6 选项的名称。
    *   例如：`name="ia_addr"`

*   `code`: 整数，范围 0 到 254
    *   相应 RFC 中定义的 DHCPv6 选项的 DHCPv6 选项代码。
    *   例如：`code=3`

*   `type`: 字符串，`domain`, `ipv6`, `mac`, 或 `str` 之一
    *   DHCPv6 选项代码的数据类型。

    *   `value: ipv6`
        *   这表示 DHCPv6 选项的值是一个 IPv6 地址（或多个）。
        *   例如：`"name=ia_addr", "code=5", "type=ipv6"`。
        *   `put_dhcpv6_opts(..., ia_addr = ae70::4,...)`

    *   `value: str`
        *   这表示 DHCPv6 选项的值是一个字符串。
        *   例如：`"name=domain_search", "code=24", "type=str"`。
        *   `put_dhcpv6_opts(..., domain_search = ovn.domain,...)`

    *   `value: mac`
        *   这表示 DHCPv6 选项的值是一个 MAC 地址。
        *   例如：`"name=server_id", "code=2", "type=mac"`。
        *   `put_dhcpv6_opts(..., server_id = 01:02:03:04L05:06,...)`

# DNS 表

此表中的每一行存储 DNS 记录。OVN 操作 `dns_lookup` 使用此表进行 DNS 解析。

## 汇总

*   `records`
    *   字符串对的映射
*   `datapaths`
    *   1 个或多个 `Datapath_Binding` 的集合
*   `options : ovn-owned`
    *   可选字符串
*   通用列:
    *   `external_ids`
        *   字符串对的映射

## 详细信息

*   `records`: 字符串对的映射
    *   DNS 记录的键值对，以 DNS 查询名称为键，以逗号或空格分隔的 IP 地址字符串为值。`ovn-northd` 以全小写形式存储 DNS 查询名称，以便于不区分大小写的查找。
    *   例如：`"vm1.ovn.org" = "10.0.0.4 aef0::4"`

*   `datapaths`: 1 个或多个 `Datapath_Binding` 的集合
    *   `records` 列中定义的 DNS 记录将仅应用于源自此列中定义的数据路径的 DNS 查询。

*   `options : ovn-owned`: 可选字符串
    *   此列指示此表中的所有域都由 OVN 拥有，并且对这些域的所有 DNS 查询都将由 IP 地址或 DNS 拒绝在本地回答。

*   **通用列**:

    *   `external_ids`: 字符串对的映射
        *   请参阅本文档开头的“外部 ID”。

# Service_Monitor 表

此表中的每一行配置监视服务的活跃度。服务可以是 IPv4 TCP 或 UDP 服务。`ovn-controller` 定期发送服务监视数据包并更新服务的状态。如果服务监视器不是本地的，则 `ovn-ic` 会在 SBDB 数据库中创建一个关于它的记录，之后另一个 OVN 部署会在其 SBDB 中创建一个关于服务监视器的记录，并且状态会传播回原始可用性区域中的初始记录。

`ovn-northd` 使用此功能来实现通过北向数据库向 CMS 提供的负载均衡器健康检查功能。

## 汇总

*   配置:
    *   `type`
        *   可选字符串，`load-balancer` 或 `network-function`
    *   `ip`
        *   字符串
    *   `mac`
        *   字符串
    *   `protocol`
        *   可选字符串，`icmp`, `tcp`, 或 `udp` 之一
    *   `port`
        *   整数，范围 0 到 65,535
    *   `logical_input_port`
        *   字符串
    *   `logical_port`
        *   字符串
    *   `src_mac`
        *   字符串
    *   `src_ip`
        *   字符串
    *   `chassis_name`
        *   字符串
    *   `remote`
        *   布尔值
    *   `ic_learned`
        *   布尔值
    *   `options : interval`
        *   可选字符串，包含整数
    *   `options : timeout`
        *   可选字符串，包含整数
    *   `options : success_count`
        *   可选字符串，包含整数
    *   `options : failure_count`
        *   可选字符串，包含整数
*   状态报告:
    *   `status`
        *   可选字符串，`error`, `offline`, 或 `online` 之一
*   通用列:
    *   `external_ids`
        *   字符串对的映射

## 详细信息

*   **配置**:

    *   `ovn-northd` 设置这些列和值以配置服务监视器。

    *   `type`: 可选字符串，`load-balancer` 或 `network-function`
        *   服务的类型。支持的值是 "load-balancer" 和 "network-function"。

    *   `ip`: 字符串
        *   监视数据包中使用的目标 IP。对于负载均衡器服务，这是要监视的服务的 IP。对于网络功能服务，此 IP 用于通过关联的网络功能端口对发送探测数据包。

    *   `mac`: 字符串
        *   监视数据包中使用的目标 MAC 地址。这仅适用于网络功能服务。

    *   `protocol`: 可选字符串，`icmp`, `tcp`, 或 `udp` 之一
        *   服务的协议。对于 `load-balancer` 类型，支持的协议是 `tcp` 和 `udp`。对于 `network-function` 类型，支持的协议是 `icmp`，并且通过将 icmp 回显请求数据包注入到 `Network_Function` 的 `inport` 然后监视从 `outport` 出来的同一数据包来进行健康探测。

    *   `port`: 整数，范围 0 到 65,535
        *   服务的 TCP 或 UDP 端口。

    *   `logical_input_port`: 字符串
        *   这仅适用于网络功能类型。必须发送监视数据包的逻辑端口的 VIF。绑定此 `logical_port` 的 `ovn-controller` 通过发送定期监视数据包来监视服务。

    *   `logical_port`: 字符串
        *   运行服务的逻辑端口的 VIF。绑定此 `logical_port` 的 `ovn-controller` 通过发送定期监视数据包来监视服务。对于负载均衡器，这是发送监视数据包并从中接收响应数据包的端口。对于网络功能，这是接收转发的监视数据包的端口。

    *   `src_mac`: 字符串
        *   在服务监视数据包中使用的源以太网地址。

    *   `src_ip`: 字符串
        *   在服务监视数据包中使用的源 IPv4 或 IPv6 地址。

    *   `chassis_name`: 字符串
        *   绑定逻辑端口的机箱的名称。

    *   `remote`: 布尔值
        *   如果后端位于远程 OVN 部署上，则设置为 true。

    *   `ic_learned`: 布尔值
        *   如果服务监视器是通过 `ovn-ic` 管理从另一个 OVN 部署传播的，则设置为 true。

    *   `options : interval`: 可选字符串，包含整数
        *   服务监视器检查之间的间隔，以秒为单位。

    *   `options : timeout`: 可选字符串，包含整数
        *   服务监视器检查超时的时间，以秒为单位。

    *   `options : success_count`: 可选字符串，包含整数
        *   服务被认为在线之前的成功检查次数。

    *   `options : failure_count`: 可选字符串，包含整数
        *   服务被认为离线之前的失败检查次数。

*   **状态报告**:

    *   托管 `logical_port` 的机箱上的 `ovn-controller` 更新此列以报告服务的状态。

    *   `status`: 可选字符串，`error`, `offline`, 或 `online` 之一
        *   对于 TCP 服务，`ovn-controller` 向服务发送 SYN 并期望 ACK 响应以认为服务在线。
        *   对于 UDP 服务，`ovn-controller` 向服务发送 UDP 数据包并且不期望任何回复。如果它收到 ICMP 回复，则它认为服务离线。

*   **通用列**:

    *   `external_ids`: 字符串对的映射
        *   请参阅本文档开头的“外部 ID”。

# IP_Multicast 表

IP 多播配置选项。目前仅适用于 IGMP。

## 汇总

*   `datapath`
    *   `Datapath_Binding` 的弱引用 (在表中必须唯一)
*   `enabled`
    *   可选布尔值
*   `querier`
    *   可选布尔值
*   `table_size`
    *   可选整数
*   `idle_timeout`
    *   可选整数
*   `query_interval`
    *   可选整数
*   `seq_no`
    *   整数
*   查询器配置选项:
    *   `eth_src`
        *   字符串
    *   `ip4_src`
        *   字符串
    *   `ip6_src`
        *   字符串
    *   `query_max_resp`
        *   可选整数

## 详细信息

*   `datapath`: `Datapath_Binding` 的弱引用 (在表中必须唯一)
    *   为其定义这些配置选项的 `Datapath_Binding` 条目。

*   `enabled`: 可选布尔值
    *   启用/禁用多播侦听。默认值：禁用。

*   `querier`: 可选布尔值
    *   启用/禁用多播查询。如果启用，则默认启用多播查询。

*   `table_size`: 可选整数
    *   限制可以学习的多播组的数量。默认值：每个数据路径 2048 个组。

*   `idle_timeout`: 可选整数
    *   如果启用了多播侦听，则配置 IP 多播组的空闲超时（以秒为单位）。默认值：300 秒。

*   `query_interval`: 可选整数
    *   如果启用了侦听和查询器，则配置发送多播查询的间隔（以秒为单位）。默认值：`idle_timeout`/2 秒。

*   `seq_no`: 整数
    *   `ovn-controller` 读取此值，并在检测到 `seq_no` 更改时刷新所有学习到的多播组。

*   **查询器配置选项**:

    *   在 OVN 管理程序节点上运行的 `ovn-controller` 进程使用以下列来确定它发起的 IGMP/MLD 查询中的字段值：

    *   `eth_src`: 字符串
        *   源以太网地址。

    *   `ip4_src`: 字符串
        *   源 IPv4 地址。

    *   `ip6_src`: 字符串
        *   源 IPv6 地址。

    *   `query_max_resp`: 可选整数
        *   在多播查询中用作“最大响应”字段的值（以秒为单位）。默认值：1 秒。

# IGMP_Group 表

包含按地址/数据路径/机箱索引的学习到的 IGMP 组。

## 汇总

*   `address`
    *   字符串
*   `protocol`
    *   字符串
*   `datapath`
    *   可选的 `Datapath_Binding` 弱引用
*   `chassis`
    *   可选的 `Chassis` 弱引用
*   `ports`
    *   `Port_Binding` 的弱引用集合
*   `chassis_name`
    *   字符串

## 详细信息

*   `address`: 字符串
    *   IGMP 组的目标 IPv4 地址。

*   `protocol`: 字符串
    *   组协议版本，IGMPv1,v2,v3 或 MLDv1,v2。

*   `datapath`: 可选的 `Datapath_Binding` 弱引用
    *   此 IGMP 组所属的数据路径。

*   `chassis`: 可选的 `Chassis` 弱引用
    *   此 IGMP 组所属的机箱。

*   `ports`: `Port_Binding` 的弱引用集合
    *   此 IGMP 组的目标端口绑定。

*   `chassis_name`: 字符串
    *   插入此记录的机箱。此列仅用于 RBAC 目的。
