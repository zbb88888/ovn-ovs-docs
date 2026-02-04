# Logical_Switch 表

此表中的每一行代表一个 L2 逻辑交换机。

有两种逻辑交换机，即完全虚拟化网络的（覆盖逻辑交换机）和提供到物理网络简单连接的（桥接逻辑交换机）。它们在提供同一机箱上的逻辑端口之间的连接时工作方式相同，但在连接远程逻辑端口时工作方式不同。覆盖逻辑交换机通过隧道连接远程逻辑端口，而桥接逻辑交换机通过本地网络端口的帮助将数据包桥接到直接连接的物理 L2 网段，从而提供到远程端口的连接。每个桥接逻辑交换机都有一个或多个 `localnet` 端口，这些端口只有一个特殊地址 `unknown`。

## 汇总

*   `ports`
    *   `Logical_Switch_Port` 集合
*   `load_balancer`
    *   `Load_Balancer` 的弱引用集合
*   `load_balancer_group`
    *   `Load_Balancer_Group` 集合
*   `acls`
    *   `ACL` 集合
*   `qos_rules`
    *   `QoS` 集合
*   `dns_records`
    *   `DNS` 的弱引用集合
*   `forwarding_groups`
    *   `Forwarding_Group` 集合
*   命名:
    *   `name`
        *   字符串
    *   `external_ids : neutron:network_name`
        *   可选字符串
    *   `other_config : dynamic-routing-bridge-ifname`
        *   可选字符串
    *   `other_config : dynamic-routing-vxlan-ifname`
        *   可选字符串
    *   `other_config : dynamic-routing-advertise-ifname`
        *   可选字符串
*   IP 地址分配:
    *   `other_config : subnet`
        *   可选字符串
    *   `other_config : exclude_ips`
        *   可选字符串
    *   `other_config : ipv6_prefix`
        *   可选字符串
    *   `other_config : dhcp_relay_port`
        *   可选字符串
    *   `other_config : mac_only`
        *   可选字符串，`true` 或 `false`
    *   `other_config : fdb_age_threshold`
        *   可选字符串，包含整数，范围 0 到 4,294,967,295
    *   `other_config : ct-zone-limit`
        *   可选字符串，包含整数，范围 0 到 4,294,967,295
    *   `other_config : enable-stateless-acl-with-lb`
        *   可选字符串，`true` 或 `false`
    *   `other_config : dynamic-routing-vni`
        *   可选字符串，包含整数，范围 0 到 16,777,215
    *   `other_config : dynamic-routing-fdb-prefer-local`
        *   可选字符串，`true` 或 `false`
    *   `other_config : dynamic-routing-arp-prefer-local`
        *   可选字符串，`true` 或 `false`
    *   `other_config : dynamic-routing-redistribute`
        *   可选字符串
*   IP 多播侦听选项:
    *   `other_config : mcast_snoop`
        *   可选字符串，`true` 或 `false`
    *   `other_config : mcast_querier`
        *   可选字符串，`true` 或 `false`
    *   `other_config : mcast_flood_unregistered`
        *   可选字符串，`true` 或 `false`
    *   `other_config : mcast_table_size`
        *   可选字符串，包含整数，范围 1 到 32,766
    *   `other_config : mcast_idle_timeout`
        *   可选字符串，包含整数，范围 15 到 3,600
    *   `other_config : mcast_query_interval`
        *   可选字符串，包含整数，范围 1 到 3,600
    *   `other_config : mcast_query_max_response`
        *   可选字符串，包含整数，范围 1 到 10
    *   `other_config : mcast_eth_src`
        *   可选字符串
    *   `other_config : mcast_ip4_src`
        *   可选字符串
    *   `other_config : mcast_ip6_src`
        *   可选字符串
*   互连:
    *   `other_config : interconn-ts`
        *   可选字符串
    *   `other_config : ic-vxlan_mode`
        *   可选字符串，`true` 或 `false`
*   隧道密钥:
    *   `other_config : requested-tnl-key`
        *   可选字符串，包含整数，范围 1 到 16,777,215
*   `copp`
    *   可选的 `Copp` 弱引用
*   其他选项:
    *   `other_config : vlan-passthru`
        *   可选字符串，`true` 或 `false`
    *   `other_config : broadcast-arps-to-all-routers`
        *   可选字符串，`true` 或 `false`
*   通用列:
    *   `external_ids`
        *   字符串对的映射

## 详细信息

*   `ports`: `Logical_Switch_Port` 集合
    *   连接到逻辑交换机的逻辑端口。
    *   多个逻辑交换机包含同一个逻辑端口是错误的。

*   `load_balancer`: `Load_Balancer` 的弱引用集合
    *   与此逻辑交换机相关联的负载均衡器集。

*   `load_balancer_group`: `Load_Balancer_Group` 集合
    *   与此逻辑交换机相关联的负载均衡器组集。

*   `acls`: `ACL` 集合
    *   适用于逻辑交换机内数据包的访问控制规则。

*   `qos_rules`: `QoS` 集合
    *   适用于逻辑交换机内数据包的 QoS 标记和计量规则。

*   `dns_records`: `DNS` 的弱引用集合
    *   此列定义用于由本机 DNS 解析器解析逻辑交换机内的内部 DNS 查询的 DNS 记录。请参阅 `DNS` 表。

*   `forwarding_groups`: `Forwarding_Group` 集合
    *   将一组逻辑端口端点分组，用于流出逻辑交换机的流量。

*   **命名**:

    *   这些列为逻辑交换机提供名称。从 OVN 的角度来看，这些名称除了为与数据库的人机交互提供便利外，没有特殊的含义或用途。名称不要求唯一。（对于逻辑交换机的唯一标识符，请使用其行 UUID。）

    *   （最初，`name` 旨在用于人类友好的名称，但 Neutron 集成使用它以 `neutron-uuid` 格式唯一标识自己的交换机对象。后来，Neutron 开始将交换机的友好名称传播为 `external_ids:neutron:network_name`。也许有一天这可以被清理掉。）

    *   `name`: 字符串
        *   逻辑交换机的名称。

    *   `external_ids : neutron:network_name`: 可选字符串
        *   逻辑交换机的另一个名称。

    *   `other_config : dynamic-routing-bridge-ifname`: 可选字符串
        *   设置用于 EVPN 集成的网桥的接口名称。默认名称为 `br-$vni`。仅当 `other_config:dynamic-routing-vni` 设置为有效 VNI 时才相关。

    *   `other_config : dynamic-routing-vxlan-ifname`: 可选字符串
        *   用于 EVPN 集成的 vxlan 设备的接口名称列表。默认名称为 `vxlan-$vni`。仅当 `other_config:dynamic-routing-vni` 设置为有效 VNI 时才相关。

    *   `other_config : dynamic-routing-advertise-ifname`: 可选字符串
        *   设置用于 EVPN 集成的通告设备的接口名称。默认名称为 `lo-$vni`。仅当 `other_config:dynamic-routing-vni` 设置为有效 VNI 时才相关。

