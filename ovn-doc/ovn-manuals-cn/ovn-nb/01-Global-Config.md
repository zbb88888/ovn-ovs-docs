# NB_Global 表

OVN 系统的北向配置。此表必须只有一行。

## 汇总

* 身份:
  * `name`
    * 字符串
* 状态:
  * `nb_cfg`
    * 整数
  * `nb_cfg_timestamp`
    * 整数
  * `sb_cfg`
    * 整数
  * `sb_cfg_timestamp`
    * 整数
  * `hv_cfg`
    * 整数
  * `hv_cfg_timestamp`
    * 整数
* 通用列:
  * `external_ids`
    * 字符串对的映射
* 通用选项:
  * `options`
    * 字符串对的映射
  * 配置 OVS BFD 的选项:
    * `options : bfd-min-rx`
      * 可选字符串
    * `options : bfd-decay-min-rx`
      * 可选字符串
    * `options : bfd-min-tx`
      * 可选字符串
    * `options : bfd-mult`
      * 可选字符串
  * `options : ignore_chassis_features`
    * 可选字符串
  * `options : mac_prefix`
    * 可选字符串
  * `options : mac_binding_removal_limit`
    * 可选字符串，包含整数，范围 0 到 4,294,967,295
  * `options : fdb_removal_limit`
    * 可选字符串，包含整数，范围 0 到 4,294,967,295
  * `options : controller_event`
    * 可选字符串，`true` 或 `false`
  * `options : northd_probe_interval`
    * 可选字符串
  * `options : ic_probe_interval`
    * 可选字符串
  * `options : nbctl_probe_interval`
    * 可选字符串
  * `options : northd_trim_timeout`
    * 可选字符串
  * `options : use_logical_dp_groups`
    * 可选字符串
  * `options : use_parallel_build`
    * 可选字符串
  * `options : ignore_lsp_down`
    * 可选字符串
  * `options : use_ct_inv_match`
    * 可选字符串
  * `options : default_acl_drop`
    * 可选字符串
  * `options : debug_drop_domain_id`
    * 可选字符串
  * `options : debug_drop_collector_set`
    * 可选字符串
  * `options : use_common_zone`
    * 可选字符串，`true` 或 `false`
  * `options : northd-backoff-interval-ms`
    * 可选字符串
  * `options : vxlan_mode`
    * 可选字符串
  * `options : always_tunnel`
    * 可选字符串，`true` 或 `false`
  * `options : ecmp_nexthop_monitor_enable`
    * 可选字符串
  * `options : enable_chassis_nb_cfg_update`
    * 可选字符串
  * 服务健康检查配置选项:
    * `options : svc_monitor_mac`
      * 可选字符串
    * `options : svc_monitor_mac_dst`
      * 可选字符串
    * `options : svc_monitor_ip`
      * 可选字符串
    * `options : svc_monitor_ip_dst`
      * 可选字符串
  * 配置互连路由通告的选项:
    * `options : ic-route-adv`
      * 可选字符串
    * `options : ic-route-learn`
      * 可选字符串
    * `options : ic-route-adv-default`
      * 可选字符串
    * `options : ic-route-learn-default`
      * 可选字符串
    * `options : ic-route-adv-lb`
      * 可选字符串
    * `options : ic-route-learn-lb`
      * 可选字符串
    * `options : ic-route-denylist`
      * 可选字符串
* 连接选项:
  * `connections`
    * `Connection` 集合
  * `ssl`
    * 可选的 `SSL`
* 安全配置:
  * `ipsec`
    * 布尔值
* 只读选项:
  * `options : max_tunid`
    * 可选字符串

## 详细信息

* **身份**:

  * `name`: 字符串
    * OVN 集群的名称，用于在所有应该相互连接的 OVN 集群中唯一标识 OVN 集群。

