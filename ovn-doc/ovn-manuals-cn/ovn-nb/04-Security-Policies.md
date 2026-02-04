# ACL 表

此表中的每一行代表逻辑交换机或端口组的一个 ACL 规则，通过其 `acls` 列指向该规则。此表中最高优先级匹配行的 `action` 列决定了数据包的处理方式。如果没有行匹配，则默认允许数据包。（可以进行默认拒绝处理：添加一条优先级为 0、匹配为 1、动作为 deny 的规则。）

## 摘要

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `label` | 0 到 4,294,967,295 的整数 | 标签。 |
| `priority` | 0 到 32,767 的整数 | 优先级。 |
| `direction` | 字符串，`from-lport` 或 `to-lport` | 方向。 |
| `match` | 字符串 | 匹配条件。 |
| `action` | 字符串，`allow-related`, `allow-stateless`, `allow`, `drop`, `pass`, 或 `reject` 之一 | 动作。 |
| `tier` | 0 到 3 的整数 | 层级。 |
| `network_function_group` | 可选 `Network_Function_Group` | 网络功能组。 |

**选项：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `options : apply-after-lb` | 可选字符串 | 是否在 LB 之后应用。 |
| `options : persist-established` | 可选字符串 | 是否持久化已建立的连接。 |

**日志记录：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `log` | 布尔值 | 是否记录日志。 |
| `name` | 可选字符串，最多 63 个字符 | 日志名称。 |
| `severity` | 可选字符串，`alert`, `debug`, `info`, `notice`, 或 `warning` 之一 | 日志严重性。 |
| `meter` | 可选字符串 | 日志计量器。 |

**采样：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `sample_new` | 可选 `Sample` | 新会话采样。 |
| `sample_est` | 可选 `Sample` | 已建立会话采样。 |

**公共列：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `options` | 字符串对映射 | 通用选项。 |
| `external_ids` | 字符串对映射 | 外部 ID。 |

**ACL 配置选项：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `options : log-related` | 可选字符串 | 是否记录相关流量。 |

## 详细信息

-   `label`: 0 到 4,294,967,295 的整数
    将标识符与 ACL 关联。相同的值将写入相应的连接跟踪器条目。该值应该是一个有效的 32 位无符号整数。此值有助于从连接跟踪器端进行调试。例如，通过此“标签”，我们可以回溯导致“泄漏”连接的 ACL 规则。连接跟踪器条目仅为允许的连接创建，因此标签仅对 `allow` 和 `allow-related` 动作有效。

    注意：如果 ACL 同时启用了采样并关联了标签，则标签值将覆盖 `sample_new` 或 `sample_est` 配置中定义的观察点 ID。

-   `priority`: 0 到 32,767 的整数
    ACL 规则的优先级。数值较高的规则优先于数值较低的规则。如果具有相同优先级的两个 ACL 规则都匹配，则实际应用于数据包的规则是未定义的。

    来自 `allow-related` 流的返回流量总是允许的，不能通过 ACL 更改。

    `allow-stateless` 流总是优先于有状态 ACL，无论其优先级如何。（`allow` 和 `allow-related` ACL 都可以是有状态的。）

-   `direction`: 字符串，`from-lport` 或 `to-lport`
    此规则应适用的流量方向：

    *   `from-lport`: 用于对来自逻辑端口的流量实施过滤器。这些规则应用于逻辑交换机的入口管道。
    *   `to-lport`: 用于对转发到逻辑端口的流量实施过滤器。这些规则应用于逻辑交换机的出口管道。

-   `match`: 字符串
    ACL 应匹配的数据包，使用与 OVN 南向数据库 `Logical_Flow` 表中的 match 列相同的表达式语言。`outport` 逻辑端口仅在 `to-lport` 方向可用（`inport` 在两个方向都可用）。

    默认情况下允许所有流量。在编写更严格的策略时，重要的是要记住允许 ARP 和 IPv6 邻居发现数据包等流量。

    注意，不能创建匹配 `type=router` 或 `type=localnet` 端口的 ACL。