*   **IP 地址分配**:

    *   这些选项控制连接到逻辑交换机的端口的自动 IP 地址管理 (IPAM)。要为 IPv4 启用 IPAM，请设置 `other_config:subnet` 并可选择设置 `other_config:exclude_ips`。要为 IPv6 启用 IPAM，请设置 `other_config:ipv6_prefix`。IPv4 和 IPv6 可以一起启用或分别启用。

    *   要为特定端口请求动态地址分配，请在端口的 `Logical_Switch_Port` 行的 `addresses` 列中使用 `dynamic` 关键字。如果同时启用了 IPv4 和 IPv6 的 IPAM，这将请求 IPv4 和 IPv6 地址。

    *   `other_config : subnet`: 可选字符串
        *   将其设置为 IPv4 子网，例如 `192.168.0.0/24`，以使 ovn-northd 能够自动分配该子网内的 IP 地址。

    *   `other_config : exclude_ips`: 可选字符串
        *   要从自动 IP 地址管理中排除某些地址，请将其设置为要排除的 IPv4 地址或 `..` 分隔范围的列表。地址或范围应该是 `other_config:subnet` 中的子集。

        *   无论是否列出，ovn-northd 永远不会分配子网中的第一个或最后一个地址，例如 `192.168.0.0/24` 中的 `192.168.0.0` 或 `192.168.0.255`。

        *   示例：
            *   `192.168.0.2 192.168.0.10`
            *   `192.168.0.4 192.168.0.30..192.168.0.60 192.168.0.110..192.168.0.120`
            *   `192.168.0.110..192.168.0.120 192.168.0.25..192.168.0.30 192.168.0.144`

    *   `other_config : ipv6_prefix`: 可选字符串
        *   将其设置为 IPv6 前缀以使 ovn-northd 能够使用此前缀自动分配 IPv6 地址。分配的 IPv6 地址将使用 IPv6 前缀和端口的 MAC 地址（转换为 IEEE EUI64 标识符）生成。此处定义的 IPv6 前缀应该是一个以 `::` 结尾的有效 IPv6 地址。

        *   示例：
            *   `aef0::`
            *   `bef0:1234:a890:5678::`
            *   `8230:5678::`

    *   `other_config : dhcp_relay_port`: 可选字符串
        *   如果设置为类型为路由器的逻辑交换机端口的名称，则如果相应的 `Logical_Router_Port` 配置了 DHCP 中继，则为此逻辑交换机启用 DHCP 中继。

    *   `other_config : mac_only`: 可选字符串，`true` 或 `false`
        *   用于仅在未指定子网或 ipv6_prefix 时请求分配 L2 地址的值。

    *   `other_config : fdb_age_threshold`: 可选字符串，包含整数，范围 0 到 4,294,967,295
        *   FDB 老化阈值（以秒为单位）。超过此超时的 FDB 将被自动删除。该值默认为 0，表示禁用。

    *   `other_config : ct-zone-limit`: 可选字符串，包含整数，范围 0 到 4,294,967,295
        *   给定 `Logical_Switch` 的 CT 区域限制值。配置后，此值将传播到所有 `Logical_Switch_Port`，但可以按 `Logical_Switch_Port` 单独覆盖。值 0 表示无限制。如果不存在该选项，则未设置限制，并且区域限制源自 OvS 默认数据路径限制。

    *   `other_config : enable-stateless-acl-with-lb`: 可选字符串，`true` 或 `false`
        *   必须将此选项设置为 true 才能使无状态 ACL 与负载均衡器一起工作。启用后，带有 `ct.inv` 标志的数据包将不会被丢弃，即使 `use_ct_inv_match` 设置为 true 也是如此。默认值：false。

    *   `other_config : dynamic-routing-vni`: 可选字符串，包含整数，范围 0 到 16,777,215
        *   这定义了与逻辑交换机应连接到的 EVPN 域关联的 vni 编号。

        *   ovn-controller 期望 BGP vrf 中存在三个接口：`br-$vni`、`lo-$vni` 和 `vxlan-$vni`。

        *   注意：此功能是实验性的，将来可能会被删除/更改。

    *   `other_config : dynamic-routing-fdb-prefer-local`: 可选字符串，`true` 或 `false`
        *   此选项定义了 FDB 查找的首选项，如果设置为 true，OVN 将首先尝试在 SB FDB 表中查找 FDB 条目。然后它尝试通过 ovn-controller 本地 EVPN FDB 缓存解析 FDB。该选项默认为 false。

        *   仅当 `other_config:dynamic-routing-vni` 设置为有效 VNI 时才相关。

        *   注意：此功能是实验性的，将来可能会被删除/更改。

    *   `other_config : dynamic-routing-arp-prefer-local`: 可选字符串，`true` 或 `false`
        *   此选项定义 ARP/ND 查找的首选项。如果设置为 true，连接到 EVPN 逻辑交换机（在其上已学习远程邻居条目（Type-2 MAC+IP EVPN 路由））的 OVN 路由器将在尝试通过 ovn-controller 本地 EVPN ARP/ND 缓存解析 MAC 地址之前，优先使用它们可能在 SB `Mac_Binding` 表中拥有的任何 ARP/ND 条目。该选项默认为 false。

        *   仅当 `other_config:dynamic-routing-vni` 设置为有效 VNI 时才相关。

        *   注意：此功能是实验性的，将来可能会被删除/更改。

    *   `other_config : dynamic-routing-redistribute`: 可选字符串
        *   仅当 `other_config:dynamic-routing-vni` 设置为有效 VNI 时才相关。

        *   如果指定了 `fdb`，则 ovn-controller 将通告机箱本地的所有工作负载。这适用于 VIF、容器端口、虚拟端口、连接的 DGP 和连接的 GW 路由器。

        *   如果指定了 `ip`，则 ovn-controller 将通告机箱本地的所有 IP/MAC 绑定。这适用于 VIF 和路由器端口。

        *   注意：此功能是实验性的，将来可能会被删除/更改。

