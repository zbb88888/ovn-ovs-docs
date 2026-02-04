# OVN 逻辑路由器流表 (Logical Router Flows)

`ovn-northd` 为每个逻辑路由器生成 Ingress (入站) 和 Egress (出站) 逻辑流管道。

## Ingress Pipeline (入站管道)

### Ingress Table 0: L2 Admission
**入站表 0: L2 准入控制**

此表是逻辑路由器的入口点。它丢弃那些不是发往路由器端口 MAC 地址的数据包。

*   **匹配**: `eth.dst == ROUTER_MAC`。
*   **动作**: `next;`。
*   **广播/组播**: 丢弃（除了特定的 ND/ARP 请求）。

### Ingress Table 1: Neighbor Lookup
**入站表 1: 邻居查找**

处理 ARP 和 IPv6 ND 邻居发现。

*   **ARP 请求**: 如果是发往路由器 IP 的 ARP 请求，进行回复。
*   **ARP 响应**: 如果是路由器发送的 ARP 请求的响应，更新邻居表。

### Ingress Table 2: IP Input
**入站表 2: IP 输入处理**

这是路由器处理的核心预处理阶段。

*   **TTL 检查**: 检查 TTL 是否大于 1。如果 TTL <= 1，丢弃并可能发送 ICMP Time Exceeded。
*   **服务端口**: 允许发往路由器接口 IP 的特定服务流量（如 BFD, DNS 等）。
*   **丢弃**: 丢弃发往路由器 IP 但未被处理的其他流量。
*   **转发**: 对于需要路由的流量（目的 IP 不是路由器接口 IP），动作 `next;`。

### Ingress Table 3: Defragmentation
**入站表 3: 分片重组**

类似于逻辑交换机，如果需要，将分片的 IP 包发送到连接跟踪器进行重组。

### Ingress Table 4: UNSNAT
**入站表 4: UNSNAT (反向 SNAT)**

处理从外部网络进入的数据包，这些数据包是内部网络发起的连接的响应。

*   **CT SNAT**: 如果数据包匹配已知的 SNAT 连接状态，执行 `ct_snat;` 将目的 IP（即 SNAT 后的公共 IP）还原为内部私有 IP。
    *   匹配 `ip && ct.trk && (ct.est || ct.rel) && !ct.new`。

### Ingress Table 5: DNAT
**入站表 5: DNAT (目标地址转换)**

处理发往 VIP（虚拟 IP）或 NAT 规则中外部 IP 的数据包。

*   **DNAT 规则**: 根据 `OVN_Northbound` 中的 NAT 表配置。
    *   匹配 `ip.dst == EXTERNAL_IP`。
    *   动作 `ct_dnat(INTERNAL_IP);`。
*   **Load Balancing**: 如果配置了负载均衡，匹配 VIP 并执行 `ct_lb;`。

### Ingress Table 6: IPv6 ND RA options
**入站表 6: IPv6 ND RA 选项**

处理 IPv6 路由器通告 (RA) 的选项。

### Ingress Table 7: ND RA Response
**入站表 7: ND RA 响应**

生成 IPv6 RA 响应。

### Ingress Table 8: IP Routing
**入站表 8: IP 路由**

执行核心的路由查找。

*   **静态路由**: 根据 `Logical_Router_Static_Route` 表生成。
*   **直连路由**: 根据路由器端口配置的网段生成。
*   **最长前缀匹配**: 优先级基于子网掩码长度。
*   **动作**:
    *   `ip.ttl--;` (TTL 减 1)
    *   `reg0 = NEXT_HOP_IP;` (设置下一跳 IP)
    *   `reg1 = OUTPORT_IP;` (设置出接口 IP)
    *   `outport = "OUTPORT_NAME";` (设置出接口)
    *   `flags.loopback = 1;` (如果是发往同子网的流量)
    *   `next;`

### Ingress Table 9: IP Routing - ECMP
**入站表 9: ECMP 路由选择**

如果存在多条等价路由 (ECMP)，此表使用哈希算法选择其中一条。

*   匹配 `ecmp_group_id`。
*   动作 `select(group_id, ...);`。

### Ingress Table 10: Policy
**入站表 10: 策略路由**

根据 `Logical_Router_Policy` 表执行策略路由（基于源 IP、端口等更灵活的条件）。

