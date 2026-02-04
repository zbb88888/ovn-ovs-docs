# Advertised_Route 表

每条记录代表应从 OVN 导出到外部网络结构的路由。它由 `ovn-northd` 根据 `OVN_Northbound.Logical_Router_Port` 的地址、路由和 NAT 条目填充。

## 汇总

*   `datapath`
    *   `Datapath_Binding`
*   `logical_port`
    *   `Port_Binding`
*   `ip_prefix`
    *   字符串
*   `tracked_port`
    *   可选的 `Port_Binding`
*   通用列:
    *   `external_ids`
        *   字符串对的映射

## 详细信息

*   `datapath`: `Datapath_Binding`
    *   此路由所属的 `OVN_Northbound.Logical_Router` 的数据路径。

*   `logical_port`: `Port_Binding`
    *   这是路由器将发送针对以下前缀接收的数据包的 `Port_Binding`。

*   `ip_prefix`: 字符串
    *   此路由的 IP 前缀（例如 192.168.100.0/24）。

*   `tracked_port`: 可选的 `Port_Binding`
    *   结合主机 `ip_prefix`，这跟踪 OVN 将把此目的地的数据包转发到的端口。如果设置，`ip_prefix` 将始终包含 /32 (对于 ipv4) 或 /128 (对于 ipv6) 前缀。宣布机箱可以使用此信息来检查此目的地是否在本地，并据此调整路由优先级。

*   **通用列**:

    *   `external_ids`: 字符串对的映射
        *   请参阅本文档开头的“外部 ID”。

# Learned_Route 表

每条记录代表 OVN 使用 OVN 外部的一些动态路由逻辑学习到的路由。它由 `ovn-controller` 使用它在本地学习到的路由填充。

## 汇总

*   `datapath`
    *   `Datapath_Binding`
*   `logical_port`
    *   `Port_Binding`
*   `ip_prefix`
    *   字符串
*   `nexthop`
    *   字符串
*   通用列:
    *   `external_ids`
        *   字符串对的映射

## 详细信息

*   `datapath`: `Datapath_Binding`
    *   此路由所属的 `OVN_Northbound.Logical_Router` 的数据路径。

*   `logical_port`: `Port_Binding`
    *   学习路由的 `Port_Binding`。

*   `ip_prefix`: 字符串
    *   此路由的 IP 前缀（例如 192.168.100.0/24）。

*   `nexthop`: 字符串
    *   我们从 OVN 外部学习到的下一跳 ip。

*   **通用列**:

    *   `external_ids`: 字符串对的映射
        *   请参阅本文档开头的“外部 ID”。

# ECMP_Nexthop 表

此表中的每一行代表使用 `--ecmp-symmetric-reply` 选项创建的 ECMP 路由的活动下一跳，该选项由 `ovn-northd` 提交给 ovs 连接跟踪器。`ECMP_Nexthop` 表由 `ovn-controller` 用于跟踪活动 ct 条目并刷新陈旧条目。

## 汇总

*   `nexthop`
    *   字符串
*   `port`
    *   `Port_Binding`
*   `datapath`
    *   `Datapath_Binding`
*   `mac`
    *   字符串
*   通用列:
    *   `external_ids`
        *   字符串对的映射

## 详细信息

*   `nexthop`: 字符串
    *   此 ECMP 路由的下一跳 IP 地址。下一跳 IP 地址应该是连接的路由器端口的 IP 地址或用作给定目的地的下一跳的外部设备的 IP 地址。

*   `port`: `Port_Binding`
    *   用于连接到配置的下一跳的端口的 `Port_Binding` 表引用。

*   `datapath`: `Datapath_Binding`
    *   `Datapath_Binding` 表的引用，用于连接到配置的下一跳的端口正在运行的数据路径。

*   `mac`: 字符串
    *   下一跳 mac 地址。

*   **通用列**:

    *   `external_ids`: 字符串对的映射
        *   请参阅本文档开头的“外部 ID”。

# ACL_ID 表

每条记录代表 `ovn-northd` 需要与 `ovn-controller` 实例同步的标识符。每条记录的 UUID 直接对应于北向数据库中的 ACL 记录。

## 汇总

*   `id`
    *   整数，范围 0 到 32,767

## 详细信息

*   `id`: 整数，范围 0 到 32,767
    *   对应于北向 `allow-established` ACL 的标识符。

# Advertised_MAC_Binding 表

每条记录代表如果数据路径上启用了 EVPN，`OVN_Northbound.Logical_Switch` 正在网络结构外部宣布的 ip/mac 绑定。

## 汇总

*   `datapath`
    *   `Datapath_Binding`
*   `logical_port`
    *   `Port_Binding`
*   `ip`
    *   字符串
*   `mac`
    *   字符串
*   通用列:
    *   `external_ids`
        *   字符串对的映射

## 详细信息

*   `datapath`: `Datapath_Binding`
    *   此 ip/mac 绑定所属的 `OVN_Northbound.Logical_Switch` 对应的数据路径。

*   `logical_port`: `Port_Binding`
    *   此 ip/mac 绑定所属的 `Port_Binding`。

*   `ip`: 字符串
    *   宣布的 IP（例如 192.168.100.1）。

*   `mac`: 字符串
    *   宣布的 mac 地址。

*   **通用列**:

    *   `external_ids`: 字符串对的映射
        *   请参阅本文档开头的“外部 ID”。

# Chassis_Template_Var 表

每条记录代表给定机箱的模板变量实例化集，并由 `ovn-northd` 从 `OVN_Northbound.Chassis_Template_Var` 表的内容填充。

## 汇总

*   `chassis`
    *   字符串 (在表中必须唯一)
*   `variables`
    *   字符串对的映射

## 详细信息

*   `chassis`: 字符串 (在表中必须唯一)
    *   这组变量值适用的机箱。

*   `variables`: 字符串对的映射
    *   给定机箱的变量值集。

# Controller_Event 表

`ovn-controller` 用于报告 CMS 相关事件的数据库表。请注意，不能保证给定事件在 db 中仅写入一次。CMS 有责任压缩重复的行或过滤掉重复的事件。

## 汇总

*   `event_type`
    *   字符串，必须是 `empty_lb_backends`
*   `event_info`
    *   字符串对的映射
*   `chassis`
    *   可选的 `Chassis` 弱引用
*   `seq_num`
    *   整数

## 详细信息

*   `event_type`: 字符串，必须是 `empty_lb_backends`
    *   发生的事件类型

*   `event_info`: 字符串对的映射
    *   用于向 CMS 指定事件信息的键值对。可能的值为：
        *   `vip`: 为 `empty_lb_backends` 事件报告的 VIP
        *   `protocol`: 为 `empty_lb_backends` 事件报告的传输协议
        *   `load_balancer`: 为 `empty_lb_backends` 事件报告的负载均衡器的 UUID

*   `chassis`: 可选的 `Chassis` 弱引用
    *   此列是 `Chassis` 记录，用于标识管理给定事件的机箱。

*   `seq_num`: 整数
    *   事件序列号。控制器生成事件的全局计数器。CMS 可以使用它来检测同一事件的可能重复。
