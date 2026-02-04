# 逻辑路由器数据路径 (Logical Router Datapaths) - Ingress

逻辑路由器数据路径仅存在于 `OVN_Northbound` 数据库中 `enabled` 未设置为 `false` 的 `Logical_Router` 行。

## Ingress Table 0: L2 Admission Control

该表根据以太网头部丢弃路由器根本不应该看到的数据包。它包含以下流：

*   优先级为 100 的流，丢弃带有 VLAN 标签或多播以太网源地址的数据包。

*   对于每个以太网地址为 `E` 的已启用路由器端口 `P`，一个优先级为 50 的流，匹配 `inport == P && (eth.mcast || eth.dst == E)`，存储路由器端口以太网地址并推进到下一个表，动作为 `xreg0[0..47]=E; next;`。

    对于分布式逻辑路由器上的网关端口（其中一个逻辑路由器端口指定了网关 chassis），上述匹配 `eth.dst == E` 的流仅在网关 chassis 上的网关端口实例上编程。如果 LRP 的逻辑交换机连接了 `vtep` 类型的 LSP，则不将 `is_chassis_resident()` 部分添加到逻辑流中，以允许源自逻辑交换机的流量到达 LR 服务（LB，NAT）。

    对于分布式逻辑路由器上的每个网关端口 `GW`，一个优先级为 120 的流，匹配 'recirculated' icmp{4,6} 错误 'packet too big' 且 `eth.dst == D && !is_chassis_resident(cr-GW)`（其中 `D` 是网关端口 mac 地址，`cr-GW` 是 `GW` 的 chassis resident 端口），交换 `inport` 和 `outport` 并将 `GW` 存储为 `inport`。

    该表添加了一个优先级为 105 的流，匹配 'recirculated' icmp{4,6} 错误 'packet too big' 并丢弃数据包。

    对于分布式逻辑路由器或端口配置了 `options:gateway_mtu` 的网关路由器，上述流的动作被修改为添加 `check_pkt_larger`，以便在大小大于 MTU 时设置 `REGBIT_PKT_LARGER` 标记数据包。如果端口还配置了 `options:gateway_mtu_bypass`，则会添加另一个优先级为 55 的流来绕过 `check_pkt_larger` 流。这对于通常不需要分段且 `check_pkt_larger`（可能无法卸载）并非真正需要的流量很有用。一个这样的例子是 TCP 流量。

*   对于分布式路由器上指定了外部以太网地址 `E` 的每个 `dnat_and_snat` NAT 规则，一个优先级为 50 的流，匹配 `inport == GW && eth.dst == E`，其中 `GW` 是对应于 NAT 规则（指定或推断）的逻辑路由器分布式网关端口，动作为 `xreg0[0..47]=E; next;`。

    此流仅在指定了 NAT 规则中 `logical_port` 所在的 chassis 上的网关端口实例上编程。

*   一个优先级为 0 的逻辑流，匹配所有尚未处理的数据包（match 1）并丢弃它们（动作 `drop;`）。

其他数据包被隐式丢弃。

## Ingress Table 1: Neighbor lookup

对于 ARP 和 IPv6 邻居发现（Neighbor Discovery）数据包，此表查找 `MAC_Binding` 记录以确定 OVN 是否需要学习 mac 绑定。添加了以下流：

*   对于拥有 IP 地址 `A`（属于前缀长度为 `L` 的子网 `S`）的每个路由器端口 `P`，如果此路由器的选项 `always_learn_from_arp_request` 为 `true`，则添加一个优先级为 100 的流，匹配 `inport == P && arp.spa == S/L && arp.op == 1` (ARP request)，动作如下：

    ```
    reg9[2] = lookup_arp(inport, arp.spa, arp.sha);
    next;
    ```

    如果选项 `always_learn_from_arp_request` 为 `false`，则添加以下两个流。

    添加一个优先级为 110 的流，匹配 `inport == P && arp.spa == S/L && arp.tpa == A && arp.op == 1` (ARP request)，动作如下：

    ```
    reg9[2] = lookup_arp(inport, arp.spa, arp.sha);
    reg9[3] = 1;
    next;
    ```

    添加一个优先级为 100 的流，匹配 `inport == P && arp.spa == S/L && arp.op == 1` (ARP request)，动作如下：

    ```
    reg9[2] = lookup_arp(inport, arp.spa, arp.sha);
    reg9[3] = lookup_arp_ip(inport, arp.spa);
    next;
    ```

    如果逻辑路由器端口 `P` 是分布式网关路由器端口，则为所有这些流添加额外的匹配 `is_chassis_resident(cr-P)`。

