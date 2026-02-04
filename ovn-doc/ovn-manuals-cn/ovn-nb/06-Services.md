# DHCP_Options 表

OVN 实现了原生的 DHCPv4 支持，以满足通过提供基于静态配置的地址映射的无状态 DHCPv4 回复来向启动实例提供 IPv4 地址的常见用例。为此，它允许配置一小部分 DHCPv4 选项，并将其应用于运行 ovn-controller 的每个计算主机。

OVN 还实现了原生的 DHCPv6 支持，提供对 DHCPv6 请求的无状态回复。

## 摘要

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `cidr` | 字符串 | CIDR。 |

**DHCPv4 选项：**

**强制性 DHCPv4 选项：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `options : server_id` | 可选字符串 | 服务器 ID。 |
| `options : server_mac` | 可选字符串 | 服务器 MAC。 |
| `options : lease_time` | 可选字符串，0 到 4,294,967,295 的整数 | 租约时间。 |

**IPv4 DHCP 选项：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `options : router` | 可选字符串 | 路由器（网关）。 |
| `options : netmask` | 可选字符串 | 子网掩码。 |
| `options : dns_server` | 可选字符串 | DNS 服务器。 |
| `options : log_server` | 可选字符串 | 日志服务器。 |
| `options : lpr_server` | 可选字符串 | LPR 服务器。 |
| `options : swap_server` | 可选字符串 | Swap 服务器。 |
| `options : policy_filter` | 可选字符串 | 策略过滤器。 |
| `options : router_solicitation` | 可选字符串 | 路由器请求。 |
| `options : nis_server` | 可选字符串 | NIS 服务器。 |
| `options : ntp_server` | 可选字符串 | NTP 服务器。 |
| `options : netbios_name_server` | 可选字符串 | NetBIOS 名称服务器。 |
| `options : classless_static_route` | 可选字符串 | 无类静态路由。 |
| `options : ms_classless_static_route` | 可选字符串 | MS 无类静态路由。 |
| `options : next_server` | 可选字符串 | 下一跳服务器。 |

**布尔 DHCP 选项：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `options : ip_forward_enable` | 可选字符串，`0` 或 `1` | 启用 IP 转发。 |
| `options : router_discovery` | 可选字符串，`0` 或 `1` | 路由器发现。 |
| `options : ethernet_encap` | 可选字符串，`0` 或 `1` | 以太网封装。 |

**整数 DHCP 选项：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `options : default_ttl` | 可选字符串，0 到 255 的整数 | 默认 TTL。 |
| `options : tcp_ttl` | 可选字符串，0 到 255 的整数 | TCP TTL。 |
| `options : mtu` | 可选字符串，68 到 65,535 的整数 | MTU。 |
| `options : T1` | 可选字符串，68 到 4,294,967,295 的整数 | T1 计时器。 |
| `options : T2` | 可选字符串，68 到 4,294,967,295 的整数 | T2 计时器。 |
| `options : arp_cache_timeout` | 可选字符串，0 到 255 的整数 | ARP 缓存超时。 |
| `options : tcp_keepalive_interval` | 可选字符串，0 到 255 的整数 | TCP 保活间隔。 |
| `options : netbios_node_type` | 可选字符串，0 到 255 的整数 | NetBIOS 节点类型。 |

**字符串 DHCP 选项：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `options : wpad` | 可选字符串 | WPAD URL。 |
| `options : bootfile_name` | 可选字符串 | 引导文件名。 |
| `options : path_prefix` | 可选字符串 | 路径前缀。 |
| `options : tftp_server_address` | 可选字符串 | TFTP 服务器地址。 |
| `options : hostname` | 可选字符串 | 主机名。 |
| `options : domain_name` | 可选字符串 | 域名。 |
| `options : bootfile_name_alt` | 可选字符串 | 备用引导文件名。 |
| `options : broadcast_address` | 可选字符串 | 广播地址。 |