-   `action`: 字符串，`allow-related`, `allow-stateless`, `allow`, `drop`, `pass`, 或 `reject` 之一
    ACL 规则匹配时采取的动作：

    *   `allow-stateless`: 总是以无状态方式转发数据包，省略连接跟踪机制，而不管为交换机定义的其他规则如何。可能需要为入站回复定义额外的规则。例如，如果你定义了一条规则允许发往 IP 地址的传出 TCP 流量，那么你可能还需要定义另一条规则允许来自同一 IP 地址的传入 TCP 流量。此外，匹配无状态 ACL 的流量将绕过负载均衡器 DNAT/unDNAT 处理。如果流量应该被负载均衡，则应改用有状态 ACL。
    *   `allow`: 转发数据包。当逻辑交换机上存在 `allow-related` 规则时，它也会通过连接跟踪发送数据包。否则，它等同于 `allow-stateless`。
    *   `allow-related`: 转发数据包及相关流量（例如对出站连接的入站回复）。
    *   `drop`: 默默丢弃数据包。
    *   `reject`: 丢弃数据包，对于 TCP 回复 RST，对于其他基于 IPv4/IPv6 的协议回复 ICMPv4/ICMPv6 不可达消息。
    *   `pass`: 传递到下一个 ACL 层级。如果使用多个 ACL 层级，匹配此 ACL 将停止在当前层级评估 ACL 并移动到下一个层级。如果不使用 ACL 层级或在最后一层匹配了 pass ACL，则使用 `NB_Global` 表中的 `options:default_acl_drop` 选项来决定如何继续。

-   `tier`: 0 到 3 的整数
    此 ACL 所属的分层层级。

    ACL 可以分配给数字层级。在评估 ACL 时，使用内部计数器来确定应评估哪一层级的 ACL。首先评估第 0 层 ACL。如果无法确定结论，则接下来评估第 1 层 ACL。这持续直到达到最大层级值。如果评估了所有层级的 ACL 仍未得出结论，则使用 `NB_Global` 表中的 `options:default_acl_drop` 选项来决定如何继续。

    在此版本的 OVN 中，ACL 的最大层级值为 3，这意味着允许 4 个 ACL 层级 (0-3)。

-   `network_function_group`: 可选 `Network_Function_Group`
    匹配此 ACL 的流量被重定向到的网络功能组。

### 选项

ACL 选项。

-   `options : apply-after-lb`: 可选字符串
    如果设置为 true，ACL 将在负载均衡阶段之后应用。仅支持 `from-lport` 方向。

    此选项的主要用例是支持匹配负载均衡器后端 IP 的目标 IP 地址的 ACL。

    OVN 将分两个阶段应用 `from-lport` ACL。未设置此选项 `apply-after-lb` 的 ACL 将在负载均衡器阶段之前应用，设置了此选项的 ACL 将在负载均衡器阶段之后应用。这些阶段之间的优先级是独立的，对 CMS 来说可能并不明显。因此，CMS 在使用此选项时应格外小心，并应仔细评估所有 ACL 以及默认拒绝/允许 ACL（如果有）的优先级。

-   `options : persist-established`: 可选字符串
    此选项仅适用于动作为 `allow-related` 的 ACL。

    `allow-related` ACL 在数据包匹配 ACL 的 match 列时创建连接跟踪条目。通常，流量必须继续匹配这些条件才能继续被 ACL 允许。如果此选项设置为 true，则一旦发生原始匹配，ACL 匹配将被绕过。取而代之的是，使用连接跟踪条目中的标记位来允许流量。这意味着即使 ACL 的匹配发生变化且不再匹配已建立的流量，流量仍将继续被允许。

    如果此选项设置为 false，或者 ACL 的动作更改为除 `allow-related` 之外的其他内容，或者 ACL 被销毁，流量将自动停止被允许。

### 日志记录

这些列控制 OVN 是否以及如何记录匹配 ACL 的数据包。

-   `log`: 布尔值
    如果设置为 true，匹配 ACL 的数据包将在执行 ACL 处理的传输节点或节点上触发日志消息。日志记录可以与任何动作结合使用。

    如果设置为 false，此组中的其余列没有意义。

-   `name`: 可选字符串，最多 63 个字符
    如果提供了此名称，它将包含在日志记录中。它为管理员和云管理系统提供了一种将日志记录与特定 ACL 关联的方法。

-   `severity`: 可选字符串，`alert`, `debug`, `info`, `notice`, 或 `warning` 之一
    ACL 的严重性。严重性级别与 syslog 的级别匹配，按严重性递减顺序排列：`alert`, `warning`, `notice`, `info`, 或 `debug`。当该列为空时，默认为 `info`。

-   `meter`: 可选字符串
    用于限制 ACL 日志消息速率的计量器名称。该字符串必须匹配 `Meter` 表中一行的 name 列。默认情况下，日志消息不进行速率限制。为了确保同一个计量器分别限制多个 ACL 日志的速率，请设置 `fair` 列。

-   `sample_new`: 可选 `Sample`
    用于对此 ACL 匹配的新会话进行采样的 `Sample` 表条目。如果 ACL 是无状态的，则用于对 ACL 匹配的所有流量进行采样。

    注意：如果 ACL 同时启用了采样并关联了标签，则标签值将覆盖 `sample_new` 配置中定义的观察点 ID。