* **状态**:

  * 这些列允许客户端跟踪系统的整体配置状态。

  * `nb_cfg`: 整数
    * 供客户端递增的序列号。当客户端修改北向数据库配置的任何部分并希望等待 ovn-northd 以及可能的所有管理程序完成应用更改时，它可以递增此序列号。

  * `nb_cfg_timestamp`: 整数
    * 自纪元以来的时间戳（以毫秒为单位），表示 ovn-northd 看到最新 nb_cfg 并开始处理的时间。

    * 要将时间戳打印为人类可读的日期：

    * `date -d "@$(ovn-nbctl get NB_Global . nb_cfg_timestamp | sed 's/...$//')"`

  * `sb_cfg`: 整数
    * ovn-northd 在完成将相应的配置更改应用到 `OVN_Southbound` 数据库后，将此序列号设置为 `nb_cfg` 的值。

  * `sb_cfg_timestamp`: 整数
    * 自纪元以来的时间戳（以毫秒为单位），表示 ovn-northd 成功完成将相应的配置更改应用到 `OVN_Southbound` 数据库的时间。

  * `hv_cfg`: 整数
    * ovn-northd 将此序列号设置为系统中所有机箱的最小序列号，如南向数据库的 `Chassis_Private` 表中所报告的那样。因此，如果所有机箱都赶上了北向配置（如果任何机箱宕机，这可能永远不会发生），则 `hv_cfg` 等于 `nb_cfg`。如果机箱从系统中移除并在赶上之前重新加入，则此值可能会回退。

    * 如果没有机箱，则 ovn-northd 将 `nb_cfg` 复制到 `hv_cfg`。因此，在这种情况下，（不存在的）管理程序始终被认为是赶上的。这意味着即使 `sb_cfg` 显示南向数据库未赶上，管理程序也可以是“赶上的”。为了检测管理程序和南向数据库何时都赶上，客户端应取 `sb_cfg` 和 `hv_cfg` 中的较小值。

  * `hv_cfg_timestamp`: 整数
    * 系统中所有机箱的最小序列号的最大时间戳（以毫秒为单位），如南向数据库的 `Chassis_Private` 表中所报告的那样。换句话说，此时间戳反映了最慢的机箱赶上北向配置的时间，这对于端到端控制平面延迟测量非常有用。

* **通用列**:

  * `external_ids`: 字符串对的映射
    * 请参阅本文档开头的“外部 ID”。

