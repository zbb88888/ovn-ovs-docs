# Load_Balancer 表

每行代表一个负载均衡器。

## 摘要

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `name` | 字符串 | 负载均衡器名称。 |
| `vips` | 字符串对映射 | 虚拟 IP 到后端的映射。 |
| `protocol` | 可选字符串，`sctp`, `tcp`, 或 `udp` 之一 | 协议。 |
| `health_check` | `Load_Balancer_Health_Check` 的集合 | 健康检查。 |
| `ip_port_mappings` | 字符串对映射 | IP 端口映射。 |
| `selection_fields` | 字符串集合 | 选择字段。 |

**公共列：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `external_ids` | 字符串对映射 | 外部 ID。 |

**Load_Balancer 选项：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `options : reject` | 可选字符串，`true` 或 `false` | 是否拒绝连接。 |
| `options : hairpin_snat_ip` | 可选字符串 | 发夹 SNAT IP。 |
| `options : skip_snat` | 可选字符串 | 跳过 SNAT。 |
| `options : add_route` | 可选字符串 | 是否添加路由。 |
| `options : neighbor_responder` | 可选字符串 | 邻居响应模式。 |
| `options : template` | 可选字符串 | 是否为模板。 |
| `options : address-family` | 可选字符串 | 地址族。 |
| `options : affinity_timeout` | 可选字符串 | 亲和性超时。 |
| `options : ct_flush` | 可选字符串，`true` 或 `false` | 是否刷新 CT。 |
| `options : use_stateless_nat` | 可选字符串，`true` 或 `false` | 是否使用无状态 NAT。 |

## 详细信息

-   `name`: 字符串
    负载均衡器的名称。此名称除了方便人类与 ovn-nb 数据库交互之外，没有特殊的含义或目的。

-   `vips`: 字符串对映射
    与此负载均衡器关联的虚拟 IP 地址（以及可选的端口号，以 : 作为分隔符）与其对应的端点 IP 地址（以及可选的端口号，以 : 作为分隔符，以逗号分隔）的映射。如果离开容器或 VM 的数据包的目标 IP 地址（和端口号）匹配此处作为键提供的虚拟 IP 地址（和端口号），则 OVN 将有状态地将目标 IP 地址替换为此映射中作为值提供的 IP 地址（和端口号）之一。负载均衡支持 IPv4 和 IPv6 地址；但是，一个地址族的 VIP 不能映射到不同地址族的目标 IP 地址。如果指定带有端口的 IPv6 地址，则地址部分必须括在方括号中。键的示例是 "192.168.1.4" 和 "[fd0f::1]:8800"。值的示例是 "10.0.0.1, 10.0.0.2" 和 "20.0.0.10:8800, 20.0.0.11:8800"。

    当 `Load_Balancer` 添加到 `logical_switch` 时，VIP 必须位于与 `logical_switch` 使用的子网不同的子网中。由于 VIP 在不同的子网中，您应该将逻辑交换机连接到 OVN 逻辑路由器或真实路由器（这是因为客户端现在可以发送以 VIP 为目标 IP 地址、以路由器 MAC 地址为目标 MAC 地址的数据包）。

-   `protocol`: 可选字符串，`sctp`, `tcp`, 或 `udp` 之一
    有效的协议是 `tcp`, `udp`, 或 `sctp`。当 `vips` 列的一部分提供了端口号时，此列很有用。如果此列为空并且 `vips` 列的一部分提供了端口号，则 OVN 假定协议为 `tcp`。

### 健康检查

OVN 支持负载均衡器端点的健康检查。当启用健康检查时，负载均衡器仅使用健康的端点。

假设 `vips` 包含键值对 `10.0.0.10:80=10.0.0.4:8080,20.0.0.4:8080`。要为此虚拟 IP 的端点启用健康检查，请向 `ip_port_mappings` 添加两个键值对，键为 `10.0.0.4` 和 `20.0.0.4`，并向 `health_check` 添加对 `vip` 设置为 `10.0.0.10` 的 `Load_Balancer_Health_Check` 行的引用。同样的方法也可用于 IPv6。

-   `health_check`: `Load_Balancer_Health_Check` 的集合
    与此负载均衡器关联的负载均衡器健康检查。

