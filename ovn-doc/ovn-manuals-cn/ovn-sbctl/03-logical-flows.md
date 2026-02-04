# OVN 逻辑流命令 (Logical Flow Commands)

*   `[--uuid] [--ovs[=remote]] [--stats] [--vflows] lflow-list [logical-datapath] [lflow...]`
    列出逻辑流 (Logical Flows)。
    *   `logical-datapath`: 如果指定，仅列出该逻辑数据路径的流。可以是 UUID 或数据路径名称。
    *   `lflow`: 如果指定，仅列出匹配的逻辑流。可以是 UUID 或 UUID 的前几个字符（可选前缀 `0x`）。
    *   `--uuid`: 输出包含每个逻辑流 UUID 的前 32 位。这有助于查找与给定逻辑流对应的 OpenFlow 流（OpenFlow cookie 通常是逻辑流 UUID 的前 32 位）。
    *   `--ovs`: 尝试获取并显示与每个 OVN 逻辑流对应的 OpenFlow 流。
        *   `ovn-sbctl` 通过 OpenFlow 连接到 `remote`（默认 `unix:/br-int.mgmt`）并检索流。
    *   `--stats`: 包含所有 OpenFlow 信息，如数据包和字节计数器、持续时间等。
    *   `--vflows`: 列出直接用于生成 OpenFlow 流的其他 Southbound 数据库记录（如 `port-bindings`, `mac-bindings`, `multicast-groups`, `chassis`）。

*   `[--uuid] dump-flows [logical-datapath]`
    `lflow-list` 的别名。

*   `count-flows [logical-datapath]`
    打印每个表和每个数据路径的逻辑流数量。