* **通用选项**:

  * `options`: 字符串对的映射
    * 此列提供常规键/值设置。支持的选项在下面单独描述。

  * **配置 OVS BFD 的选项**:

  * 当 `ovn-controller` 在隧道接口上配置 OVS BFD 时应用这些选项。请注意，这些参数指的是旧版 OVS BFD 实现，而不是 OVN BFD 实现。

  * `options : bfd-min-rx`: 可选字符串
    * 在隧道接口上配置 BFD 时使用的 BFD 选项 min-rx 值。

  * `options : bfd-decay-min-rx`: 可选字符串
    * 在隧道接口上配置 BFD 时使用的 BFD 选项 decay-min-rx 值。

  * `options : bfd-min-tx`: 可选字符串
    * 在隧道接口上配置 BFD 时使用的 BFD 选项 min-tx 值。

  * `options : bfd-mult`: 可选字符串
    * 在隧道接口上配置 BFD 时使用的 BFD 选项 mult 值。

  * `options : ignore_chassis_features`: 可选字符串
    * 当设置为 false 时，ovn-northd 将评估每个机箱支持的功能，并且仅激活所有机箱普遍支持的功能。这种方法对于在 ovn-northd 在 ovn-controller 之前更新的升级期间保持向后兼容性至关重要。但是，如果任何机箱管理不善且升级不成功，它将限制 ovn-northd 激活新功能。

    * 或者，将此选项设置为 true 指示 ovn-northd 绕过每个机箱上的功能支持状态，并直接实现最新功能。这种方法可以保护 ovn-northd 的运行免受机箱配置不匹配的不利影响。

    * 此选项的默认设置为 false。

  * `options : mac_prefix`: 可选字符串
    * 配置在动态分配 L2 地址时用作前缀的给定 OUI，例如 00:11:22。

  * `options : mac_binding_removal_limit`: 可选字符串，包含整数，范围 0 到 4,294,967,295
    * MAC 绑定老化批量移除限制。这限制了在单个事务中可以过期的行数。默认值为 0，即无限制。当我们达到限制时，下一批移除将延迟 5 秒。

  * `options : fdb_removal_limit`: 可选字符串，包含整数，范围 0 到 4,294,967,295
    * FDB 老化批量移除限制。这限制了在单个事务中可以过期的行数。默认值为 0，即无限制。当我们达到限制时，下一批移除将延迟 5 秒。

  * `options : controller_event`: 可选字符串，`true` 或 `false`
    * 由 CMS 设置的值，用于启用/禁用 ovn-controller 事件报告。进入 OVS 的流量可能会引发 'controller' 事件，从而导致将 `Controller_Event` 写入 SBDB 中的 `Controller_Event` 表。当 CMS 看到该事件并采取适当操作时，它可以删除 `Controller_Event` 表中的相应行。目的是让 CMS 看到事件并采取某种行动。请参阅 SBDB 中的 `Controller_Event` 表。可以将仪表与每个控制器事件类型相关联，以免在重负载下使 pinctrl 线程过载。每个事件类型都依赖于具有定义名称的仪表：

    * `empty_lb_backends`: event-elb

  * `options : northd_probe_interval`: 可选字符串
    * 从 ovn-northd 到 OVN 北向和南向数据库的连接的不活动探测间隔（以毫秒为单位）。如果该值为零，则禁用连接保活功能。

    * 如果该值非零，则它将被强制为至少 1000 毫秒的值。

  * `options : ic_probe_interval`: 可选字符串
    * 从 ovn-ic 到 OVN 北向和南向数据库的连接的不活动探测间隔（以毫秒为单位）。如果该值为零，则禁用连接保活功能。

    * 如果该值非零，则它将被强制为至少 1000 毫秒的值。

  * `options : nbctl_probe_interval`: 可选字符串
    * 从 ovn-nbctl 实用程序到 OVN 北向数据库的连接的不活动探测间隔（以毫秒为单位）。如果该值为零，则禁用连接保活功能。

    * 如果该值非零，则它将被强制为至少 1000 毫秒的值。

    * 如果该值小于零，则 ovn-nbctl 的默认不活动探测间隔将保持不变（120000 毫秒）。

  * `options : northd_trim_timeout`: 可选字符串
    * 使用时，此配置值指定自上次 ovn-northd 活动操作以来执行内存修整的时间（以毫秒为单位）。默认情况下，此设置为 30000（30 秒）。

  * `options : use_logical_dp_groups`: 可选字符串
    * 注意：此选项已弃用，唯一的行为是始终按数据路径组组合逻辑流。更改值或完全删除此选项将没有任何效果。

    * ovn-northd 将仅逻辑数据路径不同的逻辑流组合成附加了逻辑数据路径组的单个逻辑流。

  * `options : use_parallel_build`: 可选字符串
    * 如果设置为 true，ovn-northd 将尝试并行计算逻辑流。

    * 仅当系统有 4 个或更多核心/线程可供 ovn-northd 使用时，才会启用并行计算。

    * 默认值为 false。

  * `options : ignore_lsp_down`: 可选字符串
    * 如果设置为 false，则仅当端口启动（即被机箱声明）时，才会安装逻辑交换机端口的 ARP/ND 回复流。如果设置为 true，则无论端口状态如何，都会安装这些流，这可能会导致在相关 VM/容器运行之前就解析了对 IP 的 ARP 请求的情况。对于这不是问题的环境，将其设置为 true 可以减少控制平面的负载和延迟。默认值为 true。

  * `options : use_ct_inv_match`: 可选字符串
    * 如果设置为 false，ovn-northd 将不会在任何逻辑流匹配中使用 ct.inv 字段。默认值为 true。如果 NIC 支持卸载 OVS 数据路径流但不支持卸载 ct_state inv 标志，则匹配此标志（+inv 或 -inv）的数据路径流将不会被卸载。在这种情况下，CMS 应考虑将 use_ct_inv_match 设置为 false。这会导致无效数据包被传递到目标 VIF 的副作用，否则这些数据包将被 OVN 丢弃。

  * `options : default_acl_drop`: 可选字符串
    * 如果设置为 true，ovn-northd 将生成一个逻辑流以丢弃 ACL 阶段的所有流量。默认情况下，此选项设置为 false。

  * `options : debug_drop_domain_id`: 可选字符串
    * 如果设置为 8 位数字并且还配置了 debug_drop_collector_set，则 ovn-northd 将向包含 'drop' 操作的每个逻辑流添加一个 sample 操作。observation_domain_id 字段的 8 个最高有效位将是 debug_drop_domain_id 中指定的那些位。observation_domain_id 字段的 24 个最低有效位将是数据路径的键。

    * observation_point_id 将设置为逻辑流 UUID 的前 32 位。

    * 注意：此键已弃用，取而代之的是为 drop 应用程序在 `Sampling_App` 表中配置的值。

  * `options : debug_drop_collector_set`: 可选字符串
    * 如果设置为 32 位数字，ovn-northd 将向包含 'drop' 操作的每个逻辑流添加一个 sample 操作。sample 操作将具有指定的 collector_set_id。该值必须匹配 ovs-actions(7) 中描述的本地 OVS 配置的值。

  * `options : use_common_zone`: 可选字符串，`true` 或 `false`
    * 默认值为 false。如果设置为 true，则 SNAT 和 DNAT 发生在公共区域中，而不是根据配置发生在单独的区域中。但是，当此 LR 上配置了 DGP + LB + SNAT 时，此选项会中断流量。仅在与 GDP 的 HWOL 兼容的情况下才应使用值 true。

  * `options : northd-backoff-interval-ms`: 可选字符串
    * northd 增量引擎延迟的最大间隔（以毫秒为单位）。将该值设置为非零会延迟下一次 northd 引擎运行上一次运行时间，上限为指定值。如果该值为零，则引擎根本不会延迟。建议的周期小于 500 毫秒，超过该值，SB 更改的延迟将非常明显。

  * `options : vxlan_mode`: 可选字符串
    * 默认情况下，如果 OVN 集群中至少有一个机箱具有 VXLAN 封装，则 northd 将以 VXLAN 模式运行。有关更多详细信息，请参阅 `ovn-architecture(7)` 手册的隧道封装段落。如果仅在机箱上需要 VXLAN 封装以支持 HW VTEP 功能并且主要封装类型为 GENEVE，请将此选项设置为 false 以使用默认的非 VXLAN 模式隧道 ID 分配逻辑。请考虑当 OVN 在 OVN 互连模式下运行并且使用 VXLAN 封装类型时，非传输逻辑交换机和逻辑路由器的最大数量减少到 1024。请注意，为了为跨 AZ 流量启用 VXLAN 封装类型，必须将 `IC_NB_Global` 表中的 `vxlan_mode` 参数设置为 true。

  * `options : always_tunnel`: 可选字符串，`true` 或 `false`
    * 如果设置为 true，则发往提供商逻辑交换机（具有 localnet 端口）的 VIF 的流量将被隧道传输，而不是通过 localnet 端口发送。如果 CMS 想要将覆盖逻辑交换机（没有 localnet 端口）和提供商逻辑交换机连接到路由器，则此选项很有用。如果不设置此选项，流量路径将是隧道和 localnet 端口的混合（因为路由是分布式的），导致路由器端口 mac 地址泄漏到上游交换机，如果涉及 NAT，则行为未定义。默认情况下禁用此选项。

  * `options : ecmp_nexthop_monitor_enable`: 可选字符串
    * 如果设置为 true，ovn-northd 将在 `ECMP_Nexthop` 表中创建条目以跟踪使用 `--ecmp_symmetric_reply` 选项创建的 ECMP 路由。默认情况下，此选项设置为 false。

  * `options : enable_chassis_nb_cfg_update`: 可选字符串
    * 如果设置为 false，ovn-controller 将不再更新 `OVN_Southbound` 数据库的 `Chassis_Private` 表中的 `nb_cfg` 列。它们仍将更新本地 OVS 集成网桥中的 `external_ids:ovn-nb-cfg`。默认情况下，此选项设置为 true。

  * **服务健康检查配置选项**:

  * 当为 `Load_Balancer` 和 `Network_Function` 服务启用健康配置时，将使用这些选项。

  * `options : svc_monitor_mac`: 可选字符串
    * 在健康检查探测中用作以太网源的 MAC 地址。如果未指定，则自动生成 MAC 地址。

  * `options : svc_monitor_mac_dst`: 可选字符串
    * 在健康检查探测中用作以太网目的地的 MAC 地址。如果未指定，则自动生成 MAC 地址。这仅适用于以 inline 模式部署的网络功能健康检查探测。

  * `options : svc_monitor_ip`: 可选字符串
    * 在健康检查探测中用作源的 IP 地址（IPv4 或 IPv6）。这仅适用于以 inline 模式部署的网络功能健康检查探测。

  * `options : svc_monitor_ip_dst`: 可选字符串
    * 在健康检查探测中用作目的地的 IP 地址（IPv4 或 IPv6）。这仅适用于以 inline 模式部署的网络功能健康检查探测。

  * **配置互连路由通告的选项**:

  * 这些选项控制如何在 OVN 部署之间通告路由以进行互连。如果启用，来自不同 OVN 部署的 ovn-ic 通过全局 `OVN_IC_Southbound` 数据库相互交换路由。只有连接到互连传输交换机的端口的路由器才参与路由通告。对于这些路由器中的每一个，有三种类型的路由要通告：

  * 首先，通告路由器中配置的静态路由。

  * 其次，通告未在传输交换机上的逻辑路由器端口中配置的网络。这些被视为路由器上直接连接的子网。

  * 第三，通告与逻辑路由器关联的负载均衡器的 vip。

  * 链路本地前缀（IPv4 169.254.0.0/16 和 IPv6 FE80::/10）永远不会被通告。

  * 学习到的路由被添加到 `Logical_Router` 表的 `static_routes` 列中，其中 `external_ids:ic-learned-route` 设置为 `OVN_IC_Southbound` 数据库的 `Route` 表中行的 uuid。

  * `options : ic-route-adv`: 可选字符串
    * 一个布尔值，用于启用向全局 `OVN_IC_Southbound` 数据库的路由通告。默认为 false。

  * `options : ic-route-learn`: 可选字符串
    * 一个布尔值，用于启用从全局 `OVN_IC_Southbound` 数据库学习路由。默认为 false。

  * `options : ic-route-adv-default`: 可选字符串
    * 一个布尔值，用于启用向全局 `OVN_IC_Southbound` 数据库通告默认路由。默认为 false。此选项仅在选项 `ic-route-adv` 为 true 时生效。

  * `options : ic-route-learn-default`: 可选字符串
    * 一个布尔值，用于启用从全局 `OVN_IC_Southbound` 数据库学习默认路由。默认为 false。此选项仅在选项 `ic-route-learn` 为 true 时生效。

  * `options : ic-route-adv-lb`: 可选字符串
    * 一个布尔值，用于启用向全局 `OVN_IC_Southbound` 数据库通告负载均衡器 vip 的路由。默认为 false。此选项仅在选项 `ic-route-adv` 为 true 时生效。

  * `options : ic-route-learn-lb`: 可选字符串
    * 一个布尔值，用于启用从全局 `OVN_IC_Southbound` 数据库学习负载均衡器路由的路由。默认为 false。此选项仅在选项 `ic-route-learn` 为 true 时生效。

  * `options : ic-route-denylist`: 可选字符串
    * 包含以 "," 分隔的 CIDR 列表的字符串值。如果路由的前缀属于列出的任何 CIDR，则不会通告或学习该路由。

  * **连接选项**:

  * `connections`: `Connection` 集合
    * Open vSwitch 数据库服务器应连接到或应在其上侦听的数据库客户端，以及有关应如何配置这些连接的选项。有关更多信息，请参阅 `Connection` 表。

  * `ssl`: 可选的 `SSL`
    * 全局 SSL/TLS 配置。

  * **安全配置**:

  * `ipsec`: 布尔值
    * 隧道加密配置。如果此列设置为 true，则所有 OVN 隧道都将使用 IPsec 加密。

  * **只读选项**:

  * `options : max_tunid`: 可选字符串
    * 支持的最大隧道 ID。取决于集群中启用的封装类型。