*   一个优先级为 100 的流，匹配 ARP reply 数据包，如果选项 `always_learn_from_arp_request` 为 `true`，则应用以下动作：

    ```
    reg9[2] = lookup_arp(inport, arp.spa, arp.sha);
    next;
    ```

    如果选项 `always_learn_from_arp_request` 为 `false`，上述动作将是：

    ```
    reg9[2] = lookup_arp(inport, arp.spa, arp.sha);
    reg9[3] = 1;
    next;
    ```

*   一个优先级为 100 的流，匹配 IPv6 邻居发现通告（advertisement）数据包，如果选项 `always_learn_from_arp_request` 为 `true`，则应用以下动作：

    ```
    reg9[2] = lookup_nd(inport, nd.target, nd.tll);
    next;
    ```

    如果选项 `always_learn_from_arp_request` 为 `false`，上述动作将是：

    ```
    reg9[2] = lookup_nd(inport, nd.target, nd.tll);
    reg9[3] = 1;
    next;
    ```

*   一个优先级为 100 的流，匹配 IPv6 邻居发现请求（solicitation）数据包，如果选项 `always_learn_from_arp_request` 为 `true`，则应用以下动作：

    ```
    reg9[2] = lookup_nd(inport, ip6.src, nd.sll);
    next;
    ```

    如果选项 `always_learn_from_arp_request` 为 `false`，上述动作将是：

    ```
    reg9[2] = lookup_nd(inport, ip6.src, nd.sll);
    reg9[3] = lookup_nd_ip(inport, ip6.src);
    next;
    ```

*   一个优先级为 0 的回退流，匹配所有数据包并应用动作 `reg9[2] = 1; next;` 将数据包推进到下一个表。

## Ingress Table 2: Neighbor learning

此表添加流以从 ARP 和 IPv6 邻居请求/通告数据包中学习 mac 绑定（如果根据上一阶段的查找结果需要的话）。

`reg9[2]` 将为 1，如果在上一个表中的 `lookup_arp`/`lookup_nd` 成功或被跳过，这意味着不需要从数据包中学习 mac 绑定。

`reg9[3]` 将为 1，如果在上一个表中的 `lookup_arp_ip`/`lookup_nd_ip` 成功或被跳过，这意味着可以从数据包中学习 mac 绑定（如果 `reg9[2]` 为 0）。

*   一个优先级为 100 的流，匹配 `reg9[2] == 1 || reg9[3] == 0`，并将数据包推进到下一个表，因为不需要学习邻居。

*   一个优先级为 95 的流，匹配 `nd_ns && (ip6.src == 0 || nd.sll == 0)`，并应用动作 `next;`。

*   一个优先级为 90 的流，匹配 `arp`，并应用动作 `put_arp(inport, arp.spa, arp.sha); next;`。

*   一个优先级为 95 的流，匹配 `nd_na && nd.tll == 0`，并应用动作 `put_nd(inport, nd.target, eth.src); next;`。

*   一个优先级为 90 的流，匹配 `nd_na`，并应用动作 `put_nd(inport, nd.target, nd.tll); next;`。

*   一个优先级为 90 的流，匹配 `nd_ns`，并应用动作 `put_nd(inport, ip6.src, nd.sll); next;`。

*   一个优先级为 0 的逻辑流，匹配所有尚未处理的数据包（match 1）并丢弃它们（动作 `drop;`）。

## Ingress Table 3: IP Input

此表是逻辑路由器数据路径功能的核心。它包含以下流以实现非常基本的 IP 主机功能。

