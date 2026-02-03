# 逻辑流水线 (Logical Pipeline)

## Logical_Flow 表

这是 OVN 中最重要的表之一。它包含由 `ovn-northd` 生成的逻辑流表。

### 特性

- **平台无关**: 逻辑流不包含物理端口号或 VLAN ID，而是使用逻辑端口名称和逻辑数据路径 ID。
- **Pipeline**: 分为 `ingress` (入站) 和 `egress` (出站) 两个流水线。

### 核心列

- **logical_datapath**: 关联的逻辑数据路径（指向 `Datapath_Binding`）。
- **pipeline**: 流水线阶段 (`ingress` 或 `egress`)。
- **table_id**: 逻辑表 ID (0-31)。
- **priority**: 优先级（0-65535）。
- **match**: 匹配条件（如 `inport == "lp1" && ip4.dst == 10.0.0.2`）。
- **actions**: 动作（如 `output;`, `next;`, `ct_commit;`）。

## Datapath_Binding 表

代表一个逻辑交换机或逻辑路由器在 SB DB 中的实例。

- **tunnel_key**: 在隧道传输中使用的 24-bit 标识符 (VNI)。
- **external_ids**: 关联到 NB DB 中的 Logical_Switch 或 Logical_Router UUID。

## Multicast_Group 表

定义逻辑多播组，用于广播、组播或未知单播流量。

- **name**: 组名称。
- **datapath**: 关联的数据路径。
- **tunnel_key**: 组 ID。
- **ports**: 属于该组的逻辑端口列表（`Port_Binding` 引用）。
