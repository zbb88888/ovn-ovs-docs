# Chassis 设置

OVN 部署中的每个 Chassis 必须配置一个专用于 OVN 的 Open vSwitch 网桥，称为集成网桥。如果需要，系统启动脚本可以在启动 `ovn-controller` 之前创建此网桥。如果 `ovn-controller` 启动时此网桥不存在，它将使用下面建议的默认配置自动创建。集成网桥上的端口包括：

*   在任何 Chassis 上，OVN 用于维护逻辑网络连接的隧道端口。`ovn-controller` 添加、更新和删除这些隧道端口。

*   在管理程序上，任何要连接到逻辑网络的 VIF。对于通过软件模拟端口（如 TUN/TAP 或 VETH 对）连接的实例，管理程序本身通常会创建端口并将其插入集成网桥。对于通过代表端口（通常用于硬件卸载）连接的实例，`ovn-controller` 可以在 CMS 指示下查询 VIF 插件提供程序以进行代表端口查找，并将其插入集成网桥（有关更多信息，请参阅 `Documentation/topics/vif-plug-providers/vif-plug-providers.rst`）。在这两种情况下，都遵循 Open vSwitch 源代码树中 `Documentation/topics/integration.rst` 中描述的约定，以确保 OVN 逻辑端口和 VIF 之间的映射。（这是在支持 OVS 的管理程序上已经完成的现有集成工作。）

*   在网关上，用于逻辑网络连接的物理端口。系统启动脚本在启动 `ovn-controller` 之前将此端口添加到网桥。在更复杂的设置中，这可以是到另一个网桥的补丁端口，而不是物理端口。

其他端口不应连接到集成网桥。特别是，连接到底层网络的物理端口（相对于网关端口，即连接到逻辑网络的物理端口）不得连接到集成网桥。底层物理端口应改为连接到单独的 Open vSwitch 网桥（实际上，它们根本不需要连接到任何网桥）。

集成网桥应如下所述进行配置。每个设置的效果记录在 `ovs-vswitchd.conf.db(5)` 中：

*   `fail-mode=secure`
    避免在 `ovn-controller` 启动之前在隔离的逻辑网络之间交换数据包。有关更多信息，请参阅 `ovs-vsctl(8)` 中的“控制器故障设置”。

*   `other-config:disable-in-band=true`
    抑制集成网桥的带内控制流。无论如何，这些流出现都是不寻常的，因为 OVN 使用本地控制器（通过 Unix 域套接字）而不是远程控制器。但是，同一系统中的其他网桥可能有带内远程控制器，在这种情况下，这会抑制带内控制通常会设置的流。有关更多信息，请参阅文档。

集成网桥的惯用名称是 `br-int`，但也可能使用其他名称。