-   `sample_est`: 可选 `Sample`
    用于对此 ACL 匹配的已建立/相关会话进行采样的 `Sample` 表条目。

    注意：如果 ACL 同时启用了采样并关联了标签，则标签值将覆盖 `sample_est` 配置中定义的观察点 ID。

### 公共列

-   `options`: 字符串对映射
    此列提供通用键/值设置。下面单独描述支持的选项。

### ACL 配置选项

-   `options : log-related`: 可选字符串
    如果设置为 true，则记录从有状态 ACL 允许的回复或相关流量。为了使此选项起作用，必须将 `log` 选项设置为 true，并且必须设置一个标签，且该标签对 ACL 必须是唯一的。标签是必要的，因为它是将回复流量与其所属的 ACL 关联的唯一手段。它必须是唯一的，否则将无法确定匹配哪个 ACL。注意：如果启用了此选项，将安装额外的流以记录相关流量。因此，如果在所有 ACL 上启用此选项，则记录 ACL 流量所需的流总数将比未启用此选项时增加一倍。

-   `external_ids`: 字符串对映射
    参见本文件开头的外部 ID。

---

# QoS 表

此表中的每一行代表逻辑交换机的一个 QoS 规则，通过其 `qos_rules` 列指向该规则。支持两种类型的 QoS：DSCP 标记和计量。具有最高优先级的匹配将应用 QoS。如果指定了 `action` 列，则匹配的数据包将应用 DSCP 标记。如果指定了 `bandwidth` 列，则匹配的数据包将应用计量。`action` 和 `bandwidth` 不是互斥的，因此可以为同一个 QoS 条目定义标记和计量。如果没有行匹配，数据包将不应用任何 QoS。

## 摘要

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `priority` | 0 到 32,767 的整数 | 优先级。 |
| `direction` | 字符串，`from-lport` 或 `to-lport` | 方向。 |
| `match` | 字符串 | 匹配条件。 |
| `action` | 字符串-整数对映射 | DSCP 或标记动作。 |
| `bandwidth` | 字符串-整数对映射 | 带宽限制。 |
| `external_ids` | 字符串对映射 | 外部 ID。 |

## 详细信息

-   `priority`: 0 到 32,767 的整数
    QoS 规则的优先级。数值较高的规则优先于数值较低的规则。如果具有相同优先级的两个 QoS 规则都匹配，则实际应用于数据包的规则是未定义的。

-   `direction`: 字符串，`from-lport` 或 `to-lport`
    此字段的值类似于 OVN 北向数据库 `ACL` 表中的 `direction` 列。

-   `match`: 字符串
    QoS 规则应匹配的数据包，使用与 OVN 南向数据库 `Logical_Flow` 表中的 match 列相同的表达式语言。`outport` 逻辑端口仅在 `to-lport` 方向可用（`inport` 在两个方向都可用）。

-   `action`: 字符串-整数对映射，键为 `dscp` 或 `mark`，值在 0 到 4,294,967,295 范围内
    当指定 `dscp` 动作时，匹配的流将应用 DSCP 标记。当指定 `mark` 动作时，匹配的流将应用数据包标记。

    *   `dscp`: 此动作的值应在 0 到 63（含）范围内。
    *   `mark`: 此动作的值应为正整数。

-   `bandwidth`: 字符串-整数对映射，键为 `burst` 或 `rate`，值在 1 到 4,294,967,295 范围内
    当指定时，匹配的数据包将应用带宽计量。超过限制的流量将被丢弃。

    *   `rate`: 速率限制值，单位为 kbps。
    *   `burst`: 突发速率限制值，单位为千比特。这是可选的，需要指定速率。

-   `external_ids`: 字符串对映射
    参见本文件开头的外部 ID。

---

# Mirror 表

此表中的每一行代表一个可用于端口镜像的镜像。`Logical_Switch_Port` 表中的 `mirror_rules` 列引用这些镜像。

## 摘要

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `name` | 字符串（表内必须唯一） | 镜像名称。 |
| `filter` | 字符串，`both`, `from-lport`, 或 `to-lport` 之一 | 过滤器。 |
| `sink` | 字符串 | 接收器。 |
| `type` | 字符串，`erspan`, `gre`, `local`, 或 `lport` 之一 | 类型。 |
| `index` | 整数 | 索引。 |
| `mirror_rules` | `Mirror_Rule` 的集合 | 镜像规则。 |
| `external_ids` | 字符串对映射 | 外部 ID。 |

## 详细信息

-   `name`: 字符串（表内必须唯一）
    代表镜像的名称。

-   `filter`: 字符串，`both`, `from-lport`, 或 `to-lport` 之一
    此字段的值代表镜像的选择标准。`to-lport` 镜像进入逻辑端口的数据包。`from-lport` 镜像离开逻辑端口的数据包。`both` 镜像两个方向的数据包。

