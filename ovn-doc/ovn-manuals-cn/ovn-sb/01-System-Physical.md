# 系统与物理网络 (System and Physical Network)

本文档涵盖了 OVN 南向数据库中的系统配置和物理网络相关表，包括 `SB_Global`、`Chassis`、`Chassis_Private` 和 `Encap`。

## SB_Global 表

OVN 系统的南向配置。此表必须恰好有一行。

### 摘要 (Summary)

*   **Status**:
    *   **nb_cfg**: integer
*   **Common Columns**:
    *   **external_ids**: map of string-string pairs
    *   **options**: map of string-string pairs
*   **Common options**:
    *   **options**: map of string-string pairs
    *   **options : enable_chassis_nb_cfg_update**: optional string
*   **Options for configuring BFD**:
    *   **options : bfd-min-rx**: optional string
    *   **options : bfd-decay-min-rx**: optional string
    *   **options : bfd-min-tx**: optional string
    *   **options : bfd-mult**: optional string
    *   **options : debug_drop_domain_id**: optional string
    *   **options : debug_drop_collector_set**: optional string
*   **Options for configuring ovn-sbctl**:
    *   **options : sbctl_probe_interval**: optional string
*   **Connection Options**:
    *   **connections**: set of Connections
    *   **ssl**: optional SSL
*   **Security Configurations**:
    *   **ipsec**: boolean

### 详情 (Details)

#### Status

此列允许客户端跟踪系统的整体配置状态。

*   **nb_cfg**: integer
    配置的序列号。当 CMS 或 `ovn-nbctl` 更新北向数据库时，它会递增北向数据库中 `NB_Global` 表的 `nb_cfg` 列。反过来，当 `ovn-northd` 更新南向数据库以使其与这些更改保持最新时，它会将此列更新为相同的值。

#### Common Columns

*   **external_ids**: map of string-string pairs
    参见本文档开头的 External IDs。

*   **options**: map of string-string pairs

#### Common options

*   **options**: map of string-string pairs
    此列提供通用键/值设置。支持的选项在下面单独描述。

*   **options : enable_chassis_nb_cfg_update**: optional string
    如果设置为 false，`ovn-controller` 将不再更新 OVN_Southbound 数据库的 `Chassis_Private` 表中的 `nb_cfg` 列。它仍将更新本地 OVS 集成网桥中的 `external_ids:ovn-nb-cfg`。

#### Options for configuring BFD

当 `ovn-controller` 在隧道接口上配置 BFD 时，应用这些选项。

*   **options : bfd-min-rx**: optional string
    在隧道接口上配置 BFD 时使用的 BFD 选项 min-rx 值。

*   **options : bfd-decay-min-rx**: optional string
    在隧道接口上配置 BFD 时使用的 BFD 选项 decay-min-rx 值。

*   **options : bfd-min-tx**: optional string
    在隧道接口上配置 BFD 时使用的 BFD 选项 min-tx 值。

*   **options : bfd-mult**: optional string
    在隧道接口上配置 BFD 时使用的 BFD 选项 mult 值。

*   **options : debug_drop_domain_id**: optional string
    如果设置为 8 位数字，并且还配置了 `debug_drop_collector_set`，则 `ovn-controller` 将向每个非来自包含 'drop' 操作的逻辑流的流添加采样操作。`observation_domain_id` 字段的最高 8 位将是 `debug_drop_domain_id` 中指定的位。`observation_domain_id` 字段的最低 24 位将为零。

    `observation_point_id` 将设置为 OpenFlow 表号。

*   **options : debug_drop_collector_set**: optional string
    如果设置为 32 位数字，`ovn-controller` 将向每个非来自包含 'drop' 操作的逻辑流的流添加采样操作。采样操作将具有指定的 `collector_set_id`。该值必须与 `ovs-actions(7)` 中描述的本地 OVS 配置的值匹配。

#### Options for configuring ovn-sbctl

当 `ovn-sbctl` 连接到 OVN 南向数据库时，应用这些选项。

*   **options : sbctl_probe_interval**: optional string
    `ovn-sbctl` 实用程序连接到 OVN 南向数据库的不活动探测间隔，以毫秒为单位。如果该值为零，则禁用连接 keepalive 功能。

    如果该值非零，则将其强制为至少 1000 毫秒的值。

    如果该值小于零，则 `ovn-sbctl` 的默认不活动探测间隔将保持不变（120000 毫秒）。

#### Connection Options

*   **connections**: set of Connections
    Open vSwitch 数据库服务器应连接到或应侦听的数据库客户端，以及有关如何配置这些连接的选项。有关更多信息，请参阅 `Connection` 表。

*   **ssl**: optional SSL
    全局 SSL/TLS 配置。

#### Security Configurations

*   **ipsec**: boolean
    隧道加密配置。如果此列设置为 true，则所有 OVN 隧道都将使用 IPsec 加密。

## Chassis 表

