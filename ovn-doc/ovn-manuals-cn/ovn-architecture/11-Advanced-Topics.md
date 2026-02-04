# 高级主题

## 外部逻辑端口的原生 OVN 服务

为了向外部的云资源支持 OVN 原生服务（如 DHCP/IPv6 RA/DNS 查找），OVN 支持外部逻辑端口。

以下是可以使用外部端口的一些用例。

*   连接到 SR-IOV 网卡的 VM - 来自这些 VM 的流量绕过内核堆栈，本地 `ovn-controller` 不绑定这些端口，无法提供原生服务。

*   当 CMS 支持配置裸机服务器时。

如果 CMS 在 OVN 北向数据库中完成了以下配置，OVN 将提供原生服务。

*   在 `Logical_Switch_Port` 中创建一行，配置 `addresses` 列并将 `type` 设置为 `external`。

*   配置了 `ha_chassis_group` 列。

*   属于 HA chassis 组的 HA chassis 配置了 `ovn-bridge-mappings` 并且具有适当的 L2 连接，以便它可以从这些外部资源接收 DHCP 和其他相关请求数据包。

*   此端口的 `Logical_Switch` 具有 `localnet` 端口。

*   通过像为普通逻辑端口配置那样配置 DHCP 和其他选项来启用原生 OVN 服务。

建议为逻辑交换机的所有外部端口使用相同的 HA chassis 组。否则，当不同的 chassis 提供原生服务时，物理交换机可能会看到 MAC 翻动问题。例如，当支持原生 DHCPv4 服务时，源自不同端口的 DHCPv4 服务器 mac（在 `DHCP_Options` 表的 `options:server_mac` 列中配置）可能会导致 MAC 翻动问题。如果没有为逻辑交换机的所有外部端口设置相同的 HA chassis 组，逻辑路由器 IP 的 MAC 也可能会翻动。

## 安全

### 南向数据库的基于角色的访问控制

为了提供额外的安全性，防止 OVN chassis 以某种方式受到损害，从而允许恶意软件对南向数据库状态进行任意修改并因此破坏 OVN 网络，为南向数据库提供了基于角色的访问控制（有关其他详细信息，请参阅 `ovsdb-server(1)`）。

基于角色的访问控制 (RBAC) 的实现需要在 OVSDB 模式中添加两个表：`RBAC_Role` 表（按角色名称索引，并将给定角色可修改的各种表的名称映射到包含该角色详细权限信息的权限表中的各个行）以及权限表本身（由包含以下信息的行组成）：

*   **Table Name** (表名)
    关联表的名称。此列主要作为人类阅读此表内容的辅助。

*   **Auth Criteria** (授权标准)
    一组字符串，包含列的名称（或包含 string:string 映射的列的 column:key 对）。要修改、插入或删除的行中至少有一个列或 column:key 值的内容必须等于尝试对该行执行操作的客户端的 ID，以便通过授权检查。如果授权标准为空，则禁用授权检查，并且该角色的所有客户端都将被视为已授权。

*   **Insert/Delete** (插入/删除)
    行插入/删除权限；布尔值，指示是否允许对关联表进行行的插入和删除。如果为 true，则允许授权客户端插入和删除行。

*   **Updatable Columns** (可更新列)
    一组字符串，包含授权客户端可以更新或更改的列或 column:key 对的名称。仅当客户端的授权检查通过并且要修改的所有列都包含在这组可修改列中时，才允许修改行内的列。

OVN 南向数据库的 RBAC 配置由 `ovn-northd` 维护。启用 RBAC 后，仅允许修改 `Chassis`, `Encap`, `Port_Binding` 和 `MAC_Binding` 表，并且限制如下：

*   **Chassis**
    *   授权：客户端 ID 必须匹配 chassis 名称。
    *   插入/删除：允许授权的行插入和删除。
    *   更新：授权后可以修改 `nb_cfg`, `external_ids`, `encaps` 和 `vtep_logical_switches` 列。

*   **Encap**
    *   授权：客户端 ID 必须匹配 chassis 名称。
    *   插入/删除：允许行插入和行删除。
    *   更新：可以修改 `type`, `options` 和 `ip` 列。

*   **Port_Binding**
    *   授权：禁用（所有客户端都被视为已授权。未来的增强功能可能会添加列（或 `external_ids` 的键）以控制允许哪些 chassis 绑定每个端口。
    *   插入/删除：不允许行插入/删除（`ovn-northd` 维护此表中的行）。
    *   更新：仅允许修改 `chassis` 列。

*   **MAC_Binding**
    *   授权：禁用（所有客户端都被视为已授权）。
    *   插入/删除：允许行插入/删除。
    *   更新：`ovn-controller` 可以修改 `logical_port`, `ip`, `mac` 和 `datapath` 列。

*   **IGMP_Group**
    *   授权：禁用（所有客户端都被视为已授权）。
    *   插入/删除：允许行插入/删除。
    *   更新：`ovn-controller` 可以修改 `address`, `chassis`, `datapath` 和 `ports` 列。

为 `ovn-controller` 连接到南向数据库启用 RBAC 需要以下步骤：

1.  为每个 chassis 创建 SSL/TLS 证书，并将证书 CN 字段设置为 chassis 名称（例如，对于具有 `external-ids:system-id=chassis-1` 的 chassis，通过命令 `ovs-pki -u req+sign chassis-1 switch`）。

2.  配置每个 `ovn-controller` 在连接到南向数据库时使用 SSL/TLS（例如，通过 `ovs-vsctl set open . external-ids:ovn-remote=ssl:x.x.x.x:6642`）。

