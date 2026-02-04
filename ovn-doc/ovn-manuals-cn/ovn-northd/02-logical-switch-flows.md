# OVN 逻辑交换机流表 (Logical Switch Flows)

`ovn-northd` 为每个逻辑交换机生成两组逻辑流管道：Ingress（入站）和 Egress（出站）。

## Ingress Pipeline (入站管道)

数据包进入逻辑交换机时首先通过 Ingress 管道处理。

### Ingress Table 0: Admission Control and Port Security - L2
**入站表 0: 准入控制与端口安全 - L2**

此表负责丢弃那些不应被逻辑交换机处理的数据包，并执行 L2 端口安全检查。

*   **广播/组播源 MAC 丢弃**: 优先级 100，匹配 `eth.src[40]` (广播/组播位)，动作 `drop;`。
*   **端口安全 (Port Security)**:
    *   如果逻辑端口启用了 `port_security`，ovn-northd 会添加流表以确保数据包的源 MAC 地址与绑定的端口允许的地址匹配。
    *   优先级 50: 匹配 `inport == "PORT_NAME" && eth.src == {MAC1, MAC2, ...}`，动作 `next;`。
    *   优先级 0: 默认丢弃规则。

### Ingress Table 1: Admission Control and Port Security - IP
**入站表 1: 准入控制与端口安全 - IP**

此表执行 IP 级别的端口安全检查。

*   **IP 端口安全**:
    *   对于启用了 IP 端口安全的端口，检查源 IP 是否在允许列表中。
    *   优先级 90 (IPv4) / 80 (IPv6): 匹配 `inport == "PORT" && eth.src == MAC && ip4.src == {IP1, IP2...}`，动作 `next;`。
*   **默认规则**: 优先级 0，动作 `next;` (如果未配置 IP 安全，则允许所有)。

### Ingress Table 2: Defragmentation
**入站表 2: 分片重组**

*   **追踪分片**: 如果启用了连接跟踪 (conntrack) 且需要处理分片，此处会将数据包发送到连接跟踪器进行重组。
    *   匹配 `ip && ip.frag`，动作 `ct_next;`。

### Ingress Table 3: LB and Service Monitor
**入站表 3: 负载均衡与服务监控**

此表处理与负载均衡器 (Load Balancer) 和服务监控相关的逻辑。

*   **Service Monitor**: 如果配置了服务监控，生成流表以响应服务监控请求。
*   **Load Balancer VIPs**: 尽管实际的负载均衡通常在后续表中进行，但某些特定场景（如 SCTP）可能在此处处理。

### Ingress Table 4: Pre-ACLs
**入站表 4: ACL 前处理**

此表用于准备 ACL 处理，主要是确立连接跟踪 (CT) 状态。

*   **CT Zone 分配**: 将数据包发送到 CT 模块以获取连接状态（New, Established, Related 等）。
    *   动作 `ct_next;`。

### Ingress Table 5: Pre-LB
**入站表 5: 负载均衡前处理**

为负载均衡做准备，可能涉及捕获某些流量以跳过 LB 或进行特定标记。

### Ingress Table 6: Pre-stateful
**入站表 6: 状态化前处理**

此表通常用于处理那些不需要经过状态化防火墙（ACL）或负载均衡的流量，或者在进入状态化处理前进行最后的准备。

### Ingress Table 7: ACLs
**入站表 7: 访问控制列表**

这是核心的安全策略实施表。根据 `OVN_Northbound` 数据库中的 ACL 表生成流表。

*   **allow-related**: 允许已建立连接的流量。
    *   匹配 `ct.est`，动作 `next;`。
*   **from-lport ACLs**: 处理方向为 `from-lport` 的 ACL 规则。
    *   根据优先级和匹配条件生成流表，动作为 `allow` (即 `next;`) 或 `drop` / `reject`。
*   **Stateful ACLs**: 对于 `allow-related` 的新连接 (`ct.new`)，执行 `ct_commit;` 以记录连接状态。

### Ingress Table 8: QoS
**入站表 8: 服务质量**

根据 QoS 配置对数据包进行标记 (DSCP) 或限速。

### Ingress Table 9: LB Affinity Check
**入站表 9: 负载均衡亲和性检查**

处理负载均衡的会话保持 (Affinity) 逻辑。如果数据包属于已存在的 LB 会话，确保其被转发到相同的后端。

### Ingress Table 10: LB
**入站表 10: 负载均衡**