*   对于分布式逻辑路由器或网关端口配置了 `options:gateway_mtu` 为有效整数值 `M` 的网关路由器上的每个 `dnat_and_snat` NAT 规则，一个优先级为 160 的流，匹配 `inport == LRP && REGBIT_PKT_LARGER && REGBIT_EGRESS_LOOPBACK == 0`（其中 `LRP` 是逻辑路由器端口），分别对 ipv4 和 ipv6 应用以下动作：

    ```
    icmp4_error {
        icmp4.type = 3; /* Destination Unreachable. */
        icmp4.code = 4;  /* Frag Needed and DF was Set. */
        icmp4.frag_mtu = M;
        eth.dst = eth.src;
        eth.src = E;
        ip4.dst = ip4.src;
        ip4.src = I;
        ip.ttl = 255;
        REGBIT_EGRESS_LOOPBACK = 1;
        REGBIT_PKT_LARGER 0;
        outport = LRP;
        flags.loopback = 1;
        output;
    };
    /* ipv6 类似 */
    ```

    其中 `E` 和 `I` 分别是 NAT 规则的外部 mac 和 IP。

*   对于分布式逻辑路由器或网关端口配置了 `options:gateway_mtu` 为有效整数值的网关路由器，一个优先级为 150 的流，匹配 `inport == LRP && REGBIT_PKT_LARGER && REGBIT_EGRESS_LOOPBACK == 0`，分别对 ipv4 和 ipv6 应用类似上述的 icmp 错误动作。

*   对于类型为 `snat` 的分布式逻辑路由器（具有分布式网关路由器端口）的每个 NAT 条目，一个优先级为 120 的流，匹配 `inport == P && ip4.src == A`，将数据包推进到下一个管道，其中 `P` 是对应于 NAT 条目（指定或推断）的分布式逻辑路由器端口，`A` 是 NAT 条目中设置的 `external_ip`。这需要用于处理东西向 NAT 流量的路由。

*   对于每个 BFD 端口，添加两个优先级为 110 的流以管理 BFD 流量。

*   对于每个配置了 DHCP 中继的逻辑路由器端口，添加优先级为 110 的流以管理 DHCP 中继流量（检查并执行 `dhcp_relay_req_chk`）。

*   L3 准入控制：优先级为 120 的流允许 IGMP 和 MLD 数据包，如果路由器有配置了 `options:mcast_flood='true'` 的逻辑端口。

*   L3 准入控制：一个优先级为 100 的流丢弃匹配以下任何一项的数据包：
    *   多播源、广播源、Localhost 源或目的、零网络源或目的。
    *   路由器拥有的任何 IP 地址作为源（除非是 egress loopback）。
    *   路由器已知的任何 IP 网络的广播地址作为源。

*   一个优先级为 100 的流解析来自 IPv6 前缀代理路由器的 DHCPv6 回复。

*   对于应用此逻辑路由器的每个配置了 VIP 模板的负载均衡器，一个优先级为 100 的流，匹配 `ip4.dst` 或 `ip6.dst` 为配置的负载均衡器 VIP，动作为 `next;`。

*   ICMP echo reply。这些流回复针对路由器 IP 地址的 ICMP echo 请求。

*   Reply to ARP requests。这些流回复针对路由器自身 IP 地址的 ARP 请求。
    对于分布式逻辑路由器上的网关端口，这些流仅在网关 chassis 上的网关端口实例上编程。

*   Reply to IPv6 Neighbor Solicitations。这些流回复针对路由器自身 IPv6 地址的邻居请求。

*   这些流回复针对路由器中配置用于 NAT（DNAT 和 SNAT）或负载均衡的虚拟 IP 地址的 ARP 请求或 IPv6 邻居请求。

*   优先级为 85 的流丢弃 ARP 和 IPv6 邻居发现数据包。

*   优先级为 84 的流显式允许 IPv6 多播流量。

*   优先级为 83 的流显式丢弃发往保留多播组的 IPv6 多播流量。

*   优先级为 82 的流允许 IP 多播流量（如果 `options:mcast_relay='true'`），否则丢弃。

