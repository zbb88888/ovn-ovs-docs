# Logical_Router 表

每行代表一个 L3 逻辑路由器。

## 摘要

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `ports` | `Logical_Router_Port` 的集合 | 路由器的端口。 |
| `static_routes` | `Logical_Router_Static_Route` 的集合 | 路由器的静态路由。 |
| `policies` | `Logical_Router_Policy` 的集合 | 路由器的路由策略。 |
| `enabled` | 可选布尔值 | 路由器的启用状态。 |
| `nat` | `NAT` 的集合 | 路由器的 NAT 规则。 |
| `load_balancer` | `Load_Balancer` 的弱引用集合 | 关联的负载均衡器。 |
| `load_balancer_group` | `Load_Balancer_Group` 的集合 | 关联的负载均衡器组。 |

**命名：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `name` | 字符串 | 逻辑路由器的名称。 |
| `external_ids : neutron:router_name` | 可选字符串 | Neutron 中的路由器名称。 |
| `copp` | `Copp` 的可选弱引用 | 控制平面保护策略。 |

**选项：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `options : chassis` | 可选字符串 | 指定网关路由器的 Chassis。 |
| `options : dnat_force_snat_ip` | 可选字符串 | 强制 SNAT 的 IP 地址。 |
| `options : lb_force_snat_ip` | 可选字符串 | 负载均衡强制 SNAT 的 IP 地址。 |
| `options : mcast_relay` | 可选字符串，`true` 或 `false` | 是否启用组播中继。 |
| `options : dynamic_neigh_routers` | 可选字符串，`true` 或 `false` | 是否动态解析邻居路由器 MAC。 |
| `options : always_learn_from_arp_request` | 可选字符串，`true` 或 `false` | 是否总是从 ARP 请求学习。 |
| `options : requested-tnl-key` | 可选字符串，1 到 16,777,215 的整数 | 请求的隧道密钥。 |
| `options : snat-ct-zone` | 可选字符串，0 到 65,535 的整数 | SNAT 使用的 CT 区域。 |
| `options : mac_binding_age_threshold` | 可选字符串 | MAC 绑定的老化阈值。 |
| `options : ct-zone-limit` | 可选字符串，0 到 4,294,967,295 的整数 | CT 区域限制。 |
| `options : dynamic-routing` | 可选字符串，`true` 或 `false` | 是否启用动态路由。 |
| `options : dynamic-routing-redistribute` | 可选字符串 | 动态路由重分发配置。 |
| `options : dynamic-routing-redistribute-local-only` | 可选字符串，`true` 或 `false` | 仅重分发本地路由。 |
| `options : dynamic-routing-vrf-name` | 可选字符串 | 动态路由 VRF 名称。 |
| `options : dynamic-routing-vrf-id` | 可选字符串，1 到 4,294,967,295 的整数 | 动态路由 VRF ID。 |
| `options : dynamic-routing-no-learning` | 可选字符串，`true` 或 `false` | 禁用动态路由学习。 |
| `options : dynamic-routing-v4-prefix-nexthop` | 可选字符串 | IPv4 路由的下一跳。 |
| `options : dynamic-routing-v6-prefix-nexthop` | 可选字符串 | IPv6 路由的下一跳。 |
| `options : ct-commit-all` | 可选字符串，`true` 或 `false` | 是否提交所有流量到 CT。 |
| `options : ic-route-filter-adv` | 可选字符串 | 互联路由通告过滤。 |
| `options : ic-route-filter-learn` | 可选字符串 | 互联路由学习过滤。 |
| `options : ic-route-deny-adv` | 可选字符串 | 互联路由通告拒绝。 |
| `options : ic-route-deny-learn` | 可选字符串 | 互联路由学习拒绝。 |
| `options : disable_garp_rarp` | 可选字符串，`true` 或 `false` | 禁用 GARP/RARP 通告。 |

**公共列：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `external_ids` | 字符串对映射 | 外部 ID。 |

**中转路由器：**

中转路由器用于互联功能。

## 详细信息

-   `ports`: `Logical_Router_Port` 的集合
    路由器的端口。

-   `static_routes`: `Logical_Router_Static_Route` 的集合
    路由器的零个或多个静态路由。

-   `policies`: `Logical_Router_Policy` 的集合
    路由器的零个或多个路由策略。

-   `enabled`: 可选布尔值
    此列用于管理性地设置路由器状态。如果此列为空或设置为 true，则路由器已启用。如果此列设置为 false，则路由器已禁用。禁用的路由器将丢弃所有入口和出口流量。

-   `nat`: `NAT` 的集合
    路由器的一个或多个 NAT 规则。NAT 规则仅在网关路由器上工作，或在具有且仅有一个分布式网关端口的分布式路由器上工作。

-   `load_balancer`: `Load_Balancer` 的弱引用集合
    关联到此逻辑路由器的负载均衡器集合。负载均衡器规则仅在网关路由器或具有且仅有一个分布式网关端口 (DGP) 的路由器上无限制地工作。当多个 DGP 与同一个网关 Chassis 关联时，负载均衡器也可以工作，这样该 Chassis 可以应用/维护连接跟踪状态而不会出现问题。要在 DGP 与不同网关 Chassis 关联的场景（例如 ECMP 路由）中使用负载均衡器，请考虑将负载均衡器选项列中的 `use_stateless_nat` 选项设置为 true。

-   `load_balancer_group`: `Load_Balancer_Group` 的集合
    关联到此逻辑路由器的负载均衡器组集合。

### 命名

这些列为逻辑路由器提供名称。从 OVN 的角度来看，这些名称除了方便人类与北向数据库交互之外，没有特殊的含义或目的。不要求名称唯一。（对于逻辑路由器的唯一标识符，请使用其行 UUID。）

（最初，`name` 旨在作为一个对人类友好的名称，但 Neutron 集成使用它来唯一标识自己的路由器对象，格式为 neutron-uuid。后来，Neutron 开始将路由器的友好名称传播为 `external_ids:neutron:router_name`。也许有一天这会被清理掉。）

-   `name`: 字符串
    逻辑路由器的名称。

-   `external_ids : neutron:router_name`: 可选字符串
    逻辑路由器的另一个名称。

-   `copp`: `Copp` 的可选弱引用
    来自 `Copp` 表的控制平面保护策略，用于对从该路由器的逻辑端口发送到 `ovn-controller` 的数据包进行计量。

### 选项

逻辑路由器的附加选项。

-   `options : chassis`: 可选字符串
    如果设置，表示该逻辑路由器是一个网关路由器（即集中式的），并驻留在设置的 Chassis 中。`ovn-controller` 也使用相同的值来唯一标识 OVN 部署中的 Chassis，该值来自 `Open_vSwitch` 数据库的 `Open_vSwitch` 表中的 `external_ids:system-id`。

    网关路由器只能通过交换机连接到分布式路由器，如果要在网关路由器中配置 SNAT 和 DNAT。

-   `options : dnat_force_snat_ip`: 可选字符串
    如果设置，指示一组 IP 地址，用于强制对已在网关路由器中进行 DNAT 的数据包进行 SNAT。当配置了多个网关路由器时，数据包可能进入任何一个网关路由器，被 DNAT 并最终到达逻辑交换机端口。为了使返回流量回到同一个网关路由器（以进行 unDNAT），数据包首先需要进行 SNAT。这可以通过使用特定于网关的一组 IP 地址设置上述选项来实现。此选项可以包含恰好一个 IPv4 和/或一个 IPv6 地址，以空格分隔。

-   `options : lb_force_snat_ip`: 可选字符串
    如果设置，此选项可以采用两种可能类型的值。要么是一组 IP 地址，要么是字符串值 `router_ip`。

    如果配置了一组 IP 地址，它指示用于强制对已在网关路由器中进行负载均衡的数据包进行 SNAT。当配置了多个网关路由器时，数据包可能进入任何一个网关路由器，作为负载均衡的一部分被 DNAT，并最终到达逻辑交换机端口。为了使返回流量回到同一个网关路由器（以进行 unDNAT），数据包首先需要进行 SNAT。这可以通过使用特定于网关的一组 IP 地址设置上述选项来实现。此选项可以包含恰好一个 IPv4 和/或一个 IPv6 地址，以空格分隔。

    如果配置为值 `router_ip`，则负载均衡的数据包将使用在做出路由决策后选择作为目标的路由器端口（连接到网关路由器）的 IP 进行 SNAT。