-   `ip_port_mappings`: 字符串对映射
    从端点 IP 映射到以冒号分隔的逻辑端口名称和源 IP 对，例如 IPv4 的 `port_name:sourc_ip`。健康检查使用指定的源 IP 发送到此端口。对于 IPv6，IP 地址周围必须使用方括号，例如：`port_name:[sourc_ip]`。远程端点：在上述语法末尾指定 `:target_zone_name` 以在特定区域中创建远程健康检查。

    例如，在上面的示例中，IP 到端口的映射可以定义为 `10.0.0.4=sw0-p1:10.0.0.2` 和 `20.0.0.4=sw1-p1:20.0.0.2`，如果给定的值是合适的端口和 IP 地址。远程端点：`10.0.0.4=sw0-p1:10.0.0.2:az1`，其中 sw0-p1 是 az1 中的逻辑端口。

    对于 IPv6，IP 到端口的映射可以定义为 `[2001::1]=sw0-p1:[2002::1]`。远程端点：与 IP 相同。

-   `selection_fields`: 字符串集合，`eth_dst`, `eth_src`, `ip_dst`, `ip_src`, `ipv6_dst`, `ipv6_src`, `tp_dst`, 或 `tp_src` 之一
    OVN 原生负载均衡器支持使用类型为 `select` 的 OpenFlow 组。OVS 支持两种选择方法：`dp_hash` 和 `hash`（指定可选字段）来选择组的桶。有关选择方法的更多详细信息，请参阅 OVS 文档 (man ovs-ofctl)。每个端点 IP（如果设置了端口，则包括端口）都映射到组流中的一个桶。

    CMS 可以通过在此列中设置选择字段来选择哈希选择方法。`ovs-vswitchd` 使用指定的字段生成哈希。

    示例：`{ip_proto,ip_src,ip_dst}` 用于 3 元组匹配。
    示例：`{ip_proto,ipv6_src,ipv6_dst}` 用于 IPv6 匹配。
    示例：`{ip_proto,ip_src,ip_dst,tp_src,tp_dst}`。
    示例：`{ip_src,ip_dst,ipv6_src,ipv6_dst,tp_src,tp_dst}`。

    `dp_hash` 选择方法利用数据路径的协助来计算哈希，预计比 `hash` 选择方法更快。因此，CMS 在使用 `hash` 方法之前应考虑到这一点。请查阅 OVS 文档和 OVS 源代码以了解实现细节。

### 公共列

-   `external_ids`: 字符串对映射
    参见本文件开头的外部 ID。

### Load_Balancer 选项

-   `options : reject`: 可选字符串，`true` 或 `false`
    如果负载均衡器是使用 `--reject` 选项创建的，并且它没有活动的后端，则每当收到此负载均衡器的传入数据包时，都会发送 TCP 重置段（对于 tcp）或 ICMP 端口不可达数据包（对于所有其他类型的流量）。请注意，使用 `--reject` 选项将禁用此负载均衡器的 `empty_lb` SB 控制器事件。

-   `options : hairpin_snat_ip`: 可选字符串
    用于在负载均衡后发生发夹（hairpin）的数据包的源 IP。未设置该选项时的默认行为是使用负载均衡器 VIP 作为源 IP。此选项可以包含恰好一个 IPv4 和/或一个 IPv6 地址，以空格分隔。

-   `options : skip_snat`: 可选字符串
    如果负载均衡规则配置了 `skip_snat` 选项，则引用此负载均衡器的逻辑路由器配置的 `lb_force_snat_ip` 选项将不会应用于此负载均衡器。

-   `options : add_route`: 可选字符串
    如果设置为 true，则邻居路由器将添加逻辑流，允许路由到 VIP IP。它还将添加 ARP 解析逻辑流。通过设置此选项，意味着没有理由从邻居路由器创建到此 NAT 地址的 `Logical_Router_Static_Route`。这也意味着邻居路由器不需要 ARP 请求来学习此 VIP IP 的 IP-MAC 映射。有关为 IP 路由添加哪些流的更多信息，请参阅 ovn-northd 手册页关于 IP 路由的部分。

-   `options : neighbor_responder`: 可选字符串
    如果设置为 `all`，则应用负载均衡器的路由器将回复负载均衡器所有 VIP 的 ARP/邻居发现请求。如果设置为 `reachable`，则应用负载均衡器的路由器仅回复属于路由器子网一部分的 VIP 的 ARP/邻居发现请求。如果设置为 `none`，则应用负载均衡器的路由器从不回复任何负载均衡器 VIP 的 ARP/邻居发现请求。`options:template=true` 的负载均衡器不支持 `reachable` 作为有效模式。如果未指定，此选项的默认值对于常规负载均衡器是 `reachable`，对于模板负载均衡器是 `none`。