*   UDP port unreachable。优先级为 80 的流生成 ICMP 端口不可达消息。

*   TCP reset。优先级为 80 的流生成 TCP 重置消息。

*   Protocol or address unreachable。优先级为 70 的流生成 ICMP 协议或地址不可达消息。

*   Drop other IP traffic to this router。优先级为 60 的流丢弃发往此路由器 IP 地址的其他流量。

上述流处理所有可能直接发往路由器本身的流量。以下流（优先级较低）处理剩余流量，可能用于转发：

*   Drop Ethernet local broadcast。优先级为 50。

*   Avoid ICMP time exceeded for multicast。优先级为 32。

*   ICMP time exceeded。优先级为 31。发送 ICMP 超时回复。

*   TTL discard。优先级为 30。

*   Next table。优先级为 0 的流匹配所有尚未处理的数据包并使用动作 `next;`。

## Ingress Table 4: DHCP Relay Request

此阶段处理在 IP Input 阶段应用了 `dhcp_relay_req_chk` 动作的 DHCP 请求数据包。

## Ingress Table 5: UNSNAT

这是针对已建立连接的反向流量。即，SNAT 已经在 egress 管道中完成，现在数据包作为回复进入 ingress 管道。它在这里被 unSNAT。

*   **Gateway and Distributed Routers**: 如果路由器配置了负载均衡器，且 VIP 也作为 `external_ip` 存在于 NAT 表中，则添加优先级为 120 的流以推进数据包。

*   **Gateway Routers**:
    *   如果配置为强制 SNAT 任何先前 DNAT 的数据包，优先级 110 的流执行 `ct_snat;`。
    *   如果配置了 `lb_force_snat_ip=router_ip`，优先级 110 的流执行 `ct_snat;`。
    *   如果配置为强制 SNAT 任何先前负载均衡的数据包，优先级 100 的流执行 `ct_snat;`。
    *   对于每个 NAT 配置，优先级 90 的流执行 `ct_snat;`（如果是 `dnat_and_snat` 且 `stateless=true`，则为 `next;`）。

*   **Distributed Routers**:
    *   对于每个 NAT 配置，添加两个优先级为 100 的流。如果 NAT 规则不能以分布式方式处理，则仅在网关 chassis 上编程。

## Ingress Table 6: POST USNAT

这是为了检查数据包是否已经在 SNAT zone 中被跟踪。包含一个优先级为 0 的流，简单地将流量移动到下一个表。如果设置了 `options:ct-commit-all`，则会配置额外的流。

## Ingress Table 7: DEFRAG

这是为了将数据包发送到连接跟踪器进行跟踪和分片重组。

*   对于网关路由器的所有负载均衡规则，优先级 100 的流对 VIP 执行 `ct_dnat;`。
*   如果配置了具有对称回复的 ECMP 路由，优先级 100 的流执行 `chk_ecmp_nh_mac(); ct_next` 或 `chk_ecmp_nh(); ct_next`。
*   如果配置了负载均衡规则，优先级 50 的流匹配 ICMP 并执行 `ct_dnat;`。

## Ingress Table 8: Connection tracking field extraction

此表提取新连接的连接跟踪字段并将它们存储在寄存器中，以供后续负载均衡阶段使用。
对于所有新连接 (`ct.new`)，优先级 100 的流提取 `ct_proto()` 到 `reg1[16..23]` 和 `ct_tp_dst()` 到 `reg1[0..15]`。

## Ingress Table 9: Load balancing affinity check

负载均衡亲和性检查表。对于配置了亲和性超时的负载均衡规则，优先级 100 的流检查亲和性并设置 `reg9[6] = chk_lb_aff()`。

## Ingress Table 10: DNAT

数据包进入管道时，目标 IP 地址可能需要从虚拟 IP 地址 DNAT 到真实 IP 地址。

*   **Load balancing DNAT rules**: 为网关路由器或具有网关端口的路由器添加。优先级 150（基于亲和性检查结果）和 120（新连接）的流执行 `ct_lb_mark(args)`。
*   **DNAT on Gateway Routers**: 优先级 100 的流执行 `ct_dnat(B)`。
*   **DNAT on Distributed Routers**: 优先级 100 的流执行 `ct_dnat(B)`。