-   `options : mcast_relay`: 可选字符串，`true` 或 `false`
    启用/禁用连接到逻辑路由器的逻辑交换机之间的 IP 组播中继。默认值：False。

-   `options : dynamic_neigh_routers`: 可选字符串，`true` 或 `false`
    如果设置为 true，路由器将仅通过动态 ARP/ND 解析邻居路由器的 MAC 地址，而不是在 ARP/ND 解析阶段预先填充所有邻居路由器的静态映射。这减少了流的数量，但在需要时需要 ARP/ND 消息来解析 IP-MAC 绑定。默认为 false。当大量逻辑路由器连接到同一个逻辑交换机但大多数路由器之间从不需要发送流量时，建议设置为 true。默认情况下，`ovn-northd` 不会为 NAT 和负载均衡器地址创建映射。但是，对于添加了 `add_route` 选项的 NAT 和负载均衡器地址，`ovn-northd` 将创建将 NAT 和负载均衡器 IP 地址映射到适当 MAC 地址的逻辑流。将 `dynamic_neigh_routers` 设置为 true 将阻止自动创建这些逻辑流。

-   `options : always_learn_from_arp_request`: 可选字符串，`true` 或 `false`
    此选项控制处理 IPv4 ARP 请求或 IPv6 ND-NS 数据包时的行为 - 是否添加/更新动态邻居（MAC 绑定）条目。

    `true` - 总是学习 MAC-IP 绑定，并添加/更新 MAC 绑定条目。

    `false` - 如果该 IP 存在 MAC 绑定且 MAC 不同，或者，如果 ARP 请求的 TPA 属于此路由器上的任何路由器端口，则更新/添加该 MAC-IP 绑定。否则，不更新/添加条目。

    默认为 true。当大量逻辑路由器连接到同一个逻辑交换机但大多数路由器之间从不需要发送流量时，建议设置为 false，以减小 MAC 绑定表的大小。

-   `options : requested-tnl-key`: 可选字符串，包含 1 到 16,777,215 范围内的整数
    配置逻辑路由器的数据路径隧道密钥。这不是必需的，因为 `ovn-northd` 会自行分配一个唯一的密钥。但是，如果配置了，`ovn-northd` 会遵守配置的值。

-   `options : snat-ct-zone`: 可选字符串，包含 0 到 65,535 范围内的整数
    使用请求的连接跟踪区域对此路由器进行 SNAT。如果来自运行 OVN 的主机的出口流量既来自 OVN 又来自其他来源，这将非常有用。这样，OVN 和其他来源可以利用相同的连接跟踪区域。

-   `options : mac_binding_age_threshold`: 可选字符串
    指定基于 CIDR 的 MAC 绑定老化阈值，格式为：`entry[;entry[...]]`，其中每个条目的格式为：`[cidr:]threshold`。

    *   `cidr`: 可以是 IPv4 或 IPv6 CIDR。
    *   `threshold`: 以秒为单位的阈值。IP 地址匹配指定 CIDR 且超过此超时的 MAC 绑定将被自动删除。

    如果提供的条目没有 CIDR（仅阈值），它指定不匹配任何给定 CIDR 的 MAC 绑定的默认阈值。如果选项中有多个默认阈值条目，则行为未定义。

    如果有多个 CIDR 匹配 MAC 绑定 IP，则前缀长度最长的那个生效。如果选项中有多个具有相同 CIDR 的条目，则行为未定义。

    如果未找到匹配 MAC 绑定 IP 的 CIDR，且未指定默认阈值，则行为默认为原始行为：绑定不会基于时间被删除。

    该值也可以默认为空字符串，这意味着禁用老化阈值。任何不符合上述格式的字符串都被视为无效，并且禁用老化。

    示例：`192.168.0.0/16:300;192.168.10.0/24:0;fe80::/10:600;1200`

    这将为 192.168.0.0/16 范围内的 IP 地址的 MAC 绑定设置 300 秒的阈值，排除 192.168.1.0/24 范围（其老化被禁用），为 fe80::/10 IPv6 范围内的 IP 地址的 MAC 绑定设置 600 秒的阈值，并为所有其他 MAC 绑定设置 1200 秒的默认阈值。

-   `options : ct-zone-limit`: 可选字符串，包含 0 到 4,294,967,295 范围内的整数
    给定 Logical_Router 的 CT 区域限制值。值 0 表示无限制。当不存在该选项时，未设置限制，区域限制派生自 OvS 默认数据路径限制。

-   `options : dynamic-routing`: 可选字符串，`true` 或 `false`
    如果设置为 true，则此 `Logical_Router` 可以参与与 OVN 外部组件的动态路由。它将所有与路由器相关的路由同步到南向 `Advertised_Route` 表。这包括：

    *   所有由与此逻辑路由器关联的网络隐式创建的“连接”路由
    *   所有应用于此逻辑路由器的 `Logical_Router_Static_Route`

    用户需要使用以下设置来选择应通告的单个路由类型。参见：

    *   `Logical_Router` 上的 `options:dynamic-routing-redistribute`
    *   `Logical_Router_Port` 上的 `options:dynamic-routing-redistribute`

-   `options : dynamic-routing-redistribute`: 可选字符串
    仅当 `options:dynamic-routing` 设置为 true 时相关。

    这是由 `,` 分隔的元素列表。

    如果 `connected` 在列表中，则 northd 将所有“连接”路由同步到南向 `Advertised_Route` 表。“连接”在这里意味着由与 LRP 关联的网络隐式创建的路由。

    如果 `connected-as-host` 在列表中，则 northd 将枚举“连接”路由的所有正在使用的单个 IP，并将这些 IP 作为主机路由通告，而不是通告子网。这包括网络上的 LSP 和 LRP 地址以及此网络上的远程 `Logical_Router` 的 NAT 条目。设置此项隐含了设置 `connected`。此设置可用于：

    *   允许 OVN 外部的 fabric 丢弃发往实际未使用的 IP 地址的流量。否则，这些流量将到达此 LR 然后被丢弃。
    *   如果此 LR 有多个连接到不同 Chassis 上的 fabric 的 LRP：允许 OVN 外部的 fabric 将数据包引导到已经托管此支持 IP 地址的 Chassis。

    如果 `static` 在列表中，则 northd 将所有 `Logical_Router_Static_Route` 同步到南向 `Advertised_Route` 表。

    如果 `lb` 在列表中，则 northd 将在 `Advertised_Route` 表中为该路由器及其相邻路由器上的每个负载均衡器 VIP 创建条目。相邻路由器是那些直接连接的（通过逻辑路由器端口）或通过共享逻辑交换机连接的路由器。

    如果 `nat` 在列表中，则 northd 将在 `Advertised_Route` 表中为该路由器及其相邻路由器上的每个 NAT 外部 IP 创建条目。相邻路由器是那些直接连接的（通过逻辑路由器端口）或通过共享逻辑交换机连接的路由器。

    可以使用 `Logical_Router_Port` 上的 `options:dynamic-routing-redistribute` 在每个 LRP 基础上覆盖此值。

-   `options : dynamic-routing-redistribute-local-only`: 可选字符串，`true` 或 `false`
    仅当 `options:dynamic-routing` 设置为 true 时相关。

    这控制 `ovn-controller` 是否仅在绑定了 `tracked_port` 的 Chassis 上通告 `Advertised_Route` 记录。默认值：false。

-   `options : dynamic-routing-vrf-name`: 可选字符串
    仅当 `options:dynamic-routing` 设置为 true 时相关。

    这定义了 `ovn-controller` 将用于通告和学习路由的 VRF 的名称。如果未设置，VRF 将被命名为 "ovnvrf" 并附加 `options:dynamic-routing-vrf-id` 或逻辑路由器的数据路径 ID。

    VRF 名称必须是有效的 Linux 接口名称。如果太长，将使用生成的名称。

    VRF 表 ID 不受此设置影响。有关详细信息，请参阅 `Logical_Router` 上的 `options:dynamic-routing-maintain-vrf`。

