# Gateway_Chassis 表

`Port_Binding` 行为 `chassisredirect` 类型的行与 `Chassis` 的关联。通过特定 `chassisredirect` 端口发出的流量将被重定向到一个机箱，或者在高可用性配置中重定向到一组机箱。

## 汇总

*   `name`
    *   字符串 (在表中必须唯一)
*   `chassis`
    *   可选的 `Chassis` 弱引用
*   `priority`
    *   整数，范围 0 到 32,767
*   `options`
    *   字符串对的映射
*   通用列:
    *   `external_ids`
        *   字符串对的映射

## 详细信息

*   `name`: 字符串 (在表中必须唯一)
    *   `Gateway_Chassis` 的名称。
    *   建议但不强制的命名约定是 `${port_name}_${chassis_name}`。

*   `chassis`: 可选的 `Chassis` 弱引用
    *   我们要将流量发送到的机箱。

*   `priority`: 整数，范围 0 到 32,767
    *   这是特定机箱在属于同一 `Port_Binding` 的所有 `Gateway_Chassis` 中的优先级。

*   `options`: 字符串对的映射
    *   保留供将来使用。

*   **通用列**:

    *   这些列的总体用途在本文档开头的“通用列”下进行了描述。

    *   `external_ids`: 字符串对的映射

# HA_Chassis 表

## 汇总

*   `chassis`
    *   可选的 `Chassis` 弱引用
*   `priority`
    *   整数，范围 0 到 32,767
*   通用列:
    *   `external_ids`
        *   字符串对的映射

## 详细信息

*   `chassis`: 可选的 `Chassis` 弱引用
    *   提供 HA 功能的机箱。

*   `priority`: 整数，范围 0 到 32,767
    *   HA 机箱的优先级。具有最高优先级的机箱将成为 HA 机箱组中的活动机箱。

*   **通用列**:

    *   这些列的总体用途在本文档开头的“通用列”下进行了描述。

    *   `external_ids`: 字符串对的映射

# HA_Chassis_Group 表

表示可以提供高可用性服务的机箱组的表。组中的每个机箱由 `HA_Chassis` 表表示。具有最高优先级的 HA 机箱将成为此组的活动机箱。如果检测到活动机箱故障转移，则具有下一高优先级的 HA 机箱将接管提供 HA 的责任。如果 `Port_Binding` 表的 `ha_chassis_group` 列引用此表，则此 HA 机箱组提供网关功能并将网关流量重定向到此组的活动机箱。

## 汇总

*   `name`
    *   字符串 (在表中必须唯一)
*   `ha_chassis`
    *   `HA_Chassis` 集合
*   `ref_chassis`
    *   `Chassis` 的弱引用集合
*   通用列:
    *   `external_ids`
        *   字符串对的映射

## 详细信息

*   `name`: 字符串 (在表中必须唯一)
    *   `HA_Chassis_Group` 的名称。名称应该是唯一的。

*   `ha_chassis`: `HA_Chassis` 集合
    *   属于此组的 `HA_Chassis` 列表。

*   `ref_chassis`: `Chassis` 的弱引用集合
    *   引用此 HA 机箱组的机箱集合。为了确定正确的机箱，请找到引用此 `HA_Chassis_Group` 的 `chassisredirect` 类型 `Port_Binding`。此 `Port_Binding` 源自某个特定的逻辑路由器。从该逻辑路由器开始，找到所有连接到它的逻辑交换机和路由器，直接或间接，通过连接一个 LRP 到另一个 LRP 或 LSP 的路由器端口。对于这些逻辑交换机中的每个 LSP，找到相应的 `Port_Binding` 并将其绑定的机箱（如果有）添加到 `ref_chassis`。

*   **通用列**:

    *   这些列的总体用途在本文档开头的“通用列”下进行了描述。

    *   `external_ids`: 字符串对的映射

# BFD 表

包含用于 `ovn-controller` bfd 配置的 BFD 参数。

## 汇总

*   配置:
    *   `src_port`
        *   整数，范围 49,152 到 65,535
    *   `disc`
        *   整数
    *   `logical_port`
        *   字符串
    *   `dst_ip`
        *   字符串
    *   `min_tx`
        *   整数
    *   `min_rx`
        *   整数
    *   `detect_mult`
        *   整数
    *   `chassis_name`
        *   字符串
    *   `options`
        *   字符串对的映射
    *   `external_ids`
        *   字符串对的映射
*   状态报告:
    *   `status`
        *   字符串，`admin_down`, `down`, `init`, 或 `up` 之一

## 详细信息

*   **配置**:

    *   `src_port`: 整数，范围 49,152 到 65,535
        *   bfd 控制数据包中使用的 udp 源端口。源端口必须在 49152 到 65535 范围内（RFC5881 第 4 节）。

    *   `disc`: 整数
        *   发送系统生成的唯一的非零鉴别器值，用于在同一对系统之间解复用多个 BFD 会话。

    *   `logical_port`: 字符串
        *   BFD 引擎运行时对应的 OVN 逻辑端口。

    *   `dst_ip`: 字符串
        *   BFD 对等体 IP 地址。

    *   `min_tx`: 整数
        *   这是本地系统希望在传输 BFD 控制数据包时使用的最小间隔（以毫秒为单位），减去应用的任何抖动。值零被保留。

    *   `min_rx`: 整数
        *   这是此系统能够支持的接收 BFD 控制数据包之间的最小间隔（以毫秒为单位），减去发送方应用的任何抖动。如果此值为零，则发送系统不希望远程系统发送任何定期 BFD 控制数据包。

    *   `detect_mult`: 整数
        *   检测时间倍数。协商的传输间隔乘以该值，为接收系统在异步模式下提供检测时间。

    *   `chassis_name`: 字符串
        *   绑定逻辑端口的机箱的名称。

    *   `options`: 字符串对的映射
        *   保留供将来使用。

    *   `external_ids`: 字符串对的映射
        *   请参阅本文档开头的“外部 ID”。

*   **状态报告**:

    *   `status`: 字符串，`admin_down`, `down`, `init`, 或 `up` 之一
        *   BFD 端口逻辑状态。可能的值为：
            *   `admin_down`
            *   `down`
            *   `init`
            *   `up`
