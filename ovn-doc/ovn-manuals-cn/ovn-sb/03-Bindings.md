# Datapath_Binding 表

此表中的每一行代表一个逻辑数据路径，它在与其关联的 `Port_Binding` 表中的端口之间实现逻辑管道。实际上，给定逻辑数据路径中的管道实现逻辑交换机或逻辑路由器。

此表中行的主要目的是为逻辑数据路径提供物理绑定。逻辑数据路径没有物理位置，因此其物理绑定信息是有限的：只有 `tunnel_key`。此表中的其余数据不影响数据包转发。

## 汇总

*   `tunnel_key`
    *   整数，范围 1 到 16,777,215 (在表中必须唯一)
*   `load_balancers`
    *   uuid 集合
*   OVN_Northbound 关系:
    *   `type`
        *   可选字符串，`logical-router` 或 `logical-switch`
    *   `nb_uuid`
        *   可选 uuid
    *   `external_ids : interconn-ts`
        *   可选字符串
    *   命名:
        *   `external_ids : name`
            *   可选字符串
        *   `external_ids : name2`
            *   可选字符串
*   通用列:
    *   `external_ids`
        *   字符串对的映射

## 详细信息

*   `tunnel_key`: 整数，范围 1 到 16,777,215 (在表中必须唯一)
    *   逻辑数据路径绑定到的隧道键值。`ovn-architecture(7)` 中的隧道封装部分描述了如何为每种支持的封装构造隧道键。

*   `load_balancers`: uuid 集合
    *   不再使用；为了模式的向后兼容性而保留。

*   **OVN_Northbound 关系**:

    *   `Datapath_Binding` 中的每一行都与某个逻辑数据路径相关联。`ovn-northd` 使用这些键来跟踪逻辑数据路径与 `OVN_Northbound` 数据库中概念的关联。

    *   `type`: 可选字符串，`logical-router` 或 `logical-switch`
        *   这表示此南向 `Datapath_Binding` 所代表的北向逻辑数据路径的类型。此值的可能值为：
            *   `logical-switch`
            *   `logical-router`

    *   `nb_uuid`: 可选 uuid
        *   这是此南向 `Datapath_Binding` 所代表的相应北向逻辑数据路径的 UUID。

    *   `external_ids : interconn-ts`: 可选字符串
        *   对于表示用于互连的传输交换机的逻辑交换机的逻辑数据路径，`ovn-northd` 在此键中存储 `OVN_Northbound` 数据库中相应 `Logical_Switch` 行的 `external_ids` 列的相同 `interconn-ts` 键的值。

    *   **命名**:

    *   `ovn-northd` 从 `OVN_Northbound` 数据库中的名称字段复制这些内容，可以是从 `Logical_Router` 表中的 `name` 和 `external_ids:neutron:router_name`，也可以是从 `Logical_Switch` 表中的 `name` 和 `external_ids:neutron:network_name`。

    *   `external_ids : name`: 可选字符串
        *   逻辑数据路径的名称。

    *   `external_ids : name2`: 可选字符串
        *   逻辑数据路径的另一个名称。

*   **通用列**:

    *   这些列的总体用途在本文档开头的“通用列”下进行了描述。

    *   `external_ids`: 字符串对的映射

# Port_Binding 表

此表中的每一行将逻辑端口绑定到一个实现。对于大多数逻辑端口，这意味着绑定到某个物理位置，例如通过将逻辑端口绑定到属于在特定管理程序上运行的 VM 的 VIF。其他逻辑端口，例如逻辑补丁端口，可以在没有特定物理位置的情况下实现，但它们的绑定仍然通过此表中的行表示。

对于 `OVN_Northbound` 数据库中的每个 `Logical_Switch_Port` 记录，`ovn-northd` 在此表中创建一个记录。`ovn-northd` 填充并维护除 `chassis` 和 `virtual_parent` 列之外的每一列，它在每条新记录中将这两个列留空。