-   `options : dynamic-routing-vrf-id`: 可选字符串，包含 1 到 4,294,967,295 范围内的整数
    仅当 `options:dynamic-routing` 设置为 true 时相关。

    这定义了指定 VRF 使用的 VRF 表 ID。当未设置或设置为无效值时，将使用逻辑路由器的数据路径 ID。

    这也可能影响 VRF 名称。有关详细信息，请参阅 `Logical_Router` 上的 `options:dynamic-routing-vrf-name`。

-   `options : dynamic-routing-no-learning`: 可选字符串，`true` 或 `false`
    仅当 `options:dynamic-routing` 设置为 true 时相关。

    此选项禁用特定路由器上的路由学习，并将删除已学习的路由。

-   `options : dynamic-routing-v4-prefix-nexthop`: 可选字符串
    仅当 `options:dynamic-routing` 设置为 true 时相关。

    这定义了 IPv4 通告路由使用的下一跳，而不是将路由通告为黑洞。这可以是有效的 IPv4 或 IPv6 地址。

-   `options : dynamic-routing-v6-prefix-nexthop`: 可选字符串
    仅当 `options:dynamic-routing` 设置为 true 时相关。

    这定义了 IPv6 通告路由使用的下一跳，而不是将路由通告为黑洞。这只能是有效的 IPv6 地址。

-   `options : ct-commit-all`: 可选字符串，`true` 或 `false`
    当启用时，LR 将在有状态的区域中提交流量。流量不会提交到两个区域，而是根据是否存在有状态 DNAT/SNAT 或两者进行选择。全部提交将防止 ct.inv 数据包的问题，因为它将防止回复流量的提交，这在某些情况下可能会发生。这也有助于 HWOL，因为对于已建立的会话不应有任何对 ct.new 的匹配，因为除了已经存在的有状态 NAT 和 LB 之外，我们还将提交所有内容。

-   `options : ic-route-filter-adv`: 可选字符串
    此选项期望逻辑路由器中存在以 "," 分隔的 CIDR 列表。如果路由的前缀属于列出的任何 CIDR，则将通告该路由。这允许在 ovn-ic 守护进程通告路由的过程中过滤 CIDR 前缀。

-   `options : ic-route-filter-learn`: 可选字符串
    此选项期望逻辑路由器中存在以 "," 分隔的 CIDR 列表。如果路由的前缀属于列出的任何 CIDR，则将学习该路由。这允许在 ovn-ic 守护进程学习路由的过程中过滤 CIDR 前缀。

-   `options : ic-route-deny-adv`: 可选字符串
    此选项期望逻辑路由器中存在以 "," 分隔的 CIDR 列表。如果路由的前缀属于列出的任何 CIDR，则不会通告该路由。这允许在 ovn-ic 守护进程通告路由的过程中过滤 CIDR 前缀。

-   `options : ic-route-deny-learn`: 可选字符串
    此选项期望逻辑路由器中存在以 "," 分隔的 CIDR 列表。如果路由的前缀属于列出的任何 CIDR，则不会学习该路由。这允许在 ovn-ic 守护进程学习路由的过程中过滤 CIDR 前缀。

-   `options : disable_garp_rarp`: 可选字符串，`true` 或 `false`
    如果设置为 true，则此逻辑路由器的所有 VIF 对等端口都不会发送 GARP 和 RARP 通告。默认值为 false。

### 公共列

-   `external_ids`: 字符串对映射
    参见本文件开头的外部 ID。

### 中转路由器 (Transit Router)

-   为了使 `Logical_Router` 获得中转路由器状态，至少需要有一个被认为是远程的 `Logical_Router_Port`。只有当 LRP 的 `options:requested-chassis` 设置为被认为是远程的 Chassis 时，LRP 才能是远程的。有关更多详细信息，请参阅 `Logical_Router_Port`。

-   为了使中转路由器正常工作，中转路由器本身及其路由器端口的所有隧道密钥需要在所有 AZ 中匹配，例如 AZ1 和 AZ2 中的 TR 需要具有相同的隧道密钥。AZ1 中 AZ2 的远程端口需要与 AZ2 中的本地端口具有相同的隧道密钥，反之亦然。路由器也可能具有非中转端口，没有设置 `options:requested-chassis`；它们的密钥也需要在所有 AZ 中匹配。

-   中转路由器的行为类似于分布式路由器，这意味着它对于有状态流（如 NAT 和 LB）具有相同的限制，并且它将在 AZ 之间丢失 CT 状态。

---

# Logical_Router_Port 表

L3 逻辑路由器中的端口。

必须恰好有一行 `Logical_Router` 引用给定的逻辑路由器端口。

## 摘要

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `name` | 字符串（表内必须唯一） | 端口名称。 |
| `networks` | 字符串集合 | IP 地址和掩码。 |
| `mac` | 字符串 | MAC 地址。 |
| `enabled` | 可选布尔值 | 启用状态。 |
| `dhcp_relay` | 可选 `DHCP_Relay` | DHCP 中继。 |

**分布式网关端口：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `ha_chassis_group` | 可选 `HA_Chassis_Group` | 高可用 Chassis 组。 |
| `gateway_chassis` | `Gateway_Chassis` 的集合 | 网关 Chassis。 |

**物理 VLAN MTU 问题的选项：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `options : reside-on-redirect-chassis` | 可选字符串，`true` 或 `false` | 是否驻留在重定向 Chassis 上。 |
| `options : redirect-type` | 可选字符串，`bridged` 或 `overlay` | 重定向类型。 |

**IPv6 前缀：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `ipv6_prefix` | 字符串集合 | IPv6 前缀。 |

**IPv6 RA 配置：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `ipv6_ra_configs : address_mode` | 可选字符串 | 地址模式。 |
| `ipv6_ra_configs : router_preference` | 可选字符串 | 路由器优先级。 |
| `ipv6_ra_configs : route_info` | 可选字符串 | 路由信息。 |
| `ipv6_ra_configs : mtu` | 可选字符串 | MTU。 |
| `ipv6_ra_configs : send_periodic` | 可选字符串 | 是否定期发送。 |
| `ipv6_ra_configs : max_interval` | 可选字符串 | 最大间隔。 |
| `ipv6_ra_configs : min_interval` | 可选字符串 | 最小间隔。 |
| `ipv6_ra_configs : rdnss` | 可选字符串 | RDNSS 服务器。 |
| `ipv6_ra_configs : dnssl` | 可选字符串 | DNS 搜索列表。 |

**选项：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `options : mcast_flood` | 可选字符串，`true` 或 `false` | 组播泛洪。 |
| `options : requested-tnl-key` | 可选字符串，1 到 32,767 的整数 | 请求的隧道密钥。 |
| `options : prefix_delegation` | 可选字符串，`true` 或 `false` | 前缀代理。 |
| `options : prefix` | 可选字符串，`true` 或 `false` | 前缀。 |
| `options : route_table` | 可选字符串 | 路由表。 |
| `options : gateway_mtu` | 可选字符串，68 到 65,535 的整数 | 网关 MTU。 |
| `options : routing-protocol-redirect` | 可选字符串 | 路由协议重定向。 |
| `options : routing-protocols` | 可选字符串 | 路由协议。 |
| `options : gateway_mtu_bypass` | 可选字符串 | 网关 MTU 绕过。 |
| `options : ic-route-tag` | 可选字符串 | 互联路由标记。 |
| `options : ic-route-filter-tag` | 可选字符串 | 互联路由过滤标记。 |
| `options : requested-chassis` | 可选字符串 | 请求的 Chassis。 |
| `options : dynamic-routing-redistribute` | 可选字符串 | 动态路由重分发。 |
| `options : dynamic-routing-redistribute-local-only` | 可选字符串，`true` 或 `false` | 仅重分发本地路由。 |
| `options : dynamic-routing-maintain-vrf` | 可选字符串，`true` 或 `false` | 动态路由维护 VRF。 |
| `options : dynamic-routing-port-name` | 可选字符串 | 动态路由端口名称。 |
| `options : ic-route-filter-adv` | 可选字符串 | 互联路由通告过滤。 |
| `options : ic-route-filter-learn` | 可选字符串 | 互联路由学习过滤。 |
| `options : ic-route-deny-adv` | 可选字符串 | 互联路由通告拒绝。 |
| `options : ic-route-deny-learn` | 可选字符串 | 互联路由学习拒绝。 |
| `options : dynamic-routing-no-learning` | 可选字符串，`true` 或 `false` | 禁用动态路由学习。 |

