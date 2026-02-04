# VTEP 网关的生命周期

网关是在逻辑网络的 OVN 管理部分和物理 VLAN 之间转发流量的 Chassis，将基于隧道的逻辑网络扩展到物理网络。

以下步骤经常参考 OVN 和 VTEP 数据库模式的详细信息。有关这些数据库的完整故事，请分别参阅 `ovn-sb(5)`、`ovn-nb(5)` 和 `vtep(5)`。

1.  VTEP 网关的生命周期始于管理员在 VTEP 数据库中将 VTEP 网关注册为 `Physical_Switch` 表条目。连接到此 VTEP 数据库的 `ovn-controller-vtep` 将识别新的 VTEP 网关，并在 OVN_Southbound 数据库中为其创建一个新的 `Chassis` 表条目。

2.  然后，管理员可以创建一个新的 `Logical_Switch` 表条目，并将 VTEP 网关端口上的特定 vlan 绑定到任何 VTEP 逻辑交换机。一旦 VTEP 逻辑交换机绑定到 VTEP 网关，`ovn-controller-vtep` 将检测到它并将其名称添加到 OVN_Southbound 数据库中 `Chassis` 表的 `vtep_logical_switches` 列。注意，VTEP 逻辑交换机的 `tunnel_key` 列在创建时未填充。当相应的 vtep 逻辑交换机绑定到 OVN 逻辑网络时，`ovn-controller-vtep` 将设置该列。

3.  现在，管理员可以使用 CMS 将 VTEP 逻辑交换机添加到 OVN 逻辑网络。为此，CMS 必须首先在 OVN_Northbound 数据库中创建一个新的 `Logical_Switch_Port` 表条目。然后，必须将此条目的 `type` 列设置为 "vtep"。接下来，还必须在 `options` 列中指定 `vtep-logical-switch` 和 `vtep-physical-switch` 键，因为多个 VTEP 网关可以连接到同一个 VTEP 逻辑交换机。接下来，此逻辑端口的 `addresses` 列必须设置为 "unknown"，它将在逻辑交换机入口管道的 "ls_in_l2_lkup" 阶段添加优先级为 0 的条目。因此，OVN 未记录 mac 的流量将通过 `Logical_Switch_Port` 进入物理网络。

4.  OVN_Northbound 数据库中新创建的逻辑端口及其配置将作为新的 `Port_Binding` 表条目传递到 OVN_Southbound 数据库。`ovn-controller-vtep` 将识别更改并将逻辑端口绑定到相应的 VTEP 网关 Chassis。不允许将同一 VTEP 逻辑交换机绑定到不同的 OVN 逻辑网络，并且会在日志中生成警告。

5.  除了绑定到 VTEP 网关 Chassis 之外，`ovn-controller-vtep` 还会将 VTEP 逻辑交换机的 `tunnel_key` 列更新为绑定 OVN 逻辑网络的相应 `Datapath_Binding` 表条目的 `tunnel_key`。

6.  接下来，`ovn-controller-vtep` 将继续对 OVN_Northbound 数据库中 `Port_Binding` 中的配置更改做出反应，并更新 VTEP 数据库中的 `Ucast_Macs_Remote` 表。这允许 VTEP 网关了解将来自扩展外部网络的单播流量转发到何处。

7.  最终，当管理员从 VTEP 数据库注销 VTEP 网关时，VTEP 网关的生命周期结束。`ovn-controller-vtep` 将识别该事件并删除 OVN_Southbound 数据库中的所有相关配置（`Chassis` 表条目和端口绑定）。

8.  当 `ovn-controller-vtep` 终止时，OVN_Southbound 数据库和 VTEP 数据库中的所有相关配置将被清理，包括所有注册的 VTEP 网关的 `Chassis` 表条目及其端口绑定，以及所有 `Ucast_Macs_Remote` 表条目和 `Logical_Switch` 隧道密钥。