`ovn-controller`/`ovn-controller-vtep` 填充标识位于其管理程序/网关上的逻辑端口的记录的 `chassis` 列，`ovn-controller`/`ovn-controller-vtep` 进而通过监视本地管理程序的 `Open_vSwitch` 数据库来发现这一点，该数据库通过 `IntegrationGuide.rst` 中描述的约定标识逻辑端口。（例外情况是类型为 `l3gateway` 的 `Port_Binding` 记录，其位置由 `ovn-northd` 通过此表中的 `options:l3gateway-chassis` 列标识。`ovn-controller` 仍然负责填充 `chassis` 列。）

`ovn-controller` 还填充类型为 `virtual` 的记录的 `virtual_parent` 列。

当机箱正常关闭时，它应该清理它先前填充的 `chassis` 列。（这并不重要，因为无论是否存在行，托管在机箱上的资源都同样无法访问。）为了处理 VM 在一个机箱上突然关闭，然后在另一个机箱上再次启动的情况，`ovn-controller`/`ovn-controller-vtep` 必须用新信息覆盖 `chassis` 列。

## 汇总

*   核心特性:
    *   `datapath`
        *   `Datapath_Binding`
    *   `logical_port`
        *   字符串 (在表中必须唯一)
    *   `encap`
        *   可选的 `Encap` 弱引用
    *   `additional_encap`
        *   `Encap` 的弱引用集合
    *   `chassis`
        *   可选的 `Chassis` 弱引用
    *   `additional_chassis`
        *   `Chassis` 的弱引用集合
    *   `gateway_chassis`
        *   `Gateway_Chassis` 集合
    *   `ha_chassis_group`
        *   可选的 `HA_Chassis_Group`
    *   `up`
        *   可选布尔值
    *   `tunnel_key`
        *   整数，范围 1 到 32,767
    *   `mac`
        *   字符串集合
    *   `port_security`
        *   字符串集合
    *   `mirror_port`
        *   可选字符串
    *   `type`
        *   字符串
    *   `requested_chassis`
        *   可选的 `Chassis` 弱引用
    *   `requested_additional_chassis`
        *   `Chassis` 的弱引用集合
*   `mirror_rules`
    *   `Mirror` 的弱引用集合
*   补丁选项:
    *   `options : peer`
        *   可选字符串
    *   `nat_addresses`
        *   字符串集合
*   L3 网关选项:
    *   `options : peer`
        *   可选字符串
    *   `options : l3gateway-chassis`
        *   可选字符串
    *   `nat_addresses`
        *   字符串集合
*   Localnet 选项:
    *   `options : network_name`
        *   可选字符串
    *   `tag`
        *   可选整数，范围 1 到 4,095
*   L2 网关选项:
    *   `options : network_name`
        *   可选字符串
    *   `options : l2gateway-chassis`
        *   可选字符串
    *   `tag`
        *   可选整数，范围 1 到 4,095
*   VTEP 选项:
    *   `options : vtep-physical-switch`
        *   可选字符串
    *   `options : vtep-logical-switch`
        *   可选字符串
*   VMI (或 VIF) 选项:
    *   `options : requested-chassis`
        *   可选字符串
    *   `options : activation-strategy`
        *   可选字符串
    *   `options : additional-chassis-activated`
        *   可选字符串
    *   `options : iface-id-ver`
        *   可选字符串
    *   `options : qos_min_rate`
        *   可选字符串
    *   `options : qos_max_rate`
        *   可选字符串
    *   `options : qos_burst`
        *   可选字符串
    *   `options : qos_physical_network`
        *   可选字符串
    *   `options : qdisc_queue_id`
        *   可选字符串，包含整数，范围 1 到 61,440
*   分布式网关端口选项:
    *   `options : chassis-redirect-port`
        *   可选字符串
*   机箱重定向选项:
    *   `options : distributed-port`
        *   可选字符串
    *   `options : redirect-type`
        *   可选字符串
    *   `options : always-redirect`
        *   可选字符串
*   动态路由:
    *   `options : dynamic-routing-redistribute-local-only`
        *   可选字符串，`true` 或 `false`
    *   `options : dynamic-routing-no-learning`
        *   可选字符串，`true` 或 `false`