**连接：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `peer` | 可选字符串 | 对等端口。 |

**公共列：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `external_ids` | 字符串对映射 | 外部 ID。 |

**状态：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `status : hosting-chassis` | 可选字符串 | 托管 Chassis。 |

## 详细信息

-   `name`: 字符串（表内必须唯一）
    逻辑路由器端口的名称。

    除了为人类与北向数据库交互提供方便之外，此列还被其补丁端口在 `Logical_Switch_Port` 中引用，或者在 `Logical_Router_Port` 中被另一个逻辑路由器端口引用。

    逻辑路由器端口不能与逻辑交换机端口同名，但数据库模式无法强制执行此操作。

-   `networks`: 字符串集合
    路由器的 IP 地址和掩码。例如，`192.168.0.1/24` 表示路由器的 IP 地址是 192.168.0.1，并且发往 192.168.0.x 的数据包应路由到此端口。这些是可选的。

    逻辑路由器端口总是自动添加从接口 MAC 地址生成的链路本地 IPv6 地址 (fe80::/64)，使用修改后的 EUI-64 格式。

-   `mac`: 字符串
    属于此路由器端口的以太网地址。

-   `enabled`: 可选布尔值
    此列用于管理性地设置端口状态。如果此列为空或设置为 true，则端口已启用。如果此列设置为 false，则端口已禁用。禁用的端口将丢弃所有入口和出口流量。

-   `dhcp_relay`: 可选 `DHCP_Relay`
    此列用于启用 DHCP 中继。请参阅 `DHCP_Relay` 表。

### 分布式网关端口

网关（如 OVN 架构指南中的“网关”部分所述）在逻辑网络和物理网络之间提供有限的连接。OVN 支持多种类型的网关。`Logical_Router_Port` 表可以用两种不同的方式来配置分布式网关端口，这是一种网关。这两种配置形式的存在是由于历史原因。它们在实践中产生相同类型的 OVN 南向记录和相同的行为。

如果设置了其中任何一个，此逻辑路由器端口代表一个分布式网关端口，该端口将此路由器连接到具有 localnet 端口的逻辑交换机或连接到另一个 OVN 部署。

OVN 架构指南中也提到，分布式网关端口也可以用于可扩展性原因，在逻辑交换机专用于 Chassis 而不是分布式的部署中。

配置网关的首选方法是 `ha_chassis_group`，但也支持 `gateway_chassis` 以向后兼容。在给定的 LRP 上应该只设置其中一个，因为它们配置相同的功能。

即使配置了网关，逻辑路由器端口实际上仍然驻留在每个 Chassis 上。但是，由于物理网络中使用 L2 学习的影响，以及支持一对多 NAT（即 IP 伪装）等高级功能的需要，逻辑路由器处理的子集在网关 Chassis 上以集中方式处理。

每个逻辑路由器上可以配置多个分布式网关端口，每个连接到不同的 L2 网段。具有多个分布式网关端口的逻辑路由器尚不支持负载均衡。

对于每个分布式网关端口，它可以有多个网关 Chassis。当指定多个网关 Chassis 时，OVN 一次只使用一个。OVN 可以依靠 OVS BFD 实现来监控网关连接性，优先选择在线的最高优先级网关。优先级在 `Gateway_Chassis` 或 `HA_Chassis` 的 priority 列中指定。

`ovn-northd` 将 LRP 的 LR 中指定的 `external_mac` 规则编程到 `logical_port` 所在的 Chassis 上的对等逻辑交换机的目标查找中。此外，逻辑路由器的 MAC 地址会自动编程到网关 Chassis 上的对等逻辑交换机的目标查找流中。如果希望为 NAT 地址生成免费 ARP，则将对等 LSP 的 `options:nat-addresses` 设置为 `router`。

OVN 20.03 及更早版本支持第三种配置分布式网关端口的方法，即使用 `options:redirect-chassis` 指定网关 Chassis。此方法不再受支持。任何剩余用户应切换到较新的方法之一。可以通过命令行轻松配置 `gateway_chassis`，例如 `ovn-nbctl lrp-set-gateway-chassis lrp chassis`。

-   `ha_chassis_group`: 可选 `HA_Chassis_Group`
    指定一个 `HA_Chassis_Group` 以提供网关高可用性。

-   `gateway_chassis`: `Gateway_Chassis` 的集合
    为逻辑路由器端口指定一个或多个 `Gateway_Chassis`。

### 物理 VLAN MTU 问题的选项

在将隧道与桥接到物理 VLAN 的逻辑网络混合使用时会出现 MTU 问题。有关 MTU 问题的解释，请参阅 OVN 架构文档中的“物理 VLAN MTU 问题”。以下选项（互为替代）提供了解决方案。它们都会导致数据包通过 localnet 发送而不是隧道，但它们在是否发送部分或全部数据包方面有所不同。这些选项之间最显著的权衡是 `reside-on-redirect-chassis` 更易于配置，而 `redirect-type` 对东西向流量的性能更好。

-   `options : reside-on-redirect-chassis`: 可选字符串，`true` 或 `false`
    如果设置为 true，此选项强制通过逻辑路由器端口的所有流量使用跨 localnet 端口的跳跃通过网关 Chassis。这以两种方式改变行为：

    *   如果没有此选项，东西向流量直接在源和目标 Chassis 之间传递（甚至在单个 Chassis 内，对于共置的 VM）。有了此选项，所有东西向流量都通过网关 Chassis。
    *   如果没有此选项，网关 Chassis 和其他 Chassis 之间的流量封装在隧道中。有了此选项，流量通过 localnet 接口传递。

    此选项仅在将分布式逻辑路由器连接到具有 VIF 的逻辑交换机的逻辑路由器端口上设置才有用。它不应在分布式网关端口上设置。

    只有当逻辑路由器具有且仅有一个分布式网关端口，并且 LRP 的对等交换机具有 localnet 端口时，OVN 才会遵守此选项。

-   `options : redirect-type`: 可选字符串，`bridged` 或 `overlay`
    如果在分布式网关端口上设置为 `bridged`，此选项会导致 OVN 通过 localnet 端口而不是隧道将数据包重定向到网关 Chassis。相关 Chassis 必须共享一个 localnet 端口。

    此功能要求管理员或 CMS 通过在 Open vSwitch 数据库中设置 `ovn-chassis-mac-mappings` 为每个参与的 Chassis 配置逻辑路由器的唯一以太网地址，供 `ovn-controller` 使用。

    将此选项设置为 `overlay` 或将其保留未设置没有任何效果。此选项仅在逻辑路由器上具有且仅有一个分布式网关端口时，在分布式网关端口上设置才有用。否则将被忽略。

-   `ipv6_prefix`: 字符串集合
    此列包含根据 RFC 3633 由前缀代理路由器获得的 IPv6 前缀。

### `ipv6_ra_configs`

此列定义 `ovn-controller` 在回复 IPv6 路由器请求时要包含的 IPv6 ND RA 地址模式和 ND MTU 选项。

-   `ipv6_ra_configs : address_mode`: 可选字符串
    用于 IPv6 地址配置的地址模式。支持的值为：

    *   `slaac`: 使用路由器通告 (RA) 数据包的地址配置。`Logical_Router_Port` 表的 networks 列中定义的 IPv6 前缀将包含在 RA 的 ICMPv6 选项 - 前缀信息中。
    *   `dhcpv6_stateful`: 使用 DHCPv6 的地址配置。
    *   `dhcpv6_stateless`: 使用路由器通告 (RA) 数据包的地址配置。其他 IPv6 选项由 DHCPv6 提供。

-   `ipv6_ra_configs : router_preference`: 可选字符串
    默认路由器优先级 (PRF) 指示是否优先选择此路由器而不是其他默认路由器 (RFC 4191)。可能的值为：

    *   `HIGH`: 映射到 RA PRF 字段中的 0x01
    *   `MEDIUM`: 映射到 RA PRF 字段中的 0x00
    *   `LOW`: 映射到 RA PRF 字段中的 0x11