**host_id 类型的 DHCP 选项：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `options : tftp_server` | 可选字符串 | TFTP 服务器。 |

**domains 类型的 DHCP 选项：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `options : domain_search_list` | 可选字符串 | 域名搜索列表。 |

**DHCPv6 选项：**

**强制性 DHCPv6 选项：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `options : server_id` | 可选字符串 | 服务器 ID。 |

**IPv6 DHCPv6 选项：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `options : dns_server` | 可选字符串 | DNS 服务器。 |

**字符串 DHCPv6 选项：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `options : domain_search` | 可选字符串 | 域名搜索。 |
| `options : dhcpv6_stateless` | 可选字符串 | 无状态 DHCPv6。 |
| `options : fqdn` | 可选字符串 | FQDN。 |

**公共列：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `external_ids` | 字符串对映射 | 外部 ID。 |

## 详细信息

-   `cidr`: 字符串
    如果逻辑端口的 IP 地址在此 cidr 中，则将包含 DHCPv4/DHCPv6 选项。

### DHCPv4 选项

CMS 应将一组 DHCPv4 选项定义为此表中 options 列中的键/值对。为了让 ovn-controller 包含这些 DHCPv4 选项，`Logical_Switch_Port` 的 `dhcpv4_options` 应引用此表中的条目。

#### 强制性 DHCPv4 选项

必须定义以下选项。

-   `options : server_id`: 可选字符串
    DHCP 服务器使用的 IP 地址。这应该在提供的 IP 的子网中。这也会作为选项 54 “服务器标识符”包含在 DHCP offer 中。

-   `options : server_mac`: 可选字符串
    DHCP 服务器使用的以太网地址。

-   `options : lease_time`: 可选字符串，包含 0 到 4,294,967,295 范围内的整数
    提供的租约时间，以秒为单位。DHCPv4 选项代码为 51。

#### IPv4 DHCP 选项

以下是支持的 DHCPv4 选项，其值为 IPv4 地址，例如 192.168.1.1。某些选项接受括在花括号内的多个 IPv4 地址，例如 {192.168.1.2, 192.168.1.3}。有关 DHCPv4 选项及其代码的更多详细信息，请参阅 RFC 2132。

-   `options : router`: 可选字符串
    客户端使用的网关 IP 地址。这应该在提供的 IP 的子网中。此选项的 DHCPv4 选项代码为 3。

-   `options : netmask`: 可选字符串
    此选项的 DHCPv4 选项代码为 1。

-   `options : dns_server`: 可选字符串
    此选项的 DHCPv4 选项代码为 6。

-   `options : log_server`: 可选字符串
    此选项的 DHCPv4 选项代码为 7。

-   `options : lpr_server`: 可选字符串
    此选项的 DHCPv4 选项代码为 9。

-   `options : swap_server`: 可选字符串
    此选项的 DHCPv4 选项代码为 16。

-   `options : policy_filter`: 可选字符串
    此选项的 DHCPv4 选项代码为 21。

-   `options : router_solicitation`: 可选字符串
    此选项的 DHCPv4 选项代码为 32。

-   `options : nis_server`: 可选字符串
    此选项的 DHCPv4 选项代码为 41。

-   `options : ntp_server`: 可选字符串
    此选项的 DHCPv4 选项代码为 42。

-   `options : netbios_name_server`: 可选字符串
    此选项的 DHCPv4 选项代码为 44。

-   `options : classless_static_route`: 可选字符串
    此选项的 DHCPv4 选项代码为 121。

    此选项可以包含一个或多个静态路由，每个路由由目标描述符和用于到达该目标的路由器的 IP 地址组成。有关更多详细信息，请参阅 RFC 3442。

    示例：`{30.0.0.0/24,10.0.0.10, 0.0.0.0/0,10.0.0.1}`

