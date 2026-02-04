# OVN 逻辑网络追踪工具 (ovn-trace) 概览与选项

## 简介 (Synopsis)

```bash
ovn-trace [options] [datapath] microflow
ovn-trace [options] --detach
```

## 描述 (Description)

此工具模拟 OVN 逻辑网络内的数据包转发。它可用于运行“假设”场景：如果一个数据包源自某个逻辑端口，它会发生什么，最终会到达哪里？熟悉 Open vSwitch `ofproto/trace` 命令的用户会发现 `ovn-trace` 是用于逻辑网络的类似工具。

`ovn-trace` 通过读取 OVN Southbound 数据库中的 `Logical_Flow` 和其他表来工作。它通过在逻辑流表中重复查找数据包，遵循可能性的整个树状结构，来模拟数据包通过逻辑网络的路径。

`ovn-trace` 仅模拟 OVN 逻辑网络。它不模拟逻辑网络分层之上的物理元素。这意味着，例如，VM 如何分布在 Hypervisor 之间，或者它们的 Hypervisor 是否正常工作且可达，都不重要，`ovn-trace` 将产生相同的结果。有一个重要的例外：`ovn-northd`（生成 `ovn-trace` 模拟的逻辑流的守护进程）根据逻辑端口是 Up 还是 Down 来不同地对待它们。因此，如果您看到令人惊讶的结果，请确保模拟中涉及的端口是 Up 的。

使用 `ovn-trace` 最简单的方法是在命令行上提供 `microflow`（以及可选的 `datapath`）参数。在这种情况下，它模拟单个数据包的行为并退出。

*   `datapath` (可选): 指定逻辑数据路径的名称。可接受的名称包括 Northbound `Logical_Switch` 或 `Logical_Router` 表中的名称、这些表中记录的 UUID，或 Southbound `Datapath_Binding` 表中记录的 UUID。（数据路径是可选的，因为 `ovn-trace` 可以从微流匹配的 `inport` 推断出来。）

*   `microflow`: 描述要模拟转发的数据包，使用 `ovn-sb(5)` 中描述的 OVN 逻辑表达式语法。
    *   L2 示例: `inport == "lp11" && eth.src == 00:01:02:03:04:05 && eth.dst == ff:ff:ff:ff:ff:ff`
    *   L3 示例: `inport == "lp111" && eth.src == ... && eth.dst == ... && ip4.src == 192.168.11.1 && ip4.dst == 192.168.22.2 && ip.ttl == 64`
    *   ARP 示例: `inport == "lp123" && eth.dst == ff:ff:ff:ff:ff:ff ... && arp.op == 1 ...`

## 选项 (Options)

### 追踪选项 (Trace Options)

*   `--detailed`
    (默认) 详细输出格式。

*   `--summary`
    摘要输出格式。

*   `--minimal`
    最小输出格式。

*   `--all`
    选择所有三种输出格式。

*   `--ovs[=remote]`
    使 `ovn-trace` 尝试获取并显示与每个 OVN 逻辑流对应的 OpenFlow 流。
    *   为此，`ovn-trace` 通过 OpenFlow 连接到 `remote`（默认 `unix:/br-int.mgmt`）并检索流。
    *   注意：有些逻辑流可能不对应于给定 Hypervisor 上的 OpenFlow 流，或者对应多个流。

*   `--ct=flags`
    设置 `ct_next` 逻辑动作将报告的 `ct_state` 标志。
    *   标志: `trk`, `new`, `est`, `rel`, `rpl`, `inv`, `dnat`, `snat`。
    *   默认: `trk,est`。
    *   可以指定多个 `--ct` 选项来为多个 `ct_next` 动作指定标志。

*   `--lb-dst=ip[:port]`
    设置从 VIP 池中使用的目标 IP。在守护进程模式下不可用。

*   `--select-id=id`
    指定 `select` 动作选择的 ID。在守护进程模式下不可用。

*   `--friendly-names` / `--no-friendly-names`
    默认情况下，`ovn-trace` 会将 UUID 等长名称替换为更友好的名称（如端口名）。使用 `--no-friendly-names` 禁用此行为。

### 守护进程选项 (Daemon Options)

*   `--pidfile`, `--overwrite-pidfile`, `--detach`, `--monitor`, `--no-chdir`, `--no-self-confinement`, `--user`。
    (与其他 OVN 工具相同)

### 日志选项 (Logging Options)

*   `-v`, `--verbose`, `--log-file`, `--syslog-target`, `--syslog-method`。

### PKI 选项 (PKI Options)

*   `-p`, `--private-key`
*   `-c`, `--certificate`
*   `-C`, `--ca-cert`
*   `--ssl-server-name`

### 其他选项 (Other Options)

*   `--db database`
    要连接的 OVSDB 数据库远程地址。默认 `unix:/db.sock`。