## Ingress Table 11: Load balancing affinity learn

负载均衡亲和性学习表。对于配置了亲和性超时的规则，优先级 100 的流执行 `commit_lb_aff(...)`。

## Ingress Table 12: ECMP symmetric reply processing

如果配置了具有对称回复的 ECMP 路由，优先级 100 的流使用 `ct_commit` 提交连接并将 `eth.src` 和 ECMP 回复端口绑定隧道密钥存储在 `ct_label` 中。

## Ingress Table 13: IPv6 ND RA option processing

处理 IPv6 ND Router Solicitation 数据包，应用 `put_nd_ra_opts` 动作。

## Ingress Table 14: IPv6 ND RA responder

实现 IPv6 ND RA 响应器。如果上一表的 `put_nd_ra_opts` 成功（`reg0[5] == 1`），则构建并发送 RA 回复。

## Ingress Table 15: IP Routing Pre

如果数据包来自设置了 `options:route_table` 的逻辑路由器端口，则设置 `reg7` 为路由表标识符。

## Ingress Table 16: IP Routing

实现 IP 路由，将 `reg0`（或 `xxreg0`）设置为下一跳 IP 地址，并推进到下一个表进行 ARP 解析。

*   处理 ECMP 路由：选择 ECMP 组 ID 和成员 ID，存储在 `reg8` 中。
*   处理多播路由：匹配 IGMP/MLD 数据包和多播流量，设置 `outport` 为多播组。
*   IPv4/IPv6 路由表：匹配 `ip.dst`，设置 `reg0`（下一跳），`reg1`（源 IP），`eth.src`，`outport`，递减 TTL。

## Ingress Table 17: IP_ROUTING_ECMP

实现 ECMP 路由的第二部分。匹配 `reg8` 中的组 ID 和成员 ID，设置 `reg0`（下一跳）等。

## Ingress Table 18: Router policies

添加逻辑路由器策略流。
如果策略动作为 `reroute` 且有多个下一跳（ECMP），则选择下一跳并存储在 `reg8` 中。
如果只有一个下一跳，则直接设置 `reg0`，`eth.src`，`outport`。

## Ingress Table 19: ECMP handling for router policies

处理具有多个下一跳的路由器策略的 ECMP。根据 `reg8` 中的选择设置 `reg0` 等。

## Ingress Table 20: DHCP Relay Response Check

处理来自 DHCP 服务器的 DHCP 响应数据包。应用 `dhcp_relay_resp_chk`。

## Ingress Table 21: DHCP Relay Response

处理上一阶段检查过的 DHCP 响应数据包。如果检查成功，更新 IP/UDP 头部并发送到 LRP 端口。

## Ingress Table 22: ARP/ND Resolution

将 `reg0` (IP) 解析为 `outport` 和 `eth.dst`。

*   处理 ECMP 回复流量（使用 `ct_label` 中的信息）。
*   静态 MAC 绑定（来自 `Logical_Switch_Port` 或 `Logical_Router_Port`）。
*   来自 NAT 条目的静态 MAC 绑定。
*   动态 MAC 绑定（使用 `get_arp` / `get_nd`）。

## Ingress Table 23: Check packet length

检查 MTU。如果数据包大于 `options:gateway_mtu`，设置 `REGBIT_PKT_LARGER`。

## Ingress Table 24: Handle larger packets

如果设置了 `REGBIT_PKT_LARGER`，生成 ICMP Fragmentation Needed 错误。

## Ingress Table 25: Gateway Redirect

对于分布式逻辑路由器，将某些数据包重定向到网关 chassis 上的分布式网关端口实例。包括负载均衡流量、NAT 流量等。

## Ingress Table 26: Network ID

为 IP 数据包设置 `flags.network_id`（用于 SNAT）。

## Ingress Table 27: ARP Request

如果以太网目标地址未解析（`eth.dst == 00:00:00:00:00:00`），则发送 ARP 或 IPv6 邻居请求。
已知 MAC 地址的数据包被输出。
