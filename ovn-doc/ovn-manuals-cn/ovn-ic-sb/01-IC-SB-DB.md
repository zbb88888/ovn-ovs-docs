# OVN_IC_Southbound 数据库模式

此数据库保存用于互连不同 OVN 部署的配置和状态。数据库的内容由每个 OVN 部署中的 `ovn-ic` 程序填充和使用，不应由 CMS 或最终用户直接使用。

OVN 互连南向数据库由每个 OVN 部署中的 `ovn-ic` 程序共享。它包含来自所有相关 OVN 部署的互连信息，并用作每个 OVN 部署交换信息的中间存储。每个部署中的 `ovn-ic` 程序负责在此数据库与其自己的北向和南向数据库之间同步数据。

## 数据库结构

OVN 互连南向数据库包含具有不同属性的数据类，如下节所述。

### 可用区特定信息

这些表包含特定于可用区的对象。每个对象由一个可用区拥有和填充，并由其他可用区读取。

`Availability_Zone`, `Gateway`, `Encap` 和 `Port_Binding` 表是可用区特定的表。

### 全局信息

不属于任何特定可用区但对所有可用区通用的数据。

`Datapath_Binding` 表包含公共数据路径绑定信息。

### 公共列

此数据库中的每个表都包含一个名为 `external_ids` 的特殊列。此列在出现的每个位置都具有相同的形式和用途。

-   `external_ids`: 字符串对映射
    供 `ovn-ic` 使用的键值对。

## 表摘要

下表总结了 OVN_IC_Southbound 数据库中每个表的用途。每个表都在后面的页面上有更详细的描述。

| 表 | 用途 |
| :--- | :--- |
| `IC_SB_Global` | IC 南向配置 |
| `Availability_Zone` | 可用区信息 |
| `Gateway` | 互连网关信息 |
| `Encap` | 封装类型 |
| `Datapath_Binding` | 中转交换机数据路径绑定 |
| `Port_Binding` | 中转端口绑定 |
| `Route` | 路由 |
| `Connection` | OVSDB 客户端连接 |
| `SSL` | SSL 配置 |
| `Service_Monitor` | Service_Monitor 配置 |

---

# IC_SB_Global 表

互连南向配置。此表必须只有一行。

## 摘要

**状态：**

| 字段 | 类型 | 描述 |
| :--- | :--- | :--- |
| `nb_ic_cfg` | 整数 | 北向 IC 配置序列号。 |

**公共列：**

| 字段 | 类型 | 描述 |
| :--- | :--- | :--- |
| `external_ids` | 字符串对映射 | 外部 ID。 |
| `options` | 字符串对映射 | 选项。 |

**连接选项：**

| 字段 | 类型 | 描述 |
| :--- | :--- | :--- |
| `connections` | `Connection` 的集合 | 连接集合。 |
| `ssl` | 可选 `SSL` | SSL 配置。 |

## 详细信息

### 状态

此列允许客户端跟踪系统的整体配置状态。

-   `nb_ic_cfg`: 整数
    配置的序列号。当 CMS 或 `ovn-ic-nbctl` 更新互连北向数据库时，它会递增互连北向数据库中 `NB_IC_Global` 表中的 `nb_ic_cfg` 列。当 OVN-IC 更新南向数据库以使其与这些更改保持同步时，一个 OVN-IC 会将此列更新为相同的值。

### 公共列

-   `external_ids`: 字符串对映射
    参见本文件开头的外部 ID。

-   `options`: 字符串对映射

### 连接选项

-   `connections`: `Connection` 的集合
    Open vSwitch 数据库服务器应连接到或应侦听的数据库客户端，以及应如何配置这些连接的选项。有关更多信息，请参阅 `Connection` 表。

-   `ssl`: 可选 `SSL`
    全局 SSL/TLS 配置。

---

# Availability_Zone 表

此表中的每一行代表一个可用区。从 OVN 控制平面的角度来看，每个 OVN 部署都被视为一个可用区，具有自己的中央组件，例如北向和南向数据库以及 `ovn-northd` 守护进程。

## 摘要

| 字段 | 类型 | 描述 |
| :--- | :--- | :--- |
| `name` | 字符串（表内必须唯一） | 名称。 |
| `nb_ic_cfg` | 整数 | 北向 IC 配置序列号。 |

## 详细信息

-   `name`: 字符串（表内必须唯一）
    唯一标识可用区的名称。

-   `nb_ic_cfg`: 整数
    此列由 OVN-IC 用于通知此 IC 实例已与 INB 中的更改保持一致。

---

# Gateway 表

此表中的每一行代表可用区中的一个互连网关 Chassis。

## 摘要