# Sample_Collector 表

## 汇总

* `id`
  * 整数，范围 1 到 255 (在表中必须唯一)
* `name`
  * 字符串
* `probability`
  * 整数，范围 0 到 65,535
* `set_id`
  * 整数，范围 1 到 4,294,967,295
* `external_ids`
  * 字符串对的映射

## 详细信息

* `id`: 整数，范围 1 到 255 (在表中必须唯一)
  * 样本收集器唯一 ID，用于区分使用相同 set_id 但概率值不同的收集器。ID 的支持值范围是 1-255。

* `name`: 字符串
  * 样本收集器的名称。

* `probability`: 整数，范围 0 到 65,535
  * 此收集器的采样概率。它必须是 0 到 65535 之间的整数。值 0 对应于不采样任何数据包，而值 65535 对应于采样所有数据包。

* `set_id`: 整数，范围 1 到 4,294,967,295
  * 要向其发送数据包的收集器集的 32 位整数标识符。请参阅 ovs-vswitchd 数据库架构中的 `Flow_Sample_Collector_Set` 表。

* `external_ids`: 字符串对的映射
  * 请参阅本文档开头的“外部 ID”。

# Sample 表

此表描述了采样配置。其他表中的条目可能与 `Sample` 条目关联，以指示应如何生成样本。有关示例，请参阅 `ACL`。

