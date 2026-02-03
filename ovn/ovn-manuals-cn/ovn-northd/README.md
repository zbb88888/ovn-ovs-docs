# ovn-northd (Central Control Daemon)

## 简介

`ovn-northd` 是 OVN 的中心控制守护进程。

- **职责**：负责将 OVN Northbound DB (NB) 中的逻辑网络配置转换为 OVN Southbound DB (SB) 中的逻辑流表 (Logical Flows)。
- **位置**：通常与 OVN 数据库运行在同一节点（控制平面）。

## 启动选项

```bash
ovn-northd [options]
```

- `--ovnnb-db=server`: 指定连接到 Northbound DB 的地址（默认为 `unix:/var/run/ovn/ovnnb_db.sock`）。
- `--ovnsb-db=server`: 指定连接到 Southbound DB 的地址（默认为 `unix:/var/run/ovn/ovnsb_db.sock`）。
- `--dry-run`: 试运行模式，验证转换逻辑但不写入数据库。

## 逻辑流生成 (Logical Flow Generation)

`ovn-northd` 的核心工作是维护 SB DB 中的 `Logical_Flow` 表。它根据 NB DB 中的配置，生成对应的 Ingress 和 Egress 流水线。

### 逻辑流生成概览

下图展示了 `ovn-northd` 作为编译器在 NB DB 和 SB DB 之间的作用：

```text
    +-------------------+           +-----------------------+           +-------------------+
    |  Northbound DB    |           |      ovn-northd       |           |  Southbound DB    |
    | (Logical Config)  |           | (The "Compiler")      |           | (Logical Flows)   |
    +---------+---------+           +-----------+-----------+           +---------+---------+
              |                                 |                             |
              |     1. Pulls NB Config          |      3. Pushes Flows        |
              +-------------------------------->|---------------------------->|
              | (LS, LR, ACL, LB, NAT, etc.)    | (Logical_Flow Table)        |
              |                                 |                             |
              |                                 |                             |
              |                                 |      2. Pushes Bindings     |
              |                                 +---------------------------->|
                                                | (Datapath_Binding,          |
                                                |  Port_Binding, etc.)        |
                                                +-----------------------------+
```

### 逻辑交换机流水线 (Logical Switch Pipelines)

`ovn-northd` 为每个逻辑交换机生成入站 (Ingress) 和出站 (Egress) 两条流水线：

```text
       Ingress Pipeline (from-lport)                 Egress Pipeline (to-lport)
    +-------------------------------+             +-------------------------------+
    | 0.  ls_in_port_sec_l2         |             | 0.  ls_out_pre_acl            |
    +-------------------------------+             +-------------------------------+
    | 1.  ls_in_port_sec_ip         |             | 1.  ls_out_pre_lb             |
    +-------------------------------+             +-------------------------------+
    | ...                           |             | ...                           |
    +-------------------------------+             +-------------------------------+
    | 7.  ls_in_acl                 |             | 4.  ls_out_acl                |
    +-------------------------------+             +-------------------------------+
    | 10. ls_in_lb (Load Balancer)  |             | 7.  ls_out_lb                 |
    +-------------------------------+             +-------------------------------+
    | ...                           |             | ...                           |
    +-------------------------------+             +-------------------------------+
    | 21. ls_in_l2_lkup (Switching) |             | 10. ls_out_port_sec_l2        |
    +-------------------------------+             +-------------------------------+
```

关键阶段说明：

1. **Port Security**: 验证源 MAC/IP 是否与端口绑定一致。
2. **ACLs**: 处理入站/出站访问控制列表。
3. **L2 Switching**: 根据目的 MAC 查找输出端口。

### 逻辑路由器流水线 (Logical Router Pipelines)

对于逻辑路由器，生成的流水线如下：

```text
       Ingress Pipeline (from-lport)                 Egress Pipeline (to-lport)
    +-------------------------------+             +-------------------------------+
    | 0.  lr_in_admission           |             | 0.  lr_out_undnat             |
    +-------------------------------+             +-------------------------------+
    | 1.  lr_in_ip_input            |             | 1.  lr_out_post_undnat        |
    +-------------------------------+             +-------------------------------+
    | 2.  lr_in_unsnat              |             | 2.  lr_out_snat (NAT)         |
    +-------------------------------+             +-------------------------------+
    | 3.  lr_in_dnat (DNAT)         |             | 3.  lr_out_egr_loop           |
    +-------------------------------+             +-------------------------------+
    | ...                           |             | ...                           |
    +-------------------------------+             +-------------------------------+
    | 10. lr_in_ip_routing (L3)     |             | 8.  lr_out_delivery           |
    +-------------------------------+             +-------------------------------+
    | 12. lr_in_arp_resolve         |             +-------------------------------+
    +-------------------------------+
```

关键阶段说明：

1. **IP Input**: 处理发送给路由器接口本身的流量（如 ARP 请求、ICMP Echo）。
2. **NAT**: 执行 DNAT/SNAT。
3. **IP Routing**: 查找路由表，确定下一跳。
4. **ARP/ND Resolution**: 解析下一跳的 MAC 地址。

## 高可用性

`ovn-northd` 支持主备模式 (Active-Standby)。

- 当部署多个 `ovn-northd` 实例时，它们通过获取 SB DB 中的一个特定锁 (Lock) 来选举主节点。
- 只有持有锁的主节点才会向 SB DB 写入数据，备节点处于休眠状态。