-   `ipv6_ra_configs : route_info`: 可选字符串
    路由信息用于根据 RFC 4191 配置路由器通告中发送的路由信息选项。路由信息是一个逗号分隔的字符串，其中每个字段提供给定路由的 PRF 和前缀（例如：`HIGH-aef1::11/48,LOW-aef2::11/96`）。可能的 PRF 值为：

    *   `HIGH`: 映射到 RA PRF 字段中的 0x01
    *   `MEDIUM`: 映射到 RA PRF 字段中的 0x00
    *   `LOW`: 映射到 RA PRF 字段中的 0x11

-   `ipv6_ra_configs : mtu`: 可选字符串
    链路的推荐 MTU。默认值为 0，这意味着 `ovn-controller` 回复的 RA 数据包中不包含 MTU 选项。根据 RFC 2460，建议 mtu 值不小于 1280，因此任何小于 1280 的 mtu 值都将被视为无 MTU 选项。

-   `ipv6_ra_configs : send_periodic`: 可选字符串
    如果设置为 true，则此路由器接口将定期发送路由器通告。如果未设置 `ipv6_ra_configs:address_mode`，则此选项无效。默认值为 false。

-   `ipv6_ra_configs : max_interval`: 可选字符串
    发送定期路由器通告之间等待的最大秒数。如果 `ipv6_ra_configs:send_periodic` 为 false，则此选项无效。默认值为 600。

-   `ipv6_ra_configs : min_interval`: 可选字符串
    发送定期路由器通告之间等待的最小秒数。如果 `ipv6_ra_configs:send_periodic` 为 false，则此选项无效。默认值为 `ipv6_ra_configs:max_interval` 的三分之一，即如果该键未设置，则为 200 秒。

-   `ipv6_ra_configs : rdnss`: 可选字符串
    RA 数据包中通告的 RDNSS 服务器的 IPv6 地址。目前 OVN 仅支持一个 RDNSS 服务器。

-   `ipv6_ra_configs : dnssl`: 可选字符串
    RA 数据包中通告的 DNS 搜索列表。多个 DNS 搜索列表必须以“逗号”分隔（例如 "a.b.c, d.e.f"）。

### 选项

逻辑路由器端口的附加选项。

-   `options : mcast_flood`: 可选字符串，`true` 或 `false`
    如果设置为 true，则组播流量（包括报告）将无条件转发到特定端口。

    当端口是 `options:mcast_relay` 设置为 true 的逻辑路由器的一部分时，此选项适用。

    默认值：false。

-   `options : requested-tnl-key`: 可选字符串，包含 1 到 32,767 范围内的整数
    配置端口的端口绑定隧道密钥。通常这不是必需的，因为 `ovn-northd` 会自行分配一个唯一的密钥。但是，如果配置了，`ovn-northd` 会遵守配置的值。

-   `options : prefix_delegation`: 可选字符串，`true` 或 `false`
    如果设置为 true，则在此逻辑路由器端口上启用 IPv6 前缀代理状态机 (RFC3633)。IPv6 前缀代理仅在网关路由器或网关路由器端口上可用。

-   `options : prefix`: 可选字符串，`true` 或 `false`
    如果设置为 true，此接口将根据 RFC3663 接收 IPv6 前缀。

-   `options : route_table`: 可选字符串
    指定具有指定 `route_table` 值的 `Logical_Router_Static_Route` 的查找。有关详细信息，请参阅 `Logical_Router_Static_Route:route_table` 字段描述中的路由查找行为。

-   `options : gateway_mtu`: 可选字符串，包含 68 到 65,535 范围内的整数
    如果设置，将向路由器管道添加逻辑流以检查数据包长度。如果数据包长度大于设置的值，将生成 ICMPv4 类型 3（目标不可达）代码 4（需要分片且设置了不分片位）或 ICMPv6 类型 2（数据包太大）代码 0（无到达目标的路由）数据包。这允许路径 MTU 发现。

-   `options : routing-protocol-redirect`: 可选字符串
    此选项期望对等逻辑交换机中存在的逻辑交换机端口的名称。如果设置，它会导致任何发往逻辑路由器端口 IP 地址（包括其 IPv6 LLA）和 `routing-protocols` 选项中定义的与路由协议关联的端口的流量被重定向到指定的逻辑交换机端口。这允许外部路由守护进程绑定到 OVN 逻辑交换机中的端口，并像监听逻辑路由器端口的 IP 地址一样行动。

-   `options : routing-protocols`: 可选字符串
    此选项期望一个逗号分隔的路由和路由相关协议列表，其控制平面流量将被重定向到 `routing-protocol-redirect` 选项中指定的端口。目前支持的选项有：

    *   BGP (转发 TCP 端口 179)
    *   BFD (转发 UDP 端口 3784)

    注意，为了使 BGP 在“无编号模式”下工作（通过 IPv6 网络通告 IPv4 路由，具有自动链路对等发现），逻辑路由器端口需要启用定期 IPv6 路由器通告的发送（参见 `ipv6_ra_configs:send_periodic`）。BGP 无编号的推荐最小定期 RA 配置：

    *   `ipv6_ra_configs:address_mode` = slaac (任何有效值都可以，但需要设置该选项)
    *   `ipv6_ra_configs:send_periodic` = true
    *   `ipv6_ra_configs:max_interval` = 10
    *   `ipv6_ra_configs:min_interval` = 5

    请随意调整最大和最小间隔值，但请注意它们会影响建立初始 BGP 会话的速度。使用上述值，会话将在 5 到 10 秒内建立。有关通过 IPv6 下一跳地址通告 IPv4 网络的更多详细信息，请参阅 RFC 8950。

-   `options : gateway_mtu_bypass`: 可选字符串
    配置时，表示一个匹配表达式，使用与 OVN 南向数据库 `Logical_Flow` 表中的 match 列相同的表达式语言。匹配此表达式的数据包将绕过通过 `options:gateway_mtu` 选项配置的长度检查。

-   `options : ic-route-tag`: 可选字符串
    此选项期望 `Logical Router Port` 中存在的路由标记名称。如果设置，它会导致 `Logical Router Port` 通告的任何路由在 `OVN_IC_Southbound` 数据库的 `Route` 表的通告路由条目的 `external_ids` 寄存器中包含该路由标记。这允许在 ovn-ic 守护进程通告和学习路由的过程中标记和过滤路由标记。

-   `options : ic-route-filter-tag`: 可选字符串
    此选项期望 `Logical Router Port` 中存在的过滤路由标记名称。如果设置，它会导致 `Logical Router Port` 学习的任何路由（如果 `OVN_IC_Southbound` 数据库的 `Route` 表的通告路由条目的 `external_ids` 寄存器中存在该路由标记）将被过滤，并且不会被 ovn-ic 守护进程学习。

-   `options : requested-chassis`: 可选字符串
    如果设置，标识允许绑定此端口的特定 Chassis（通过名称或主机名）。此选项仅对具有 `options:is-remote=true` 的 Chassis 有效，换句话说，对于位于不同可用性区域的 Chassis。该选项仅接受单个值。

    通过分配远程 Chassis，`Logical_Router` 获得中转路由器状态，有关更多详细信息，请参阅 `Logical_Router` 表。

