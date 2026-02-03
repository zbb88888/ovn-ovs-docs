# 数据包生命周期 (Life Cycle of a Packet)

本节描述了一个数据包如何从一个虚拟机（VM A）通过 OVN 逻辑网络传输到另一个虚拟机（VM B）。

以下是该过程的 ASCII 流程图（参考自 ovn-architecture(7)）：

```text
       Source Hypervisor (Chassis A)          Physical Network          Destination Hypervisor (Chassis B)
    +---------------------------------+      +----------------+      +------------------------------------+
    |      VM A (Source Port)         |      |                |      |        VM B (Destination Port)     |
    +--------------+------------------+      |                |      +-----------------+------------------+
                   |                         |                |                        ^
                   v                         |                |                        |
    +--------------+------------------+      |                |      +-----------------+------------------+
    |  OVS Port (Physical-to-Logical) |      |                |      |  OVS Port (Logical-to-Physical)    |
    |  - Sets logical_inport          |      |                |      |  - Final output to VM B            |
    +--------------+------------------+      |                |      +-----------------+------------------+
                   |                         |                |                        ^
                   v                         |                |                        |
    +--------------+------------------+      |                |      +-----------------+------------------+
    |    Ingress Pipeline             |      |                |      |      Egress Pipeline               |
    |  - L2/L3 Processing             |      |                |      |  - Egress ACLs                     |
    |  - ACLs (from-lport)            |      |                |      |  - Port Security                   |
    |  - Sets logical_outport         |      |                |      |                                    |
    +--------------+------------------+      |                |      +-----------------+------------------+
                   |                         |                |                        ^
                   v                         |                |                        |
    +--------------+------------------+      |                |      +-----------------+------------------+
    |    Output Action                |      |                |      |      Decapsulation                 |
    |  - Find Destination Chassis     |      |                |      |  - Restore Logical Metadata        |
    |  - Encapsulate (Geneve/STT)     +----->|   UDP/IP/Eth   +----->|    (Inport, Outport, Datapath)     |
    +---------------------------------+      +----------------+      +------------------------------------+
```

## 1. 物理到逻辑 (Ingress)

1. **VM A 发送数据包**: 数据包到达源 Hypervisor 的 OVS 网桥。
2. **分类与元数据**: OVS 根据入端口（Port ID）识别该包属于哪个逻辑端口，并为其打上 `logical_inport` 和 `logical_datapath` (逻辑交换机 ID) 的元数据。
3. **Ingress Pipeline (逻辑入站流水线)**:
    - `ovn-controller` 生成的 OpenFlow 表会根据 SB DB 中的逻辑流处理该包。
    - **L2 查找**: 根据目的 MAC 地址查找 `logical_outport`。
    - **ACL 处理**: 检查入站安全策略。
    - **路由决策**: 如果是跨网段流量，在此阶段进行 L3 路由，更新源/目的 MAC，并将 `logical_datapath` 更新为目标逻辑交换机。

## 2. 逻辑传输 (Transport)

1. **输出决策**: 确定 `logical_outport` 后，OVS 查找 Port Binding 表，得知目标端口位于哪个 Chassis（假设是 Chassis B）。
2. **封装**: 源 Chassis 将逻辑元数据（Logical Datapath ID, Logical Inport, Logical Outport）编码到隧道头（如 Geneve）中。
3. **物理传输**: 封装后的数据包通过物理网络发送到 Chassis B 的 IP 地址。

## 3. 逻辑到物理 (Egress)

1. **解封装**: Chassis B 接收到隧道包，解封装并提取逻辑元数据。
2. **Egress Pipeline (逻辑出站流水线)**:
    - 根据逻辑元数据继续处理。
    - **ACL 处理**: 检查出站安全策略（在目标端检查）。
3. **物理输出**: 最终，OVS 将数据包发送到对应 VM B 的本地接口。

## 关键点

- **分布式处理**: 所有的逻辑决策（交换、路由、源端 ACL）尽量在源节点（Local Chassis）完成。
- **隧道携带状态**: 隧道不仅仅传输数据，还传输了逻辑上下文，使得目标节点知道如何处理该包。
- **本地交换**: 如果 VM A 和 VM B 在同一个 Chassis，数据包不会经过物理网卡，直接在内存中通过 OVS 转发，但逻辑流水线处理步骤完全相同。
