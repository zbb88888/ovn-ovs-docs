# VM 内容器接口的生命周期

OVN 通过将 OVN_NB 数据库中写入的信息转换为每个管理程序中的 OpenFlow 流来提供虚拟网络抽象。只有当 `ovn-controller` 是唯一可以修改 Open vSwitch 中流的实体时，才能提供多租户的安全虚拟网络。当 Open vSwitch 集成网桥驻留在管理程序中时，可以合理地假设在 VM 内运行的租户工作负载无法对 Open vSwitch 流进行任何更改。

如果基础设施提供商信任容器内的应用程序不会突破并修改 Open vSwitch 流，那么容器可以在管理程序中运行。当容器在 VM 内运行并且添加了 OVN 控制器流的 Open vSwitch 集成网桥驻留在同一 VM 中时，情况也是如此。对于上述两种情况，工作流程与上一节（“VIF 的生命周期”）中用示例解释的工作流程相同。

本节讨论在 VM 中创建容器且 Open vSwitch 集成网桥驻留在管理程序内时，容器接口 (CIF) 的生命周期。在这种情况下，即使容器应用程序突破，其他租户也不会受到影响，因为在 VM 内运行的容器无法修改 Open vSwitch 集成网桥中的流。

当在 VM 内创建多个容器时，会有多个 CIF 与之关联。与这些 CIF 关联的网络流量需要到达运行在管理程序中的 Open vSwitch 集成网桥，以便 OVN 支持虚拟网络抽象。OVN 还应该能够区分来自不同 CIF 的网络流量。有两种方法可以区分 CIF 的网络流量。

一种方法是为每个 CIF 提供一个 VIF（1:1 模型）。这意味着管理程序中可能有很多网络设备。由于管理所有 VIF 需要额外的 CPU 周期，这会减慢 OVS。这也意味着在 VM 中创建容器的实体也应该能够在管理程序中创建相应的 VIF。

第二种方法是为所有 CIF 提供单个 VIF（1:多 模型）。然后，OVN 可以通过写入每个数据包的标记来区分来自不同 CIF 的网络流量。OVN 使用此机制，并使用 VLAN 作为标记机制。

1.  当创建 VM 的同一 CMS 或拥有该 VM 的租户甚至不同于最初创建 VM 的 CMS 的容器编排系统在 VM 内生成容器时，CIF 的生命周期开始。无论实体是谁，它都需要知道与 VM 的网络接口关联的 `vif-id`，容器接口的网络流量预期将通过该接口。创建容器接口的实体还需要在该 VM 内选择一个未使用的 VLAN。

2.  容器生成实体（直接或通过管理底层基础设施的 CMS）通过向 `Logical_Switch_Port` 表添加一行来更新 OVN 北向数据库以包含新 CIF。在新行中，`name` 是任何唯一标识符，`parent_name` 是 CIF 网络流量预期通过的 VM 的 `vif-id`，`tag` 是标识该 CIF 网络流量的 VLAN 标记。

3.  `ovn-northd` 接收 OVN 北向数据库更新。反过来，它通过向 OVN 南向数据库的 `Logical_Flow` 表添加行来反映新端口，并创建一个新的 `Binding` 表行并填充除非标识 Chassis 的列之外的所有列，从而对 OVN 南向数据库进行相应的更新。

4.  在每个管理程序上，`ovn-controller` 订阅 `Binding` 表中的更改。当 `ovn-northd` 创建一个新行并在 `Binding` 表的 `parent_port` 列中包含一个值时，其 OVN 集成网桥在 `external_ids:iface-id` 中具有相同 `vif-id` 值的管理程序中的 `ovn-controller` 会更新本地管理程序的 OpenFlow 表，以便正确处理具有特定 VLAN 标记的进出 VIF 的数据包。之后，它更新 `Binding` 的 `chassis` 列以反映物理位置。

5.  人们只能在底层网络准备就绪后才能启动容器内的应用程序。为了支持这一点，`ovn-northd` 注意到 `Binding` 表中更新的 `chassis` 列，并更新 OVN 北向数据库 `Logical_Switch_Port` 表中的 `up` 列以指示 CIF 现在已启动。负责启动容器应用程序的实体查询此值并启动应用程序。

6.  最终，创建并启动容器的实体停止它。该实体通过 CMS（或直接）删除其在 `Logical_Switch_Port` 表中的行。

7.  `ovn-northd` 接收 OVN 北向更新，并反过来相应地更新 OVN 南向数据库，方法是从 OVN 南向数据库 `Logical_Flow` 表中删除或更新与现已销毁的 CIF 相关的行。它还删除该 CIF 在 `Binding` 表中的行。

8.  在每个管理程序上，`ovn-controller` 接收 `ovn-northd` 在上一步中进行的 `Logical_Flow` 表更新。`ovn-controller` 更新 OpenFlow 表以反映更新。