-   `options : dynamic-routing-redistribute`: 可选字符串
    仅当相应 `Logical_Router` 上的 `options:dynamic-routing` 设置为 true 时相关。

    这是由 `,` 分隔的元素列表。

    如果 `connected` 在列表中，则 northd 将所有“连接”路由同步到南向 `Advertised_Route` 表。“连接”在这里意味着由与 LRP 关联的网络隐式创建的路由。

    如果 `connected-as-host` 在列表中，则 northd 将枚举“连接”路由的所有正在使用的单个 IP，并将这些 IP 作为主机路由通告，而不是通告子网。这包括网络上的 LSP 和 LRP 地址以及此网络上的远程 `Logical_Router` 的 NAT 条目。设置此项隐含了设置 `connected`。此设置可用于：

    *   允许 OVN 外部的 fabric 丢弃发往实际未使用的 IP 地址的流量。否则，这些流量将到达此 LR 然后被丢弃。
    *   如果此 LR 有多个连接到不同 Chassis 上的 fabric 的 LRP：允许 OVN 外部的 fabric 将数据包引导到已经托管此支持 IP 地址的 Chassis。

    如果 `static` 在列表中，则 northd 将所有 `Logical_Router_Static_Route` 同步到南向 `Advertised_Route` 表。

    如果 `lb` 在列表中，则 northd 将在 `Advertised_Route` 表中为该端口的路由器及其相邻路由器上的每个负载均衡器 VIP 创建条目。相邻路由器是那些直接连接到此逻辑路由器端口的路由器，或通过共享逻辑交换机连接的路由器。

    如果 `nat` 在列表中，则 northd 将在 `Advertised_Route` 表中为该端口的路由器及其相邻路由器上的每个 NAT 外部 IP 创建条目。相邻路由器是那些直接连接到此逻辑路由器端口的路由器，或通过共享逻辑交换机连接的路由器。

    如果未设置，将使用 `Logical_Router` 上的 `options:dynamic-routing-redistribute` 的值。

-   `options : dynamic-routing-redistribute-local-only`: 可选字符串，`true` 或 `false`
    仅当 `options:dynamic-routing` 设置为 true 时相关。

    这控制 `ovn-controller` 是否仅在绑定了 `tracked_port` 的 Chassis 上通告 `Advertised_Route` 记录。

    如果未设置，将使用 `Logical_Router` 上的 `options:dynamic-routing-redistribute-local-only` 的值。

-   `options : dynamic-routing-maintain-vrf`: 可选字符串，`true` 或 `false`
    仅当相应 `Logical_Router` 上的 `options:dynamic-routing` 设置为 true 时相关。

    如果此 LRP 绑定到特定 Chassis，则此 Chassis 的 `ovn-controller` 将维护一个 VRF。此 VRF 将包含从此 LRP 通告的所有路由。除非设置了 `options:dynamic-routing-vrf-name`，否则 VRF 将被命名为 "ovnvrf" 并附加 `options:dynamic-routing-vrf-id` 或逻辑路由器的数据路径 ID。

    如果设置未设置或为 false，则 `ovn-controller` 将期望此 VRF 已经存在。OVN 外部的一些工具需要确保这一点。

    VRF 表 ID 与 `Logical_Router` 数据路径的隧道密钥相同。如果此设置为 false，则 OVN 外部的工具需要确保情况如此。

-   `options : dynamic-routing-port-name`: 可选字符串
    仅当相应 `Logical_Router` 上的 `options:dynamic-routing` 设置为 true 时相关。仅学习与此处指定的 LSP 或 LRP 本地绑定的接口关联的路由。这允许单个 Chassis 学习绑定到此 Chassis 的单独 LRP 上的不同路由。例如，在具有多个通往网络 fabric 的链路的 Chassis 的情况下，所有链路都单独运行 BGP，这是非常有用的。此选项允许在单个 LRP 和单个链路之间建立 1:1 映射。如果此名称引用的端口本地绑定在 `ovn-controller` 上，我们会查找此端口的 linux 接口名称。然后使用此接口名称进行路由过滤，因此只会学习以该接口为下一跳的路由。由于 `ovn-controller` 上可能并不总是绑定这样的端口，因此该值也可以是任意字符串。`ovn-controller` 将在本地 `external_ids:dynamic-routing-port-mapping` 中查找端口名称。这是一个以 `,` 分隔的列表，包含 `port=interfacename` 对。如果在其中找到匹配项，则使用配置的接口名称代替自动发现。而且此时端口是否本地绑定也变得无关紧要。

-   `options : ic-route-filter-adv`: 可选字符串
    此选项期望 `Logical Router Port` 中存在以 "," 分隔的 CIDR 列表。如果路由的前缀属于列出的任何 CIDR，则将通告该路由。这允许在 ovn-ic 守护进程通告路由的过程中过滤 CIDR 前缀。

-   `options : ic-route-filter-learn`: 可选字符串
    此选项期望 `Logical Router Port` 中存在以 "," 分隔的 CIDR 列表。如果路由的前缀属于列出的任何 CIDR，则将学习该路由。这允许在 ovn-ic 守护进程学习路由的过程中过滤 CIDR 前缀。

-   `options : ic-route-deny-adv`: 可选字符串
    此选项期望 `Logical Router Port` 中存在以 "," 分隔的 CIDR 列表。如果路由的前缀属于列出的任何 CIDR，则不会通告该路由。这允许在 ovn-ic 守护进程通告路由的过程中过滤 CIDR 前缀。

-   `options : ic-route-deny-learn`: 可选字符串
    此选项期望 `Logical Router Port` 中存在以 "," 分隔的 CIDR 列表。如果路由的前缀属于列出的任何 CIDR，则不会学习该路由。这允许在 ovn-ic 守护进程学习路由的过程中过滤 CIDR 前缀。

-   `options : dynamic-routing-no-learning`: 可选字符串，`true` 或 `false`
    仅当 `options:dynamic-routing` 设置为 true 时相关。

    此选项禁用特定路由器端口上的路由学习，并将删除已学习的路由。它也优先于此选项的路由器版本。

### 连接 (Attachment)

给定的路由器端口有两个用途之一：

*   将逻辑交换机连接到逻辑路由器。这种类型的逻辑路由器端口恰好被一个类型为 `router` 的 `Logical_Switch_Port` 引用。`name` 的值在 `Logical_Switch_Port` 的 `options` 列中设置为 `router-port`。在这种情况下，`peer` 列为空。
*   将一个逻辑路由器连接到另一个。这需要一对逻辑路由器端口，每个连接到不同的路由器。对中的每个路由器端口在其 `peer` 列中指定另一个。没有 `Logical_Switch` 引用该路由器端口。

-   `peer`: 可选字符串
    对于用于连接两个逻辑路由器的路由器端口，这通过名称标识对中的另一个路由器端口。

    对于连接到逻辑交换机的路由器端口，此列为空。

### 公共列

-   `external_ids`: 字符串对映射
    参见本文件开头的外部 ID。

    `ovn-northd` 程序将所有这些对复制到 `OVN_Southbound` 数据库中 `Port_Binding` 表的 `external_ids` 列。

### 状态

有关逻辑路由器端口的附加状态。

-   `status : hosting-chassis`: 可选字符串
    此选项由 `ovn-northd` 填充。

    当分布式网关端口绑定到 OVN 南向数据库 `Port_Binding` 中的某个位置时，`ovn-northd` 将使用当前托管此端口的 Chassis 名称填充此键。

---

# Logical_Router_Static_Route 表

每条记录代表一条静态路由。

当多条路由匹配数据包时，选择最长前缀匹配。对于给定的前缀长度，`dst-ip` 路由优先于 `src-ip` 路由。

当存在 ECMP 路由时，即具有相同前缀和策略的多条路由，将根据数据包头的 5 元组哈希选择其中一条。

## 摘要

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `ip_prefix` | 字符串 | 路由的 IP 前缀。 |
| `policy` | 可选字符串，`dst-ip` 或 `src-ip` | 路由策略。 |
| `nexthop` | 字符串 | 下一跳 IP。 |
| `output_port` | 可选字符串 | 输出端口。 |
| `bfd` | `BFD` 的可选弱引用 | BFD 会话引用。 |
| `selection_fields` | 字符串集合 | ECMP 选择字段。 |
| `route_table` | 字符串 | 路由表名称。 |
| `external_ids : ic-learned-route` | 可选字符串 | 是否为 IC 学习的路由。 |

**公共列：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `external_ids` | 字符串对映射 | 外部 ID。 |

**公共选项：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `options` | 字符串对映射 | 通用选项。 |
| `options : ecmp_symmetric_reply` | 可选字符串 | ECMP 对称回复。 |
| `options : origin` | 可选字符串 | 路由来源。 |

## 详细信息

-   `ip_prefix`: 字符串
    此路由的 IP 前缀（例如 192.168.100.0/24）。