-   `options : template`: 可选字符串
    如果负载均衡器是模板，则将该选项设置为 true。负载均衡器 VIP 和后端必须在其定义中使用 `Chassis_Template_Var`。

    负载均衡器模板 VIP 支持的格式为：

    `^VIP_VAR[:^PORT_VAR|:port]`

    其中 `VIP_VAR` 和 `PORT_VAR` 是 `Chassis_Template_Var` 变量记录的键。

    注意：VIP 和 PORT 不能合并为单个模板变量。例如，扩展为 `10.0.0.1:8080` 的 `Chassis_Template_Var` 变量如果用作 VIP 是无效的。

    负载均衡器模板后端支持的格式为：

    `^BACKEND_VAR1[:^PORT_VAR1|:port],^BACKEND_VAR2[:^PORT_VAR2|:port]`
    或
    `^BACKENDS_VAR1,^BACKENDS_VAR2`

    其中 `BACKEND_VAR1`, `PORT_VAR1`, `BACKEND_VAR2`, `PORT_VAR2`, `BACKENDS_VAR1` 和 `BACKENDS_VAR2` 是 `Chassis_Template_Var` 变量记录的键。

    在上面的示例中，方括号仅用于指示在包含的选项之间进行选择。但是，当添加到 `Chassis_Template_Var` 变量时，后端 IPv6 地址必须括在 [] 中。VIP 不得括在 [] 中。例如：`lb_vip="3001::10",lbport=12010,lbback="[2001::1]", ip6_back="[2001::1]:12010"`

-   `options : address-family`: 可选字符串
    负载均衡器使用的地址族。支持的值为 `ipv4` 和 `ipv6`。`address-family` 仅用于 `options:template=true` 的负载均衡器。对于显式负载均衡器，设置 `address-family` 没有效果。

-   `options : affinity_timeout`: 可选字符串
    如果 CMS 为 `affinity_timeout` 提供正值（以秒为单位），OVN 将在亲和性时间段内将从同一客户端接收到的到此 lb 的连接 DNAT 到同一后端。最大支持的 `affinity_timeout` 为 65535 秒。

-   `options : ct_flush`: 可选字符串，`true` 或 `false`
    该值指示 `ovn-controller` 是否应刷新与此 LB 相关的 CT 条目。如果 LB 被删除、任何后端被更新/删除或 LB 不再被 `ovn-controller` 视为本地，则会发生刷新。默认情况下，此选项设置为 false。

-   `options : use_stateless_nat`: 可选字符串，`true` 或 `false`
    如果负载均衡器配置了 `use_stateless_nat` 选项为 true，则当逻辑路由器具有多个分布式网关端口 (DGP) 时，引用此负载均衡器的逻辑路由器将使用无状态 NAT 规则。否则，在每个 DGP 具有不同 Chassis 的场景中，出站流量可能会被丢弃。默认情况下，此选项设置为 false。

---

# Load_Balancer_Group 表

每行代表一组负载均衡器的逻辑分组。由 CMS 决定负载均衡器的分组标准。为了简化配置并优化其处理，必须关联到同一组逻辑交换机和/或逻辑路由器的负载均衡器应该分组在一起。

## 摘要

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `name` | 字符串（表内必须唯一） | 负载均衡器组名称。 |
| `load_balancer` | `Load_Balancer` 的弱引用集合 | 负载均衡器集合。 |

## 详细信息

-   `name`: 字符串（表内必须唯一）
    负载均衡器组的名称。此名称除了方便人类与 ovn-nb 数据库交互之外，没有特殊的含义或目的。

-   `load_balancer`: `Load_Balancer` 的弱引用集合
    一组负载均衡器。

---

# Load_Balancer_Health_Check 表

每行代表一个负载均衡器健康检查。

## 摘要

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `vip` | 字符串 | VIP。 |

**健康检查选项：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `options : interval` | 可选字符串，包含整数 | 检查间隔。 |
| `options : timeout` | 可选字符串，包含整数 | 超时时间。 |
| `options : success_count` | 可选字符串，包含整数 | 成功计数。 |
| `options : failure_count` | 可选字符串，包含整数 | 失败计数。 |

**公共列：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `external_ids` | 字符串对映射 | 外部 ID。 |

## 详细信息

-   `vip`: 字符串
    应监控其端点以进行健康检查的 vip。

### 健康检查选项

-   `options : interval`: 可选字符串，包含整数
    健康检查之间的间隔，以秒为单位。

-   `options : timeout`: 可选字符串，包含整数
    健康检查超时的秒数。

-   `options : success_count`: 可选字符串，包含整数
    端点被认为在线之前的成功检查次数。

-   `options : failure_count`: 可选字符串，包含整数
    端点被认为离线之前的失败检查次数。

### 公共列

-   `external_ids`: 字符串对映射
    参见本文件开头的外部 ID。
