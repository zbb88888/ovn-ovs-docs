# 逻辑交换机数据路径 (Logical Switch Datapaths) - Egress

本节描述逻辑交换机的 Egress 流表。

## Egress Table 0: Lookup MAC address learning table

这与 Ingress 表 **Lookup MAC address learning table** 类似，区别在于 MAC 地址学习查找仅针对类型为 `remote`、端口安全（port security）被禁用且地址集合为 `unknown` 的端口进行。此阶段有助于连接多个可用区的中转交换机（transit switch）上的 MAC 学习。

## Egress Table 1: Learn MAC of 'unknown' ports

这与 Ingress 表 **Learn MAC of 'unknown' ports** 类似，区别在于 MAC 地址学习仅针对类型为 `remote`、端口安全（port security）被禁用且地址集合为 `unknown` 的端口进行。此阶段有助于连接多个可用区的中转交换机（transit switch）上的 MAC 学习。

## Egress Table 2: to-lport Pre-ACLs

这与 Ingress 表 **Pre-ACLs** 类似，除了它是针对 `to-lport` 流量的。

该表还包含一个优先级为 110 的流，匹配规则为 `eth.src == E`，用于所有逻辑交换机数据路径，将流量移动到下一个表。其中 `E` 是 `NB_Global` 表的 `options:svc_monitor_mac` 列中定义的 Service Monitor MAC。

该表还包含一个优先级为 110 的流，匹配规则为 `outport == I`，用于所有逻辑交换机数据路径，将流量移动到下一个表。其中 `I` 是逻辑路由器端口的对等端（peer）。添加此流是为了跳过将从逻辑交换机数据路径进入逻辑路由器数据路径进行路由的数据包的连接跟踪（connection tracking）。

该表还为每个在端口 `P` 中的 `network_function` 包含一个优先级为 110 的流，匹配规则为 `inport == P`。动作是跳过直到 **Network Function** 表之前的所有 egress 表，并将数据包直接推进到其后的表。这是为了处理数据包重定向发生在 egress **Network Function** 表中的情况。当同一个数据包从网络功能的另一个端口出来时，它们不应再次被相同的 egress 阶段处理，特别是它们应该跳过 conntrack 处理。

## Egress Table 3: Pre-LB

该表与 Ingress 表 **Pre-LB** 类似。它包含一个优先级为 0 的流，简单地将流量移动到下一个表。此外，它包含两个优先级为 110 的流，将多播、IPv6 邻居发现（Neighbor Discovery）和 MLD 流量移动到下一个表。如果数据路径存在任何负载均衡规则，则会添加一个优先级为 100 的流，匹配 `ip` 并执行动作 `reg0[2] = 1; next;`，作为 **Pre-stateful** 表的提示，将 IP 数据包发送到连接跟踪器进行数据包分片重组，并可能将目标 VIP DNAT 到已提交的负载均衡流量的选定后端之一。

该表还包含一个优先级为 110 的流，匹配规则为 `eth.src == E`，用于所有逻辑交换机数据路径，将流量移动到下一个表。其中 `E` 是 `NB_Global` 表的 `options:svc_monitor_mac` 列中定义的 Service Monitor MAC。

该表还包含一个优先级为 110 的流，匹配规则为 `outport == I`，用于所有逻辑交换机数据路径，将流量移动到下一个表，并且如果不存在 `stateful_acl`，则清除 `ct_state`。其中 `I` 是逻辑路由器端口的对等端（peer）。添加此流是为了跳过将从逻辑交换机数据路径进入逻辑路由器数据路径进行路由的数据包的连接跟踪。

当启用了 `enable-stateless-acl-with-lb` 时，会添加额外的优先级为 115 的流，以匹配设置了 `REGBIT_ACL_STATELESS` 的流量并跳过连接跟踪。

## Egress Table 4: Pre-stateful

这与 Ingress 表 **Pre-stateful** 类似。该表添加了以下 3 个逻辑流：

*   一个优先级为 120 的流，使用 `ct_lb_mark;` 动作将数据包发送到连接跟踪器，以便根据前一个表提供的提示（匹配 `reg0[2] == 1`），将已建立的流量从后端 IP unDNAT 回负载均衡器 VIP。如果数据包早先没有被 DNAT，那么 `ct_lb_mark` 的功能类似于 `ct_next`。