*   **IP 多播侦听选项**:

    *   这些选项控制逻辑交换机的 IP 多播侦听配置。要启用 IP 多播侦听，请将 `other_config:mcast_snoop` 设置为 true。要启用 IP 多播查询器，请将 `other_config:mcast_querier` 设置为 true。如果启用了 IP 多播查询器，则必须设置 `other_config:mcast_eth_src` 和 `other_config:mcast_ip4_src`。

    *   `other_config : mcast_snoop`: 可选字符串，`true` 或 `false`
        *   启用/禁用逻辑交换机上的 IP 多播侦听。默认值：false。

    *   `other_config : mcast_querier`: 可选字符串，`true` 或 `false`
        *   启用/禁用逻辑交换机上的 IP 多播查询器。仅当启用了 `other_config:mcast_snoop` 时才适用。默认值：true。

    *   `other_config : mcast_flood_unregistered`: 可选字符串，`true` 或 `false`
        *   确定是否应泛洪未注册的多播流量。仅当启用了 `other_config:mcast_snoop` 时才适用。默认值：false。

    *   `other_config : mcast_table_size`: 可选字符串，包含整数，范围 1 到 32,766
        *   要存储的多播组数。默认值：2048。

    *   `other_config : mcast_idle_timeout`: 可选字符串，包含整数，范围 15 到 3,600
        *   配置 IP 多播侦听组空闲超时（以秒为单位）。默认值：300 秒。

    *   `other_config : mcast_query_interval`: 可选字符串，包含整数，范围 1 到 3,600
        *   配置 IP 多播查询器查询之间的间隔（以秒为单位）。默认值：`other_config:mcast_idle_timeout` / 2。

    *   `other_config : mcast_query_max_response`: 可选字符串，包含整数，范围 1 到 10
        *   配置逻辑交换机发起的组播查询中“最大响应”字段的值。默认值：1 秒。

    *   `other_config : mcast_eth_src`: 可选字符串
        *   配置逻辑交换机发起的查询的源以太网地址。

    *   `other_config : mcast_ip4_src`: 可选字符串
        *   配置逻辑交换机发起的查询的源 IPv4 地址。

    *   `other_config : mcast_ip6_src`: 可选字符串
        *   配置逻辑交换机发起的查询的源 IPv6 地址。

*   **互连**:

    *   `other_config : interconn-ts`: 可选字符串
        *   `OVN_IC_Northbound` 数据库中相应传输交换机的名称。这种逻辑交换机由 ovn-ic 创建和控制。

    *   `other_config : ic-vxlan_mode`: 可选字符串，`true` 或 `false`
        *   当 ovn-ic 运行 VXLAN 作为跨 AZ 流量的封装协议时，`ic-vxlan_mode` 设置为 true。默认值为 false。

*   **隧道密钥**:

    *   `other_config : requested-tnl-key`: 可选字符串，包含整数，范围 1 到 16,777,215
        *   配置逻辑交换机的数据路径隧道密钥。通常不需要这样做，因为 ovn-northd 将自行分配每个数据路径的唯一密钥。但是，如果已配置，ovn-northd 会遵守配置的值。典型的用例是互连：传输交换机的隧道密钥需要全局唯一，因此它们在全局 `OVN_IC_Southbound` 数据库中维护，并且 ovn-ic 只是通过此配置从 `OVN_IC_Southbound` 同步该值。

    *   `copp`: 可选的 `Copp` 弱引用
        *   来自表 `Copp` 的控制平面保护策略，用于计量从此逻辑交换机的端口发送到 ovn-controller 的数据包。

*   **其他选项**:

    *   `other_config : vlan-passthru`: 可选字符串，`true` 或 `false`
        *   确定是否应允许 VLAN 标记的传入流量。请注意，当为具有 tag=0 localnet 端口的逻辑交换机启用此功能时，可能会有安全隐患。如果未与其他 localnet 端口正确隔离，则属于其他标记网络的结构流量可能会通过此类端口传递。

    *   `other_config : broadcast-arps-to-all-routers`: 可选字符串，`true` 或 `false`
        *   确定 arp 请求和 ipv6 邻居请求是应该发送到所有路由器和其他交换机端口（默认），还是应该仅发送到 ip/mac 地址未知的交换机端口。如果逻辑交换机可以接收它不知道的 ip 的 arp 请求，则将其设置为 false 可以显着减少负载。但是，将其设置为 false 也可以意味着 garps 不再转发到所有路由器，因此路由器的 mac 绑定不再更新。

*   **通用列**:

    *   `external_ids`: 字符串对的映射
        *   请参阅本文档开头的“外部 ID”。

# Logical_Switch_Port 表

L2 逻辑交换机内的端口。

## 汇总

*   核心特性:
    *   `name`
        *   字符串 (在表中必须唯一)
    *   `type`
        *   字符串
*   选项:
    *   `options`
        *   字符串对的映射
    *   路由器端口选项:
        *   `options : router-port`
            *   可选字符串
        *   `options : nat-addresses`
            *   可选字符串
        *   `options : exclude-lb-vips-from-garp`
            *   可选字符串
        *   `options : exclude-router-ips-from-garp`
            *   可选字符串
        *   `options : arp_proxy`
            *   可选字符串
        *   `options : enable_router_port_acl`
            *   可选字符串，`true` 或 `false`
        *   `options : ct-zone-limit`
            *   可选字符串，包含整数，范围 0 到 4,294,967,295
    *   Localnet 端口选项:
        *   `options : network_name`
            *   可选字符串
        *   `options : ethtype`
            *   可选字符串
        *   `options : localnet_learn_fdb`
            *   可选字符串，`true` 或 `false`
    *   l2gateway 端口选项:
        *   `options : network_name`
            *   可选字符串
        *   `options : l2gateway-chassis`
            *   可选字符串
    *   vtep 端口选项:
        *   `options : vtep-physical-switch`
            *   可选字符串
        *   `options : vtep-logical-switch`
            *   可选字符串
    *   VMI (或 VIF) 选项:
        *   `options : requested-chassis`
            *   可选字符串
        *   `options : activation-strategy`
            *   可选字符串
        *   `options : iface-id-ver`
            *   可选字符串
        *   `options : qos_min_rate`
            *   可选字符串
        *   `options : qos_max_rate`
            *   可选字符串
        *   `options : qos_burst`
            *   可选字符串
        *   `options : hostname`
            *   可选字符串
        *   `options : force_fdb_lookup`
            *   可选字符串，`true` 或 `false`
        *   `options : disable_garp_rarp`
            *   可选字符串，`true` 或 `false`
        *   `options : pkt_clone_type`
            *   可选字符串，必须是 `mc_unknown`
        *   `options : disable_arp_nd_rsp`
            *   可选字符串，`true` 或 `false`
        *   `options : lsp_learn_mac`
            *   可选字符串，`true` 或 `false`
        *   `options : receive_multicast`
            *   可选字符串，`true` 或 `false`
        *   `options : is-nf`
            *   可选字符串，`true` 或 `false`
        *   `options : nf-linked-port`
            *   可选字符串
        *   VIF 插入选项:
            *   `options : vif-plug-type`
                *   可选字符串
            *   `options : vif-plug-mtu-request`
                *   可选字符串
    *   虚拟端口选项:
        *   `options : virtual-ip`
            *   可选字符串
        *   `options : virtual-parents`
            *   可选字符串
    *   IP 多播侦听选项:
        *   `options : mcast_flood`
            *   可选字符串，`true` 或 `false`
        *   `options : mcast_flood_reports`
            *   可选字符串，`true` 或 `false`
*   容器:
    *   `parent_name`
        *   可选字符串
    *   `tag_request`
        *   可选整数，范围 0 到 4,095
    *   `tag`
        *   可选整数，范围 1 到 4,095
*   端口状态:
    *   `up`
        *   可选布尔值
    *   `enabled`
        *   可选布尔值
*   寻址:
    *   `addresses`
        *   字符串集合
    *   `dynamic_addresses`
        *   可选字符串
    *   `port_security`
        *   字符串集合
*   `peer`
    *   可选字符串