| 字段 | 类型 | 描述 |
| :--- | :--- | :--- |
| `name` | 字符串（表内必须唯一） | 名称。 |
| `availability_zone` | `Availability_Zone` | 可用区。 |
| `hostname` | 字符串 | 主机名。 |

**公共列：**

| 字段 | 类型 | 描述 |
| :--- | :--- | :--- |
| `external_ids` | 字符串对映射 | 外部 ID。 |

**封装配置：**

| 字段 | 类型 | 描述 |
| :--- | :--- | :--- |
| `encaps` | 1 个或多个 `Encap` 的集合 | 封装集合。 |

## 详细信息

-   `name`: 字符串（表内必须唯一）
    网关的名称。参见 OVN 南向数据库的 `Chassis` 表的 name 列。

-   `availability_zone`: `Availability_Zone`
    网关所属的可用区。

-   `hostname`: 字符串
    网关的主机名。

### 公共列

这些列的总体用途在本文件开头的“公共列”下进行了描述。

-   `external_ids`: 字符串对映射

### 封装配置

OVN 使用封装在网关之间传输逻辑数据平面数据包。

-   `encaps`: 1 个或多个 `Encap` 的集合
    指向支持的封装配置，以向此网关传输逻辑数据平面数据包。每个条目都是描述配置的 `Encap` 记录。参见 OVN 南向数据库的 `Chassis` 表的 encaps 列。

---

# Encap 表

`Gateway` 表中的 encaps 列引用此表中的行，以标识 OVN 如何向此网关传输逻辑数据平面数据包。

## 摘要

| 字段 | 类型 | 描述 |
| :--- | :--- | :--- |
| `type` | 字符串，`geneve` 或 `vxlan` | 类型。 |
| `options` | 字符串对映射 | 选项。 |
| `ip` | 字符串 | IP 地址。 |
| `gateway_name` | 字符串 | 网关名称。 |

## 详细信息

-   `type`: 字符串，`geneve` 或 `vxlan`
    用于向此网关传输数据包的封装。参见 OVN 南向数据库的 `Encap` 表的 type 列。

-   `options`: 字符串对映射
    用于配置封装的选项，可能是特定于类型的。参见 OVN 南向数据库的 `Encap` 表的 options 列。

-   `ip`: 字符串
    封装隧道端点的 IPv4 地址。

-   `gateway_name`: 字符串
    创建此 encap 的网关的名称。

---

# Datapath_Binding 表

此表中的每一行代表在 OVN 互连北向数据库的 `Transit_Switch` 表中配置的中转逻辑交换机的逻辑数据路径。

## 摘要

| 字段 | 类型 | 描述 |
| :--- | :--- | :--- |
| `transit_switch` | 字符串 | 中转交换机名称。 |
| `type` | 可选字符串，`transit-router` 或 `transit-switch` | 类型。 |
| `nb_ic_uuid` | 可选 uuid | 北向 IC UUID。 |
| `tunnel_key` | 1 到 16,777,215 的整数 | 隧道密钥。 |

**公共列：**

| 字段 | 类型 | 描述 |
| :--- | :--- | :--- |
| `external_ids` | 字符串对映射 | 外部 ID。 |

## 详细信息

-   `transit_switch`: 字符串
    在 OVN 互连北向数据库的 `Transit_Switch` 或 `Transit_Router` 表中配置的中转逻辑交换机或路由器的名称。注意：由于向后兼容性，中转路由器的名称存储在同一列中。

-   `type`: 可选字符串，`transit-router` 或 `transit-switch`
    可以是 `transit-switch` 或 `transit-router` 之一。

-   `nb_ic_uuid`: 可选 uuid
    这是由此南向 `Datapath_Binding` 表示的相应北向 IC 逻辑数据路径的 UUID。

-   `tunnel_key`: 1 到 16,777,215 的整数
    逻辑数据路径绑定到的隧道密钥值。密钥可以由任何 `ovn-ic` 生成，但所有可用区共享相同的密钥，以便逻辑数据路径可以跨它们对等。中转交换机数据路径绑定的隧道密钥必须是全局唯一的。

    有关隧道密钥含义的更多信息，请参阅 OVN 南向数据库的 `Datapath_Binding` 表的 tunnel_key 列。

### 公共列

这些列的总体用途在本文件开头的“公共列”下进行了描述。

-   `external_ids`: 字符串对映射

---

# Port_Binding 表

此表中的每一行将中转交换机上的逻辑端口绑定到物理网关和隧道密钥。中转交换机上的每个端口属于特定的可用区。

## 摘要

**核心功能：**