-   `sink`: 字符串
    此字段的值代表镜像的目的地/接收器。如果类型是 `gre` 或 `erspan`，该值指示隧道远程 IP（IPv4 或 IPv6）。对于 `local` 类型，此字段定义 OVS 集成网桥上的本地接口，用作镜像目的地。该接口必须拥有匹配此字符串的 `external-ids:mirror-id`。

-   `type`: 字符串，`erspan`, `gre`, `local`, 或 `lport` 之一
    此字段的值指定镜像类型 - `gre`, `erspan`, `local` 或 `lport`。

-   `index`: 整数
    此字段的值代表隧道 ID。如果配置的隧道类型是 `gre`，此字段代表 GRE 密钥值；如果配置的隧道类型是 `erspan`，它代表 `erspan_idx` 值。如果类型是 `local`，则忽略此字段。

-   `mirror_rules`: `Mirror_Rule` 的集合
    此字段的值代表用于过滤镜像流量的镜像规则。

-   `external_ids`: 字符串对映射
    参见本文件开头的外部 ID。

---

# Mirror_Rule 表

此表中的每一行代表一个镜像规则，可用于过滤 lport 镜像流量。

## 摘要

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `match` | 字符串 | 匹配条件。 |
| `action` | 字符串，`mirror` 或 `skip` | 动作。 |
| `priority` | 0 到 32,767 的整数 | 优先级。 |

## 详细信息

-   `match`: 字符串
    匹配表达式，描述应对哪些数据包执行镜像规则动作。使用 `Logical_Flow` 表达式语言。

-   `action`: 字符串，`mirror` 或 `skip`
    镜像规则匹配时采取的动作：

    *   `mirror`: 镜像匹配的数据包。
    *   `skip`: 不镜像匹配的数据包。

-   `priority`: 0 到 32,767 的整数
    镜像规则优先级。优先级较高的规则优先于优先级较低的规则。规则由优先级和匹配字符串唯一标识。

---

# Meter 表

此表中的每一行代表一个可用于 QoS 或速率限制的计量器。

## 摘要

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `name` | 字符串（表内必须唯一） | 计量器名称。 |
| `unit` | 字符串，`kbps` 或 `pktps` | 单位。 |
| `bands` | 1 个或多个 `Meter_Band` 的集合 | 频带。 |
| `fair` | 可选布尔值 | 是否公平。 |
| `external_ids` | 字符串对映射 | 外部 ID。 |

## 详细信息

-   `name`: 字符串（表内必须唯一）
    此计量器的名称。

    以 "__"（两个下划线）开头的名称保留供 OVN 内部使用，不应手动添加。

-   `unit`: 字符串，`kbps` 或 `pktps`
    频带条目中 `rate` 和 `burst_rate` 参数的单位。`kbps` 指定每秒千比特，`pktps` 指定每秒数据包数。

-   `bands`: 1 个或多个 `Meter_Band` 的集合
    与此计量器关联的频带。每个频带指定一个速率，超过该速率时将采取动作。如果超过了多个频带的速率，则选择超过的频带中速率最高的频带。

-   `fair`: 可选布尔值
    此列用于进一步描述当有多个引用指向同一个计量器时的期望行为。如果此列为空或设置为 false，则速率将在所有引用相同计量器名称的行之间共享。相反，当此列设置为 true 时，同一计量器的每个用户将单独进行速率限制。

-   `external_ids`: 字符串对映射
    参见本文件开头的外部 ID。

---

# Meter_Band 表

此表中的每一行代表一个计量器频带，指定超过该速率时应应用配置的动作。`Meter` 表中的 `bands` 列引用这些频带。

## 摘要

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `action` | 字符串，必须为 `drop` | 动作。 |
| `rate` | 1 到 4,294,967,295 的整数 | 速率。 |
| `burst_size` | 0 到 4,294,967,295 的整数 | 突发大小。 |
| `external_ids` | 字符串对映射 | 外部 ID。 |

## 详细信息

-   `action`: 字符串，必须为 `drop`
    此频带匹配时执行的动作。唯一支持的动作是 `drop`。

-   `rate`: 1 到 4,294,967,295 的整数
    此频带的速率限制，单位为每秒千比特或每秒比特，取决于父 `Meter` 条目的 `unit` 列指定的是 `kbps` 还是 `pktps`。

-   `burst_size`: 0 到 4,294,967,295 的整数
    此频带允许的最大突发，单位为千比特或数据包，取决于父 `Meter` 条目的 `unit` 列中选择的是 `kbps` 还是 `pktps`。如果大小为零，交换机可以根据其配置自由选择一些合理的值。

-   `external_ids`: 字符串对映射
    参见本文件开头的外部 ID。