*   嵌套容器:
    *   `parent_port`
        *   可选字符串
    *   `tag`
        *   可选整数，范围 1 到 4,095
*   虚拟端口:
    *   `virtual_parent`
        *   可选字符串
*   命名:
    *   `external_ids : name`
        *   可选字符串
*   通用列:
    *   `external_ids`
        *   字符串对的映射

## 详细信息

*   **核心特性**:

    *   `datapath`: `Datapath_Binding`
        *   逻辑端口所属的逻辑数据路径。

    *   `logical_port`: 字符串 (在表中必须唯一)
        *   一个逻辑端口。对于逻辑交换机端口，这取自 `OVN_Northbound` 数据库的 `Logical_Switch_Port` 表中的名称。对于逻辑路由器端口，这取自 `OVN_Northbound` 数据库的 `Logical_Router_port` 表中的名称。（这意味着逻辑交换机端口和路由器端口名称在 OVN 部署中不得共享名称。）OVN 没有为逻辑端口 ID 规定特定的格式。

    *   `encap`: 可选的 `Encap` 弱引用
        *   指向首选封装配置，以将逻辑数据平面数据包传输到此机箱。该条目是对 `Encap` 记录的引用。

    *   `additional_encap`: `Encap` 的弱引用集合
        *   指向首选封装配置，以将逻辑数据平面数据包传输到此附加机箱。该条目是对 `Encap` 记录的引用。另请参阅 `additional_chassis`。

    *   `chassis`: 可选的 `Chassis` 弱引用
        *   此列的含义取决于 `type` 列的值。这是每种类型的含义：

        *   (空字符串)
            *   逻辑端口的物理位置。为了成功识别机箱，此列必须是 `Chassis` 记录。这由 `ovn-controller` 填充。

        *   `vtep`
            *   `hardware_vtep` 网关的物理位置。为了成功识别机箱，此列必须是 `Chassis` 记录。这由 `ovn-controller-vtep` 填充。

        *   `localnet`
            *   始终为空。`localnet` 端口在每个连接到相应物理网络的机箱上实现。

        *   `localport`
            *   始终为空。`localport` 端口存在于每个机箱上。

        *   `l3gateway`
            *   L3 网关的物理位置。为了成功识别机箱，此列必须是 `Chassis` 记录。这由 `ovn-controller` 根据此表中 `options:l3gateway-chassis` 列的值填充。

        *   `l2gateway`
            *   此 L2 网关的物理位置。为了成功识别机箱，此列必须是 `Chassis` 记录。这由 `ovn-controller` 根据此表中 `options:l2gateway-chassis` 列的值填充。

    *   `additional_chassis`: `Chassis` 的弱引用集合
        *   此列的含义与 `chassis` 相同。该列用于跟踪逻辑端口的附加物理位置。与常规（空类型）端口绑定一起使用。

    *   `gateway_chassis`: `Gateway_Chassis` 集合
        *   `Gateway_Chassis` 列表。
        *   这仅应为类型设置为 `chassisredirect` 的端口填充。此列定义用作网关的机箱列表，流量将通过这些网关重定向。

    *   `ha_chassis_group`: 可选的 `HA_Chassis_Group`
        *   这仅应为类型设置为 `chassisredirect` 的端口填充。此列定义 HA 机箱组，其中包含用作网关的 HA 机箱列表，流量将通过这些网关重定向。

    *   `up`: 可选布尔值
        *   只要安装了此 `Port_Binding` 所需的所有 OVS 流，此值就会设置为 true。这由 `ovn-controller` 填充。

    *   `tunnel_key`: 整数，范围 1 到 32,767
        *   在隧道协议数据包内携带的密钥（例如 Geneve TLV）字段中表示逻辑端口的数字。
        *   隧道 ID 必须在逻辑数据路径的范围内唯一。

    *   `mac`: 字符串集合
        *   此列是一个误称，因为它可能包含 MAC 地址和 IP 地址。它是从北向数据库中 `Logical_Switch_Port` 表的 `addresses` 列复制的。它遵循与该列相同的格式。

    *   `port_security`: 字符串集合
        *   此列控制允许连接到逻辑端口的主机（“主机”）从哪些地址发送数据包以及允许它接收哪些地址的数据包。如果此列为空，则允许所有地址。
        *   它是从北向数据库中 `Logical_Switch_Port` 表的 `port_security` 列复制的。它遵循与该列相同的格式。

    *   `mirror_port`: 可选字符串
        *   指向 `lport` 镜像类型的镜像目标端口。

    *   `type`: 字符串
        *   此逻辑端口的类型。逻辑端口可用于将其他类型的连接建模到 OVN 逻辑交换机中。定义了以下类型：

        *   (空字符串)
            *   VM (或 VIF) 接口。

        *   `patch`
            *   一对逻辑端口之一，其作用就像通过跳线连接一样。用于连接两个逻辑数据路径，例如将逻辑路由器连接到逻辑交换机或另一个逻辑路由器。

        *   `l3gateway`
            *   一对逻辑端口之一，其作用就像通过跨越多个机箱的跳线连接一样。用于将逻辑交换机与网关路由器（仅驻留在特定机箱上）连接。

        *   `localnet`
            *   从具有相应网桥映射的 `ovn-controller` 实例连接到本地可访问的网络。一个逻辑交换机可以连接多个 `localnet` 端口。此类型用于对现有网络的直接连接进行建模。在这种情况下，每个机箱应该只有一个物理网络的映射。注意：上面所说的并不意味着机箱不能插入多个物理网络，只要它们属于不同的交换机。

        *   `localport`
            *   连接到本地 VIF。到达 `localport` 的流量永远不会通过隧道转发到另一个机箱。这些端口存在于每个机箱上，并且在所有机箱中具有相同的地址。这用于对运行在每个管理程序上的本地服务的连接进行建模。

        *   `l2gateway`
            *   到物理网络的 L2 连接。此 `Port_Binding` 绑定到的机箱将作为由 `options:network_name` 命名的网络的 L2 网关。

        *   `vtep`
            *   VTEP 网关机箱上的逻辑交换机端口。为了让 OVN 控制器正确识别此端口，还必须定义 `options:vtep-physical-switch` 和 `options:vtep-logical-switch`。

        *   `chassisredirect`
            *   一个逻辑端口，代表一个特定实例，绑定到一个特定机箱，属于一个原本分布式的父端口（例如类型为 `patch`）。`chassisredirect` 端口永远不应用作 `inport`。当入口管道设置 `outport` 时，它可以将值设置为类型为 `chassisredirect` 的逻辑端口。这将导致数据包被定向到特定机箱以执行出口管道。在出口管道的开始处，`outport` 将被重置为分布式端口的值。

        *   `virtual`
            *   代表一个具有虚拟 ip 的逻辑端口。此虚拟 ip 可以在逻辑端口（称为虚拟父级）上配置。

    *   `requested_chassis`: 可选的 `Chassis` 弱引用
        *   存在此列是为了让 `ovn-controller` 可以有效地监视所有发往它的 `Port_Binding` 记录，并且是对 `options:requested-chassis` 选项的补充。该选项仍然是必需的，以便 `ovn-controller` 可以在指向的机箱当前不存在时检查 CMS 意图，例如当 `ovn-controller` 在没有传递 `-restart` 参数的情况下停止时就会发生这种情况。此列必须是 `Chassis` 记录。当定义了 `options:requested-chassis` 并且包含与现有机箱的名称或主机名匹配的字符串时，这由 `ovn-northd` 填充。另请参阅 `requested_additional_chassis`。

    *   `requested_additional_chassis`: `Chassis` 的弱引用集合
        *   存在此列是为了让 `ovn-controller` 可以有效地监视所有发往它的 `Port_Binding` 记录，并且是列出多个机箱时对 `options:requested-chassis` 选项的补充。此列必须是 `Chassis` 记录列表。当 `options:requested-chassis` 定义为机箱名称或主机名列表时，这由 `ovn-northd` 填充。另请参阅 `requested_chassis`。

    *   `mirror_rules`: `Mirror` 的弱引用集合
        *   适用于端口绑定的镜像规则。请参阅 `Mirror` 表。

    *   **补丁选项**:

    *   这些选项适用于类型为 `patch` 的逻辑端口。

    *   `options : peer`: 可选字符串
        *   补丁另一侧的 `Port_Binding` 记录中的 `logical_port`。命名的 `logical_port` 必须在其自己的 `peer` 选项中指定此 `logical_port`。也就是说，两个补丁逻辑端口必须具有反转的 `logical_port` 和 `peer` 值。

    *   `nat_addresses`: 字符串集合
        *   MAC 地址后跟 SNAT 和 DNAT 外部 IP 地址列表，后跟 `is_chassis_resident("lport")`，其中 `lport` 是应用相应 NAT 规则的同一机箱上的逻辑端口名称。这用于从 `lport` 驻留的机箱通过 `localnet` 发送 SNAT 和 DNAT 外部 IP 地址的免费 ARP。例如：`80:fa:5b:06:72:b7 158.36.44.22 158.36.44.24 is_chassis_resident("foo1")`。这将导致从逻辑端口 "foo1" 驻留的机箱生成 IP 地址 158.36.44.22 和 158.36.44.24 的免费 ARP，MAC 地址为 80:fa:5b:06:72:b7。

    *   **L3 网关选项**:

    *   这些选项适用于类型为 `l3gateway` 的逻辑端口。

    *   `options : peer`: 可选字符串
        *   `l3gateway` 端口另一侧的 `Port_Binding` 记录中的 `logical_port`。命名的 `logical_port` 必须在其自己的 `peer` 选项中指定此 `logical_port`。也就是说，两个 `l3gateway` 逻辑端口必须具有反转的 `logical_port` 和 `peer` 值。

    *   `options : l3gateway-chassis`: 可选字符串
        *   端口所在的机箱。

    *   `nat_addresses`: 字符串集合
        *   `l3gateway` 端口的 MAC 地址，后跟 SNAT 和 DNAT 外部 IP 地址列表。这用于通过 `localnet` 发送 SNAT 和 DNAT 外部 IP 地址的免费 ARP。例如：`80:fa:5b:06:72:b7 158.36.44.22 158.36.44.24`。这将导致生成 IP 地址 158.36.44.22 和 158.36.44.24 的免费 ARP，MAC 地址为 80:fa:5b:06:72:b7。这在 OVS 2.8 及更高版本中使用。

    *   **Localnet 选项**:

    *   这些选项适用于类型为 `localnet` 的逻辑端口。

    *   `options : network_name`: 可选字符串
        *   必需。`ovn-controller` 使用配置条目 `ovn-bridge-mappings` 来确定如何连接到此网络。`ovn-bridge-mappings` 是映射到提供对该网络访问的本地 OVS 网桥的网络名称列表。配置 `ovn-bridge-mappings` 的示例如下：
        *   `$ ovs-vsctl set open . external-ids:ovn-bridge-mappings=physnet1:br-eth0,physnet2:br-eth1`
        *   当逻辑交换机连接了 `localnet` 端口时，每个可能连接了该逻辑交换机的本地 vif 的机箱都必须配置网桥映射以到达该 `localnet`。到达 `localnet` 端口的流量永远不会通过隧道转发到另一个机箱。如果逻辑交换机中有多个 `localnet` 端口，则每个机箱应仅具有其中一个物理网络的单个网桥映射。注意：在多个 `localnet` 端口的情况下，为了在位于具有不同结构连接的不同机箱上的所有 VIF 之间提供互连，结构应实现段之间某种形式的路由。

    *   `tag`: 可选整数，范围 1 到 4,095
        *   如果设置，则指示该端口代表连接到本地可访问网络上的特定 VLAN。VLAN ID 用于匹配传入流量，也添加到传出流量。

    *   **L2 网关选项**:

    *   这些选项适用于类型为 `l2gateway` 的逻辑端口。

    *   `options : network_name`: 可选字符串
        *   必需。`ovn-controller` 使用配置条目 `ovn-bridge-mappings` 来确定如何连接到此网络。`ovn-bridge-mappings` 是映射到提供对该网络访问的本地 OVS 网桥的网络名称列表。配置 `ovn-bridge-mappings` 的示例如下：
        *   `$ ovs-vsctl set open . external-ids:ovn-bridge-mappings=physnet1:br-eth0,physnet2:br-eth1`
        *   当逻辑交换机连接了 `l2gateway` 端口时，`l2gateway` 端口绑定到的机箱必须配置网桥映射以到达由 `network_name` 标识的网络。

    *   `options : l2gateway-chassis`: 可选字符串
        *   必需。端口所在的机箱。

    *   `tag`: 可选整数，范围 1 到 4,095
        *   如果设置，则指示网关连接到物理网络上的特定 VLAN。VLAN ID 用于匹配传入流量，也添加到传出流量。

    *   **VTEP 选项**:

    *   这些选项适用于类型为 `vtep` 的逻辑端口。

    *   `options : vtep-physical-switch`: 可选字符串
        *   必需。VTEP 网关的名称。

    *   `options : vtep-logical-switch`: 可选字符串
        *   必需。VTEP 网关连接的逻辑交换机名称。当类型为 `vtep` 时必须设置。

    *   **VMI (或 VIF) 选项**:

    *   这些选项适用于类型为 (空字符串) 的逻辑端口

    *   `options : requested-chassis`: 可选字符串
        *   如果设置，则标识允许绑定此端口的特定机箱（按名称或主机名）。使用此选项将防止在实时迁移期间尝试绑定同一端口的两个机箱之间发生抖动。如果端口意外在多个机箱上创建，它还可以防止由于错误配置而导致的类似抖动。
        *   如果设置为逗号分隔的列表，则第一个条目标识主机箱，其余条目是允许绑定同一端口的一个或多个附加机箱。
        *   当为端口设置多个机箱，并且逻辑交换机通过 `localnet` 端口连接到外部网络时，将对端口强制执行隧道传输，以保证发往该端口的数据包传递到其所有位置。这具有 MTU 含义，因为用于隧道的网络必须具有比 `localnet` 更大的 MTU 才能实现稳定的连接。

    *   `options : activation-strategy`: 可选字符串
        *   如果在 `requested-chassis` 中设置了多个机箱时使用，则指定所有附加机箱的激活策略。默认情况下，不使用激活策略，这意味着附加端口位置可立即使用。该选项支持逗号分隔的列表，您可以在其中组合 3 种协议："rarp"、"garp" 和 "na"。当设置任何协议时，端口将被阻止进行入口和出口通信，直到从新位置发送指定的协议数据包。激活策略在虚拟机的实时迁移方案中很有用。

    *   `options : additional-chassis-activated`: 可选字符串
        *   当设置了 `activation-strategy` 时，此选项指示端口已使用指定的策略激活。

    *   `options : iface-id-ver`: 可选字符串
        *   如果设置，则仅当 `Open_vSwitch` 数据库的 `Interface` 表中的 `external_ids` 列中配置了相同的键和值时，`ovn-controller` 才会绑定此端口。

    *   `options : qos_min_rate`: 可选字符串
        *   如果设置，则指示从此接口发送的数据的最小保证速率，单位为 bit/s。

    *   `options : qos_max_rate`: 可选字符串
        *   如果设置，则指示从此接口发送的数据的最大速率，单位为 bit/s。流量将根据此限制进行整形。

    *   `options : qos_burst`: 可选字符串
        *   如果设置，则指示从此接口发送的数据的最大突发大小，单位为 bits。

    *   `options : qos_physical_network`: 可选字符串
        *   如果设置，则指示将应用流量整形的出口网络名称。

    *   `options : qdisc_queue_id`: 可选字符串，包含整数，范围 1 到 61,440
        *   指示物理设备上的队列号。这与 `struct ofp_action_enqueue` 中 OpenFlow 使用的 `queue_id` 相同。

    *   **分布式网关端口选项**:

    *   这些选项适用于类型为 `chasssisredirect` 的逻辑端口的分布式父端口。

    *   `options : chassis-redirect-port`: 可选字符串
        *   如果此端口是机箱重定向端口的分布式父端口，则为此端口派生的机箱重定向端口的名称。

    *   **机箱重定向选项**:

    *   这些选项适用于类型为 `chassisredirect` 的逻辑端口。

    *   `options : distributed-port`: 可选字符串
        *   此 `chassisredirect` 端口代表其特定实例的分布式端口的名称。

    *   `options : redirect-type`: 可选字符串
        *   该值是从此端口的分布式父端口的 `OVN_Northbound` 数据库的 `Logical_Router_Port` 表中的 `options` 列复制的。

    *   `options : always-redirect`: 可选字符串
        *   一个布尔选项，如果此机箱重定向端口的分布式父端口不需要分布式处理，则设置为 true。

    *   **动态路由**:

    *   `options : dynamic-routing-redistribute-local-only`: 可选字符串，`true` 或 `false`
        *   仅当 `options:dynamic-routing` 设置为 true 时才相关。
        *   这控制 `ovn-controller` 是否仅在绑定了 `tracked_port` 的机箱上通告 `Advertised_Route` 记录。默认值：false。

    *   `options : dynamic-routing-no-learning`: 可选字符串，`true` 或 `false`
        *   仅当 `options:dynamic-routing` 设置为 true 时才相关。
        *   此选项禁用向 `Learned_Route` 添加路由，并且还将删除学习到的路由。

    *   **嵌套容器**:

    *   这些列支持嵌套在 VM 内的容器。具体来说，当 `type` 为空且 `logical_port` 标识在 VM 内生成的容器的接口时使用它们。对于直接在管理程序上运行的容器或 VM，它们为空。

    *   `parent_port`: 可选字符串
        *   这取自 `OVN_Northbound` 数据库的 `Logical_Switch_Port` 表中的 `parent_name`。

    *   `tag`: 可选整数，范围 1 到 4,095
        *   标识与该容器的网络接口关联的网络流量中的 VLAN 标记。
        *   当 `type` 为 `localnet`（参见上面的 Localnet 选项）或 `l2gateway`（参见上面的 L2 网关选项）时，此列用于不同的目的。

    *   **虚拟端口**:

    *   `virtual_parent`: 可选字符串
        *   当执行 OVN 操作 `bind_vport` 时，`ovn-controller` 使用 `OVN_Northbound` 数据库的 `Logical_Switch_Port` 表中的 `options:virtual-parents` 中的值之一设置此列。`ovn-controller` 在使用其机箱 ID 执行此操作时也会设置 `chassis` 列。
        *   仅当类型为 "virtual" 时，`ovn-controller` 才会设置此列。

    *   **命名**:

    *   `external_ids : name`: 可选字符串
        *   对于逻辑交换机端口，如果它是非空字符串，`ovn-northd` 会从 `OVN_Northbound` 数据库的 `Logical_Switch_Port` 表中的 `external_ids:neutron:port_name` 复制此内容。
        *   对于逻辑交换机端口，`ovn-northd` 目前不设置此键。

    *   **通用列**:

    *   `external_ids`: 字符串对的映射
        *   请参阅本文档开头的“外部 ID”。
        *   `ovn-northd` 程序使用 `OVN_Northbound` 数据库的 `Logical_Switch_Port` 和 `Logical_Router_Port` 表的 `external_ids` 列中的所有条目填充此列。