-   `options : ms_classless_static_route`: 可选字符串
    此选项的 DHCPv4 选项代码为 249。此选项类似于 Microsoft Windows DHCPv4 客户端支持的 `classless_static_route`。

-   `options : next_server`: 可选字符串
    用于设置 DHCP 头中“下一跳服务器 IP 地址”字段的 DHCPv4 选项代码。

#### 布尔 DHCP 选项

这些选项接受布尔值，表示为 0（假）或 1（真）。

-   `options : ip_forward_enable`: 可选字符串，`0` 或 `1`
    此选项的 DHCPv4 选项代码为 19。

-   `options : router_discovery`: 可选字符串，`0` 或 `1`
    此选项的 DHCPv4 选项代码为 31。

-   `options : ethernet_encap`: 可选字符串，`0` 或 `1`
    此选项的 DHCPv4 选项代码为 36。

#### 整数 DHCP 选项

这些选项接受非负整数值。

-   `options : default_ttl`: 可选字符串，包含 0 到 255 范围内的整数
    此选项的 DHCPv4 选项代码为 23。

-   `options : tcp_ttl`: 可选字符串，包含 0 到 255 范围内的整数
    此选项的 DHCPv4 选项代码为 37。

-   `options : mtu`: 可选字符串，包含 68 到 65,535 范围内的整数
    此选项的 DHCPv4 选项代码为 26。

-   `options : T1`: 可选字符串，包含 68 到 4,294,967,295 范围内的整数
    这指定了从地址分配到客户端开始尝试续订其地址的时间间隔。此选项的 DHCPv4 选项代码为 58。

-   `options : T2`: 可选字符串，包含 68 到 4,294,967,295 范围内的整数
    这指定了从地址分配到客户端开始尝试重新绑定其地址的时间间隔。此选项的 DHCPv4 选项代码为 59。

-   `options : arp_cache_timeout`: 可选字符串，包含 0 到 255 范围内的整数
    此选项的 DHCPv4 选项代码为 35。此选项指定 ARP 缓存条目的超时时间（以秒为单位）。

-   `options : tcp_keepalive_interval`: 可选字符串，包含 0 到 255 范围内的整数
    此选项的 DHCPv4 选项代码为 38。此选项指定客户端 TCP 在 TCP 连接上发送保活消息之前应等待的间隔。

-   `options : netbios_node_type`: 可选字符串，包含 0 到 255 范围内的整数
    此选项的 DHCPv4 选项代码为 46。

#### 字符串 DHCP 选项

这些选项接受字符串值。

-   `options : wpad`: 可选字符串
    此选项的 DHCPv4 选项代码为 252。此选项用作 Web 代理自动发现的一部分，以提供 Web 代理的 URL。

-   `options : bootfile_name`: 可选字符串
    此选项的 DHCPv4 选项代码为 67。此选项用于标识引导文件。

-   `options : path_prefix`: 可选字符串
    此选项的 DHCPv4 选项代码为 210。在 PXELINUX 的情况下，此选项用于设置通用路径前缀，而不是从引导文件名派生。

-   `options : tftp_server_address`: 可选字符串
    此选项的 DHCPv4 选项代码为 150。该选项包含客户端可以使用的其中一个或多个 IPv4 地址。此选项是 Cisco 专有的，与此要求匹配的 IEEE 标准是选项 66 (tftp_server)。

-   `options : hostname`: 可选字符串
    此选项的 DHCPv4 选项代码为 12。如果设置，则表示 DHCPv4 选项“主机名”。或者，可以在 `Logical_Switch_Port` 表的 `options:hostname` 列中配置此选项。如果在冲突的 `Logical_Switch_Port` 和 `DHCP_Options` 表中都设置了 Hostname 选项值，则 `Logical_Switch_Port` 优先。

-   `options : domain_name`: 可选字符串
    此选项的 DHCPv4 选项代码为 15。此选项指定客户端在通过域名系统解析主机名时应使用的域名。

