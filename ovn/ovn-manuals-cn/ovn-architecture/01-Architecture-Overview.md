# OVN 架构概览 (Architecture Overview)

## 简介

OVN (Open Virtual Network) 是一个为虚拟机和容器环境提供逻辑网络抽象的系统。它补充了 Open vSwitch (OVS) 的现有功能，增加了对逻辑 L2 和 L3 覆盖网络（Overlays）、安全组（Security Groups）等功能的原生支持。

OVN 的设计目标是提供一个生产级的实现，能够在大规模环境下运行。

## 物理网络与逻辑网络

- **物理网络**：由物理线缆、交换机和路由器组成。
- **虚拟网络**：将物理网络扩展到 Hypervisor 或容器平台，将 VM 或容器桥接到物理网络。
- **OVN 逻辑网络**：在软件中实现的网络，通过隧道或其他封装技术与物理网络隔离。这允许逻辑网络使用与物理网络重叠的 IP 地址空间而不发生冲突。逻辑网络拓扑可以独立于物理网络拓扑进行安排。

## 核心组件

下图展示了 OVN 的主要组件及其交互关系（参考自 ovn-architecture(7)）：

```text
                                         CMS
                                          |
                                          |
                              +-----------v-----------+
                              |                       |
                              |  OVN Northbound DB    |
                              |                       |
                              +-----------+-----------+
                                          |
                                          |
                                   ovn-northd
                                          |
                                          |
                              +-----------v-----------+
                              |                       |
                              |  OVN Southbound DB    |
                              |                       |
                              +-------+-----------+---+
                                      |           |
                                      |           |
                               ovn-controller   ovn-controller
                                      |           |
                                      |           |
                                     OVS         OVS
```

OVN 部署由以下几个主要组件组成：

### 1. 云管理系统 (CMS)

- OVN 的最终客户端（通过用户和管理员）。
- 需要安装 CMS 特定的插件（如 OpenStack Neutron 插件或 Kubernetes CNI 插件）。
- CMS 负责将用户定义的逻辑网络配置转换为 OVN 理解的中间表示。

### 2. OVN Northbound Database (NB DB)

- 接收来自 CMS 的逻辑网络配置。
- 数据库模式（Schema）旨在与 CMS 的概念“阻抗匹配”，直接反映逻辑交换机、路由器、ACL 等概念。

### 3. ovn-northd

- 连接 NB DB 和 Southbound DB 的守护进程。
- 负责将 NB DB 中的逻辑网络配置翻译成 SB DB 中的逻辑数据路径流表（Logical Datapath Flows）。

### 4. OVN Southbound Database (SB DB)

- 系统的中心数据交换点。
- 包含三类数据：
  - **物理网络数据**：如 Chassis（底盘/节点）信息，由 ovn-controller 填充。
  - **逻辑网络数据**：由 ovn-northd 从 NB DB 生成。
  - **绑定数据**：物理位置与逻辑组件的映射关系。

### 5. ovn-controller

- 运行在每个 Hypervisor 和 Gateway 上的守护进程。
- 是 OVN 在物理节点上的代理。
- 向 SB DB 注册 Chassis 信息。
- 监听 SB DB 中的逻辑流表，并将其转换为本地 OVS 的 OpenFlow 流表。
- 负责处理 logical port 到 physical port 的绑定。

## 信息流

1. **CMS** 更新 **NB DB**（例如创建逻辑交换机）。
2. **ovn-northd** 监听到 **NB DB** 的变化，将其转换为逻辑流，写入 **SB DB**。
3. **ovn-controller** 监听到 **SB DB** 的变化，将相关的逻辑流转换为物理流表（OpenFlow），配置本地 **OVS**。
4. **ovn-controller** 还会将本地物理状态（如 VM 启动位置）更新到 **SB DB**。