| 字段 | 类型 | 描述 |
| :--- | :--- | :--- |
| `transit_switch` | 字符串 | 中转交换机名称。 |
| `logical_port` | 字符串（表内必须唯一） | 逻辑端口。 |
| `availability_zone` | `Availability_Zone` | 可用区。 |
| `encap` | 可选 `Encap` 的弱引用 | 封装。 |
| `gateway` | 字符串 | 网关名称。 |
| `tunnel_key` | 1 到 32,767 的整数 | 隧道密钥。 |
| `address` | 字符串 | 地址。 |
| `type` | 可选字符串，`transit-router-port` 或 `transit-switch-port` | 类型。 |
| `nb_ic_uuid` | 可选 uuid | 北向 IC UUID。 |

**公共列：**

| 字段 | 类型 | 描述 |
| :--- | :--- | :--- |
| `external_ids` | 字符串对映射 | 外部 ID。 |

## 详细信息

### 核心功能

-   `transit_switch`: 字符串
    相应逻辑端口所属的中转交换机的名称。

-   `logical_port`: 字符串（表内必须唯一）
    逻辑端口，取自 OVN_Northbound 数据库的 `Logical_Switch_Port` 表中的名称。逻辑端口名称必须在所有可用区中唯一。

-   `availability_zone`: `Availability_Zone`
    端口所属的可用区。

-   `encap`: 可选 `Encap` 的弱引用
    指向支持的封装配置，以向此网关传输逻辑数据平面数据包。每个条目都是描述配置的 `Encap` 记录。

-   `gateway`: 字符串
    此端口物理所在的网关的名称。

-   `tunnel_key`: 1 到 32,767 的整数
    在隧道协议数据包内携带的密钥（例如 Geneve TLV）字段中表示逻辑端口的数字。密钥可以由任何 `ovn-ic` 生成，但所有可用区共享相同的密钥，以便数据包可以通过不同可用区的数据路径管道。

    隧道 ID 必须在逻辑数据路径的范围内唯一。

    有关隧道密钥的更多信息，请参阅 OVN 南向数据库的 `Port_Binding` 表的 tunnel_key 列。

-   `address`: 字符串
    与中转交换机端口对等的相应逻辑路由器端口使用的以太网地址和 IP 地址。它是 mac 列的值后跟 `Logical_Router_Port` 表中 networks 列的值组合而成的字符串。

-   `type`: 可选字符串，`transit-router-port` 或 `transit-switch-port`
    可以是 `switch-port` 或 `router-port` 之一。

-   `nb_ic_uuid`: 可选 uuid
    这是由此南向 `Datapath_Binding` 表示的相应北向 IC 逻辑数据路径的 UUID。

### 公共列

-   `external_ids`: 字符串对映射
    参见本文件开头的外部 ID。

---

# Route 表

此表中的每一行代表一条通告的路由。

## 摘要

**核心功能：**

| 字段 | 类型 | 描述 |
| :--- | :--- | :--- |
| `transit_switch` | 字符串 | 中转交换机名称。 |
| `availability_zone` | `Availability_Zone` | 可用区。 |
| `route_table` | 字符串 | 路由表。 |
| `ip_prefix` | 字符串 | IP 前缀。 |
| `nexthop` | 字符串 | 下一跳。 |
| `origin` | 字符串，`connected`, `loadbalancer`, 或 `static` 之一 | 来源。 |

**公共列：**

| 字段 | 类型 | 描述 |
| :--- | :--- | :--- |
| `external_ids` | 字符串对映射 | 外部 ID。 |

## 详细信息

### 核心功能

-   `transit_switch`: 字符串
    通告路由的中转交换机的名称。

-   `availability_zone`: `Availability_Zone`
    通告路由的可用区。

-   `route_table`: 字符串
    创建此路由的路由表。空值表示 <main> 路由表。

    直连网络的路由将被学习到 <main> 路由表，如果逻辑路由器有多个互连它们的中转交换机，则直连路由将通过每个中转交换机端口添加并配置为 ECMP 路由。

    路由表中的静态路由只有在互连中转交换机的 LRP 分别在 `options:route_table` 中具有与 NB `route_table` 或 ICSB `route_table` 值相同的值时才会被通告和学习。

-   `ip_prefix`: 字符串
    此路由的 IP 前缀（例如 192.168.100.0/24）。

-   `nexthop`: 字符串
    此路由的下一跳 IP 地址。

-   `origin`: 字符串，`connected`, `loadbalancer`, 或 `static` 之一
    可以是 `connected`、`static` 或 `loadbalancer` 之一。到直连子网的路由 - LRP 的 CIDR 以 `origin` 为 `connected` 值插入到 OVN IC SB DB 中。静态路由以 `static` 值插入到 OVN IC SB DB 中。LB VIP 的路由以 `loadbalancer` 值插入到 OVN IC SB DB 中。接下来，当 `ovn-ic` 将路由学习到另一个 AZ NB DB 时，路由来源将同步到 `options:origin`。

### 公共列

-   `external_ids`: 字符串对映射
    参见本文件开头的外部 ID。