*   一个优先级为 100 的流，根据前一个表提供的提示（匹配 `reg0[0] == 1`），使用 `ct_next;` 动作将数据包发送到连接跟踪器。

*   一个优先级为 0 的流，匹配所有数据包并推进到下一个表。

## Egress Table 5: from-lport ACL hints

这与 Ingress 表 **ACL hints** 类似。

## Egress Table 6: to-lport ACL evaluation

这与 Ingress 表 **ACL eval** 类似，除了它是针对 `to-lport` ACL 的。作为提醒，这些流使用以下寄存器位来指示它们的判定结果：Allow 类型的 ACL 设置 `reg8[16]`，drop ACL 设置 `reg8[17]`，reject ACL 设置 `reg8[18]`。

也像 Ingress ACL 一样，Egress ACL 可以拥有 `network_function_group` id，在这种情况下，流将设置 `reg8[21] = 1; reg8[22] = 1; reg0[22..29] = id`。这些寄存器在 **Network Function** 表中使用。

也像 Ingress ACL 一样，Egress ACL 可以配置层级（tier）。如果配置了层级，则除了 ACL 的匹配之外，还会根据 ACL 配置的层级评估当前层级计数器。当前层级计数器存储在 `reg8[30..31]` 中。

与 Ingress 表类似，添加了一个优先级为 65532 的流，以允许 IPv6 邻居请求（Neighbor solicitation）、邻居发现（Neighbor discover）、路由器请求（Router solicitation）、路由器通告（Router advertisement）和 MLD 数据包，而不管定义了其他什么 ACL。

此外，还添加了以下流：

*   为每个定义了 DHCPv4 选项的逻辑端口添加一个优先级为 34000 的逻辑流，以允许 DHCPv4 回复数据包；如果定义了 DHCPv6 选项，则允许来自 **Ingress Table 26: DHCP responses** 的 DHCPv6 回复数据包。这是通过设置 allow 位来指示的。

*   为每个配置了 DNS 记录的逻辑交换机数据路径添加一个优先级为 34000 的逻辑流，匹配 `udp.dst = 53`，以允许来自 **Ingress Table 28: DNS responses** 的 DNS 回复数据包。这是通过设置 allow 位来指示的。

*   为每个逻辑交换机数据路径添加一个优先级为 34000 的逻辑流，匹配 `eth.src = E`，以允许由 `ovn-controller` 生成的 Service Monitor 请求数据包，动作为 `next`，其中 `E` 是 `NB_Global` 表的 `options:svc_monitor_mac` 列中定义的 Service Monitor MAC。这是通过设置 allow 位来指示的。

## Egress Table 7: to-lport ACL sampling

这与 Ingress 表 **ACL sampling** 类似。

## Egress Table 8: to-lport ACL action

这与 Ingress 表 **ACL action** 类似。

## Egress Table 9: Mirror

Overlay 远程镜像（remote mirror）表包含以下逻辑流：

*   对于每个附加了镜像的逻辑交换机端口，添加一个优先级为 100 的逻辑流。此流匹配所有发往该附加端口的传出数据包，克隆它们，并将克隆的数据包转发到镜像目标端口。

*   添加一个优先级为 0 的流，匹配所有数据包并应用动作 `next;`。

*   为附加到逻辑交换机端口的 Mirror 表中的每个镜像规则添加一个逻辑流，匹配所有符合规则的传出数据包，克隆数据包并将克隆的数据包发送到镜像目标端口。

## Egress Table 10: to-lport QoS

这与 Ingress 表 **QoS** 类似，除了它们应用于 `to-lport` QoS 规则。

## Egress Table 11: Stateful

这与 Ingress 表 **Stateful** 类似，除了没有为负载均衡新连接添加规则。当启用 `enable-stateless-acl-with-lb` 时，新的无状态连接会绕过连接跟踪。