*   DHCP:
    *   `dhcpv4_options`
        *   可选的 `DHCP_Options` 弱引用
    *   `dhcpv6_options`
        *   可选的 `DHCP_Options` 弱引用
*   `mirror_rules`
    *   `Mirror` 的弱引用集合
*   `ha_chassis_group`
    *   可选的 `HA_Chassis_Group`
*   命名:
    *   `external_ids : neutron:port_name`
        *   可选字符串
*   隧道密钥:
    *   `options : requested-tnl-key`
        *   可选字符串，包含整数，范围 1 到 32,767
*   通用列:
    *   `external_ids`
        *   字符串对的映射

## 详细信息

*   **核心特性**:

    *   `name`: 字符串 (在表中必须唯一)
        *   逻辑端口名称。

        *   对于在管理程序中生成的实体（VM 或容器），此处使用的名称必须与 Open_vSwitch 数据库的 `Interface` 表中的 `external_ids:iface-id` 中使用的名称匹配，因为管理程序使用 `external_ids:iface-id` 作为查找键来识别该实体的网络接口。

        *   对于在 VM 内共享 VIF 的容器，名称可以是任何唯一标识符。有关更多信息，请参阅下面的“容器”。

        *   逻辑交换机端口不能与逻辑路由器端口具有相同的名称，但数据库架构无法强制执行此操作。

    *   `type`: 字符串
        *   为此逻辑端口指定一种类型。逻辑端口可用于将其他类型的连接建模到 OVN 逻辑交换机中。定义了以下类型：

        *   (空字符串)
            *   VM (或 VIF) 接口。

        *   `router`
            *   到逻辑路由器的连接。`options:router-port` 的值指定此逻辑交换机端口连接到的 `Logical_Router_Port` 的名称。

        *   `switch`
            *   到另一个逻辑交换机的连接。`peer` 的值指定此逻辑交换机端口连接到的 `Logical_Switch_Port` 的名称。此类端口始终具有隐式的 "unknown" 地址，因为地址信息不会在直接连接的交换机之间泄漏。

        *   `localnet`
            *   从具有相应网桥映射的 `ovn-controller` 实例连接到本地可访问的网络。一个逻辑交换机可以连接多个 `localnet` 端口。此类型用于对现有网络的直接连接进行建模。在这种情况下，每个机箱应该只有一个物理网络的映射。注意：上面所说的并不意味着机箱不能插入多个物理网络，只要它们属于不同的交换机。

        *   `localport`
            *   连接到本地 VIF。到达 `localport` 的流量永远不会通过隧道转发到另一个机箱。这些端口存在于每个机箱上，并且在所有机箱中具有相同的地址。这用于对运行在每个管理程序上的本地服务的连接进行建模。

        *   `l2gateway`
            *   到物理网络的连接。

        *   `vtep`
            *   VTEP 网关上的逻辑交换机端口。

        *   `external`
            *   表示一个逻辑端口，它是外部的，并且在集成网桥中没有 OVS 端口。OVN 将永远不会从该端口接收任何流量或向该端口发送任何流量。OVN 可以为此端口支持本机服务，如 DHCPv4/DHCPv6/DNS。如果定义了 `ha_chassis_group`，则在 HA 机箱组的活动机箱中运行的 `ovn-controller` 将绑定此端口以提供这些本机服务。预计此端口属于桥接逻辑交换机（具有 localnet 端口）。

            *   建议为逻辑交换机的所有外部端口使用相同的 HA 机箱组。否则，当不同的机箱提供本机服务时，物理交换机可能会看到 MAC 翻动问题。例如，当支持本机 DHCPv4 服务时，来自不同端口的 DHCPv4 服务器 mac（在 `DHCP_Options` 表的 `options:server_mac` 列中配置）可能会导致 MAC 翻动问题。如果未为逻辑交换机的所有外部端口设置相同的 HA 机箱组，则逻辑路由器 IP 的 MAC 也可能会翻动。

            *   以下是可以使用外部端口的一些用例。

            *   连接到 SR-IOV 网卡的 VM - 来自这些 VM 的流量绕过内核堆栈，并且本地 `ovn-controller` 不绑定这些端口，也无法提供本机服务。

            *   当 CMS 支持配置裸机服务器时。

        *   `virtual`
            *   表示一个逻辑端口，它在集成网桥中没有 OVS 端口，并且在 `options:virtual-ip` 列中配置了虚拟 ip。此虚拟 ip 可以在 `options:virtual-parents` 列中配置的逻辑端口之间移动。

            *   可以使用虚拟端口的用例之一是：

            *   虚拟 ip 代表负载均衡器 vip，虚拟父级在主备设置中提供负载均衡器服务，其中活动虚拟父级拥有虚拟 ip。

        *   `remote`
            *   远程端口用于对远程驻留在另一个 OVN 上的端口进行建模，该 OVN 位于用于 OVN 互连的传输逻辑交换机的另一侧。此类端口由 ovn-ic 而不是由 CMS 创建。对端口的任何更改都将被 ovn-ic 自动覆盖。