# MAC_Binding 表

此表中的每一行指定从 IP 地址到已通过 ARP（对于 IPv4）或邻居发现（对于 IPv6）发现的以太网地址的绑定。此表主要用于发现物理网络上的绑定，因为虚拟机的 IP 到 MAC 绑定通常静态填充到 `Port_Binding` 表中。

此表表示函数关系：`MAC_Binding(logical_port, ip) = mac`。

概括地说，逻辑路由器的 MAC 绑定的生命周期如下所示：

1.  在管理程序 1 上，逻辑路由器确定数据包应转发到其路由器端口之一上的 IP 地址 A。它使用其逻辑流表确定 A 缺少静态 IP 到 MAC 绑定，并使用 `get_arp` 操作确定它缺少动态 IP 到 MAC 绑定。

2.  使用 OVN 逻辑 `arp` 操作，逻辑路由器生成并向路由器端口发送广播 ARP 请求。它丢弃 IP 数据包。

3.  连接到路由器端口的逻辑交换机将 ARP 请求传递给其所有端口。（仅将其传递给没有静态 IP 到 MAC 绑定的端口可能是有意义的，但这这也可能是令人惊讶的行为。）

4.  连接到逻辑交换机的管理程序 2（可能与管理程序 1 相同）上的主机或 VM 拥有有问题的 IP 地址。它组成一个 ARP 回复并将其单播到逻辑路由器端口的以太网地址。