*   为每个 `network_function` 端口 `P` 添加一个优先级为 120 的流，该流与优先级 100 的流相同，除了额外的匹配 `outport == P` 和额外的动作 `ct_label.tun_if_id = reg5[16..31]`。如果通过网络功能逻辑重定向的数据包从 host1 隧道传输到网络功能端口所在的 host2，host2 的物理表 0 会用接收数据包的 openflow 隧道接口 ID 填充 `reg5[16..31]`。这个优先级 120 的流将隧道 ID 提交到 `ct_label`。这样，当同一个数据包从网络功能的另一个端口出来时，它可以从对等端口的 CT 条目中检索此信息，并将数据包隧道传回 host1。这是使跨主机流量重定向在 VLAN 子网中工作所必需的。

## Egress Table 12: Network Function

此表与 Ingress 表 **Network Function** 类似，除了 `from-lport` 和 `to-lport` ACL 的角色互换，并且数据包重定向发生在所选网络功能的 `outport` 而不是其 `inport`。另一个区别是动作将数据包注入回 Ingress 管道。

*   与 Ingress **Network Function** 类似，为每个 `network_function` 端口添加一个优先级为 100 的流，匹配 `inport` 为网络功能端口并将数据包推进到下一个表。

*   对于每个 `network_function_group` id，一个优先级为 99 的流匹配 `reg8[21] == 1 && reg8[22] == 1 && reg0[22..29] == id` 并设置 `outport=P; reg8[23] = 1; next(pipeline=ingress, table=T)`，其中 `P` 是所选 `network_function` 的 `outport`，`T` 是 Ingress 表 **Destination Lookup**。这确保了匹配带有 `network_function` 的 `to-lport` ACL 的请求数据包的重定向。数据包被注入回 Ingress 管道，从那里它们被发送出去，由于 `reg8[23]` 而跳过任何进一步的查找。

*   对于每个 `network_function_group` id，一个优先级为 99 的规则匹配 `reg8[21] == 1 && reg8[22] == 0 && ct_label.nf_group_id == id` 并采取与上述相同的动作。这确保了如果响应和相关数据包匹配带有 `network_function` 的 `from-lport` ACL 时的重定向。

*   在上述每种情况下，当同一个数据包通过 `network_function` 的另一个端口原样出来时，它将匹配优先级 100 的流并转发到下一个表。

*   一个优先级为 100 的多播匹配流，与 Ingress **Network Function** 相同。

*   一个优先级为 1 的流，与 Ingress **Network Function** 相同。

*   一个优先级为 0 的流，与 Ingress **Network Function** 相同。

## Egress Table 13: Egress Port Security - check

这与表 **Ingress Port Security check** 中的端口安全逻辑类似，除了使用动作 `check_out_port_sec` 来检查端口安全规则。此表添加了以下逻辑流：

*   一个优先级为 100 的流，匹配多播流量并应用动作 `REGBIT_PORT_SEC_DROP = 0; next;` 以跳过出端口安全检查。

*   添加一个优先级为 0 的逻辑流，匹配所有数据包并应用动作 `REGBIT_PORT_SEC_DROP = check_out_port_sec(); next;`。动作 `check_out_port_sec` 在将数据包传递到 `outport` 之前，根据 `Logical_Switch_Port` 表的 `port_security` 列中定义的地址应用端口安全规则。

## Egress Table 14: Egress Port Security - Apply

这与 Ingress 表 **Ingress Port Security - Apply** 中的 Ingress 端口安全逻辑类似。如果上一阶段端口安全检查失败，即寄存器位 `REGBIT_PORT_SEC_DROP` 设置为 1，此表将丢弃数据包。

添加了以下流：

*   对于每个在 `Logical_Switch_Port` 的 `options:qdisc_queue_id` 列中配置了 egress qos 并在同一逻辑交换机上运行 localnet 端口的端口，添加一个优先级为 110 的流，匹配 localnet `outport` 和端口 `inport`，并应用动作 `set_queue(id); output;`。

*   对于每个在 `Logical_Switch_Port` 的 `options:qdisc_queue_id` 列中配置了 egress qos 的 localnet 端口，添加一个优先级为 100 的流，匹配 localnet `outport` 并应用动作 `set_queue(id); output;`。

    请记得在 `external_ids` 中将相应的物理接口标记为 `ovn-egress-iface` 设置为 true。

*   一个优先级为 50 的流，如果寄存器位 `REGBIT_PORT_SEC_DROP` 设置为 1，则丢弃数据包。

*   一个优先级为 0 的流，将数据包输出到 `outport`。