-   `policy`: 可选字符串，`dst-ip` 或 `src-ip`
    如果指定，此设置描述用于做出路由决策的策略。此设置必须是以下字符串之一：

    *   `src-ip`: 当数据包的源 IP 地址匹配 `ip_prefix` 时，此策略将数据包发送到下一跳。
    *   `dst-ip`: 当数据包的目标 IP 地址匹配 `ip_prefix` 时，此策略将数据包发送到下一跳。

    如果未指定，默认为 `dst-ip`。

-   `nexthop`: 字符串
    此路由的下一跳 IP 地址。下一跳 IP 地址应该是连接的路由器端口的 IP 地址或逻辑端口的 IP 地址，或者可以设置为 `discard` 以丢弃匹配给定路由的数据包。

-   `output_port`: 可选字符串
    需要发送数据包的 `Logical_Router_Port` 的名称。这是可选的，当未指定时，OVN 将根据下一跳自动计算出来。当指定了此项并且路由器端口上有多个 IP 地址且没有一个在下一跳的同一子网中时，OVN 选择第一个 IP 地址作为下一跳可达的 IP 地址。

-   `bfd`: `BFD` 的可选弱引用
    如果路由关联了 BFD 会话，则引用 BFD 行。

-   `selection_fields`: 字符串集合，`eth_dst`, `eth_src`, `ip_dst`, `ip_proto`, `ip_src`, `ipv6_dst`, `ipv6_src`, `tp_dst`, 或 `tp_src` 之一
    ECMP 路由使用类型为 `select` 的 OpenFlow 组在可用下一跳列表中选择下一跳。OVS 支持两种选择方法：`dp_hash` 和 `hash`，用于哈希计算和选择组的桶。OVN 默认使用 `dp_hash`。为了使用 `hash` 选择方法，请在 `selection_fields` 中指定允许的匹配字段。有关选择方法和字段的更多详细信息，请参阅 OVS 文档 (man ovs-ofctl)。

    要匹配第 4 层端口，请使用 `tp_src` 和 `tp_dst`。此匹配仅适用于 TCP、UDP、SCTP，对于所有其他 IP 数据包将被忽略。当匹配第 4 层端口时，将在 select 动作中隐式添加对 `ip_proto` 的匹配。

    示例：`{ip_proto,ip_src,ip_dst}` 用于 3 元组匹配。
    示例：`{ip_proto,ipv6_src,ipv6_dst}` 用于 IPv6 匹配。
    示例：`{ip_proto,ip_src,ip_dst,tp_src,tp_dst}`。
    示例：`{ip_src,ip_dst,ipv6_src,ipv6_dst,tp_src,tp_dst}`。

-   `route_table`: 字符串
    指定任何字符串以将路由分配给单独的路由表。当 `Logical_Router_Port` 在 `options:route_table` 中配置了值时，仅考虑具有相同路由表值的静态路由。下面提供了路由查找行为的更详细描述：

    当数据包进入逻辑路由器 (LR) 时，它会检查以下路由列表：

    *   LR 的所有直接连接网络的路由（包括通过 OVN-IC 从同一 LR 内的其他可用性区域学习的网络）。
    *   具有与逻辑路由器端口的 `options:route_table` 相同的 `route_table` 字段值的 LR 的所有静态路由（如果该选项不存在，则考虑具有空 `route_table` 字段的静态路由）。

    从结果路由列表中，具有最长前缀匹配的路由优先于其他路由。

-   `external_ids : ic-learned-route`: 可选字符串
    如果路由是从全局 `OVN_IC_Southbound` 数据库学习的，则 `ovn-ic` 填充此键。在这种情况下，该值将设置为 `OVN_IC_Southbound` 数据库的 `Route` 表中行的 uuid。

### 公共列

-   `external_ids`: 字符串对映射
    参见本文件开头的外部 ID。

### 公共选项

-   `options`: 字符串对映射
    此列提供通用键/值设置。下面单独描述支持的选项。

-   `options : ecmp_symmetric_reply`: 可选字符串
    如果为 true，则

    *   通过此路由到达的新的入口发起流量将使其回复流量绕过 ECMP 路由选择，并改为通过此路由发出。
    *   对于出口发起流量，入口回复流量路由被保存。后续流量将绕过 ECMP 路由选择，并改为通过同一路由发出。

    注意，此选项覆盖 `Logical_Router_policy` 表中设置的任何规则。此选项仅在网关路由器（设置了 `options:chassis` 的路由器）上工作。

-   `options : origin`: 可选字符串
    如果 ovn-interconnection 已学习此路由，它将设置其来源：`connected` 或 `static`。此键应该仅由 `ovn-ic` 守护进程写入。`ovn-northd` 在生成逻辑流时检查此值。同一逻辑路由器内具有相同 `ip_prefix` 的 `Logical_Router_Static_Route` 记录将具有基于 `origin` 键值的下一个查找顺序：

    1.  connected
    2.  static

---

# Logical_Router_Policy 表

此表中的每一行代表逻辑路由器的一个路由策略，逻辑路由器通过其 `policies` 列指向该策略。此表中最高优先级匹配行的 `action` 列决定了数据包的处理方式。如果没有行匹配，则默认允许数据包。（可以进行默认拒绝处理：添加一条优先级为 0、匹配为 1、动作为 drop 的规则。）

## 摘要

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `priority` | 0 到 32,767 的整数 | 优先级。 |
| `chain` | 可选字符串 | 链名称。 |
| `match` | 字符串 | 匹配条件。 |
| `action` | 字符串，`allow`, `drop`, `jump`, 或 `reroute` | 动作。 |
| `jump_chain` | 可选字符串 | 跳转链名称。 |
| `nexthop` | 可选字符串 | 下一跳 IP（已弃用）。 |
| `nexthops` | 字符串集合 | 下一跳 ECMP IP 列表。 |
| `output_port` | `Logical_Router_Port` 的可选弱引用 | 输出端口。 |
| `bfd_sessions` | `BFD` 的弱引用集合 | BFD 会话。 |
| `options : pkt_mark` | 可选字符串 | 数据包标记。 |

**公共列：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `external_ids` | 字符串对映射 | 外部 ID。 |

## 详细信息

-   `priority`: 0 到 32,767 的整数
    路由策略的优先级。数值较高的规则优先于数值较低的规则。规则由优先级、链和匹配字符串唯一标识。

-   `chain`: 可选字符串
    路由策略规则的链名称。默认情况下仅遍历具有空链名称的规则。其他链作为跳转动作的响应进行遍历。

-   `match`: 字符串
    路由策略应匹配的数据包，使用与 OVN 南向数据库 `Logical_Flow` 表中的 match 列相同的表达式语言。

    默认情况下允许所有流量。在编写更严格的策略时，重要的是要记住允许 ARP 和 IPv6 邻居发现数据包等流量。

-   `action`: 字符串，`allow`, `drop`, `jump`, 或 `reroute`
    路由策略匹配时采取的动作：

    *   `allow`: 转发数据包。
    *   `drop`: 默默丢弃数据包。
    *   `reroute`: 将数据包重路由到下一跳或多个下一跳。
    *   `jump`: 开始检查具有与 `jump_chain` 中指定值相同的链值的规则。

-   `jump_chain`: 可选字符串
    选择接下来要检查的路由策略规则的链名称。

-   `nexthop`: 可选字符串
    注意：此列已弃用，建议使用 `nexthops`，并且被 northd 忽略。

    此路由的下一跳 IP 地址，应该是连接的路由器端口的 IP 地址或逻辑端口的 IP 地址。

-   `nexthops`: 字符串集合
    此路由的下一跳 ECMP IP 地址。列表中的每个 IP 应该是连接的路由器端口的 IP 地址或逻辑端口的 IP 地址。

    从列表中选择一个 IP 作为下一跳。

-   `output_port`: `Logical_Router_Port` 的可选弱引用
    需要发送数据包的 `Logical_Router_Port`。这是可选的，当未指定时，OVN 将根据 `nexthops` 列自动计算出来。当指定了此项并且路由器端口上有多个 IP 地址且没有一个在 `nexthops` 的同一子网中时，OVN 选择第一个 IP 地址作为下一跳可达的 IP 地址。注意：目前 OVN 不支持在 ECMP 重路由策略（`nexthops` 列中有多个值）上配置输出端口。

-   `bfd_sessions`: `BFD` 的弱引用集合
    如果路由策略关联了一些 BFD 会话，则引用 BFD 行。