---

# Connection 表

Open vSwitch 数据库 (OVSDB) 客户端的数据库连接配置。

此表主要配置 Open vSwitch 数据库服务器 (ovsdb-server)。

Open vSwitch 数据库服务器可以发起并维护到远程客户端的主动连接。它还可以侦听数据库连接。

## 摘要

**核心功能：**

| 字段 | 类型 | 描述 |
| :--- | :--- | :--- |
| `target` | 字符串（表内必须唯一） | 目标。 |

**客户端故障检测和处理：**

| 字段 | 类型 | 描述 |
| :--- | :--- | :--- |
| `max_backoff` | 可选整数，至少 1,000 | 最大回退时间。 |
| `inactivity_probe` | 可选整数 | 非活动探测。 |

**状态：**

| 字段 | 类型 | 描述 |
| :--- | :--- | :--- |
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
| :--- | :--- | :--- |
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
        给定主机上指定的 TCP 端口，可以是 DNS 名称（如果构建时使用了 unbound 库）或 IP 地址（IPv4 或 IPv6）。如果主机是 IPv6 地址，请将其括在方括号中，例如 `tcp:[::1]:6640`。

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

这些列的总体用途在本文件开头的“公共列”下进行了描述。

-   `external_ids`: 字符串对映射

-   `other_config`: 字符串对映射

---

# SSL 表

用于 ovn-sb 数据库访问的 SSL/TLS 配置。

## 摘要

| 字段 | 类型 | 描述 |
| :--- | :--- | :--- |
| `private_key` | 字符串 | 私钥。 |
| `certificate` | 字符串 | 证书。 |
| `ca_cert` | 字符串 | CA 证书。 |
| `bootstrap_ca_cert` | 布尔值 | 引导 CA 证书。 |
| `ssl_protocols` | 字符串 | SSL 协议。 |
| `ssl_ciphers` | 字符串 | SSL 密码。 |
| `ssl_ciphersuites` | 字符串 | SSL 密码套件。 |

**公共列：**

| 字段 | 类型 | 描述 |
| :--- | :--- | :--- |
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

这些列的总体用途在本文件开头的“公共列”下进行了描述。

-   `external_ids`: 字符串对映射

---

# Service_Monitor 表

## 摘要

| 字段 | 类型 | 描述 |
| :--- | :--- | :--- |
| `type` | 可选字符串，必须为 `load-balancer` | 类型。 |
| `ip` | 字符串 | IP 地址。 |
| `protocol` | 可选字符串，`tcp` 或 `udp` | 协议。 |
| `port` | 0 到 65,535 的整数 | 端口。 |
| `logical_port` | 字符串 | 逻辑端口。 |
| `src_ip` | 字符串 | 源 IP 地址。 |
| `src_mac` | 字符串 | 源 MAC 地址。 |
| `status` | 可选字符串，`error`, `offline`, 或 `online` 之一 | 状态。 |
| `target_availability_zone` | 字符串 | 目标可用区。 |
| `source_availability_zone` | 字符串 | 源可用区。 |
| `options` | 字符串对映射 | 选项。 |
| `external_ids` | 字符串对映射 | 外部 ID。 |

## 详细信息

-   `type`: 可选字符串，必须为 `load-balancer`
    服务的类型。仅支持值“load-balancer”。

-   `ip`: 字符串
    要监控的服务的 IP。从 SBDB 记录复制。

-   `protocol`: 可选字符串，`tcp` 或 `udp`
    服务的协议。从源南向数据库记录复制。

-   `port`: 0 到 65,535 的整数
    服务的 TCP 或 UDP 端口。从源南向数据库记录复制。

-   `logical_port`: 字符串
    服务运行所在的逻辑端口的 VIF。绑定此 `logical_port` 的 `ovn-controller` 通过发送定期监控数据包来监控服务。从源南向数据库记录复制。

-   `src_ip`: 字符串
    在服务监控数据包中使用的源 IPv4 或 IPv6 地址。从源南向数据库记录复制。

-   `src_mac`: 字符串
    在服务监控数据包中使用的源以太网地址。从南向数据库记录复制。

-   `status`: 可选字符串，`error`, `offline`, 或 `online` 之一
    服务监控的健康状态，从目标南向数据库同步。

-   `target_availability_zone`: 字符串
    受监控服务端口所在的可用区。

-   `source_availability_zone`: 字符串
    在 ICSB 中发起此监控条目并保留生命周期管理所有权的可用区。

-   `options`: 字符串对映射
    与 SBDB 中的 `Service_Monitor` 表相同。从源 SBDB 记录复制。

-   `external_ids`: 字符串对映射
    与 SBDB 中的 `Service_Monitor` 表相同。从源 SBDB 记录复制。