## 汇总

* `collectors`
  * `Sample_Collector` 集合
* `metadata`
  * 整数，范围 1 到 4,294,967,295 (在表中必须唯一)

## 详细信息

* `collectors`: `Sample_Collector` 集合
  * 生成样本（例如 IPFIX）时要使用的 `Sample_Collector` 记录的引用列表。样本可以同时发送到多个收集器。

* `metadata`: 整数，范围 1 到 4,294,967,295 (在表中必须唯一)
  * 将用作每个样本中的观察点 ID。观察域 ID 将由 ovn-northd 生成，并包括逻辑数据路径键作为最低有效 24 位，以及采样应用程序类型（例如，丢弃调试）作为最高有效 8 位。

# Copp 表

此表用于定义控制平面保护策略，即，将 `Meter` 表中的条目与控制协议名称相关联。

## 汇总

* `name`
  * 字符串 (在表中必须唯一)
* `meters : arp`
  * 可选字符串
* `meters : arp-resolve`
  * 可选字符串
* `meters : dhcpv4-opts`
  * 可选字符串
* `meters : dhcpv6-opts`
  * 可选字符串
* `meters : dns`
  * 可选字符串
* `meters : event-elb`
  * 可选字符串
* `meters : icmp4-error`
  * 可选字符串
* `meters : icmp6-error`
  * 可选字符串
