# OVN 文档汉化项目任务追踪

## 项目概览
本项目致力于将 Open Virtual Network (OVN) 的官方手册页 (manpages) 翻译为中文文档。
当前阶段已完成主要组件的初版文档翻译工作，覆盖了核心架构、数据库模式、守护进程及命令行工具。

## 翻译进度概览

| 组件/手册页 | 对应目录 | 状态 | 说明 |
| :--- | :--- | :--- | :--- |
| **ovn-architecture(7)** | `ovn-manuals-cn/ovn-architecture/` | ✅ 已完成 | OVN 架构概览与生命周期 |
| **ovn-nb(5)** | `ovn-manuals-cn/ovn-nb/` | ✅ 已完成 | 北向数据库 (Northbound DB) 模式定义 |
| **ovn-sb(5)** | `ovn-manuals-cn/ovn-sb/` | ✅ 已完成 | 南向数据库 (Southbound DB) 模式定义 |
| **ovn-northd(8)** | `ovn-manuals-cn/ovn-northd/` | ✅ 已完成 | 中央控制守护进程 (ovn-northd) |
| **ovn-controller(8)** | `ovn-manuals-cn/ovn-controller/` | ✅ 已完成 | 宿主机控制守护进程 (ovn-controller) |
| **ovn-nbctl(8)** | `ovn-manuals-cn/ovn-nbctl/` | ✅ 已完成 | 北向数据库命令行工具 |
| **ovn-sbctl(8)** | `ovn-manuals-cn/ovn-sbctl/` | ✅ 已完成 | 南向数据库命令行工具 |
| **ovn-ic(8)** | `ovn-manuals-cn/ovn-ic/` | ✅ 已完成 | OVN 互连 (Interconnection) 守护进程 |
| **ovn-ic-nb(5)** | `ovn-manuals-cn/ovn-ic-nb/` | ✅ 已完成 | 互连北向数据库模式 |
| **ovn-ic-sb(5)** | `ovn-manuals-cn/ovn-ic-sb/` | ✅ 已完成 | 互连南向数据库模式 |
| **ovn-trace(8)** | `ovn-manuals-cn/ovn-trace/` | ✅ 已完成 | 逻辑流数据包追踪工具 |
| **ovn-appctl(8)** | `ovn-manuals-cn/ovn-appctl/` | ✅ 已完成 | 守护进程运行时管理工具 |
| **ovn-ctl(8)** | `ovn-manuals-cn/ovn-helpers/` | ✅ 已完成 | 服务启动脚本 (ovn-ctl) |
| **ovn-detrace(1)** | `ovn-manuals-cn/ovn-helpers/` | ✅ 已完成 | Cookie 堆栈转换工具 (ovn-detrace) |

## 详细文件结构

### 核心守护进程
#### ovn-northd (`ovn-manuals-cn/ovn-northd/`)
*最新完整版：*
- `01-Overview-and-Options.md`: 概述、选项与管理命令
- `02-Logical-Switch-Ingress.md`: 逻辑交换机 Ingress 流表详解
- `03-Logical-Switch-Egress.md`: 逻辑交换机 Egress 流表详解
- `04-Logical-Router-Ingress.md`: 逻辑路由器 Ingress 流表详解
- `05-Logical-Router-Egress.md`: 逻辑路由器 Egress 流表详解
*(注：该目录下可能存在部分旧版本文件，建议以编号 01-05 的上述文件为准)*

#### ovn-controller (`ovn-manuals-cn/ovn-controller/`)
- `01-overview-and-options.md`: 概述与选项
- `02-configuration.md`: 配置详情
- `03-database-usage.md`: 数据库使用
- `04-runtime-commands.md`: 运行时命令

### 数据库模式
#### ovn-nb (`ovn-manuals-cn/ovn-nb/`)
- `01-Global-Config.md`: 全局配置 (NB_Global, DHCP, DNS 等)
- `02-Logical-Switch.md`: 逻辑交换机相关表
- `03-Logical-Router.md`: 逻辑路由器相关表
- `04-Security-Policies.md`: 安全策略 (ACL, QoS 等)
- `05-Load-Balancer.md`: 负载均衡相关表
- `06-Services.md`: 其他服务表
- `07-HA-Gateway.md`: HA 网关相关表
- `08-Network-Functions.md`: 网络功能相关表

#### ovn-sb (`ovn-manuals-cn/ovn-sb/`)
- `01-System-Physical.md`: 物理系统与底盘 (Chassis)
- `02-Logical-Network.md`: 逻辑网络数据路径
- `03-Bindings.md`: 端口绑定与组播
- `04-Services.md`: MAC 绑定、DHCP 等服务
- `05-Connection-Security.md`: 连接与安全
- `06-HA-Gateway.md`: HA 网关
- `07-Load-Balancer.md`: 负载均衡
- `08-Routing-Others.md`: 路由与杂项

### OVN 互连 (Interconnection)
#### ovn-ic (`ovn-manuals-cn/ovn-ic/`)
- `01-overview-options.md`
- `02-runtime-management.md`

#### ovn-ic-nb (`ovn-manuals-cn/ovn-ic-nb/`)
- `01-IC-NB-DB.md`: 互连北向数据库全表详解

#### ovn-ic-sb (`ovn-manuals-cn/ovn-ic-sb/`)
- `01-IC-SB-DB.md`: 互连南向数据库全表详解

### 命令行工具
#### ovn-nbctl (`ovn-manuals-cn/ovn-nbctl/`)
- `01-overview-options.md`
- `02-general-and-switch.md`
- `03-acl-and-qos.md`
- `04-logical-switch-ports.md`
- `05-logical-router.md`
- `06-load-balancer.md`
- `07-dhcp-and-others.md`
- `08-database-commands.md`

#### ovn-sbctl (`ovn-manuals-cn/ovn-sbctl/`)
- `01-overview.md`
- `02-chassis-and-ports.md`
- `03-logical-flows.md`
- `04-connection-ssl-database.md`

#### ovn-trace (`ovn-manuals-cn/ovn-trace/`)
- `01-overview-options.md`
- `02-usage-and-output.md`

#### ovn-appctl (`ovn-manuals-cn/ovn-appctl/`)
- `01-ovn-appctl.md`: 通用命令与日志管理

### 辅助工具 (`ovn-manuals-cn/ovn-helpers/`)
- `01-ovn-ctl.md`: ovn-ctl 脚本说明
- `02-ovn-detrace.md`: ovn-detrace 工具说明

## 后续规划
- [ ] 文档目录结构标准化与重组
- [ ] 术语一致性校对与修订
- [ ] 补充缺失的边缘功能文档
- [ ] 生成静态站点 (如 MkDocs)
