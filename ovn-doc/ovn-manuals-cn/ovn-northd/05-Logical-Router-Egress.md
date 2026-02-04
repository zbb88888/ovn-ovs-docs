# 逻辑路由器数据路径 (Logical Router Datapaths) - Egress

本节描述逻辑路由器的 Egress 流表。

## Egress Table 0: Check DNAT local

此表检查数据包在被 SNAT 并环回（loop back）到 Ingress 管道后，是否需要在路由器 Ingress 表 `lr_in_dnat` 中进行 DNAT。此检查仅对配置了分布式网关端口和 NAT 条目的路由器执行。执行此检查是为了让 SNAT 和 DNAT 在不同的区域（zone）中完成，而不是在同一个区域中。

*   一个优先级为 0 的逻辑流，匹配 1，动作为 `REGBIT_DST_NAT_IP_LOCAL = 0; next;`。

## Egress Table 1: UNDNAT

这是针对已建立连接的反向流量。即，DNAT 已经在 Ingress 管道中完成，现在数据包作为回复进入 Egress 管道。此流量在这里被 unDNAT。

*   一个优先级为 0 的逻辑流，匹配 1，动作为 `next;`。

### Egress Table 1: UNDNAT on Gateway Routers

*   对于 IPv6 邻居发现或路由器请求/通告流量，一个优先级为 100 的流，动作为 `next;`。

*   对于所有 IP 数据包，一个优先级为 50 的流，动作为 `flags.loopback = 1; ct_dnat;`。

### Egress Table 1: UNDNAT on Distributed Routers

*   对于 `OVN_Northbound` 数据库中包含 IPv4 地址 VIP 的具有网关端口的路由器的所有配置的负载均衡规则，对于为 VIP 定义的每个后端 IPv4 地址 `B`，在网关 chassis 上编程一个优先级为 120 的流，匹配 `ip && ip4.src == B && outport == GW`，其中 `GW` 是逻辑路由器网关端口，动作为 `ct_dnat;`。如果后端 IPv4 地址 `B` 还配置了协议 `P` 的 L4 端口 `PORT`，则匹配还包括 `P.src == PORT`。这些流不会为具有 IPv6 VIP 的负载均衡器添加。

    如果路由器配置为强制 SNAT 任何负载均衡的数据包，上述动作将被替换为 `flags.force_snat_for_lb = 1; ct_dnat;`。

*   对于 `OVN_Northbound` 数据库中要求将数据包的目标 IP 地址从 IP 地址 `A` 更改为 `B` 的每个配置，一个优先级为 100 的流，匹配 `ip && ip4.src == B && outport == GW`，其中 `GW` 是逻辑路由器网关端口，动作为 `ct_dnat;`。如果 NAT 规则类型为 `dnat_and_snat` 且选项中有 `stateless=true`，则动作为 `next;`。

    如果 NAT 规则无法以分布式方式处理，则上述优先级 100 的流仅在网关 chassis 上编程，动作为 `ct_dnat`。

    如果 NAT 规则可以以分布式方式处理，则有一个额外的动作 `eth.src = EA;`，其中 `EA` 是与 NAT 规则中的 IP 地址 `A` 关联的以太网地址。这允许上游 MAC 学习指向正确的 chassis。

## Egress Table 2: Post UNDNAT

*   添加一个优先级为 70 的逻辑流，为配置为在分布式路由器上进行 SNAT 的流量初始化 CT 状态。这允许下一个表 `lr_out_snat` 有效地匹配各种 CT 状态。

*   添加一个优先级为 50 的逻辑流，提交来自前一个表 `lr_out_undnat` 的任何未跟踪（untracked）流（针对网关路由器）。此流匹配 `ct.new && ip`，动作为 `ct_commit { } ; next;`。

*   如果 `options:ct-commit-all` 设置为 `true`，则配置以下流：匹配 `ip && (!ct.trk || !ct.rpl) && flags.unsnat_not_tracked == 1`，动作为 `ct_next(snat);`；以及匹配 `ip && flags.unsnat_new == 1`，动作为 `next;`。当配置了 DGP `outport == DGP && is_chassis_resident(CHASSIS)` 时会有额外的匹配。

*   一个优先级为 0 的逻辑流，匹配 1，动作为 `next;`。

## Egress Table 3: SNAT

配置为 SNAT 的数据包会根据 `OVN_Northbound` 数据库中的配置更改其源 IP 地址。