-   `options : bootfile_name_alt`: 可选字符串
    "bootfile_name_alt" 选项用于支持 iPXE。当 CMS 同时提供 "bootfile_name" 和 "bootfile_name_alt" 时，如果 dhcp 请求包含 etherboot 选项 (175)，则将使用 "bootfile_name" 作为选项 67，否则将使用 "bootfile_name_alt"。

-   `options : broadcast_address`: 可选字符串
    此选项的 DHCPv4 选项代码为 28。此选项指定用作广播地址的 IP 地址。

#### host_id 类型的 DHCP 选项

这些选项接受 IPv4 地址或字符串值。

-   `options : tftp_server`: 可选字符串
    此选项的 DHCPv4 选项代码为 66。

#### domains 类型的 DHCP 选项

这些选项接受字符串值，即逗号分隔的域名列表。域名基于 RFC 1035 进行编码。

-   `options : domain_search_list`: 可选字符串
    此选项的 DHCPv4 选项代码为 119。

### DHCPv6 选项

OVN 还实现了原生的 DHCPv6 支持。CMS 应将一组 DHCPv6 选项定义为键/值对。定义的 DHCPv6 选项将包含在对来自 cidr 中具有 IPv6 地址的逻辑端口的 DHCPv6 Solicit/Request/Confirm 数据包的 DHCPv6 响应中。

#### 强制性 DHCPv6 选项

必须定义以下选项。

-   `options : server_id`: 可选字符串
    DHCP 服务器使用的以太网地址。这也包含在 DHCPv6 回复中作为选项 2“服务器标识符”，以携带标识客户端和服务器之间服务器的 DUID。ovn-controller 基于链路层地址 [DUID-LL] 定义 DUID。

#### IPv6 DHCPv6 选项

以下是支持的 DHCPv6 选项，其值为 IPv6 地址，例如 aef0::4。某些选项接受括在花括号内的多个 IPv6 地址，例如 {aef0::4, aef0::5}。有关 DHCPv6 选项及其代码的更多详细信息，请参阅 RFC 3315。

-   `options : dns_server`: 可选字符串
    此选项的 DHCPv6 选项代码为 23。此选项指定 VM 应使用的 DNS 服务器。

#### 字符串 DHCPv6 选项

这些选项接受字符串值。

-   `options : domain_search`: 可选字符串
    此选项的 DHCPv6 选项代码为 24。此选项指定客户端应使用的域名搜索列表，以便使用 DNS 解析主机名。

    示例："ovn.org"。

-   `options : dhcpv6_stateless`: 可选字符串
    此选项指定 OVN 原生 DHCPv6 将以无状态模式工作，这意味着 OVN 原生 DHCPv6 将不为 VM/VIF 端口提供 IPv6 地址，而仅回复其他配置，例如 DNS 和域名搜索列表。当将此选项设置为字符串值 "true" 时，VM/VIF 将通过无状态方式配置 IPv6 地址。此选项的默认值为 false。

-   `options : fqdn`: 可选字符串
    此选项的 DHCPv6 选项代码为 39。如果设置，则表示 DHCPv6 选项 "FQDN"。

### 公共列

-   `external_ids`: 字符串对映射
    参见本文件开头的外部 ID。

---

# DHCP_Relay 表

OVN 实现了原生的 DHCPv4 中继支持，以满足将 DHCP 请求中继到外部 DHCP 服务器的常见用例。

## 摘要

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `name` | 字符串 | 名称。 |
| `servers` | 可选字符串 | 服务器。 |
| `options` | 字符串对映射 | 选项。 |

**公共列：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `external_ids` | 字符串对映射 | 外部 ID。 |

## 详细信息

-   `name`: 字符串
    DHCP 中继的名称。

-   `servers`: 可选字符串
    DHCPv4 服务器 IP 地址。

-   `options`: 字符串对映射
    未来用途。

### 公共列