* `meters : igmp`
  * 可选字符串
* `meters : nd-na`
  * 可选字符串
* `meters : nd-ns`
  * 可选字符串
* `meters : nd-ns-resolve`
  * 可选字符串
* `meters : nd-ra-opts`
  * 可选字符串
* `meters : tcp-reset`
  * 可选字符串
* `meters : bfd`
  * 可选字符串
* `meters : reject`
  * 可选字符串
* `meters : svc-monitor`
  * 可选字符串
* `meters : dhcpv4-relay`
  * 可选字符串
* `external_ids`
  * 字符串对的映射

## 详细信息

* `name`: 字符串 (在表中必须唯一)
  * CoPP 名称。

* `meters : arp`: 可选字符串
  * 用于学习邻居的 ARP 数据包（请求/回复）的速率限制仪表。

* `meters : arp-resolve`: 可选字符串
  * 需要解析下一跳（通过 ARP）的数据包的速率限制仪表。

* `meters : dhcpv4-opts`: 可选字符串
  * 需要添加 DHCPv4 选项的数据包的速率限制仪表。

* `meters : dhcpv6-opts`: 可选字符串
  * 需要添加 DHCPv6 选项的数据包的速率限制仪表。

* `meters : dns`: 可选字符串
  * 需要回复的 DNS 查询数据包的速率限制仪表。

