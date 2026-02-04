# OVN DHCP 与其他高级命令

## DHCP 选项命令 (DHCP Options commands)

*   `dhcp-options-create cidr [key=value]`
    在 `DHCP_Options` 表中创建一个新的 DHCP 选项条目，指定 CIDR 和可选的外部 ID。

*   `dhcp-options-list`
    列出 DHCP 选项条目。

*   `dhcp-options-del dhcp-option`
    删除由 UUID 引用的 DHCP 选项条目。

*   `dhcp-options-set-options dhcp-option [key=value]...`
    为指定的 `dhcp-option` UUID 设置 DHCP 选项（如 `router`, `dns_server` 等）。

*   `dhcp-options-get-options dhcp-option`
    列出指定 `dhcp-option` 的配置选项。

## 端口组命令 (Port Group commands)

*   `pg-add group [port]...`
    创建一个名为 `group` 的新端口组，可选择添加端口。

*   `pg-set-ports group port...`
    设置端口组 `group` 中的端口列表（覆盖现有）。

*   `pg-get-ports group`
    获取端口组 `group` 中的端口。

*   `pg-del group`
    删除端口组 `group`。

## HA 机箱组命令 (HA Chassis Group commands)

*   `ha-chassis-group-add group`
    创建一个名为 `group` 的新 HA 机箱组。

*   `ha-chassis-group-del group`
    删除 HA 机箱组 `group`。

*   `ha-chassis-group-list [ha-chassis-group]`
    列出 HA 机箱组及其关联的 HA 机箱。

*   `ha-chassis-group-add-chassis group chassis priority`
    向 HA 机箱组 `group` 添加具有指定优先级的 `chassis`。
    *   优先级越高越优先。

*   `ha-chassis-group-remove-chassis group chassis`
    从 HA 机箱组 `group` 中移除 `chassis`。

## 控制平面保护策略命令 (Control Plane Protection Policy commands)

这些命令管理 `Copp` 表中配置的计量器，通过 `Logical_Switch` 或 `Logical_Router` 表中的 `copp` 列将它们链接到逻辑数据路径。这用于保护 `ovn-controller` 免受过多控制协议数据包的影响（如 ARP, ND, DNS, BFD 等）。

*   `copp-add name proto meter`
    将控制协议 `proto` 到 `meter` 的映射添加到名为 `name` 的 CoPP 策略中。
    *   `proto`: 如 `arp`, `nd_ns`, `dns` 等。

*   `copp-del name [proto]`
    从 CoPP 策略 `name` 中移除 `proto` 的映射。如果未指定 `proto`，则删除整个策略。

*   `copp-list name`
    显示 `name` 的当前 CoPP 策略。

*   `ls-copp-add name switch`
    将 CoPP 策略 `name` 应用到逻辑交换机 `switch`。

*   `lr-copp-add name router`
    将 CoPP 策略 `name` 应用到逻辑路由器 `router`。

## 镜像命令 (Mirror commands)

*   `mirror-add m type [index] filter dest`
    创建一个名为 `m` 的新镜像。
    *   `type`: `gre`, `erspan`, `local`。
    *   `index`: 隧道索引（用于 `gre` 或 `erspan`）。
    *   `filter`: 镜像源选择 (`from-lport`, `to-lport`, `both`)。
    *   `dest`: 目标 IP (GRE/ERSPAN) 或本地接口名称 (Local)。

*   `mirror-del m`
    删除镜像 `m`。

*   `mirror-list`
    列出镜像。
