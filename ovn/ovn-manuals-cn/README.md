# OVN 官方文档中文梳理库

本仓库包含了对 OVN (Open Virtual Network) 官方手册页的中文梳理和结构化整理。文档依据 OVN 官方提供的 plain text manpages 进行分类和翻译。

## 目录结构

### 1. 核心架构与数据库

- **[ovn-architecture/](./ovn-architecture/)**: OVN 架构总览 (ovn-architecture(7))。包含核心组件交互图、数据包生命周期等。
- **[ovn-nb/](./ovn-nb/)**: 北向数据库 (Northbound DB) 模式。包含逻辑交换机、路由器、ACL、负载均衡等配置。
- **[ovn-sb/](./ovn-sb/)**: 南向数据库 (Southbound DB) 模式。包含 Chassis、逻辑流表 (Logical Flows) 和端口绑定状态。

### 2. 核心守护进程

- **[ovn-northd/](./ovn-northd/)**: 中心控制守护进程，负责 NB 到 SB 的逻辑转换。
- **[ovn-controller/](./ovn-controller/)**: 本地控制守护进程，负责 OpenFlow 下发。

### 3. 命令行工具

- **[ovn-nbctl/](./ovn-nbctl/)**: 北向数据库管理工具。
- **[ovn-sbctl/](./ovn-sbctl/)**: 南向数据库管理工具。
- **[ovn-trace/](./ovn-trace/)**: 逻辑网络追踪工具 (最重要的调试工具)。
- **[ovn-appctl/](./ovn-appctl/)**: 运行时守护进程管理工具。

### 4. 高级功能与辅助

- **[ovn-ic/](./ovn-ic/)**: OVN 互联 (Interconnection) 架构，用于跨可用区 (AZ) 连接。
- **[ovn-helpers/](./ovn-helpers/)**:
  - `ovn-ctl`: 服务启动脚本。
  - `ovn-detrace`: 物理流表反向解析工具。
  - `ovn-debug`: 开发者调试辅助。

## 参考资料

- [OVN Official Documentation](https://www.ovn.org/support/dist-docs/)