此表中的每一行代表物理网络中的虚拟机管理程序或网关（机箱）。每个机箱通过 `ovn-controller`/`ovn-controller-vtep` 添加和更新自己的行，并保留其余行的副本以确定如何到达其他虚拟机管理程序。

当机箱正常关闭时，它应该删除自己的行。（这并不重要，因为无论行是否存在，机箱上托管的资源都同样不可达。）如果机箱永久关闭而不删除其行，最终需要某种手动或自动清理；我们可以根据需要设计一个流程。

### 摘要 (Summary)

*   **name**: string (must be unique within table)
*   **hostname**: string
*   **nb_cfg**: integer
*   **other_config : ovn-bridge-mappings**: optional string
*   **other_config : datapath-type**: optional string
*   **other_config : iface-types**: optional string
*   **other_config : ovn-cms-options**: optional string
*   **other_config : is-interconn**: optional string
*   **other_config : is-remote**: optional string
*   **transport_zones**: set of strings
*   **other_config : ovn-chassis-mac-mappings**: optional string
*   **other_config : port-up-notif**: optional string
*   **Common Columns**:
    *   **external_ids**: map of string-string pairs
*   **Encapsulation Configuration**:
    *   **encaps**: set of 1 or more Encaps
*   **Gateway Configuration**:
    *   **vtep_logical_switches**: set of strings

### 详情 (Details)

*   **name**: string (must be unique within table)
    OVN 不规定机箱名称的特定格式。`ovn-controller` 使用 Open_vSwitch 数据库的 Open_vSwitch 表中的 `external_ids:system-id` 填充此列。`ovn-controller-vtep` 使用 hardware_vtep 数据库的 Physical_Switch 表中的 `name` 填充此列。

*   **hostname**: string
    机箱的主机名（如果适用）。`ovn-controller` 将使用其运行的主机的主机名填充此列。`ovn-controller-vtep` 将使此列为空。

*   **nb_cfg**: integer
    已弃用。此列由 `Chassis_Private` 表的 `nb_cfg` 列替换。

*   **other_config : ovn-bridge-mappings**: optional string
    `ovn-controller` 使用其配置为使用的网桥映射集填充此键。其他应用程序应将此键视为只读。有关更多信息，请参阅 `ovn-controller(8)`。

*   **other_config : datapath-type**: optional string
    `ovn-controller` 使用在 Open_vSwitch 数据库的 Bridge 表的 `datapath_type` 列中配置的数据路径类型填充此键。其他应用程序应将此键视为只读。有关更多信息，请参阅 `ovn-controller(8)`。

*   **other_config : iface-types**: optional string
    `ovn-controller` 使用在 Open_vSwitch 数据库的 Open_vSwitch 表的 `iface_types` 列中配置的接口类型填充此键。其他应用程序应将此键视为只读。有关更多信息，请参阅 `ovn-controller(8)`。

*   **other_config : ovn-cms-options**: optional string
    `ovn-controller` 使用在 Open_vSwitch 数据库的 Open_vSwitch 表的 `external_ids:ovn-cms-options` 列中配置的选项集填充此键。有关更多信息，请参阅 `ovn-controller(8)`。

*   **other_config : is-interconn**: optional string
    `ovn-controller` 使用在 Open_vSwitch 数据库的 Open_vSwitch 表的 `external_ids:ovn-is-interconn` 列中配置的设置填充此键。如果设置为 true，则机箱用作互连网关。有关更多信息，请参阅 `ovn-controller(8)`。

*   **other_config : is-remote**: optional string
    `ovn-ic` 将此键设置为 true，用于从互连南向数据库学习的远程互连网关机箱。有关更多信息，请参阅 `ovn-ic(8)`。

*   **transport_zones**: set of strings
    `ovn-controller` 使用在 Open_vSwitch 数据库的 Open_vSwitch 表的 `external_ids:ovn-transport-zones` 列中配置的传输区域填充此键。有关更多信息，请参阅 `ovn-controller(8)`。

*   **other_config : ovn-chassis-mac-mappings**: optional string
    `ovn-controller` 使用在 Open_vSwitch 数据库的 Open_vSwitch 表的 `external_ids:ovn-chassis-mac-mappings` 列中配置的选项集填充此键。有关更多信息，请参阅 `ovn-controller(8)`。

*   **other_config : port-up-notif**: optional string
    当 `ovn-controller` 支持 `Port_Binding.up` 时，它会用 true 填充此键。

#### Common Columns

*   **external_ids**: map of string-string pairs
    这些列的总体用途在本文档开头的 Common Columns 下进行了描述。

#### Encapsulation Configuration

OVN 使用封装在机箱之间传输逻辑数据平面数据包。

*   **encaps**: set of 1 or more Encaps
    指向支持的封装配置，以将逻辑数据平面数据包传输到此机箱。每个条目都是描述配置的 `Encap` 记录。

#### Gateway Configuration

网关是在逻辑网络的 OVN 管理部分和物理 VLAN 之间转发流量的机箱，将基于隧道的逻辑网络扩展到物理网络。网关通常是不托管 VM 的专用节点，并将由 `ovn-controller-vtep` 控制。