* `meters : event-elb`: 可选字符串
  * 空负载均衡器事件的速率限制仪表。

* `meters : icmp4-error`: 可选字符串
  * 需要回复 ICMP 错误的数据包的速率限制仪表。

* `meters : icmp6-error`: 可选字符串
  * 需要回复 ICMPv6 错误的数据包的速率限制仪表。

* `meters : igmp`: 可选字符串
  * IGMP 数据包的速率限制仪表。

* `meters : nd-na`: 可选字符串
  * 用于学习邻居的 ND 邻居通告数据包的速率限制仪表。

* `meters : nd-ns`: 可选字符串
  * 用于学习邻居的 ND 邻居请求数据包的速率限制仪表。

* `meters : nd-ns-resolve`: 可选字符串
  * 需要解析下一跳（通过 ND）的数据包的速率限制仪表。

* `meters : nd-ra-opts`: 可选字符串
  * 需要添加 ND 路由器通告选项的数据包的速率限制仪表。

* `meters : tcp-reset`: 可选字符串
  * 需要回复 TCP RST 数据包的数据包的速率限制仪表。

* `meters : bfd`: 可选字符串
  * BFD 数据包的速率限制仪表。

* `meters : reject`: 可选字符串
  * 触发拒绝操作的数据包的速率限制仪表。

* `meters : svc-monitor`: 可选字符串
  * 到达服务监视器 MAC 地址的数据包的速率限制仪表。

* `meters : dhcpv4-relay`: 可选字符串
  * 启用 DHCPv4 中继功能时，DHCPv4 中继数据包（请求/响应）的速率限制仪表。

* `external_ids`: 字符串对的映射
  * 请参阅本文档开头的“外部 ID”。

# Sampling_App 表

## 汇总

* `type`
  * 字符串，`acl-est`, `acl-new`, 或 `drop` 之一 (在表中必须唯一)
* `id`
  * 整数，范围 1 到 255
* 通用列:
  * `external_ids`
    * 字符串对的映射

## 详细信息

* `type`: 字符串，`acl-est`, `acl-new`, 或 `drop` 之一 (在表中必须唯一)
  * 要配置为采样的应用程序类型。目前支持的选项是："drop"、"acl-new"、"acl-est"。

* `id`: 整数，范围 1 到 255
  * 要为此类应用程序生成的样本中编码的标识符。此标识符用作样本观察域 ID 的一部分。

* **通用列**:

  * `external_ids`: 字符串对的映射
    * 请参阅本文档开头的“外部 ID”。

# Chassis_Template_Var 表

每个机箱一条记录，每个记录包含一个映射 `variables`，位于模板变量名称及其特定于该机箱的值之间。模板变量具有名称，并且在 OVN 集群中的不同管理程序上可能具有不同的值。例如，两行 `R1 = (.chassis=C1, variables={(N: V1)})` 和 `R2 = (.chassis=C2, variables={(N: V2)})` 将使在机箱 C1 和 C2 上运行的 ovn-controller 将标记 N 解释为 V1（在 C1 上）或 V2（在 C2 上）。用户可以从其他逻辑组件中引用模板变量，例如，在 ACL、QoS 或 `Logical_Router_Policy` 匹配中，或从 `Load_Balancer` VIP 和后端定义中。

如果在未定义该变量的机箱上引用模板变量，则在该机箱上运行的 ovn-controller 将仅将其解释为原始字符串字面量。

## 汇总

* `chassis`
  * 字符串 (在表中必须唯一)
* `variables`
  * 字符串对的映射
* 通用列:
  * `external_ids`
    * 字符串对的映射

## 详细信息

* `chassis`: 字符串 (在表中必须唯一)
  * 这组变量值适用的机箱。

* `variables`: 字符串对的映射
  * 给定机箱的一组变量值。

* **通用列**:

  * `external_ids`: 字符串对的映射
    * 请参阅本文档开头的“外部 ID”。