*   一个优先级为 120 的流，将 IPv6 邻居请求数据包推进到下一个表以跳过 SNAT。在 `ovn-controller` 注入 IPv6 邻居请求数据包（用于 `nd_ns` 动作）的情况下，我们不希望数据包通过 conntrack。

### Egress Table 3: SNAT on Gateway Routers

*   如果 `OVN_Northbound` 数据库中的网关路由器已配置为强制 SNAT 一个数据包（先前已被 DNAT）到 `B`，则一个优先级为 100 的流匹配 `flags.force_snat_for_dnat == 1 && ip`，动作为 `ct_snat(B);`。

*   如果配置为 `skip snat` 的负载均衡器已应用于网关路由器管道，则一个优先级为 120 的流匹配 `flags.skip_snat_for_lb == 1 && ip`，动作为 `next;`。

*   如果 `OVN_Northbound` 数据库中的网关路由器已配置为使用路由器 IP 强制 SNAT 一个数据包（先前已被负载均衡）（即 `options:lb_force_snat_ip=router_ip`），那么对于连接到网关路由器的每个逻辑路由器端口 `P`，以及为此端口配置的每个网络，一个优先级为 110 的流匹配 `flags.force_snat_for_lb == 1 && ip4 && flags.network_id == N && outport == P`，其中 `N` 是网络索引，动作为 `ct_snat(R);`，其中 `R` 是路由器端口上配置的 IP。对于 IPv6，创建一个类似的流，使用 `ip6` 代替 `ip4`。对于网络 17 及以上，网络索引 `N` 将为 0。

*   一个优先级为 105 的流匹配旧行为（如果 northd 在 controller 之前升级且 `flags.network_id` 未被识别）。仅当至少配置了一个网络（不包括 IPv6 的 LLA）时才会添加。匹配：`flags.force_snat_for_lb == 1 && ip4 && outport == P`，动作：`ct_snat(R)`。`R` 是按字典顺序排列的第一个配置的 IP 地址。对于 IPv6 也有类似的流。

*   如果 `OVN_Northbound` 数据库中的网关路由器已配置为强制 SNAT 一个数据包（先前已被负载均衡）到 `B`，则一个优先级为 100 的流匹配 `flags.force_snat_for_lb == 1 && ip`，动作为 `ct_snat(B);`。

*   对于 `OVN_Northbound` 数据库中要求将数据包的源 IP 地址从 IP 地址 `A` 更改，或将属于网络 `A` 的数据包的源 IP 地址更改为 `B` 的每个配置，一个流匹配 `ip && ip4.src == A && (!ct.trk || !ct.rpl)`，动作为 `ct_snat(B);`。流的优先级基于 `A` 的掩码计算，匹配较大掩码的具有较高优先级。如果 NAT 规则类型为 `dnat_and_snat` 且选项中有 `stateless=true`，则动作为 `ip4/6.src= (B)`。

*   如果 NAT 规则配置了 `allowed_ext_ips`，则有一个额外的匹配 `ip4.dst == allowed_ext_ips`。类似地，对于 IPv6，匹配将是 `ip6.dst == allowed_ext_ips`。

*   如果 NAT 规则设置了 `exempted_ext_ips`，则在对应 NAT 规则的优先级 + 1 处配置一个额外的流。该流匹配如果目标 ip 是 `exempted_ext_ip`，动作为 `next;`。此流用于绕过发往 `exempted_ext_ips` 的数据包的 `ct_snat` 动作。

*   如果 `options:ct-commit-all` 设置为 `true`，则配置两个流，动作为 `ct_commit_to_zone(snat);`。

*   一个优先级为 0 的逻辑流，匹配 1，动作为 `next;`。

### Egress Table 3: SNAT on Distributed Routers

*   对于 `OVN_Northbound` 数据库中要求将数据包的源 IP 地址从 IP 地址 `A` 更改，或将属于网络 `A` 的数据包的源 IP 地址更改为 `B` 的每个配置，添加两个流。这些流的优先级 `P` 基于 `A` 的掩码计算。

    如果 NAT 规则无法以分布式方式处理，则以下流仅在网关 chassis 上编程，流优先级增加 128 以便首先运行。

    *   第一个流添加时具有计算出的优先级 `P`，匹配 `ip && ip4.src == A && outport == GW`，其中 `GW` 是逻辑路由器网关端口，动作为 `ct_snat(B);` 以在公共区域（common zone）中进行 SNAT。如果 NAT 规则类型为 `dnat_and_snat` 且选项中有 `stateless=true`，则动作为 `ip4/6.src=(B)`。

    如果 NAT 规则可以以分布式方式处理，则有一个额外的动作（对于两个流）`eth.src = EA;`，其中 `EA` 是与 NAT 规则中的 IP 地址 `A` 关联的以太网地址。这允许上游 MAC 学习指向正确的 chassis。

    如果 NAT 规则配置了 `allowed_ext_ips`，则有一个额外的匹配 `ip4.dst == allowed_ext_ips`。类似地，对于 IPv6，匹配将是 `ip6.dst == allowed_ext_ips`。

    如果 NAT 规则设置了 `exempted_ext_ips`，则在对应 NAT 规则的优先级 `P + 2` 处配置一个额外的流。该流匹配如果目标 ip 是 `exempted_ext_ip`，动作为 `next;`。