*   **vtep_logical_switches**: set of strings
    存储此网关机箱连接的所有 VTEP 逻辑交换机名称。`options:vtep-physical-switch` 等于 Chassis 名称且 `options:vtep-logical-switch` 值在 Chassis `vtep_logical_switches` 中的 `Port_Binding` 表条目将与此 Chassis 关联。

## Chassis_Private 表

此表中的每一行维护每个机箱的私有数据，这些数据仅由拥有该机箱的机箱（只写）和 `ovn-northd` 访问，不由任何其他机箱访问。出于性能考虑，这些数据存储在这个单独的表中，而不是 Chassis 表中：机箱可以有条件地监视此表中的行，以便每个机箱仅获取其自己行的更新通知，从而避免在大规模部署中不必要的机箱私有数据更新泛滥。

### 摘要 (Summary)

*   **name**: string (must be unique within table)
*   **chassis**: optional weak reference to Chassis
*   **nb_cfg**: integer
*   **nb_cfg_timestamp**: integer
*   **Common Columns**:
    *   **external_ids**: map of string-string pairs

### 详情 (Details)

*   **name**: string (must be unique within table)
    拥有这些机箱私有数据的机箱的名称。

*   **chassis**: optional weak reference to Chassis
    对拥有这些机箱私有数据的机箱的 Chassis 表的引用。

*   **nb_cfg**: integer
    配置的序列号。当 `ovn-controller` 从南向数据库的内容更新机箱的配置时，它将 `SB_Global` 表中的 `nb_cfg` 复制到此列中。

*   **nb_cfg_timestamp**: integer
    `ovn-controller` 完成处理对应于 `nb_cfg` 的更改时的时间戳。

#### Common Columns

*   **external_ids**: map of string-string pairs
    这些列的总体用途在本文档开头的 Common Columns 下进行了描述。

## Encap 表

Chassis 表中的 `encaps` 列引用此表中的行，以标识 OVN 如何将逻辑数据平面数据包传输到此机箱。每个机箱通过 `ovn-controller(8)` 或 `ovn-controller-vtep(8)` 添加和更新自己的行，并保留其余行的副本以确定如何到达其他机箱。

### 摘要 (Summary)

*   **type**: string, either geneve or vxlan
*   **options**: map of string-string pairs
*   **options : csum**: optional string, either true or false
*   **options : dst_port**: optional string, containing an integer
*   **options : is_default**: optional string, either true or false
*   **ip**: string
*   **chassis_name**: string

### 详情 (Details)

*   **type**: string, either geneve or vxlan
    用于向此机箱传输数据包的封装。虚拟机管理程序和网关必须使用 geneve 或 vxlan。

*   **options**: map of string-string pairs
    用于配置封装的选项，可能是特定于类型的。

*   **options : csum**: optional string, either true or false
    csum 指示此机箱是否可以以合理的性能传输和接收包含校验和的数据包。它提示向此机箱传输数据的发送者，他们应该使用校验和来保护 OVN 元数据。`ovn-controller` 使用 Open_vSwitch 数据库的 Open_vSwitch 表的 `external_ids:ovn-encap-csum` 列中定义的值填充此键。其他应用程序应将此键视为只读。有关更多信息，请参阅 `ovn-controller(8)`。

    就性能而言，在不支持封装硬件卸载的基于 Linux 的主机上运行时，校验和实际上在大多数常见情况下会显着增加吞吐量（对于批量流量约为 60%）。原因是通常所有 NIC 都能卸载传输和接收的 TCP/UDP 校验和（被视为普通数据包而不是隧道）。好处来自于接收端，其中经过验证的外部校验和可用于额外验证内部校验和（如 TCP），这反过来允许堆栈的其余部分更有效地处理数据包聚合。

    并非所有设备都能看到这种好处。最明显的例外是硬件 VTEP。这些设备设计为不在其交换引擎中缓冲整个数据包，因此无法有效地计算或验证完整的数据包校验和。此外，某些版本的 Linux 内核在存在校验和的情况下无法充分利用封装 NIC 卸载。（不过这实际上是一个非常狭窄的极端情况：早期的 Linux 版本根本不支持封装卸载，而后来的版本很好地支持卸载和校验和。）

    对于硬件 VTEP，csum 默认为 false，对于所有其他情况，默认为 true。

    此选项适用于 geneve 和 vxlan 封装。

*   **options : dst_port**: optional string, containing an integer
    如果设置，则覆盖 UDP 目标端口。

*   **options : is_default**: optional string, either true or false
    当一个机箱有多个具有不同 IP 的封装时，此选项指示该封装是否是与 Open_vSwitch 数据库的 Open_vSwitch 表的 `external_ids:ovn-encap-ip-default` 列中的 IP 匹配的默认封装。

*   **ip**: string
    封装隧道端点的 IPv4 地址。

*   **chassis_name**: string
    创建此封装的机箱的名称。
