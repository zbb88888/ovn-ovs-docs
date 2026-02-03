# ovn-controller (Local Control Daemon)

## 简介

`ovn-controller` 是 OVN 的本地控制守护进程，运行在每个参与 OVN 网络的物理节点（Chassis）上。

### Chassis 架构

下图展示了 `ovn-controller` 在 Chassis 上的位置及其与 OVS 的交互：

```text
       +-------------------------+
       |   OVN Southbound DB     | (Central DB)
       +------------+------------+
                    |
              (OVSDB Protocol)
                    |
       +------------v------------+
       |      ovn-controller     | (Local Daemon)
       +---+------------+---- ---+
           |            |
     (OpenFlow)      (OVSDB)
           |            |
       +---v------------v---+
       |   Open vSwitch     | (br-int)
       +--------------------+
```

- **职责**：
    1. 向 Southbound DB 注册 Chassis 信息。
    2. 监听 Southbound DB 中的逻辑流表和绑定信息。
    3. 将逻辑流转换为本地 OVS 的物理 OpenFlow 流表。
    4. 处理隧道封装。
- **位置**：运行在每个 Hypervisor 或 Gateway 节点上。

## 配置

`ovn-controller` 的配置不通过命令行参数传递，而是从本地 OVS 数据库 (`Open_vSwitch` 表的 `external_ids` 列) 中读取。

### 关键配置项

- **ovn-remote**: OVN Southbound DB 的连接地址（如 `tcp:192.168.1.1:6642`）。
- **ovn-encap-type**: 隧道封装类型 (`geneve`, `stt`, `vxlan`)。
- **ovn-encap-ip**: 用于建立隧道的本地 IP 地址。
- **ovn-bridge-mappings**: 物理网桥映射（用于连接物理网络），格式为 `physnet1:br-eth1`。
- **system-id**: Chassis 的唯一标识符（默认为 `/etc/machine-id` 或 `hostname`）。

## 运行时管理 (Runtime Management)

可以通过 `ovn-appctl` 向运行中的 `ovn-controller` 发送命令。

- `ovn-appctl -t ovn-controller exit`: 优雅退出。
- `ovn-appctl -t ovn-controller status`: 查看连接状态和处理进度。
- `ovn-appctl -t ovn-controller inject-pkt`: 注入报文以测试流表。
- `ovn-appctl -t ovn-controller lflow-cache/show`: 查看逻辑流缓存统计。

## 工作流程

`ovn-controller` 将平台无关的**逻辑流**转换为平台相关的**OpenFlow 规则**。转换后的物理流水线结构如下：

```text
   PACKET IN (from VIF or Tunnel)
          |
+---------v---------+
| Stage 0: Ingress  |  (Physical-to-Logical: Map physical port to logical_inport)
+---------+---------+
          |
+---------v---------+
| Stage 1: Logical  |  (Logical Ingress Pipeline: ACLs, L2 Switching, L3 Routing)
| Ingress (0-31)    |  [Maps SB Logical_Flow table_id to OF tables 8-39]
+---------+---------+
          |
+---------v---------+
| Stage 2: Egress   |  (Logical-to-Physical: Map logical_outport to physical port)
+---------+---------+
          |
+---------v---------+
| Stage 3: Logical  |  (Logical Egress Pipeline: ACLs, Port Security)
| Egress (0-31)     |  [Maps SB Logical_Flow table_id to OF tables 40-71]
+---------+---------+
          |
   PACKET OUT (to VIF or Tunnel)
```

1. **连接**: 启动后连接到本地 OVS 和远程 SB DB。
2. **注册**: 将本地配置（IP、隧道类型）写入 SB DB 的 `Chassis` 表。
3. **计算**:
    - 获取 SB DB 中的 `Logical_Flow`。
    - 结合本地 `Port_Binding`（哪些逻辑端口在本地运行）。
    - 结合本地物理端口状态。
    - 计算出对应的 OpenFlow 流表。
4. **下发**: 通过 OpenFlow 协议配置本地 OVS 网桥 (`br-int`)。