*   **选项**:

    *   `options`: 字符串对的映射
        *   此列提供特定于逻辑端口类型的键/值设置。特定于类型的选项在下面单独描述。

    *   **路由器端口选项**:

    *   当 type 为 router 时，应用这些选项。

    *   `options : router-port`: 可选字符串
        *   必需。此逻辑交换机端口连接到的 `Logical_Router_Port` 的名称。

    *   `options : nat-addresses`: 可选字符串
        *   这用于通过连接到与此类型路由器端口相同的逻辑交换机的 localnet 端口发送 SNAT 和 DNAT IP 地址的免费 ARP。此选项在连接到网关路由器的逻辑交换机端口或连接到逻辑路由器上的分布式网关端口的逻辑交换机端口上指定。

        *   这必须采用以下形式之一：

        *   `router`
            *   将使用 `options:router-port` 的 MAC 地址，为 `options:router-port` 的逻辑路由器上定义的所有 SNAT 和 DNAT 外部 IP 地址以及所有负载均衡器 IP 地址发送免费 ARP。

            *   此形式的 `options:nat-addresses` 对于 `options:router-port` 是网关路由器上的端口名称或分布式网关端口名称的逻辑交换机端口有效。

            *   仅在 OVN 2.8 及更高版本中受支持。早期版本需要手动同步 NAT 地址。

        *   `Ethernet address followed by one or more IPv4 addresses`
            *   示例：`80:fa:5b:06:72:b7 158.36.44.22 158.36.44.24`。这将导致使用 MAC 地址 `80:fa:5b:06:72:b7` 为 IP 地址 `158.36.44.22` 和 `158.36.44.24` 生成免费 ARP。

            *   此形式的 `options:nat-addresses` 仅对 `options:router-port` 是网关路由器上的端口名称的逻辑交换机端口有效。

    *   `options : exclude-lb-vips-from-garp`: 可选字符串
        *   如果 `options:nat-addresses` 设置为 `router`，则将使用 `options:router-port` 的 MAC 地址，为 `options:router-port` 的逻辑路由器上定义的所有 SNAT 和 DNAT 外部 IP 地址发送免费 ARP，而不考虑配置的负载均衡器。

    *   `options : exclude-router-ips-from-garp`: 可选字符串
        *   如果 `options:nat-addresses` 设置为 `router`，则不会为在其对等路由器端口上定义的路由器端口 IP 地址和 SNAT IP 地址（如果 SNAT IP 与路由器端口 IP 相同）发送免费 ARP。如果路由器端口 IP 也用作 SNAT 和 DNAT IP，请勿设置此选项。

    *   `options : arp_proxy`: 可选字符串
        *   可选。此逻辑交换机路由器端口将回复 ARP/NDP 请求的 MAC 和地址/cidr 或仅地址/cidr 的列表。示例：`169.254.239.254 169.254.239.2`, `0a:58:a9:fe:01:01 169.254.239.254 169.254.239.2 169.254.238.0/24`, `fd7b:6b4d:7b25:d22f::1 fd7b:6b4d:7b25:d22f::2`, `0a:58:a9:fe:01:01 fd7b:6b4d:7b25:d22f::0/64`。`options:router-port` 的逻辑路由器应该有一条路由，将发送到配置的代理 ARP MAC/IP 的数据包转发到适当的目的地。

    *   `options : enable_router_port_acl`: 可选字符串，`true` 或 `false`
        *   可选。如果设置为 true，则为对等体为 l3dgw_port 的路由器端口启用 conntrack。默认值为 false。

    *   `options : ct-zone-limit`: 可选字符串，包含整数，范围 0 到 4,294,967,295
        *   给定 `Logical_Switch_Port` 的 CT 区域限制值。配置后，此值优先于在 `Logical_Switch` 上指定的限制。值 0 表示无限制。如果不存在该选项，则未设置限制，并且区域限制源自 OvS 默认数据路径限制。

    *   **Localnet 端口选项**:

    *   当 type 为 localnet 时，应用这些选项。

    *   `options : network_name`: 可选字符串
        *   必需。localnet 端口连接到的网络的名称。每个管理程序通过 ovn-controller 使用其本地配置来准确确定如何连接到此本地可访问的网络（如果有）。

    *   `options : ethtype`: 可选字符串
        *   可选。用于封装 VLAN 标头的 VLAN EtherType 字段值。支持的值：802.1q（默认），802.1ad。

    *   `options : localnet_learn_fdb`: 可选字符串，`true` 或 `false`
        *   可选。如果设置为 true，则允许 localnet 端口学习 MAC 并将其存储在 FDB 表中。默认值为 false。

    *   **l2gateway 端口选项**:

    *   当 type 为 l2gateway 时，应用这些选项。

    *   `options : network_name`: 可选字符串
        *   必需。l2gateway 端口连接到的网络的名称。L2 网关通过 ovn-controller 使用其本地配置来准确确定如何连接到此网络。

    *   `options : l2gateway-chassis`: 可选字符串
        *   必需。l2gateway 逻辑端口应绑定到的机箱。在定义的机箱上运行的 ovn-controller 将此逻辑端口连接到物理网络。

    *   **vtep 端口选项**:

    *   当 type 为 vtep 时，应用这些选项。

    *   `options : vtep-physical-switch`: 可选字符串
        *   必需。VTEP 网关的名称。

    *   `options : vtep-logical-switch`: 可选字符串
        *   必需。VTEP 网关连接的逻辑交换机名称。

    *   **VMI (或 VIF) 选项**:

    *   这些选项适用于类型为（空字符串）的逻辑端口

    *   `options : requested-chassis`: 可选字符串
        *   如果设置，则标识允许绑定此端口的特定机箱（按名称或主机名）。使用此选项将防止在实时迁移期间尝试绑定同一端口的两个机箱之间发生抖动。如果端口意外在多个机箱上创建，它还可以防止由于错误配置而导致的类似抖动。

        *   如果设置为逗号分隔的列表，则第一个条目标识主机箱，其余条目是允许绑定同一端口的一个或多个附加机箱。

        *   当为端口设置多个机箱，并且逻辑交换机通过 localnet 端口连接到外部网络时，将对端口强制执行隧道传输，以保证发往该端口的数据包传递到其所有位置。这具有 MTU 含义，因为用于隧道的网络必须具有比 localnet 更大的 MTU 才能实现稳定的连接。

        *   如果同一主机共同托管多个控制器实例（属于同一集群或不同集群），则应特别注意一致地使用此选项中使用的唯一机箱名称。建议在此选项中使用机箱名称，而不是主机名。

    *   `options : activation-strategy`: 可选字符串
        *   如果在 `requested-chassis` 中设置了多个机箱时使用，则指定所有附加机箱的激活策略。默认情况下，不使用激活策略，这意味着附加端口位置可立即使用。该选项支持逗号分隔的列表，您可以在其中组合 3 种协议："rarp"、"garp" 和 "na"。当设置任何协议时，端口将被阻止进行入口和出口通信，直到从新位置发送指定的协议数据包。激活策略在虚拟机的实时迁移方案中很有用。

    *   `options : iface-id-ver`: 可选字符串
        *   如果设置，则仅当 Open_vSwitch 数据库的 `Interface` 表中的 `external_ids` 列中配置了相同的键和值时，`ovn-controller` 才会绑定此端口。

    *   `options : qos_min_rate`: 可选字符串
        *   如果设置，则指示从此接口发送的数据的最小保证速率，单位为 bit/s。

    *   `options : qos_max_rate`: 可选字符串
        *   如果设置，则指示从此接口发送的数据的最大速率，单位为 bit/s。流量将根据此限制进行整形。

    *   `options : qos_burst`: 可选字符串
        *   如果设置，则指示从此接口发送的数据的最大突发大小，单位为 bits。

    *   `options : hostname`: 可选字符串
        *   如果设置，则指示与此逻辑交换机端口关联的 DHCPv4 选项“主机名”（选项代码 12）。如果为此逻辑交换机端口启用了 DHCPv4，则主机名 dhcp 选项将包含在 DHCP 回复中。

    *   `options : force_fdb_lookup`: 可选字符串，`true` 或 `false`
        *   仅当逻辑交换机端口为默认类型（即类型设置为空字符串）且 `addresses` 列包含 `unknown` 时，才支持此选项。如果设置为 true，MAC 地址（如果已配置）不会安装在 l2 查找表中，而是学习 MAC 地址并将其存储在 FDB 表中。默认值为 false。

    *   `options : disable_garp_rarp`: 可选字符串，`true` 或 `false`
        *   如果设置为 true，则在桥接逻辑交换机上创建 VIF 端口时不会发送 GARP 和 RARP 通告。默认值为 false。

    *   `options : pkt_clone_type`: 可选字符串，必须是 `mc_unknown`
        *   如果设置为 `mc_unknown`，则发往此 VIF 的数据包将被克隆到连接到同一逻辑交换机的所有未知端口。

    *   `options : disable_arp_nd_rsp`: 可选字符串，`true` 或 `false`
        *   如果设置为 true，则不会为在此逻辑端口上配置的 IP 地址安装 ARP/ND 响应流。默认值：false。

    *   `options : lsp_learn_mac`: 可选字符串，`true` 或 `false`
        *   如果设置为 false，则在此端口上接收数据包时，FDB 不会学习源 MAC 地址。默认值为 true。

    *   `options : receive_multicast`: 可选字符串，`true` 或 `false`
        *   如果设置为 false，则不会将多播流量转发到此端口。默认值为 true。

    *   `options : is-nf`: 可选字符串，`true` 或 `false`
        *   对于网络功能端口，需要设置为 true。这些是在 `Network_Function` 表中用作入站端口或出站端口的端口。默认值为 false。

    *   `options : nf-linked-port`: 可选字符串
        *   `Network_Function` 表中的每一行都引用 `inport` 和 `outport` 列下的两个逻辑交换机端口。标识为 `inport` 的端口需要将此选项设置为标识为 `outport` 的端口，反之亦然。

    *   **VIF 插入选项**:

    *   `options : vif-plug-type`: 可选字符串
        *   如果设置，OVN 将尝试执行此 VIF 的插入。为了让 OVN 控制器插入此端口，OVN 必须构建为支持 VIF 插入。默认行为是 CMS 执行 VIF 插入。每个 VIF 插入提供程序都有自己的按名称命名的选项，例如 "vif-plug:representor:key"。有关更多信息，请参阅位于 `Documentation/topics/vif-plug-providers/` 的 VIF 插入提供程序文档。

    *   `options : vif-plug-mtu-request`: 可选字符串
        *   插入接口的请求 MTU。设置后，OVN 控制器将填充 Open vSwitch 数据库的 `Interface` 表的 `mtu_request` 列。这反过来将使 OVS vswitchd 更新链接接口的 MTU。

    *   **虚拟端口选项**:

    *   当 type 为 virtual 时，应用这些选项。

    *   `options : virtual-ip`: 可选字符串
        *   此选项代表虚拟 IPv4 地址。

    *   `options : virtual-parents`: 可选字符串
        *   此选项代表一组逻辑端口名称（在同一逻辑交换机内），它们可以拥有在 `options:virtual-ip` 中配置的虚拟 ip。如果启用了端口安全地址，则所有这些虚拟父级都应在端口安全中添加虚拟 ip。

    *   **IP 多播侦听选项**:

    *   当端口是 `other_config:mcast_snoop` 设置为 true 的逻辑交换机的一部分时，应用这些选项。

    *   `options : mcast_flood`: 可选字符串，`true` 或 `false`
        *   如果设置为 true，则多播数据包（报告除外）将无条件转发到特定端口。默认值：false。

    *   `options : mcast_flood_reports`: 可选字符串，`true` 或 `false`
        *   如果设置为 true，则多播报告将无条件转发到特定端口。默认值：false。

    *   **容器**:

    *   当大量容器嵌套在 VM 内时，为每个容器分配一个 VIF 可能太昂贵。OVN 可以使用 VLAN 标记来支持这种情况。每个容器分配一个 VLAN ID，在管理程序和 VM 之间传递的每个数据包都标记有容器的相应 ID。此类 VLAN ID 永远不会出现在物理线路上，甚至不会出现在隧道内，因此除了相对于管理程序上的单个 VM 之外，它们不需要是唯一的。

    *   这些列用于表示使用共享 VIF 的嵌套容器的 VIF。对于具有专用 VIF 的 VM 和容器，它们为空。

    *   `parent_name`: 可选字符串
        *   嵌套容器发送其网络流量所通过的 VM 接口。这必须与某个其他 `Logical_Switch_Port` 的 `name` 列匹配。注意：为了提高 OVN 南向数据库条件监视的性能，与常规 VIF 不同，`ovn-controller` 将注册以获取有关与嵌套容器端口对应的所有 OVN 南向数据库 `Port_Binding` 表记录的更新，即使 `external_ids:ovn-monitor-all` 设置为 false。有关更多信息，请参阅 `ovn-controller(8)`。

    *   `tag_request`: 可选整数，范围 0 到 4,095
        *   与容器网络接口关联的网络流量中的 VLAN 标记。客户端可以通过在此列中设置值 0 来请求 ovn-northd 分配一个在特定父级（在 `parent_name` 中指定）范围内唯一的标记。分配的值由 ovn-northd 写入 `tag` 列。（请注意，这些标记是在 ovn-northd 中本地分配和管理的，因此如果数据库丢失，则无法重建它们。）客户端还可以请求特定的非零标记，ovn-northd 将遵守它并将该值复制到 `tag` 列。

        *   当 `type` 设置为 localnet 或 l2gateway 时，可以设置此项以指示该端口代表连接到本地可访问网络上的特定 VLAN。VLAN ID 用于匹配传入流量，也添加到传出流量。

    *   `tag`: 可选整数，范围 1 到 4,095
        *   ovn-northd 根据 `tag_request` 列的内容分配的 VLAN 标记。