5.  逻辑交换机将 ARP 回复传递给逻辑路由器端口。

6.  逻辑路由器流表执行 `put_arp` 操作。为了记录 IP 到 MAC 绑定，`ovn-controller` 向 `MAC_Binding` 表添加一行。

7.  在管理程序 1 上，`ovn-controller` 从 OVN 南向数据库接收更新的 `MAC_Binding` 表。下一个通过逻辑路由器发往 A 的数据包将直接发送到绑定的以太网地址。

## 汇总

*   `logical_port`
    *   字符串
*   `ip`
    *   字符串
*   `mac`
    *   字符串
*   `timestamp`
    *   整数
*   `datapath`
    *   `Datapath_Binding`

## 详细信息

*   `logical_port`: 字符串
    *   发现绑定的逻辑端口。

*   `ip`: 字符串
    *   绑定的 IP 地址。

*   `mac`: 字符串
    *   IP 绑定到的以太网地址。

*   `timestamp`: 整数
    *   添加或更新 MAC 绑定时的时间戳（以毫秒为单位）。在此列之前存在的记录将为 0。

*   `datapath`: `Datapath_Binding`
    *   逻辑端口所属的逻辑数据路径。

# FDB 表

此表主要用于学习在 VIF（或启用了 `localnet_learn_fdb` 的 `localnet` 端口）上观察到的 MAC，该 VIF 属于 `OVN_Northbound` 中端口安全性已禁用且地址集为 `unknown` 的 `Logical_Switch_Port` 记录。如果 `Logical_Switch_Port` 记录上禁用了端口安全性，则 OVN 应允许来自 VIF 的具有任何源 mac 的流量。如果数据包的 `eth.dst` 已被学习，此表将用于将数据包传递到 VIF。

