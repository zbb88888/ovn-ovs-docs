# OVN 逻辑交换机端口命令 (Logical Switch Port Commands)

## 端口管理

*   `[--may-exist] lsp-add switch port`
    在 `switch` 上创建一个名为 `port` 的新逻辑交换机端口。

*   `[--may-exist] lsp-add switch port parent tag_request`
    在 `switch` 上创建一个名为 `port` 的逻辑交换机端口，它是 `parent` 的子端口，并用 VLAN ID `tag_request` 标识。
    *   `tag_request`: 必须在 0 到 4095 之间。如果为 0，`ovn-northd` 生成一个在 `parent` 范围内唯一的标记。
    *   这在虚拟化容器环境中有用，其中 Open vSwitch 没有直接连接到容器端口，必须与虚拟机的端口共享。

*   `[--if-exists] lsp-del port`
    删除 `port`。

*   `lsp-list switch`
    列出 `switch` 中的所有逻辑交换机端口。

*   `lsp-get-parent port`
    如果设置了，获取 `port` 的父端口。

*   `lsp-get-tag port`
    如果设置了，获取 `port` 流量的标记。

*   `lsp-get-ls port`
    获取端口所属的逻辑交换机。

## 地址与安全

*   `lsp-set-addresses port [address]...`
    设置与 `port` 关联的地址。每个地址应为以下之一：
    *   以太网地址，可选地后跟空格和一个或多个 IP 地址。
    *   `unknown`: OVN 将目标 MAC 地址不在任何逻辑端口地址列中的单播以太网数据包传递给地址为 `unknown` 的端口。
    *   `dynamic`: 让 `ovn-northd` 生成全局唯一的 MAC 地址，并从逻辑端口的子网中选择未使用的 IPv4 地址。
    *   `router`: 仅当逻辑交换机端口类型为 `router` 时接受。表示地址应从连接的逻辑路由器端口获取。

*   `lsp-get-addresses port`
    列出与 `port` 关联的所有地址。

*   `lsp-set-port-security port [addrs]...`
    设置与 `port` 关联的端口安全地址。
    *   限制逻辑端口可以发送数据包的源地址和接收数据包的目的地址。

*   `lsp-get-port-security port`
    列出与 `port` 关联的所有端口安全地址。

## 状态与类型

*   `lsp-get-up port`
    打印端口状态：`up` 或 `down`。

*   `lsp-set-enabled port state`
    设置端口的管理状态：`enabled` 或 `disabled`。禁用时，不允许流量进出。

*   `lsp-get-enabled port`
    打印端口的管理状态。

*   `lsp-set-type port type [peer=peer]`
    设置逻辑端口的类型：
    *   (空字符串): VM 或 VIF 接口。
    *   `router`: 连接到逻辑路由器。
    *   `switch`: 连接到另一个逻辑交换机（需要 `peer`）。
    *   `localnet`: 连接到本地可访问的网络。每个逻辑交换机只能附加一个 `localnet` 端口。
    *   `localport`: 连接到本地 VIF。流量不会通过隧道转发到其他机箱。
    *   `l2gateway`: 连接到物理网络。
    *   `vtep`: VTEP 网关上的逻辑交换机端口。

*   `lsp-get-type port`
    获取逻辑端口的类型。

## 选项与配置

*   `lsp-set-options port [key=value]...`
    设置逻辑端口的类型特定选项。

*   `lsp-get-options port`
    获取逻辑端口的类型特定选项。

*   `lsp-set-dhcpv4-options port dhcp_options`
    设置逻辑端口的 DHCPv4 选项（引用 DHCP_Options 表的 UUID）。

*   `lsp-get-dhcpv4-options port`
    获取配置的 DHCPv4 选项。

*   `lsp-set-dhcpv6-options port dhcp_options`
    设置逻辑端口的 DHCPv6 选项。

*   `lsp-get-dhcpv6-options port`
    获取配置的 DHCPv6 选项。

*   `lsp-attach-mirror port m`
    将镜像 `m` 附加到逻辑端口 `port`。

*   `lsp-detach-mirror port m`
    从逻辑端口 `port` 分离镜像 `m`。

## 转发组命令 (Forwarding Group Commands)

*   `[--liveness] fwd-group-add group switch vip vmac ports`
    创建一个名为 `group` 的新转发组，具有提供的 `vip` 和 `vmac`。
    *   `ports`: 放入转发组的逻辑交换机端口名称列表（如 `lsp1 lsp2`）。
    *   发往转发组虚拟 IP 的流量将负载均衡到所有子端口。
    *   `--liveness`: 指定时，子端口预期绑定到外部设备（如路由器），并应配置 BFD。

*   `[--if-exists] fwd-group-del group`
    删除转发组。

*   `fwd-group-list [switch]`
    列出所有现有的转发组。