*   **端口状态**:

    *   `up`: 可选布尔值
        *   此列由 ovn-northd 填充，而不是像此数据库的大部分内容那样由 CMS 插件填充。当逻辑端口绑定到 OVN 南向数据库 `Binding` 表中的物理位置时，ovn-northd 将此列设置为 true；否则，或者如果端口稍后解除绑定，它将其设置为 false。如果此列为空，则不认为端口已启动。这允许 CMS 等待 VM（或容器）的网络连接变为活动状态，然后再允许 VM（或容器）启动。

        *   路由器类型的逻辑端口是此规则的例外。它们被认为始终处于启动状态，即此列始终设置为 true。

    *   `enabled`: 可选布尔值
        *   此列用于管理设置端口状态。如果此列为空或设置为 true，则端口已启用。如果此列设置为 false，则端口已禁用。禁用的端口将丢弃所有入口和出口流量。

*   **寻址**:

    *   `addresses`: 字符串集合
        *   逻辑端口拥有的地址。

        *   集合中的每个元素必须采用以下形式之一：

        *   `Ethernet address followed by zero or more IPv4 or IPv6 addresses (or both)`
            *   定义的以太网地址由逻辑端口拥有。像物理以太网 NIC 一样，逻辑端口通常具有单个固定的以太网地址。

            *   当 OVN 逻辑交换机处理目标 MAC 地址在逻辑端口的 `addresses` 列中的单播以太网帧时，它仅将其传递到该端口，就像 MAC 学习过程已在该端口上学习了该 MAC 地址一样。

            *   如果定义了 IPv4 或 IPv6 地址（或两者），则表示逻辑端口拥有给定的 IP 地址。

            *   如果定义了 IPv4 地址，OVN 逻辑交换机使用此信息来合成对 ARP 请求的响应，而无需遍历物理网络。连接到逻辑交换机的 OVN 逻辑路由器（如果有）使用此信息来避免为逻辑交换机端口发出 ARP 请求。

            *   请注意，这里的顺序很重要。如果定义了 IP 地址，则必须在 IP 地址之前列出以太网地址。

            *   示例：
                *   `80:fa:5b:06:72:b7`
                    *   这表示逻辑端口拥有上述 mac 地址。
                *   `80:fa:5b:06:72:b7 10.0.0.4 20.0.0.4`
                    *   这表示逻辑端口拥有 mac 地址和两个 IPv4 地址。
                *   `80:fa:5b:06:72:b7 fdaa:15f2:72cf:0:f816:3eff:fe20:3f41`
                    *   这表示逻辑端口拥有 mac 地址和 1 个 IPv6 地址。
                *   `80:fa:5b:06:72:b7 10.0.0.4 fdaa:15f2:72cf:0:f816:3eff:fe20:3f41`
                    *   这表示逻辑端口拥有 mac 地址和 1 个 IPv4 地址和 1 个 IPv6 地址。

        *   `unknown`
            *   这表示逻辑端口具有一组未知的以太网地址。当 OVN 逻辑交换机处理单播以太网帧，且其目标 MAC 地址不在任何逻辑端口的 `addresses` 列中时，它将其传递给 `addresses` 列包含 `unknown` 的端口（或多个端口）。

        *   `dynamic`
            *   使用 `dynamic` 使 ovn-northd 生成全局唯一的 MAC 地址，选择逻辑端口子网未使用的 IPv4 地址（如果在端口的 `Logical_Switch` 中设置了 `other_config:subnet`），并从 MAC 地址生成 IPv6 地址（如果在端口的 `Logical_Switch` 中设置了 `other_config:ipv6_prefix`），并将它们存储在端口的 `dynamic_addresses` 列中。

            *   `addresses` 中只能出现一个包含 `dynamic` 的元素。

        *   `dynamic ip`
        *   `dynamic ipv6`
        *   `dynamic ip ipv6`
            *   这些行为类似于单独的 `dynamic`，但指定要使用的特定 IPv4 或 IPv6 地址。如果配置得当，OVN IPAM 仍将自动分配其他地址。示例：`dynamic 192.168.0.1 2001::1`。

        *   `mac dynamic`
            *   这类似于单独的 `dynamic`，但指定要使用的特定 MAC 地址。如果配置得当，OVN IPAM 仍将自动分配 IPv4 或 IPv6 地址或两者。示例：`80:fa:5b:06:72:b7 dynamic`

        *   `router`
            *   仅当 type 为 router 时接受。这表示此逻辑交换机端口的以太网、IPv4 和 IPv6 地址应从连接的逻辑路由器端口获取，如 options 中的 `router-port` 所指定。

            *   生成的地址用于填充逻辑交换机的目标查找，也用于逻辑交换机生成 ARP 和 ND 回复。

            *   如果连接的逻辑路由器端口指定了分布式网关端口，并且逻辑路由器在 nat 中指定了具有 external_mac 的规则，则这些地址也用于填充交换机的目标查找。

            *   仅在 OVN 2.7 及更高版本中受支持。早期版本需要手动同步路由器地址。

    *   `dynamic_addresses`: 可选字符串
        *   如果 `addresses` 中指定了 `dynamic`，则为 ovn-northd 分配给逻辑端口的地址。地址将具有与填充 `addresses` 列的格式相同的格式。请注意，动态分配的地址是在 ovn-northd 中本地构造和管理的，因此如果数据库丢失，则无法重建它们。

    *   `port_security`: 字符串集合
        *   此列控制允许连接到逻辑端口的主机（“主机”）从哪些地址发送数据包以及允许它接收哪些地址的数据包。如果此列为空，则允许所有地址。

        *   集合中的每个元素必须以一个以太网地址开头。这将限制主机只能从逻辑端口的 `port_security` 列中定义的以太网地址发送数据包和接收数据包。它还限制了主机在 ARP 和 IPv6 邻居发现数据包中可能发送的内部源 MAC 地址。始终允许主机接收多播和广播以太网地址的数据包。

        *   集合中的每个元素可以另外包含一个或多个 IPv4 或 IPv6 地址（或两者），带有可选掩码。如果给出了掩码，它必须是 CIDR 掩码。除了上述针对以太网地址的限制外，此类元素还限制主机可以发送数据包的 IPv4 或 IPv6 地址，以及它可以接收数据包的指定地址。掩码地址，如果主机部分为零，则表示允许主机使用子网中的任何地址；如果主机部分非零，则掩码仅表示子网的大小。此外：

        *   如果给出了任何 IPv4 地址，则还允许主机接收 IPv4 本地广播地址 `255.255.255.255` 和 IPv4 多播地址 (`224.0.0.0/4`) 的数据包。如果给出了带有掩码的 IPv4 地址，则还允许主机接收该指定子网中的广播地址的数据包。

        *   如果给出了任何 IPv4 地址，则主机还被限制为发送具有指定源 IPv4 地址的 ARP 数据包。（RARP 不受限制。）

        *   如果给出了任何 IPv6 地址，则还允许主机接收 IPv6 多播地址 (`ff00::/8`) 的数据包。

        *   如果给出了任何 IPv6 地址，则主机还被限制为发送具有指定源地址的 IPv6 邻居发现请求或通告数据包，或者对于请求，发送未指定的地址。

        *   如果元素包含 IPv4 地址，但不包含 IPv6 地址，则不允许 IPv6 流量。如果元素包含 IPv6 地址，但不包含 IPv4 地址，则不允许 IPv4 和 ARP 流量。

        *   此列使用与 OVN 南向数据库的 `Pipeline` 表中的 `match` 列相同的词法语法。元素中的多个地址可以用空格或逗号分隔。

        *   此列是作为云管理系统的便利提供的，但它实现的所有功能都可以使用 `ACL` 表作为 ACL 来实现。

        *   示例：
            *   `80:fa:5b:06:72:b7`
                *   主机可以从指定的 MAC 地址发送流量和接收流量，并接收以太网多播和广播地址的流量，但除此之外不行。主机不得发送具有指定以太网地址以外的内部源以太网地址的 ARP 或 IPv6 邻居发现数据包。
            *   `80:fa:5b:06:72:b7 192.168.1.10/24`
                *   这在第一个示例中添加了进一步的限制。主机只能从 `192.168.1.10` 发送 IPv4 数据包或接收 IPv4 数据包，除了它还可以接收 `192.168.1.255`（基于子网掩码）、`255.255.255.255` 和 `224.0.0.0/4` 中的任何地址的 IPv4 数据包。主机不得发送源以太网地址不是 `80:fa:5b:06:72:b7` 或源 IPv4 地址不是 `192.168.1.10` 的 ARP。主机不得发送或接收任何 IPv6（包括 IPv6 邻居发现）流量。
            *   `"80:fa:5b:12:42:ba", "80:fa:5b:06:72:b7 192.168.1.10/24"`
                *   主机可以从指定的 MAC 地址发送流量和接收流量，并接收以太网多播和广播地址的流量，但除此之外不行。对于 MAC `80:fa:5b:12:42:ba`，主机可以从任何 L3 地址发送流量和接收流量。对于 MAC `80:fa:5b:06:72:b7`，主机只能从 `192.168.1.10` 发送 IPv4 数据包或接收 IPv4 数据包，除了它还可以接收 `192.168.1.255`（基于子网掩码）、`255.255.255.255` 和 `224.0.0.0/4` 中的任何地址的 IPv4 数据包。主机不得发送或接收任何 IPv6（包括 IPv6 邻居发现）流量。

    *   `peer`: 可选字符串
        *   对于用于连接两个逻辑交换机的交换机端口，这通过名称标识对中的另一个交换机端口。

        *   对于连接到逻辑路由器的交换机端口，此列为空。