3.  使用 `ovn-controller` 角色配置南向数据库 SSL/TLS 远程连接（例如，通过 `ovn-sbctl set-connection role=ovn-controller pssl:6642`）。

### 使用 IPsec 加密隧道流量

OVN 隧道流量经过物理路由器和交换机。这些物理设备可能是不可信的（公共网络中的设备）或可能已受到损害。启用隧道流量加密可以防止流量数据被监控和操纵。

隧道流量使用 IPsec 加密。CMS 设置北向 `NB_Global` 表中的 `ipsec` 列以启用或禁用 IPsec 加密。如果 `ipsec` 为 true，则所有 OVN 隧道都将被加密。如果 `ipsec` 为 false，则没有 OVN 隧道被加密。

当 CMS 更新北向 `NB_Global` 表中的 `ipsec` 列时，`ovn-northd` 将该值复制到南向 `SB_Global` 表中的 `ipsec` 列。每个 chassis 中的 `ovn-controller` 监视南向数据库并相应地设置 OVS 隧道接口的选项。OVS 隧道接口选项由 `ovs-monitor-ipsec` 守护进程监视，该守护进程配置 IKE 守护进程以建立 IPsec 连接。

Chassis 使用证书相互验证。如果隧道另一端出示由受信任 CA 签名的证书并且公用名 (CN) 与预期的 chassis 名称匹配，则身份验证成功。基于角色的访问控制 (RBAC) 中使用的 SSL/TLS 证书可用于 IPsec。或者使用 `ovs-pki` 创建不同的证书。证书必须是 x.509 版本 3，并且 CN 字段和 subjectAltName 字段设置为 chassis 名称。

在启用 IPsec 之前，需要在每个 chassis 中安装 CA 证书、chassis 证书和私钥。有关设置基于 CA 的 IPsec 身份验证的信息，请参阅 `ovs-vswitchd.conf.db(5)`。

## 设计决策

### 隧道封装

通常，OVN 用以下三个元数据注释它从一个管理程序发送到另一个管理程序的逻辑网络数据包，这些元数据以特定于封装的方式编码：

*   24 位逻辑数据路径标识符，来自 OVN 南向 `Datapath_Binding` 表中的 `tunnel_key` 列。

*   15 位逻辑入口端口标识符。ID 0 保留供 OVN 内部使用。ID 1 到 32767（含）可以分配给逻辑端口（请参阅 OVN 南向 `Port_Binding` 表中的 `tunnel_key` 列）。

*   16 位逻辑出口端口标识符。ID 0 到 32767 的含义与逻辑入口端口相同。ID 32768 到 65535（含）可以分配给逻辑组播组（请参阅 OVN 南向 `Multicast_Group` 表中的 `tunnel_key` 列）。

当在集群中的任何管理程序上启用 VXLAN 时，数据路径和出口端口标识符范围减少到 12 位。这样做是因为只有 Geneve 提供大空间用于元数据（每个数据包超过 32 位）。具有减少范围的模式称为 VXLAN 模式。为了适应 VXLAN，可用的 24 位拆分如下：

*   12 位逻辑数据路径标识符，源自 OVN 南向 `Datapath_Binding` 表中的 `tunnel_key` 列。

*   12 位逻辑出口端口标识符。ID 0 到 2047 用于单播输出端口。ID 2048 到 4095（含）可以分配给逻辑组播组（请参阅 OVN 南向 `Multicast_Group` 表中的 `tunnel_key` 列）。对于组播组隧道密钥，使用特殊的映射方案在通过 VXLAN 隧道发送数据包之前将内部 OVN 16 位密钥转换为 12 位值，并在从 VXLAN 隧道接收数据包时将 12 位隧道密钥转换回 16 位值。

*   没有逻辑入口端口标识符。

当在集群中启用 VXLAN 隧道时，元数据的可用空间有限，这对用户可用的功能施加了以下功能限制：

*   最大网络数减少到 4096。

*   每个网络的最大端口数减少到 2048。

*   不支持针对逻辑入口端口标识符匹配的 ACL。

*   不支持 OVN 互连功能。

除了上述功能限制外，在集群中启用它之前还应考虑以下事项：

*   Geneve 使用随机 UDP 或 TCP 源端口，允许在使用 ECMP 的底层环境中在多条路径之间高效分发。

*   网卡可用于卸载 Geneve 封装和解封装。

由于其灵活性，管理程序之间的首选封装是 Geneve。对于 Geneve 封装，OVN 在 Geneve VNI 中传输逻辑数据路径标识符。OVN 在 TLV 中传输逻辑入口和逻辑出口端口，类为 0x0102，类型为 0x80，32 位值编码如下，从 MSB 到 LSB：

```text
  1       15          16
+---+------------+-----------+
|rsv|ingress port|egress port|
+---+------------+-----------+
  0
```

对于连接到网关，除了 Geneve 之外，OVN 还支持 VXLAN，因为仅支持 VXLAN 在架顶式 (ToR) 交换机上很常见。目前，网关具有与 VTEP 模式定义的相匹配的功能集，因此需要较少的元数据位。将来，不支持具有大量元数据的封装的网关可能会继续具有减少的功能集。

如果仅需要管理程序处的 VXLAN 封装来支持 HW VTEP L2 网关功能，则建议禁用 VXLAN 模式。有关更多详细信息，请参阅 `ovn-nb(5)` 中表 `NB_Global` 列 `options` 键 `vxlan_mode`。