执行 DNAT 类型的负载均衡。

*   **VIP 匹配**: 匹配 `ip && ip.dst == VIP && tcp/udp.dst == PORT`。
*   **动作**: `ct_lb(backends);`。这将选择一个后端并修改目标 IP/端口。

### Ingress Table 11: LB Hairpin
**入站表 11: 负载均衡发卡 (Hairpin)**

处理 Hairpin 流量（即源和目的在同一子网，但经过 LB 的流量）。需要进行 SNAT 以确保回程流量经过 LB。

### Ingress Table 12: ARP/ND Responder
**入站表 12: ARP/ND 响应器**

OVN 在此处代答 ARP/ND 请求，减少广播流量。

*   **Known IPs**: 对于逻辑交换机端口已知的 IP 地址，直接回复 ARP 响应。
    *   匹配 `arp.tpa == IP && arp.op == 1`。
    *   动作 `arp { ...; output; };`。
*   **Virtual IPs**: 对于负载均衡 VIP 或 NAT 地址，也在此处代答。

### Ingress Table 13: DHCP option processing
**入站表 13: DHCP 选项处理**

如果数据包是 DHCP 请求，OVN 在此处准备 DHCP 选项（如 IP 地址、网关、DNS 等）。

*   匹配 `udp.src == 68 && udp.dst == 67`。
*   动作 `put_dhcp_opts(...); next;`。

### Ingress Table 14: DHCP responses
**入站表 14: DHCP 响应**

生成 DHCP 响应数据包并发回给请求者。

*   匹配 `udp.src == 68 && udp.dst == 67 && dhcp_opts_present`。
*   动作 `dhcp_response;` (或 `dhcpv6_response;`)。

### Ingress Table 15: DNS Lookup
**入站表 15: DNS 查询**

处理 DNS 请求，根据配置的 DNS 记录进行回复。

### Ingress Table 16: External Logic
**入站表 16: 外部逻辑**

保留给外部系统插入自定义逻辑的位置。

### Ingress Table 17: Destination Lookup
**入站表 17: 目的查找**

这是 Ingress 管道的最后一步，决定数据包的去向（输出端口）。

*   **已知单播 MAC**: 匹配 `eth.dst == MAC`，动作 `outport = "PORT"; output;`。
*   **广播/组播**: 匹配 `eth.dst[40]`，动作 `mc_flood;` (泛洪到所有属于该广播域的端口)。
*   **未知单播**: 通常也进行泛洪 (`mc_unknown`)。

---

## Egress Pipeline (出站管道)

数据包被 Ingress 管道的 `output` 动作发送到某个端口后，进入该端口的 Egress 管道。

### Egress Table 0: Pre-LB
**出站表 0: 负载均衡前处理**

类似于 Ingress 的 Pre-LB，为出站方向的 LB 做准备。

### Egress Table 1: Pre-ACLs
**出站表 1: ACL 前处理**

准备出站 ACL 处理，通常涉及 `ct_next` 以获取连接状态。

### Egress Table 2: Pre-stateful
**出站表 2: 状态化前处理**

类似于 Ingress 的 Pre-stateful。

### Egress Table 3: LB
**出站表 3: 负载均衡**

通常用于 SNAT 类型的负载均衡，或者在出站方向需要反转某些 LB 操作的场景。

### Egress Table 4: Stateful
**出站表 4: 状态化处理**

处理 `ct_commit` 等状态化逻辑。

### Egress Table 5: ACLs
**出站表 5: 访问控制列表**

处理方向为 `to-lport` 的 ACL 规则。

*   **to-lport ACLs**: 检查发往目标端口的流量是否被允许。
    *   匹配条件并执行 `allow` (next) 或 `drop`。

### Egress Table 6: QoS
**出站表 6: 服务质量**

出站方向的 QoS 标记。

### Egress Table 7: Port Security - IP
**出站表 7: 端口安全 - IP**

检查发往端口的数据包的 IP 地址是否与端口配置匹配（防止欺骗或错误转发）。但通常 Port Security 主要在 Ingress 检查源，Egress 检查目的的情况较少，视配置而定。

### Egress Table 8: Port Security - L2
**出站表 8: 端口安全 - L2**

检查发往端口的数据包的 MAC 地址。

*   **MAC 匹配**: 确保 `eth.dst` 与输出端口的 MAC 地址匹配。
*   **动作**: `output;`。这是 Egress 管道的最后一步，数据包被实际发送到物理接口或隧道。