## 汇总

*   `mac`
    *   字符串
*   `dp_key`
    *   整数，范围 1 到 16,777,215
*   `port_key`
    *   整数，范围 1 到 16,777,215
*   `timestamp`
    *   整数

## 详细信息

*   `mac`: 字符串
    *   学习到的 mac 地址。

*   `dp_key`: 整数，范围 1 到 16,777,215
    *   学习此 FDB 的数据路径的键。

*   `port_key`: 整数，范围 1 到 16,777,215
    *   学习此 FDB 的端口绑定的键。

*   `timestamp`: 整数
    *   添加或更新 FDB 时的时间戳（以毫秒为单位）。在此列之前存在的记录将为 0。

# Static_MAC_Binding 表

每条记录代表逻辑路由器的 `Static_MAC_Binding` 条目。

## 汇总

*   `logical_port`
    *   字符串
*   `ip`
    *   字符串
*   `mac`
    *   字符串
*   `override_dynamic_mac`
    *   布尔值
*   `datapath`
    *   `Datapath_Binding`

## 详细信息

*   `logical_port`: 字符串
    *   绑定的逻辑路由器端口。

*   `ip`: 字符串
    *   绑定的 IP 地址。

*   `mac`: 字符串
    *   IP 绑定到的以太网地址。

*   `override_dynamic_mac`: 布尔值
    *   覆盖动态学习的 MAC。

*   `datapath`: `Datapath_Binding`
    *   逻辑路由器端口所属的逻辑数据路径。
