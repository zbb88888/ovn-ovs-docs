# 物理基础设施 (Physical Infrastructure)

## Chassis 表

每个记录代表一个物理节点（Hypervisor 或 Gateway）。此表通常由 `ovn-controller` 在启动时自动注册和维护。

### 核心列

- **name**: Chassis 的唯一标识符（通常是主机名或 UUID）。
- **hostname**: 主机名。
- **nb_cfg**: 该 Chassis 已处理到的配置版本号。
- **encaps**: 指向 `Encap` 表的引用列表，定义该节点支持的隧道类型。
- **vtep_logical_switches**: 关联的 VTEP 逻辑交换机（用于硬件网关）。
- **other_config**:
  - `datapath-type`: 数据路径类型（如 `system` 或 `netdev`）。
  - `is-interconn`: 是否为互联网关。

## Encap 表

定义 Chassis 之间通信的隧道封装。

### 核心列

- **type**: 隧道协议类型。
  - `geneve`: (推荐) 通用网络虚拟化封装，支持携带元数据。
  - `stt`: 无状态传输隧道。
  - `vxlan`: 仅用于与非 OVN 设备互通。
- **ip**: 隧道端点 IP 地址。
- **options**: 协议特定选项（如 `csum` 校验和）。

## Chassis_Private 表

存储每个 Chassis 的私有状态，这些信息不需要广播给集群中的其他 Chassis。这有助于减少不必要的数据库流量。

### 核心列

- **name**: 对应 Chassis 的名称。
- **chassis**: 指向 `Chassis` 表的引用。
- **nb_cfg**: 私有的配置版本进度。
- **nb_cfg_timestamp**: 配置应用的时间戳。
