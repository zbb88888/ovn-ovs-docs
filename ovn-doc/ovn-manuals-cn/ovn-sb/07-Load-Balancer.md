# Load_Balancer 表

每一行代表一个负载均衡器。

## 汇总

*   `name`
    *   字符串
*   `vips`
    *   字符串对的映射
*   `protocol`
    *   可选字符串，`sctp`, `tcp`, 或 `udp` 之一
*   `datapaths`
    *   `Datapath_Binding` 集合
*   `datapath_group`
    *   可选的 `Logical_DP_Group`
*   `ls_datapath_group`
    *   可选的 `Logical_DP_Group`
*   `lr_datapath_group`
    *   可选的 `Logical_DP_Group`
*   Load_Balancer 选项:
    *   `options : hairpin_snat_ip`
        *   可选字符串
*   通用列:
    *   `external_ids`
        *   字符串对的映射

## 详细信息

*   `name`: 字符串
    *   负载均衡器的名称。除了为与 `ovn-nb` 数据库的人机交互提供便利之外，此名称没有特殊含义或用途。

*   `vips`: 字符串对的映射
    *   与此负载均衡器关联的虚拟 IP 地址（以及可选的端口号，以 `:` 为分隔符）及其对应的端点 IP 地址（以及可选的端口号，以 `:` 为分隔符，以逗号分隔）的映射。

*   `protocol`: 可选字符串，`sctp`, `tcp`, 或 `udp` 之一
    *   有效的协议是 `tcp`、`udp` 或 `sctp`。当 `vips` 列中提供端口号时，此列很有用。如果此列为空并且 `vips` 列中提供了端口号，则 OVN 假定协议为 `tcp`。

*   `datapaths`: `Datapath_Binding` 集合
    *   此负载均衡器适用的数据路径。

*   `datapath_group`: 可选的 `Logical_DP_Group`
    *   已弃用。此负载均衡器适用的数据路径组。这意味着同一个负载均衡器适用于组中的所有数据路径。

*   `ls_datapath_group`: 可选的 `Logical_DP_Group`
    *   此负载均衡器适用的数据路径组。这意味着同一个负载均衡器适用于组中的所有数据路径。

*   `lr_datapath_group`: 可选的 `Logical_DP_Group`
    *   此负载均衡器适用的逻辑路由器数据路径组。这意味着同一个负载均衡器适用于组中的所有数据路径。

*   **Load_Balancer 选项**:

    *   `options : hairpin_snat_ip`: 可选字符串
        *   用于负载平衡后已发夹的数据包的源 IP。此值由 `ovn-northd` 自动填充。

*   **通用列**:

    *   这些列的总体用途在本文档开头的“通用列”下进行了描述。

    *   `external_ids`: 字符串对的映射