### Ingress Table 11: ARP/ND Resolution
**入站表 11: ARP/ND 解析**

在数据包发送之前，需要解析下一跳 IP 的 MAC 地址。

*   **已知邻居**: 如果 ARP/ND 表中有下一跳的 MAC 地址。
    *   匹配 `ip4.dst == NEXT_HOP_IP`。
    *   动作 `eth.dst = NEXT_HOP_MAC; next;`。
*   **未知邻居**: 如果没有 MAC 地址，需要生成 ARP 请求。
    *   动作 `get_arp(outport, NEXT_HOP_IP);`。

### Ingress Table 12: Check DNAT local
**入站表 12: 检查 DNAT 本地性**

检查经过 DNAT 的数据包是否需要发送到分布式网关端口。

### Ingress Table 13: Gateway Redirect
**入站表 13: 网关重定向**

对于分布式网关端口，如果流量需要进入物理网络，可能需要重定向到特定的 Chassis（通常是网关节点）。

### Ingress Table 14: ARP Request
**入站表 14: ARP 请求生成**

处理 `get_arp` 动作生成的 ARP 请求逻辑。

---

## Egress Pipeline (出站管道)

### Egress Table 0: Check DNAT local
**出站表 0: 检查 DNAT 本地性**

此表检查在 Ingress 管道中被 SNAT 并环回的数据包是否需要在 `lr_in_dnat` 中进行 DNAT。这仅适用于配置了分布式网关端口和 NAT 条目的路由器。目的是确保 SNAT 和 DNAT 在不同的区域（Zone）中完成，而不是在同一个公共区域。

### Egress Table 1: UNDNAT
**出站表 1: UNDNAT (反向 DNAT)**

处理已建立连接的返回流量。即在 Ingress 管道中做过 DNAT 的流量，其回复报文在此处需要做 unDNAT（将源 IP 从内部 IP 改回外部 VIP）。

*   **Gateway Routers**:
    *   匹配 `ip`，动作 `flags.loopback = 1; ct_dnat;`。`ct_dnat` 在没有参数时，会根据 CT 状态反转之前的 DNAT。
*   **Distributed Routers**:
    *   针对负载均衡和 NAT 规则生成的流表，匹配源 IP 为后端 IP，动作为 `ct_dnat;`。

### Egress Table 2: Post UNDNAT
**出站表 2: UNDNAT 后处理**

*   **CT 初始化**: 对于分布式路由器上配置为 SNAT 的流量，初始化 CT 状态。
*   **CT Commit**: 对于网关路由器，提交未跟踪的流。`ct_commit { }; next;`。

### Egress Table 3: SNAT
**出站表 3: SNAT (源地址转换)**

根据 `OVN_Northbound` 数据库配置更改源 IP 地址。

*   **Gateway Routers**:
    *   **Force SNAT**: 如果配置了强制 SNAT（如负载均衡流量），执行 `ct_snat(ROUTER_IP);`。
    *   **NAT Rules**: 根据 NAT 规则，匹配 `ip4.src == INTERNAL_IP`，动作 `ct_snat(EXTERNAL_IP);`。
*   **Distributed Routers**:
    *   类似于网关路由器，但处理更复杂，可能涉及分布式处理和 MAC 地址调整。

### Egress Table 4: Post SNAT
**出站表 4: SNAT 后处理**

*   **SNAT Zone Commit**: 对于直接进入配置了 SNAT 的网络的流量（且由外部网络发起），将其提交到 SNAT CT Zone。这确保了从 SNAT 网络返回的回复不会被错误地 SNAT。

### Egress Table 5: Egress Loopback
**出站表 5: 出站环回**

对于分布式逻辑路由器，某些流量（如经过 NAT 处理后的流量）需要强制通过分布式网关端口进行“出站环回”，以便再次进入 Ingress 管道进行路由和 ARP 解析。

*   **Clone & Loopback**:
    *   匹配 `ip4.dst == EXTERNAL_IP`。
    *   动作 `clone { ct_clear; inport = outport; outport = ""; flags.loopback = 1; next(pipeline=ingress, table=0); };`。

### Egress Table 6: Delivery
**出站表 6: 交付**

数据包准备好发送。

*   **Output**: 匹配每个启用的逻辑路由器端口，动作 `output;`。
*   **Drop**: 默认丢弃未匹配的数据包。