-   `external_ids`: 字符串对映射
    参见本文件开头的外部 ID。

---

# Connection 表

配置 Open vSwitch 数据库 (OVSDB) 客户端的数据库连接。

此表主要配置 Open vSwitch 数据库服务器 (ovsdb-server)。

Open vSwitch 数据库服务器可以发起并维护到远程客户端的主动连接。它还可以侦听数据库连接。

## 摘要

**核心功能：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `target` | 字符串（表内必须唯一） | 目标。 |

**客户端故障检测和处理：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `max_backoff` | 可选整数，至少 1,000 | 最大回退时间。 |
| `inactivity_probe` | 可选整数 | 非活动探测。 |

**状态：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `is_connected` | 布尔值 | 是否已连接。 |
| `status : last_error` | 可选字符串 | 最后错误。 |
| `status : state` | 可选字符串，`ACTIVE`, `BACKOFF`, `CONNECTING`, `IDLE`, 或 `VOID` 之一 | 状态。 |
| `status : sec_since_connect` | 可选字符串，包含整数，至少 0 | 连接后秒数。 |
| `status : sec_since_disconnect` | 可选字符串，包含整数，至少 0 | 断开后秒数。 |
| `status : locks_held` | 可选字符串 | 持有的锁。 |
| `status : locks_waiting` | 可选字符串 | 等待的锁。 |
| `status : locks_lost` | 可选字符串 | 丢失的锁。 |
| `status : n_connections` | 可选字符串，包含整数，至少 2 | 连接数。 |
| `status : bound_port` | 可选字符串，包含整数 | 绑定端口。 |

**公共列：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `external_ids` | 字符串对映射 | 外部 ID。 |
| `other_config` | 字符串对映射 | 其他配置。 |

## 详细信息

### 核心功能

-   `target`: 字符串（表内必须唯一）
    客户端的连接方法。

    目前支持以下连接方法：

    *   `ssl:host[:port]`
        给定主机上指定的 SSL/TLS 端口，可以是 DNS 名称（如果构建时使用了 unbound 库）或 IP 地址。使用此形式时必须提供有效的 SSL/TLS 配置，可以通过命令行选项或 SSL 表指定此配置。

        如果未指定端口，默认为 6640。

        SSL/TLS 支持是一个可选功能，并不总是作为 OVN 或 Open vSwitch 的一部分构建。

    *   `tcp:host[:port]`
        给定主机上指定的 TCP 端口，可以是 DNS 名称（如果构建时使用了 unbound 库）或 IP 地址。如果主机是 IPv6 地址，请将其括在方括号中，例如 `tcp:[::1]:6640`。

        如果未指定端口，默认为 6640。

    *   `pssl:[port][:host]`
        侦听指定 TCP 端口上的 SSL/TLS 连接。指定 0 作为端口以让内核自动选择可用端口。如果指定了主机（可以是 DNS 名称（如果构建时使用了 unbound 库）或 IP 地址），则连接仅限于解析的或指定的本地 IP 地址（IPv4 或 IPv6 地址）。如果主机是 IPv6 地址，请将其括在方括号中，例如 `pssl:6640:[::1]`。如果未指定主机，则仅侦听 IPv4（但不侦听 IPv6）地址。使用此形式时必须提供有效的 SSL/TLS 配置，可以通过命令行选项或 SSL 表指定此配置。

        如果未指定端口，默认为 6640。

        SSL/TLS 支持是一个可选功能，并不总是作为 OVN 或 Open vSwitch 的一部分构建。

    *   `ptcp:[port][:host]`
        侦听指定 TCP 端口上的连接。指定 0 作为端口以让内核自动选择可用端口。如果指定了主机（可以是 DNS 名称（如果构建时使用了 unbound 库）或 IP 地址），则连接仅限于解析的或指定的本地 IP 地址（IPv4 或 IPv6 地址）。如果主机是 IPv6 地址，请将其括在方括号中，例如 `ptcp:6640:[::1]`。如果未指定主机，则仅侦听 IPv4 地址。

        如果未指定端口，默认为 6640。

    当配置了多个客户端时，目标值必须是唯一的。重复的目标值产生未定义的结果。

