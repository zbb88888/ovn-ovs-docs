# Network_Function_Group 表

每行包含一个 Network_Function 列表。流量重定向是通过从 ACL 引用 Network_Function_Group 来实现的。每个 Network_Function 的健康监控是根据 Network_Function_Health_Check 中定义的参数执行的。匹配 ACL 的流量被重定向到活动的 Network_Function 之一。如果检测到所有 Network_Function 都已关闭，则无论状态如何，流量都会重定向到其中一个 Network_Function。

## 摘要

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `name` | 字符串（表内必须唯一） | 组名称。 |
| `id` | 1 到 255 的整数（表内必须唯一） | 组 ID。 |
| `fallback` | 可选字符串，`fail-close` 或 `fail-open` | 回退机制。 |
| `network_function` | `Network_Function` 的集合 | 网络功能集合。 |
| `network_function_active` | 可选 `Network_Function` | 活动网络功能。 |
| `mode` | 字符串，必须为 `inline` | 模式。 |

**公共列：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `external_ids` | 字符串对映射 | 外部 ID。 |

## 详细信息

-   `name`: 字符串（表内必须唯一）
    Network_Function_Group 的名称。名称应该是唯一的。

-   `id`: 1 到 255 的整数（表内必须唯一）
    必须为每个 Network_Function_Group 分配一个 1 到 255 之间的唯一整数。

-   `fallback`: 可选字符串，`fail-close` 或 `fail-open`
    当没有可用的活动网络功能时的回退设置。

    支持以下回退机制。如果未指定，当没有可用的活动网络功能时，将应用 `fail-close`。

    *   `fail-open`
        当没有可用的活动网络功能时，流量绕过网络功能并被允许。

    *   `fail-close`
        当没有可用的活动网络功能时，流量被丢弃。

-   `network_function`: `Network_Function` 的集合
    属于此组的网络功能列表。

-   `network_function_active`: 可选 `Network_Function`
    当前活动的 Network_Function。此列由 northd 根据健康监控状态填充。

-   `mode`: 字符串，必须为 `inline`
    流量转发模式，默认且唯一值为 "inline"。"inline" 模式意味着网络功能直接在流量路径中，流量通过它重定向。

### 公共列

-   `external_ids`: 字符串对映射
    参见本文件开头的外部 ID。

---

# Network_Function 表

每行代表一个网络功能实体。这包含一对 `logical_switch_ports`。匹配 ACL 的流量对于 `from-lport` ACL 被重定向到 `inport`，对于 `to-lport` ACL 被重定向到 `outport`。一旦在另一个端口上接收到流量，它将继续通过标准 OVN 管道。响应流量遵循反向路径：对于 `from-lport` ACL 被重定向到 `outport`，对于 `to-lport` ACL 被重定向到 `inport`。一旦在另一个端口上接收到流量，它将由常规 OVN 管道处理。

注意：
1.  网络功能不得修改数据包头。
2.  当与负载均衡器结合使用时，不支持网络功能。

## 摘要

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `name` | 字符串（表内必须唯一） | 名称。 |
| `inport` | `Logical_Switch_Port` | 入口端口。 |
| `outport` | `Logical_Switch_Port` | 出口端口。 |
| `health_check` | 可选 `Network_Function_Health_Check` | 健康检查。 |

**公共列：**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `external_ids` | 字符串对映射 | 外部 ID。 |

## 详细信息

-   `name`: 字符串（表内必须唯一）
    Network_Function 的名称。名称应该是唯一的。

-   `inport`: `Logical_Switch_Port`
    `from-lport` ACL 的请求流量和 `to-lport` ACL 的响应流量重定向到的 `Logical_Switch_Port`。

-   `outport`: `Logical_Switch_Port`
    `to-lport` ACL 的请求流量和 `from-lport` ACL 的响应流量重定向到的 `Logical_Switch_Port`。

-   `health_check`: 可选 `Network_Function_Health_Check`
    与此网络功能关联的 `Network_Function_Health_Check`。

### 公共列

-   `external_ids`: 字符串对映射
    参见本文件开头的外部 ID。

---

# Network_Function_Health_Check 表

每行代表一个网络功能健康检查。

## 摘要

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `name` | 字符串（表内必须唯一） | 名称。 |

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

-   `name`: 字符串（表内必须唯一）
    Network_Function_Health_Check 的名称。名称应该是唯一的。

### 健康检查选项

-   `options : interval`: 可选字符串，包含整数
    健康检查之间的间隔，以秒为单位。默认值：5s。

-   `options : timeout`: 可选字符串，包含整数
    健康检查超时的秒数。默认值：3s。

-   `options : success_count`: 可选字符串，包含整数
    网络功能被认为在线之前的成功检查次数。默认值：1。

-   `options : failure_count`: 可选字符串，包含整数
    网络功能被认为离线之前的失败检查次数。默认值：1。

### 公共列

-   `external_ids`: 字符串对映射
    参见本文件开头的外部 ID。