*   **DHCP**:

    *   `dhcpv4_options`: 可选的 `DHCP_Options` 弱引用
        *   此列定义 ovn-controller 在回复 DHCPv4 请求时要包含的 DHCPv4 选项。请参阅 `DHCP_Options` 表。

    *   `dhcpv6_options`: 可选的 `DHCP_Options` 弱引用
        *   此列定义 ovn-controller 在回复 DHCPv6 请求时要包含的 DHCPv6 选项。请参阅 `DHCP_Options` 表。

    *   `mirror_rules`: `Mirror` 的弱引用集合
        *   适用于作为源的逻辑交换机端口的镜像规则。请参阅 `Mirror` 表。

    *   `ha_chassis_group`: 可选的 `HA_Chassis_Group`
        *   引用 OVN 北向数据库的 `HA_Chassis_Group` 表中的一行。它指示如果类型设置为 `external`，则使用的 HA 机箱组。如果类型不是 `external`，则忽略此列。

*   **命名**:

    *   `external_ids : neutron:port_name`: 可选字符串
        *   此列给出了端口的可选人类友好名称。此名称除了为与北向数据库的人机交互提供便利外，没有特殊的含义或用途。

        *   Neutron 从其自己的端口对象的名称复制此内容。（Neutron 端口默认未分配人类友好名称，因此它通常为空。）