*   为反向流量（即进入配置了 SNAT 的网络）添加一个额外的流。上述流匹配 `ip4.src == A && outport == GW`，此流匹配 `ip4.dst == A && inport == GW`。为此流量启动 CT 状态，以便下一个表 `lr_out_post_snat` 可以识别流量流是从内部网络还是外部网络发起的。

*   如果 `options:ct-commit-all` 设置为 `true`，则配置两个流，动作为 `ct_commit_to_zone(snat);`。

*   一个优先级为 0 的逻辑流，匹配 1，动作为 `next;`。

## Egress Table 4: Post SNAT

到达此表的数据包按照以下流进行处理：

*   直接进入分布式路由器上配置了 SNAT 的网络且从外部网络发起（即匹配 `ct.new`）的流量，被提交到 SNAT CT 区域。这确保了从 SNAT 网络返回的回复不会对其源地址进行转换。

*   一个优先级为 0 的逻辑流，匹配所有尚未处理的数据包（match 1），动作为 `next;`。

## Egress Table 5: Egress Loopback

对于其中一个逻辑路由器端口指定了网关 chassis 的分布式逻辑路由器。

虽然此时已经进行了 UNDNAT 和 SNAT 处理，但此流量需要被强制通过此分布式网关端口实例上的 egress loopback，以便应用 UNSNAT 和 DNAT 处理，以及在所有 NAT 处理之后的 IP 路由和 ARP 解析，以便数据包可以转发到目的地。

此表包含以下流：

*   对于分布式路由器上 `OVN_Northbound` 数据库中的每个 NAT 规则，一个优先级为 100 的逻辑流，匹配 `ip4.dst == E && outport == GW && is_chassis_resident(P)`，其中 `E` 是 NAT 规则中指定的外部 IP 地址，`GW` 是对应于 NAT 规则（指定或推断）的分布式网关端口。对于 `dnat_and_snat` NAT 规则，`P` 是 NAT 规则中指定的逻辑端口。如果 NAT 表的 `logical_port` 列未设置，则 `P` 是 `GW` 的 chassisredirect 端口。动作如下：

    ```
    clone {
        ct_clear;
        inport = outport;
        outport = "";
        flags = 0;
        flags.loopback = 1;
        reg0 = 0;
        reg1 = 0;
        ...
        reg9 = 0;
        REGBIT_EGRESS_LOOPBACK = 1;
        next(pipeline=ingress, table=0);
    };
    ```

    设置 `flags.loopback` 是因为 `in_port` 未更改，数据包可能会在 NAT 处理后返回到该端口。设置 `REGBIT_EGRESS_LOOPBACK` 是为了指示已发生 egress loopback，以便跳过针对路由器地址的源 IP 地址检查。

*   一个优先级为 0 的逻辑流，匹配 1，动作为 `next;`。

## Egress Table 6: Delivery

到达此表的数据包已准备好进行投递。它包含：

*   优先级为 110 的逻辑流，匹配每个已启用逻辑路由器端口上的 IP 多播数据包，将数据包的以太网源地址修改为端口的以太网地址，然后执行动作 `output;`。

*   优先级为 100 的逻辑流，匹配每个已启用逻辑路由器端口上的数据包，动作为 `output;`。

*   一个优先级为 0 的逻辑流，匹配所有尚未处理的数据包（match 1）并丢弃它们（动作 `drop;`）。

## DROP SAMPLING

如前一节所述，在很多地方 `ovn-northd` 可能会决定通过显式创建一个带有 `drop;` 动作的 `Logical_Flow` 来丢弃数据包。

当在 `OVN_Northbound` 数据库中配置了调试 drop-sampling 时，`ovn-northd` 将把所有的 `drop;` 动作替换为 `sample(priority=65535, collector_set=id, obs_domain=obs_id, obs_point=@cookie)` 动作。