### 客户端故障检测和处理

-   `max_backoff`: 可选整数，至少 1,000
    连接尝试之间等待的最大毫秒数。默认值取决于具体实现。

-   `inactivity_probe`: 可选整数
    向客户端发送非活动探测消息之前连接空闲时间的最大毫秒数。如果 Open vSwitch 在指定的秒数内未与客户端通信，它将发送探测。如果在相同的额外时间内未收到响应，Open vSwitch 假定连接已断开并尝试重新连接。默认值取决于具体实现。值 0 禁用非活动探测。

### 状态

-   `is_connected`: 布尔值
    如果当前已连接到此客户端，则为 true，否则为 false。

-   `status : last_error`: 可选字符串
    连接到管理器时发生的最后错误的易读描述；即 `strerror(errno)`。仅当发生错误时此键才存在。

-   `status : state`: 可选字符串，`ACTIVE`, `BACKOFF`, `CONNECTING`, `IDLE`, 或 `VOID` 之一
    到管理器的连接状态：

    *   `VOID`: 连接已禁用。
    *   `BACKOFF`: 尝试以增加的时间段重新连接。
    *   `CONNECTING`: 正在尝试连接。
    *   `ACTIVE`: 已连接，远程主机响应。
    *   `IDLE`: 连接空闲。等待保活响应。

    这些值将来可能会更改。它们仅供人类使用。

-   `status : sec_since_connect`: 可选字符串，包含整数，至少 0
    自此客户端上次成功连接到数据库以来的时间（秒）。如果客户端从未成功连接，则值为空。

-   `status : sec_since_disconnect`: 可选字符串，包含整数，至少 0
    自此客户端上次从数据库断开连接以来的时间（秒）。如果客户端从未断开连接，则值为空。

-   `status : locks_held`: 可选字符串
    连接持有的 OVSDB 锁名称的空格分隔列表。如果连接未持有任何锁，则省略。

-   `status : locks_waiting`: 可选字符串
    连接当前正在等待获取的 OVSDB 锁名称的空格分隔列表。如果连接未等待任何锁，则省略。

-   `status : locks_lost`: 可选字符串
    被另一个 OVSDB 客户端从连接中窃取的 OVSDB 锁名称的空格分隔列表。如果没有从该连接窃取锁，则省略。

-   `status : n_connections`: 可选字符串，包含整数，至少 2
    当目标指定侦听入站连接的连接方法（例如 `ptcp:` 或 `pssl:`）并且实际上有多个连接处于活动状态时，该值是活动连接数。否则，省略此键值对。

-   `status : bound_port`: 可选字符串，包含整数
    当目标是 `ptcp:` 或 `pssl:` 时，这是 OVSDB 服务器正在侦听的 TCP 端口。（这在目标指定端口 0 以允许内核选择任何可用端口时特别有用。）

### 公共列

-   `external_ids`: 字符串对映射
-   `other_config`: 字符串对映射

---

# DNS 表

此表中的每一行存储 DNS 记录。`Logical_Switch` 表的 `dns_records` 引用这些记录。

## 摘要

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `records` | 字符串对映射 | DNS 记录。 |
| `options : ovn-owned` | 可选字符串 | OVN 拥有。 |
| `external_ids` | 字符串对映射 | 外部 ID。 |

## 详细信息

-   `records`: 字符串对映射
    DNS 记录的键值对，其中 DNS 查询名称作为键，IP 地址字符串（以逗号或空格分隔）作为值。对于 PTR 请求，键值对可以是 `Reverse IPv4 address.in-addr.arpa` 和值 DNS 域名。对于 IPv6 地址，键必须是 `Reverse IPv6 address.ip6.arpa`。

    示例：`"vm1.ovn.org" = "10.0.0.4 aef0::4"`

    示例：`"4.0.0.10.in-addr.arpa" = "vm1.ovn.org"`