*   **隧道密钥**:

    *   `options : requested-tnl-key`: 可选字符串，包含整数，范围 1 到 32,767
        *   配置端口的端口绑定隧道密钥。通常不需要这样做，因为 ovn-northd 将自行分配每个端口的唯一密钥。但是，如果已配置，ovn-northd 会遵守配置的值。典型的用例是互连：传输交换机上的端口的隧道密钥需要全局唯一，因此它们在全局 `OVN_IC_Southbound` 数据库中维护，并且 ovn-ic 只是通过此配置从 `OVN_IC_Southbound` 同步该值。

*   **通用列**:

    *   `external_ids`: 字符串对的映射
        *   请参阅本文档开头的“外部 ID”。

        *   ovn-northd 程序将所有这些对复制到 `OVN_Southbound` 数据库中 `Port_Binding` 表的 `external_ids` 列中。

# Forwarding_Group 表

每一行代表一个转发组。

## 汇总

*   `name`
    *   字符串
*   `vip`
    *   字符串
*   `vmac`
    *   字符串
*   `liveness`
    *   布尔值
*   `child_port`
    *   1 个或多个字符串的集合
*   通用列:
    *   `external_ids`
        *   字符串对的映射

## 详细信息

*   `name`: 字符串
    *   转发组的名称。此名称除了为与 ovn-nb 数据库的人机交互提供便利外，没有特殊的含义或用途。

*   `vip`: 字符串
    *   分配给转发组的虚拟 IP 地址。当针对 vip 发送 ARP 请求时，它将以 vmac 响应。

*   `vmac`: 字符串
    *   分配给转发组的虚拟 MAC 地址。

*   `liveness`: 布尔值
    *   如果设置为 true，则为子端口启用活跃度，否则禁用。

*   `child_port`: 1 个或多个字符串的集合
    *   转发组中的子端口列表。

*   **通用列**:

    *   `external_ids`: 字符串对的映射
        *   请参阅本文档开头的“外部 ID”。

# Address_Set 表

此表中的每一行代表一个命名的地址集。地址集可以包含以太网、IPv4 或 IPv6 地址，带有可选的按位或 CIDR 掩码。地址集最终可用于 ACL 中，以与诸如 `ip4.src` 或 `ip6.src` 之类的字段进行比较。单个地址集必须包含相同类型的地址。作为示例，以下将创建一个包含三个 IP 地址的地址集：

`ovn-nbctl create Address_Set name=set1 addresses='10.0.0.1 10.0.0.2 10.0.0.3'`

地址集可用于 ACL 表的 `match` 列。有关语法信息，请参阅 `OVN_Southbound` 数据库的 `Logical_Flow` 表的 `match` 列使用的表达式语言的详细信息。

## 汇总

*   `name`
    *   字符串 (在表中必须唯一)
*   `addresses`
    *   字符串集合
*   通用列:
    *   `external_ids`
        *   字符串对的映射

## 详细信息

*   `name`: 字符串 (在表中必须唯一)
    *   地址集的名称。名称是 ASCII 码，必须匹配 `[a-zA-Z_.][a-zA-Z_.0-9]*`。

*   `addresses`: 字符串集合
    *   字符串形式的地址集。

*   **通用列**:

    *   `external_ids`: 字符串对的映射
        *   请参阅本文档开头的“外部 ID”。

# Port_Group 表

此表中的每一行代表一组命名的逻辑交换机端口。

端口组可用于 ACL 表的 `match` 列。有关语法信息，请参阅 `OVN_Southbound` 数据库的 `Logical_Flow` 表的 `match` 列使用的表达式语言的详细信息。

对于每个端口组，都会在 `OVN_Southbound` 数据库的 `Address_Set` 表中生成两个地址集，其中包含该组端口的 IP 地址，一个用于 IPv4，另一个用于 IPv6，名称为 `Port_Group` 的名称后跟后缀 `_ip4`（对于 IPv4）和 `_ip6`（对于 IPv6）。生成的地址集可以像 ACL 表的 `match` 列中的常规地址集一样使用。有关语法信息，请参阅 `OVN_Southbound` 数据库的 `Logical_Flow` 表的 `match` 列使用的表达式语言的详细信息。

## 汇总

*   `name`
    *   字符串 (在表中必须唯一)
*   `ports`
    *   `Logical_Switch_Port` 的弱引用集合
*   `acls`
    *   `ACL` 集合
*   通用列:
    *   `external_ids`
        *   字符串对的映射

## 详细信息

*   `name`: 字符串 (在表中必须唯一)
    *   端口组的名称。名称是 ASCII 码，必须匹配 `[a-zA-Z_.][a-zA-Z_.0-9]*`。

*   `ports`: `Logical_Switch_Port` 的弱引用集合
    *   属于该组的逻辑交换机端口的 uuid。

*   `acls`: `ACL` 集合
    *   适用于端口组的访问控制规则。将 ACL 应用于端口组与将 ACL 应用于端口组所属的所有逻辑交换机具有相同的效果。

*   **通用列**:

    *   `external_ids`: 字符串对的映射
        *   请参阅本文档开头的“外部 ID”。
