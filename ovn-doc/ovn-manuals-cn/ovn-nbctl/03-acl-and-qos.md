# OVN ACL 与 QoS 命令

## ACL 命令 (ACL Commands)

这些命令对给定实体的 ACL 对象进行操作。实体可以是逻辑交换机或端口组。实体可以指定为 UUID 或名称。如果逻辑交换机和端口组存在相同的名称，可以使用 `--type` 选项指定实体的类型。`type` 必须是 `switch` 或 `port-group`。

*   `[--type={switch | port-group}] [--log] [--meter=meter] [--severity=severity] [--name=name] [--label=label] [--sample-new=sample] [--sample-est=sample] [--may-exist] [--apply-after-lb] [--tier] acl-add entity direction priority match verdict [network-function-group]`
    向 `entity` 添加指定的 ACL。
    *   `direction`: 必须是 `from-lport` 或 `to-lport`。
    *   `priority`: 必须在 0 到 32767 之间（含）。
    *   `match`: 匹配条件。
    *   `verdict`: 动作（如 `allow`, `drop`, `reject`）。
    *   `--may-exist`: 如果指定，添加重复的 ACL 会成功但不会真正创建新的 ACL。
    *   `--log`: 启用 ACL 的数据包日志记录。
    *   `--severity`: 指定日志条目的严重性（`alert`, `warning`, `notice`, `info`, `debug`，默认为 `info`）。
    *   `--name`: 为日志条目指定名称。
    *   `--meter`: 用于对数据包日志记录进行速率限制。`meter` 参数命名由 `meter-add` 配置的计量器。
    *   `--sample-new` / `--sample-est`: 启用 ACL 采样。必须提供 Sample 表行的有效 UUID。
    *   `--apply-after-lb`: 设置 ACL 表中的 `options:apply-after-lb=true`。ACL 将在逻辑交换机负载均衡阶段之后应用。
    *   `--tier`: 设置 ACL 的层级 (tier)。

*   `[--type={switch | port-group}] [--tier] acl-del entity [direction [priority match]]`
    从 `entity` 中删除 ACL。
    *   如果仅提供 `entity`，则删除该实体的所有 ACL。
    *   如果还指定了 `direction`，则删除该方向的所有流。
    *   如果给出了所有字段，则删除匹配所有字段的单个流。
    *   如果提供了 `--tier` 选项，则除了其他条件外，仅删除给定层级值的 ACL。

*   `[--type={switch | port-group}] [--all] acl-list entity`
    列出 `entity` 上的 ACL。
    *   `--all`: 当提供逻辑交换机作为参数时，还会列出与每个逻辑交换机端口关联的端口组 ACL。

## 网络功能命令 (Network Function Commands)

*   `[--may-exist] nf-add nf inport outport`
    创建一个名为 `nf` 的新网络功能，包含逻辑交换机端口 `inport` 和 `outport`。
    *   两个端口必须在同一个逻辑交换机上且已创建。
    *   当用于 ACL 动作时，匹配 ACL 的流量如果是 `from-lport` ACL 则重定向到 `inport`，如果是 `to-lport` ACL 则重定向到 `outport`。响应数据包按相反顺序通过相同的端口发送。

*   `[--if-exists] nf-del nf`
    删除指定为名称或 UUID 的 `nf`。

*   `nf-list`
    列出所有网络功能。

*   `[--may-exist] nfg-add nfg id mode [nf]...`
    创建一个名为 `nfg` 的新网络功能组，可选择添加一个或多个 `nf` 到组中。
    *   `id`: 必须在 1 到 255 之间唯一。
    *   `mode`: 必须是 `inline`（目前唯一支持的模式）。
    *   流量重定向将基于健康监控指向其中一个活动的网络功能。

*   `[--if-exists] nfg-del nfg`
    删除 `nfg`。

*   `nfg-list`
    列出所有网络功能组。

*   `[--may-exist] nfg-add-nf nfg nf`
    将名为 `nf` 的网络功能添加到名为 `nfg` 的网络功能组。

*   `nfg-del-nf nfg nf`
    从名为 `nfg` 的网络功能组中删除名为 `nf` 的网络功能。

## 逻辑交换机 QoS 规则命令 (Logical Switch QoS Rule Commands)

*   `[--may-exist] qos-add switch direction priority match [mark=mark] [dscp=dscp] [rate=rate [burst=burst]]`
    向 `switch` 添加 QoS 标记和计量规则。
    *   `direction`: `from-lport` 或 `to-lport`。
    *   `priority`: 0 到 32767。
    *   `dscp=dscp`: 应用 DSCP 标记 (0-63)。
    *   `rate=rate`: 应用速率限制 (kbps)。
    *   `burst=burst`: 突发速率限制 (kilobits)。
    *   `mark=mark`: 应用数据包标记 (`pkt.mark`)。
    *   `dscp` 和/或 `rate` 是必需参数。

*   `qos-del switch [direction [priority match]]`
    从 `switch` 中删除 QoS 规则。
    *   如果提供了 `switch` 和 UUID，则删除具有指定 UUID 的 QoS 规则。

*   `qos-list switch`
    列出 `switch` 上的 QoS 规则。

## 计量器命令 (Meter Commands)

*   `meter-add name action rate unit [burst]`
    添加指定的计量器。
    *   `name`: 计量器的唯一名称。以 `__` 开头的名称保留供 OVN 内部使用。
    *   `action`: 超出计量时的动作，唯一支持的是 `drop`。
    *   `rate`: 速率。
    *   `unit`: 单位，`kbps` (千比特每秒) 或 `pktps` (包每秒)。
    *   `burst`: 最大突发量。

*   `meter-del [name]`
    删除计量器。默认删除所有。

*   `meter-list`
    列出所有计量器。