-   `options : pkt_mark`: 可选字符串
    在应用路由策略时用指定的值标记数据包。如果需要，CMS 可以检查此数据包标记并做出一些决定。当数据包发出时，此值不会保留。

### 公共列

-   `external_ids`: 字符串对映射
    参见本文件开头的外部 ID。

---

# NAT 表

每条记录代表一个 NAT 规则。

## 摘要

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `type` | 字符串，`dnat`, `dnat_and_snat`, 或 `snat` | NAT 类型。 |
| `external_ip` | 字符串 | 外部 IP。 |
| `external_mac` | 可选字符串 | 外部 MAC。 |
| `external_port_range` | 字符串 | 外部端口范围。 |
| `logical_ip` | 字符串 | 逻辑 IP。 |
| `logical_port` | 可选字符串 | 逻辑端口。 |
| `allowed_ext_ips` | 可选 `Address_Set` | 允许的外部 IP 集合。 |
| `exempted_ext_ips` | 可选 `Address_Set` | 豁免的外部 IP 集合。 |
| `gateway_port` | `Logical_Router_Port` 的可选弱引用 | 网关端口。 |
| `match` | 字符串 | 匹配条件。 |
| `priority` | 0 到 32,767 的整数 | 优先级。 |
| `options : stateless` | 可选字符串 | 是否无状态。 |
| `options : add_route` | 可选字符串 | 是否添加路由。 |

**公共列：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `external_ids` | 字符串对映射 | 外部 ID。 |

## 详细信息

-   `type`: 字符串，`dnat`, `dnat_and_snat`, 或 `snat`
    NAT 规则的类型。

    *   当类型为 `dnat` 时，外部可见的 IP 地址 `external_ip` 被 DNAT 到逻辑空间中的 IP 地址 `logical_ip`。
    *   当类型为 `snat` 时，源 IP 地址匹配 `logical_ip` 中的 IP 地址或在 `logical_ip` 提供的网络中的 IP 数据包被 SNAT 为 `external_ip` 中的 IP 地址。
    *   当类型为 `dnat_and_snat` 时，外部可见的 IP 地址 `external_ip` 被 DNAT 到逻辑空间中的 IP 地址 `logical_ip`。此外，源 IP 地址匹配 `logical_ip` 的 IP 数据包被 SNAT 为 `external_ip` 中的 IP 地址。

-   `external_ip`: 字符串
    一个 IPv4 地址。

-   `external_mac`: 可选字符串
    一个 MAC 地址。

    这仅用于分布式路由器上的网关端口。必须指定此项以便 NAT 规则在所有 Chassis 上以分布式方式处理。如果在分布式路由器上的 NAT 规则未指定此项，则此 NAT 规则将在网关 Chassis 上的网关端口实例上以集中方式处理。

    此 MAC 地址在网关端口连接的逻辑交换机上必须是唯一的。如果 `logical_port` 上使用的 MAC 地址是全局唯一的，则可以将该 MAC 地址指定为 `external_mac`。

-   `external_port_range`: 字符串
    L4 源端口范围。

    将从中选择一个端口号来替换要进行 NAT 的数据包的源端口的端口范围。这基本上是 PAT（端口地址转换）。

    列值的格式为 `port_lo-port_hi`。例如：`external_port_range : "1-30000"`

    有效端口范围是 1-65535。

-   `logical_ip`: 字符串
    一个 IPv4 网络（例如 192.168.1.0/24）或一个 IPv4 地址。

-   `logical_port`: 可选字符串
    `logical_ip` 所在的逻辑端口的名称。

    这仅用于分布式路由器。必须指定此项以便 NAT 规则在所有 Chassis 上以分布式方式处理。如果在分布式路由器上的 NAT 规则未指定此项，则此 NAT 规则将在网关 Chassis 上的网关端口实例上以集中方式处理。

-   `allowed_ext_ips`: 可选 `Address_Set`
    它代表 NAT 规则适用的外部 ip 的地址集。对于 SNAT 类型的 NAT 规则，这指的是目标地址。对于 DNAT 类型的 NAT 规则，这指的是源地址。

    此配置覆盖仅基于内部 IP 应用规则的默认 NAT 行为。没有此配置，NAT 发生时不考虑外部 IP（即 snat/dnat 类型规则的目标/源）。有了此配置，仅当外部 ip 在输入地址集中时才应用 NAT 规则。

-   `exempted_ext_ips`: 可选 `Address_Set`
    它代表 NAT 规则**不**适用的外部 ip 的地址集。对于 SNAT 类型的 NAT 规则，这指的是目标地址。对于 DNAT 类型的 NAT 规则，这指的是源地址。

    此配置覆盖仅基于内部 IP 应用规则的默认 NAT 行为。没有此配置，NAT 发生时不考虑外部 IP（即 snat/dnat 类型规则的目标/源）。有了此配置，如果外部 ip 在输入地址集中，则**不**应用 NAT 规则。

    如果逻辑路由器中存在具有重叠 IP 前缀（包括 /32）的 NAT 规则，则应避免在以下场景中使用 `exempted_ext_ips`。 a. SNAT 规则（假设为 RULE1）具有 `logical_ip` PREFIX/MASK（假设为 50.0.0.0/24）。 b. SNAT 规则（假设为 RULE2）具有 `logical_ip` PREFIX/MASK+1（假设为 50.0.0.0/25）。 c. 现在，如果 `exempted_ext_ips` 与 RULE2 关联，则同时匹配 50.0.0.0/24 和 50.0.0.0/25 的逻辑 ip 可能会应用 RULE2 而不是 RULE1。

    `allowed_ext_ips` 和 `exempted_ext_ips` 互斥。如果规则同时设置了两个地址集，则不考虑该 NAT 规则。

-   `gateway_port`: `Logical_Router_Port` 的可选弱引用
    需要应用 NAT 规则的 `Logical_Router_Port` 表中的分布式网关端口。

    当在 `Logical_Router` 上配置多个分布式网关端口时，可能不希望在每个分布式网关端口上应用 NAT 规则。考虑这样一种情况：逻辑路由器有 2 个分布式网关端口，一个具有网络 50.0.0.10/24，另一个具有网络 60.0.0.10/24。如果逻辑路由器有一个类型为 snat、`logical_ip` 为 10.1.1.0/24、`external_ip` 为 50.1.1.20/24 的 NAT 规则，则该规则需要选择性地应用于通过具有网络 50.0.0.10/24 的分布式网关端口进入/离开的匹配数据包。

    当逻辑路由器具有多个分布式网关端口且未为 NAT 规则设置此列时，如果存在这样的路由器端口，则规则将应用于与 NAT 规则的 `external_ip` 在同一网络中的分布式网关端口。如果逻辑路由器具有单个分布式网关端口且未为 NAT 规则设置此列，即使路由器端口与 NAT 规则的 `external_ip` 不在同一网络中，规则也将应用于分布式网关端口。

-   `match`: 字符串
    NAT 规则应匹配的数据包，除了基于 NAT 类型创建的匹配之外，使用与 OVN 南向数据库 `Logical_Flow` 表中的 match 列相同的表达式语言。这允许对 NAT 规则进行更细粒度的控制。

-   `priority`: 0 到 32,767 的整数
    NAT 规则的优先级。数值较高的规则优先于数值较低的规则。仅当定义了 match 时才考虑优先级。

-   `options : stateless`: 可选字符串
    指示 `dnat_and_snat` 规则是否应导致连接跟踪状态。

-   `options : add_route`: 可选字符串
    如果设置为 true，则邻居路由器将添加逻辑流，允许路由到 NAT 地址。它还将添加 ARP 解析逻辑流。通过设置此选项，意味着没有理由从邻居路由器创建到此 NAT 地址的 `Logical_Router_Static_Route`。这也意味着邻居路由器不需要 ARP 请求来学习此 NAT 地址的 IP-MAC 映射。此选项仅适用于 `dnat` 和 `dnat_and_snat` 类型的 NAT。有关为 IP 路由添加哪些流的更多信息，请参阅 ovn-northd 手册页关于 IP 路由的部分。

### 公共列

-   `external_ids`: 字符串对映射
    参见本文件开头的外部 ID。