-   `options : ovn-owned`: 可选字符串
    如果设置为 true，则 OVN 将主要负责此行中的 DNS 记录。

    可以将此选项设置为 true 的 DNS 行用于用户需要在本地配置且不关心 IPv6 仅对 IPv4 感兴趣（或反之）的域。这将允许 ovn 发送 IPv4 DNS 回复并拒绝/忽略 IPv6 查询，以节省等待这些不感兴趣查询超时的时间。

-   `external_ids`: 字符串对映射
    参见本文件开头的外部 ID。

---

# SSL 表

用于 ovn-nb 数据库访问的 SSL/TLS 配置。

## 摘要

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `private_key` | 字符串 | 私钥。 |
| `certificate` | 字符串 | 证书。 |
| `ca_cert` | 字符串 | CA 证书。 |
| `bootstrap_ca_cert` | 布尔值 | 引导 CA 证书。 |
| `ssl_protocols` | 字符串 | SSL 协议。 |
| `ssl_ciphers` | 字符串 | SSL 密码。 |
| `ssl_ciphersuites` | 字符串 | SSL 密码套件。 |

**公共列：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `external_ids` | 字符串对映射 | 外部 ID。 |

## 详细信息

-   `private_key`: 字符串
    包含用作交换机对控制器的 SSL/TLS 连接身份的私钥的 PEM 文件名。

-   `certificate`: 字符串
    包含证书的 PEM 文件名，该证书由控制器和管理器使用的证书颁发机构 (CA) 签名，证明交换机的私钥，标识可信交换机。

-   `ca_cert`: 字符串
    包含 CA 证书的 PEM 文件名，用于验证交换机是否连接到可信控制器。

-   `bootstrap_ca_cert`: 布尔值
    如果设置为 true，则 Open vSwitch 将尝试在其第一个 SSL/TLS 连接上从控制器获取 CA 证书并将其保存到命名的 PEM 文件中。如果成功，它将立即断开连接并重新连接，从那时起，所有 SSL/TLS 连接都必须由由此获得的 CA 证书签名的证书进行身份验证。此选项将 SSL/TLS 连接暴露给获取初始 CA 证书的中间人攻击。它对于引导可能仍然有用。

-   `ssl_protocols`: 字符串
    要为 SSL/TLS 连接启用的 SSL/TLS 协议的范围或逗号或空格分隔的列表。

    支持的协议包括 TLSv1.2 和 TLSv1.3。范围可以以用破折号分隔的两个协议名称的形式提供 (TLSv1.2-TLSv1.3)，或者作为带有加号的单个协议名称 (TLSv1.2+)。该值可以是协议列表或恰好一个范围。范围是指定协议的首选方式，并且配置始终表现为好像提供了指定的最小和最大版本之间的范围，即，如果该值设置为 TLSv1.X,TLSv1.(X+2)，则 TLSv1.(X+1) 也将被启用，就像它是一个范围一样。无论顺序如何，建立连接时都会选择双方都支持的最高协议。

    当省略此选项时，默认为 TLSv1.2+。

-   `ssl_ciphers`: 字符串
    要为使用 TLSv1.2 的 SSL/TLS 连接支持的密码列表（采用 OpenSSL 密码字符串格式）。当省略此选项时，默认为 DEFAULT:@SECLEVEL=2。

-   `ssl_ciphersuites`: 字符串
    要为使用 TLSv1.3 及更高版本的 SSL/TLS 连接支持的密码套件列表（采用 OpenSSL 密码套件字符串格式）。当省略此选项时，将使用 OpenSSL 的默认值。

### 公共列

-   `external_ids`: 字符串对映射
    参见本文件开头的外部 ID。
