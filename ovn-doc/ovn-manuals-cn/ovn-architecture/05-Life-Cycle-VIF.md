# VIF 的生命周期

单独展示表及其模式很难理解。这里有一个例子。

管理程序上的 VIF 是连接到该管理程序上直接运行的 VM 或容器的虚拟网络接口（这不同于在 VM 内运行的容器的接口）。

此示例中的步骤经常参考 OVN 和 OVN 北向数据库模式的详细信息。有关这些数据库的完整故事，请分别参阅 `ovn-sb(5)` 和 `ovn-nb(5)`。

1.  当 CMS 管理员使用 CMS 用户界面或 API 创建新 VIF 并将其添加到交换机（由 OVN 实现为逻辑交换机）时，VIF 的生命周期开始。CMS 更新其自己的配置。这包括将唯一、持久的标识符 `vif-id` 和以太网地址 `mac` 与 VIF 关联。

2.  CMS 插件通过向 `Logical_Switch_Port` 表添加一行来更新 OVN 北向数据库以包含新 VIF。在新行中，`name` 为 `vif-id`，`mac` 为 `mac`，`switch` 指向 OVN 逻辑交换机的 `Logical_Switch` 记录，其他列也进行了适当的初始化。

3.  `ovn-northd` 接收 OVN 北向数据库更新。反过来，它通过向 OVN 南向数据库 `Logical_Flow` 表添加行来反映新端口，从而对 OVN 南向数据库进行相应的更新，例如添加一个流以识别发往新端口 MAC 地址的数据包应传递给它，并更新传递广播和组播数据包的流以包含新端口。它还在 `Binding` 表中创建一条记录，并填充除非标识 Chassis 的列之外的所有列。

4.  在每个管理程序上，`ovn-controller` 接收 `ovn-northd` 在上一步中进行的 `Logical_Flow` 表更新。只要拥有 VIF 的 VM 处于关机状态，`ovn-controller` 就做不了什么；例如，它无法安排向 VIF 发送数据包或从 VIF 接收数据包，因为 VIF 实际上并不存在于任何地方。

5.  最终，用户启动拥有 VIF 的 VM。在 VM 启动的管理程序上，管理程序和 Open vSwitch 之间的集成（在 Open vSwitch 源代码树中的 `Documentation/topics/integration.rst` 中描述）将 VIF 添加到 OVN 集成网桥，并将 `vif-id` 存储在 `external_ids:iface-id` 中以指示接口是新 VIF 的实例化。（这些代码都不是 OVN 中的新代码；这是在支持 OVS 的管理程序上已经完成的现有集成工作。）

6.  在 VM 启动的管理程序上，`ovn-controller` 注意到新接口中的 `external_ids:iface-id`。作为响应，在 OVN 南向数据库中，它更新将逻辑端口从 `external_ids:iface-id` 链接到管理程序的行的 `Binding` 表的 `chassis` 列。之后，`ovn-controller` 更新本地管理程序的 OpenFlow 表，以便正确处理进出 VIF 的数据包。

7.  一些 CMS 系统（包括 OpenStack）仅在网络准备就绪时才完全启动 VM。为了支持这一点，`ovn-northd` 注意到 `Binding` 表中该行的 `chassis` 列已更新，并通过更新 OVN 北向数据库 `Logical_Switch_Port` 表中的 `up` 列来向上推送此信息，以指示 VIF 现在已启动。CMS 如果使用此功能，则可以通过允许 VM 继续执行来做出反应。

8.  在除 VIF 所在管理程序之外的每个管理程序上，`ovn-controller` 注意到 `Binding` 表中完全填充的行。这为 `ovn-controller` 提供了逻辑端口的物理位置，因此每个实例都会更新其交换机的 OpenFlow 表（基于 OVN DB `Logical_Flow` 表中的逻辑数据路径流），以便可以通过隧道正确处理进出 VIF 的数据包。

9.  最终，用户关闭拥有 VIF 的 VM。在 VM 关闭的管理程序上，VIF 从 OVN 集成网桥中删除。

10. 在 VM 关闭的管理程序上，`ovn-controller` 注意到 VIF 已被删除。作为响应，它删除逻辑端口 `Binding` 表中的 `Chassis` 列内容。

11. 在每个管理程序上，`ovn-controller` 注意到逻辑端口 `Binding` 表行中的空 `Chassis` 列。这意味着 `ovn-controller` 不再知道逻辑端口的物理位置，因此每个实例都会更新其 OpenFlow 表以反映这一点。

12. 最终，当任何人都不再需要 VIF（或其整个 VM）时，管理员使用 CMS 用户界面或 API 删除 VIF。CMS 更新其自己的配置。

13. CMS 插件通过删除 `Logical_Switch_Port` 表中的行，从 OVN 北向数据库中删除 VIF。

14. `ovn-northd` 接收 OVN 北向更新，并反过来相应地更新 OVN 南向数据库，方法是从 OVN 南向数据库 `Logical_Flow` 表和 `Binding` 表中删除或更新与现已销毁的 VIF 相关的行。

15. 在每个管理程序上，`ovn-controller` 接收 `ovn-northd` 在上一步中进行的 `Logical_Flow` 表更新。`ovn-controller` 更新 OpenFlow 表以反映更新，尽管可能没有什么可做的，因为 VIF 在上一步中从 `Binding` 表中删除时已经变得不可达。
