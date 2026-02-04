# Gateway_Chassis 表

将 Chassis 关联到逻辑路由器端口。通过特定路由器端口发出的流量将被重定向到一个 Chassis，或者在高可用性配置中重定向到一组 Chassis。

## 摘要

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `name` | 字符串（表内必须唯一） | Gateway Chassis 名称。 |
| `chassis_name` | 字符串 | Chassis 名称。 |
| `priority` | 0 到 32,767 的整数 | 优先级。 |
| `options` | 字符串对映射 | 选项。 |

**公共列：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `external_ids` | 字符串对映射 | 外部 ID。 |

## 详细信息

-   `name`: 字符串（表内必须唯一）
    Gateway_Chassis 的名称。

    建议但非必须的命名约定是 `${port_name}_${chassis_name}`。

-   `chassis_name`: 字符串
    我们希望关联的逻辑路由器端口的流量通过的 Chassis 的名称。该值必须与 `OVN_Southbound` 数据库中 `Chassis` 表的 name 列匹配。

-   `priority`: 0 到 32,767 的整数
    这是属于同一逻辑路由器端口的所有 Gateway_Chassis 中 Chassis 的优先级。

-   `options`: 字符串对映射
    保留供将来使用。

### 公共列

-   `external_ids`: 字符串对映射
    参见本文件开头的外部 ID。

---

# HA_Chassis_Group 表

代表一组可提供高可用性服务的 Chassis 的表。组中的每个 Chassis 由 `HA_Chassis` 表表示。具有最高优先级的 HA Chassis 将是该组的活动 Chassis。如果检测到活动 Chassis 故障转移，则具有下一高优先级的 HA Chassis 将接管提供 HA 的责任。如果分布式网关路由器端口引用此表中的行，则此组中的活动 HA Chassis 提供网关功能。

## 摘要

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `name` | 字符串（表内必须唯一） | HA Chassis 组名称。 |
| `ha_chassis` | `HA_Chassis` 的集合 | HA Chassis 集合。 |

**公共列：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `external_ids` | 字符串对映射 | 外部 ID。 |

## 详细信息

-   `name`: 字符串（表内必须唯一）
    HA_Chassis_Group 的名称。名称应该是唯一的。

-   `ha_chassis`: `HA_Chassis` 的集合
    属于此组的 HA Chassis 列表。

### 公共列

-   `external_ids`: 字符串对映射
    参见本文件开头的外部 ID。

---

# HA_Chassis 表

## 摘要

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `chassis_name` | 字符串 | Chassis 名称。 |
| `priority` | 0 到 32,767 的整数 | 优先级。 |

**公共列：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `external_ids` | 字符串对映射 | 外部 ID。 |

## 详细信息

-   `chassis_name`: 字符串
    作为 HA Chassis 组一部分的 Chassis 的名称。该值必须与 `OVN_Southbound` 数据库中 `Chassis` 表的 name 列匹配。

-   `priority`: 0 到 32,767 的整数
    Chassis 的优先级。具有最高优先级的 Chassis 将是活动 Chassis。

### 公共列

-   `external_ids`: 字符串对映射
    参见本文件开头的外部 ID。

---

# BFD 表

包含用于 ovn-controller BFD 配置的 BFD 参数。OVN BFD 实现用于提供相邻转发引擎（包括 OVN 接口）之间路径故障的检测。OVN BFD 向 OVN northd 提供链路状态信息，以便根据 BFD 端点的状态更新逻辑流。在当前的实现中，OVN BFD 用于检查 ECMP 路由的下一跳状态。请注意，BFD 表指的是 OVN BFD 实现，而不是 OVS 遗留的那个。

## 摘要

**配置：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `logical_port` | 字符串 | 逻辑端口。 |
| `dst_ip` | 字符串 | 目标 IP。 |
| `min_tx` | 可选整数，至少 1 | 最小 TX。 |
| `min_rx` | 可选整数 | 最小 RX。 |
| `detect_mult` | 可选整数，至少 1 | 检测倍数。 |
| `options` | 字符串对映射 | 选项。 |
| `external_ids` | 字符串对映射 | 外部 ID。 |

**状态报告：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `status` | 可选字符串，`admin_down`, `down`, `init`, 或 `up` 之一 | 状态。 |

## 详细信息

### 配置

ovn-northd 从这些列读取配置。

-   `logical_port`: 字符串
    运行 BFD 引擎时的 OVN 逻辑端口。

-   `dst_ip`: 字符串
    BFD 对等 IP 地址。

-   `min_tx`: 可选整数，至少 1
    这是本地系统在发送 BFD 控制数据包时希望使用的最小间隔（以毫秒为单位），减去应用的任何抖动。值为零是保留的。默认值为 1000 毫秒。

-   `min_rx`: 可选整数
    这是此系统能够支持的接收 BFD 控制数据包之间的最小间隔（以毫秒为单位），减去发送方应用的任何抖动。如果此值为零，则发送系统不希望远程系统发送任何定期的 BFD 控制数据包。

-   `detect_mult`: 可选整数，至少 1
    检测时间倍数。协商的发送间隔乘以该值，为接收系统在异步模式下提供检测时间。默认值为 5。

-   `options`: 字符串对映射
    保留供将来使用。

-   `external_ids`: 字符串对映射
    参见本文件开头的外部 ID。

### 状态报告

ovn-northd 将 BFD 状态写入这些列。

-   `status`: 可选字符串，`admin_down`, `down`, `init`, 或 `up` 之一
    BFD 端口逻辑状态。可能的值为：

    *   `admin_down`
    *   `down`
    *   `init`
    *   `up`

---

# Static_MAC_Binding 表

每条记录代表逻辑路由器的一个 Static_MAC_Binding 条目。

## 摘要

**配置：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `logical_port` | 字符串 | 逻辑端口。 |
| `ip` | 字符串 | IP 地址。 |
| `mac` | 字符串 | MAC 地址。 |
| `override_dynamic_mac` | 布尔值 | 覆盖动态 MAC。 |

## 详细信息

### 配置

ovn-northd 从这些列读取配置并将值传播到 SBDB。

-   `logical_port`: 字符串
    绑定的逻辑路由器端口。

-   `ip`: 字符串
    绑定的 IP 地址。

-   `mac`: 字符串
    IP 绑定到的以太网地址。

-   `override_dynamic_mac`: 布尔值
    覆盖动态学习的 MAC。
